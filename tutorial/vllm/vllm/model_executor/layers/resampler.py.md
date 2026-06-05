# resampler.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/resampler.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements feature resampling layers that reshape external features into model hidden space. / 实现把外部特征重采样到模型隐藏空间的层。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 29-34)
```python
"""
Shared resampler perceiver network used in multimodal models and
related helpers for sincos positional embeddings.

Example models: Qwen (Qwen-VL), MiniCPM-V 2.0
"""
```
**EN:** This docstring gives the module author's high-level intent: Shared resampler perceiver network used in multimodal models and related helpers for sincos positional embeddings. It is useful for reading the rest of the file because later classes and helpers refine this stated purpose.
**CN:** 这个文档字符串给出了模块作者的高层意图：Shared resampler perceiver network used in multimodal models and related helpers for sincos positional embeddings. 在阅读后续类和辅助函数时，可以把它视为整个文件的总纲。

### Imports and module setup (lines 36-46)
```python
import math
from collections.abc import Callable
from functools import partial

import numpy as np
import torch
import torch.nn.functional as F
from torch import nn

from vllm.model_executor.layers.linear import ReplicatedLinear
from vllm.model_executor.layers.quantization import QuantizationConfig
```
**EN:** This opening block pulls in external dependencies such as `math`, `collections`, `functools`, `numpy`, `torch` and internal modules such as `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.quantization`. That import mix shows the file is part of the model-executor layers and helpers stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `math`, `collections`, `functools`, `numpy`, `torch`）以及内部模块（如 `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.quantization`）。这些导入关系表明该文件属于模型执行层与辅助组件栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Module constants and metadata (lines 48-48)
```python
DEFAULT_LN = partial(nn.LayerNorm, eps=1e-6)
```
**EN:** This block defines module-level metadata or constants such as `DEFAULT_LN`. Those symbols usually capture defaults, aliases, export lists, or registry data that later classes/functions reuse inside the model-executor layers and helpers pipeline.
**CN:** 这个代码块定义了模块级元数据或常量，例如 `DEFAULT_LN`。这些符号通常用于保存默认值、别名、导出列表或注册信息，供后续类和函数在模型执行层与辅助组件流程中复用。

### Function `get_abs_pos` (lines 51-71)
```python
def get_abs_pos(abs_pos: torch.Tensor, tgt_size: torch.Tensor | int) -> torch.Tensor:
    # abs_pos: L, C
    # tgt_size: (H, W)
    # return: M, C
    src_size = int(math.sqrt(abs_pos.size(0)))
    dtype = abs_pos.dtype
    if isinstance(tgt_size, int):
        tgt_size = (tgt_size, tgt_size)
    if src_size == tgt_size[0] and src_size == tgt_size[1]:
        return abs_pos
    return (
        F.interpolate(
            abs_pos.float().reshape(1, src_size, src_size, -1).permute(0, 3, 1, 2),
            size=(tgt_size[0], tgt_size[1]),
            mode="bicubic",
            align_corners=False,
        )
        .permute(0, 2, 3, 1)
        .flatten(0, 2)
        .to(dtype=dtype)
    )
```
**EN:** Defines function `get_abs_pos` with signature `get_abs_pos(abs_pos: torch.Tensor, tgt_size: torch.Tensor | int) -> torch.Tensor`. It mainly works with `abs_pos`, `tgt_size`; returns a derived property or capability check. The body uses branching. Key calls include `int`, `isinstance`, `F.interpolate.permute.flatten.to`, `math.sqrt`, `abs_pos.size`, `F.interpolate.permute.flatten`.
**CN:** 定义函数 `get_abs_pos`，其签名为 `get_abs_pos(abs_pos: torch.Tensor, tgt_size: torch.Tensor | int) -> torch.Tensor`。它主要围绕 `abs_pos`, `tgt_size` 展开；返回派生属性或能力判断结果。函数体包含分支判断。关键调用包括 `int`, `isinstance`, `F.interpolate.permute.flatten.to`, `math.sqrt`, `abs_pos.size`, `F.interpolate.permute.flatten`。

