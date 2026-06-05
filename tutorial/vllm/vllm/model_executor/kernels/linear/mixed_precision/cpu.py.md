# cpu.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/kernels/linear/mixed_precision/cpu.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the CPU mixed-precision linear kernel. / 实现 CPU 混合精度线性内核。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-14)
```python
import torch

from vllm import _custom_ops as ops
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    pack_quantized_values_into_int32,
    unpack_quantized_values_into_int32,
)
from vllm.platforms import current_platform
from vllm.scalar_type import scalar_types

from .MPLinearKernel import MPLinearKernel, MPLinearLayerConfig
```
**EN:** This import block loads `torch`, `vllm`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.platforms`, `vllm.scalar_type`, `.MPLinearKernel`, establishing the tensor libraries, backend hooks, and vLLM helpers needed by the rest of `cpu.py`.
**CN:** 该导入代码块加载了 `torch`, `vllm`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.platforms`, `vllm.scalar_type`, `.MPLinearKernel`，为 `cpu.py` 后续逻辑准备张量库、后端钩子以及 vLLM 辅助工具。

### Constants / assignments (lines 16-16)
```python
_CPUWNA16_SUPPORTED_QUANT_TYPES = (scalar_types.uint4, scalar_types.uint4b8)
```
**EN:** This block defines module constants (`_CPUWNA16_SUPPORTED_QUANT_TYPES`) that encode defaults, feature flags, or operator metadata used later in the file.
**CN:** 该代码块定义了模块常量（`_CPUWNA16_SUPPORTED_QUANT_TYPES`），用于表示默认值、特性开关或后续逻辑要使用的算子元数据。

### Class `CPUWNA16LinearKernel` (lines 19-139)
```python
class CPUWNA16LinearKernel(MPLinearKernel):
    @classmethod
    def get_min_capability(cls) -> int:
        return -1

    @classmethod
    def can_implement(cls, c: MPLinearLayerConfig) -> tuple[bool, str | None]:
        if not current_platform.is_cpu():
            return False, "CPUWNA16 only supported on CPU"

        if c.weight_type not in _CPUWNA16_SUPPORTED_QUANT_TYPES:
            return (
                False,
                f"Quant type ({c.weight_type}) not supported by "
                "CPUWNA16, supported types are: "
                f"{_CPUWNA16_SUPPORTED_QUANT_TYPES}",
            )

        if c.group_size != -1 and c.group_size % 2 != 0:
            return (
                False,
                f"Group size ({c.group_size}) not supported by "
                "CPUWNA16, supported group sizes are multiples of 2",
            )

        if c.partition_weight_shape[0] % 32 != 0:
            return (
                False,
                f"Input size ({c.partition_weight_shape[0]}) not supported by "
                "CPUWNA16, supported sizes are multiples of 32",
            )

        if c.partition_weight_shape[1] % 32 != 0:
            return (
                False,
                f"Output size ({c.partition_weight_shape[1]}) not supported by "
                "CPUWNA16, supported sizes are multiples of 32",
            )

        return True, None

    # note assumes that
    #  `weight_packed` is: {input_dim = 0, output_dim = 1, packed_dim = 0} (marlin)
    #                  or: {input_dim = 1, output_dim = 0, packed_dim = 1} (CT)
    #  `weight_scale`  is: {input_dim = 0, output_dim = 1} (marlin)
    #                  or: {input_dim = 1, output_dim = 0} (CT)
    #  `weight_zp`     is: {input_dim = 0, output_dim = 1, packed_dim = 1} (marlin)
    #                  or: {input_dim = 1, output_dim = 0, packed_dim = 0} (CT)
    def _process_gptq_weights(self, layer: torch.nn.Module):
        packed_weight = getattr(layer, self.w_q_name)
        assert packed_weight.input_dim == packed_weight.packed_dim
        is_ct_format = packed_weight.input_dim == 1
        if is_ct_format:
            packed_weight = packed_weight.t()
        bits = self.config.weight_type.mantissa
        pack_factor = 32 // bits
        p_w_k, _ = packed_weight.size()
        input_size = p_w_k * pack_factor
        isa_hint = _get_isa_hint(getattr(layer, self.w_s_name).dtype)
        layer.isa_hint = isa_hint

        # convert input dim packed to output dim packed
        weight = unpack_quantized_values_into_int32(
            packed_weight, self.config.weight_type, 0
        )
        weight = pack_quantized_values_into_int32(weight, self.config.weight_type, 1)
        # make 16 output channel as a block and transpose to the make
        # the block contiguous
        weight = (
            weight.view(input_size, -1, 16 // pack_factor)
            .permute(1, 0, 2)
            .reshape(-1, input_size * 16 // pack_factor)
            .contiguous()
        )
        getattr(layer, self.w_q_name).data = weight

        # transpose scale, zp for CT format
        if is_ct_format:
            scales = getattr(layer, self.w_s_name)
            scales.data = scales.t().contiguous()
            if self.config.zero_points:
                assert self.w_zp_name
                zp = getattr(layer, self.w_zp_name)
                zp.data = zp.t().contiguous()

    def process_weights_after_loading(self, layer: torch.nn.Module):
        if (not self.config.zero_points) and (self.w_zp_name is not None):
            setattr(layer, self.w_zp_name, None)

        if (not self.config.has_g_idx) and (self.w_gidx_name is not None):
            setattr(layer, self.w_gidx_name, None)

        w_input_dim = getattr(layer, self.w_q_name).input_dim
        w_pack_dim = getattr(layer, self.w_q_name).packed_dim
        quant_method = "gptq" if w_pack_dim == w_input_dim else "awq"

        if quant_method == "gptq":
            # GPTQ
            self._process_gptq_weights(layer)
        else:
            # AWQ
            raise NotImplementedError("AWQ is not supported in CPUWNA16LinearKernel")

    def apply_weights(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        w_q, w_s, w_zp, w_gidx = self._get_weight_params(layer)
        x = ops.cpu_gemm_wna16(
            input=x,
            q_weight=w_q,
            scales=w_s,
            zeros=w_zp,
            g_idx=w_gidx,
            bias=bias,
            pack_factor=8,  # 32 // 4
            isa_hint=layer.isa_hint,
        )
        return x
```
**EN:** This kernel class defines `CPUWNA16LinearKernel`. It organizes the state and behavior needed by this kernel path. It inherits from `MPLinearKernel`. Key methods include `get_min_capability`, `can_implement`, `_process_gptq_weights`, `process_weights_after_loading`, `apply_weights`.
**CN:** 该内核类定义了 `CPUWNA16LinearKernel`。 它组织了该内核路径所需的状态与行为。 它继承自 `MPLinearKernel`。 关键方法包括 `get_min_capability`, `can_implement`, `_process_gptq_weights`, `process_weights_after_loading`, `apply_weights`。

