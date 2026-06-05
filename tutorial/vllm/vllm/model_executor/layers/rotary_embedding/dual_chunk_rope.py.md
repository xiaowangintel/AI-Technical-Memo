# dual_chunk_rope.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/rotary_embedding/dual_chunk_rope.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `DualChunkRotaryEmbedding` and related helpers for rotary positional embedding variants. / 实现 `DualChunkRotaryEmbedding` 及其相关辅助逻辑，用于旋转位置编码变体。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 5-9)
```python
import torch

from vllm.model_executor.custom_op import CustomOp

from .common import rotate_gptj, rotate_neox
```
**EN:** This opening block pulls in external dependencies such as `torch` and internal modules such as `vllm.model_executor.custom_op`, `.common`. That import mix shows the file is part of the rotary positional embedding variants stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `torch`）以及内部模块（如 `vllm.model_executor.custom_op`, `.common`）。这些导入关系表明该文件属于旋转位置编码变体栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Class `DualChunkRotaryEmbedding` overview (lines 14-217)
```python
class DualChunkRotaryEmbedding(CustomOp):
    """Rotary positional embedding for Dual Chunk Attention."""

    # --8<-- [end:dual_chunk_rotary_embedding]

    def __init__(
        self,
        head_size: int,
        rotary_dim: int,
        max_position_embeddings: int,
        base: float,
        is_neox_style: bool,
        dtype: torch.dtype,
        chunk_size: int,
        local_size: int,
    ) -> None:
        super().__init__()
        self.head_size = head_size
        self.rotary_dim = rotary_dim
        self.max_position_embeddings = max_position_embeddings
        self.base = base
        self.is_neox_style = is_neox_style
        self.chunk_size = chunk_size
        self.local_size = local_size
        self.dtype = dtype
```
**EN:** Defines class `DualChunkRotaryEmbedding` with base classes `CustomOp` and decorators `CustomOp.register('dual_chunk_rotary_embedding')`. It acts as an embedding layer with model-parallel awareness and exposes 7 direct methods, with notable entries `__init__`, `_compute_inv_freq`, `_compute_cos_sin_cache`, `forward_native`, `forward_cuda`, `_apply_rotary_embedding`. Its docstring says: Rotary positional embedding for Dual Chunk Attention.
**CN:** 定义类 `DualChunkRotaryEmbedding`，其基类为 `CustomOp`，装饰器为 `CustomOp.register('dual_chunk_rotary_embedding')`。它在整体实现中充当具备模型并行意识的嵌入层，并直接暴露 7 个方法，较重要的包括 `__init__`, `_compute_inv_freq`, `_compute_cos_sin_cache`, `forward_native`, `forward_cuda`, `_apply_rotary_embedding`。 文档字符串进一步说明了该类的定位。

### Method `DualChunkRotaryEmbedding.__init__` (lines 19-51)
```python
    def __init__(
        self,
        head_size: int,
        rotary_dim: int,
        max_position_embeddings: int,
        base: float,
        is_neox_style: bool,
        dtype: torch.dtype,
        chunk_size: int,
        local_size: int,
    ) -> None:
        super().__init__()
        self.head_size = head_size
        self.rotary_dim = rotary_dim
        self.max_position_embeddings = max_position_embeddings
        self.base = base
        self.is_neox_style = is_neox_style
        self.chunk_size = chunk_size
        self.local_size = local_size
        self.dtype = dtype
        device_idx = torch.accelerator.current_device_index()
        self.device = torch.device(f"cuda:{device_idx}")
        (q_cache, qc_cache, k_cache, qc_no_clamp_cache, q_inter_cache) = (
            self._compute_cos_sin_cache()
        )

        self.register_buffer("cos_sin_q_cache", q_cache, persistent=False)
        self.register_buffer("cos_sin_qc_cache", qc_cache, persistent=False)
        self.register_buffer("cos_sin_k_cache", k_cache, persistent=False)
        self.register_buffer(
            "cos_sin_qc_no_clamp_cache", qc_no_clamp_cache, persistent=False
        )
        self.register_buffer("cos_sin_q_inter_cache", q_inter_cache, persistent=False)
```
**EN:** Defines function `DualChunkRotaryEmbedding.__init__` with signature `__init__(self, head_size: int, rotary_dim: int, max_position_embeddings: int, base: float, is_neox_style: bool, dtype: torch.dtype, chunk_size: int, local_size: int) -> None`. It mainly works with `head_size`, `rotary_dim`, `max_position_embeddings`, `base`, `is_neox_style`, `dtype`, `chunk_size`, `local_size`; initializes the object state and cached resources. The body uses tensor/kernel operations. Key calls include `super.__init__`, `torch.accelerator.current_device_index`, `torch.device`, `self._compute_cos_sin_cache`, `self.register_buffer`, `super`.
**CN:** 定义函数 `DualChunkRotaryEmbedding.__init__`，其签名为 `__init__(self, head_size: int, rotary_dim: int, max_position_embeddings: int, base: float, is_neox_style: bool, dtype: torch.dtype, chunk_size: int, local_size: int) -> None`。它主要围绕 `head_size`, `rotary_dim`, `max_position_embeddings`, `base`, `is_neox_style`, `dtype`, `chunk_size`, `local_size` 展开；负责初始化对象状态和缓存资源。函数体包含张量或内核操作。关键调用包括 `super.__init__`, `torch.accelerator.current_device_index`, `torch.device`, `self._compute_cos_sin_cache`, `self.register_buffer`, `super`。

