# ernie45_vl_rope.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/rotary_embedding/ernie45_vl_rope.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `Ernie4_5_VLRotaryEmbedding` and related helpers for rotary positional embedding variants. / 实现 `Ernie4_5_VLRotaryEmbedding` 及其相关辅助逻辑，用于旋转位置编码变体。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 5-7)
```python
import torch

from .mrope import MRotaryEmbedding
```
**EN:** This opening block pulls in external dependencies such as `torch` and internal modules such as `.mrope`. That import mix shows the file is part of the rotary positional embedding variants stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `torch`）以及内部模块（如 `.mrope`）。这些导入关系表明该文件属于旋转位置编码变体栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Class `Ernie4_5_VLRotaryEmbedding` overview (lines 10-82)
```python
class Ernie4_5_VLRotaryEmbedding(MRotaryEmbedding):
    """3D rotary positional embedding. 3D is t:time h:height w:width"""

    def forward_native(  # type: ignore[override]
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor | None = None,
    ) -> tuple[torch.Tensor, torch.Tensor | None]:
        assert positions.ndim == 1 or positions.ndim == 2
        assert key is not None

        num_tokens = positions.shape[-1]
        cos_sin = self.cos_sin_cache[positions]
        cos, sin = cos_sin.chunk(2, dim=-1)
        if positions.ndim == 2:
            assert self.mrope_section

            section_h = self.mrope_section[0]  # 22
            section_w = self.mrope_section[1]  # 22
            section_t = self.mrope_section[2]  # 20
            assert section_h == section_w
            # Split according to [h w h w h w h w... t t t...]
            section_cos_t = cos[..., -section_t:]
            section_cos_h = cos[..., : section_h + section_w : 2]
```
**EN:** Defines class `Ernie4_5_VLRotaryEmbedding` with base classes `MRotaryEmbedding` and decorators none. It acts as an embedding layer with model-parallel awareness and exposes 2 direct methods, with notable entries `forward_native`, `forward_cuda`. Its docstring says: 3D rotary positional embedding.
**CN:** 定义类 `Ernie4_5_VLRotaryEmbedding`，其基类为 `MRotaryEmbedding`，装饰器为 无。它在整体实现中充当具备模型并行意识的嵌入层，并直接暴露 2 个方法，较重要的包括 `forward_native`, `forward_cuda`。 文档字符串进一步说明了该类的定位。

### Method `Ernie4_5_VLRotaryEmbedding.forward_native` (lines 13-74)
```python
    def forward_native(  # type: ignore[override]
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor | None = None,
    ) -> tuple[torch.Tensor, torch.Tensor | None]:
        assert positions.ndim == 1 or positions.ndim == 2
        assert key is not None

        num_tokens = positions.shape[-1]
        cos_sin = self.cos_sin_cache[positions]
        cos, sin = cos_sin.chunk(2, dim=-1)
        if positions.ndim == 2:
            assert self.mrope_section

            section_h = self.mrope_section[0]  # 22
            section_w = self.mrope_section[1]  # 22
            section_t = self.mrope_section[2]  # 20
            assert section_h == section_w
            # Split according to [h w h w h w h w... t t t...]
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
# ... truncated for analysis ...
        )
        query = torch.cat((query_rot, query_pass), dim=-1).reshape(query_shape)

        key_shape = key.shape
        key = key.view(num_tokens, -1, self.head_size)
        key_rot = key[..., : self.rotary_dim]
        key_pass = key[..., self.rotary_dim :]
        key_rot = self.apply_rotary_emb.forward_native(
            key_rot,
            cos,
            sin,
        )
        key = torch.cat((key_rot, key_pass), dim=-1).reshape(key_shape)
        return query, key
```
**EN:** Defines function `Ernie4_5_VLRotaryEmbedding.forward_native` with signature `forward_native(self, positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor | None=None) -> tuple[torch.Tensor, torch.Tensor | None]`. It mainly works with `positions`, `query`, `key`; adjusts rotary-position-encoding parameters or application logic. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `cos_sin.chunk`, `query.view`, `self.apply_rotary_emb.forward_native`, `torch.cat.reshape`, `key.view`, `torch.stack.reshape`.
**CN:** 定义函数 `Ernie4_5_VLRotaryEmbedding.forward_native`，其签名为 `forward_native(self, positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor | None=None) -> tuple[torch.Tensor, torch.Tensor | None]`。它主要围绕 `positions`, `query`, `key` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `cos_sin.chunk`, `query.view`, `self.apply_rotary_emb.forward_native`, `torch.cat.reshape`, `key.view`, `torch.stack.reshape`。

### Method `Ernie4_5_VLRotaryEmbedding.forward_cuda` (lines 76-82)
```python
    def forward_cuda(  # type: ignore[override]
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor | None = None,
    ) -> tuple[torch.Tensor, torch.Tensor | None]:
        return self.forward_native(positions, query, key)
```
**EN:** Defines function `Ernie4_5_VLRotaryEmbedding.forward_cuda` with signature `forward_cuda(self, positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor | None=None) -> tuple[torch.Tensor, torch.Tensor | None]`. It mainly works with `positions`, `query`, `key`; adjusts rotary-position-encoding parameters or application logic. The body uses mostly straightforward data movement and object wiring. Key calls include `self.forward_native`.
**CN:** 定义函数 `Ernie4_5_VLRotaryEmbedding.forward_cuda`，其签名为 `forward_cuda(self, positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor | None=None) -> tuple[torch.Tensor, torch.Tensor | None]`。它主要围绕 `positions`, `query`, `key` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `self.forward_native`。

## Key Concepts / 关键概念
- **EN:** The file implements a RoPE-related variant, meaning frequency scaling, cache construction, and position application are key ideas.
  **CN:** 该文件实现了与 RoPE 相关的变体，因此频率缩放、缓存构建和位置应用是关键概念。
- **EN:** Top-level classes include `Ernie4_5_VLRotaryEmbedding`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `Ernie4_5_VLRotaryEmbedding`，它们组织了主要的可复用抽象。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。
- **EN:** Internal vLLM imports show that this module is tightly integrated with the broader runtime, parallelism, or custom-layer stack.
  **CN:** vLLM 内部导入说明该模块与更大的运行时、并行机制或自定义层栈紧密集成。

## Dependencies / 依赖关系
- **External / 外部**: `torch`
- **Internal / 内部**: `.mrope`
