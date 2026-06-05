# gemm_batched.py — Code Analysis / 代码分析

## Source / 来源

- Path / 路径: `test/python/cutlass/gemm/gemm_batched.py`
- **EN:** High-level correctness tests for batched GEMM that compare CUTLASS execution against a PyTorch reference and explicitly skip Intel Xe devices where this path is unsupported.
- **CN:** 这是一个面向高层的 batched GEMM 正确性测试文件：它用 CUTLASS 执行结果对比 PyTorch 参考实现，并且在当前不支持该路径的 Intel Xe 设备上直接跳过。

## Line-by-Line Analysis / 逐行分析

### Lines 37-50

```python
from functools import partial
import logging
from math import prod
import unittest

import cutlass_cppgen
from cutlass_cppgen.backend.utils.device import device_cc
import torch

from utils import LayoutCombination

cutlass_cppgen.set_log_level(logging.WARNING)

torch.manual_seed(2023)
```
**EN:** This setup block imports the CUTLASS Python bindings, device capability helper, and PyTorch. `partial` and `LayoutCombination` are imported but not used later in this file. The module reduces logging noise to warnings and fixes the PyTorch RNG seed so randomly generated tensors are reproducible.

**CN:** 这一段完成基础环境设置：导入 CUTLASS Python 绑定、设备计算能力查询函数以及 PyTorch。`partial` 和 `LayoutCombination` 在本文件后续并未实际使用。模块把日志级别降到 warning，并固定 PyTorch 随机种子，使随机张量可复现。

### Lines 53-79

```python
def pytorch_reference(A, B, C, alpha, beta):
    # Get the batch count. Assume that any of A, B, and C
    # with a batch dimension ahve matching batch count. Thus,
    # we break out of the loop once we have found the first
    # tensor containing a batch dimension.
    batch_count = (1,)
    for tensor in [A, B, C]:
        if len(tensor.shape) > 2:
            batch_count = tensor.shape[:-2]
            break

    int_batch_count = prod(batch_count)

    def add_batch(tensor):
        if len(tensor.shape) == 2:
            return tensor.unsqueeze(0).repeat(int_batch_count, 1, 1)
        else:
            return tensor.reshape(-1, tensor.size(-2), tensor.size(-1))

    # Reshape tensors to have batch dimension
    A = add_batch(A)
    B = add_batch(B)
    C = add_batch(C)

    ret = (torch.bmm(A, B) * alpha) + (C * beta)
    reshape_vals = batch_count + C.shape[-2:]
    return ret.reshape(*reshape_vals)
```
**EN:** `pytorch_reference` builds the expected batched result. It scans `A`, `B`, and `C` to find the first operand that already has batch dimensions, flattens that batch shape with `prod`, broadcasts any plain 2-D operand with `unsqueeze().repeat()`, runs `torch.bmm`, applies the GEMM epilogue `alpha * (A @ B) + beta * C`, and finally reshapes the answer back to the original batch rank.

**CN:** `pytorch_reference` 用来构造期望结果。它会依次检查 `A`、`B`、`C`，找到第一个已经带批次维度的操作数；然后用 `prod` 把批次形状展平，对普通二维张量使用 `unsqueeze().repeat()` 做广播，再调用 `torch.bmm`，并加上 GEMM 的后处理 `alpha * (A @ B) + beta * C`，最后把结果重新恢复为原始批次维度。

### Lines 82-89

```python
def initialize(rows, cols, batch):
    tensor = torch.randint(-3, 3, size=(rows*cols*prod(batch),), device='cuda').half()
    if len(batch) > 0 and prod(batch) > 1:
        reshape_vals = batch + (rows, cols)
        return tensor.reshape(*reshape_vals)
    else:
        return tensor.reshape(rows, cols)
```
**EN:** `initialize` creates random FP16 data on CUDA in a flat buffer and reshapes it into either a matrix or a batched tensor. Because the code only keeps batch dimensions when `prod(batch) > 1`, passing `(1,)` intentionally produces an ordinary 2-D operand instead of a rank-3 tensor.

**CN:** `initialize` 在 CUDA 上先生成扁平的随机 FP16 数据，再把它重塑为矩阵或带批次维度的张量。由于代码只有在 `prod(batch) > 1` 时才保留批次维度，因此传入 `(1,)` 会有意生成普通二维操作数，而不是三维张量。

### Lines 91-109

```python
@unittest.skipIf(device_cc() >= 12 and device_cc() <= 20, "Batched GEMM test not supported on Xe")
class GemmF16Batched(unittest.TestCase):
    def run_batched(self, batch_count: tuple, batch_A: bool, batch_B: bool, batch_C: bool):
        M = 512
        N = 256
        K = 128
        alpha = 1.
        beta = 2.

        A = initialize(M, K, batch_count if batch_A else (1,))
        B = initialize(K, N, batch_count if batch_B else (1,))
        C = initialize(M, N, batch_count if batch_C else (1,))
        D = initialize(M, N, batch_count)

        plan = cutlass_cppgen.op.Gemm(A=A, B=B, C=C, D=D, element_accumulator=cutlass_cppgen.DataType.f32)
        plan.run(A, B, C, D, alpha, beta)
        reference = pytorch_reference(A, B, C, alpha, beta)
        assert reference.equal(D)
```
**EN:** The test class is skipped for compute capabilities in the Xe range. `run_batched` fixes one GEMM problem size, chooses whether `A`, `B`, and `C` should be batched or broadcastable according to the boolean flags, allocates `D` with the full requested batch shape, constructs a CUTLASS GEMM plan with FP32 accumulation for FP16 inputs, executes it, and checks the result with exact tensor equality.

