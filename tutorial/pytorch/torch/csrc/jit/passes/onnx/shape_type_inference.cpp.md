# shape_type_inference.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/passes/onnx/shape_type_inference.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements the TorchScript JIT pass logic for shape type inference, including graph analysis and rewrites.
- 用途 (CN): 实现与 shape type inference 相关的 TorchScript JIT pass 逻辑，包括图分析与改写。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14
```cpp
#include <torch/csrc/jit/passes/onnx/shape_type_inference.h>

#include <c10/util/irange.h>
#include <torch/csrc/jit/jit_log.h>
#include <torch/csrc/jit/passes/onnx/constant_fold.h>
#include <torch/csrc/jit/passes/onnx/constant_map.h>
#include <torch/csrc/jit/passes/onnx/fixup_onnx_controlflow.h>
#include <torch/csrc/jit/passes/onnx/helper.h>
#include <torch/csrc/jit/passes/onnx/scalar_type_analysis.h>
#include <torch/csrc/jit/python/python_arg_flatten.h>
#include <torch/csrc/jit/serialization/export.h>
#include <torch/csrc/jit/serialization/onnx.h>
#include <torch/csrc/utils/python_strings.h>

```
- EN: Pulls in the headers needed by the shape type inference logic. Internal dependencies: `torch/csrc/jit/passes/onnx/shape_type_inference.h`, `c10/util/irange.h`, `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/onnx/constant_fold.h`, `torch/csrc/jit/passes/onnx/constant_map.h`, `...`; external dependencies: none.
- CN: 为 shape type inference 相关逻辑引入所需头文件。内部依赖：`torch/csrc/jit/passes/onnx/shape_type_inference.h`, `c10/util/irange.h`, `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/onnx/constant_fold.h`, `torch/csrc/jit/passes/onnx/constant_map.h`, `...`；外部依赖：无。

### Lines 15-26
```cpp
#include <onnx/shape_inference/implementation.h>
#include <algorithm>
#include <cmath>
#include <iterator>
#include <limits>
#include <unordered_set>
#include <utility>

namespace torch::jit {

static inline bool PyNone_Check(PyObject* o) {
  return Py_IsNone(o);
```
- EN: This block produces a result or forwards a computed value; performs optimization-oriented rewriting. Key symbols: `PyNone_Check`, `Py_IsNone`.
- CN: 该代码块返回结果或转发已计算的值；执行面向优化的改写。关键符号：`PyNone_Check`, `Py_IsNone`。

### Lines 27-38
```cpp
}

std::pair<TypePtr, bool> MergeInferredType(
    const TypePtr& existing_type,
    const TypePtr& inferred_type) {
  auto new_list_type = inferred_type->cast<ListType>();
  auto use_inferred_type = false;
  if (new_list_type) {
    return std::make_pair(inferred_type, true);
  }
  auto new_tensor_type = inferred_type->cast<TensorType>();
  auto old_tensor_type = existing_type->cast<TensorType>();
```
- EN: This block handles conditional branches; produces a result or forwards a computed value. Key symbols: `MergeInferredType`, `make_pair`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值。关键符号：`MergeInferredType`, `make_pair`。

### Lines 39-51
```cpp

  if (new_tensor_type && old_tensor_type) {
    if (!old_tensor_type->device()) {
      // device not available means this is an invalid tensor type (most likely
      // an empty one) return inferred type directly.
      return std::make_pair(new_tensor_type, true);
    }
    auto type = old_tensor_type;
    if (new_tensor_type->dim()) {
      type = type->withSymbolicShapes(new_tensor_type->symbolic_sizes());
      use_inferred_type = true;
    }
    if (new_tensor_type->scalarType().has_value()) {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `device`, `make_pair`, `dim`, `withSymbolicShapes`, `symbolic_sizes`, `scalarType`, `...`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`device`, `make_pair`, `dim`, `withSymbolicShapes`, `symbolic_sizes`, `scalarType`, `...`。

### Lines 52-63
```cpp
      type = type->withScalarType(new_tensor_type->scalarType());
      use_inferred_type = true;
    }
    return std::make_pair(type, use_inferred_type);
  }

  if (old_tensor_type) {
    return std::make_pair(existing_type, false);
  }

  auto old_list_type = existing_type->cast<ListType>();
  if (new_tensor_type && old_list_type) {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value. Key symbols: `withScalarType`, `scalarType`, `make_pair`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值。关键符号：`withScalarType`, `scalarType`, `make_pair`。

### Lines 64-77
```cpp
    if (new_tensor_type->sizes().isComplete()) {
      return std::make_pair(inferred_type, true);
    }
    return std::make_pair(existing_type, false);
  }

  return std::make_pair(inferred_type, true);
}

void MergeInferredTypeAndSetMap(
    Value* dest_v,
    const TypePtr& existing_type,
    const TypePtr& inferred_type) {
  auto [mergedType, inferred] = MergeInferredType(existing_type, inferred_type);
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `sizes`, `isComplete`, `make_pair`, `MergeInferredTypeAndSetMap`, `MergeInferredType`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`sizes`, `isComplete`, `make_pair`, `MergeInferredTypeAndSetMap`, `MergeInferredType`。

### Lines 78-94
```cpp
  dest_v->setType(mergedType);
  ConstantValueMap::SetUseInferredType(dest_v->debugName(), inferred);
}

namespace {
namespace onnx_torch = ::torch::onnx;
namespace onnx = ::ONNX_NAMESPACE;

// SymbolDimMap is a Torch-to-ONNX shape look-up. This is built so it can be
// returned by the export function. During the export however, when we come
// across new ONNX shapes, the reverse look-up is needed. To avoid incurring
// a linear-time look-up, we maintain DimSymbolMap in parallel.
c10::ShapeSymbol ONNXDimToShapeSymbol(
    const onnx::TensorShapeProto_Dimension& dim,
    SymbolDimMap& symbol_dim_map,
    DimSymbolMap& dim_symbol_map) {
  if (dim.has_dim_value()) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `setType`, `SetUseInferredType`, `debugName`, `ONNXDimToShapeSymbol`, `has_dim_value`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`setType`, `SetUseInferredType`, `debugName`, `ONNXDimToShapeSymbol`, `has_dim_value`。

### Lines 95-106
```cpp
    return c10::ShapeSymbol::fromStaticSize(dim.dim_value());
  }
  std::optional<c10::ShapeSymbol> sym = std::nullopt;
  if (dim.has_dim_param()) {
    // If this param is already known, assign the same Symbol.
    GRAPH_UPDATE("Got dim_param:", dim.dim_param());
    auto maybe_symbol = dim_symbol_map.find(dim.dim_param());
    if (maybe_symbol != dim_symbol_map.end()) {
      sym = maybe_symbol->second;
    }
  }
  if (!sym) {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `fromStaticSize`, `dim_value`, `has_dim_param`, `dim_param`, `find`, `end`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`fromStaticSize`, `dim_value`, `has_dim_param`, `dim_param`, `find`, `end`。

### Lines 107-120
```cpp
    sym = c10::ShapeSymbol::newSymbol();
    // If dim.dim_param() is empty, no need to keep track
    // because there won't be duplicates.
    symbol_dim_map[sym.value()] = dim.dim_param();
    dim_symbol_map[dim.dim_param()] = sym.value();
  }
  return sym.value();
}

TensorTypePtr TorchTensorTypeFromONNX(
    const onnx::TypeProto_Tensor& onnx_tensor_type,
    SymbolDimMap& symbol_dim_map,
    DimSymbolMap& dim_symbol_map) {
  std::optional<at::ScalarType> scalar_type;
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `newSymbol`, `value`, `dim_param`, `TorchTensorTypeFromONNX`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`newSymbol`, `value`, `dim_param`, `TorchTensorTypeFromONNX`。

### Lines 121-132
```cpp
  if (onnx_tensor_type.has_elem_type()) {
    scalar_type = ONNXTypeToATenType(onnx_tensor_type.elem_type());
  }

  auto v_type = TensorType::create(
      scalar_type,
      at::kCPU,
      c10::SymbolicShape(),
      c10::VaryingShape<c10::Stride>{},
      {});
  if (onnx_tensor_type.has_shape()) {
    std::vector<c10::ShapeSymbol> sizes;
```
- EN: This block handles conditional branches. Key symbols: `has_elem_type`, `ONNXTypeToATenType`, `elem_type`, `create`, `SymbolicShape`, `has_shape`.
- CN: 该代码块处理条件分支。关键符号：`has_elem_type`, `ONNXTypeToATenType`, `elem_type`, `create`, `SymbolicShape`, `has_shape`。

### Lines 133-148
```cpp
    const auto& onnx_shape = onnx_tensor_type.shape();

    for (const auto i : c10::irange(onnx_shape.dim_size())) {
      sizes.emplace_back(ONNXDimToShapeSymbol(
          onnx_shape.dim(i), symbol_dim_map, dim_symbol_map));
    }
    v_type = TensorType::create(scalar_type, at::kCPU, sizes.size(), {});
    v_type = v_type->withSymbolicShapes(c10::SymbolicShape(sizes));

    if (v_type->sizes().concrete_sizes().has_value()) {
      // Populate strides based on sizes info, if sizes are all static.
      // Creating strides ensures yielding True for isCompleteTensor.
      v_type = v_type->contiguous();
    }
  }

```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `shape`, `irange`, `dim_size`, `emplace_back`, `ONNXDimToShapeSymbol`, `dim`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`shape`, `irange`, `dim_size`, `emplace_back`, `ONNXDimToShapeSymbol`, `dim`, `...`。

### Lines 149-162
```cpp
  return v_type;
}

ListTypePtr TorchListTypeFromONNX(
    const onnx::TypeProto_Sequence& onnx_sequence_type,
    SymbolDimMap& symbol_dim_map,
    DimSymbolMap& dim_symbol_map) {
  if (onnx_sequence_type.has_elem_type()) {
    const auto& onnx_seq_elem_type = onnx_sequence_type.elem_type();
    if (onnx_seq_elem_type.has_tensor_type()) {
      const auto& onnx_tensor_type = onnx_seq_elem_type.tensor_type();
      const auto v_tensor_type = TorchTensorTypeFromONNX(
          onnx_tensor_type, symbol_dim_map, dim_symbol_map);
      auto v_type = ListType::create(v_tensor_type);
```
- EN: This block handles conditional branches; produces a result or forwards a computed value. Key symbols: `TorchListTypeFromONNX`, `has_elem_type`, `elem_type`, `has_tensor_type`, `tensor_type`, `TorchTensorTypeFromONNX`, `...`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值。关键符号：`TorchListTypeFromONNX`, `has_elem_type`, `elem_type`, `has_tensor_type`, `tensor_type`, `TorchTensorTypeFromONNX`, `...`。

### Lines 163-174
```cpp
      return v_type;
    }
  }
  return nullptr;
}

void UpdateTorchValueByOnnxValueInfo(
    Value* v,
    const onnx::ValueInfoProto& p_info,
    SymbolDimMap& symbol_dim_map,
    DimSymbolMap& dim_symbol_map) {
  if (!p_info.has_type()) {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `UpdateTorchValueByOnnxValueInfo`, `has_type`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`UpdateTorchValueByOnnxValueInfo`, `has_type`。

### Lines 175-188
```cpp
    return;
  }

  const auto& p_type = p_info.type();
  if (p_type.has_tensor_type()) {
    const auto torch_tensor_type = TorchTensorTypeFromONNX(
        p_type.tensor_type(), symbol_dim_map, dim_symbol_map);
    if (torch_tensor_type) {
      MergeInferredTypeAndSetMap(v, v->type(), torch_tensor_type);
    }
  } else if (p_type.has_sequence_type()) {
    const auto torch_list_type = TorchListTypeFromONNX(
        p_type.sequence_type(), symbol_dim_map, dim_symbol_map);
    if (torch_list_type) {
```
- EN: This block handles conditional branches. Key symbols: `type`, `has_tensor_type`, `TorchTensorTypeFromONNX`, `tensor_type`, `MergeInferredTypeAndSetMap`, `has_sequence_type`, `...`.
- CN: 该代码块处理条件分支。关键符号：`type`, `has_tensor_type`, `TorchTensorTypeFromONNX`, `tensor_type`, `MergeInferredTypeAndSetMap`, `has_sequence_type`, `...`。

### Lines 189-200
```cpp
      MergeInferredTypeAndSetMap(v, v->type(), torch_list_type);
    }
  }
}

bool IsValidONNXControlflowNode(const Node* n) {
  // Skip when block size is zero. This is when the node is being created,
  // and doesn't have subblocks attached yet. Run shape inference for these
  // nodes later, when the subgraph has already completed shape inferencing.
  auto node_kind = n->kind();
  if (node_kind == ::c10::onnx::Loop || node_kind == ::c10::onnx::If) {
    if (n->blocks().empty()) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `MergeInferredTypeAndSetMap`, `type`, `IsValidONNXControlflowNode`, `kind`, `blocks`, `empty`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`MergeInferredTypeAndSetMap`, `type`, `IsValidONNXControlflowNode`, `kind`, `blocks`, `empty`。

### Lines 201-213
```cpp
      return false;
    }
  }

  return true;
}

bool IsValidONNXNode(const Node* n) {
  auto node_kind = n->kind();

  if (!node_kind.is_onnx()) {
    // node kind is not ONNX, skipped.
    return false;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `IsValidONNXNode`, `kind`, `is_onnx`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`IsValidONNXNode`, `kind`, `is_onnx`。

### Lines 214-227
```cpp
  }

  if (!IsValidONNXControlflowNode(n)) {
    return false;
  }

  for (auto b : n->blocks()) {
    for (auto b_n : b->nodes()) {
      if (!IsValidONNXNode(b_n)) {
        return false;
      }
    }
  }

```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `IsValidONNXControlflowNode`, `blocks`, `nodes`, `IsValidONNXNode`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`IsValidONNXControlflowNode`, `blocks`, `nodes`, `IsValidONNXNode`。

### Lines 228-239
```cpp
  return true;
}