### Method `CPUWNA16LinearKernel.get_min_capability` (lines 21-22)
```python
    def get_min_capability(cls) -> int:
        return -1
```
**EN:** This method implements `CPUWNA16LinearKernel.get_min_capability`, encapsulating a focused unit of runtime logic for the module.
**CN:** 该方法 `CPUWNA16LinearKernel.get_min_capability` 封装了此模块中的一段关键运行时逻辑，重点处理 get min capability 相关工作。

### Method `CPUWNA16LinearKernel.can_implement` (lines 25-58)
```python
    def can_implement(cls, c: MPLinearLayerConfig) -> tuple[bool, str | None]:
        if not current_platform.is_cpu():
            return False, "CPUWNA16 only supported on CPU"

        if c.weight_type not in _CPUWNA16_SUPPORTED_QUANT_TYPES:
            return (
                False,
                f"Quant type ({c.weight_type}) not supported by "
                "CPUWNA16, supported types are: "
                f"{_CPUWNA16_SUPPORTED_QUANT_TYPES}",
            )

        if c.group_size != -1 and c.group_size % 2 != 0:
            return (
                False,
                f"Group size ({c.group_size}) not supported by "
                "CPUWNA16, supported group sizes are multiples of 2",
            )

        if c.partition_weight_shape[0] % 32 != 0:
            return (
                False,
                f"Input size ({c.partition_weight_shape[0]}) not supported by "
                "CPUWNA16, supported sizes are multiples of 32",
            )

        if c.partition_weight_shape[1] % 32 != 0:
            return (
                False,
                f"Output size ({c.partition_weight_shape[1]}) not supported by "
                "CPUWNA16, supported sizes are multiples of 32",
            )

        return True, None
```
**EN:** This method implements `CPUWNA16LinearKernel.can_implement`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `is_cpu`.
**CN:** 该方法 `CPUWNA16LinearKernel.can_implement` 封装了此模块中的一段关键运行时逻辑，重点处理 can implement 相关工作。 它内部会调用 `is_cpu` 等例程。

