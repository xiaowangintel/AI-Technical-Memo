# test_cutlass_mla.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/tests/test_cutlass_mla.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This pytest module validates `cutlass MLA` behavior, numerical correctness, and edge conditions for the sgl-kernel stack. / 该 pytest 模块验证 `cutlass MLA` 在 sgl-kernel 栈中的行为、数值正确性与边界条件。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: Imports and module setup
````python
import sys

import pytest
import torch
import torch.nn.functional as F
from sgl_kernel import cutlass_mla_decode, cutlass_mla_get_workspace_size
from torch import Tensor
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 8-9: Comments and local context
````python

# Disable tests on SM103 until the accuracy issues are fixed.
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 10-14: Conditional logic
````python
if torch.cuda.get_device_capability() != (10, 0):
    pytest.skip(
        reason="Cutlass MLA Requires compute capability of 10.",
        allow_module_level=True,
    )
````
**EN:** This section selects different behavior according to runtime conditions such as platform, environment, or feature availability.
**CN:** 该部分根据平台、环境或功能可用性等运行时条件选择不同的行为。

### Lines 17-38: `ref_mla` definition
````python
def ref_mla(
    out: Tensor,  # (bs, num_heads, v_head_dim)
    query: Tensor,  # (bs, num_heads, head_dim)
    kv_cache: Tensor,  # (num_blocks, block_size, head_dim)
    scale: float,
    block_tables: Tensor,  # (bs, max_num_blocks)
    seq_lens: Tensor,  # (bs,)
):
    bs, num_heads, v_head_dim = out.shape
    head_dim = query.shape[2]

    for i in range(bs):
        # gather and flatten KV-cache
        kv = kv_cache[block_tables[i]]  # (max_num_blocks, block_size, head_dim)
        kv = kv.view(1, -1, head_dim)[:, : seq_lens[i]]  # (1, seq_len, head_dim)
        v = kv[:, :, :v_head_dim]

        q = query[i].view(num_heads, 1, head_dim)
        o = F.scaled_dot_product_attention(q, kv, v, scale=scale, enable_gqa=True)
        out[i] = o.view(num_heads, v_head_dim)

    return out
````
**EN:** This section defines `ref_mla` and implements the core logic associated with ref MLA.
**CN:** 该部分定义 `ref_mla`，并实现与 ref MLA 相关的核心逻辑。

### Lines 41-102: `test_cutlass_mla_decode` definition
````python
@pytest.mark.parametrize("dtype", [torch.bfloat16, torch.float16])
@pytest.mark.parametrize("mean_seq_len", [128, 1024, 4096])
@pytest.mark.parametrize("bs", [1, 2, 4])
@pytest.mark.parametrize("varlen", [False, True])
@pytest.mark.parametrize("block_size", [1, 16, 64, 128])
@pytest.mark.parametrize("num_heads", [16, 32, 64, 128])
@pytest.mark.parametrize("num_kv_splits", [-1, 1])
def test_cutlass_mla_decode(
    dtype: torch.dtype,
    mean_seq_len: int,
    bs: int,
    varlen: bool,
    block_size: int,
    num_heads: int,
    num_kv_splits: int,
):
    torch.set_default_dtype(dtype)
    torch.set_default_device("cuda")
    torch.manual_seed(42)

    d = 576
    h_q = num_heads
    dv = 512

    q_nope_dim = 128
    q_pe_dim = 64
    scale = (q_nope_dim + q_pe_dim) ** (-0.5)
    if varlen:
        seq_lens = torch.empty(bs).normal_(mean_seq_len, mean_seq_len / 2)
        seq_lens = seq_lens.clip(2).to(torch.int32)
    else:
        seq_lens = torch.full((bs,), mean_seq_len, dtype=torch.int32)
    max_seq_len = seq_lens.max().item()
    block_num = (max_seq_len + block_size - 1) // block_size

    # Pad block_num so that small blocks can be packed into full 128-sized CUTLASS tiles.
    # One 128-wide tile can hold (128 // block_size) small blocks.
    pack_factor = 128 // block_size
    block_num = ((block_num + pack_factor - 1) // pack_factor) * pack_factor

    # Lager q values to detect split kv error
    q = torch.randn(bs, h_q, d) * 100.0
    block_table = torch.randint(0, bs * block_num, (bs, block_num), dtype=torch.int32)

    kv_cache = torch.randn(block_table.numel(), block_size, d)

    workspace_size = cutlass_mla_get_workspace_size(
        block_num * block_size, bs, num_kv_splits=num_kv_splits
    )
    workspace = torch.empty(workspace_size, device="cuda", dtype=torch.uint8)

    q_nope = torch.empty((h_q, bs, dv)).transpose(0, 1)
    q_nope.copy_(q[:, :, :dv])
    q_pe = q[:, :, dv:].clone()

    out_ref = q.new_zeros(bs, h_q, dv)
    ref_mla(out_ref, q, kv_cache, scale, block_table, seq_lens)
    out = cutlass_mla_decode(
        q_nope, q_pe, kv_cache, seq_lens, block_table, workspace, scale, num_kv_splits
    )

    torch.testing.assert_close(out, out_ref, atol=1e-2, rtol=1e-2)
````
**EN:** This section defines the test `test_cutlass_mla_decode`. It sets up inputs, runs the target path, and checks the expected result. Parameterized decorators expand the coverage across multiple input cases.
**CN:** 该部分定义测试 `test_cutlass_mla_decode`。它会准备输入、执行目标路径，并检查预期结果。 参数化装饰器会把覆盖范围扩展到多组输入场景。

### Lines 105-106: Command-line entry point
````python
if __name__ == "__main__":
    sys.exit(pytest.main([__file__]))
````
**EN:** This block is the script entry point. It triggers the top-level execution path when the file is run directly.
**CN:** 该部分是脚本入口；当文件被直接执行时，它会触发顶层执行路径。

## Key Concepts / 关键概念
- **Role / 角色**: Validation module / 验证模块
- **Primary symbols / 主要符号**: `ref_mla`, `test_cutlass_mla_decode`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量, pytest parameterization / pytest 参数化
- **Execution style / 执行方式**: Runnable script as well as importable module / 既可直接运行也可作为可导入模块

## Dependencies / 依赖关系
- **Internal / 内部**: `sgl_kernel`
- **External / 外部**: `pytest`, `sys`, `torch`, `torch.nn.functional`
- **Runtime hooks / 运行时钩子**: direct script execution path / 直接脚本执行路径