### Function `get_1d_sincos_pos_embed_from_grid` (lines 76-100)
```python
def get_1d_sincos_pos_embed_from_grid(
    embed_dim: int, pos: np.ndarray, version: tuple[int, int] = (2, 0)
) -> torch.Tensor:
    """
    embed_dim: output dimension for each position
    pos: a list of positions to be encoded: size (M,) / (H, W)
    out: (M, D) / (H, W, D)
    """
    assert embed_dim % 2 == 0
    omega = np.arange(embed_dim // 2, dtype=np.float32)
    omega /= embed_dim / 2.0
    omega = 1.0 / 10000**omega  # (D/2,)

    if version == (2, 0):
        pos = pos.reshape(-1)  # (M,)
        out = np.einsum("m,d->md", pos, omega)  # (M, D/2), outer product
        emb_sin = np.sin(out)  # (M, D/2)
        emb_cos = np.cos(out)  # (M, D/2)
        emb = np.concatenate([emb_sin, emb_cos], axis=1)  # (M, D)
    else:
        out = np.einsum("hw,d->hwd", pos, omega)  # (H, W, D/2), outer product
        emb_sin = np.sin(out)  # (H, W, D/2)
        emb_cos = np.cos(out)  # (H, W, D/2)
        emb = np.concatenate([emb_sin, emb_cos], axis=-1)  # (H, W, D)
    return emb
```
**EN:** Defines function `get_1d_sincos_pos_embed_from_grid` with signature `get_1d_sincos_pos_embed_from_grid(embed_dim: int, pos: np.ndarray, version: tuple[int, int]=(2, 0)) -> torch.Tensor`. It mainly works with `embed_dim`, `pos`, `version`; returns a derived property or capability check. The body uses branching, validation/error handling. Key calls include `np.arange`, `pos.reshape`, `np.einsum`, `np.sin`, `np.cos`, `np.concatenate`.
**CN:** 定义函数 `get_1d_sincos_pos_embed_from_grid`，其签名为 `get_1d_sincos_pos_embed_from_grid(embed_dim: int, pos: np.ndarray, version: tuple[int, int]=(2, 0)) -> torch.Tensor`。它主要围绕 `embed_dim`, `pos`, `version` 展开；返回派生属性或能力判断结果。函数体包含分支判断、校验或报错逻辑。关键调用包括 `np.arange`, `pos.reshape`, `np.einsum`, `np.sin`, `np.cos`, `np.concatenate`。

### Function `get_2d_sincos_pos_embed_from_grid` (lines 103-120)
```python
def get_2d_sincos_pos_embed_from_grid(
    embed_dim: int, grid: np.ndarray, version: tuple[int, int] = (2, 0)
) -> torch.Tensor:
    assert embed_dim % 2 == 0

    # use half of dimensions to encode grid_h
    emb_h = get_1d_sincos_pos_embed_from_grid(
        embed_dim // 2, grid[0], version
    )  # (H*W, D/2) or (H, W, D/2)
    emb_w = get_1d_sincos_pos_embed_from_grid(
        embed_dim // 2, grid[1], version
    )  # (H*W, D/2) or (H, W, D/2)

    if version == (2, 0):
        emb = np.concatenate([emb_h, emb_w], axis=1)  # (H*W, D)
    else:
        emb = np.concatenate([emb_h, emb_w], axis=-1)  # (H, W, D)
    return emb
```
**EN:** Defines function `get_2d_sincos_pos_embed_from_grid` with signature `get_2d_sincos_pos_embed_from_grid(embed_dim: int, grid: np.ndarray, version: tuple[int, int]=(2, 0)) -> torch.Tensor`. It mainly works with `embed_dim`, `grid`, `version`; returns a derived property or capability check. The body uses branching, validation/error handling. Key calls include `get_1d_sincos_pos_embed_from_grid`, `np.concatenate`.
**CN:** 定义函数 `get_2d_sincos_pos_embed_from_grid`，其签名为 `get_2d_sincos_pos_embed_from_grid(embed_dim: int, grid: np.ndarray, version: tuple[int, int]=(2, 0)) -> torch.Tensor`。它主要围绕 `embed_dim`, `grid`, `version` 展开；返回派生属性或能力判断结果。函数体包含分支判断、校验或报错逻辑。关键调用包括 `get_1d_sincos_pos_embed_from_grid`, `np.concatenate`。

