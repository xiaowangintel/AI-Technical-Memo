# awq_kernels.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/hardware_backend/gpu/quantization/awq_kernels.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements GPU backend support for quantization helpers inside the SGLang runtime. / 为 SGLang 运行时提供面向 GPU 后端的量化辅助逻辑支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-29: Module setup and shared state / 模块设置与共享状态
```python
from __future__ import annotations

from typing import TYPE_CHECKING, Optional

import torch

from sglang.srt.layers.moe import MoeRunner
from sglang.srt.layers.moe.moe_runner.marlin import MarlinMoeQuantInfo
from sglang.srt.layers.quantization.marlin_utils import (
    apply_awq_marlin_linear,
    awq_to_marlin_zero_points,
    marlin_make_empty_g_idx,
    marlin_make_workspace,
    marlin_moe_permute_scales,
    marlin_permute_scales,
    moe_awq_to_marlin_zero_points,
)
from sglang.srt.layers.quantization.utils import get_scalar_types, replace_parameter
from sglang.srt.utils import is_hip, is_xpu

if TYPE_CHECKING:
    from sglang.srt.layers.moe.token_dispatcher import (
        CombineInput,
        StandardDispatchOutput,
    )
    from sglang.srt.layers.quantization.base_config import QuantizationConfig

awq_marlin_moe_repack = None
awq_marlin_repack = None
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `__future__`, `typing`, `torch`, `sglang.srt.layers.moe`, `sglang.srt.layers.moe.moe_runner.marlin`, `sglang.srt.layers.quantization.marlin_utils`. It also defines symbols such as `awq_marlin_moe_repack`, `awq_marlin_repack` for later sections.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `__future__`, `typing`, `torch`, `sglang.srt.layers.moe`, `sglang.srt.layers.moe.moe_runner.marlin`, `sglang.srt.layers.quantization.marlin_utils`。 同时定义了 `awq_marlin_moe_repack`, `awq_marlin_repack` 等符号，供后续逻辑使用。

### Lines 32-33: Function `_unsupported_awq_dequantize` / 函数 `_unsupported_awq_dequantize`
```python
def _unsupported_awq_dequantize(*args, **kwargs):
    raise RuntimeError("AWQ GPU kernels are unavailable on the current platform.")
```
**EN:** This function implements `_unsupported_awq_dequantize`. It primarily calls `RuntimeError` to complete its work.
**CN:** 该函数实现了 `_unsupported_awq_dequantize`。 它主要通过调用 `RuntimeError` 来完成任务。

### Lines 36-76: Conditional and fallback logic / 条件与回退逻辑
```python
awq_dequantize = _unsupported_awq_dequantize

if is_xpu():
    try:
        from sgl_kernel import awq_dequantize
    except ImportError:
        pass
elif is_hip():
    try:
        from sglang.srt.layers.quantization.awq.awq_triton import (
            awq_dequantize_triton as awq_dequantize,
        )
    except ImportError:
        pass
