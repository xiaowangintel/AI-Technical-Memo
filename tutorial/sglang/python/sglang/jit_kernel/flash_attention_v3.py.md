# flash_attention_v3.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/flash_attention_v3.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "import logging". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“import logging”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Imports and module setup
```python
import logging
import os
from typing import Optional, Union

import torch

from sglang.jit_kernel.utils import cache_once
from sglang.kernel_api_logging import debug_kernel_api
from sglang.srt.environ import envs
from sglang.srt.utils import get_device_capability, is_musa

logger = logging.getLogger(__name__)

SGL_FA3_KERNEL_REPO = "kernels-community/sgl-flash-attn3"
SGL_FA3_KERNEL_REVISION = "v1"
DEFAULT_FA3_KERNEL_LOCKFILE = "kernels.lock"
```
**EN:** This block imports dependencies and prepares names used later in the module. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 19-30: Function `_call_fa3_kernel`
```python
def _call_fa3_kernel(kernel, *args, out=None, **kwargs):
    if out is None:
        return kernel(*args, **kwargs)
    try:
        return kernel(*args, **kwargs, out=out)
    except TypeError as exc:
        if "unexpected keyword argument 'out'" not in str(exc):
            raise
        return kernel(*args, **kwargs)


@cache_once
```
**EN:** This block defines `_call_fa3_kernel`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_call_fa3_kernel`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 31-70: Imports and module setup
```python
def _load_fa3_kernels():
    # By default, we use the implementation from sgl-kernel,
    # which is expected to be more stable and compatible
    if envs.SGLANG_USE_SGL_FA3_KERNEL.get():
        logger.debug(
            f"SGLANG_USE_SGL_FA3_KERNEL=True, use sgl-kernel implementation for FlashAttention v3 "
        )
        return _load_fa3_kernel_from_sgl()

    # Otherwise, we try to load the kernels from the kernels community cache directory or kernels community repo
    lockfile_path = os.path.join(
        envs.SGLANG_CACHE_DIR.get(), DEFAULT_FA3_KERNEL_LOCKFILE
    )

    try:
        from kernels import get_kernel, load_kernel

        # When the lock file provided, load from the kernel cache directory,
        # otherwise, load from the repo, which require download from huggingface hub
        # but always works as long as the repo is accessible.
        if os.path.exists(lockfile_path):
            ops = load_kernel(SGL_FA3_KERNEL_REPO, lockfile_path)
        else:
            ops = get_kernel(SGL_FA3_KERNEL_REPO, revision=SGL_FA3_KERNEL_REVISION)

        return {
            "flash_attn_with_kvcache": ops.flash_attn_with_kvcache,
            "flash_attn_varlen_func": ops.flash_attn_varlen_func,
        }
    except Exception as e:
        # When the kernels from the repo or the cache directory cannot be loaded
        # we catch the exception and log a warning, and then fallback to the implementation
        # from sgl-kernel, which is expected to be less efficient but more compatible.
        logger.warning(
            f"Rollback to implementation from sgl-kernel since loading FlashAttention v3 "
            f"kernels from {SGL_FA3_KERNEL_REPO} with lockfile {lockfile_path} failed: {e}"
        )
        return _load_fa3_kernel_from_sgl()
```
**EN:** This block imports dependencies and prepares names used later in the module. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 71-83: Imports and module setup
```python
def _load_fa3_kernel_from_sgl():
    from sgl_kernel.flash_attn import (
        flash_attn_varlen_func,
        flash_attn_with_kvcache,
    )

    return {
        "flash_attn_with_kvcache": flash_attn_with_kvcache,
        "flash_attn_varlen_func": flash_attn_varlen_func,
    }


@cache_once
```
**EN:** This block imports dependencies and prepares names used later in the module. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 84-101: Function `_is_fa3_supported`
```python
def _is_fa3_supported(device=None) -> bool:
    #  There some fa3 FYI
    #  FA3 can fail without a enough shared memory for a some shapes, such as higher
    #  hidden_dim or some special cases.
    #  Right now, fa3 is supported for sm80/sm87 and sm86/sm89. The main different
    #  Between sm80/sm87 and sm86/sm89 is the shared memory size. you can follow the link below for more information
    #  https://docs.nvidia.com/cuda/cuda-c-programming-guide/#shared-memory-8-x
    #  And for sgl-kernel right now, we can build fa3 on sm80/sm86/sm89/sm90a.
    #  That means if you use A100/A*0/L20/L40/L40s/4090 you can use fa3.
    major, minor = get_device_capability()
    if is_musa():
        return major >= 3
    if torch.version.cuda is not None and torch.version.cuda >= "12.3":
        return major == 9 or major == 8
    return False


