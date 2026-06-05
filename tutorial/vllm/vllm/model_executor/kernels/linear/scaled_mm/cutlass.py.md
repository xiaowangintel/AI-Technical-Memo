# cutlass.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/kernels/linear/scaled_mm/cutlass.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the CUTLASS scaled matrix-multiplication kernel path. / 实现 CUTLASS 的缩放矩阵乘内核路径。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 5-26)
```python
import torch

from vllm import _custom_ops as ops
from vllm.model_executor.layers.quantization.input_quant_fp8 import QuantFP8
from vllm.model_executor.layers.quantization.utils import replace_parameter
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    GroupShape,
)
from vllm.model_executor.layers.quantization.utils.w8a8_utils import (
    CUTLASS_BLOCK_FP8_SUPPORTED,
    convert_to_channelwise,
)
from vllm.platforms import current_platform
from vllm.utils.torch_utils import direct_register_custom_op

from .BlockScaledMMLinearKernel import Fp8BlockScaledMMLinearKernel
from .ScaledMMLinearKernel import (
    FP8ScaledMMLinearKernel,
    FP8ScaledMMLinearLayerConfig,
    Int8ScaledMMLinearKernel,
    Int8ScaledMMLinearLayerConfig,
)
```
**EN:** This import block loads `torch`, `vllm`, `vllm.model_executor.layers.quantization.input_quant_fp8`, `vllm.model_executor.layers.quantization.utils`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.model_executor.layers.quantization.utils.w8a8_utils`, ..., establishing the tensor libraries, backend hooks, and vLLM helpers needed by the rest of `cutlass.py`.
**CN:** 该导入代码块加载了 `torch`, `vllm`, `vllm.model_executor.layers.quantization.input_quant_fp8`, `vllm.model_executor.layers.quantization.utils`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.model_executor.layers.quantization.utils.w8a8_utils`, ...，为 `cutlass.py` 后续逻辑准备张量库、后端钩子以及 vLLM 辅助工具。

