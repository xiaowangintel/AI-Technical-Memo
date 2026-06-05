# input_quant_fp8.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/quantization/input_quant_fp8.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `QuantFP8` for quantization backends, schemes, and utilities. / 实现 `QuantFP8`，用于量化后端、方案与工具。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 4-21)
```python
import torch
import torch.nn.functional as F

from vllm import _custom_ops as ops
from vllm._aiter_ops import rocm_aiter_ops
from vllm.model_executor.custom_op import CustomOp
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    GroupShape,
    get_fp8_min_max,
    group_broadcast,
    prep_scale_for_group_broadcast,
)
from vllm.platforms import current_platform
from vllm.utils.deep_gemm import (
    DeepGemmQuantScaleFMT,
    is_deep_gemm_e8m0_used,
    is_deep_gemm_supported,
)
```
**EN:** This opening block pulls in external dependencies such as `torch` and internal modules such as `vllm`, `vllm._aiter_ops`, `vllm.model_executor.custom_op`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.platforms`, `vllm.utils.deep_gemm`. That import mix shows the file is part of the quantization backends, schemes, and utilities stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `torch`）以及内部模块（如 `vllm`, `vllm._aiter_ops`, `vllm.model_executor.custom_op`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.platforms`, `vllm.utils.deep_gemm`）。这些导入关系表明该文件属于量化后端、方案与工具栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Module constants and metadata (lines 23-25)
```python
_FP8_DTYPE = current_platform.fp8_dtype()
_FP8_MIN, _FP8_MAX = get_fp8_min_max()
_FP8_MIN_SCALING_FACTOR = 1.0 / (_FP8_MAX * 512.0)
```
**EN:** This block defines module-level metadata or constants such as `_FP8_DTYPE`, `_FP8_MIN`, `_FP8_MAX`, `_FP8_MIN_SCALING_FACTOR`. Those symbols usually capture defaults, aliases, export lists, or registry data that later classes/functions reuse inside the quantization backends, schemes, and utilities pipeline.
**CN:** 这个代码块定义了模块级元数据或常量，例如 `_FP8_DTYPE`, `_FP8_MIN`, `_FP8_MAX`, `_FP8_MIN_SCALING_FACTOR`。这些符号通常用于保存默认值、别名、导出列表或注册信息，供后续类和函数在量化后端、方案与工具流程中复用。

### Class `QuantFP8` overview (lines 30-261)
```python
class QuantFP8(CustomOp):
    """
    Quantize input tensor to FP8 (per-tensor, per-token, per-channel, or per-group).
    This CustomOp supports both static and dynamic quantization.
    """

    # --8<-- [end:quant_fp8]

    def __init__(
        self,
        static: bool,
        group_shape: GroupShape,
        num_token_padding: int | None = None,
        column_major_scales: bool = False,
        tma_aligned_scales: bool = False,
        use_ue8m0: bool | None = None,  # for Torch compile
        compile_native: bool = True,
    ):
        """
        :param static: static or dynamic quantization
        :param group_shape: quantization group shape (PER_TOKEN, PER_TENSOR,
            PER_CHANNEL, or arbitrary block size)
        :param num_token_padding: Pad the token dimension of output to this
            size
        :param tma_aligned_scales: For group quantization, output scales in
```
**EN:** Defines class `QuantFP8` with base classes `CustomOp` and decorators `CustomOp.register('quant_fp8')`. It acts as a quantization-oriented module building block and exposes 6 direct methods, with notable entries `__init__`, `forward_cuda`, `forward_hip`, `forward_xpu`, `forward_native`, `_quantize_group_native`. Its docstring says: Quantize input tensor to FP8 (per-tensor, per-token, per-channel, or per-group).
**CN:** 定义类 `QuantFP8`，其基类为 `CustomOp`，装饰器为 `CustomOp.register('quant_fp8')`。它在整体实现中充当面向量化的模块构件，并直接暴露 6 个方法，较重要的包括 `__init__`, `forward_cuda`, `forward_hip`, `forward_xpu`, `forward_native`, `_quantize_group_native`。 文档字符串进一步说明了该类的定位。

