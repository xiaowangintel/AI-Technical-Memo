# fope.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/rotary_embedding/fope.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `FourierRotaryEmbedding` and related helpers for rotary positional embedding variants. / 实现 `FourierRotaryEmbedding` 及其相关辅助逻辑，用于旋转位置编码变体。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 3-13)
```python
import torch
import torch.nn.functional as F
from torch import nn

from vllm.distributed import (
    get_tensor_model_parallel_rank,
    get_tensor_model_parallel_world_size,
)

from .base import RotaryEmbedding
from .common import rotate_neox
```
**EN:** This opening block pulls in external dependencies such as `torch` and internal modules such as `vllm.distributed`, `.base`, `.common`. That import mix shows the file is part of the rotary positional embedding variants stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `torch`）以及内部模块（如 `vllm.distributed`, `.base`, `.common`）。这些导入关系表明该文件属于旋转位置编码变体栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Class `FourierRotaryEmbedding` overview (lines 16-199)
```python
class FourierRotaryEmbedding(RotaryEmbedding):
    def __init__(
        self,
        head_size: int,
        rotary_dim: int,
        max_position_embeddings: int,
        base: float,
        is_neox_style: bool,
        dtype: torch.dtype,
        init_cache: bool,
        # extra parameters for FoPE
        num_key_value_heads: int,
        num_inv_freq: int,
        fope_sep_head: bool,
        fope_init_factor: float,
    ):
        # fope related parameters
        self.num_key_value_heads = num_key_value_heads
        self.num_inv_freq = num_inv_freq
        self.fope_sep_head = fope_sep_head
        self.fope_init_factor = fope_init_factor

        super().__init__(
            head_size=head_size,
            rotary_dim=rotary_dim,
```
**EN:** Defines class `FourierRotaryEmbedding` with base classes `RotaryEmbedding` and decorators none. It acts as an embedding layer with model-parallel awareness and exposes 5 direct methods, with notable entries `__init__`, `_compute_inv_freq`, `_compute_cos_sin_cache`, `forward_native`, `weight_loader`.
**CN:** 定义类 `FourierRotaryEmbedding`，其基类为 `RotaryEmbedding`，装饰器为 无。它在整体实现中充当具备模型并行意识的嵌入层，并直接暴露 5 个方法，较重要的包括 `__init__`, `_compute_inv_freq`, `_compute_cos_sin_cache`, `forward_native`, `weight_loader`。

