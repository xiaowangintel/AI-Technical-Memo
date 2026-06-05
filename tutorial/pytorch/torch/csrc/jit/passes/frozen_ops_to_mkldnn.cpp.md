# frozen_ops_to_mkldnn.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/passes/frozen_ops_to_mkldnn.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements the TorchScript JIT pass logic for frozen ops to mkldnn, including graph analysis and rewrites.
- 用途 (CN): 实现与 frozen ops to mkldnn 相关的 TorchScript JIT pass 逻辑，包括图分析与改写。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10
```cpp
#include <ATen/ATen.h>
#include <ATen/Config.h>
#include <ATen/NativeFunctions.h>
#include <ATen/Utils.h>
#include <ATen/core/symbol.h>
#include <ATen/native/layer_norm.h>
#include <c10/core/ScalarType.h>
#include <c10/util/Exception.h>
#include <c10/util/irange.h>

```
- EN: Pulls in the headers needed by the frozen ops to mkldnn logic. Internal dependencies: `ATen/ATen.h`, `ATen/Config.h`, `ATen/NativeFunctions.h`, `ATen/Utils.h`, `ATen/core/symbol.h`, `...`; external dependencies: none.
- CN: 为 frozen ops to mkldnn 相关逻辑引入所需头文件。内部依赖：`ATen/ATen.h`, `ATen/Config.h`, `ATen/NativeFunctions.h`, `ATen/Utils.h`, `ATen/core/symbol.h`, `...`；外部依赖：无。

### Lines 11-28
```cpp
#include <torch/csrc/jit/ir/alias_analysis.h>
#include <torch/csrc/jit/ir/constants.h>
#include <torch/csrc/jit/ir/ir.h>
#include <torch/csrc/jit/jit_log.h>
#include <torch/csrc/jit/passes/common_subexpression_elimination.h>
#include <torch/csrc/jit/passes/constant_propagation.h>
#include <torch/csrc/jit/passes/dead_code_elimination.h>
#include <torch/csrc/jit/passes/fold_conv_bn.h>
#include <torch/csrc/jit/passes/frozen_conv_folding.h>
#include <torch/csrc/jit/passes/frozen_ops_to_mkldnn.h>
#include <torch/csrc/jit/passes/graph_rewrite_helper.h>
#include <torch/csrc/jit/passes/peephole.h>
#include <torch/csrc/jit/passes/remove_mutation.h>
#include <torch/csrc/jit/passes/utils/subgraph_utils.h>
#include <torch/csrc/jit/runtime/custom_operator.h>
#include <torch/csrc/jit/runtime/operator_options.h>
#include <torch/csrc/jit/tensorexpr/types.h>
// clang-format off
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: no dominant local symbol names.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：无明显局部符号。

### Lines 29-44
```cpp
// moving ConvUtils include induces import cycle
#include <ATen/native/ConvUtils.h>
#include <algorithm>
#include <memory>
#include <ATen/core/stack.h>
#include <c10/core/Layout.h>
#include <c10/util/StringUtil.h>

#if AT_MKLDNN_ENABLED()
#include <ATen/CPUFunctions.h>
#include <dnnl_types.h>
#include <ATen/native/mkldnn/Utils.h>
#include <ATen/native/mkldnn/MKLDNNCommon.h>
#include <ideep.hpp>
#endif

```
- EN: This block implements local helper logic for frozen ops to mkldnn. Key symbols: no dominant local symbol names.
- CN: 该代码块实现与 frozen ops to mkldnn 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 45-53
```cpp
// clang-format on

namespace torch::jit {

#if AT_MKLDNN_ENABLED()

using Tensor = at::Tensor;

namespace {
```
- EN: This block implements local helper logic for frozen ops to mkldnn. Key symbols: `Tensor`.
- CN: 该代码块实现与 frozen ops to mkldnn 相关的局部辅助逻辑。关键符号：`Tensor`。

### Lines 54-62
```cpp

c10::AliasAnalysisKind aliasAnalysisFromSchema() {
  return AliasAnalysisKind::FROM_SCHEMA;
}

using ValueSet = std::unordered_set<Value*>;
using ValueSetPtr = std::shared_ptr<std::unordered_set<Value*>>;

Node* getLastUse(Value* v) {
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `aliasAnalysisFromSchema`, `ValueSet`, `ValueSetPtr`, `getLastUse`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`aliasAnalysisFromSchema`, `ValueSet`, `ValueSetPtr`, `getLastUse`。

### Lines 63-71
```cpp
  auto last_use_node = v->node();
  for (const auto& use : v->uses()) {
    if (use.user->isAfter(last_use_node)) {
      last_use_node = use.user;
    }
  }
  return last_use_node;
}

```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `node`, `uses`, `isAfter`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`node`, `uses`, `isAfter`。

### Lines 72-80
```cpp
void merge_sets(
    std::unordered_map<Value*, ValueSetPtr>& alias_mapping,
    Value* existing,
    Value* new_v) {
  if (alias_mapping[existing] == alias_mapping[new_v]) {
    return;
  }
  auto existing_set = alias_mapping[existing];
  auto set_to_remove = alias_mapping[new_v];
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `merge_sets`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`merge_sets`。

### Lines 81-89
```cpp
  for (auto it = set_to_remove->begin(); it != set_to_remove->end(); it++) {
    existing_set->insert(*it);
    alias_mapping[*it] = existing_set;
  }
}

// no uses of tensors in container types
void assertNonTensorTypeDoesNotContainTensors(const TypePtr& type) {
  if (type->cast<TensorType>()) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures. Key symbols: `begin`, `end`, `insert`, `assertNonTensorTypeDoesNotContainTensors`.
- CN: 该代码块处理条件分支；遍历集合或图结构。关键符号：`begin`, `end`, `insert`, `assertNonTensorTypeDoesNotContainTensors`。

### Lines 90-107
```cpp
    return;
  }
  for (const auto& t : type->containedTypes()) {
    TORCH_INTERNAL_ASSERT(!t->cast<TensorType>());
  }
}

void InplaceMKLDNNSubgraph(const std::shared_ptr<Graph>& graph) {
  // This function first calculates aliasing sets,
  // then calculates the last node each aliasing set is alive for.
  // Then we go through each node, if it's a node which has an equivalent
  // inplace node and the aliasing set for its input is dead after this node, we
  // inplace it. Then we merge the aliasing sets for the input and output of the
  // node and extend the liveness of the set. To inplace a node you need to
  // prove device and dtype of the input and output are the same, which we've
  // already done, and prove that the output size is the same as the input size,
  // which is achieved by explicit Broadcast nodes (which we inserted for other
  // reasons).
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `containedTypes`, `InplaceMKLDNNSubgraph`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`containedTypes`, `InplaceMKLDNNSubgraph`。

### Lines 108-116
```cpp
  // The graphs here are simple subgraphs without uses of Tensors in
  // containers (Lists, GetAttrs, etc)

  // CALCULATE ALIASING SETS

  auto aliasDb = std::make_unique<AliasDb>(graph);

  // map from Value to its Aliasing Set
  std::unordered_map<Value*, ValueSetPtr> alias_mapping;
```
- EN: This block manipulates TorchScript IR objects. Key symbols: no dominant local symbol names.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：无明显局部符号。

### Lines 117-127
```cpp
  ValueSet set;
  ValueSetPtr input_set = std::make_shared<ValueSet>(set);
  for (Value* v : graph->inputs()) {
    if (v->type()->cast<TensorType>()) {
      input_set->insert(v);
      alias_mapping[v] = input_set;
    } else {
      assertNonTensorTypeDoesNotContainTensors(v->type());
    }
  }

```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `inputs`, `type`, `insert`, `assertNonTensorTypeDoesNotContainTensors`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`inputs`, `type`, `insert`, `assertNonTensorTypeDoesNotContainTensors`。

### Lines 128-137
```cpp
  for (Node* n : graph->nodes()) {
    for (Value* output : n->outputs()) {
      if (!output->type()->cast<TensorType>()) {
        assertNonTensorTypeDoesNotContainTensors(output->type());
        continue;
      }

      std::unordered_set<Value*> new_set = {output};
      alias_mapping[output] = std::make_shared<ValueSet>(new_set);
      for (Value* input : n->inputs()) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `nodes`, `outputs`, `type`, `assertNonTensorTypeDoesNotContainTensors`, `inputs`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`nodes`, `outputs`, `type`, `assertNonTensorTypeDoesNotContainTensors`, `inputs`。

### Lines 138-146
```cpp
        if (aliasDb->mayAlias(input, output)) {
          merge_sets(alias_mapping, input, output);
        }
      }
    }
  }

  // CALCULATE ALIASING SET LIVENESS

```
- EN: This block handles conditional branches. Key symbols: `mayAlias`, `merge_sets`.
- CN: 该代码块处理条件分支。关键符号：`mayAlias`, `merge_sets`。

### Lines 147-155
```cpp
  // map from aliased set -> last use of set
  std::unordered_map<ValueSetPtr, Node*> set_liveness;
  for (auto& set : alias_mapping) {
    if (set_liveness.count(set.second)) {
      continue;
    }
    Node* last = nullptr;
    for (const auto& v : *set.second) {
      auto k = v->node()->kind();
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `count`, `node`, `kind`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`count`, `node`, `kind`。

### Lines 156-169
```cpp
      if (k == prim::Constant || k == prim::ConstantMKLDNNTensor ||
          k == prim::Param) {
        last = graph->return_node();
        continue;
      }

      auto last_use = getLastUse(v);
      if (!last || last_use->isAfter(last)) {
        last = last_use;
      }
    }
    set_liveness[set.second] = last;
  }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `return_node`, `getLastUse`, `isAfter`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`return_node`, `getLastUse`, `isAfter`。

### Lines 170-178
```cpp
  // REUSING MEMORY BY REINPLACING NODES
  std::vector<Node*> nodes_to_inplace;

  auto add_to_inplace_set = [&](Node* node) {
    // defer making the inplacing change because that would invalidate the old
    // Node output Value*
    nodes_to_inplace.push_back(node);
    TORCH_INTERNAL_ASSERT(node->outputs().size() == 1);
    auto output_liveness_end =
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `push_back`, `outputs`, `size`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`push_back`, `outputs`, `size`。

### Lines 179-191
```cpp
        set_liveness[alias_mapping[node->outputs().at(0)]];
    merge_sets(alias_mapping, node->inputs().at(0), node->output());
    set_liveness[alias_mapping[node->output()]] = output_liveness_end;
  };

