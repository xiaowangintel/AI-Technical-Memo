# mrope_interleaved.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/rotary_embedding/mrope_interleaved.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `MRotaryEmbeddingInterleaved` and related helpers for rotary positional embedding variants. / 实现 `MRotaryEmbeddingInterleaved` 及其相关辅助逻辑，用于旋转位置编码变体。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 22-24)
```python
import torch

from .mrope import MRotaryEmbedding
```
**EN:** This opening block pulls in external dependencies such as `torch` and internal modules such as `.mrope`. That import mix shows the file is part of the rotary positional embedding variants stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `torch`）以及内部模块（如 `.mrope`）。这些导入关系表明该文件属于旋转位置编码变体栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Class `MRotaryEmbeddingInterleaved` overview (lines 28-185)
```python
class MRotaryEmbeddingInterleaved(MRotaryEmbedding):
    """Rotary Embedding with Multimodal Sections and Interleaved Support."""

    def __init__(
        self,
        head_size: int,
        rotary_dim: int,
        max_position_embeddings: int,
        base: float,
        is_neox_style: bool,
        dtype: torch.dtype,
        mrope_section: list[int],
        mrope_interleaved: bool = True,
    ) -> None:
        # Enlarge max_position_embeddings for video inputs
        self.cache_max_position_num = max_position_embeddings
        super().__init__(
            head_size,
            rotary_dim,
            self.cache_max_position_num,
            base,
            is_neox_style,
            dtype,
        )
```
**EN:** Defines class `MRotaryEmbeddingInterleaved` with base classes `MRotaryEmbedding` and decorators none. It acts as a rotary-position-encoding variant and exposes 4 direct methods, with notable entries `__init__`, `_rebuild_pos_emb`, `forward`, `get_mrope_interleaved_id_list`. Its docstring says: Rotary Embedding with Multimodal Sections and Interleaved Support.
**CN:** 定义类 `MRotaryEmbeddingInterleaved`，其基类为 `MRotaryEmbedding`，装饰器为 无。它在整体实现中充当旋转位置编码变体，并直接暴露 4 个方法，较重要的包括 `__init__`, `_rebuild_pos_emb`, `forward`, `get_mrope_interleaved_id_list`。 文档字符串进一步说明了该类的定位。

### Method `MRotaryEmbeddingInterleaved.__init__` (lines 31-82)
```python
    def __init__(
        self,
        head_size: int,
        rotary_dim: int,
        max_position_embeddings: int,
        base: float,
        is_neox_style: bool,
        dtype: torch.dtype,
        mrope_section: list[int],
        mrope_interleaved: bool = True,
    ) -> None:
        # Enlarge max_position_embeddings for video inputs
        self.cache_max_position_num = max_position_embeddings
        super().__init__(
            head_size,
            rotary_dim,
            self.cache_max_position_num,
            base,
            is_neox_style,
            dtype,
        )

        self.mrope_section = mrope_section
        self.mrope_interleaved = mrope_interleaved

        if self.mrope_section is None:
            raise ValueError("mrope_section cannot be None.")
        if sum(self.mrope_section) != rotary_dim // 2:
            raise ValueError("Sum of mrope_section must equal rotary_dim // 2.")
        if not self.mrope_interleaved:
            raise ValueError(
                "mrope_interleaved must be True when mrope_section is provided."
            )

        # Generate interleaved indices
        if len(mrope_section) == 2:
            h_num, w_num = mrope_section[0], mrope_section[1]
            mrope_dim = self.get_mrope_interleaved_id_list(h_num, w_num, 0)
        elif len(mrope_section) == 3:
            t_num, h_num, w_num = mrope_section[0], mrope_section[1], mrope_section[2]
            mrope_dim = self.get_mrope_interleaved_id_list(
                t_num, h_num, w_num, force_last=True
            )
        else:
            raise AssertionError(
                "Cannot support the length of mrope section is not 2 or 3."
            )

        mrope_dim = mrope_dim * 2
        self.mrope_dim = mrope_dim

        self.layer_cache = None
```
**EN:** Defines function `MRotaryEmbeddingInterleaved.__init__` with signature `__init__(self, head_size: int, rotary_dim: int, max_position_embeddings: int, base: float, is_neox_style: bool, dtype: torch.dtype, mrope_section: list[int], mrope_interleaved: bool=True) -> None`. It mainly works with `head_size`, `rotary_dim`, `max_position_embeddings`, `base`, `is_neox_style`, `dtype`, `mrope_section`, `mrope_interleaved`; initializes the object state and cached resources. The body uses branching, validation/error handling. Key calls include `super.__init__`, `ValueError`, `sum`, `len`, `self.get_mrope_interleaved_id_list`, `super`.
**CN:** 定义函数 `MRotaryEmbeddingInterleaved.__init__`，其签名为 `__init__(self, head_size: int, rotary_dim: int, max_position_embeddings: int, base: float, is_neox_style: bool, dtype: torch.dtype, mrope_section: list[int], mrope_interleaved: bool=True) -> None`。它主要围绕 `head_size`, `rotary_dim`, `max_position_embeddings`, `base`, `is_neox_style`, `dtype`, `mrope_section`, `mrope_interleaved` 展开；负责初始化对象状态和缓存资源。函数体包含分支判断、校验或报错逻辑。关键调用包括 `super.__init__`, `ValueError`, `sum`, `len`, `self.get_mrope_interleaved_id_list`, `super`。