### Method `DualChunkRotaryEmbedding._compute_inv_freq` (lines 53-70)
```python
    def _compute_inv_freq(self, base: float) -> torch.Tensor:
        """Compute the inverse frequency."""
        # NOTE(woosuk): The HF implementation uses `torch.arange(...).float()`.
        # However, we use `torch.arange(..., dtype=torch.float)` instead to
        # avoid numerical issues with large base values (e.g., 10000000).
        # This may cause a slight numerical difference between the HF
        # implementation and ours.
        # NOTE(woosuk): To exactly match the HF implementation, we need to
        # use CPU to compute the cache and then move it to GPU. However, we
        # create the cache on GPU for faster initialization. This may cause
        # a slight numerical difference between the HF implementation and ours.
        inv_freq = 1.0 / (
            base
            ** (
                torch.arange(0, self.rotary_dim, 2, dtype=torch.float) / self.rotary_dim
            )
        )
        return inv_freq
```
**EN:** Defines function `DualChunkRotaryEmbedding._compute_inv_freq` with signature `_compute_inv_freq(self, base: float) -> torch.Tensor`. It mainly works with `base`; adjusts rotary-position-encoding parameters or application logic. The body uses tensor/kernel operations. Key calls include `torch.arange`.
**CN:** 定义函数 `DualChunkRotaryEmbedding._compute_inv_freq`，其签名为 `_compute_inv_freq(self, base: float) -> torch.Tensor`。它主要围绕 `base` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含张量或内核操作。关键调用包括 `torch.arange`。

### Method `DualChunkRotaryEmbedding._compute_cos_sin_cache` (lines 72-120)
```python
    def _compute_cos_sin_cache(self) -> torch.Tensor:
        """Compute the cos and sin cache."""
        inv_freq = self._compute_inv_freq(self.base)
        chunk_len = self.chunk_size - self.local_size
        q_t = torch.arange(chunk_len, dtype=torch.float)
        qc_t = (torch.arange(chunk_len, dtype=torch.float) + chunk_len).clamp(
            max=self.chunk_size
        )
        k_t = torch.arange(self.max_position_embeddings, dtype=torch.float) % chunk_len

        # count from chunk_len, no clamp(self.chunk_size) restriction
        qc_no_clamp_t = torch.arange(chunk_len, dtype=torch.float) + chunk_len
        # count from self.chunk_size for q_inter's rope
        q_inter_t = torch.arange(chunk_len, dtype=torch.float) + self.chunk_size

        q_freqs = torch.outer(q_t, inv_freq)
        qc_freqs = torch.outer(qc_t, inv_freq)
        k_freqs = torch.outer(k_t, inv_freq)
        qc_no_clamp_freqs = torch.outer(qc_no_clamp_t, inv_freq)
        q_inter_freqs = torch.outer(q_inter_t, inv_freq)

        q_cos = q_freqs.cos()
        q_sin = q_freqs.sin()
        qc_cos = qc_freqs.cos()
        qc_sin = qc_freqs.sin()
        k_cos = k_freqs.cos()
        k_sin = k_freqs.sin()

        qc_no_clamp_cos = qc_no_clamp_freqs.cos()
        qc_no_clamp_sin = qc_no_clamp_freqs.sin()
        q_inter_cos = q_inter_freqs.cos()
        q_inter_sin = q_inter_freqs.sin()

        q_cache = torch.cat((q_cos, q_sin), dim=-1).to(
            dtype=self.dtype, device=self.device
        )
        qc_cache = torch.cat((qc_cos, qc_sin), dim=-1).to(
            dtype=self.dtype, device=self.device
        )
        k_cache = torch.cat((k_cos, k_sin), dim=-1).to(
            dtype=self.dtype, device=self.device
        )
        qc_no_clamp_cache = torch.cat((qc_no_clamp_cos, qc_no_clamp_sin), dim=-1).to(
            dtype=self.dtype, device=self.device
        )
        q_inter_cache = torch.cat((q_inter_cos, q_inter_sin), dim=-1).to(
            dtype=self.dtype, device=self.device
        )
        return q_cache, qc_cache, k_cache, qc_no_clamp_cache, q_inter_cache
```
**EN:** Defines function `DualChunkRotaryEmbedding._compute_cos_sin_cache` with signature `_compute_cos_sin_cache(self) -> torch.Tensor`. It mainly works with object context only; adjusts rotary-position-encoding parameters or application logic. The body uses tensor/kernel operations. Key calls include `self._compute_inv_freq`, `torch.arange`, `clamp`, `torch.outer`, `q_freqs.cos`, `q_freqs.sin`.
**CN:** 定义函数 `DualChunkRotaryEmbedding._compute_cos_sin_cache`，其签名为 `_compute_cos_sin_cache(self) -> torch.Tensor`。它主要围绕 仅依赖对象上下文 展开；调整旋转位置编码参数或其应用逻辑。函数体包含张量或内核操作。关键调用包括 `self._compute_inv_freq`, `torch.arange`, `clamp`, `torch.outer`, `q_freqs.cos`, `q_freqs.sin`。