bool CustomSettype(Node* node) {
  // This is a helper function to decide if the non-ONNX node actually has
  // custom setType from user
  // Go through every symbolic_sizes and if any one of them is static, we say
  // this is set by user. On the other hand, if all of them are * (dynamic), we
  // take this node does not have given type, since unreliable nodes have *
  // shape anyway.
  auto all_output_has_type = [](Value* output) {
    if (auto output_type = output->type()->cast<TensorType>()) {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `CustomSettype`, `type`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`CustomSettype`, `type`。

### Lines 240-252
```cpp
      if (auto sizes = output_type->symbolic_sizes().sizes()) {
        return std::any_of(std::begin(*sizes), std::end(*sizes), [](auto size) {
          return size.is_static();
        });
      }
    }
    return false;
  };

  return std::all_of(
      node->outputs().begin(), node->outputs().end(), all_output_has_type);
}

```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `symbolic_sizes`, `sizes`, `any_of`, `begin`, `end`, `is_static`, `...`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`symbolic_sizes`, `sizes`, `any_of`, `begin`, `end`, `is_static`, `...`。

### Lines 253-266
```cpp
Value* CloneValueFromListConstruct(
    Value* v,
    const std::shared_ptr<Graph>& n_graph,
    int opset_version) {
  auto lc_node = v->node();
  TORCH_INTERNAL_ASSERT(lc_node->kind() == ::c10::prim::ListConstruct);
  // In jit/passes/onnx/peephole.cpp::eraseListConstruct,
  // prim::ListConstruct is converted to onnx::Concat. The conversion should
  // eventually be moved to symbolic. For now, treat this operator as
  // special case, and change from list type to tensor type. The scalar type
  // is preserved. If the elemtype is Int, insert a onnx::Concat node into
  // the graph.
  TypePtr elem = v->type()->castRaw<ListType>()->getElementType();
  std::optional<at::ScalarType> scalar_type = std::nullopt;
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `CloneValueFromListConstruct`, `node`, `kind`, `type`, `getElementType`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`CloneValueFromListConstruct`, `node`, `kind`, `type`, `getElementType`。

### Lines 267-281
```cpp
  if (elem->cast<IntType>()) {
    scalar_type = at::kLong;
    if (isValidToTransformToONNXConcatNode(v->node())) {
      auto concat_node = transformToONNXConcatNode(
          n_graph.get(), v->node(), true, opset_version);
      return concat_node->output();
    }
  } else if (elem->cast<FloatType>()) {
    scalar_type = at::kFloat;
  } else if (elem->cast<BoolType>()) {
    scalar_type = at::kBool;
  } else if (auto t_type = elem->cast<TensorType>()) {
    scalar_type = t_type->scalarType();
  }

```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `isValidToTransformToONNXConcatNode`, `node`, `transformToONNXConcatNode`, `get`, `output`, `scalarType`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`isValidToTransformToONNXConcatNode`, `node`, `transformToONNXConcatNode`, `get`, `output`, `scalarType`。

### Lines 282-294
```cpp
  auto input = n_graph->addInput();
  if (scalar_type) {
    auto v_type = TensorType::create(
        scalar_type,
        at::kCPU,
        c10::SymbolicShape(),
        c10::VaryingShape<c10::Stride>{},
        {});
    input->setType(v_type);
  }
  return input;
}

```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `addInput`, `create`, `SymbolicShape`, `setType`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`addInput`, `create`, `SymbolicShape`, `setType`。

### Lines 295-308
```cpp
// Clone the node n for the new graph.
Node* CloneNodeToGraph(
    Node* n,
    std::shared_ptr<Graph> n_graph,
    const ParamMap& params_dict,
    int opset_version) {
  auto clone_node = n_graph->createClone(
      n, [&n_graph, &params_dict, opset_version](Value* v) {
        auto v_n = v->node();
        switch (v_n->kind()) {
          case ::c10::prim::Constant:
          case ::c10::onnx::Constant: {
            // Clone the input if it is constant.
            auto constant_n = n_graph->insertNode(
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `CloneNodeToGraph`, `createClone`, `node`, `kind`, `insertNode`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`CloneNodeToGraph`, `createClone`, `node`, `kind`, `insertNode`。

### Lines 309-325
```cpp
                n_graph->createClone(v_n, [](Value* v) { return v; }));
            return constant_n->output();
          }
          case ::c10::prim::ListConstruct: {
            return CloneValueFromListConstruct(v, n_graph, opset_version);
          }
          case ::c10::prim::PackPadded: {
            auto input = n_graph->addInput();
            if (v == v_n->output(0)) {
              // Only the first output requires this workaround.
              // In `peephole` pass, user nodes are modified to consume the
              // input instead.
              input->copyMetadata(v_n->input(0));
            } else {
              input->copyMetadata(v);
            }
            return input;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `createClone`, `output`, `CloneValueFromListConstruct`, `addInput`, `copyMetadata`, `input`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`createClone`, `output`, `CloneValueFromListConstruct`, `addInput`, `copyMetadata`, `input`。

### Lines 326-338
```cpp
          }
          default: {
            // Try to lookup input value and insert it into the graph.
            // If the input value is unknown, set it to graph input in the new
            // graph, and copy over metadata, such as datatype and shape.
            ::std::optional<at::Tensor> val = ::std::nullopt;
            auto v0 = params_dict.find(v->debugName());
            if (v0 != params_dict.end()) {
              val = v0->second.toTensor();
            } else {
              val = ConstantValueMap::GetValue(v->debugName());
            }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `find`, `debugName`, `end`, `toTensor`, `GetValue`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`find`, `debugName`, `end`, `toTensor`, `GetValue`。

### Lines 339-351
```cpp
            if (val.has_value()) {
              return n_graph
                  ->insertNode(n_graph->create(::c10::onnx::Constant)
                                   ->t_(attr::value, val.value()))
                  ->output();
            }
            auto input = n_graph->addInput();
            input->copyMetadata(v);
            return input;
          }
        }
      });
  return clone_node;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `has_value`, `insertNode`, `create`, `t_`, `value`, `output`, `...`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`has_value`, `insertNode`, `create`, `t_`, `value`, `output`, `...`。

### Lines 352-364
```cpp
}

bool HasValidType(const TypePtr& type, const std::string& name) {
  if (auto t_type = type->cast<TensorType>()) {
    if (!t_type->scalarType().has_value()) {
      GRAPH_UPDATE("Input ", name, " is missing tensor datatype.");
      return false;
    }
  } else if (auto s_type = type->cast<ListType>()) {
    auto e_type = s_type->getElementType();
    return HasValidType(e_type, name);
  } else if (auto o_type = type->cast<OptionalType>()) {
    auto e_type = o_type->getElementType();
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `HasValidType`, `scalarType`, `has_value`, `getElementType`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`HasValidType`, `scalarType`, `has_value`, `getElementType`。

### Lines 365-376
```cpp
    return HasValidType(e_type, name);
  }
  return true;
}

bool IsGraphValidForInference(const std::shared_ptr<Graph>& graph) {
  // Verify if every input has type (either Tensor, Sequence or Optional) and
  // scalar type. This is a requirement for ONNX graph inputs.
  for (auto in : graph->inputs()) {
    return HasValidType(in->type(), in->debugName());
  }
  return true;
```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `HasValidType`, `IsGraphValidForInference`, `inputs`, `type`, `debugName`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`HasValidType`, `IsGraphValidForInference`, `inputs`, `type`, `debugName`。

### Lines 377-400
```cpp
}

void ConvertGraphToONNXProto(
    const std::shared_ptr<Graph>& graph,
    std::shared_ptr<onnx::ModelProto>& model_proto,
    SymbolDimMap& symbol_dim_map,
    DimSymbolMap& dim_symbol_map,
    int opset_version) {
  auto
      [model_proto_tmp,
       export_map,
       new_symbol_dim_map,
       val_use_external_data_format,
       node_names] =
          export_onnx(
              graph,
              {},
              opset_version,
              {},
              false,
              onnx_torch::OperatorExportTypes::ONNX,
              true,
              true,
              {},
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `ConvertGraphToONNXProto`, `export_onnx`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`ConvertGraphToONNXProto`, `export_onnx`。

### Lines 401-413
```cpp
              true,
              false,
              std::string());
  model_proto = std::move(model_proto_tmp);
  symbol_dim_map.insert(new_symbol_dim_map.begin(), new_symbol_dim_map.end());
  for (const auto& pair : new_symbol_dim_map) {
    dim_symbol_map[pair.second] = pair.first;
  }
  for (int i = 0; i < model_proto->graph().output_size(); ++i) {
    model_proto->mutable_graph()->mutable_output(i)->clear_type();
  }
}

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `string`, `move`, `insert`, `begin`, `end`, `graph`, `...`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`string`, `move`, `insert`, `begin`, `end`, `graph`, `...`。

### Lines 414-426
```cpp
std::optional<at::Tensor> ComputeConstantFolding(
    const Node* n,
    int opset_version) {
  if (n->inputs().empty()) {
    return std::nullopt;
  }
  std::vector<at::Tensor> inputTensorValues;
  for (auto i : c10::irange(n->inputs().size())) {
    if (TensorTypePtr input_type = n->input(i)->type()->cast<TensorType>()) {
      if (!ConstantValueMap::HasValue(n->input(i)->debugName())) {
        return std::nullopt;
      }
      auto tensor_value =
```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `ComputeConstantFolding`, `inputs`, `empty`, `irange`, `size`, `input`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`ComputeConstantFolding`, `inputs`, `empty`, `irange`, `size`, `input`, `...`。

### Lines 427-438
```cpp
          ConstantValueMap::GetValue(n->input(i)->debugName()).value();
      inputTensorValues.emplace_back(tensor_value);
    }
  }
  if (inputTensorValues.size() < n->inputs().size()) {
    return std::nullopt;
  }
  try {
    return onnx_constant_fold::runTorchBackendForOnnx(
        n, inputTensorValues, opset_version);
  } catch (const std::exception& ex) {
    auto ex_str = std::string(ex.what());
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `GetValue`, `input`, `debugName`, `value`, `emplace_back`, `size`, `...`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`GetValue`, `input`, `debugName`, `value`, `emplace_back`, `size`, `...`。

### Lines 439-451
```cpp
    ex_str = ex_str.substr(0, ex_str.find('\n'));
    TORCH_WARN("Constant folding in symbolic shape inference fails: ", ex_str);
    return std::nullopt;
  }
}

// Similar to the function above, but for symbolic shapes.
std::optional<::c10::SymbolicShape> ComputeShapeFromReshape(
    Node* n,
    const c10::SymbolicShape& input_shape,
    const c10::SymbolicShape& shape,
    int opset_version) {
  std::vector<c10::ShapeSymbol> input_shape_vector =
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `substr`, `find`, `ComputeShapeFromReshape`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`substr`, `find`, `ComputeShapeFromReshape`。

### Lines 452-463
```cpp
      input_shape.sizes().value();
  std::vector<c10::ShapeSymbol> shape_vector = shape.sizes().value();
  TORCH_INTERNAL_ASSERT(
      !input_shape_vector.empty() || !shape_vector.empty(),
      "Reshape node should have at least one input size > 0 when constant folding.");
  if (shape_vector.empty()) {
    return input_shape;
  }
  if (input_shape_vector.empty()) {
    return shape;
  }

```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `sizes`, `value`, `empty`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`sizes`, `value`, `empty`。

### Lines 464-475
```cpp
  auto is_zero = [](c10::ShapeSymbol& ss) { return ss.value() == 0; };
  auto it_0 = std::find_if(shape_vector.begin(), shape_vector.end(), is_zero);
  bool shape_has_zero = it_0 != shape_vector.end();

  int64_t minus_one_pos = -1;
  for (auto i : c10::irange(shape_vector.size())) {
    if (shape_vector[i].value() == -1) {
      minus_one_pos = i;
      break;
    }
  }

```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `value`, `find_if`, `begin`, `end`, `irange`, `size`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`value`, `find_if`, `begin`, `end`, `irange`, `size`。

### Lines 476-488
```cpp
  int allowzero = 0;
  if (opset_version >= 14 && n->hasAttributeS("allowzero")) {
    allowzero = n->i(attr::allowzero);
  }

  TORCH_CHECK(
      !(shape_has_zero && allowzero == 1 && minus_one_pos != -1),
      "0 and -1 cannot both be present in `Shape` input of `Reshape` node, when `allowzero=1`.");

  if (minus_one_pos == -1 && (!shape_has_zero || allowzero)) {
    return shape;
  }
  std::vector<c10::ShapeSymbol> final_shape;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `hasAttributeS`, `i`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`hasAttributeS`, `i`。

### Lines 489-503
```cpp
  uint64_t shape_ratio = 1;
  std::unordered_map<int64_t, int64_t> sym_map;
  for (const c10::ShapeSymbol& input_shape : input_shape_vector) {
    // input_shape.static_size() could be zero when torch.tensor([]) is used.
    if (input_shape.is_static() && input_shape.static_size() != 0) {
      if (shape_ratio >=
          std::numeric_limits<uint64_t>::max() / input_shape.static_size()) {
        TORCH_WARN(
            "ComputeShapeFromReshape(), shape_ratio overflows, skip shape inference.");
        return std::nullopt;
      } else {
        shape_ratio *= static_cast<uint64_t>(input_shape.static_size());
      }
    } else {
      auto value = input_shape.value();
```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `is_static`, `static_size`, `max`, `ComputeShapeFromReshape`, `value`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`is_static`, `static_size`, `max`, `ComputeShapeFromReshape`, `value`。

### Lines 504-516
```cpp
      sym_map.emplace(value, 0).first->second += 1;
    }
  }
  int shape_size = static_cast<int>(shape_vector.size());
  for (const int i : c10::irange(shape_size)) {
    if (i == minus_one_pos) {
      continue;
    }
    c10::ShapeSymbol& target_shape = shape_vector[i];
    if (target_shape.value() == 0) {
      target_shape = input_shape_vector[i];
    }
    if (target_shape.is_static()) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `emplace`, `size`, `irange`, `value`, `is_static`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`emplace`, `size`, `irange`, `value`, `is_static`。

### Lines 517-529
```cpp
      shape_ratio /= static_cast<uint64_t>(target_shape.static_size());
    } else {
      auto value = target_shape.value();
      if (sym_map.find(value) == sym_map.end()) {
        return std::nullopt;
      }
      sym_map[value]--;
      if (sym_map[value] == 0) {
        sym_map.erase(value);
      }
    }
  }

```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `static_size`, `value`, `find`, `end`, `erase`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`static_size`, `value`, `find`, `end`, `erase`。

### Lines 530-545
```cpp
  // sym_map is used to match shape symbols between the input and shape.
  // If there is a mismatch, the output shape cannot be estimated.
  if (!sym_map.empty()) {
    return std::nullopt;
  }

  TORCH_INTERNAL_ASSERT(
      minus_one_pos != -1,
      "There are no examples for shape_has_zero = true && minus_one_pos == -1.");

  for (const auto i : c10::irange(minus_one_pos)) {
    c10::ShapeSymbol cur_shape(
        shape_vector[i].value() == 0 ? input_shape_vector[i] : shape_vector[i]);
    final_shape.push_back(cur_shape);
  }
  if (minus_one_pos != -1) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `empty`, `irange`, `cur_shape`, `value`, `push_back`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`empty`, `irange`, `cur_shape`, `value`, `push_back`。

### Lines 546-557
```cpp
    final_shape.push_back(
        c10::ShapeSymbol::fromStaticSize(static_cast<int64_t>(shape_ratio)));
  }
  for (auto i = minus_one_pos + 1; i < shape_size; i++) {
    c10::ShapeSymbol cur_shape(
        shape_vector[i].value() == 0 ? input_shape_vector[i] : shape_vector[i]);
    final_shape.push_back(cur_shape);
  }
  c10::SymbolicShape final_shape_0(final_shape);
  return final_shape_0;
}

```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `push_back`, `fromStaticSize`, `cur_shape`, `value`, `final_shape_0`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`push_back`, `fromStaticSize`, `cur_shape`, `value`, `final_shape_0`。

### Lines 558-570
```cpp
std::optional<::c10::SymbolicShape> ComputeShapeFromExpand(
    const std::vector<::c10::ShapeSymbol>& input_shape,
    const std::vector<int64_t>& reshape) {
  for (const auto& it : reshape) {
    if (it < 0) {
      return std::nullopt;
    }
  }
  std::vector<::c10::ShapeSymbol> final_shape;
  if (input_shape.size() >= reshape.size()) {
    final_shape = input_shape;
  } else {
    for (auto v : reshape) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value. Key symbols: `ComputeShapeFromExpand`, `size`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值。关键符号：`ComputeShapeFromExpand`, `size`。

### Lines 571-587
```cpp
      final_shape.emplace_back(::c10::ShapeSymbol::fromStaticSize(v));
    }
  }
  auto min_size = std::min(input_shape.size(), reshape.size());
  for (const auto i : c10::irange(min_size)) {
    auto idx = final_shape.size() - i - 1;
    auto input_shape_idx = input_shape.size() - i - 1;
    auto reshape_idx = reshape.size() - i - 1;
    if (input_shape[input_shape_idx].is_static()) {
      auto input_shape_value = input_shape[input_shape_idx].static_size();
      auto reshape_value = reshape[reshape_idx];
      TORCH_INTERNAL_ASSERT(
          input_shape_value == reshape_value || input_shape_value == 1 ||
              reshape_value == 1,
          "ONNX Expand input shape constraint not satisfied.");
      final_shape[idx] = ::c10::ShapeSymbol::fromStaticSize(
          std::max(input_shape_value, reshape_value));
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `emplace_back`, `fromStaticSize`, `min`, `size`, `irange`, `is_static`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`emplace_back`, `fromStaticSize`, `min`, `size`, `irange`, `is_static`, `...`。

### Lines 588-603
```cpp

    } else {
      final_shape[idx] = ::c10::ShapeSymbol::newSymbol();
    }
  }
  ::c10::SymbolicShape shape(final_shape);
  return shape;
}

std::optional<::c10::SymbolicShape> ComputeShapeFromTile(
    const std::vector<::c10::ShapeSymbol>& input_shape,
    const std::vector<int64_t>& reshape) {
  TORCH_INTERNAL_ASSERT(
      input_shape.size() == reshape.size(),
      "ONNX Tile input shapes do not match.");
  for (const auto& it : reshape) {
```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value. Key symbols: `newSymbol`, `shape`, `ComputeShapeFromTile`, `size`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值。关键符号：`newSymbol`, `shape`, `ComputeShapeFromTile`, `size`。

### Lines 604-619
```cpp
    if (it < 0) {
      return std::nullopt;
    }
  }
  std::vector<::c10::ShapeSymbol> final_shape;
  final_shape.reserve(input_shape.size());
  for (const auto i : c10::irange(input_shape.size())) {
    if (input_shape[i].is_static()) {
      final_shape.emplace_back(::c10::ShapeSymbol::fromStaticSize(
          input_shape[i].static_size() * reshape[i]));
    } else {
      final_shape.emplace_back(::c10::ShapeSymbol::newSymbol());
    }
  }
  ::c10::SymbolicShape shape(final_shape);
  return shape;
```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value. Key symbols: `reserve`, `size`, `irange`, `is_static`, `emplace_back`, `fromStaticSize`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值。关键符号：`reserve`, `size`, `irange`, `is_static`, `emplace_back`, `fromStaticSize`, `...`。

### Lines 620-631
```cpp
}

void UpdateRank(Value* value, size_t rank) {
  ConstantValueMap::SetRank(value->debugName(), rank);
  if (TensorTypePtr value_type = value->type()->cast<TensorType>()) {
    std::optional<size_t> rank_opt = rank;
    auto shape = ::c10::SymbolicShape(rank_opt);
    value->setType(value_type->withSymbolicShapes(shape));
  }
}

void UpdateShapeFromVector(
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `UpdateRank`, `SetRank`, `debugName`, `type`, `SymbolicShape`, `setType`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`UpdateRank`, `SetRank`, `debugName`, `type`, `SymbolicShape`, `setType`, `...`。

### Lines 632-645
```cpp
    Value* value,
    const std::vector<int64_t>& shape_size) {
  ::c10::SymbolicShape shape(shape_size);
  ConstantValueMap::SetShape(value->debugName(), shape);
  if (shape_size.empty()) {
    UpdateRank(value, 0);
    return;
  }
  ConstantValueMap::SetRank(value->debugName(), shape_size.size());
  if (TensorTypePtr value_type = value->type()->cast<TensorType>()) {
    value->setType(value_type->withSymbolicShapes(shape));
  }
}

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `shape`, `SetShape`, `debugName`, `empty`, `UpdateRank`, `SetRank`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`shape`, `SetShape`, `debugName`, `empty`, `UpdateRank`, `SetRank`, `...`。

### Lines 646-660
```cpp
void UpdateShape(Value* value, const ::c10::SymbolicShape& shape) {
  ConstantValueMap::SetShape(value->debugName(), shape);
  if (shape.rank().has_value()) {
    auto rank = shape.rank().value();
    if (rank == 0) {
      UpdateRank(value, 0);
      return;
    }
    ConstantValueMap::SetRank(value->debugName(), rank);
    if (TensorTypePtr value_type = value->type()->cast<TensorType>()) {
      value->setType(value_type->withSymbolicShapes(shape));
    }
  }
}

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `UpdateShape`, `SetShape`, `debugName`, `rank`, `has_value`, `value`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`UpdateShape`, `SetShape`, `debugName`, `rank`, `has_value`, `value`, `...`。

### Lines 661-672
```cpp
void UpdateShapeConstantValueMap(
    const Value* value,
    const ::c10::SymbolicShape& shape) {
  ConstantValueMap::SetShape(value->debugName(), shape);
  if (shape.rank().has_value()) {
    auto rank = shape.rank().value();
    ConstantValueMap::SetRank(value->debugName(), rank);
  }
}

std::optional<std::vector<int64_t>> GetValueFromListConstructNode(
    Node* lc_node) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `UpdateShapeConstantValueMap`, `SetShape`, `debugName`, `rank`, `has_value`, `value`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`UpdateShapeConstantValueMap`, `SetShape`, `debugName`, `rank`, `has_value`, `value`, `...`。

### Lines 673-684
```cpp
  std::vector<int64_t> shape_size;
  for (const auto& input : lc_node->inputs()) {
    if (input->type()->cast<TensorType>() &&
        ConstantValueMap::HasValue(input->debugName())) {
      auto lc_value = ConstantValueMap::GetValue(input->debugName()).value();
      if (lc_value.dim() == 0) {
        int64_t lc_value_0 = lc_value.item<int64_t>();
        shape_size.emplace_back(lc_value_0);
      }
    }
  }
  return lc_node->inputs().size() == shape_size.size()
```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `inputs`, `type`, `HasValue`, `debugName`, `GetValue`, `value`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`inputs`, `type`, `HasValue`, `debugName`, `GetValue`, `value`, `...`。

### Lines 685-700
```cpp
      ? std::optional<std::vector<int64_t>>(shape_size)
      : std::nullopt;
}

void SetShapeValueFromListConstructNode(Node* lc_node) {
  std::vector<c10::ShapeSymbol> shape_size;
  for (const auto& input : lc_node->inputs()) {
    if (TensorTypePtr shape_type = input->type()->cast<TensorType>()) {
      if (ConstantValueMap::HasValue(input->debugName())) {
        auto lc_value = ConstantValueMap::GetValue(input->debugName()).value();
        if (lc_value.dim() == 0) {
          int64_t lc_value_0 = lc_value.item<int64_t>();
          shape_size.emplace_back(c10::ShapeSymbol::fromStaticSize(lc_value_0));
        }
      } else if (ConstantValueMap::HasShapeValue(input->debugName())) {
        auto lc_value =
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `SetShapeValueFromListConstructNode`, `inputs`, `type`, `HasValue`, `debugName`, `GetValue`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`SetShapeValueFromListConstructNode`, `inputs`, `type`, `HasValue`, `debugName`, `GetValue`, `...`。

### Lines 701-714
```cpp
            ConstantValueMap::GetShapeValue(input->debugName()).value();
        if (lc_value.rank() == 1U) {
          shape_size.emplace_back(lc_value.at(0));
        }
      }
    }
  }
  if (lc_node->inputs().size() == shape_size.size()) {
    c10::SymbolicShape final_shape(shape_size);
    ConstantValueMap::SetShapeValue(
        lc_node->output()->debugName(), final_shape);
  }
}

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `GetShapeValue`, `debugName`, `value`, `rank`, `emplace_back`, `inputs`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`GetShapeValue`, `debugName`, `value`, `rank`, `emplace_back`, `inputs`, `...`。

### Lines 715-726
```cpp
std::vector<::c10::ShapeSymbol> Broadcast(
    const std::vector<::c10::ShapeSymbol>& input_shape_value_0,
    const std::vector<::c10::ShapeSymbol>& input_shape_value_1) {
  size_t rank_0 = input_shape_value_0.size();
  size_t rank_1 = input_shape_value_1.size();
  size_t rank_max = std::max(rank_0, rank_1);
  size_t rank_min = std::min(rank_0, rank_1);
  std::vector<::c10::ShapeSymbol> final_shape;
  final_shape.reserve(rank_max);
  std::generate_n(
      std::back_inserter(final_shape), rank_max, ::c10::ShapeSymbol::newSymbol);
  for (auto idx : c10::irange(rank_min)) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `Broadcast`, `size`, `max`, `min`, `reserve`, `generate_n`, `...`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`Broadcast`, `size`, `max`, `min`, `reserve`, `generate_n`, `...`。

### Lines 727-739
```cpp
    const c10::ShapeSymbol& ss_shape_0 = input_shape_value_0[rank_0 - 1 - idx];
    const c10::ShapeSymbol& ss_shape_1 = input_shape_value_1[rank_1 - 1 - idx];
    bool is_static_0 = ss_shape_0.is_static();
    bool is_static_1 = ss_shape_1.is_static();
    size_t shape_idx = rank_max - 1 - idx;
    if (is_static_0 && is_static_1) {
      int64_t static_0_sz = ss_shape_0.static_size();
      int64_t static_1_sz = ss_shape_1.static_size();
      // condition for corner case of 0d tensor
      // 0d tensor with 1d tensor would give us 0d tensor
      if (std::min(static_0_sz, static_1_sz) == 0) {
        final_shape[shape_idx] = ::c10::ShapeSymbol::fromStaticSize(
            std::min(static_0_sz, static_1_sz));
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `is_static`, `static_size`, `min`, `fromStaticSize`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`is_static`, `static_size`, `min`, `fromStaticSize`。

### Lines 740-751
```cpp
      } else {
        final_shape[shape_idx] = ::c10::ShapeSymbol::fromStaticSize(
            std::max(static_0_sz, static_1_sz));
      }
    } else if (!is_static_0 && !is_static_1) {
      if (ss_shape_0.value() == ss_shape_1.value()) {
        final_shape[shape_idx] = ss_shape_0;
      }
    }
  }
  if (rank_0 < rank_1) {
    for (size_t idx = rank_min; idx < rank_max; idx++) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `fromStaticSize`, `max`, `value`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`fromStaticSize`, `max`, `value`。

### Lines 752-763
```cpp
      size_t shape_idx = rank_max - 1 - idx;
      final_shape[shape_idx] = input_shape_value_1[shape_idx];
    }
  } else {
    for (size_t idx = rank_min; idx < rank_max; idx++) {
      size_t shape_idx = rank_max - 1 - idx;
      final_shape[shape_idx] = input_shape_value_0[shape_idx];
    }
  }
  return final_shape;
}

```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: no dominant local symbol names.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：无明显局部符号。

### Lines 764-776
```cpp
void ProcessBroadcastNode(Node* n) {
  TORCH_INTERNAL_ASSERT(n->inputs().size() == 2);
  if (ConstantValueMap::HasShape(n->input(0)->debugName()) &&
      ConstantValueMap::HasShape(n->input(1)->debugName())) {
    auto input_shape_0 = ConstantValueMap::GetShape(n->input(0)->debugName());
    auto input_shape_value_0 = input_shape_0.value().sizes().value();
    auto input_shape_1 = ConstantValueMap::GetShape(n->input(1)->debugName());
    auto input_shape_value_1 = input_shape_1.value().sizes().value();
    auto final_shape = Broadcast(input_shape_value_0, input_shape_value_1);
    UpdateShape(n->output(0), c10::SymbolicShape(final_shape));
  }
}

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `ProcessBroadcastNode`, `inputs`, `size`, `HasShape`, `input`, `debugName`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`ProcessBroadcastNode`, `inputs`, `size`, `HasShape`, `input`, `debugName`, `...`。

### Lines 777-789
```cpp
void ProcessShapeForConcatNode(Node* n) {
  auto axis = n->i(attr::axis);
  if (ConstantValueMap::HasRank(n->input(0)->debugName())) {
    auto rank = ConstantValueMap::GetRank(n->input(0)->debugName()).value();
    size_t axis_adjust = 0;
    if (axis >= 0) {
      axis_adjust = static_cast<size_t>(axis);
    } else {
      axis_adjust = static_cast<size_t>(axis + static_cast<int>(rank));
    }
    std::vector<::c10::ShapeSymbol> final_shape;
    final_shape.reserve(rank);
    for (auto idx : c10::irange(rank)) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `ProcessShapeForConcatNode`, `i`, `HasRank`, `input`, `debugName`, `GetRank`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`ProcessShapeForConcatNode`, `i`, `HasRank`, `input`, `debugName`, `GetRank`, `...`。

### Lines 790-807
```cpp
      if (idx == axis_adjust) {
        auto flag = true;
        int64_t size_total = 0;
        for (auto input_idx : c10::irange(n->inputs().size())) {
          if (ConstantValueMap::HasShape(n->input(input_idx)->debugName())) {
            auto input_shape =
                ConstantValueMap::GetShape(n->input(input_idx)->debugName());
            auto input_shape_value = input_shape.value().sizes();
            auto shape_symbol = input_shape_value.value()[idx];
            if (shape_symbol.is_static()) {
              size_total += shape_symbol.static_size();
            } else {
              flag = false;
              break;
            }
          }
        }
        if (flag) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `irange`, `inputs`, `size`, `HasShape`, `input`, `debugName`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`irange`, `inputs`, `size`, `HasShape`, `input`, `debugName`, `...`。

### Lines 808-819
```cpp
          final_shape.emplace_back(
              ::c10::ShapeSymbol::fromStaticSize(size_total));
        } else {
          final_shape.emplace_back(::c10::ShapeSymbol::newSymbol());
        }
      } else {
        auto flag = false;
        for (auto input_idx : c10::irange(n->inputs().size())) {
          if (ConstantValueMap::HasShape(n->input(input_idx)->debugName())) {
            auto input_shape =
                ConstantValueMap::GetShape(n->input(input_idx)->debugName());
            auto input_shape_value = input_shape.value().sizes();
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `emplace_back`, `fromStaticSize`, `newSymbol`, `irange`, `inputs`, `size`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`emplace_back`, `fromStaticSize`, `newSymbol`, `irange`, `inputs`, `size`, `...`。

### Lines 820-837
```cpp
            auto shape_symbol = input_shape_value.value()[idx];
            if (shape_symbol.is_static()) {
              final_shape.emplace_back(::c10::ShapeSymbol::fromStaticSize(
                  shape_symbol.static_size()));
              flag = true;
              break;
            }
          }
        }
        if (!flag) {
          final_shape.emplace_back(::c10::ShapeSymbol::newSymbol());
        }
      }
    }
    UpdateShape(n->output(0), c10::SymbolicShape(final_shape));
  }
}

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `value`, `is_static`, `emplace_back`, `fromStaticSize`, `static_size`, `newSymbol`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`value`, `is_static`, `emplace_back`, `fromStaticSize`, `static_size`, `newSymbol`, `...`。

### Lines 838-851
```cpp
void ProcessShapeValueForConcatNode(Node* n) {
  auto rank = n->inputs().size();
  std::vector<c10::ShapeSymbol> shape_size;
  for (const auto& input : n->inputs()) {
    if (ConstantValueMap::HasValue(input->debugName())) {
      auto concat_value =
          ConstantValueMap::GetValue(input->debugName()).value();
      if (concat_value.dim() == 1 && concat_value.size(0) == 1) {
        auto concat_value_0 = concat_value[0].item<int64_t>();
        shape_size.emplace_back(
            c10::ShapeSymbol::fromStaticSize(concat_value_0));
      }
    } else if (ConstantValueMap::HasShapeValue(input->debugName())) {
      auto concat_value =
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `ProcessShapeValueForConcatNode`, `inputs`, `size`, `HasValue`, `debugName`, `GetValue`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`ProcessShapeValueForConcatNode`, `inputs`, `size`, `HasValue`, `debugName`, `GetValue`, `...`。

### Lines 852-863
```cpp
          ConstantValueMap::GetShapeValue(input->debugName()).value();
      if (concat_value.rank() == 1U) {
        shape_size.emplace_back(concat_value.at(0));
      }
    }
  }
  if (rank == shape_size.size()) {
    c10::SymbolicShape final_shape(shape_size);
    ConstantValueMap::SetShapeValue(n->output(0)->debugName(), final_shape);
  }
}

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `GetShapeValue`, `debugName`, `value`, `rank`, `emplace_back`, `size`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`GetShapeValue`, `debugName`, `value`, `rank`, `emplace_back`, `size`, `...`。

### Lines 864-875
```cpp
void ProcessConcatNode(Node* n) {
  ProcessShapeForConcatNode(n);
  ProcessShapeValueForConcatNode(n);
}

void ProcessMatMulNode(Node* n) {
  if (ConstantValueMap::HasShape(n->input(0)->debugName()) &&
      ConstantValueMap::HasShape(n->input(1)->debugName())) {
    auto input_shape_0 =
        ConstantValueMap::GetShape(n->input(0)->debugName()).value();
    auto input_shape_value_0 = input_shape_0.sizes().value();
    auto input_shape_1 =
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `ProcessConcatNode`, `ProcessShapeForConcatNode`, `ProcessShapeValueForConcatNode`, `ProcessMatMulNode`, `HasShape`, `input`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`ProcessConcatNode`, `ProcessShapeForConcatNode`, `ProcessShapeValueForConcatNode`, `ProcessMatMulNode`, `HasShape`, `input`, `...`。

### Lines 876-889
```cpp
        ConstantValueMap::GetShape(n->input(1)->debugName()).value();
    auto input_shape_value_1 = input_shape_1.sizes().value();
    size_t rank_0 = input_shape_value_0.size();
    size_t rank_1 = input_shape_value_1.size();
    // Handle inputs of rank 1 just like numpy.matmul:
    // https://numpy.org/doc/stable/reference/generated/numpy.matmul.html
    auto is_rank_0_1 = false;
    if (rank_0 == 1) {
      input_shape_value_0.insert(
          input_shape_value_0.begin(), ::c10::ShapeSymbol::fromStaticSize(1));
      rank_0 = 2;
      is_rank_0_1 = true;
    }
    auto is_rank_1_1 = false;
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `GetShape`, `input`, `debugName`, `value`, `sizes`, `size`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`GetShape`, `input`, `debugName`, `value`, `sizes`, `size`, `...`。

### Lines 890-902
```cpp
    if (rank_1 == 1) {
      input_shape_value_1.emplace_back(::c10::ShapeSymbol::fromStaticSize(1));
      rank_1 = 2;
      is_rank_1_1 = true;
    }
    // Per https://pytorch.org/docs/stable/generated/torch.matmul.html
    // the broadcasting logic only applies to the batch dimensions, and not the
    // matrix dimensions so we remove the matrix dimensions which are the last 2
    // dimensions before broadcasting
    auto final_shape = Broadcast(
        std::vector<::c10::ShapeSymbol>(
            input_shape_value_0.begin(), input_shape_value_0.end() - 2),
        std::vector<::c10::ShapeSymbol>(
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `emplace_back`, `fromStaticSize`, `Broadcast`, `begin`, `end`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`emplace_back`, `fromStaticSize`, `Broadcast`, `begin`, `end`。

### Lines 903-916
```cpp
            input_shape_value_1.begin(), input_shape_value_1.end() - 2));
    // add the last 2 dimensions back, unless they do not exist in the first
    // place and inserted by this function Then apply [n,k]X[k,m]=[n,m], where
    // n=input_shape_value_0[rank_0 - 2], m=input_shape_value_1[rank_1 - 1]
    if (!is_rank_0_1) {
      final_shape.emplace_back(input_shape_value_0[rank_0 - 2]);
    }
    if (!is_rank_1_1) {
      final_shape.emplace_back(input_shape_value_1[rank_1 - 1]);
    }
    UpdateShape(n->output(0), c10::SymbolicShape(final_shape));
  }
}

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `begin`, `end`, `emplace_back`, `UpdateShape`, `output`, `SymbolicShape`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`begin`, `end`, `emplace_back`, `UpdateShape`, `output`, `SymbolicShape`。

### Lines 917-930
```cpp
void ProcessReduceNode(Node* n) {
  if (ConstantValueMap::HasShape(n->input(0)->debugName())) {
    auto input_shape_0 = ConstantValueMap::GetShape(n->input(0)->debugName());
    auto input_shape_value_0 = input_shape_0.value().sizes();
    size_t rank_0 = input_shape_value_0.value().size();
    std::vector<::c10::ShapeSymbol> final_shape;
    std::vector<int64_t> axes_vector(rank_0);
    if (n->hasAttributeS("axes")) {
      axes_vector = n->is(attr::axes);
    } else if (n->inputs().size() > 1) {
      axes_vector =
          ConstantValueMap::GetValueInto1DInt64Vector(n->input(1)->debugName());
    } else {
      std::iota(axes_vector.begin(), axes_vector.end(), 0);
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `ProcessReduceNode`, `HasShape`, `input`, `debugName`, `GetShape`, `value`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`ProcessReduceNode`, `HasShape`, `input`, `debugName`, `GetShape`, `value`, `...`。

### Lines 931-944
```cpp
    }

    for (auto idx : c10::irange(axes_vector.size())) {
      if (axes_vector[idx] < 0) {
        axes_vector[idx] += rank_0;
      }
    }
    final_shape.reserve(rank_0);
    // ONNX keepdims defaults to 1 when not set.
    int64_t keepdims = 1;
    if (n->hasAttributeS("keepdims")) {
      keepdims = n->i(attr::keepdims);
    }
    for (auto idx : c10::irange(rank_0)) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures. Key symbols: `irange`, `size`, `reserve`, `hasAttributeS`, `i`.
- CN: 该代码块处理条件分支；遍历集合或图结构。关键符号：`irange`, `size`, `reserve`, `hasAttributeS`, `i`。

### Lines 945-957
```cpp
      auto it = std::find(axes_vector.begin(), axes_vector.end(), idx);
      if (it != axes_vector.end()) {
        if (keepdims != 0) {
          final_shape.emplace_back(::c10::ShapeSymbol::fromStaticSize(1));
        }
      } else {
        final_shape.emplace_back(input_shape_value_0.value()[idx]);
      }
    }
    UpdateShape(n->output(0), c10::SymbolicShape(final_shape));
  }
}

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `find`, `begin`, `end`, `emplace_back`, `fromStaticSize`, `value`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`find`, `begin`, `end`, `emplace_back`, `fromStaticSize`, `value`, `...`。

### Lines 958-973
```cpp
void ProcessReshapeNode(Node* n, int opset_version) {
  const auto& input_name = n->input(0)->debugName();
  const auto& shape_name = n->input(1)->debugName();

  // When `shape` input value is statically known, compute output shape.
  if (ConstantValueMap::HasValue(shape_name)) {
    auto static_shape_value =
        ConstantValueMap::GetValueInto1DInt64Vector(shape_name);
    auto symbolic_input_shape = ConstantValueMap::GetShape(input_name);
    if (symbolic_input_shape && !static_shape_value.empty()) {
      auto final_shape = ComputeShapeFromReshape(
          n,
          symbolic_input_shape.value(),
          c10::SymbolicShape(static_shape_value),
          opset_version);
      if (final_shape) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `ProcessReshapeNode`, `input`, `debugName`, `HasValue`, `GetValueInto1DInt64Vector`, `GetShape`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`ProcessReshapeNode`, `input`, `debugName`, `HasValue`, `GetValueInto1DInt64Vector`, `GetShape`, `...`。

### Lines 974-986
```cpp
        UpdateShape(n->output(), final_shape.value());
        return;
      }
    }
  }

  // When `shape` input value is symbolically known, compute output shape.
  if (ConstantValueMap::HasShapeValue(shape_name) &&
      ConstantValueMap::HasShape(input_name)) {
    auto symbolic_input_shape = ConstantValueMap::GetShape(input_name).value();
    auto symbolic_shape_value =
        ConstantValueMap::GetShapeValue(shape_name).value();
    auto final_shape = ComputeShapeFromReshape(
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `UpdateShape`, `output`, `value`, `HasShapeValue`, `HasShape`, `GetShape`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`UpdateShape`, `output`, `value`, `HasShapeValue`, `HasShape`, `GetShape`, `...`。

### Lines 987-1003
```cpp
        n, symbolic_input_shape, symbolic_shape_value, opset_version);
    if (final_shape.has_value()) {
      UpdateShape(n->output(), final_shape.value());
      return;
    }
  }

  // Only shape of new shape is known, assign output rank.
  if (ConstantValueMap::HasShape(shape_name)) {
    auto output_rank = ConstantValueMap::GetShapeInto1DInt64Vector(shape_name);
    if (output_rank.has_value()) {
      TORCH_INTERNAL_ASSERT(output_rank.value().size() == 1);
      UpdateRank(n->output(), output_rank.value()[0]);
      return;
    }
  }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `has_value`, `UpdateShape`, `output`, `value`, `HasShape`, `GetShapeInto1DInt64Vector`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`has_value`, `UpdateShape`, `output`, `value`, `HasShape`, `GetShapeInto1DInt64Vector`, `...`。

### Lines 1004-1015
```cpp
  // ListConstruct is handled at the beginning of ProcessConstantValueMap, no
  // further process here.
  if (TensorTypePtr shape_type = n->input(1)->type()->cast<TensorType>()) {
    // Set rank to Reshape output if possible.
    // From shape inference, we have:
    // %4236 : Float(*, device=cpu) = onnx::Transpose[perm=[0]](%4235)
    // %4237 : Long(2, strides=[1], device=cpu) = onnx::Concat[axis=0](%4232)
    // %4238 : FloatTensor(device=cpu) = onnx::Reshape(%4236, %4237)
    // We can have it as SymbolicShape with known rank:
    // %4238 : Float(*, *, strides=[2480, 1], requires_grad=0, device=cpu) =
    // onnx::Reshape(%4236, %4237)
    auto shape_type_dim = shape_type->dim();
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `input`, `type`, `dim`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`input`, `type`, `dim`。

### Lines 1016-1036
```cpp
    if (shape_type_dim.has_value()) {
      auto shape_type_size = shape_type->sizes()[0];
      if (shape_type_size.has_value()) {
        size_t rank = shape_type_size.value();
        UpdateRank(n->output(), rank);
      }
    }
  }
}

c10::SymbolicShape ComputeShapeForSlice(
    const std::vector<c10::ShapeSymbol>& input_shape,
    const std::vector<int64_t>& start_vector,
    const std::vector<int64_t>& end_vector,
    const std::vector<int64_t>& axes_vector,
    const std::vector<int64_t>& step_vector) {
  TORCH_INTERNAL_ASSERT(axes_vector.size() <= input_shape.size());
  TORCH_INTERNAL_ASSERT(axes_vector.size() == start_vector.size());
  TORCH_INTERNAL_ASSERT(axes_vector.size() == end_vector.size());
  TORCH_INTERNAL_ASSERT(axes_vector.size() == step_vector.size());
  std::vector<c10::ShapeSymbol> final_shape;
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `has_value`, `sizes`, `value`, `UpdateRank`, `output`, `ComputeShapeForSlice`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`has_value`, `sizes`, `value`, `UpdateRank`, `output`, `ComputeShapeForSlice`, `...`。

### Lines 1037-1048
```cpp
  final_shape = input_shape;
  for (const auto idx : c10::irange(axes_vector.size())) {
    auto axis = axes_vector[idx];
    TORCH_INTERNAL_ASSERT(axis >= 0);
    if (!input_shape[axis].is_static()) {
      final_shape[axis] = c10::ShapeSymbol::newSymbol();
      continue;
    }
    auto input_shape_axis_value = input_shape[axis].static_size();
    auto cur_start = start_vector[idx];
    auto cur_end = end_vector[idx];
    auto cur_step = step_vector[idx];
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `irange`, `size`, `is_static`, `newSymbol`, `static_size`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`irange`, `size`, `is_static`, `newSymbol`, `static_size`。

### Lines 1049-1064
```cpp
    if (cur_start < -input_shape_axis_value) {
      cur_start = 0;
    } else if (cur_start < 0) {
      cur_start = input_shape_axis_value + cur_start;
    } else if (cur_start > input_shape_axis_value - 1) {
      cur_start = input_shape_axis_value;
    }
    if (cur_end < -input_shape_axis_value) {
      cur_end = -1;
    } else if (cur_end < 0) {
      cur_end = input_shape_axis_value + cur_end;
    } else if (cur_end > input_shape_axis_value - 1) {
      cur_end = input_shape_axis_value;
    }
    TORCH_INTERNAL_ASSERT(cur_step != 0);
    if (cur_step > 0) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: no dominant local symbol names.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：无明显局部符号。

### Lines 1065-1076
```cpp
      final_shape[axis] = c10::ShapeSymbol::fromStaticSize(
          (cur_end - cur_start - 1) / cur_step + 1);
    } else {
      final_shape[axis] = c10::ShapeSymbol::fromStaticSize(
          (cur_start - cur_end - 1) / (-cur_step) + 1);
    }
  }
  return c10::SymbolicShape(final_shape);
}

void ProcessSliceNode(Node* n, int opset_version) {
  bool valid = ConstantValueMap::HasShape(n->input(0)->debugName());
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `fromStaticSize`, `SymbolicShape`, `ProcessSliceNode`, `HasShape`, `input`, `debugName`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`fromStaticSize`, `SymbolicShape`, `ProcessSliceNode`, `HasShape`, `input`, `debugName`。

### Lines 1077-1088
```cpp

  // For opset version <= 9, starts, ends, axes, steps are attributes,
  // so their values are always valid.
  if (opset_version >= 10) {
    // We can only infer shapes if 'axes' is known.
    if (n->inputs().size() > 3) {
      valid = valid && ConstantValueMap::HasValue(n->input(3)->debugName());
    }
  }

  if (!valid) {
    if (ConstantValueMap::HasRank(n->input(0)->debugName())) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `inputs`, `size`, `HasValue`, `input`, `debugName`, `HasRank`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`inputs`, `size`, `HasValue`, `input`, `debugName`, `HasRank`。

### Lines 1089-1100
```cpp
      auto rank = ConstantValueMap::GetRank(n->input(0)->debugName()).value();
      UpdateRank(n->output(), rank);
    }
    return;
  } else {
    auto shape_size_0 =
        ConstantValueMap::GetShape(n->input(0)->debugName()).value();
    if (shape_size_0.rank().has_value()) {
      auto input0_shape_value = shape_size_0.sizes().value();

      std::vector<int64_t> start_vector;
      std::vector<int64_t> end_vector;
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `GetRank`, `input`, `debugName`, `value`, `UpdateRank`, `output`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`GetRank`, `input`, `debugName`, `value`, `UpdateRank`, `output`, `...`。

### Lines 1101-1113
```cpp
      std::vector<int64_t> step_vector;

      std::vector<int64_t> axes_vector(input0_shape_value.size(), 0);
      for (const auto i : c10::irange(input0_shape_value.size())) {
        axes_vector[i] = i;
      }
      if (opset_version >= 10 && n->inputs().size() > 3) {
        axes_vector = ConstantValueMap::GetValueInto1DInt64Vector(
            n->input(3)->debugName());
      } else if (opset_version < 10 && n->hasAttributeS("axes")) {
        axes_vector = n->is(attr::axes);
      }
      for (auto& axis : axes_vector) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `axes_vector`, `size`, `irange`, `inputs`, `GetValueInto1DInt64Vector`, `input`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`axes_vector`, `size`, `irange`, `inputs`, `GetValueInto1DInt64Vector`, `input`, `...`。

### Lines 1114-1125
```cpp
        if (axis < 0) {
          axis += input0_shape_value.size();
        }
      }

      if (opset_version < 10) {
        start_vector = n->is(attr::starts);
        end_vector = n->is(attr::ends);
      } else {
        // If starts, ends, or step are unknown,
        // then mark all dimensions in 'axes' as unknown.
        std::vector<uint64_t> indices = {1U, 2U, 4U};
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `size`, `is`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`size`, `is`。

### Lines 1126-1139
```cpp
        bool start_end_step_known =
            std::all_of(indices.begin(), indices.end(), [&n](auto i) {
              return (i >= n->inputs().size()) ||
                  ConstantValueMap::HasValue(n->input(i)->debugName());
            });
        if (!start_end_step_known) {
          auto final_shape = input0_shape_value;
          for (const auto axis : axes_vector) {
            final_shape[axis] = c10::ShapeSymbol::newSymbol();
          }
          UpdateShape(n->output(), final_shape);
          return;
        }

```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `all_of`, `begin`, `end`, `inputs`, `size`, `HasValue`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`all_of`, `begin`, `end`, `inputs`, `size`, `HasValue`, `...`。

### Lines 1140-1153
```cpp
        start_vector = ConstantValueMap::GetValueInto1DInt64Vector(
            n->input(1)->debugName());
        end_vector = ConstantValueMap::GetValueInto1DInt64Vector(
            n->input(2)->debugName());
        if (n->inputs().size() > 4) {
          step_vector = ConstantValueMap::GetValueInto1DInt64Vector(
              n->input(4)->debugName());
        }
      }

      if (step_vector.empty()) {
        step_vector = std::vector<int64_t>(axes_vector.size(), 1);
      }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `GetValueInto1DInt64Vector`, `input`, `debugName`, `inputs`, `size`, `empty`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`GetValueInto1DInt64Vector`, `input`, `debugName`, `inputs`, `size`, `empty`。

### Lines 1154-1165
```cpp
      auto final_shape = ComputeShapeForSlice(
          input0_shape_value,
          start_vector,
          end_vector,
          axes_vector,
          step_vector);
      UpdateShape(n->output(), final_shape);
    }
  }
}

void ProcessUnchangeNode(Node* n) {
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `ComputeShapeForSlice`, `UpdateShape`, `output`, `ProcessUnchangeNode`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`ComputeShapeForSlice`, `UpdateShape`, `output`, `ProcessUnchangeNode`。

### Lines 1166-1179
```cpp
  if (ConstantValueMap::HasShape(n->input(0)->debugName())) {
    auto shape_size_0 =
        ConstantValueMap::GetShape(n->input(0)->debugName()).value();
    UpdateShape(n->output(), shape_size_0);
  }
}

void ProcessTimeSeriesNode(Node* n) {
  auto input0_shape = ConstantValueMap::GetShape(n->input(0)->debugName());
  auto input1_shape = ConstantValueMap::GetShape(n->input(1)->debugName());
  if (!(input0_shape.has_value() && input1_shape.has_value())) {
    return;
  }
  auto input0_shape_value = input0_shape.value().sizes();
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `HasShape`, `input`, `debugName`, `GetShape`, `value`, `UpdateShape`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`HasShape`, `input`, `debugName`, `GetShape`, `value`, `UpdateShape`, `...`。

### Lines 1180-1192
```cpp
  auto input1_shape_value = input1_shape.value().sizes();
  c10::ShapeSymbol seq_length;
  c10::ShapeSymbol num_directions;
  c10::ShapeSymbol batch_size;
  c10::ShapeSymbol hidden_size;
  if (input0_shape_value.has_value()) {
    seq_length = input0_shape_value.value()[0];
    batch_size = input0_shape_value.value()[1];
  }

  if (input1_shape_value.has_value()) {
    num_directions = input1_shape_value.value()[0];
    if (input1_shape_value.value()[1].is_static()) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `value`, `sizes`, `has_value`, `is_static`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`value`, `sizes`, `has_value`, `is_static`。

### Lines 1193-1206
```cpp
      auto input1_value = input1_shape_value.value()[1].static_size();
      switch (n->kind()) {
        case ::c10::onnx::RNN:
          hidden_size = c10::ShapeSymbol::fromStaticSize(input1_value);
          break;
        case ::c10::onnx::LSTM:
          hidden_size = c10::ShapeSymbol::fromStaticSize(input1_value / 4);
          break;
        case ::c10::onnx::GRU:
          hidden_size = c10::ShapeSymbol::fromStaticSize(input1_value / 3);
          break;
        default:
          throw std::runtime_error(
              std::string() + "This is not a valid TimeSeries Node with type " +
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `value`, `static_size`, `kind`, `fromStaticSize`, `runtime_error`, `string`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`value`, `static_size`, `kind`, `fromStaticSize`, `runtime_error`, `string`。

### Lines 1207-1219
```cpp
              n->kind().toDisplayString());
      }
    } else {
      hidden_size = c10::ShapeSymbol::newSymbol();
    }
  }

  if (n->outputs().size() > 1) {
    std::vector<c10::ShapeSymbol> final_shape = {
        seq_length, num_directions, batch_size, hidden_size};
    UpdateShape(n->output(0), c10::SymbolicShape(final_shape));
  }
  for (const auto idx : c10::irange(2U, 4U)) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures. Key symbols: `kind`, `toDisplayString`, `newSymbol`, `outputs`, `size`, `UpdateShape`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构。关键符号：`kind`, `toDisplayString`, `newSymbol`, `outputs`, `size`, `UpdateShape`, `...`。

### Lines 1220-1233
```cpp
    if (n->outputs().size() > idx) {
      std::vector<c10::ShapeSymbol> final_shape = {
          num_directions, batch_size, hidden_size};
      UpdateShape(n->output(idx - 1), c10::SymbolicShape(final_shape));
    }
  }
}

void ProcessUnsqueezeNode(Node* n) {
  TensorTypePtr output_type = n->output(0)->type()->cast<TensorType>();
  if (output_type == nullptr) {
    return;
  }
  if (output_type->dim().has_value() && output_type->dim().value() == 1 &&
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `outputs`, `size`, `UpdateShape`, `output`, `SymbolicShape`, `ProcessUnsqueezeNode`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`outputs`, `size`, `UpdateShape`, `output`, `SymbolicShape`, `ProcessUnsqueezeNode`, `...`。

### Lines 1234-1246
```cpp
      ConstantValueMap::HasShapeValue(n->input(0)->debugName())) {
    auto shape_value =
        ConstantValueMap::GetShapeValue(n->input(0)->debugName()).value();
    // When the scalar represents a shape, it is the same as the shape value
    // when it gets unsqueezed.
    ConstantValueMap::SetShapeValue(n->output()->debugName(), shape_value);
  }
}

// As an addition to onnx shape inference, this function leverages constant
// folding and a per-Op based process to update rank/shape for the graph, also
// it update ConstantValueMap accordingly.
void ComputeConstant(Node* n, int opset_version) {
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `HasShapeValue`, `input`, `debugName`, `GetShapeValue`, `value`, `SetShapeValue`, `...`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`HasShapeValue`, `input`, `debugName`, `GetShapeValue`, `value`, `SetShapeValue`, `...`。

### Lines 1247-1259
```cpp
  if (n->kind() == ::c10::onnx::Constant) {
    if (n->kindOf(attr::value) == AttributeKind::t) {
      const at::Tensor& const_val = n->t(attr::value);
      at::Tensor const_val_copy =
          at::empty(const_val.sizes(), const_val.options());
      const_val_copy.copy_(const_val);
      ConstantValueMap::SetValue(n->output()->debugName(), const_val_copy);
    }
    return;
  }
  auto only_rank_available = false;
  size_t rank = 0;

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `kind`, `kindOf`, `t`, `empty`, `sizes`, `options`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`kind`, `kindOf`, `t`, `empty`, `sizes`, `options`, `...`。

### Lines 1260-1283
```cpp
  // Constant folding.
  auto const_fold_val = ComputeConstantFolding(n, opset_version);
  if (const_fold_val.has_value()) {
    at::Tensor const_fold_val_copy = at::empty(
        const_fold_val.value().sizes(), const_fold_val.value().options());
    const_fold_val_copy.copy_(const_fold_val.value());
    ConstantValueMap::SetValue(n->output()->debugName(), const_fold_val_copy);
    UpdateShapeFromVector(n->output(), const_fold_val_copy.sizes().vec());
    return;
  }

  switch (n->kind()) {
    case ::c10::onnx::Add:
    case ::c10::onnx::Div:
    case ::c10::onnx::Equal:
    case ::c10::onnx::Greater:
    case ::c10::onnx::GreaterOrEqual:
    case ::c10::onnx::Less:
    case ::c10::onnx::LessOrEqual:
    case ::c10::onnx::Mod:
    case ::c10::onnx::Mul:
    case ::c10::onnx::Pow:
    case ::c10::onnx::Sub: {
      ProcessBroadcastNode(n);
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `ComputeConstantFolding`, `has_value`, `empty`, `value`, `sizes`, `options`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`ComputeConstantFolding`, `has_value`, `empty`, `value`, `sizes`, `options`, `...`。

### Lines 1284-1301
```cpp
      break;
    }
    case ::c10::onnx::Shape: {
      auto input_shape =
          ConstantValueMap::GetShapeInto1DInt64Vector(n->input()->debugName());
      if (input_shape.has_value()) {
        auto shape_value = input_shape.value();
        // TODO: getDevice() ?
        auto options = c10::TensorOptions().dtype(at::kLong).device(at::kCPU);
        auto shape_value_size = static_cast<int64_t>(shape_value.size());
        auto f =
            at::from_blob(shape_value.data(), {shape_value_size}, at::kLong)
                .to(at::kCPU);
        // Need copy here
        at::Tensor f_copy = at::empty({shape_value_size}, options);
        f_copy.copy_(f);
        ConstantValueMap::SetValue(n->output()->debugName(), f_copy);
        std::vector<::c10::ShapeSymbol> final_shape_vector(
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `GetShapeInto1DInt64Vector`, `input`, `debugName`, `has_value`, `value`, `TensorOptions`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`GetShapeInto1DInt64Vector`, `input`, `debugName`, `has_value`, `value`, `TensorOptions`, `...`。

### Lines 1302-1313
```cpp
            1, c10::ShapeSymbol::fromStaticSize(shape_value_size));
        ::c10::SymbolicShape final_shape(final_shape_vector);
        UpdateShape(n->output(), final_shape);
      }
      break;
    }
    case ::c10::onnx::Reshape: {
      ProcessReshapeNode(n, opset_version);
      break;
    }
    case ::c10::onnx::Transpose: {
      if (n->hasAttributeS("perm")) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `fromStaticSize`, `final_shape`, `UpdateShape`, `output`, `ProcessReshapeNode`, `hasAttributeS`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`fromStaticSize`, `final_shape`, `UpdateShape`, `output`, `ProcessReshapeNode`, `hasAttributeS`。

### Lines 1314-1326
```cpp
        auto perm_v = n->is(attr::perm);
        rank = perm_v.size();
        auto is_default_perm = false;
        if (rank == 2 && perm_v[0] == 1 && perm_v[1] == 0) {
          is_default_perm = true;
        }
        auto shape_updated = false;
        if (ConstantValueMap::HasShape(n->input(0)->debugName())) {
          auto shape_size_0 =
              ConstantValueMap::GetShape(n->input(0)->debugName())
                  .value()
                  .sizes();
          if (shape_size_0.has_value()) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `is`, `size`, `HasShape`, `input`, `debugName`, `GetShape`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`is`, `size`, `HasShape`, `input`, `debugName`, `GetShape`, `...`。

### Lines 1327-1345
```cpp
            auto shape_vector_0 = shape_size_0.value();
            std::vector<::c10::ShapeSymbol> final_shape_vector(
                shape_vector_0.size(), ::c10::ShapeSymbol());
            if (is_default_perm) {
              std::reverse_copy(
                  std::begin(shape_vector_0),
                  std::end(shape_vector_0),
                  std::begin(final_shape_vector));
            } else {
              for (const auto i : c10::irange(shape_vector_0.size())) {
                final_shape_vector[i] = shape_vector_0[perm_v[i]];
              }
            }
            ::c10::SymbolicShape final_shape(final_shape_vector);
            UpdateShape(n->output(), final_shape);
            shape_updated = true;
          }
        }
        if (!shape_updated) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `value`, `final_shape_vector`, `size`, `ShapeSymbol`, `reverse_copy`, `begin`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`value`, `final_shape_vector`, `size`, `ShapeSymbol`, `reverse_copy`, `begin`, `...`。

### Lines 1346-1361
```cpp
          if (!is_default_perm) {
            only_rank_available = true;
          } else if (ConstantValueMap::HasRank(n->input(0)->debugName())) {
            rank = ConstantValueMap::GetRank(n->input(0)->debugName()).value();
            only_rank_available = true;
          }
        }
      }
      break;
    }
    case ::c10::onnx::Concat: {
      ProcessConcatNode(n);
      break;
    }
    case ::c10::onnx::ConstantOfShape: {
      if (ConstantValueMap::HasValue(n->input()->debugName())) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `HasRank`, `input`, `debugName`, `GetRank`, `value`, `ProcessConcatNode`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`HasRank`, `input`, `debugName`, `GetRank`, `value`, `ProcessConcatNode`, `...`。

### Lines 1362-1380
```cpp
        auto shape_temp = ConstantValueMap::GetValueInto1DInt64Vector(
            n->input()->debugName());
        UpdateShapeFromVector(n->output(), shape_temp);
        if (!shape_temp.empty()) {
          if (n->hasAttributeS("value")) {
            auto value = n->t(attr::value).repeat(shape_temp);
            ConstantValueMap::SetValue(n->output()->debugName(), value);
          } else {
            auto options =
                c10::TensorOptions().dtype(at::kFloat).device(at::kCPU);
            auto value = at::full({1}, 0.0, options).repeat(shape_temp);
            ConstantValueMap::SetValue(n->output()->debugName(), value);
          }
        }
      }
      break;
    }
    case ::c10::onnx::Expand: {
      if (ConstantValueMap::HasShape(n->input(0)->debugName())) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `GetValueInto1DInt64Vector`, `input`, `debugName`, `UpdateShapeFromVector`, `output`, `empty`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`GetValueInto1DInt64Vector`, `input`, `debugName`, `UpdateShapeFromVector`, `output`, `empty`, `...`。

### Lines 1381-1392
```cpp
        auto input0_shape_size =
            ConstantValueMap::GetShape(n->input(0)->debugName())
                .value()
                .sizes();
        if (input0_shape_size.has_value()) {
          const auto& input0_shape_value = input0_shape_size.value();
          if (ConstantValueMap::HasValue(n->input(1)->debugName())) {
            // When value of `shape` is statically known,
            // output shape can be computed.
            auto shape_temp = ConstantValueMap::GetValueInto1DInt64Vector(
                n->input(1)->debugName());
            auto final_shape =
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `GetShape`, `input`, `debugName`, `value`, `sizes`, `has_value`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`GetShape`, `input`, `debugName`, `value`, `sizes`, `has_value`, `...`。

### Lines 1393-1407
```cpp
                ComputeShapeFromExpand(input0_shape_value, shape_temp);
            if (final_shape.has_value()) {
              UpdateShape(n->output(), final_shape.value());
            }
          } else if (
              auto expand_shape =
                  ConstantValueMap::GetShapeInto1DInt64VectorWithOneUnknown(
                      n->input(1)->debugName())) {
            // When shape of `shape` is statically known,
            // output rank can be computed.
            TORCH_INTERNAL_ASSERT(
                expand_shape.value().size() == 1,
                "`Shape` input to `Expand` should be a 1-D tensor. Instead got rank ",
                expand_shape.value().size());
            if (expand_shape.value()[0] > 0) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `ComputeShapeFromExpand`, `has_value`, `UpdateShape`, `output`, `value`, `GetShapeInto1DInt64VectorWithOneUnknown`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`ComputeShapeFromExpand`, `has_value`, `UpdateShape`, `output`, `value`, `GetShapeInto1DInt64VectorWithOneUnknown`, `...`。

### Lines 1408-1421
```cpp
              std::vector<c10::ShapeSymbol> final_shape;
              std::generate_n(
                  std::back_inserter(final_shape),
                  expand_shape.value()[0],
                  ::c10::ShapeSymbol::newSymbol);
              UpdateShape(n->output(), c10::SymbolicShape(final_shape));
            }
          }
        }
      }
      break;
    }
    case ::c10::onnx::NonZero: {
      if (ConstantValueMap::HasRank(n->input()->debugName())) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `generate_n`, `back_inserter`, `value`, `UpdateShape`, `output`, `SymbolicShape`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`generate_n`, `back_inserter`, `value`, `UpdateShape`, `output`, `SymbolicShape`, `...`。

### Lines 1422-1433
```cpp
        auto rank = ConstantValueMap::GetRank(n->input()->debugName()).value();
        std::vector<c10::ShapeSymbol> dims;
        dims.emplace_back(
            c10::ShapeSymbol::fromStaticSize(static_cast<int64_t>(rank)));
        auto input_node = n->input()->node();
        if (input_node->kind() == ::c10::onnx::ConstantOfShape) {
          if (input_node->hasAttributeS("value")) {
            auto value =
                input_node->t(attr::value).toType(at::ScalarType::Float);
            auto value_a = value.accessor<float, 1>();
            if (value_a.size(0) == 1 && std::abs(value_a[0]) > 1e-6) {
              if (ConstantValueMap::HasShape(n->input()->debugName())) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `GetRank`, `input`, `debugName`, `value`, `emplace_back`, `fromStaticSize`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`GetRank`, `input`, `debugName`, `value`, `emplace_back`, `fromStaticSize`, `...`。

### Lines 1434-1449
```cpp
                auto shape_size_0 =
                    ConstantValueMap::GetShape(n->input()->debugName()).value();
                if (shape_size_0.isComplete()) {
                  auto shape_vector_0 = shape_size_0.sizes().value();
                  int64_t num_elements = 1;
                  for (auto cur_dim : shape_vector_0) {
                    num_elements *= cur_dim.static_size();
                  }
                  dims.emplace_back(
                      c10::ShapeSymbol::fromStaticSize(num_elements));
                }
              }
            }
          }
        }
        if (dims.size() == 1) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `GetShape`, `input`, `debugName`, `value`, `isComplete`, `sizes`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`GetShape`, `input`, `debugName`, `value`, `isComplete`, `sizes`, `...`。

### Lines 1450-1473
```cpp
          dims.emplace_back(c10::ShapeSymbol::newSymbol());
        }
        c10::SymbolicShape shape_v(dims);
        UpdateShape(n->output(), shape_v);
      }
      break;
    }
    case ::c10::onnx::MatMul: {
      ProcessMatMulNode(n);
      break;
    }
    case ::c10::onnx::ReduceMean:
    case ::c10::onnx::ReduceProd: {
      ProcessReduceNode(n);
      break;
    }
    case ::c10::onnx::RNN:
    case ::c10::onnx::LSTM:
    case ::c10::onnx::GRU: {
      ProcessTimeSeriesNode(n);
      break;
    }
    case ::c10::onnx::Size: {
      if (ConstantValueMap::HasShape(n->input(0)->debugName())) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `emplace_back`, `newSymbol`, `shape_v`, `UpdateShape`, `output`, `ProcessMatMulNode`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`emplace_back`, `newSymbol`, `shape_v`, `UpdateShape`, `output`, `ProcessMatMulNode`, `...`。

### Lines 1474-1490
```cpp
        auto input0_shape_size =
            ConstantValueMap::GetShape(n->input(0)->debugName())
                .value()
                .sizes();
        if (input0_shape_size.has_value()) {
          const auto& input0_shape_value = input0_shape_size.value();
          int64_t total_size = 1;
          auto is_full_static = true;
          for (const auto i : c10::irange(input0_shape_value.size())) {
            if (input0_shape_value[i].is_static()) {
              total_size *= input0_shape_value[i].static_size();
            } else {
              is_full_static = false;
              break;
            }
          }
          if (is_full_static) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `GetShape`, `input`, `debugName`, `value`, `sizes`, `has_value`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`GetShape`, `input`, `debugName`, `value`, `sizes`, `has_value`, `...`。

### Lines 1491-1509
```cpp
            auto f_final = onnx_constant_fold::IntToTensor(total_size);
            ConstantValueMap::SetValue(n->output(0)->debugName(), f_final);
          }
        }
      }
      break;
    }
    case ::c10::onnx::Slice: {
      ProcessSliceNode(n, opset_version);
      break;
    }
    case ::c10::onnx::Cast:
    case ::c10::onnx::Relu:
    case ::c10::onnx::Softmax: {
      ProcessUnchangeNode(n);
      break;
    }
    case ::c10::onnx::Tile: {
      if (ConstantValueMap::HasShape(n->input(0)->debugName())) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `IntToTensor`, `SetValue`, `output`, `debugName`, `ProcessSliceNode`, `ProcessUnchangeNode`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`IntToTensor`, `SetValue`, `output`, `debugName`, `ProcessSliceNode`, `ProcessUnchangeNode`, `...`。

### Lines 1510-1521
```cpp
        auto input0_shape_size =
            ConstantValueMap::GetShape(n->input(0)->debugName())
                .value()
                .sizes();
        if (input0_shape_size.has_value()) {
          const auto& input0_shape_value = input0_shape_size.value();
          if (ConstantValueMap::HasValue(n->input(1)->debugName())) {
            auto shape_temp = ConstantValueMap::GetValueInto1DInt64Vector(
                n->input(1)->debugName());
            auto final_shape =
                ComputeShapeFromTile(input0_shape_value, shape_temp);
            if (final_shape.has_value()) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `GetShape`, `input`, `debugName`, `value`, `sizes`, `has_value`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`GetShape`, `input`, `debugName`, `value`, `sizes`, `has_value`, `...`。

### Lines 1522-1537
```cpp
              UpdateShape(n->output(), final_shape.value());
            }
          }
        }
      }
      break;
    }
    case ::c10::onnx::Unsqueeze: {
      ProcessUnsqueezeNode(n);
      break;
    }
    default: {
      break;
    }
  }
  if (n->outputs().size() > 1 ||
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `UpdateShape`, `output`, `value`, `ProcessUnsqueezeNode`, `outputs`, `size`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`UpdateShape`, `output`, `value`, `ProcessUnsqueezeNode`, `outputs`, `size`。

### Lines 1538-1549
```cpp
      ConstantValueMap::HasShape(n->output(0)->debugName())) {
    return;
  }
  if (only_rank_available) {
    UpdateRank(n->output(), rank);
  }
}

bool IsListConstructIntType(const Value* v) {
  if (v->node()->kind() == prim::ListConstruct) {
    auto listType = v->node()->output()->type();
    auto containedType = listType->containedTypes().at(0);
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `HasShape`, `output`, `debugName`, `UpdateRank`, `IsListConstructIntType`, `node`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`HasShape`, `output`, `debugName`, `UpdateRank`, `IsListConstructIntType`, `node`, `...`。

### Lines 1550-1561
```cpp
    if (containedType == IntType::get()) {
      return true;
    }
  }
  return false;
}

