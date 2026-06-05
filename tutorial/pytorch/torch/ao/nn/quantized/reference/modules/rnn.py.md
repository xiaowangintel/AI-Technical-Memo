# rnn.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/ao/nn/quantized/reference/modules/rnn.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN) / 用途（英文）**: Implements quantization-aware neural-network module wrappers, fused building blocks, and reference/quantized module variants. This specific file centers on `rnn.py`. The file is especially concerned with quantization metadata, calibration, or fake-quant behavior.
- **Purpose (CN) / 用途（中文）**: 实现面向量化的神经网络模块包装、融合构件以及参考/量化模块变体。 该文件具体围绕 `rnn.py` 展开。 该文件尤其关注量化元数据、校准流程或 fake-quant 行为。
## Line-by-Line Analysis / 逐行分析

### Lines 1-25 / 第 1-25 行
```python
# mypy: allow-untyped-defs
from typing import Any

import torch
import torch.nn as nn
from torch import _VF, Tensor
from torch.nn.utils.rnn import PackedSequence

from .utils import _quantize_and_dequantize_weight, _quantize_weight


__all__ = [
    "RNNCellBase",
    "RNNCell",
    "LSTMCell",
    "GRUCell",
    "RNNBase",
    "LSTM",
    "GRU",
    "get_quantized_weight",
]


def _apply_permutation(tensor: Tensor, permutation: Tensor, dim: int = 1) -> Tensor:
    return tensor.index_select(dim, permutation)
```
- **EN**: The `__all__` assignment defines the intended public surface that other modules are expected to import or re-export. Key callable entry points in this range include `_apply_permutation`. They package a focused unit of behavior behind named helpers or APIs. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: `__all__` 赋值定义了预期的公开接口，供其他模块导入或再次导出。 这一段的重要可调用入口包括 `_apply_permutation`，它们把聚焦的行为封装成具名辅助函数或 API。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 28-55 / 第 28-55 行
```python
def _get_weight_and_quantization_params(module, wn):
    weight = getattr(module, wn)
    params = [weight]
    for param_name in [
        wn + n for n in ["_qscheme", "_dtype", "_scale", "_zero_point", "_axis_int"]
    ]:
        if hasattr(module, param_name):
            param = getattr(module, param_name)
        else:
            param = None
        params.append(param)
    return params


def get_quantized_weight(module, wn):
    if not hasattr(module, wn):
        return None
    params = _get_weight_and_quantization_params(module, wn)
    weight = _quantize_weight(*params)
    return weight


def _get_quantize_and_dequantized_weight(module, wn):
    if not hasattr(module, wn):
        return None
    params = _get_weight_and_quantization_params(module, wn)
    weight = _quantize_and_dequantize_weight(*params)
    return weight
```
- **EN**: Key callable entry points in this range include `_get_weight_and_quantization_params`, `get_quantized_weight`, `_get_quantize_and_dequantized_weight`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `_get_weight_and_quantization_params`, `get_quantized_weight`, `_get_quantize_and_dequantized_weight`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 58-89 / 第 58-89 行
```python
class RNNCellBase(nn.RNNCellBase):
    def __init__(
        self,
        input_size: int,
        hidden_size: int,
        bias: bool,
        num_chunks: int,
        device=None,
        dtype=None,
        weight_qparams_dict=None,
    ) -> None:
        super().__init__(
            input_size, hidden_size, bias, num_chunks, device=device, dtype=dtype
        )
        # TODO(jerryzh168): maybe make this arg a required arg
        if weight_qparams_dict is None:
            weight_qparams = {
                "qscheme": torch.per_tensor_affine,
                "dtype": torch.quint8,
                "scale": 1.0,
                "zero_point": 0,
            }
            weight_qparams_dict = {
                "weight_ih": weight_qparams,
                "weight_hh": weight_qparams,
                "is_decomposed": False,
            }
        if len(weight_qparams_dict) != 3:
            raise AssertionError(
                f"Expected length for weight_qparams_dict to be 3 for QuantizedRNNCellBase(Reference), "
                f"got {len(weight_qparams_dict)}"
            )
```
- **EN**: It introduces or extends class-level abstractions such as `RNNCellBase`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `RNNCellBase` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 90-121 / 第 90-121 行
```python
        self._init_weight_qparams_dict(weight_qparams_dict, device)

    def _init_weight_qparams_dict(self, weight_qparams_dict, device):
        if weight_qparams_dict is None:
            raise AssertionError("weight_qparams_dict must not be None")
        self.is_decomposed = weight_qparams_dict["is_decomposed"]
        for key, weight_qparams in weight_qparams_dict.items():
            if key == "is_decomposed":
                continue
            # TODO: refactor the duplicated code to utils.py
            weight_qscheme = weight_qparams["qscheme"]
            weight_dtype = weight_qparams["dtype"]
            setattr(self, key + "_qscheme", weight_qscheme)
            setattr(self, key + "_dtype", weight_dtype)
            if weight_qscheme not in [
                None,
                torch.per_tensor_affine,
                torch.per_channel_affine,
            ]:
                raise AssertionError(
                    f"qscheme: {weight_qscheme} is not supported in {self._get_name()}"
                )
            if weight_qscheme is not None:
                scale = weight_qparams["scale"]
                scale_tensor = (
                    scale.detach().clone()
                    if isinstance(scale, torch.Tensor)
                    else torch.tensor(scale, dtype=torch.float, device=device)
                )
                self.register_buffer(key + "_scale", scale_tensor)
                zp = weight_qparams["zero_point"]
                zp_tensor = (
```
- **EN**: It introduces or extends class-level abstractions such as `RNNCellBase`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `RNNCellBase` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 122-152 / 第 122-152 行
```python
                    zp.detach().clone()
                    if isinstance(zp, torch.Tensor)
                    else torch.tensor(zp, dtype=torch.int, device=device)
                )
                self.register_buffer(key + "_zero_point", zp_tensor)
                if weight_qscheme == torch.per_channel_affine:
                    axis = weight_qparams["axis"]
                    axis_tensor = (
                        axis.detach().clone()
                        if isinstance(axis, torch.Tensor)
                        else torch.tensor(axis, dtype=torch.int, device=device)
                    )
                    self.register_buffer(key + "_axis", axis_tensor)
                else:
                    # added for TorchScriptability, not used
                    self.register_buffer(
                        key + "_axis", torch.tensor(0, dtype=torch.int, device=device)
                    )
                setattr(self, key + "_axis_int", getattr(self, key + "_axis").item())

    def _get_name(self):
        return "QuantizedRNNCellBase(Reference)"

    def get_quantized_weight_ih(self):
        return get_quantized_weight(self, "weight_ih")

    def get_quantized_weight_hh(self):
        return get_quantized_weight(self, "weight_hh")

    def get_weight_ih(self):
        return _get_quantize_and_dequantized_weight(self, "weight_ih")
```
- **EN**: It introduces or extends class-level abstractions such as `RNNCellBase`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `RNNCellBase` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 154-184 / 第 154-184 行
```python
    def get_weight_hh(self):
        return _get_quantize_and_dequantized_weight(self, "weight_hh")


class RNNCell(RNNCellBase):
    """
    We'll store weight_qparams for all the weights (weight_ih and weight_hh),
    we need to pass in a `weight_qparams_dict` that maps from weight name,
    e.g. weight_ih, to the weight_qparams for that weight
    """

    def __init__(
        self,
        input_size: int,
        hidden_size: int,
        bias: bool = True,
        nonlinearity: str = "tanh",
        device=None,
        dtype=None,
        weight_qparams_dict: dict[str, Any] | None = None,
    ) -> None:
        factory_kwargs = {
            "device": device,
            "dtype": dtype,
            "weight_qparams_dict": weight_qparams_dict,
        }
        super().__init__(input_size, hidden_size, bias, num_chunks=1, **factory_kwargs)
        self.nonlinearity = nonlinearity

    def _get_name(self):
        return "QuantizedRNNCell(Reference)"
```
- **EN**: It introduces or extends class-level abstractions such as `RNNCellBase`, `RNNCell`, which organize state and behavior for this subsystem. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `RNNCellBase`, `RNNCell` 等类级抽象，用于组织该子系统的状态与行为。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 186-217 / 第 186-217 行
```python
    # TODO: refactor nn.RNNCell to have a _forward that takes weight_ih and weight_hh as input
    # and remove duplicated code, same for the other two Cell modules
    def forward(self, input: Tensor, hx: Tensor | None = None) -> Tensor:
        if input.dim() not in (1, 2):
            raise AssertionError(
                f"RNNCell: Expected input to be 1-D or 2-D but received {input.dim()}-D tensor"
            )
        is_batched = input.dim() == 2
        if not is_batched:
            input = input.unsqueeze(0)

        if hx is None:
            hx = torch.zeros(
                input.size(0), self.hidden_size, dtype=input.dtype, device=input.device
            )
        else:
            hx = hx.unsqueeze(0) if not is_batched else hx

        if self.nonlinearity == "tanh":
            ret = _VF.rnn_tanh_cell(
                input,
                hx,
                self.get_weight_ih(),
                self.get_weight_hh(),
                self.bias_ih,
                self.bias_hh,
            )
        elif self.nonlinearity == "relu":
            ret = _VF.rnn_relu_cell(
                input,
                hx,
                self.get_weight_ih(),
```
- **EN**: It introduces or extends class-level abstractions such as `RNNCell`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `RNNCell` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 218-246 / 第 218-246 行
```python
                self.get_weight_hh(),
                self.bias_ih,
                self.bias_hh,
            )
        else:
            ret = input  # TODO: remove when jit supports exception flow
            raise RuntimeError(f"Unknown nonlinearity: {self.nonlinearity}")

        if not is_batched:
            ret = ret.squeeze(0)

        return ret

    @classmethod
    def from_float(cls, mod, weight_qparams_dict):
        ref_mod = cls(
            mod.input_size,
            mod.hidden_size,
            mod.bias,
            mod.nonlinearity,
            mod.weight_ih.device,
            mod.weight_ih.dtype,
            weight_qparams_dict,
        )
        ref_mod.weight_ih = mod.weight_ih
        ref_mod.weight_hh = mod.weight_hh
        ref_mod.bias_ih = mod.bias_ih
        ref_mod.bias_hh = mod.bias_hh
        return ref_mod
```
- **EN**: It introduces or extends class-level abstractions such as `RNNCell`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `RNNCell` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 249-273 / 第 249-273 行
```python
class LSTMCell(RNNCellBase):
    """
    We'll store weight_qparams for all the weights (weight_ih and weight_hh),
    we need to pass in a `weight_qparams_dict` that maps from weight name,
    e.g. weight_ih, to the weight_qparams for that weight
    """

    def __init__(
        self,
        input_size: int,
        hidden_size: int,
        bias: bool = True,
        device=None,
        dtype=None,
        weight_qparams_dict: dict[str, Any] | None = None,
    ) -> None:
        factory_kwargs = {
            "device": device,
            "dtype": dtype,
            "weight_qparams_dict": weight_qparams_dict,
        }
        super().__init__(input_size, hidden_size, bias, num_chunks=4, **factory_kwargs)

    def _get_name(self):
        return "QuantizedLSTMCell(Reference)"
```
- **EN**: It introduces or extends class-level abstractions such as `LSTMCell`, which organize state and behavior for this subsystem. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `LSTMCell` 等类级抽象，用于组织该子系统的状态与行为。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 275-305 / 第 275-305 行
```python
    def forward(
        self, input: Tensor, hx: tuple[Tensor, Tensor] | None = None
    ) -> tuple[Tensor, Tensor]:
        if input.dim() not in (1, 2):
            raise AssertionError(
                f"LSTMCell: Expected input to be 1-D or 2-D but received {input.dim()}-D tensor"
            )
        is_batched = input.dim() == 2
        if not is_batched:
            input = input.unsqueeze(0)

        if hx is None:
            zeros = torch.zeros(
                input.size(0), self.hidden_size, dtype=input.dtype, device=input.device
            )
            hx = (zeros, zeros)
        else:
            hx = (hx[0].unsqueeze(0), hx[1].unsqueeze(0)) if not is_batched else hx

        ret = _VF.lstm_cell(
            input,
            hx,
            self.get_weight_ih(),
            self.get_weight_hh(),
            self.bias_ih,
            self.bias_hh,
        )

        if not is_batched:
            ret = (ret[0].squeeze(0), ret[1].squeeze(0))
        return ret
```
- **EN**: It introduces or extends class-level abstractions such as `LSTMCell`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `LSTMCell` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 307-329 / 第 307-329 行
```python
    @classmethod
    def from_float(cls, mod, weight_qparams_dict, use_precomputed_fake_quant=False):
        ref_mod = cls(
            mod.input_size,
            mod.hidden_size,
            mod.bias,
            mod.weight_ih.device,
            mod.weight_ih.dtype,
            weight_qparams_dict,
        )
        ref_mod.weight_ih = mod.weight_ih
        ref_mod.weight_hh = mod.weight_hh
        ref_mod.bias_ih = mod.bias_ih
        ref_mod.bias_hh = mod.bias_hh
        return ref_mod


class GRUCell(RNNCellBase):
    """
    We'll store weight_qparams for all the weights (weight_ih and weight_hh),
    we need to pass in a `weight_qparams_dict` that maps from weight name,
    e.g. weight_ih, to the weight_qparams for that weight
    """
```
- **EN**: It introduces or extends class-level abstractions such as `LSTMCell`, `GRUCell`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `LSTMCell`, `GRUCell` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 331-357 / 第 331-357 行
```python
    def __init__(
        self,
        input_size: int,
        hidden_size: int,
        bias: bool = True,
        device=None,
        dtype=None,
        weight_qparams_dict: dict[str, Any] | None = None,
    ) -> None:
        factory_kwargs = {
            "device": device,
            "dtype": dtype,
            "weight_qparams_dict": weight_qparams_dict,
        }
        super().__init__(input_size, hidden_size, bias, num_chunks=3, **factory_kwargs)

    def _get_name(self):
        return "QuantizedGRUCell(Reference)"

    def forward(self, input: Tensor, hx: Tensor | None = None) -> Tensor:
        if input.dim() not in (1, 2):
            raise AssertionError(
                f"GRUCell: Expected input to be 1-D or 2-D but received {input.dim()}-D tensor"
            )
        is_batched = input.dim() == 2
        if not is_batched:
            input = input.unsqueeze(0)
```
- **EN**: It introduces or extends class-level abstractions such as `GRUCell`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `GRUCell` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 359-390 / 第 359-390 行
```python
        if hx is None:
            hx = torch.zeros(
                input.size(0), self.hidden_size, dtype=input.dtype, device=input.device
            )
        else:
            hx = hx.unsqueeze(0) if not is_batched else hx

        ret = _VF.gru_cell(
            input,
            hx,
            self.get_weight_ih(),
            self.get_weight_hh(),
            self.bias_ih,
            self.bias_hh,
        )

        if not is_batched:
            ret = ret.squeeze(0)

        return ret

    @classmethod
    def from_float(cls, mod, weight_qparams_dict):
        ref_mod = cls(
            mod.input_size,
            mod.hidden_size,
            mod.bias,
            mod.weight_ih.device,
            mod.weight_ih.dtype,
            weight_qparams_dict,
        )
        ref_mod.weight_ih = mod.weight_ih
```
- **EN**: It introduces or extends class-level abstractions such as `GRUCell`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `GRUCell` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 391-422 / 第 391-422 行
```python
        ref_mod.weight_hh = mod.weight_hh
        ref_mod.bias_ih = mod.bias_ih
        ref_mod.bias_hh = mod.bias_hh
        return ref_mod


class RNNBase(nn.RNNBase):
    def __init__(
        self,
        mode: str,
        input_size: int,
        hidden_size: int,
        num_layers: int = 1,
        bias: bool = True,
        batch_first: bool = False,
        dropout: float = 0.0,
        bidirectional: bool = False,
        proj_size: int = 0,
        device=None,
        dtype=None,
        weight_qparams_dict: dict[str, Any] | None = None,
    ) -> None:
        super().__init__(
            mode,
            input_size,
            hidden_size,
            num_layers,
            bias,
            batch_first,
            dropout,
            bidirectional,
            proj_size,
```
- **EN**: It introduces or extends class-level abstractions such as `GRUCell`, `RNNBase`, which organize state and behavior for this subsystem. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `GRUCell`, `RNNBase` 等类级抽象，用于组织该子系统的状态与行为。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 423-454 / 第 423-454 行
```python
            device,
            dtype,
        )
        # TODO(jerryzh168): maybe make this arg a required arg
        if weight_qparams_dict is None:
            weight_qparams = {
                "qscheme": torch.per_tensor_affine,
                "dtype": torch.quint8,
                "scale": 1.0,
                "zero_point": 0,
            }
            weight_qparams_dict = {"is_decomposed": False}  # type: ignore[dict-item]
            for wn in self._flat_weights_names:
                if wn.startswith("weight"):
                    weight_qparams_dict[wn] = weight_qparams
        self._init_weight_qparams_dict(weight_qparams_dict, device)

    def _init_weight_qparams_dict(self, weight_qparams_dict, device):
        self.is_decomposed = weight_qparams_dict["is_decomposed"]
        for key, weight_qparams in weight_qparams_dict.items():
            if key == "is_decomposed":
                continue
            weight_qscheme = weight_qparams["qscheme"]
            weight_dtype = weight_qparams["dtype"]
            setattr(self, key + "_qscheme", weight_qscheme)
            setattr(self, key + "_dtype", weight_dtype)
            if weight_qscheme not in [
                None,
                torch.per_tensor_affine,
                torch.per_channel_affine,
            ]:
                raise AssertionError(
```
- **EN**: It introduces or extends class-level abstractions such as `RNNBase`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `RNNBase` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 455-482 / 第 455-482 行
```python
                    f"qscheme: {weight_qscheme} is not supported in {self._get_name()}"
                )
            if weight_qscheme is not None:
                self.register_buffer(
                    key + "_scale",
                    torch.tensor(
                        weight_qparams["scale"], dtype=torch.float, device=device
                    ),
                )
                self.register_buffer(
                    key + "_zero_point",
                    torch.tensor(
                        weight_qparams["zero_point"], dtype=torch.int, device=device
                    ),
                )
                if weight_qscheme == torch.per_channel_affine:
                    self.register_buffer(
                        key + "_axis",
                        torch.tensor(
                            weight_qparams["axis"], dtype=torch.int, device=device
                        ),
                    )
                else:
                    # added for TorchScriptability, not used
                    self.register_buffer(
                        key + "_axis", torch.tensor(0, dtype=torch.int, device=device)
                    )
                setattr(self, key + "_axis_int", getattr(self, key + "_axis").item())
```
- **EN**: It introduces or extends class-level abstractions such as `RNNBase`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `RNNBase` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 485-505 / 第 485-505 行
```python
class LSTM(RNNBase):
    """Reference Quantized LSTM Module
    We'll store weight_qparams for all the weights in _flat_weights, we need to pass in
    a `weight_qparams_dict` that maps from weight name, e.g. weight_ih_l0,
    to the weight_qparams for that weight
    """

    def __init__(self, *args, **kwargs):
        super().__init__("LSTM", *args, **kwargs)

    # Same as above, see torch/nn/modules/module.py::_forward_unimplemented
    def permute_hidden(  # type: ignore[override]
        self,
        hx: tuple[Tensor, Tensor],
        permutation: Tensor | None,
    ) -> tuple[Tensor, Tensor]:
        if permutation is None:
            return hx
        return _apply_permutation(hx[0], permutation), _apply_permutation(
            hx[1], permutation
        )
```
- **EN**: It introduces or extends class-level abstractions such as `LSTM`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `LSTM` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 507-538 / 第 507-538 行
```python
    def get_expected_cell_size(
        self, input: Tensor, batch_sizes: Tensor | None
    ) -> tuple[int, int, int]:
        if batch_sizes is not None:
            mini_batch = int(batch_sizes[0])
        else:
            mini_batch = input.size(0) if self.batch_first else input.size(1)
        num_directions = 2 if self.bidirectional else 1
        expected_hidden_size = (
            self.num_layers * num_directions,
            mini_batch,
            self.hidden_size,
        )
        return expected_hidden_size

    # In the future, we should prevent mypy from applying contravariance rules here.
    # See torch/nn/modules/module.py::_forward_unimplemented
    def check_forward_args(  # type: ignore[override]
        self,
        input: Tensor,
        hidden: tuple[Tensor, Tensor],
        batch_sizes: Tensor | None,
    ):
        self.check_input(input, batch_sizes)
        self.check_hidden_size(
            hidden[0],
            self.get_expected_hidden_size(input, batch_sizes),
            "Expected hidden[0] size {}, got {}",
        )
        self.check_hidden_size(
            hidden[1],
            self.get_expected_cell_size(input, batch_sizes),
```
- **EN**: It introduces or extends class-level abstractions such as `LSTM`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `LSTM` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 539-561 / 第 539-561 行
```python
            "Expected hidden[1] size {}, got {}",
        )

    def get_quantized_weight_bias_dict(self):
        """dictionary from flat_weight_name to quantized weight or (unquantized) bias
        e.g.
        {
          "weight_ih_l0": quantized_weight,
          "bias_ih_l0": unquantized_bias,
          ...
        }
        """
        quantized_weight_bias_dict = {}
        for wn in self._flat_weights_names:
            if hasattr(self, wn):
                if wn.startswith("weight"):
                    weight_or_bias = get_quantized_weight(self, wn)
                else:
                    weight_or_bias = getattr(self, wn)
            else:
                weight_or_bias = None
            quantized_weight_bias_dict[wn] = weight_or_bias
        return quantized_weight_bias_dict
```
- **EN**: It introduces or extends class-level abstractions such as `LSTM`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `LSTM` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 563-591 / 第 563-591 行
```python
    def get_flat_weights(self):
        flat_weights = []
        for wn in self._flat_weights_names:
            if hasattr(self, wn):
                weight = getattr(self, wn)
                if wn.startswith("weight"):
                    params = _get_weight_and_quantization_params(self, wn)
                    weight = _quantize_and_dequantize_weight(*params)
            else:
                weight = None
            flat_weights.append(weight)
        return flat_weights

    def forward(self, input, hx=None):
        orig_input = input
        # xxx: isinstance check needs to be in conditional for TorchScript to compile
        batch_sizes = None
        if isinstance(orig_input, PackedSequence):
            input, batch_sizes, sorted_indices, unsorted_indices = input
            max_batch_size = int(batch_sizes[0])
        else:
            batch_sizes = None
            is_batched = input.dim() == 3
            batch_dim = 0 if self.batch_first else 1
            if not is_batched:
                input = input.unsqueeze(batch_dim)
            max_batch_size = input.size(0) if self.batch_first else input.size(1)
            sorted_indices = None
            unsorted_indices = None
```
- **EN**: It introduces or extends class-level abstractions such as `LSTM`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `LSTM` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 593-624 / 第 593-624 行
```python
        if hx is None:
            num_directions = 2 if self.bidirectional else 1
            real_hidden_size = (
                self.proj_size if self.proj_size > 0 else self.hidden_size
            )
            h_zeros = torch.zeros(
                self.num_layers * num_directions,
                max_batch_size,
                real_hidden_size,
                dtype=input.dtype,
                device=input.device,
            )
            c_zeros = torch.zeros(
                self.num_layers * num_directions,
                max_batch_size,
                self.hidden_size,
                dtype=input.dtype,
                device=input.device,
            )
            hx = (h_zeros, c_zeros)
        else:
            if batch_sizes is None:  # If not PackedSequence input.
                if is_batched:  # type: ignore[possibly-undefined]
                    if hx[0].dim() != 3 or hx[1].dim() != 3:
                        msg = (
                            "For batched 3-D input, hx and cx should "
                            f"also be 3-D but got ({hx[0].dim()}-D, {hx[1].dim()}-D) tensors"
                        )
                        raise RuntimeError(msg)
                else:
                    if hx[0].dim() != 2 or hx[1].dim() != 2:
                        msg = (
```
- **EN**: It introduces or extends class-level abstractions such as `LSTM`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `LSTM` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 625-656 / 第 625-656 行
```python
                            "For unbatched 2-D input, hx and cx should "
                            f"also be 2-D but got ({hx[0].dim()}-D, {hx[1].dim()}-D) tensors"
                        )
                        raise RuntimeError(msg)
                    hx = (hx[0].unsqueeze(1), hx[1].unsqueeze(1))

            # Each batch of the hidden state should match the input sequence that
            # the user believes he/she is passing in.
            hx = self.permute_hidden(hx, sorted_indices)

        self.check_forward_args(input, hx, batch_sizes)
        if batch_sizes is None:
            result = _VF.lstm(
                input,
                hx,
                self.get_flat_weights(),
                self.bias,
                self.num_layers,
                self.dropout,
                self.training,
                self.bidirectional,
                self.batch_first,
            )
        else:
            result = _VF.lstm(
                input,
                batch_sizes,
                hx,
                self.get_flat_weights(),
                self.bias,
                self.num_layers,
                self.dropout,
```
- **EN**: It introduces or extends class-level abstractions such as `LSTM`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `LSTM` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 657-678 / 第 657-678 行
```python
                self.training,
                self.bidirectional,
            )
        output = result[0]
        hidden = result[1:]
        # xxx: isinstance check needs to be in conditional for TorchScript to compile
        if isinstance(orig_input, PackedSequence):
            output_packed = PackedSequence(
                output,
                batch_sizes,
                sorted_indices,
                unsorted_indices,
            )
            return output_packed, self.permute_hidden(hidden, unsorted_indices)
        else:
            if not is_batched:  # type: ignore[possibly-undefined]
                output = output.squeeze(batch_dim)  # type: ignore[possibly-undefined]
                hidden = (hidden[0].squeeze(1), hidden[1].squeeze(1))
            return output, self.permute_hidden(hidden, unsorted_indices)

    def _get_name(self):
        return "QuantizedLSTM(Reference)"
```
- **EN**: It introduces or extends class-level abstractions such as `LSTM`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `LSTM` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 680-709 / 第 680-709 行
```python
    @classmethod
    def from_float(cls, mod, weight_qparams_dict):
        ref_mod = cls(
            mod.input_size,
            mod.hidden_size,
            mod.num_layers,
            mod.bias,
            mod.batch_first,
            mod.dropout,
            mod.bidirectional,
            weight_qparams_dict=weight_qparams_dict,
        )
        for wn in mod._flat_weights_names:
            setattr(ref_mod, wn, getattr(mod, wn))
        return ref_mod


class GRU(RNNBase):
    """Reference Quantized GRU Module
    We'll store weight_qparams for all the weights in _flat_weights, we need to pass in
    a `weight_qparams_dict` that maps from weight name, e.g. weight_ih_l0,
    to the weight_qparams for that weight
    """

    def __init__(self, *args, **kwargs):
        if "proj_size" in kwargs:
            raise ValueError(
                "proj_size argument is only supported for LSTM, not RNN or GRU"
            )
        super().__init__("GRU", *args, **kwargs)
```
- **EN**: It introduces or extends class-level abstractions such as `LSTM`, `GRU`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `LSTM`, `GRU` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 711-742 / 第 711-742 行
```python
    def get_quantized_weight_bias_dict(self):
        """dictionary from flat_weight_name to quantized weight or (unquantized) bias
        e.g.
        {
          "weight_ih_l0": quantized_weight,
          "bias_ih_l0": unquantized_bias,
          ...
        }
        """
        quantized_weight_bias_dict = {}
        for wn in self._flat_weights_names:
            if hasattr(self, wn):
                if wn.startswith("weight"):
                    weight_or_bias = get_quantized_weight(self, wn)
                else:
                    weight_or_bias = getattr(self, wn)
            else:
                weight_or_bias = None
            quantized_weight_bias_dict[wn] = weight_or_bias
        return quantized_weight_bias_dict

    def get_flat_weights(self):
        flat_weights = []
        for wn in self._flat_weights_names:
            if hasattr(self, wn):
                weight = getattr(self, wn)
                if wn.startswith("weight"):
                    params = _get_weight_and_quantization_params(self, wn)
                    weight = _quantize_and_dequantize_weight(*params)
            else:
                weight = None
            flat_weights.append(weight)
```
- **EN**: It introduces or extends class-level abstractions such as `GRU`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `GRU` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 743-774 / 第 743-774 行
```python
        return flat_weights

    def forward(self, input, hx=None):
        # Note: this is copied from the forward of GRU in https://github.com/pytorch/pytorch/blob/master/torch/nn/modules/rnn.py
        # only changed self._flat_weights to self.get_flat_weights()
        # TODO: maybe we can try inheriting from that class and define get_flat_weights
        # as a @property? this might interfere with TorchScript, if we remove that
        # requirement in the future we should be able to do this
        orig_input = input
        # xxx: isinstance check needs to be in conditional for TorchScript to compile
        if isinstance(orig_input, PackedSequence):
            input, batch_sizes, sorted_indices, unsorted_indices = input
            max_batch_size = int(batch_sizes[0])
        else:
            batch_sizes = None
            if input.dim() not in (2, 3):
                raise AssertionError(
                    f"GRU: Expected input to be 2-D or 3-D but received {input.dim()}-D tensor"
                )
            is_batched = input.dim() == 3
            batch_dim = 0 if self.batch_first else 1
            if not is_batched:
                input = input.unsqueeze(batch_dim)
                if hx is not None:
                    if hx.dim() != 2:
                        raise RuntimeError(
                            f"For unbatched 2-D input, hx should also be 2-D but got {hx.dim()}-D tensor"
                        )
                    hx = hx.unsqueeze(1)
            else:
                if hx is not None and hx.dim() != 3:
                    raise RuntimeError(
```
- **EN**: It introduces or extends class-level abstractions such as `GRU`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `GRU` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 775-806 / 第 775-806 行
```python
                        f"For batched 3-D input, hx should also be 3-D but got {hx.dim()}-D tensor"
                    )
            max_batch_size = input.size(0) if self.batch_first else input.size(1)
            sorted_indices = None
            unsorted_indices = None

        if hx is None:
            num_directions = 2 if self.bidirectional else 1
            hx = torch.zeros(
                self.num_layers * num_directions,
                max_batch_size,
                self.hidden_size,
                dtype=input.dtype,
                device=input.device,
            )
        else:
            # Each batch of the hidden state should match the input sequence that
            # the user believes he/she is passing in.
            hx = self.permute_hidden(hx, sorted_indices)

        self.check_forward_args(input, hx, batch_sizes)
        if batch_sizes is None:
            result = _VF.gru(
                input,
                hx,
                self.get_flat_weights(),
                self.bias,
                self.num_layers,
                self.dropout,
                self.training,
                self.bidirectional,
                self.batch_first,
```
- **EN**: It introduces or extends class-level abstractions such as `GRU`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `GRU` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 807-837 / 第 807-837 行
```python
            )
        else:
            result = _VF.gru(
                input,
                batch_sizes,
                hx,
                self.get_flat_weights(),
                self.bias,
                self.num_layers,
                self.dropout,
                self.training,
                self.bidirectional,
            )
        output = result[0]
        hidden = result[1]

        # xxx: isinstance check needs to be in conditional for TorchScript to compile
        if isinstance(orig_input, PackedSequence):
            output_packed = PackedSequence(
                output,
                batch_sizes,
                sorted_indices,
                unsorted_indices,
            )
            return output_packed, self.permute_hidden(hidden, unsorted_indices)
        else:
            if not is_batched:  # type: ignore[possibly-undefined]
                output = output.squeeze(batch_dim)  # type: ignore[possibly-undefined]
                hidden = hidden.squeeze(1)

            return output, self.permute_hidden(hidden, unsorted_indices)
```
- **EN**: It introduces or extends class-level abstractions such as `GRU`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `GRU` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 839-856 / 第 839-856 行
```python
    def _get_name(self):
        return "QuantizedGRU(Reference)"

    @classmethod
    def from_float(cls, mod, weight_qparams_dict):
        ref_mod = cls(
            mod.input_size,
            mod.hidden_size,
            mod.num_layers,
            mod.bias,
            mod.batch_first,
            mod.dropout,
            mod.bidirectional,
            weight_qparams_dict=weight_qparams_dict,
        )
        for wn in mod._flat_weights_names:
            setattr(ref_mod, wn, getattr(mod, wn))
        return ref_mod
```
- **EN**: It introduces or extends class-level abstractions such as `GRU`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `GRU` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

