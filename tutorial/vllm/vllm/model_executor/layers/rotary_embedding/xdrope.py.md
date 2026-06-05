# xdrope.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/rotary_embedding/xdrope.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `XDRotaryEmbedding` and related helpers for rotary positional embedding variants. / 实现 `XDRotaryEmbedding` 及其相关辅助逻辑，用于旋转位置编码变体。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 4-7)
```python
import numpy as np
import torch

from .dynamic_ntk_alpha_rope import DynamicNTKAlphaRotaryEmbedding
```
**EN:** This opening block pulls in external dependencies such as `numpy`, `torch` and internal modules such as `.dynamic_ntk_alpha_rope`. That import mix shows the file is part of the rotary positional embedding variants stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `numpy`, `torch`）以及内部模块（如 `.dynamic_ntk_alpha_rope`）。这些导入关系表明该文件属于旋转位置编码变体栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Class `XDRotaryEmbedding` overview (lines 10-160)
```python
class XDRotaryEmbedding(DynamicNTKAlphaRotaryEmbedding):
    """DynamicNTKAlphaRotaryEmbedding extended with MultiModal(XD) Sections.

    Based on the original DynamicNTKAlphaRotaryEmbedding implementation.
    """

    def __init__(
        self,
        head_size: int,
        rotary_dim: int,
        max_position_embeddings: int,
        base: float,
        is_neox_style: bool,
        scaling_alpha: float,
        dtype: torch.dtype,
        xdrope_section: list[int],
    ) -> None:
        self.xdrope_section = xdrope_section
        super().__init__(
            head_size,
            rotary_dim,
            max_position_embeddings,
            base,
            is_neox_style,
            scaling_alpha,
```
**EN:** Defines class `XDRotaryEmbedding` with base classes `DynamicNTKAlphaRotaryEmbedding` and decorators none. It acts as an embedding layer with model-parallel awareness and exposes 5 direct methods, with notable entries `__init__`, `forward_native`, `forward_cuda`, `get_next_input_positions`, `get_next_input_positions_tensor`. Its docstring says: DynamicNTKAlphaRotaryEmbedding extended with MultiModal(XD) Sections.
**CN:** 定义类 `XDRotaryEmbedding`，其基类为 `DynamicNTKAlphaRotaryEmbedding`，装饰器为 无。它在整体实现中充当具备模型并行意识的嵌入层，并直接暴露 5 个方法，较重要的包括 `__init__`, `forward_native`, `forward_cuda`, `get_next_input_positions`, `get_next_input_positions_tensor`。 文档字符串进一步说明了该类的定位。

### Method `XDRotaryEmbedding.__init__` (lines 16-36)
```python
    def __init__(
        self,
        head_size: int,
        rotary_dim: int,
        max_position_embeddings: int,
        base: float,
        is_neox_style: bool,
        scaling_alpha: float,
        dtype: torch.dtype,
        xdrope_section: list[int],
    ) -> None:
        self.xdrope_section = xdrope_section
        super().__init__(
            head_size,
            rotary_dim,
            max_position_embeddings,
            base,
            is_neox_style,
            scaling_alpha,
            dtype,
        )
```
**EN:** Defines function `XDRotaryEmbedding.__init__` with signature `__init__(self, head_size: int, rotary_dim: int, max_position_embeddings: int, base: float, is_neox_style: bool, scaling_alpha: float, dtype: torch.dtype, xdrope_section: list[int]) -> None`. It mainly works with `head_size`, `rotary_dim`, `max_position_embeddings`, `base`, `is_neox_style`, `scaling_alpha`, `dtype`, `xdrope_section`; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include `super.__init__`, `super`.
**CN:** 定义函数 `XDRotaryEmbedding.__init__`，其签名为 `__init__(self, head_size: int, rotary_dim: int, max_position_embeddings: int, base: float, is_neox_style: bool, scaling_alpha: float, dtype: torch.dtype, xdrope_section: list[int]) -> None`。它主要围绕 `head_size`, `rotary_dim`, `max_position_embeddings`, `base`, `is_neox_style`, `scaling_alpha`, `dtype`, `xdrope_section` 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `super.__init__`, `super`。