### Function `get_2d_sincos_pos_embed` (lines 123-153)
```python
def get_2d_sincos_pos_embed(
    embed_dim: int,
    grid_size: int | tuple[int, int],
    cls_token: bool = False,
    version: tuple[int, int] = (2, 0),
) -> torch.Tensor:
    """
    grid_size: int of the grid height and width
    return:
    pos_embed: [grid_size*grid_size, embed_dim] or
                [1+grid_size*grid_size, embed_dim] (w/ or w/o cls_token)
    """
    if isinstance(grid_size, int):
        grid_h_size, grid_w_size = grid_size, grid_size
    else:
        grid_h_size, grid_w_size = grid_size[0], grid_size[1]

    grid_h = np.arange(grid_h_size, dtype=np.float32)
    grid_w = np.arange(grid_w_size, dtype=np.float32)
    grid = np.meshgrid(grid_w, grid_h)  # here w goes first
    grid = np.stack(grid, axis=0)
    assert isinstance(grid, np.ndarray) and grid.shape == (2, grid_h_size, grid_w_size)

    if version == (2, 0):
        grid = grid.reshape([2, 1, grid_h_size, grid_w_size])
        pos_embed = get_2d_sincos_pos_embed_from_grid(embed_dim, grid, version)
        if cls_token:
            pos_embed = np.concatenate([np.zeros([1, embed_dim]), pos_embed], axis=0)
    else:
        pos_embed = get_2d_sincos_pos_embed_from_grid(embed_dim, grid, version)
    return pos_embed
```
**EN:** Defines function `get_2d_sincos_pos_embed` with signature `get_2d_sincos_pos_embed(embed_dim: int, grid_size: int | tuple[int, int], cls_token: bool=False, version: tuple[int, int]=(2, 0)) -> torch.Tensor`. It mainly works with `embed_dim`, `grid_size`, `cls_token`, `version`; returns a derived property or capability check. The body uses branching, validation/error handling. Key calls include `isinstance`, `np.arange`, `np.meshgrid`, `np.stack`, `grid.reshape`, `get_2d_sincos_pos_embed_from_grid`.
**CN:** 定义函数 `get_2d_sincos_pos_embed`，其签名为 `get_2d_sincos_pos_embed(embed_dim: int, grid_size: int | tuple[int, int], cls_token: bool=False, version: tuple[int, int]=(2, 0)) -> torch.Tensor`。它主要围绕 `embed_dim`, `grid_size`, `cls_token`, `version` 展开；返回派生属性或能力判断结果。函数体包含分支判断、校验或报错逻辑。关键调用包括 `isinstance`, `np.arange`, `np.meshgrid`, `np.stack`, `grid.reshape`, `get_2d_sincos_pos_embed_from_grid`。

