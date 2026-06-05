# lstm_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/ao/quantization/fx/lstm_utils.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN) / 用途（英文）**: Implements quantization observers, fake-quant modules, preparation/conversion passes, and backend-specific quantization helpers. This specific file centers on `lstm_utils.py`. The file is especially concerned with quantization metadata, calibration, or fake-quant behavior. It also participates in graph/module transformation before or after quantized execution. Runtime configuration, environment overrides, or feature gating are important in this module.
- **Purpose (CN) / 用途（中文）**: 实现量化 observer、fake-quant 模块、prepare/convert pass 以及后端相关的量化辅助逻辑。 该文件具体围绕 `lstm_utils.py` 展开。 该文件尤其关注量化元数据、校准流程或 fake-quant 行为。 它也参与量化执行前后的图/模块转换。 运行时配置、环境变量覆盖或特性开关是此模块的重要主题。
## Line-by-Line Analysis / 逐行分析

### Lines 1-19 / 第 1-19 行
```python
import copy
import operator
from typing import Any, TYPE_CHECKING

import torch
from torch.ao.quantization import (
    default_weight_fake_quant,
    default_weight_observer,
    FakeQuantizeBase,
    QConfig,
    QConfigMapping,
)
from torch.ao.quantization.backend_config import BackendConfig
from torch.ao.quantization.observer import _PartialWrapper
from torch.ao.quantization.quantize_fx import convert_to_reference_fx, prepare_fx


if TYPE_CHECKING:
    from collections.abc import Callable
```
- **EN**: This block establishes the module dependencies, pulling in internal torch modules such as torch, torch.ao.quantization:default_weight_fake_quant, torch.ao.quantization:default_weight_observer, torch.ao.quantization:FakeQuantizeBase; standard-library helpers such as copy, operator, typing:Any, typing:TYPE_CHECKING. Type-checking-only branches keep static analyzers informed without changing runtime behavior. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段建立模块依赖，引入了内部 torch 模块，如 torch, torch.ao.quantization:default_weight_fake_quant, torch.ao.quantization:default_weight_observer, torch.ao.quantization:FakeQuantizeBase；标准库辅助模块，如 copy, operator, typing:Any, typing:TYPE_CHECKING。 仅用于类型检查的分支在不改变运行时行为的前提下为静态分析器提供信息。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 22-42 / 第 22-42 行
```python
# TODO: move all LSTM util functions from fx/utils.py to this file
def _get_lstm_with_individually_observed_parts(
    float_lstm: torch.nn.LSTM,
    example_inputs: tuple[Any, ...],
    backend_config: BackendConfig | None = None,
    linear_output_obs_ctr: _PartialWrapper | None = None,
    sigmoid_obs_ctr: _PartialWrapper | None = None,
    tanh_obs_ctr: _PartialWrapper | None = None,
    cell_state_obs_ctr: _PartialWrapper | None = None,
    hidden_state_obs_ctr: _PartialWrapper | None = None,
    split_gates: bool = False,
) -> torch.ao.nn.quantizable.LSTM:
    """
    Return an observed `torch.ao.nn.quantizable.LSTM` created from a `torch.nn.LSTM`
    with specific observers or fake quantizes assigned to the inner ops or submodules.

    In both eager and FX graph mode quantization, `torch.ao.nn.quantizable.LSTM` is
    used as an observed custom module, which is responsible for inserting its own
    observers. By default, all inner ops inherit the parent custom module's QConfig.
    Users who wish to override this behavior may extend `torch.ao.nn.quantizable.LSTM`
    and use this helper function to customize the observer insertion logic.
```
- **EN**: Key callable entry points in this range include `_get_lstm_with_individually_observed_parts`. They package a focused unit of behavior behind named helpers or APIs. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Context-manager usage makes resource, mode, or temporary-state boundaries explicit. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `_get_lstm_with_individually_observed_parts`，它们把聚焦的行为封装成具名辅助函数或 API。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 上下文管理器的使用让资源、模式或临时状态的边界更加明确。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 44-63 / 第 44-63 行
```python
    This is meant to be used to convert a float module to an observed module in the
    custom module flow.

    Args:
        `float_lstm`: The float LSTM module
        `example_inputs`: example inputs for the forward function of the LSTM module
        `backend_config`: BackendConfig to use to observe the LSTM module
        `linear_output_obs_ctr`: observer or fake quantize for linear outputs Wx + b,
            where W is the weight matrix, b is the bias, and x is either the inputs
            or the hidden state from the previous layer (if any)
        `sigmoid_obs_ctr`: observer or fake quantize for sigmoid activations
        `tanh_obs_ctr`: observer or fake quantize for tanh activations
        `cell_state_obs_ctr`: observer or fake quantize for the cell state
        `hidden_state_obs_ctr`: observer or fake quantize for the hidden state and
            the output

    Return:
        A `torch.ao.nn.quantizable.LSTM` with the specified observers or fake quantizes
        assigned to the inner ops.
    """
```
- **EN**: Key callable entry points in this range include `_get_lstm_with_individually_observed_parts`. They package a focused unit of behavior behind named helpers or APIs. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `_get_lstm_with_individually_observed_parts`，它们把聚焦的行为封装成具名辅助函数或 API。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 65-85 / 第 65-85 行
```python
    def make_qconfig(obs_ctr: _PartialWrapper) -> QConfig:
        """
        Make a QConfig with fixed qparams observers or fake quantizes.
        """
        if isinstance(obs_ctr(), FakeQuantizeBase):
            weight = default_weight_fake_quant
        else:
            weight = default_weight_observer
        return QConfig(activation=obs_ctr, weight=weight)

    quantizable_lstm = torch.ao.nn.quantizable.LSTM(
        float_lstm.input_size,
        float_lstm.hidden_size,
        float_lstm.num_layers,
        float_lstm.bias,
        float_lstm.batch_first,
        float_lstm.dropout,
        float_lstm.bidirectional,
        split_gates=split_gates,
    )
    quantizable_lstm.qconfig = float_lstm.qconfig
```
- **EN**: Key callable entry points in this range include `_get_lstm_with_individually_observed_parts`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `_get_lstm_with_individually_observed_parts`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 87-106 / 第 87-106 行
```python
    for idx in range(float_lstm.num_layers):
        quantizable_lstm.layers[idx] = (
            torch.ao.nn.quantizable.modules.rnn._LSTMLayer.from_float(
                float_lstm,
                idx,
                float_lstm.qconfig,
                batch_first=False,
                split_gates=split_gates,
            )
        )

    # Build QConfigMapping for the LSTM cell
    # Note: FloatFunctional qconfigs will be configured separately below
    cell_qm = QConfigMapping().set_global(float_lstm.qconfig)  # type: ignore[arg-type]
    if sigmoid_obs_ctr is not None:
        cell_qm.set_module_name("input_gate", make_qconfig(sigmoid_obs_ctr))
        cell_qm.set_module_name("forget_gate", make_qconfig(sigmoid_obs_ctr))
        cell_qm.set_module_name("output_gate", make_qconfig(sigmoid_obs_ctr))
    if tanh_obs_ctr is not None:
        cell_qm.set_module_name("cell_gate", make_qconfig(tanh_obs_ctr))
```
- **EN**: Key callable entry points in this range include `_get_lstm_with_individually_observed_parts`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `_get_lstm_with_individually_observed_parts`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 108-131 / 第 108-131 行
```python
    # Insert observers into each LSTM cell
    # TODO: maybe make this work for layer_bw as well
    for layer in quantizable_lstm.layers:
        cell = layer.layer_fw.cell  # type: ignore[union-attr]
        if not isinstance(cell, torch.nn.Module):
            raise AssertionError("cell should be a nn.Module")
        cell = prepare_fx(cell, cell_qm, example_inputs, backend_config=backend_config)
        # HACK: Manually replace the activation_post_process following these ops.
        # This is needed for FloatFunctional ops because there is currently no way
        # to configure these ops in FX graph mode quantization today. This is because
        # the FloatFunctional modules simply disappear from the graph after tracing.
        # In the future, we should rewrite quantizable LSTM without FloatFunctionals.
        if not split_gates:
            op_index_to_activation_post_process_ctr = {
                (torch.add, 0): linear_output_obs_ctr,  # gates.add
                (torch.mul, 0): cell_state_obs_ctr,  # fgate_cx.mul
                (torch.mul, 1): cell_state_obs_ctr,  # igate_cgate.mul
                (torch.add, 1): cell_state_obs_ctr,  # fgate_cx_igate_cgate.add
                (torch.mul, 2): hidden_state_obs_ctr,  # ogate_cy.mul
            }
        else:
            op_index_to_activation_post_process_ctr = {
                (torch.add, 0): linear_output_obs_ctr,  # gates.add (input)
                (torch.add, 1): linear_output_obs_ctr,  # gates.add (forget)
```
- **EN**: Key callable entry points in this range include `_get_lstm_with_individually_observed_parts`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `_get_lstm_with_individually_observed_parts`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 132-155 / 第 132-155 行
```python
                (torch.add, 2): linear_output_obs_ctr,  # gates.add (cell)
                (torch.add, 3): linear_output_obs_ctr,  # gates.add (output)
                (torch.mul, 0): cell_state_obs_ctr,  # fgate_cx.mul
                (torch.mul, 1): cell_state_obs_ctr,  # igate_cgate.mul
                (torch.add, 4): cell_state_obs_ctr,  # fgate_cx_igate_cgate.add
                (torch.mul, 2): hidden_state_obs_ctr,  # ogate_cy.mul
            }
        add_count = 0
        mul_count = 0
        for node in cell.graph.nodes:
            op_index: tuple[Callable, int] | None = None  # e.g. (torch.add, 1)
            if node.target is torch.add:
                op_index = (torch.add, add_count)
                add_count += 1
            elif node.target is torch.mul:
                op_index = (torch.mul, mul_count)
                mul_count += 1
            else:
                # Neither torch.add nor torch.mul
                continue
            if op_index not in op_index_to_activation_post_process_ctr:
                continue
            if len(node.users) != 1:
                raise AssertionError("expected exactly one user for the node")
```
- **EN**: Key callable entry points in this range include `_get_lstm_with_individually_observed_parts`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `_get_lstm_with_individually_observed_parts`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 156-178 / 第 156-178 行
```python
            activation_post_process_name = next(iter(node.users.keys())).name
            activation_post_process_ctr = op_index_to_activation_post_process_ctr[
                op_index
            ]
            if activation_post_process_ctr is not None:
                setattr(
                    cell, activation_post_process_name, activation_post_process_ctr()
                )
        layer.layer_fw.cell = cell  # type: ignore[union-attr]
    return quantizable_lstm


def _get_reference_quantized_lstm_module(
    observed_lstm: torch.ao.nn.quantizable.LSTM,
    backend_config: BackendConfig | None = None,
) -> torch.ao.nn.quantized.LSTM:
    """
    Return a `torch.ao.nn.quantized.LSTM` created from a `torch.ao.nn.quantizable.LSTM`
    with observers or fake quantizes inserted through `prepare_fx`, e.g. from
    `_get_lstm_with_individually_observed_parts`.

    This is meant to be used to convert an observed module to a quantized module in the
    custom module flow.
```
- **EN**: Key callable entry points in this range include `_get_lstm_with_individually_observed_parts`, `_get_reference_quantized_lstm_module`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Context-manager usage makes resource, mode, or temporary-state boundaries explicit. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `_get_lstm_with_individually_observed_parts`, `_get_reference_quantized_lstm_module`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 上下文管理器的使用让资源、模式或临时状态的边界更加明确。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 180-195 / 第 180-195 行
```python
    Args:
        `observed_lstm`: a `torch.ao.nn.quantizable.LSTM` observed through `prepare_fx`
        `backend_config`: BackendConfig to use to produce the reference quantized model

    Return:
        A reference `torch.ao.nn.quantized.LSTM` module.
    """
    quantized_lstm = torch.ao.nn.quantized.LSTM(
        observed_lstm.input_size,
        observed_lstm.hidden_size,
        observed_lstm.num_layers,
        observed_lstm.bias,
        observed_lstm.batch_first,
        observed_lstm.dropout,
        observed_lstm.bidirectional,
    )
```
- **EN**: Key callable entry points in this range include `_get_reference_quantized_lstm_module`. They package a focused unit of behavior behind named helpers or APIs. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `_get_reference_quantized_lstm_module`，它们把聚焦的行为封装成具名辅助函数或 API。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 197-220 / 第 197-220 行
```python
    for i, layer in enumerate(quantized_lstm.layers):
        cell = copy.deepcopy(observed_lstm.layers.get_submodule(str(i)).layer_fw.cell)  # type: ignore[union-attr]
        cell = convert_to_reference_fx(cell, backend_config=backend_config)  # type: ignore[arg-type]
        if not isinstance(cell, torch.fx.GraphModule):
            raise AssertionError("cell must be converted to a torch.fx.GraphModule")
        # HACK: Manually remove input quantize nodes and output dequantize nodes,
        # since custom modules expect quint8 inputs and outputs for now. Note that
        # this functionality is supposedly handled through PrepareCustomConfig's
        # `set_input_quantized_indexes` and `set_output_quantized_indexes`, but that
        # API doesn't currently handle tuple inputs and outputs, so we have to do
        # this manually for now. In the future we should (1) relax the restriction
        # on custom module input/output dtypes, and (2) expand support for complex
        # input/output structures.
        for node in cell.graph.nodes:
            if node.target is torch.quantize_per_tensor:
                arg = node.args[0]
                # Remove quantize(x), quantize(hidden[0]), and quantize(hidden[1])
                if arg.target == "x" or (
                    arg.target is operator.getitem and arg.args[0].target == "hidden"
                ):
                    with cell.graph.inserting_before(node):
                        node.replace_all_uses_with(arg)
                        cell.graph.erase_node(node)
            if node.target == "output":
```
- **EN**: Key callable entry points in this range include `_get_reference_quantized_lstm_module`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Context-manager usage makes resource, mode, or temporary-state boundaries explicit. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `_get_reference_quantized_lstm_module`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 上下文管理器的使用让资源、模式或临时状态的边界更加明确。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 221-228 / 第 221-228 行
```python
                # Remove all dequantize nodes in the output tuple
                for arg in node.args[0]:
                    with cell.graph.inserting_before(node):
                        node.replace_input_with(arg, arg.args[0])
        cell.graph.eliminate_dead_code()
        cell.recompile()
        layer.layer_fw.cell = cell  # type: ignore[union-attr]
    return quantized_lstm
```
- **EN**: Key callable entry points in this range include `_get_reference_quantized_lstm_module`. They package a focused unit of behavior behind named helpers or APIs. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Context-manager usage makes resource, mode, or temporary-state boundaries explicit. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `_get_reference_quantized_lstm_module`，它们把聚焦的行为封装成具名辅助函数或 API。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 上下文管理器的使用让资源、模式或临时状态的边界更加明确。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

