# common.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/rotary_embedding/common.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Collects shared helpers and data structures for rotary positional embedding variants. / 汇总旋转位置编码变体的共享辅助逻辑与数据结构。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 4-12)
```python
import math
from importlib.util import find_spec

import torch

from vllm.logger import init_logger
from vllm.model_executor.custom_op import CustomOp
from vllm.platforms import current_platform
from vllm.utils.torch_utils import direct_register_custom_op
```
**EN:** This opening block pulls in external dependencies such as `math`, `importlib`, `torch` and internal modules such as `vllm.logger`, `vllm.model_executor.custom_op`, `vllm.platforms`, `vllm.utils.torch_utils`. That import mix shows the file is part of the rotary positional embedding variants stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `math`, `importlib`, `torch`）以及内部模块（如 `vllm.logger`, `vllm.model_executor.custom_op`, `vllm.platforms`, `vllm.utils.torch_utils`）。这些导入关系表明该文件属于旋转位置编码变体栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Module constants and metadata (lines 14-14)
```python
logger = init_logger(__name__)
```
**EN:** This block defines module-level metadata or constants such as `logger`. Those symbols usually capture defaults, aliases, export lists, or registry data that later classes/functions reuse inside the rotary positional embedding variants pipeline.
**CN:** 这个代码块定义了模块级元数据或常量，例如 `logger`。这些符号通常用于保存默认值、别名、导出列表或注册信息，供后续类和函数在旋转位置编码变体流程中复用。

### Function `rotate_neox` (lines 18-21)
```python
def rotate_neox(x: torch.Tensor) -> torch.Tensor:
    x1 = x[..., : x.shape[-1] // 2]
    x2 = x[..., x.shape[-1] // 2 :]
    return torch.cat((-x2, x1), dim=-1)
```
**EN:** Defines function `rotate_neox` with signature `rotate_neox(x: torch.Tensor) -> torch.Tensor`. It mainly works with `x`; adjusts rotary-position-encoding parameters or application logic. The body uses tensor/kernel operations. Key calls include `torch.cat`.
**CN:** 定义函数 `rotate_neox`，其签名为 `rotate_neox(x: torch.Tensor) -> torch.Tensor`。它主要围绕 `x` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含张量或内核操作。关键调用包括 `torch.cat`。

### Function `rotate_gptj` (lines 24-28)
```python
def rotate_gptj(x: torch.Tensor) -> torch.Tensor:
    x1 = x[..., ::2]
    x2 = x[..., 1::2]
    x = torch.stack((-x2, x1), dim=-1)
    return x.flatten(-2)
```
**EN:** Defines function `rotate_gptj` with signature `rotate_gptj(x: torch.Tensor) -> torch.Tensor`. It mainly works with `x`; adjusts rotary-position-encoding parameters or application logic. The body uses tensor/kernel operations. Key calls include `torch.stack`, `x.flatten`.
**CN:** 定义函数 `rotate_gptj`，其签名为 `rotate_gptj(x: torch.Tensor) -> torch.Tensor`。它主要围绕 `x` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含张量或内核操作。关键调用包括 `torch.stack`, `x.flatten`。

### Function `yarn_find_correction_dim` (lines 33-41)
```python
def yarn_find_correction_dim(
    num_rotations: int,
    dim: int,
    base: float = 10000,
    max_position_embeddings: int = 2048,
) -> float:
    return (dim * math.log(max_position_embeddings / (num_rotations * 2 * math.pi))) / (
        2 * math.log(base)
    )
```
**EN:** Defines function `yarn_find_correction_dim` with signature `yarn_find_correction_dim(num_rotations: int, dim: int, base: float=10000, max_position_embeddings: int=2048) -> float`. It mainly works with `num_rotations`, `dim`, `base`, `max_position_embeddings`; adjusts rotary-position-encoding parameters or application logic. The body uses mostly straightforward data movement and object wiring. Key calls include `math.log`.
**CN:** 定义函数 `yarn_find_correction_dim`，其签名为 `yarn_find_correction_dim(num_rotations: int, dim: int, base: float=10000, max_position_embeddings: int=2048) -> float`。它主要围绕 `num_rotations`, `dim`, `base`, `max_position_embeddings` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `math.log`。