### Method `XDRotaryEmbedding.forward_native` (lines 38-87)
```python
    def forward_native(
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor | None = None,
        offsets: torch.Tensor | None = None,
    ) -> tuple[torch.Tensor, torch.Tensor | None]:
        """PyTorch-native implementation equivalent to forward().

        Args:
            positions:
                [4, num_tokens] (P/W/H/T positions with multimodal inputs)
            query: [num_tokens, num_heads * head_size]
            key: [num_tokens, num_kv_heads * head_size]
        """
        assert positions.ndim == 2
        assert key is not None

        num_tokens = positions.shape[-1]
        cos_sin = self.cos_sin_cache[positions]
        cos, sin = cos_sin.chunk(2, dim=-1)
        cos = torch.cat(
            [m[i] for i, m in enumerate(cos.split(self.xdrope_section, dim=-1))], dim=-1
        )
        sin = torch.cat(
            [m[i] for i, m in enumerate(sin.split(self.xdrope_section, dim=-1))], dim=-1
        )

        query_shape = query.shape
        query = query.view(num_tokens, -1, self.head_size)
        query_rot = query[..., : self.rotary_dim]
        query_pass = query[..., self.rotary_dim :]
        query_rot = self.apply_rotary_emb.forward_native(
            query_rot,
            cos,
            sin,
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
**EN:** Defines function `XDRotaryEmbedding.forward_native` with signature `forward_native(self, positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor | None=None, offsets: torch.Tensor | None=None) -> tuple[torch.Tensor, torch.Tensor | None]`. It mainly works with `positions`, `query`, `key`, `offsets`; adjusts rotary-position-encoding parameters or application logic. The body uses comprehensions, validation/error handling, tensor/kernel operations. Key calls include `cos_sin.chunk`, `torch.cat`, `query.view`, `self.apply_rotary_emb.forward_native`, `torch.cat.reshape`, `key.view`.
**CN:** 定义函数 `XDRotaryEmbedding.forward_native`，其签名为 `forward_native(self, positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor | None=None, offsets: torch.Tensor | None=None) -> tuple[torch.Tensor, torch.Tensor | None]`。它主要围绕 `positions`, `query`, `key`, `offsets` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含推导式、校验或报错逻辑、张量或内核操作。关键调用包括 `cos_sin.chunk`, `torch.cat`, `query.view`, `self.apply_rotary_emb.forward_native`, `torch.cat.reshape`, `key.view`。

### Method `XDRotaryEmbedding.forward_cuda` (lines 89-138)
```python
    def forward_cuda(
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor | None = None,
        offsets: torch.Tensor | None = None,
    ) -> tuple[torch.Tensor, torch.Tensor | None]:
        """PyTorch-native implementation equivalent to forward().

        Args:
            positions:
                [4, num_tokens] (P/W/H/T positions with multimodal inputs)
            query: [num_tokens, num_heads * head_size]
            key: [num_tokens, num_kv_heads * head_size]
        """
        assert positions.ndim == 2
        assert key is not None

        num_tokens = positions.shape[-1]
        cos_sin = self.cos_sin_cache[positions]
        cos, sin = cos_sin.chunk(2, dim=-1)
        cos = torch.cat(
            [m[i] for i, m in enumerate(cos.split(self.xdrope_section, dim=-1))], dim=-1
        )
        sin = torch.cat(
            [m[i] for i, m in enumerate(sin.split(self.xdrope_section, dim=-1))], dim=-1
        )

        query_shape = query.shape
        query = query.view(num_tokens, -1, self.head_size)
        query_rot = query[..., : self.rotary_dim]
        query_pass = query[..., self.rotary_dim :]
        query_rot = self.apply_rotary_emb(
            query_rot,
            cos,
            sin,
        )
        query = torch.cat((query_rot, query_pass), dim=-1).reshape(query_shape)

        key_shape = key.shape
        key = key.view(num_tokens, -1, self.head_size)
        key_rot = key[..., : self.rotary_dim]
        key_pass = key[..., self.rotary_dim :]
        key_rot = self.apply_rotary_emb(
            key_rot,
            cos,
            sin,
        )
        key = torch.cat((key_rot, key_pass), dim=-1).reshape(key_shape)
        return query, key
