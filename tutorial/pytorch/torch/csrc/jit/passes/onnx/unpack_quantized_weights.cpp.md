# unpack_quantized_weights.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/passes/onnx/unpack_quantized_weights.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements the TorchScript JIT pass logic for unpack quantized weights, including graph analysis and rewrites.
- 用途 (CN): 实现与 unpack quantized weights 相关的 TorchScript JIT pass 逻辑，包括图分析与改写。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11
```cpp
#include <torch/csrc/jit/passes/onnx/unpack_quantized_weights.h>

#include <ATen/native/quantized/PackedParams.h>
#include <c10/util/irange.h>
#include <torch/csrc/jit/ir/constants.h>
#include <torch/csrc/jit/ir/irparser.h>
#include <torch/csrc/jit/ir/subgraph_matcher.h>
#include <torch/csrc/jit/jit_log.h>
#include <torch/csrc/jit/passes/onnx/helper.h>
#include <torch/csrc/jit/passes/subgraph_rewrite.h>

```
- EN: Pulls in the headers needed by the unpack quantized weights logic. Internal dependencies: `torch/csrc/jit/passes/onnx/unpack_quantized_weights.h`, `ATen/native/quantized/PackedParams.h`, `c10/util/irange.h`, `torch/csrc/jit/ir/constants.h`, `torch/csrc/jit/ir/irparser.h`, `...`; external dependencies: none.
- CN: 为 unpack quantized weights 相关逻辑引入所需头文件。内部依赖：`torch/csrc/jit/passes/onnx/unpack_quantized_weights.h`, `ATen/native/quantized/PackedParams.h`, `c10/util/irange.h`, `torch/csrc/jit/ir/constants.h`, `torch/csrc/jit/ir/irparser.h`, `...`；外部依赖：无。

### Lines 12-21
```cpp
// TODO: Switch to per operator headers after
// https://github.com/pytorch/pytorch/pull/68693 is merged
#include <ATen/Functions.h>

using ::c10::Dispatcher;

namespace torch::jit {
namespace onnx {
using namespace ::c10::onnx;

```
- EN: This block implements local helper logic for unpack quantized weights. Key symbols: `namespace`.
- CN: 该代码块实现与 unpack quantized weights 相关的局部辅助逻辑。关键符号：`namespace`。

### Lines 22-30
```cpp
}

static std::vector<Node*> CreateQuantizedWeights(
    std::shared_ptr<Graph>& graph,
    const at::Tensor& weight,
    int8_t* data,
    const std::vector<int64_t>& shapes,
    const std::vector<int64_t>& strides) {
  auto qscheme = weight.qscheme();
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `CreateQuantizedWeights`, `qscheme`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`CreateQuantizedWeights`, `qscheme`。

### Lines 31-39
```cpp
  std::vector<Node*> unpacked_wt;

  // Retrieve scales and zero_points. Their formats are different depending on
  // different weight qscheme.
  std::vector<float> scale_data;
  std::vector<int64_t> scale_shapes;
  std::vector<int64_t> zero_point_data;
  std::vector<int64_t> zero_point_shapes;
  std::vector<int64_t> axis_data;
```
- EN: This block manipulates TorchScript IR objects. Key symbols: no dominant local symbol names.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：无明显局部符号。

### Lines 40-51
```cpp
  switch (qscheme) {
    case c10::kPerTensorAffine: {
      // Cast to float since ONNX (De)QuantizeLinear only supports float scale.
      scale_data = {static_cast<float>(weight.q_scale())};
      scale_shapes = {1};
      zero_point_data = {weight.q_zero_point()};
      zero_point_shapes = {1};
      break;
    }
    case c10::kPerChannelAffine:
    case c10::kPerChannelAffineFloatQParams: {
      auto q_scales = weight.q_per_channel_scales();
```
- EN: This block implements local helper logic for unpack quantized weights. Key symbols: `q_scale`, `q_zero_point`, `q_per_channel_scales`.
- CN: 该代码块实现与 unpack quantized weights 相关的局部辅助逻辑。关键符号：`q_scale`, `q_zero_point`, `q_per_channel_scales`。

### Lines 52-64
```cpp
      auto* scale_data_raw = q_scales.const_data_ptr<double>();
      scale_shapes = q_scales.sizes().vec();
      TORCH_INTERNAL_ASSERT(
          scale_shapes.size() == 1,
          "quantized per channel scales are expected as 1-d array.");
      scale_data.resize(scale_shapes[0]);
      // Cast to float since ONNX (De)QuantizeLinear only supports float scale.
      std::transform(
          scale_data_raw,
          scale_data_raw + scale_shapes[0],
          scale_data.begin(),
          [](double x) { return static_cast<float>(x); });

```
- EN: This block produces a result or forwards a computed value. Key symbols: `sizes`, `vec`, `size`, `resize`, `transform`, `begin`.
- CN: 该代码块返回结果或转发已计算的值。关键符号：`sizes`, `vec`, `size`, `resize`, `transform`, `begin`。

### Lines 65-80
```cpp
      auto q_zero_points = weight.q_per_channel_zero_points();
      auto* zero_point_data_raw = q_zero_points.const_data_ptr<int64_t>();
      zero_point_shapes = q_zero_points.sizes().vec();
      TORCH_INTERNAL_ASSERT(
          zero_point_shapes.size() == 1,
          "quantized per channel zero points are expected as 1-d array.");
      zero_point_data = std::vector<int64_t>(
          zero_point_data_raw, zero_point_data_raw + zero_point_shapes[0]);
      axis_data = {weight.q_per_channel_axis()};
      break;
    }
    default:
      TORCH_CHECK(
          false, "Unsupported qscheme for weight, got ", toString(qscheme));
  }