### Method `DualChunkRotaryEmbedding.forward_native` (lines 122-184)
```python
    def forward_native(
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor,
        offsets: torch.Tensor | None = None,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        query = query.view(*query.shape[:-1], -1, self.head_size)
        key = key.view(*key.shape[:-1], -1, self.head_size)
        query_rot = query[..., : self.rotary_dim]
        key_rot = key[..., : self.rotary_dim]
        if self.rotary_dim < self.head_size:
            query_pass = query[..., self.rotary_dim :]
            key_pass = key[..., self.rotary_dim :]
        else:
            query_pass = None
            key_pass = None

        positions_with_offsets = (
            torch.add(positions, offsets) if offsets is not None else positions
        )
        key = self._apply_rotary_embedding(
            self.cos_sin_k_cache[positions_with_offsets], key_rot, key_pass
        )
        chunk_len = self.chunk_size - self.local_size
        query = self._apply_rotary_embedding(
            self.cos_sin_q_cache[positions_with_offsets % chunk_len],
            query_rot,
            query_pass,
        )
        query_succ = self._apply_rotary_embedding(
            self.cos_sin_qc_cache[positions_with_offsets % chunk_len],
            query_rot,
            query_pass,
        )
        query_inter = self._apply_rotary_embedding(
            self.cos_sin_qc_cache[chunk_len - 1].repeat(positions.shape[0], 1),
            query_rot,
# ... truncated for analysis ...
        )

        # merge query into one tensor to simplify the interfaces
        query = torch.cat(
            (
                query,
                query_succ,
                query_inter,
                query_succ_critical,
                query_inter_critical,
            ),
            dim=-1,
        )
        return query, key
```
**EN:** Defines function `DualChunkRotaryEmbedding.forward_native` with signature `forward_native(self, positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor, offsets: torch.Tensor | None=None) -> tuple[torch.Tensor, torch.Tensor]`. It mainly works with `positions`, `query`, `key`, `offsets`; adjusts rotary-position-encoding parameters or application logic. The body uses branching, tensor/kernel operations. Key calls include `query.view`, `key.view`, `self._apply_rotary_embedding`, `torch.cat`, `torch.add`, `self.cos_sin_qc_cache.repeat`.
**CN:** 定义函数 `DualChunkRotaryEmbedding.forward_native`，其签名为 `forward_native(self, positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor, offsets: torch.Tensor | None=None) -> tuple[torch.Tensor, torch.Tensor]`。它主要围绕 `positions`, `query`, `key`, `offsets` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含分支判断、张量或内核操作。关键调用包括 `query.view`, `key.view`, `self._apply_rotary_embedding`, `torch.cat`, `torch.add`, `self.cos_sin_qc_cache.repeat`。

