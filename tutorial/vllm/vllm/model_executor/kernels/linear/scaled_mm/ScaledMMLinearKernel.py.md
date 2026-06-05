# ScaledMMLinearKernel.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/kernels/linear/scaled_mm/ScaledMMLinearKernel.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Defines the shared abstractions for scaled matrix-multiplication kernels. / 定义缩放矩阵乘内核共享的抽象层。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-17)
```python
from abc import ABC, abstractmethod
from collections.abc import Sequence
from dataclasses import dataclass
from typing import Generic, TypeVar

import torch

from vllm.model_executor.layers.quantization.input_quant_fp8 import QuantFP8
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    QuantKey,
)
from vllm.platforms import current_platform

from ..base import MMLinearLayerConfig
```
**EN:** This import block loads `abc`, `collections.abc`, `dataclasses`, `typing`, `torch`, `vllm.model_executor.layers.quantization.input_quant_fp8`, ..., establishing the tensor libraries, backend hooks, and vLLM helpers needed by the rest of `ScaledMMLinearKernel.py`.
**CN:** 该导入代码块加载了 `abc`, `collections.abc`, `dataclasses`, `typing`, `torch`, `vllm.model_executor.layers.quantization.input_quant_fp8`, ...，为 `ScaledMMLinearKernel.py` 后续逻辑准备张量库、后端钩子以及 vLLM 辅助工具。

### Class `Int8ScaledMMLinearLayerConfig` (lines 21-25)
```python
class Int8ScaledMMLinearLayerConfig(MMLinearLayerConfig):
    # TODO: Change to QuantKey like FP8ScaledMMLinearLayerConfig
    is_static_input_scheme: bool
    is_channelwise: bool
    input_symmetric: bool
```
**EN:** This dataclass defines `Int8ScaledMMLinearLayerConfig`. It organizes the state and behavior needed by this kernel path. It inherits from `MMLinearLayerConfig`.
**CN:** 该数据类定义了 `Int8ScaledMMLinearLayerConfig`。 它组织了该内核路径所需的状态与行为。 它继承自 `MMLinearLayerConfig`。

### Class `FP8ScaledMMLinearLayerConfig` (lines 29-34)
```python
class FP8ScaledMMLinearLayerConfig(MMLinearLayerConfig):
    weight_quant_key: QuantKey
    activation_quant_key: QuantKey
    weight_shape: tuple[int, int]
    input_dtype: torch.dtype
    out_dtype: torch.dtype
```
**EN:** This dataclass defines `FP8ScaledMMLinearLayerConfig`. It organizes the state and behavior needed by this kernel path. It inherits from `MMLinearLayerConfig`.
**CN:** 该数据类定义了 `FP8ScaledMMLinearLayerConfig`。 它组织了该内核路径所需的状态与行为。 它继承自 `MMLinearLayerConfig`。

### Constants / assignments (lines 37-52)
```python
_FP8ParamsT = tuple[
    torch.Tensor,  # weight
    torch.Tensor,  # weight_scale
    torch.Tensor | None,  # input_scale,
    torch.Tensor | None,  # input_scale_ub,
]
_Int8ParamsT = tuple[
    torch.Tensor,  # weight
    torch.Tensor,  # weight_scale
    torch.Tensor | None,  # input_scale,
    torch.Tensor | None,  # input_zp
    torch.Tensor | None,  # azp_adj
]

_ParamsT = TypeVar("_ParamsT", _Int8ParamsT, _FP8ParamsT)
_ConfigT = TypeVar("_ConfigT", bound=MMLinearLayerConfig)
```
**EN:** This assignment block initializes `_FP8ParamsT`, `_Int8ParamsT`, `_ParamsT`, `_ConfigT`, capturing shared state or configuration consumed by later helper functions and kernel classes.
**CN:** 该赋值代码块初始化了 `_FP8ParamsT`, `_Int8ParamsT`, `_ParamsT`, `_ConfigT`，保存后续辅助函数和内核类会消费的共享状态或配置。