```
- EN: This block implements local helper logic for unpack quantized weights. Key symbols: `q_per_channel_zero_points`, `sizes`, `vec`, `size`, `q_per_channel_axis`, `toString`.
- CN: 该代码块实现与 unpack quantized weights 相关的局部辅助逻辑。关键符号：`q_per_channel_zero_points`, `sizes`, `vec`, `size`, `q_per_channel_axis`, `toString`。

### Lines 81-89
```cpp
  Node* data_node = graph->create(prim::Constant);
  auto data_value =
      at::from_blob(
          data, c10::IntArrayRef(shapes), c10::IntArrayRef(strides), at::kChar)
          .to(at::kCPU);
  // Need clone because at::from_blob does not take ownership of data.
  data_node->t_(Symbol::attr("value"), data_value.clone());

  Node* scale_node = graph->create(prim::Constant);
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `create`, `from_blob`, `IntArrayRef`, `to`, `t_`, `attr`, `...`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`create`, `from_blob`, `IntArrayRef`, `to`, `t_`, `attr`, `...`。

### Lines 90-102
```cpp
  auto scale_value =
      at::from_blob(
          scale_data.data(), c10::IntArrayRef(scale_shapes), at::kFloat)
          .to(at::kCPU);
  scale_node->t_(Symbol::attr("value"), scale_value.clone());

  Node* zero_point_node = graph->create(prim::Constant);
  auto zero_point_value =
      at::from_blob(
          zero_point_data.data(), c10::IntArrayRef(zero_point_shapes), at::kInt)
          .to(at::kCPU);
  zero_point_node->t_(Symbol::attr("value"), zero_point_value.clone());

```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `from_blob`, `data`, `IntArrayRef`, `to`, `t_`, `attr`, `...`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`from_blob`, `data`, `IntArrayRef`, `to`, `t_`, `attr`, `...`。

### Lines 103-113
```cpp
  Node* axis_node = graph->create(prim::Constant);
  if (!axis_data.empty()) {
    auto axis_value =
        at::from_blob(
            axis_data.data(), c10::IntArrayRef(axis_data.size()), at::kLong)
            .to(at::kCPU);
    axis_node->t_(attr::value, axis_value.clone());
  } else {
    axis_node->output()->setType(NoneType::get());
  }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `create`, `empty`, `from_blob`, `data`, `IntArrayRef`, `size`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`create`, `empty`, `from_blob`, `data`, `IntArrayRef`, `size`, `...`。

### Lines 114-122
```cpp
  return {data_node, scale_node, zero_point_node, axis_node};
}

