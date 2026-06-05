# triton.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/kernels/linear/scaled_mm/triton.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the Triton scaled matrix-multiplication kernel path. / 实现 Triton 的缩放矩阵乘内核路径。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 5-24)
```python
import torch

from vllm import _custom_ops as ops
from vllm.model_executor.layers.quantization.compressed_tensors.triton_scaled_mm import (  # noqa: E501
    triton_scaled_mm,
)
from vllm.model_executor.layers.quantization.utils import replace_parameter
from vllm.model_executor.layers.quantization.utils.w8a8_utils import (
    convert_to_channelwise,
)
from vllm.platforms import current_platform
from vllm.utils.torch_utils import direct_register_custom_op

from .BlockScaledMMLinearKernel import (
    Fp8BlockScaledMMLinearKernel,
)
from .cutlass import CutlassInt8ScaledMMLinearKernel
from .ScaledMMLinearKernel import (
    Int8ScaledMMLinearLayerConfig,
)
```
**EN:** This import block loads `torch`, `vllm`, `vllm.model_executor.layers.quantization.compressed_tensors.triton_scaled_mm`, `vllm.model_executor.layers.quantization.utils`, `vllm.model_executor.layers.quantization.utils.w8a8_utils`, `vllm.platforms`, ..., establishing the tensor libraries, backend hooks, and vLLM helpers needed by the rest of `triton.py`.
**CN:** 该导入代码块加载了 `torch`, `vllm`, `vllm.model_executor.layers.quantization.compressed_tensors.triton_scaled_mm`, `vllm.model_executor.layers.quantization.utils`, `vllm.model_executor.layers.quantization.utils.w8a8_utils`, `vllm.platforms`, ...，为 `triton.py` 后续逻辑准备张量库、后端钩子以及 vLLM 辅助工具。

