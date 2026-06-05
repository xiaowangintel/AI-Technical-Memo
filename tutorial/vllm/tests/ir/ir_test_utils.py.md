# ir_test_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/ir/ir_test_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Shared test utilities for vLLM IR op correctness tests. / 该文件主要围绕 Ir Test Utils 组织测试，并验证关键行为与边界场景。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-18)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
"""
Shared test utilities for vLLM IR op correctness tests.
"""

import torch

from vllm.ir.op import IrOp

NUM_TOKENS = [1, 8, 17, 32, 512, 2048]
COMMON_HIDDEN_SIZES = [
    2048,  # Llama 3.2 1B, Qwen 3 MoE 30B-A3B, Gemma 3n
    4096,  # Llama 3 8B, Qwen 3 8B
    5120,  # Llama 4 Scout 17B-16E
    7168,  # DeepSeek V3
    8192,  # Llama 3 70B
]
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `torch`, `vllm.ir.op`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Helper: clone_args (lines 21-22)
```python
def clone_args(args: tuple) -> tuple:
    return tuple(a.clone() if isinstance(a, torch.Tensor) else a for a in args)
```
**EN:** Implements a reusable helper for Clone Args, reducing duplication across related tests. It coordinates operations such as `tuple`, `isinstance`, `a.clone`.
**CN:** 该辅助函数为 Clone Args 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `tuple`, `isinstance`, `a.clone` 等操作。

### Helper: supported_providers (lines 25-28)
```python
def supported_providers(op: IrOp) -> list[str]:
    return [
        name for name, impl in op.impls.items() if name != "native" and impl.supported
    ]
```
**EN:** Implements a reusable helper for Supported Providers, reducing duplication across related tests. It coordinates operations such as `op.impls.items`.
**CN:** 该辅助函数为 Supported Providers 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `op.impls.items` 等操作。

### Helper: assert_close (lines 31-46)
```python
def assert_close(op: IrOp, actual, expected):
    if isinstance(actual, torch.Tensor):
        tol = op.get_tolerance(actual.dtype)
        try:
            torch.testing.assert_close(actual, expected, **tol)
        except AssertionError as e:
            raise AssertionError(
                f"{e}\n\nTo adjust tolerance, use:\n"
                f"  ir.ops.{op.name}.override_tolerance("
                f"{actual.dtype}, atol=..., rtol=...)"
            ) from None
    elif isinstance(actual, (tuple, list)):
        for a, ex in zip(actual, expected):
            assert_close(op, a, ex)
    else:
        assert actual == expected
```
**EN:** Implements a reusable helper for Close, reducing duplication across related tests. It coordinates operations such as `isinstance`, `op.get_tolerance`, `torch.testing.assert_close`.
**CN:** 该辅助函数为 Close 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `isinstance`, `op.get_tolerance`, `torch.testing.assert_close` 等操作。

## Key Concepts / 关键概念
- **GPU execution / GPU 执行**
  - **EN:** The tests exercise CUDA-like execution paths, device placement, or tensor kernels.
  - **CN:** 这些测试覆盖 CUDA 类执行路径、设备放置或张量内核行为。
- **Assertions / 断言**
  - **EN:** The file relies on explicit assertions to pin down expected state transitions and outputs.
  - **CN:** 该文件通过显式断言固定预期的状态变化与输出结果。

## Dependencies / 依赖关系
- **Third-party / 第三方依赖**: `torch`
- **vLLM internal / vLLM 内部依赖**: `vllm.ir.op`
