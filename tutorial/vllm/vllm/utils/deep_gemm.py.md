# deep_gemm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/utils/deep_gemm.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Compatibility wrapper for DeepGEMM API changes / 该模块围绕 `deep_gemm` 相关逻辑组织实现，提供与源文件相匹配的核心功能。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-29)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
"""Compatibility wrapper for DeepGEMM API changes.

Users of vLLM should always import **only** these wrappers.
"""

import functools
import importlib
import os
from collections.abc import Callable
from enum import Enum
from typing import Any, NoReturn

import torch

import vllm.envs as envs
from vllm.logger import logger
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    get_fp8_min_max,
)
from vllm.platforms import current_platform
from vllm.utils.import_utils import has_deep_gemm
from vllm.utils.math_utils import cdiv

_DEEPGEMM_BLACKWELL_EXCLUDED_MODEL_TYPES: set[str] = {
    "qwen3_5_text",
    "qwen3_5_moe_text",
}
```
**EN:** Sets up the module with standard-library support such as `functools`, `importlib`, `os`, external packages such as `torch`, vLLM modules such as `vllm.envs`, `vllm.logger`, `vllm.model_executor.layers.quantization.utils.quant_utils`. It prepares the symbols later used by `DeepGemmQuantScaleFMT`, `should_auto_disable_deep_gemm`, `is_deep_gemm_supported`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义，并接入 `vllm.envs`, `vllm.logger`, `vllm.model_executor.layers.quantization.utils.quant_utils` 等 vLLM 内部依赖。 这些准备工作为后续的 `DeepGemmQuantScaleFMT`, `should_auto_disable_deep_gemm`, `is_deep_gemm_supported` 提供上下文。

### should_auto_disable_deep_gemm (lines 32-42)
```python
def should_auto_disable_deep_gemm(model_type: str | None) -> bool:
    """Check if DeepGemm should be auto-disabled for this model on Blackwell.

    Returns True if the model is known to have accuracy degradation with
    DeepGemm's E8M0 scale format on Blackwell GPUs (SM100+).
    """
    if model_type is None:
        return False
    if not current_platform.is_device_capability_family(100):
        return False
    return model_type in _DEEPGEMM_BLACKWELL_EXCLUDED_MODEL_TYPES
