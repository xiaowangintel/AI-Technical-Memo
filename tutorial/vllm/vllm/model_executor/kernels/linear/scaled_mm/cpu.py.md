# cpu.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/kernels/linear/scaled_mm/cpu.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the CPU scaled matrix-multiplication kernel path. / 实现 CPU 的缩放矩阵乘内核路径。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 5-24)
```python
import torch

from vllm import _custom_ops as ops
from vllm import envs
from vllm.model_executor.layers.quantization.utils import replace_parameter
from vllm.model_executor.layers.quantization.utils.w8a8_utils import (
    convert_to_channelwise,
)
from vllm.model_executor.layers.utils import check_cpu_sgl_kernel
from vllm.platforms import current_platform
from vllm.platforms.interface import CpuArchEnum

from .BlockScaledMMLinearKernel import (
    Fp8BlockScaledMMLinearKernel,
    FP8ScaledMMLinearLayerConfig,
)
from .ScaledMMLinearKernel import (
    Int8ScaledMMLinearKernel,
    Int8ScaledMMLinearLayerConfig,
)
```
**EN:** This import block loads `torch`, `vllm`, `vllm.model_executor.layers.quantization.utils`, `vllm.model_executor.layers.quantization.utils.w8a8_utils`, `vllm.model_executor.layers.utils`, `vllm.platforms`, ..., establishing the tensor libraries, backend hooks, and vLLM helpers needed by the rest of `cpu.py`.
**CN:** 该导入代码块加载了 `torch`, `vllm`, `vllm.model_executor.layers.quantization.utils`, `vllm.model_executor.layers.quantization.utils.w8a8_utils`, `vllm.model_executor.layers.utils`, `vllm.platforms`, ...，为 `cpu.py` 后续逻辑准备张量库、后端钩子以及 vLLM 辅助工具。

