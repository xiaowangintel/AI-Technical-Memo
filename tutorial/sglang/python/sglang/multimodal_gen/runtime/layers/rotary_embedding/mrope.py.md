# mrope.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/layers/rotary_embedding/mrope.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the runtime layer implementation. It centers on `OneDRotaryEmbedding`, `NDRotaryEmbedding`, and `_to_tuple`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: MRotaryEmbedding, YaRNScalingMRotaryEmbedding, NDRotaryEmbedding, OneDRotaryEmbedding. / 该文件属于运行时算子层。它围绕 `OneDRotaryEmbedding`、`NDRotaryEmbedding` 和 `_to_tuple` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: module setup and imports / 模块初始化与导入
```python
"""MRotaryEmbedding, YaRNScalingMRotaryEmbedding, NDRotaryEmbedding, OneDRotaryEmbedding."""

import functools

import torch

from sglang.multimodal_gen.runtime.distributed.parallel_state import get_sp_group
```
**EN:** This block establishes the module context and imports `functools`, `torch`, and `sglang.multimodal_gen.runtime.distributed.parallel_state`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `functools`、`torch` 和 `sglang.multimodal_gen.runtime.distributed.parallel_state`。这些依赖为后续实现提供所需符号。

### Lines 10-16: `_to_tuple` implementation / `_to_tuple` 实现
```python
def _to_tuple(x: int | tuple[int, ...], dim: int = 2) -> tuple[int, ...]:
    if isinstance(x, int):
        return (x,) * dim
    elif len(x) == dim:
        return x
    else:
        raise ValueError(f"Expected length {dim} or int, but got {x}")
```
**EN:** This block defines function `_to_tuple`. It converts to tuple. Key calls include `isinstance`, `len`, and `ValueError`. The implementation branches on conditions. Parameters such as `x`, and `dim` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_to_tuple`。 它用于转换为tuple。 关键调用包括 `isinstance`、`len` 和 `ValueError`。 实现中包含条件分支。 本段逻辑主要由 `x` 和 `dim` 等参数驱动。

### Lines 19-69: `get_1d_rotary_pos_embed` implementation / `get_1d_rotary_pos_embed` 实现
```python
def get_1d_rotary_pos_embed(
    dim: int,
    pos: torch.FloatTensor | int,
    theta: float = 10000.0,
    theta_rescale_factor: float = 1.0,
    interpolation_factor: float = 1.0,
    dtype: torch.dtype = torch.float32,
    device: torch.device | str | None = None,
) -> tuple[torch.Tensor, torch.Tensor]:
    """
    Precompute the frequency tensor for complex exponential (cis) with given dimensions.
    (Note: `cis` means `cos + i * sin`, where i is the imaginary unit.)

    This function calculates a frequency tensor with complex exponential using the given dimension 'dim'
    and the end index 'end'. The 'theta' parameter scales the frequencies.

    Args:
        dim (int): Dimension of the frequency tensor.
        pos (int or torch.FloatTensor): Position indices for the frequency tensor. [S] or scalar
        theta (float, optional): Scaling factor for frequency computation. Defaults to 10000.0.
        theta_rescale_factor (float, optional): Rescale factor for theta. Defaults to 1.0.
        interpolation_factor (float, optional): Factor to scale positions. Defaults to 1.0.

    Returns:
        freqs_cos, freqs_sin: Precomputed frequency tensor with real and imaginary parts separately. [S, D]
    """
    if isinstance(pos, int):
        pos = torch.arange(pos, dtype=dtype, device=device)
    elif (
        isinstance(pos, torch.Tensor)
        and device is not None
        and pos.device != torch.device(device)
    ):
        # Ensure positions are on the requested device to avoid implicit CPU ops.
        pos = pos.to(device)

    # proposed by reddit user bloc97, to rescale rotary embeddings to longer sequence length without fine-tuning
    # has some connection to NTK literature
    if theta_rescale_factor != 1.0:
        theta *= theta_rescale_factor ** (dim / (dim - 2))

    freqs = 1.0 / (
        theta
        ** (torch.arange(0, dim, 2, device=device)[: (dim // 2)].to(dtype) / dim).to(
            device=device
        )
    )  # [D/2]
    freqs = torch.outer(pos * interpolation_factor, freqs)  # [S, D/2]
    freqs_cos = freqs.cos()  # [S, D/2]
    freqs_sin = freqs.sin()  # [S, D/2]
    return freqs_cos, freqs_sin
```
**EN:** This block defines function `get_1d_rotary_pos_embed`. Precompute the frequency tensor for complex exponential (cis) with given dimensions. (Note: `cis` means `cos + i * sin`, where i is the imaginary unit.) This function calculates a frequency tensor with complex exponential using the given dimension 'dim' and the end index 'end'. Key calls include `isinstance`, `torch.outer`, `freqs.cos`, `freqs.sin`, and `torch.arange`. The implementation branches on conditions. Parameters such as `dim`, `pos`, `theta`, `theta_rescale_factor`, and `interpolation_factor` drive the behavior in this section.
**CN:** 该代码块定义了函数 `get_1d_rotary_pos_embed`。 它用于获取1d rotary pos embed。 关键调用包括 `isinstance`、`torch.outer`、`freqs.cos`、`freqs.sin` 和 `torch.arange`。 实现中包含条件分支。 本段逻辑主要由 `dim`、`pos`、`theta`、`theta_rescale_factor` 和 `interpolation_factor` 等参数驱动。

### Lines 72-74: `OneDRotaryEmbedding` class overview / `OneDRotaryEmbedding` 类概览
```python
class OneDRotaryEmbedding(torch.nn.Module):
    """1D rotary positional embedding with caching."""
```
**EN:** This block defines class `OneDRotaryEmbedding`. 1D rotary positional embedding with caching. It inherits from `torch.nn.Module`.
**CN:** 该代码块定义了类 `OneDRotaryEmbedding`。 它用于封装 one drotary embedding 相关行为。 它继承自 `torch.nn.Module`。

### Lines 75-94: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        dim: int,
        theta: float = 10000.0,
        theta_rescale_factor: float = 1.0,
        interpolation_factor: float = 1.0,
        dtype: torch.dtype = torch.float32,
        use_real: bool = False,
        repeat_interleave_real: bool = False,
    ):
        super().__init__()
        assert dim % 2 == 0
        self.dim = dim
        self.theta = theta
        self.theta_rescale_factor = theta_rescale_factor
        self.interpolation_factor = interpolation_factor
        # dtype of freqs
        self.dtype = dtype
        self.use_real = use_real
        self.repeat_interleave_real = repeat_interleave_real
```
**EN:** This block defines method `__init__` on `OneDRotaryEmbedding`. It initializes the instance state. Key calls include `super.__init__`, and `super`. Parameters such as `dim`, `theta`, `theta_rescale_factor`, `interpolation_factor`, and `dtype` drive the behavior in this section.
**CN:** 该代码块定义了 `OneDRotaryEmbedding` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__` 和 `super`。 本段逻辑主要由 `dim`、`theta`、`theta_rescale_factor`、`interpolation_factor` 和 `dtype` 等参数驱动。

### Lines 96-106: `build_freqs` implementation / `build_freqs` 实现
```python
    def build_freqs(self, device):
        freqs = 1.0 / (
            self.theta
            ** (
                torch.arange(0, self.dim, 2, dtype=self.dtype, device=device)[
                    : (self.dim // 2)
                ]
                / self.dim
            ).to(device=device)
        )
        return freqs
```
**EN:** This block defines method `build_freqs` on `OneDRotaryEmbedding`. It builds freqs. Key calls include `to`, and `torch.arange`. Parameters such as `device` drive the behavior in this section.
**CN:** 该代码块定义了 `OneDRotaryEmbedding` 的方法 `build_freqs`。 它用于构建freqs。 关键调用包括 `to` 和 `torch.arange`。 本段逻辑主要由 `device` 等参数驱动。

### Lines 108-125: `build_freqs_outer` implementation / `build_freqs_outer` 实现
```python
    def build_freqs_outer(self, pos: torch.Tensor, device):
        theta = self.theta
        # proposed by reddit user bloc97, to rescale rotary embeddings to longer sequence length without fine-tuning
        # has some connection to NTK literature
        if self.theta_rescale_factor != 1.0:
            theta *= self.theta_rescale_factor ** (self.dim / (self.dim - 2))

        freqs = self.build_freqs(device)

        freqs = torch.outer(pos * self.interpolation_factor, freqs)
        freqs_cos = freqs.cos()
        freqs_sin = freqs.sin()

        if self.use_real and self.repeat_interleave_real:
            freqs_cos = freqs_cos.repeat_interleave(2, dim=1)
            freqs_sin = freqs_sin.repeat_interleave(2, dim=1)

        return freqs_cos.float(), freqs_sin.float()
```
**EN:** This block defines method `build_freqs_outer` on `OneDRotaryEmbedding`. It builds freqs outer. Key calls include `self.build_freqs`, `torch.outer`, `freqs.cos`, `freqs.sin`, and `freqs_cos.repeat_interleave`. The implementation branches on conditions. Parameters such as `pos`, and `device` drive the behavior in this section.
**CN:** 该代码块定义了 `OneDRotaryEmbedding` 的方法 `build_freqs_outer`。 它用于构建freqs outer。 关键调用包括 `self.build_freqs`、`torch.outer`、`freqs.cos`、`freqs.sin` 和 `freqs_cos.repeat_interleave`。 实现中包含条件分支。 本段逻辑主要由 `pos` 和 `device` 等参数驱动。

### Lines 127-137: `forward_from_grid` implementation / `forward_from_grid` 实现
```python
    @functools.lru_cache(maxsize=16)
    def forward_from_grid(
        self, seq_len: int, start_pos: int, device_str: str
    ) -> tuple[torch.Tensor, torch.Tensor]:
        device = torch.device(device_str)
        pos = torch.arange(
            start_pos, start_pos + seq_len, dtype=self.dtype, device=device
        )

        freqs_cos, freqs_sin = self.build_freqs_outer(pos, device)
        return freqs_cos, freqs_sin
```
**EN:** This block defines method `forward_from_grid` on `OneDRotaryEmbedding`. It executes from grid. Key calls include `functools.lru_cache`, `torch.device`, `torch.arange`, and `self.build_freqs_outer`. Parameters such as `seq_len`, `start_pos`, and `device_str` drive the behavior in this section.
**CN:** 该代码块定义了 `OneDRotaryEmbedding` 的方法 `forward_from_grid`。 它用于执行前向计算from grid。 关键调用包括 `functools.lru_cache`、`torch.device`、`torch.arange` 和 `self.build_freqs_outer`。 本段逻辑主要由 `seq_len`、`start_pos` 和 `device_str` 等参数驱动。

### Lines 139-148: `forward` implementation / `forward` 实现
```python
    def forward(self, pos: torch.Tensor) -> tuple[torch.Tensor, torch.Tensor]:
        """
        Calculates 1D rotary embeddings for the given positions.

        This method converts the input tensor to a hashable representation
        and calls a cached helper method to perform the computation.
        """
        pos_tuple = tuple(pos.tolist())
        device_str = str(pos.device)
        return self._forward_cached(pos_tuple, device_str)
```
**EN:** This block defines method `forward` on `OneDRotaryEmbedding`. Calculates 1D rotary embeddings for the given positions. This method converts the input tensor to a hashable representation and calls a cached helper method to perform the computation. Key calls include `tuple`, `str`, `self._forward_cached`, and `pos.tolist`. Parameters such as `pos` drive the behavior in this section.
**CN:** 该代码块定义了 `OneDRotaryEmbedding` 的方法 `forward`。 它用于执行前向计算函数。 关键调用包括 `tuple`、`str`、`self._forward_cached` 和 `pos.tolist`。 本段逻辑主要由 `pos` 等参数驱动。

### Lines 150-161: `_forward_cached` implementation / `_forward_cached` 实现
```python
    @functools.lru_cache(maxsize=16)
    def _forward_cached(
        self, pos_tuple: tuple, device_str: str
    ) -> tuple[torch.Tensor, torch.Tensor]:
        """
        The core implementation that computes 1D rotary embeddings.
        This method is wrapped by an LRU cache.
        """
        device = torch.device(device_str)
        pos = torch.as_tensor(pos_tuple, dtype=self.dtype, device=device)
        freqs_cos, freqs_sin = self.build_freqs_outer(pos, device)
        return freqs_cos, freqs_sin
```
**EN:** This block defines method `_forward_cached` on `OneDRotaryEmbedding`. The core implementation that computes 1D rotary embeddings. This method is wrapped by an LRU cache. Key calls include `functools.lru_cache`, `torch.device`, `torch.as_tensor`, and `self.build_freqs_outer`. Parameters such as `pos_tuple`, and `device_str` drive the behavior in this section.
**CN:** 该代码块定义了 `OneDRotaryEmbedding` 的方法 `_forward_cached`。 它用于执行前向计算cached。 关键调用包括 `functools.lru_cache`、`torch.device`、`torch.as_tensor` 和 `self.build_freqs_outer`。 本段逻辑主要由 `pos_tuple` 和 `device_str` 等参数驱动。

### Lines 164-166: `NDRotaryEmbedding` class overview / `NDRotaryEmbedding` 类概览
```python
class NDRotaryEmbedding(torch.nn.Module):
    """N-dimensional rotary positional embedding."""
```
**EN:** This block defines class `NDRotaryEmbedding`. N-dimensional rotary positional embedding. It inherits from `torch.nn.Module`.
**CN:** 该代码块定义了类 `NDRotaryEmbedding`。 它用于封装 ndrotary embedding 相关行为。 它继承自 `torch.nn.Module`。

### Lines 167-229: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        rope_dim_list: list[int],
        rope_theta: float,
        theta_rescale_factor: float | list[float] = 1.0,
        interpolation_factor: float | list[float] = 1.0,
        use_real: bool = False,
        repeat_interleave_real: bool = False,
        dtype: torch.dtype = torch.float32,
    ):
        super().__init__()
        self.rope_dim_list = rope_dim_list
        self.ndim = len(rope_dim_list)
        self.rope_theta = rope_theta
        # dtype of freqs
        # does not control the output dtype
        self.dtype = dtype

        if isinstance(theta_rescale_factor, (int, float)):
            self.theta_rescale_factor = [theta_rescale_factor] * self.ndim
        elif isinstance(theta_rescale_factor, list) and len(theta_rescale_factor) == 1:
            self.theta_rescale_factor = [theta_rescale_factor[0]] * self.ndim
        else:
            self.theta_rescale_factor = theta_rescale_factor
        assert (
            len(self.theta_rescale_factor) == self.ndim
        ), "len(theta_rescale_factor) should equal to len(rope_dim_list)"

        if isinstance(interpolation_factor, (int, float)):
            self.interpolation_factor = [interpolation_factor] * self.ndim
        elif isinstance(interpolation_factor, list) and len(interpolation_factor) == 1:
            self.interpolation_factor = [interpolation_factor[0]] * self.ndim
        else:
            self.interpolation_factor = interpolation_factor
        assert (
            len(self.interpolation_factor) == self.ndim
        ), "len(interpolation_factor) should equal to len(rope_dim_list)"

        self.rope_generators: list[OneDRotaryEmbedding] = torch.nn.ModuleList()
        _config_to_gen_idx: dict[tuple, int] = {}
        self.dim_idx_to_gen_idx: list[int] = []

        for i in range(self.ndim):
            dim = self.rope_dim_list[i]
            rescale = self.theta_rescale_factor[i]
            interp = self.interpolation_factor[i]

            config_key = (dim, rescale, interp, use_real, repeat_interleave_real)
            if config_key not in _config_to_gen_idx:
                generator = OneDRotaryEmbedding(
                    dim=dim,
                    theta=self.rope_theta,
                    theta_rescale_factor=rescale,
                    interpolation_factor=interp,
                    dtype=self.dtype,
                    use_real=use_real,
                    repeat_interleave_real=repeat_interleave_real,
                )
                _config_to_gen_idx[config_key] = len(self.rope_generators)
                self.rope_generators.append(generator)

            gen_idx = _config_to_gen_idx[config_key]
            self.dim_idx_to_gen_idx.append(gen_idx)
```
**EN:** This block defines method `__init__` on `NDRotaryEmbedding`. It initializes the instance state. Key calls include `super.__init__`, `len`, `isinstance`, `torch.nn.ModuleList`, and `range`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `rope_dim_list`, `rope_theta`, `theta_rescale_factor`, `interpolation_factor`, and `use_real` drive the behavior in this section.
**CN:** 该代码块定义了 `NDRotaryEmbedding` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__`、`len`、`isinstance`、`torch.nn.ModuleList` 和 `range`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `rope_dim_list`、`rope_theta`、`theta_rescale_factor`、`interpolation_factor` 和 `use_real` 等参数驱动。

### Lines 231-245: `forward` implementation / `forward` 实现
```python
    def forward(self, positions: torch.Tensor) -> tuple[torch.Tensor, torch.Tensor]:
        """
        Calculates n-d rotary embeddings for given absolute positions.

        Args:
            positions (torch.Tensor): A tensor of shape `[num_tokens, ndim]`
                containing the integer coordinates for each token.

        Returns:
            A tuple of (cos, sin) tensors.
        """
        # Caching wrapper: convert tensor to a hashable tuple of tuples.
        pos_tuple = tuple(map(tuple, positions.tolist()))
        device_str = str(positions.device)
        return self._forward_cached(pos_tuple, device_str)
```
**EN:** This block defines method `forward` on `NDRotaryEmbedding`. Calculates n-d rotary embeddings for given absolute positions. Args: positions (torch.Tensor): A tensor of shape `[num_tokens, ndim]` containing the integer coordinates for each token. Key calls include `tuple`, `str`, `self._forward_cached`, `map`, and `positions.tolist`. Parameters such as `positions` drive the behavior in this section.
**CN:** 该代码块定义了 `NDRotaryEmbedding` 的方法 `forward`。 它用于执行前向计算函数。 关键调用包括 `tuple`、`str`、`self._forward_cached`、`map` 和 `positions.tolist`。 本段逻辑主要由 `positions` 等参数驱动。

### Lines 247-257: `_forward_cached` implementation / `_forward_cached` 实现
```python
    @functools.lru_cache(maxsize=16)
    def _forward_cached(
        self, pos_tuple: tuple[tuple[int, ...], ...], device_str: str
    ) -> tuple[torch.Tensor, torch.Tensor]:
        """
        The core implementation that computes embeddings from a position tensor.
        This method is wrapped by an LRU cache.
        """
        device = torch.device(device_str)
        positions = torch.tensor(pos_tuple, dtype=torch.long, device=device)
        return self.forward_uncached(pos=positions)
```
**EN:** This block defines method `_forward_cached` on `NDRotaryEmbedding`. The core implementation that computes embeddings from a position tensor. This method is wrapped by an LRU cache. Key calls include `functools.lru_cache`, `torch.device`, `torch.tensor`, and `self.forward_uncached`. Parameters such as `pos_tuple`, and `device_str` drive the behavior in this section.
**CN:** 该代码块定义了 `NDRotaryEmbedding` 的方法 `_forward_cached`。 它用于执行前向计算cached。 关键调用包括 `functools.lru_cache`、`torch.device`、`torch.tensor` 和 `self.forward_uncached`。 本段逻辑主要由 `pos_tuple` 和 `device_str` 等参数驱动。

### Lines 259-294: `forward_uncached` implementation / `forward_uncached` 实现
```python
    def forward_uncached(self, pos: torch.Tensor) -> tuple[torch.Tensor, torch.Tensor]:
        """
        The core implementation that computes embeddings from a position tensor.
        This method is wrapped by an LRU cache.
        """
        device = pos.device

        # Pre-allocate the final tensors for efficiency.
        num_tokens = pos.shape[0]
        first_generator = self.rope_generators[0]
        if first_generator.use_real and first_generator.repeat_interleave_real:
            head_dim = sum(self.rope_dim_list)
        else:
            head_dim = sum(self.rope_dim_list) // 2

        cos = torch.empty((num_tokens, head_dim), device=device, dtype=self.dtype)
        sin = torch.empty((num_tokens, head_dim), device=device, dtype=self.dtype)

        col_offset = 0
        for i in range(self.ndim):
            # Extract position coordinates for the current dimension for all tokens.
            pos_i = pos[:, i].to(self.dtype)

            # Get the appropriate 1D generator.
            gen_idx = self.dim_idx_to_gen_idx[i]
            generator = self.rope_generators[gen_idx]

            # Calculate 1D embeddings.
            cos_1d, sin_1d = generator(pos_i)

            slice_width = cos_1d.shape[1]
            cos[:, col_offset : col_offset + slice_width] = cos_1d
            sin[:, col_offset : col_offset + slice_width] = sin_1d
            col_offset += slice_width

        return cos.float(), sin.float()
```
**EN:** This block defines method `forward_uncached` on `NDRotaryEmbedding`. The core implementation that computes embeddings from a position tensor. This method is wrapped by an LRU cache. Key calls include `torch.empty`, `range`, `sum`, `pos.to`, and `generator`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `pos` drive the behavior in this section.
**CN:** 该代码块定义了 `NDRotaryEmbedding` 的方法 `forward_uncached`。 它用于执行前向计算uncached。 关键调用包括 `torch.empty`、`range`、`sum`、`pos.to` 和 `generator`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `pos` 等参数驱动。

### Lines 296-311: `forward_from_grid` implementation / `forward_from_grid` 实现
```python
    def forward_from_grid(
        self,
        grid_size: tuple[int, ...],
        shard_dim: int = 0,
        start_frame: int = 0,
        device: torch.device | str | None = None,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        """
        Handles sp internally
        """
        # Caching wrapper: use grid parameters directly as the key.
        # grid_tuple = _to_tuple(grid_size, dim=self.ndim)
        device_str = str(device) if device is not None else "cpu"
        return self._forward_cached_from_grid(
            grid_size, shard_dim, start_frame, device_str
        )
```
**EN:** This block defines method `forward_from_grid` on `NDRotaryEmbedding`. Handles sp internally Key calls include `self._forward_cached_from_grid`, and `str`. Parameters such as `grid_size`, `shard_dim`, `start_frame`, and `device` drive the behavior in this section.
**CN:** 该代码块定义了 `NDRotaryEmbedding` 的方法 `forward_from_grid`。 它用于执行前向计算from grid。 关键调用包括 `self._forward_cached_from_grid` 和 `str`。 本段逻辑主要由 `grid_size`、`shard_dim`、`start_frame` 和 `device` 等参数驱动。

### Lines 313-392: `_forward_cached_from_grid` implementation / `_forward_cached_from_grid` 实现
```python
    @functools.lru_cache(maxsize=16)
    def _forward_cached_from_grid(
        self,
        grid_size: tuple[int, ...],
        shard_dim: int,
        start_frame: int,
        device_str: str,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        """
        Computes embeddings for a structured grid, using a highly efficient
        implementation that avoids materializing the full position tensor.
        This method is wrapped by an LRU cache.
        """
        device = torch.device(device_str)
        sp_group = get_sp_group()
        sp_rank = sp_group.rank_in_group
        sp_world_size = sp_group.world_size

        sizes = _to_tuple(grid_size, dim=self.ndim)
        starts = (0,) * self.ndim

        # Apply sequence parallel sharding to the sizes and compute shard offset
        shard_sizes = list(sizes)
        shard_offsets = [0] * self.ndim
        if sp_world_size > 1:
            assert sizes[shard_dim] % sp_world_size == 0, (
                f"Dimension {shard_dim} with size {sizes[shard_dim]} is not divisible "
                f"by sequence parallel world size {sp_world_size}"
            )
            shard_size = sizes[shard_dim] // sp_world_size
            shard_offsets[shard_dim] = sp_rank * shard_size
            shard_sizes[shard_dim] = shard_size

        # Pre-allocate outputs on the requested device to avoid CPU ops and extra cats
        num_tokens = 1
        for s in shard_sizes:
            num_tokens *= int(s)
        head_dim_half = sum(self.rope_dim_list) // 2
        cos = torch.empty((num_tokens, head_dim_half), device=device, dtype=self.dtype)
        sin = torch.empty((num_tokens, head_dim_half), device=device, dtype=self.dtype)

        # Compute per-axis 1D embeddings once and expand via repeats to [N, d_i/2]
        col_offset = 0
        for i in range(self.ndim):
            dim_i = self.rope_dim_list[i]
            dim_i_half = dim_i // 2
            size_i = int(shard_sizes[i])

            # Starting position for this axis, with optional frame offset for time axis (i==0)
            base_offset = starts[i]
            if i == 0 and start_frame > 0:
                base_offset += start_frame
            if sp_world_size > 1 and i == shard_dim:
                base_offset += shard_offsets[i]

            gen_idx = self.dim_idx_to_gen_idx[i]
            generator = self.rope_generators[gen_idx]
            cos_1d, sin_1d = generator.forward_from_grid(
                size_i, base_offset, device_str
            )

            # Expand to [num_tokens, dim_i/2] matching flatten order (last dims vary fastest)
            repeats_per_entry = 1
            for j in range(i + 1, self.ndim):
                repeats_per_entry *= int(shard_sizes[j])
            tile_count = 1
            for j in range(0, i):
                tile_count *= int(shard_sizes[j])

            cos_expanded = cos_1d.repeat_interleave(repeats_per_entry, dim=0)
            sin_expanded = sin_1d.repeat_interleave(repeats_per_entry, dim=0)
            if tile_count > 1:
                cos_expanded = cos_expanded.repeat(tile_count, 1)
                sin_expanded = sin_expanded.repeat(tile_count, 1)

            cos[:, col_offset : col_offset + dim_i_half] = cos_expanded
            sin[:, col_offset : col_offset + dim_i_half] = sin_expanded
            col_offset += dim_i_half

        return cos.float(), sin.float()
```
**EN:** This block defines method `_forward_cached_from_grid` on `NDRotaryEmbedding`. Computes embeddings for a structured grid, using a highly efficient implementation that avoids materializing the full position tensor. This method is wrapped by an LRU cache. Key calls include `functools.lru_cache`, `torch.device`, `get_sp_group`, `_to_tuple`, and `list`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `grid_size`, `shard_dim`, `start_frame`, and `device_str` drive the behavior in this section.
**CN:** 该代码块定义了 `NDRotaryEmbedding` 的方法 `_forward_cached_from_grid`。 它用于执行前向计算cached from grid。 关键调用包括 `functools.lru_cache`、`torch.device`、`get_sp_group`、`_to_tuple` 和 `list`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `grid_size`、`shard_dim`、`start_frame` 和 `device_str` 等参数驱动。

## Key Concepts / 关键概念
- `_to_tuple`: Top-level function that converts to tuple. / 顶层函数，用于转换为tuple。
- `get_1d_rotary_pos_embed`: Precompute the frequency tensor for complex exponential (cis) with given dimensions. / 顶层函数，用于获取1d rotary pos embed。
- `OneDRotaryEmbedding`: 1D rotary positional embedding with caching. / 核心类，用于封装 one drotary embedding 相关行为。
- `NDRotaryEmbedding`: N-dimensional rotary positional embedding. / 核心类，用于封装 ndrotary embedding 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `functools`
- **Third-party / 第三方依赖**: `torch`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.distributed.parallel_state`

- **Total lines / 总行数**: 392
