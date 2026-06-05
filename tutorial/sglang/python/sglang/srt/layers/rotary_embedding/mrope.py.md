# mrope.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/rotary_embedding/mrope.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements rotary position embedding logic with embedding lookup and preprocessing for the SGLang SRT runtime. It exposes symbols such as `apply_interleaved_rope`, `MRotaryEmbedding`, `YaRNScalingMRotaryEmbedding`, and `Ernie4_5_VLRotaryEmbedding` and connects them to backend-specific paths such as `CUDA`, `Triton`, `NPU`, and `XPU`. / 该模块为 SGLang 的 SRT 运行时实现了旋转位置编码逻辑，并结合嵌入查找与预处理。它提供了 `apply_interleaved_rope`、`MRotaryEmbedding`、`YaRNScalingMRotaryEmbedding` 以及 `Ernie4_5_VLRotaryEmbedding` 等符号，并把这些符号连接到 `CUDA`、`Triton`、`NPU` 以及 `XPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-34: Imports, conditional backend setup, and runtime guards
```python
"""MRotaryEmbedding, YaRNScalingMRotaryEmbedding, Ernie4_5_VLRotaryEmbedding,
apply_interleaved_rope for multimodal RoPE."""

from __future__ import annotations

from typing import List, Optional, Tuple

import torch

from sglang.srt.layers.rotary_embedding.base import RotaryEmbedding
from sglang.srt.layers.rotary_embedding.triton_kernels import (
    triton_ernie45_rope_fused_inplace,
    triton_mrope_fused,
)
from sglang.srt.layers.rotary_embedding.utils import apply_rotary_emb
from sglang.srt.layers.rotary_embedding.yarn import (
    yarn_find_correction_range,
    yarn_get_mscale_simple,
    yarn_linear_ramp_mask,
)
from sglang.srt.server_args import get_global_server_args
from sglang.srt.utils import cpu_has_amx_support, is_cuda, is_npu

_is_cuda = is_cuda()
_is_npu = is_npu()
_is_cpu_amx_available = cpu_has_amx_support()

if _is_cuda:
    from sglang.jit_kernel.rope import apply_rope_with_cos_sin_cache_inplace

if _is_npu:
    import torch_npu
```
**EN:** This section prepares the module namespace. It imports `__future__.annotations`, `typing.List`, `typing.Optional`, `typing.Tuple`, `torch`, and `sglang.srt.layers.rotary_embedding.base.RotaryEmbedding`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `_is_cuda`, `_is_npu`, and `_is_cpu_amx_available` capture configuration, cached handles, or feature flags. Control structures like `If` indicate compatibility branches or backend-specific setup.
**CN:** 该部分负责准备模块命名空间。 它导入了 `__future__.annotations`、`typing.List`、`typing.Optional`、`typing.Tuple`、`torch` 以及 `sglang.srt.layers.rotary_embedding.base.RotaryEmbedding`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `_is_cuda`、`_is_npu` 以及 `_is_cpu_amx_available` 这样的共享名称用于保存配置、缓存句柄或特性开关。 `If` 等控制结构说明这里还承担兼容分支或后端专用初始化。

