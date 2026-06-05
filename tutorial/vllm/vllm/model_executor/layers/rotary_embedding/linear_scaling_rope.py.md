# linear_scaling_rope.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/rotary_embedding/linear_scaling_rope.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `LinearScalingRotaryEmbedding` and related helpers for rotary positional embedding variants. / 实现 `LinearScalingRotaryEmbedding` 及其相关辅助逻辑，用于旋转位置编码变体。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 26-28)
```python
import torch

from .base import RotaryEmbedding
```
**EN:** This opening block pulls in external dependencies such as `torch` and internal modules such as `.base`. That import mix shows the file is part of the rotary positional embedding variants stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `torch`）以及内部模块（如 `.base`）。这些导入关系表明该文件属于旋转位置编码变体栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Class `LinearScalingRotaryEmbedding` overview (lines 31-115)
```python
class LinearScalingRotaryEmbedding(RotaryEmbedding):
    """RotaryEmbedding extended with linear scaling.

    It supports multiple scaling factors. Since multiple LoRA adapters may have
    different scaling factors, we need multiple cos/sin caches. In this way,
    instead of running rotary embedding kernel per lora, we can run multiple
    lora in a batched way.

    In addition to that, we also keep the cos/sin cache for the scaling factor
    of 1 (default) at all times.

    Exemplary for two scaling factors x=1, y and z with embeddings
    [[x11, x12, ... x1m], ..., [xn1, xn2, ..., xnm]] and
    [[y11, y12, ... y1o], ..., [yn1, yn2, ..., yno]], and
    [[z11, z12, ... z1p], ..., [zn1, zn2, ..., znp]],

    we construct the cos/sin cache as follows:
    [[x11, x12, ... x1m, y11, y12, ... y1o, z11, z12, ... z1p],
        ...
     [xn1, xn2, ... xnm, yn1, yn2, ... yno, zn1, zn2, ... znp]]

    We then use offsets to index into the cos/sin cache for
    the respective scaling factors.

    The offset to cache can be accessed via `scaling_factor_to_offset` API.
```
**EN:** Defines class `LinearScalingRotaryEmbedding` with base classes `RotaryEmbedding` and decorators none. It acts as an embedding layer with model-parallel awareness and exposes 3 direct methods, with notable entries `__init__`, `_compute_cos_sin_cache`, `scaling_factor_to_offset`. Its docstring says: RotaryEmbedding extended with linear scaling.
**CN:** 定义类 `LinearScalingRotaryEmbedding`，其基类为 `RotaryEmbedding`，装饰器为 无。它在整体实现中充当具备模型并行意识的嵌入层，并直接暴露 3 个方法，较重要的包括 `__init__`, `_compute_cos_sin_cache`, `scaling_factor_to_offset`。 文档字符串进一步说明了该类的定位。

### Method `LinearScalingRotaryEmbedding.__init__` (lines 60-77)
```python
    def __init__(
        self,
        head_size: int,
        rotary_dim: int,
        max_position_embeddings: int,
        base: float,
        is_neox_style: bool,
        scaling_factors: list[float] | float,
        dtype: torch.dtype,
    ) -> None:
        if isinstance(scaling_factors, float):
            scaling_factors = [scaling_factors]
        self.scaling_factors: list[float] = scaling_factors  # noqa
        super().__init__(
            head_size, rotary_dim, max_position_embeddings, base, is_neox_style, dtype
        )
        # Lazy initialized.
        self._scaling_factor_to_offset: dict[float, int]
```
**EN:** Defines function `LinearScalingRotaryEmbedding.__init__` with signature `__init__(self, head_size: int, rotary_dim: int, max_position_embeddings: int, base: float, is_neox_style: bool, scaling_factors: list[float] | float, dtype: torch.dtype) -> None`. It mainly works with `head_size`, `rotary_dim`, `max_position_embeddings`, `base`, `is_neox_style`, `scaling_factors`, `dtype`; initializes the object state and cached resources. The body uses branching. Key calls include `isinstance`, `super.__init__`, `super`.
**CN:** 定义函数 `LinearScalingRotaryEmbedding.__init__`，其签名为 `__init__(self, head_size: int, rotary_dim: int, max_position_embeddings: int, base: float, is_neox_style: bool, scaling_factors: list[float] | float, dtype: torch.dtype) -> None`。它主要围绕 `head_size`, `rotary_dim`, `max_position_embeddings`, `base`, `is_neox_style`, `scaling_factors`, `dtype` 展开；负责初始化对象状态和缓存资源。函数体包含分支判断。关键调用包括 `isinstance`, `super.__init__`, `super`。