### Function `yarn_find_correction_range` (lines 45-58)
```python
def yarn_find_correction_range(
    low_rot: int,
    high_rot: int,
    dim: int,
    base: float = 10000,
    max_position_embeddings: int = 2048,
    truncate: bool = True,
) -> tuple[float | int, float | int]:
    low = yarn_find_correction_dim(low_rot, dim, base, max_position_embeddings)
    high = yarn_find_correction_dim(high_rot, dim, base, max_position_embeddings)
    if truncate:
        low = math.floor(low)
        high = math.ceil(high)
    return max(low, 0), min(high, dim - 1)  # Clamp values just in case
```
**EN:** Defines function `yarn_find_correction_range` with signature `yarn_find_correction_range(low_rot: int, high_rot: int, dim: int, base: float=10000, max_position_embeddings: int=2048, truncate: bool=True) -> tuple[float | int, float | int]`. It mainly works with `low_rot`, `high_rot`, `dim`, `base`, `max_position_embeddings`, `truncate`; adjusts rotary-position-encoding parameters or application logic. The body uses branching. Key calls include `yarn_find_correction_dim`, `math.floor`, `math.ceil`, `max`, `min`.
**CN:** 定义函数 `yarn_find_correction_range`，其签名为 `yarn_find_correction_range(low_rot: int, high_rot: int, dim: int, base: float=10000, max_position_embeddings: int=2048, truncate: bool=True) -> tuple[float | int, float | int]`。它主要围绕 `low_rot`, `high_rot`, `dim`, `base`, `max_position_embeddings`, `truncate` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含分支判断。关键调用包括 `yarn_find_correction_dim`, `math.floor`, `math.ceil`, `max`, `min`。

### Function `yarn_linear_ramp_mask` (lines 61-69)
```python
def yarn_linear_ramp_mask(
    low: float, high: float, dim: int, dtype: torch.dtype
) -> torch.Tensor:
    if low == high:
        high += 0.001  # Prevent singularity

    linear_func = (torch.arange(dim, dtype=dtype) - low) / (high - low)
    ramp_func = torch.clamp(linear_func, 0, 1)
    return ramp_func
```
**EN:** Defines function `yarn_linear_ramp_mask` with signature `yarn_linear_ramp_mask(low: float, high: float, dim: int, dtype: torch.dtype) -> torch.Tensor`. It mainly works with `low`, `high`, `dim`, `dtype`; adjusts rotary-position-encoding parameters or application logic. The body uses branching, tensor/kernel operations. Key calls include `torch.clamp`, `torch.arange`.
**CN:** 定义函数 `yarn_linear_ramp_mask`，其签名为 `yarn_linear_ramp_mask(low: float, high: float, dim: int, dtype: torch.dtype) -> torch.Tensor`。它主要围绕 `low`, `high`, `dim`, `dtype` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含分支判断、张量或内核操作。关键调用包括 `torch.clamp`, `torch.arange`。

### Function `_flashinfer_rotary_embedding` (lines 78-99)
```python
def _flashinfer_rotary_embedding(
    positions: torch.Tensor,
    query: torch.Tensor,
    key: torch.Tensor,
    head_size: int,
    cos_sin_cache: torch.Tensor,
    is_neox: bool,
) -> None:
    """Custom op wrapper for flashinfer's rotary embedding.

    This is an in-place operation that modifies query and key tensors directly.
    """
    from flashinfer.rope import apply_rope_with_cos_sin_cache_inplace

    apply_rope_with_cos_sin_cache_inplace(
        positions=positions,
        query=query,
        key=key,
        head_size=head_size,
        cos_sin_cache=cos_sin_cache,
        is_neox=is_neox,
    )
```
**EN:** Defines function `_flashinfer_rotary_embedding` with signature `_flashinfer_rotary_embedding(positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor, head_size: int, cos_sin_cache: torch.Tensor, is_neox: bool) -> None`. It mainly works with `positions`, `query`, `key`, `head_size`, `cos_sin_cache`, `is_neox`; adjusts rotary-position-encoding parameters or application logic. The body uses mostly straightforward data movement and object wiring. Key calls include `apply_rope_with_cos_sin_cache_inplace`.
**CN:** 定义函数 `_flashinfer_rotary_embedding`，其签名为 `_flashinfer_rotary_embedding(positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor, head_size: int, cos_sin_cache: torch.Tensor, is_neox: bool) -> None`。它主要围绕 `positions`, `query`, `key`, `head_size`, `cos_sin_cache`, `is_neox` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `apply_rope_with_cos_sin_cache_inplace`。