  for (Node* node : graph->nodes()) {
    auto k = node->kind();
    if (k == aten::relu || k == aten::sigmoid || k == aten::dropout ||
        k == prim::MKLDNNHardSwish || k == prim::MKLDNNHardSigmoid ||
        k == prim::MKLDNNHardTanh || k == aten::tanh ||
        k == prim::MKLDNNClamp || k == Symbol::prim("MKLDNNScalarMul") ||
        k == Symbol::prim("MKLDNNLayerNorm")) {
      if (set_liveness[alias_mapping[node->inputs().at(0)]]->isAfter(node)) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `outputs`, `merge_sets`, `inputs`, `output`, `nodes`, `kind`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`outputs`, `merge_sets`, `inputs`, `output`, `nodes`, `kind`, `...`。

### Lines 192-201
```cpp
        continue;
      }
      add_to_inplace_set(node);
    } else if (k == aten::mul || k == aten::add) {
      // the binary operators (add/mul) are commutative and only take tensor
      // inputs, so we can inplace either the first or second input
      int64_t reusable_value_index = -1;
      for (const auto i : c10::irange(2)) {
        TORCH_INTERNAL_ASSERT(node->inputs().at(i)->type()->cast<TensorType>());
        if (!set_liveness[alias_mapping[node->inputs().at(i)]]->isAfter(node)) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `add_to_inplace_set`, `irange`, `inputs`, `type`, `isAfter`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`add_to_inplace_set`, `irange`, `inputs`, `type`, `isAfter`。

### Lines 202-210
```cpp
          reusable_value_index = i;
          break;
        }
      }

      if (reusable_value_index == -1) {
        continue;
      }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: no dominant local symbol names.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：无明显局部符号。

### Lines 211-219
```cpp
      if (reusable_value_index == 1) {
        node->insertInput(0, node->inputs().at(1));
        node->removeInput(2);
      }
      add_to_inplace_set(node);
    }
  }

  for (Node* node : nodes_to_inplace) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `insertInput`, `inputs`, `removeInput`, `add_to_inplace_set`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`insertInput`, `inputs`, `removeInput`, `add_to_inplace_set`。

### Lines 220-237
```cpp
    node->replaceWithNewSymbol(
        Symbol::fromQualString(node->schema().name() + "_"));
    node->destroy();
  }
}

// This is a factory function that creates an Operation that takes
// MKLDNN tensors and unpacks them into 1D contiguous tensors that we can
// run aten operations on. The precondition for using this function is that the
// aten operations in `aten_op` should be an identity for zero inputs. In other
// words, this should: `aten_op(0) = 0` The reason for this precondition has to
// do with blocked formats MKLDNN uses to lay tensor elements (nChw8c, nChw16c,
// etc). It splits the channel dimension into chunks of 8/16 makes it the
// innermost dimension. Whenever the channel dim isn't divisible by 8/16 the
// innermost dimension is padded with 0s. The precondition, `aten_op(0) == 0`
// allows us to avoid any special casing of padded elements.
Operation createUnaryOp(
    const std::function<void(at::Tensor output, at::Tensor input)>& aten_op,
```
- EN: This block manipulates TorchScript IR objects; advances frontend parsing or lowering state; protects shared state or ordering assumptions. Key symbols: `replaceWithNewSymbol`, `fromQualString`, `schema`, `name`, `destroy`, `createUnaryOp`, `...`.
- CN: 该代码块操作 TorchScript IR 对象；推进前端解析或降级状态；保护共享状态或执行顺序假设。关键符号：`replaceWithNewSymbol`, `fromQualString`, `schema`, `name`, `destroy`, `createUnaryOp`, `...`。

### Lines 238-246
```cpp
    bool inplace = false) {
  return [aten_op, inplace](Stack& stack) {
    auto a = pop(stack).toTensor();
    c10::impl::ExcludeDispatchKeyGuard edkg(c10::autograd_dispatch_keyset);
    // we cast `a` to an `ideep::tensor`, so we can get at its descriptor
    // which we then use to set up `out` tensor w/ the same props as a
    auto a_it = at::native::itensor_from_mkldnn(a);
    auto mkldnn_raw_data = a_it.get_data_handle();
    auto a_options_with_strided = a.options().layout(c10::kStrided);
```
- EN: This block produces a result or forwards a computed value; protects shared state or ordering assumptions. Key symbols: `pop`, `toTensor`, `edkg`, `itensor_from_mkldnn`, `get_data_handle`, `options`, `...`.
- CN: 该代码块返回结果或转发已计算的值；保护共享状态或执行顺序假设。关键符号：`pop`, `toTensor`, `edkg`, `itensor_from_mkldnn`, `get_data_handle`, `options`, `...`。

### Lines 247-256
```cpp

    // tensor's physical size could be bigger than a logical one
    // `a_it.get_desc().get_size()` returns the real physical size (in bytes)
    // we use it to compute `nelem` for `aten` ops
    auto nelem = static_cast<int64_t>(
        a_it.get_desc().get_size() / elementSize(a.scalar_type()));
    // we also wrap `a` storage into an aten tensor
    auto in_aten =
        at::from_blob(mkldnn_raw_data, {nelem}, a_options_with_strided);

```
- EN: This block implements local helper logic for frozen ops to mkldnn. Key symbols: `get_desc`, `get_size`, `elementSize`, `scalar_type`, `from_blob`.
- CN: 该代码块实现与 frozen ops to mkldnn 相关的局部辅助逻辑。关键符号：`get_desc`, `get_size`, `elementSize`, `scalar_type`, `from_blob`。

### Lines 257-265
```cpp
    auto out_raw_data = mkldnn_raw_data;
    auto out = a;
    if (!inplace) {
      // `a_it.get_desc()` will allocate a tensor
      // of the right physical size.
      auto it_empty = ideep::tensor(a_it.get_desc());
      TORCH_INTERNAL_ASSERT(it_empty.get_desc() == a_it.get_desc());
      out = at::native::new_with_itensor_mkldnn(
          std::move(it_empty),
```
- EN: This block handles conditional branches. Key symbols: `tensor`, `get_desc`, `new_with_itensor_mkldnn`, `move`.
- CN: 该代码块处理条件分支。关键符号：`tensor`, `get_desc`, `new_with_itensor_mkldnn`, `move`。

### Lines 266-274
```cpp
          c10::optTypeMetaToScalarType(a.options().dtype_opt()),
          a.options().device_opt());

      out_raw_data = at::native::itensor_from_mkldnn(out).get_data_handle();
    }

    TORCH_INTERNAL_ASSERT(
        a_it.get_desc().get_size() % elementSize(a.scalar_type()) == 0);

```
- EN: This block implements local helper logic for frozen ops to mkldnn. Key symbols: `optTypeMetaToScalarType`, `options`, `dtype_opt`, `device_opt`, `itensor_from_mkldnn`, `get_data_handle`, `...`.
- CN: 该代码块实现与 frozen ops to mkldnn 相关的局部辅助逻辑。关键符号：`optTypeMetaToScalarType`, `options`, `dtype_opt`, `device_opt`, `itensor_from_mkldnn`, `get_data_handle`, `...`。

### Lines 275-283
```cpp
    auto out_aten = at::from_blob(out_raw_data, nelem, a_options_with_strided);
    aten_op(out_aten, in_aten);
    push(stack, out);
  };
}

void MKLDNNLayerNormOp(Stack& stack, bool inplace) {
  c10::impl::ExcludeDispatchKeyGuard edkg(c10::autograd_dispatch_keyset);

```
- EN: This block protects shared state or ordering assumptions. Key symbols: `from_blob`, `aten_op`, `push`, `MKLDNNLayerNormOp`, `edkg`.
- CN: 该代码块保护共享状态或执行顺序假设。关键符号：`from_blob`, `aten_op`, `push`, `MKLDNNLayerNormOp`, `edkg`。

### Lines 284-293
```cpp
  // enable_cudnn not used
  pop(stack);
  auto eps = pop(stack).toDouble();

  Tensor bias{};
  Tensor weight{};
  auto bias_ival = pop(stack);
  TORCH_INTERNAL_ASSERT(bias_ival.isTensor());
  bias = bias_ival.toTensor();

```
- EN: This block implements local helper logic for frozen ops to mkldnn. Key symbols: `pop`, `toDouble`, `isTensor`, `toTensor`.
- CN: 该代码块实现与 frozen ops to mkldnn 相关的局部辅助逻辑。关键符号：`pop`, `toDouble`, `isTensor`, `toTensor`。

### Lines 294-306
```cpp
  auto weight_ival = pop(stack);
  TORCH_INTERNAL_ASSERT(weight_ival.isTensor());
  weight = weight_ival.toTensor();

  auto shape = pop(stack).toDimVector();
  auto input = pop(stack).toTensor();

  auto [dst, mean, rstd] =
      at::native::mkldnn_layer_norm_last_index_weight_bias_f32(
          input, shape, weight, bias, eps, inplace);
  push(stack, dst);
}