### Method `FourierRotaryEmbedding.__init__` (lines 17-72)
```python
    def __init__(
        self,
        head_size: int,
        rotary_dim: int,
        max_position_embeddings: int,
        base: float,
        is_neox_style: bool,
        dtype: torch.dtype,
        init_cache: bool,
        # extra parameters for FoPE
        num_key_value_heads: int,
        num_inv_freq: int,
        fope_sep_head: bool,
        fope_init_factor: float,
    ):
        # fope related parameters
        self.num_key_value_heads = num_key_value_heads
        self.num_inv_freq = num_inv_freq
        self.fope_sep_head = fope_sep_head
        self.fope_init_factor = fope_init_factor

        super().__init__(
            head_size=head_size,
            rotary_dim=rotary_dim,
            max_position_embeddings=max_position_embeddings,
            base=base,
            is_neox_style=is_neox_style,
            dtype=dtype,
            init_cache=init_cache,
        )

        # setup buffers and parameters
        self.inv_freq: torch.Tensor
        self.register_buffer(
            "inv_freq", self._compute_inv_freq(self.base), persistent=False
        )

        self.input_dim = self.inv_freq.shape[-1]
        self.output_dim = self.inv_freq.shape[-1]
        self.cos_coef = nn.Parameter(
            torch.empty(num_key_value_heads, self.input_dim, self.output_dim),
            requires_grad=False,
        )
        self.sin_coef = nn.Parameter(
            torch.empty(num_key_value_heads, self.input_dim, self.output_dim),
            requires_grad=False,
        )
        self.sin_coef.weight_loader = self.weight_loader
        self.cos_coef.weight_loader = self.weight_loader

        self.cos_sin_cache: torch.Tensor
        cache = self._compute_cos_sin_cache().to(dtype)
        self.register_buffer("cos_sin_cache", cache, persistent=False)

        # update cache in the first forward, where sin/cos_coef weights are ready
        self.update_cache = True
```
**EN:** Defines function `FourierRotaryEmbedding.__init__` with signature `__init__(self, head_size: int, rotary_dim: int, max_position_embeddings: int, base: float, is_neox_style: bool, dtype: torch.dtype, init_cache: bool, num_key_value_heads: int, num_inv_freq: int, fope_sep_head: bool, fope_init_factor: float)`. It mainly works with `head_size`, `rotary_dim`, `max_position_embeddings`, `base`, `is_neox_style`, `dtype`, `init_cache`, `num_key_value_heads`; initializes the object state and cached resources. The body uses tensor/kernel operations. Key calls include `super.__init__`, `self.register_buffer`, `nn.Parameter`, `self._compute_cos_sin_cache.to`, `self._compute_inv_freq`, `torch.empty`.
**CN:** 定义函数 `FourierRotaryEmbedding.__init__`，其签名为 `__init__(self, head_size: int, rotary_dim: int, max_position_embeddings: int, base: float, is_neox_style: bool, dtype: torch.dtype, init_cache: bool, num_key_value_heads: int, num_inv_freq: int, fope_sep_head: bool, fope_init_factor: float)`。它主要围绕 `head_size`, `rotary_dim`, `max_position_embeddings`, `base`, `is_neox_style`, `dtype`, `init_cache`, `num_key_value_heads` 展开；负责初始化对象状态和缓存资源。函数体包含张量或内核操作。关键调用包括 `super.__init__`, `self.register_buffer`, `nn.Parameter`, `self._compute_cos_sin_cache.to`, `self._compute_inv_freq`, `torch.empty`。

### Method `FourierRotaryEmbedding._compute_inv_freq` (lines 74-92)
```python
    def _compute_inv_freq(self, base: float) -> torch.Tensor:
        """Compute the inverse frequency."""
        inv_freq = 1.0 / (
            base
            ** (
                torch.arange(0, self.rotary_dim, 2, dtype=torch.float) / self.rotary_dim
            )
        )

        inv_freq_idx_selected = torch.ones_like(inv_freq, dtype=torch.bool)
        if self.num_inv_freq is not None:
            inv_freq_idx_selected[self.num_inv_freq :] = False
        else:
            inv_freq_idx_selected = inv_freq > (
                2.0 * torch.pi / self.max_position_embeddings
            )

        inv_freq = inv_freq[inv_freq_idx_selected]
        return inv_freq
```
**EN:** Defines function `FourierRotaryEmbedding._compute_inv_freq` with signature `_compute_inv_freq(self, base: float) -> torch.Tensor`. It mainly works with `base`; adjusts rotary-position-encoding parameters or application logic. The body uses branching, tensor/kernel operations. Key calls include `torch.ones_like`, `torch.arange`.
**CN:** 定义函数 `FourierRotaryEmbedding._compute_inv_freq`，其签名为 `_compute_inv_freq(self, base: float) -> torch.Tensor`。它主要围绕 `base` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含分支判断、张量或内核操作。关键调用包括 `torch.ones_like`, `torch.arange`。