```
**EN:** `should_auto_disable_deep_gemm`: Check if DeepGemm should be auto-disabled for this model on Blackwell. It mainly works with `model_type`. Inside the body, it relies on `current_platform.is_device_capability_family` to complete the main steps.
**CN:** `should_auto_disable_deep_gemm` 负责判断当前流程是否应当执行。 它主要处理 `model_type` 等参数。 实现过程中会调用 `current_platform.is_device_capability_family` 等函数完成关键步骤。

### DeepGemmQuantScaleFMT (lines 45-83)
```python
class DeepGemmQuantScaleFMT(Enum):
    # Float32 scales in Float32 tensor
    FLOAT32 = 0
    # Compute float32 scales and ceil the scales to UE8M0.
    # Keep the scales in Float32 tensor.
    FLOAT32_CEIL_UE8M0 = 1
    # Compute float32 scales and ceil the scales to UE8M0.
    # Pack the scales into a int32 tensor where each int32
    # element contains 4 scale values.
    UE8M0 = 2

    @classmethod
    def init_oracle_cache(cls) -> None:
        """Initialize the oracle decision and store it in the class cache"""
        cached = getattr(cls, "_oracle_cache", None)
        if cached is not None:
            return

        use_e8m0 = (
            envs.VLLM_USE_DEEP_GEMM_E8M0
            and is_deep_gemm_supported()
            and (_fp8_gemm_nt_impl is not None)
        )
        if not use_e8m0:
            cls._oracle_cache = cls.FLOAT32  # type: ignore
            return

        cls._oracle_cache = (  # type: ignore
    # ...
    @classmethod
    def from_oracle(cls) -> "DeepGemmQuantScaleFMT":
        """Return the pre-initialized oracle decision"""
        cached = getattr(cls, "_oracle_cache", None)
        assert cached is not None, "DeepGemmQuantScaleFMT oracle cache not initialized"
        return cached
```
**EN:** Defines the `DeepGemmQuantScaleFMT` class used by this module. It extends `Enum`. Key methods include `init_oracle_cache`, `from_oracle`.
**CN:** `DeepGemmQuantScaleFMT` 是该文件中的核心类，用于封装与 `DeepGemmQuantScaleFMT` 相关的状态和行为。 它继承自 `Enum`。 关键方法包括 `init_oracle_cache`, `from_oracle`。

### is_deep_gemm_supported (lines 87-92)
```python
def is_deep_gemm_supported() -> bool:
    """Return `True` if DeepGEMM is supported on the current platform.
    Currently, only Hopper and Blackwell GPUs are supported.
    """
    is_supported_arch = current_platform.support_deep_gemm()
    return envs.VLLM_USE_DEEP_GEMM and has_deep_gemm() and is_supported_arch
```
**EN:** `is_deep_gemm_supported`: Return `True` if DeepGEMM is supported on the current platform. Inside the body, it relies on `current_platform.support_deep_gemm`, `has_deep_gemm` to complete the main steps.
**CN:** `is_deep_gemm_supported` 负责检查条件并返回布尔结果。 实现过程中会调用 `current_platform.support_deep_gemm`, `has_deep_gemm` 等函数完成关键步骤。

### is_deep_gemm_e8m0_used (lines 96-117)
```python
def is_deep_gemm_e8m0_used() -> bool:
    """Return `True` if vLLM is configured to use DeepGEMM "
    "E8M0 scale on a Hopper or Blackwell-class GPU.
    """
    if not is_deep_gemm_supported():
        logger.debug_once(
            "DeepGEMM E8M0 disabled: DeepGEMM not supported on this system."
        )
        return False

    _lazy_init()

    if _fp8_gemm_nt_impl is None:
        logger.info_once("DeepGEMM E8M0 disabled: _fp8_gemm_nt_impl not found")
        return False

    if envs.VLLM_USE_DEEP_GEMM_E8M0:
        logger.info_once("DeepGEMM E8M0 enabled on current platform.")
        return True

    logger.info_once("DeepGEMM E8M0 disabled on current configuration.")
    return False
```
**EN:** `is_deep_gemm_e8m0_used`: Return `True` if vLLM is configured to use DeepGEMM " "E8M0 scale on a Hopper or Blackwell-class GPU. Inside the body, it relies on `_lazy_init`, `logger.info_once`, `is_deep_gemm_supported` to complete the main steps.
**CN:** `is_deep_gemm_e8m0_used` 负责检查条件并返回布尔结果。 实现过程中会调用 `_lazy_init`, `logger.info_once`, `is_deep_gemm_supported` 等函数完成关键步骤。

### _missing (lines 120-125)
```python
def _missing(*_: Any, **__: Any) -> NoReturn:
    """Placeholder for unavailable DeepGEMM backend."""
    raise RuntimeError(
        "DeepGEMM backend is not available or outdated. Please install or "
        "update the `deep_gemm` to a newer version to enable FP8 kernels."
    )
```
**EN:** `_missing`: Placeholder for unavailable DeepGEMM backend. It mainly works with `*_`, `**__`. Inside the body, it relies on `RuntimeError` to complete the main steps.
**CN:** `_missing` 负责实现本模块使用的辅助逻辑。 它主要处理 `*_`, `**__` 等参数。 实现过程中会调用 `RuntimeError` 等函数完成关键步骤。

### _import_deep_gemm (lines 143-175)
```python
def _import_deep_gemm():
    """Import the deep_gemm module.

    Prefers an externally installed ``deep_gemm`` package (so users can
    pin a specific version), then falls back to the vendored copy bundled
    in the vLLM wheel.

    Returns ``None`` when neither source is usable.
    """
    # 1. Try the external (pip-installed) package first.
    try:
        module = importlib.import_module("deep_gemm")
        logger.debug_once("Imported deep_gemm module from site-packages")
        return module
    except ImportError:
        logger.debug_once(
            "deep_gemm not found in site-packages, "
            "trying vendored vllm.third_party.deep_gemm"
        )

    # 2. Fall back to the vendored copy bundled in the vLLM wheel.
    try:
        module = importlib.import_module("vllm.third_party.deep_gemm")
        logger.debug_once("Imported deep_gemm module from vllm.third_party.deep_gemm")
    # ...
    except Exception as e:
        # The vendored module may raise RuntimeError during _C.init()
        # if JIT include files are missing (e.g. incomplete wheel).
        logger.warning_once("Failed to import vendored deep_gemm: %s", e)

    return None
```
**EN:** `_import_deep_gemm`: Import the deep_gemm module. Inside the body, it relies on `importlib.import_module`, `logger.debug_once`, `logger.warning_once` to complete the main steps.
**CN:** `_import_deep_gemm` 负责实现本模块使用的辅助逻辑。 实现过程中会调用 `importlib.import_module`, `logger.debug_once`, `logger.warning_once` 等函数完成关键步骤。

### _lazy_init (lines 178-243)
```python
def _lazy_init() -> None:
    """Import deep_gemm and resolve symbols on first use."""
    global _cublaslt_gemm_nt_impl
    global _fp8_gemm_nt_impl, _fp8_einsum_impl
    global _grouped_impl, _grouped_masked_impl, _grouped_fp4_impl
    global _fp8_fp4_mqa_logits_impl, _fp8_fp4_paged_mqa_logits_impl
    global _get_paged_mqa_logits_metadata_impl
    global _tf32_hc_prenorm_gemm_impl
    global _get_mn_major_tma_aligned_tensor_impl
    global _get_mk_alignment_for_contiguous_layout_impl
    global _transform_sf_into_required_layout_impl
    # fast path
    if (
        _cublaslt_gemm_nt_impl is not None
        or _fp8_gemm_nt_impl is not None
        or _fp8_einsum_impl is not None
        or _grouped_impl is not None
        or _grouped_masked_impl is not None
        or _grouped_fp4_impl is not None
        or _fp8_fp4_mqa_logits_impl is not None
        or _fp8_fp4_paged_mqa_logits_impl is not None
        or _get_paged_mqa_logits_metadata_impl is not None
        or _tf32_hc_prenorm_gemm_impl is not None
        or _get_mk_alignment_for_contiguous_layout_impl is not None
    # ...
        _dg, "get_mk_alignment_for_contiguous_layout", None
    )
    _transform_sf_into_required_layout_impl = getattr(
        _dg, "transform_sf_into_required_layout", None
    )
    DeepGemmQuantScaleFMT.init_oracle_cache()
```
**EN:** `_lazy_init`: Import deep_gemm and resolve symbols on first use. Inside the body, it relies on `_import_deep_gemm`, `DeepGemmQuantScaleFMT.init_oracle_cache`, `has_deep_gemm` to complete the main steps.
**CN:** `_lazy_init` 负责实现本模块使用的辅助逻辑。 实现过程中会调用 `_import_deep_gemm`, `DeepGemmQuantScaleFMT.init_oracle_cache`, `has_deep_gemm` 等函数完成关键步骤。

### fp8_fp4_mqa_logits (lines 341-383)
```python
def fp8_fp4_mqa_logits(
    q: tuple[torch.Tensor, torch.Tensor | None],
    kv: tuple[torch.Tensor, torch.Tensor],
    weights: torch.Tensor,
    cu_seqlen_ks: torch.Tensor,
    cu_seqlen_ke: torch.Tensor,
    clean_logits: bool,
) -> torch.Tensor:
    """Compute MQA logits for a single sequence without KV paging.

    Unified FP8/FP4 dispatch — the underlying DeepGEMM kernel takes
    ``q = (values, scales_or_None)`` where ``scales`` is None for FP8 Q
    (per-token scale is folded into ``weights``) and a packed block-scale
    tensor for MXFP4 Q.

    Args:
        q: Tuple ``(q_values, q_scale)``. FP8 path: q_values is [M, H, D]
            float8_e4m3fn and q_scale is None (per-token scale is folded
            into ``weights``). FP4 path: q_values is packed uint8 and
            q_scale is the companion block-scale tensor.
        kv: Tuple `(k_packed, k_scales)` — FP8 layout is [N, D]
            float8_e4m3fn plus fp32 scales [N]; FP4 layout is packed uint8.
        weights: weights of shape [M, H], dtype `torch.float32`.
        cu_seqlen_ks: Start indices (inclusive) for valid K per query
    # ...
        kv,
        weights,
        cu_seqlen_ks,
        cu_seqlen_ke,
        clean_logits=clean_logits,
    )
```
**EN:** `fp8_fp4_mqa_logits`: Compute MQA logits for a single sequence without KV paging. It mainly works with `q`, `kv`, `weights`, `cu_seqlen_ks`. Inside the body, it relies on `_lazy_init`, `_fp8_fp4_mqa_logits_impl`, `_missing` to complete the main steps.
**CN:** `fp8_fp4_mqa_logits` 负责实现本模块使用的辅助逻辑。 它主要处理 `q`, `kv`, `weights`, `cu_seqlen_ks` 等参数。 实现过程中会调用 `_lazy_init`, `_fp8_fp4_mqa_logits_impl`, `_missing` 等函数完成关键步骤。

### get_paged_mqa_logits_metadata (lines 386-404)
```python
def get_paged_mqa_logits_metadata(
    context_lens: torch.Tensor, block_size: int, num_sms: int
) -> torch.Tensor:
    """Build scheduling metadata for paged MQA logits.

    Args:
        context_lens: Tensor of shape [B], dtype int32; effective context length
            per batch element.
        block_size: KV-cache block size in tokens (e.g., 64).
        num_sms: Number of SMs available. 132 for Hopper

    Returns:
        Backend-specific tensor consumed by `fp8_fp4_paged_mqa_logits` to
        schedule work across SMs.
    """
    _lazy_init()
    if _get_paged_mqa_logits_metadata_impl is None:
        return _missing()
    return _get_paged_mqa_logits_metadata_impl(context_lens, block_size, num_sms)
```
**EN:** `get_paged_mqa_logits_metadata`: Build scheduling metadata for paged MQA logits. It mainly works with `context_lens`, `block_size`, `num_sms`. Inside the body, it relies on `_lazy_init`, `_get_paged_mqa_logits_metadata_impl`, `_missing` to complete the main steps.
**CN:** `get_paged_mqa_logits_metadata` 负责获取流水线所需的数据或状态。 它主要处理 `context_lens`, `block_size`, `num_sms` 等参数。 实现过程中会调用 `_lazy_init`, `_get_paged_mqa_logits_metadata_impl`, `_missing` 等函数完成关键步骤。

### fp8_fp4_paged_mqa_logits (lines 407-457)
```python
def fp8_fp4_paged_mqa_logits(
    q: tuple[torch.Tensor, torch.Tensor | None],
    kv_cache: torch.Tensor,
    weights: torch.Tensor,
    context_lens: torch.Tensor,
    block_tables: torch.Tensor,
    schedule_metadata: torch.Tensor,
    max_model_len: int,
    clean_logits: bool,
) -> torch.Tensor:
    """Compute MQA logits using a paged KV-cache.

    Unified FP8/FP4 dispatch — the underlying DeepGEMM kernel takes
    ``q = (values, scales_or_None)``; pass ``(q_tensor, None)`` for the FP8
    path and ``(q_values, q_scale)`` for MXFP4.

    Args:
        q: Tuple ``(q_values, q_scale)``. FP8 path: q_values is
            [B, next_n, H, D] float8_e4m3fn and q_scale is None. FP4 path:
            q_values is packed uint8 and q_scale is the companion
            block-scale tensor.
        kv_cache: Paged KV-cache. FP8 layout is [num_blocks, block_size, 1,
            D+4], dtype `torch.uint8`, with the last 4 bytes per (block, pos)
            storing the float dequant scale.
    # ...
        context_lens,
        block_tables,
        schedule_metadata,
        max_model_len,
        clean_logits=clean_logits,
    )