### Class `CutlassInt8ScaledMMLinearKernel` (lines 29-149)
```python
class CutlassInt8ScaledMMLinearKernel(Int8ScaledMMLinearKernel):
    @classmethod
    def is_supported(
        cls, compute_capability: int | None = None
    ) -> tuple[bool, str | None]:
        if not current_platform.is_cuda():
            return False, "requires CUDA."
        return True, None

    @classmethod
    def can_implement(cls, c: Int8ScaledMMLinearLayerConfig) -> tuple[bool, str | None]:
        return True, None

    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        w_q_name, w_s_name, i_s_name, i_zp_name, azp_adj_name = self.layer_param_names
        config = self.config
        # WEIGHT
        # Cutlass kernels need transposed weight.
        weight = getattr(layer, w_q_name)
        replace_parameter(
            layer,
            w_q_name,
            torch.nn.Parameter(weight.t().data, requires_grad=False),
        )

        # WEIGHT SCALE
        # Cutlass kernels support only per-tensor and per-channel.
        # If we have a fused module (QKV, MLP) with per tensor scales (thus N
        # scales being passed to the kernel), convert to the per-channel case.
        is_fused_module = len(layer.logical_widths) > 1
        weight_scale = getattr(layer, w_s_name)
        if is_fused_module and not config.is_channelwise:
            weight_scale = convert_to_channelwise(weight_scale, layer.logical_widths)
        replace_parameter(
            layer,
            w_s_name,
            torch.nn.Parameter(weight_scale.data, requires_grad=False),
        )

        # INPUT SCALE
        if config.is_static_input_scheme:
            input_scale = getattr(layer, i_s_name)

            if config.input_symmetric:
                replace_parameter(
                    layer,
                    i_s_name,
                    torch.nn.Parameter(input_scale.max(), requires_grad=False),
                )
                setattr(layer, i_zp_name, None)
            else:
                input_zero_point = getattr(layer, i_zp_name)

                # reconstruct the ranges
                int8_traits = torch.iinfo(torch.int8)
                azps = input_zero_point.to(dtype=torch.int32)
                range_max = (input_scale * (int8_traits.max - azps)).max()
                range_min = (input_scale * (int8_traits.min - azps)).min()

                scale = (range_max - range_min) / (int8_traits.max - int8_traits.min)
                replace_parameter(
                    layer, i_s_name, torch.nn.Parameter(scale, requires_grad=False)
                )

                # AZP loaded as int8 but used as int32
                azp = (int8_traits.min - range_min / scale).to(dtype=torch.int32)
                replace_parameter(
                    layer, i_zp_name, torch.nn.Parameter(azp, requires_grad=False)
                )

        # azp_adj is the AZP adjustment term, used to account for weights.
        # It does not depend on scales or azp, so it is the same for
        # static and dynamic quantization.
        # For more details, see csrc/quantization/w8a8/cutlass/Epilogues.md
        # https://github.com/vllm-project/vllm/blob/main/csrc/quantization/w8a8/cutlass/Epilogues.md
        if not config.input_symmetric:
            weight = getattr(layer, w_q_name)
            azp_adj = weight.sum(dim=0, keepdim=True, dtype=torch.int32)
            if config.is_static_input_scheme:
                # cutlass_w8a8 requires azp to be folded into azp_adj
                # in the per-tensor case
                azp_adj = getattr(layer, i_zp_name) * azp_adj
            setattr(
                layer,
                azp_adj_name,
                torch.nn.Parameter(azp_adj, requires_grad=False),
            )

    def apply_weights(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        w_q, w_s, i_s, i_zp, azp_adj = self._get_layer_params(layer)

        # ops.scaled_int8_quant supports both dynamic and static quant:
        # * dynamic, i_s is None and x_s computed from x.
        # * static, i_s is scalar and x_s is i_s.
        symmetric = azp_adj is None
        x_q, x_s, x_zp = ops.scaled_int8_quant(
            x.contiguous(), i_s, i_zp, symmetric=symmetric
        )

        if x_zp is not None:
            # Currently, static is always per-tensor and dynamic is per-token
            static = i_zp is not None
            azp = None if static else x_zp
            return ops.cutlass_scaled_mm_azp(
                x_q,
                w_q,
                scale_a=x_s,
                scale_b=w_s,
                out_dtype=x.dtype,
                azp_adj=azp_adj,
                azp=azp,
                bias=bias,
            )
        return ops.cutlass_scaled_mm(
            x_q, w_q, scale_a=x_s, scale_b=w_s, out_dtype=x.dtype, bias=bias
        )
```
**EN:** This kernel class defines `CutlassInt8ScaledMMLinearKernel`. It organizes the state and behavior needed by this kernel path. It inherits from `Int8ScaledMMLinearKernel`. Key methods include `is_supported`, `can_implement`, `process_weights_after_loading`, `apply_weights`.
**CN:** 该内核类定义了 `CutlassInt8ScaledMMLinearKernel`。 它组织了该内核路径所需的状态与行为。 它继承自 `Int8ScaledMMLinearKernel`。 关键方法包括 `is_supported`, `can_implement`, `process_weights_after_loading`, `apply_weights`。

### Method `CutlassInt8ScaledMMLinearKernel.is_supported` (lines 31-36)
```python
    def is_supported(
        cls, compute_capability: int | None = None
    ) -> tuple[bool, str | None]:
        if not current_platform.is_cuda():
            return False, "requires CUDA."
        return True, None
```
**EN:** This method implements `CutlassInt8ScaledMMLinearKernel.is_supported`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `is_cuda`.
**CN:** 该方法 `CutlassInt8ScaledMMLinearKernel.is_supported` 封装了此模块中的一段关键运行时逻辑，重点处理 is supported 相关工作。 它内部会调用 `is_cuda` 等例程。

### Method `CutlassInt8ScaledMMLinearKernel.can_implement` (lines 39-40)
```python
    def can_implement(cls, c: Int8ScaledMMLinearLayerConfig) -> tuple[bool, str | None]:
        return True, None
```
**EN:** This method implements `CutlassInt8ScaledMMLinearKernel.can_implement`, encapsulating a focused unit of runtime logic for the module.
**CN:** 该方法 `CutlassInt8ScaledMMLinearKernel.can_implement` 封装了此模块中的一段关键运行时逻辑，重点处理 can implement 相关工作。

