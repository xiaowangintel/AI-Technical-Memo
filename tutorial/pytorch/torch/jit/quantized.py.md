# quantized.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/jit/quantized.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Implements Python-side TorchScript/JIT helpers for scripting, tracing, serialization, and runtime integration. The file mainly revolves around `QuantizedLinear`.
- **Purpose (CN)**: 实现 Python 侧的 TorchScript/JIT 辅助逻辑，用于脚本化、追踪、序列化与运行时集成。 该文件主要围绕 `QuantizedLinear` 展开。

## Line-by-Line Analysis / 逐行分析

### Lines 1-22 / 第 1-22 行

````python
0001: # mypy: allow-untyped-defs
0002: import torch
0003: 
0004: 
0005: class QuantizedLinear(torch.jit.ScriptModule):
0006:     def __init__(self, other):
0007:         raise RuntimeError(
0008:             "torch.jit.QuantizedLinear is no longer supported. Please use "
0009:             "torch.ao.nn.quantized.dynamic.Linear instead."
0010:         )
0011: 
0012: 
0013: # FP16 weights
0014: class QuantizedLinearFP16(torch.jit.ScriptModule):
0015:     def __init__(self, other):
0016:         super().__init__()
0017:         raise RuntimeError(
0018:             "torch.jit.QuantizedLinearFP16 is no longer supported. "
0019:             "Please use the torch.ao.nn.quantized.dynamic.Linear instead."
0020:         )
0021: 
0022: 
````

- **L1** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L2** EN: Imports module dependencies: `torch`. | CN: 导入模块依赖：`torch`。
- **L3** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L4** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L5** EN: Defines class `QuantizedLinear` with bases `torch.jit.ScriptModule`, which packages reusable module behavior behind an object-oriented interface. | CN: 定义类 `QuantizedLinear`，其基类为 `torch.jit.ScriptModule`，作用是通过面向对象接口封装可复用模块行为。
- **L6** EN: Defines function `__init__`, which implements Python-facing TorchScript/JIT behavior. | CN: 定义函数 `__init__`，其作用是实现面向 Python 的 TorchScript/JIT 行为。
- **L7** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L8** EN: Continues `QuantizedLinear.__init__`, which implements Python-facing TorchScript/JIT behavior. | CN: 继续 `QuantizedLinear.__init__` 的实现，其作用是实现面向 Python 的 TorchScript/JIT 行为。
- **L9** EN: Continues `QuantizedLinear.__init__`, which implements Python-facing TorchScript/JIT behavior. | CN: 继续 `QuantizedLinear.__init__` 的实现，其作用是实现面向 Python 的 TorchScript/JIT 行为。
- **L10** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L11** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L12** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L13** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L14** EN: Defines class `QuantizedLinearFP16` with bases `torch.jit.ScriptModule`, which packages reusable module behavior behind an object-oriented interface. | CN: 定义类 `QuantizedLinearFP16`，其基类为 `torch.jit.ScriptModule`，作用是通过面向对象接口封装可复用模块行为。
- **L15** EN: Defines function `__init__`, which implements Python-facing TorchScript/JIT behavior. | CN: 定义函数 `__init__`，其作用是实现面向 Python 的 TorchScript/JIT 行为。
- **L16** EN: Invokes `super` to advance the surrounding implementation. | CN: 调用 `super` 来推进周围的实现逻辑。
- **L17** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L18** EN: Continues `QuantizedLinearFP16.__init__`, which implements Python-facing TorchScript/JIT behavior. | CN: 继续 `QuantizedLinearFP16.__init__` 的实现，其作用是实现面向 Python 的 TorchScript/JIT 行为。
- **L19** EN: Continues `QuantizedLinearFP16.__init__`, which implements Python-facing TorchScript/JIT behavior. | CN: 继续 `QuantizedLinearFP16.__init__` 的实现，其作用是实现面向 Python 的 TorchScript/JIT 行为。
- **L20** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L21** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L22** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 23-39 / 第 23-39 行