```
**EN:** `fp8_fp4_paged_mqa_logits`: Compute MQA logits using a paged KV-cache. It mainly works with `q`, `kv_cache`, `weights`, `context_lens`. Inside the body, it relies on `_lazy_init`, `_fp8_fp4_paged_mqa_logits_impl`, `_missing` to complete the main steps.
**CN:** `fp8_fp4_paged_mqa_logits` 负责实现本模块使用的辅助逻辑。 它主要处理 `q`, `kv_cache`, `weights`, `context_lens` 等参数。 实现过程中会调用 `_lazy_init`, `_fp8_fp4_paged_mqa_logits_impl`, `_missing` 等函数完成关键步骤。

### tf32_hc_prenorm_gemm (lines 460-483)
```python
def tf32_hc_prenorm_gemm(
    x: torch.Tensor,
    fn: torch.Tensor,
    out: torch.Tensor,
    sqrsum: torch.Tensor,
    num_split: int,
) -> torch.Tensor:
    """
    Perform the following computation:
        out = x.float() @ fn.T
        sqrsum = x.float().square().sum(-1)

    See the caller function for shape requirement
    """
    _lazy_init()
    if _tf32_hc_prenorm_gemm_impl is None:
        return _missing()
    return _tf32_hc_prenorm_gemm_impl(
        x,
        fn,
        out,
        sqrsum,
        num_split,
    )