### Method `CutlassInt8ScaledMMLinearKernel.process_weights_after_loading` (lines 42-115)
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        w_q_name, w_s_name, i_s_name, i_zp_name, azp_adj_name = self.layer_param_names
        config = self.config
        # WEIGHT
        # Cutlass kernels need transposed weight.
        weight = getattr(layer, w_q_name)
        replace_parameter(
            layer,
            w_q_name,
            torch.nn.Parameter(weight.t().data, requires_grad=False),
        )

        # WEIGHT SCALE
        # Cutlass kernels support only per-tensor and per-channel.
        # If we have a fused module (QKV, MLP) with per tensor scales (thus N
        # scales being passed to the kernel), convert to the per-channel case.
        is_fused_module = len(layer.logical_widths) > 1
        weight_scale = getattr(layer, w_s_name)
        if is_fused_module and not config.is_channelwise:
            weight_scale = convert_to_channelwise(weight_scale, layer.logical_widths)
        replace_parameter(
            layer,
            w_s_name,
            torch.nn.Parameter(weight_scale.data, requires_grad=False),
        )

        # INPUT SCALE
        if config.is_static_input_scheme:
            input_scale = getattr(layer, i_s_name)

            if config.input_symmetric:
                replace_parameter(
                    layer,
                    i_s_name,
                    torch.nn.Parameter(input_scale.max(), requires_grad=False),
                )
                setattr(layer, i_zp_name, None)
            else:
                input_zero_point = getattr(layer, i_zp_name)

                # reconstruct the ranges
                int8_traits = torch.iinfo(torch.int8)
                azps = input_zero_point.to(dtype=torch.int32)
                range_max = (input_scale * (int8_traits.max - azps)).max()
                range_min = (input_scale * (int8_traits.min - azps)).min()

                scale = (range_max - range_min) / (int8_traits.max - int8_traits.min)
                replace_parameter(
                    layer, i_s_name, torch.nn.Parameter(scale, requires_grad=False)
                )

                # AZP loaded as int8 but used as int32
                azp = (int8_traits.min - range_min / scale).to(dtype=torch.int32)
                replace_parameter(
                    layer, i_zp_name, torch.nn.Parameter(azp, requires_grad=False)
                )

        # azp_adj is the AZP adjustment term, used to account for weights.
        # It does not depend on scales or azp, so it is the same for
        # static and dynamic quantization.
        # For more details, see csrc/quantization/w8a8/cutlass/Epilogues.md
        # https://github.com/vllm-project/vllm/blob/main/csrc/quantization/w8a8/cutlass/Epilogues.md
        if not config.input_symmetric:
            weight = getattr(layer, w_q_name)
            azp_adj = weight.sum(dim=0, keepdim=True, dtype=torch.int32)
            if config.is_static_input_scheme:
                # cutlass_w8a8 requires azp to be folded into azp_adj
                # in the per-tensor case
                azp_adj = getattr(layer, i_zp_name) * azp_adj
            setattr(
                layer,
                azp_adj_name,
                torch.nn.Parameter(azp_adj, requires_grad=False),
            )