### Function `_flashinfer_rotary_embedding_fake` (lines 102-110)
```python
def _flashinfer_rotary_embedding_fake(
    positions: torch.Tensor,
    query: torch.Tensor,
    key: torch.Tensor,
    head_size: int,
    cos_sin_cache: torch.Tensor,
    is_neox: bool,
) -> None:
    return
```
**EN:** Defines function `_flashinfer_rotary_embedding_fake` with signature `_flashinfer_rotary_embedding_fake(positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor, head_size: int, cos_sin_cache: torch.Tensor, is_neox: bool) -> None`. It mainly works with `positions`, `query`, `key`, `head_size`, `cos_sin_cache`, `is_neox`; adjusts rotary-position-encoding parameters or application logic. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `_flashinfer_rotary_embedding_fake`，其签名为 `_flashinfer_rotary_embedding_fake(positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor, head_size: int, cos_sin_cache: torch.Tensor, is_neox: bool) -> None`。它主要围绕 `positions`, `query`, `key`, `head_size`, `cos_sin_cache`, `is_neox` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Class `ApplyRotaryEmb` overview (lines 124-290)
```python
class ApplyRotaryEmb(CustomOp):
    # --8<-- [end:apply_rotary_emb]

    def __init__(
        self,
        enforce_enable: bool = False,
        is_neox_style: bool = True,
        enable_fp32_compute: bool = False,
    ) -> None:
        super().__init__(enforce_enable=enforce_enable)
        self.is_neox_style = is_neox_style
        self.enable_fp32_compute = enable_fp32_compute

        self.apply_rotary_emb_flash_attn = None
        if not current_platform.is_cpu() and find_spec("flash_attn") is not None:
            from flash_attn.ops.triton.rotary import apply_rotary

            self.apply_rotary_emb_flash_attn = apply_rotary

    @staticmethod
    def forward_static(
        x: torch.Tensor,
        cos: torch.Tensor,
        sin: torch.Tensor,
        is_neox_style: bool = True,
```
**EN:** Defines class `ApplyRotaryEmb` with base classes `CustomOp` and decorators `CustomOp.register('apply_rotary_emb')`. It acts as a rotary-position-encoding variant and exposes 9 direct methods, with notable entries `__init__`, `forward_static`, `_pre_process`, `_post_process`, `forward_native`, `forward_cuda`.
**CN:** 定义类 `ApplyRotaryEmb`，其基类为 `CustomOp`，装饰器为 `CustomOp.register('apply_rotary_emb')`。它在整体实现中充当旋转位置编码变体，并直接暴露 9 个方法，较重要的包括 `__init__`, `forward_static`, `_pre_process`, `_post_process`, `forward_native`, `forward_cuda`。

### Method `ApplyRotaryEmb.__init__` (lines 127-141)
```python
    def __init__(
        self,
        enforce_enable: bool = False,
        is_neox_style: bool = True,
        enable_fp32_compute: bool = False,
    ) -> None:
        super().__init__(enforce_enable=enforce_enable)
        self.is_neox_style = is_neox_style
        self.enable_fp32_compute = enable_fp32_compute

        self.apply_rotary_emb_flash_attn = None
        if not current_platform.is_cpu() and find_spec("flash_attn") is not None:
            from flash_attn.ops.triton.rotary import apply_rotary

            self.apply_rotary_emb_flash_attn = apply_rotary
```
**EN:** Defines function `ApplyRotaryEmb.__init__` with signature `__init__(self, enforce_enable: bool=False, is_neox_style: bool=True, enable_fp32_compute: bool=False) -> None`. It mainly works with `enforce_enable`, `is_neox_style`, `enable_fp32_compute`; initializes the object state and cached resources. The body uses branching. Key calls include `super.__init__`, `super`, `current_platform.is_cpu`, `find_spec`.
**CN:** 定义函数 `ApplyRotaryEmb.__init__`，其签名为 `__init__(self, enforce_enable: bool=False, is_neox_style: bool=True, enable_fp32_compute: bool=False) -> None`。它主要围绕 `enforce_enable`, `is_neox_style`, `enable_fp32_compute` 展开；负责初始化对象状态和缓存资源。函数体包含分支判断。关键调用包括 `super.__init__`, `super`, `current_platform.is_cpu`, `find_spec`。

