# quantization_patterns.h — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/passes/quantization/quantization_patterns.h`
- Repository: `pytorch`
- Purpose (EN): Declares the TorchScript JIT pass interfaces, helpers, and data structures for quantization patterns.
- 用途 (CN): 声明与 quantization patterns 相关的 TorchScript JIT pass 接口、辅助函数和数据结构。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12
```cpp
#pragma once

#include <c10/util/irange.h>
#include <torch/csrc/jit/ir/ir.h>
#include <torch/csrc/jit/ir/subgraph_matcher.h>
#include <torch/csrc/jit/jit_log.h>
#include <torch/csrc/jit/passes/quantization/helper.h>
#include <torch/csrc/jit/passes/subgraph_rewrite.h>
#include <string>
#include <unordered_map>
#include <utility>

```
- EN: Pulls in the headers needed by the quantization patterns logic. Internal dependencies: `c10/util/irange.h`, `torch/csrc/jit/ir/ir.h`, `torch/csrc/jit/ir/subgraph_matcher.h`, `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/quantization/helper.h`, `...`; external dependencies: `string`, `unordered_map`, `utility`.
- CN: 为 quantization patterns 相关逻辑引入所需头文件。内部依赖：`c10/util/irange.h`, `torch/csrc/jit/ir/ir.h`, `torch/csrc/jit/ir/subgraph_matcher.h`, `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/quantization/helper.h`, `...`；外部依赖：`string`, `unordered_map`, `utility`。

### Lines 13-21
```cpp
namespace torch::jit {

struct QuantFusionInfo {
  std::string quantized_op_name;
  std::string pattern;
  std::string replacement;
  std::vector<MatchFilter> filters;
};

```
- EN: Declares core types or data containers for this file. Prominent symbols: `QuantFusionInfo`.
- CN: 声明该文件使用的核心类型或数据容器。主要符号：`QuantFusionInfo`。

### Lines 22-32
```cpp
namespace {
std::string getExtraArgList(std::vector<std::string> extra_args) {
  return std::accumulate(
      extra_args.begin(),
      extra_args.end(),
      std::string(),
      [](const std::string& acc, const std::string& arg) {
        return acc + ", " + arg;
      });
}

```
- EN: This block produces a result or forwards a computed value. Key symbols: `getExtraArgList`, `accumulate`, `begin`, `end`, `string`.
- CN: 该代码块返回结果或转发已计算的值。关键符号：`getExtraArgList`, `accumulate`, `begin`, `end`, `string`。

### Lines 33-41
```cpp
// Get the pattern we want to replace the match with
std::string getAtenOpPattern(
    const std::string& graph_header,
    const std::string& op_name,
    const std::vector<std::string>& extra_op_args,
    bool scalar_args = false) {
  std::vector<std::string> _extra_op_args = extra_op_args;
  std::string aten_op_pattern = graph_header;
  if (scalar_args) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `getAtenOpPattern`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`getAtenOpPattern`。

### Lines 42-51
```cpp
    for (const auto& extra_arg : _extra_op_args) {
      aten_op_pattern
          .append(R"(
          )")
          .append(extra_arg)
          .append("_scalar = aten::item(")
          .append(extra_arg)
          .append(")");
    }

```
- EN: This block iterates over collections or graph structures. Key symbols: `append`, `item`.
- CN: 该代码块遍历集合或图结构。关键符号：`append`, `item`。

### Lines 52-61
```cpp
    for (auto& _extra_op_arg : _extra_op_args) {
      _extra_op_arg.append("_scalar");
    }
  }
  const auto& extra_op_arg_list = getExtraArgList(std::move(_extra_op_args));
  aten_op_pattern += R"(
          %r = )";
  aten_op_pattern += op_name + "(" + "%a_quant" + extra_op_arg_list + ")";
  aten_op_pattern += R"(
          return (%r) )";
```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value. Key symbols: `append`, `getExtraArgList`, `move`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值。关键符号：`append`, `getExtraArgList`, `move`。

### Lines 62-79
```cpp
  return aten_op_pattern;
}

// generate ops for quantize pattern for a scalar value
std::string getQuantizeForScalar(const std::string& value) {
  // 6 is `torch.float` ScalarType, we are creating a float scalar
  // tensor from a scalar value
  std::string quantize_pattern = R"(
          )" +
      value + "_float_scalar_type : int = prim::Constant[value=6]()";
  quantize_pattern += R"(
          )" +
      value + "_none : None = prim::Constant()";
  quantize_pattern += R"(
          )" +
      value + "_tensor : Tensor = aten::scalar_tensor(" + value + ", " + value +
      "_float_scalar_type";
  for ([[maybe_unused]] const auto i : c10::irange(3)) {
```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `getQuantizeForScalar`, `Constant`, `scalar_tensor`, `irange`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`getQuantizeForScalar`, `Constant`, `scalar_tensor`, `irange`。

### Lines 80-90
```cpp
    quantize_pattern += ", " + value + "_none";
  }
  quantize_pattern += ")";
  quantize_pattern +=
      R"(
          )" +
      value + "_quant = aten::quantize_per_tensor(" + value + "_tensor" +
      getExtraArgList(
          {value + "_scale", value + "_zero_point", value + "_dtype"}) +
      ")";
  return quantize_pattern;
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `quantize_per_tensor`, `getExtraArgList`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`quantize_per_tensor`, `getExtraArgList`。

### Lines 91-99
```cpp
}

std::string getDequantize(const std::string& value) {
  return R"(
          )" +
      value + "_dequant = aten::dequantize(" + value + "_quant)";
}

std::string getItem(const std::string& value) {
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `getDequantize`, `dequantize`, `getItem`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`getDequantize`, `dequantize`, `getItem`。

### Lines 100-110
```cpp
  return R"(
          )" +
      value + "_scalar : float = aten::item(" + value + "_dequant)";
}

// Patterns for the ops that inherit parameters from input
std::string getInputTensorQParamOpPattern(
    const std::string& op_name,
    const std::vector<std::string>& extra_op_args) {
  const auto& extra_op_arg_list = getExtraArgList(extra_op_args);
  std::string op_pattern = "graph(%a_quant" + extra_op_arg_list + "):" + R"(
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `item`, `getInputTensorQParamOpPattern`, `getExtraArgList`, `graph`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`item`, `getInputTensorQParamOpPattern`, `getExtraArgList`, `graph`。

### Lines 111-119
```cpp
          %a_dequant = aten::dequantize(%a_quant)
          %r = )" +
      op_name + "(" + "%a_dequant" + extra_op_arg_list + ")" + R"(
          %r_scale : float = aten::q_scale(%a_quant)
          %r_zero_point : int = aten::q_zero_point(%a_quant)
          %r_dtype : int = prim::dtype(%a_quant)
          %r_quant = aten::quantize_per_tensor(%r, %r_scale, %r_zero_point, %r_dtype)
          return (%r_quant) )";
  return op_pattern;
```
- EN: This block produces a result or forwards a computed value. Key symbols: `dequantize`, `q_scale`, `q_zero_point`, `dtype`, `quantize_per_tensor`.
- CN: 该代码块返回结果或转发已计算的值。关键符号：`dequantize`, `q_scale`, `q_zero_point`, `dtype`, `quantize_per_tensor`。

### Lines 120-129
```cpp
}

// QuantFusionInfo for the ops that inherit parameters from input
QuantFusionInfo getInputTensorQParamOpFusionInfo(
    const std::string& op_name,
    const std::vector<std::string>& extra_op_args) {
  std::string op_pattern =
      getInputTensorQParamOpPattern(op_name, extra_op_args);
  const auto& extra_op_arg_list = getExtraArgList(extra_op_args);
  std::string graph_header = "graph(%a_quant" + extra_op_arg_list + "):";
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `getInputTensorQParamOpFusionInfo`, `getInputTensorQParamOpPattern`, `getExtraArgList`, `graph`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`getInputTensorQParamOpFusionInfo`, `getInputTensorQParamOpPattern`, `getExtraArgList`, `graph`。

### Lines 130-143
```cpp
  std::string op_replacement =
      getAtenOpPattern(graph_header, op_name, extra_op_args);

  return {op_name, std::move(op_pattern), std::move(op_replacement)};
}

// quant fusion for ops like `quantized::add_scalar`, `quantized::mul_scalar`
QuantFusionInfo getBinaryOpScalarFusionInfo(
    const std::string& op_name,
    const std::vector<std::string>& extra_op_args,
    const std::string& quantized_op_name,
    const std::vector<std::string>& extra_quantized_op_args,
    const std::vector<MatchFilter>& filters = {}) {
  std::string op_pattern =
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `getAtenOpPattern`, `move`, `getBinaryOpScalarFusionInfo`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`getAtenOpPattern`, `move`, `getBinaryOpScalarFusionInfo`。

### Lines 144-153
```cpp
      getInputTensorQParamOpPattern(op_name, extra_op_args);

  const auto& extra_op_arg_list = getExtraArgList(extra_op_args);
  std::string graph_header = "graph(%a_quant" + extra_op_arg_list + "):";
  std::string op_replacement = getAtenOpPattern(
      graph_header, quantized_op_name, extra_quantized_op_args);

  return {op_name, std::move(op_pattern), std::move(op_replacement), filters};
}

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `getInputTensorQParamOpPattern`, `getExtraArgList`, `graph`, `getAtenOpPattern`, `move`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`getInputTensorQParamOpPattern`, `getExtraArgList`, `graph`, `getAtenOpPattern`, `move`。

### Lines 154-164
```cpp
QuantFusionInfo getClampOpFusionInfo(
    const std::string& op_name,
    const std::vector<std::string>& extra_op_args) {
  std::vector<std::string> header_args = extra_op_args;
  std::vector<std::string> input_qparams = {"_scale", "_zero_point", "_dtype"};
  for (const auto& arg : extra_op_args) {
    for (const auto& qparam : input_qparams) {
      header_args.push_back(arg + qparam);
    }
  }
  for (const auto& qparam : input_qparams) {
```
- EN: This block iterates over collections or graph structures. Key symbols: `getClampOpFusionInfo`, `push_back`.
- CN: 该代码块遍历集合或图结构。关键符号：`getClampOpFusionInfo`, `push_back`。

### Lines 165-178
```cpp
    header_args.push_back("%r" + qparam);
  }
  const auto& extra_header_arg_list = getExtraArgList(std::move(header_args));
  std::string graph_header = "graph(%a_quant" + extra_header_arg_list + "):";
  std::string op_pattern = graph_header;
  for (const auto& arg : extra_op_args) {
    op_pattern += getQuantizeForScalar(arg);
    op_pattern += getDequantize(arg);
    op_pattern += getItem(arg);
  }
  op_pattern += getDequantize("%a");
  op_pattern += R"(
          %r = )";
  std::vector<std::string> scalar_extra_args;
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `push_back`, `getExtraArgList`, `move`, `graph`, `getQuantizeForScalar`, `getDequantize`, `...`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`push_back`, `getExtraArgList`, `move`, `graph`, `getQuantizeForScalar`, `getDequantize`, `...`。

### Lines 179-188
```cpp
  scalar_extra_args.reserve(extra_op_args.size());
  for (const auto& arg : extra_op_args) {
    scalar_extra_args.push_back(arg + "_scalar");
  }
  op_pattern += op_name + "(" + "%a_dequant" +
      getExtraArgList(std::move(scalar_extra_args)) + ")";
  // IR pattern common to all ops that inherit qparam from input
  op_pattern += R"(
          %r_quant = aten::quantize_per_tensor(%r, %r_scale, %r_zero_point, %r_dtype)
          return (%r_quant) )";
```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value. Key symbols: `reserve`, `size`, `push_back`, `getExtraArgList`, `move`, `quantize_per_tensor`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值。关键符号：`reserve`, `size`, `push_back`, `getExtraArgList`, `move`, `quantize_per_tensor`。

### Lines 189-200
```cpp

  std::string aten_op_pattern =
      getAtenOpPattern(graph_header, op_name, extra_op_args);

  return {op_name, std::move(op_pattern), std::move(aten_op_pattern)};
}

// Patterns for the ops that has fixed quantization parameters
QuantFusionInfo getFixedQParamOpFusionInfo(
    const std::string& op_name,
    const std::vector<std::string>& extra_op_args,
    bool is_symmetric) {
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `getAtenOpPattern`, `move`, `getFixedQParamOpFusionInfo`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`getAtenOpPattern`, `move`, `getFixedQParamOpFusionInfo`。

### Lines 201-210
```cpp
  const auto& extra_op_arg_list = getExtraArgList(extra_op_args);
  std::string graph_header = "graph(%a_quant" + extra_op_arg_list + "):";
  std::string op_pattern = graph_header;
  op_pattern += R"(
          %a_dequant = aten::dequantize(%a_quant)
          %r = )";
  op_pattern += op_name + "(" + "%a_dequant" + extra_op_arg_list + ")";
  // IR pattern common to all ops with fixed quantization parameters for
  // asymmetric quantization
  std::string asym_fixed_qparam_op_suffix = R"(
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `getExtraArgList`, `graph`, `dequantize`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`getExtraArgList`, `graph`, `dequantize`。

