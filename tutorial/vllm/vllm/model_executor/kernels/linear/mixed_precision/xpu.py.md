# xpu.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/kernels/linear/mixed_precision/xpu.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the XPU mixed-precision linear kernel. / 实现 XPU 混合精度线性内核。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 5-13)
```python
import torch
from torch.nn.parameter import Parameter

from vllm.logger import init_logger
from vllm.model_executor.layers.quantization.utils import replace_parameter
from vllm.platforms import current_platform
from vllm.scalar_type import scalar_types

from .MPLinearKernel import MPLinearKernel, MPLinearLayerConfig
```
**EN:** This import block loads `torch`, `torch.nn.parameter`, `vllm.logger`, `vllm.model_executor.layers.quantization.utils`, `vllm.platforms`, `vllm.scalar_type`, ..., establishing the tensor libraries, backend hooks, and vLLM helpers needed by the rest of `xpu.py`.
**CN:** 该导入代码块加载了 `torch`, `torch.nn.parameter`, `vllm.logger`, `vllm.model_executor.layers.quantization.utils`, `vllm.platforms`, `vllm.scalar_type`, ...，为 `xpu.py` 后续逻辑准备张量库、后端钩子以及 vLLM 辅助工具。

### Constants / assignments (lines 15-17)
```python
_XPUWNA16_SUPPORTED_QUANT_TYPES = (scalar_types.uint4, scalar_types.uint4b8)

logger = init_logger(__name__)
```
**EN:** This block defines module constants (`_XPUWNA16_SUPPORTED_QUANT_TYPES`, `logger`) that encode defaults, feature flags, or operator metadata used later in the file.
**CN:** 该代码块定义了模块常量（`_XPUWNA16_SUPPORTED_QUANT_TYPES`, `logger`），用于表示默认值、特性开关或后续逻辑要使用的算子元数据。

### Class `XPUwNa16LinearKernel` (lines 20-92)
```python
class XPUwNa16LinearKernel(MPLinearKernel):
    @classmethod
    def get_min_capability(cls) -> int:
        return -1

    @classmethod
    def can_implement(cls, c: MPLinearLayerConfig) -> tuple[bool, str | None]:
        if not current_platform.is_xpu():
            return False, "XPUwNa16 only supported on XPU"

        if c.act_type != torch.bfloat16 and c.act_type != torch.float16:
            return False, "XPUwNa16 only supports BF16/FP16 activations"

        if c.weight_type not in _XPUWNA16_SUPPORTED_QUANT_TYPES:
            return (
                False,
                f"Quant type ({c.weight_type}) not supported by "
                "XPUwNa16, supported types are: "
                f"{_XPUWNA16_SUPPORTED_QUANT_TYPES}",
            )
        if c.group_size != -1 and c.group_size % 32 != 0:
            return (
                False,
                f"Group size ({c.group_size}) not supported by "
                "XPUwNa16, supported group sizes are multiples of 32",
            )

        if c.partition_weight_shape[0] % 32 != 0:
            return (
                False,
                f"Input size ({c.partition_weight_shape[0]}) not supported by "
                "XPUwNa16, supported sizes are multiples of 32",
            )

        if c.partition_weight_shape[1] % 32 != 0:
            return (
                False,
                f"Output size ({c.partition_weight_shape[1]}) not supported by "
                "XPUWNA16, supported sizes are multiples of 32",
            )

        return True, None

    def process_weights_after_loading(self, layer: torch.nn.Module):
        layer.weight_scale.data = layer.weight_scale.t().contiguous()

        if self.config.zero_points:
            layer.weight_zero_point.data = layer.weight_zero_point.t().contiguous()
        else:
            weight_zero_point = torch.Tensor([8]).to(torch.int8).to("xpu")
            layer.weight_zero_point = Parameter(weight_zero_point, requires_grad=False)
        if self.config.has_g_idx:
            layer.g_idx.data = layer.g_idx.t().contiguous()
        else:
            layer.g_idx = None

    def apply_weights(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        reshaped_x = x.reshape(-1, x.shape[-1])
        out = torch.ops._xpu_C.int4_gemm_w4a16(
            reshaped_x,
            layer.weight_packed.t(),
            bias,
            layer.weight_scale,
            layer.weight_zero_point,
            self.config.group_size,
            layer.g_idx,
        )
        return out
```
**EN:** This kernel class defines `XPUwNa16LinearKernel`. It organizes the state and behavior needed by this kernel path. It inherits from `MPLinearKernel`. Key methods include `get_min_capability`, `can_implement`, `process_weights_after_loading`, `apply_weights`.
**CN:** 该内核类定义了 `XPUwNa16LinearKernel`。 它组织了该内核路径所需的状态与行为。 它继承自 `MPLinearKernel`。 关键方法包括 `get_min_capability`, `can_implement`, `process_weights_after_loading`, `apply_weights`。