````python
0023: # Quantized RNN cell implementations
0024: class QuantizedRNNCellBase(torch.jit.ScriptModule):
0025:     def __init__(self, other):
0026:         raise RuntimeError(
0027:             "torch.jit.QuantizedRNNCellBase is no longer supported. "
0028:             "Please use the torch.ao.nn.quantized.dynamic.RNNCell instead."
0029:         )
0030: 
0031: 
0032: class QuantizedRNNCell(QuantizedRNNCellBase):
0033:     def __init__(self, other):
0034:         raise RuntimeError(
0035:             "torch.jit.QuantizedRNNCell is no longer supported. "
0036:             "Please use the torch.ao.nn.quantized.dynamic.RNNCell instead."
0037:         )
0038: 
0039: 
````

- **L23** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L24** EN: Defines class `QuantizedRNNCellBase` with bases `torch.jit.ScriptModule`, which packages reusable module behavior behind an object-oriented interface. | CN: 定义类 `QuantizedRNNCellBase`，其基类为 `torch.jit.ScriptModule`，作用是通过面向对象接口封装可复用模块行为。
- **L25** EN: Defines function `__init__`, which implements Python-facing TorchScript/JIT behavior. | CN: 定义函数 `__init__`，其作用是实现面向 Python 的 TorchScript/JIT 行为。
- **L26** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L27** EN: Continues `QuantizedRNNCellBase.__init__`, which implements Python-facing TorchScript/JIT behavior. | CN: 继续 `QuantizedRNNCellBase.__init__` 的实现，其作用是实现面向 Python 的 TorchScript/JIT 行为。
- **L28** EN: Continues `QuantizedRNNCellBase.__init__`, which implements Python-facing TorchScript/JIT behavior. | CN: 继续 `QuantizedRNNCellBase.__init__` 的实现，其作用是实现面向 Python 的 TorchScript/JIT 行为。
- **L29** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L30** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L31** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L32** EN: Defines class `QuantizedRNNCell` with bases `QuantizedRNNCellBase`, which collects related state and methods into a named abstraction for this module. | CN: 定义类 `QuantizedRNNCell`，其基类为 `QuantizedRNNCellBase`，作用是把相关状态与方法组织成该模块中的命名抽象。
- **L33** EN: Defines function `__init__`, which implements Python-facing TorchScript/JIT behavior. | CN: 定义函数 `__init__`，其作用是实现面向 Python 的 TorchScript/JIT 行为。
- **L34** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L35** EN: Continues `QuantizedRNNCell.__init__`, which implements Python-facing TorchScript/JIT behavior. | CN: 继续 `QuantizedRNNCell.__init__` 的实现，其作用是实现面向 Python 的 TorchScript/JIT 行为。
- **L36** EN: Continues `QuantizedRNNCell.__init__`, which implements Python-facing TorchScript/JIT behavior. | CN: 继续 `QuantizedRNNCell.__init__` 的实现，其作用是实现面向 Python 的 TorchScript/JIT 行为。
- **L37** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L38** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L39** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 40-57 / 第 40-57 行

````python
0040: class QuantizedLSTMCell(QuantizedRNNCellBase):
0041:     def __init__(self, other):
0042:         super().__init__(other)
0043:         raise RuntimeError(
0044:             "torch.jit.QuantizedLSTMCell is no longer supported. "
0045:             "Please use the torch.ao.nn.quantized.dynamic.LSTMCell instead."
0046:         )
0047: 
0048: 
0049: class QuantizedGRUCell(QuantizedRNNCellBase):
0050:     def __init__(self, other):
0051:         super().__init__(other)
0052:         raise RuntimeError(
0053:             "torch.jit.QuantizedGRUCell is no longer supported. "
0054:             "Please use the torch.ao.nn.quantized.dynamic.GRUCell instead."
0055:         )
0056: 
0057: 
````

