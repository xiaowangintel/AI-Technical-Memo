# rnn.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/ao/nn/quantizable/modules/rnn.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN) / 用途（英文）**: Implements quantization-aware neural-network module wrappers, fused building blocks, and reference/quantized module variants. This specific file centers on `rnn.py`. Key abstractions such as `LSTMCell, _LSTMSingleLayer` shape the public surface of the file.
- **Purpose (CN) / 用途（中文）**: 实现面向量化的神经网络模块包装、融合构件以及参考/量化模块变体。 该文件具体围绕 `rnn.py` 展开。 `LSTMCell, _LSTMSingleLayer` 等关键抽象塑造了该文件的公开接口。
## Line-by-Line Analysis / 逐行分析

### Lines 1-28 / 第 1-28 行
```python
"""
We will recreate all the RNN modules as we require the modules to be decomposed
into its building blocks to be able to observe.
"""

# mypy: allow-untyped-defs

import numbers
import warnings

import torch
from torch import Tensor


__all__ = ["LSTMCell", "LSTM"]


class LSTMCell(torch.nn.Module):
    r"""A quantizable long short-term memory (LSTM) cell.

    For the description and the argument types, please, refer to :class:`~torch.nn.LSTMCell`

    `split_gates`: specify True to compute the input/forget/cell/output gates separately
    to avoid an intermediate tensor which is subsequently chunk'd. This optimization can
    be beneficial for on-device inference latency. This flag is cascaded down from the
    parent classes.

    Examples::
```
- **EN**: The opening docstring frames the module contract, telling readers what subsystem responsibilities the rest of the file implements. The `__all__` assignment defines the intended public surface that other modules are expected to import or re-export. It introduces or extends class-level abstractions such as `LSTMCell`, which organize state and behavior for this subsystem. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 开头的模块文档字符串给出了整体契约，说明后续代码要实现的子系统职责。 `__all__` 赋值定义了预期的公开接口，供其他模块导入或再次导出。 它引入或扩展了 `LSTMCell` 等类级抽象，用于组织该子系统的状态与行为。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 30-59 / 第 30-59 行
```python
        >>> import torch.ao.nn.quantizable as nnqa
        >>> rnn = nnqa.LSTMCell(10, 20)
        >>> input = torch.randn(6, 10)
        >>> hx = torch.randn(3, 20)
        >>> cx = torch.randn(3, 20)
        >>> output = []
        >>> for i in range(6):
        ...     hx, cx = rnn(input[i], (hx, cx))
        ...     output.append(hx)
    """

    _FLOAT_MODULE = torch.nn.LSTMCell
    __constants__ = ["split_gates"]  # for jit.script

    def __init__(
        self,
        input_dim: int,
        hidden_dim: int,
        bias: bool = True,
        device=None,
        dtype=None,
        *,
        split_gates=False,
    ) -> None:
        factory_kwargs = {"device": device, "dtype": dtype}
        super().__init__()
        self.input_size = input_dim
        self.hidden_size = hidden_dim
        self.bias = bias
        self.split_gates = split_gates
```
- **EN**: It introduces or extends class-level abstractions such as `LSTMCell`, which organize state and behavior for this subsystem. Named constants such as `_FLOAT_MODULE` centralize shared configuration or sentinel values. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `LSTMCell` 等类级抽象，用于组织该子系统的状态与行为。 `_FLOAT_MODULE` 等具名常量把共享配置或哨兵值集中定义在一起。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 61-89 / 第 61-89 行
```python
        if not split_gates:
            self.igates: torch.nn.Module = torch.nn.Linear(
                input_dim, 4 * hidden_dim, bias=bias, **factory_kwargs
            )
            self.hgates: torch.nn.Module = torch.nn.Linear(
                hidden_dim, 4 * hidden_dim, bias=bias, **factory_kwargs
            )
            self.gates: torch.nn.Module = torch.ao.nn.quantized.FloatFunctional()
        else:
            # keep separate Linear layers for each gate
            self.igates = torch.nn.ModuleDict()
            self.hgates = torch.nn.ModuleDict()
            self.gates = torch.nn.ModuleDict()
            for g in ["input", "forget", "cell", "output"]:
                # pyre-fixme[29]: `Union[torch._tensor.Tensor, torch.nn.modules.module.Module]`
                self.igates[g] = torch.nn.Linear(
                    input_dim, hidden_dim, bias=bias, **factory_kwargs
                )
                # pyre-fixme[29]: `Union[torch._tensor.Tensor, torch.nn.modules.module.Module]`
                self.hgates[g] = torch.nn.Linear(
                    hidden_dim, hidden_dim, bias=bias, **factory_kwargs
                )
                # pyre-fixme[29]: `Union[torch._tensor.Tensor, torch.nn.modules.module.Module]`
                self.gates[g] = torch.ao.nn.quantized.FloatFunctional()

        self.input_gate = torch.nn.Sigmoid()
        self.forget_gate = torch.nn.Sigmoid()
        self.cell_gate = torch.nn.Tanh()
        self.output_gate = torch.nn.Sigmoid()
```
- **EN**: It introduces or extends class-level abstractions such as `LSTMCell`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `LSTMCell` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 91-114 / 第 91-114 行
```python
        self.fgate_cx = torch.ao.nn.quantized.FloatFunctional()
        self.igate_cgate = torch.ao.nn.quantized.FloatFunctional()
        self.fgate_cx_igate_cgate = torch.ao.nn.quantized.FloatFunctional()

        self.ogate_cy = torch.ao.nn.quantized.FloatFunctional()

        self.initial_hidden_state_qparams: tuple[float, int] = (1.0, 0)
        self.initial_cell_state_qparams: tuple[float, int] = (1.0, 0)
        self.hidden_state_dtype: torch.dtype = torch.quint8
        self.cell_state_dtype: torch.dtype = torch.quint8

    def forward(
        self, x: Tensor, hidden: tuple[Tensor, Tensor] | None = None
    ) -> tuple[Tensor, Tensor]:
        if hidden is None or hidden[0] is None or hidden[1] is None:
            hidden = self.initialize_hidden(x.shape[0], x.is_quantized)
        hx, cx = hidden

        if not self.split_gates:
            igates = self.igates(x)
            hgates = self.hgates(hx)
            gates = self.gates.add(igates, hgates)  # type: ignore[operator]

            input_gate, forget_gate, cell_gate, out_gate = gates.chunk(4, 1)
```
- **EN**: It introduces or extends class-level abstractions such as `LSTMCell`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `LSTMCell` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 116-143 / 第 116-143 行
```python
            input_gate = self.input_gate(input_gate)
            forget_gate = self.forget_gate(forget_gate)
            cell_gate = self.cell_gate(cell_gate)
            out_gate = self.output_gate(out_gate)
        else:
            # apply each input + hidden projection and add together
            gate = {}
            for (key, gates), igates, hgates in zip(
                self.gates.items(),  # type: ignore[operator]
                self.igates.values(),  # type: ignore[operator]
                self.hgates.values(),  # type: ignore[operator]
            ):
                gate[key] = gates.add(igates(x), hgates(hx))

            input_gate = self.input_gate(gate["input"])
            forget_gate = self.forget_gate(gate["forget"])
            cell_gate = self.cell_gate(gate["cell"])
            out_gate = self.output_gate(gate["output"])

        fgate_cx = self.fgate_cx.mul(forget_gate, cx)
        igate_cgate = self.igate_cgate.mul(input_gate, cell_gate)
        fgate_cx_igate_cgate = self.fgate_cx_igate_cgate.add(fgate_cx, igate_cgate)
        cy = fgate_cx_igate_cgate

        # TODO: make this tanh a member of the module so its qparams can be configured
        tanh_cy = torch.tanh(cy)
        hy = self.ogate_cy.mul(out_gate, tanh_cy)
        return hy, cy
```
- **EN**: It introduces or extends class-level abstractions such as `LSTMCell`, which organize state and behavior for this subsystem. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `LSTMCell` 等类级抽象，用于组织该子系统的状态与行为。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 145-168 / 第 145-168 行
```python
    def initialize_hidden(
        self, batch_size: int, is_quantized: bool = False
    ) -> tuple[Tensor, Tensor]:
        h, c = (
            torch.zeros((batch_size, self.hidden_size)),
            torch.zeros((batch_size, self.hidden_size)),
        )
        if is_quantized:
            (h_scale, h_zp) = self.initial_hidden_state_qparams
            (c_scale, c_zp) = self.initial_cell_state_qparams
            h = torch.quantize_per_tensor(
                h, scale=h_scale, zero_point=h_zp, dtype=self.hidden_state_dtype
            )
            c = torch.quantize_per_tensor(
                c, scale=c_scale, zero_point=c_zp, dtype=self.cell_state_dtype
            )
        return h, c

    def _get_name(self):
        return "QuantizableLSTMCell"

    @classmethod
    def from_params(cls, wi, wh, bi=None, bh=None, split_gates=False):
        """Uses the weights and biases to create a new LSTM cell.
```
- **EN**: It introduces or extends class-level abstractions such as `LSTMCell`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `LSTMCell` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 170-200 / 第 170-200 行
```python
        Args:
            wi, wh: Weights for the input and hidden layers
            bi, bh: Biases for the input and hidden layers
        """
        if (bi is None) != (bh is None):
            raise AssertionError("bi and bh must both be None or both have values")
        input_size = wi.shape[1]
        hidden_size = wh.shape[1]
        cell = cls(
            input_dim=input_size,
            hidden_dim=hidden_size,
            bias=(bi is not None),
            split_gates=split_gates,
        )

        if not split_gates:
            cell.igates.weight = torch.nn.Parameter(wi)
            if bi is not None:
                cell.igates.bias = torch.nn.Parameter(bi)
            cell.hgates.weight = torch.nn.Parameter(wh)
            if bh is not None:
                cell.hgates.bias = torch.nn.Parameter(bh)
        else:
            # split weight/bias
            for w, b, gates in zip([wi, wh], [bi, bh], [cell.igates, cell.hgates]):
                for w_chunk, gate in zip(w.chunk(4, dim=0), gates.values()):  # type: ignore[operator]
                    gate.weight = torch.nn.Parameter(w_chunk)

                if b is not None:
                    for b_chunk, gate in zip(b.chunk(4, dim=0), gates.values()):  # type: ignore[operator]
                        gate.bias = torch.nn.Parameter(b_chunk)
```
- **EN**: It introduces or extends class-level abstractions such as `LSTMCell`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `LSTMCell` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 202-232 / 第 202-232 行
```python
        return cell

    @classmethod
    def from_float(cls, other, use_precomputed_fake_quant=False, split_gates=False):
        if type(other) is not cls._FLOAT_MODULE:
            raise AssertionError(
                f"Expected module type {cls._FLOAT_MODULE}, got {type(other)}"
            )
        if not hasattr(other, "qconfig"):
            raise AssertionError("The float module must have 'qconfig'")
        observed = cls.from_params(
            other.weight_ih,
            other.weight_hh,
            other.bias_ih,
            other.bias_hh,
            split_gates=split_gates,
        )
        observed.qconfig = other.qconfig
        observed.igates.qconfig = other.qconfig
        observed.hgates.qconfig = other.qconfig
        if split_gates:
            # also apply qconfig directly to Linear modules
            for g in observed.igates.values():
                g.qconfig = other.qconfig
            for g in observed.hgates.values():
                g.qconfig = other.qconfig
        return observed


class _LSTMSingleLayer(torch.nn.Module):
    r"""A single one-directional LSTM layer.
```
- **EN**: It introduces or extends class-level abstractions such as `LSTMCell`, `_LSTMSingleLayer`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `LSTMCell`, `_LSTMSingleLayer` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 234-261 / 第 234-261 行
```python
    The difference between a layer and a cell is that the layer can process a
    sequence, while the cell only expects an instantaneous value.
    """

    def __init__(
        self,
        input_dim: int,
        hidden_dim: int,
        bias: bool = True,
        device=None,
        dtype=None,
        *,
        split_gates=False,
    ) -> None:
        factory_kwargs = {"device": device, "dtype": dtype}
        super().__init__()
        self.cell = LSTMCell(
            input_dim, hidden_dim, bias=bias, split_gates=split_gates, **factory_kwargs
        )

    def forward(self, x: Tensor, hidden: tuple[Tensor, Tensor] | None = None):
        result = []
        seq_len = x.shape[0]
        for i in range(seq_len):
            hidden = self.cell(x[i], hidden)
            result.append(hidden[0])  # type: ignore[index]
        result_tensor = torch.stack(result, 0)
        return result_tensor, hidden
```
- **EN**: It introduces or extends class-level abstractions such as `_LSTMSingleLayer`, which organize state and behavior for this subsystem. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `_LSTMSingleLayer` 等类级抽象，用于组织该子系统的状态与行为。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 263-294 / 第 263-294 行
```python
    @classmethod
    def from_params(cls, *args, **kwargs):
        cell = LSTMCell.from_params(*args, **kwargs)
        layer = cls(
            cell.input_size, cell.hidden_size, cell.bias, split_gates=cell.split_gates
        )
        layer.cell = cell
        return layer


class _LSTMLayer(torch.nn.Module):
    r"""A single bi-directional LSTM layer."""

    def __init__(
        self,
        input_dim: int,
        hidden_dim: int,
        bias: bool = True,
        batch_first: bool = False,
        bidirectional: bool = False,
        device=None,
        dtype=None,
        *,
        split_gates=False,
    ) -> None:
        factory_kwargs = {"device": device, "dtype": dtype}
        super().__init__()
        self.batch_first = batch_first
        self.bidirectional = bidirectional
        self.layer_fw = _LSTMSingleLayer(
            input_dim, hidden_dim, bias=bias, split_gates=split_gates, **factory_kwargs
        )
```
- **EN**: It introduces or extends class-level abstractions such as `_LSTMSingleLayer`, `_LSTMLayer`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `_LSTMSingleLayer`, `_LSTMLayer` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 295-326 / 第 295-326 行
```python
        if self.bidirectional:
            self.layer_bw = _LSTMSingleLayer(
                input_dim,
                hidden_dim,
                bias=bias,
                split_gates=split_gates,
                **factory_kwargs,
            )

    def forward(self, x: Tensor, hidden: tuple[Tensor, Tensor] | None = None):
        if self.batch_first:
            x = x.transpose(0, 1)
        if hidden is None:
            hx_fw, cx_fw = (None, None)
        else:
            hx_fw, cx_fw = hidden
        hidden_bw: tuple[Tensor, Tensor] | None = None
        if self.bidirectional:
            if hx_fw is None:
                hx_bw = None
            else:
                hx_bw = hx_fw[1]
                hx_fw = hx_fw[0]
            if cx_fw is None:
                cx_bw = None
            else:
                cx_bw = cx_fw[1]
                cx_fw = cx_fw[0]
            if hx_bw is not None and cx_bw is not None:
                hidden_bw = hx_bw, cx_bw
        if hx_fw is None and cx_fw is None:
            hidden_fw = None
```
- **EN**: It introduces or extends class-level abstractions such as `_LSTMLayer`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `_LSTMLayer` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 327-357 / 第 327-357 行
```python
        else:
            hidden_fw = (
                torch.jit._unwrap_optional(hx_fw),
                torch.jit._unwrap_optional(cx_fw),
            )
        result_fw, hidden_fw = self.layer_fw(x, hidden_fw)

        if hasattr(self, "layer_bw") and self.bidirectional:
            x_reversed = x.flip(0)
            result_bw, hidden_bw = self.layer_bw(x_reversed, hidden_bw)
            result_bw = result_bw.flip(0)

            result = torch.cat([result_fw, result_bw], result_fw.dim() - 1)
            if hidden_fw is None and hidden_bw is None:
                h = None
                c = None
            elif hidden_fw is None:
                (h, c) = torch.jit._unwrap_optional(hidden_bw)
            elif hidden_bw is None:
                (h, c) = torch.jit._unwrap_optional(hidden_fw)
            else:
                h = torch.stack([hidden_fw[0], hidden_bw[0]], 0)  # type: ignore[list-item]
                c = torch.stack([hidden_fw[1], hidden_bw[1]], 0)  # type: ignore[list-item]
        else:
            result = result_fw
            h, c = torch.jit._unwrap_optional(hidden_fw)  # type: ignore[assignment]

        if self.batch_first:
            result.transpose_(0, 1)

        return result, (h, c)
```
- **EN**: It introduces or extends class-level abstractions such as `_LSTMLayer`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `_LSTMLayer` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 359-390 / 第 359-390 行
```python
    @classmethod
    def from_float(cls, other, layer_idx=0, qconfig=None, **kwargs):
        r"""
        There is no FP equivalent of this class. This function is here just to
        mimic the behavior of the `prepare` within the `torch.ao.quantization`
        flow.
        """
        if not hasattr(other, "qconfig") and qconfig is None:
            raise AssertionError("other must have qconfig or qconfig must be provided")

        input_size = kwargs.get("input_size", other.input_size)
        hidden_size = kwargs.get("hidden_size", other.hidden_size)
        bias = kwargs.get("bias", other.bias)
        batch_first = kwargs.get("batch_first", other.batch_first)
        bidirectional = kwargs.get("bidirectional", other.bidirectional)
        split_gates = kwargs.get("split_gates", False)

        layer = cls(
            # pyrefly: ignore [bad-argument-type]
            input_size,
            # pyrefly: ignore [bad-argument-type]
            hidden_size,
            # pyrefly: ignore [bad-argument-type]
            bias,
            # pyrefly: ignore [bad-argument-type]
            batch_first,
            # pyrefly: ignore [bad-argument-type]
            bidirectional,
            split_gates=split_gates,
        )
        # pyrefly: ignore [bad-argument-type]
        layer.qconfig = getattr(other, "qconfig", qconfig)
```
- **EN**: It introduces or extends class-level abstractions such as `_LSTMLayer`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `_LSTMLayer` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 391-421 / 第 391-421 行
```python
        wi = getattr(other, f"weight_ih_l{layer_idx}")
        wh = getattr(other, f"weight_hh_l{layer_idx}")
        bi = getattr(other, f"bias_ih_l{layer_idx}", None)
        bh = getattr(other, f"bias_hh_l{layer_idx}", None)

        layer.layer_fw = _LSTMSingleLayer.from_params(
            wi, wh, bi, bh, split_gates=split_gates
        )

        if other.bidirectional:
            wi = getattr(other, f"weight_ih_l{layer_idx}_reverse")
            wh = getattr(other, f"weight_hh_l{layer_idx}_reverse")
            bi = getattr(other, f"bias_ih_l{layer_idx}_reverse", None)
            bh = getattr(other, f"bias_hh_l{layer_idx}_reverse", None)
            layer.layer_bw = _LSTMSingleLayer.from_params(
                wi, wh, bi, bh, split_gates=split_gates
            )
        return layer


class LSTM(torch.nn.Module):
    r"""A quantizable long short-term memory (LSTM).

    For the description and the argument types, please, refer to :class:`~torch.nn.LSTM`

    Attributes:
        layers : instances of the `_LSTMLayer`

    .. note::
        To access the weights and biases, you need to access them per layer.
        See examples below.
```
- **EN**: It introduces or extends class-level abstractions such as `_LSTMLayer`, `LSTM`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `_LSTMLayer`, `LSTM` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 423-454 / 第 423-454 行
```python
    Examples::

        >>> import torch.ao.nn.quantizable as nnqa
        >>> rnn = nnqa.LSTM(10, 20, 2)
        >>> input = torch.randn(5, 3, 10)
        >>> h0 = torch.randn(2, 3, 20)
        >>> c0 = torch.randn(2, 3, 20)
        >>> output, (hn, cn) = rnn(input, (h0, c0))
        >>> # To get the weights:
        >>> # xdoctest: +SKIP
        >>> print(rnn.layers[0].weight_ih)
        tensor([[...]])
        >>> print(rnn.layers[0].weight_hh)
        AssertionError: There is no reverse path in the non-bidirectional layer
    """

    _FLOAT_MODULE = torch.nn.LSTM

    def __init__(
        self,
        input_size: int,
        hidden_size: int,
        num_layers: int = 1,
        bias: bool = True,
        batch_first: bool = False,
        dropout: float = 0.0,
        bidirectional: bool = False,
        device=None,
        dtype=None,
        *,
        split_gates: bool = False,
    ) -> None:
```
- **EN**: It introduces or extends class-level abstractions such as `LSTM`, which organize state and behavior for this subsystem. Named constants such as `_FLOAT_MODULE` centralize shared configuration or sentinel values. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `LSTM` 等类级抽象，用于组织该子系统的状态与行为。 `_FLOAT_MODULE` 等具名常量把共享配置或哨兵值集中定义在一起。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 455-475 / 第 455-475 行
```python
        factory_kwargs = {"device": device, "dtype": dtype}
        super().__init__()
        self.input_size = input_size
        self.hidden_size = hidden_size
        self.num_layers = num_layers
        self.bias = bias
        self.batch_first = batch_first
        self.dropout = float(dropout)
        self.bidirectional = bidirectional
        self.training = False  # Default to eval mode. If we want to train, we will explicitly set to training.

        if (
            not isinstance(dropout, numbers.Number)
            or not 0 <= dropout <= 1
            or isinstance(dropout, bool)
        ):
            raise ValueError(
                "dropout should be a number in range [0, 1] "
                "representing the probability of an element being "
                "zeroed"
            )
```
- **EN**: It introduces or extends class-level abstractions such as `LSTM`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `LSTM` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 477-508 / 第 477-508 行
```python
        if dropout > 0:
            warnings.warn(
                "dropout option for quantizable LSTM is ignored. "
                "If you are training, please, use nn.LSTM version "
                "followed by `prepare` step.",
                stacklevel=2,
            )
            if num_layers == 1:
                warnings.warn(
                    "dropout option adds dropout after all but last "
                    "recurrent layer, so non-zero dropout expects "
                    f"num_layers greater than 1, but got dropout={dropout} "
                    f"and num_layers={num_layers}",
                    stacklevel=2,
                )

        layers = [
            _LSTMLayer(
                self.input_size,
                self.hidden_size,
                self.bias,
                batch_first=False,
                bidirectional=self.bidirectional,
                split_gates=split_gates,
                **factory_kwargs,
            )
        ]
        layers.extend(
            _LSTMLayer(
                self.hidden_size,
                self.hidden_size,
                self.bias,
```
- **EN**: It introduces or extends class-level abstractions such as `LSTM`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `LSTM` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 509-540 / 第 509-540 行
```python
                batch_first=False,
                bidirectional=self.bidirectional,
                split_gates=split_gates,
                **factory_kwargs,
            )
            for _ in range(1, num_layers)
        )
        self.layers = torch.nn.ModuleList(layers)

    def forward(self, x: Tensor, hidden: tuple[Tensor, Tensor] | None = None):
        if self.batch_first:
            x = x.transpose(0, 1)

        max_batch_size = x.size(1)
        num_directions = 2 if self.bidirectional else 1
        if hidden is None:
            zeros = torch.zeros(
                num_directions,
                max_batch_size,
                self.hidden_size,
                dtype=torch.float,
                device=x.device,
            )
            zeros.squeeze_(0)
            if x.is_quantized:
                zeros = torch.quantize_per_tensor(
                    zeros, scale=1.0, zero_point=0, dtype=x.dtype
                )
            hxcx = [(zeros, zeros) for _ in range(self.num_layers)]
        else:
            hidden_non_opt = torch.jit._unwrap_optional(hidden)
            if isinstance(hidden_non_opt[0], Tensor):
```
- **EN**: It introduces or extends class-level abstractions such as `LSTM`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `LSTM` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 541-571 / 第 541-571 行
```python
                hx = hidden_non_opt[0].reshape(
                    self.num_layers, num_directions, max_batch_size, self.hidden_size
                )
                cx = hidden_non_opt[1].reshape(
                    self.num_layers, num_directions, max_batch_size, self.hidden_size
                )
                hxcx = [
                    (hx[idx].squeeze(0), cx[idx].squeeze(0))
                    for idx in range(self.num_layers)
                ]
            else:
                hxcx = hidden_non_opt

        hx_list = []
        cx_list = []
        for idx, layer in enumerate(self.layers):
            x, (h, c) = layer(x, hxcx[idx])
            hx_list.append(torch.jit._unwrap_optional(h))
            cx_list.append(torch.jit._unwrap_optional(c))
        hx_tensor = torch.stack(hx_list)
        cx_tensor = torch.stack(cx_list)

        # We are creating another dimension for bidirectional case
        # need to collapse it
        hx_tensor = hx_tensor.reshape(-1, hx_tensor.shape[-2], hx_tensor.shape[-1])
        cx_tensor = cx_tensor.reshape(-1, cx_tensor.shape[-2], cx_tensor.shape[-1])

        if self.batch_first:
            x = x.transpose(0, 1)

        return x, (hx_tensor, cx_tensor)
```
- **EN**: It introduces or extends class-level abstractions such as `LSTM`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `LSTM` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 573-599 / 第 573-599 行
```python
    def _get_name(self):
        return "QuantizableLSTM"

    @classmethod
    def from_float(cls, other, qconfig=None, split_gates=False):
        if not isinstance(other, cls._FLOAT_MODULE):
            raise AssertionError(
                f"Expected module type {cls._FLOAT_MODULE}, got {type(other)}"
            )
        if not hasattr(other, "qconfig") and not qconfig:
            raise AssertionError("other must have qconfig or qconfig must be provided")
        observed = cls(
            other.input_size,
            other.hidden_size,
            other.num_layers,
            other.bias,
            other.batch_first,
            other.dropout,
            other.bidirectional,
            split_gates=split_gates,
        )
        # pyrefly: ignore [bad-argument-type]
        observed.qconfig = getattr(other, "qconfig", qconfig)
        for idx in range(other.num_layers):
            observed.layers[idx] = _LSTMLayer.from_float(
                other, idx, qconfig, batch_first=False, split_gates=split_gates
            )
```
- **EN**: It introduces or extends class-level abstractions such as `LSTM`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `LSTM` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 601-618 / 第 601-618 行
```python
        # Prepare the model
        if other.training:
            observed.train()
            observed = torch.ao.quantization.prepare_qat(observed, inplace=True)
        else:
            observed.eval()
            observed = torch.ao.quantization.prepare(observed, inplace=True)
        return observed

    @classmethod
    def from_observed(cls, other):
        # The whole flow is float -> observed -> quantized
        # This class does float -> observed only
        raise NotImplementedError(
            "It looks like you are trying to convert a "
            "non-quantizable LSTM module. Please, see "
            "the examples on quantizable LSTMs."
        )
```
- **EN**: It introduces or extends class-level abstractions such as `LSTM`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `LSTM` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