static Node* CreateQuantizedBias(
    std::vector<float> data,
    std::shared_ptr<Graph>& graph,
    const std::vector<int64_t>& shapes) {
  Node* const_node_1 = graph->create(prim::Constant);
  auto const_bias =
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `CreateQuantizedBias`, `create`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`CreateQuantizedBias`, `create`。

### Lines 123-131
```cpp
      at::from_blob(data.data(), c10::IntArrayRef(shapes), at::kFloat)
          .to(at::kCPU);
  auto options = c10::TensorOptions().dtype(at::kFloat).device(at::kCPU);
  at::Tensor const_bias_copy = at::empty(c10::IntArrayRef(shapes), options);
  const_bias_copy.copy_(const_bias);
  const_node_1->t_(Symbol::attr("value"), const_bias_copy);
  return const_node_1;
}

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `from_blob`, `data`, `IntArrayRef`, `to`, `TensorOptions`, `dtype`, `...`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`from_blob`, `data`, `IntArrayRef`, `to`, `TensorOptions`, `dtype`, `...`。

### Lines 132-140
```cpp
static Node* createIntTuple(
    const std::vector<int64_t>& is,
    std::shared_ptr<Graph>& graph) {
  Node* const_node = graph->create(Symbol::onnx("Constant"));
  const_node->is_(Symbol::attr("value"), is);
  return const_node;
}

static Node* createInt(int64_t i, std::shared_ptr<Graph>& graph) {
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `createIntTuple`, `create`, `onnx`, `is_`, `attr`, `createInt`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`createIntTuple`, `create`, `onnx`, `is_`, `attr`, `createInt`。

### Lines 141-150
```cpp
  Node* const_node = graph->create(Symbol::onnx("Constant"));
  const_node->i_(Symbol::attr("value"), i);
  return const_node;
}

static void ConvertQuantizedWeight(
    std::shared_ptr<Graph>& graph,
    Node* node,
    at::Tensor& weight) {
  std::vector<int64_t> wt_sizes = weight.sizes().vec();
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `create`, `onnx`, `i_`, `attr`, `ConvertQuantizedWeight`, `sizes`, `...`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`create`, `onnx`, `i_`, `attr`, `ConvertQuantizedWeight`, `sizes`, `...`。

### Lines 151-161
```cpp
  std::vector<int64_t> wt_strides = weight.strides().vec();
  // Remove packed_params
  node->removeInput(1);

  auto* wt_data =
      reinterpret_cast<int8_t*>(weight.mutable_data_ptr<c10::qint8>());

  std::vector<Node*> unpacked_wt =
      CreateQuantizedWeights(graph, weight, wt_data, wt_sizes, wt_strides);
  graph->setInsertPoint(node);
  Node* quant_node = graph->create(prim::TupleConstruct);
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `strides`, `vec`, `removeInput`, `CreateQuantizedWeights`, `setInsertPoint`, `create`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`strides`, `vec`, `removeInput`, `CreateQuantizedWeights`, `setInsertPoint`, `create`。

### Lines 162-173
```cpp
  for (auto* n : unpacked_wt) {
    n->insertBefore(node);
    quant_node->addInput(n->output());
  }
  quant_node->insertBefore(node);
  node->insertInput(1, quant_node->output());
}

// CONV1D needs a different unpacking from CONV, since it's
// packed as CONV2D intentionally at the first place.
// See: https://github.com/pytorch/pytorch/pull/38248
enum class QuantizedParamsType { CONV1D, CONV, LINEAR };
```
- EN: Declares core types or data containers for this file. Prominent symbols: `insertBefore`, `addInput`, `output`, `insertInput`, `class`.
- CN: 声明该文件使用的核心类型或数据容器。主要符号：`insertBefore`, `addInput`, `output`, `insertInput`, `class`。

### Lines 174-182
```cpp

// This is called before the onnx pass. Using pattern matching we
// find the relevant nodes and extract the packed_params. The packed_params are
// passed to the appropriate unpack function using c10::Dispatcher. We insert
// the unpacked weights and bias into the graph using
// caffe2::Int8GivenTensorFill nodes.
static void unpackQuantizedWeightsHelper(
    std::shared_ptr<Graph>& graph,
    std::map<std::string, IValue>& paramsDict,
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `unpackQuantizedWeightsHelper`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`unpackQuantizedWeightsHelper`。

### Lines 183-191
```cpp
    const std::string& pattern,
    const std::string& unpack_fn,
    QuantizedParamsType params_type,
    bool expect_output_padding = false) {
  Graph pattern_graph;
  std::unordered_map<std::string, Value*> vmap;
  parseIR(pattern, &pattern_graph, vmap);
  const auto& matches = findPatternMatches(pattern_graph, *graph);

```
- EN: This block manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `parseIR`, `findPatternMatches`.
- CN: 该代码块操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`parseIR`, `findPatternMatches`。

### Lines 192-204
```cpp
  for (const auto& match : matches) {
    auto match_vmap = match.values_map;
    auto qlinear_node = match_vmap.at(vmap.at("r"))->node();
    std::string quantized_weight =
        match_vmap.at(vmap.at("r"))->node()->inputs()[1]->debugName();

    auto itr = paramsDict.find(quantized_weight);
    if (itr == paramsDict.end()) {
      throw std::runtime_error(
          "getValues: Quantized weight value not found amongst constant parameters.");
    }
    at::Tensor unpacked_weight;
    std::optional<at::Tensor> bias;
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `node`, `inputs`, `debugName`, `find`, `end`, `runtime_error`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`node`, `inputs`, `debugName`, `find`, `end`, `runtime_error`。

### Lines 205-218
```cpp
    constexpr int64_t stride_idx = 2;
    constexpr int64_t padding_idx = 3;
    int64_t output_padding_idx = 0;
    int64_t dilation_idx = 0;
    int64_t groups_idx = 0;
    if (expect_output_padding) {
      output_padding_idx = 4;
      dilation_idx = 5;
      groups_idx = 6;
    } else {
      dilation_idx = 4;
      groups_idx = 5;
    }
    std::optional<torch::List<int64_t>> stride, padding, dilation,
```
- EN: This block handles conditional branches. Key symbols: no dominant local symbol names.
- CN: 该代码块处理条件分支。关键符号：无明显局部符号。

### Lines 219-229
```cpp
        output_padding;
    std::optional<int64_t> groups;
    std::optional<int64_t> transpose;

    torch::List<int64_t> stride_int, padding_int, dilation_int,
        output_padding_int;

    if (itr->second.isTuple()) {
      // Pre-unpacked weights. Comes from Conv/Linear weights which are
      // stored as bound C++ classes.
      auto ser_tup = itr->second.toTuple();
```
- EN: This block handles conditional branches. Key symbols: `isTuple`, `toTuple`.
- CN: 该代码块处理条件分支。关键符号：`isTuple`, `toTuple`。

### Lines 230-238
```cpp

      if (params_type == QuantizedParamsType::CONV &&
          ser_tup->elements()[0].isInt()) {
        const auto& elements = ser_tup->elements();
        auto version = elements[0].toInt();
        TORCH_INTERNAL_ASSERT(version == 3, "Unknown serialization version");
        TORCH_INTERNAL_ASSERT(elements.size() == 3, "Wrong tuple size.");

        auto config_vals = elements[1].to<std::vector<int64_t>>();
```
- EN: This block handles conditional branches. Key symbols: `elements`, `isInt`, `toInt`, `size`.
- CN: 该代码块处理条件分支。关键符号：`elements`, `isInt`, `toInt`, `size`。

### Lines 239-250
```cpp
        auto tensors = elements[2].to<std::vector<std::optional<at::Tensor>>>();

        const std::optional<at::Tensor>& weight = tensors[1];
        TORCH_INTERNAL_ASSERT(
            weight, "Weight should always be present in serialized qconv.");
        unpacked_weight = *weight;
        bias = tensors[2];

        const int64_t kSpatialDim = config_vals.at(0);
        // skip kSpatialDim
        unsigned idx = 1;
        for ([[maybe_unused]] const auto i : c10::irange(kSpatialDim)) {
```
- EN: This block iterates over collections or graph structures. Key symbols: `irange`.
- CN: 该代码块遍历集合或图结构。关键符号：`irange`。

### Lines 251-262
```cpp
          stride_int.emplace_back(config_vals.at(idx));
          idx++;
        }
        for ([[maybe_unused]] const auto i : c10::irange(kSpatialDim)) {
          padding_int.emplace_back(config_vals.at(idx));
          idx++;
        }
        for ([[maybe_unused]] const auto i : c10::irange(kSpatialDim)) {
          dilation_int.emplace_back(config_vals.at(idx));
          idx++;
        }
        for ([[maybe_unused]] const auto i : c10::irange(kSpatialDim)) {
```
- EN: This block iterates over collections or graph structures. Key symbols: `emplace_back`, `irange`.
- CN: 该代码块遍历集合或图结构。关键符号：`emplace_back`, `irange`。

### Lines 263-276
```cpp
          output_padding_int.emplace_back(config_vals.at(idx));
          idx++;
        }
        int64_t groups_int = config_vals.at(idx);
        idx++;
        int64_t flags = config_vals.at(idx);
        idx++;
        TORCH_INTERNAL_ASSERT(
            idx == config_vals.size(),
            "Unexpected length of config_vals, expected ",
            idx,
            " got ",
            config_vals.size());

```
- EN: This block implements local helper logic for unpack quantized weights. Key symbols: `emplace_back`, `size`.
- CN: 该代码块实现与 unpack quantized weights 相关的局部辅助逻辑。关键符号：`emplace_back`, `size`。

### Lines 277-287
```cpp
        bool transpose_int = flags & (1 << 0);

        int64_t other_flags = flags & ~(1 << 0);
        TORCH_CHECK(other_flags == 0, "Unexpected flags set in ", flags, ".");

        stride = stride_int;
        padding = padding_int;
        dilation = dilation_int;
        groups = groups_int;
        transpose = transpose_int;
        if (expect_output_padding) {
```
- EN: This block handles conditional branches. Key symbols: `~`.
- CN: 该代码块处理条件分支。关键符号：`~`。

### Lines 288-297
```cpp
          output_padding = output_padding_int;
        }
      } else if (
          (params_type == QuantizedParamsType::CONV ||
           params_type == QuantizedParamsType::CONV1D) &&
          ser_tup->elements()[0].isString()) {
        const auto& elements = ser_tup->elements();
        auto version = elements[0].toStringRef();
        TORCH_INTERNAL_ASSERT(version == "2", "Unknown serialization version");
        std::vector<at::Tensor> non_optional = elements[1].toTensorVector();
```
- EN: This block handles conditional branches. Key symbols: `elements`, `isString`, `toStringRef`, `toTensorVector`.
- CN: 该代码块处理条件分支。关键符号：`elements`, `isString`, `toStringRef`, `toTensorVector`。

### Lines 298-308
```cpp

        const at::Tensor& conv_params_packed = non_optional[0];
        unpacked_weight = non_optional[1];

        const int64_t kSpatialDim = conv_params_packed[0].item<int64_t>();
        // skip kSpatialDim
        int64_t idx = 1;
        // kSpatialDim = 2 even it's for Conv1D from torch.op to adopt Conv2D,
        // so we need a special unpack for Conv1D which has Conv2D dim.
        // See: https://github.com/pytorch/pytorch/pull/38248
        for (const auto i : c10::irange(kSpatialDim)) {
```
- EN: This block iterates over collections or graph structures. Key symbols: `irange`.
- CN: 该代码块遍历集合或图结构。关键符号：`irange`。

### Lines 309-320
```cpp
          if (params_type != QuantizedParamsType::CONV1D || i != 0) {
            stride_int.emplace_back(conv_params_packed[idx].item<int64_t>());
          }
          idx++;
        }
        for (const auto i : c10::irange(kSpatialDim)) {
          if (params_type != QuantizedParamsType::CONV1D || i != 0) {
            padding_int.emplace_back(conv_params_packed[idx].item<int64_t>());
          }
          idx++;
        }
        for (const auto i : c10::irange(kSpatialDim)) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures. Key symbols: `emplace_back`, `irange`.
- CN: 该代码块处理条件分支；遍历集合或图结构。关键符号：`emplace_back`, `irange`。

### Lines 321-333
```cpp
          if (params_type != QuantizedParamsType::CONV1D || i != 0) {
            dilation_int.emplace_back(conv_params_packed[idx].item<int64_t>());
          }
          idx++;
        }
        for (const auto i : c10::irange(kSpatialDim)) {
          if (params_type != QuantizedParamsType::CONV1D || i != 0) {
            output_padding_int.emplace_back(
                conv_params_packed[idx].item<int64_t>());
          }
          idx++;
        }
        auto groups_int = conv_params_packed[idx].item<int64_t>();
```
- EN: This block handles conditional branches; iterates over collections or graph structures. Key symbols: `emplace_back`, `irange`.
- CN: 该代码块处理条件分支；遍历集合或图结构。关键符号：`emplace_back`, `irange`。

### Lines 334-343
```cpp
        idx++;
        auto transpose_int = conv_params_packed[idx].item<int64_t>();
        idx++;
        TORCH_INTERNAL_ASSERT(
            idx == conv_params_packed.numel(),
            "Unexpected length of conv_params_packed, expected ",
            idx,
            " got ",
            conv_params_packed.numel());

```
- EN: This block implements local helper logic for unpack quantized weights. Key symbols: `numel`.
- CN: 该代码块实现与 unpack quantized weights 相关的局部辅助逻辑。关键符号：`numel`。

### Lines 344-355
```cpp
        torch::List<c10::IValue> optional = elements[2].toList();
        bias = optional.get(0).toOptional<at::Tensor>();

        if (params_type == QuantizedParamsType::CONV1D) {
          unpacked_weight = unpacked_weight.squeeze_(2);
        }
        stride = stride_int;
        padding = padding_int;
        dilation = dilation_int;
        groups = groups_int;
        transpose = transpose_int;
        if (expect_output_padding) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `toList`, `get`, `squeeze_`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`toList`, `get`, `squeeze_`。

### Lines 356-364
```cpp
          output_padding = output_padding_int;
        }
      } else { // Legacy
        unpacked_weight = ser_tup->elements()[0].toTensor();
        bias = ser_tup->elements()[1].toOptional<at::Tensor>();
        // conv only parameters
        if (ser_tup->elements().size() > 2) {
          auto stride_ivalue = ser_tup->elements()[stride_idx].toListRef();
          auto padding_ivalue = ser_tup->elements()[padding_idx].toListRef();
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `elements`, `toTensor`, `size`, `toListRef`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`elements`, `toTensor`, `size`, `toListRef`。

### Lines 365-374
```cpp
          auto dilation_ivalue = ser_tup->elements()[dilation_idx].toListRef();
          auto groups_ivalue = ser_tup->elements()[groups_idx];

          for (const auto& s : stride_ivalue) {
            stride_int.emplace_back(s.toTensor()[0].item<int64_t>());
          }
          for (const auto& p : padding_ivalue) {
            padding_int.emplace_back(p.toTensor()[0].item<int64_t>());
          }
          for (const auto& d : dilation_ivalue) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `elements`, `toListRef`, `emplace_back`, `toTensor`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`elements`, `toListRef`, `emplace_back`, `toTensor`。

### Lines 375-383
```cpp
            dilation_int.emplace_back(d.toTensor()[0].item<int64_t>());
          }
          groups = groups_ivalue.toTensor()[0].item<int64_t>();
          stride = stride_int;
          padding = padding_int;
          dilation = dilation_int;

          if (expect_output_padding) {
            auto output_padding_ivalue =
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `emplace_back`, `toTensor`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`emplace_back`, `toTensor`。

### Lines 384-395
```cpp
                ser_tup->elements()[output_padding_idx].toListRef();
            for (const auto& d : output_padding_ivalue) {
              output_padding_int.emplace_back(d.toTensor()[0].item<int64_t>());
            }
            output_padding = output_padding_int;
          }
        }
      }
    } else {
      TORCH_INTERNAL_ASSERT(itr->second.isTensor());
      at::Tensor packed_weight = itr->second.toTensor();
      auto op = Dispatcher::singleton()
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `elements`, `toListRef`, `emplace_back`, `toTensor`, `isTensor`, `singleton`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`elements`, `toListRef`, `emplace_back`, `toTensor`, `isTensor`, `singleton`。

### Lines 396-406
```cpp
                    .findSchemaOrThrow(unpack_fn.c_str(), "")
                    .typed<std::tuple<at::Tensor, std::optional<at::Tensor>>(
                        at::Tensor)>();
      std::tie(unpacked_weight, bias) = op.call(packed_weight);
    }

    ConvertQuantizedWeight(graph, qlinear_node, unpacked_weight);

    // Add bias
    at::Tensor original_bias;
    if (bias.has_value()) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `findSchemaOrThrow`, `c_str`, `tie`, `call`, `ConvertQuantizedWeight`, `has_value`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`findSchemaOrThrow`, `c_str`, `tie`, `call`, `ConvertQuantizedWeight`, `has_value`。

### Lines 407-415
```cpp
      original_bias = bias.value();
      original_bias.set_requires_grad(false);
    } else {
      int64_t bias_size = unpacked_weight.size(0);
      original_bias =
          at::zeros(bias_size, unpacked_weight.options().dtype(at::kFloat));
    }

    auto input_val = match_vmap.at(vmap.at("r"))->node()->inputs()[0];
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `value`, `set_requires_grad`, `size`, `zeros`, `options`, `dtype`, `...`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`value`, `set_requires_grad`, `size`, `zeros`, `options`, `dtype`, `...`。