- **L40** EN: Defines class `QuantizedLSTMCell` with bases `QuantizedRNNCellBase`, which collects related state and methods into a named abstraction for this module. | CN: 定义类 `QuantizedLSTMCell`，其基类为 `QuantizedRNNCellBase`，作用是把相关状态与方法组织成该模块中的命名抽象。
- **L41** EN: Defines function `__init__`, which implements Python-facing TorchScript/JIT behavior. | CN: 定义函数 `__init__`，其作用是实现面向 Python 的 TorchScript/JIT 行为。
- **L42** EN: Invokes `super` to advance the surrounding implementation. | CN: 调用 `super` 来推进周围的实现逻辑。
- **L43** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L44** EN: Continues `QuantizedLSTMCell.__init__`, which implements Python-facing TorchScript/JIT behavior. | CN: 继续 `QuantizedLSTMCell.__init__` 的实现，其作用是实现面向 Python 的 TorchScript/JIT 行为。
- **L45** EN: Continues `QuantizedLSTMCell.__init__`, which implements Python-facing TorchScript/JIT behavior. | CN: 继续 `QuantizedLSTMCell.__init__` 的实现，其作用是实现面向 Python 的 TorchScript/JIT 行为。
- **L46** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L47** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L48** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L49** EN: Defines class `QuantizedGRUCell` with bases `QuantizedRNNCellBase`, which collects related state and methods into a named abstraction for this module. | CN: 定义类 `QuantizedGRUCell`，其基类为 `QuantizedRNNCellBase`，作用是把相关状态与方法组织成该模块中的命名抽象。
- **L50** EN: Defines function `__init__`, which implements Python-facing TorchScript/JIT behavior. | CN: 定义函数 `__init__`，其作用是实现面向 Python 的 TorchScript/JIT 行为。
- **L51** EN: Invokes `super` to advance the surrounding implementation. | CN: 调用 `super` 来推进周围的实现逻辑。
- **L52** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L53** EN: Continues `QuantizedGRUCell.__init__`, which implements Python-facing TorchScript/JIT behavior. | CN: 继续 `QuantizedGRUCell.__init__` 的实现，其作用是实现面向 Python 的 TorchScript/JIT 行为。
- **L54** EN: Continues `QuantizedGRUCell.__init__`, which implements Python-facing TorchScript/JIT behavior. | CN: 继续 `QuantizedGRUCell.__init__` 的实现，其作用是实现面向 Python 的 TorchScript/JIT 行为。
- **L55** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L56** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L57** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 58-73 / 第 58-73 行

````python
0058: class QuantizedRNNBase(torch.jit.ScriptModule):
0059:     def __init__(self, other, dtype=torch.int8):
0060:         raise RuntimeError(
0061:             "torch.jit.QuantizedRNNBase is no longer supported. "
0062:             "Please use the torch.ao.nn.quantized.dynamic instead."
0063:         )
0064: 
0065: 
0066: class QuantizedLSTM(QuantizedRNNBase):
0067:     def __init__(self, other, dtype):
0068:         raise RuntimeError(
0069:             "torch.jit.QuantizedLSTM is no longer supported. "
0070:             "Please use the torch.ao.nn.quantized.dynamic.LSTM instead."
0071:         )
0072: 
0073: 
````