### Method `XPUwNa16LinearKernel.get_min_capability` (lines 22-23)
```python
    def get_min_capability(cls) -> int:
        return -1
```
**EN:** This method implements `XPUwNa16LinearKernel.get_min_capability`, encapsulating a focused unit of runtime logic for the module.
**CN:** 该方法 `XPUwNa16LinearKernel.get_min_capability` 封装了此模块中的一段关键运行时逻辑，重点处理 get min capability 相关工作。

### Method `XPUwNa16LinearKernel.can_implement` (lines 26-61)
```python
    def can_implement(cls, c: MPLinearLayerConfig) -> tuple[bool, str | None]:
        if not current_platform.is_xpu():
            return False, "XPUwNa16 only supported on XPU"

        if c.act_type != torch.bfloat16 and c.act_type != torch.float16:
            return False, "XPUwNa16 only supports BF16/FP16 activations"

        if c.weight_type not in _XPUWNA16_SUPPORTED_QUANT_TYPES:
            return (
                False,
                f"Quant type ({c.weight_type}) not supported by "
                "XPUwNa16, supported types are: "
                f"{_XPUWNA16_SUPPORTED_QUANT_TYPES}",
            )
        if c.group_size != -1 and c.group_size % 32 != 0:
            return (
                False,
                f"Group size ({c.group_size}) not supported by "
                "XPUwNa16, supported group sizes are multiples of 32",
            )

        if c.partition_weight_shape[0] % 32 != 0:
            return (
                False,
                f"Input size ({c.partition_weight_shape[0]}) not supported by "
                "XPUwNa16, supported sizes are multiples of 32",
            )

        if c.partition_weight_shape[1] % 32 != 0:
            return (
                False,
                f"Output size ({c.partition_weight_shape[1]}) not supported by "
                "XPUWNA16, supported sizes are multiples of 32",
            )

        return True, None
```
**EN:** This method implements `XPUwNa16LinearKernel.can_implement`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `is_xpu`.
**CN:** 该方法 `XPUwNa16LinearKernel.can_implement` 封装了此模块中的一段关键运行时逻辑，重点处理 can implement 相关工作。 它内部会调用 `is_xpu` 等例程。

### Method `XPUwNa16LinearKernel.process_weights_after_loading` (lines 63-74)
```python
    def process_weights_after_loading(self, layer: torch.nn.Module):
        layer.weight_scale.data = layer.weight_scale.t().contiguous()

        if self.config.zero_points:
            layer.weight_zero_point.data = layer.weight_zero_point.t().contiguous()
        else:
            weight_zero_point = torch.Tensor([8]).to(torch.int8).to("xpu")
            layer.weight_zero_point = Parameter(weight_zero_point, requires_grad=False)
        if self.config.has_g_idx:
            layer.g_idx.data = layer.g_idx.t().contiguous()
        else:
            layer.g_idx = None
```
**EN:** This method implements `XPUwNa16LinearKernel.process_weights_after_loading`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `contiguous`, `to`, `Parameter`, `t`.
**CN:** 该方法 `XPUwNa16LinearKernel.process_weights_after_loading` 封装了此模块中的一段关键运行时逻辑，重点处理 process weights after loading 相关工作。 它内部会调用 `contiguous`, `to`, `Parameter`, `t` 等例程。