### Method `FourierRotaryEmbedding._compute_cos_sin_cache` (lines 94-132)
```python
    def _compute_cos_sin_cache(self) -> torch.Tensor:
        """Compute the cos and sin cache."""
        device = self.inv_freq.device
        t = torch.arange(self.max_position_embeddings, dtype=torch.float, device=device)

        freqs = torch.einsum("j,i -> ji", t, self.inv_freq)
        if self.fope_sep_head:
            pos_cos = freqs.cos().unsqueeze(0).expand(self.num_key_value_heads, -1, -1)
            pos_sin = freqs.sin().unsqueeze(0).expand(self.num_key_value_heads, -1, -1)
        else:
            pos_cos = freqs.cos()
            pos_sin = freqs.sin()

        if self.fope_sep_head:
            sin = torch.einsum("htD, hDd -> thd", pos_sin, self.sin_coef.float())
            cos = torch.einsum("htD, hDd -> thd", pos_cos, self.cos_coef.float())
        else:
            sin = torch.einsum("tD, Dd -> td", pos_sin, self.sin_coef.float())
            cos = torch.einsum("tD, Dd -> td", pos_cos, self.cos_coef.float())

        sin = F.pad(
            input=sin,
            pad=(0, self.head_size // 2 - sin.size(-1)),
            mode="constant",
            value=1,
        )
        cos = F.pad(
            input=cos,
            pad=(0, self.head_size // 2 - cos.size(-1)),
            mode="constant",
            value=1,
        )

        sin = torch.cat((sin, sin), dim=-1)
        cos = torch.cat((cos, cos), dim=-1)

        # cache: (max_position_embeddings, num_kv_heads, kv_size * 2)
        cache = torch.cat((cos, sin), dim=-1)
        return cache
```
**EN:** Defines function `FourierRotaryEmbedding._compute_cos_sin_cache` with signature `_compute_cos_sin_cache(self) -> torch.Tensor`. It mainly works with object context only; adjusts rotary-position-encoding parameters or application logic. The body uses branching, tensor/kernel operations. Key calls include `torch.arange`, `torch.einsum`, `F.pad`, `torch.cat`, `freqs.cos.unsqueeze.expand`, `freqs.sin.unsqueeze.expand`.
**CN:** 定义函数 `FourierRotaryEmbedding._compute_cos_sin_cache`，其签名为 `_compute_cos_sin_cache(self) -> torch.Tensor`。它主要围绕 仅依赖对象上下文 展开；调整旋转位置编码参数或其应用逻辑。函数体包含分支判断、张量或内核操作。关键调用包括 `torch.arange`, `torch.einsum`, `F.pad`, `torch.cat`, `freqs.cos.unsqueeze.expand`, `freqs.sin.unsqueeze.expand`。

### Method `FourierRotaryEmbedding.forward_native` (lines 134-185)
```python
    def forward_native(
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor | None = None,
        offsets: torch.Tensor | None = None,
    ) -> tuple[torch.Tensor, torch.Tensor | None]:
        # update cos/sin cache in the first forward
        if self.update_cache:
            cache = self._compute_cos_sin_cache().to(self.dtype)
            self.cos_sin_cache.copy_(cache)
            self.update_cache = False

        positions = positions.flatten()
        cos_sin = self.cos_sin_cache.index_select(0, positions)
        cos, sin = cos_sin.chunk(2, dim=-1)

        # apply rotary embedding
        # query: (seq_len, num_heads, head_size)
        # key: (seq_len, num_kv_heads, head_size)
        query = query.unflatten(-1, (-1, self.head_size))
        assert key is not None, "Key tensor is required for FoPE."
        key = key.unflatten(-1, (-1, self.head_size))

        assert query.dim() == key.dim() == 3, (
            "Expected query key (seq_len, heads, head_dim)"
        )
        assert cos.dim() <= 3 and sin.dim() <= 3

        need_reshape = False
        if cos.dim() == 3:
            # for fope
            need_reshape = True
            query_shape = query.shape
            key_shape = key.shape
            cos = cos.flatten(0, 1)
            sin = sin.flatten(0, 1)
            seq_len = cos.size(0)
            query = query.view(seq_len, -1, query.size(-1))
            key = key.view(seq_len, -1, key.size(-1))

        # native implementation of apply rope for neox style
        cos = cos.unsqueeze(1)
        sin = sin.unsqueeze(1)
        query = (query * cos) + (rotate_neox(query) * sin)
        key = (key * cos) + (rotate_neox(key) * sin)

        if need_reshape:
            query = query.view(query_shape)
            key = key.view(key_shape)

        return query, key
```
**EN:** Defines function `FourierRotaryEmbedding.forward_native` with signature `forward_native(self, positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor | None=None, offsets: torch.Tensor | None=None) -> tuple[torch.Tensor, torch.Tensor | None]`. It mainly works with `positions`, `query`, `key`, `offsets`; adjusts rotary-position-encoding parameters or application logic. The body uses branching, validation/error handling. Key calls include `positions.flatten`, `self.cos_sin_cache.index_select`, `cos_sin.chunk`, `query.unflatten`, `key.unflatten`, `cos.unsqueeze`.
**CN:** 定义函数 `FourierRotaryEmbedding.forward_native`，其签名为 `forward_native(self, positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor | None=None, offsets: torch.Tensor | None=None) -> tuple[torch.Tensor, torch.Tensor | None]`。它主要围绕 `positions`, `query`, `key`, `offsets` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含分支判断、校验或报错逻辑。关键调用包括 `positions.flatten`, `self.cos_sin_cache.index_select`, `cos_sin.chunk`, `query.unflatten`, `key.unflatten`, `cos.unsqueeze`。