```
**EN:** `tf32_hc_prenorm_gemm`: Perform the following computation: out = x.float() @ fn.T sqrsum = x.float().square().sum(-1) See the caller function for shape requirement. It mainly works with `x`, `fn`, `out`, `sqrsum`. Inside the body, it relies on `_lazy_init`, `_tf32_hc_prenorm_gemm_impl`, `_missing` to complete the main steps.
**CN:** `tf32_hc_prenorm_gemm` 负责实现本模块使用的辅助逻辑。 它主要处理 `x`, `fn`, `out`, `sqrsum` 等参数。 实现过程中会调用 `_lazy_init`, `_tf32_hc_prenorm_gemm_impl`, `_missing` 等函数完成关键步骤。

### calc_diff (lines 526-539)
```python
def calc_diff(x: torch.Tensor, y: torch.Tensor):
    """Return a global difference metric for unit tests.

    DeepGEMM kernels on Blackwell/B200 currently exhibit noticeable per-element
    error, causing `torch.testing.assert_close` to fail.  Instead of checking
    every element, we compute a cosine-style similarity over the whole tensor
    and report `1 - sim`.  Once kernel accuracy improves this helper can be
    removed.
    """

    x, y = x.double(), y.double()
    denominator = (x * x + y * y).sum()
    sim = 2 * (x * y).sum() / denominator
    return 1 - sim