else:
    try:
        from sglang.jit_kernel.awq_dequantize import awq_dequantize
        from sglang.jit_kernel.awq_marlin_repack import (
            awq_marlin_moe_repack,
            awq_marlin_repack,
        )
        from sglang.srt.utils.custom_op import register_custom_op_from_extern

        awq_dequantize = register_custom_op_from_extern(
            awq_dequantize,
            fake_impl=lambda qweight, scales, qzeros: qweight.new_empty(
                qweight.shape[:-1] + (qweight.shape[-1] * 8,), dtype=scales.dtype
            ),
# ... omitted for brevity ...
            try:
                from sgl_kernel import awq_dequantize
            except ImportError:
                pass

_, scalar_types = get_scalar_types()
```
**EN:** This block contains conditional, fallback, or guarded runtime logic that adapts behavior to the current environment. It also updates symbols such as `awq_dequantize`, `awq_dequantize`.
**CN:** 该代码块包含条件分支、回退路径或受保护的运行时逻辑，用于根据当前环境调整行为。 它还会更新 `awq_dequantize`, `awq_dequantize` 等符号。

### Lines 79-79: Class `AWQLinearKernel` declaration / 类 `AWQLinearKernel` 声明
```python
class AWQLinearKernel:
```
**EN:** This class establishes `AWQLinearKernel` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `__init__`, `process_weights_after_loading`, `apply`.
**CN:** 该类将 `AWQLinearKernel` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `__init__`, `process_weights_after_loading`, `apply` 等方法。

### Lines 80-81: Method `AWQLinearKernel.__init__` / 方法 `AWQLinearKernel.__init__`
```python
    def __init__(self, quant_config: Optional["QuantizationConfig"] = None):
        self.quant_config = quant_config
```
**EN:** This method implements `__init__` on `AWQLinearKernel`. State updates are written into `self.quant_config`.
**CN:** 该方法（属于 `AWQLinearKernel`）实现了 `__init__`。 状态更新主要写入 `self.quant_config`。

### Lines 83-86: Method `AWQLinearKernel.process_weights_after_loading` / 方法 `AWQLinearKernel.process_weights_after_loading`
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        layer.qweight = torch.nn.Parameter(layer.qweight.data, requires_grad=False)
        layer.qzeros = torch.nn.Parameter(layer.qzeros.data, requires_grad=False)
        layer.scales = torch.nn.Parameter(layer.scales.data, requires_grad=False)
```
**EN:** This method implements `process_weights_after_loading` on `AWQLinearKernel`. It primarily calls `torch.nn.Parameter` to complete its work. State updates are written into `layer.qweight`, `layer.qzeros`, `layer.scales`.
**CN:** 该方法（属于 `AWQLinearKernel`）实现了 `process_weights_after_loading`。 它主要通过调用 `torch.nn.Parameter` 来完成任务。 状态更新主要写入 `layer.qweight`, `layer.qzeros`, `layer.scales`。

### Lines 88-105: Method `AWQLinearKernel.apply` / 方法 `AWQLinearKernel.apply`
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        qweight = layer.qweight
        scales = layer.scales
        qzeros = layer.qzeros
        pack_factor = self.quant_config.pack_factor
        out_shape = x.shape[:-1] + (qweight.shape[-1] * pack_factor,)
        reshaped_x = x.reshape(-1, x.shape[-1])
        out = awq_dequantize(qweight, scales, qzeros)
        out = torch.matmul(reshaped_x, out)

        if bias is not None:
            out.add_(bias)
        return out.reshape(out_shape)
```
**EN:** This method implements `apply` on `AWQLinearKernel`. It primarily calls `x.reshape`, `awq_dequantize`, `torch.matmul`, `out.reshape`, `out.add_` to complete its work. State updates are written into `qweight`, `scales`, `qzeros`, `pack_factor`, `out_shape`, `reshaped_x`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `AWQLinearKernel`）实现了 `apply`。 它主要通过调用 `x.reshape`, `awq_dequantize`, `torch.matmul`, `out.reshape`, `out.add_` 来完成任务。 状态更新主要写入 `qweight`, `scales`, `qzeros`, `pack_factor`, `out_shape`, `reshaped_x`。 实现中使用了条件分支。

### Lines 108-108: Class `AWQMarlinLinearKernel` declaration / 类 `AWQMarlinLinearKernel` 声明
```python
class AWQMarlinLinearKernel:
```
**EN:** This class establishes `AWQMarlinLinearKernel` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `__init__`, `process_weights_after_loading`, `apply`.
**CN:** 该类将 `AWQMarlinLinearKernel` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `__init__`, `process_weights_after_loading`, `apply` 等方法。

### Lines 109-110: Method `AWQMarlinLinearKernel.__init__` / 方法 `AWQMarlinLinearKernel.__init__`
```python
    def __init__(self, quant_config: Optional["QuantizationConfig"] = None):
        self.quant_config = quant_config
```
**EN:** This method implements `__init__` on `AWQMarlinLinearKernel`. State updates are written into `self.quant_config`.
**CN:** 该方法（属于 `AWQMarlinLinearKernel`）实现了 `__init__`。 状态更新主要写入 `self.quant_config`。

### Lines 112-145: Method `AWQMarlinLinearKernel.process_weights_after_loading` / 方法 `AWQMarlinLinearKernel.process_weights_after_loading`
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        device = layer.qweight.device
        layer.qweight = torch.nn.Parameter(layer.qweight.data, requires_grad=False)
        layer.qzeros = torch.nn.Parameter(layer.qzeros.data, requires_grad=False)
        layer.scales = torch.nn.Parameter(layer.scales.data, requires_grad=False)

        layer.workspace = marlin_make_workspace(device)

        marlin_qweight = awq_marlin_repack(
            layer.qweight,
            size_k=layer.input_size_per_partition,
            size_n=layer.output_size_per_partition,
            num_bits=self.quant_config.quant_type.size_bits,
        )
        replace_parameter(layer, "qweight", marlin_qweight)

        marlin_scales = marlin_permute_scales(
            layer.scales,
            size_k=layer.input_size_per_partition,
            size_n=layer.output_size_per_partition,
            group_size=self.quant_config.group_size,
        )
        replace_parameter(layer, "scales", marlin_scales)

        marlin_zp = awq_to_marlin_zero_points(
            layer.qzeros,
            size_k=layer.num_groups,
            size_n=layer.output_size_per_partition,
            num_bits=self.quant_config.quant_type.size_bits,
        )
        replace_parameter(layer, "qzeros", marlin_zp)

        layer.g_idx = marlin_make_empty_g_idx(device)
        layer.g_idx_sort_indices = marlin_make_empty_g_idx(device)
```
**EN:** This method implements `process_weights_after_loading` on `AWQMarlinLinearKernel`. It primarily calls `torch.nn.Parameter`, `marlin_make_workspace`, `awq_marlin_repack`, `replace_parameter`, `marlin_permute_scales`, `awq_to_marlin_zero_points` to complete its work. State updates are written into `device`, `layer.qweight`, `layer.qzeros`, `layer.scales`, `layer.workspace`, `marlin_qweight`.
**CN:** 该方法（属于 `AWQMarlinLinearKernel`）实现了 `process_weights_after_loading`。 它主要通过调用 `torch.nn.Parameter`, `marlin_make_workspace`, `awq_marlin_repack`, `replace_parameter`, `marlin_permute_scales`, `awq_to_marlin_zero_points` 来完成任务。 状态更新主要写入 `device`, `layer.qweight`, `layer.qzeros`, `layer.scales`, `layer.workspace`, `marlin_qweight`。

### Lines 147-165: Method `AWQMarlinLinearKernel.apply` / 方法 `AWQMarlinLinearKernel.apply`
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        return apply_awq_marlin_linear(
            input=x,
            weight=layer.qweight,
            weight_scale=layer.scales,
            weight_zp=layer.qzeros,
            g_idx=layer.g_idx,
            g_idx_sort_indices=layer.g_idx_sort_indices,
            workspace=layer.workspace,
            quant_type=self.quant_config.quant_type,
            output_size_per_partition=layer.output_size_per_partition,
            input_size_per_partition=layer.input_size_per_partition,
            bias=bias,
        )
```
**EN:** This method implements `apply` on `AWQMarlinLinearKernel`. It primarily calls `apply_awq_marlin_linear` to complete its work.
**CN:** 该方法（属于 `AWQMarlinLinearKernel`）实现了 `apply`。 它主要通过调用 `apply_awq_marlin_linear` 来完成任务。

### Lines 168-168: Class `AWQMoEKernel` declaration / 类 `AWQMoEKernel` 声明
```python
class AWQMoEKernel:
```
**EN:** This class establishes `AWQMoEKernel` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `__init__`, `process_weights_after_loading`, `apply`.
**CN:** 该类将 `AWQMoEKernel` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `__init__`, `process_weights_after_loading`, `apply` 等方法。

### Lines 169-171: Method `AWQMoEKernel.__init__` / 方法 `AWQMoEKernel.__init__`
```python
    def __init__(self, quant_config: Optional["QuantizationConfig"] = None):
        self.quant_config = quant_config
        self.runner: Optional[MoeRunner] = None
```
**EN:** This method implements `__init__` on `AWQMoEKernel`. State updates are written into `self.quant_config`, `self.runner`.
**CN:** 该方法（属于 `AWQMoEKernel`）实现了 `__init__`。 状态更新主要写入 `self.quant_config`, `self.runner`。

### Lines 173-234: Method `AWQMoEKernel.process_weights_after_loading` / 方法 `AWQMoEKernel.process_weights_after_loading`
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        num_experts = layer.w13_qweight.shape[0]
        device = layer.w13_qweight.device

        layer.w13_g_idx_sort_indices = torch.nn.Parameter(
            torch.empty((num_experts, 0), dtype=torch.int32, device=device),
            requires_grad=False,
        )
        layer.w2_g_idx_sort_indices = torch.nn.Parameter(
            torch.empty((num_experts, 0), dtype=torch.int32, device=device),
            requires_grad=False,
        )

        marlin_w13_qweight = awq_marlin_moe_repack(
            layer.w13_qweight,
            layer.w13_g_idx_sort_indices,
            size_k=layer.w13_qweight.shape[1],
            size_n=layer.w13_qweight.shape[2] * self.quant_config.pack_factor,
            num_bits=self.quant_config.weight_bits,
        )
        replace_parameter(layer, "w13_qweight", marlin_w13_qweight)

        marlin_w2_qweight = awq_marlin_moe_repack(
            layer.w2_qweight,
            layer.w2_g_idx_sort_indices,
            size_k=layer.w2_qweight.shape[1],
            size_n=layer.w2_qweight.shape[2] * self.quant_config.pack_factor,
            num_bits=self.quant_config.weight_bits,
# ... omitted for brevity ...
            layer.w2_qzeros,
            size_k=layer.w2_qzeros.shape[1],
            size_n=layer.w2_qzeros.shape[2] * self.quant_config.pack_factor,
            num_bits=self.quant_config.weight_bits,
        )
        replace_parameter(layer, "w2_qzeros", marlin_w2_zp)
```
**EN:** This method implements `process_weights_after_loading` on `AWQMoEKernel`. It primarily calls `torch.nn.Parameter`, `awq_marlin_moe_repack`, `replace_parameter`, `marlin_moe_permute_scales`, `moe_awq_to_marlin_zero_points`, `torch.empty` to complete its work. State updates are written into `num_experts`, `device`, `layer.w13_g_idx_sort_indices`, `layer.w2_g_idx_sort_indices`, `marlin_w13_qweight`, `marlin_w2_qweight`.
**CN:** 该方法（属于 `AWQMoEKernel`）实现了 `process_weights_after_loading`。 它主要通过调用 `torch.nn.Parameter`, `awq_marlin_moe_repack`, `replace_parameter`, `marlin_moe_permute_scales`, `moe_awq_to_marlin_zero_points`, `torch.empty` 来完成任务。 状态更新主要写入 `num_experts`, `device`, `layer.w13_g_idx_sort_indices`, `layer.w2_g_idx_sort_indices`, `marlin_w13_qweight`, `marlin_w2_qweight`。

### Lines 236-255: Method `AWQMoEKernel.apply` / 方法 `AWQMoEKernel.apply`
```python
    def apply(
        self,
        layer: torch.nn.Module,
        dispatch_output: "StandardDispatchOutput",
    ) -> "CombineInput":
        if self.runner is None:
            raise RuntimeError("moe runner is not initialized")

        quant_info = MarlinMoeQuantInfo(
            w13_qweight=layer.w13_qweight,
            w2_qweight=layer.w2_qweight,
            w13_scales=layer.w13_scales,
            w2_scales=layer.w2_scales,
            w13_g_idx_sort_indices=layer.w13_g_idx_sort_indices,
            w2_g_idx_sort_indices=layer.w2_g_idx_sort_indices,
            w13_qzeros=layer.w13_qzeros,
            w2_qzeros=layer.w2_qzeros,
            weight_bits=self.quant_config.weight_bits,
        )
        return self.runner.run(dispatch_output, quant_info)
```
**EN:** This method implements `apply` on `AWQMoEKernel`. It primarily calls `MarlinMoeQuantInfo`, `self.runner.run`, `RuntimeError` to complete its work. State updates are written into `quant_info`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `AWQMoEKernel`）实现了 `apply`。 它主要通过调用 `MarlinMoeQuantInfo`, `self.runner.run`, `RuntimeError` 来完成任务。 状态更新主要写入 `quant_info`。 实现中使用了条件分支。

## Key Concepts / 关键概念
- **Classes / 类**: `AWQLinearKernel`, `AWQMarlinLinearKernel`, `AWQMoEKernel`
- **Functions / 函数**: `_unsupported_awq_dequantize`, `__init__`, `process_weights_after_loading`, `apply`, `__init__`, `process_weights_after_loading`, `apply`, `__init__`
- **Themes / 主题**: `quant`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.layers.moe`, `sglang.srt.layers.moe.moe_runner.marlin`, `sglang.srt.layers.quantization.marlin_utils`, `sglang.srt.layers.quantization.utils`, `sglang.srt.utils`, `sglang.srt.layers.moe.token_dispatcher`, `sglang.srt.layers.quantization.base_config`, `sglang.srt.layers.quantization.awq.awq_triton`, `sglang.jit_kernel.awq_dequantize`, `sglang.jit_kernel.awq_marlin_repack`, `sglang.srt.utils.custom_op`
- **External / 外部依赖**: `torch`, `sgl_kernel`
- **Standard library / 标准库**: `__future__`, `typing`