### Method `QuantFP8.__init__` (lines 38-81)
```python
    def __init__(
        self,
        static: bool,
        group_shape: GroupShape,
        num_token_padding: int | None = None,
        column_major_scales: bool = False,
        tma_aligned_scales: bool = False,
        use_ue8m0: bool | None = None,  # for Torch compile
        compile_native: bool = True,
    ):
        """
        :param static: static or dynamic quantization
        :param group_shape: quantization group shape (PER_TOKEN, PER_TENSOR,
            PER_CHANNEL, or arbitrary block size)
        :param num_token_padding: Pad the token dimension of output to this
            size
        :param tma_aligned_scales: For group quantization, output scales in
            TMA-aligned layout
        :param column_major_scales: For group quantization, output scales in
            column major format
        :param compile_native: Manually compile forward_native if compile mode > None
        """
        super().__init__(compile_native=compile_native)
        self.static = static
        self.group_shape = group_shape
        self.use_per_token_if_dynamic = group_shape == GroupShape.PER_TOKEN
        self.num_token_padding = num_token_padding
        self.column_major_scales = column_major_scales
        self.tma_aligned_scales = tma_aligned_scales
        self.use_ue8m0 = is_deep_gemm_e8m0_used() if use_ue8m0 is None else use_ue8m0
        self.use_deep_gemm_supported = is_deep_gemm_supported()

        self.use_aiter = rocm_aiter_ops.is_linear_fp8_enabled()

        self.is_group_quant = group_shape.is_per_group()
        if self.is_group_quant:
            self.group_size = group_shape.col
        else:
            self.use_per_token_if_dynamic = group_shape == GroupShape.PER_TOKEN
            if not static:
                assert group_shape in (GroupShape.PER_TOKEN, GroupShape.PER_TENSOR), (
                    "Only per-token or per-tensor scales are supported for dynamic "
                    "non-group quantization."
                )
```
**EN:** Defines function `QuantFP8.__init__` with signature `__init__(self, static: bool, group_shape: GroupShape, num_token_padding: int | None=None, column_major_scales: bool=False, tma_aligned_scales: bool=False, use_ue8m0: bool | None=None, compile_native: bool=True)`. It mainly works with `static`, `group_shape`, `num_token_padding`, `column_major_scales`, `tma_aligned_scales`, `use_ue8m0`, `compile_native`; initializes the object state and cached resources. The body uses branching, validation/error handling. Key calls include `super.__init__`, `is_deep_gemm_supported`, `rocm_aiter_ops.is_linear_fp8_enabled`, `group_shape.is_per_group`, `is_deep_gemm_e8m0_used`, `super`.
**CN:** 定义函数 `QuantFP8.__init__`，其签名为 `__init__(self, static: bool, group_shape: GroupShape, num_token_padding: int | None=None, column_major_scales: bool=False, tma_aligned_scales: bool=False, use_ue8m0: bool | None=None, compile_native: bool=True)`。它主要围绕 `static`, `group_shape`, `num_token_padding`, `column_major_scales`, `tma_aligned_scales`, `use_ue8m0`, `compile_native` 展开；负责初始化对象状态和缓存资源。函数体包含分支判断、校验或报错逻辑。关键调用包括 `super.__init__`, `is_deep_gemm_supported`, `rocm_aiter_ops.is_linear_fp8_enabled`, `group_shape.is_per_group`, `is_deep_gemm_e8m0_used`, `super`。

