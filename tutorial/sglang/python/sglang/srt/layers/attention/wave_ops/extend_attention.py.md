# extend_attention.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/wave_ops/extend_attention.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module manages extend attention logic around KV-cache layout, indexing, or paging for SGLang attention execution. / 该模块管理与 extend attention 相关的 KV 缓存布局、索引或分页逻辑，用于 SGLang 注意力执行。
## Line-by-Line Analysis / 逐行分析
### Lines 1-4: docstring
```python
"""
Memory-efficient attention for prefill.
It support page size = 1.
"""
```
**EN:** Provides inline documentation that explains the scope of the surrounding module or class.
**CN:** 提供内联文档，用于说明周围模块或类的职责范围。

### Lines 6-17: imports
```python
import functools
import os

import torch
from wave_lang.kernel.lang.global_symbols import *
from wave_lang.kernel.wave.compile import WaveCompileOptions, wave_compile
from wave_lang.kernel.wave.constraints import MMAType
from wave_lang.kernel.wave.scheduling.schedule import SchedulingType
from wave_lang.kernel.wave.templates.attention_common import AttentionShape
from wave_lang.kernel.wave.templates.extend_attention import get_extend_attention_kernel
from wave_lang.kernel.wave.utils.general_utils import get_default_scheduling_params
from wave_lang.kernel.wave.utils.run_utils import set_default_run_config
```
**EN:** Imports PyTorch, optional accelerator libraries, and internal SGLang modules required by the attention path.
**CN:** 导入该注意力路径所需的 PyTorch、可选加速库以及 SGLang 内部模块。

### Lines 19-19: module constants
```python
dump_generated_mlir = int(os.environ.get("WAVE_DUMP_MLIR", 0))
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 22-78: function get_wave_kernel
```python
@functools.lru_cache
def get_wave_kernel(
    shape: AttentionShape,
    q_shape: tuple[int],
    k_shape: tuple[int],
    v_shape: tuple[int],
    k_cache_shape: tuple[int],
    v_cache_shape: tuple[int],
    o_shape: tuple[int],
    input_dtype: torch.dtype,
    output_dtype: torch.dtype,
    size_dtype: torch.dtype,
    is_causal: bool,
    logit_cap: float,
    layer_scaling: float,
):
    assert shape.num_query_heads % shape.num_kv_heads == 0

    mfma_variant = (MMAType.F32_16x16x32_K8_F16, MMAType.F32_16x16x16_F16)
    (
        extend_attention,
        hyperparams,
        dynamic_symbols,
    ) = get_extend_attention_kernel(
        shape,
        mfma_variant,
        q_shape,
        k_shape,
        v_shape,
        k_cache_shape,
        v_cache_shape,
        o_shape,
# ... omitted 13 lines ...
        schedule=SchedulingType.NONE,
        use_scheduling_barriers=False,
        dynamic_symbols=dynamic_symbols,
        use_buffer_ops=True,
        waves_per_eu=2,
        denorm_fp_math_f32="preserve-sign",
        wave_runtime=True,
    )
    options = set_default_run_config(options)
    extend_attention = wave_compile(options, extend_attention)

    return extend_attention
```
**EN:** Computes and returns get wave kernel from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get wave kernel。

### Lines 81-147: function extend_attention_wave
```python
def extend_attention_wave(
    q_extend,
    k_extend,
    v_extend,
    k_buffer,
    v_buffer,
    qo_indptr,
    kv_indptr,
    kv_indices,
    custom_mask,
    mask_indptr,
    max_seq_len,
    output,
    is_causal=True,
    layer_scaling=None,
    logit_cap=0,
):
    shape = AttentionShape(
        num_query_heads=q_extend.shape[1],
        num_kv_heads=k_extend.shape[1],
        head_size=q_extend.shape[2],
        head_size_kv=k_extend.shape[2],
        num_seqs=kv_indptr.shape[0] - 1,
        max_seq_len=max_seq_len,
    )

    # Run the wave kernel.
    extend_attention = get_wave_kernel(
        shape,
        q_extend.shape,
        k_extend.shape,
        v_extend.shape,
# ... omitted 23 lines ...

    if dump_generated_mlir:
        shape_list = [
            q_extend.shape[0],
            q_extend.shape[1],
            k_extend.shape[1],
            q_extend.shape[2],
            k_extend.shape[2],
        ]
        filename = f"wave_prefill_attention_{'x'.join(map(str, shape_list))}.mlir"
        with open(filename, "w") as f:
            f.write(mb.module_op.get_asm())
```
**EN:** Implements the extend attention wave routine used by this attention module.
**CN:** 实现该注意力模块使用的 extend attention wave 例程。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** KV-cache management / **CN:** KV 缓存管理
- **EN:** Paged attention layouts / **CN:** 分页注意力布局

## Dependencies / 依赖关系
- `functools`
- `os`
- `torch`
- `wave_lang.kernel.lang.global_symbols.*`
- `wave_lang.kernel.wave.compile.WaveCompileOptions`
- `wave_lang.kernel.wave.compile.wave_compile`
- `wave_lang.kernel.wave.constraints.MMAType`
- `wave_lang.kernel.wave.scheduling.schedule.SchedulingType`
- `wave_lang.kernel.wave.templates.attention_common.AttentionShape`
- `wave_lang.kernel.wave.templates.extend_attention.get_extend_attention_kernel`
- `wave_lang.kernel.wave.utils.general_utils.get_default_scheduling_params`
- `wave_lang.kernel.wave.utils.run_utils.set_default_run_config`