### Class `ScaledMMLinearKernel` (lines 55-90)
```python
class ScaledMMLinearKernel(Generic[_ConfigT, _ParamsT], ABC):
    @classmethod
    @abstractmethod
    def is_supported(
        cls, compute_capability: int | None = None
    ) -> tuple[bool, str | None]:
        raise NotImplementedError

    @classmethod
    @abstractmethod
    def can_implement(cls, c: _ConfigT) -> tuple[bool, str | None]:
        raise NotImplementedError

    def __init__(self, c: _ConfigT, layer_param_names: Sequence[str]) -> None:
        assert self.can_implement(c)[0]
        assert self.is_supported()[0]
        self.config = c
        self.layer_param_names = layer_param_names

    @abstractmethod
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        raise NotImplementedError

    @abstractmethod
    def apply_weights(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        raise NotImplementedError

    # return a covariant type in the subclass
    @abstractmethod
    def _get_layer_params(self, layer) -> _ParamsT:
        raise NotImplementedError
```
**EN:** This abstract base class defines `ScaledMMLinearKernel`. It organizes the state and behavior needed by this kernel path. It inherits from `Generic[_ConfigT, _ParamsT]`, `ABC`. Key methods include `is_supported`, `can_implement`, `__init__`, `process_weights_after_loading`, `apply_weights`, ....
**CN:** 该抽象基类定义了 `ScaledMMLinearKernel`。 它组织了该内核路径所需的状态与行为。 它继承自 `Generic[_ConfigT, _ParamsT]`, `ABC`。 关键方法包括 `is_supported`, `can_implement`, `__init__`, `process_weights_after_loading`, `apply_weights`, ...。

### Method `ScaledMMLinearKernel.is_supported` (lines 58-61)
```python
    def is_supported(
        cls, compute_capability: int | None = None
    ) -> tuple[bool, str | None]:
        raise NotImplementedError
```
**EN:** This method implements `ScaledMMLinearKernel.is_supported`, encapsulating a focused unit of runtime logic for the module.
**CN:** 该方法 `ScaledMMLinearKernel.is_supported` 封装了此模块中的一段关键运行时逻辑，重点处理 is supported 相关工作。

### Method `ScaledMMLinearKernel.can_implement` (lines 65-66)
```python
    def can_implement(cls, c: _ConfigT) -> tuple[bool, str | None]:
        raise NotImplementedError
```
**EN:** This method implements `ScaledMMLinearKernel.can_implement`, encapsulating a focused unit of runtime logic for the module.
**CN:** 该方法 `ScaledMMLinearKernel.can_implement` 封装了此模块中的一段关键运行时逻辑，重点处理 can implement 相关工作。

### Method `ScaledMMLinearKernel.__init__` (lines 68-72)
```python
    def __init__(self, c: _ConfigT, layer_param_names: Sequence[str]) -> None:
        assert self.can_implement(c)[0]
        assert self.is_supported()[0]
        self.config = c
        self.layer_param_names = layer_param_names
```
**EN:** This method implements `ScaledMMLinearKernel.__init__`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `can_implement`, `is_supported`.
**CN:** 该方法 `ScaledMMLinearKernel.__init__` 封装了此模块中的一段关键运行时逻辑，重点处理 init 相关工作。 它内部会调用 `can_implement`, `is_supported` 等例程。