### Method `QuantFP8.forward_cuda` (lines 83-132)
```python
    def forward_cuda(
        self,
        x: torch.Tensor,
        scale: torch.Tensor | None = None,
        scale_ub: torch.Tensor | None = None,
        use_triton: bool = False,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        from vllm.model_executor.layers.quantization.utils import fp8_utils

        if (
            self.is_group_quant
            and self.use_ue8m0
            and self.use_deep_gemm_supported
            and (DeepGemmQuantScaleFMT.from_oracle() == DeepGemmQuantScaleFMT.UE8M0)
        ):
            return fp8_utils.per_token_group_quant_fp8_packed_for_deepgemm(
                x,
                group_size=self.group_size,
                use_ue8m0=True,
            )

        if self.is_group_quant and not self.static:
            assert scale is None, "Dynamic group quantization does not use scale"

            return fp8_utils.per_token_group_quant_fp8(
                x,
                group_size=self.group_size,
                column_major_scales=self.column_major_scales,
                tma_aligned_scales=self.tma_aligned_scales,
                dtype=_FP8_DTYPE,
                use_ue8m0=self.use_ue8m0,
            )

        assert (scale is not None) == self.static
        assert scale_ub is None or (
            not self.static
            and self.group_shape == GroupShape.PER_TOKEN
            and scale_ub.numel() == 1
        )

        return ops.scaled_fp8_quant(
            x,
            scale,
            num_token_padding=self.num_token_padding,
            scale_ub=scale_ub,
            use_per_token_if_dynamic=self.use_per_token_if_dynamic,
            group_shape=(self.group_shape.row, self.group_shape.col)
            if self.static
            else None,
        )
```
**EN:** Defines function `QuantFP8.forward_cuda` with signature `forward_cuda(self, x: torch.Tensor, scale: torch.Tensor | None=None, scale_ub: torch.Tensor | None=None, use_triton: bool=False) -> tuple[torch.Tensor, torch.Tensor]`. It mainly works with `x`, `scale`, `scale_ub`, `use_triton`; implements one step in the quantized-weight execution flow. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `ops.scaled_fp8_quant`, `fp8_utils.per_token_group_quant_fp8_packed_for_deepgemm`, `fp8_utils.per_token_group_quant_fp8`, `DeepGemmQuantScaleFMT.from_oracle`, `scale_ub.numel`.
**CN:** 定义函数 `QuantFP8.forward_cuda`，其签名为 `forward_cuda(self, x: torch.Tensor, scale: torch.Tensor | None=None, scale_ub: torch.Tensor | None=None, use_triton: bool=False) -> tuple[torch.Tensor, torch.Tensor]`。它主要围绕 `x`, `scale`, `scale_ub`, `use_triton` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `ops.scaled_fp8_quant`, `fp8_utils.per_token_group_quant_fp8_packed_for_deepgemm`, `fp8_utils.per_token_group_quant_fp8`, `DeepGemmQuantScaleFMT.from_oracle`, `scale_ub.numel`。

### Method `QuantFP8.forward_hip` (lines 134-167)
```python
    def forward_hip(
        self,
        x: torch.Tensor,
        scale: torch.Tensor | None = None,
        scale_ub: torch.Tensor | None = None,
        use_triton: bool = False,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        if self.is_group_quant and use_triton:
            assert scale is None, "Dynamic group quantization does not use scale"

            return torch.ops.vllm.triton_per_token_group_quant_fp8(x, self.group_size)

        use_aiter_quant = self.use_aiter and scale_ub is None and x.is_contiguous()
        use_aiter_per_tensor_quant = (
            use_aiter_quant and self.group_shape.is_per_tensor()
        )
        use_aiter_per_token_quant = use_aiter_quant and self.group_shape.is_per_token()

        use_aiter_per_group_quant = use_aiter_quant and self.group_shape.is_per_group()

        if use_aiter_per_group_quant:
            return rocm_aiter_ops.group_fp8_quant(x, self.group_size)
        if use_aiter_per_tensor_quant:
            return rocm_aiter_ops.per_tensor_quant(x, _FP8_DTYPE, scale)
        if use_aiter_per_token_quant:
            return rocm_aiter_ops.per_token_quant(x, _FP8_DTYPE, scale)

        # Fallback to native implementation for group quantization.
        if self.is_group_quant:
            assert scale is None, "Dynamic group quantization does not use scale"
            return self._quantize_group_native(x)

        # Fallback to CUDA implementation
        return self.forward_cuda(x, scale, scale_ub)
```
**EN:** Defines function `QuantFP8.forward_hip` with signature `forward_hip(self, x: torch.Tensor, scale: torch.Tensor | None=None, scale_ub: torch.Tensor | None=None, use_triton: bool=False) -> tuple[torch.Tensor, torch.Tensor]`. It mainly works with `x`, `scale`, `scale_ub`, `use_triton`; implements one step in the quantized-weight execution flow. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `self.forward_cuda`, `torch.ops.vllm.triton_per_token_group_quant_fp8`, `x.is_contiguous`, `self.group_shape.is_per_tensor`, `self.group_shape.is_per_token`, `self.group_shape.is_per_group`.
**CN:** 定义函数 `QuantFP8.forward_hip`，其签名为 `forward_hip(self, x: torch.Tensor, scale: torch.Tensor | None=None, scale_ub: torch.Tensor | None=None, use_triton: bool=False) -> tuple[torch.Tensor, torch.Tensor]`。它主要围绕 `x`, `scale`, `scale_ub`, `use_triton` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `self.forward_cuda`, `torch.ops.vllm.triton_per_token_group_quant_fp8`, `x.is_contiguous`, `self.group_shape.is_per_tensor`, `self.group_shape.is_per_token`, `self.group_shape.is_per_group`。

