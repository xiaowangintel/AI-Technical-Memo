# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/tests/utils.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This pytest module validates `utilities` behavior, numerical correctness, and edge conditions for the sgl-kernel stack. / 该 pytest 模块验证 `utilities` 在 sgl-kernel 栈中的行为、数值正确性与边界条件。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: Imports and module setup
````python
import torch
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 4-5: `is_sm10x` definition
````python
def is_sm10x():
    return torch.cuda.get_device_capability() >= (10, 0)
````
**EN:** This section defines `is_sm10x` and implements the core logic associated with is sm10x.
**CN:** 该部分定义 `is_sm10x`，并实现与 is sm10x 相关的核心逻辑。

### Lines 8-9: `is_hopper` definition
````python
def is_hopper():
    return torch.cuda.get_device_capability() == (9, 0)
````
**EN:** This section defines `is_hopper` and implements the core logic associated with is hopper.
**CN:** 该部分定义 `is_hopper`，并实现与 is hopper 相关的核心逻辑。

## Key Concepts / 关键概念
- **Role / 角色**: Validation module / 验证模块
- **Primary symbols / 主要符号**: `is_sm10x`, `is_hopper`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量

## Dependencies / 依赖关系
- **Internal / 内部**: None obvious / 无明显内部依赖
- **External / 外部**: `torch`
