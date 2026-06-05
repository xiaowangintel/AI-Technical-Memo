# conch.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/kernels/linear/mixed_precision/conch.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the Conch mixed-precision linear kernel. / 实现 Conch 混合精度线性内核。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-12)
```python
from importlib.util import find_spec
from typing import Final

import torch

from vllm.model_executor.parameter import BasevLLMParameter, permute_param_layout_
from vllm.scalar_type import scalar_types

from .MPLinearKernel import MPLinearKernel, MPLinearLayerConfig
```
**EN:** This import block loads `importlib.util`, `typing`, `torch`, `vllm.model_executor.parameter`, `vllm.scalar_type`, `.MPLinearKernel`, establishing the tensor libraries, backend hooks, and vLLM helpers needed by the rest of `conch.py`.
**CN:** 该导入代码块加载了 `importlib.util`, `typing`, `torch`, `vllm.model_executor.parameter`, `vllm.scalar_type`, `.MPLinearKernel`，为 `conch.py` 后续逻辑准备张量库、后端钩子以及 vLLM 辅助工具。

### Constants / assignments (lines 14-20)
```python
_CONCH_SUPPORTED_WEIGHT_TYPES: Final = [
    scalar_types.uint4,
    scalar_types.uint8,
    scalar_types.uint4b8,
    scalar_types.uint8b128,
]
_CONCH_SUPPORTED_GROUP_SIZES: Final = [-1, 128]
```
**EN:** This block defines module constants (`_CONCH_SUPPORTED_WEIGHT_TYPES`, `_CONCH_SUPPORTED_GROUP_SIZES`) that encode defaults, feature flags, or operator metadata used later in the file.
**CN:** 该代码块定义了模块常量（`_CONCH_SUPPORTED_WEIGHT_TYPES`, `_CONCH_SUPPORTED_GROUP_SIZES`），用于表示默认值、特性开关或后续逻辑要使用的算子元数据。

### Class `ConchLinearKernel` (lines 23-153)
```python
class ConchLinearKernel(MPLinearKernel):
    @classmethod
    def get_min_capability(cls) -> int:
        return 80

    @classmethod
    def can_implement(cls, c: MPLinearLayerConfig) -> tuple[bool, str | None]:
        if c.weight_type not in _CONCH_SUPPORTED_WEIGHT_TYPES:
            error_msg = (
                f"Weight type ({c.weight_type}) not supported by "
                "ConchLinearKernel, supported types are: "
                f"{_CONCH_SUPPORTED_WEIGHT_TYPES}"
            )
            return False, error_msg

        if c.group_size not in _CONCH_SUPPORTED_GROUP_SIZES:
            error_msg = (
                f"Group size ({c.group_size}) not supported by "
                "ConchLinearKernel, supported group sizes are: "
                f"{_CONCH_SUPPORTED_GROUP_SIZES}"
            )
            return False, error_msg

        if find_spec("conch") is None:
            error_msg = (
                "conch-triton-kernels is not installed, please "
                "install it via `pip install conch-triton-kernels` "
                "and try again!"
            )
            return False, error_msg

        return True, None

    # note assumes that
    #  `weight_packed` is: {input_dim = 0, output_dim = 1, packed_dim = 0}
    #  `weight_scale` is: {input_dim = 0, output_dim = 1}
    #  `weight_zero_point` is: {input_dim = 1, output_dim = 0, packed_dim = 0}
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        def transform_w_q(x):
            assert isinstance(x, BasevLLMParameter)
            permute_param_layout_(x, input_dim=0, output_dim=1, packed_dim=0)
            x.data = x.data.contiguous()
            return x

        def transform_w_s(x):
            assert isinstance(x, BasevLLMParameter)
            permute_param_layout_(x, input_dim=0, output_dim=1)
            x.data = x.data.contiguous()
            return x

        def transform_w_zp(x):
            # Zero points are stored PACKED as [N//pack_factor, K//G]
            # The Conch kernel expects UNPACKED zeros: [K//G, N]
            # We need to unpack and reorder
            assert isinstance(x, BasevLLMParameter)
            packed = x.data  # shape: [N//pack_factor, K//G], dtype: int32

            # Determine packing based on weight bit width
            size_bits = self.config.weight_type.size_bits
            pack_factor = 32 // size_bits  # 8 for 4-bit, 4 for 8-bit
            mask = (1 << size_bits) - 1  # 0xF for 4-bit, 0xFF for 8-bit

            n_packed, k_groups = packed.shape
            n_full = n_packed * pack_factor

            # Unpack using vectorized bitwise ops
            # shifts = [0, size_bits, 2*size_bits, ...] for each packed position
            shifts = torch.arange(
                0, 32, size_bits, dtype=torch.int32, device=packed.device
            )
            # packed: [N//pack_factor, K//G] -> [N//pack_factor, K//G, 1]
            # shifts: [pack_factor] -> [1, 1, pack_factor]
            # Result: [N//pack_factor, K//G, pack_factor]
            unpacked = (packed.unsqueeze(-1) >> shifts) & mask

            # Permute to [K//G, N//pack_factor, pack_factor] then reshape to [K//G, N]
            unpacked = unpacked.permute(1, 0, 2).reshape(k_groups, n_full)

            x.data = unpacked.to(torch.uint8).contiguous()

            # Update metadata - zeros are no longer packed
            if hasattr(x, "_input_dim"):
                x._input_dim = 0
            if hasattr(x, "_output_dim"):
                x._output_dim = 1
            if hasattr(x, "_packed_factor"):
                x._packed_factor = 1
            return x

        self._transform_param(layer, self.w_q_name, transform_w_q)
        self._transform_param(layer, self.w_s_name, transform_w_s)
        if self.config.zero_points:
            self._transform_param(layer, self.w_zp_name, transform_w_zp)
        elif self.w_zp_name is not None:
            layer.register_parameter(self.w_zp_name, None)

    def apply_weights(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        from conch.ops.quantization.gemm import mixed_precision_gemm

        w_q, w_s, w_zp, _ = self._get_weight_params(layer)

        # Map channelwise group_size=-1 to the actual input dimension K.
        # The conch kernel computes stride_mul = block_k / group_size;
        # passing -1 produces a negative stride that reads out-of-bounds
        # scale values for all K-blocks after the first.
        group_size = self.config.group_size
        if group_size == -1:
            group_size = x.shape[-1]

        x_2d = x.reshape(-1, x.shape[-1])
        out_shape = x.shape[:-1] + (self.config.partition_weight_shape[1],)

        output = mixed_precision_gemm(
            x=x_2d,
            w_q_packed=w_q.data,
            w_s=w_s.data,
            w_zp=w_zp.data if w_zp is not None else None,
            weight_size_bits=self.config.weight_type.size_bits,
            weight_bias=self.config.weight_type.bias,
            group_size=group_size,
        )

        if bias is not None:
            output.add_(bias)  # In-place add

        return output.reshape(out_shape)
```
**EN:** This kernel class defines `ConchLinearKernel`. It organizes the state and behavior needed by this kernel path. It inherits from `MPLinearKernel`. Key methods include `get_min_capability`, `can_implement`, `process_weights_after_loading`, `apply_weights`.
**CN:** 该内核类定义了 `ConchLinearKernel`。 它组织了该内核路径所需的状态与行为。 它继承自 `MPLinearKernel`。 关键方法包括 `get_min_capability`, `can_implement`, `process_weights_after_loading`, `apply_weights`。