// Check if all graph inputs are static and allow a cached value to return.
// Since this traverses all inputs of the graph (including weights), it can be
// costly for large graphs. Since this is called for each node in an export,
// and the inputs remain unchanged, we can cut down export time by caching.
bool AllGraphInputsStaticWithCaching(const Graph* g) {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `get`, `AllGraphInputsStaticWithCaching`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`get`, `AllGraphInputsStaticWithCaching`。

### Lines 1562-1577
```cpp
  auto maybe_is_static = ConstantValueMap::GetAllGraphInputsStatic();
  if (maybe_is_static.has_value()) {
    return maybe_is_static.value();
  } else {
    bool ret = AllGraphInputsStatic(g);
    ConstantValueMap::SetAllGraphInputsStatic(ret);
    return ret;
  }
}

void ProcessConstantValueMap(Node* n, int opset_version) {
  // Update ConstantValueMap on node outputs from onnx shape inference
  // For outputs, only update static shapes. For input, we update symbolic
  // shapes also. ONNX If can have different types on different branches, skip
  // here.

```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `GetAllGraphInputsStatic`, `has_value`, `value`, `AllGraphInputsStatic`, `SetAllGraphInputsStatic`, `ProcessConstantValueMap`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`GetAllGraphInputsStatic`, `has_value`, `value`, `AllGraphInputsStatic`, `SetAllGraphInputsStatic`, `ProcessConstantValueMap`。

### Lines 1578-1589
```cpp
  // Update the shape reliability for each node before processing
  // ConstantValueMap to prevent unreliable nodes from producing static
  // shapes
  UpdateReliable(n);

  auto static_input_shape = AllGraphInputsStaticWithCaching(n->owningGraph());
  for (auto i : c10::irange(n->outputs().size())) {
    if (TensorTypePtr output_type = n->output(i)->type()->cast<TensorType>()) {
      if (output_type->dim().has_value()) {
        size_t rank = static_cast<size_t>(output_type->dim().value());
        ConstantValueMap::SetRank(n->output(i)->debugName(), rank);
        auto shape = output_type->symbolic_sizes();
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `UpdateReliable`, `AllGraphInputsStaticWithCaching`, `owningGraph`, `irange`, `outputs`, `size`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`UpdateReliable`, `AllGraphInputsStaticWithCaching`, `owningGraph`, `irange`, `outputs`, `size`, `...`。

### Lines 1590-1601
```cpp
        if (shape.isComplete()) {
          UpdateShape(n->output(i), shape);
        }
      }
    }
  }
  // Update ConstantValueMap on node inputs from onnx shape inference.
  // ListConstruct is handled here (we only consider IntType, not TensorType) ,
  // no need to have a per-op based process.
  for (auto i : c10::irange(n->inputs().size())) {
    if (TensorTypePtr input_type = n->input(i)->type()->cast<TensorType>()) {
      if (input_type->dim().has_value()) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `isComplete`, `UpdateShape`, `output`, `irange`, `inputs`, `size`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`isComplete`, `UpdateShape`, `output`, `irange`, `inputs`, `size`, `...`。

### Lines 1602-1618
```cpp
        size_t rank = static_cast<size_t>(input_type->dim().value());
        ConstantValueMap::SetRank(n->input(i)->debugName(), rank);
        // Only update shape if the input is onnx node.
        // If it is aten operators, for example,
        //   Float(20, 20, strides=[1, 0], requires_grad=0, device=cpu),
        //     %399 : Float(20, 20, strides=[0, 1], requires_grad=0, device=cpu)
        //     = prim::ListUnpack(%397)
        // The tracer shape may not be correct when dynamic_axes is enabled.
        if (n->input(i)->node()->kind().is_onnx() || static_input_shape) {
          auto shape = input_type->symbolic_sizes();
          if (!ConstantValueMap::HasShape(n->input(i)->debugName())) {
            UpdateShape(n->input(i), shape);
          }
        }
      }
    } else if (IsListConstructIntType(n->input(i))) {
      auto lc_node = n->input(i)->node();
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `dim`, `value`, `SetRank`, `input`, `debugName`, `node`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`dim`, `value`, `SetRank`, `input`, `debugName`, `node`, `...`。

### Lines 1619-1641
```cpp
      auto rank = lc_node->inputs().size();
      auto lc_vector_optional = GetValueFromListConstructNode(lc_node);
      if (lc_vector_optional.has_value()) {
        auto lc_vector = lc_vector_optional.value();
        auto options = c10::TensorOptions().dtype(at::kLong).device(at::kCPU);
        auto lc_vector_size = static_cast<int64_t>(lc_vector.size());
        auto f = at::from_blob(lc_vector.data(), {lc_vector_size}, at::kLong)
                     .to(at::kCPU);
        // Need copy here
        at::Tensor f_copy = at::empty({lc_vector_size}, options);
        f_copy.copy_(f);
        ConstantValueMap::SetValue(n->input(i)->debugName(), f_copy);
        UpdateShapeFromVector(n->input(i), {lc_vector_size});
      } else {
        UpdateShapeFromVector(n->input(i), {static_cast<int64_t>(rank)});
      }
      SetShapeValueFromListConstructNode(lc_node);
    }
  }
  // Additional logic to update the graph and ConstantValueMap
  ComputeConstant(n, opset_version);
}

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `inputs`, `size`, `GetValueFromListConstructNode`, `has_value`, `value`, `TensorOptions`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`inputs`, `size`, `GetValueFromListConstructNode`, `has_value`, `value`, `TensorOptions`, `...`。