### Method `ApplyRotaryEmb.forward_static` (lines 144-183)
```python
    def forward_static(
        x: torch.Tensor,
        cos: torch.Tensor,
        sin: torch.Tensor,
        is_neox_style: bool = True,
        enable_fp32_compute: bool = False,
    ) -> torch.Tensor:
        """
        Args:
            x: [batch_size (optional), seq_len, num_heads, head_size]
            cos: [seq_len, head_size // 2]
            sin: [seq_len, head_size // 2]
            is_neox_style: Whether to use the Neox-style or GPT-J-style.
            enable_fp32_compute: Temporarily convert x, cos, sin to FP32 dtype
                                 for higher accuracy.
        """
        origin_dtype = x.dtype
        if enable_fp32_compute:
            x = x.float()

        cos = cos.unsqueeze(-2).to(x.dtype)
        sin = sin.unsqueeze(-2).to(x.dtype)

        if is_neox_style:
            x1, x2 = torch.chunk(x, 2, dim=-1)
        else:
            x1 = x[..., ::2]
            x2 = x[..., 1::2]

        o1 = x1 * cos - x2 * sin
        o2 = x2 * cos + x1 * sin

        if is_neox_style:
            output = torch.cat((o1, o2), dim=-1)
        else:
            output = torch.stack((o1, o2), dim=-1).flatten(-2)

        if enable_fp32_compute:
            output = output.to(origin_dtype)
        return output
```
**EN:** Defines function `ApplyRotaryEmb.forward_static` with signature `forward_static(x: torch.Tensor, cos: torch.Tensor, sin: torch.Tensor, is_neox_style: bool=True, enable_fp32_compute: bool=False) -> torch.Tensor`. It mainly works with `x`, `cos`, `sin`, `is_neox_style`, `enable_fp32_compute`; adjusts rotary-position-encoding parameters or application logic. The body uses branching, tensor/kernel operations. Key calls include `cos.unsqueeze.to`, `sin.unsqueeze.to`, `x.float`, `torch.chunk`, `torch.cat`, `torch.stack.flatten`.
**CN:** 定义函数 `ApplyRotaryEmb.forward_static`，其签名为 `forward_static(x: torch.Tensor, cos: torch.Tensor, sin: torch.Tensor, is_neox_style: bool=True, enable_fp32_compute: bool=False) -> torch.Tensor`。它主要围绕 `x`, `cos`, `sin`, `is_neox_style`, `enable_fp32_compute` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含分支判断、张量或内核操作。关键调用包括 `cos.unsqueeze.to`, `sin.unsqueeze.to`, `x.float`, `torch.chunk`, `torch.cat`, `torch.stack.flatten`。

### Method `ApplyRotaryEmb.forward_native` (lines 216-225)
```python
    def forward_native(
        self,
        x: torch.Tensor,
        cos: torch.Tensor,
        sin: torch.Tensor,
    ) -> torch.Tensor:
        output = self.forward_static(
            x, cos, sin, self.is_neox_style, self.enable_fp32_compute
        )
        return output
```
**EN:** Defines function `ApplyRotaryEmb.forward_native` with signature `forward_native(self, x: torch.Tensor, cos: torch.Tensor, sin: torch.Tensor) -> torch.Tensor`. It mainly works with `x`, `cos`, `sin`; adjusts rotary-position-encoding parameters or application logic. The body uses mostly straightforward data movement and object wiring. Key calls include `self.forward_static`.
**CN:** 定义函数 `ApplyRotaryEmb.forward_native`，其签名为 `forward_native(self, x: torch.Tensor, cos: torch.Tensor, sin: torch.Tensor) -> torch.Tensor`。它主要围绕 `x`, `cos`, `sin` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `self.forward_static`。

### Method `ApplyRotaryEmb.forward_cuda` (lines 227-248)
```python
    def forward_cuda(
        self,
        x: torch.Tensor,
        cos: torch.Tensor,
        sin: torch.Tensor,
    ) -> torch.Tensor:
        from vllm.vllm_flash_attn.layers.rotary import apply_rotary_emb

        x, cos, sin, origin_shape, origin_dtype = self._pre_process(x, cos, sin)

        """
        Arguments of apply_rotary_emb() in vllm_flash_attn:
            x: [batch_size, seq_len, nheads, headdim]
            cos, sin: [seqlen_rotary, rotary_dim / 2]
            interleaved: default as False (Neox-style).
            ...
        """
        interleaved = not self.is_neox_style
        output = apply_rotary_emb(x, cos, sin, interleaved)

        output = self._post_process(output, origin_shape, origin_dtype)
        return output
```
**EN:** Defines function `ApplyRotaryEmb.forward_cuda` with signature `forward_cuda(self, x: torch.Tensor, cos: torch.Tensor, sin: torch.Tensor) -> torch.Tensor`. It mainly works with `x`, `cos`, `sin`; adjusts rotary-position-encoding parameters or application logic. The body uses mostly straightforward data movement and object wiring. Key calls include `self._pre_process`, `apply_rotary_emb`, `self._post_process`.
**CN:** 定义函数 `ApplyRotaryEmb.forward_cuda`，其签名为 `forward_cuda(self, x: torch.Tensor, cos: torch.Tensor, sin: torch.Tensor) -> torch.Tensor`。它主要围绕 `x`, `cos`, `sin` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `self._pre_process`, `apply_rotary_emb`, `self._post_process`。