- **L58** EN: Defines class `QuantizedRNNBase` with bases `torch.jit.ScriptModule`, which packages reusable module behavior behind an object-oriented interface. | CN: 定义类 `QuantizedRNNBase`，其基类为 `torch.jit.ScriptModule`，作用是通过面向对象接口封装可复用模块行为。
- **L59** EN: Defines function `__init__`, which implements Python-facing TorchScript/JIT behavior. | CN: 定义函数 `__init__`，其作用是实现面向 Python 的 TorchScript/JIT 行为。
- **L60** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L61** EN: Continues `QuantizedRNNBase.__init__`, which implements Python-facing TorchScript/JIT behavior. | CN: 继续 `QuantizedRNNBase.__init__` 的实现，其作用是实现面向 Python 的 TorchScript/JIT 行为。
- **L62** EN: Continues `QuantizedRNNBase.__init__`, which implements Python-facing TorchScript/JIT behavior. | CN: 继续 `QuantizedRNNBase.__init__` 的实现，其作用是实现面向 Python 的 TorchScript/JIT 行为。
- **L63** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L64** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L65** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L66** EN: Defines class `QuantizedLSTM` with bases `QuantizedRNNBase`, which collects related state and methods into a named abstraction for this module. | CN: 定义类 `QuantizedLSTM`，其基类为 `QuantizedRNNBase`，作用是把相关状态与方法组织成该模块中的命名抽象。
- **L67** EN: Defines function `__init__`, which implements Python-facing TorchScript/JIT behavior. | CN: 定义函数 `__init__`，其作用是实现面向 Python 的 TorchScript/JIT 行为。
- **L68** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L69** EN: Continues `QuantizedLSTM.__init__`, which implements Python-facing TorchScript/JIT behavior. | CN: 继续 `QuantizedLSTM.__init__` 的实现，其作用是实现面向 Python 的 TorchScript/JIT 行为。
- **L70** EN: Continues `QuantizedLSTM.__init__`, which implements Python-facing TorchScript/JIT behavior. | CN: 继续 `QuantizedLSTM.__init__` 的实现，其作用是实现面向 Python 的 TorchScript/JIT 行为。
- **L71** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L72** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L73** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 74-95 / 第 74-95 行

````python
0074: class QuantizedGRU(QuantizedRNNBase):
0075:     def __init__(self, *args, **kwargs):
0076:         raise RuntimeError(
0077:             "torch.jit.QuantizedGRU is no longer supported. "
0078:             "Please use the torch.ao.nn.quantized.dynamic.GRU instead."
0079:         )
0080: 
0081: 
0082: def quantize_rnn_cell_modules(module):
0083:     raise RuntimeError(
0084:         "quantize_rnn_cell_modules function is no longer supported. "
0085:         "Please use torch.ao.quantization.quantize_dynamic API instead."
0086:     )
0087: 
0088: 
0089: def quantize_linear_modules(module, dtype=torch.int8):
0090:     raise RuntimeError(
0091:         "quantize_linear_modules function is no longer supported. "
0092:         "Please use torch.ao.quantization.quantize_dynamic API instead."
0093:     )
0094: 
0095: 
````

- **L74** EN: Defines class `QuantizedGRU` with bases `QuantizedRNNBase`, which collects related state and methods into a named abstraction for this module. | CN: 定义类 `QuantizedGRU`，其基类为 `QuantizedRNNBase`，作用是把相关状态与方法组织成该模块中的命名抽象。
- **L75** EN: Defines function `__init__`, which implements Python-facing TorchScript/JIT behavior. | CN: 定义函数 `__init__`，其作用是实现面向 Python 的 TorchScript/JIT 行为。
- **L76** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L77** EN: Continues `QuantizedGRU.__init__`, which implements Python-facing TorchScript/JIT behavior. | CN: 继续 `QuantizedGRU.__init__` 的实现，其作用是实现面向 Python 的 TorchScript/JIT 行为。
- **L78** EN: Continues `QuantizedGRU.__init__`, which implements Python-facing TorchScript/JIT behavior. | CN: 继续 `QuantizedGRU.__init__` 的实现，其作用是实现面向 Python 的 TorchScript/JIT 行为。
- **L79** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L80** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L81** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L82** EN: Defines function `quantize_rnn_cell_modules`, which implements quantization-specific numerics or graph handling. | CN: 定义函数 `quantize_rnn_cell_modules`，其作用是实现量化特定数值逻辑或图处理。
- **L83** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L84** EN: Continues `quantize_rnn_cell_modules`, which implements quantization-specific numerics or graph handling. | CN: 继续 `quantize_rnn_cell_modules` 的实现，其作用是实现量化特定数值逻辑或图处理。
- **L85** EN: Continues `quantize_rnn_cell_modules`, which implements quantization-specific numerics or graph handling. | CN: 继续 `quantize_rnn_cell_modules` 的实现，其作用是实现量化特定数值逻辑或图处理。
- **L86** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L87** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L88** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L89** EN: Defines function `quantize_linear_modules`, which implements quantization-specific numerics or graph handling. | CN: 定义函数 `quantize_linear_modules`，其作用是实现量化特定数值逻辑或图处理。
- **L90** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L91** EN: Continues `quantize_linear_modules`, which implements quantization-specific numerics or graph handling. | CN: 继续 `quantize_linear_modules` 的实现，其作用是实现量化特定数值逻辑或图处理。
- **L92** EN: Continues `quantize_linear_modules`, which implements quantization-specific numerics or graph handling. | CN: 继续 `quantize_linear_modules` 的实现，其作用是实现量化特定数值逻辑或图处理。
- **L93** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L94** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L95** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 96-100 / 第 96-100 行