### Method `QuantFP8.forward_xpu` (lines 169-177)
```python
    def forward_xpu(
        self,
        x: torch.Tensor,
        scale: torch.Tensor | None = None,
        scale_ub: torch.Tensor | None = None,
        use_triton: bool = False,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        # XPU can use same code path as CUDA.
        return self.forward_cuda(x, scale, scale_ub, use_triton)
```
**EN:** Defines function `QuantFP8.forward_xpu` with signature `forward_xpu(self, x: torch.Tensor, scale: torch.Tensor | None=None, scale_ub: torch.Tensor | None=None, use_triton: bool=False) -> tuple[torch.Tensor, torch.Tensor]`. It mainly works with `x`, `scale`, `scale_ub`, `use_triton`; implements one step in the quantized-weight execution flow. The body uses tensor/kernel operations. Key calls include `self.forward_cuda`.
**CN:** 定义函数 `QuantFP8.forward_xpu`，其签名为 `forward_xpu(self, x: torch.Tensor, scale: torch.Tensor | None=None, scale_ub: torch.Tensor | None=None, use_triton: bool=False) -> tuple[torch.Tensor, torch.Tensor]`。它主要围绕 `x`, `scale`, `scale_ub`, `use_triton` 展开；实现量化权重执行流程中的一个步骤。函数体包含张量或内核操作。关键调用包括 `self.forward_cuda`。

### Method `QuantFP8.forward_native` (lines 179-226)
```python
    def forward_native(
        self,
        x: torch.Tensor,
        scale: torch.Tensor | None = None,
        scale_ub: torch.Tensor | None = None,
        use_triton: bool = False,
    ):
        if self.is_group_quant and not self.static:
            assert scale is None, "Dynamic group quantization does not use scale"
            return self._quantize_group_native(x)

        assert (scale is not None) == self.static
        assert scale_ub is None or (
            not self.static
            and self.group_shape == GroupShape.PER_TOKEN
            and scale_ub.numel() == 1
        )

        if scale is None:
            if self.group_shape == GroupShape.PER_TOKEN:
                x_max, _ = x.abs().max(dim=-1)
                x_max = x_max.unsqueeze(-1).to(torch.float32)
                if scale_ub is not None:
                    x_max = x_max.clamp(max=scale_ub)
            else:
                x_max = x.abs().max().unsqueeze(-1).to(torch.float32)

            scale = (x_max / _FP8_MAX).clamp(min=_FP8_MIN_SCALING_FACTOR)
        else:
            scale = prep_scale_for_group_broadcast(scale, x, self.group_shape)

        # Even for dynamic per-token scales,
        # reciprocal performs slightly better than division
        out = (
            x.to(torch.float32)
            * group_broadcast(scale.to(torch.float32), x.shape[-2:]).reciprocal()
        )
        out = out.clamp(_FP8_MIN, _FP8_MAX).to(_FP8_DTYPE)

        # This currently generates an extra Triton kernel in compilation.
        # Fortunately, we don't use padding if compiling.
        # TODO(luka): benchmark torch._scaled_mm to hopefully remove padding
        #  in general.
        if self.num_token_padding is not None:
            padding = max(self.num_token_padding - out.size(0), 0)
            out = F.pad(out, (0, 0, 0, padding), "constant", 0.0)

        return out, scale
```
**EN:** Defines function `QuantFP8.forward_native` with signature `forward_native(self, x: torch.Tensor, scale: torch.Tensor | None=None, scale_ub: torch.Tensor | None=None, use_triton: bool=False)`. It mainly works with `x`, `scale`, `scale_ub`, `use_triton`; implements one step in the quantized-weight execution flow. The body uses branching, validation/error handling. Key calls include `out.clamp.to`, `self._quantize_group_native`, `clamp`, `prep_scale_for_group_broadcast`, `x.to`, `group_broadcast.reciprocal`.
**CN:** 定义函数 `QuantFP8.forward_native`，其签名为 `forward_native(self, x: torch.Tensor, scale: torch.Tensor | None=None, scale_ub: torch.Tensor | None=None, use_triton: bool=False)`。它主要围绕 `x`, `scale`, `scale_ub`, `use_triton` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、校验或报错逻辑。关键调用包括 `out.clamp.to`, `self._quantize_group_native`, `clamp`, `prep_scale_for_group_broadcast`, `x.to`, `group_broadcast.reciprocal`。

