# machete.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/kernels/linear/mixed_precision/machete.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the Machete mixed-precision linear kernel. / 实现 Machete 混合精度线性内核。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-21)
```python
from functools import partial

import torch

from vllm import _custom_ops as ops
from vllm.model_executor.layers.quantization.utils.machete_utils import (
    check_machete_supports_shape,
    query_machete_supported_group_sizes,
    query_machete_supported_quant_types,
)
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    pack_quantized_values_into_int32,
    unpack_quantized_values_into_int32,
)
from vllm.model_executor.parameter import BasevLLMParameter, permute_param_layout_
from vllm.platforms import current_platform

from .MPLinearKernel import MPLinearKernel, MPLinearLayerConfig
```
**EN:** This import block loads `functools`, `torch`, `vllm`, `vllm.model_executor.layers.quantization.utils.machete_utils`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.model_executor.parameter`, ..., establishing the tensor libraries, backend hooks, and vLLM helpers needed by the rest of `machete.py`.
**CN:** 该导入代码块加载了 `functools`, `torch`, `vllm`, `vllm.model_executor.layers.quantization.utils.machete_utils`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.model_executor.parameter`, ...，为 `machete.py` 后续逻辑准备张量库、后端钩子以及 vLLM 辅助工具。

### Class `MacheteLinearKernel` (lines 24-159)
```python
class MacheteLinearKernel(MPLinearKernel):
    @classmethod
    def get_min_capability(cls) -> int:
        return 90

    @classmethod
    def can_implement(cls, c: MPLinearLayerConfig) -> tuple[bool, str | None]:
        # Machete uses CUTLASS, so it can only be compatible with Nvidia
        if not current_platform.is_cuda():
            return False, "Machete only supported on CUDA"

        if not current_platform.is_device_capability(90):
            return False, "Machete requires compute capability of 90 (Hopper)"

        if c.has_g_idx and c.partition_weight_shape[0] != c.full_weight_shape[0]:
            return (
                False,
                "Act reordering currently not supported by Machete, "
                "when the input features are partitioned across "
                "devices",
            )

        if c.weight_type not in query_machete_supported_quant_types(c.zero_points):
            return (
                False,
                f"Quant type ({c.weight_type}) not supported by "
                "Machete, supported types are: "
                f"{query_machete_supported_quant_types(c.zero_points)}",
            )

        if c.group_size not in query_machete_supported_group_sizes(c.act_type):
            return (
                False,
                f"Group size ({c.group_size}) not supported by "
                "Machete, supported group sizes are: "
                f"{query_machete_supported_group_sizes(c.act_type)}",
            )

        return check_machete_supports_shape(
            c.partition_weight_shape[0], c.partition_weight_shape[1]
        )

    # note assumes that
    #  `weight_packed` is: {input_dim = 0, output_dim = 1, packed_dim = 0}
    #  `weight_scale`  is: {input_dim = 0, output_dim = 1}
    #  `weight_zp`     is: {input_dim = 0, output_dim = 1, packed_dim = 1}
    def process_weights_after_loading(self, layer: torch.nn.Module):
        c = self.config

        if c.has_g_idx:
            assert self.w_gidx_name is not None
            perm = torch.argsort(getattr(layer, self.w_gidx_name)).to(torch.int)

            self.act_perm = lambda x: x[:, perm]
            # use `ops.permute_cols` if possible
            if (
                c.act_type in [torch.float16, torch.bfloat16]
                and c.partition_weight_shape[0] % 8 == 0
            ):
                self.act_perm = partial(ops.permute_cols, perm=perm)

        def transform_w_q(x):
            assert isinstance(x, BasevLLMParameter)
            permute_param_layout_(x, input_dim=0, output_dim=1, packed_dim=0)
            if c.has_g_idx:
                x_unpacked = unpack_quantized_values_into_int32(
                    x.data, c.weight_type, packed_dim=0
                )
                x_perm = x_unpacked[perm, :]
                x.data = pack_quantized_values_into_int32(
                    x_perm, c.weight_type, packed_dim=0
                )
            x.data = ops.machete_prepack_B(
                x.data.t().contiguous().t(),
                a_type=c.act_type,
                b_type=c.weight_type,
                group_scales_type=c.act_type,
            )
            return x

        def transform_w_s(x):
            assert isinstance(x, BasevLLMParameter)
            permute_param_layout_(x, input_dim=0, output_dim=1)
            x.data = x.data.contiguous()
            return x

        def transform_w_zp(x):
            assert isinstance(x, BasevLLMParameter)
            permute_param_layout_(x, input_dim=0, output_dim=1, packed_dim=1)
            x_unpacked = unpack_quantized_values_into_int32(
                x.data, c.weight_type, packed_dim=1
            )
            w_s = getattr(layer, self.w_s_name).data
            # pre-apply scales to zero-points
            x.data = (-1.0 * w_s * (x_unpacked.to(w_s.dtype))).contiguous()
            return x

        # Repack weights and scales for Machete
        self._transform_param(layer, self.w_q_name, transform_w_q)
        self._transform_param(layer, self.w_s_name, transform_w_s)
        if c.zero_points:
            self._transform_param(layer, self.w_zp_name, transform_w_zp)

    def apply_weights(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        c = self.config
        w_q, w_s, w_zp, _ = self._get_weight_params(layer)

        x_2d = x.reshape(-1, x.shape[-1])
        out_shape = x.shape[:-1] + (c.partition_weight_shape[1],)

        if c.has_g_idx:
            x_2d = self.act_perm(x_2d)

        if c.zero_points:
            assert w_zp is not None
        else:
            w_zp = None

        output = ops.machete_mm(
            a=x_2d,
            b_q=w_q,
            b_type=c.weight_type,
            b_group_zeros=w_zp,
            b_group_scales=w_s,
            b_group_size=c.group_size,
        )

        if bias is not None:
            output.add_(bias)  # In-place add

        return output.reshape(out_shape)
```
**EN:** This kernel class defines `MacheteLinearKernel`. It organizes the state and behavior needed by this kernel path. It inherits from `MPLinearKernel`. Key methods include `get_min_capability`, `can_implement`, `process_weights_after_loading`, `apply_weights`.
**CN:** 该内核类定义了 `MacheteLinearKernel`。 它组织了该内核路径所需的状态与行为。 它继承自 `MPLinearKernel`。 关键方法包括 `get_min_capability`, `can_implement`, `process_weights_after_loading`, `apply_weights`。