### Class `BaseResampler` overview (lines 156-207)
```python
class BaseResampler(nn.Module):
    """
    A 2D perceiver-resampler network with one cross attention layers by
        (grid_size**2) learnable queries and 2d sincos pos_emb.
    Outputs:
        A tensor with the shape of (grid_size**2, embed_dim)
    """

    def __init__(
        self,
        num_queries: int,
        embed_dim: int,
        num_heads: int,
        kv_dim: int | None = None,
        norm_layer: Callable[[int], nn.LayerNorm] = DEFAULT_LN,
        do_post_projection: bool = True,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()

        self.num_queries = num_queries
        self.embed_dim = embed_dim
        self.num_heads = num_heads
```
**EN:** Defines class `BaseResampler` with base classes `nn.Module` and decorators none. It acts as a resampling layer and exposes 2 direct methods, with notable entries `__init__`, `_repeat`. Its docstring says: A 2D perceiver-resampler network with one cross attention layers by (grid_size**2) learnable queries and 2d sincos pos_emb.
**CN:** 定义类 `BaseResampler`，其基类为 `nn.Module`，装饰器为 无。它在整体实现中充当重采样层，并直接暴露 2 个方法，较重要的包括 `__init__`, `_repeat`。 文档字符串进一步说明了该类的定位。

### Method `BaseResampler.__init__` (lines 164-204)
```python
    def __init__(
        self,
        num_queries: int,
        embed_dim: int,
        num_heads: int,
        kv_dim: int | None = None,
        norm_layer: Callable[[int], nn.LayerNorm] = DEFAULT_LN,
        do_post_projection: bool = True,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()

        self.num_queries = num_queries
        self.embed_dim = embed_dim
        self.num_heads = num_heads

        self.query = nn.Parameter(torch.empty(self.num_queries, embed_dim))

        if kv_dim is not None and kv_dim != embed_dim:
            self.kv_proj = ReplicatedLinear(
                kv_dim,
                embed_dim,
                bias=False,
                quant_config=quant_config,
                prefix=f"{prefix}.kv_proj",
            )
        else:
            # Maintain the same return value with ReplicatedLinear.forward
            self.kv_proj = lambda *args, **kwargs: (  # type: ignore # noqa
                nn.Identity()(*args, **kwargs),
                None,
            )
        self.attn = nn.MultiheadAttention(embed_dim, num_heads)
        self.ln_q = norm_layer(embed_dim)
        self.ln_kv = norm_layer(embed_dim)
        self.do_post_projection = do_post_projection
        if self.do_post_projection:
            self.ln_post = norm_layer(embed_dim)
            data = (embed_dim**-0.5) * torch.empty(embed_dim, embed_dim)
            self.proj = nn.Parameter(data=data)
```
**EN:** Defines function `BaseResampler.__init__` with signature `__init__(self, num_queries: int, embed_dim: int, num_heads: int, kv_dim: int | None=None, norm_layer: Callable[[int], nn.LayerNorm]=DEFAULT_LN, do_post_projection: bool=True, quant_config: QuantizationConfig | None=None, prefix: str='') -> None`. It mainly works with `num_queries`, `embed_dim`, `num_heads`, `kv_dim`, `norm_layer`, `do_post_projection`, `quant_config`, `prefix`; initializes the object state and cached resources. The body uses branching, tensor/kernel operations. Key calls include `super.__init__`, `nn.Parameter`, `nn.MultiheadAttention`, `norm_layer`, `torch.empty`, `ReplicatedLinear`.
**CN:** 定义函数 `BaseResampler.__init__`，其签名为 `__init__(self, num_queries: int, embed_dim: int, num_heads: int, kv_dim: int | None=None, norm_layer: Callable[[int], nn.LayerNorm]=DEFAULT_LN, do_post_projection: bool=True, quant_config: QuantizationConfig | None=None, prefix: str='') -> None`。它主要围绕 `num_queries`, `embed_dim`, `num_heads`, `kv_dim`, `norm_layer`, `do_post_projection`, `quant_config`, `prefix` 展开；负责初始化对象状态和缓存资源。函数体包含分支判断、张量或内核操作。关键调用包括 `super.__init__`, `nn.Parameter`, `nn.MultiheadAttention`, `norm_layer`, `torch.empty`, `ReplicatedLinear`。