@debug_kernel_api
```
**EN:** This block defines `_is_fa3_supported`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_is_fa3_supported`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 102-181: Function `flash_attn_with_kvcache`
```python
def flash_attn_with_kvcache(
    q,
    k_cache,
    v_cache,
    k=None,
    v=None,
    qv=None,
    rotary_cos=None,
    rotary_sin=None,
    cache_seqlens: Optional[Union[int, torch.Tensor]] = None,
    cache_batch_idx: Optional[torch.Tensor] = None,
    cache_leftpad: Optional[torch.Tensor] = None,
    page_table: Optional[torch.Tensor] = None,
    cu_seqlens_q: Optional[torch.Tensor] = None,
    cu_seqlens_k_new: Optional[torch.Tensor] = None,
    max_seqlen_q: Optional[int] = None,
    rotary_seqlens: Optional[torch.Tensor] = None,
    q_descale: Optional[torch.Tensor] = None,
    k_descale: Optional[torch.Tensor] = None,
    v_descale: Optional[torch.Tensor] = None,
    softmax_scale=None,
    causal=False,
    window_size=(-1, -1),  # -1 means infinite context window
    attention_chunk: Optional[int] = None,
    softcap=0.0,  # 0.0 means deactivated
    rotary_interleaved=True,
    scheduler_metadata=None,
    num_splits=0,  # Can be tuned for speed
    pack_gqa=None,  # Can be tuned for speed
    sm_margin=0,  # Can be tuned if some SMs are used for communication
    return_softmax_lse=False,
    sinks=None,
    out=None,
):
    if not _is_fa3_supported():
        raise NotImplementedError(
            "flash_attn at sgl-kernel is only supported on sm90 and above"
        )

    assert k_cache.stride(-1) == 1, "k_cache must have contiguous last dimension"
    assert v_cache.stride(-1) == 1, "v_cache must have contiguous last dimension"

    return _call_fa3_kernel(
        _load_fa3_kernels()["flash_attn_with_kvcache"],
        q,
        k_cache,
        v_cache,
        k,
        v,
        qv,
        rotary_cos,
        rotary_sin,
        cache_seqlens,
        cache_batch_idx,
        cache_leftpad,
        page_table,
        cu_seqlens_q,
        cu_seqlens_k_new,
        max_seqlen_q,
        rotary_seqlens,
        q_descale,
        k_descale,
        v_descale,
        softmax_scale,
        causal,
        window_size,
        attention_chunk,
        softcap,
        rotary_interleaved,
        scheduler_metadata,
        num_splits,
        pack_gqa,
        sm_margin,
        return_softmax_lse,
        sinks,
        out=out,
    )


@debug_kernel_api
```
**EN:** This block defines `flash_attn_with_kvcache`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `flash_attn_with_kvcache`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 182-272: Imports and module setup
```python
def flash_attn_varlen_func(
    q,
    k,
    v,
    cu_seqlens_q,
    cu_seqlens_k,
    max_seqlen_q=None,
    max_seqlen_k=None,
    seqused_q=None,
    seqused_k=None,
    page_table=None,
    softmax_scale=None,
    causal=False,
    qv=None,
    q_descale=None,
    k_descale=None,
    v_descale=None,
    window_size=(-1, -1),
    attention_chunk=0,
    softcap=0.0,
    num_splits=1,
    pack_gqa=None,
    sm_margin=0,
    return_softmax_lse=False,
    sinks=None,
    out=None,
):

    if not _is_fa3_supported():
        # Fall back to flash_attn package (FA2) on platforms without sgl-kernel FA3
        # (e.g. ROCm, or CUDA < sm90)
        if cu_seqlens_q is not None:
            from flash_attn import flash_attn_varlen_func as fa2_flash_attn_varlen_func

            return fa2_flash_attn_varlen_func(
                q,
                k,
                v,
                cu_seqlens_q,
                cu_seqlens_k,
                max_seqlen_q,
                max_seqlen_k,
                softmax_scale=softmax_scale,
                causal=causal,
                window_size=window_size,
                softcap=softcap,
                return_attn_probs=return_softmax_lse,
            )
        else:
            # 4D inputs (batch, seqlen, nheads, headdim) without cu_seqlens
            from flash_attn import flash_attn_func as fa2_flash_attn_func

            return fa2_flash_attn_func(
                q,
                k,
                v,
                softmax_scale=softmax_scale,
                causal=causal,
                window_size=window_size,
                softcap=softcap,
                return_attn_probs=return_softmax_lse,
            )

    return _call_fa3_kernel(
        _load_fa3_kernels()["flash_attn_varlen_func"],
        q=q,
        k=k,
        v=v,
        cu_seqlens_q=cu_seqlens_q,
        cu_seqlens_k=cu_seqlens_k,
        max_seqlen_q=max_seqlen_q,
        max_seqlen_k=max_seqlen_k,
        seqused_q=seqused_q,
        seqused_k=seqused_k,
        page_table=page_table,
        softmax_scale=softmax_scale,
        causal=causal,
        qv=qv,
        q_descale=q_descale,
        k_descale=k_descale,
# ...
```
**EN:** This block imports dependencies and prepares names used later in the module. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `logging`
- `os`
- `typing -> Optional`
- `torch`
- `sglang.jit_kernel.utils -> cache_once`
- `sglang.kernel_api_logging -> debug_kernel_api`
- `sglang.srt.environ -> envs`
- `sglang.srt.utils -> get_device_capability`
- `kernels -> get_kernel`
- `sgl_kernel.flash_attn -> (`
- `flash_attn -> flash_attn_varlen_func`
- `flash_attn -> flash_attn_func`