### Method `XPUwNa16LinearKernel.apply_weights` (lines 76-92)
```python
    def apply_weights(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        reshaped_x = x.reshape(-1, x.shape[-1])
        out = torch.ops._xpu_C.int4_gemm_w4a16(
            reshaped_x,
            layer.weight_packed.t(),
            bias,
            layer.weight_scale,
            layer.weight_zero_point,
            self.config.group_size,
            layer.g_idx,
        )
        return out
```
**EN:** This method implements `XPUwNa16LinearKernel.apply_weights`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `reshape`, `int4_gemm_w4a16`, `t`.
**CN:** 该方法 `XPUwNa16LinearKernel.apply_weights` 封装了此模块中的一段关键运行时逻辑，重点处理 apply weights 相关工作。 它内部会调用 `reshape`, `int4_gemm_w4a16`, `t` 等例程。

### Class `XPUW4A8IntLinearKernel` (lines 95-201)
```python
class XPUW4A8IntLinearKernel(MPLinearKernel):
    """XPU kernel for W4A8 integer quantization using oneDNN int4_gemm_w4a8.

    Weights are symmetric group-quantized int4 packed as uint4.
    Activations are dynamically quantized per-token to symmetric int8.
    """

    @classmethod
    def get_min_capability(cls) -> int:
        return -1

    @classmethod
    def can_implement(cls, c: MPLinearLayerConfig) -> tuple[bool, str | None]:
        if not current_platform.is_xpu():
            return False, "XPUW4A8Int only supported on XPU"
        if c.act_type not in (torch.bfloat16, torch.float16):
            return False, "XPUW4A8Int requires BF16/FP16 activations"
        if c.weight_type != scalar_types.int4:
            return (
                False,
                f"XPUW4A8Int requires int4 weights, got {c.weight_type}",
            )
        if c.zero_points:
            return False, "XPUW4A8Int only supports symmetric weight quantization"
        if c.group_size != -1 and c.group_size % 32 != 0:
            return (
                False,
                f"Group size ({c.group_size}) not supported by XPUW4A8Int, "
                "must be a multiple of 32",
            )
        in_size, out_size = c.partition_weight_shape
        if in_size % 8 != 0 or out_size % 8 != 0:
            return (
                False,
                f"in/out sizes ({in_size}, {out_size}) must be multiples of 8",
            )

        if c.act_type != torch.float16:
            logger.warning_once(
                "XPUW4A8IntLinearKernel is running with model dtype %s, "
                "but int4_gemm_w4a8 produces float16 output. Recommend "
                "setting --dtype float16 for best performance.",
                c.act_type,
            )

        return True, None

    def _pack_int4_weight(self, w: torch.Tensor) -> torch.Tensor:
        # w is [N, K] int8 with values in [-8, 7]
        w_u4 = w.to(torch.int32) + 8  # shift to [0, 15]
        w_u4 = w_u4.reshape(w.shape[0], w.shape[1] // 8, 8)  # [N, K/8, 8]
        shifts = torch.arange(0, 32, 4, dtype=torch.int32, device=w.device)
        packed = ((w_u4 & 0xF) << shifts[None, None, :]).sum(dim=2).to(torch.int32)
        return packed

    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        layer.weight_scale.data = layer.weight_scale.data.t().contiguous()

        device = layer.weight_packed.device
        # TODO: support asymmetric quantization
        weight_zero_point = torch.tensor([8], dtype=torch.int8, device=device)
        layer.weight_zero_point = Parameter(weight_zero_point, requires_grad=False)

        # weight_packed is [out, in] int8, signed int4 values in [-8, 7]
        w = layer.weight_packed.data  # [out, in]

        # TODO: implement asym case
        packed = self._pack_int4_weight(w)  # [out, in/8] packed uint4

        replace_parameter(
            layer,
            self.w_q_name,
            torch.nn.Parameter(packed, requires_grad=False),
        )

        # Free the original unpacked int8 weight (still registered as "weight")
        # to avoid double-storing both int8 [N, K] and int32 [N, K/8] in memory.
        layer.register_parameter("weight", None)

    def apply_weights(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        reshaped_x = x.reshape(-1, x.shape[-1])  # [M, K]
        from vllm._xpu_ops import xpu_ops as ops

        # TODO: static and asymmetric quantization case
        # Common code for CompressedTensorsW4A8Int does not read act symmetry data
        quant_x, x_scale, x_zero = ops.dynamic_per_token_int8_quant_ref(
            reshaped_x, True, 8
        )

        out = torch.ops._xpu_C.int4_gemm_w4a8(
            quant_x,
            x_scale,
            x_zero,
            layer.weight_packed.t(),
            layer.weight_scale,
            layer.weight_zero_point,
            self.config.group_size,
            None,  # g_idx not currently supported
            bias,
        )

        return out.to(x.dtype)
```
**EN:** This kernel class defines `XPUW4A8IntLinearKernel`. XPU kernel for W4A8 integer quantization using oneDNN int4_gemm_w4a8. It inherits from `MPLinearKernel`. Key methods include `get_min_capability`, `can_implement`, `_pack_int4_weight`, `process_weights_after_loading`, `apply_weights`.
**CN:** 该内核类定义了 `XPUW4A8IntLinearKernel`。 它主要负责与 `XPUW4A8IntLinearKernel` 对应的数据组织、接口约束或执行流程。 它继承自 `MPLinearKernel`。 关键方法包括 `get_min_capability`, `can_implement`, `_pack_int4_weight`, `process_weights_after_loading`, `apply_weights`。