### Lines 211-222
```cpp
          %r_scale : float = prim::Constant[value=0.00390625]()
          %r_zero_point : int = prim::Constant[value=0]()
          %r_dtype : int = prim::Constant[value=13]()
          %r_quant = aten::quantize_per_tensor(%r, %r_scale, %r_zero_point, %r_dtype)
          return (%r_quant) )";

  std::string sym_fixed_qparam_op_suffix = R"(
          %r_scale : float = prim::Constant[value=0.0078125]()
          %r_zero_point : int = prim::Constant[value=128]()
          %r_dtype : int = prim::Constant[value=13]()
          %r_quant = aten::quantize_per_tensor(%r, %r_scale, %r_zero_point, %r_dtype)
          return (%r_quant) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `quantize_per_tensor`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`quantize_per_tensor`。

### Lines 223-231
```cpp
  op_pattern +=
      is_symmetric ? sym_fixed_qparam_op_suffix : asym_fixed_qparam_op_suffix;

  std::string aten_op_pattern =
      getAtenOpPattern(graph_header, op_name, extra_op_args);

  return {op_name, std::move(op_pattern), std::move(aten_op_pattern)};
}

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `getAtenOpPattern`, `move`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`getAtenOpPattern`, `move`。

### Lines 232-240
```cpp
// filter that checks %b_scalar is a scalar
bool input_b_is_scalar(
    const Match& match,
    const std::unordered_map<std::string, Value*>& vmap) {
  const auto& match_vmap = match.values_map;
  auto b_scalar = match_vmap.at(vmap.at("b_scalar"));
  return isScalar(b_scalar);
}

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `input_b_is_scalar`, `isScalar`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`input_b_is_scalar`, `isScalar`。

### Lines 241-258
```cpp
// Patterns for ops that require observation for output quantization parameters
// Example:
//
// before fusion:
//
// graph(%a_quant, %r_scale, %r_zero_point, %r_dtype):
//     %a_dequant = aten::dequantize(%a_quant)
//     %r = {op_name}(%a_dequant, {extra_args})
//     %r_quant = aten::quantize_per_tensor(%r, %r_scale, %r_zero_point,
//     %r_dtype) return (%r_quant)
//
// after fusion:
//
// graph(%a_quant, %r_scale, %r_zero_point, %r_dtype):
//     %r_quant = {quantized_op_name}(%a_quant, {extra_args}, %r_scale,
//     %r_zero_point) return (%r_quant)
QuantFusionInfo getObservedQParamOpFusionInfo(
    const std::string& fp_op_name,
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `getObservedQParamOpFusionInfo`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`getObservedQParamOpFusionInfo`。

### Lines 259-271
```cpp
    const std::string& q_op_name,
    const std::vector<std::string>& fp_extra_args,
    const std::vector<std::string>& q_extra_args) {
  const auto& fp_extra_arg_list = getExtraArgList(fp_extra_args);
  const auto& q_extra_arg_list = getExtraArgList(q_extra_args);

  std::string op_pattern = "graph(%a_quant" + fp_extra_arg_list +
      ", %r_scale, %r_zero_point, %r_dtype):" + R"(
          %a_dequant = aten::dequantize(%a_quant)
          %r = )" +
      fp_op_name + "(" + "%a_dequant" + fp_extra_arg_list + ")" + R"(
          %r_quant = aten::quantize_per_tensor(%r, %r_scale, %r_zero_point, %r_dtype)
          return (%r_quant) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `getExtraArgList`, `graph`, `dequantize`, `quantize_per_tensor`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`getExtraArgList`, `graph`, `dequantize`, `quantize_per_tensor`。

### Lines 272-280
```cpp

  std::string aten_op_pattern = "graph(%a_quant" + fp_extra_arg_list +
      ", %r_scale, %r_zero_point, %r_dtype):" + R"(
          %r_quant = )" +
      q_op_name + "(%a_quant" + q_extra_arg_list +
      ", %r_scale, %r_zero_point)" + R"(
          return (%r_quant) )";

  return {q_op_name, std::move(op_pattern), std::move(aten_op_pattern)};
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `move`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `move`。

### Lines 281-294
```cpp
}

} // namespace

static std::vector<QuantFusionInfo> quant_fusion_pattern_and_replacements() {
  // aten::conv1d
  std::string conv1d = R"(
graph(%a_quant, %packed_params, %r_scale, %r_zero_point, %r_dtype, %stride, %padding, %dilation, %groups):
        %a_dequant = aten::dequantize(%a_quant)
        %w_quant : Tensor, %b : Tensor? = quantized::conv1d_unpack(%packed_params)
        %w_dequant = aten::dequantize(%w_quant)
        %r = aten::conv1d(%a_dequant, %w_dequant, %b, %stride, %padding, %dilation, %groups)
        %r_quant = aten::quantize_per_tensor(%r, %r_scale, %r_zero_point, %r_dtype)
        return (%r_quant) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `quant_fusion_pattern_and_replacements`, `graph`, `dequantize`, `conv1d_unpack`, `conv1d`, `quantize_per_tensor`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`quant_fusion_pattern_and_replacements`, `graph`, `dequantize`, `conv1d_unpack`, `conv1d`, `quantize_per_tensor`。

### Lines 295-305
```cpp

  // aten::conv1d - aten::relu
  std::string conv1d_relu = R"(
graph(%a_quant, %packed_params, %r_scale, %r_zero_point, %r_dtype, %stride, %padding, %dilation, %groups):
        %a_dequant = aten::dequantize(%a_quant)
        %w_quant : Tensor, %b : Tensor? = quantized::conv1d_unpack(%packed_params)
        %w_dequant = aten::dequantize(%w_quant)
        %conv_out = aten::conv1d(%a_dequant, %w_dequant, %b, %stride, %padding, %dilation, %groups)
        %r = aten::relu(%conv_out)
        %r_quant = aten::quantize_per_tensor(%r, %r_scale, %r_zero_point, %r_dtype)
        return (%r_quant) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `dequantize`, `conv1d_unpack`, `conv1d`, `relu`, `quantize_per_tensor`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `dequantize`, `conv1d_unpack`, `conv1d`, `relu`, `quantize_per_tensor`。

### Lines 306-316
```cpp

  // aten::conv1d - aten::relu_
  std::string conv1d_inplace_relu = R"(
graph(%a_quant, %packed_params, %r_scale, %r_zero_point, %r_dtype, %stride, %padding, %dilation, %groups):
        %a_dequant = aten::dequantize(%a_quant)
        %w_quant : Tensor, %b : Tensor? = quantized::conv1d_unpack(%packed_params)
        %w_dequant = aten::dequantize(%w_quant)
        %conv_out = aten::conv1d(%a_dequant, %w_dequant, %b, %stride, %padding, %dilation, %groups)
        %r = aten::relu_(%conv_out)
        %r_quant = aten::quantize_per_tensor(%r, %r_scale, %r_zero_point, %r_dtype)
        return (%r_quant) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `dequantize`, `conv1d_unpack`, `conv1d`, `relu_`, `quantize_per_tensor`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `dequantize`, `conv1d_unpack`, `conv1d`, `relu_`, `quantize_per_tensor`。

### Lines 317-325
```cpp

  // quantized::conv1d
  std::string quantized_conv1d = R"(
graph(%a_quant, %packed_params, %r_scale, %r_zero_point, %r_dtype, %stride, %padding, %dilation, %groups):
        %r_quant = quantized::conv1d(%a_quant, %packed_params, %r_scale, %r_zero_point)
        return (%r_quant) )";

  // quantized::conv1d_relu
  std::string quantized_conv1d_relu = R"(
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `conv1d`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `conv1d`。

### Lines 326-338
```cpp
graph(%a_quant, %packed_params, %r_scale, %r_zero_point, %r_dtype, %stride, %padding, %dilation, %groups):
        %r_quant = quantized::conv1d_relu(%a_quant, %packed_params, %r_scale, %r_zero_point)
        return (%r_quant) )";

  // aten::conv2d
  std::string conv2d = R"(
graph(%a_quant, %packed_params, %r_scale, %r_zero_point, %r_dtype, %stride, %padding, %dilation, %groups):
        %a_dequant = aten::dequantize(%a_quant)
        %w_quant : Tensor, %b : Tensor? = quantized::conv2d_unpack(%packed_params)
        %w_dequant = aten::dequantize(%w_quant)
        %r = aten::conv2d(%a_dequant, %w_dequant, %b, %stride, %padding, %dilation, %groups)
        %r_quant = aten::quantize_per_tensor(%r, %r_scale, %r_zero_point, %r_dtype)
        return (%r_quant) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `conv1d_relu`, `dequantize`, `conv2d_unpack`, `conv2d`, `quantize_per_tensor`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `conv1d_relu`, `dequantize`, `conv2d_unpack`, `conv2d`, `quantize_per_tensor`。

### Lines 339-349
```cpp

  // aten::conv2d - aten::relu
  std::string conv2d_relu = R"(
graph(%a_quant, %packed_params, %r_scale, %r_zero_point, %r_dtype, %stride, %padding, %dilation, %groups):
        %a_dequant = aten::dequantize(%a_quant)
        %w_quant : Tensor, %b : Tensor? = quantized::conv2d_unpack(%packed_params)
        %w_dequant = aten::dequantize(%w_quant)
        %conv_out = aten::conv2d(%a_dequant, %w_dequant, %b, %stride, %padding, %dilation, %groups)
        %r = aten::relu(%conv_out)
        %r_quant = aten::quantize_per_tensor(%r, %r_scale, %r_zero_point, %r_dtype)
        return (%r_quant) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `dequantize`, `conv2d_unpack`, `conv2d`, `relu`, `quantize_per_tensor`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `dequantize`, `conv2d_unpack`, `conv2d`, `relu`, `quantize_per_tensor`。