### Method `ScaledMMLinearKernel.process_weights_after_loading` (lines 75-76)
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        raise NotImplementedError
```
**EN:** This method implements `ScaledMMLinearKernel.process_weights_after_loading`, encapsulating a focused unit of runtime logic for the module.
**CN:** 该方法 `ScaledMMLinearKernel.process_weights_after_loading` 封装了此模块中的一段关键运行时逻辑，重点处理 process weights after loading 相关工作。

### Method `ScaledMMLinearKernel.apply_weights` (lines 79-85)
```python
    def apply_weights(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        raise NotImplementedError
```
**EN:** This method implements `ScaledMMLinearKernel.apply_weights`, encapsulating a focused unit of runtime logic for the module.
**CN:** 该方法 `ScaledMMLinearKernel.apply_weights` 封装了此模块中的一段关键运行时逻辑，重点处理 apply weights 相关工作。

### Method `ScaledMMLinearKernel._get_layer_params` (lines 89-90)
```python
    def _get_layer_params(self, layer) -> _ParamsT:
        raise NotImplementedError
```
**EN:** This method implements `ScaledMMLinearKernel._get_layer_params`, encapsulating a focused unit of runtime logic for the module.
**CN:** 该方法 `ScaledMMLinearKernel._get_layer_params` 封装了此模块中的一段关键运行时逻辑，重点处理 get layer params 相关工作。

### Class `FP8ScaledMMLinearKernel` (lines 93-172)
```python
class FP8ScaledMMLinearKernel(
    ScaledMMLinearKernel[FP8ScaledMMLinearLayerConfig, _FP8ParamsT], ABC
):
    def __init__(
        self, c: FP8ScaledMMLinearLayerConfig, layer_param_names: Sequence[str]
    ) -> None:
        act_scale_descriptor = c.activation_quant_key.scale
        self.quant_fp8 = QuantFP8(
            static=act_scale_descriptor.static,
            group_shape=act_scale_descriptor.group_shape,
            num_token_padding=self.get_output_padding(),
        )
        self.fp8_dtype = current_platform.fp8_dtype()
        super().__init__(c, layer_param_names)

    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        pass

    def _get_layer_params(self, layer) -> _FP8ParamsT:
        w, w_s, x_s, x_s_ub = self.layer_param_names
        return (
            getattr(layer, w),
            getattr(layer, w_s),
            getattr(layer, x_s, None),
            getattr(layer, x_s_ub, None),
        )

    def apply_weights(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        fp8_dtype = self.fp8_dtype
        maybe_out_dtype = self.config.out_dtype
        w, w_s, x_s, x_s_ub = self._get_layer_params(layer)

        #   ops.scaled_fp8_quant supports both dynamic and static quant.
        #   If dynamic, layer.input_scale is None and x_s computed from x.
        #   If static, layer.input_scale is scalar and x_s is input_scale.
        # View input as 2D matrix for fp8 methods
        x_2d = x.view(-1, x.shape[-1])
        output_shape = [*x.shape[:-1], w.shape[1]]
        out_dtype = x.dtype if maybe_out_dtype is None else maybe_out_dtype

        # If input not quantized
        # TODO(luka) remove this path if not used anymore
        x_2d_q = x_2d
        if x.dtype != fp8_dtype:
            x_2d_q, x_s = self.quant_fp8(
                x_2d,
                x_s,
                x_s_ub,
            )
        return self.apply_scaled_mm(
            A=x_2d_q,
            B=w,
            out_dtype=out_dtype,
            As=x_s,
            Bs=w_s,
            bias=bias,
            output_shape=output_shape,
        )

    @abstractmethod
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
        raise NotImplementedError

    def get_output_padding(self) -> int | None:
        return None
```
**EN:** This abstract base class defines `FP8ScaledMMLinearKernel`. It organizes the state and behavior needed by this kernel path. It inherits from `ScaledMMLinearKernel[FP8ScaledMMLinearLayerConfig, _FP8ParamsT]`, `ABC`. Key methods include `__init__`, `process_weights_after_loading`, `_get_layer_params`, `apply_weights`, `apply_scaled_mm`, ....
**CN:** 该抽象基类定义了 `FP8ScaledMMLinearKernel`。 它组织了该内核路径所需的状态与行为。 它继承自 `ScaledMMLinearKernel[FP8ScaledMMLinearLayerConfig, _FP8ParamsT]`, `ABC`。 关键方法包括 `__init__`, `process_weights_after_loading`, `_get_layer_params`, `apply_weights`, `apply_scaled_mm`, ...。

### Method `FP8ScaledMMLinearKernel.__init__` (lines 96-106)
```python
    def __init__(
        self, c: FP8ScaledMMLinearLayerConfig, layer_param_names: Sequence[str]
    ) -> None:
        act_scale_descriptor = c.activation_quant_key.scale
        self.quant_fp8 = QuantFP8(
            static=act_scale_descriptor.static,
            group_shape=act_scale_descriptor.group_shape,
            num_token_padding=self.get_output_padding(),
        )
        self.fp8_dtype = current_platform.fp8_dtype()
        super().__init__(c, layer_param_names)
```
**EN:** This method implements `FP8ScaledMMLinearKernel.__init__`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `QuantFP8`, `fp8_dtype`, `__init__`, `get_output_padding`.
**CN:** 该方法 `FP8ScaledMMLinearKernel.__init__` 封装了此模块中的一段关键运行时逻辑，重点处理 init 相关工作。 它内部会调用 `QuantFP8`, `fp8_dtype`, `__init__`, `get_output_padding` 等例程。

### Method `FP8ScaledMMLinearKernel.process_weights_after_loading` (lines 108-109)
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        pass
```
**EN:** This method implements `FP8ScaledMMLinearKernel.process_weights_after_loading`, encapsulating a focused unit of runtime logic for the module.
**CN:** 该方法 `FP8ScaledMMLinearKernel.process_weights_after_loading` 封装了此模块中的一段关键运行时逻辑，重点处理 process weights after loading 相关工作。

### Method `FP8ScaledMMLinearKernel._get_layer_params` (lines 111-118)
```python
    def _get_layer_params(self, layer) -> _FP8ParamsT:
        w, w_s, x_s, x_s_ub = self.layer_param_names
        return (
            getattr(layer, w),
            getattr(layer, w_s),
            getattr(layer, x_s, None),
            getattr(layer, x_s_ub, None),
        )
```
**EN:** This method implements `FP8ScaledMMLinearKernel._get_layer_params`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `getattr`.
**CN:** 该方法 `FP8ScaledMMLinearKernel._get_layer_params` 封装了此模块中的一段关键运行时逻辑，重点处理 get layer params 相关工作。 它内部会调用 `getattr` 等例程。

### Method `FP8ScaledMMLinearKernel.apply_weights` (lines 120-155)
```python
    def apply_weights(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        fp8_dtype = self.fp8_dtype
        maybe_out_dtype = self.config.out_dtype
        w, w_s, x_s, x_s_ub = self._get_layer_params(layer)

        #   ops.scaled_fp8_quant supports both dynamic and static quant.
        #   If dynamic, layer.input_scale is None and x_s computed from x.
        #   If static, layer.input_scale is scalar and x_s is input_scale.
        # View input as 2D matrix for fp8 methods
        x_2d = x.view(-1, x.shape[-1])
        output_shape = [*x.shape[:-1], w.shape[1]]
        out_dtype = x.dtype if maybe_out_dtype is None else maybe_out_dtype

        # If input not quantized
        # TODO(luka) remove this path if not used anymore
        x_2d_q = x_2d
        if x.dtype != fp8_dtype:
            x_2d_q, x_s = self.quant_fp8(
                x_2d,
                x_s,
                x_s_ub,
            )
        return self.apply_scaled_mm(
            A=x_2d_q,
            B=w,
            out_dtype=out_dtype,
            As=x_s,
            Bs=w_s,
            bias=bias,
            output_shape=output_shape,
        )
```
**EN:** This method implements `FP8ScaledMMLinearKernel.apply_weights`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `_get_layer_params`, `view`, `apply_scaled_mm`, `quant_fp8`.
**CN:** 该方法 `FP8ScaledMMLinearKernel.apply_weights` 封装了此模块中的一段关键运行时逻辑，重点处理 apply weights 相关工作。 它内部会调用 `_get_layer_params`, `view`, `apply_scaled_mm`, `quant_fp8` 等例程。

### Method `FP8ScaledMMLinearKernel.apply_scaled_mm` (lines 158-169)
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
        raise NotImplementedError
```
**EN:** This method implements `FP8ScaledMMLinearKernel.apply_scaled_mm`, encapsulating a focused unit of runtime logic for the module.
**CN:** 该方法 `FP8ScaledMMLinearKernel.apply_scaled_mm` 封装了此模块中的一段关键运行时逻辑，重点处理 apply scaled mm 相关工作。

### Method `FP8ScaledMMLinearKernel.get_output_padding` (lines 171-172)
```python
    def get_output_padding(self) -> int | None:
        return None
```
**EN:** This method implements `FP8ScaledMMLinearKernel.get_output_padding`, encapsulating a focused unit of runtime logic for the module.
**CN:** 该方法 `FP8ScaledMMLinearKernel.get_output_padding` 封装了此模块中的一段关键运行时逻辑，重点处理 get output padding 相关工作。

### Class `Int8ScaledMMLinearKernel` (lines 175-186)
```python
class Int8ScaledMMLinearKernel(
    ScaledMMLinearKernel[Int8ScaledMMLinearLayerConfig, _Int8ParamsT], ABC
):
    def _get_layer_params(self, layer) -> _Int8ParamsT:
        w_q, w_s, i_s, i_zp, azp_adj = self.layer_param_names
        return (
            getattr(layer, w_q),
            getattr(layer, w_s),
            getattr(layer, i_s, None),
            getattr(layer, i_zp, None),
            getattr(layer, azp_adj, None),
        )
```
**EN:** This abstract base class defines `Int8ScaledMMLinearKernel`. It organizes the state and behavior needed by this kernel path. It inherits from `ScaledMMLinearKernel[Int8ScaledMMLinearLayerConfig, _Int8ParamsT]`, `ABC`. Key methods include `_get_layer_params`.
**CN:** 该抽象基类定义了 `Int8ScaledMMLinearKernel`。 它组织了该内核路径所需的状态与行为。 它继承自 `ScaledMMLinearKernel[Int8ScaledMMLinearLayerConfig, _Int8ParamsT]`, `ABC`。 关键方法包括 `_get_layer_params`。

### Method `Int8ScaledMMLinearKernel._get_layer_params` (lines 178-186)
```python
    def _get_layer_params(self, layer) -> _Int8ParamsT:
        w_q, w_s, i_s, i_zp, azp_adj = self.layer_param_names
        return (
            getattr(layer, w_q),
            getattr(layer, w_s),
            getattr(layer, i_s, None),
            getattr(layer, i_zp, None),
            getattr(layer, azp_adj, None),
        )
```
**EN:** This method implements `Int8ScaledMMLinearKernel._get_layer_params`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `getattr`.
**CN:** 该方法 `Int8ScaledMMLinearKernel._get_layer_params` 封装了此模块中的一段关键运行时逻辑，重点处理 get layer params 相关工作。 它内部会调用 `getattr` 等例程。

## Key Concepts / 关键概念
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
- **vLLM internal / vLLM 内部依赖**: `vllm.model_executor.layers.quantization.input_quant_fp8`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.platforms`, `..base`
- **External / 外部依赖**: `abc`, `collections.abc`, `dataclasses`, `typing`, `torch`
