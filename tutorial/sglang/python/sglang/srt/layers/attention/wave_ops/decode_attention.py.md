# decode_attention.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/wave_ops/decode_attention.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module manages decode attention logic around KV-cache layout, indexing, or paging for SGLang attention execution. / 该模块管理与 decode attention 相关的 KV 缓存布局、索引或分页逻辑，用于 SGLang 注意力执行。
## Line-by-Line Analysis / 逐行分析
### Lines 1-4: docstring
```python
"""
Memory-efficient attention for decoding.
It supports page size = 1.
"""
```
**EN:** Provides inline documentation that explains the scope of the surrounding module or class.
**CN:** 提供内联文档，用于说明周围模块或类的职责范围。

### Lines 6-18: imports
```python
import functools
import logging

from wave_lang.kernel.lang.global_symbols import *
from wave_lang.kernel.wave.compile import WaveCompileOptions, wave_compile
from wave_lang.kernel.wave.constraints import GenericDot, MMAOperand, MMAType
from wave_lang.kernel.wave.templates.paged_decode_attention import (
    get_paged_decode_attention_kernels,
    get_paged_decode_intermediate_arrays_shapes,
    paged_decode_attention_shape,
)
from wave_lang.kernel.wave.utils.general_utils import get_default_scheduling_params
from wave_lang.kernel.wave.utils.run_utils import set_default_run_config
```
**EN:** Imports the external and internal dependencies consumed by the code that follows.
**CN:** 导入后续代码所依赖的外部与内部模块。

### Lines 20-20: module constants
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 21-21: imports
```python
import os
```
**EN:** Imports the external and internal dependencies consumed by the code that follows.
**CN:** 导入后续代码所依赖的外部与内部模块。

### Lines 23-23: module constants
```python
dump_generated_mlir = int(os.environ.get("WAVE_DUMP_MLIR", 0))
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 26-87: function get_wave_kernel
```python
@functools.lru_cache(maxsize=4096)
def get_wave_kernel(
    shape: paged_decode_attention_shape,
    max_kv_splits,
    input_dtype,
    output_dtype,
    logit_cap,
):
    mha = (shape.num_query_heads // shape.num_kv_heads) == 1

    # Get the kernels (either compile or load from cache).
    if mha:
        mfma_variant = (
            GenericDot(along_dim=MMAOperand.M, k_vec_size=4, k_mult=1),
            GenericDot(along_dim=MMAOperand.M, k_vec_size=1, k_mult=64),
        )
    else:
        mfma_variant = (MMAType.F32_16x16x16_F16, MMAType.F32_16x16x16_F16)

    (
        phase_0,
        phase_1,
        hyperparams_0,
        hyperparams_1,
        dynamic_symbols_0,
        dynamic_symbols_1,
    ) = get_paged_decode_attention_kernels(
        shape,
        mfma_variant,
        max_kv_splits,
        input_dtype=input_dtype,
        output_dtype=output_dtype,
# ... omitted 18 lines ...
        subs=hyperparams_1,
        canonicalize=True,
        run_bench=False,
        use_buffer_ops=False,
        waves_per_eu=4,
        dynamic_symbols=dynamic_symbols_1,
        wave_runtime=True,
    )
    options = set_default_run_config(options)
    phase_1 = wave_compile(options, phase_1)

    return phase_0, phase_1
```
**EN:** Computes and returns get wave kernel from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get wave kernel。

### Lines 90-102: function decode_attention_intermediate_arrays_shapes
```python
def decode_attention_intermediate_arrays_shapes(
    num_seqs, head_size_kv, num_query_heads, max_kv_splits
):
    # Not all fields are used, but we need to pass them to the function
    shape = paged_decode_attention_shape(
        num_query_heads=num_query_heads,
        num_kv_heads=0,
        head_size=0,
        head_size_kv=head_size_kv,
        block_size=0,
        num_seqs=num_seqs,
    )
    return get_paged_decode_intermediate_arrays_shapes(shape, max_kv_splits)
```
**EN:** Implements the decode attention intermediate arrays shapes routine used by this attention module.
**CN:** 实现该注意力模块使用的 decode attention intermediate arrays shapes 例程。

### Lines 105-154: function decode_attention_wave
```python
def decode_attention_wave(
    q,
    k_buffer,
    v_buffer,
    o,
    b_req_idx,
    req_to_token,
    attn_logits,
    attn_logits_max,
    num_kv_splits,
    max_kv_splits,
    sm_scale,
    logit_cap,
):
    num_seqs, num_query_heads, head_size = q.shape
    _, num_kv_heads, _ = k_buffer.shape
    _, _, head_size_kv = v_buffer.shape
    block_size = 32
    shape = paged_decode_attention_shape(
        num_query_heads,
        num_kv_heads,
        head_size,
        head_size_kv,
        block_size,
        num_seqs,
    )

    phase_0, phase_1 = get_wave_kernel(
        shape, max_kv_splits, q.dtype, o.dtype, logit_cap
    )

    mb_qk = phase_0(
# ... omitted 6 lines ...
        attn_logits_max,
    )
    if dump_generated_mlir:
        filename = f"wave_decode_attention_phase0_{'x'.join(map(str, shape))}.mlir"
        with open(filename, "w") as f:
            f.write(mb_qk.module_op.get_asm())

    mb_sv = phase_1(attn_logits, attn_logits_max, b_req_idx, o)
    if dump_generated_mlir:
        filename = f"wave_decode_attention_phase1_{'x'.join(map(str, shape))}.mlir"
        with open(filename, "w") as f:
            f.write(mb_sv.module_op.get_asm())
```
**EN:** Implements the decode attention wave routine used by this attention module.
**CN:** 实现该注意力模块使用的 decode attention wave 例程。

### Lines 157-184: function decode_attention_fwd
```python
def decode_attention_fwd(
    q,
    k_buffer,
    v_buffer,
    o,
    b_req_idx,
    req_to_token,
    attn_logits,
    attn_logits_max,
    num_kv_splits,
    max_kv_splits,
    sm_scale,
    logit_cap=0.0,
):
    decode_attention_wave(
        q,
        k_buffer,
        v_buffer,
        o,
        b_req_idx,
        req_to_token,
        attn_logits,
        attn_logits_max,
        num_kv_splits,
        max_kv_splits,
        sm_scale,
        logit_cap,
    )
```
**EN:** Implements the decode attention fwd routine used by this attention module.
**CN:** 实现该注意力模块使用的 decode attention fwd 例程。

## Key Concepts / 关键概念
- **EN:** Paged attention layouts / **CN:** 分页注意力布局
- **EN:** Quantized cache and layout handling / **CN:** 量化缓存与布局处理

## Dependencies / 依赖关系
- `functools`
- `logging`
- `wave_lang.kernel.lang.global_symbols.*`
- `wave_lang.kernel.wave.compile.WaveCompileOptions`
- `wave_lang.kernel.wave.compile.wave_compile`
- `wave_lang.kernel.wave.constraints.GenericDot`
- `wave_lang.kernel.wave.constraints.MMAOperand`
- `wave_lang.kernel.wave.constraints.MMAType`
- `wave_lang.kernel.wave.templates.paged_decode_attention.get_paged_decode_attention_kernels`
- `wave_lang.kernel.wave.templates.paged_decode_attention.get_paged_decode_intermediate_arrays_shapes`
- `wave_lang.kernel.wave.templates.paged_decode_attention.paged_decode_attention_shape`
- `wave_lang.kernel.wave.utils.general_utils.get_default_scheduling_params`
- `wave_lang.kernel.wave.utils.run_utils.set_default_run_config`
- `os`