### Lines 350-360
```cpp

  // aten::conv2d - aten::relu_
  std::string conv2d_inplace_relu = R"(
graph(%a_quant, %packed_params, %r_scale, %r_zero_point, %r_dtype, %stride, %padding, %dilation, %groups):
        %a_dequant = aten::dequantize(%a_quant)
        %w_quant : Tensor, %b : Tensor? = quantized::conv2d_unpack(%packed_params)
        %w_dequant = aten::dequantize(%w_quant)
        %conv_out = aten::conv2d(%a_dequant, %w_dequant, %b, %stride, %padding, %dilation, %groups)
        %r = aten::relu_(%conv_out)
        %r_quant = aten::quantize_per_tensor(%r, %r_scale, %r_zero_point, %r_dtype)
        return (%r_quant) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `dequantize`, `conv2d_unpack`, `conv2d`, `relu_`, `quantize_per_tensor`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `dequantize`, `conv2d_unpack`, `conv2d`, `relu_`, `quantize_per_tensor`。

### Lines 361-369
```cpp

  // quantized::conv2d
  std::string quantized_conv2d = R"(
graph(%a_quant, %packed_params, %r_scale, %r_zero_point, %r_dtype, %stride, %padding, %dilation, %groups):
        %r_quant = quantized::conv2d(%a_quant, %packed_params, %r_scale, %r_zero_point)
        return (%r_quant) )";

  // quantized::conv2d_relu
  std::string quantized_conv2d_relu = R"(
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `conv2d`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `conv2d`。

### Lines 370-382
```cpp
graph(%a_quant, %packed_params, %r_scale, %r_zero_point, %r_dtype, %stride, %padding, %dilation, %groups):
        %r_quant = quantized::conv2d_relu(%a_quant, %packed_params, %r_scale, %r_zero_point)
        return (%r_quant) )";

  // aten::conv3d
  std::string conv3d = R"(
graph(%a_quant, %packed_params, %r_scale, %r_zero_point, %r_dtype, %stride, %padding, %dilation, %groups):
        %a_dequant = aten::dequantize(%a_quant)
        %w_quant : Tensor, %b : Tensor? = quantized::conv3d_unpack(%packed_params)
        %w_dequant = aten::dequantize(%w_quant)
        %r = aten::conv3d(%a_dequant, %w_dequant, %b, %stride, %padding, %dilation, %groups)
        %r_quant = aten::quantize_per_tensor(%r, %r_scale, %r_zero_point, %r_dtype)
        return (%r_quant) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `conv2d_relu`, `dequantize`, `conv3d_unpack`, `conv3d`, `quantize_per_tensor`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `conv2d_relu`, `dequantize`, `conv3d_unpack`, `conv3d`, `quantize_per_tensor`。

### Lines 383-393
```cpp

  // aten::conv3d - aten::relu
  std::string conv3d_relu = R"(
graph(%a_quant, %packed_params, %r_scale, %r_zero_point, %r_dtype, %stride, %padding, %dilation, %groups):
        %a_dequant = aten::dequantize(%a_quant)
        %w_quant : Tensor, %b : Tensor? = quantized::conv3d_unpack(%packed_params)
        %w_dequant = aten::dequantize(%w_quant)
        %conv_out = aten::conv3d(%a_dequant, %w_dequant, %b, %stride, %padding, %dilation, %groups)
        %r = aten::relu(%conv_out)
        %r_quant = aten::quantize_per_tensor(%r, %r_scale, %r_zero_point, %r_dtype)
        return (%r_quant) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `dequantize`, `conv3d_unpack`, `conv3d`, `relu`, `quantize_per_tensor`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `dequantize`, `conv3d_unpack`, `conv3d`, `relu`, `quantize_per_tensor`。

### Lines 394-404
```cpp

  // aten::conv3d - aten::relu_
  std::string conv3d_inplace_relu = R"(
graph(%a_quant, %packed_params, %r_scale, %r_zero_point, %r_dtype, %stride, %padding, %dilation, %groups):
        %a_dequant = aten::dequantize(%a_quant)
        %w_quant : Tensor, %b : Tensor? = quantized::conv3d_unpack(%packed_params)
        %w_dequant = aten::dequantize(%w_quant)
        %conv_out = aten::conv3d(%a_dequant, %w_dequant, %b, %stride, %padding, %dilation, %groups)
        %r = aten::relu_(%conv_out)
        %r_quant = aten::quantize_per_tensor(%r, %r_scale, %r_zero_point, %r_dtype)
        return (%r_quant) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `dequantize`, `conv3d_unpack`, `conv3d`, `relu_`, `quantize_per_tensor`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `dequantize`, `conv3d_unpack`, `conv3d`, `relu_`, `quantize_per_tensor`。

### Lines 405-413
```cpp

  // quantized::conv3d
  std::string quantized_conv3d = R"(
graph(%a_quant, %packed_params, %r_scale, %r_zero_point, %r_dtype, %stride, %padding, %dilation, %groups):
        %r_quant = quantized::conv3d(%a_quant, %packed_params, %r_scale, %r_zero_point)
        return (%r_quant) )";

  // quantized::conv3d_relu
  std::string quantized_conv3d_relu = R"(
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `conv3d`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `conv3d`。

### Lines 414-426
```cpp
graph(%a_quant, %packed_params, %r_scale, %r_zero_point, %r_dtype, %stride, %padding, %dilation, %groups):
        %r_quant = quantized::conv3d_relu(%a_quant, %packed_params, %r_scale, %r_zero_point)
        return (%r_quant) )";

  // aten::conv_transpose1d
  std::string conv_transpose1d = R"(
graph(%a_quant, %packed_params, %r_scale, %r_zero_point, %r_dtype, %stride, %padding, %output_padding, %groups, %dilation):
        %a_dequant = aten::dequantize(%a_quant)
        %w_quant : Tensor, %b : Tensor? = quantized::conv_transpose1d_unpack(%packed_params)
        %w_dequant = aten::dequantize(%w_quant)
        %r = aten::conv_transpose1d(%a_dequant, %w_dequant, %b, %stride, %padding, %output_padding, %groups, %dilation)
        %r_quant = aten::quantize_per_tensor(%r, %r_scale, %r_zero_point, %r_dtype)
        return (%r_quant) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `conv3d_relu`, `dequantize`, `conv_transpose1d_unpack`, `conv_transpose1d`, `quantize_per_tensor`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `conv3d_relu`, `dequantize`, `conv_transpose1d_unpack`, `conv_transpose1d`, `quantize_per_tensor`。

### Lines 427-435
```cpp

  // quantized::conv_transpose1d
  std::string quantized_conv_transpose1d = R"(
graph(%a_quant, %packed_params, %r_scale, %r_zero_point, %r_dtype, %stride, %padding, %output_padding, %groups, %dilation):
        %r_quant = quantized::conv_transpose1d(%a_quant, %packed_params, %r_scale, %r_zero_point)
        return (%r_quant) )";

  // aten::conv_transpose2d
  std::string conv_transpose2d = R"(
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `conv_transpose1d`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `conv_transpose1d`。

### Lines 436-445
```cpp
graph(%a_quant, %packed_params, %r_scale, %r_zero_point, %r_dtype, %stride, %padding, %output_padding, %groups, %dilation):
        %a_dequant = aten::dequantize(%a_quant)
        %w_quant : Tensor, %b : Tensor? = quantized::conv_transpose2d_unpack(%packed_params)
        %w_dequant = aten::dequantize(%w_quant)
        %r = aten::conv_transpose2d(%a_dequant, %w_dequant, %b, %stride, %padding, %output_padding, %groups, %dilation)
        %r_quant = aten::quantize_per_tensor(%r, %r_scale, %r_zero_point, %r_dtype)
        return (%r_quant) )";

  // quantized::conv_transpose1d
  std::string quantized_conv_transpose2d = R"(
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `dequantize`, `conv_transpose2d_unpack`, `conv_transpose2d`, `quantize_per_tensor`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `dequantize`, `conv_transpose2d_unpack`, `conv_transpose2d`, `quantize_per_tensor`。

### Lines 446-457
```cpp
graph(%a_quant, %packed_params, %r_scale, %r_zero_point, %r_dtype, %stride, %padding, %output_padding, %groups, %dilation):
        %r_quant = quantized::conv_transpose2d(%a_quant, %packed_params, %r_scale, %r_zero_point)
        return (%r_quant) )";

  std::string add_relu = R"(
graph(%a_quant, %b_quant, %alpha, %scale, %zero_point, %dtype):
         %a_dequant = aten::dequantize(%a_quant)
         %b_dequant = aten::dequantize(%b_quant)
         %r_add = aten::add(%a_dequant, %b_dequant, %alpha)
         %r_relu = aten::relu(%r_add)
         %r = aten::quantize_per_tensor(%r_relu, %scale, %zero_point, %dtype)
         return (%r) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `conv_transpose2d`, `dequantize`, `add`, `relu`, `quantize_per_tensor`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `conv_transpose2d`, `dequantize`, `add`, `relu`, `quantize_per_tensor`。

### Lines 458-466
```cpp

  std::string add_inplace_relu = R"(
graph(%a_quant, %b_quant, %alpha, %scale, %zero_point, %dtype):
         %a_dequant = aten::dequantize(%a_quant)
         %b_dequant = aten::dequantize(%b_quant)
         %r_add = aten::add(%a_dequant, %b_dequant, %alpha)
         %r_relu = aten::relu_(%r_add)
         %r = aten::quantize_per_tensor(%r_relu, %scale, %zero_point, %dtype)
         return (%r) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `dequantize`, `add`, `relu_`, `quantize_per_tensor`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `dequantize`, `add`, `relu_`, `quantize_per_tensor`。

### Lines 467-475
```cpp

  std::string inplace_add_relu = R"(
graph(%a_quant, %b_quant, %alpha, %scale, %zero_point, %dtype):
         %a_dequant = aten::dequantize(%a_quant)
         %b_dequant = aten::dequantize(%b_quant)
         %r_add = aten::add_(%a_dequant, %b_dequant, %alpha)
         %r_relu = aten::relu(%r_add)
         %r = aten::quantize_per_tensor(%r_relu, %scale, %zero_point, %dtype)
         return (%r) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `dequantize`, `add_`, `relu`, `quantize_per_tensor`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `dequantize`, `add_`, `relu`, `quantize_per_tensor`。

### Lines 476-484
```cpp

  std::string inplace_add_inplace_relu = R"(
graph(%a_quant, %b_quant, %alpha, %scale, %zero_point, %dtype):
         %a_dequant = aten::dequantize(%a_quant)
         %b_dequant = aten::dequantize(%b_quant)
         %r_add = aten::add_(%a_dequant, %b_dequant, %alpha)
         %r_relu = aten::relu_(%r_add)
         %r = aten::quantize_per_tensor(%r_relu, %scale, %zero_point, %dtype)
         return (%r) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `dequantize`, `add_`, `relu_`, `quantize_per_tensor`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `dequantize`, `add_`, `relu_`, `quantize_per_tensor`。

### Lines 485-499
```cpp

  std::string quantized_add_relu = R"(