### Method `MRotaryEmbeddingInterleaved._rebuild_pos_emb` (lines 84-99)
```python
    def _rebuild_pos_emb(
        self,
        positions: torch.Tensor,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        """Interleave the rotary embedding"""
        cos_sin = self.cos_sin_cache[positions]
        mrope_section_3d = [1] * len(self.mrope_dim)
        mrope_dim = self.mrope_dim
        cos_sin = torch.cat(
            [
                m[mrope_dim[i]]
                for i, m in enumerate(cos_sin.split(mrope_section_3d, dim=-1))
            ],
            dim=-1,
        )
        return cos_sin, torch.arange(cos_sin.shape[0], device=positions.device)
```
**EN:** Defines function `MRotaryEmbeddingInterleaved._rebuild_pos_emb` with signature `_rebuild_pos_emb(self, positions: torch.Tensor) -> tuple[torch.Tensor, torch.Tensor]`. It mainly works with `positions`; adjusts rotary-position-encoding parameters or application logic. The body uses comprehensions, tensor/kernel operations. Key calls include `torch.cat`, `len`, `torch.arange`, `enumerate`, `cos_sin.split`.
**CN:** 定义函数 `MRotaryEmbeddingInterleaved._rebuild_pos_emb`，其签名为 `_rebuild_pos_emb(self, positions: torch.Tensor) -> tuple[torch.Tensor, torch.Tensor]`。它主要围绕 `positions` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含推导式、张量或内核操作。关键调用包括 `torch.cat`, `len`, `torch.arange`, `enumerate`, `cos_sin.split`。

