# gemma4_rope.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/rotary_embedding/gemma4_rope.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `Gemma4RotaryEmbedding` and related helpers for rotary positional embedding variants. / 实现 `Gemma4RotaryEmbedding` 及其相关辅助逻辑，用于旋转位置编码变体。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-9)
```python
"""Gemma4-specific Rotary Positional Embeddings (proportional scaling).

Gemma4 uses "proportional" RoPE which computes inv_freq frequencies scaled
by head_dim (not rotary_dim), and zero-pads for non-rotated dimensions when
partial_rotary_factor < 1. The actual rotation uses standard neox-style
rotate_half, matching HF transformers' apply_rotary_pos_emb.
"""
```
**EN:** This docstring gives the module author's high-level intent: Gemma4-specific Rotary Positional Embeddings (proportional scaling). It is useful for reading the rest of the file because later classes and helpers refine this stated purpose.
**CN:** 这个文档字符串给出了模块作者的高层意图：Gemma4-specific Rotary Positional Embeddings (proportional scaling). 在阅读后续类和辅助函数时，可以把它视为整个文件的总纲。

### Imports and module setup (lines 11-13)
```python
import torch

from .base import RotaryEmbedding
```
**EN:** This opening block pulls in external dependencies such as `torch` and internal modules such as `.base`. That import mix shows the file is part of the rotary positional embedding variants stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `torch`）以及内部模块（如 `.base`）。这些导入关系表明该文件属于旋转位置编码变体栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Class `Gemma4RotaryEmbedding` overview (lines 16-84)
```python
class Gemma4RotaryEmbedding(RotaryEmbedding):
    """Gemma4 proportional RoPE.

    Extends RotaryEmbedding (which provides standard neox-style rotation
    via ops.rotary_embedding CUDA kernel) but overrides the inv_freq
    computation to match HF's _compute_proportional_rope_parameters:
    - Frequency exponents use head_dim (not rotary_dim) as denominator
    - Non-rotated dims are zero-padded (cos=1, sin=0 = identity rotation)

    When partial_rotary_factor=1.0 (the default for some variants), ALL dims are
    rotated and this is equivalent to standard RotaryEmbedding with
    head_dim-scaled frequencies.
    """

    def __init__(
        self,
        head_size: int,
        rotary_dim: int,
        max_position_embeddings: int,
        base: float,
        is_neox_style: bool,
        dtype: torch.dtype,
    ) -> None:
        # Number of rotation angle pairs (from partial_rotary_factor)
        self.rope_angles = rotary_dim // 2
```
**EN:** Defines class `Gemma4RotaryEmbedding` with base classes `RotaryEmbedding` and decorators none. It acts as an embedding layer with model-parallel awareness and exposes 3 direct methods, with notable entries `__init__`, `_compute_inv_freq`, `extra_repr`. Its docstring says: Gemma4 proportional RoPE.
**CN:** 定义类 `Gemma4RotaryEmbedding`，其基类为 `RotaryEmbedding`，装饰器为 无。它在整体实现中充当具备模型并行意识的嵌入层，并直接暴露 3 个方法，较重要的包括 `__init__`, `_compute_inv_freq`, `extra_repr`。 文档字符串进一步说明了该类的定位。

### Method `Gemma4RotaryEmbedding.__init__` (lines 30-55)
```python
    def __init__(
        self,
        head_size: int,
        rotary_dim: int,
        max_position_embeddings: int,
        base: float,
        is_neox_style: bool,
        dtype: torch.dtype,
    ) -> None:
        # Number of rotation angle pairs (from partial_rotary_factor)
        self.rope_angles = rotary_dim // 2
        # Non-rotated angle pairs per half
        self.nope_angles = (head_size // 2) - self.rope_angles

        # Important: set rotary_dim = head_size so the base class's
        # forward_static applies rotation to ALL dims of the cos/sin cache.
        # The non-rotated dims will have cos=1, sin=0 (identity) thanks
        # to our _compute_inv_freq zero-padding.
        super().__init__(
            head_size,
            head_size,  # rotary_dim = head_size (full application)
            max_position_embeddings,
            base,
            is_neox_style,
            dtype,
        )
```
**EN:** Defines function `Gemma4RotaryEmbedding.__init__` with signature `__init__(self, head_size: int, rotary_dim: int, max_position_embeddings: int, base: float, is_neox_style: bool, dtype: torch.dtype) -> None`. It mainly works with `head_size`, `rotary_dim`, `max_position_embeddings`, `base`, `is_neox_style`, `dtype`; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include `super.__init__`, `super`.
**CN:** 定义函数 `Gemma4RotaryEmbedding.__init__`，其签名为 `__init__(self, head_size: int, rotary_dim: int, max_position_embeddings: int, base: float, is_neox_style: bool, dtype: torch.dtype) -> None`。它主要围绕 `head_size`, `rotary_dim`, `max_position_embeddings`, `base`, `is_neox_style`, `dtype` 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `super.__init__`, `super`。