### Lines 416-426
```cpp
    TORCH_INTERNAL_ASSERT(
        input_val->type()->isSubtypeOf(*TensorType::get()),
        "Unsupported input type. Expected TensorType, got ",
        input_val->type()->str());

    std::vector<float> bias_values(original_bias.numel());
    auto bias_data = original_bias.const_data_ptr<float>();
    for (const auto i : c10::irange(original_bias.numel())) {
      bias_values[i] = bias_data[i];
    }
    Node* bias_node =
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `type`, `isSubtypeOf`, `get`, `str`, `bias_values`, `numel`, `...`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`type`, `isSubtypeOf`, `get`, `str`, `bias_values`, `numel`, `...`。

### Lines 427-437
```cpp
        CreateQuantizedBias(bias_values, graph, original_bias.sizes().vec());
    bias_node->insertBefore(qlinear_node);
    // For quantized_linear inputs, the order is input, weight, bias, ....
    // Therefore bias is at location 2.
    qlinear_node->insertInput(2, bias_node->output());

    // add conv arguments: stride, padding, dilation, groups, output_padding
    if (stride.has_value() && padding.has_value() && dilation.has_value() &&
        groups.has_value() &&
        (!expect_output_padding || output_padding.has_value())) {
      std::vector<std::optional<torch::List<int64_t>>> conv_ints_args;
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `CreateQuantizedBias`, `sizes`, `vec`, `insertBefore`, `insertInput`, `output`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`CreateQuantizedBias`, `sizes`, `vec`, `insertBefore`, `insertInput`, `output`, `...`。

### Lines 438-446
```cpp
      conv_ints_args.push_back(stride);
      conv_ints_args.push_back(padding);
      if (expect_output_padding) {
        conv_ints_args.push_back(output_padding);
      }
      conv_ints_args.push_back(dilation);
      // skip (input, weight, bias)
      const size_t arg_offset = 3;
      for (const auto i : c10::irange(conv_ints_args.size())) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures. Key symbols: `push_back`, `irange`, `size`.
- CN: 该代码块处理条件分支；遍历集合或图结构。关键符号：`push_back`, `irange`, `size`。

### Lines 447-456
```cpp
        Node* ints_node =
            createIntTuple(conv_ints_args[i].value().vec(), graph);
        ints_node->insertBefore(qlinear_node);
        qlinear_node->insertInput(arg_offset + i, ints_node->output());
      }
      Node* groups_node = createInt(groups.value(), graph);
      groups_node->insertBefore(qlinear_node);
      qlinear_node->insertInput(groups_idx + 1, groups_node->output());
    }
    auto b = graph->block();