### Method `XPUW4A8IntLinearKernel.get_min_capability` (lines 103-104)
```python
    def get_min_capability(cls) -> int:
        return -1
```
**EN:** This method implements `XPUW4A8IntLinearKernel.get_min_capability`, encapsulating a focused unit of runtime logic for the module.
**CN:** 该方法 `XPUW4A8IntLinearKernel.get_min_capability` 封装了此模块中的一段关键运行时逻辑，重点处理 get min capability 相关工作。

### Method `XPUW4A8IntLinearKernel.can_implement` (lines 107-140)
```python
    def can_implement(cls, c: MPLinearLayerConfig) -> tuple[bool, str | None]:
        if not current_platform.is_xpu():
            return False, "XPUW4A8Int only supported on XPU"
        if c.act_type not in (torch.bfloat16, torch.float16):
            return False, "XPUW4A8Int requires BF16/FP16 activations"
        if c.weight_type != scalar_types.int4:
            return (
                False,
                f"XPUW4A8Int requires int4 weights, got {c.weight_type}",
            )
        if c.zero_points:
            return False, "XPUW4A8Int only supports symmetric weight quantization"
        if c.group_size != -1 and c.group_size % 32 != 0:
            return (
                False,
                f"Group size ({c.group_size}) not supported by XPUW4A8Int, "
                "must be a multiple of 32",
            )
        in_size, out_size = c.partition_weight_shape
        if in_size % 8 != 0 or out_size % 8 != 0:
            return (
                False,
                f"in/out sizes ({in_size}, {out_size}) must be multiples of 8",
            )

        if c.act_type != torch.float16:
            logger.warning_once(
                "XPUW4A8IntLinearKernel is running with model dtype %s, "
                "but int4_gemm_w4a8 produces float16 output. Recommend "
                "setting --dtype float16 for best performance.",
                c.act_type,
            )

        return True, None
```
**EN:** This method implements `XPUW4A8IntLinearKernel.can_implement`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `is_xpu`, `warning_once`.
**CN:** 该方法 `XPUW4A8IntLinearKernel.can_implement` 封装了此模块中的一段关键运行时逻辑，重点处理 can implement 相关工作。 它内部会调用 `is_xpu`, `warning_once` 等例程。

### Method `XPUW4A8IntLinearKernel._pack_int4_weight` (lines 142-148)
```python
    def _pack_int4_weight(self, w: torch.Tensor) -> torch.Tensor:
        # w is [N, K] int8 with values in [-8, 7]
        w_u4 = w.to(torch.int32) + 8  # shift to [0, 15]
        w_u4 = w_u4.reshape(w.shape[0], w.shape[1] // 8, 8)  # [N, K/8, 8]
        shifts = torch.arange(0, 32, 4, dtype=torch.int32, device=w.device)
        packed = ((w_u4 & 0xF) << shifts[None, None, :]).sum(dim=2).to(torch.int32)
        return packed
```
**EN:** This method implements `XPUW4A8IntLinearKernel._pack_int4_weight`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `reshape`, `arange`, `to`, `sum`.
**CN:** 该方法 `XPUW4A8IntLinearKernel._pack_int4_weight` 封装了此模块中的一段关键运行时逻辑，重点处理 pack int 4 weight 相关工作。 它内部会调用 `reshape`, `arange`, `to`, `sum` 等例程。