graph(%a_quant, %b_quant, %alpha, %scale, %zero_point, %dtype):
         %r = quantized::add_relu(%a_quant, %b_quant, %scale, %zero_point)
         return (%r) )";

  // aten::linear
  std::string linear = R"(
graph(%a_quant, %packed_params, %r_scale, %r_zero_point, %r_dtype):
        %a_dequant = aten::dequantize(%a_quant)
        %w_quant : Tensor, %b : Tensor? = quantized::linear_unpack(%packed_params)
        %w_dequant = aten::dequantize(%w_quant)
        %r = aten::linear(%a_dequant, %w_dequant, %b)
        %r_quant = aten::quantize_per_tensor(%r, %r_scale, %r_zero_point, %r_dtype)
        return (%r_quant) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `add_relu`, `dequantize`, `linear_unpack`, `linear`, `quantize_per_tensor`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `add_relu`, `dequantize`, `linear_unpack`, `linear`, `quantize_per_tensor`。

### Lines 500-509
```cpp

  std::string linear_relu = R"(
graph(%a_quant, %packed_params, %r_scale, %r_zero_point, %r_dtype):
        %a_dequant = aten::dequantize(%a_quant)
        %w_quant : Tensor, %b : Tensor? = quantized::linear_unpack(%packed_params)
        %w_dequant = aten::dequantize(%w_quant)
        %linear_out = aten::linear(%a_dequant, %w_dequant, %b)
        %r = aten::relu(%linear_out)
        %r_quant = aten::quantize_per_tensor(%r, %r_scale, %r_zero_point, %r_dtype)
        return (%r_quant) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `dequantize`, `linear_unpack`, `linear`, `relu`, `quantize_per_tensor`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `dequantize`, `linear_unpack`, `linear`, `relu`, `quantize_per_tensor`。

### Lines 510-519
```cpp

  std::string linear_inplace_relu = R"(
graph(%a_quant, %packed_params, %r_scale, %r_zero_point, %r_dtype):
        %a_dequant = aten::dequantize(%a_quant)
        %w_quant : Tensor, %b : Tensor? = quantized::linear_unpack(%packed_params)
        %w_dequant = aten::dequantize(%w_quant)
        %linear_out = aten::linear(%a_dequant, %w_dequant, %b)
        %r = aten::relu_(%linear_out)
        %r_quant = aten::quantize_per_tensor(%r, %r_scale, %r_zero_point, %r_dtype)
        return (%r_quant) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `dequantize`, `linear_unpack`, `linear`, `relu_`, `quantize_per_tensor`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `dequantize`, `linear_unpack`, `linear`, `relu_`, `quantize_per_tensor`。

### Lines 520-530
```cpp

  // quantized::linear
  std::string quantized_linear = R"(
graph(%a_quant, %packed_params, %r_scale, %r_zero_point, %r_dtype):
        %r = quantized::linear(%a_quant, %packed_params, %r_scale, %r_zero_point)
        return (%r) )";

  std::string quantized_linear_relu = R"(
graph(%a_quant, %packed_params, %r_scale, %r_zero_point, %r_dtype):
        %r = quantized::linear_relu(%a_quant, %packed_params, %r_scale, %r_zero_point)
        return (%r) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `linear`, `linear_relu`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `linear`, `linear_relu`。

### Lines 531-539
```cpp

  std::string cat = R"(
graph(%input_quant, %dim, %r_scale, %r_zero_point, %r_dtype):
        %input_dequant = aten::dequantize(%input_quant)
        %r = aten::cat(%input_dequant, %dim)
        %r_quant = aten::quantize_per_tensor(%r, %r_scale, %r_zero_point, %r_dtype)
        return (%r_quant) )";

  std::string quantized_cat = R"(
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `dequantize`, `cat`, `quantize_per_tensor`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `dequantize`, `cat`, `quantize_per_tensor`。

### Lines 540-551
```cpp
graph(%input_quant, %dim, %r_scale, %r_zero_point, %r_dtype):
         %r_quant = quantized::cat(%input_quant, %dim, %r_scale, %r_zero_point)
         return (%r_quant) )";

  // aten::add
  std::string add = R"(
graph(%a_quant, %b_quant, %alpha, %scale, %zero_point, %dtype):
         %a_dequant = aten::dequantize(%a_quant)
         %b_dequant = aten::dequantize(%b_quant)
         %r_add = aten::add(%a_dequant, %b_dequant, %alpha)
         %r = aten::quantize_per_tensor(%r_add, %scale, %zero_point, %dtype)
         return (%r) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `cat`, `dequantize`, `add`, `quantize_per_tensor`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `cat`, `dequantize`, `add`, `quantize_per_tensor`。

### Lines 552-560
```cpp

  // TODO: add %dtype after when https://github.com/pytorch/pytorch/issues/34351
  // is fixed
  // quantized::add
  std::string quantized_add = R"(
graph(%a_quant, %b_quant, %alpha, %scale, %zero_point, %dtype):
         %r = quantized::add(%a_quant, %b_quant, %scale, %zero_point)
         return (%r) )";

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `add`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `add`。

### Lines 561-569
```cpp
  // aten::add_
  std::string inplace_add = R"(
graph(%a_quant, %b_quant, %alpha, %scale, %zero_point, %dtype):
         %a_dequant = aten::dequantize(%a_quant)
         %b_dequant = aten::dequantize(%b_quant)
         %r_add = aten::add_(%a_dequant, %b_dequant, %alpha)
         %r = aten::quantize_per_tensor(%r_add, %scale, %zero_point, %dtype)
         return (%r) )";

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `dequantize`, `add_`, `quantize_per_tensor`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `dequantize`, `add_`, `quantize_per_tensor`。

### Lines 570-583
```cpp
  auto add_scalar = getBinaryOpScalarFusionInfo(
      "aten::add",
      {"%b_scalar", "%alpha"},
      "quantized::add_scalar",
      {"%b_scalar"},
      {aten_add_alpha_is_one, input_b_is_scalar});

  auto add_scalar_out = getBinaryOpScalarFusionInfo(
      "aten::add_",
      {"%b_scalar", "%alpha"},
      "quantized::add_scalar_out",
      {"%b_scalar", "%a_quant"},
      {aten_add_alpha_is_one, input_b_is_scalar});

```
- EN: This block implements local helper logic for quantization patterns. Key symbols: `getBinaryOpScalarFusionInfo`.
- CN: 该代码块实现与 quantization patterns 相关的局部辅助逻辑。关键符号：`getBinaryOpScalarFusionInfo`。

### Lines 584-592
```cpp
  // quantized::add_scalar_relu -- fusing quantized::add_scalar
  // and aten::relu
  auto quantized_add_scalar_relu_pattern = R"(
graph(%a_quant, %b_scalar):
         %r_add = quantized::add_scalar(%a_quant, %b_scalar)
         %r = aten::relu(%r_add)
         return (%r) )";

  auto quantized_add_scalar_inplace_relu_pattern = R"(
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `add_scalar`, `relu`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `add_scalar`, `relu`。

### Lines 593-601
```cpp
graph(%a_quant, %b_scalar):
         %r_add = quantized::add_scalar(%a_quant, %b_scalar)
         %r = aten::relu_(%r_add)
         return (%r) )";

  auto quantized_add_scalar_relu_replacement = R"(
graph(%a_quant, %b_scalar):
         %r = quantized::add_scalar_relu(%a_quant, %b_scalar)
         return (%r) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `add_scalar`, `relu_`, `add_scalar_relu`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `add_scalar`, `relu_`, `add_scalar_relu`。

### Lines 602-610
```cpp

  // quantized::add_scalar_relu_out -- fusing quantized::add_scalarOut
  // and aten::relu
  auto quantized_add_scalar_relu_out_pattern = R"(
graph(%a_quant, %b_scalar):
         %r_add = quantized::add_scalar_out(%a_quant, %b_scalar, %a_quant)
         %r = aten::relu(%r_add)
         return (%r) )";

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `add_scalar_out`, `relu`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `add_scalar_out`, `relu`。

### Lines 611-620
```cpp
  auto quantized_add_scalar_inplace_relu_out_pattern = R"(
graph(%a_quant, %b_scalar):
         %r_add = quantized::add_scalar_out(%a_quant, %b_scalar, %a_quant)
         %r = aten::relu_(%r_add)
         return (%r) )";

  auto quantized_add_scalar_relu_out_replacement = R"(
graph(%a_quant, %b_scalar):
         %r = quantized::add_scalar_relu_out(%a_quant, %b_scalar, %a_quant)
         return (%r) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `add_scalar_out`, `relu_`, `add_scalar_relu_out`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `add_scalar_out`, `relu_`, `add_scalar_relu_out`。

### Lines 621-629
```cpp

  // quantized::batch_norm
  std::string batch_norm = R"(
graph(%a_quant, %weight, %bias, %mean, %var, %training, %eaf, %eps, %7, %scale, %zero_point, %scalar_type):
         %a_dequant = aten::dequantize(%a_quant)
         %r_bn = aten::batch_norm(%a_dequant, %weight, %bias, %mean, %var, %training, %eaf, %eps, %7)
         %r = aten::quantize_per_tensor(%r_bn, %scale, %zero_point, %scalar_type)
         return (%r) )";
  std::string quantized_batch_norm = R"(
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `dequantize`, `batch_norm`, `quantize_per_tensor`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `dequantize`, `batch_norm`, `quantize_per_tensor`。

### Lines 630-640
```cpp
graph(%a_quant, %weight, %bias, %mean, %var, %training, %eaf, %eps, %7, %scale, %zero_point, %scalar_type):
         %r = quantized::batch_norm(%a_quant, %weight, %bias, %mean, %var, %eps, %scale, %zero_point)
         return (%r) )";

  std::string batch_norm_relu = R"(
graph(%a_quant, %weight, %bias, %mean, %var, %training, %eaf, %eps, %7, %scale, %zero_point, %scalar_type):
         %a_dequant = aten::dequantize(%a_quant)
         %bn_out = aten::batch_norm(%a_dequant, %weight, %bias, %mean, %var, %training, %eaf, %eps, %7)
         %relu = aten::relu(%bn_out)
         %r = aten::quantize_per_tensor(%relu, %scale, %zero_point, %scalar_type)
         return (%r) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `batch_norm`, `dequantize`, `relu`, `quantize_per_tensor`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `batch_norm`, `dequantize`, `relu`, `quantize_per_tensor`。

### Lines 641-649
```cpp
  std::string batch_norm_inplace_relu = R"(
graph(%a_quant, %weight, %bias, %mean, %var, %training, %eaf, %eps, %7, %scale, %zero_point, %scalar_type):
         %a_dequant = aten::dequantize(%a_quant)
         %bn_out = aten::batch_norm(%a_dequant, %weight, %bias, %mean, %var, %training, %eaf, %eps, %7)
         %relu = aten::relu_(%bn_out)
         %r = aten::quantize_per_tensor(%relu, %scale, %zero_point, %scalar_type)
         return (%r) )";

  std::string quantized_batch_norm_relu = R"(
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `dequantize`, `batch_norm`, `relu_`, `quantize_per_tensor`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `dequantize`, `batch_norm`, `relu_`, `quantize_per_tensor`。