```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `createIntTuple`, `value`, `vec`, `insertBefore`, `insertInput`, `output`, `...`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`createIntTuple`, `value`, `vec`, `insertBefore`, `insertInput`, `output`, `...`。

### Lines 457-469
```cpp
    auto valsToParamsMap = buildValueToParamsMap(b, paramsDict);
    eraseUnusedValuesFromMap(valsToParamsMap);
  }
}

static std::
    unordered_map<c10::ScalarType, c10::ScalarType, ScalarTypeHashFunction>
        qTypeToValType = {
            {c10::ScalarType::QInt8, c10::ScalarType::Char},
            {c10::ScalarType::QUInt8, c10::ScalarType::Byte},
            {c10::ScalarType::QInt32, c10::ScalarType::Int},
            {c10::ScalarType::QUInt4x2, c10::ScalarType::Byte},
};
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `buildValueToParamsMap`, `eraseUnusedValuesFromMap`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`buildValueToParamsMap`, `eraseUnusedValuesFromMap`。

### Lines 470-481
```cpp

// Unpack quantized tensor inputs into {value, scale, zero_point},
// Then create a prim::TupleConstruct node based on these three values.
static void UnpackQuantizedTensorInputs(std::shared_ptr<Graph>& graph) {
  for (size_t index = 0; index < graph->inputs().size();) {
    auto g_input = graph->inputs()[index];
    TensorTypePtr shape_type = g_input->type()->cast<TensorType>();
    if (!shape_type || !shape_type->scalarType().has_value()) {
      index++;
      continue;
    }
    auto scalar_type = shape_type->scalarType().value();
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `UnpackQuantizedTensorInputs`, `inputs`, `size`, `type`, `scalarType`, `has_value`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`UnpackQuantizedTensorInputs`, `inputs`, `size`, `type`, `scalarType`, `has_value`, `...`。

### Lines 482-491
```cpp
    if (qTypeToValType.find(scalar_type) == qTypeToValType.end()) {
      index++;
      continue;
    }
    std::string input_name = g_input->debugName();
    auto input_value =
        graph->insertInput(index, input_name + "_value")
            ->setType(shape_type->withScalarType(qTypeToValType[scalar_type]));
    // scale and zero_point type can be found at torch/include/ATen/Operators.h
    auto input_scale =
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `find`, `end`, `debugName`, `insertInput`, `setType`, `withScalarType`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`find`, `end`, `debugName`, `insertInput`, `setType`, `withScalarType`。