### Method `FourierRotaryEmbedding.weight_loader` (lines 187-199)
```python
    def weight_loader(self, param: nn.Parameter, loaded_weight: torch.Tensor):
        """load fope weights"""
        world_size = get_tensor_model_parallel_world_size()
        rank = get_tensor_model_parallel_rank()
        num_key_value_heads = loaded_weight.size(0)

        if num_key_value_heads < world_size:
            n_replicate = world_size // num_key_value_heads
            world_size = num_key_value_heads
            rank = rank // n_replicate

        loaded_weight = loaded_weight.chunk(world_size, dim=0)[rank]
        param.data.copy_(loaded_weight)
```
**EN:** Defines function `FourierRotaryEmbedding.weight_loader` with signature `weight_loader(self, param: nn.Parameter, loaded_weight: torch.Tensor)`. It mainly works with `param`, `loaded_weight`; adjusts rotary-position-encoding parameters or application logic. The body uses branching. Key calls include `get_tensor_model_parallel_world_size`, `get_tensor_model_parallel_rank`, `loaded_weight.size`, `param.data.copy_`, `loaded_weight.chunk`.
**CN:** 定义函数 `FourierRotaryEmbedding.weight_loader`，其签名为 `weight_loader(self, param: nn.Parameter, loaded_weight: torch.Tensor)`。它主要围绕 `param`, `loaded_weight` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含分支判断。关键调用包括 `get_tensor_model_parallel_world_size`, `get_tensor_model_parallel_rank`, `loaded_weight.size`, `param.data.copy_`, `loaded_weight.chunk`。

## Key Concepts / 关键概念
- **EN:** The file implements a RoPE-related variant, meaning frequency scaling, cache construction, and position application are key ideas.
  **CN:** 该文件实现了与 RoPE 相关的变体，因此频率缩放、缓存构建和位置应用是关键概念。
- **EN:** Top-level classes include `FourierRotaryEmbedding`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `FourierRotaryEmbedding`，它们组织了主要的可复用抽象。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。
- **EN:** Internal vLLM imports show that this module is tightly integrated with the broader runtime, parallelism, or custom-layer stack.
  **CN:** vLLM 内部导入说明该模块与更大的运行时、并行机制或自定义层栈紧密集成。

## Dependencies / 依赖关系
- **External / 外部**: `torch`
- **Internal / 内部**: `vllm.distributed`, `.base`, `.common`
