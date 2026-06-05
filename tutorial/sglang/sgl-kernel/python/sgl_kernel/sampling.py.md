# sampling.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/python/sgl_kernel/sampling.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module wraps native operators related to `sampling`, performs Python-side validation, and exposes a convenient interface. / 该模块封装与 `sampling` 相关的原生算子，执行 Python 侧校验，并暴露便捷接口。

## Line-by-Line Analysis / 逐行分析
### Lines 1-4: Imports and module setup
````python
from typing import Optional, Union

import torch
from sgl_kernel.utils import _to_tensor_scalar_tuple
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 6-11: Optional dependency handling
````python
try:
    import flashinfer.sampling as _flashinfer_sampling

    _has_flashinfer = True
except ImportError:
    _has_flashinfer = False
````
**EN:** This block attempts an operation that may fail, then provides fallback behavior so the module can keep working in reduced mode.
**CN:** 该部分尝试执行可能失败的操作，并提供回退行为，使模块能够以降级模式继续工作。

### Lines 14-25: `_top_k_renorm_probs_internal` definition
````python
def _top_k_renorm_probs_internal(
    probs: torch.Tensor,
    maybe_top_k_arr: Optional[torch.Tensor],
    top_k_val: int,
) -> torch.Tensor:
    probs = probs.float()
    maybe_top_k_arr = maybe_top_k_arr.int() if maybe_top_k_arr is not None else None
    renorm_probs = torch.empty_like(probs)
    torch.ops.sgl_kernel.top_k_renorm_probs.default(
        probs, renorm_probs, maybe_top_k_arr, top_k_val
    )
    return renorm_probs
````
**EN:** This section defines `_top_k_renorm_probs_internal` and implements the core logic associated with top k renorm probs internal. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `_top_k_renorm_probs_internal`，并实现与 top k renorm probs internal 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 28-59: `top_k_renorm_probs` definition
````python
def top_k_renorm_probs(
    probs: torch.Tensor,
    top_k: Union[torch.Tensor, int],
) -> torch.Tensor:
    r"""Adapt from https://github.com/flashinfer-ai/flashinfer/flashinfer/sampling.py
    Fused GPU kernel for renormalizing probabilities by top-k thresholding.

    Parameters
    ----------
    probs: torch.Tensor
        Probabilities, shape ``(batch_size, num_classes)``.
    top_k: Union[torch.Tensor, int]
        Either a scalar or a tensor of shape ``(batch_size,)``, representing the top-k threshold for for
        for re-normalizing probabilities, should be in ``(0, num_classes)``.
        If a scalar, the same threshold is used for all requests.
        If a tensor, each request has its own threshold.
        We keep the top-k probabilities, set the rest to zero, and renormalize the probabilities.

    Returns
    -------
    renorm_probs: torch.Tensor
        Renormalized probabilities, shape ``(batch_size, num_classes)``.

    Note
    ----
    This combination of ``top_k_renorm_probs`` and ``sampling_from_probs`` should be equivalent to
    ``top_k_sampling_from_probs``.
    """
    if probs.device.type == "musa" or not _has_flashinfer:
        return _top_k_renorm_probs_internal(probs, *_to_tensor_scalar_tuple(top_k))
    else:
        return _flashinfer_sampling.top_k_renorm_probs(probs, top_k)
````
**EN:** This section defines `top_k_renorm_probs` and implements the core logic associated with top k renorm probs. Docstring summary: Adapt from https://github.com/flashinfer-ai/flashinfer/flashinfer/sampling.py Fused GPU kernel for renormalizing probabilities by top-k thresholding.
**CN:** 该部分定义 `top_k_renorm_probs`，并实现与 top k renorm probs 相关的核心逻辑。 文档字符串摘要：Adapt from https://github.com/flashinfer-ai/flashinfer/flashinfer/sampling.py Fused GPU kernel for renormalizing probabilities by top-k thresholding.

### Lines 62-62: Constants and configuration
````python
top_k_renorm_prob = top_k_renorm_probs
````
**EN:** This block defines shared constants or configuration values such as `top_k_renorm_prob`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `top_k_renorm_prob`），供后续函数或控制流程复用。