### Method `MRotaryEmbeddingInterleaved.forward` (lines 101-136)
```python
    def forward(
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor | None = None,
    ) -> tuple[torch.Tensor, torch.Tensor | None]:
        """Forward pass with interleaved rotary embedding."""
        cos_sin, positions = self._rebuild_pos_emb(positions)
        cos, sin = cos_sin.chunk(2, dim=-1)

        query_shape = query.shape
        positions = positions.flatten()
        num_tokens = positions.shape[0]
        query = query.view(num_tokens, -1, self.head_size)
        query_rot = query[..., : self.rotary_dim]
        query_pass = query[..., self.rotary_dim :]
        query_rot = self.apply_rotary_emb.forward_native(
            query_rot,
            cos,
            sin,
        )
        query = torch.cat((query_rot, query_pass), dim=-1).reshape(query_shape)

        # key may be None in some cases, e.g. cross-layer KV sharing
        if key is not None:
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
**EN:** Defines function `MRotaryEmbeddingInterleaved.forward` with signature `forward(self, positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor | None=None) -> tuple[torch.Tensor, torch.Tensor | None]`. It mainly works with `positions`, `query`, `key`; runs the main forward-path computation. The body uses branching, tensor/kernel operations. Key calls include `self._rebuild_pos_emb`, `cos_sin.chunk`, `positions.flatten`, `query.view`, `self.apply_rotary_emb.forward_native`, `torch.cat.reshape`.
**CN:** 定义函数 `MRotaryEmbeddingInterleaved.forward`，其签名为 `forward(self, positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor | None=None) -> tuple[torch.Tensor, torch.Tensor | None]`。它主要围绕 `positions`, `query`, `key` 展开；执行主要的前向计算路径。函数体包含分支判断、张量或内核操作。关键调用包括 `self._rebuild_pos_emb`, `cos_sin.chunk`, `positions.flatten`, `query.view`, `self.apply_rotary_emb.forward_native`, `torch.cat.reshape`。

### Method `MRotaryEmbeddingInterleaved.get_mrope_interleaved_id_list` (lines 139-185)
```python
    def get_mrope_interleaved_id_list(
        a: int, b: int, c: int, force_last: bool = False
    ) -> list[int]:
        """
        Generate an interleaved list of indices for multi-modal rotary embedding.

        Args:
            a: Number of indices for first modality
            b: Number of indices for second modality
            c: Number of indices for third modality
            force_last: Whether to force the last element to be from the first modality

        Returns:
            List of interleaved indices
        """
        if force_last:
            a -= 1

        counts = {0: a, 1: b, 2: c}
        placed = {k: 0 for k in counts}
        rem = counts.copy()
        seq: list[int] = []
        last = None

        total = a + b + c
        for _ in range(total):
            # Candidates: remaining > 0 and ≠ last
            cands = [k for k in rem if rem[k] > 0 and k != last]
            if not cands:
                # If only last remains, relax the condition
                cands = [k for k in rem if rem[k] > 0]

            # Select the rarest candidate
            try:
                best = min(cands, key=lambda k: (placed[k] / counts[k], k))
            except KeyError:
                best = 0

            seq.append(best)
            placed[best] += 1
            rem[best] -= 1
            last = best

        if force_last:
            seq.append(0)

        return seq
```
**EN:** Defines function `MRotaryEmbeddingInterleaved.get_mrope_interleaved_id_list` with signature `get_mrope_interleaved_id_list(a: int, b: int, c: int, force_last: bool=False) -> list[int]`. It mainly works with `a`, `b`, `c`, `force_last`; returns a derived property or capability check. The body uses branching, iteration, comprehensions. Key calls include `counts.copy`, `range`, `seq.append`, `min`.
**CN:** 定义函数 `MRotaryEmbeddingInterleaved.get_mrope_interleaved_id_list`，其签名为 `get_mrope_interleaved_id_list(a: int, b: int, c: int, force_last: bool=False) -> list[int]`。它主要围绕 `a`, `b`, `c`, `force_last` 展开；返回派生属性或能力判断结果。函数体包含分支判断、循环处理、推导式。关键调用包括 `counts.copy`, `range`, `seq.append`, `min`。

## Key Concepts / 关键概念
- **EN:** The file implements a RoPE-related variant, meaning frequency scaling, cache construction, and position application are key ideas.
  **CN:** 该文件实现了与 RoPE 相关的变体，因此频率缩放、缓存构建和位置应用是关键概念。
- **EN:** Top-level classes include `MRotaryEmbeddingInterleaved`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `MRotaryEmbeddingInterleaved`，它们组织了主要的可复用抽象。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。
- **EN:** Internal vLLM imports show that this module is tightly integrated with the broader runtime, parallelism, or custom-layer stack.
  **CN:** vLLM 内部导入说明该模块与更大的运行时、并行机制或自定义层栈紧密集成。

## Dependencies / 依赖关系
- **External / 外部**: `torch`
- **Internal / 内部**: `.mrope`