**CN:** 这个测试类会在 Xe 范围的计算能力上被跳过。`run_batched` 固定了一组 GEMM 尺寸，然后根据布尔开关决定 `A`、`B`、`C` 是真正分批还是依靠广播；`D` 总是按完整批次形状分配。随后它构造一个对 FP16 输入使用 FP32 累加的 CUTLASS GEMM plan，执行后再用严格的张量相等比较校验结果。

### Lines 110-132

```python
    def test_batched_ABC(self):
        self.run_batched((3,), True, True, True)
        self.run_batched((2, 3), True, True, True)

    def test_batched_AB(self):
        self.run_batched((3,), True, True, False)
        self.run_batched((2, 3), True, True, False)

    def test_batched_AC(self):
        self.run_batched((3,), True, False, True)
        self.run_batched((2, 3), True, False, True)

    def test_batched_BC(self):
        self.run_batched((3,), False, True, True)
        self.run_batched((2, 3), False, True, True)

    def test_batched_A(self):
        self.run_batched((3,), True, False, False)
        self.run_batched((2, 3), True, False, False)

    def test_batched_B(self):
        self.run_batched((3,), False, True, False)
        self.run_batched((2, 3), False, True, False)
```
**EN:** Each unit test covers one batching pattern: all of `A/B/C`, only `A/B`, only `A/C`, only `B/C`, only `A`, or only `B` are batched. Every pattern is exercised with both a one-dimensional batch `(3,)` and a two-dimensional batch `(2, 3)`, so the file verifies both simple batching and multi-axis batch flattening/restoration.

**CN:** 每个单元测试对应一种批处理模式：`A/B/C` 全部分批、仅 `A/B` 分批、仅 `A/C` 分批、仅 `B/C` 分批、仅 `A` 分批或仅 `B` 分批。每种模式都同时测试一维批次 `(3,)` 和二维批次 `(2, 3)`，因此既覆盖简单批处理，也覆盖多轴批次的展平与恢复逻辑。

### Lines 134-135

```python
if __name__ == '__main__':
    unittest.main()
```
**EN:** The standard `unittest.main()` entry point allows the module to be run directly as a test script.

**CN:** 标准的 `unittest.main()` 入口让该模块可以作为独立测试脚本直接运行。

## Key Concepts / 关键概念

- **EN:** Batched GEMM broadcasting: individual operands may be full batched tensors or shared matrices that are expanded across batches.
  **CN:** 批处理 GEMM 的广播：某些操作数可以是真正的批张量，也可以是被扩展到所有批次上的共享矩阵。
- **EN:** Reference checking is done with `torch.bmm` plus the same `alpha`/`beta` epilogue used by GEMM.
  **CN:** 参考结果通过 `torch.bmm` 再叠加与 GEMM 相同的 `alpha`/`beta` 后处理来计算。
- **EN:** The CUTLASS plan uses FP16 inputs but FP32 accumulation, which is a common mixed-precision configuration.
  **CN:** CUTLASS plan 使用 FP16 输入、FP32 累加，这是常见的混合精度配置。
- **EN:** The file is a high-level behavior test rather than a kernel-enumeration file because it manually creates tensors and calls `plan.run`.
  **CN:** 这个文件属于高层行为测试，而不是内核枚举式测试，因为它手动构造张量并直接调用 `plan.run`。

## Dependencies / 依赖关系

- **EN:** External: `cutlass_cppgen` for `op.Gemm` construction and kernel execution.
  **CN:** 外部依赖：`cutlass_cppgen`，用于构造 `op.Gemm` 并执行内核。
- **EN:** External: `torch` for CUDA tensor creation, `torch.bmm`, reshaping, and exact-result comparison.
  **CN:** 外部依赖：`torch`，用于创建 CUDA 张量、调用 `torch.bmm`、重塑张量以及进行精确结果比较。
- **EN:** Internal/external helper: `device_cc` from `cutlass_cppgen.backend.utils.device` gates the test on supported hardware.
  **CN:** 内部/外部辅助：来自 `cutlass_cppgen.backend.utils.device` 的 `device_cc` 用于按硬件能力决定是否跳过测试。
- **EN:** Standard library: `math.prod`, `logging`, and `unittest` support batch-shape math, logging control, and test execution.
  **CN:** 标准库依赖：`math.prod`、`logging` 和 `unittest` 分别用于批次形状计算、日志控制与测试执行。
- **EN:** Notable imports: `partial` and `LayoutCombination` are present but unused in this specific file.
  **CN:** 额外说明：`partial` 和 `LayoutCombination` 在这个文件中被导入了，但没有真正使用。