### Lines 1642-1653
```cpp
// Any additional post process that are specific to individual node kind.
void SpecialPostProcess(Node* n) {
  switch (n->kind()) {
    case ::c10::onnx::SequenceInsert: {
      // Special case when input sequence to SequenceInsert is empty.
      // onnx Sequence type requires element type to be set.
      // If the list to insert is empty, we set the elem type by
      // looking at the tensor being inserted.
      auto seq_node = n->input(0)->node();
      auto t_type = n->input(1)->type()->cast<TensorType>();

      auto update_sequence_empty_dtype = [](Node* n,
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `SpecialPostProcess`, `kind`, `input`, `node`, `type`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`SpecialPostProcess`, `kind`, `input`, `node`, `type`。

### Lines 1654-1665
```cpp
                                            const TensorTypePtr& t_type) {
        TORCH_INTERNAL_ASSERT(n && n->kind() == ::c10::onnx::SequenceEmpty);
        TORCH_INTERNAL_ASSERT(t_type && t_type->scalarType().has_value());
        auto scalar_type = t_type->scalarType().value();
        auto onnx_type = ATenTypeToOnnxType(scalar_type);
        n->i_(attr::dtype, onnx_type);
        n->output()->setType(ListType::create(t_type));
      };

      auto find_sequence_empty = [](Value* input,
                                    const TensorTypePtr& t_type) -> Node* {
        auto find_sequence_empty_impl =
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `kind`, `scalarType`, `has_value`, `value`, `ATenTypeToOnnxType`, `i_`, `...`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`kind`, `scalarType`, `has_value`, `value`, `ATenTypeToOnnxType`, `i_`, `...`。

### Lines 1666-1679
```cpp
            [](Value* input,
               const TensorTypePtr& t_type,
               auto& find_sequence_empty_ref) -> Node* {
          auto input_node = input->node();
          TORCH_INTERNAL_ASSERT(input_node);

          // 1. Input is from SequenceEmpty.
          if (input_node->kind() == ::c10::onnx::SequenceEmpty) {
            return input_node;
          }

          // 2. Input is subblock input of a Loop node, which takes outer block
          // SequenceEmpty as input.
          if (input_node->kind() == prim::Param) {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `node`, `kind`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`node`, `kind`。

### Lines 1680-1691
```cpp
            auto loop_n = input_node->owningBlock()->owningNode();
            if (nullptr == loop_n || loop_n->kind() != ::c10::onnx::Loop) {
              return nullptr;
            }

            auto it = std::find(
                input_node->outputs().begin(),
                input_node->outputs().end(),
                input);
            auto idx = std::distance(input_node->outputs().begin(), it);

            auto outer_block_node = loop_n->input(idx)->node();
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `owningBlock`, `owningNode`, `kind`, `find`, `outputs`, `begin`, `...`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`owningBlock`, `owningNode`, `kind`, `find`, `outputs`, `begin`, `...`。

### Lines 1692-1705
```cpp
            if (outer_block_node &&
                outer_block_node->kind() == ::c10::onnx::SequenceEmpty) {
              // Found SequenceEmpty
              input->setType(ListType::create(t_type));
              return outer_block_node;
            } else {
              // Outer block node still not SequenceEmpty, call recursively in
              // case of nested loop.
              auto found_n = find_sequence_empty_ref(
                  loop_n->input(idx), t_type, find_sequence_empty_ref);
              if (found_n) {
                input->setType(ListType::create(t_type));
              }
              return found_n;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `kind`, `setType`, `create`, `find_sequence_empty_ref`, `input`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`kind`, `setType`, `create`, `find_sequence_empty_ref`, `input`。

### Lines 1706-1717
```cpp
            }
          }

          // Could not find source SequenceEmpty node.
          return nullptr;
        };
        return find_sequence_empty_impl(
            input, t_type, find_sequence_empty_impl);
      };

      if (seq_node && t_type && t_type->scalarType()) {
        if (seq_node->kind() == ::c10::onnx::SequenceEmpty) {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `find_sequence_empty_impl`, `scalarType`, `kind`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`find_sequence_empty_impl`, `scalarType`, `kind`。

### Lines 1718-1738
```cpp
          update_sequence_empty_dtype(seq_node, t_type);
        } else if (seq_node->kind() == prim::Param) {
          // Try to find original onnx::SequenceEmpty node in outer block.
          auto seq_empty_n = find_sequence_empty(n->input(0), t_type);
          if (seq_empty_n) {
            update_sequence_empty_dtype(seq_empty_n, t_type);
          }
        }
        n->output()->setType(ListType::create(t_type));
      }

      break;
    }
    case ::c10::onnx::Cast: {
      // ONNX shape inference is not able to assign output tensor shape,
      // when input to onnx::Cast has incomplete tensor shape, for example
      // missing shape, rank, dtype, etc. This postprocess sets the correct
      // dtype for output tensor, since the dtype info is stored in Cast
      // attribute.
      TensorTypePtr t_type = n->output()->type()->cast<TensorType>();
      if (nullptr != t_type && !t_type->scalarType().has_value()) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `update_sequence_empty_dtype`, `kind`, `find_sequence_empty`, `input`, `output`, `setType`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`update_sequence_empty_dtype`, `kind`, `find_sequence_empty`, `input`, `output`, `setType`, `...`。

### Lines 1739-1752
```cpp
        auto onnx_dtype = n->i(attr::to);
        auto aten_dtype = ONNXTypeToATenType(onnx_dtype);
        n->output()->setType(t_type->withScalarType(aten_dtype));
      }
      break;
    }
    case ::c10::onnx::ConstantOfShape: {
      // ONNX shape inference is not able to propagate output tensor shape
      // for onnx::ConstantOfShape if input `shape` is not constant.
      // This is a temporary solution when some partial information is
      // available, for example, knowing rank of output tensor, or knowing
      // symbolic shape. This solution won't be needed once we have proper
      // symbolic propagation.
      auto shape_node = n->input(0)->node();
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `i`, `ONNXTypeToATenType`, `output`, `setType`, `withScalarType`, `input`, `...`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`i`, `ONNXTypeToATenType`, `output`, `setType`, `withScalarType`, `input`, `...`。

### Lines 1753-1773
```cpp
      if (shape_node->kind() == ::c10::onnx::Shape) {
        // Shape -> ConstantOfShape
        auto orig_type = shape_node->input()->type()->cast<TensorType>();
        auto v_type = n->output()->type()->cast<TensorType>();
        if (v_type && !v_type->sizes().concrete_sizes()) {
          if (orig_type && orig_type->dim()) {
            // Assign symbolic shape of original input of onnx::Shape.
            v_type = v_type->withSymbolicShapes(orig_type->symbolic_sizes());
            n->output()->setType(v_type);
          } else if (
              shape_node->input()->node()->kind() ==
              ::c10::prim::ListConstruct) {
            // Assign rank of original input of onnx::Shape.
            v_type = v_type->withSizes({static_cast<int64_t>(
                shape_node->input()->node()->inputs().size())});
            n->output()->setType(v_type);
          }
        }
      } else if (shape_node->kind() == ::c10::prim::ListConstruct) {
        // ListConstruct -> ConstantOfShape
        auto v_type = n->output()->type()->cast<TensorType>();
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `kind`, `input`, `type`, `output`, `sizes`, `concrete_sizes`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`kind`, `input`, `type`, `output`, `sizes`, `concrete_sizes`, `...`。

### Lines 1774-1786
```cpp
        if (v_type && !v_type->sizes().concrete_sizes()) {
          auto value = n->t(attr::value);
          v_type = v_type->withScalarType(value.scalar_type());
          std::vector<c10::ShapeSymbol> sizes(
              shape_node->inputs().size(), c10::ShapeSymbol::newSymbol());
          v_type = v_type->withSymbolicShapes(c10::SymbolicShape(sizes));
          n->output()->setType(v_type);
        }
      }
      break;
    }
    case ::c10::onnx::If: {
      if (!IsValidONNXControlflowNode(n)) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `sizes`, `concrete_sizes`, `t`, `withScalarType`, `scalar_type`, `inputs`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`sizes`, `concrete_sizes`, `t`, `withScalarType`, `scalar_type`, `inputs`, `...`。

### Lines 1787-1801
```cpp
        break;
      }
      FixupONNXControlflowNodeOutputs(n);
      break;
    }
    case ::c10::onnx::Loop: {
      if (!IsValidONNXControlflowNode(n)) {
        break;
      }
      FixupONNXControlflowNodeOutputs(n);
      break;
    }
  }
}

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `FixupONNXControlflowNodeOutputs`, `IsValidONNXControlflowNode`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`FixupONNXControlflowNodeOutputs`, `IsValidONNXControlflowNode`。

### Lines 1802-1813
```cpp
void UpdateOutputTypeByONNXProto(
    Node* n,
    Node* clone_node,
    const onnx::ModelProto& model_proto,
    SymbolDimMap& symbol_dim_map,
    DimSymbolMap& dim_symbol_map) {
  const auto& graph_proto = model_proto.graph();

  // get data from value_info and updated original graph.
  const auto updateNodeOutputsByONNXValueInfo =
      [&](const onnx::ValueInfoProto& v_info) {
        for (size_t i = 0; i < n->outputs().size(); ++i) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `UpdateOutputTypeByONNXProto`, `graph`, `outputs`, `size`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`UpdateOutputTypeByONNXProto`, `graph`, `outputs`, `size`。

### Lines 1814-1825
```cpp
          if (clone_node->output(i)->debugName() == v_info.name()) {
            UpdateTorchValueByOnnxValueInfo(
                n->output(i), v_info, symbol_dim_map, dim_symbol_map);
          }
        }
      };

  // Check graph outputs for inferred shapes.
  for (const auto i : c10::irange(graph_proto.output_size())) {
    updateNodeOutputsByONNXValueInfo(graph_proto.output(i));
  }

```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `output`, `debugName`, `name`, `UpdateTorchValueByOnnxValueInfo`, `irange`, `output_size`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`output`, `debugName`, `name`, `UpdateTorchValueByOnnxValueInfo`, `irange`, `output_size`, `...`。

### Lines 1826-1841
```cpp
  // Check value_infos for inferred shapes.
  for (const auto i : c10::irange(graph_proto.value_info_size())) {
    updateNodeOutputsByONNXValueInfo(graph_proto.value_info(i));
  }
}

void FetchBlockInputMetadataFromParent(Block* b) {
  auto n = b->owningNode();
  if (nullptr != n && n->kind() == ::c10::onnx::Loop) {
    // Copy node input metadata to subgraph input.
    for (size_t i = 0; i < n->inputs().size(); ++i) {
      b->inputs().at(i)->setType(n->inputs().at(i)->type());
    }
  }
}

```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `irange`, `value_info_size`, `updateNodeOutputsByONNXValueInfo`, `value_info`, `FetchBlockInputMetadataFromParent`, `owningNode`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`irange`, `value_info_size`, `updateNodeOutputsByONNXValueInfo`, `value_info`, `FetchBlockInputMetadataFromParent`, `owningNode`, `...`。

### Lines 1842-1854
```cpp
void RemoveProcessedInputs(const Node* n) {
  // After processing a node for shape inference, remove intermediate tensors
  // that are stored in ConstantValueMap to reduce memory usage.
  // This will only remove tensors that are no longer needed by any other node.

  // Returns whether a node was already processed for shape inference.
  const auto isNodeProcessed = [](const Node* node) {
    const auto& outputs = node->outputs();
    return std::any_of(outputs.begin(), outputs.end(), [](const Value* output) {
      // Assumes shape inference can at least determine the rank of the outputs.
      // If this assumption is wrong, some intermediate tensors will only be
      // deleted once shape inference is completed for the entire graph.
      return ConstantValueMap::HasRank(output->debugName());
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `RemoveProcessedInputs`, `outputs`, `any_of`, `begin`, `end`, `HasRank`, `...`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`RemoveProcessedInputs`, `outputs`, `any_of`, `begin`, `end`, `HasRank`, `...`。

### Lines 1855-1866
```cpp
    });
  };

  // An input value is no longer needed if all of its consumer nodes
  // have already been processed.
  const auto isValueNoLongerNeeded = [isNodeProcessed](const Value* input) {
    const auto& uses = input->uses();
    return std::all_of(
        uses.begin(), uses.end(), [isNodeProcessed](const Use& use) {
          return isNodeProcessed(use.user);
        });
  };
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `uses`, `all_of`, `begin`, `end`, `isNodeProcessed`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`uses`, `all_of`, `begin`, `end`, `isNodeProcessed`。