### Lines 492-500
```cpp
        graph->insertInput(index + 1, input_name + "_scale")
            ->setType(TensorType::create(
                at::kDouble, at::kCPU, 0, /*requires_grad=*/std::nullopt));
    auto input_zero_point =
        graph->insertInput(index + 2, input_name + "_zero_point")
            ->setType(TensorType::create(
                at::kLong, at::kCPU, 0, /*requires_grad=*/std::nullopt));
    std::vector<Value*> converted{input_value, input_scale, input_zero_point};
    auto input_tuple =
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `insertInput`, `setType`, `create`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`insertInput`, `setType`, `create`。

### Lines 501-510
```cpp
        graph->prependNode(graph->createTuple(converted))->output();
    g_input->replaceAllUsesWith(input_tuple);
    // Erase the original quantized tensor input.
    graph->eraseInput(index + converted.size());
    index += 3;
  }
}

// https://github.com/pytorch/pytorch/wiki/PyTorch-ONNX-exporter#quantized-model-export
void UnpackQuantizedWeights(
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `prependNode`, `createTuple`, `output`, `replaceAllUsesWith`, `eraseInput`, `size`, `...`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`prependNode`, `createTuple`, `output`, `replaceAllUsesWith`, `eraseInput`, `size`, `...`。

### Lines 511-520
```cpp
    std::shared_ptr<Graph>& graph,
    std::map<std::string, IValue>& paramsDict) {
  std::string qlinear = R"(
  graph(%input, %packed_weight, %w_scale, %w_zero_point):
        %r = quantized::linear(%input, %packed_weight, %w_scale, %w_zero_point)
        return (%r) )";
  std::string qlinear_relu = R"(
  graph(%input, %packed_weight, %w_scale, %w_zero_point):
        %r = quantized::linear_relu(%input, %packed_weight, %w_scale, %w_zero_point)
        return (%r) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `linear`, `linear_relu`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `linear`, `linear_relu`。

### Lines 521-529
```cpp
  std::string qconv1d = R"(
  graph(%input, %packed_params, %scale, %zero_point):
        %r = quantized::conv1d(%input, %packed_params, %scale, %zero_point)
        return (%r) )";
  std::string qconv1d_relu = R"(
  graph(%input, %packed_params, %scale, %zero_point):
        %r = quantized::conv1d_relu(%input, %packed_params, %scale, %zero_point)
        return (%r) )";
  std::string qconv2d = R"(
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `conv1d`, `conv1d_relu`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `conv1d`, `conv1d_relu`。