### Method `MacheteLinearKernel.get_min_capability` (lines 26-27)
```python
    def get_min_capability(cls) -> int:
        return 90
```
**EN:** This method implements `MacheteLinearKernel.get_min_capability`, encapsulating a focused unit of runtime logic for the module.
**CN:** 该方法 `MacheteLinearKernel.get_min_capability` 封装了此模块中的一段关键运行时逻辑，重点处理 get min capability 相关工作。

### Method `MacheteLinearKernel.can_implement` (lines 30-64)
```python
    def can_implement(cls, c: MPLinearLayerConfig) -> tuple[bool, str | None]:
        # Machete uses CUTLASS, so it can only be compatible with Nvidia
        if not current_platform.is_cuda():
            return False, "Machete only supported on CUDA"

        if not current_platform.is_device_capability(90):
            return False, "Machete requires compute capability of 90 (Hopper)"

        if c.has_g_idx and c.partition_weight_shape[0] != c.full_weight_shape[0]:
            return (
                False,
                "Act reordering currently not supported by Machete, "
                "when the input features are partitioned across "
                "devices",
            )

        if c.weight_type not in query_machete_supported_quant_types(c.zero_points):
            return (
                False,
                f"Quant type ({c.weight_type}) not supported by "
                "Machete, supported types are: "
                f"{query_machete_supported_quant_types(c.zero_points)}",
            )

        if c.group_size not in query_machete_supported_group_sizes(c.act_type):
            return (
                False,
                f"Group size ({c.group_size}) not supported by "
                "Machete, supported group sizes are: "
                f"{query_machete_supported_group_sizes(c.act_type)}",
            )

        return check_machete_supports_shape(
            c.partition_weight_shape[0], c.partition_weight_shape[1]
        )
```
**EN:** This method implements `MacheteLinearKernel.can_implement`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `check_machete_supports_shape`, `is_cuda`, `is_device_capability`, `query_machete_supported_quant_types`.
**CN:** 该方法 `MacheteLinearKernel.can_implement` 封装了此模块中的一段关键运行时逻辑，重点处理 can implement 相关工作。 它内部会调用 `check_machete_supports_shape`, `is_cuda`, `is_device_capability`, `query_machete_supported_quant_types` 等例程。