### Class `TritonInt8ScaledMMLinearKernel` (lines 27-156)
```python
class TritonInt8ScaledMMLinearKernel(CutlassInt8ScaledMMLinearKernel):
    @classmethod
    def is_supported(
        cls, compute_capability: int | None = None
    ) -> tuple[bool, str | None]:
        if current_platform.is_cuda_alike():
            return True, None
        return False, "requires ROCm or CUDA."

    @classmethod
    def can_implement(cls, c: Int8ScaledMMLinearLayerConfig) -> tuple[bool, str | None]:
        return True, None

    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        w_q, _, i_s, _, _ = self._get_layer_params(layer)
        w_q_name, w_s_name, i_s_name, i_zp_name, azp_adj_name = self.layer_param_names

        replace_parameter(
            layer,
            w_q_name,
            torch.nn.Parameter(w_q.t().data, requires_grad=False),
        )

        # WEIGHT SCALE
        # Triton kernel supports only per-tensor and per-channel.
        # If we have a fused module (QKV, MLP) with per tensor scales (thus N
        # scales being passed to the kernel), convert to the per-channel case.
        is_fused_module = len(layer.logical_widths) > 1
        weight_scale = getattr(layer, w_s_name)
        if is_fused_module and not self.config.is_channelwise:
            weight_scale = convert_to_channelwise(weight_scale, layer.logical_widths)
        replace_parameter(
            layer,
            w_s_name,
            torch.nn.Parameter(weight_scale.data, requires_grad=False),
        )

        # INPUT SCALE
        if self.config.is_static_input_scheme:
            assert i_s is not None

            if self.config.input_symmetric:
                replace_parameter(
                    layer,
                    i_s_name,
                    torch.nn.Parameter(i_s.max(), requires_grad=False),
                )
                setattr(layer, i_zp_name, None)
            else:
                input_zero_point = getattr(layer, i_zp_name)

                # Reconstruct the ranges to find a single scale and azp
                int8_traits = torch.iinfo(torch.int8)
                azps = input_zero_point.to(dtype=torch.int32)
                range_max = (i_s * (int8_traits.max - azps)).max()
                range_min = (i_s * (int8_traits.min - azps)).min()

                scale = (range_max - range_min) / (int8_traits.max - int8_traits.min)
                replace_parameter(
                    layer,
                    i_s_name,
                    torch.nn.Parameter(scale, requires_grad=False),
                )

                # AZP loaded as int8 but used as int32
                azp = (int8_traits.min - range_min / scale).to(dtype=torch.int32)
                replace_parameter(
                    layer,
                    i_zp_name,
                    torch.nn.Parameter(azp, requires_grad=False),
                )
        else:
            setattr(layer, i_s_name, None)
            setattr(layer, i_zp_name, None)

        # azp_adj is the AZP adjustment term, used to account for weights.
        # It does not depend on scales or azp, so it is the same for
        # static and dynamic quantization.
        # See csrc/quantization/w8a8/cutlass/Epilogues.md for the math.
        if not self.config.input_symmetric:
            weight = getattr(layer, w_q_name)
            # weight is already transposed to [K, N], sum over K (dim=0)
            azp_adj = weight.sum(dim=0, keepdim=True, dtype=torch.int32)
            if self.config.is_static_input_scheme:
                # Fold azp into azp_adj for the per-tensor case
                azp_adj = getattr(layer, i_zp_name) * azp_adj
            setattr(
                layer,
                azp_adj_name,
                torch.nn.Parameter(azp_adj, requires_grad=False),
            )
        else:
            setattr(layer, azp_adj_name, None)

    def apply_weights(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        w_q, w_s, i_s, i_zp, azp_adj = self._get_layer_params(layer)

        symmetric = azp_adj is None
        x_q, x_s, x_zp = ops.scaled_int8_quant(
            x.contiguous(), i_s, i_zp, symmetric=symmetric
        )

        out = triton_scaled_mm(
            x_q, w_q, scale_a=x_s, scale_b=w_s, out_dtype=x.dtype, bias=bias
        )

        if azp_adj is not None:
            # Asymmetric quantization: subtract the zero-point correction.
            # D = scale_a * scale_b * (A_q @ B_q - azp * azp_adj) + bias
            # triton_scaled_mm already computed scale_a * scale_b * (A_q @ B_q) + bias
            # so we subtract scale_a * scale_b * azp * azp_adj
            #
            # x_s: [M, 1] or scalar, w_s: [N, 1] or scalar, azp_adj: [1, N]
            # Reshape w_s from [N, 1] to [1, N] for proper broadcasting.
            w_s_row = w_s.view(1, -1) if w_s.dim() > 0 else w_s
            static = i_zp is not None
            if not static and x_zp is not None:
                # Dynamic per-token: azp is per-token, azp_adj is per-channel
                # x_zp: [M, 1], azp_adj: [1, N]
                out -= x_s * w_s_row * (x_zp * azp_adj).to(x.dtype)
            else:
                # Static per-tensor: azp already folded into azp_adj
                out -= (x_s * w_s_row * azp_adj).to(x.dtype)

        return out
```
**EN:** This kernel class defines `TritonInt8ScaledMMLinearKernel`. It organizes the state and behavior needed by this kernel path. It inherits from `CutlassInt8ScaledMMLinearKernel`. Key methods include `is_supported`, `can_implement`, `process_weights_after_loading`, `apply_weights`.
**CN:** 该内核类定义了 `TritonInt8ScaledMMLinearKernel`。 它组织了该内核路径所需的状态与行为。 它继承自 `CutlassInt8ScaledMMLinearKernel`。 关键方法包括 `is_supported`, `can_implement`, `process_weights_after_loading`, `apply_weights`。

### Method `TritonInt8ScaledMMLinearKernel.is_supported` (lines 29-34)
```python
    def is_supported(
        cls, compute_capability: int | None = None
    ) -> tuple[bool, str | None]:
        if current_platform.is_cuda_alike():
            return True, None
        return False, "requires ROCm or CUDA."
```
**EN:** This method implements `TritonInt8ScaledMMLinearKernel.is_supported`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `is_cuda_alike`.
**CN:** 该方法 `TritonInt8ScaledMMLinearKernel.is_supported` 封装了此模块中的一段关键运行时逻辑，重点处理 is supported 相关工作。 它内部会调用 `is_cuda_alike` 等例程。

