# dynamic_4bit.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/kernels/linear/mixed_precision/dynamic_4bit.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the dynamic 4-bit mixed-precision linear kernel. / 实现 动态 4bit 混合精度线性内核。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 5-11)
```python
import torch

from vllm.model_executor.layers.quantization.utils import replace_parameter
from vllm.platforms import CpuArchEnum, current_platform
from vllm.scalar_type import scalar_types

from .MPLinearKernel import MPLinearKernel, MPLinearLayerConfig
```
**EN:** This import block loads `torch`, `vllm.model_executor.layers.quantization.utils`, `vllm.platforms`, `vllm.scalar_type`, `.MPLinearKernel`, establishing the tensor libraries, backend hooks, and vLLM helpers needed by the rest of `dynamic_4bit.py`.
**CN:** 该导入代码块加载了 `torch`, `vllm.model_executor.layers.quantization.utils`, `vllm.platforms`, `vllm.scalar_type`, `.MPLinearKernel`，为 `dynamic_4bit.py` 后续逻辑准备张量库、后端钩子以及 vLLM 辅助工具。

### Class `Dynamic4bitLinearKernel` (lines 26-159)
```python
class Dynamic4bitLinearKernel(MPLinearKernel):
    SUPPORTED_QUANT_TYPES = [scalar_types.int4]

    @classmethod
    def get_min_capability(cls) -> int:
        return 1

    @classmethod
    def can_implement(cls, c: MPLinearLayerConfig) -> tuple[bool, str | None]:
        if not current_platform.is_cpu():
            return False, "Only CPU is supported"
        if c.weight_type not in cls.SUPPORTED_QUANT_TYPES:
            return False, f"Unsupported quant type {c.weight_type}"
        if (
            current_platform.get_cpu_architecture() == CpuArchEnum.ARM
            and c.act_type
            not in [
                torch.float32,
                torch.bfloat16,
                torch.float16,
            ]
        ):
            return (
                False,
                "Dynamic4bitLinearKernel on Arm requires Float32 or"
                " BFloat16 or Float16 activations",
            )
        if c.full_weight_shape[0] % c.group_size != 0:
            return (
                False,
                f"Group size ({c.group_size}) does not evenly divide"
                " the number of input features "
                f"({c.full_weight_shape[0]})",
            )
        if current_platform.get_cpu_architecture() == CpuArchEnum.ARM:
            try:
                # Attempt to retrieve the operation
                _ = torch.ops.aten._dyn_quant_matmul_4bit
            except AttributeError:
                return (
                    False,
                    f"PyTorch {torch.__version__} does not support"
                    " _dyn_quant_matmul_4bit. Install a newer version",
                )
        return True, None

    def process_weights_after_loading(self, layer: torch.nn.Module):
        c = self.config
        packed_weight = getattr(layer, self.w_q_name)
        packed_weight = packed_weight.add(8)
        uint8_packed = (packed_weight[::, 1::2] << 4 | packed_weight[::, ::2]).to(
            torch.uint8
        )

        scales = getattr(layer, self.w_s_name)
        block_size = c.group_size

        # Handle scaling factors for partitioned weights
        if block_size == c.partition_weight_shape[0]:
            scales = scales.to(
                torch.float32
            )  # Float32 & Bfloat16 variants requires float32 scales
            scales = scales.view(-1, 1)  # Channel-wise scales
            if layer.bias is not None:
                # Float32 & Bfloat16 variants requires float32 bias
                replace_parameter(
                    layer,
                    "bias",
                    torch.nn.Parameter(
                        layer.bias.to(torch.float32), requires_grad=False
                    ),
                )
        else:
            # KleidiAI kernel requires bfloat16 scales with groupwise scheme
            scales = scales.to(torch.bfloat16)

        # Repack weights as per kernel requirement
        w = torch.ops.aten._dyn_quant_pack_4bit_weight(
            uint8_packed,
            scales,
            layer.bias,
            block_size,
            c.partition_weight_shape[0],
            c.partition_weight_shape[1],
        )
        replace_parameter(
            layer, self.w_q_name, torch.nn.Parameter(w, requires_grad=False)
        )
        setattr(layer, self.w_s_name, None)

    def apply_weights(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        # PyTorch / KleidiAI kernels natively support the following configs:
        # - channelwise with bfloat16 / float32 activations
        # - groupwise with float32 activations
        # To support:
        # - groupwise with bfloat16/float16 activations: we need to upcast
        #   activations to float32 before matmul and downcast back to bfloat16/float16
        # - channelwise with float16 activations, we need to upcast activations to
        #   float32 before matmul and downcast back to float16
        # Note: these activations will be dynamically quantized to int8 by the kernel.

        c = self.config
        is_groupwise = c.group_size != c.partition_weight_shape[0]
        # dtype of activations before they get dynamically quantized to int8
        original_pre_quant_act_dtype = x.dtype
        pre_quant_act_dtype = original_pre_quant_act_dtype
        if (
            is_groupwise and pre_quant_act_dtype == torch.bfloat16
        ) or pre_quant_act_dtype == torch.float16:
            pre_quant_act_dtype = torch.float32

        x_2d = x.reshape(-1, x.shape[-1])
        if pre_quant_act_dtype != original_pre_quant_act_dtype:
            x_2d = x_2d.to(pre_quant_act_dtype)

        out_shape = x.shape[:-1] + (c.partition_weight_shape[1],)

        w_q = getattr(layer, self.w_q_name)
        output = torch.ops.aten._dyn_quant_matmul_4bit(
            x_2d,
            w_q,
            c.group_size,
            c.partition_weight_shape[0],
            c.partition_weight_shape[1],
        ).reshape(out_shape)

        if pre_quant_act_dtype != original_pre_quant_act_dtype:
            output = output.to(original_pre_quant_act_dtype)
        return output
```
**EN:** This kernel class defines `Dynamic4bitLinearKernel`. It organizes the state and behavior needed by this kernel path. It inherits from `MPLinearKernel`. Key methods include `get_min_capability`, `can_implement`, `process_weights_after_loading`, `apply_weights`.
**CN:** 该内核类定义了 `Dynamic4bitLinearKernel`。 它组织了该内核路径所需的状态与行为。 它继承自 `MPLinearKernel`。 关键方法包括 `get_min_capability`, `can_implement`, `process_weights_after_loading`, `apply_weights`。