### Class `CPUInt8ScaledMMLinearKernel` (lines 27-221)
```python
class CPUInt8ScaledMMLinearKernel(Int8ScaledMMLinearKernel):
    @classmethod
    def is_supported(
        cls, compute_capability: int | None = None
    ) -> tuple[bool, str | None]:
        if not current_platform.is_cpu():
            return False, "requires CPU."
        return True, None

    @classmethod
    def can_implement(cls, c: Int8ScaledMMLinearLayerConfig) -> tuple[bool, str | None]:
        return True, None

    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        w_q_name, _, _, _, _ = self.layer_param_names
        weight = getattr(layer, w_q_name)
        dtype = weight.dtype
        N, K = weight.size()
        if (
            current_platform.get_cpu_architecture() == CpuArchEnum.X86
            and envs.VLLM_CPU_SGL_KERNEL
            and self.config.input_symmetric
            and check_cpu_sgl_kernel(N, K, dtype)
        ):
            self.linear_method = self._apply_weights_sgl
            self.process_weights_for_sgl(layer)
        else:
            self.linear_method = self._apply_weights_onednn
            self.process_weights_for_onednn(layer)

    def process_weights_for_onednn(self, layer: torch.nn.Module) -> None:
        # WEIGHT
        # Transpose to [K, N] for convenience
        w_q_name, w_s_name, i_s_name, i_zp_name, azp_adj_name = self.layer_param_names
        weight = getattr(layer, w_q_name)
        replace_parameter(
            layer,
            w_q_name,
            torch.nn.Parameter(weight.t().data, requires_grad=False),
        )

        # WEIGHT SCALE
        # oneDNN kernels support only per-tensor and per-channel.
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
            input_scale = getattr(layer, i_s_name)

            if self.config.input_symmetric:
                replace_parameter(
                    layer,
                    i_s_name,
                    torch.nn.Parameter(input_scale.max(), requires_grad=False),
                )
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

                azp = (
                    (int8_traits.min - range_min / scale).round().to(dtype=torch.int32)
                )
                replace_parameter(
                    layer, i_zp_name, torch.nn.Parameter(azp, requires_grad=False)
                )

        # Different from cutlass, oneDNN kernels only need the AZP adjustment
        # term for dynamic quantization. And s_b should be folded into the
        # term. Such as:
        # s_a * s_b * [(A - zp_a)B] + bias =
        # s_a * (s_b * AB) - s_a * s_b * zp_a * B + bias =
        # s_a * GEMM_output - s_a * zp_a * adj + bias
        if not (self.config.input_symmetric and self.config.is_static_input_scheme):
            weight = getattr(layer, w_q_name)
            weight_scale = getattr(layer, w_s_name)
            azp_adj = weight.sum(dim=0, keepdim=True, dtype=torch.float32)
            azp_adj = azp_adj * weight_scale.squeeze()
            setattr(
                layer,
                azp_adj_name,
                torch.nn.Parameter(azp_adj, requires_grad=False),
            )

        weight = getattr(layer, w_q_name)
        self.dnnl_handler = ops.create_onednn_scaled_mm(
            weight,
            getattr(layer, w_s_name),
            torch.get_default_dtype(),
            getattr(layer, i_s_name) is None,
            not self.config.input_symmetric,
            32,
        )
        # weight is prepacked and maintained by the dnnl_handler,
        # release the original weight
        setattr(layer, w_q_name, None)
        del weight

    def process_weights_for_sgl(self, layer: torch.nn.Module) -> None:
        w_q_name, w_s_name, _, _, _ = self.layer_param_names
        # WEIGHT
        weight = getattr(layer, w_q_name)
        packed_weight = torch.ops._C.convert_weight_packed(weight)
        replace_parameter(
            layer, w_q_name, torch.nn.Parameter(packed_weight, requires_grad=False)
        )

        if layer.bias is not None:
            bias = layer.bias
            layer.register_parameter(
                "bias_fp32", torch.nn.Parameter(bias.float().data, requires_grad=False)
            )

        # WEIGHT SCALE
        # CPU SGL kernels only support per-channel.
        # For per-tensor quant, convert to the per-channel case.
        weight_scale = getattr(layer, w_s_name)
        if not self.config.is_channelwise:
            weight_scale = convert_to_channelwise(weight_scale, layer.logical_widths)
        replace_parameter(
            layer,
            w_s_name,
            torch.nn.Parameter(weight_scale.data, requires_grad=False),
        )

    def apply_weights(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        return self.linear_method(
            layer,
            x,
            bias,
        )

    def _apply_weights_onednn(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        x_shape = x.shape
        x = x.reshape(-1, x_shape[-1]) if len(x_shape) > 2 else x
        w_q, w_s, i_s, i_zp, azp_adj = self._get_layer_params(layer)

        # ops.scaled_int8_quant supports both dynamic and static quant:
        # * dynamic, i_s is None and x_s computed from x.
        # * static, i_s is scalar and x_s is i_s.
        x_q, x_s, x_zp = ops.onednn_scaled_int8_quant(
            x, i_s, i_zp, self.config.input_symmetric
        )

        m = x.size(0)
        n = self.dnnl_handler.n
        out = torch.empty((m, n), dtype=x.dtype)
        ops.onednn_scaled_mm(self.dnnl_handler, x_q, out, x_s, x_zp, azp_adj, bias)
        out = out.reshape(x_shape[:-1] + (n,)) if len(x_shape) > 2 else out
        return out

    def _apply_weights_sgl(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        w_q, w_s, _, _, _ = self._get_layer_params(layer)
        return torch.ops._C.int8_scaled_mm_with_quant(
            x,
            w_q,
            w_s,
            layer.bias_fp32 if bias is not None else None,
            x.dtype,
            True,
        )
```
**EN:** This kernel class defines `CPUInt8ScaledMMLinearKernel`. It organizes the state and behavior needed by this kernel path. It inherits from `Int8ScaledMMLinearKernel`. Key methods include `is_supported`, `can_implement`, `process_weights_after_loading`, `process_weights_for_onednn`, `process_weights_for_sgl`, ....
**CN:** 该内核类定义了 `CPUInt8ScaledMMLinearKernel`。 它组织了该内核路径所需的状态与行为。 它继承自 `Int8ScaledMMLinearKernel`。 关键方法包括 `is_supported`, `can_implement`, `process_weights_after_loading`, `process_weights_for_onednn`, `process_weights_for_sgl`, ...。