### Method `Gemma4RotaryEmbedding._compute_inv_freq` (lines 57-77)
```python
    def _compute_inv_freq(self, base: float) -> torch.Tensor:
        """Compute frequencies matching HF proportional RoPE.

        Key difference from base: exponent denominator is head_size (not
        rotary_dim), and non-rotated dims are zero-padded.
        """
        # HF formula: base ** (arange(0, 2*rope_angles, 2) / head_dim)
        freq_exponents = (
            torch.arange(0, 2 * self.rope_angles, 2, dtype=torch.float) / self.head_size
        )
        inv_freq = 1.0 / (base**freq_exponents)

        # Zero-pad for non-rotated dims (identity rotation: cos=1, sin=0)
        if self.nope_angles > 0:
            inv_freq = torch.cat(
                [
                    inv_freq,
                    torch.zeros(self.nope_angles, dtype=torch.float),
                ]
            )
        return inv_freq
```
**EN:** Defines function `Gemma4RotaryEmbedding._compute_inv_freq` with signature `_compute_inv_freq(self, base: float) -> torch.Tensor`. It mainly works with `base`; adjusts rotary-position-encoding parameters or application logic. The body uses branching, tensor/kernel operations. Key calls include `torch.arange`, `torch.cat`, `torch.zeros`.
**CN:** 定义函数 `Gemma4RotaryEmbedding._compute_inv_freq`，其签名为 `_compute_inv_freq(self, base: float) -> torch.Tensor`。它主要围绕 `base` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含分支判断、张量或内核操作。关键调用包括 `torch.arange`, `torch.cat`, `torch.zeros`。

### Method `Gemma4RotaryEmbedding.extra_repr` (lines 79-84)
```python
    def extra_repr(self) -> str:
        s = f"head_size={self.head_size}, rotary_dim={self.rotary_dim}"
        s += f", rope_angles={self.rope_angles}, nope_angles={self.nope_angles}"
        s += f", max_position_embeddings={self.max_position_embeddings}"
        s += f", base={self.base}, is_neox_style={self.is_neox_style}"
        return s
```
**EN:** Defines function `Gemma4RotaryEmbedding.extra_repr` with signature `extra_repr(self) -> str`. It mainly works with object context only; adjusts rotary-position-encoding parameters or application logic. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `Gemma4RotaryEmbedding.extra_repr`，其签名为 `extra_repr(self) -> str`。它主要围绕 仅依赖对象上下文 展开；调整旋转位置编码参数或其应用逻辑。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

## Key Concepts / 关键概念
- **EN:** The file implements a RoPE-related variant, meaning frequency scaling, cache construction, and position application are key ideas.
  **CN:** 该文件实现了与 RoPE 相关的变体，因此频率缩放、缓存构建和位置应用是关键概念。
- **EN:** Top-level classes include `Gemma4RotaryEmbedding`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `Gemma4RotaryEmbedding`，它们组织了主要的可复用抽象。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。
- **EN:** Internal vLLM imports show that this module is tightly integrated with the broader runtime, parallelism, or custom-layer stack.
  **CN:** vLLM 内部导入说明该模块与更大的运行时、并行机制或自定义层栈紧密集成。

## Dependencies / 依赖关系
- **External / 外部**: `torch`
- **Internal / 内部**: `.base`