### Method `ApplyRotaryEmb.forward_hip` (lines 250-276)
```python
    def forward_hip(
        self,
        x: torch.Tensor,
        cos: torch.Tensor,
        sin: torch.Tensor,
    ) -> torch.Tensor:
        if self.apply_rotary_emb_flash_attn is not None:
            x, cos, sin, origin_shape, origin_dtype = self._pre_process(x, cos, sin)

            """
            Arguments of apply_rotary() in flash_attn:
                x: [batch_size, seq_len, nheads, headdim]
                cos, sin: [seqlen_rotary, rotary_dim / 2]
                interleaved: default as False (Neox-style).
                ...
            """
            interleaved = not self.is_neox_style
            output = self.apply_rotary_emb_flash_attn(
                x, cos, sin, interleaved=interleaved
            ).type_as(x)

            output = self._post_process(output, origin_shape, origin_dtype)
        else:
            # Falling back to PyTorch native implementation.
            output = self.forward_native(x, cos, sin)

        return output
```
**EN:** Defines function `ApplyRotaryEmb.forward_hip` with signature `forward_hip(self, x: torch.Tensor, cos: torch.Tensor, sin: torch.Tensor) -> torch.Tensor`. It mainly works with `x`, `cos`, `sin`; adjusts rotary-position-encoding parameters or application logic. The body uses branching. Key calls include `self._pre_process`, `self.apply_rotary_emb_flash_attn.type_as`, `self._post_process`, `self.forward_native`, `self.apply_rotary_emb_flash_attn`.
**CN:** 定义函数 `ApplyRotaryEmb.forward_hip`，其签名为 `forward_hip(self, x: torch.Tensor, cos: torch.Tensor, sin: torch.Tensor) -> torch.Tensor`。它主要围绕 `x`, `cos`, `sin` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含分支判断。关键调用包括 `self._pre_process`, `self.apply_rotary_emb_flash_attn.type_as`, `self._post_process`, `self.forward_native`, `self.apply_rotary_emb_flash_attn`。

### Method `ApplyRotaryEmb.forward_cpu` (lines 278-285)
```python
    def forward_cpu(
        self,
        x: torch.Tensor,
        cos: torch.Tensor,
        sin: torch.Tensor,
    ) -> torch.Tensor:
        # TODO (bigPYJ1151): need to enable fused CPU ROPE here
        return self.forward_native(x, cos, sin)
```
**EN:** Defines function `ApplyRotaryEmb.forward_cpu` with signature `forward_cpu(self, x: torch.Tensor, cos: torch.Tensor, sin: torch.Tensor) -> torch.Tensor`. It mainly works with `x`, `cos`, `sin`; adjusts rotary-position-encoding parameters or application logic. The body uses mostly straightforward data movement and object wiring. Key calls include `self.forward_native`.
**CN:** 定义函数 `ApplyRotaryEmb.forward_cpu`，其签名为 `forward_cpu(self, x: torch.Tensor, cos: torch.Tensor, sin: torch.Tensor) -> torch.Tensor`。它主要围绕 `x`, `cos`, `sin` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `self.forward_native`。

## Key Concepts / 关键概念
- **EN:** The file implements a RoPE-related variant, meaning frequency scaling, cache construction, and position application are key ideas.
  **CN:** 该文件实现了与 RoPE 相关的变体，因此频率缩放、缓存构建和位置应用是关键概念。
- **EN:** Top-level classes include `ApplyRotaryEmb`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `ApplyRotaryEmb`，它们组织了主要的可复用抽象。
- **EN:** Top-level helpers such as `rotate_neox`, `rotate_gptj`, `yarn_find_correction_dim`, `yarn_find_correction_range`, `yarn_linear_ramp_mask` expose procedural entry points around the main classes.
  **CN:** 顶层辅助函数如 `rotate_neox`, `rotate_gptj`, `yarn_find_correction_dim`, `yarn_find_correction_range`, `yarn_linear_ramp_mask` 为主要类提供了过程式入口。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。

## Dependencies / 依赖关系
- **External / 外部**: `math`, `importlib`, `torch`
- **Internal / 内部**: `vllm.logger`, `vllm.model_executor.custom_op`, `vllm.platforms`, `vllm.utils.torch_utils`