### Method `Dynamic4bitLinearKernel.get_min_capability` (lines 30-31)
```python
    def get_min_capability(cls) -> int:
        return 1
```
**EN:** This method implements `Dynamic4bitLinearKernel.get_min_capability`, encapsulating a focused unit of runtime logic for the module.
**CN:** 该方法 `Dynamic4bitLinearKernel.get_min_capability` 封装了此模块中的一段关键运行时逻辑，重点处理 get min capability 相关工作。

### Method `Dynamic4bitLinearKernel.can_implement` (lines 34-70)
```python
    def can_implement(cls, c: MPLinearLayerConfig) -> tuple[bool, str | None]:
        if not current_platform.is_cpu():
            return False, "Only CPU is supported"
        if c.weight_type not in cls.SUPPORTED_QUANT_TYPES:
            return False, f"Unsupported quant type {c.weight_type}"
        if (
            current_platform.get_cpu_architecture() == CpuArchEnum.ARM
            and c.act_type
            not in [
                torch.float32,
                torch.bfloat16,
                torch.float16,
            ]
        ):
            return (
                False,
                "Dynamic4bitLinearKernel on Arm requires Float32 or"
                " BFloat16 or Float16 activations",
            )
        if c.full_weight_shape[0] % c.group_size != 0:
            return (
                False,
                f"Group size ({c.group_size}) does not evenly divide"
                " the number of input features "
                f"({c.full_weight_shape[0]})",
            )
        if current_platform.get_cpu_architecture() == CpuArchEnum.ARM:
            try:
                # Attempt to retrieve the operation
                _ = torch.ops.aten._dyn_quant_matmul_4bit
            except AttributeError:
                return (
                    False,
                    f"PyTorch {torch.__version__} does not support"
                    " _dyn_quant_matmul_4bit. Install a newer version",
                )
        return True, None
```
**EN:** This method implements `Dynamic4bitLinearKernel.can_implement`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `is_cpu`, `get_cpu_architecture`.
**CN:** 该方法 `Dynamic4bitLinearKernel.can_implement` 封装了此模块中的一段关键运行时逻辑，重点处理 can implement 相关工作。 它内部会调用 `is_cpu`, `get_cpu_architecture` 等例程。

