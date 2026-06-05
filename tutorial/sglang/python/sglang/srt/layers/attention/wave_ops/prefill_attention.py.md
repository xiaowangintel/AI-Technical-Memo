# prefill_attention.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/wave_ops/prefill_attention.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module collects prefill attention helper functions used to prepare, validate, or dispatch attention operations in SGLang. / 该模块汇集 prefill attention 辅助函数，用于在 SGLang 中准备、校验或分发注意力操作。
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
import math
import os

from wave_lang.kernel.lang.global_symbols import *
from wave_lang.kernel.wave.compile import WaveCompileOptions, wave_compile
from wave_lang.kernel.wave.constraints import MMAType
from wave_lang.kernel.wave.templates.attention_common import AttentionShape
from wave_lang.kernel.wave.templates.prefill_attention import (
    get_prefill_attention_kernel,
)
from wave_lang.kernel.wave.utils.general_utils import get_default_scheduling_params
from wave_lang.kernel.wave.utils.run_utils import set_default_run_config
```
**EN:** Imports the external and internal dependencies consumed by the code that follows.
**CN:** 导入后续代码所依赖的外部与内部模块。

### Lines 19-19: module constants
```python
dump_generated_mlir = int(os.environ.get("WAVE_DUMP_MLIR", 0))
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 22-79: function prefill_attention_wave
```python
def prefill_attention_wave(
    q, k, v, o, b_start_loc, b_seq_len, max_seq_len, is_causal=True
):

    shape = AttentionShape(
        num_query_heads=q.shape[1],
        num_kv_heads=k.shape[1],
        head_size=q.shape[2],
        head_size_kv=k.shape[2],
        num_seqs=b_seq_len.shape[0],
        max_seq_len=max_seq_len,
        total_seq_len=q.shape[0],
    )

    assert shape.num_query_heads % shape.num_kv_heads == 0

    output_shape = (shape.total_seq_len, shape.num_query_heads, shape.head_size_kv)
    # Run the wave kernel.
    mfma_variant = (MMAType.F32_16x16x16_F16, MMAType.F32_16x16x16_F16)
    prefill, hyperparams = get_prefill_attention_kernel(
        shape,
        mfma_variant,
        q.shape,
        k.shape,
        v.shape,
        output_shape,
        input_dtype=q.dtype,
        output_dtype=o.dtype,
        size_dtype=b_seq_len.dtype,
    )

    hyperparams.update(get_default_scheduling_params())
# ... omitted 14 lines ...
        q * dk_sqrt * log2e,
        k,
        v,
        b_start_loc,
        b_seq_len,
        o,
    )
    if dump_generated_mlir:
        shape_list = [q.shape[0], q.shape[1], k.shape[1], q.shape[2], k.shape[2]]
        filename = f"wave_prefill_attention_{'x'.join(map(str, shape_list))}.mlir"
        with open(filename, "w") as f:
            f.write(mb.module_op.get_asm())
```
**EN:** Implements the prefill attention wave routine used by this attention module.
**CN:** 实现该注意力模块使用的 prefill attention wave 例程。

## Key Concepts / 关键概念
- **EN:** Paged attention layouts / **CN:** 分页注意力布局

## Dependencies / 依赖关系
- `math`
- `os`
- `wave_lang.kernel.lang.global_symbols.*`
- `wave_lang.kernel.wave.compile.WaveCompileOptions`
- `wave_lang.kernel.wave.compile.wave_compile`
- `wave_lang.kernel.wave.constraints.MMAType`
- `wave_lang.kernel.wave.templates.attention_common.AttentionShape`
- `wave_lang.kernel.wave.templates.prefill_attention.get_prefill_attention_kernel`
- `wave_lang.kernel.wave.utils.general_utils.get_default_scheduling_params`
- `wave_lang.kernel.wave.utils.run_utils.set_default_run_config`