### Class `CPUFp8BlockScaledMMKernel` (lines 224-327)
```python
class CPUFp8BlockScaledMMKernel(Fp8BlockScaledMMLinearKernel):
    """FP8 W8A16 block-quantized GEMM via AMX BRGEMM on CPU."""

    # Input stays BF16 — no FP8 activation quantization.
    apply_input_quant = False

    @classmethod
    def is_supported(
        cls, compute_capability: int | None = None
    ) -> tuple[bool, str | None]:
        if not current_platform.is_cpu():
            return False, "requires CPU platform."
        if not torch.cpu._is_amx_tile_supported():
            return False, "requires AMX tile support (Sapphire Rapids or newer)."
        if not ops._supports_cpu_fp8_w8a16:
            return False, "fp8_scaled_mm_cpu op not available."
        return True, None

    @classmethod
    def can_implement(
        cls, config: FP8ScaledMMLinearLayerConfig
    ) -> tuple[bool, str | None]:
        # Validate weight block shape
        weight_gs = config.weight_quant_key.scale.group_shape
        if weight_gs.col <= 0 or weight_gs.col != 128:
            return False, (
                "CPU FP8 kernel requires K-dimension block size of 128, "
                f"got {weight_gs.col}."
            )
        if weight_gs.row <= 0 or weight_gs.row % 32 != 0:
            return False, (
                "CPU FP8 kernel requires N-dimension block size to be "
                f"a positive multiple of 32, got {weight_gs.row}."
            )
        if config.out_dtype not in (torch.bfloat16, torch.float32):
            return False, "Only bfloat16/float32 output dtype supported."
        return True, None

    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        # Skip the base class process (FP8 padding / fnuz normalization)
        # which is GPU-oriented.  Instead, VNNI-prepack weights for AMX.
        params = self._get_layer_params(layer)
        packed_weight = torch.ops._C.convert_weight_packed(params.weight)
        replace_parameter(
            layer,
            params.WEIGHT,
            torch.nn.Parameter(packed_weight, requires_grad=False),
        )

        # Re-wrap scale as a plain Parameter so the kernel can read it
        # without weight-loader metadata interfering.
        scale_attr = (
            params.WEIGHT_SCALE_INV
            if params.weight_scale_inv is not None
            else params.WEIGHT_SCALE
        )
        weight_scale = (
            params.weight_scale_inv
            if params.weight_scale_inv is not None
            else params.weight_scale
        )
        assert weight_scale is not None
        replace_parameter(
            layer,
            scale_attr,
            torch.nn.Parameter(weight_scale.data, requires_grad=False),
        )

    def apply_weights(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
        **kwargs,
    ) -> torch.Tensor:
        params = self._get_layer_params(layer)
        weight_scale = (
            params.weight_scale_inv
            if params.weight_scale_inv is not None
            else params.weight_scale
        )

        x_2d = x.reshape(-1, x.shape[-1]) if x.dim() > 2 else x
        out = torch.ops._C.fp8_scaled_mm_cpu(
            x_2d,
            params.weight,
            weight_scale,
            list(self.weight_group_shape),
            bias,
            x.dtype,
            True,  # is_vnni (weight already prepacked)
        )
        return out.reshape(x.shape[:-1] + (out.size(-1),)) if x.dim() > 2 else out

    def apply_block_scaled_mm(
        self,
        A: torch.Tensor,
        B: torch.Tensor,
        As: torch.Tensor,
        Bs: torch.Tensor,
    ) -> torch.Tensor:
        raise NotImplementedError(
            "CPUFp8BlockScaledMMKernel overrides apply_weights directly."
        )
```
**EN:** This kernel class defines `CPUFp8BlockScaledMMKernel`. FP8 W8A16 block-quantized GEMM via AMX BRGEMM on CPU. It inherits from `Fp8BlockScaledMMLinearKernel`. Key methods include `is_supported`, `can_implement`, `process_weights_after_loading`, `apply_weights`, `apply_block_scaled_mm`.
**CN:** 该内核类定义了 `CPUFp8BlockScaledMMKernel`。 它主要负责与 `CPUFp8BlockScaledMMKernel` 对应的数据组织、接口约束或执行流程。 它继承自 `Fp8BlockScaledMMLinearKernel`。 关键方法包括 `is_supported`, `can_implement`, `process_weights_after_loading`, `apply_weights`, `apply_block_scaled_mm`。

### Method `CPUFp8BlockScaledMMKernel.is_supported` (lines 231-240)
```python
    def is_supported(
        cls, compute_capability: int | None = None
    ) -> tuple[bool, str | None]:
        if not current_platform.is_cpu():
            return False, "requires CPU platform."
        if not torch.cpu._is_amx_tile_supported():
            return False, "requires AMX tile support (Sapphire Rapids or newer)."
        if not ops._supports_cpu_fp8_w8a16:
            return False, "fp8_scaled_mm_cpu op not available."
        return True, None
```
**EN:** This method implements `CPUFp8BlockScaledMMKernel.is_supported`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `is_cpu`, `_is_amx_tile_supported`.
**CN:** 该方法 `CPUFp8BlockScaledMMKernel.is_supported` 封装了此模块中的一段关键运行时逻辑，重点处理 is supported 相关工作。 它内部会调用 `is_cpu`, `_is_amx_tile_supported` 等例程。

### Method `CPUFp8BlockScaledMMKernel.can_implement` (lines 243-260)
```python
    def can_implement(
        cls, config: FP8ScaledMMLinearLayerConfig
    ) -> tuple[bool, str | None]:
        # Validate weight block shape
        weight_gs = config.weight_quant_key.scale.group_shape
        if weight_gs.col <= 0 or weight_gs.col != 128:
            return False, (
                "CPU FP8 kernel requires K-dimension block size of 128, "
                f"got {weight_gs.col}."
            )
        if weight_gs.row <= 0 or weight_gs.row % 32 != 0:
            return False, (
                "CPU FP8 kernel requires N-dimension block size to be "
                f"a positive multiple of 32, got {weight_gs.row}."
            )
        if config.out_dtype not in (torch.bfloat16, torch.float32):
            return False, "Only bfloat16/float32 output dtype supported."
        return True, None
```
**EN:** This method implements `CPUFp8BlockScaledMMKernel.can_implement`, encapsulating a focused unit of runtime logic for the module.
**CN:** 该方法 `CPUFp8BlockScaledMMKernel.can_implement` 封装了此模块中的一段关键运行时逻辑，重点处理 can implement 相关工作。

