# test_per_token_group_quant_8bit.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/tests/test_per_token_group_quant_8bit.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This pytest module validates `per token group quantization 8bit` behavior, numerical correctness, and edge conditions for the sgl-kernel stack. / 该 pytest 模块验证 `per token group quantization 8bit` 在 sgl-kernel 栈中的行为、数值正确性与边界条件。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: Imports and module setup
````python
import itertools
import os
import sys
import time
from pathlib import Path

import pytest
import torch
from sgl_kernel.test_utils import (
    assert_all_close_or_tiny_diff,
    create_per_token_group_quant_test_data,
)

from sglang.srt.layers.quantization.fp8_kernel import (
    per_token_group_quant_8bit as triton_per_token_group_quant_8bit,
)
from sglang.srt.layers.quantization.fp8_kernel import (
    sglang_per_token_group_quant_8bit,
)
from sglang.srt.utils import get_bool_env_var, is_hip
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 22-102: Constants and configuration
````python
_is_hip = is_hip()
fp8_type_ = torch.float8_e4m3fnuz if _is_hip else torch.float8_e4m3fn

configs = list(
    itertools.product(
        [1, 4, 16, 64, 127, 128, 512, 1024, 4096, 8192],  # num_tokens
        [128, 256, 384, 512, 1024, 1536, 1664, 2048, 4096, 7168, 16384],  # hidden_dim
        [16, 32, 64, 128],  # group_size
        [None],  # num_ranks
        [fp8_type_, torch.int8],  # dtype
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
        # TODO support more
        [2048],
        [128],
        [8, 16, 32, 48],
        [fp8_type_],
        [
            dict(
                column_major_scales=True,
                scale_tma_aligned=True,
                scale_ue8m0=True,
                fuse_silu_and_mul=True,
                masked_layout_mode=None,
            ),
            dict(
                column_major_scales=True,
                scale_tma_aligned=True,
                scale_ue8m0=True,
                fuse_silu_and_mul=True,
                masked_layout_mode="balanced",
            ),
            dict(
                column_major_scales=True,
                scale_tma_aligned=True,
                scale_ue8m0=True,
                fuse_silu_and_mul=True,
                masked_layout_mode="imbalanced",
            ),
            dict(
                column_major_scales=True,
                scale_tma_aligned=True,
                scale_ue8m0=True,
                fuse_silu_and_mul=True,
                masked_layout_mode="extreme",
            ),
        ],
    )
)
````
**EN:** This block defines shared constants or configuration values such as `_is_hip`, `fp8_type_`, `configs`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `_is_hip`, `fp8_type_`, `configs`），供后续函数或控制流程复用。

### Lines 105-182: `test_per_token_group_quant_with_column_major` definition
````python
@pytest.mark.parametrize(
    "num_tokens, hidden_dim, group_size, num_ranks, dst_dtype, flags", configs
)
def test_per_token_group_quant_with_column_major(
    num_tokens,
    hidden_dim,
    group_size,
    num_ranks,
    dst_dtype,
    flags,
):
    print(
        f"{num_tokens=} {hidden_dim=} {group_size=} {num_ranks=} {dst_dtype=} {flags=}"
    )

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

    # print("hack data!!!")
    # x = torch.full_like(x, fill_value=100)

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
    x_q_sglang, x_s_sglang = _postprocess(
        *sglang_per_token_group_quant_8bit(**execute_kwargs, enable_v2=True)
    )

    try:
        assert_all_close_or_tiny_diff(x_q_triton, x_q_sglang)
        torch.testing.assert_close(
            x_s_triton.contiguous(),
            x_s_sglang.contiguous(),
            rtol=1e-3,
            atol=1e-5,
            msg=lambda message: message + f" {x_s_triton=} {x_s_sglang=}",
        )
    except AssertionError:
        print(
            f"{x.shape=} {x_q_triton.shape=} {x_s_triton.shape=} {x_q_sglang.shape=} {x_s_sglang.shape=}"
        )
        print(f"{x=}")
        print(f"{masked_m=}")
        print(f"{x_q_triton=}")
        print(f"{x_s_triton=}")
        print(f"{x_q_sglang=}")
        print(f"{x_s_sglang=}")

        raise
````
**EN:** This section defines the test `test_per_token_group_quant_with_column_major`. It sets up inputs, runs the target path, and checks the expected result. Parameterized decorators expand the coverage across multiple input cases.
**CN:** 该部分定义测试 `test_per_token_group_quant_with_column_major`。它会准备输入、执行目标路径，并检查预期结果。 参数化装饰器会把覆盖范围扩展到多组输入场景。

### Lines 185-186: Command-line entry point
````python
if __name__ == "__main__":
    sys.exit(pytest.main([__file__]))
````
**EN:** This block is the script entry point. It triggers the top-level execution path when the file is run directly.
**CN:** 该部分是脚本入口；当文件被直接执行时，它会触发顶层执行路径。

## Key Concepts / 关键概念
- **Role / 角色**: Validation module / 验证模块
- **Primary symbols / 主要符号**: `test_per_token_group_quant_with_column_major`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量, pytest parameterization / pytest 参数化, Triton benchmarking / Triton 基准工具
- **Execution style / 执行方式**: Runnable script as well as importable module / 既可直接运行也可作为可导入模块

## Dependencies / 依赖关系
- **Internal / 内部**: `sgl_kernel.test_utils`, `sglang.srt.layers.quantization.fp8_kernel`, `sglang.srt.utils`
- **External / 外部**: `itertools`, `os`, `pathlib`, `pytest`, `sys`, `time`, `torch`
- **Runtime hooks / 运行时钩子**: direct script execution path / 直接脚本执行路径
