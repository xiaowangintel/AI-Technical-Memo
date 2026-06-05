# elementwise.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/layers/elementwise.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the runtime layer implementation. It centers on `MulAdd`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于运行时算子层。它围绕 `MulAdd` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-4: module setup and imports / 模块初始化与导入
```python
import torch

from sglang.jit_kernel.diffusion.triton.scale_shift import fuse_scale_shift_kernel
from sglang.multimodal_gen.runtime.layers.custom_op import CustomOp
```
**EN:** This block establishes the module context and imports `torch`, `sglang.jit_kernel.diffusion.triton.scale_shift`, and `sglang.multimodal_gen.runtime.layers.custom_op`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `torch`、`sglang.jit_kernel.diffusion.triton.scale_shift` 和 `sglang.multimodal_gen.runtime.layers.custom_op`。这些依赖为后续实现提供所需符号。

### Lines 7-13: `MulAdd` class overview / `MulAdd` 类概览
```python
class MulAdd(CustomOp):
    """
    Fuse elementwise mul and add
    Input: a, b, c, OptionalInt[k]
    Output: a * (k + b) + c
    """
```
**EN:** This block defines class `MulAdd`. Fuse elementwise mul and add Input: a, b, c, OptionalInt[k] Output: a * (k + b) + c It inherits from `CustomOp`.
**CN:** 该代码块定义了类 `MulAdd`。 它用于封装 mul add 相关行为。 它继承自 `CustomOp`。

### Lines 14-15: `__init__` implementation / `__init__` 实现
```python
    def __init__(self, prefix: str = ""):
        super().__init__()
```
**EN:** This block defines method `__init__` on `MulAdd`. It initializes the instance state. Key calls include `super.__init__`, and `super`. Parameters such as `prefix` drive the behavior in this section.
**CN:** 该代码块定义了 `MulAdd` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__` 和 `super`。 本段逻辑主要由 `prefix` 等参数驱动。

### Lines 17-30: `forward_native` implementation / `forward_native` 实现
```python
    def forward_native(
        self, a: torch.Tensor, b: torch.Tensor, c: torch.Tensor, k: int = 0
    ) -> torch.Tensor:
        # a.shape: [batch_size, seq_len, inner_dim]
        if b.dim() == 4:
            # b.shape: [batch_size, num_frames, 1, inner_dim]
            num_frames = b.shape[1]
            frame_seqlen = a.shape[1] // num_frames
            return c + (
                a.unflatten(dim=1, sizes=(num_frames, frame_seqlen)) * (k + b)
            ).flatten(1, 2)
        else:
            # b.shape: [batch_size, 1, inner_dim]
            return c + a * (k + b)
```
**EN:** This block defines method `forward_native` on `MulAdd`. It executes native. Key calls include `b.dim`, `flatten`, and `a.unflatten`. The implementation branches on conditions. Parameters such as `a`, `b`, `c`, and `k` drive the behavior in this section.
**CN:** 该代码块定义了 `MulAdd` 的方法 `forward_native`。 它用于执行前向计算native。 关键调用包括 `b.dim`、`flatten` 和 `a.unflatten`。 实现中包含条件分支。 本段逻辑主要由 `a`、`b`、`c` 和 `k` 等参数驱动。

### Lines 32-35: `forward_cuda` implementation / `forward_cuda` 实现
```python
    def forward_cuda(
        self, a: torch.Tensor, b: torch.Tensor, c: torch.Tensor, k: int = 0
    ):
        return fuse_scale_shift_kernel(a, b, c, scale_constant=k)
```
**EN:** This block defines method `forward_cuda` on `MulAdd`. It executes cuda. Key calls include `fuse_scale_shift_kernel`. Parameters such as `a`, `b`, `c`, and `k` drive the behavior in this section.
**CN:** 该代码块定义了 `MulAdd` 的方法 `forward_cuda`。 它用于执行前向计算cuda。 关键调用包括 `fuse_scale_shift_kernel`。 本段逻辑主要由 `a`、`b`、`c` 和 `k` 等参数驱动。

### Lines 37-40: `forward_xpu` implementation / `forward_xpu` 实现
```python
    def forward_xpu(
        self, a: torch.Tensor, b: torch.Tensor, c: torch.Tensor, k: int = 0
    ):
        return self.forward_native(a, b, c, k=k)
```
**EN:** This block defines method `forward_xpu` on `MulAdd`. It executes xpu. Key calls include `self.forward_native`. Parameters such as `a`, `b`, `c`, and `k` drive the behavior in this section.
**CN:** 该代码块定义了 `MulAdd` 的方法 `forward_xpu`。 它用于执行前向计算xpu。 关键调用包括 `self.forward_native`。 本段逻辑主要由 `a`、`b`、`c` 和 `k` 等参数驱动。

### Lines 42-46: `forward_musa` implementation / `forward_musa` 实现
```python
    @torch.compile
    def forward_musa(
        self, a: torch.Tensor, b: torch.Tensor, c: torch.Tensor, k: int = 0
    ):
        return self.forward_native(a, b, c, k=k)
```
**EN:** This block defines method `forward_musa` on `MulAdd`. It executes musa. Key calls include `self.forward_native`. Parameters such as `a`, `b`, `c`, and `k` drive the behavior in this section.
**CN:** 该代码块定义了 `MulAdd` 的方法 `forward_musa`。 它用于执行前向计算musa。 关键调用包括 `self.forward_native`。 本段逻辑主要由 `a`、`b`、`c` 和 `k` 等参数驱动。

### Lines 48-53: `forward_npu` implementation / `forward_npu` 实现
```python
    def forward_npu(
        self, a: torch.Tensor, b: torch.Tensor, c: torch.Tensor, k: int = 0
    ):
        from sgl_kernel_npu.norm.scale_shift import fused_scale_shift

        return fused_scale_shift(a, b, c, scale_constant=k)
```
**EN:** This block defines method `forward_npu` on `MulAdd`. It executes npu. Key calls include `fused_scale_shift`. Parameters such as `a`, `b`, `c`, and `k` drive the behavior in this section.
**CN:** 该代码块定义了 `MulAdd` 的方法 `forward_npu`。 它用于执行前向计算npu。 关键调用包括 `fused_scale_shift`。 本段逻辑主要由 `a`、`b`、`c` 和 `k` 等参数驱动。

## Key Concepts / 关键概念
- `MulAdd`: Fuse elementwise mul and add Input: a, b, c, OptionalInt[k] Output: a * (k + b) + c / 核心类，用于封装 mul add 相关行为。

## Dependencies / 依赖关系
- **Third-party / 第三方依赖**: `torch`, `sgl_kernel_npu.norm.scale_shift`
- **Internal modules / 内部模块**: `sglang.jit_kernel.diffusion.triton.scale_shift`, `sglang.multimodal_gen.runtime.layers.custom_op`

- **Total lines / 总行数**: 53