## Key Concepts / 关键概念

- **Module role**
  - EN: Implements quantization-aware neural-network module wrappers, fused building blocks, and reference/quantized module variants.
  - CN: 实现面向量化的神经网络模块包装、融合构件以及参考/量化模块变体。
- **Quantization pipeline**
  - EN: Tracks calibration statistics, quantization parameters, or graph/module rewrites for lower-precision execution.
  - CN: 跟踪校准统计信息、量化参数，或面向低精度执行的图/模块改写。
- **Data loading**
  - EN: Coordinates dataset traversal, batching, multiprocessing workers, and sample collation.
  - CN: 协调数据集遍历、批处理、多进程 worker 以及样本拼装。
- **RNNCellBase**
  - EN: `RNNCellBase` is one of the main classes that structures the file's behavior.
  - CN: `RNNCellBase` 是组织该文件行为的核心类之一。
- **RNNCell**
  - EN: `RNNCell` is one of the main classes that structures the file's behavior.
  - CN: `RNNCell` 是组织该文件行为的核心类之一。
- **_apply_permutation**
  - EN: `_apply_permutation` is a representative function that exposes or coordinates an important action in this module.
  - CN: `_apply_permutation` 是一个具有代表性的函数，用于暴露或协调此模块中的关键动作。
## Dependencies / 依赖关系

- **Internal PyTorch modules / 内部 PyTorch 模块**: `torch`, `torch.nn`, `torch:_VF`, `torch:Tensor`, `torch.nn.utils.rnn:PackedSequence`, `.utils:_quantize_and_dequantize_weight`, `.utils:_quantize_weight`
- **Python standard library / Python 标准库**: `typing:Any`
- **Explicit exports / 显式导出**: `RNNCellBase`, `RNNCell`, `LSTMCell`, `GRUCell`, `RNNBase`, `LSTM`, `GRU`, `get_quantized_weight`
- **Primary symbols / 核心符号**: `RNNCellBase`, `RNNCell`, `LSTMCell`, `GRUCell`, `RNNBase`, `LSTM`, `GRU`, `_apply_permutation`, `_get_weight_and_quantization_params`, `get_quantized_weight`, `_get_quantize_and_dequantized_weight`
