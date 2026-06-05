# fold_conv_bn.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/passes/fold_conv_bn.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements the TorchScript JIT pass logic for fold conv bn, including graph analysis and rewrites.
- 用途 (CN): 实现与 fold conv bn 相关的 TorchScript JIT pass 逻辑，包括图分析与改写。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7
```cpp
#include <torch/csrc/jit/passes/fold_conv_bn.h>

#include <torch/csrc/jit/ir/subgraph_matcher.h>
#include <torch/csrc/jit/jit_log.h>
#include <torch/csrc/jit/passes/graph_rewrite_helper.h>
#include <torch/csrc/jit/passes/quantization/helper.h>

```
- EN: Pulls in the headers needed by the fold conv bn logic. Internal dependencies: `torch/csrc/jit/passes/fold_conv_bn.h`, `torch/csrc/jit/ir/subgraph_matcher.h`, `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/graph_rewrite_helper.h`, `torch/csrc/jit/passes/quantization/helper.h`; external dependencies: none.
- CN: 为 fold conv bn 相关逻辑引入所需头文件。内部依赖：`torch/csrc/jit/passes/fold_conv_bn.h`, `torch/csrc/jit/ir/subgraph_matcher.h`, `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/graph_rewrite_helper.h`, `torch/csrc/jit/passes/quantization/helper.h`；外部依赖：无。

### Lines 8-17
```cpp
#include <ATen/TensorOperators.h>

#ifndef AT_PER_OPERATOR_HEADERS
#include <ATen/Functions.h>
#else
#include <ATen/ops/ones_like.h>
#include <ATen/ops/rsqrt.h>
#include <ATen/ops/zeros_like.h>
#endif

```
- EN: Pulls in the headers needed by the fold conv bn logic. Internal dependencies: `ATen/TensorOperators.h`, `ATen/Functions.h`, `ATen/ops/ones_like.h`, `ATen/ops/rsqrt.h`, `ATen/ops/zeros_like.h`; external dependencies: none.
- CN: 为 fold conv bn 相关逻辑引入所需头文件。内部依赖：`ATen/TensorOperators.h`, `ATen/Functions.h`, `ATen/ops/ones_like.h`, `ATen/ops/rsqrt.h`, `ATen/ops/zeros_like.h`；外部依赖：无。

### Lines 18-29
```cpp
#include <stack>
#include <utility>

namespace torch::jit {

std::tuple<at::Tensor, at::Tensor> computeUpdatedConvWeightAndBias(
    const ConvBNParameters& p) {
  at::Tensor bn_var_rsqrt = at::rsqrt(p.bn_rv + p.bn_eps);
  const int64_t ndim = p.conv_w.dim();
  at::DimVector sizes(ndim, 1);
  sizes.at(0) = -1;

```
- EN: This block implements local helper logic for fold conv bn. Key symbols: `computeUpdatedConvWeightAndBias`, `rsqrt`, `dim`, `sizes`.
- CN: 该代码块实现与 fold conv bn 相关的局部辅助逻辑。关键符号：`computeUpdatedConvWeightAndBias`, `rsqrt`, `dim`, `sizes`。

### Lines 30-37
```cpp
  auto conv_w_dtype = p.conv_w.dtype();
  auto conv_b_dtype = p.conv_b.dtype();

  at::Tensor new_w = p.conv_w * (p.bn_w * bn_var_rsqrt).reshape(sizes);
  at::Tensor new_b = (p.conv_b - p.bn_rm) * bn_var_rsqrt * p.bn_w + p.bn_b;
  return std::make_tuple(new_w.to(conv_w_dtype), new_b.to(conv_b_dtype));
}

```
- EN: This block produces a result or forwards a computed value. Key symbols: `dtype`, `reshape`, `make_tuple`, `to`.
- CN: 该代码块返回结果或转发已计算的值。关键符号：`dtype`, `reshape`, `make_tuple`, `to`。

