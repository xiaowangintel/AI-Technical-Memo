# test_ngram_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/tests/speculative/test_ngram_utils.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This pytest module validates `ngram utilities` behavior, numerical correctness, and edge conditions for the sgl-kernel stack. / 该 pytest 模块验证 `ngram utilities` 在 sgl-kernel 栈中的行为、数值正确性与边界条件。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: Imports and module setup
````python
import sys

import pytest
import torch
import torch.nn.functional as F
from sgl_kernel import reconstruct_indices_from_tree_mask
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 9-73: `test_reconstruct_indices_from_tree_mask` definition
````python
def test_reconstruct_indices_from_tree_mask():
    bs = 1
    num_branch_token = 4
    seq_lens = torch.tensor([12], device="cuda", dtype=torch.int64)

    retrive_index = torch.full(
        (bs, num_branch_token), -1, device="cuda", dtype=torch.int64
    )
    retrive_next_token = torch.full(
        (bs, num_branch_token), -1, device="cuda", dtype=torch.int64
    )
    retrive_next_sibling = torch.full(
        (bs, num_branch_token), -1, device="cuda", dtype=torch.int64
    )
    positions = torch.empty((bs * num_branch_token), device="cuda", dtype=torch.int64)

    tree_mask = torch.tensor(
        [
            1,
            0,
            0,
            0,
            1,
            1,
            0,
            0,
            1,
            0,
            1,
            0,
            1,
            0,
            1,
            1,
        ],
        device="cuda",
        dtype=torch.int32,
    ).to(torch.bool)

    reconstruct_indices_from_tree_mask(
        tree_mask,
        seq_lens,
        positions,  # mutable
        retrive_index,  # mutable
        retrive_next_token,  # mutable
        retrive_next_sibling,  # mutable
        bs,
        num_branch_token,
    )
    # print(f"debug: \n\n{tree_mask=}, {retrive_index=}, {retrive_next_token=}, {retrive_next_sibling=}, {positions=}\n\n")
    assert retrive_index.tolist() == [
        [0, 1, 2, 3],
    ], f"{retrive_index=}"
    assert retrive_next_token.tolist() == [
        [1, -1, 3, -1],
    ], f"{retrive_next_token=}"
    assert retrive_next_sibling.tolist() == [
        [-1, 2, -1, -1],
    ], f"{retrive_next_sibling=}"
    assert positions.tolist() == [
        12,
        13,
        13,
        14,
    ], f"{positions=}"
````
**EN:** This section defines the test `test_reconstruct_indices_from_tree_mask`. It sets up inputs, runs the target path, and checks the expected result. It also performs explicit assertions or shape checks before continuing.
**CN:** 该部分定义测试 `test_reconstruct_indices_from_tree_mask`。它会准备输入、执行目标路径，并检查预期结果。 它还会在继续执行前进行显式断言或形状检查。

### Lines 76-78: Command-line entry point
````python
if __name__ == "__main__":
    test_reconstruct_indices_from_tree_mask()
    sys.exit(pytest.main([__file__]))
````
**EN:** This block is the script entry point. It triggers the top-level execution path when the file is run directly.
**CN:** 该部分是脚本入口；当文件被直接执行时，它会触发顶层执行路径。

## Key Concepts / 关键概念
- **Role / 角色**: Validation module / 验证模块
- **Primary symbols / 主要符号**: `test_reconstruct_indices_from_tree_mask`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量, pytest parameterization / pytest 参数化
- **Execution style / 执行方式**: Runnable script as well as importable module / 既可直接运行也可作为可导入模块

## Dependencies / 依赖关系
- **Internal / 内部**: `sgl_kernel`
- **External / 外部**: `pytest`, `sys`, `torch`, `torch.nn.functional`
- **Runtime hooks / 运行时钩子**: direct script execution path / 直接脚本执行路径