### Method `TritonInt8ScaledMMLinearKernel.can_implement` (lines 37-38)
```python
    def can_implement(cls, c: Int8ScaledMMLinearLayerConfig) -> tuple[bool, str | None]:
        return True, None
```
**EN:** This method implements `TritonInt8ScaledMMLinearKernel.can_implement`, encapsulating a focused unit of runtime logic for the module.
**CN:** 该方法 `TritonInt8ScaledMMLinearKernel.can_implement` 封装了此模块中的一段关键运行时逻辑，重点处理 can implement 相关工作。

### Method `TritonInt8ScaledMMLinearKernel.process_weights_after_loading` (lines 40-119)
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        w_q, _, i_s, _, _ = self._get_layer_params(layer)
        w_q_name, w_s_name, i_s_name, i_zp_name, azp_adj_name = self.layer_param_names

        replace_parameter(
            layer,
            w_q_name,
            torch.nn.Parameter(w_q.t().data, requires_grad=False),
        )

        # WEIGHT SCALE
        # Triton kernel supports only per-tensor and per-channel.
        # If we have a fused module (QKV, MLP) with per tensor scales (thus N
        # scales being passed to the kernel), convert to the per-channel case.
        is_fused_module = len(layer.logical_widths) > 1
        weight_scale = getattr(layer, w_s_name)
        if is_fused_module and not self.config.is_channelwise:
            weight_scale = convert_to_channelwise(weight_scale, layer.logical_widths)
        replace_parameter(
            layer,
            w_s_name,
            torch.nn.Parameter(weight_scale.data, requires_grad=False),
        )

        # INPUT SCALE
        if self.config.is_static_input_scheme:
            assert i_s is not None

            if self.config.input_symmetric:
                replace_parameter(
                    layer,
                    i_s_name,
                    torch.nn.Parameter(i_s.max(), requires_grad=False),
                )
                setattr(layer, i_zp_name, None)
            else:
                input_zero_point = getattr(layer, i_zp_name)

                # Reconstruct the ranges to find a single scale and azp
                int8_traits = torch.iinfo(torch.int8)
                azps = input_zero_point.to(dtype=torch.int32)
                range_max = (i_s * (int8_traits.max - azps)).max()
                range_min = (i_s * (int8_traits.min - azps)).min()

                scale = (range_max - range_min) / (int8_traits.max - int8_traits.min)
                replace_parameter(
                    layer,
                    i_s_name,
                    torch.nn.Parameter(scale, requires_grad=False),
                )

                # AZP loaded as int8 but used as int32
                azp = (int8_traits.min - range_min / scale).to(dtype=torch.int32)
                replace_parameter(
                    layer,
                    i_zp_name,
                    torch.nn.Parameter(azp, requires_grad=False),
                )
        else:
            setattr(layer, i_s_name, None)
            setattr(layer, i_zp_name, None)

        # azp_adj is the AZP adjustment term, used to account for weights.
        # It does not depend on scales or azp, so it is the same for
        # static and dynamic quantization.
        # See csrc/quantization/w8a8/cutlass/Epilogues.md for the math.
        if not self.config.input_symmetric:
            weight = getattr(layer, w_q_name)
            # weight is already transposed to [K, N], sum over K (dim=0)
            azp_adj = weight.sum(dim=0, keepdim=True, dtype=torch.int32)
            if self.config.is_static_input_scheme:
                # Fold azp into azp_adj for the per-tensor case
                azp_adj = getattr(layer, i_zp_name) * azp_adj
            setattr(
                layer,
                azp_adj_name,
                torch.nn.Parameter(azp_adj, requires_grad=False),
            )
        else:
            setattr(layer, azp_adj_name, None)