### Lines 650-661
```cpp
graph(%a_quant, %weight, %bias, %mean, %var, %training, %eaf, %eps, %7, %scale, %zero_point, %scalar_type):
         %r = quantized::batch_norm_relu(%a_quant, %weight, %bias, %mean, %var, %eps, %scale, %zero_point)
         return (%r) )";

  // aten::mul
  std::string mul = R"(
graph(%a_quant, %b_quant, %scale, %zero_point, %dtype):
         %a_dequant = aten::dequantize(%a_quant)
         %b_dequant = aten::dequantize(%b_quant)
         %r_mul = aten::mul(%a_dequant, %b_dequant)
         %r = aten::quantize_per_tensor(%r_mul, %scale, %zero_point, %dtype)
         return (%r) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `batch_norm_relu`, `dequantize`, `mul`, `quantize_per_tensor`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `batch_norm_relu`, `dequantize`, `mul`, `quantize_per_tensor`。

### Lines 662-670
```cpp

  // aten::mul_
  std::string inplace_mul = R"(
graph(%a_quant, %b_quant, %scale, %zero_point, %dtype):
         %a_dequant = aten::dequantize(%a_quant)
         %b_dequant = aten::dequantize(%b_quant)
         %r_mul = aten::mul_(%a_dequant, %b_dequant)
         %r = aten::quantize_per_tensor(%r_mul, %scale, %zero_point, %dtype)
         return (%r) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `dequantize`, `mul_`, `quantize_per_tensor`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `dequantize`, `mul_`, `quantize_per_tensor`。

### Lines 671-684
```cpp

  // quantized::mul
  std::string quantized_mul = R"(
graph(%a_quant, %b_quant, %scale, %zero_point, %dtype):
         %r = quantized::mul(%a_quant, %b_quant, %scale, %zero_point)
         return (%r) )";

  auto mul_scalar = getBinaryOpScalarFusionInfo(
      "aten::mul",
      {"%b_scalar"},
      "quantized::mul_scalar",
      {"%b_scalar"},
      {input_b_is_scalar});

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `mul`, `getBinaryOpScalarFusionInfo`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `mul`, `getBinaryOpScalarFusionInfo`。

### Lines 685-693
```cpp
  auto mul_scalar_out = getBinaryOpScalarFusionInfo(
      "aten::mul_",
      {"%b_scalar"},
      "quantized::mul_scalar_out",
      {"%b_scalar", "%a_quant"},
      {input_b_is_scalar});

  // quantized::mul_relu
  std::string mul_relu = R"(
```
- EN: This block implements local helper logic for quantization patterns. Key symbols: `getBinaryOpScalarFusionInfo`.
- CN: 该代码块实现与 quantization patterns 相关的局部辅助逻辑。关键符号：`getBinaryOpScalarFusionInfo`。

### Lines 694-702
```cpp
graph(%a_quant, %b_quant, %scale, %zero_point, %dtype):
         %a_dequant = aten::dequantize(%a_quant)
         %b_dequant = aten::dequantize(%b_quant)
         %r_mul = aten::mul(%a_dequant, %b_dequant)
         %r_relu = aten::relu(%r_mul)
         %r = aten::quantize_per_tensor(%r_relu, %scale, %zero_point, %dtype)
         return (%r) )";

  std::string mul_inplace_relu = R"(
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `dequantize`, `mul`, `relu`, `quantize_per_tensor`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `dequantize`, `mul`, `relu`, `quantize_per_tensor`。

### Lines 703-711
```cpp
graph(%a_quant, %b_quant, %scale, %zero_point, %dtype):
         %a_dequant = aten::dequantize(%a_quant)
         %b_dequant = aten::dequantize(%b_quant)
         %r_mul = aten::mul(%a_dequant, %b_dequant)
         %r_relu = aten::relu_(%r_mul)
         %r = aten::quantize_per_tensor(%r_relu, %scale, %zero_point, %dtype)
         return (%r) )";

  std::string inplace_mul_relu = R"(
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `dequantize`, `mul`, `relu_`, `quantize_per_tensor`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `dequantize`, `mul`, `relu_`, `quantize_per_tensor`。

### Lines 712-720
```cpp
graph(%a_quant, %b_quant, %scale, %zero_point, %dtype):
         %a_dequant = aten::dequantize(%a_quant)
         %b_dequant = aten::dequantize(%b_quant)
         %r_mul = aten::mul_(%a_dequant, %b_dequant)
         %r_relu = aten::relu(%r_mul)
         %r = aten::quantize_per_tensor(%r_relu, %scale, %zero_point, %dtype)
         return (%r) )";

  std::string inplace_mul_inplace_relu = R"(
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `dequantize`, `mul_`, `relu`, `quantize_per_tensor`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `dequantize`, `mul_`, `relu`, `quantize_per_tensor`。

### Lines 721-729
```cpp
graph(%a_quant, %b_quant, %scale, %zero_point, %dtype):
         %a_dequant = aten::dequantize(%a_quant)
         %b_dequant = aten::dequantize(%b_quant)
         %r_mul = aten::mul_(%a_dequant, %b_dequant)
         %r_relu = aten::relu_(%r_mul)
         %r = aten::quantize_per_tensor(%r_relu, %scale, %zero_point, %dtype)
         return (%r) )";

  std::string quantized_mul_relu = R"(
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `dequantize`, `mul_`, `relu_`, `quantize_per_tensor`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `dequantize`, `mul_`, `relu_`, `quantize_per_tensor`。

### Lines 730-740
```cpp
graph(%a_quant, %b_quant, %scale, %zero_point, %dtype):
         %r = quantized::mul_relu(%a_quant, %b_quant, %scale, %zero_point)
         return (%r) )";

  // quantized::mul_scalar_relu -- fusing quantized::mul_scalar
  // and aten::relu
  auto quantized_mul_scalar_relu_pattern = R"(
graph(%a_quant, %b_scalar):
         %r_mul = quantized::mul_scalar(%a_quant, %b_scalar)
         %r = aten::relu(%r_mul)
         return (%r) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `mul_relu`, `mul_scalar`, `relu`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `mul_relu`, `mul_scalar`, `relu`。

### Lines 741-751
```cpp

  auto quantized_mul_scalar_inplace_relu_pattern = R"(
graph(%a_quant, %b_scalar):
         %r_mul = quantized::mul_scalar(%a_quant, %b_scalar)
         %r = aten::relu_(%r_mul)
         return (%r) )";

  auto quantized_mul_scalar_relu_replacement = R"(
graph(%a_quant, %b_scalar):
         %r = quantized::mul_scalar_relu(%a_quant, %b_scalar)
         return (%r) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `mul_scalar`, `relu_`, `mul_scalar_relu`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `mul_scalar`, `relu_`, `mul_scalar_relu`。

### Lines 752-760
```cpp

  // quantized::mul_scalar_relu_out -- fusing quantized::mul_scalarOut
  // and aten::relu
  auto quantized_mul_scalar_relu_out_pattern = R"(
graph(%a_quant, %b_scalar):
         %r_mul = quantized::mul_scalar_out(%a_quant, %b_scalar, %a_quant)
         %r = aten::relu(%r_mul)
         return (%r) )";

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `mul_scalar_out`, `relu`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `mul_scalar_out`, `relu`。

### Lines 761-770
```cpp
  auto quantized_mul_scalar_inplace_relu_out_pattern = R"(
graph(%a_quant, %b_scalar):
         %r_mul = quantized::mul_scalar_out(%a_quant, %b_scalar, %a_quant)
         %r = aten::relu_(%r_mul)
         return (%r) )";

  auto quantized_mul_scalar_relu_out_replacement = R"(
graph(%a_quant, %b_scalar):
         %r = quantized::mul_scalar_relu_out(%a_quant, %b_scalar, %a_quant)
         return (%r) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `mul_scalar_out`, `relu_`, `mul_scalar_relu_out`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `mul_scalar_out`, `relu_`, `mul_scalar_relu_out`。

### Lines 771-779
```cpp

  // quantized::elu
  std::string elu = R"(
graph(%a_quant, %alpha, %scale, %input_scale, %r_scale, %r_zero_point, %r_dtype):
         %a_dequant = aten::dequantize(%a_quant)
         %r = aten::elu(%a_dequant, %alpha, %scale, %input_scale)
         %r_quant = aten::quantize_per_tensor(%r, %r_scale, %r_zero_point, %r_dtype)
         return (%r_quant) )";

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `dequantize`, `elu`, `quantize_per_tensor`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `dequantize`, `elu`, `quantize_per_tensor`。

### Lines 780-790
```cpp
  std::string quantized_elu = R"(
graph(%a_quant, %alpha, %scale, %input_scale, %r_scale, %r_zero_point, %r_dtype):
         %r_quant = quantized::elu(%a_quant, %r_scale, %r_zero_point, %alpha, %scale, %input_scale)
         return (%r_quant) )";

  std::string elu_ = R"(
graph(%a_quant, %alpha, %scale, %input_scale, %r_scale, %r_zero_point, %r_dtype):
         %a_dequant = aten::dequantize(%a_quant)
         %r = aten::elu_(%a_dequant, %alpha, %scale, %input_scale)
         %r_quant = aten::quantize_per_tensor(%r, %r_scale, %r_zero_point, %r_dtype)
         return (%r_quant) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `elu`, `dequantize`, `elu_`, `quantize_per_tensor`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `elu`, `dequantize`, `elu_`, `quantize_per_tensor`。

### Lines 791-801
```cpp

  // ============= General Ops that inherit quantization parameters from input
  // tensor =============
  auto avg_pool1d = getInputTensorQParamOpFusionInfo(
      "aten::avg_pool1d",
      {"%kernel_size",
       "%stride",
       "%padding",
       "%ceil_mode",
       "%count_include_pad"});

```
- EN: This block implements local helper logic for quantization patterns. Key symbols: `getInputTensorQParamOpFusionInfo`.
- CN: 该代码块实现与 quantization patterns 相关的局部辅助逻辑。关键符号：`getInputTensorQParamOpFusionInfo`。

### Lines 802-810
```cpp
  auto avg_pool2d = getInputTensorQParamOpFusionInfo(
      "aten::avg_pool2d",
      {"%kernel_size",
       "%stride",
       "%padding",
       "%ceil_mode",
       "%count_include_pad",
       "%divisor_override"});

```
- EN: This block implements local helper logic for quantization patterns. Key symbols: `getInputTensorQParamOpFusionInfo`.
- CN: 该代码块实现与 quantization patterns 相关的局部辅助逻辑。关键符号：`getInputTensorQParamOpFusionInfo`。

### Lines 811-819
```cpp
  auto avg_pool3d = getInputTensorQParamOpFusionInfo(
      "aten::avg_pool3d",
      {"%kernel_size",
       "%stride",
       "%padding",
       "%ceil_mode",
       "%count_include_pad",
       "%divisor_override"});

```
- EN: This block implements local helper logic for quantization patterns. Key symbols: `getInputTensorQParamOpFusionInfo`.
- CN: 该代码块实现与 quantization patterns 相关的局部辅助逻辑。关键符号：`getInputTensorQParamOpFusionInfo`。

### Lines 820-828
```cpp
  auto adaptive_avg_pool1d = getInputTensorQParamOpFusionInfo(
      "aten::adaptive_avg_pool1d", {"%output_size"});

  auto adaptive_avg_pool2d = getInputTensorQParamOpFusionInfo(
      "aten::adaptive_avg_pool2d", {"%output_size"});

  auto adaptive_avg_pool3d = getInputTensorQParamOpFusionInfo(
      "aten::adaptive_avg_pool3d", {"%output_size"});