### Method `CPUWNA16LinearKernel._process_gptq_weights` (lines 67-102)
```python
    def _process_gptq_weights(self, layer: torch.nn.Module):
        packed_weight = getattr(layer, self.w_q_name)
        assert packed_weight.input_dim == packed_weight.packed_dim
        is_ct_format = packed_weight.input_dim == 1
        if is_ct_format:
            packed_weight = packed_weight.t()
        bits = self.config.weight_type.mantissa
        pack_factor = 32 // bits
        p_w_k, _ = packed_weight.size()
        input_size = p_w_k * pack_factor
        isa_hint = _get_isa_hint(getattr(layer, self.w_s_name).dtype)
        layer.isa_hint = isa_hint

        # convert input dim packed to output dim packed
        weight = unpack_quantized_values_into_int32(
            packed_weight, self.config.weight_type, 0
        )
        weight = pack_quantized_values_into_int32(weight, self.config.weight_type, 1)
        # make 16 output channel as a block and transpose to the make
        # the block contiguous
        weight = (
            weight.view(input_size, -1, 16 // pack_factor)
            .permute(1, 0, 2)
            .reshape(-1, input_size * 16 // pack_factor)
            .contiguous()
        )
        getattr(layer, self.w_q_name).data = weight

        # transpose scale, zp for CT format
        if is_ct_format:
            scales = getattr(layer, self.w_s_name)
            scales.data = scales.t().contiguous()
            if self.config.zero_points:
                assert self.w_zp_name
                zp = getattr(layer, self.w_zp_name)
                zp.data = zp.t().contiguous()
```
**EN:** This method implements `CPUWNA16LinearKernel._process_gptq_weights`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `getattr`, `size`, `_get_isa_hint`, `unpack_quantized_values_into_int32`.
**CN:** 该方法 `CPUWNA16LinearKernel._process_gptq_weights` 封装了此模块中的一段关键运行时逻辑，重点处理 process gptq weights 相关工作。 它内部会调用 `getattr`, `size`, `_get_isa_hint`, `unpack_quantized_values_into_int32` 等例程。

### Method `CPUWNA16LinearKernel.process_weights_after_loading` (lines 104-120)
```python
    def process_weights_after_loading(self, layer: torch.nn.Module):
        if (not self.config.zero_points) and (self.w_zp_name is not None):
            setattr(layer, self.w_zp_name, None)

        if (not self.config.has_g_idx) and (self.w_gidx_name is not None):
            setattr(layer, self.w_gidx_name, None)

        w_input_dim = getattr(layer, self.w_q_name).input_dim
        w_pack_dim = getattr(layer, self.w_q_name).packed_dim
        quant_method = "gptq" if w_pack_dim == w_input_dim else "awq"

        if quant_method == "gptq":
            # GPTQ
            self._process_gptq_weights(layer)
        else:
            # AWQ
            raise NotImplementedError("AWQ is not supported in CPUWNA16LinearKernel")
```
**EN:** This method implements `CPUWNA16LinearKernel.process_weights_after_loading`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `setattr`, `getattr`, `_process_gptq_weights`, `NotImplementedError`.
**CN:** 该方法 `CPUWNA16LinearKernel.process_weights_after_loading` 封装了此模块中的一段关键运行时逻辑，重点处理 process weights after loading 相关工作。 它内部会调用 `setattr`, `getattr`, `_process_gptq_weights`, `NotImplementedError` 等例程。

### Method `CPUWNA16LinearKernel.apply_weights` (lines 122-139)
```python
    def apply_weights(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        w_q, w_s, w_zp, w_gidx = self._get_weight_params(layer)
        x = ops.cpu_gemm_wna16(
            input=x,
            q_weight=w_q,
            scales=w_s,
            zeros=w_zp,
            g_idx=w_gidx,
            bias=bias,
            pack_factor=8,  # 32 // 4
            isa_hint=layer.isa_hint,
        )
        return x
```
**EN:** This method implements `CPUWNA16LinearKernel.apply_weights`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `_get_weight_params`, `cpu_gemm_wna16`.
**CN:** 该方法 `CPUWNA16LinearKernel.apply_weights` 封装了此模块中的一段关键运行时逻辑，重点处理 apply weights 相关工作。 它内部会调用 `_get_weight_params`, `cpu_gemm_wna16` 等例程。

### Function `_get_isa_hint` (lines 142-147)
```python
def _get_isa_hint(dtype: torch.dtype) -> str:
    supports_amx = torch.cpu._is_amx_tile_supported()
    if supports_amx and dtype in (torch.bfloat16,):
        return "amx"
    else:
        return "vec"
```
**EN:** This lookup helper implements `_get_isa_hint`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `_is_amx_tile_supported`.
**CN:** 该函数 `_get_isa_hint` 封装了此模块中的一段关键运行时逻辑，重点处理 get isa hint 相关工作。 它内部会调用 `_is_amx_tile_supported` 等例程。

## Key Concepts / 关键概念
- **Marlin backend / Marlin 后端**
  - **EN:** The file exposes Marlin-specific kernels or selection logic for quantized linear layers.
  - **CN:** 该文件暴露 Marlin 专用内核或量化线性层的选择逻辑。
- **Mixed-precision linear kernels / 混合精度线性内核**
  - **EN:** The file implements mixed-precision linear layers over several backend providers.
  - **CN:** 该文件实现基于多个后端提供者的混合精度线性层。
- **Linear kernel abstraction / 线性内核抽象**
  - **EN:** The module defines abstractions or selectors for quantized linear operators.
  - **CN:** 该模块定义量化线性算子的抽象或选择器。

## Dependencies / 依赖关系
- **vLLM internal / vLLM 内部依赖**: `vllm`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.platforms`, `vllm.scalar_type`, `.MPLinearKernel`
- **External / 外部依赖**: `torch`