### Method `QuantFP8._quantize_group_native` (lines 228-261)
```python
    def _quantize_group_native(
        self, x: torch.Tensor
    ) -> tuple[torch.Tensor, torch.Tensor]:
        orig_shape = x.shape
        hidden_dim = x.shape[-1]
        num_groups = (hidden_dim + self.group_size - 1) // self.group_size
        padded_dim = num_groups * self.group_size

        if padded_dim != hidden_dim:
            padding = padded_dim - hidden_dim
            x = F.pad(x, (0, padding), mode="constant", value=0.0)

        x_grouped = x.view(-1, num_groups, self.group_size)
        absmax = x_grouped.abs().max(dim=-1, keepdim=True)[0].float()
        scales_raw = absmax / _FP8_MAX
        if self.use_ue8m0:
            scales_raw = torch.exp2(torch.ceil(torch.log2(scales_raw)))
        scales = (scales_raw).clamp(min=_FP8_MIN_SCALING_FACTOR)

        x_scaled = x_grouped / scales
        x_quant = x_scaled.clamp(_FP8_MIN, _FP8_MAX).to(_FP8_DTYPE)

        x_quant = x_quant.view(-1, padded_dim)
        if padded_dim != hidden_dim:
            x_quant = x_quant[..., :hidden_dim]
        x_quant = x_quant.view(orig_shape)

        scales = scales.squeeze(-1)
        scales = scales.reshape(orig_shape[:-1] + (num_groups,))

        if self.column_major_scales:
            scales = scales.transpose(-2, -1).contiguous().transpose(-1, -2)

        return x_quant, scales
```
**EN:** Defines function `QuantFP8._quantize_group_native` with signature `_quantize_group_native(self, x: torch.Tensor) -> tuple[torch.Tensor, torch.Tensor]`. It mainly works with `x`; handles quantization-related transformation logic. The body uses branching, tensor/kernel operations. Key calls include `x.view`, `x_grouped.abs.max.float`, `scales_raw.clamp`, `x_scaled.clamp.to`, `x_quant.view`, `scales.squeeze`.
**CN:** 定义函数 `QuantFP8._quantize_group_native`，其签名为 `_quantize_group_native(self, x: torch.Tensor) -> tuple[torch.Tensor, torch.Tensor]`。它主要围绕 `x` 展开；处理量化相关的变换逻辑。函数体包含分支判断、张量或内核操作。关键调用包括 `x.view`, `x_grouped.abs.max.float`, `scales_raw.clamp`, `x_scaled.clamp.to`, `x_quant.view`, `scales.squeeze`。

## Key Concepts / 关键概念
- **EN:** The file lives in the quantization stack, so weight packing, backend dispatch, and post-load processing are core concerns.
  **CN:** 该文件位于量化栈中，因此权重打包、后端分发和加载后处理是核心关注点。
- **EN:** Top-level classes include `QuantFP8`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `QuantFP8`，它们组织了主要的可复用抽象。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。
- **EN:** Internal vLLM imports show that this module is tightly integrated with the broader runtime, parallelism, or custom-layer stack.
  **CN:** vLLM 内部导入说明该模块与更大的运行时、并行机制或自定义层栈紧密集成。

## Dependencies / 依赖关系
- **External / 外部**: `torch`
- **Internal / 内部**: `vllm`, `vllm._aiter_ops`, `vllm.model_executor.custom_op`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.platforms`, `vllm.utils.deep_gemm`