```
**EN:** Defines function `XDRotaryEmbedding.forward_cuda` with signature `forward_cuda(self, positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor | None=None, offsets: torch.Tensor | None=None) -> tuple[torch.Tensor, torch.Tensor | None]`. It mainly works with `positions`, `query`, `key`, `offsets`; adjusts rotary-position-encoding parameters or application logic. The body uses comprehensions, validation/error handling, tensor/kernel operations. Key calls include `cos_sin.chunk`, `torch.cat`, `query.view`, `self.apply_rotary_emb`, `torch.cat.reshape`, `key.view`.
**CN:** 定义函数 `XDRotaryEmbedding.forward_cuda`，其签名为 `forward_cuda(self, positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor | None=None, offsets: torch.Tensor | None=None) -> tuple[torch.Tensor, torch.Tensor | None]`。它主要围绕 `positions`, `query`, `key`, `offsets` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含推导式、校验或报错逻辑、张量或内核操作。关键调用包括 `cos_sin.chunk`, `torch.cat`, `query.view`, `self.apply_rotary_emb`, `torch.cat.reshape`, `key.view`。

### Method `XDRotaryEmbedding.get_next_input_positions` (lines 141-146)
```python
    def get_next_input_positions(
        context_len: int,
        seq_len: int,
        xd_sections: int = 4,
    ) -> list[list[int]]:
        return [list(range(context_len, seq_len)) for _ in range(xd_sections)]
```
**EN:** Defines function `XDRotaryEmbedding.get_next_input_positions` with signature `get_next_input_positions(context_len: int, seq_len: int, xd_sections: int=4) -> list[list[int]]`. It mainly works with `context_len`, `seq_len`, `xd_sections`; returns a derived property or capability check. The body uses comprehensions. Key calls include `list`, `range`.
**CN:** 定义函数 `XDRotaryEmbedding.get_next_input_positions`，其签名为 `get_next_input_positions(context_len: int, seq_len: int, xd_sections: int=4) -> list[list[int]]`。它主要围绕 `context_len`, `seq_len`, `xd_sections` 展开；返回派生属性或能力判断结果。函数体包含推导式。关键调用包括 `list`, `range`。

### Method `XDRotaryEmbedding.get_next_input_positions_tensor` (lines 149-160)
```python
    def get_next_input_positions_tensor(
        out: np.ndarray,
        out_offset: int,
        context_len: int,
        num_new_tokens: int,
    ):
        values = np.arange(
            context_len,
            context_len + num_new_tokens,
            dtype=out.dtype,
        )
        out[:, out_offset : out_offset + num_new_tokens] = values
```
**EN:** Defines function `XDRotaryEmbedding.get_next_input_positions_tensor` with signature `get_next_input_positions_tensor(out: np.ndarray, out_offset: int, context_len: int, num_new_tokens: int)`. It mainly works with `out`, `out_offset`, `context_len`, `num_new_tokens`; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include `np.arange`.
**CN:** 定义函数 `XDRotaryEmbedding.get_next_input_positions_tensor`，其签名为 `get_next_input_positions_tensor(out: np.ndarray, out_offset: int, context_len: int, num_new_tokens: int)`。它主要围绕 `out`, `out_offset`, `context_len`, `num_new_tokens` 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `np.arange`。

## Key Concepts / 关键概念
- **EN:** The file implements a RoPE-related variant, meaning frequency scaling, cache construction, and position application are key ideas.
  **CN:** 该文件实现了与 RoPE 相关的变体，因此频率缩放、缓存构建和位置应用是关键概念。
- **EN:** Top-level classes include `XDRotaryEmbedding`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `XDRotaryEmbedding`，它们组织了主要的可复用抽象。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。
- **EN:** Internal vLLM imports show that this module is tightly integrated with the broader runtime, parallelism, or custom-layer stack.
  **CN:** vLLM 内部导入说明该模块与更大的运行时、并行机制或自定义层栈紧密集成。

## Dependencies / 依赖关系
- **External / 外部**: `numpy`, `torch`
- **Internal / 内部**: `.dynamic_ntk_alpha_rope`