### Lines 1867-1881
```cpp

  for (const auto* input : n->inputs()) {
    if (ConstantValueMap::HasValue(input->debugName()) &&
        isValueNoLongerNeeded(input)) {
      ConstantValueMap::EraseValue(input->debugName());
    }
  }
}

void ONNXShapeTypeInference(
    Block* b,
    const ParamMap& params_dict,
    int opset_version) {
  FetchBlockInputMetadataFromParent(b);
  auto valsToParamsMap = buildValueToParamsMap(b, params_dict);
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `inputs`, `HasValue`, `debugName`, `isValueNoLongerNeeded`, `EraseValue`, `ONNXShapeTypeInference`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`inputs`, `HasValue`, `debugName`, `isValueNoLongerNeeded`, `EraseValue`, `ONNXShapeTypeInference`, `...`。

### Lines 1882-1900
```cpp
  for (auto const& it : valsToParamsMap) {
    auto key = it.first;
    auto value = it.second;
    if (key->node()->kind() == prim::Param) {
      if (value.second.isTensor()) {
        ConstantValueMap::SetValue(value.first, value.second.toTensor());
      }
    } else if (key->node()->kind() == ::c10::onnx::Constant) {
      at::Tensor const_val = key->node()->t(attr::value);
      at::Tensor const_val_copy =
          at::empty(const_val.sizes(), const_val.options());
      const_val_copy.copy_(const_val);
      ConstantValueMap::SetValue(value.first, const_val_copy);
    } else {
      throw std::runtime_error(
          "ONNXShapeTypeInference - Unsupported kind of constant node found.");
    }
  }
  for (auto n : b->nodes()) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `node`, `kind`, `isTensor`, `SetValue`, `toTensor`, `t`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`node`, `kind`, `isTensor`, `SetValue`, `toTensor`, `t`, `...`。