### Method `XPUW4A8IntLinearKernel.process_weights_after_loading` (lines 150-172)
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        layer.weight_scale.data = layer.weight_scale.data.t().contiguous()

        device = layer.weight_packed.device
        # TODO: support asymmetric quantization
        weight_zero_point = torch.tensor([8], dtype=torch.int8, device=device)
        layer.weight_zero_point = Parameter(weight_zero_point, requires_grad=False)

        # weight_packed is [out, in] int8, signed int4 values in [-8, 7]
        w = layer.weight_packed.data  # [out, in]

        # TODO: implement asym case
        packed = self._pack_int4_weight(w)  # [out, in/8] packed uint4

        replace_parameter(
            layer,
            self.w_q_name,
            torch.nn.Parameter(packed, requires_grad=False),
        )

        # Free the original unpacked int8 weight (still registered as "weight")
        # to avoid double-storing both int8 [N, K] and int32 [N, K/8] in memory.
        layer.register_parameter("weight", None)
```
**EN:** This method implements `XPUW4A8IntLinearKernel.process_weights_after_loading`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `contiguous`, `tensor`, `Parameter`, `_pack_int4_weight`.
**CN:** 该方法 `XPUW4A8IntLinearKernel.process_weights_after_loading` 封装了此模块中的一段关键运行时逻辑，重点处理 process weights after loading 相关工作。 它内部会调用 `contiguous`, `tensor`, `Parameter`, `_pack_int4_weight` 等例程。

### Method `XPUW4A8IntLinearKernel.apply_weights` (lines 174-201)
```python
    def apply_weights(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        reshaped_x = x.reshape(-1, x.shape[-1])  # [M, K]
        from vllm._xpu_ops import xpu_ops as ops

        # TODO: static and asymmetric quantization case
        # Common code for CompressedTensorsW4A8Int does not read act symmetry data
        quant_x, x_scale, x_zero = ops.dynamic_per_token_int8_quant_ref(
            reshaped_x, True, 8
        )

        out = torch.ops._xpu_C.int4_gemm_w4a8(
            quant_x,
            x_scale,
            x_zero,
            layer.weight_packed.t(),
            layer.weight_scale,
            layer.weight_zero_point,
            self.config.group_size,
            None,  # g_idx not currently supported
            bias,
        )

        return out.to(x.dtype)
```
**EN:** This method implements `XPUW4A8IntLinearKernel.apply_weights`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `reshape`, `dynamic_per_token_int8_quant_ref`, `int4_gemm_w4a8`, `to`.
**CN:** 该方法 `XPUW4A8IntLinearKernel.apply_weights` 封装了此模块中的一段关键运行时逻辑，重点处理 apply weights 相关工作。 它内部会调用 `reshape`, `dynamic_per_token_int8_quant_ref`, `int4_gemm_w4a8`, `to` 等例程。

## Key Concepts / 关键概念
- **XPU support / XPU 支持**
  - **EN:** The file adds Intel XPU-specific dispatch, operators, or compatibility checks.
  - **CN:** 该文件加入 Intel XPU 专用的分派、算子或兼容性检查。
- **INT8 quantization / INT8 量化**
  - **EN:** The implementation contains INT8 scaling, packing, or matrix-multiplication logic.
  - **CN:** 该实现包含 INT8 的缩放、打包或矩阵乘逻辑。
- **Mixed-precision linear kernels / 混合精度线性内核**
  - **EN:** The file implements mixed-precision linear layers over several backend providers.
  - **CN:** 该文件实现基于多个后端提供者的混合精度线性层。
- **Linear kernel abstraction / 线性内核抽象**
  - **EN:** The module defines abstractions or selectors for quantized linear operators.
  - **CN:** 该模块定义量化线性算子的抽象或选择器。
- **Kernel registration / 内核注册**
  - **EN:** The file registers operators, backends, or preset configurations with PyTorch/vLLM.
  - **CN:** 该文件向 PyTorch/vLLM 注册算子、后端或预设配置。

## Dependencies / 依赖关系
- **vLLM internal / vLLM 内部依赖**: `vllm.logger`, `vllm.model_executor.layers.quantization.utils`, `vllm.platforms`, `vllm.scalar_type`, `.MPLinearKernel`
- **External / 外部依赖**: `torch`, `torch.nn.parameter`