### Method `LinearScalingRotaryEmbedding._compute_cos_sin_cache` (lines 79-111)
```python
    def _compute_cos_sin_cache(self) -> torch.Tensor:
        inv_freq = self._compute_inv_freq(self.base)
        cache_list: list[torch.Tensor] = []
        # offsets to the next cache in a tensor.
        # Each offset corresponds to the same index in scaling_factors.
        offsets: list[int] = []
        for scaling_factor in self.scaling_factors:
            # NOTE(woosuk): self.max_position_embeddings is the original
            # maximum length before applying the rope scaling.
            # Thus, the maximum length after applying the rope scaling is
            # self.max_position_embeddings * self.scaling_factor.
            max_len = self.max_position_embeddings * scaling_factor
            t = torch.arange(max_len, dtype=torch.float)
            t = t / scaling_factor

            freqs = torch.einsum("i,j -> ij", t, inv_freq)
            cos = freqs.cos()
            sin = freqs.sin()
            cache = torch.cat((cos, sin), dim=-1)
            if not cache_list:
                offset = 0
            else:
                last_offset = offsets[-1]
                next_max_len = cache_list[-1].shape[0]
                offset = last_offset + next_max_len
            offsets.append(offset)
            cache_list.append(cache)
        self._scaling_factor_to_offset = {
            float(scaling_factor): offsets[i]
            for i, scaling_factor in enumerate(self.scaling_factors)
        }
        assert len(self.scaling_factors) == len(offsets)
        return torch.cat(cache_list, dim=0)
```
**EN:** Defines function `LinearScalingRotaryEmbedding._compute_cos_sin_cache` with signature `_compute_cos_sin_cache(self) -> torch.Tensor`. It mainly works with object context only; adjusts rotary-position-encoding parameters or application logic. The body uses branching, iteration, comprehensions, validation/error handling, tensor/kernel operations. Key calls include `self._compute_inv_freq`, `torch.cat`, `torch.arange`, `torch.einsum`, `freqs.cos`, `freqs.sin`.
**CN:** 定义函数 `LinearScalingRotaryEmbedding._compute_cos_sin_cache`，其签名为 `_compute_cos_sin_cache(self) -> torch.Tensor`。它主要围绕 仅依赖对象上下文 展开；调整旋转位置编码参数或其应用逻辑。函数体包含分支判断、循环处理、推导式、校验或报错逻辑、张量或内核操作。关键调用包括 `self._compute_inv_freq`, `torch.cat`, `torch.arange`, `torch.einsum`, `freqs.cos`, `freqs.sin`。

### Method `LinearScalingRotaryEmbedding.scaling_factor_to_offset` (lines 114-115)
```python
    def scaling_factor_to_offset(self) -> dict[float, int]:
        return self._scaling_factor_to_offset
```
**EN:** Defines function `LinearScalingRotaryEmbedding.scaling_factor_to_offset` with signature `scaling_factor_to_offset(self) -> dict[float, int]`. It mainly works with object context only; adjusts rotary-position-encoding parameters or application logic. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `LinearScalingRotaryEmbedding.scaling_factor_to_offset`，其签名为 `scaling_factor_to_offset(self) -> dict[float, int]`。它主要围绕 仅依赖对象上下文 展开；调整旋转位置编码参数或其应用逻辑。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

## Key Concepts / 关键概念
- **EN:** The file implements a RoPE-related variant, meaning frequency scaling, cache construction, and position application are key ideas.
  **CN:** 该文件实现了与 RoPE 相关的变体，因此频率缩放、缓存构建和位置应用是关键概念。
- **EN:** Top-level classes include `LinearScalingRotaryEmbedding`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `LinearScalingRotaryEmbedding`，它们组织了主要的可复用抽象。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。
- **EN:** Internal vLLM imports show that this module is tightly integrated with the broader runtime, parallelism, or custom-layer stack.
  **CN:** vLLM 内部导入说明该模块与更大的运行时、并行机制或自定义层栈紧密集成。

## Dependencies / 依赖关系
- **External / 外部**: `torch`
- **Internal / 内部**: `.base`