````python
0096: def quantize_rnn_modules(module, dtype=torch.int8):
0097:     raise RuntimeError(
0098:         "quantize_rnn_modules function is no longer supported. "
0099:         "Please use torch.ao.quantization.quantize_dynamic API instead."
0100:     )
````

- **L96** EN: Defines function `quantize_rnn_modules`, which implements quantization-specific numerics or graph handling. | CN: 定义函数 `quantize_rnn_modules`，其作用是实现量化特定数值逻辑或图处理。
- **L97** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L98** EN: Continues `quantize_rnn_modules`, which implements quantization-specific numerics or graph handling. | CN: 继续 `quantize_rnn_modules` 的实现，其作用是实现量化特定数值逻辑或图处理。
- **L99** EN: Continues `quantize_rnn_modules`, which implements quantization-specific numerics or graph handling. | CN: 继续 `quantize_rnn_modules` 的实现，其作用是实现量化特定数值逻辑或图处理。
- **L100** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。

## Key Concepts / 关键概念

- **EN**: TorchScript/JIT integration — The module coordinates scripting, tracing, compilation, or Python-facing JIT behavior.
  **CN**: TorchScript/JIT integration——该模块协调脚本化、追踪、编译或面向 Python 的 JIT 行为。
- **EN**: Frontend/runtime bridge — Python helpers often translate user intent into lower-level JIT runtime operations.
  **CN**: Frontend/runtime bridge——Python 辅助逻辑通常会把用户意图翻译为更底层的 JIT 运行时操作。
- **EN**: Compatibility helpers — Annotations, decomposition utilities, and wrappers smooth the user experience around JIT features.
  **CN**: Compatibility helpers——注解、分解工具与包装器帮助 JIT 特性在用户侧更易用。
- **EN**: Quantization — Low-precision conversion or calibration logic drives the implementation.
  **CN**: Quantization——低精度转换或校准逻辑是实现重点。
- **EN**: TorchScript/JIT — TorchScript/JIT concepts appear directly in the implementation.
  **CN**: TorchScript/JIT——TorchScript/JIT 概念直接出现在实现中。
- **EN**: Primary type `QuantizedLinear` — the file exposes `QuantizedLinear` as a central abstraction or implementation unit.
  **CN**: 核心类型 `QuantizedLinear`——该文件把 `QuantizedLinear` 作为重要抽象或实现单元。

## Dependencies / 依赖关系

- **Torch imports / Torch 导入**: `torch`
- **Other imports / 其他导入**: 无
- **Top-level classes / 顶层类**: `QuantizedLinear`、`QuantizedLinearFP16`、`QuantizedRNNCellBase`、`QuantizedRNNCell`、`QuantizedLSTMCell`、`QuantizedGRUCell`、`QuantizedRNNBase`、`QuantizedLSTM`、`QuantizedGRU`
- **Top-level functions / 顶层函数**: `quantize_rnn_cell_modules`、`quantize_linear_modules`、`quantize_rnn_modules`
- **Base classes / 基类**: `torch.jit.ScriptModule`、`QuantizedRNNCellBase`、`QuantizedRNNBase`
- **Decorators / 装饰器**: 无
- **Module assignments / 模块级赋值**: 无
