# test_per_token_group_quant_8bit.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/tests/test_per_token_group_quant_8bit.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "import itertools". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“import itertools”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-112: Imports and module setup
```python
import itertools
import sys

import pytest
import torch

from sglang.srt.utils import is_hip

_is_hip = is_hip()
fp8_type_ = torch.float8_e4m3fnuz if _is_hip else torch.float8_e4m3fn

from sgl_kernel.test_utils import (
    assert_all_close_or_tiny_diff,
    create_per_token_group_quant_test_data,
)

from sglang.jit_kernel.per_token_group_quant_8bit import (
    per_token_group_quant_8bit as sglang_per_token_group_quant_8bit,
)
from sglang.srt.layers.quantization.fp8_kernel import (
    create_per_token_group_quant_fp8_output_scale,
)
from sglang.srt.layers.quantization.fp8_kernel import (
    per_token_group_quant_8bit as triton_per_token_group_quant_8bit,
)
from sglang.test.ci.ci_register import register_cuda_ci

register_cuda_ci(est_time=16, suite="base-b-kernel-unit-1-gpu-large")
register_cuda_ci(est_time=120, suite="nightly-kernel-1-gpu", nightly=True)

configs = list(
    itertools.product(
        [1, 4, 16, 64, 127, 128, 512, 1024, 4096, 8192],  # num_tokens
        [128, 256, 384, 512, 1024, 1536, 1664, 2048, 4096, 7168, 16384],  # hidden_dim
        [16, 32, 64, 128],  # group_size
        [None],  # num_ranks
        [fp8_type_],  # dtype
        [
            dict(
                column_major_scales=False,
                scale_tma_aligned=False,
                scale_ue8m0=False,
                fuse_silu_and_mul=False,
                masked_layout_mode=None,
            ),
            dict(
                column_major_scales=True,
                scale_tma_aligned=False,
                scale_ue8m0=False,
                fuse_silu_and_mul=False,
                masked_layout_mode=None,
            ),
            dict(
                column_major_scales=True,
                scale_tma_aligned=True,
                scale_ue8m0=False,
                fuse_silu_and_mul=False,
                masked_layout_mode=None,
            ),
            dict(
                column_major_scales=True,
                scale_tma_aligned=True,
                scale_ue8m0=True,
                fuse_silu_and_mul=False,
                masked_layout_mode=None,
            ),
        ],
    )
) + list(
    itertools.product(
        [1, 4, 1 * 8, 4 * 8, 64 * 8, 256 * 8, 768 * 8],
        [2048],
        [128],
        [8, 16, 32, 48],
        [fp8_type_],
        [
            dict(
                column_major_scales=True,
                scale_tma_aligned=True,
                scale_ue8m0=True,
# ...
```
**EN:** This block imports dependencies and prepares names used later in the module. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 113-210: Function `test_per_token_group_quant_with_column_major`
```python
def test_per_token_group_quant_with_column_major(
    num_tokens,
    hidden_dim,
    group_size,
    num_ranks,
    dst_dtype,
    flags,
):
    arch_major, _ = torch.cuda.get_device_capability(torch.cuda.current_device())
    if flags["scale_ue8m0"] and (arch_major <= 9):
        pytest.skip("Only Blackwell need ue8m0 fusion")
        return

    if (flags["scale_ue8m0"] and (group_size != 128)) or (
        (dst_dtype == torch.int8) and flags["column_major_scales"]
    ):
        pytest.skip()
        return

    x, masked_m = create_per_token_group_quant_test_data(
        num_tokens=num_tokens, hidden_dim=hidden_dim, num_ranks=num_ranks, flags=flags
    )

    execute_kwargs = dict(
        x=x,
        masked_m=masked_m,
        group_size=group_size,
        eps=1e-10,
        dst_dtype=dst_dtype,
        **{k: v for k, v in flags.items() if k not in ["masked_layout_mode"]},
    )

    def _postprocess(x_q, x_s):
        if masked_m is not None:
            print(f"Mask tokens after {masked_m} to be zero")
            for i in range(len(masked_m)):
                x_q[i, masked_m[i] :, :] = 0
                x_s[i, masked_m[i] :, :] = 0
        return x_q, x_s

    x_q_triton, x_s_triton = _postprocess(
        *triton_per_token_group_quant_8bit(**execute_kwargs)
    )

    fuse_silu_and_mul = False
    out_shape = (*x.shape[:-1], x.shape[-1] // (2 if fuse_silu_and_mul else 1))

    fp8_dtype = torch.float8_e4m3fn
    fp8_max = torch.finfo(fp8_dtype).max
    fp8_min = -fp8_max
    x_q = torch.empty(out_shape, device=x.device, dtype=fp8_dtype)
    x_s = create_per_token_group_quant_fp8_output_scale(
        x_shape=out_shape,
        device=x.device,
        group_size=group_size,
        column_major_scales=False,
        scale_tma_aligned=False,
        scale_ue8m0=False,
    )

    execute_kwargs = dict(
        input=x,
        output_q=x_q,
        output_s=x_s,
        group_size=group_size,
        eps=1e-10,
        fp8_max=fp8_max,
        fp8_min=fp8_min,
    )
    x_q_sglang, x_s_sglang = _postprocess(
        *sglang_per_token_group_quant_8bit(**execute_kwargs)
    )

    try:
        assert_all_close_or_tiny_diff(x_q_triton, x_q_sglang)
        torch.testing.assert_close(
            x_s_triton.contiguous(),
            x_s_sglang.contiguous(),
            rtol=1e-3,
            atol=1e-5,
# ...
```
**EN:** This block defines `test_per_token_group_quant_with_column_major`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_per_token_group_quant_with_column_major`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核
- **Behavior validation**: How expected outputs or invariants are checked / 如何检查预期输出或不变量

## Dependencies / 依赖关系
- `itertools`
- `sys`
- `pytest`
- `torch`
- `sglang.srt.utils -> is_hip`
- `sgl_kernel.test_utils -> (`
- `sglang.jit_kernel.per_token_group_quant_8bit -> (`
- `sglang.srt.layers.quantization.fp8_kernel -> (`
- `sglang.test.ci.ci_register -> register_cuda_ci`