```
- EN: This block implements local helper logic for frozen ops to mkldnn. Key symbols: `pop`, `isTensor`, `toTensor`, `toDimVector`, `mkldnn_layer_norm_last_index_weight_bias_f32`, `push`.
- CN: 该代码块实现与 frozen ops to mkldnn 相关的局部辅助逻辑。关键符号：`pop`, `isTensor`, `toTensor`, `toDimVector`, `mkldnn_layer_norm_last_index_weight_bias_f32`, `push`。

### Lines 307-319
```cpp
Operation BroadOp(const Node* node) {
  return [](Stack& stack) {
    auto b = pop(stack).toTensor();
    auto a = pop(stack).toTensor();
    auto b_size = b.sizes();
    auto a_size = a.sizes();
    if (a_size.equals(b_size)) {
      // TODO: follow up with MKLDNN what the best way is
      // to handle perf incompatible formats
      push(stack, a, b);
      return;
    } else {
      auto out_size = at::infer_size(a_size, b_size);
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `BroadOp`, `pop`, `toTensor`, `sizes`, `equals`, `push`, `...`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`BroadOp`, `pop`, `toTensor`, `sizes`, `equals`, `push`, `...`。

### Lines 320-329
```cpp
      int64_t out_numel = out_size[0];
      for (size_t i = 1, end = out_size.size(); i < end; ++i) {
        out_numel = out_numel * out_size[i];
      }

      auto exp_a = a;
      auto exp_b = b;
      int stacked = 0;
      // mkldnn tensors only support reshape, not expand or view operators
      if (a_size.equals(out_size)) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures. Key symbols: `size`, `equals`.
- CN: 该代码块处理条件分支；遍历集合或图结构。关键符号：`size`, `equals`。

### Lines 330-339
```cpp
        push(stack, a);
        ++stacked;
      } else if (out_numel == a.numel()) {
        exp_a = a.reshape(out_size);
      } else {
        // TODO: consider to initializing to a blocked layout
        // directly if needed
        exp_a = a.to_dense().expand(out_size).to_mkldnn();
      }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `push`, `numel`, `reshape`, `to_dense`, `expand`, `to_mkldnn`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`push`, `numel`, `reshape`, `to_dense`, `expand`, `to_mkldnn`。

### Lines 340-348
```cpp
      if (b_size.equals(out_size)) {
        push(stack, b);
        ++stacked;
      } else if (out_numel == b.numel()) {
        exp_b = b.reshape(out_size);
      } else {
        exp_b = b.to_dense().expand(out_size).to_mkldnn();
      }

```
- EN: This block handles conditional branches. Key symbols: `equals`, `push`, `numel`, `reshape`, `to_dense`, `expand`, `...`.
- CN: 该代码块处理条件分支。关键符号：`equals`, `push`, `numel`, `reshape`, `to_dense`, `expand`, `...`。

### Lines 349-361
```cpp
      if (stacked < 2) {
        if (stacked == 1) {
          pop(stack);
        }
        // If one of the inputs was expanded and converted to nchw/nhwc
        // we might end up in a very bad spot if the second argument
        // is in a blocked format. In this case, MKLDNN uses its
        // reference implementation for a binary operation that follows
        // these broadcasts and it could be up to ~100x slower.
        // We use a very simple heuristic to convert an arg in nchw
        // to the blocked format of the other argument.
        c10::impl::ExcludeDispatchKeyGuard edkg(c10::autograd_dispatch_keyset);
        auto a_it = at::native::itensor_from_mkldnn(exp_a);
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `pop`, `edkg`, `itensor_from_mkldnn`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`pop`, `edkg`, `itensor_from_mkldnn`。

### Lines 362-371
```cpp
        auto b_it = at::native::itensor_from_mkldnn(exp_b);

        // `is_public_format` means a tensor's physical layout isn't in MKLDNN
        // blocked layout e.g. nchw or nhwc but not nChw8c
        if (!a_it.is_public_format()) {
          if (b_it.is_public_format()) {
            b_it = b_it.reorder_if_differ_in(a_it.get_desc());
          }
        } else if (!b_it.is_public_format()) {
          if (a_it.is_public_format()) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `itensor_from_mkldnn`, `is_public_format`, `reorder_if_differ_in`, `get_desc`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`itensor_from_mkldnn`, `is_public_format`, `reorder_if_differ_in`, `get_desc`。

### Lines 372-382
```cpp
            a_it = a_it.reorder_if_differ_in(b_it.get_desc());
          }
        }

        auto a_options = exp_a.options();
        auto a_out = at::native::new_with_itensor_mkldnn(
            std::move(a_it),
            c10::optTypeMetaToScalarType(a_options.dtype_opt()),
            a_options.device_opt());
        push(stack, a_out);
        auto b_options = exp_b.options();
```
- EN: This block implements local helper logic for frozen ops to mkldnn. Key symbols: `reorder_if_differ_in`, `get_desc`, `options`, `new_with_itensor_mkldnn`, `move`, `optTypeMetaToScalarType`, `...`.
- CN: 该代码块实现与 frozen ops to mkldnn 相关的局部辅助逻辑。关键符号：`reorder_if_differ_in`, `get_desc`, `options`, `new_with_itensor_mkldnn`, `move`, `optTypeMetaToScalarType`, `...`。

### Lines 383-392
```cpp
        auto b_out = at::native::new_with_itensor_mkldnn(
            std::move(b_it),
            c10::optTypeMetaToScalarType(b_options.dtype_opt()),
            b_options.device_opt());
        push(stack, b_out);
      };
    }
  };
}

```
- EN: This block implements local helper logic for frozen ops to mkldnn. Key symbols: `new_with_itensor_mkldnn`, `move`, `optTypeMetaToScalarType`, `dtype_opt`, `device_opt`, `push`.
- CN: 该代码块实现与 frozen ops to mkldnn 相关的局部辅助逻辑。关键符号：`new_with_itensor_mkldnn`, `move`, `optTypeMetaToScalarType`, `dtype_opt`, `device_opt`, `push`。

### Lines 393-401
```cpp
static std::function<void(at::Tensor output, at::Tensor input)> hardtanh_helper(
    const Node* n) {
  auto min_val = n->f(attr::min_val);
  auto max_val = n->f(attr::max_val);
  return [min_val, max_val](at::Tensor output, const at::Tensor& input) {
    at::cpu::hardtanh_out(output, input, min_val, max_val);
  };
}

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `void`, `hardtanh_helper`, `f`, `hardtanh_out`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`void`, `hardtanh_helper`, `f`, `hardtanh_out`。

### Lines 402-410
```cpp
static std::function<void(at::Tensor output, at::Tensor input)> clamp_helper(
    const Node* n) {
  auto min_val = n->f(attr::min_val);
  auto max_val = n->f(attr::max_val);
  return [min_val, max_val](at::Tensor output, const at::Tensor& input) {
    at::cpu::clamp_out(output, input, min_val, max_val);
  };
}

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `void`, `clamp_helper`, `f`, `clamp_out`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`void`, `clamp_helper`, `f`, `clamp_out`。

### Lines 411-428
```cpp
// any op added to this registry needs to meet
// the precondition: `aten_op(0) == 0`
const RegisterOperators MKLDNNHardSwishOpReg({
    torch::jit::Operator(
        "prim::MKLDNNHardSwish_(Tensor(a!) self) -> Tensor(a!)",
        createUnaryOp(
            [](at::Tensor output, const at::Tensor& input) {
              at::cpu::hardswish_out(output, input);
            },
            true),
        AliasAnalysisKind::FROM_SCHEMA),
    torch::jit::Operator(
        "prim::MKLDNNHardSigmoid_(Tensor(a!) self) -> Tensor(a!)",
        createUnaryOp(
            [](at::Tensor output, const at::Tensor& input) {
              at::cpu::hardsigmoid_out(output, input);
            },
            true),
```
- EN: Registers callable entities, pass hooks, or exported interfaces so runtime code can discover frozen ops to mkldnn behavior. Symbols: `MKLDNNHardSwishOpReg`, `Operator`, `MKLDNNHardSwish_`, `Tensor`, `createUnaryOp`, `hardswish_out`, `...`.
- CN: 注册可调用实体、pass 钩子或导出接口，使运行时能够发现 frozen ops to mkldnn 的行为。符号：`MKLDNNHardSwishOpReg`, `Operator`, `MKLDNNHardSwish_`, `Tensor`, `createUnaryOp`, `hardswish_out`, `...`。

### Lines 429-439
```cpp
        AliasAnalysisKind::FROM_SCHEMA),
    torch::jit::Operator(
        "prim::MKLDNNHardTanh_(Tensor(a!) self) -> Tensor(a!)",
        [](const Node* n) -> Operation {
          return createUnaryOp(hardtanh_helper(n), true);
        },
        AliasAnalysisKind::FROM_SCHEMA),
    torch::jit::Operator(
        "prim::MKLDNNClamp_(Tensor(a!) self) -> Tensor(a!)",
        [](const Node* n) -> Operation {
          return createUnaryOp(clamp_helper(n), true);
```
- EN: Registers callable entities, pass hooks, or exported interfaces so runtime code can discover frozen ops to mkldnn behavior. Symbols: `Operator`, `MKLDNNHardTanh_`, `Tensor`, `createUnaryOp`, `hardtanh_helper`, `MKLDNNClamp_`, `...`.
- CN: 注册可调用实体、pass 钩子或导出接口，使运行时能够发现 frozen ops to mkldnn 的行为。符号：`Operator`, `MKLDNNHardTanh_`, `Tensor`, `createUnaryOp`, `hardtanh_helper`, `MKLDNNClamp_`, `...`。

### Lines 440-457
```cpp
        },
        AliasAnalysisKind::FROM_SCHEMA),
    torch::jit::Operator(
        "prim::MKLDNNHardSwish(Tensor a) -> Tensor",
        createUnaryOp(
            [](at::Tensor output, const at::Tensor& input) {
              at::cpu::hardswish_out(output, input);
            },
            false),
        AliasAnalysisKind::FROM_SCHEMA),
    torch::jit::Operator(
        "prim::MKLDNNHardSigmoid(Tensor a) -> Tensor",
        createUnaryOp(
            [](at::Tensor output, const at::Tensor& input) {
              at::cpu::hardsigmoid_out(output, input);
            },
            false),
        AliasAnalysisKind::FROM_SCHEMA),
```
- EN: Registers callable entities, pass hooks, or exported interfaces so runtime code can discover frozen ops to mkldnn behavior. Symbols: `Operator`, `MKLDNNHardSwish`, `createUnaryOp`, `hardswish_out`, `MKLDNNHardSigmoid`, `hardsigmoid_out`.
- CN: 注册可调用实体、pass 钩子或导出接口，使运行时能够发现 frozen ops to mkldnn 的行为。符号：`Operator`, `MKLDNNHardSwish`, `createUnaryOp`, `hardswish_out`, `MKLDNNHardSigmoid`, `hardsigmoid_out`。

### Lines 458-467
```cpp
    torch::jit::Operator(
        "prim::MKLDNNHardTanh(Tensor self) -> Tensor",
        [](const Node* n) -> Operation {
          return createUnaryOp(hardtanh_helper(n), false);
        },
        AliasAnalysisKind::FROM_SCHEMA),
    torch::jit::Operator(
        "prim::MKLDNNClamp(Tensor self) -> Tensor",
        [](const Node* n) -> Operation {
          return createUnaryOp(clamp_helper(n), false);
```
- EN: Registers callable entities, pass hooks, or exported interfaces so runtime code can discover frozen ops to mkldnn behavior. Symbols: `Operator`, `MKLDNNHardTanh`, `createUnaryOp`, `hardtanh_helper`, `MKLDNNClamp`, `clamp_helper`.
- CN: 注册可调用实体、pass 钩子或导出接口，使运行时能够发现 frozen ops to mkldnn 的行为。符号：`Operator`, `MKLDNNHardTanh`, `createUnaryOp`, `hardtanh_helper`, `MKLDNNClamp`, `clamp_helper`。

### Lines 468-478
```cpp
        },
        AliasAnalysisKind::FROM_SCHEMA),
});

const RegisterOperators BroadOpReg({
    torch::jit::Operator(
        prim::BroadcastMKLDNNTensors,
        BroadOp,
        AliasAnalysisKind::INTERNAL_SPECIAL_CASE),
});

```
- EN: Registers callable entities, pass hooks, or exported interfaces so runtime code can discover frozen ops to mkldnn behavior. Symbols: `BroadOpReg`, `Operator`.
- CN: 注册可调用实体、pass 钩子或导出接口，使运行时能够发现 frozen ops to mkldnn 的行为。符号：`BroadOpReg`, `Operator`。

### Lines 479-489
```cpp
const RegisterOperators MKLDNNLayerNormOpReg({
    torch::jit::Operator(
        "prim::MKLDNNLayerNorm(Tensor input, int[] normalized_shape, Tensor? weight=None, Tensor? bias=None, float eps=1e-05, bool cudnn_enable=True) -> Tensor",
        [](Stack& stack) { MKLDNNLayerNormOp(stack, false); },
        AliasAnalysisKind::FROM_SCHEMA),
    torch::jit::Operator(
        "prim::MKLDNNLayerNorm_(Tensor(a!) input, int[] normalized_shape, Tensor? weight=None, Tensor? bias=None, float eps=1e-05, bool cudnn_enable=True) -> Tensor(a!)",
        [](Stack& stack) { MKLDNNLayerNormOp(stack, true); },
        AliasAnalysisKind::FROM_SCHEMA),
});

```
- EN: Registers callable entities, pass hooks, or exported interfaces so runtime code can discover frozen ops to mkldnn behavior. Symbols: `MKLDNNLayerNormOpReg`, `Operator`, `MKLDNNLayerNorm`, `MKLDNNLayerNormOp`, `MKLDNNLayerNorm_`, `Tensor`.
- CN: 注册可调用实体、pass 钩子或导出接口，使运行时能够发现 frozen ops to mkldnn 的行为。符号：`MKLDNNLayerNormOpReg`, `Operator`, `MKLDNNLayerNorm`, `MKLDNNLayerNormOp`, `MKLDNNLayerNorm_`, `Tensor`。

### Lines 490-507
```cpp
Operation ConstantMKLDNNTensorOp(const Node* node) {
  const auto& t = node->t(attr::value);
  return [t](Stack& stack) {
    push(stack, t);
    return 0;
  };
}

Tensor mkldnn_tensor_scalar_mul(Tensor& tensor, Tensor& out, float scalar) {
  ideep::tensor& x = at::native::itensor_from_mkldnn(tensor);
  ideep::tensor& z = at::native::itensor_from_mkldnn(out);
  ideep::eltwise_forward::compute(
      x,
      z,
      ideep::algorithm::eltwise_linear,
      ideep::prop_kind::forward_inference,
      /*alpha*/ scalar);
  return out;
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `ConstantMKLDNNTensorOp`, `t`, `push`, `mkldnn_tensor_scalar_mul`, `itensor_from_mkldnn`, `compute`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`ConstantMKLDNNTensorOp`, `t`, `push`, `mkldnn_tensor_scalar_mul`, `itensor_from_mkldnn`, `compute`。

### Lines 508-522
```cpp
}

// aten::convolution does a lot of precomputation and dispatching before
// mkldnn_convolution is called. registering here we can directly invoke the op
// and avoid overhead. avoiding dispatch overhead for other operators - relu,
// add, etc - did not benchmark as speeding up models noticeably. the additional
// overhead of `convolution` warrants the custom operator.
jit::RegisterOperators reg_fut_ops({
    jit::Operator(
        // XXX: this follows the schema convention of conv2d/conv3d, not
        // aten::mkldnn_convolution, which is different for some reason!
        "prim::mkldnn_convolution(Tensor input, Tensor weight, Tensor? bias, int[] stride, int[] padding, int[] dilation, int groups) -> Tensor",
        [](jit::Stack& stack) {
          int64_t groups = pop(stack).toInt();
          auto dilation = pop(stack).toIntVector();
```
- EN: Registers callable entities, pass hooks, or exported interfaces so runtime code can discover frozen ops to mkldnn behavior. Symbols: `reg_fut_ops`, `Operator`, `mkldnn_convolution`, `pop`, `toInt`, `toIntVector`.
- CN: 注册可调用实体、pass 钩子或导出接口，使运行时能够发现 frozen ops to mkldnn 的行为。符号：`reg_fut_ops`, `Operator`, `mkldnn_convolution`, `pop`, `toInt`, `toIntVector`。

### Lines 523-533
```cpp
          auto padding = pop(stack).toIntVector();
          auto stride = pop(stack).toIntVector();

          Tensor bias;
          IValue bias_ival = pop(stack);
          if (!bias_ival.isNone()) {
            bias = bias_ival.toTensor();
          }
          Tensor weight = pop(stack).toTensor();
          Tensor input = pop(stack).toTensor();

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `pop`, `toIntVector`, `isNone`, `toTensor`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`pop`, `toIntVector`, `isNone`, `toTensor`。

### Lines 534-551
```cpp
          at::AutoDispatchBelowAutograd mode;
          // aten::convolution takes care of 0 dim case before calls into
          // backends
          if (input.size(0) == 0) {
            std::vector<int64_t> o = at::native::conv_output_size(
                input.sizes(), weight.sizes(), padding, stride, dilation);
            push(
                stack,
                at::native::empty_mkldnn(
                    o,
                    c10::optTypeMetaToScalarType(input.options().dtype_opt()),
                    input.options().layout_opt(),
                    input.options().device_opt(),
                    input.options().pinned_memory_opt()));
            return;
          }
          // aten::convolution also checks dtype mismatches
          TORCH_CHECK(
```
- EN: This block handles conditional branches. Key symbols: `size`, `conv_output_size`, `sizes`, `push`, `empty_mkldnn`, `optTypeMetaToScalarType`, `...`.
- CN: 该代码块处理条件分支。关键符号：`size`, `conv_output_size`, `sizes`, `push`, `empty_mkldnn`, `optTypeMetaToScalarType`, `...`。

### Lines 552-569
```cpp
              input.options().type_equal(weight.options()),
              "Input type (",
              input.toString(),
              ") and weight type (",
              weight.toString(),
              ") should be the same");

          push(
              stack,
              at::native::mkldnn_convolution(
                  input, weight, bias, padding, stride, dilation, groups));
        },
        aliasAnalysisFromSchema()),
    // registering as custom operators avoids Scalar->Tensor->Scalar conversion
    // in default bindings
    jit::Operator(
        "prim::MKLDNNScalarMul(Tensor self, Scalar other) -> Tensor",
        [](jit::Stack& stack) {
```
- EN: Registers callable entities, pass hooks, or exported interfaces so runtime code can discover frozen ops to mkldnn behavior. Symbols: `options`, `type_equal`, `type`, `toString`, `push`, `mkldnn_convolution`, `...`.
- CN: 注册可调用实体、pass 钩子或导出接口，使运行时能够发现 frozen ops to mkldnn 的行为。符号：`options`, `type_equal`, `type`, `toString`, `push`, `mkldnn_convolution`, `...`。

### Lines 570-580
```cpp
          c10::impl::ExcludeDispatchKeyGuard edkg(
              c10::autograd_dispatch_keyset);
          float other = pop(stack).toScalar().toFloat();
          Tensor self = pop(stack).toTensor();
          auto out = at::native::empty_mkldnn(
              self.sizes(),
              c10::optTypeMetaToScalarType(self.options().dtype_opt()),
              self.options().layout_opt(),
              self.options().device_opt(),
              self.options().pinned_memory_opt());

```
- EN: This block protects shared state or ordering assumptions. Key symbols: `edkg`, `pop`, `toScalar`, `toFloat`, `toTensor`, `empty_mkldnn`, `...`.
- CN: 该代码块保护共享状态或执行顺序假设。关键符号：`edkg`, `pop`, `toScalar`, `toFloat`, `toTensor`, `empty_mkldnn`, `...`。

### Lines 581-597
```cpp
          mkldnn_tensor_scalar_mul(self, out, other);
          push(stack, out);
        },
        aliasAnalysisFromSchema()),
    jit::Operator(
        "prim::MKLDNNScalarMul_(Tensor(a!) self, Scalar other) -> Tensor(a!)",
        [](jit::Stack& stack) {
          c10::impl::ExcludeDispatchKeyGuard edkg(
              c10::autograd_dispatch_keyset);
          float other = pop(stack).toScalar().toFloat();
          Tensor self = pop(stack).toTensor();
          mkldnn_tensor_scalar_mul(self, self, other);
          push(stack, self);
        },
        aliasAnalysisFromSchema()),
});

```
- EN: Registers callable entities, pass hooks, or exported interfaces so runtime code can discover frozen ops to mkldnn behavior. Symbols: `mkldnn_tensor_scalar_mul`, `push`, `aliasAnalysisFromSchema`, `Operator`, `MKLDNNScalarMul_`, `Tensor`, `...`.
- CN: 注册可调用实体、pass 钩子或导出接口，使运行时能够发现 frozen ops to mkldnn 的行为。符号：`mkldnn_tensor_scalar_mul`, `push`, `aliasAnalysisFromSchema`, `Operator`, `MKLDNNScalarMul_`, `Tensor`, `...`。

### Lines 598-607
```cpp
// This is registered as its own op instead of as prim::Constant bc it does not
// serialize which is an invariant of prim::Constant
// TODO: make mkldnn tensor serialize...
const RegisterOperators MKLDNNConstantOp({
    torch::jit::Operator(
        prim::ConstantMKLDNNTensor,
        ConstantMKLDNNTensorOp,
        AliasAnalysisKind::INTERNAL_SPECIAL_CASE),
});

```
- EN: Registers callable entities, pass hooks, or exported interfaces so runtime code can discover frozen ops to mkldnn behavior. Symbols: `MKLDNNConstantOp`, `Operator`.
- CN: 注册可调用实体、pass 钩子或导出接口，使运行时能够发现 frozen ops to mkldnn 的行为。符号：`MKLDNNConstantOp`, `Operator`。

### Lines 608-616
```cpp
Node* createConstantMKLDNNTensorOp(Graph* g, const Tensor& mkldnn_tensor) {
  TORCH_INTERNAL_ASSERT(mkldnn_tensor.is_mkldnn());
  auto op = g->create(prim::ConstantMKLDNNTensor);
  op->t_(attr::value, mkldnn_tensor);
  return op;
}

bool supportedMKLDNNWeight(const Tensor& weight) {
  return weight.device().is_cpu() && weight.dtype() == c10::ScalarType::Float &&
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `createConstantMKLDNNTensorOp`, `is_mkldnn`, `create`, `t_`, `supportedMKLDNNWeight`, `device`, `...`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`createConstantMKLDNNTensorOp`, `is_mkldnn`, `create`, `t_`, `supportedMKLDNNWeight`, `device`, `...`。

### Lines 617-630
```cpp
      weight.ndimension() != 0;
}

void replaceInputWithMKLDNNTensor(Node* n, size_t index) {
  Value* input = n->inputs().at(index);
  auto mkldnn_tensor = constant_as<Tensor>(input)->to_mkldnn();
  auto mkldnn_tensor_value =
      createConstantMKLDNNTensorOp(n->owningGraph(), mkldnn_tensor)
          ->insertBefore(n)
          ->output();
  mkldnn_tensor_value->setDebugName(input->debugName() + "_mkldnn");
  n->replaceInputWith(input, mkldnn_tensor_value);
}

```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `ndimension`, `replaceInputWithMKLDNNTensor`, `inputs`, `to_mkldnn`, `createConstantMKLDNNTensorOp`, `owningGraph`, `...`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`ndimension`, `replaceInputWithMKLDNNTensor`, `inputs`, `to_mkldnn`, `createConstantMKLDNNTensorOp`, `owningGraph`, `...`。

### Lines 631-643
```cpp
void replaceInputWithMKLDNNTensor(
    Node* n,
    const std::string& name,
    const at::Tensor& mkldnn_tensor) {
  Value* input = n->namedInput(name);
  auto mkldnn_tensor_value =
      createConstantMKLDNNTensorOp(n->owningGraph(), mkldnn_tensor)
          ->insertBefore(n)
          ->output();
  mkldnn_tensor_value->setDebugName(input->debugName() + "_mkldnn");
  n->replaceInputWith(input, mkldnn_tensor_value);
}

```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `replaceInputWithMKLDNNTensor`, `namedInput`, `createConstantMKLDNNTensorOp`, `owningGraph`, `insertBefore`, `output`, `...`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`replaceInputWithMKLDNNTensor`, `namedInput`, `createConstantMKLDNNTensorOp`, `owningGraph`, `insertBefore`, `output`, `...`。

### Lines 644-653
```cpp
void replaceInputWithMKLDNNTensor(Node* n, const std::string& name) {
  Value* input = n->namedInput(name);
  auto mkldnn_tensor = constant_as<Tensor>(input)->to_mkldnn();
  replaceInputWithMKLDNNTensor(n, name, mkldnn_tensor);
}

void moveConvWeightsToMKLDNN(Node* conv) {
  auto conv_w_mkldnn =
      constant_as<Tensor>(conv->namedInput("weight")).value().to_mkldnn();
  std::vector<int64_t> padding =
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `replaceInputWithMKLDNNTensor`, `namedInput`, `to_mkldnn`, `moveConvWeightsToMKLDNN`, `value`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`replaceInputWithMKLDNNTensor`, `namedInput`, `to_mkldnn`, `moveConvWeightsToMKLDNN`, `value`。

### Lines 654-671
```cpp
      toIValue(conv->namedInput("padding"))->toIntVector();
  std::vector<int64_t> stride =
      toIValue(conv->namedInput("stride"))->toIntVector();
  std::vector<int64_t> dilation =
      toIValue(conv->namedInput("dilation"))->toIntVector();
  auto groups = constant_as<int64_t>(conv->namedInput("groups")).value();

  if (conv->kind() == aten::conv2d) {
    conv_w_mkldnn = mkldnn_reorder_conv2d_weight(
        conv_w_mkldnn, padding, stride, dilation, groups);
  } else if (conv->kind() == aten::conv3d) {
    conv_w_mkldnn = mkldnn_reorder_conv3d_weight(
        conv_w_mkldnn, padding, stride, dilation, groups);
  } else {
    TORCH_INTERNAL_ASSERT(false);
  }
  replaceInputWithMKLDNNTensor(conv, "weight", conv_w_mkldnn);

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `toIValue`, `namedInput`, `toIntVector`, `value`, `kind`, `mkldnn_reorder_conv2d_weight`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`toIValue`, `namedInput`, `toIntVector`, `value`, `kind`, `mkldnn_reorder_conv2d_weight`, `...`。

### Lines 672-680
```cpp
  if (conv->namedInput("bias")->type() != NoneType::get()) {
    replaceInputWithMKLDNNTensor(conv, "bias");
  }
}

void moveWeightsToMKLDNN(Node* n) {
  // conv goes through special pathway so we can call mkldnn reorder conv
  // primitive
  if (n->kind() == aten::conv2d || n->kind() == aten::conv3d) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `namedInput`, `type`, `get`, `replaceInputWithMKLDNNTensor`, `moveWeightsToMKLDNN`, `kind`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`namedInput`, `type`, `get`, `replaceInputWithMKLDNNTensor`, `moveWeightsToMKLDNN`, `kind`。

### Lines 681-692
```cpp
    moveConvWeightsToMKLDNN(n);
  } else {
    for (size_t i = 0; i < n->inputs().size(); ++i) {
      if (!n->input(i)->type()->cast<TensorType>() ||
          n->input(i)->node()->kind() != prim::Constant) {
        continue;
      }
      replaceInputWithMKLDNNTensor(n, i);
    }
  }
}

```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `moveConvWeightsToMKLDNN`, `inputs`, `size`, `input`, `type`, `node`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`moveConvWeightsToMKLDNN`, `inputs`, `size`, `input`, `type`, `node`, `...`。

### Lines 693-705
```cpp
static void clamp_node_creator(
    Node* body_node,
    c10::Symbol kind,
    double min_val,
    double max_val) {
  WithInsertPoint insert_guard{body_node};
  auto out_node =
      body_node->owningGraph()->create({kind}, {body_node->input(0)}, 1);
  // N.B. we can't use `insert` as it calls `getOperation` (via
  // `emitBuiltinCall`) which uses `min_val` and `max_val` attrs which we
  // haven't set yet.
  body_node->owningGraph()->insertNode(out_node);
  auto out_val = out_node->output();
```
- EN: This block manipulates TorchScript IR objects; advances frontend parsing or lowering state; protects shared state or ordering assumptions. Key symbols: `clamp_node_creator`, `owningGraph`, `create`, `input`, `insertNode`, `output`.
- CN: 该代码块操作 TorchScript IR 对象；推进前端解析或降级状态；保护共享状态或执行顺序假设。关键符号：`clamp_node_creator`, `owningGraph`, `create`, `input`, `insertNode`, `output`。

### Lines 706-714
```cpp
  out_node->f_(attr::min_val, min_val);
  out_node->f_(attr::max_val, max_val);
  out_val->copyMetadata(body_node->output());
  body_node->output()->replaceAllUsesWith(out_val);
  body_node->destroy();
}

void ComputeSubgraphInMKLDNN(Node* subgraph_node) {
  auto graph = subgraph_node->owningGraph();
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `f_`, `copyMetadata`, `output`, `replaceAllUsesWith`, `destroy`, `ComputeSubgraphInMKLDNN`, `...`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`f_`, `copyMetadata`, `output`, `replaceAllUsesWith`, `destroy`, `ComputeSubgraphInMKLDNN`, `...`。

### Lines 715-725
```cpp
  Value* none_value = nullptr;
  {
    WithInsertPoint guard(subgraph_node);
    none_value = graph->insertConstant(IValue());
  }
  for (size_t i = 0; i < subgraph_node->inputs().size(); ++i) {
    Value* v = subgraph_node->inputs().at(i);
    if (!v->type()->cast<TensorType>()) {
      continue;
    }
    auto to_mkldnn =
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `guard`, `insertConstant`, `IValue`, `inputs`, `size`, `type`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`guard`, `insertConstant`, `IValue`, `inputs`, `size`, `type`。

### Lines 726-734
```cpp
        graph->create(c10::Symbol::fromQualString("aten::to_mkldnn"), 1)
            ->insertBefore(subgraph_node);
    to_mkldnn->addInput(v);
    to_mkldnn->addInput(none_value);
    subgraph_node->replaceInput(i, to_mkldnn->output());
  }

  for (size_t i = 0; i < subgraph_node->outputs().size(); ++i) {
    Value* v = subgraph_node->outputs().at(i);
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `create`, `fromQualString`, `insertBefore`, `addInput`, `replaceInput`, `output`, `...`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`create`, `fromQualString`, `insertBefore`, `addInput`, `replaceInput`, `output`, `...`。

### Lines 735-745
```cpp
    if (!v->type()->cast<TensorType>()) {
      continue;
    }
    auto from_mkldnn = graph
                           ->create(
                               c10::Symbol::fromQualString("aten::to_dense"),
                               {v, none_value, none_value})
                           ->insertAfter(subgraph_node);
    v->replaceAllUsesAfterNodeWith(from_mkldnn, from_mkldnn->output());
  }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `type`, `create`, `fromQualString`, `insertAfter`, `replaceAllUsesAfterNodeWith`, `output`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`type`, `create`, `fromQualString`, `insertAfter`, `replaceAllUsesAfterNodeWith`, `output`。

### Lines 746-754
```cpp
  auto subgraph = SubgraphUtils::getSubgraph(subgraph_node);
  for (auto it = subgraph->block()->nodes().begin();
       it != subgraph->block()->nodes().end();) {
    Node* body_node = *it;
    it++;

    moveWeightsToMKLDNN(body_node);

    if (body_node->kind() == aten::add ||
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `getSubgraph`, `block`, `nodes`, `begin`, `end`, `moveWeightsToMKLDNN`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`getSubgraph`, `block`, `nodes`, `begin`, `end`, `moveWeightsToMKLDNN`, `...`。

### Lines 755-765
```cpp
        (body_node->kind() == aten::mul &&
         body_node->input(1)->type()->cast<TensorType>())) {
      auto node = body_node->owningGraph()->create(
          Symbol::prim("BroadcastMKLDNNTensors"),
          {body_node->inputs().at(0), body_node->inputs().at(1)},
          2);
      node->insertBefore(body_node);
      body_node->replaceInput(0, node->outputs().at(0));
      body_node->replaceInput(1, node->outputs().at(1));
    }
    if (body_node->kind() == aten::mul &&
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `kind`, `input`, `type`, `owningGraph`, `create`, `prim`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`kind`, `input`, `type`, `owningGraph`, `create`, `prim`, `...`。

### Lines 766-778
```cpp
        body_node->input(1)->type()->isSubtypeOf(*NumberType::get())) {
      body_node->replaceWithNewSymbol(Symbol::prim("MKLDNNScalarMul"));
      body_node->destroy();
      continue;
    }

    if (body_node->matches(
            "aten::layer_norm(Tensor input, int[] normalized_shape, Tensor? weight=None, Tensor? bias=None, float eps=1e-05, bool cudnn_enable=True) -> Tensor")) {
      body_node->replaceWithNewSymbol(Symbol::prim("MKLDNNLayerNorm"));
      body_node->destroy();
      continue;
    }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `input`, `type`, `isSubtypeOf`, `get`, `replaceWithNewSymbol`, `prim`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`input`, `type`, `isSubtypeOf`, `get`, `replaceWithNewSymbol`, `prim`, `...`。

### Lines 779-790
```cpp
    if (body_node->kind() == aten::hardswish) {
      body_node->replaceWithNewSymbol(prim::MKLDNNHardSwish);
      body_node->destroy();
      continue;
    }

    if (body_node->kind() == aten::hardsigmoid) {
      body_node->replaceWithNewSymbol(prim::MKLDNNHardSigmoid);
      body_node->destroy();
      continue;
    }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `kind`, `replaceWithNewSymbol`, `destroy`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`kind`, `replaceWithNewSymbol`, `destroy`。

### Lines 791-799
```cpp
    if (body_node->kind() == aten::relu6) {
      clamp_node_creator(body_node, prim::MKLDNNHardTanh, 0., 6.);
      continue;
    }

    if (body_node->kind() == aten::hardtanh) {
      auto min_val =
          constant_as<double>(body_node->namedInput("min_val")).value();
      auto max_val =
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `kind`, `clamp_node_creator`, `namedInput`, `value`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`kind`, `clamp_node_creator`, `namedInput`, `value`。

### Lines 800-811
```cpp
          constant_as<double>(body_node->namedInput("max_val")).value();
      clamp_node_creator(body_node, prim::MKLDNNHardTanh, min_val, max_val);
      continue;
    }

    if (body_node->kind() == aten::clamp) {
      auto min_val = constant_as<double>(body_node->namedInput("min")).value();
      auto max_val = constant_as<double>(body_node->namedInput("max")).value();
      clamp_node_creator(body_node, prim::MKLDNNClamp, min_val, max_val);
      continue;
    }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `namedInput`, `value`, `clamp_node_creator`, `kind`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`namedInput`, `value`, `clamp_node_creator`, `kind`。

### Lines 812-823
```cpp
    if (body_node->kind() == aten::conv2d ||
        body_node->kind() == aten::conv3d) {
      // this node doesn't handle string padding yet...
      if (!body_node->namedInput("padding")->type()->cast<StringType>()) {
        body_node->replaceWithNewSymbol(Symbol::prim("mkldnn_convolution"));
        body_node->destroy();
        continue;
      }
    }
  }
}

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `kind`, `namedInput`, `type`, `replaceWithNewSymbol`, `prim`, `destroy`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`kind`, `namedInput`, `type`, `replaceWithNewSymbol`, `prim`, `destroy`。

### Lines 824-832
```cpp
bool nonConstantParameters(Node* n) {
  for (size_t i = 1; i < n->inputs().size(); i++) {
    if (n->inputs().at(i)->node()->kind() != prim::Constant) {
      return true;
    }
  }
  return false;
}

```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `nonConstantParameters`, `inputs`, `size`, `node`, `kind`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`nonConstantParameters`, `inputs`, `size`, `node`, `kind`。

### Lines 833-841
```cpp
bool frozenMkldnnCompatibleLinearNode(Node* n) {
  if (nonConstantParameters(n)) {
    return false;
  }

  if (n->kind() != aten::linear) {
    return false;
  }

```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `frozenMkldnnCompatibleLinearNode`, `nonConstantParameters`, `kind`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`frozenMkldnnCompatibleLinearNode`, `nonConstantParameters`, `kind`。

### Lines 842-852
```cpp
  auto weight = constant_as<Tensor>(n->namedInput("weight")).value();
  return supportedMKLDNNWeight(weight);
}

bool frozenMkldnnCompatibleConvNode(Node* n) {
  if (nonConstantParameters(n)) {
    return false;
  }
  // mkldnn does not support conv1d
  // _convolution is rewritten before this pass is invoked
  if (n->kind() != aten::conv2d && n->kind() != aten::conv3d) {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `namedInput`, `value`, `supportedMKLDNNWeight`, `frozenMkldnnCompatibleConvNode`, `nonConstantParameters`, `kind`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`namedInput`, `value`, `supportedMKLDNNWeight`, `frozenMkldnnCompatibleConvNode`, `nonConstantParameters`, `kind`。

### Lines 853-870
```cpp
    return false;
  }

  auto weight = constant_as<Tensor>(n->namedInput("weight")).value();
  return supportedMKLDNNWeight(weight);
}

// [mkldnn perf strategy]
// Certain ops - aten::linear, aten::conv2d, aten::conv3d - provide a huge speed
// up just by converting the constant weights to MKLDNN AOT, and then at runtime
// converting the non-constant input to_mkldnn before the op, and then back to
// its original layout after the op. The speed up holds even if you end up
// converting the input to_mkldnn and output back to_dense. We start groups of
// ops to compute in MKLDNN only from these ops that are a strict speedup. Then,
// we expand the groups to include operators which are computable in MKLDNN &
// are roughly perf equal to eager. We do this in the hopes of joining multiple
// fast nodes together, saving to_mkldnn and to_dense conversions.
//
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `namedInput`, `value`, `supportedMKLDNNWeight`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`namedInput`, `value`, `supportedMKLDNNWeight`。

### Lines 871-880
```cpp
// MKLDNN only supports float32 inputs for aten::linear, aten::conv2d &
// aten::conv3d. We only fuse these nodes if the weights are float32, and then
// we only include operators which we can prove will execute in float32. By
// fusing topologically we can maintain the invariant that all tensor types in
// the graph are floating point. In fusing Conv-> Add -> Relu -> Conv we start
// with the first Conv, know that the output is float, and can then safely merge
// Add and Relu. If we started with the last Conv, it would be difficult to
// prove in our first pass that the Add's inputs were both float32 without first
// fusing the first conv.

```
- EN: Documents the intent, constraints, or compatibility notes that guide the surrounding implementation.
- CN: 说明周围实现所遵循的目标、限制条件或兼容性要求。

### Lines 881-889
```cpp
class MKLDNNSubgraphSlicer {
 public:
  MKLDNNSubgraphSlicer(
      Block* block,
      std::shared_ptr<Graph> graph,
      AliasDb& aliasDb)
      : block_(block), graph_(std::move(graph)), aliasDb_(aliasDb) {}

  void run() {
```
- EN: Declares core types or data containers for this file. Prominent symbols: `MKLDNNSubgraphSlicer`, `block_`, `graph_`, `move`, `aliasDb_`, `run`.
- CN: 声明该文件使用的核心类型或数据容器。主要符号：`MKLDNNSubgraphSlicer`, `block_`, `graph_`, `move`, `aliasDb_`, `run`。

### Lines 890-900
```cpp
    // We maintain alias db correctness in-place while building up the autodiff
    // subgraphs, however it is difficult to preserve correctness when
    // un-inlining autodiff subgraphs. We first recursively construct all
    // subgraphs and then unmerge them into the graph
    buildupSubgraphs();
    computeSubgraphsInMKLDNN();
    // Run CSE globally onceto eliminate duplicates that may have occurred
    // while inlining subgraphs.
    EliminateCommonSubexpression(graph_);
  }

```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `buildupSubgraphs`, `computeSubgraphsInMKLDNN`, `EliminateCommonSubexpression`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`buildupSubgraphs`, `computeSubgraphsInMKLDNN`, `EliminateCommonSubexpression`。

### Lines 901-916
```cpp
  void buildupSubgraphs() {
    // We need to run the slicer multiple times in order to get all merge
    // opportunities. This is because moveBeforeTopologicalValid may reorder
    // nodes to be AFTER the current iteration point. In order to properly
    // consider those nodes for merging, we need run the pass until no changes
    // have been made.
    //
    // Example:
    //   c = f(a, b)
    //   d = f(c)
    //   e = f(d)  <- iter is here, moving upward
    // After c.moveBeforeTopologicallyValid(e), we have:
    //   c = f(a, b)
    //   e = f(d)  <- iter still here
    //   d = f(c)  <- this was node moved on the other side.

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `buildupSubgraphs`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`buildupSubgraphs`。

### Lines 917-926
```cpp
    bool any_changed = true;
    while (any_changed) {
      any_changed = false;
      for (auto it = block_->nodes().begin(); it != block_->nodes().end();) {
        bool changed = false;
        std::tie(it, changed) = scanNode(*it);
        any_changed |= changed;
      }
    }

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `nodes`, `begin`, `end`, `tie`, `scanNode`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`nodes`, `begin`, `end`, `tie`, `scanNode`。

### Lines 927-935
```cpp
    // Construct Subgraphs Recursively
    for (Node* n : block_->nodes()) {
      for (auto subBlock : n->blocks()) {
        MKLDNNSubgraphSlicer(subBlock, graph_, aliasDb_).buildupSubgraphs();
      }
    }
  }

  static bool MKLDNNGroupStart(Node* node) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `nodes`, `blocks`, `MKLDNNSubgraphSlicer`, `buildupSubgraphs`, `MKLDNNGroupStart`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`nodes`, `blocks`, `MKLDNNSubgraphSlicer`, `buildupSubgraphs`, `MKLDNNGroupStart`。

### Lines 936-948
```cpp
    // if we're already in the process of merging
    if (node->kind() == prim::MKLDNNGroup) {
      return true;
    }
    // see [mkldnn perf strategy]
    return frozenMkldnnCompatibleConvNode(node);
  }

 private:
  // MKLDNN only supports floats of dimension > 0, so we only support
  // Tensors who have a known type or were previously verified
  // to be usable in an MKLDNN Group
  bool tensorInputIsMKLDNNSupported(Value* v, Node* v_use) {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `kind`, `frozenMkldnnCompatibleConvNode`, `tensorInputIsMKLDNNSupported`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`kind`, `frozenMkldnnCompatibleConvNode`, `tensorInputIsMKLDNNSupported`。

### Lines 949-958
```cpp
    auto const_tensor = constant_as<Tensor>(v);
    if (const_tensor) {
      return supportedMKLDNNWeight(*const_tensor);
    }
    auto k = v->node()->kind();
    if (k == prim::MKLDNNGroup || k == prim::ConstantMKLDNNTensor ||
        k == aten::to_mkldnn) {
      return true;
    }
    for (const auto& use : v->uses()) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `supportedMKLDNNWeight`, `node`, `kind`, `uses`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`supportedMKLDNNWeight`, `node`, `kind`, `uses`。

### Lines 959-969
```cpp
      if (use.user->kind() == aten::to_mkldnn &&
          v_use->owningBlock() == use.user->owningBlock()) {
        return true;
      }
    }
    return false;
  }

  // We include ops here which are roughly perf-equivalent in mkldnn as with
  // aten (single & multithreaded) and whose inputs & outputs are float32.
  bool computableInMKLDNN(Node* n) {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `kind`, `owningBlock`, `computableInMKLDNN`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`kind`, `owningBlock`, `computableInMKLDNN`。

### Lines 970-981
```cpp
    for (Value* v : n->inputs()) {
      if (v->type()->cast<TensorType>() &&
          !(tensorInputIsMKLDNNSupported(v, n))) {
        return false;
      }
    }

    if (n->matches(
            "aten::layer_norm(Tensor input, int[] normalized_shape, Tensor? weight=None, Tensor? bias=None, float eps=1e-05, bool cudnn_enable=True) -> Tensor") &&
        n->namedInput("weight")->type() != NoneType::get() &&
        n->namedInput("bias")->type() != NoneType::get()) {
      auto norm_shape =
```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `inputs`, `type`, `tensorInputIsMKLDNNSupported`, `matches`, `layer_norm`, `namedInput`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`inputs`, `type`, `tensorInputIsMKLDNNSupported`, `matches`, `layer_norm`, `namedInput`, `...`。

### Lines 982-999
```cpp
          constant_as<std::vector<int64_t>>(n->namedInput("normalized_shape"));
      return norm_shape.has_value() && norm_shape->size() == 1;
    }

    // unary ops we dont need to prove anything else than
    // the input is mkldnn supported
    switch (n->kind()) {
      case aten::relu:
      case aten::relu6:
      case aten::gelu:
      case aten::prelu:
      case aten::sigmoid:
      case aten::hardsigmoid:
      case aten::hardswish:
      case aten::tanh:
      case aten::batch_norm:
      case aten::max_pool2d:
      case aten::max_pool3d:
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `namedInput`, `has_value`, `size`, `kind`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`namedInput`, `has_value`, `size`, `kind`。

### Lines 1000-1008
```cpp
      case aten::avg_pool2d:
      case aten::adaptive_avg_pool2d:
      case aten::avg_pool3d:
        // case aten::adaptive_max_pool2d: // return tuples which break fusion
        // case aten::adaptive_max_pool3d: // return tuples which break fusion
        // case aten::adaptive_avg_pool3d: // no ideep binding
        return true;
    }

```
- EN: This block produces a result or forwards a computed value. Key symbols: no dominant local symbol names.
- CN: 该代码块返回结果或转发已计算的值。关键符号：无明显局部符号。

### Lines 1009-1017
```cpp
    if ((n->kind() == aten::hardtanh || n->kind() == aten::clamp) &&
        !nonConstantParameters(n)) {
      const size_t MIN_INDEX = 1, MAX_INDEX = 2;
      auto min_val = constant_as<double>(n->input(MIN_INDEX)).value();
      auto max_val = constant_as<double>(n->input(MAX_INDEX)).value();
      // we need to maintain the following invariant `pointwise_func(0) == 0`,
      // see `createUnaryOp`
      if (min_val <= 0. && max_val >= 0.) {
        return true;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `kind`, `nonConstantParameters`, `input`, `value`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`kind`, `nonConstantParameters`, `input`, `value`。

### Lines 1018-1028
```cpp
      }
    }

    if (n->kind() == aten::add) {
      // mkldnn doesn't currently support Tensor-Scalar add
      for (const auto i : c10::irange(2)) {
        if (!n->inputs().at(i)->type()->cast<TensorType>()) {
          return false;
        }
      }
      return true;
```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value. Key symbols: `kind`, `irange`, `inputs`, `type`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值。关键符号：`kind`, `irange`, `inputs`, `type`。

### Lines 1029-1037
```cpp
    }
    if (n->kind() == aten::mul) {
      return n->input(0)->type()->cast<TensorType>() &&
          (n->input(1)->type()->cast<TensorType>() ||
           n->input(1)->type()->isSubtypeOf(*NumberType::get()));
    }

    if (n->kind() == aten::dropout) {
      auto train = constant_as<bool>(n->namedInput("train")).value();
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `kind`, `input`, `type`, `isSubtypeOf`, `get`, `namedInput`, `...`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`kind`, `input`, `type`, `isSubtypeOf`, `get`, `namedInput`, `...`。

### Lines 1038-1046
```cpp
      return train == false;
    }
    return false;
  }

  void computeSubgraphsInMKLDNN() {
    auto curNode = *block_->nodes().begin();
    while (curNode != *block_->nodes().end()) {
      auto nextNode = curNode->next();
```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `computeSubgraphsInMKLDNN`, `nodes`, `begin`, `end`, `next`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`computeSubgraphsInMKLDNN`, `nodes`, `begin`, `end`, `next`。

### Lines 1047-1055
```cpp
      if (curNode->kind() == prim::MKLDNNGroup) {
        ComputeSubgraphInMKLDNN(curNode);
        InplaceMKLDNNSubgraph(SubgraphUtils::getSubgraph(curNode));
        SubgraphUtils::unmergeSubgraph(curNode);
      }
      curNode = nextNode;
    }
    for (Node* n : block_->nodes()) {
      for (Block* b : n->blocks()) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `kind`, `ComputeSubgraphInMKLDNN`, `InplaceMKLDNNSubgraph`, `getSubgraph`, `unmergeSubgraph`, `nodes`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`kind`, `ComputeSubgraphInMKLDNN`, `InplaceMKLDNNSubgraph`, `getSubgraph`, `unmergeSubgraph`, `nodes`, `...`。

### Lines 1056-1064
```cpp
        MKLDNNSubgraphSlicer(b, graph_, aliasDb_).computeSubgraphsInMKLDNN();
      }
    }
  }

  bool shouldConsiderForMerge(Node* node) {
    // if we're already in the process of merging
    if (node->kind() == prim::MKLDNNGroup) {
      return true;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `MKLDNNSubgraphSlicer`, `computeSubgraphsInMKLDNN`, `shouldConsiderForMerge`, `kind`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`MKLDNNSubgraphSlicer`, `computeSubgraphsInMKLDNN`, `shouldConsiderForMerge`, `kind`。

### Lines 1065-1076
```cpp
    }
    return frozenMkldnnCompatibleLinearNode(node) ||
        frozenMkldnnCompatibleConvNode(node) || computableInMKLDNN(node);
  }

  std::pair<graph_node_list::iterator, bool> scanNode(Node* producer) {
    if (MKLDNNGroupStart(producer)) {
      if (producer->kind() != prim::MKLDNNGroup) {
        producer = SubgraphUtils::createSingletonSubgraphAndUpdateAliasing(
            producer, prim::MKLDNNGroup, aliasDb_);
      }
      std::vector<Node*> output_nodes;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `frozenMkldnnCompatibleLinearNode`, `frozenMkldnnCompatibleConvNode`, `computableInMKLDNN`, `scanNode`, `MKLDNNGroupStart`, `kind`, `...`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`frozenMkldnnCompatibleLinearNode`, `frozenMkldnnCompatibleConvNode`, `computableInMKLDNN`, `scanNode`, `MKLDNNGroupStart`, `kind`, `...`。

### Lines 1077-1086
```cpp
      for (Value* v : producer->outputs()) {
        for (const auto& use : v->uses()) {
          output_nodes.push_back(use.user);
        }
      }
      std::sort(
          output_nodes.begin(), output_nodes.end(), [&](Node* a, Node* b) {
            return a->isBefore(b);
          });
      for (auto output_node : output_nodes) {
```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `outputs`, `uses`, `push_back`, `sort`, `begin`, `end`, `...`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`outputs`, `uses`, `push_back`, `sort`, `begin`, `end`, `...`。

### Lines 1087-1095
```cpp
        if (auto group = tryMerge(producer, output_node)) {
          // we successfully merged, so the new group's `outputs` may have
          // changed. So rescan the new group for more merging opportunities.
          return std::make_pair(group.value()->iterator()++, true);
        }
      }
    }

    return std::make_pair(++producer->iterator(), false);
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `tryMerge`, `make_pair`, `value`, `iterator`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`tryMerge`, `make_pair`, `value`, `iterator`。

### Lines 1096-1104
```cpp
  }

  // Try to merge `consumer` into `producer`. If successful, this destroys
  // `consumer` and returns the `producer` group.
  std::optional<Node*> tryMerge(Node* producer, Node* consumer) {
    AT_ASSERT(producer->kind() == prim::MKLDNNGroup);
    bool canMerge = shouldConsiderForMerge(consumer) &&
        aliasDb_.moveAfterTopologicallyValid(consumer, producer);

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `tryMerge`, `kind`, `shouldConsiderForMerge`, `moveAfterTopologicallyValid`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`tryMerge`, `kind`, `shouldConsiderForMerge`, `moveAfterTopologicallyValid`。

### Lines 1105-1114
```cpp
    if (!canMerge) {
      return std::nullopt;
    }

    SubgraphUtils::mergeNodeIntoSubgraphAndUpdateAliasing(
        consumer, producer, aliasDb_);

    return producer;
  }

```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `mergeNodeIntoSubgraphAndUpdateAliasing`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`mergeNodeIntoSubgraphAndUpdateAliasing`。

### Lines 1115-1123
```cpp
  Block* block_;
  std::shared_ptr<Graph> graph_;
  AliasDb& aliasDb_;
};

bool containsMKLDNNGroup(Block* b) {
  for (Node* n : b->nodes()) {
    for (Block* block : n->blocks()) {
      if (containsMKLDNNGroup(block)) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `containsMKLDNNGroup`, `nodes`, `blocks`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`containsMKLDNNGroup`, `nodes`, `blocks`。

### Lines 1124-1133
```cpp
        return true;
      }
    }
    if (MKLDNNSubgraphSlicer::MKLDNNGroupStart(n)) {
      return true;
    }
  }
  return false;
}

```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `MKLDNNGroupStart`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`MKLDNNGroupStart`。

### Lines 1134-1146
```cpp
} // namespace

void ConvertFrozenOpsToMKLDNN(std::shared_ptr<Graph>& graph) {
  GRAPH_DUMP("Before convert frozen ops to mkldnn", graph);
  // TODO: replace conv1d with conv2d ?
  graph_rewrite_helper::replaceConvolutionWithAtenConv(graph);
  if (containsMKLDNNGroup(graph->block())) {
    // Only remove tensor mutation if we know we're going to create speedups
    // with mkldnn. Only supporting functional ops simplifies this pass bc
    // running an op in mkldnn removes the aliasing relationships that
    // previously existed between input and output.
    RemoveTensorMutation(graph, [](Node* node_to_functionalize) {
      static std::unordered_set<Symbol> mkldnn_ops = {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `ConvertFrozenOpsToMKLDNN`, `replaceConvolutionWithAtenConv`, `containsMKLDNNGroup`, `block`, `RemoveTensorMutation`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`ConvertFrozenOpsToMKLDNN`, `replaceConvolutionWithAtenConv`, `containsMKLDNNGroup`, `block`, `RemoveTensorMutation`。

### Lines 1147-1159
```cpp
          aten::add_,
          aten::mul_,
          aten::relu_,
          aten::relu6_,
          aten::gelu_,
          aten::hardswish_,
          aten::dropout_,
          aten::sigmoid_,
          aten::hardsigmoid_,
          aten::hardtanh_,
          aten::tanh_,
          aten::clamp_,
      };
```
- EN: This block implements local helper logic for frozen ops to mkldnn. Key symbols: no dominant local symbol names.
- CN: 该代码块实现与 frozen ops to mkldnn 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 1160-1171
```cpp
      return mkldnn_ops.count(node_to_functionalize->kind()) != 0;
    });

    AliasDb db(graph);
    MKLDNNSubgraphSlicer(graph->block(), graph, db).run();
    EliminateDeadCode(graph);
    GRAPH_DUMP("After convert frozen ops to mkldnn", graph);
  } else {
    GRAPH_DUMP("No mkldnn compatible frozen nodes", graph);
  }
}

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `count`, `kind`, `db`, `MKLDNNSubgraphSlicer`, `block`, `run`, `...`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`count`, `kind`, `db`, `MKLDNNSubgraphSlicer`, `block`, `run`, `...`。

### Lines 1172-1180
```cpp
#else

void ConvertFrozenOpsToMKLDNN(std::shared_ptr<Graph>& graph) {
  GRAPH_DUMP("MKLDNN Not enabled", graph);
}

#endif

} // namespace torch::jit
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `ConvertFrozenOpsToMKLDNN`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`ConvertFrozenOpsToMKLDNN`。


## Key Concepts / 关键概念
- EN: IR graph manipulation — the file inspects or rewrites TorchScript graphs and nodes.
  CN: IR 图操作——该文件会检查或改写 TorchScript 的图、节点与块。
- EN: Alias and mutation reasoning — the code tracks side effects and write/read interactions.
  CN: 别名与变更分析——代码会跟踪副作用以及读写关系。
- EN: Schema handling — the implementation parses or normalizes operator/function signatures.
  CN: 模式处理——实现会解析或规范化算子/函数签名。
- EN: Optimization pass — the code simplifies graphs by removing redundant work or folding values.
  CN: 优化 pass——代码通过删除冗余计算或常量折叠来简化图。
- EN: Fusion support — the file prepares groups of operations for fused execution.
  CN: 融合支持——该文件会为融合执行准备算子分组。
- EN: Type and shape reasoning — the code propagates static information across the IR.
  CN: 类型与形状推理——代码会在 IR 中传播静态信息。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `ATen/ATen.h`, `ATen/Config.h`, `ATen/NativeFunctions.h`, `ATen/Utils.h`, `ATen/core/symbol.h`, `ATen/native/layer_norm.h`, `c10/core/ScalarType.h`, `c10/util/Exception.h`, `c10/util/irange.h`, `torch/csrc/jit/ir/alias_analysis.h`, `...`
- External includes / 外部头文件: `algorithm`, `memory`, `dnnl_types.h`, `ideep.hpp`
- Namespaces / 命名空间: `torch::jit`
- Representative symbols / 代表性符号: `Tensor`, `aliasAnalysisFromSchema`, `ValueSet`, `ValueSetPtr`, `getLastUse`, `node`, `uses`, `isAfter`, `merge_sets`, `begin`, `...`