### Method `ConchLinearKernel.get_min_capability` (lines 25-26)
```python
    def get_min_capability(cls) -> int:
        return 80
```
**EN:** This method implements `ConchLinearKernel.get_min_capability`, encapsulating a focused unit of runtime logic for the module.
**CN:** 该方法 `ConchLinearKernel.get_min_capability` 封装了此模块中的一段关键运行时逻辑，重点处理 get min capability 相关工作。

### Method `ConchLinearKernel.can_implement` (lines 29-54)
```python
    def can_implement(cls, c: MPLinearLayerConfig) -> tuple[bool, str | None]:
        if c.weight_type not in _CONCH_SUPPORTED_WEIGHT_TYPES:
            error_msg = (
                f"Weight type ({c.weight_type}) not supported by "
                "ConchLinearKernel, supported types are: "
                f"{_CONCH_SUPPORTED_WEIGHT_TYPES}"
            )
            return False, error_msg

        if c.group_size not in _CONCH_SUPPORTED_GROUP_SIZES:
            error_msg = (
                f"Group size ({c.group_size}) not supported by "
                "ConchLinearKernel, supported group sizes are: "
                f"{_CONCH_SUPPORTED_GROUP_SIZES}"
            )
            return False, error_msg

        if find_spec("conch") is None:
            error_msg = (
                "conch-triton-kernels is not installed, please "
                "install it via `pip install conch-triton-kernels` "
                "and try again!"
            )
            return False, error_msg

        return True, None
```
**EN:** This method implements `ConchLinearKernel.can_implement`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `find_spec`.
**CN:** 该方法 `ConchLinearKernel.can_implement` 封装了此模块中的一段关键运行时逻辑，重点处理 can implement 相关工作。 它内部会调用 `find_spec` 等例程。