### Method `DualChunkRotaryEmbedding.forward_cuda` (lines 186-193)
```python
    def forward_cuda(
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor,
        offsets: torch.Tensor | None = None,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        return self.forward_native(positions, query, key, offsets)
```
**EN:** Defines function `DualChunkRotaryEmbedding.forward_cuda` with signature `forward_cuda(self, positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor, offsets: torch.Tensor | None=None) -> tuple[torch.Tensor, torch.Tensor]`. It mainly works with `positions`, `query`, `key`, `offsets`; adjusts rotary-position-encoding parameters or application logic. The body uses mostly straightforward data movement and object wiring. Key calls include `self.forward_native`.
**CN:** 定义函数 `DualChunkRotaryEmbedding.forward_cuda`，其签名为 `forward_cuda(self, positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor, offsets: torch.Tensor | None=None) -> tuple[torch.Tensor, torch.Tensor]`。它主要围绕 `positions`, `query`, `key`, `offsets` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `self.forward_native`。

### Method `DualChunkRotaryEmbedding._apply_rotary_embedding` (lines 195-210)
```python
    def _apply_rotary_embedding(self, cos_sin, hidden_rot, hidden_pass):
        cos, sin = cos_sin.chunk(2, dim=-1)
        if self.is_neox_style:
            cos = torch.cat((cos, cos), dim=-1).unsqueeze(-2)
            sin = torch.cat((sin, sin), dim=-1).unsqueeze(-2)
        else:
            cos = cos.repeat_interleave(2, dim=-1).unsqueeze(-2)
            sin = sin.repeat_interleave(2, dim=-1).unsqueeze(-2)
        rotate_fn = rotate_neox if self.is_neox_style else rotate_gptj
        hidden_rot = hidden_rot * cos + rotate_fn(hidden_rot) * sin

        if self.rotary_dim < self.head_size:
            hidden = torch.cat((hidden_rot, hidden_pass), dim=-1)
        else:
            hidden = hidden_rot
        return hidden.flatten(-2).squeeze(0)
```
**EN:** Defines function `DualChunkRotaryEmbedding._apply_rotary_embedding` with signature `_apply_rotary_embedding(self, cos_sin, hidden_rot, hidden_pass)`. It mainly works with `cos_sin`, `hidden_rot`, `hidden_pass`; adjusts rotary-position-encoding parameters or application logic. The body uses branching, tensor/kernel operations. Key calls include `cos_sin.chunk`, `hidden.flatten.squeeze`, `torch.cat.unsqueeze`, `cos.repeat_interleave.unsqueeze`, `sin.repeat_interleave.unsqueeze`, `torch.cat`.
**CN:** 定义函数 `DualChunkRotaryEmbedding._apply_rotary_embedding`，其签名为 `_apply_rotary_embedding(self, cos_sin, hidden_rot, hidden_pass)`。它主要围绕 `cos_sin`, `hidden_rot`, `hidden_pass` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含分支判断、张量或内核操作。关键调用包括 `cos_sin.chunk`, `hidden.flatten.squeeze`, `torch.cat.unsqueeze`, `cos.repeat_interleave.unsqueeze`, `sin.repeat_interleave.unsqueeze`, `torch.cat`。

### Method `DualChunkRotaryEmbedding.extra_repr` (lines 212-217)
```python
    def extra_repr(self) -> str:
        s = f"head_size={self.head_size}, rotary_dim={self.rotary_dim}"
        s += f", max_position_embeddings={self.max_position_embeddings}"
        s += f", base={self.base}, is_neox_style={self.is_neox_style}"
        s += f", chunk_size={self.chunk_size}, local_size={self.local_size}"
        return s
```
**EN:** Defines function `DualChunkRotaryEmbedding.extra_repr` with signature `extra_repr(self) -> str`. It mainly works with object context only; adjusts rotary-position-encoding parameters or application logic. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `DualChunkRotaryEmbedding.extra_repr`，其签名为 `extra_repr(self) -> str`。它主要围绕 仅依赖对象上下文 展开；调整旋转位置编码参数或其应用逻辑。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

## Key Concepts / 关键概念
- **EN:** The file implements a RoPE-related variant, meaning frequency scaling, cache construction, and position application are key ideas.
  **CN:** 该文件实现了与 RoPE 相关的变体，因此频率缩放、缓存构建和位置应用是关键概念。
- **EN:** Top-level classes include `DualChunkRotaryEmbedding`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `DualChunkRotaryEmbedding`，它们组织了主要的可复用抽象。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。
- **EN:** Internal vLLM imports show that this module is tightly integrated with the broader runtime, parallelism, or custom-layer stack.
  **CN:** vLLM 内部导入说明该模块与更大的运行时、并行机制或自定义层栈紧密集成。

## Dependencies / 依赖关系
- **External / 外部**: `torch`
- **Internal / 内部**: `vllm.model_executor.custom_op`, `.common`