### Lines 530-540
```cpp
  graph(%input, %packed_params, %scale, %zero_point):
        %r = quantized::conv2d(%input, %packed_params, %scale, %zero_point)
        return (%r) )";
  std::string qconv2d_relu = R"(
  graph(%input, %packed_params, %scale, %zero_point):
        %r = quantized::conv2d_relu(%input, %packed_params, %scale, %zero_point)
        return (%r) )";
  std::string qconv3d = R"(
  graph(%input, %packed_params, %scale, %zero_point):
        %r = quantized::conv3d(%input, %packed_params, %scale, %zero_point)
        return (%r) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `conv2d`, `conv2d_relu`, `conv3d`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `conv2d`, `conv2d_relu`, `conv3d`。

### Lines 541-549
```cpp
  std::string qconv3d_relu = R"(
  graph(%input, %packed_params, %scale, %zero_point):
        %r = quantized::conv3d_relu(%input, %packed_params, %scale, %zero_point)
        return (%r) )";
  std::string qconv_transpose1d = R"(
  graph(%input, %packed_params, %scale, %zero_point):
        %r = quantized::conv_transpose1d(%input, %packed_params, %scale, %zero_point)
        return (%r) )";
  std::string qconv_transpose2d = R"(
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `conv3d_relu`, `conv_transpose1d`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `conv3d_relu`, `conv_transpose1d`。