### Method `Dynamic4bitLinearKernel.process_weights_after_loading` (lines 72-114)
```python
    def process_weights_after_loading(self, layer: torch.nn.Module):
        c = self.config
        packed_weight = getattr(layer, self.w_q_name)
        packed_weight = packed_weight.add(8)
        uint8_packed = (packed_weight[::, 1::2] << 4 | packed_weight[::, ::2]).to(
            torch.uint8
        )

        scales = getattr(layer, self.w_s_name)
        block_size = c.group_size

        # Handle scaling factors for partitioned weights
        if block_size == c.partition_weight_shape[0]:
            scales = scales.to(
                torch.float32
            )  # Float32 & Bfloat16 variants requires float32 scales
            scales = scales.view(-1, 1)  # Channel-wise scales
            if layer.bias is not None:
                # Float32 & Bfloat16 variants requires float32 bias
                replace_parameter(
                    layer,
                    "bias",
                    torch.nn.Parameter(
                        layer.bias.to(torch.float32), requires_grad=False
                    ),
                )
        else:
            # KleidiAI kernel requires bfloat16 scales with groupwise scheme
            scales = scales.to(torch.bfloat16)

        # Repack weights as per kernel requirement
        w = torch.ops.aten._dyn_quant_pack_4bit_weight(
            uint8_packed,
            scales,
            layer.bias,
            block_size,
            c.partition_weight_shape[0],
            c.partition_weight_shape[1],
        )
        replace_parameter(
            layer, self.w_q_name, torch.nn.Parameter(w, requires_grad=False)
        )
        setattr(layer, self.w_s_name, None)
```
**EN:** This method implements `Dynamic4bitLinearKernel.process_weights_after_loading`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `getattr`, `add`, `to`, `_dyn_quant_pack_4bit_weight`.
**CN:** 该方法 `Dynamic4bitLinearKernel.process_weights_after_loading` 封装了此模块中的一段关键运行时逻辑，重点处理 process weights after loading 相关工作。 它内部会调用 `getattr`, `add`, `to`, `_dyn_quant_pack_4bit_weight` 等例程。

### Method `Dynamic4bitLinearKernel.apply_weights` (lines 116-159)
```python
    def apply_weights(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        # PyTorch / KleidiAI kernels natively support the following configs:
        # - channelwise with bfloat16 / float32 activations
        # - groupwise with float32 activations
        # To support:
        # - groupwise with bfloat16/float16 activations: we need to upcast
        #   activations to float32 before matmul and downcast back to bfloat16/float16
        # - channelwise with float16 activations, we need to upcast activations to
        #   float32 before matmul and downcast back to float16
        # Note: these activations will be dynamically quantized to int8 by the kernel.

        c = self.config
        is_groupwise = c.group_size != c.partition_weight_shape[0]
        # dtype of activations before they get dynamically quantized to int8
        original_pre_quant_act_dtype = x.dtype
        pre_quant_act_dtype = original_pre_quant_act_dtype
        if (
            is_groupwise and pre_quant_act_dtype == torch.bfloat16
        ) or pre_quant_act_dtype == torch.float16:
            pre_quant_act_dtype = torch.float32

        x_2d = x.reshape(-1, x.shape[-1])
        if pre_quant_act_dtype != original_pre_quant_act_dtype:
            x_2d = x_2d.to(pre_quant_act_dtype)

        out_shape = x.shape[:-1] + (c.partition_weight_shape[1],)

        w_q = getattr(layer, self.w_q_name)
        output = torch.ops.aten._dyn_quant_matmul_4bit(
            x_2d,
            w_q,
            c.group_size,
            c.partition_weight_shape[0],
            c.partition_weight_shape[1],
        ).reshape(out_shape)

        if pre_quant_act_dtype != original_pre_quant_act_dtype:
            output = output.to(original_pre_quant_act_dtype)
        return output
```
**EN:** This method implements `Dynamic4bitLinearKernel.apply_weights`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `reshape`, `getattr`, `to`, `_dyn_quant_matmul_4bit`.
**CN:** 该方法 `Dynamic4bitLinearKernel.apply_weights` 封装了此模块中的一段关键运行时逻辑，重点处理 apply weights 相关工作。 它内部会调用 `reshape`, `getattr`, `to`, `_dyn_quant_matmul_4bit` 等例程。

## Key Concepts / 关键概念
- **INT8 quantization / INT8 量化**
  - **EN:** The implementation contains INT8 scaling, packing, or matrix-multiplication logic.
  - **CN:** 该实现包含 INT8 的缩放、打包或矩阵乘逻辑。
- **Mixed-precision linear kernels / 混合精度线性内核**
  - **EN:** The file implements mixed-precision linear layers over several backend providers.
  - **CN:** 该文件实现基于多个后端提供者的混合精度线性层。
- **Linear kernel abstraction / 线性内核抽象**
  - **EN:** The module defines abstractions or selectors for quantized linear operators.
  - **CN:** 该模块定义量化线性算子的抽象或选择器。

## Dependencies / 依赖关系
- **vLLM internal / vLLM 内部依赖**: `vllm.model_executor.layers.quantization.utils`, `vllm.platforms`, `vllm.scalar_type`, `.MPLinearKernel`
- **External / 外部依赖**: `torch`