```
- EN: This block implements local helper logic for quantization patterns. Key symbols: `getInputTensorQParamOpFusionInfo`.
- CN: 该代码块实现与 quantization patterns 相关的局部辅助逻辑。关键符号：`getInputTensorQParamOpFusionInfo`。

### Lines 829-837
```cpp
  auto mean1 = getInputTensorQParamOpFusionInfo("aten::mean", {"%dim"});

  auto mean2 = getInputTensorQParamOpFusionInfo(
      "aten::mean", {"%dim", "%keepdim", "%out"});

  auto upsample_nearest1d_vec = getInputTensorQParamOpFusionInfo(
      "aten::upsample_nearest1d", {"%output_size", "%scale_factors"});

  auto upsample_nearest2d_vec = getInputTensorQParamOpFusionInfo(
```
- EN: This block implements local helper logic for quantization patterns. Key symbols: `getInputTensorQParamOpFusionInfo`.
- CN: 该代码块实现与 quantization patterns 相关的局部辅助逻辑。关键符号：`getInputTensorQParamOpFusionInfo`。

### Lines 838-846
```cpp
      "aten::upsample_nearest2d", {"%output_size", "%scale_factors"});

  auto upsample_nearest3d_vec = getInputTensorQParamOpFusionInfo(
      "aten::upsample_nearest3d", {"%output_size", "%scale_factors"});

  auto upsample_linear1d_vec = getInputTensorQParamOpFusionInfo(
      "aten::upsample_linear1d",
      {"%output_size", "%align_corners", "%scale_factors"});

```
- EN: This block implements local helper logic for quantization patterns. Key symbols: `getInputTensorQParamOpFusionInfo`.
- CN: 该代码块实现与 quantization patterns 相关的局部辅助逻辑。关键符号：`getInputTensorQParamOpFusionInfo`。

### Lines 847-855
```cpp
  auto upsample_bilinear2d_vec = getInputTensorQParamOpFusionInfo(
      "aten::upsample_bilinear2d",
      {"%output_size", "%align_corners", "%scale_factors"});

  auto upsample_trilinear3d_vec = getInputTensorQParamOpFusionInfo(
      "aten::upsample_trilinear3d",
      {"%output_size", "%align_corners", "%scale_factors"});

  auto upsample_nearest1d = getInputTensorQParamOpFusionInfo(
```
- EN: This block implements local helper logic for quantization patterns. Key symbols: `getInputTensorQParamOpFusionInfo`.
- CN: 该代码块实现与 quantization patterns 相关的局部辅助逻辑。关键符号：`getInputTensorQParamOpFusionInfo`。

### Lines 856-864
```cpp
      "aten::upsample_nearest1d", {"%output_size", "%scales"});

  auto upsample_nearest2d = getInputTensorQParamOpFusionInfo(
      "aten::upsample_nearest2d", {"%output_size", "%scale_h", "%scale_w"});

  auto upsample_nearest3d = getInputTensorQParamOpFusionInfo(
      "aten::upsample_nearest3d",
      {"%output_size", "%scale_d", "%scale_h", "%scale_w"});

```
- EN: This block implements local helper logic for quantization patterns. Key symbols: `getInputTensorQParamOpFusionInfo`.
- CN: 该代码块实现与 quantization patterns 相关的局部辅助逻辑。关键符号：`getInputTensorQParamOpFusionInfo`。

### Lines 865-875
```cpp
  auto upsample_linear1d = getInputTensorQParamOpFusionInfo(
      "aten::upsample_linear1d", {"%output_size", "%align_corners", "%scales"});

  auto upsample_bilinear2d = getInputTensorQParamOpFusionInfo(
      "aten::upsample_bilinear2d",
      {"%output_size", "%align_corners", "%scale_h", "%scale_w"});

  auto upsample_trilinear3d = getInputTensorQParamOpFusionInfo(
      "aten::upsample_trilinear3d",
      {"%output_size", "%align_corners", "%scale_d", "%scale_h", "%scale_w"});

```
- EN: This block implements local helper logic for quantization patterns. Key symbols: `getInputTensorQParamOpFusionInfo`.
- CN: 该代码块实现与 quantization patterns 相关的局部辅助逻辑。关键符号：`getInputTensorQParamOpFusionInfo`。

### Lines 876-884
```cpp
  auto clamp = getClampOpFusionInfo("aten::clamp", {"%min", "%max"});

  auto hardtanh = getClampOpFusionInfo("aten::hardtanh", {"%min", "%max"});

  auto hardtanh_ = getClampOpFusionInfo("aten::hardtanh_", {"%min", "%max"});

  auto leaky_relu =
      getInputTensorQParamOpFusionInfo("aten::leaky_relu", {"%negative_slope"});

```
- EN: This block implements local helper logic for quantization patterns. Key symbols: `getClampOpFusionInfo`, `getInputTensorQParamOpFusionInfo`.
- CN: 该代码块实现与 quantization patterns 相关的局部辅助逻辑。关键符号：`getClampOpFusionInfo`, `getInputTensorQParamOpFusionInfo`。

### Lines 885-893
```cpp
  auto leaky_relu_ = getInputTensorQParamOpFusionInfo(
      "aten::leaky_relu_", {"%negative_slope"});

  // Ops with fixed quantization parameters
  auto hardsigmoid = getFixedQParamOpFusionInfo("aten::hardsigmoid", {}, false);

  auto hardsigmoid_ =
      getFixedQParamOpFusionInfo("aten::hardsigmoid_", {}, false);

```
- EN: This block implements local helper logic for quantization patterns. Key symbols: `getInputTensorQParamOpFusionInfo`, `getFixedQParamOpFusionInfo`.
- CN: 该代码块实现与 quantization patterns 相关的局部辅助逻辑。关键符号：`getInputTensorQParamOpFusionInfo`, `getFixedQParamOpFusionInfo`。

### Lines 894-902
```cpp
  auto sigmoid = getFixedQParamOpFusionInfo("aten::sigmoid", {}, false);

  auto sigmoid_ = getFixedQParamOpFusionInfo("aten::sigmoid_", {}, false);

  auto tanh = getFixedQParamOpFusionInfo("aten::tanh", {}, true);

  auto tanh_ = getFixedQParamOpFusionInfo("aten::tanh_", {}, true);

  auto hardswish = getObservedQParamOpFusionInfo(
```
- EN: This block implements local helper logic for quantization patterns. Key symbols: `getFixedQParamOpFusionInfo`, `getObservedQParamOpFusionInfo`.
- CN: 该代码块实现与 quantization patterns 相关的局部辅助逻辑。关键符号：`getFixedQParamOpFusionInfo`, `getObservedQParamOpFusionInfo`。

### Lines 903-913
```cpp
      "aten::hardswish", "quantized::hardswish", {}, {});

  auto hardswish_ = getObservedQParamOpFusionInfo(
      "aten::hardswish_", "quantized::hardswish", {}, {});

  auto layer_norm = getObservedQParamOpFusionInfo(
      "aten::layer_norm",
      "quantized::layer_norm",
      {"%normalized_shape", "%weight", "%bias", "%eps", "%cudnn_enabled"},
      {"%normalized_shape", "%weight", "%bias", "%eps"});

```
- EN: This block implements local helper logic for quantization patterns. Key symbols: `getObservedQParamOpFusionInfo`.
- CN: 该代码块实现与 quantization patterns 相关的局部辅助逻辑。关键符号：`getObservedQParamOpFusionInfo`。

### Lines 914-931
```cpp
  auto group_norm = getObservedQParamOpFusionInfo(
      "aten::group_norm",
      "quantized::group_norm",
      {"%num_groups", "%weight", "%bias", "%eps", "%cudnn_enabled"},
      {"%num_groups", "%weight", "%bias", "%eps"});

  auto instance_norm = getObservedQParamOpFusionInfo(
      "aten::instance_norm",
      "quantized::instance_norm",
      {"%weight",
       "%bias",
       "%running_mean",
       "%running_var",
       "%use_input_stats",
       "%momentum",
       "%eps",
       "%cudnn_enabled"},
      {"%weight", "%bias", "%eps"});