```
**EN:** This method implements `CutlassInt8ScaledMMLinearKernel.process_weights_after_loading`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `getattr`, `replace_parameter`, `Parameter`, `len`.
**CN:** 该方法 `CutlassInt8ScaledMMLinearKernel.process_weights_after_loading` 封装了此模块中的一段关键运行时逻辑，重点处理 process weights after loading 相关工作。 它内部会调用 `getattr`, `replace_parameter`, `Parameter`, `len` 等例程。

### Method `CutlassInt8ScaledMMLinearKernel.apply_weights` (lines 117-149)
```python
    def apply_weights(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        w_q, w_s, i_s, i_zp, azp_adj = self._get_layer_params(layer)

        # ops.scaled_int8_quant supports both dynamic and static quant:
        # * dynamic, i_s is None and x_s computed from x.
        # * static, i_s is scalar and x_s is i_s.
        symmetric = azp_adj is None
        x_q, x_s, x_zp = ops.scaled_int8_quant(
            x.contiguous(), i_s, i_zp, symmetric=symmetric
        )

        if x_zp is not None:
            # Currently, static is always per-tensor and dynamic is per-token
            static = i_zp is not None
            azp = None if static else x_zp
            return ops.cutlass_scaled_mm_azp(
                x_q,
                w_q,
                scale_a=x_s,
                scale_b=w_s,
                out_dtype=x.dtype,
                azp_adj=azp_adj,
                azp=azp,
                bias=bias,
            )
        return ops.cutlass_scaled_mm(
            x_q, w_q, scale_a=x_s, scale_b=w_s, out_dtype=x.dtype, bias=bias
        )
```
**EN:** This method implements `CutlassInt8ScaledMMLinearKernel.apply_weights`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `_get_layer_params`, `scaled_int8_quant`, `cutlass_scaled_mm`, `contiguous`.
**CN:** 该方法 `CutlassInt8ScaledMMLinearKernel.apply_weights` 封装了此模块中的一段关键运行时逻辑，重点处理 apply weights 相关工作。 它内部会调用 `_get_layer_params`, `scaled_int8_quant`, `cutlass_scaled_mm`, `contiguous` 等例程。

### Class `CutlassFP8ScaledMMLinearKernel` (lines 152-222)
```python
class CutlassFP8ScaledMMLinearKernel(FP8ScaledMMLinearKernel):
    @classmethod
    def is_supported(
        cls, compute_capability: int | None = None
    ) -> tuple[bool, str | None]:
        if not current_platform.is_cuda():
            return False, "requires CUDA."
        return True, None

    @classmethod
    def can_implement(cls, c: FP8ScaledMMLinearLayerConfig) -> tuple[bool, str | None]:
        return True, None

    @staticmethod
    def _pad_to_alignment(
        x: torch.Tensor, dim: int, alignment: int, value: float = 0.0
    ) -> torch.Tensor:
        """Pad tensor ``x`` along ``dim`` to the next multiple of
        ``alignment``."""
        remainder = x.shape[dim] % alignment
        if remainder == 0:
            return x
        pad_size = alignment - remainder
        pad_spec = [0] * (2 * x.dim())
        pad_spec[-(2 * dim + 1)] = pad_size
        return torch.nn.functional.pad(x, pad_spec, value=value)

    def apply_scaled_mm(
        self,
        *,
        A: torch.Tensor,
        B: torch.Tensor,
        out_dtype: torch.dtype,
        As: torch.Tensor,
        Bs: torch.Tensor,
        bias: torch.Tensor | None,
        output_shape: list,
    ) -> torch.Tensor:
        # Per-tensor/Per-channel padding to use Cutlass instead of Triton.
        K, N = B.shape
        pad_k = (16 - K % 16) % 16
        pad_n = (16 - N % 16) % 16

        if pad_k > 0 or pad_n > 0:
            # B is column-major [K, N].  Transpose to row-major [N, K],
            # pad both dims in one call, then transpose back so the
            # result keeps column-major layout with stride (1, K_padded).
            B = torch.nn.functional.pad(B.t().contiguous(), (0, pad_k, 0, pad_n)).t()

            if pad_k > 0:
                A = self._pad_to_alignment(A, dim=1, alignment=16)
            if pad_n > 0:
                if bias is not None:
                    bias = self._pad_to_alignment(bias, dim=0, alignment=16)
                # Bs is per-tensor (numel==1) or per-channel (numel==N)
                # in this kernel class — never 2D block-wise.
                if Bs.numel() > 1:
                    Bs = self._pad_to_alignment(
                        Bs.view(-1), dim=0, alignment=16, value=1.0
                    )
                    if Bs.dim() == 1 and B.shape[1] > 1:
                        Bs = Bs.view(-1, 1)

        output = ops.cutlass_scaled_mm(
            A, B, out_dtype=out_dtype, scale_a=As, scale_b=Bs, bias=bias
        )

        if pad_n > 0:
            output = output[..., :N].contiguous()

        return output.view(*output_shape)
```
**EN:** This kernel class defines `CutlassFP8ScaledMMLinearKernel`. It organizes the state and behavior needed by this kernel path. It inherits from `FP8ScaledMMLinearKernel`. Key methods include `is_supported`, `can_implement`, `_pad_to_alignment`, `apply_scaled_mm`.
**CN:** 该内核类定义了 `CutlassFP8ScaledMMLinearKernel`。 它组织了该内核路径所需的状态与行为。 它继承自 `FP8ScaledMMLinearKernel`。 关键方法包括 `is_supported`, `can_implement`, `_pad_to_alignment`, `apply_scaled_mm`。

### Method `CutlassFP8ScaledMMLinearKernel.is_supported` (lines 154-159)
```python
    def is_supported(
        cls, compute_capability: int | None = None
    ) -> tuple[bool, str | None]:
        if not current_platform.is_cuda():
            return False, "requires CUDA."
        return True, None
```
**EN:** This method implements `CutlassFP8ScaledMMLinearKernel.is_supported`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `is_cuda`.
**CN:** 该方法 `CutlassFP8ScaledMMLinearKernel.is_supported` 封装了此模块中的一段关键运行时逻辑，重点处理 is supported 相关工作。 它内部会调用 `is_cuda` 等例程。

### Method `CutlassFP8ScaledMMLinearKernel.can_implement` (lines 162-163)
```python
    def can_implement(cls, c: FP8ScaledMMLinearLayerConfig) -> tuple[bool, str | None]:
        return True, None
```
**EN:** This method implements `CutlassFP8ScaledMMLinearKernel.can_implement`, encapsulating a focused unit of runtime logic for the module.
**CN:** 该方法 `CutlassFP8ScaledMMLinearKernel.can_implement` 封装了此模块中的一段关键运行时逻辑，重点处理 can implement 相关工作。

### Method `CutlassFP8ScaledMMLinearKernel._pad_to_alignment` (lines 166-177)
```python
    def _pad_to_alignment(
        x: torch.Tensor, dim: int, alignment: int, value: float = 0.0
    ) -> torch.Tensor:
        """Pad tensor ``x`` along ``dim`` to the next multiple of
        ``alignment``."""
        remainder = x.shape[dim] % alignment
        if remainder == 0:
            return x
        pad_size = alignment - remainder
        pad_spec = [0] * (2 * x.dim())
        pad_spec[-(2 * dim + 1)] = pad_size
        return torch.nn.functional.pad(x, pad_spec, value=value)
```
**EN:** This method implements `CutlassFP8ScaledMMLinearKernel._pad_to_alignment`. Pad tensor ``x`` along ``dim`` to the next multiple of ``alignment``. Internally it relies on calls such as `pad`, `dim`.
**CN:** 该方法 `CutlassFP8ScaledMMLinearKernel._pad_to_alignment` 封装了此模块中的一段关键运行时逻辑，重点处理 pad to alignment 相关工作。 它内部会调用 `pad`, `dim` 等例程。

### Method `CutlassFP8ScaledMMLinearKernel.apply_scaled_mm` (lines 179-222)
```python
    def apply_scaled_mm(
        self,
        *,
        A: torch.Tensor,
        B: torch.Tensor,
        out_dtype: torch.dtype,
        As: torch.Tensor,
        Bs: torch.Tensor,
        bias: torch.Tensor | None,
        output_shape: list,
    ) -> torch.Tensor:
        # Per-tensor/Per-channel padding to use Cutlass instead of Triton.
        K, N = B.shape
        pad_k = (16 - K % 16) % 16
        pad_n = (16 - N % 16) % 16

        if pad_k > 0 or pad_n > 0:
            # B is column-major [K, N].  Transpose to row-major [N, K],
            # pad both dims in one call, then transpose back so the
            # result keeps column-major layout with stride (1, K_padded).
            B = torch.nn.functional.pad(B.t().contiguous(), (0, pad_k, 0, pad_n)).t()

            if pad_k > 0:
                A = self._pad_to_alignment(A, dim=1, alignment=16)
            if pad_n > 0:
                if bias is not None:
                    bias = self._pad_to_alignment(bias, dim=0, alignment=16)
                # Bs is per-tensor (numel==1) or per-channel (numel==N)
                # in this kernel class — never 2D block-wise.
                if Bs.numel() > 1:
                    Bs = self._pad_to_alignment(
                        Bs.view(-1), dim=0, alignment=16, value=1.0
                    )
                    if Bs.dim() == 1 and B.shape[1] > 1:
                        Bs = Bs.view(-1, 1)

        output = ops.cutlass_scaled_mm(
            A, B, out_dtype=out_dtype, scale_a=As, scale_b=Bs, bias=bias
        )

        if pad_n > 0:
            output = output[..., :N].contiguous()

        return output.view(*output_shape)
```
**EN:** This method implements `CutlassFP8ScaledMMLinearKernel.apply_scaled_mm`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `cutlass_scaled_mm`, `view`, `t`, `contiguous`.
**CN:** 该方法 `CutlassFP8ScaledMMLinearKernel.apply_scaled_mm` 封装了此模块中的一段关键运行时逻辑，重点处理 apply scaled mm 相关工作。 它内部会调用 `cutlass_scaled_mm`, `view`, `t`, `contiguous` 等例程。

### Class `CutlassFp8BlockScaledMMKernel` (lines 225-288)
```python
class CutlassFp8BlockScaledMMKernel(Fp8BlockScaledMMLinearKernel):
    def __init__(self, config: FP8ScaledMMLinearLayerConfig) -> None:
        super().__init__(config)
        act_scale_descriptor = config.activation_quant_key.scale
        self.weight_group_shape = config.weight_quant_key.scale.group_shape
        self.quant_fp8 = QuantFP8(
            static=act_scale_descriptor.static,
            group_shape=act_scale_descriptor.group_shape,
            num_token_padding=self.get_output_padding(),
            use_ue8m0=False,
            column_major_scales=True,
        )
        self.is_hopper = current_platform.is_device_capability(90)

    @classmethod
    def is_supported(cls, compute_capability=None):
        if not CUTLASS_BLOCK_FP8_SUPPORTED:
            return (
                False,
                "The device compute capability of"
                f"{compute_capability} is not supported.",
            )
        return True, None

    @classmethod
    def can_implement(cls, config: FP8ScaledMMLinearLayerConfig):
        can_implement_base, reason = super().can_implement(config)
        if not can_implement_base:
            return can_implement_base, reason

        act_quant_desc = config.activation_quant_key.scale
        if act_quant_desc.group_shape != GroupShape(1, 128):
            return (
                False,
                "Supports only dynamic per token group activation "
                "quantization with group_shape=(1,128).",
            )
        return True, None

    def apply_block_scaled_mm(
        self,
        A: torch.Tensor,
        B: torch.Tensor,
        As: torch.Tensor,
        Bs: torch.Tensor,
    ) -> torch.Tensor:
        out_dtype = self.config.out_dtype
        if self.is_hopper:
            return torch.ops.vllm.padded_cutlass(
                A,
                B,
                As,
                Bs,
                list(self.weight_group_shape),
                out_dtype,
            )
        else:
            return ops.cutlass_scaled_mm(
                A,
                B.T,
                out_dtype=out_dtype,
                scale_a=As,
                scale_b=Bs.T,
            )
```
**EN:** This kernel class defines `CutlassFp8BlockScaledMMKernel`. It organizes the state and behavior needed by this kernel path. It inherits from `Fp8BlockScaledMMLinearKernel`. Key methods include `__init__`, `is_supported`, `can_implement`, `apply_block_scaled_mm`.
**CN:** 该内核类定义了 `CutlassFp8BlockScaledMMKernel`。 它组织了该内核路径所需的状态与行为。 它继承自 `Fp8BlockScaledMMLinearKernel`。 关键方法包括 `__init__`, `is_supported`, `can_implement`, `apply_block_scaled_mm`。

### Method `CutlassFp8BlockScaledMMKernel.__init__` (lines 226-237)
```python
    def __init__(self, config: FP8ScaledMMLinearLayerConfig) -> None:
        super().__init__(config)
        act_scale_descriptor = config.activation_quant_key.scale
        self.weight_group_shape = config.weight_quant_key.scale.group_shape
        self.quant_fp8 = QuantFP8(
            static=act_scale_descriptor.static,
            group_shape=act_scale_descriptor.group_shape,
            num_token_padding=self.get_output_padding(),
            use_ue8m0=False,
            column_major_scales=True,
        )
        self.is_hopper = current_platform.is_device_capability(90)
```
**EN:** This method implements `CutlassFp8BlockScaledMMKernel.__init__`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `__init__`, `QuantFP8`, `is_device_capability`, `super`.
**CN:** 该方法 `CutlassFp8BlockScaledMMKernel.__init__` 封装了此模块中的一段关键运行时逻辑，重点处理 init 相关工作。 它内部会调用 `__init__`, `QuantFP8`, `is_device_capability`, `super` 等例程。

### Method `CutlassFp8BlockScaledMMKernel.is_supported` (lines 240-247)
```python
    def is_supported(cls, compute_capability=None):
        if not CUTLASS_BLOCK_FP8_SUPPORTED:
            return (
                False,
                "The device compute capability of"
                f"{compute_capability} is not supported.",
            )
        return True, None
```
**EN:** This method implements `CutlassFp8BlockScaledMMKernel.is_supported`, encapsulating a focused unit of runtime logic for the module.
**CN:** 该方法 `CutlassFp8BlockScaledMMKernel.is_supported` 封装了此模块中的一段关键运行时逻辑，重点处理 is supported 相关工作。

### Method `CutlassFp8BlockScaledMMKernel.can_implement` (lines 250-262)
```python
    def can_implement(cls, config: FP8ScaledMMLinearLayerConfig):
        can_implement_base, reason = super().can_implement(config)
        if not can_implement_base:
            return can_implement_base, reason

        act_quant_desc = config.activation_quant_key.scale
        if act_quant_desc.group_shape != GroupShape(1, 128):
            return (
                False,
                "Supports only dynamic per token group activation "
                "quantization with group_shape=(1,128).",
            )
        return True, None
```
**EN:** This method implements `CutlassFp8BlockScaledMMKernel.can_implement`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `can_implement`, `GroupShape`, `super`.
**CN:** 该方法 `CutlassFp8BlockScaledMMKernel.can_implement` 封装了此模块中的一段关键运行时逻辑，重点处理 can implement 相关工作。 它内部会调用 `can_implement`, `GroupShape`, `super` 等例程。

### Method `CutlassFp8BlockScaledMMKernel.apply_block_scaled_mm` (lines 264-288)
```python
    def apply_block_scaled_mm(
        self,
        A: torch.Tensor,
        B: torch.Tensor,
        As: torch.Tensor,
        Bs: torch.Tensor,
    ) -> torch.Tensor:
        out_dtype = self.config.out_dtype
        if self.is_hopper:
            return torch.ops.vllm.padded_cutlass(
                A,
                B,
                As,
                Bs,
                list(self.weight_group_shape),
                out_dtype,
            )
        else:
            return ops.cutlass_scaled_mm(
                A,
                B.T,
                out_dtype=out_dtype,
                scale_a=As,
                scale_b=Bs.T,
            )
```
**EN:** This method implements `CutlassFp8BlockScaledMMKernel.apply_block_scaled_mm`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `padded_cutlass`, `cutlass_scaled_mm`, `list`.
**CN:** 该方法 `CutlassFp8BlockScaledMMKernel.apply_block_scaled_mm` 封装了此模块中的一段关键运行时逻辑，重点处理 apply block scaled mm 相关工作。 它内部会调用 `padded_cutlass`, `cutlass_scaled_mm`, `list` 等例程。

### Function `cutlass_scaled_mm` (lines 291-305)
```python
def cutlass_scaled_mm(
    A: torch.Tensor,
    B: torch.Tensor,
    As: torch.Tensor,
    Bs: torch.Tensor,
    block_size: list[int],
    output_dtype: torch.dtype = torch.float16,
) -> torch.Tensor:
    return ops.cutlass_scaled_mm(
        A,
        B.T,
        out_dtype=output_dtype,
        scale_a=As,
        scale_b=Bs.T,
    )
```
**EN:** This helper implements `cutlass_scaled_mm`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `cutlass_scaled_mm`.
**CN:** 该函数 `cutlass_scaled_mm` 封装了此模块中的一段关键运行时逻辑，重点处理 cutlass scaled mm 相关工作。 它内部会调用 `cutlass_scaled_mm` 等例程。

### Function `_padded_cutlass` (lines 308-342)
```python
def _padded_cutlass(
    qx: torch.Tensor,
    weight: torch.Tensor,
    x_scale: torch.Tensor,
    weight_scale: torch.Tensor,
    block_size: list[int],
    output_dtype: torch.dtype,
) -> torch.Tensor:
    pad_multiple = 4
    dim = qx.shape[0]
    padded = (
        dim if dim % pad_multiple == 0 else dim + pad_multiple - (dim % pad_multiple)
    )

    has_pad = padded > dim

    if has_pad:
        padded_shape = [padded, *qx.shape[1:]]
        padded_qx = torch.zeros(padded_shape, device=qx.device, dtype=qx.dtype)
        padded_qx[0 : qx.shape[0], ...].copy_(qx)

        padded_x_scale_shape = [*x_scale.shape[1:], padded]
        padded_x_scale = torch.ones(
            padded_x_scale_shape, device=x_scale.device, dtype=x_scale.dtype
        ).permute(-1, -2)
        padded_x_scale[0 : x_scale.shape[0], ...].copy_(x_scale)

        output = cutlass_scaled_mm(
            padded_qx, weight, padded_x_scale, weight_scale, block_size, output_dtype
        )
        return output[0 : qx.shape[0], ...]
    else:
        return cutlass_scaled_mm(
            qx, weight, x_scale, weight_scale, block_size, output_dtype
        )
```
**EN:** This helper implements `_padded_cutlass`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `zeros`, `copy_`, `permute`, `cutlass_scaled_mm`.
**CN:** 该函数 `_padded_cutlass` 封装了此模块中的一段关键运行时逻辑，重点处理 padded cutlass 相关工作。 它内部会调用 `zeros`, `copy_`, `permute`, `cutlass_scaled_mm` 等例程。

### Function `_padded_cutlass_fake` (lines 345-355)
```python
def _padded_cutlass_fake(
    qx: torch.Tensor,
    weight: torch.Tensor,
    x_scale: torch.Tensor,
    weight_scale: torch.Tensor,
    block_size: list[int],
    output_dtype: torch.dtype,
) -> torch.Tensor:
    return torch.empty(
        (qx.size(0), weight.size(0)), dtype=output_dtype, device=qx.device
    )
```
**EN:** This helper implements `_padded_cutlass_fake`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `empty`, `size`.
**CN:** 该函数 `_padded_cutlass_fake` 封装了此模块中的一段关键运行时逻辑，重点处理 padded cutlass fake 相关工作。 它内部会调用 `empty`, `size` 等例程。

### Top-level block (lines 358-362)
```python
direct_register_custom_op(
    "padded_cutlass",
    _padded_cutlass,
    fake_impl=_padded_cutlass_fake,
)
```
**EN:** This top-level control-flow block coordinates module initialization before the main kernels are used.
**CN:** 这个顶层控制流代码块在主内核使用前协调模块初始化流程。

## Key Concepts / 关键概念
- **Triton kernels / Triton 内核**
  - **EN:** The implementation relies on Triton-generated GPU kernels for low-level tensor work.
  - **CN:** 该实现依赖 Triton 生成的 GPU 内核完成底层张量计算。
- **CUTLASS backend / CUTLASS 后端**
  - **EN:** The module selects or wraps CUTLASS kernels for NVIDIA-oriented matrix multiplication.
  - **CN:** 该模块选择或封装 CUTLASS 内核，以支持面向 NVIDIA 的矩阵乘计算。
- **FP8 quantization / FP8 量化**
  - **EN:** The module handles FP8 data, scales, or kernels for low-precision inference.
  - **CN:** 该模块处理 FP8 数据、缩放因子或相关低精度推理内核。
- **INT8 quantization / INT8 量化**
  - **EN:** The implementation contains INT8 scaling, packing, or matrix-multiplication logic.
  - **CN:** 该实现包含 INT8 的缩放、打包或矩阵乘逻辑。
- **Scaled matrix multiplication / 缩放矩阵乘**
  - **EN:** The code centers on scaled matrix multiplication kernels and their dispatch rules.
  - **CN:** 代码围绕缩放矩阵乘内核及其分派规则展开。

## Dependencies / 依赖关系
- **vLLM internal / vLLM 内部依赖**: `vllm`, `vllm.model_executor.layers.quantization.input_quant_fp8`, `vllm.model_executor.layers.quantization.utils`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.model_executor.layers.quantization.utils.w8a8_utils`, `vllm.platforms`, `vllm.utils.torch_utils`, `.BlockScaledMMLinearKernel`, `.ScaledMMLinearKernel`
- **External / 外部依赖**: `torch`