### Method `MacheteLinearKernel.process_weights_after_loading` (lines 70-125)
```python
    def process_weights_after_loading(self, layer: torch.nn.Module):
        c = self.config

        if c.has_g_idx:
            assert self.w_gidx_name is not None
            perm = torch.argsort(getattr(layer, self.w_gidx_name)).to(torch.int)

            self.act_perm = lambda x: x[:, perm]
            # use `ops.permute_cols` if possible
            if (
                c.act_type in [torch.float16, torch.bfloat16]
                and c.partition_weight_shape[0] % 8 == 0
            ):
                self.act_perm = partial(ops.permute_cols, perm=perm)

        def transform_w_q(x):
            assert isinstance(x, BasevLLMParameter)
            permute_param_layout_(x, input_dim=0, output_dim=1, packed_dim=0)
            if c.has_g_idx:
                x_unpacked = unpack_quantized_values_into_int32(
                    x.data, c.weight_type, packed_dim=0
                )
                x_perm = x_unpacked[perm, :]
                x.data = pack_quantized_values_into_int32(
                    x_perm, c.weight_type, packed_dim=0
                )
            x.data = ops.machete_prepack_B(
                x.data.t().contiguous().t(),
                a_type=c.act_type,
                b_type=c.weight_type,
                group_scales_type=c.act_type,
            )
            return x

        def transform_w_s(x):
            assert isinstance(x, BasevLLMParameter)
            permute_param_layout_(x, input_dim=0, output_dim=1)
            x.data = x.data.contiguous()
            return x

        def transform_w_zp(x):
            assert isinstance(x, BasevLLMParameter)
            permute_param_layout_(x, input_dim=0, output_dim=1, packed_dim=1)
            x_unpacked = unpack_quantized_values_into_int32(
                x.data, c.weight_type, packed_dim=1
            )
            w_s = getattr(layer, self.w_s_name).data
            # pre-apply scales to zero-points
            x.data = (-1.0 * w_s * (x_unpacked.to(w_s.dtype))).contiguous()
            return x

        # Repack weights and scales for Machete
        self._transform_param(layer, self.w_q_name, transform_w_q)
        self._transform_param(layer, self.w_s_name, transform_w_s)
        if c.zero_points:
            self._transform_param(layer, self.w_zp_name, transform_w_zp)
```
**EN:** This method implements `MacheteLinearKernel.process_weights_after_loading`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `_transform_param`, `to`, `isinstance`, `permute_param_layout_`.
**CN:** 该方法 `MacheteLinearKernel.process_weights_after_loading` 封装了此模块中的一段关键运行时逻辑，重点处理 process weights after loading 相关工作。 它内部会调用 `_transform_param`, `to`, `isinstance`, `permute_param_layout_` 等例程。

### Method `MacheteLinearKernel.apply_weights` (lines 127-159)
```python
    def apply_weights(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        c = self.config
        w_q, w_s, w_zp, _ = self._get_weight_params(layer)

        x_2d = x.reshape(-1, x.shape[-1])
        out_shape = x.shape[:-1] + (c.partition_weight_shape[1],)

        if c.has_g_idx:
            x_2d = self.act_perm(x_2d)

        if c.zero_points:
            assert w_zp is not None
        else:
            w_zp = None

        output = ops.machete_mm(
            a=x_2d,
            b_q=w_q,
            b_type=c.weight_type,
            b_group_zeros=w_zp,
            b_group_scales=w_s,
            b_group_size=c.group_size,
        )

        if bias is not None:
            output.add_(bias)  # In-place add

        return output.reshape(out_shape)
```
**EN:** This method implements `MacheteLinearKernel.apply_weights`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `_get_weight_params`, `reshape`, `machete_mm`, `act_perm`.
**CN:** 该方法 `MacheteLinearKernel.apply_weights` 封装了此模块中的一段关键运行时逻辑，重点处理 apply weights 相关工作。 它内部会调用 `_get_weight_params`, `reshape`, `machete_mm`, `act_perm` 等例程。

## Key Concepts / 关键概念
- **CUTLASS backend / CUTLASS 后端**
  - **EN:** The module selects or wraps CUTLASS kernels for NVIDIA-oriented matrix multiplication.
  - **CN:** 该模块选择或封装 CUTLASS 内核，以支持面向 NVIDIA 的矩阵乘计算。
- **Mixed-precision linear kernels / 混合精度线性内核**
  - **EN:** The file implements mixed-precision linear layers over several backend providers.
  - **CN:** 该文件实现基于多个后端提供者的混合精度线性层。
- **Linear kernel abstraction / 线性内核抽象**
  - **EN:** The module defines abstractions or selectors for quantized linear operators.
  - **CN:** 该模块定义量化线性算子的抽象或选择器。

## Dependencies / 依赖关系
- **vLLM internal / vLLM 内部依赖**: `vllm`, `vllm.model_executor.layers.quantization.utils.machete_utils`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.model_executor.parameter`, `vllm.platforms`, `.MPLinearKernel`
- **External / 外部依赖**: `functools`, `torch`