## Key Concepts / 关键概念

- **Module role**
  - EN: Implements quantization-aware neural-network module wrappers, fused building blocks, and reference/quantized module variants.
  - CN: 实现面向量化的神经网络模块包装、融合构件以及参考/量化模块变体。
- **Quantization pipeline**
  - EN: Tracks calibration statistics, quantization parameters, or graph/module rewrites for lower-precision execution.
  - CN: 跟踪校准统计信息、量化参数，或面向低精度执行的图/模块改写。
- **Fake quantization**
  - EN: Simulates quantized numerics during training while keeping tensors in differentiable forms.
  - CN: 在训练阶段模拟量化数值行为，同时保持张量处于可求导形式。
- **Data loading**
  - EN: Coordinates dataset traversal, batching, multiprocessing workers, and sample collation.
  - CN: 协调数据集遍历、批处理、多进程 worker 以及样本拼装。
- **Configuration management**
  - EN: Centralizes feature flags, environment overrides, and runtime-config wiring.
  - CN: 集中管理特性开关、环境变量覆盖以及运行时配置接线。
- **LSTMCell**
  - EN: `LSTMCell` is one of the main classes that structures the file's behavior.
  - CN: `LSTMCell` 是组织该文件行为的核心类之一。
## Dependencies / 依赖关系

- **Internal PyTorch modules / 内部 PyTorch 模块**: `torch`, `torch:Tensor`
- **Python standard library / Python 标准库**: `numbers`, `warnings`
- **Explicit exports / 显式导出**: `LSTMCell`, `LSTM`
- **Primary symbols / 核心符号**: `LSTMCell`, `_LSTMSingleLayer`, `_LSTMLayer`, `LSTM`