### Method `CPUFp8BlockScaledMMKernel.process_weights_after_loading` (lines 262-290)
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        # Skip the base class process (FP8 padding / fnuz normalization)
        # which is GPU-oriented.  Instead, VNNI-prepack weights for AMX.
        params = self._get_layer_params(layer)
        packed_weight = torch.ops._C.convert_weight_packed(params.weight)
        replace_parameter(
            layer,
            params.WEIGHT,
            torch.nn.Parameter(packed_weight, requires_grad=False),
        )

        # Re-wrap scale as a plain Parameter so the kernel can read it
        # without weight-loader metadata interfering.
        scale_attr = (
            params.WEIGHT_SCALE_INV
            if params.weight_scale_inv is not None
            else params.WEIGHT_SCALE
        )
        weight_scale = (
            params.weight_scale_inv
            if params.weight_scale_inv is not None
            else params.weight_scale
        )
        assert weight_scale is not None
        replace_parameter(
            layer,
            scale_attr,
            torch.nn.Parameter(weight_scale.data, requires_grad=False),
        )
```
**EN:** This method implements `CPUFp8BlockScaledMMKernel.process_weights_after_loading`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `_get_layer_params`, `convert_weight_packed`, `replace_parameter`, `Parameter`.
**CN:** 该方法 `CPUFp8BlockScaledMMKernel.process_weights_after_loading` 封装了此模块中的一段关键运行时逻辑，重点处理 process weights after loading 相关工作。 它内部会调用 `_get_layer_params`, `convert_weight_packed`, `replace_parameter`, `Parameter` 等例程。

### Method `CPUFp8BlockScaledMMKernel.apply_weights` (lines 292-316)
```python
    def apply_weights(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
        **kwargs,
    ) -> torch.Tensor:
        params = self._get_layer_params(layer)
        weight_scale = (
            params.weight_scale_inv
            if params.weight_scale_inv is not None
            else params.weight_scale
        )

        x_2d = x.reshape(-1, x.shape[-1]) if x.dim() > 2 else x
        out = torch.ops._C.fp8_scaled_mm_cpu(
            x_2d,
            params.weight,
            weight_scale,
            list(self.weight_group_shape),
            bias,
            x.dtype,
            True,  # is_vnni (weight already prepacked)
        )
        return out.reshape(x.shape[:-1] + (out.size(-1),)) if x.dim() > 2 else out
```
**EN:** This method implements `CPUFp8BlockScaledMMKernel.apply_weights`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `_get_layer_params`, `fp8_scaled_mm_cpu`, `reshape`, `list`.
**CN:** 该方法 `CPUFp8BlockScaledMMKernel.apply_weights` 封装了此模块中的一段关键运行时逻辑，重点处理 apply weights 相关工作。 它内部会调用 `_get_layer_params`, `fp8_scaled_mm_cpu`, `reshape`, `list` 等例程。

### Method `CPUFp8BlockScaledMMKernel.apply_block_scaled_mm` (lines 318-327)
```python
    def apply_block_scaled_mm(
        self,
        A: torch.Tensor,
        B: torch.Tensor,
        As: torch.Tensor,
        Bs: torch.Tensor,
    ) -> torch.Tensor:
        raise NotImplementedError(
            "CPUFp8BlockScaledMMKernel overrides apply_weights directly."
        )
```
**EN:** This method implements `CPUFp8BlockScaledMMKernel.apply_block_scaled_mm`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `NotImplementedError`.
**CN:** 该方法 `CPUFp8BlockScaledMMKernel.apply_block_scaled_mm` 封装了此模块中的一段关键运行时逻辑，重点处理 apply block scaled mm 相关工作。 它内部会调用 `NotImplementedError` 等例程。

## Key Concepts / 关键概念
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
- **Linear kernel abstraction / 线性内核抽象**
  - **EN:** The module defines abstractions or selectors for quantized linear operators.
  - **CN:** 该模块定义量化线性算子的抽象或选择器。

## Dependencies / 依赖关系
- **vLLM internal / vLLM 内部依赖**: `vllm`, `vllm.model_executor.layers.quantization.utils`, `vllm.model_executor.layers.quantization.utils.w8a8_utils`, `vllm.model_executor.layers.utils`, `vllm.platforms`, `vllm.platforms.interface`, `.BlockScaledMMLinearKernel`, `.ScaledMMLinearKernel`
- **External / 外部依赖**: `torch`