### Lines 35-41: `apply_interleaved_rope` apply step for interleaved RoPE
```python
def apply_interleaved_rope(x: torch.Tensor, mrope_section: list) -> torch.Tensor:
    x_t = x[0].clone()
    x_t[..., 1 : mrope_section[1] * 3 : 3] = x[1, ..., 1 : mrope_section[1] * 3 : 3]
    x_t[..., 2 : mrope_section[2] * 3 : 3] = x[2, ..., 2 : mrope_section[2] * 3 : 3]
    return x_t
```
**EN:** This block defines `apply_interleaved_rope` and contains the main logic for this step. It mainly invokes `x.clone`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `x_t` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `apply_interleaved_rope`，并承载这一阶段的核心逻辑。 它主要调用 `x.clone`，说明该流程会编排底层辅助函数或计算内核。 像 `x_t` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 42-44: Class `MRotaryEmbedding` declaration and shared state
```python
class MRotaryEmbedding(RotaryEmbedding):
    """Rotary Embedding with Multimodal Sections."""
```
**EN:** This block introduces class `MRotaryEmbedding` and the state shared by its methods. It inherits from `RotaryEmbedding`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Rotary Embedding with Multimodal Sections.
**CN:** 该代码块引入类 `MRotaryEmbedding`，并定义其方法共享的状态。 它继承自 `RotaryEmbedding`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 45-124: `MRotaryEmbedding` initialization and state setup
```python
    def __init__(
        self,
        head_size: int,
        rotary_dim: int,
        max_position_embeddings: int,
        base: int,
        is_neox_style: bool,
        dtype: torch.dtype,
        mrope_section: Optional[List[int]] = None,
        mrope_interleaved: bool = False,
        mrope_interleaved_glm: bool = False,
    ) -> None:
        super().__init__(
            head_size, rotary_dim, max_position_embeddings, base, is_neox_style, dtype
        )
        self.mrope_section = mrope_section
        self.mrope_interleaved = mrope_interleaved
        self.mrope_interleaved_glm = mrope_interleaved_glm
        if self.mrope_section:
            expected_sum = rotary_dim // 2
            actual_sum = sum(self.mrope_section)
            if actual_sum != expected_sum:
                print(
                    f"MRoPE section sum mismatch: expected {expected_sum}, got {actual_sum}. "
                    f"Adjusting mrope_section to match rotary_dim // 2 = {expected_sum}"
                )
                if actual_sum > 0:
                    scale_factor = expected_sum / actual_sum
                    self.mrope_section = [
                        max(1, int(section * scale_factor))
                        for section in self.mrope_section
                    ]
                    current_sum = sum(self.mrope_section)
                    if current_sum != expected_sum:
                        self.mrope_section[-1] += expected_sum - current_sum
                else:
                    self.mrope_section = [
                        expected_sum // len(self.mrope_section)
                    ] * len(self.mrope_section)
                    remainder = expected_sum % len(self.mrope_section)
                    for i in range(remainder):
                        self.mrope_section[i] += 1
                print(
                    f"Corrected mrope_section: {self.mrope_section} (sum={sum(self.mrope_section)})"
                )

        # MRoPE axis_map interleaving pattern depends on mrope_section sizes.
        # The algorithm cycles through axes [0(T), 1(H), 2(W)] round-robin,
        # skipping any axis that has exhausted its allocated pairs.
        #
        # For GLM-V (mrope_section=[8,12,12]):
        #   T(8) < H(12) = W(12), so T exhausts first at pair 24.
        #   Result: [0,1,2, 0,1,2, 0,1,2, 0,1,2, 0,1,2, 0,1,2, 0,1,2, 0,1,2, 1,1,2, 1,1,2, 2,2]
        #   After T runs out, only H and W fill the remaining slots.
        #
        # For Qwen3-VL (mrope_section=[24,20,20]):
        #   T(24) > H(20) = W(20), so H and W exhaust first near the tail.
        #   Result: [0,1,2, 0,1,2, ...repeated evenly..., 0,1, 0,1, 0,0]
        #   After H/W run out, T fills the remaining slots.

        if self.mrope_interleaved_glm:
            num_pairs = rotary_dim // 2
            axis_map = torch.empty(num_pairs, dtype=torch.long)
            assert sum(self.mrope_section) == num_pairs
            counts = [0, 0, 0]
            current_ax = 0

            for i in range(num_pairs):
                current_ax = i % 3
                while counts[current_ax] >= self.mrope_section[current_ax]:
                    current_ax = (current_ax + 1) % 3

                axis_map[i] = current_ax
                counts[current_ax] += 1
            self.register_buffer("axis_map", axis_map, persistent=False)
        else:
            self.axis_map = None
        if get_global_server_args().rl_on_policy_target is not None:
            self._forward_method = self.forward_native
```
**EN:** This block defines `MRotaryEmbedding.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, `sum`, `torch.empty`, `range`, and `self.register_buffer`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.mrope_section`, `self.mrope_interleaved`, `self.mrope_interleaved_glm`, `expected_sum`, and `actual_sum` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `MRotaryEmbedding.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`、`sum`、`torch.empty`、`range` 以及 `self.register_buffer`，说明该流程会编排底层辅助函数或计算内核。 像 `self.mrope_section`、`self.mrope_interleaved`、`self.mrope_interleaved_glm`、`expected_sum` 以及 `actual_sum` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 125-147: `MRotaryEmbedding.get_cos_sin_with_position` getter for cos sin with position
```python
    def get_cos_sin_with_position(self, positions):
        if positions.ndim == 1:
            return super().get_cos_sin_with_position(positions)
        assert positions.ndim == 2
        assert self.mrope_section
        cos_sin = self.cos_sin_cache[positions]
        last_dim = cos_sin.size()[-1]
        cos, sin = cos_sin.chunk(2, dim=-1)
        if self.mrope_interleaved:
            cos = apply_interleaved_rope(cos, self.mrope_section)
            sin = apply_interleaved_rope(sin, self.mrope_section)
        else:
            cos = torch.cat(
                [m[i] for i, m in enumerate(cos.split(self.mrope_section, dim=-1))],
                dim=-1,
            )
            sin = torch.cat(
                [m[i] for i, m in enumerate(sin.split(self.mrope_section, dim=-1))],
                dim=-1,
            )
        self.position_cos = cos.repeat(1, 2).view(-1, 1, 1, last_dim).contiguous()
        self.position_sin = sin.repeat(1, 2).view(-1, 1, 1, last_dim).contiguous()
```
**EN:** This block defines `MRotaryEmbedding.get_cos_sin_with_position` and contains the main logic for this step. It mainly invokes `cos_sin.chunk`, `cos.repeat.view.contiguous`, `sin.repeat.view.contiguous`, `super.get_cos_sin_with_position`, and `cos_sin.size`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `cos_sin`, `last_dim`, `cos`, `sin`, and `self.position_cos` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MRotaryEmbedding.get_cos_sin_with_position`，并承载这一阶段的核心逻辑。 它主要调用 `cos_sin.chunk`、`cos.repeat.view.contiguous`、`sin.repeat.view.contiguous`、`super.get_cos_sin_with_position` 以及 `cos_sin.size`，说明该流程会编排底层辅助函数或计算内核。 像 `cos_sin`、`last_dim`、`cos`、`sin` 以及 `self.position_cos` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 148-154: Internal helper `MRotaryEmbedding._match_cos_sin_cache_dtype`
```python
    def _match_cos_sin_cache_dtype(self, query: torch.Tensor) -> None:
        if (
            self.cos_sin_cache.device != query.device
            or self.cos_sin_cache.dtype != query.dtype
        ):
            self.cos_sin_cache = self.cos_sin_cache.to(query.device, dtype=query.dtype)