### Lines 1901-1915
```cpp
    for (auto subblock : n->blocks()) {
      ONNXShapeTypeInference(subblock, params_dict, opset_version);
    }
    ONNXShapeTypeInference(n, params_dict, opset_version);
    RemoveProcessedInputs(n);
  }
}

} // namespace

// For some operators, there are some inputs not related to shape inference.
// For example, LSTM input 4 (sequence_lens) is optional,
// and the shape inference can be done through other required inputs.
// When we compute reliable, we don't need this input be reliable.
static std::unordered_map<std::string, std::unordered_set<int64_t>>
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `blocks`, `ONNXShapeTypeInference`, `RemoveProcessedInputs`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`blocks`, `ONNXShapeTypeInference`, `RemoveProcessedInputs`。

### Lines 1916-1929
```cpp
    non_required_shape_inference_idx_map = {{"onnx::LSTM", {4}}};

bool AllGraphInputsStatic(const Graph* g) {
  for (auto n : g->inputs()) {
    if (TensorTypePtr input_type = n->type()->cast<TensorType>()) {
      if (input_type->dim()) {
        auto shape = input_type->symbolic_sizes();
        if (!ConstantValueMap::HasShape(n->debugName())) {
          UpdateShapeConstantValueMap(n, shape);
        }
      }
    }
  }
  for (auto n : g->inputs()) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `AllGraphInputsStatic`, `inputs`, `type`, `dim`, `symbolic_sizes`, `HasShape`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`AllGraphInputsStatic`, `inputs`, `type`, `dim`, `symbolic_sizes`, `HasShape`, `...`。

### Lines 1930-1941
```cpp
    // Some inputs can be non-Tensor type, e.g.,
    // __torch__.torch.classes.quantized.LinearPackedParamsBase
    // so we only need check Tensor type here.
    if (n->type()->cast<TensorType>() && !n->isCompleteTensor()) {
      return false;
    }
  }
  return true;
}

std::pair<bool, bool> AreInputsReliableOrStatic(Node* n) {
  auto reliable = true;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `type`, `isCompleteTensor`, `AreInputsReliableOrStatic`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`type`, `isCompleteTensor`, `AreInputsReliableOrStatic`。

### Lines 1942-1955
```cpp
  auto complete = true;
  auto input_size = n->inputs().size();
  std::unordered_set<int64_t> non_required_idx = {};
  if (non_required_shape_inference_idx_map.find(n->kind().toDisplayString()) !=
      non_required_shape_inference_idx_map.end()) {
    non_required_idx =
        non_required_shape_inference_idx_map[n->kind().toDisplayString()];
  }
  for (auto idx : c10::irange(input_size)) {
    if (!non_required_idx.empty() &&
        non_required_idx.find(idx) != non_required_idx.end()) {
      continue;
    }
    auto input = n->inputs()[idx];
```
- EN: This block handles conditional branches; iterates over collections or graph structures. Key symbols: `inputs`, `size`, `find`, `kind`, `toDisplayString`, `end`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构。关键符号：`inputs`, `size`, `find`, `kind`, `toDisplayString`, `end`, `...`。

### Lines 1956-1969
```cpp
    // Always consider None reliable and complete, because it represents
    // unspecified optional inputs in ONNX.
    if (input->node()->mustBeNone()) {
      continue;
    }
    reliable &=
        ConstantValueMap::GetTypeReliable(input->debugName()).value_or(false);
    if (auto pt = input->type()->cast<TensorType>()) {
      if (!pt->sizes().isComplete()) {
        complete = false;
      }
    }
  }
  return std::make_pair(reliable, complete);
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `node`, `mustBeNone`, `GetTypeReliable`, `debugName`, `value_or`, `type`, `...`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`node`, `mustBeNone`, `GetTypeReliable`, `debugName`, `value_or`, `type`, `...`。

### Lines 1970-1981
```cpp
}

// There is no need to put onnx type here, but we need this
// for some legacy tests when onnx_shape_inference=False.
static std::unordered_set<std::string> nodeTypeReliableForTracer = {
    "prim::ListConstruct",
    "onnx::Cast",
    "onnx::Constant",
    "onnx::Relu",
    "com.microsoft::Gelu",
    "aten::ATen"};

```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: no dominant local symbol names.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：无明显局部符号。

### Lines 1982-2000
```cpp
void UpdateReliable(
    torch::jit::Value* output,
    const std::pair<bool, bool>& inferred_type_reliable,
    bool no_type_warning) {
  auto inferred =
      ConstantValueMap::GetUseInferredType(output->debugName()).value_or(false);
  auto isTypeReliableForTracer =
      nodeTypeReliableForTracer.find(
          output->node()->kind().toDisplayString()) !=
      nodeTypeReliableForTracer.end();
  if (!inferred && !isTypeReliableForTracer &&
      !output->node()->kind().is_onnx() && no_type_warning) {
    TORCH_WARN(
        "The shape inference of ",
        output->node()->kind().toDisplayString(),
        " type is missing, so it may result in wrong shape inference for the exported graph. ",
        "Please consider adding it in symbolic function.");
  }
  auto reliable = false;
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `UpdateReliable`, `GetUseInferredType`, `debugName`, `value_or`, `find`, `node`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`UpdateReliable`, `GetUseInferredType`, `debugName`, `value_or`, `find`, `node`, `...`。

### Lines 2001-2014
```cpp
  if (inferred) {
    reliable = inferred_type_reliable.first;
  } else {
    if (inferred_type_reliable.second && isTypeReliableForTracer) {
      reliable = true;
    }
  }
  // Assume that the tracer can estimate rank correctly,
  // then the output tensor of Shape should always be reliable.
  if (output->node()->kind() == ::c10::onnx::Shape) {
    reliable = true;
  }
  ConstantValueMap::SetTypeReliable(output->debugName(), reliable);
  if (!reliable) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `node`, `kind`, `SetTypeReliable`, `debugName`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`node`, `kind`, `SetTypeReliable`, `debugName`。

### Lines 2015-2028
```cpp
    if (auto output_tensor_type = output->type()->cast<TensorType>()) {
      output->setType(output_tensor_type->withSymbolicShapes(
          ::c10::SymbolicShape(output_tensor_type->dim())));
    }
  }
}

void UpdateReliable(Node* n) {
  auto input_reliable = AreInputsReliableOrStatic(n);
  for (auto output : n->outputs()) {
    UpdateReliable(output, input_reliable);
  }
}

```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `type`, `setType`, `withSymbolicShapes`, `SymbolicShape`, `dim`, `UpdateReliable`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`type`, `setType`, `withSymbolicShapes`, `SymbolicShape`, `dim`, `UpdateReliable`, `...`。

### Lines 2029-2042
```cpp
// Traverse the graph inputs and compute reliability (e.g., are shapes static).
// Since the inputs do not change during export, we save computation time by
// marking it as computed and subsequently skipping.
static void SetGraphInputTypeReliable(const Graph* g) {
  if (!ConstantValueMap::GetAllGraphInputsReliableComputed()) {
    for (auto graph_input : g->inputs()) {
      if (!ConstantValueMap::HasTypeReliable(graph_input->debugName())) {
        ConstantValueMap::SetTypeReliable(graph_input->debugName(), true);
      }
    }
    ConstantValueMap::SetAllGraphInputsReliableComputed(true);
  }
}

```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `SetGraphInputTypeReliable`, `GetAllGraphInputsReliableComputed`, `inputs`, `HasTypeReliable`, `debugName`, `SetTypeReliable`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`SetGraphInputTypeReliable`, `GetAllGraphInputsReliableComputed`, `inputs`, `HasTypeReliable`, `debugName`, `SetTypeReliable`, `...`。

### Lines 2043-2057
```cpp
void ONNXShapeTypeInference(
    Node* n,
    const ParamMap& params_dict,
    int opset_version) {
  std::unordered_map<std::string, std::string> torch_to_onnx_input;
  std::unordered_map<std::string, std::string> torch_to_onnx_output;
  auto& original_shape_data = ConstantValueMap::GetInferredShapeData();
  ShapeDataMap inferred_shape_data;
  auto& symbol_dim_map = ConstantValueMap::GetSymbolDimMap();
  auto& dim_symbol_map = ConstantValueMap::GetDimSymbolMap();

  SetGraphInputTypeReliable(n->owningGraph());
  GRAPH_UPDATE(
      "Running ONNX shape inference for node: ", n->kind().toDisplayString());

```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `ONNXShapeTypeInference`, `GetInferredShapeData`, `GetSymbolDimMap`, `GetDimSymbolMap`, `SetGraphInputTypeReliable`, `owningGraph`, `...`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`ONNXShapeTypeInference`, `GetInferredShapeData`, `GetSymbolDimMap`, `GetDimSymbolMap`, `SetGraphInputTypeReliable`, `owningGraph`, `...`。

### Lines 2058-2069
```cpp
  if (IsValidONNXNode(n)) {
    // Create a Graph containing only the single node n.
    // This graph is later converted to ONNX to run shape inference.
    auto n_graph = std::make_shared<Graph>();
    auto clone_node = CloneNodeToGraph(n, n_graph, params_dict, opset_version);
    n_graph->insertNode(clone_node);

    // Register all node outputs as graph outputs.
    for (auto output : clone_node->outputs()) {
      n_graph->registerOutput(output);
    }

```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `IsValidONNXNode`, `CloneNodeToGraph`, `insertNode`, `outputs`, `registerOutput`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`IsValidONNXNode`, `CloneNodeToGraph`, `insertNode`, `outputs`, `registerOutput`。

### Lines 2070-2084
```cpp
    // Map original PyTorch graph's i/o name
    // to temporal ONNX graph's i/o name for shape inference
    for (size_t i = 0; i < clone_node->inputs().size(); ++i) {
      torch_to_onnx_input[n->input(i)->debugName()] =
          clone_node->input(i)->debugName();
    }

    for (size_t i = 0; i < clone_node->outputs().size(); ++i) {
      torch_to_onnx_output[n->output(i)->debugName()] =
          clone_node->output(i)->debugName();
    }
    // Make inferred_shape_data use name from temporal ONNX graph
    // instead of original PyTorch graph. Only copy what we need,
    // which are the inputs of n.
    for (auto input : n->inputs()) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `inputs`, `size`, `input`, `debugName`, `outputs`, `output`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`inputs`, `size`, `input`, `debugName`, `outputs`, `output`。

### Lines 2085-2096
```cpp
      const auto maybe_shape = original_shape_data.find(input->debugName());
      if (maybe_shape != original_shape_data.end()) {
        const auto onnx_output_name =
            torch_to_onnx_input.find(input->debugName());
        if (onnx_output_name != torch_to_onnx_input.end()) {
          inferred_shape_data[onnx_output_name->second] = maybe_shape->second;
        }
      }
    }
    // Use scalar_type_analysis without low precision cast
    ScalarTypeAnalysisForONNX(n_graph, false, opset_version);

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `find`, `debugName`, `end`, `ScalarTypeAnalysisForONNX`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`find`, `debugName`, `end`, `ScalarTypeAnalysisForONNX`。