### Lines 65-76: `_top_p_renorm_probs_internal` definition
````python
def _top_p_renorm_probs_internal(
    probs: torch.Tensor,
    maybe_top_p_arr: Optional[torch.Tensor],
    top_p_val: float,
) -> torch.Tensor:
    probs = probs.float()
    maybe_top_p_arr = maybe_top_p_arr.float() if maybe_top_p_arr is not None else None
    renorm_probs = torch.empty_like(probs)
    torch.ops.sgl_kernel.top_p_renorm_probs.default(
        probs, renorm_probs, maybe_top_p_arr, top_p_val
    )
    return renorm_probs
````
**EN:** This section defines `_top_p_renorm_probs_internal` and implements the core logic associated with top p renorm probs internal. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `_top_p_renorm_probs_internal`，并实现与 top p renorm probs internal 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 79-112: `top_p_renorm_probs` definition
````python
def top_p_renorm_probs(
    probs: torch.Tensor,
    top_p: Union[torch.Tensor, float],
) -> torch.Tensor:
    r"""Adapt from https://github.com/flashinfer-ai/flashinfer/flashinfer/sampling.py
    Fused GPU kernel for renormalizing probabilities by top-p thresholding.

    Parameters
    ----------
    probs: torch.Tensor
        Probabilities, shape ``(batch_size, num_classes)``.
    top_p: Union[torch.Tensor, float]
        Either a scalar or a tensor of shape ``(batch_size,)``, representing the top-p threshold for for
        re-normalizing probabilities, should be in ``(0, 1)``.
        If a scalar, the same threshold is used for all requests.
        If a tensor, each request has its own threshold.
        We mask out the probabilities less than `threshold` where the cumulative sum
        of ``probs[probs >= threshold]`` is `top_p`, and renormalize the probabilities.

    Returns
    -------
    renorm_probs: torch.Tensor
        Renormalized probabilities, shape ``(batch_size, num_classes)``.

    Note
    ----
    This combination of ``top_p_renorm_probs`` and ``sampling_from_probs`` should be equivalent to
    ``top_p_sampling_from_probs``.

    """
    if probs.device.type == "musa" or not _has_flashinfer:
        return _top_p_renorm_probs_internal(probs, *_to_tensor_scalar_tuple(top_p))
    else:
        return _flashinfer_sampling.top_p_renorm_probs(probs, top_p)
````
**EN:** This section defines `top_p_renorm_probs` and implements the core logic associated with top p renorm probs. Docstring summary: Adapt from https://github.com/flashinfer-ai/flashinfer/flashinfer/sampling.py Fused GPU kernel for renormalizing probabilities by top-p thresholding.
**CN:** 该部分定义 `top_p_renorm_probs`，并实现与 top p renorm probs 相关的核心逻辑。 文档字符串摘要：Adapt from https://github.com/flashinfer-ai/flashinfer/flashinfer/sampling.py Fused GPU kernel for renormalizing probabilities by top-p thresholding.

### Lines 115-115: Constants and configuration
````python
top_p_renorm_prob = top_p_renorm_probs
````
**EN:** This block defines shared constants or configuration values such as `top_p_renorm_prob`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `top_p_renorm_prob`），供后续函数或控制流程复用。

## Key Concepts / 关键概念
- **Role / 角色**: Python wrapper over compiled ops / 已编译算子的 Python 封装
- **Primary symbols / 主要符号**: `_top_k_renorm_probs_internal`, `top_k_renorm_probs`, `_top_p_renorm_probs_internal`, `top_p_renorm_probs`
- **Core technologies / 核心技术**: compiled custom ops / 已编译自定义算子, PyTorch tensors / PyTorch 张量

## Dependencies / 依赖关系
- **Internal / 内部**: `sgl_kernel.utils`
- **External / 外部**: `flashinfer.sampling`, `torch`, `typing`
- **Runtime hooks / 运行时钩子**: `torch.ops.sgl_kernel` custom operator namespace