### Lines 38-44
```cpp
namespace {
using graph_rewrite_helper::PatternInfo;

static bool hastensor(Module& m, const char* name) {
  return m.hasattr(name) && m.attr(name).isTensor();
}

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph_rewrite_helper`, `hastensor`, `hasattr`, `attr`, `isTensor`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph_rewrite_helper`, `hastensor`, `hasattr`, `attr`, `isTensor`。

### Lines 45-57
```cpp
void replaceConvBiasWithGetAttr(Module& module) {
  for (const auto& method : module.get_methods()) {
    auto graph = method.graph();
    // Only looks for _convolution pattern.
    // Thus assumes that tracing will have always gotten rid of aten::conv2d or
    // aten::conv3d. If it did not, BN folding will fail.
    const PatternInfo& pattern_convolution = PatternInfo::parse_from_str(R"(
        graph(%a, %w, %b, %stride:int[], %padding:int[], %dilation:int[],
            %transposed:bool, %output_padding:int[], %groups:int, %benchmark:bool,
            %deterministic:bool, %cudnn_enabled:bool, %allow_tf32:bool):
          %conv_out = aten::_convolution(%a, %w, %b, %stride, %padding, %dilation,
              %transposed, %output_padding, %groups, %benchmark, %deterministic, %cudnn_enabled, %allow_tf32)
          return (%conv_out) )");
```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects; advances frontend parsing or lowering state; performs optimization-oriented rewriting. Key symbols: `replaceConvBiasWithGetAttr`, `get_methods`, `graph`, `parse_from_str`, `_convolution`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象；推进前端解析或降级状态；执行面向优化的改写。关键符号：`replaceConvBiasWithGetAttr`, `get_methods`, `graph`, `parse_from_str`, `_convolution`。

### Lines 58-65
```cpp
    const PatternInfo& pattern_convolution_deprecated =
        PatternInfo::parse_from_str(R"(
        graph(%a, %w, %b, %stride:int[], %padding:int[], %dilation:int[],
            %transposed:bool, %output_padding:int[], %groups:int, %benchmark:bool,
            %deterministic:bool, %cudnn_enabled:bool):
          %conv_out = aten::_convolution(%a, %w, %b, %stride, %padding, %dilation,
              %transposed, %output_padding, %groups, %benchmark, %deterministic, %cudnn_enabled)
          return (%conv_out) )");
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `parse_from_str`, `graph`, `_convolution`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`parse_from_str`, `graph`, `_convolution`。

### Lines 66-73
```cpp
    auto replace_pattern = [&](const PatternInfo& pattern_convolution) {
      const Graph& pattern_convolution_graph =
          *pattern_convolution.pattern_graph;
      const auto& convolution_vmap = pattern_convolution.vmap;

      const auto& matches =
          findPatternMatches(pattern_convolution_graph, *graph);
      for (const auto& match : matches) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `findPatternMatches`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`findPatternMatches`。

### Lines 74-81
```cpp
        // We come here only if the bias was not present in the module.
        // In that case, the corresponding graph will not have getAttr("bias")
        // Insert that in the graph.
        // And change _convolution to take the new value.
        auto conv_node =
            match.values_map.at(convolution_vmap.at("conv_out"))->node();
        WithInsertPoint ins(conv_node);
        Value* bias_attr_val = graph->insertGetAttr(graph->inputs()[0], "bias")
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `node`, `ins`, `insertGetAttr`, `inputs`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`node`, `ins`, `insertGetAttr`, `inputs`。

### Lines 82-91
```cpp
                                   ->setType(TensorType::get());
        constexpr size_t conv_bias_index = 2;
        conv_node->replaceInput(conv_bias_index, bias_attr_val);
      }
    };
    replace_pattern(pattern_convolution);
    replace_pattern(pattern_convolution_deprecated);
  }
}

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `setType`, `get`, `replaceInput`, `replace_pattern`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`setType`, `get`, `replaceInput`, `replace_pattern`。

### Lines 92-101
```cpp
void addBiasForConvIfNone(Module& module, const std::string& pattern_name) {
  auto t = module.type()->expect<ClassType>();

  const std::string real_typename = t->name()->qualifiedName();
  const std::string demangled_typename = removeTorchMangle(real_typename);
  bool is_floating_point_conv =
      ((demangled_typename == "__torch__.torch.nn.modules.conv.Conv1d") ||
       (demangled_typename == "__torch__.torch.nn.modules.conv.Conv2d") ||
       (demangled_typename == "__torch__.torch.nn.modules.conv.Conv3d"));

```
- EN: This block implements local helper logic for fold conv bn. Key symbols: `addBiasForConvIfNone`, `type`, `name`, `qualifiedName`, `removeTorchMangle`.
- CN: 该代码块实现与 fold conv bn 相关的局部辅助逻辑。关键符号：`addBiasForConvIfNone`, `type`, `name`, `qualifiedName`, `removeTorchMangle`。

### Lines 102-111
```cpp
  if (is_floating_point_conv) {
    if (!t->hasAttribute("bias")) {
      auto optional_tensor_type = OptionalType::create(TensorType::get());
      t->addAttribute("bias", std::move(optional_tensor_type), true);
      auto optional_tensor = std::optional<at::Tensor>();
      module.setattr("bias", std::move(optional_tensor));
      replaceConvBiasWithGetAttr(module);
    }
  }
  for (Module m : module.children()) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures. Key symbols: `hasAttribute`, `create`, `get`, `addAttribute`, `move`, `setattr`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构。关键符号：`hasAttribute`, `create`, `get`, `addAttribute`, `move`, `setattr`, `...`。

### Lines 112-124
```cpp
    addBiasForConvIfNone(m, pattern_name);
  }
}

class FoldConvBatchNormHelper {
 public:
  /**
   * In this step we find all Conv - BatchNorm patterns in the graph
   * and extract the corresponding parameters for these two modules,
   * and record information for the modifications of the graph without
   * actually performing these modifications.
   */
  void analyze(Module& module, const PatternInfo& pattern);
```
- EN: Declares core types or data containers for this file. Prominent symbols: `addBiasForConvIfNone`, `FoldConvBatchNormHelper`, `analyze`.
- CN: 声明该文件使用的核心类型或数据容器。主要符号：`addBiasForConvIfNone`, `FoldConvBatchNormHelper`, `analyze`。

### Lines 125-131
```cpp
  /**
   * In this step we perform all the modifications including
   * setting the attributes for conv module, rewriting values
   * and deleting nodes in the graph
   */
  void transform();

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `transform`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`transform`。

### Lines 132-138
```cpp
 private:
  bool tryExtractingConvBNParameters(
      Module& conv,
      Module& bn,
      ConvBNParameters& r);

  std::unordered_map<ModulePtr, std::tuple<at::Tensor, at::Tensor>>
```
- EN: This block implements local helper logic for fold conv bn. Key symbols: `tryExtractingConvBNParameters`.
- CN: 该代码块实现与 fold conv bn 相关的局部辅助逻辑。关键符号：`tryExtractingConvBNParameters`。

### Lines 139-151
```cpp
      conv_module_and_params_;

  // A map from graph to a list of tuple of paths of matched conv and bn module
  // e.g. if we have a graph `g` containing following code
  // x = self.sub.conv1(..)
  // x = self.sub.bn1(..)
  // x = self.sub.conv2(..)
  // x = self.sub.bn2(..)
  // then the value for graph `g` in this map will be:
  // [(['sub', 'conv1'], ['sub', 'bn1']), (['sub', 'conv2'], ['sub', 'bn2'])]
  // the first entry of the list is the paths to first conv-bn match
  // the second entry of the list is the path to second match
  std::unordered_map<
```
- EN: This block manipulates TorchScript IR objects. Key symbols: no dominant local symbol names.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：无明显局部符号。

### Lines 152-158
```cpp
      Graph*,
      std::vector<
          std::tuple<std::vector<std::string>, std::vector<std::string>>>>
      conv_bn_paths_;

  std::unordered_map<Value*, Value*> rewrite_map_;
  std::vector<Value*> values_to_rewrite_;
```
- EN: This block manipulates TorchScript IR objects. Key symbols: no dominant local symbol names.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：无明显局部符号。

### Lines 159-170
```cpp
  std::unordered_set<Node*> nodes_to_delete_;
};

bool extractOptionalBNParams(const script::Module& bn, ConvBNParameters& r) {
  auto bn_forward = bn.get_method("forward");
  auto graph = bn_forward.graph();
  const PatternInfo& pattern_bn = PatternInfo::parse_from_str(R"(
      graph(%a, %weight, %bias, %running_mean, %running_var,
          %training, %momentum, %eps, %cudnn_enabled):
        %bn_out = aten::batch_norm(%a, %weight, %bias, %running_mean,
            %running_var, %training, %momentum, %eps, %cudnn_enabled)
        return (%bn_out) )");
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `extractOptionalBNParams`, `get_method`, `graph`, `parse_from_str`, `batch_norm`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`extractOptionalBNParams`, `get_method`, `graph`, `parse_from_str`, `batch_norm`。

### Lines 171-177
```cpp
  const Graph& pattern_bn_graph = *pattern_bn.pattern_graph;
  const auto& bn_vmap = pattern_bn.vmap;

  const auto& matches = findPatternMatches(pattern_bn_graph, *graph);

  if (matches.size() > 1) {
    return false;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `findPatternMatches`, `size`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`findPatternMatches`, `size`。

### Lines 178-184
```cpp
  }

  if (bn.hasattr("eps")) {
    r.bn_eps = bn.attr("eps").toDouble();
  } else {
    auto optional_eps = toIValue(matches[0].values_map.at(bn_vmap.at("eps")));
    if (!optional_eps) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `hasattr`, `attr`, `toDouble`, `toIValue`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`hasattr`, `attr`, `toDouble`, `toIValue`。

### Lines 185-191
```cpp
      return false;
    }
    r.bn_eps = optional_eps.value().toDouble();
  }
  r.bn_w = at::ones_like(bn.attr("running_mean").toTensor());
  if (bn.hasattr("weight")) {
    if (bn.attr("weight").isTensor()) {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `value`, `toDouble`, `ones_like`, `attr`, `toTensor`, `hasattr`, `...`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`value`, `toDouble`, `ones_like`, `attr`, `toTensor`, `hasattr`, `...`。

### Lines 192-198
```cpp
      r.bn_w = bn.attr("weight").toTensor();
    }
  } else {
    auto optional_bn_weight =
        toIValue(matches[0].values_map.at(bn_vmap.at("weight")));
    if (!optional_bn_weight) {
      return false;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `attr`, `toTensor`, `toIValue`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`attr`, `toTensor`, `toIValue`。

### Lines 199-205
```cpp
    }
    if (optional_bn_weight.value().isTensor()) {
      r.bn_w = optional_bn_weight.value().toTensor();
    }
  }
  r.bn_b = at::zeros_like(bn.attr("running_mean").toTensor());
  if (bn.hasattr("bias")) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `value`, `isTensor`, `toTensor`, `zeros_like`, `attr`, `hasattr`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`value`, `isTensor`, `toTensor`, `zeros_like`, `attr`, `hasattr`。

### Lines 206-212
```cpp
    if (bn.attr("bias").isTensor()) {
      r.bn_b = bn.attr("bias").toTensor();
    }
  } else {
    auto optional_bn_bias =
        toIValue(matches[0].values_map.at(bn_vmap.at("bias")));
    if (!optional_bn_bias) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `attr`, `isTensor`, `toTensor`, `toIValue`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`attr`, `isTensor`, `toTensor`, `toIValue`。

### Lines 213-220
```cpp
      return false;
    }

    if (optional_bn_bias.value().isTensor()) {
      r.bn_b = optional_bn_bias.value().toTensor();
    }
  }
  return true;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `value`, `isTensor`, `toTensor`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`value`, `isTensor`, `toTensor`。

### Lines 221-227
```cpp
}

bool FoldConvBatchNormHelper::tryExtractingConvBNParameters(
    Module& conv,
    Module& bn,
    ConvBNParameters& r) {
  if (!hastensor(conv, "weight") || !conv.hasattr("bias") ||
```
- EN: This block handles conditional branches; performs optimization-oriented rewriting. Key symbols: `tryExtractingConvBNParameters`, `hastensor`, `hasattr`.
- CN: 该代码块处理条件分支；执行面向优化的改写。关键符号：`tryExtractingConvBNParameters`, `hastensor`, `hasattr`。

### Lines 228-234
```cpp
      !hastensor(bn, "running_mean") || !hastensor(bn, "running_var")) {
    return false;
  }

  r.bn_rm = bn.attr("running_mean").toTensor();
  r.bn_rv = bn.attr("running_var").toTensor();
  if (!extractOptionalBNParams(bn, r)) {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value. Key symbols: `hastensor`, `attr`, `toTensor`, `extractOptionalBNParams`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值。关键符号：`hastensor`, `attr`, `toTensor`, `extractOptionalBNParams`。

### Lines 235-241
```cpp
    return false;
  }

  r.conv_w = conv.attr("weight").toTensor();
  r.conv_b = at::zeros_like(r.bn_rm);
  auto bias_opt = conv.attr("bias").toOptional<at::Tensor>();
  if (bias_opt) {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value. Key symbols: `attr`, `toTensor`, `zeros_like`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值。关键符号：`attr`, `toTensor`, `zeros_like`。

### Lines 242-248
```cpp
    r.conv_b = *bias_opt;
  }

  return true;
}

void FoldConvBatchNormHelper::analyze(
```
- EN: This block produces a result or forwards a computed value; performs optimization-oriented rewriting. Key symbols: `analyze`.
- CN: 该代码块返回结果或转发已计算的值；执行面向优化的改写。关键符号：`analyze`。

### Lines 249-255
```cpp
    Module& module,
    const PatternInfo& pattern) {
  const Graph& pattern_graph = *pattern.pattern_graph;
  const auto& vmap = pattern.vmap;
  Value* pattern_conv_out = vmap.at("conv_out");
  Value* pattern_bn_out = vmap.at("bn_out");
  Value* pattern_bn_submodule = vmap.at("batchnorm");
```
- EN: This block manipulates TorchScript IR objects. Key symbols: no dominant local symbol names.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：无明显局部符号。

### Lines 256-262
```cpp
  Node* pattern_conv = pattern_conv_out->node();
  Node* pattern_bn = pattern_bn_out->node();

  // We will put submodules into this worklist and keep processing items from it
  // one by one. We start by just putting the top module there.
  std::stack<Module> worklist({module});
  while (!worklist.empty()) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `node`, `worklist`, `empty`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`node`, `worklist`, `empty`。

### Lines 263-270
```cpp
    Module current = worklist.top();
    worklist.pop();

    // Queue submodules for processing
    for (const Module& submodule : current.children()) {
      worklist.push(submodule);
    }

```
- EN: This block iterates over collections or graph structures. Key symbols: `top`, `pop`, `children`, `push`.
- CN: 该代码块遍历集合或图结构。关键符号：`top`, `pop`, `children`, `push`。

### Lines 271-278
```cpp
    // Process all method of the current module
    for (auto& method : current.get_methods()) {
      GRAPH_DUMP(
          current.type()->name()->name() + "::" + method.name() +
              "() before Conv-BatchNorm folding",
          method.graph());
      const auto& matches = findPatternMatches(pattern_graph, *method.graph());

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `get_methods`, `type`, `name`, `graph`, `findPatternMatches`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`get_methods`, `type`, `name`, `graph`, `findPatternMatches`。

### Lines 279-285
```cpp
      GRAPH_DEBUG("number of Conv-BatchNorm matches: ", matches.size());
      Graph* g = method.graph().get();
      if (!conv_bn_paths_.count(g)) {
        // This is to make sure we don't visit one graph multiple times
        conv_bn_paths_[g] = {};
        for (const Match& match : matches) {
          if (!std::all_of(
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `size`, `graph`, `get`, `count`, `all_of`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`size`, `graph`, `get`, `count`, `all_of`。

### Lines 286-293
```cpp
                  pattern.filters.begin(),
                  pattern.filters.end(),
                  [&](const MatchFilter& f) { return f(match, vmap); })) {
            continue;
          }
          GRAPH_DEBUG("Checking next match...");
          // Get the conv and bn submodule
          Node* matched_conv = match.nodes_map.at(pattern_conv);
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `begin`, `end`, `f`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`begin`, `end`, `f`。

### Lines 294-300
```cpp
          Node* matched_bn = match.nodes_map.at(pattern_bn);
          Node* matched_bn_submodule =
              match.values_map.at(pattern_bn_submodule)->node();
          Value* conv_instance = matched_conv->input(0);
          Value* bn_instance = matched_bn->input(0);
          Value* self = g->inputs()[0];
          auto conv_module_path = getModuleAccessPath(conv_instance, self);
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `node`, `input`, `inputs`, `getModuleAccessPath`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`node`, `input`, `inputs`, `getModuleAccessPath`。

### Lines 301-314
```cpp
          auto bn_module_path = getModuleAccessPath(bn_instance, self);
          Module conv_submodule = findChildModule(current, conv_module_path);
          Module bn_submodule = findChildModule(current, bn_module_path);

          ConvBNParameters params;
          if (!tryExtractingConvBNParameters(
                  conv_submodule, bn_submodule, params)) {
            GRAPH_DEBUG(
                "Conv and BN modules didn't have all required parameters or attributes...");
            continue;
          }
          conv_bn_paths_[g].emplace_back(conv_module_path, bn_module_path);
          // We are using a separate vector for saving Values we want to rewrite
          // to make sure that the order in which we perform these
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `getModuleAccessPath`, `findChildModule`, `tryExtractingConvBNParameters`, `emplace_back`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`getModuleAccessPath`, `findChildModule`, `tryExtractingConvBNParameters`, `emplace_back`。

### Lines 315-325
```cpp
          // transformations is deterministic. Iterating through keys of
          // rewrite_map would result in non-determinism that might not manifest
          // as a bug now, but can bite us later.
          values_to_rewrite_.push_back(matched_bn->output());
          rewrite_map_[matched_bn->output()] = matched_conv->output();
          GRAPH_UPDATE(
              "Rewriting %",
              matched_bn->output()->debugName(),
              " with %",
              matched_conv->output()->debugName());

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `push_back`, `output`, `debugName`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`push_back`, `output`, `debugName`。

### Lines 326-337
```cpp
          nodes_to_delete_.insert(matched_bn);
          nodes_to_delete_.insert(matched_bn_submodule);
          GRAPH_UPDATE("Deleting ", *matched_bn);
          GRAPH_UPDATE("Deleting ", *matched_bn_submodule);

          auto slot = conv_submodule.type()->getAttributeSlot("bias");
          TORCH_CHECK(
              conv_submodule.type()->is_parameter(slot),
              "Expected conv module to have a bias parameter");
        } // matches
      }

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `insert`, `type`, `getAttributeSlot`, `is_parameter`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`insert`, `type`, `getAttributeSlot`, `is_parameter`。

### Lines 338-345
```cpp
      for (const auto& conv_bn : conv_bn_paths_.at(g)) {
        Module conv_submodule = findChildModule(current, std::get<0>(conv_bn));
        Module bn_submodule = findChildModule(current, std::get<1>(conv_bn));

        ConvBNParameters params;
        TORCH_INTERNAL_ASSERT(tryExtractingConvBNParameters(
            conv_submodule, bn_submodule, params));
        auto new_w_b = computeUpdatedConvWeightAndBias(params);
```
- EN: This block iterates over collections or graph structures. Key symbols: `findChildModule`, `tryExtractingConvBNParameters`, `computeUpdatedConvWeightAndBias`.
- CN: 该代码块遍历集合或图结构。关键符号：`findChildModule`, `tryExtractingConvBNParameters`, `computeUpdatedConvWeightAndBias`。

### Lines 346-352
```cpp
        conv_module_and_params_[conv_submodule._ivalue()] = new_w_b;
      } // conv_bn module
    } // methods
  } // while
}

void FoldConvBatchNormHelper::transform() {
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `_ivalue`, `transform`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`_ivalue`, `transform`。

### Lines 353-359
```cpp
  for (const auto& item : conv_module_and_params_) {
    Module conv(item.first);
    auto w_b = item.second;
    conv.setattr("weight", std::get<0>(w_b));
    conv.setattr("bias", std::get<1>(w_b));
  }

```
- EN: This block iterates over collections or graph structures. Key symbols: `conv`, `setattr`.
- CN: 该代码块遍历集合或图结构。关键符号：`conv`, `setattr`。

### Lines 360-366
```cpp
  // Perform planned rewritings
  for (auto v : values_to_rewrite_) {
    v->replaceAllUsesWith(rewrite_map_.at(v));
  }

  // Perform planned deletions
  for (auto n : nodes_to_delete_) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `replaceAllUsesWith`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`replaceAllUsesWith`。

### Lines 367-373
```cpp
    n->removeAllInputs();
  }
  for (auto n : nodes_to_delete_) {
    n->destroy();
  }
}

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `removeAllInputs`, `destroy`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`removeAllInputs`, `destroy`。

### Lines 374-387
```cpp
} // namespace

Module FoldConvBatchNorm(const Module& module) {
  Module m = module.clone();

  addBiasForConvIfNone(m, "Conv2d");
  addBiasForConvIfNone(m, "Conv3d");
  // Conv2d + BatchNorm2d
  const PatternInfo pattern2d = PatternInfo::parse_from_str(
      R"(
graph(%self, %input, %conv, %batchnorm):
    %conv_out = prim::CallMethod[name="forward"](%conv, %input)
    %bn_out = prim::CallMethod[name="forward"](%batchnorm, %conv_out)
    return (%bn_out))",
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; advances frontend parsing or lowering state; performs optimization-oriented rewriting. Key symbols: `FoldConvBatchNorm`, `clone`, `addBiasForConvIfNone`, `parse_from_str`, `graph`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；推进前端解析或降级状态；执行面向优化的改写。关键符号：`FoldConvBatchNorm`, `clone`, `addBiasForConvIfNone`, `parse_from_str`, `graph`。

### Lines 388-395
```cpp
      {is_conv2d_module, is_batchnorm2d_module});
  // Conv3d + BatchNorm3d
  const PatternInfo pattern3d = PatternInfo::parse_from_str(
      R"(
graph(%self, %input, %conv, %batchnorm):
    %conv_out = prim::CallMethod[name="forward"](%conv, %input)
    %bn_out = prim::CallMethod[name="forward"](%batchnorm, %conv_out)
    return (%bn_out))",
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `parse_from_str`, `graph`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`parse_from_str`, `graph`。

### Lines 396-405
```cpp
      {is_conv3d_module, is_batchnorm3d_module});

  const std::vector<std::reference_wrapper<const PatternInfo>> patterns = {
      pattern2d, pattern3d};
  for (const auto& pattern : patterns) {
    FoldConvBatchNormHelper h;
    h.analyze(m, pattern);
    h.transform();
  }
  return m;
```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value; performs optimization-oriented rewriting. Key symbols: `analyze`, `transform`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值；执行面向优化的改写。关键符号：`analyze`, `transform`。

### Lines 406-408
```cpp
}

} // namespace torch::jit
```
- EN: Opens or closes namespace scopes so the remaining declarations live in the expected JIT/frontend module.
- CN: 打开或关闭命名空间作用域，使后续声明归属于预期的 JIT/前端模块。


## Key Concepts / 关键概念
- EN: IR graph manipulation — the file inspects or rewrites TorchScript graphs and nodes.
  CN: IR 图操作——该文件会检查或改写 TorchScript 的图、节点与块。
- EN: Parsing pipeline — the logic translates source text into TorchScript syntax structures.
  CN: 解析流水线——相关逻辑将源码文本转换为 TorchScript 语法结构。
- EN: Optimization pass — the code simplifies graphs by removing redundant work or folding values.
  CN: 优化 pass——代码通过删除冗余计算或常量折叠来简化图。
- EN: Fusion support — the file prepares groups of operations for fused execution.
  CN: 融合支持——该文件会为融合执行准备算子分组。
- EN: Type and shape reasoning — the code propagates static information across the IR.
  CN: 类型与形状推理——代码会在 IR 中传播静态信息。
- EN: Subsystem integration — the implementation collaborates with nearby JIT/frontend headers and runtime helpers.
  CN: 子系统集成——实现会与附近的 JIT/前端头文件以及运行时辅助模块协同工作。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `torch/csrc/jit/passes/fold_conv_bn.h`, `torch/csrc/jit/ir/subgraph_matcher.h`, `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/graph_rewrite_helper.h`, `torch/csrc/jit/passes/quantization/helper.h`, `ATen/TensorOperators.h`, `ATen/Functions.h`, `ATen/ops/ones_like.h`, `ATen/ops/rsqrt.h`, `ATen/ops/zeros_like.h`
- External includes / 外部头文件: `stack`, `utility`
- Namespaces / 命名空间: `torch::jit`
- Representative symbols / 代表性符号: `computeUpdatedConvWeightAndBias`, `rsqrt`, `dim`, `sizes`, `dtype`, `reshape`, `make_tuple`, `to`, `graph_rewrite_helper`, `hastensor`, `...`