### Lines 2097-2110
```cpp
    GRAPH_DEBUG("Original torch graph: ", n->owningGraph()->toString());
    GRAPH_DEBUG(
        "Cloned torch graph to run shape inference: ", n_graph->toString());

    if (IsGraphValidForInference(n_graph)) {
      // TODO: Some ops have conversion happen at Peephole pass.
      //       The conversion here is incomplete for these ops.
      //       e.g: ListConstruct, ListUnpack, etc.
      std::shared_ptr<onnx::ModelProto> model_proto;
      ConvertGraphToONNXProto(
          n_graph, model_proto, symbol_dim_map, dim_symbol_map, opset_version);
      GRAPH_DEBUG(
          "ONNX graph to run shape inference: ", prettyPrint(*model_proto));

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `owningGraph`, `toString`, `IsGraphValidForInference`, `ConvertGraphToONNXProto`, `prettyPrint`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`owningGraph`, `toString`, `IsGraphValidForInference`, `ConvertGraphToONNXProto`, `prettyPrint`。

### Lines 2111-2134
```cpp
      // infer shape
      try {
        // TODO(#79208): Enable more operators to support data propagation
        switch (n->kind()) {
          case ::c10::onnx::Shape:
          case ::c10::onnx::Gather: {
            auto* schema_registry = onnx::OpSchemaRegistry::Instance();
            onnx::ShapeInferenceOptions options{
                /*check_type_val=*/false,
                /*strict_mode_val=*/0,
                /*data_prop_val=*/true};
            onnx::shape_inference::InferShapes(
                *model_proto, schema_registry, options, &inferred_shape_data);
            break;
          }
          default: {
            onnx::shape_inference::InferShapes(*model_proto);
            break;
          }
        }
        UpdateOutputTypeByONNXProto(
            n, clone_node, *model_proto, symbol_dim_map, dim_symbol_map);
      } catch (std::runtime_error& ex) {
        // TODO: include this as warning once we have a more consolidated
```
- EN: This block manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `kind`, `Instance`, `InferShapes`, `UpdateOutputTypeByONNXProto`.
- CN: 该代码块操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`kind`, `Instance`, `InferShapes`, `UpdateOutputTypeByONNXProto`。

### Lines 2135-2155
```cpp
        // warning system.
        GRAPH_DEBUG(
            "ONNX shape inference fails with: ",
            ex.what(),
            " on graph: ",
            n_graph->toString());
        constexpr const char* shape_err = "ShapeInferenceError";
        constexpr const char* type_err = "TypeInferenceError";
        if ((strstr(ex.what(), shape_err) == nullptr) &&
            (strstr(ex.what(), type_err) == nullptr)) {
          throw;
        }
      }
      GRAPH_DEBUG(
          "ONNX graph after shape inference: ", prettyPrint(*model_proto));
    }
  } else if (CustomSettype(n)) {
    // If the node is not ONNX standard, go through every output to check if
    // they all have shape. If they all do, this should be reliable even if the
    // Op is not from ONNX.
    for (auto node_output : n->outputs()) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `what`, `toString`, `strstr`, `prettyPrint`, `CustomSettype`, `outputs`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`what`, `toString`, `strstr`, `prettyPrint`, `CustomSettype`, `outputs`。

### Lines 2156-2167
```cpp
      // Custom setType output should get in here if it's set correctly. They
      // will be updated to inferred for later updatereliable function.
      ConstantValueMap::SetUseInferredType(node_output->debugName(), true);
    }
  }

  SpecialPostProcess(n);
  // Get data propagation result from ONNX shape inference
  for (const auto& output : n->outputs()) {
    const auto inferred_shape_pair =
        inferred_shape_data.find(torch_to_onnx_output[output->debugName()]);
    if (inferred_shape_pair != inferred_shape_data.end()) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `SetUseInferredType`, `debugName`, `SpecialPostProcess`, `outputs`, `find`, `end`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`SetUseInferredType`, `debugName`, `SpecialPostProcess`, `outputs`, `find`, `end`。

### Lines 2168-2184
```cpp
      const auto& inferred_shape = inferred_shape_pair->second;
      int rank = inferred_shape.dim_size();
      std::vector<::c10::ShapeSymbol> final_shape(rank);
      for (int i = 0; i < rank; ++i) {
        final_shape[i] = ONNXDimToShapeSymbol(
            inferred_shape.dim(i), symbol_dim_map, dim_symbol_map);
      }
      c10::SymbolicShape shape_value(final_shape);
      // Store data propagation result into shapeValueMap
      ConstantValueMap::SetShapeValue(output->debugName(), shape_value);
      // Use original name in PyTorch graph instead of
      // temporary name in intermediate ONNX graph
      // Add this back to original_shape_data
      original_shape_data[output->debugName()] = inferred_shape;
    }
  }

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `dim_size`, `final_shape`, `ONNXDimToShapeSymbol`, `dim`, `shape_value`, `SetShapeValue`, `...`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`dim_size`, `final_shape`, `ONNXDimToShapeSymbol`, `dim`, `shape_value`, `SetShapeValue`, `...`。

### Lines 2185-2196
```cpp
  if (IsValidONNXNode(n)) {
    ProcessConstantValueMap(n, opset_version);
    if (n->kind() != prim::ListConstruct) {
      for (auto input : n->inputs()) {
        if (input->node()->kind() == prim::ListConstruct) {
          UpdateReliable(input, AreInputsReliableOrStatic(input->node()));
        }
      }
    }
  }
  UpdateReliable(n);

```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `IsValidONNXNode`, `ProcessConstantValueMap`, `kind`, `inputs`, `node`, `UpdateReliable`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`IsValidONNXNode`, `ProcessConstantValueMap`, `kind`, `inputs`, `node`, `UpdateReliable`, `...`。