### Method `BaseResampler._repeat` (lines 206-207)
```python
    def _repeat(self, query, N: int):
        return query.unsqueeze(1).repeat(1, N, 1)
```
**EN:** Defines function `BaseResampler._repeat` with signature `_repeat(self, query, N: int)`. It mainly works with `query`, `N`; implements one step of the module control flow. The body uses mostly straightforward data movement and object wiring. Key calls include `query.unsqueeze.repeat`, `query.unsqueeze`.
**CN:** 定义函数 `BaseResampler._repeat`，其签名为 `_repeat(self, query, N: int)`。它主要围绕 `query`, `N` 展开；实现模块控制流中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `query.unsqueeze.repeat`, `query.unsqueeze`。

### Class `Resampler2` overview (lines 210-283)
```python
class Resampler2(BaseResampler):
    """Resampler-perceiver network to be used for a variety of model types,
    e.g., Qwen-vl / Minicpmv 2.0. The main difference is the addition of the
    do_post_projection arg, which indicates whether or not there should be
    a post layer normalization and projector after the attention. This is
    present in minicpmv2.0, but not qwen-vl.
    """

    def __init__(
        self,
        grid_size: int,
        embed_dim: int,
        num_heads: int,
        kv_dim: int | None = None,
        norm_layer: Callable[[int], nn.LayerNorm] = DEFAULT_LN,
        adaptive: bool = False,
        do_post_projection: bool = True,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__(
            grid_size**2,
            embed_dim,
            num_heads,
            kv_dim,
```
**EN:** Defines class `Resampler2` with base classes `BaseResampler` and decorators none. It acts as a reusable module building block and exposes 2 direct methods, with notable entries `__init__`, `forward`. Its docstring says: Resampler-perceiver network to be used for a variety of model types, e.g., Qwen-vl / Minicpmv 2.0.
**CN:** 定义类 `Resampler2`，其基类为 `BaseResampler`，装饰器为 无。它在整体实现中充当可复用的模块构件，并直接暴露 2 个方法，较重要的包括 `__init__`, `forward`。 文档字符串进一步说明了该类的定位。

### Method `Resampler2.__init__` (lines 218-246)
```python
    def __init__(
        self,
        grid_size: int,
        embed_dim: int,
        num_heads: int,
        kv_dim: int | None = None,
        norm_layer: Callable[[int], nn.LayerNorm] = DEFAULT_LN,
        adaptive: bool = False,
        do_post_projection: bool = True,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__(
            grid_size**2,
            embed_dim,
            num_heads,
            kv_dim,
            norm_layer,
            do_post_projection=do_post_projection,
            quant_config=quant_config,
            prefix=prefix,
        )

        self.adaptive = adaptive
        pos_embed_arr = get_2d_sincos_pos_embed(embed_dim, grid_size, version=(2, 0))

        self.pos_embed = nn.Parameter(
            torch.from_numpy(pos_embed_arr).requires_grad_(False)
        )
```
**EN:** Defines function `Resampler2.__init__` with signature `__init__(self, grid_size: int, embed_dim: int, num_heads: int, kv_dim: int | None=None, norm_layer: Callable[[int], nn.LayerNorm]=DEFAULT_LN, adaptive: bool=False, do_post_projection: bool=True, quant_config: QuantizationConfig | None=None, prefix: str='') -> None`. It mainly works with `grid_size`, `embed_dim`, `num_heads`, `kv_dim`, `norm_layer`, `adaptive`, `do_post_projection`, `quant_config`; initializes the object state and cached resources. The body uses tensor/kernel operations. Key calls include `super.__init__`, `get_2d_sincos_pos_embed`, `nn.Parameter`, `torch.from_numpy.requires_grad_`, `super`, `torch.from_numpy`.
**CN:** 定义函数 `Resampler2.__init__`，其签名为 `__init__(self, grid_size: int, embed_dim: int, num_heads: int, kv_dim: int | None=None, norm_layer: Callable[[int], nn.LayerNorm]=DEFAULT_LN, adaptive: bool=False, do_post_projection: bool=True, quant_config: QuantizationConfig | None=None, prefix: str='') -> None`。它主要围绕 `grid_size`, `embed_dim`, `num_heads`, `kv_dim`, `norm_layer`, `adaptive`, `do_post_projection`, `quant_config` 展开；负责初始化对象状态和缓存资源。函数体包含张量或内核操作。关键调用包括 `super.__init__`, `get_2d_sincos_pos_embed`, `nn.Parameter`, `torch.from_numpy.requires_grad_`, `super`, `torch.from_numpy`。