```
**EN:** This block defines `MRotaryEmbedding._match_cos_sin_cache_dtype` and contains the main logic for this step. It mainly invokes `self.cos_sin_cache.to`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.cos_sin_cache` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `MRotaryEmbedding._match_cos_sin_cache_dtype`，并承载这一阶段的核心逻辑。 它主要调用 `self.cos_sin_cache.to`，说明该流程会编排底层辅助函数或计算内核。 像 `self.cos_sin_cache` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 155-200: `MRotaryEmbedding.forward_native` NATIVE execution path
```python
    def forward_native(
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor,
        fused_set_kv_buffer_arg=None,
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        assert (
            fused_set_kv_buffer_arg is None
        ), "save kv cache is not supported for MRotaryEmbedding."
        assert positions.ndim == 1 or positions.ndim == 2

        cos_sin = self.cos_sin_cache[positions]
        cos, sin = cos_sin.chunk(2, dim=-1)
        if positions.ndim == 2:
            assert self.mrope_section
            if self.mrope_interleaved:
                cos = apply_interleaved_rope(cos, self.mrope_section)
                sin = apply_interleaved_rope(sin, self.mrope_section)
            else:
                cos = torch.cat(
                    [m[i] for i, m in enumerate(cos.split(self.mrope_section, dim=-1))],
                    dim=-1,
                )
                sin = torch.cat(
                    [m[i] for i, m in enumerate(sin.split(self.mrope_section, dim=-1))],
                    dim=-1,
                )

        seq_len_q = query.shape[0]
        query_shape = query.shape
        query = query.view(seq_len_q, -1, self.head_size)
        query_rot = query[..., : self.rotary_dim]
        query_pass = query[..., self.rotary_dim :]
        query_rot = apply_rotary_emb(query_rot, cos, sin, self.is_neox_style)
        query = torch.cat((query_rot, query_pass), dim=-1).reshape(query_shape)

        seq_len_k = key.shape[0]
        key_shape = key.shape
        key = key.view(seq_len_k, -1, self.head_size)
        key_rot = key[..., : self.rotary_dim]
        key_pass = key[..., self.rotary_dim :]
        key_rot = apply_rotary_emb(key_rot, cos, sin, self.is_neox_style)
        key = torch.cat((key_rot, key_pass), dim=-1).reshape(key_shape)
        return query, key
```
**EN:** This block defines `MRotaryEmbedding.forward_native` and contains the main logic for this step. It mainly invokes `cos_sin.chunk`, `query.view`, `apply_rotary_emb`, `torch.cat.reshape`, and `key.view`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `cos_sin`, `cos`, `sin`, `seq_len_q`, and `query_shape` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MRotaryEmbedding.forward_native`，并承载这一阶段的核心逻辑。 它主要调用 `cos_sin.chunk`、`query.view`、`apply_rotary_emb`、`torch.cat.reshape` 以及 `key.view`，说明该流程会编排底层辅助函数或计算内核。 像 `cos_sin`、`cos`、`sin`、`seq_len_q` 以及 `query_shape` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 201-220: `MRotaryEmbedding.forward_cpu` CPU execution path
```python
    def forward_cpu(
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor,
        fused_set_kv_buffer_arg=None,
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        if _is_cpu_amx_available:
            return torch.ops.sgl_kernel.multimodal_rotary_embedding_cpu(
                positions,
                query,
                key,
                self.head_size,
                self.cos_sin_cache,
                self.mrope_section if self.mrope_section else None,
                self.mrope_interleaved,
                self.is_neox_style,
            )
        return self.forward_native(positions, query, key, fused_set_kv_buffer_arg)
```
**EN:** This block defines `MRotaryEmbedding.forward_cpu` and contains the main logic for this step. It mainly invokes `self.forward_native` and `torch.ops.sgl_kernel.multimodal_rotary_embedding_cpu`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MRotaryEmbedding.forward_cpu`，并承载这一阶段的核心逻辑。 它主要调用 `self.forward_native` 和 `torch.ops.sgl_kernel.multimodal_rotary_embedding_cpu`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 221-232: `MRotaryEmbedding.forward_cuda` CUDA execution path
```python
    def forward_cuda(
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor,
        fused_set_kv_buffer_arg=None,
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        assert positions.ndim == 1 or positions.ndim == 2
        if positions.ndim == 2 and self.mrope_section:
            return self.forward_triton(positions, query, key)
        return self.forward_native(positions, query, key, fused_set_kv_buffer_arg)
```
**EN:** This block defines `MRotaryEmbedding.forward_cuda` and contains the main logic for this step. It mainly invokes `self.forward_native` and `self.forward_triton`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MRotaryEmbedding.forward_cuda`，并承载这一阶段的核心逻辑。 它主要调用 `self.forward_native` 和 `self.forward_triton`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 233-255: `MRotaryEmbedding.forward_triton` TRITON execution path
```python
    def forward_triton(
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor,
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        assert self.mrope_section
        self._match_cos_sin_cache_dtype(query)
        triton_mrope_fused(
            query,
            key,
            self.cos_sin_cache,
            positions,
            self.mrope_section,
            self.head_size,
            self.rotary_dim,
            self.mrope_interleaved,
            self.mrope_interleaved_glm,
            self.is_neox_style,
            self.axis_map,
        )
        return query, key
```
**EN:** This block defines `MRotaryEmbedding.forward_triton` and contains the main logic for this step. It mainly invokes `self._match_cos_sin_cache_dtype` and `triton_mrope_fused`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MRotaryEmbedding.forward_triton`，并承载这一阶段的核心逻辑。 它主要调用 `self._match_cos_sin_cache_dtype` 和 `triton_mrope_fused`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 256-280: `MRotaryEmbedding.forward_npu` NPU execution path
```python
    def forward_npu(
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor,
        fused_set_kv_buffer_arg=None,
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        assert (
            fused_set_kv_buffer_arg is None
        ), "fused_set_kv_buffer_arg is not supported for npu implementation"
        if query.shape[1] > 4096:
            return self.forward_native(positions, query, key, fused_set_kv_buffer_arg)
        rotary_mode = "half" if self.is_neox_style else "interleave"
        mrope_section = [0, 0, 0]
        query_out, key_out = torch_npu.npu_mrope(
            positions,
            query,
            key,
            self.cos_sin_cache,
            self.head_size,
            mrope_section=mrope_section,
            rotary_mode=rotary_mode,
        )
        return query_out, key_out
```
**EN:** This block defines `MRotaryEmbedding.forward_npu` and contains the main logic for this step. It mainly invokes `torch_npu.npu_mrope` and `self.forward_native`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `rotary_mode`, `mrope_section`, `query_out`, and `key_out` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MRotaryEmbedding.forward_npu`，并承载这一阶段的核心逻辑。 它主要调用 `torch_npu.npu_mrope` 和 `self.forward_native`，说明该流程会编排底层辅助函数或计算内核。 像 `rotary_mode`、`mrope_section`、`query_out` 以及 `key_out` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 281-292: `MRotaryEmbedding.forward_xpu` XPU execution path
```python
    def forward_xpu(
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor,
        fused_set_kv_buffer_arg=None,
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        assert positions.ndim in (1, 2)
        if positions.ndim == 2 and self.mrope_section:
            return self.forward_triton(positions, query, key)
        return self.forward_native(positions, query, key, fused_set_kv_buffer_arg)
```
**EN:** This block defines `MRotaryEmbedding.forward_xpu` and contains the main logic for this step. It mainly invokes `self.forward_native` and `self.forward_triton`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MRotaryEmbedding.forward_xpu`，并承载这一阶段的核心逻辑。 它主要调用 `self.forward_native` 和 `self.forward_triton`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 293-322: `MRotaryEmbedding.get_rope_index` getter for RoPE index
```python
    @staticmethod
    def get_rope_index(
        spatial_merge_size,
        image_token_id,
        video_token_id,
        vision_start_token_id,
        model_type,
        tokens_per_second=None,
        input_ids=None,
        image_grid_thw=None,
        video_grid_thw=None,
        second_per_grid_ts=None,
        **kwargs,
    ):
        from sglang.srt.layers.rotary_embedding.mrope_rope_index import get_rope_index

        return get_rope_index(
            spatial_merge_size,
            image_token_id,
            video_token_id,
            vision_start_token_id,
            model_type,
            tokens_per_second,
            input_ids,
            image_grid_thw,
            video_grid_thw,
            second_per_grid_ts,
            **kwargs,
        )
```
**EN:** This block defines `MRotaryEmbedding.get_rope_index` and contains the main logic for this step. Decorators like `staticmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `get_rope_index`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MRotaryEmbedding.get_rope_index`，并承载这一阶段的核心逻辑。 像 `staticmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `get_rope_index`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 323-352: `MRotaryEmbedding.get_rope_index_qwen3_omni` getter for RoPE index qwen 3 omni
```python
    @staticmethod
    def get_rope_index_qwen3_omni(
        spatial_merge_size,
        image_token_id,
        video_token_id,
        vision_start_token_id,
        tokens_per_second=None,
        input_ids=None,
        image_grid_thw=None,
        video_grid_thw=None,
        second_per_grid_ts=None,
        **kwargs,
    ):
        from sglang.srt.layers.rotary_embedding.mrope_rope_index import (
            get_rope_index_qwen3_omni,
        )

        return get_rope_index_qwen3_omni(
            spatial_merge_size,
            image_token_id,
            video_token_id,
            vision_start_token_id,
            tokens_per_second,
            input_ids,
            image_grid_thw,
            video_grid_thw,
            second_per_grid_ts,
            **kwargs,
        )
```
**EN:** This block defines `MRotaryEmbedding.get_rope_index_qwen3_omni` and contains the main logic for this step. Decorators like `staticmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `get_rope_index_qwen3_omni`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MRotaryEmbedding.get_rope_index_qwen3_omni`，并承载这一阶段的核心逻辑。 像 `staticmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `get_rope_index_qwen3_omni`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 353-369: `MRotaryEmbedding.get_rope_index_glm4v` getter for RoPE index glm 4 v
```python
    @staticmethod
    def get_rope_index_glm4v(
        input_ids, hf_config, image_grid_thw, video_grid_thw, attention_mask, **kwargs
    ):
        from sglang.srt.layers.rotary_embedding.mrope_rope_index import (
            get_rope_index_glm4v,
        )

        return get_rope_index_glm4v(
            input_ids,
            hf_config,
            image_grid_thw,
            video_grid_thw,
            attention_mask,
            **kwargs,
        )
```
**EN:** This block defines `MRotaryEmbedding.get_rope_index_glm4v` and contains the main logic for this step. Decorators like `staticmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `get_rope_index_glm4v`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MRotaryEmbedding.get_rope_index_glm4v`，并承载这一阶段的核心逻辑。 像 `staticmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `get_rope_index_glm4v`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 370-382: `MRotaryEmbedding.get_rope_index_ernie45` getter for RoPE index ernie 45
```python
    @staticmethod
    def get_rope_index_ernie45(
        input_ids, hf_config, image_grid_thw, video_grid_thw, **kwargs
    ):
        from sglang.srt.layers.rotary_embedding.mrope_rope_index import (
            get_rope_index_ernie45,
        )

        return get_rope_index_ernie45(
            input_ids, hf_config, image_grid_thw, video_grid_thw, **kwargs
        )
```
**EN:** This block defines `MRotaryEmbedding.get_rope_index_ernie45` and contains the main logic for this step. Decorators like `staticmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `get_rope_index_ernie45`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MRotaryEmbedding.get_rope_index_ernie45`，并承载这一阶段的核心逻辑。 像 `staticmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `get_rope_index_ernie45`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 383-385: Class `YaRNScalingMRotaryEmbedding` declaration and shared state
```python
class YaRNScalingMRotaryEmbedding(MRotaryEmbedding):
    """MRoPE-enabled rotary embedding with YaRN context scaling."""
```
**EN:** This block introduces class `YaRNScalingMRotaryEmbedding` and the state shared by its methods. It inherits from `MRotaryEmbedding`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: MRoPE-enabled rotary embedding with YaRN context scaling.
**CN:** 该代码块引入类 `YaRNScalingMRotaryEmbedding`，并定义其方法共享的状态。 它继承自 `MRotaryEmbedding`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 386-421: `YaRNScalingMRotaryEmbedding` initialization and state setup
```python
    def __init__(
        self,
        head_size: int,
        rotary_dim: int,
        max_position_embeddings: int,
        base: int,
        is_neox_style: bool,
        scaling_factor: float,
        dtype: torch.dtype,
        *,
        mrope_section: Optional[List[int]] = None,
        mrope_interleaved: bool = False,
        extrapolation_factor: float = 1,
        attn_factor: float = 1,
        beta_fast: int = 32,
        beta_slow: int = 1,
        truncate: bool = True,
    ) -> None:
        self.scaling_factor = scaling_factor
        self.extrapolation_factor = extrapolation_factor
        self.attn_factor = attn_factor
        self.beta_fast = beta_fast
        self.beta_slow = beta_slow
        self.truncate = truncate
        self.mscale = float(yarn_get_mscale_simple(self.scaling_factor) * attn_factor)
        super().__init__(
            head_size,
            rotary_dim,
            max_position_embeddings,
            base,
            is_neox_style,
            dtype,
            mrope_section=mrope_section,
            mrope_interleaved=mrope_interleaved,
        )
```
**EN:** This block defines `YaRNScalingMRotaryEmbedding.__init__` and contains the main logic for this step. It mainly invokes `float`, `super.__init__`, and `yarn_get_mscale_simple`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.scaling_factor`, `self.extrapolation_factor`, `self.attn_factor`, `self.beta_fast`, and `self.beta_slow` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `YaRNScalingMRotaryEmbedding.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `float`、`super.__init__` 以及 `yarn_get_mscale_simple`，说明该流程会编排底层辅助函数或计算内核。 像 `self.scaling_factor`、`self.extrapolation_factor`、`self.attn_factor`、`self.beta_fast` 以及 `self.beta_slow` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 422-445: Internal helper `YaRNScalingMRotaryEmbedding._compute_inv_freq`
```python
    def _compute_inv_freq(self, scaling_factor: float) -> torch.Tensor:
        pos_freqs = self.base ** (
            torch.arange(0, self.rotary_dim, 2, dtype=torch.float) / self.rotary_dim
        )
        inv_freq_extrapolation = 1.0 / pos_freqs
        inv_freq_interpolation = 1.0 / (scaling_factor * pos_freqs)
        low, high = yarn_find_correction_range(
            self.beta_fast,
            self.beta_slow,
            self.rotary_dim,
            self.base,
            self.max_position_embeddings,
            self.truncate,
        )
        inv_freq_mask = (
            1
            - yarn_linear_ramp_mask(low, high, self.rotary_dim // 2, dtype=torch.float)
        ) * self.extrapolation_factor
        inv_freq = (
            inv_freq_interpolation * (1 - inv_freq_mask)
            + inv_freq_extrapolation * inv_freq_mask
        )
        return inv_freq
```
**EN:** This block defines `YaRNScalingMRotaryEmbedding._compute_inv_freq` and contains the main logic for this step. It mainly invokes `yarn_find_correction_range`, `torch.arange`, and `yarn_linear_ramp_mask`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `pos_freqs`, `inv_freq_extrapolation`, `inv_freq_interpolation`, `low`, and `high` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `YaRNScalingMRotaryEmbedding._compute_inv_freq`，并承载这一阶段的核心逻辑。 它主要调用 `yarn_find_correction_range`、`torch.arange` 以及 `yarn_linear_ramp_mask`，说明该流程会编排底层辅助函数或计算内核。 像 `pos_freqs`、`inv_freq_extrapolation`、`inv_freq_interpolation`、`low` 以及 `high` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 446-457: Internal helper `YaRNScalingMRotaryEmbedding._compute_cos_sin_cache`
```python
    def _compute_cos_sin_cache(self) -> torch.Tensor:
        inv_freq = self._compute_inv_freq(self.scaling_factor)
        t = torch.arange(
            self.max_position_embeddings * self.scaling_factor, dtype=torch.float32
        )
        freqs = torch.einsum("i,j -> ij", t, inv_freq)
        cos = freqs.cos() * self.mscale
        sin = freqs.sin() * self.mscale
        cache = torch.cat((cos, sin), dim=-1)
        return cache
```
**EN:** This block defines `YaRNScalingMRotaryEmbedding._compute_cos_sin_cache` and contains the main logic for this step. It mainly invokes `self._compute_inv_freq`, `torch.arange`, `torch.einsum`, `torch.cat`, and `freqs.cos`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `inv_freq`, `t`, `freqs`, `cos`, and `sin` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `YaRNScalingMRotaryEmbedding._compute_cos_sin_cache`，并承载这一阶段的核心逻辑。 它主要调用 `self._compute_inv_freq`、`torch.arange`、`torch.einsum`、`torch.cat` 以及 `freqs.cos`，说明该流程会编排底层辅助函数或计算内核。 像 `inv_freq`、`t`、`freqs`、`cos` 以及 `sin` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 458-460: Class `Ernie4_5_VLRotaryEmbedding` declaration and shared state
```python
class Ernie4_5_VLRotaryEmbedding(MRotaryEmbedding):
    """3D rotary positional embedding. [h w h w h w h w... t t t...]"""
```
**EN:** This block introduces class `Ernie4_5_VLRotaryEmbedding` and the state shared by its methods. It inherits from `MRotaryEmbedding`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: 3D rotary positional embedding.
**CN:** 该代码块引入类 `Ernie4_5_VLRotaryEmbedding`，并定义其方法共享的状态。 它继承自 `MRotaryEmbedding`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 461-483: `Ernie4_5_VLRotaryEmbedding` initialization and state setup
```python
    def __init__(
        self,
        head_size: int,
        rotary_dim: int,
        max_position_embeddings: int,
        base: int,
        is_neox_style: bool,
        dtype: torch.dtype,
        mrope_section: Optional[List[int]] = None,
        mrope_interleaved: bool = False,
    ) -> None:
        super().__init__(
            head_size,
            rotary_dim,
            max_position_embeddings,
            base,
            is_neox_style,
            dtype,
            mrope_section=mrope_section,
            mrope_interleaved=mrope_interleaved,
        )
        self._apply_rotary_emb_wrapped = torch.compile(dynamic=True)(apply_rotary_emb)
```
**EN:** This block defines `Ernie4_5_VLRotaryEmbedding.__init__` and contains the main logic for this step. It mainly invokes `super.__init__` and `torch.compile`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self._apply_rotary_emb_wrapped` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `Ernie4_5_VLRotaryEmbedding.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__` 和 `torch.compile`，说明该流程会编排底层辅助函数或计算内核。 像 `self._apply_rotary_emb_wrapped` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 484-535: `Ernie4_5_VLRotaryEmbedding.forward_native` NATIVE execution path
```python
    def forward_native(
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor = None,
    ):
        assert positions.ndim == 1 or positions.ndim == 2
        assert key is not None

        num_tokens = positions.shape[-1]
        cos_sin = self.cos_sin_cache[positions]
        cos, sin = cos_sin.chunk(2, dim=-1)
        if positions.ndim == 2:
            assert self.mrope_section
            section_h = self.mrope_section[0]
            section_w = self.mrope_section[1]
            section_t = self.mrope_section[2]
            assert section_h == section_w
            section_cos_t = cos[..., -section_t:]
            section_cos_h = cos[..., : section_h + section_w : 2]
            section_cos_w = cos[..., 1 : section_h + section_w : 2]
            cos_t, cos_h, cos_w = section_cos_t[0], section_cos_h[1], section_cos_w[2]
            cos_hw = torch.stack([cos_h, cos_w], dim=-1).reshape(
                cos_h.shape[:-1] + (cos_h.shape[-1] * 2,)
            )
            cos = torch.cat([cos_hw, cos_t], dim=-1)
            section_sin_t = sin[..., -section_t:]
            section_sin_h = sin[..., : section_h + section_w : 2]
            section_sin_w = sin[..., 1 : section_h + section_w : 2]
            sin_t, sin_h, sin_w = section_sin_t[0], section_sin_h[1], section_sin_w[2]
            sin_hw = torch.stack([sin_h, sin_w], dim=-1).reshape(
                sin_h.shape[:-1] + (sin_h.shape[-1] * 2,)
            )
            sin = torch.cat([sin_hw, sin_t], dim=-1)

        query_shape = query.shape
        query = query.view(num_tokens, -1, self.head_size)
        query_rot = query[..., : self.rotary_dim]
        query_pass = query[..., self.rotary_dim :]
        query_rot = self._apply_rotary_emb_wrapped(
            query_rot, cos, sin, self.is_neox_style
        )
        query = torch.cat((query_rot, query_pass), dim=-1).reshape(query_shape)

        key_shape = key.shape
        key = key.view(num_tokens, -1, self.head_size)
        key_rot = key[..., : self.rotary_dim]
        key_pass = key[..., self.rotary_dim :]
        key_rot = self._apply_rotary_emb_wrapped(key_rot, cos, sin, self.is_neox_style)
        key = torch.cat((key_rot, key_pass), dim=-1).reshape(key_shape)
        return query, key
```
**EN:** This block defines `Ernie4_5_VLRotaryEmbedding.forward_native` and contains the main logic for this step. It mainly invokes `cos_sin.chunk`, `query.view`, `self._apply_rotary_emb_wrapped`, `torch.cat.reshape`, and `key.view`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `num_tokens`, `cos_sin`, `cos`, `sin`, and `query_shape` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `Ernie4_5_VLRotaryEmbedding.forward_native`，并承载这一阶段的核心逻辑。 它主要调用 `cos_sin.chunk`、`query.view`、`self._apply_rotary_emb_wrapped`、`torch.cat.reshape` 以及 `key.view`，说明该流程会编排底层辅助函数或计算内核。 像 `num_tokens`、`cos_sin`、`cos`、`sin` 以及 `query_shape` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 536-572: `Ernie4_5_VLRotaryEmbedding.forward_cuda` CUDA execution path
```python
    def forward_cuda(
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor = None,
    ):
        assert key is not None
        assert positions.ndim in (1, 2)
        self._match_cos_sin_cache_dtype(query)

        if positions.ndim == 2:
            assert self.mrope_section is not None
            triton_ernie45_rope_fused_inplace(
                q=query,
                k=key,
                cos_sin_cache=self.cos_sin_cache,
                positions=positions,
                mrope_section=self.mrope_section,
                head_size=self.head_size,
                rotary_dim=self.rotary_dim,
                is_neox_style=self.is_neox_style,
            )
            return query, key

        if _is_cuda and (apply_rope_with_cos_sin_cache_inplace is not None):
            apply_rope_with_cos_sin_cache_inplace(
                positions=positions,
                query=query,
                key=key,
                head_size=self.head_size,
                cos_sin_cache=self.cos_sin_cache,
                is_neox=self.is_neox_style,
            )
            return query, key

        return self.forward_native(positions, query, key)
```
**EN:** This block defines `Ernie4_5_VLRotaryEmbedding.forward_cuda` and contains the main logic for this step. It mainly invokes `self._match_cos_sin_cache_dtype`, `self.forward_native`, `triton_ernie45_rope_fused_inplace`, and `apply_rope_with_cos_sin_cache_inplace`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `Ernie4_5_VLRotaryEmbedding.forward_cuda`，并承载这一阶段的核心逻辑。 它主要调用 `self._match_cos_sin_cache_dtype`、`self.forward_native`、`triton_ernie45_rope_fused_inplace` 以及 `apply_rope_with_cos_sin_cache_inplace`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 573-581: `Ernie4_5_VLRotaryEmbedding.forward` main forward path
```python
    def forward(
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor,
        fused_set_kv_buffer_arg=None,
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        assert positions.ndim == 1 or positions.ndim == 2
        return self.forward_cuda(positions, query, key)
```
**EN:** This block defines `Ernie4_5_VLRotaryEmbedding.forward` and contains the main logic for this step. It mainly invokes `self.forward_cuda`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `Ernie4_5_VLRotaryEmbedding.forward`，并承载这一阶段的核心逻辑。 它主要调用 `self.forward_cuda`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `apply_interleaved_rope`, `MRotaryEmbedding`, `YaRNScalingMRotaryEmbedding`, and `Ernie4_5_VLRotaryEmbedding`. / **主要符号**：核心入口包括 `apply_interleaved_rope`、`MRotaryEmbedding`、`YaRNScalingMRotaryEmbedding` 以及 `Ernie4_5_VLRotaryEmbedding`。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。
- **Position encoding**: Describes how rotary embeddings or related position transforms are applied. / **位置编码**：说明如何应用旋转位置编码或相关位置变换。
- **Projection layers**: Focuses on matrix multiplication, weight layout, and projection-oriented wrappers. / **投影层**：关注矩阵乘法、权重布局与面向投影的包装层。

## Dependencies / 依赖关系
- **Standard library**: `__future__.annotations`, `typing.List`, `typing.Optional`, and `typing.Tuple` / **标准库**：`__future__.annotations`、`typing.List`、`typing.Optional` 以及 `typing.Tuple`
- **Third-party**: `torch` and `torch_npu` / **第三方依赖**：`torch` 和 `torch_npu`
- **Internal SGLang modules**: `sglang.srt.layers.rotary_embedding.base.RotaryEmbedding`, `sglang.srt.layers.rotary_embedding.triton_kernels.triton_ernie45_rope_fused_inplace`, `sglang.srt.layers.rotary_embedding.triton_kernels.triton_mrope_fused`, `sglang.srt.layers.rotary_embedding.utils.apply_rotary_emb`, `sglang.srt.layers.rotary_embedding.yarn.yarn_find_correction_range`, `sglang.srt.layers.rotary_embedding.yarn.yarn_get_mscale_simple`, `sglang.srt.layers.rotary_embedding.yarn.yarn_linear_ramp_mask`, `sglang.srt.server_args.get_global_server_args`, `sglang.srt.utils.cpu_has_amx_support`, `sglang.srt.utils.is_cuda`, `sglang.srt.utils.is_npu`, and `sglang.jit_kernel.rope.apply_rope_with_cos_sin_cache_inplace` / **SGLang 内部模块**：`sglang.srt.layers.rotary_embedding.base.RotaryEmbedding`、`sglang.srt.layers.rotary_embedding.triton_kernels.triton_ernie45_rope_fused_inplace`、`sglang.srt.layers.rotary_embedding.triton_kernels.triton_mrope_fused`、`sglang.srt.layers.rotary_embedding.utils.apply_rotary_emb`、`sglang.srt.layers.rotary_embedding.yarn.yarn_find_correction_range`、`sglang.srt.layers.rotary_embedding.yarn.yarn_get_mscale_simple`、`sglang.srt.layers.rotary_embedding.yarn.yarn_linear_ramp_mask`、`sglang.srt.server_args.get_global_server_args`、`sglang.srt.utils.cpu_has_amx_support`、`sglang.srt.utils.is_cuda`、`sglang.srt.utils.is_npu` 以及 `sglang.jit_kernel.rope.apply_rope_with_cos_sin_cache_inplace`