### Method `ConchLinearKernel.process_weights_after_loading` (lines 60-117)
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        def transform_w_q(x):
            assert isinstance(x, BasevLLMParameter)
            permute_param_layout_(x, input_dim=0, output_dim=1, packed_dim=0)
            x.data = x.data.contiguous()
            return x

        def transform_w_s(x):
            assert isinstance(x, BasevLLMParameter)
            permute_param_layout_(x, input_dim=0, output_dim=1)
            x.data = x.data.contiguous()
            return x

        def transform_w_zp(x):
            # Zero points are stored PACKED as [N//pack_factor, K//G]
            # The Conch kernel expects UNPACKED zeros: [K//G, N]
            # We need to unpack and reorder
            assert isinstance(x, BasevLLMParameter)
            packed = x.data  # shape: [N//pack_factor, K//G], dtype: int32

            # Determine packing based on weight bit width
            size_bits = self.config.weight_type.size_bits
            pack_factor = 32 // size_bits  # 8 for 4-bit, 4 for 8-bit
            mask = (1 << size_bits) - 1  # 0xF for 4-bit, 0xFF for 8-bit

            n_packed, k_groups = packed.shape
            n_full = n_packed * pack_factor

            # Unpack using vectorized bitwise ops
            # shifts = [0, size_bits, 2*size_bits, ...] for each packed position
            shifts = torch.arange(
                0, 32, size_bits, dtype=torch.int32, device=packed.device
            )
            # packed: [N//pack_factor, K//G] -> [N//pack_factor, K//G, 1]
            # shifts: [pack_factor] -> [1, 1, pack_factor]
            # Result: [N//pack_factor, K//G, pack_factor]
            unpacked = (packed.unsqueeze(-1) >> shifts) & mask

            # Permute to [K//G, N//pack_factor, pack_factor] then reshape to [K//G, N]
            unpacked = unpacked.permute(1, 0, 2).reshape(k_groups, n_full)

            x.data = unpacked.to(torch.uint8).contiguous()

            # Update metadata - zeros are no longer packed
            if hasattr(x, "_input_dim"):
                x._input_dim = 0
            if hasattr(x, "_output_dim"):
                x._output_dim = 1
            if hasattr(x, "_packed_factor"):
                x._packed_factor = 1
            return x

        self._transform_param(layer, self.w_q_name, transform_w_q)
        self._transform_param(layer, self.w_s_name, transform_w_s)
        if self.config.zero_points:
            self._transform_param(layer, self.w_zp_name, transform_w_zp)
        elif self.w_zp_name is not None:
            layer.register_parameter(self.w_zp_name, None)
```
**EN:** This method implements `ConchLinearKernel.process_weights_after_loading`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `_transform_param`, `isinstance`, `permute_param_layout_`, `contiguous`.
**CN:** 该方法 `ConchLinearKernel.process_weights_after_loading` 封装了此模块中的一段关键运行时逻辑，重点处理 process weights after loading 相关工作。 它内部会调用 `_transform_param`, `isinstance`, `permute_param_layout_`, `contiguous` 等例程。

### Method `ConchLinearKernel.apply_weights` (lines 119-153)
```python
    def apply_weights(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        from conch.ops.quantization.gemm import mixed_precision_gemm

        w_q, w_s, w_zp, _ = self._get_weight_params(layer)

        # Map channelwise group_size=-1 to the actual input dimension K.
        # The conch kernel computes stride_mul = block_k / group_size;
        # passing -1 produces a negative stride that reads out-of-bounds
        # scale values for all K-blocks after the first.
        group_size = self.config.group_size
        if group_size == -1:
            group_size = x.shape[-1]

        x_2d = x.reshape(-1, x.shape[-1])
        out_shape = x.shape[:-1] + (self.config.partition_weight_shape[1],)

        output = mixed_precision_gemm(
            x=x_2d,
            w_q_packed=w_q.data,
            w_s=w_s.data,
            w_zp=w_zp.data if w_zp is not None else None,
            weight_size_bits=self.config.weight_type.size_bits,
            weight_bias=self.config.weight_type.bias,
            group_size=group_size,
        )

        if bias is not None:
            output.add_(bias)  # In-place add

        return output.reshape(out_shape)
```
**EN:** This method implements `ConchLinearKernel.apply_weights`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `_get_weight_params`, `reshape`, `mixed_precision_gemm`, `add_`.
**CN:** 该方法 `ConchLinearKernel.apply_weights` 封装了此模块中的一段关键运行时逻辑，重点处理 apply weights 相关工作。 它内部会调用 `_get_weight_params`, `reshape`, `mixed_precision_gemm`, `add_` 等例程。

## Key Concepts / 关键概念
- **Triton kernels / Triton 内核**
  - **EN:** The implementation relies on Triton-generated GPU kernels for low-level tensor work.
  - **CN:** 该实现依赖 Triton 生成的 GPU 内核完成底层张量计算。
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
- **vLLM internal / vLLM 内部依赖**: `vllm.model_executor.parameter`, `vllm.scalar_type`, `.MPLinearKernel`
- **External / 外部依赖**: `importlib.util`, `typing`, `torch`