```
**EN:** This method implements `TritonInt8ScaledMMLinearKernel.process_weights_after_loading`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `_get_layer_params`, `replace_parameter`, `getattr`, `Parameter`.
**CN:** 该方法 `TritonInt8ScaledMMLinearKernel.process_weights_after_loading` 封装了此模块中的一段关键运行时逻辑，重点处理 process weights after loading 相关工作。 它内部会调用 `_get_layer_params`, `replace_parameter`, `getattr`, `Parameter` 等例程。

### Method `TritonInt8ScaledMMLinearKernel.apply_weights` (lines 121-156)
```python
    def apply_weights(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        w_q, w_s, i_s, i_zp, azp_adj = self._get_layer_params(layer)

        symmetric = azp_adj is None
        x_q, x_s, x_zp = ops.scaled_int8_quant(
            x.contiguous(), i_s, i_zp, symmetric=symmetric
        )

        out = triton_scaled_mm(
            x_q, w_q, scale_a=x_s, scale_b=w_s, out_dtype=x.dtype, bias=bias
        )

        if azp_adj is not None:
            # Asymmetric quantization: subtract the zero-point correction.
            # D = scale_a * scale_b * (A_q @ B_q - azp * azp_adj) + bias
            # triton_scaled_mm already computed scale_a * scale_b * (A_q @ B_q) + bias
            # so we subtract scale_a * scale_b * azp * azp_adj
            #
            # x_s: [M, 1] or scalar, w_s: [N, 1] or scalar, azp_adj: [1, N]
            # Reshape w_s from [N, 1] to [1, N] for proper broadcasting.
            w_s_row = w_s.view(1, -1) if w_s.dim() > 0 else w_s
            static = i_zp is not None
            if not static and x_zp is not None:
                # Dynamic per-token: azp is per-token, azp_adj is per-channel
                # x_zp: [M, 1], azp_adj: [1, N]
                out -= x_s * w_s_row * (x_zp * azp_adj).to(x.dtype)
            else:
                # Static per-tensor: azp already folded into azp_adj
                out -= (x_s * w_s_row * azp_adj).to(x.dtype)

        return out
```
**EN:** This method implements `TritonInt8ScaledMMLinearKernel.apply_weights`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `_get_layer_params`, `scaled_int8_quant`, `triton_scaled_mm`, `contiguous`.
**CN:** 该方法 `TritonInt8ScaledMMLinearKernel.apply_weights` 封装了此模块中的一段关键运行时逻辑，重点处理 apply weights 相关工作。 它内部会调用 `_get_layer_params`, `scaled_int8_quant`, `triton_scaled_mm`, `contiguous` 等例程。

### Class `TritonFp8BlockScaledMMKernel` (lines 159-180)
```python
class TritonFp8BlockScaledMMKernel(Fp8BlockScaledMMLinearKernel):
    @classmethod
    def is_supported(cls, compute_capability=None):
        if not current_platform.is_cuda_alike():
            return False, "only cuda like devices are supported."
        return True, None

    def apply_block_scaled_mm(
        self,
        A: torch.Tensor,
        B: torch.Tensor,
        As: torch.Tensor,
        Bs: torch.Tensor,
    ) -> torch.Tensor:
        return torch.ops.vllm.w8a8_triton_block_scaled_mm_func(
            A,
            B,
            As,
            Bs,
            list(self.weight_group_shape),
            self.config.out_dtype,
        )
```
**EN:** This kernel class defines `TritonFp8BlockScaledMMKernel`. It organizes the state and behavior needed by this kernel path. It inherits from `Fp8BlockScaledMMLinearKernel`. Key methods include `is_supported`, `apply_block_scaled_mm`.
**CN:** 该内核类定义了 `TritonFp8BlockScaledMMKernel`。 它组织了该内核路径所需的状态与行为。 它继承自 `Fp8BlockScaledMMLinearKernel`。 关键方法包括 `is_supported`, `apply_block_scaled_mm`。

### Method `TritonFp8BlockScaledMMKernel.is_supported` (lines 161-164)
```python
    def is_supported(cls, compute_capability=None):
        if not current_platform.is_cuda_alike():
            return False, "only cuda like devices are supported."
        return True, None
```
**EN:** This method implements `TritonFp8BlockScaledMMKernel.is_supported`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `is_cuda_alike`.
**CN:** 该方法 `TritonFp8BlockScaledMMKernel.is_supported` 封装了此模块中的一段关键运行时逻辑，重点处理 is supported 相关工作。 它内部会调用 `is_cuda_alike` 等例程。

### Method `TritonFp8BlockScaledMMKernel.apply_block_scaled_mm` (lines 166-180)
```python
    def apply_block_scaled_mm(
        self,
        A: torch.Tensor,
        B: torch.Tensor,
        As: torch.Tensor,
        Bs: torch.Tensor,
    ) -> torch.Tensor:
        return torch.ops.vllm.w8a8_triton_block_scaled_mm_func(
            A,
            B,
            As,
            Bs,
            list(self.weight_group_shape),
            self.config.out_dtype,
        )
```
**EN:** This method implements `TritonFp8BlockScaledMMKernel.apply_block_scaled_mm`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `w8a8_triton_block_scaled_mm_func`, `list`.
**CN:** 该方法 `TritonFp8BlockScaledMMKernel.apply_block_scaled_mm` 封装了此模块中的一段关键运行时逻辑，重点处理 apply block scaled mm 相关工作。 它内部会调用 `w8a8_triton_block_scaled_mm_func`, `list` 等例程。

### Function `_w8a8_triton_block_scaled_mm_func` (lines 186-200)
```python
def _w8a8_triton_block_scaled_mm_func(
    qx: torch.Tensor,
    weight: torch.Tensor,
    x_scale: torch.Tensor,
    weight_scale: torch.Tensor,
    block_size: list[int],
    output_dtype: torch.dtype,
) -> torch.Tensor:
    from vllm.model_executor.layers.quantization.utils.fp8_utils import (
        w8a8_triton_block_scaled_mm,
    )

    return w8a8_triton_block_scaled_mm(
        qx, weight, x_scale, weight_scale, block_size, output_dtype
    )
```
**EN:** This helper implements `_w8a8_triton_block_scaled_mm_func`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `w8a8_triton_block_scaled_mm`.
**CN:** 该函数 `_w8a8_triton_block_scaled_mm_func` 封装了此模块中的一段关键运行时逻辑，重点处理 w 8 a 8 triton block scaled mm func 相关工作。 它内部会调用 `w8a8_triton_block_scaled_mm` 等例程。

### Function `_w8a8_triton_block_scaled_mm_fake` (lines 203-213)
```python
def _w8a8_triton_block_scaled_mm_fake(
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
**EN:** This helper implements `_w8a8_triton_block_scaled_mm_fake`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `empty`, `size`.
**CN:** 该函数 `_w8a8_triton_block_scaled_mm_fake` 封装了此模块中的一段关键运行时逻辑，重点处理 w 8 a 8 triton block scaled mm fake 相关工作。 它内部会调用 `empty`, `size` 等例程。

### Top-level block (lines 216-220)
```python
direct_register_custom_op(
    "w8a8_triton_block_scaled_mm_func",
    _w8a8_triton_block_scaled_mm_func,
    fake_impl=_w8a8_triton_block_scaled_mm_fake,
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
- **ROCm support / ROCm 支持**
  - **EN:** The implementation targets ROCm-capable devices and their kernel constraints.
  - **CN:** 该实现面向 ROCm 设备及其内核约束。
- **FP8 quantization / FP8 量化**
  - **EN:** The module handles FP8 data, scales, or kernels for low-precision inference.
  - **CN:** 该模块处理 FP8 数据、缩放因子或相关低精度推理内核。
- **INT8 quantization / INT8 量化**
  - **EN:** The implementation contains INT8 scaling, packing, or matrix-multiplication logic.
  - **CN:** 该实现包含 INT8 的缩放、打包或矩阵乘逻辑。

## Dependencies / 依赖关系
- **vLLM internal / vLLM 内部依赖**: `vllm`, `vllm.model_executor.layers.quantization.compressed_tensors.triton_scaled_mm`, `vllm.model_executor.layers.quantization.utils`, `vllm.model_executor.layers.quantization.utils.w8a8_utils`, `vllm.platforms`, `vllm.utils.torch_utils`, `.BlockScaledMMLinearKernel`, `.cutlass`, `.ScaledMMLinearKernel`
- **External / 外部依赖**: `torch`