### Lines 2197-2208
```cpp
  // For the node type that does not have ComputeConstant logic, it may have
  // reliable shape but its shape is not in ConstantValueMap. So we need this
  // logic to update ConstantValueMap.
  for (auto node_output : n->outputs()) {
    UpdateShapeConstantIfReliable(node_output);
  }

  GRAPH_DEBUG(
      "Torch graph after shape inference:", n->owningGraph()->toString());
}

void ONNXSetDynamicInputShape(
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `outputs`, `UpdateShapeConstantIfReliable`, `owningGraph`, `toString`, `ONNXSetDynamicInputShape`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`outputs`, `UpdateShapeConstantIfReliable`, `owningGraph`, `toString`, `ONNXSetDynamicInputShape`。

### Lines 2209-2221
```cpp
    std::shared_ptr<Graph>& graph,
    const std::unordered_map<
        std::string,
        std::unordered_map<int64_t, std::string>>& dynamic_axes,
    const std::vector<std::string>& input_names) {
  GRAPH_UPDATE("ONNX set dynamic input shape.");
  GRAPH_UPDATE("dynamic axes tensor names:", [&]() {
    std::vector<std::string> res(dynamic_axes.size());
    std::transform(
        dynamic_axes.begin(), dynamic_axes.end(), res.begin(), [](auto pair) {
          return pair.first;
        });
    return res;
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `res`, `size`, `transform`, `begin`, `end`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`res`, `size`, `transform`, `begin`, `end`。

### Lines 2222-2235
```cpp
  }());

  std::map<std::string, ::c10::ShapeSymbol> name_to_sym;

  for (const auto i : c10::irange(input_names.size())) {
    const auto& input_name = input_names[i];
    if (dynamic_axes.find(input_name) != dynamic_axes.end()) {
      auto axes_names = dynamic_axes.find(input_name)->second;
      TORCH_INTERNAL_ASSERT(i < graph->inputs().size());
      auto input_tensor_type = graph->inputs()[i]->type()->cast<TensorType>();
      if (!input_tensor_type) {
        continue;
      }

```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `irange`, `size`, `find`, `end`, `inputs`, `type`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`irange`, `size`, `find`, `end`, `inputs`, `type`。

### Lines 2236-2253
```cpp
      auto shape_ref = input_tensor_type->symbolic_sizes().sizes();
      TORCH_CHECK(
          shape_ref.has_value(), "Input tensor shape should have value.");
      auto shape = shape_ref.value();

      for (const auto& pair : axes_names) {
        const auto axis = pair.first;
        const auto name = pair.second;
        if (name_to_sym.find(name) == name_to_sym.end()) {
          name_to_sym[name] = ::c10::ShapeSymbol::newSymbol();
        }
        TORCH_CHECK(
            axis < static_cast<int64_t>(shape.size()),
            "Dynamic shape axis should be no more than the shape dimension for ",
            name);
        shape[axis] = name_to_sym[name];
      }

```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `symbolic_sizes`, `sizes`, `has_value`, `value`, `find`, `end`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`symbolic_sizes`, `sizes`, `has_value`, `value`, `find`, `end`, `...`。

### Lines 2254-2267
```cpp
      graph->inputs()[i]->setType(
          input_tensor_type->withSymbolicShapes(::c10::SymbolicShape(shape)));
    }
  }
}

static bool HasSequenceTypeOutput(const Node* node) {
  if (node->kind() == ::c10::onnx::SplitToSequence ||
      node->kind() == ::c10::onnx::SequenceInsert ||
      node->kind() == ::c10::onnx::SequenceEmpty ||
      node->kind() == ::c10::onnx::SequenceErase ||
      node->kind() == ::c10::onnx::SequenceConstruct ||
      node->kind() == ::c10::onnx::Loop || node->kind() == ::c10::onnx::If)
    return true;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `inputs`, `setType`, `withSymbolicShapes`, `SymbolicShape`, `HasSequenceTypeOutput`, `kind`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`inputs`, `setType`, `withSymbolicShapes`, `SymbolicShape`, `HasSequenceTypeOutput`, `kind`。

### Lines 2268-2282
```cpp
  return false;
}

static void ONNXUpdateTypeFromTensor(
    Value* graph_output,
    const at::Tensor& output,
    bool onnx_shape_inference) {
  if (onnx_shape_inference) {
    MergeInferredTypeAndSetMap(
        graph_output, TensorType::create(output), graph_output->type());
  } else {
    graph_output->inferTypeFrom(output);
  }
}

```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `ONNXUpdateTypeFromTensor`, `MergeInferredTypeAndSetMap`, `create`, `type`, `inferTypeFrom`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`ONNXUpdateTypeFromTensor`, `MergeInferredTypeAndSetMap`, `create`, `type`, `inferTypeFrom`。

### Lines 2283-2294
```cpp
// Recursively look into elements in `output_obj`, and assign shape/type info
// into flattened graph outputs. `outputs_index` is passed in to point to the
// current index in flattened graph outputs. The updated `outputs_index` is
// returned at the end of the function.
static size_t ONNXAssignOutputShape(
    std::shared_ptr<Graph>& graph,
    size_t outputs_index,
    PyObject* output_obj,
    bool onnx_shape_inference,
    bool is_script,
    int opset_version) {
  auto index_check = [&]() {
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `ONNXAssignOutputShape`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`ONNXAssignOutputShape`。

### Lines 2295-2309
```cpp
    TORCH_INTERNAL_ASSERT(
        outputs_index <= graph->outputs().size(),
        "Incorrect number of elements provided as example outputs.");
  };

  index_check();

  if (THPVariable_Check(output_obj)) {
    const at::Tensor& var = THPVariable_Unpack(output_obj);
    ONNXUpdateTypeFromTensor(
        graph->outputs().at(outputs_index), var, onnx_shape_inference);
    outputs_index++;
  } else if (PyTuple_Check(output_obj)) {
    size_t tuple_len = PyTuple_GET_SIZE(output_obj);
    for (const auto i : c10::irange(tuple_len)) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `outputs`, `size`, `index_check`, `THPVariable_Check`, `THPVariable_Unpack`, `ONNXUpdateTypeFromTensor`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`outputs`, `size`, `index_check`, `THPVariable_Check`, `THPVariable_Unpack`, `ONNXUpdateTypeFromTensor`, `...`。

### Lines 2310-2321
```cpp
      outputs_index = ONNXAssignOutputShape(
          graph,
          outputs_index,
          PyTuple_GET_ITEM(output_obj, i),
          onnx_shape_inference,
          is_script,
          opset_version);
    }
  } else if (PyList_Check(output_obj)) {
    const auto list_len = PyList_GET_SIZE(output_obj);
    if (HasSequenceTypeOutput(graph->outputs().at(outputs_index)->node())) {
      auto output_type = graph->outputs().at(outputs_index)->type();
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `ONNXAssignOutputShape`, `PyTuple_GET_ITEM`, `PyList_Check`, `PyList_GET_SIZE`, `HasSequenceTypeOutput`, `outputs`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`ONNXAssignOutputShape`, `PyTuple_GET_ITEM`, `PyList_Check`, `PyList_GET_SIZE`, `HasSequenceTypeOutput`, `outputs`, `...`。

### Lines 2322-2339
```cpp
      TORCH_CHECK(
          output_type->cast<ListType>(),
          "Expected a sequence type, but received a non-iterable type in graph output index ",
          outputs_index);
      if (list_len > 0) {
        auto list_elem = PyList_GET_ITEM(output_obj, 0);
        TORCH_INTERNAL_ASSERT(THPVariable_Check(list_elem));
        auto& var = THPVariable_Unpack(list_elem);
        for (const auto i : c10::irange(1, list_len)) {
          list_elem = PyList_GET_ITEM(output_obj, i);
          TORCH_INTERNAL_ASSERT(THPVariable_Check(list_elem));
          auto& new_var = THPVariable_Unpack(list_elem);
          TORCH_CHECK(
              var.scalar_type() == new_var.scalar_type(),
              "Unsupported sequence with mixed element types in model outputs. "
              "ONNX supports only sequences of elements of the same data type.");
        }
        auto elem_type = graph->outputs()
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `PyList_GET_ITEM`, `THPVariable_Check`, `THPVariable_Unpack`, `irange`, `scalar_type`, `outputs`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`PyList_GET_ITEM`, `THPVariable_Check`, `THPVariable_Unpack`, `irange`, `scalar_type`, `outputs`。

### Lines 2340-2358
```cpp
                             .at(outputs_index)
                             ->type()
                             ->castRaw<ListType>()
                             ->getElementType()
                             ->cast<TensorType>();
        elem_type = elem_type->withScalarType(var.scalar_type());
        auto graph_output = graph->outputs().at(outputs_index);
        MergeInferredTypeAndSetMap(
            graph_output, graph_output->type(), ListType::create(elem_type));
      } else {
        graph->outputs()
            .at(outputs_index)
            ->setType(graph->outputs().at(outputs_index)->type());
      }
      outputs_index++;
    } else {
      // When torch output is a list type, but ONNX node is not a
      // sequence type. Like prim::ListConstruct
      for (const auto i : c10::irange(list_len)) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `type`, `getElementType`, `withScalarType`, `scalar_type`, `outputs`, `MergeInferredTypeAndSetMap`, `...`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`type`, `getElementType`, `withScalarType`, `scalar_type`, `outputs`, `MergeInferredTypeAndSetMap`, `...`。

### Lines 2359-2373
```cpp
        outputs_index = ONNXAssignOutputShape(
            graph,
            outputs_index,
            PyList_GET_ITEM(output_obj, i),
            onnx_shape_inference,
            is_script,
            opset_version);
      }
    }
  } else if (PyDict_Check(output_obj)) {
    // Support for dict data type is limited to fixed size dictionaries in
    // ONNX.
    // Dictionary values are unrolled and keys are not preserved.
    auto* items = PyDict_Items(output_obj);
    auto unrolled_dict = py::reinterpret_borrow<py::list>(items);
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `ONNXAssignOutputShape`, `PyList_GET_ITEM`, `PyDict_Check`, `PyDict_Items`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`ONNXAssignOutputShape`, `PyList_GET_ITEM`, `PyDict_Check`, `PyDict_Items`。

### Lines 2374-2396
```cpp
    TORCH_INTERNAL_ASSERT(PyList_Check(unrolled_dict.ptr()));
    for (const auto i : c10::irange(unrolled_dict.size())) {
      outputs_index = ONNXAssignOutputShape(
          graph,
          outputs_index,
          PyList_GET_ITEM(unrolled_dict.ptr(), i),
          onnx_shape_inference,
          is_script,
          opset_version);
    }
    Py_DECREF(items);
  } else if (THPUtils_checkString(output_obj)) {
    // Ignore string, since they are not supported as output in ONNX.
  } else if (PyNone_Check(output_obj)) {
    // Tracing:
    //    Ignore None, since it is not captured in IR graph as output.
    // Scripting:
    //    Ignore None, if observing a fixed `None` node in IR graph. Because
    //    it is meaningless to include it as graph output as it carries no
    //    data/information. Plus that static `None` is not supported in ONNX
    //    IR. Otherwise, the output should have type `Optional`, and should be
    //    converted to ONNX `Optional`.

```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `PyList_Check`, `ptr`, `irange`, `size`, `ONNXAssignOutputShape`, `PyList_GET_ITEM`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`PyList_Check`, `ptr`, `irange`, `size`, `ONNXAssignOutputShape`, `PyList_GET_ITEM`, `...`。

### Lines 2397-2416
```cpp
    // More context:
    // Cause: in tracing we flatten the outputs in ONNXTracedModule.forward
    // in torch/jit/_trace.py while tracing. This means the traced IR graph
    // has None outputs omitted.
    // But then the outputs passed in here are un-flattened, which means they
    // contain None objects. Ideally we'd remove this difference.
    if (is_script && outputs_index < graph->outputs().size()) {
      if (graph->outputs().at(outputs_index)->node()->mustBeNone()) {
        if (opset_version >= 15) {
          ReplaceGraphOutputNoneWithOptional(graph, outputs_index);
          outputs_index++;
        } else {
          graph->eraseOutput(outputs_index);
        }
      } else {
        outputs_index++;
      }
    }
  } else {
    std::string msg =
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `outputs`, `size`, `node`, `mustBeNone`, `ReplaceGraphOutputNoneWithOptional`, `eraseOutput`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`outputs`, `size`, `node`, `mustBeNone`, `ReplaceGraphOutputNoneWithOptional`, `eraseOutput`。

### Lines 2417-2428
```cpp
        "Model output has unsupported type. See "
        "https://pytorch.org/docs/stable/onnx.html#types. Got type: ";
    msg += THPUtils_typename(output_obj);
    throw std::runtime_error(msg);
  }

  index_check();

  return outputs_index;
}

Node* ONNXOptionalNodeForNone(std::shared_ptr<Graph>& graph) {
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `THPUtils_typename`, `runtime_error`, `index_check`, `ONNXOptionalNodeForNone`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`THPUtils_typename`, `runtime_error`, `index_check`, `ONNXOptionalNodeForNone`。

### Lines 2429-2441
```cpp
  TypePtr elem_type = TensorType::get()->withScalarType(at::ScalarType::Float);
  Node* opt_node = graph->create(::c10::onnx::Optional, 1);
  opt_node->ty_(Symbol::attr("type"), elem_type);
  opt_node->output()->setType(OptionalType::create(elem_type));
  return opt_node;
}

void ReplaceGraphOutputNoneWithOptional(
    std::shared_ptr<Graph>& graph,
    size_t outputs_index) {
  Node* opt_node = ONNXOptionalNodeForNone(graph);
  opt_node->insertBefore(graph->return_node());
  Value* graph_output = graph->outputs().at(outputs_index);
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `get`, `withScalarType`, `create`, `ty_`, `attr`, `output`, `...`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`get`, `withScalarType`, `create`, `ty_`, `attr`, `output`, `...`。

### Lines 2442-2455
```cpp
  // replace only the last value as Optional type only affects
  // the value right before output
  graph_output->replaceAllUsesAfterNodeWith(opt_node, opt_node->output());
  if (!graph_output->type()->cast<NoneType>()) {
    opt_node->addInput(graph_output);
    opt_node->copyMetadata(graph_output->node());
  }
}

void ONNXAssignOutputShape(
    std::shared_ptr<Graph>& graph,
    at::ArrayRef<at::Tensor> outputs,
    const python::IODescriptor& desc,
    bool onnx_shape_inference,
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `replaceAllUsesAfterNodeWith`, `output`, `type`, `addInput`, `copyMetadata`, `node`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`replaceAllUsesAfterNodeWith`, `output`, `type`, `addInput`, `copyMetadata`, `node`, `...`。

### Lines 2456-2469
```cpp
    bool is_script,
    int opset_version) {
  size_t outputs_index = 0;
  PyObject* py_obj = unflatten(outputs, desc);
  TORCH_INTERNAL_ASSERT(PyTuple_Check(py_obj));

  outputs_index = ONNXAssignOutputShape(
      graph,
      outputs_index,
      py_obj,
      onnx_shape_inference,
      is_script,
      opset_version);

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `unflatten`, `PyTuple_Check`, `ONNXAssignOutputShape`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`unflatten`, `PyTuple_Check`, `ONNXAssignOutputShape`。

### Lines 2470-2487
```cpp
  TORCH_INTERNAL_ASSERT(
      outputs_index == graph->outputs().size(),
      "Incorrect number of elements provided as example outputs.");

  Py_DECREF(py_obj);
  GRAPH_DUMP("After ONNXAssignOutputShape", graph);
}

void ONNXShapeTypeInference(
    std::shared_ptr<Graph>& graph,
    const ParamMap& params_dict,
    int opset_version) {
  ConstantValueMap::ClearMaps();
  SetGraphInputTypeReliable(graph.get());
  ONNXShapeTypeInference(graph->block(), params_dict, opset_version);
  ConstantValueMap::ClearMaps();
}

```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `outputs`, `size`, `Py_DECREF`, `ONNXShapeTypeInference`, `ClearMaps`, `SetGraphInputTypeReliable`, `...`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`outputs`, `size`, `Py_DECREF`, `ONNXShapeTypeInference`, `ClearMaps`, `SetGraphInputTypeReliable`, `...`。

### Lines 2488-2503
```cpp
void UpdateShapeConstantIfReliable(torch::jit::Value* node_output) {
  if (ConstantValueMap::HasTypeReliable(node_output->debugName())) {
    auto reliable = ConstantValueMap::GetTypeReliable(node_output->debugName())
                        .value_or(false);
    if (reliable && !ConstantValueMap::HasShape(node_output->debugName())) {
      // TODO: ListType case
      if (auto output_tensor_type = node_output->type()->cast<TensorType>()) {
        if (output_tensor_type->dim()) {
          auto symbolic_sizes = output_tensor_type->symbolic_sizes();
          UpdateShapeConstantValueMap(node_output, symbolic_sizes);
        }
      }
    }
  }
}

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `UpdateShapeConstantIfReliable`, `HasTypeReliable`, `debugName`, `GetTypeReliable`, `value_or`, `HasShape`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`UpdateShapeConstantIfReliable`, `HasTypeReliable`, `debugName`, `GetTypeReliable`, `value_or`, `HasShape`, `...`。

### Lines 2504-2504
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
- EN: Optimization pass — the code simplifies graphs by removing redundant work or folding values.
  CN: 优化 pass——代码通过删除冗余计算或常量折叠来简化图。
- EN: Inlining — the implementation merges callees or blocks into their call sites.
  CN: 内联——实现会把被调用者或代码块并入调用位置。
- EN: Fusion support — the file prepares groups of operations for fused execution.
  CN: 融合支持——该文件会为融合执行准备算子分组。
- EN: Type and shape reasoning — the code propagates static information across the IR.
  CN: 类型与形状推理——代码会在 IR 中传播静态信息。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `torch/csrc/jit/passes/onnx/shape_type_inference.h`, `c10/util/irange.h`, `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/onnx/constant_fold.h`, `torch/csrc/jit/passes/onnx/constant_map.h`, `torch/csrc/jit/passes/onnx/fixup_onnx_controlflow.h`, `torch/csrc/jit/passes/onnx/helper.h`, `torch/csrc/jit/passes/onnx/scalar_type_analysis.h`, `torch/csrc/jit/python/python_arg_flatten.h`, `torch/csrc/jit/serialization/export.h`, `...`
- External includes / 外部头文件: `onnx/shape_inference/implementation.h`, `algorithm`, `cmath`, `iterator`, `limits`, `unordered_set`, `utility`
- Namespaces / 命名空间: `torch::jit`, `onnx_torch`, `onnx`
- Representative symbols / 代表性符号: `PyNone_Check`, `Py_IsNone`, `MergeInferredType`, `make_pair`, `device`, `dim`, `withSymbolicShapes`, `symbolic_sizes`, `scalarType`, `has_value`, `...`