```
- EN: This block implements local helper logic for quantization patterns. Key symbols: `getObservedQParamOpFusionInfo`.
- CN: 该代码块实现与 quantization patterns 相关的局部辅助逻辑。关键符号：`getObservedQParamOpFusionInfo`。

### Lines 932-942
```cpp

  return {
      {"quantized::conv1d", std::move(conv1d), std::move(quantized_conv1d)},
      {"quantized::conv1d_relu", std::move(conv1d_relu), quantized_conv1d_relu},
      {"quantized::conv1d_relu",
       std::move(conv1d_inplace_relu),
       std::move(quantized_conv1d_relu)},
      {"quantized::conv2d", std::move(conv2d), std::move(quantized_conv2d)},
      {"quantized::conv2d_relu", std::move(conv2d_relu), quantized_conv2d_relu},
      {"quantized::conv2d_relu",
       std::move(conv2d_inplace_relu),
```
- EN: This block produces a result or forwards a computed value. Key symbols: `move`.
- CN: 该代码块返回结果或转发已计算的值。关键符号：`move`。

### Lines 943-951
```cpp
       std::move(quantized_conv2d_relu)},
      {"quantized::conv3d", std::move(conv3d), std::move(quantized_conv3d)},
      {"quantized::conv3d_relu", std::move(conv3d_relu), quantized_conv3d_relu},
      {"quantized::conv3d_relu",
       std::move(conv3d_inplace_relu),
       std::move(quantized_conv3d_relu)},
      {"quantized::conv_transpose1d",
       std::move(conv_transpose1d),
       std::move(quantized_conv_transpose1d)},
```
- EN: This block implements local helper logic for quantization patterns. Key symbols: `move`.
- CN: 该代码块实现与 quantization patterns 相关的局部辅助逻辑。关键符号：`move`。

### Lines 952-961
```cpp
      {"quantized::conv_transpose2d",
       std::move(conv_transpose2d),
       std::move(quantized_conv_transpose2d)},
      {"quantized::linear", std::move(linear), std::move(quantized_linear)},
      {"quantized::linear_relu", std::move(linear_relu), quantized_linear_relu},
      {"quantized::linear_relu",
       std::move(linear_inplace_relu),
       std::move(quantized_linear_relu)},
      {"quantized::add_relu",
       std::move(add_relu),
```
- EN: This block implements local helper logic for quantization patterns. Key symbols: `move`.
- CN: 该代码块实现与 quantization patterns 相关的局部辅助逻辑。关键符号：`move`。

### Lines 962-973
```cpp
       quantized_add_relu,
       {aten_add_alpha_is_one}},
      {"quantized::add_relu",
       std::move(add_inplace_relu),
       quantized_add_relu,
       {aten_add_alpha_is_one}},
      {"quantized::add_relu",
       std::move(inplace_add_relu),
       quantized_add_relu,
       {aten_add_alpha_is_one}},
      {"quantized::add_relu",
       std::move(inplace_add_inplace_relu),
```
- EN: This block implements local helper logic for quantization patterns. Key symbols: `move`.
- CN: 该代码块实现与 quantization patterns 相关的局部辅助逻辑。关键符号：`move`。

### Lines 974-991
```cpp
       std::move(quantized_add_relu),
       {aten_add_alpha_is_one}},
      std::move(add_scalar),
      std::move(add_scalar_out),
      // note that these must come after quantized::add_scalar and
      // quantized::add_scalar_out patterns
      {"quantized::add_scalar_relu",
       quantized_add_scalar_relu_pattern,
       quantized_add_scalar_relu_replacement},
      {"quantized::add_scalar_relu",
       quantized_add_scalar_inplace_relu_pattern,
       quantized_add_scalar_relu_replacement},
      {"quantized::add_scalar_relu_out",
       quantized_add_scalar_relu_out_pattern,
       quantized_add_scalar_relu_out_replacement},
      {"quantized::add_scalar_relu_out",
       quantized_add_scalar_inplace_relu_out_pattern,
       quantized_add_scalar_relu_out_replacement},
```
- EN: This block implements local helper logic for quantization patterns. Key symbols: `move`.
- CN: 该代码块实现与 quantization patterns 相关的局部辅助逻辑。关键符号：`move`。

### Lines 992-1002
```cpp
      {"quantized::add",
       std::move(add),
       quantized_add,
       {aten_add_alpha_is_one}},
      {"quantized::add",
       std::move(inplace_add),
       std::move(quantized_add),
       {aten_add_alpha_is_one}},
      {"quantized::cat", std::move(cat), std::move(quantized_cat)},
      {"quantized::batch_norm",
       std::move(batch_norm),
```
- EN: This block implements local helper logic for quantization patterns. Key symbols: `move`.
- CN: 该代码块实现与 quantization patterns 相关的局部辅助逻辑。关键符号：`move`。

### Lines 1003-1011
```cpp
       std::move(quantized_batch_norm)},
      {"quantized::batch_norm_relu",
       std::move(batch_norm_relu),
       quantized_batch_norm_relu},
      {"quantized::batch_norm_relu",
       std::move(batch_norm_inplace_relu),
       std::move(quantized_batch_norm_relu)},
      std::move(mul_scalar),
      std::move(mul_scalar_out),
```
- EN: This block implements local helper logic for quantization patterns. Key symbols: `move`.
- CN: 该代码块实现与 quantization patterns 相关的局部辅助逻辑。关键符号：`move`。

### Lines 1012-1029
```cpp
      // note that these must come after quantized::mul_scalar and
      // quantized::mul_scalar_out patterns
      {"quantized::mul_scalar_relu",
       quantized_mul_scalar_relu_pattern,
       quantized_mul_scalar_relu_replacement},
      {"quantized::mul_scalar_relu",
       quantized_mul_scalar_inplace_relu_pattern,
       quantized_mul_scalar_relu_replacement},
      {"quantized::mul_scalar_relu_out",
       quantized_mul_scalar_relu_out_pattern,
       quantized_mul_scalar_relu_out_replacement},
      {"quantized::mul_scalar_relu_out",
       quantized_mul_scalar_inplace_relu_out_pattern,
       quantized_mul_scalar_relu_out_replacement},
      {"quantized::mul_relu", std::move(mul_relu), quantized_mul_relu},
      {"quantized::mul_relu", std::move(mul_inplace_relu), quantized_mul_relu},
      {"quantized::mul_relu", std::move(inplace_mul_relu), quantized_mul_relu},
      {"quantized::mul_relu",
```
- EN: This block implements local helper logic for quantization patterns. Key symbols: `move`.
- CN: 该代码块实现与 quantization patterns 相关的局部辅助逻辑。关键符号：`move`。

### Lines 1030-1038
```cpp
       std::move(inplace_mul_inplace_relu),
       std::move(quantized_mul_relu)},
      {"quantized::mul", std::move(mul), quantized_mul},
      {"quantized::mul", std::move(inplace_mul), std::move(quantized_mul)},
      std::move(hardswish),
      std::move(hardswish_),
      std::move(layer_norm),
      std::move(group_norm),
      std::move(instance_norm),
```
- EN: This block implements local helper logic for quantization patterns. Key symbols: `move`.
- CN: 该代码块实现与 quantization patterns 相关的局部辅助逻辑。关键符号：`move`。

### Lines 1039-1047
```cpp
      {"quantized::elu", std::move(elu), quantized_elu},
      {"quantized::elu_", std::move(elu_), std::move(quantized_elu)},
      std::move(avg_pool1d),
      std::move(avg_pool2d),
      std::move(avg_pool3d),
      std::move(adaptive_avg_pool1d),
      std::move(adaptive_avg_pool2d),
      std::move(adaptive_avg_pool3d),
      std::move(mean1),
```
- EN: This block implements local helper logic for quantization patterns. Key symbols: `move`.
- CN: 该代码块实现与 quantization patterns 相关的局部辅助逻辑。关键符号：`move`。

### Lines 1048-1056
```cpp
      std::move(mean2),
      std::move(upsample_nearest1d),
      std::move(upsample_nearest2d),
      std::move(upsample_nearest3d),
      std::move(upsample_linear1d),
      std::move(upsample_bilinear2d),
      std::move(upsample_trilinear3d),
      std::move(upsample_nearest1d_vec),
      std::move(upsample_nearest2d_vec),
```
- EN: This block implements local helper logic for quantization patterns. Key symbols: `move`.
- CN: 该代码块实现与 quantization patterns 相关的局部辅助逻辑。关键符号：`move`。

### Lines 1057-1065
```cpp
      std::move(upsample_nearest3d_vec),
      std::move(upsample_linear1d_vec),
      std::move(upsample_bilinear2d_vec),
      std::move(upsample_trilinear3d_vec),
      std::move(clamp),
      std::move(hardtanh),
      std::move(hardtanh_),
      std::move(leaky_relu),
      std::move(leaky_relu_),
```
- EN: This block implements local helper logic for quantization patterns. Key symbols: `move`.
- CN: 该代码块实现与 quantization patterns 相关的局部辅助逻辑。关键符号：`move`。

### Lines 1066-1075
```cpp
      // fixed qparam ops
      std::move(hardsigmoid),
      std::move(hardsigmoid_),
      std::move(sigmoid),
      std::move(sigmoid_),
      std::move(tanh),
      std::move(tanh_),
  };
}

```
- EN: This block implements local helper logic for quantization patterns. Key symbols: `move`.
- CN: 该代码块实现与 quantization patterns 相关的局部辅助逻辑。关键符号：`move`。

### Lines 1076-1084
```cpp
inline std::vector<QuantFusionInfo>
dynamic_quantized_linear_pattern_and_replacements() {
  std::string linear_dynamic = R"(
graph(%packed_params, %a):
        %w_quant : Tensor, %b : Tensor? = quantized::linear_unpack(%packed_params)
        %w_dequant = aten::dequantize(%w_quant)
        %r = aten::linear(%a, %w_dequant, %b)
        return (%r) )";

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `dynamic_quantized_linear_pattern_and_replacements`, `graph`, `linear_unpack`, `dequantize`, `linear`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`dynamic_quantized_linear_pattern_and_replacements`, `graph`, `linear_unpack`, `dequantize`, `linear`。

### Lines 1085-1093
```cpp
  // This pattern ignores reduce range
  // Set the reduce range to default to true, since qnnpack backend ignores this
  // argument.
  std::string quantized_linear_dynamic = R"(
graph(%packed_params, %a):
        %reduce_range : bool = prim::Constant[value=1]()
        %r = quantized::linear_dynamic(%a, %packed_params, %reduce_range)
        return (%r) )";

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `graph`, `linear_dynamic`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`graph`, `linear_dynamic`。

### Lines 1094-1103
```cpp
  return {
      {"quantized::linear_dynamic",
       std::move(linear_dynamic),
       std::move(quantized_linear_dynamic)},
  };
}

static std::vector<QuantFusionInfo>
dynamic_quant_fusion_pattern_and_replacements() {
  std::string linear_dynamic = R"(
```
- EN: This block produces a result or forwards a computed value. Key symbols: `move`, `dynamic_quant_fusion_pattern_and_replacements`.
- CN: 该代码块返回结果或转发已计算的值。关键符号：`move`, `dynamic_quant_fusion_pattern_and_replacements`。

### Lines 1104-1112
```cpp
graph(%packed_params, %a, %reduce_range, %a_dtype):
        %a_scale : float, %a_zero_point : int = aten::_choose_qparams_per_tensor(%a, %reduce_range)
        %a_quant = aten::quantize_per_tensor(%a, %a_scale, %a_zero_point, %a_dtype)
        %a_dequant = aten::dequantize(%a_quant)
        %w_quant : Tensor, %b : Tensor? = quantized::linear_unpack(%packed_params)
        %w_dequant = aten::dequantize(%w_quant)
        %r = aten::linear(%a_dequant, %w_dequant, %b)
        return (%r) )";

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `_choose_qparams_per_tensor`, `quantize_per_tensor`, `dequantize`, `linear_unpack`, `linear`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `_choose_qparams_per_tensor`, `quantize_per_tensor`, `dequantize`, `linear_unpack`, `linear`。

### Lines 1113-1122
```cpp
  std::string quantized_linear_dynamic = R"(
graph(%packed_params, %a, %reduce_range, %a_dtype):
        %r = quantized::linear_dynamic(%a, %packed_params, %reduce_range)
        return (%r) )";

  std::string linear_dynamic_fp16 = R"(
graph(%packed_params, %a):
        %w_unpacked : Tensor, %b : Tensor? = quantized::linear_unpack_fp16(%packed_params)
        %r = aten::linear(%a, %w_unpacked, %b)
        return (%r) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `linear_dynamic`, `linear_unpack_fp16`, `linear`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `linear_dynamic`, `linear_unpack_fp16`, `linear`。

### Lines 1123-1131
```cpp

  std::string quantized_linear_dynamic_fp16 = R"(