### Lines 550-567
```cpp
  graph(%input, %packed_params, %scale, %zero_point):
        %r = quantized::conv_transpose2d(%input, %packed_params, %scale, %zero_point)
        return (%r) )";
  std::string qconv_transpose3d = R"(
  graph(%input, %packed_params, %scale, %zero_point):
        %r = quantized::conv_transpose3d(%input, %packed_params, %scale, %zero_point)
        return (%r) )";
  unpackQuantizedWeightsHelper(
      graph,
      paramsDict,
      qlinear,
      "quantized::linear_unpack",
      QuantizedParamsType::LINEAR);
  unpackQuantizedWeightsHelper(
      graph,
      paramsDict,
      qlinear_relu,
      "quantized::linear_unpack",
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `conv_transpose2d`, `conv_transpose3d`, `unpackQuantizedWeightsHelper`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `conv_transpose2d`, `conv_transpose3d`, `unpackQuantizedWeightsHelper`。

### Lines 568-585
```cpp
      QuantizedParamsType::LINEAR);
  unpackQuantizedWeightsHelper(
      graph,
      paramsDict,
      qconv1d,
      "quantized::conv1d_unpack",
      QuantizedParamsType::CONV1D);
  unpackQuantizedWeightsHelper(
      graph,
      paramsDict,
      qconv2d,
      "quantized::conv2d_unpack",
      QuantizedParamsType::CONV);
  unpackQuantizedWeightsHelper(
      graph,
      paramsDict,
      qconv1d_relu,
      "quantized::conv1d_unpack",
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `unpackQuantizedWeightsHelper`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`unpackQuantizedWeightsHelper`。

### Lines 586-603
```cpp
      QuantizedParamsType::CONV1D);
  unpackQuantizedWeightsHelper(
      graph,
      paramsDict,
      qconv2d_relu,
      "quantized::conv2d_unpack",
      QuantizedParamsType::CONV);
  unpackQuantizedWeightsHelper(
      graph,
      paramsDict,
      qconv3d,
      "quantized::conv3d_unpack",
      QuantizedParamsType::CONV);
  unpackQuantizedWeightsHelper(
      graph,
      paramsDict,
      qconv3d_relu,
      "quantized::conv3d_unpack",
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `unpackQuantizedWeightsHelper`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`unpackQuantizedWeightsHelper`。

### Lines 604-621
```cpp
      QuantizedParamsType::CONV);
  unpackQuantizedWeightsHelper(
      graph,
      paramsDict,
      qconv_transpose1d,
      "quantized::conv_transpose1d_unpack",
      QuantizedParamsType::CONV1D,
      true);
  unpackQuantizedWeightsHelper(
      graph,
      paramsDict,
      qconv_transpose2d,
      "quantized::conv_transpose2d_unpack",
      QuantizedParamsType::CONV,
      true);
  unpackQuantizedWeightsHelper(
      graph,
      paramsDict,
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `unpackQuantizedWeightsHelper`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`unpackQuantizedWeightsHelper`。

### Lines 622-633
```cpp
      qconv_transpose3d,
      "quantized::conv_transpose3d_unpack",
      QuantizedParamsType::CONV,
      true);
  UnpackQuantizedTensorInputs(graph);
  GRAPH_DUMP("After UnpackQuantizedWeights: ", graph);
}

// Caffe2 expects quantized ops to be in NHWC format while pytorch inputs are in
// NCHW. This pass inserts permutes to convert from NCHW to NHWC before each
// conv op and add another permute from NHWC to NCHW after the conv op.
static void insertPermutesHelper(
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `UnpackQuantizedTensorInputs`, `insertPermutesHelper`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`UnpackQuantizedTensorInputs`, `insertPermutesHelper`。

### Lines 634-642
```cpp
    std::shared_ptr<Graph>& graph,
    std::map<std::string, IValue>& paramsDict,
    const std::string& pattern) {
  Graph pattern_graph;
  std::unordered_map<std::string, Value*> vmap;
  parseIR(pattern, &pattern_graph, vmap);

  const auto& matches = findPatternMatches(pattern_graph, *graph);

```
- EN: This block manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `parseIR`, `findPatternMatches`.
- CN: 该代码块操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`parseIR`, `findPatternMatches`。

### Lines 643-653
```cpp
  for (const auto& match : matches) {
    auto match_vmap = match.values_map;
    auto op_node = match_vmap.at(vmap.at("r"))->node();
    auto input_node = match_vmap.at(vmap.at("r"))->node()->inputs()[0]->node();

    Node* permute_node_before = graph->create(
        Symbol::fromQualString("quantized::nchw2nhwc"), {input_node->output()});
    permute_node_before->insertBefore(op_node);
    op_node->removeInput(0);
    op_node->insertInput(0, permute_node_before->output());

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `node`, `inputs`, `create`, `fromQualString`, `output`, `insertBefore`, `...`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`node`, `inputs`, `create`, `fromQualString`, `output`, `insertBefore`, `...`。

### Lines 654-664
```cpp
    Node* permute_node_after = graph->create(
        Symbol::fromQualString("quantized::nhwc2nchw"),
        {op_node->outputs()[0]});
    permute_node_after->insertAfter(op_node);
    auto v = op_node->outputs().at(0);
    v->replaceAllUsesWith(permute_node_after->outputs().at(0));
    permute_node_after->removeInput(0);
    permute_node_after->addInput(v);
  }
}

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `create`, `fromQualString`, `outputs`, `insertAfter`, `replaceAllUsesWith`, `removeInput`, `...`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`create`, `fromQualString`, `outputs`, `insertAfter`, `replaceAllUsesWith`, `removeInput`, `...`。

### Lines 665-675
```cpp
void insertPermutes(
    std::shared_ptr<Graph>& graph,
    std::map<std::string, IValue>& paramsDict) {
  std::string qconv = R"(
  graph(%input, %weight, %bias, %stride, %padding, %dilation, %groups, %w_scale, %w_zero_point):
        %r = quantized::conv2d(%input, %weight, %bias, %stride, %padding, %dilation, %groups, %w_scale, %w_zero_point)
        return (%r) )";
  std::string qconv_relu = R"(
  graph(%input, %weight, %bias, %stride, %padding, %dilation, %groups, %w_scale, %w_zero_point):
        %r = quantized::conv2d_relu(%input, %weight, %bias, %stride, %padding, %dilation, %groups, %w_scale, %w_zero_point)
        return (%r) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `insertPermutes`, `graph`, `conv2d`, `conv2d_relu`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`insertPermutes`, `graph`, `conv2d`, `conv2d_relu`。

### Lines 676-686
```cpp
  std::string qconv_transpose = R"(
  graph(%input, %weight, %bias, %stride, %padding, %dilation, %output_padding, %groups, %w_scale, %w_zero_point):
        %r = quantized::conv_transpose2d(%input, %weight, %bias, %stride, %padding, %output_padding, %dilation, %groups, %w_scale, %w_zero_point)
        return (%r) )";

  insertPermutesHelper(graph, paramsDict, qconv);
  insertPermutesHelper(graph, paramsDict, qconv_relu);
  insertPermutesHelper(graph, paramsDict, qconv_transpose);
  GRAPH_DUMP("After insertPermutes: ", graph);
}

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `conv_transpose2d`, `insertPermutesHelper`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `conv_transpose2d`, `insertPermutesHelper`。

### Lines 687-687
```cpp
} // namespace torch::jit
```
- EN: Opens or closes namespace scopes so the remaining declarations live in the expected JIT/frontend module.
- CN: 打开或关闭命名空间作用域，使后续声明归属于预期的 JIT/前端模块。


## Key Concepts / 关键概念
- EN: IR graph manipulation — the file inspects or rewrites TorchScript graphs and nodes.
  CN: IR 图操作——该文件会检查或改写 TorchScript 的图、节点与块。
- EN: Schema handling — the implementation parses or normalizes operator/function signatures.
  CN: 模式处理——实现会解析或规范化算子/函数签名。
- EN: Parsing pipeline — the logic translates source text into TorchScript syntax structures.
  CN: 解析流水线——相关逻辑将源码文本转换为 TorchScript 语法结构。
- EN: Optimization pass — the code simplifies graphs by removing redundant work or folding values.
  CN: 优化 pass——代码通过删除冗余计算或常量折叠来简化图。
- EN: Fusion support — the file prepares groups of operations for fused execution.
  CN: 融合支持——该文件会为融合执行准备算子分组。
- EN: Type and shape reasoning — the code propagates static information across the IR.
  CN: 类型与形状推理——代码会在 IR 中传播静态信息。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `torch/csrc/jit/passes/onnx/unpack_quantized_weights.h`, `ATen/native/quantized/PackedParams.h`, `c10/util/irange.h`, `torch/csrc/jit/ir/constants.h`, `torch/csrc/jit/ir/irparser.h`, `torch/csrc/jit/ir/subgraph_matcher.h`, `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/onnx/helper.h`, `torch/csrc/jit/passes/subgraph_rewrite.h`, `ATen/Functions.h`
- External includes / 外部头文件: 无
- Namespaces / 命名空间: `torch::jit`, `onnx`
- Representative symbols / 代表性符号: `namespace`, `CreateQuantizedWeights`, `qscheme`, `q_scale`, `q_zero_point`, `q_per_channel_scales`, `sizes`, `vec`, `size`, `resize`, `...`