## Key Concepts / 关键概念

- **Module role**
  - EN: Implements quantization observers, fake-quant modules, preparation/conversion passes, and backend-specific quantization helpers.
  - CN: 实现量化 observer、fake-quant 模块、prepare/convert pass 以及后端相关的量化辅助逻辑。
- **Quantization pipeline**
  - EN: Tracks calibration statistics, quantization parameters, or graph/module rewrites for lower-precision execution.
  - CN: 跟踪校准统计信息、量化参数，或面向低精度执行的图/模块改写。
- **Observer statistics**
  - EN: Collects runtime ranges or histograms so later code can derive quantization settings.
  - CN: 收集运行时范围或直方图，以便后续逻辑推导量化设置。
- **Fake quantization**
  - EN: Simulates quantized numerics during training while keeping tensors in differentiable forms.
  - CN: 在训练阶段模拟量化数值行为，同时保持张量处于可求导形式。
- **Graph rewriting**
  - EN: Rewrites module or FX graph structure so later execution follows a transformed pipeline.
  - CN: 改写模块或 FX 图结构，使后续执行遵循转换后的流水线。
- **Data loading**
  - EN: Coordinates dataset traversal, batching, multiprocessing workers, and sample collation.
  - CN: 协调数据集遍历、批处理、多进程 worker 以及样本拼装。
## Dependencies / 依赖关系

- **Internal PyTorch modules / 内部 PyTorch 模块**: `torch`, `torch.ao.quantization:default_weight_fake_quant`, `torch.ao.quantization:default_weight_observer`, `torch.ao.quantization:FakeQuantizeBase`, `torch.ao.quantization:QConfig`, `torch.ao.quantization:QConfigMapping`, `torch.ao.quantization.backend_config:BackendConfig`, `torch.ao.quantization.observer:_PartialWrapper`, `torch.ao.quantization.quantize_fx:convert_to_reference_fx`, `torch.ao.quantization.quantize_fx:prepare_fx`
- **Python standard library / Python 标准库**: `copy`, `operator`, `typing:Any`, `typing:TYPE_CHECKING`
- **Primary symbols / 核心符号**: `_get_lstm_with_individually_observed_parts`, `_get_reference_quantized_lstm_module`