graph(%packed_params, %a):
        %r = quantized::linear_dynamic_fp16(%a, %packed_params)
        return (%r) )";

  return {
      {"quantized::linear_dynamic",
       std::move(linear_dynamic),
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `linear_dynamic_fp16`, `move`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `linear_dynamic_fp16`, `move`。

### Lines 1132-1140
```cpp
       std::move(quantized_linear_dynamic)},
      {"quantized::linear_dynamic_fp16",
       std::move(linear_dynamic_fp16),
       std::move(quantized_linear_dynamic_fp16)},
  };
}

static std::vector<QuantFusionInfo> linear_prepack_unpack_patterns() {
  std::string linear_with_quant = R"(
```
- EN: This block implements local helper logic for quantization patterns. Key symbols: `move`, `linear_prepack_unpack_patterns`.
- CN: 该代码块实现与 quantization patterns 相关的局部辅助逻辑。关键符号：`move`, `linear_prepack_unpack_patterns`。

### Lines 1141-1152
```cpp
graph(%a_dequant, %w_quant, %b):
        %w_dequant = aten::dequantize(%w_quant)
        %r = aten::linear(%a_dequant, %w_dequant, %b)
        return (%r) )";

  std::string linear_with_quant_prepack = R"(
graph(%a_dequant, %w_quant, %b):
        %packed_params = quantized::linear_prepack(%w_quant, %b)
        %w_quant_unpacked : Tensor, %b_unpacked : Tensor? = quantized::linear_unpack(%packed_params)
        %w_dequant = aten::dequantize(%w_quant_unpacked)
        %r = aten::linear(%a_dequant, %w_dequant, %b_unpacked)
        return (%r) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `dequantize`, `linear`, `linear_prepack`, `linear_unpack`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `dequantize`, `linear`, `linear_prepack`, `linear_unpack`。

### Lines 1153-1163
```cpp
  std::string linear_fp16_with_cast = R"(
graph(%w, %a_dq, %b):
        %fp16_tensor = aten::_saturate_weight_to_fp16(%w)
        %r = aten::linear(%a_dq, %fp16_tensor, %b)
        return (%r) )";
  std::string linear_fp16_with_prepack = R"(
graph(%w, %a_dq, %b):
        %packed_params = quantized::linear_prepack_fp16(%w, %b)
        %w_unpacked : Tensor, %b_unpacked : Tensor? = quantized::linear_unpack_fp16(%packed_params)
        %r = aten::linear(%a_dq, %w_unpacked, %b_unpacked)
        return (%r) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `_saturate_weight_to_fp16`, `linear`, `linear_prepack_fp16`, `linear_unpack_fp16`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `_saturate_weight_to_fp16`, `linear`, `linear_prepack_fp16`, `linear_unpack_fp16`。

### Lines 1164-1172
```cpp

  return {
      {"linear_prepack_unpack",
       std::move(linear_with_quant),
       std::move(linear_with_quant_prepack)},
      {"linear_fp16_prepack_unpack",
       std::move(linear_fp16_with_cast),
       std::move(linear_fp16_with_prepack)},
  };
```
- EN: This block produces a result or forwards a computed value. Key symbols: `move`.
- CN: 该代码块返回结果或转发已计算的值。关键符号：`move`。

### Lines 1173-1181
```cpp
}

static std::vector<QuantFusionInfo> conv_prepack_unpack_patterns() {
  std::string conv1d_with_quant = R"(
graph(%a_dequant, %w_quant, %b, %stride, %padding, %dilation, %groups):
        %w_dequant = aten::dequantize(%w_quant)
        %r = aten::conv1d(%a_dequant, %w_dequant, %b, %stride, %padding, %dilation, %groups)
        return (%r) )";

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `conv_prepack_unpack_patterns`, `graph`, `dequantize`, `conv1d`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`conv_prepack_unpack_patterns`, `graph`, `dequantize`, `conv1d`。

### Lines 1182-1190
```cpp
  std::string conv1d_with_quant_prepack = R"(
graph(%a_dequant, %w_quant, %b, %stride, %padding, %dilation, %groups):
        %packed_params : __torch__.torch.classes.quantized.Conv2dPackedParamsBase = quantized::conv1d_prepack(%w_quant, %b, %stride, %padding, %dilation, %groups)
        %w_quant_unpacked : Tensor, %b_unpacked : Tensor? = quantized::conv1d_unpack(%packed_params)
        %w_dequant = aten::dequantize(%w_quant_unpacked)
        %r = aten::conv1d(%a_dequant, %w_dequant, %b_unpacked, %stride, %padding, %dilation, %groups)
        return (%r) )";

  std::string conv2d_with_quant = R"(
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `conv1d_prepack`, `conv1d_unpack`, `dequantize`, `conv1d`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `conv1d_prepack`, `conv1d_unpack`, `dequantize`, `conv1d`。

### Lines 1191-1202
```cpp
graph(%a_dequant, %w_quant, %b, %stride, %padding, %dilation, %groups):
        %w_dequant = aten::dequantize(%w_quant)
        %r = aten::conv2d(%a_dequant, %w_dequant, %b, %stride, %padding, %dilation, %groups)
        return (%r) )";

  std::string conv2d_with_quant_prepack = R"(
graph(%a_dequant, %w_quant, %b, %stride, %padding, %dilation, %groups):
        %packed_params : __torch__.torch.classes.quantized.Conv2dPackedParamsBase = quantized::conv2d_prepack(%w_quant, %b, %stride, %padding, %dilation, %groups)
        %w_quant_unpacked : Tensor, %b_unpacked : Tensor? = quantized::conv2d_unpack(%packed_params)
        %w_dequant = aten::dequantize(%w_quant_unpacked)
        %r = aten::conv2d(%a_dequant, %w_dequant, %b_unpacked, %stride, %padding, %dilation, %groups)
        return (%r) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `dequantize`, `conv2d`, `conv2d_prepack`, `conv2d_unpack`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `dequantize`, `conv2d`, `conv2d_prepack`, `conv2d_unpack`。

### Lines 1203-1216
```cpp

  std::string conv3d_with_quant = R"(
graph(%a_dequant, %w_quant, %b, %stride, %padding, %dilation, %groups):
        %w_dequant = aten::dequantize(%w_quant)
        %r = aten::conv3d(%a_dequant, %w_dequant, %b, %stride, %padding, %dilation, %groups)
        return (%r) )";

  std::string conv3d_with_quant_prepack = R"(
graph(%a_dequant, %w_quant, %b, %stride, %padding, %dilation, %groups):
        %packed_params : __torch__.torch.classes.quantized.Conv3dPackedParamsBase = quantized::conv3d_prepack(%w_quant, %b, %stride, %padding, %dilation, %groups)
        %w_quant_unpacked : Tensor, %b_unpacked : Tensor? = quantized::conv3d_unpack(%packed_params)
        %w_dequant = aten::dequantize(%w_quant_unpacked)
        %r = aten::conv3d(%a_dequant, %w_dequant, %b_unpacked, %stride, %padding, %dilation, %groups)
        return (%r) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `dequantize`, `conv3d`, `conv3d_prepack`, `conv3d_unpack`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `dequantize`, `conv3d`, `conv3d_prepack`, `conv3d_unpack`。

### Lines 1217-1230
```cpp

  std::string conv_transpose1d_with_quant = R"(
graph(%a_dequant, %w_quant, %b, %stride, %padding, %output_padding, %groups, %dilation):
        %w_dequant = aten::dequantize(%w_quant)
        %r = aten::conv_transpose1d(%a_dequant, %w_dequant, %b, %stride, %padding, %output_padding, %groups, %dilation)
        return (%r) )";

  std::string conv_transpose1d_with_quant_prepack = R"(
graph(%a_dequant, %w_quant, %b, %stride, %padding, %output_padding, %groups, %dilation):
        %packed_params : __torch__.torch.classes.quantized.Conv2dPackedParamsBase = quantized::conv_transpose1d_prepack(%w_quant, %b, %stride, %padding, %output_padding, %dilation, %groups)
        %w_quant_unpacked : Tensor, %b_unpacked : Tensor? = quantized::conv_transpose1d_unpack(%packed_params)
        %w_dequant = aten::dequantize(%w_quant_unpacked)
        %r = aten::conv_transpose1d(%a_dequant, %w_dequant, %b_unpacked, %stride, %padding, %output_padding, %groups, %dilation)
        return (%r) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `dequantize`, `conv_transpose1d`, `conv_transpose1d_prepack`, `conv_transpose1d_unpack`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `dequantize`, `conv_transpose1d`, `conv_transpose1d_prepack`, `conv_transpose1d_unpack`。

### Lines 1231-1244
```cpp

  std::string conv_transpose2d_with_quant = R"(
graph(%a_dequant, %w_quant, %b, %stride, %padding, %output_padding, %groups, %dilation):
        %w_dequant = aten::dequantize(%w_quant)
        %r = aten::conv_transpose2d(%a_dequant, %w_dequant, %b, %stride, %padding, %output_padding, %groups, %dilation)
        return (%r) )";

  std::string conv_transpose2d_with_quant_prepack = R"(
graph(%a_dequant, %w_quant, %b, %stride, %padding, %output_padding, %groups, %dilation):
        %packed_params : __torch__.torch.classes.quantized.Conv2dPackedParamsBase = quantized::conv_transpose2d_prepack(%w_quant, %b, %stride, %padding, %output_padding, %dilation, %groups)
        %w_quant_unpacked : Tensor, %b_unpacked : Tensor? = quantized::conv_transpose2d_unpack(%packed_params)
        %w_dequant = aten::dequantize(%w_quant_unpacked)
        %r = aten::conv_transpose2d(%a_dequant, %w_dequant, %b_unpacked, %stride, %padding, %output_padding, %groups, %dilation)
        return (%r) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `dequantize`, `conv_transpose2d`, `conv_transpose2d_prepack`, `conv_transpose2d_unpack`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `dequantize`, `conv_transpose2d`, `conv_transpose2d_prepack`, `conv_transpose2d_unpack`。

### Lines 1245-1254
```cpp

  return {
      {"conv1d_prepack_unpack",
       std::move(conv1d_with_quant),
       std::move(conv1d_with_quant_prepack)},
      {"conv2d_prepack_unpack",
       std::move(conv2d_with_quant),
       std::move(conv2d_with_quant_prepack)},
      {"conv3d_prepack_unpack",
       std::move(conv3d_with_quant),
```
- EN: This block produces a result or forwards a computed value. Key symbols: `move`.
- CN: 该代码块返回结果或转发已计算的值。关键符号：`move`。

### Lines 1255-1263
```cpp
       std::move(conv3d_with_quant_prepack)},
      {"conv_transpose1d_prepack_unpack",
       std::move(conv_transpose1d_with_quant),
       std::move(conv_transpose1d_with_quant_prepack)},
      {"conv_transpose2d_prepack_unpack",
       std::move(conv_transpose2d_with_quant),
       std::move(conv_transpose2d_with_quant_prepack)}};
}

```
- EN: This block implements local helper logic for quantization patterns. Key symbols: `move`.
- CN: 该代码块实现与 quantization patterns 相关的局部辅助逻辑。关键符号：`move`。

### Lines 1264-1264
```cpp
} // namespace torch::jit
```
- EN: Opens or closes namespace scopes so the remaining declarations live in the expected JIT/frontend module.
- CN: 打开或关闭命名空间作用域，使后续声明归属于预期的 JIT/前端模块。


## Key Concepts / 关键概念
- EN: IR graph manipulation — the file inspects or rewrites TorchScript graphs and nodes.
  CN: IR 图操作——该文件会检查或改写 TorchScript 的图、节点与块。
- EN: Optimization pass — the code simplifies graphs by removing redundant work or folding values.
  CN: 优化 pass——代码通过删除冗余计算或常量折叠来简化图。
- EN: Inlining — the implementation merges callees or blocks into their call sites.
  CN: 内联——实现会把被调用者或代码块并入调用位置。
- EN: Fusion support — the file prepares groups of operations for fused execution.
  CN: 融合支持——该文件会为融合执行准备算子分组。
- EN: Type and shape reasoning — the code propagates static information across the IR.
  CN: 类型与形状推理——代码会在 IR 中传播静态信息。
- EN: Subsystem integration — the implementation collaborates with nearby JIT/frontend headers and runtime helpers.
  CN: 子系统集成——实现会与附近的 JIT/前端头文件以及运行时辅助模块协同工作。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `c10/util/irange.h`, `torch/csrc/jit/ir/ir.h`, `torch/csrc/jit/ir/subgraph_matcher.h`, `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/quantization/helper.h`, `torch/csrc/jit/passes/subgraph_rewrite.h`
- External includes / 外部头文件: `string`, `unordered_map`, `utility`
- Namespaces / 命名空间: `torch::jit`
- Representative symbols / 代表性符号: `QuantFusionInfo`, `getExtraArgList`, `accumulate`, `begin`, `end`, `string`, `getAtenOpPattern`, `append`, `item`, `move`, `...`