```
**EN:** `calc_diff`: Return a global difference metric for unit tests. It mainly works with `x`, `y`. Inside the body, it relies on `sum`, `x.double`, `y.double` to complete the main steps.
**CN:** `calc_diff` 负责实现本模块使用的辅助逻辑。 它主要处理 `x`, `y` 等参数。 实现过程中会调用 `sum`, `x.double`, `y.double` 等函数完成关键步骤。

## Key Concepts / 关键概念
- **`DeepGemmQuantScaleFMT`**: Core class that organizes module behavior. / **`DeepGemmQuantScaleFMT`**：组织模块行为的核心类。
- **`should_auto_disable_deep_gemm`**: Key helper or entry point in this file. / **`should_auto_disable_deep_gemm`**：本文件中的关键辅助函数或入口。
- **`is_deep_gemm_supported`**: Key helper or entry point in this file. / **`is_deep_gemm_supported`**：本文件中的关键辅助函数或入口。
- **`is_deep_gemm_e8m0_used`**: Key helper or entry point in this file. / **`is_deep_gemm_e8m0_used`**：本文件中的关键辅助函数或入口。

## Dependencies / 依赖关系
- **Standard library / 标准库**: functools, importlib, os, collections.abc, enum, typing
- **Third-party / 第三方**: torch
- **Internal vLLM / vLLM 内部依赖**: vllm.envs, vllm.logger, vllm.model_executor.layers.quantization.utils.quant_utils, vllm.platforms, vllm.utils.import_utils, vllm.utils.math_utils