### Method `Resampler2.forward` (lines 248-283)
```python
    def forward(
        self,
        x: torch.Tensor,
        tgt_sizes: torch.Tensor | None = None,
        attn_mask: torch.Tensor | None = None,
    ) -> torch.Tensor:
        if tgt_sizes is None:
            tgt_sizes = int(math.sqrt(x.size(1)))
        if self.adaptive:
            pos_embed_arr = get_2d_sincos_pos_embed(
                self.embed_dim, tgt_sizes, version=(2, 0)
            )
            pos_embed = torch.from_numpy(pos_embed_arr).to(
                device=x.device, dtype=x.dtype
            )
        else:
            pos_embed = get_abs_pos(self.pos_embed, tgt_sizes).to(
                device=x.device, dtype=x.dtype
            )

        x, _ = self.kv_proj(x)
        x = self.ln_kv(x).permute(1, 0, 2)

        N = x.shape[1]
        q = self.ln_q(self.query)
        out = self.attn(
            self._repeat(q, N) + self.pos_embed.unsqueeze(1),
            x + pos_embed.unsqueeze(1),
            x,
            attn_mask=attn_mask,
        )[0]
        x = out.permute(1, 0, 2)
        if self.do_post_projection:
            x = self.ln_post(x)
            x = x @ self.proj
        return x
```
**EN:** Defines function `Resampler2.forward` with signature `forward(self, x: torch.Tensor, tgt_sizes: torch.Tensor | None=None, attn_mask: torch.Tensor | None=None) -> torch.Tensor`. It mainly works with `x`, `tgt_sizes`, `attn_mask`; runs the main forward-path computation. The body uses branching, tensor/kernel operations. Key calls include `self.kv_proj`, `self.ln_kv.permute`, `self.ln_q`, `out.permute`, `int`, `get_2d_sincos_pos_embed`.
**CN:** 定义函数 `Resampler2.forward`，其签名为 `forward(self, x: torch.Tensor, tgt_sizes: torch.Tensor | None=None, attn_mask: torch.Tensor | None=None) -> torch.Tensor`。它主要围绕 `x`, `tgt_sizes`, `attn_mask` 展开；执行主要的前向计算路径。函数体包含分支判断、张量或内核操作。关键调用包括 `self.kv_proj`, `self.ln_kv.permute`, `self.ln_q`, `out.permute`, `int`, `get_2d_sincos_pos_embed`。

## Key Concepts / 关键概念
- **EN:** The file provides a reusable layer/helper inside the model-executor subsystem.
  **CN:** 该文件在模型执行子系统中提供可复用的层或辅助逻辑。
- **EN:** Top-level classes include `BaseResampler`, `Resampler2`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `BaseResampler`, `Resampler2`，它们组织了主要的可复用抽象。
- **EN:** Top-level helpers such as `get_abs_pos`, `get_1d_sincos_pos_embed_from_grid`, `get_2d_sincos_pos_embed_from_grid`, `get_2d_sincos_pos_embed` expose procedural entry points around the main classes.
  **CN:** 顶层辅助函数如 `get_abs_pos`, `get_1d_sincos_pos_embed_from_grid`, `get_2d_sincos_pos_embed_from_grid`, `get_2d_sincos_pos_embed` 为主要类提供了过程式入口。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。

## Dependencies / 依赖关系
- **External / 外部**: `math`, `collections`, `functools`, `numpy`, `torch`
- **Internal / 内部**: `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.quantization`
