# pattern_conversion.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/passes/onnx/pattern_conversion/pattern_conversion.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements the TorchScript JIT pass logic for pattern conversion, including graph analysis and rewrites.
- 用途 (CN): 实现与 pattern conversion 相关的 TorchScript JIT pass 逻辑，包括图分析与改写。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7
```cpp
#include <c10/util/irange.h>
#include <torch/csrc/jit/passes/dead_code_elimination.h>
#include <torch/csrc/jit/passes/erase_number_types.h>
#include <torch/csrc/jit/passes/onnx.h>
#include <torch/csrc/jit/passes/onnx/pattern_conversion/common.h>
#include <torch/csrc/jit/passes/onnx/pattern_conversion/pattern_conversion.h>

```
- EN: Pulls in the headers needed by the pattern conversion logic. Internal dependencies: `c10/util/irange.h`, `torch/csrc/jit/passes/dead_code_elimination.h`, `torch/csrc/jit/passes/erase_number_types.h`, `torch/csrc/jit/passes/onnx.h`, `torch/csrc/jit/passes/onnx/pattern_conversion/common.h`, `...`; external dependencies: none.
- CN: 为 pattern conversion 相关逻辑引入所需头文件。内部依赖：`c10/util/irange.h`, `torch/csrc/jit/passes/dead_code_elimination.h`, `torch/csrc/jit/passes/erase_number_types.h`, `torch/csrc/jit/passes/onnx.h`, `torch/csrc/jit/passes/onnx/pattern_conversion/common.h`, `...`；外部依赖：无。

### Lines 8-14
```cpp
#include <ATen/ScalarOps.h>

#include <iostream>

// EDITING THIS FILE? READ THIS FIRST!
// see Note [Edit Pattern Conversion] in pattern_conversion.h

```
- EN: This block implements local helper logic for pattern conversion. Key symbols: no dominant local symbol names.
- CN: 该代码块实现与 pattern conversion 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 15-21
```cpp
namespace torch::jit {

// Converting inplace index_put to ONNX
namespace {

Value* CreateSizeOfDim(Value* input, int64_t dim, Node* insertBefore) {
  auto graph = input->owningGraph();
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `CreateSizeOfDim`, `owningGraph`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`CreateSizeOfDim`, `owningGraph`。

### Lines 22-29
```cpp
  WithInsertPoint guard(insertBefore);
  auto size = graph->insert(aten::size, {input, dim});
  return size;
}

Value* ConvertSelectToIndex(Value* index, Node* insertBefore) {
  // Create index tensor based on index input of aten::select node.
  auto graph = insertBefore->owningGraph();
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `guard`, `insert`, `ConvertSelectToIndex`, `owningGraph`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`guard`, `insert`, `ConvertSelectToIndex`, `owningGraph`。

### Lines 30-36
```cpp
  WithInsertPoint guard(insertBefore);
  return graph->insert(aten::unsqueeze, {index, 0});
}

Value* ConvertSliceToIndex(Node* slice, Value* size, Node* insertBefore) {
  // Create index tensor based on aten::slice node.
  auto graph = slice->owningGraph();
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `guard`, `insert`, `ConvertSliceToIndex`, `owningGraph`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`guard`, `insert`, `ConvertSliceToIndex`, `owningGraph`。

### Lines 37-44
```cpp
  WithInsertPoint guard(insertBefore);
  TORCH_INTERNAL_ASSERT((slice->inputs()).size() == 5);
  auto start = slice->inputs()[2];
  auto end = slice->inputs()[3];
  auto step = slice->inputs()[4];
  auto index =
      graph->insert(aten::arange, {size}, {NamedValue("dtype", c10::kLong)});
  auto sliced_index_n = graph->create(
```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `guard`, `inputs`, `size`, `insert`, `NamedValue`, `create`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`guard`, `inputs`, `size`, `insert`, `NamedValue`, `create`。

### Lines 45-52
```cpp
      aten::slice,
      {index,
       graph->insertConstant(
           scalar_to_tensor(at::Scalar(0)), std::nullopt, slice->scope()),
       start,
       end,
       step});

```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `insertConstant`, `scalar_to_tensor`, `Scalar`, `scope`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`insertConstant`, `scalar_to_tensor`, `Scalar`, `scope`。

### Lines 53-61
```cpp
  sliced_index_n->copyMetadata(insertBefore);
  auto sliced_index = sliced_index_n->insertBefore(insertBefore)->output();
  return sliced_index;
}

struct ConvertedIndex {
  ConvertedIndex(Value* index, c10::Symbol orig_node_kind)
      : index(index), orig_node_kind(orig_node_kind) {}

```
- EN: Declares core types or data containers for this file. Prominent symbols: `copyMetadata`, `insertBefore`, `output`, `ConvertedIndex`, `index`, `orig_node_kind`.
- CN: 声明该文件使用的核心类型或数据容器。主要符号：`copyMetadata`, `insertBefore`, `output`, `ConvertedIndex`, `index`, `orig_node_kind`。

### Lines 62-68
```cpp
  Value* index = nullptr;
  c10::Symbol orig_node_kind;
};

std::unordered_map<int64_t, ConvertedIndex> MergeSliceAndSelectToIndices(
    Graph* graph,
    Node* index_put_node,
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `MergeSliceAndSelectToIndices`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`MergeSliceAndSelectToIndices`。

### Lines 69-79
```cpp
    const std::vector<Node*>& slice_and_select_nodes,
    Value* orig_data,
    const py::dict& env) {
  std::unordered_map<int64_t, ConvertedIndex> dim_index_map;

  // Loop over fetched slice and select nodes and convert them to index tensors.
  // keep track of which dimension the current slice/select node is applying to.
  int64_t cur_dim = 0;
  int64_t dim_offset = 0;
  const auto orig_tensor_indices = index_put_node->input(1)->node()->inputs();
  for (auto it = slice_and_select_nodes.rbegin();
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `input`, `node`, `inputs`, `rbegin`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`input`, `node`, `inputs`, `rbegin`。

### Lines 80-87
```cpp
       it != slice_and_select_nodes.rend();
       ++it) {
    auto node = *it;
    // select does not keep dims,
    // this creates offset for latter slice and select nodes.
    // NOTE: Cannot rely on get(attr::dim), because op no longer match schema.
    int64_t dim = node->inputs().at(1)->node()->t(attr::value).item().toLong();

```
- EN: This block manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `rend`, `inputs`, `node`, `t`, `item`, `toLong`.
- CN: 该代码块操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`rend`, `inputs`, `node`, `t`, `item`, `toLong`。

### Lines 88-94
```cpp
    if (dim < 0) {
      // auto input_type = env.at(orig_data)->type()->expect<TensorType>();
      auto py_value = env[py::cast(orig_data)];
      Value* value = py_value.cast<Value*>();
      auto input_type = value->type()->expect<TensorType>();
      if (input_type->dim().has_value()) {
        auto rank = static_cast<int64_t>(input_type->dim().value());
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `cast`, `type`, `dim`, `has_value`, `value`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`cast`, `type`, `dim`, `has_value`, `value`。

### Lines 95-106
```cpp
        // Rank of original tensor to index on.
        // Minus the offset created by select operators.
        dim = dim + rank - dim_offset;
      } else {
        std::cerr
            << "Error: Cannot export ellipsis indexing for input "
            << "of unknown rank. Check https://pytorch.org/docs/stable/onnx.html#indexing"
            << "for details.";
      }
    }
    dim = dim + dim_offset;
    while (cur_dim < dim) {
```
- EN: This block iterates over collections or graph structures. Key symbols: no dominant local symbol names.
- CN: 该代码块遍历集合或图结构。关键符号：无明显局部符号。

### Lines 107-113
```cpp
      // Handle skipped dims, these are created from ..., or tensor indices
      // E.g.: x[torch.tensor([1, 0]), ..., 0] = update, where x has rank 3.
      // Both torch.tensor([1, 0]) and ... are skipped, we only observe
      // aten::select node with dim == 2. Tensor indices will be handled later.
      // Ellipsis(...) are treated as a complete slice over the axes, thus we
      // create index tensors here accordingly.
      if (cur_dim - dim_offset >= (int64_t)orig_tensor_indices.size() ||
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `size`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`size`。

### Lines 114-121
```cpp
          index_put_node->input(1)
              ->node()
              ->input(cur_dim - dim_offset)
              ->node()
              ->mustBeNone()) {
        auto size = CreateSizeOfDim(orig_data, cur_dim, index_put_node);
        WithInsertPoint guard(index_put_node);
        auto index_tensor = graph->insert(
```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `input`, `node`, `mustBeNone`, `CreateSizeOfDim`, `guard`, `insert`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`input`, `node`, `mustBeNone`, `CreateSizeOfDim`, `guard`, `insert`。

### Lines 122-129
```cpp
            aten::arange, {size}, {NamedValue("dtype", c10::kLong)});
        dim_index_map.emplace(
            std::piecewise_construct,
            std::forward_as_tuple(cur_dim),
            std::forward_as_tuple(index_tensor, aten::slice));
      } else if (cur_dim - dim_offset < (int64_t)orig_tensor_indices.size()) {
        dim_index_map.emplace(
            std::piecewise_construct,
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `NamedValue`, `emplace`, `forward_as_tuple`, `size`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`NamedValue`, `emplace`, `forward_as_tuple`, `size`。

### Lines 130-136
```cpp
            std::forward_as_tuple(cur_dim),
            std::forward_as_tuple(
                orig_tensor_indices[cur_dim - dim_offset], aten::index));
      }
      cur_dim++;
    }

```
- EN: This block implements local helper logic for pattern conversion. Key symbols: `forward_as_tuple`.
- CN: 该代码块实现与 pattern conversion 相关的局部辅助逻辑。关键符号：`forward_as_tuple`。

### Lines 137-143
```cpp
    TORCH_INTERNAL_ASSERT(cur_dim == dim);
    if (node->kind() == aten::slice) {
      auto size = CreateSizeOfDim(orig_data, dim, index_put_node);
      auto index_tensor = ConvertSliceToIndex(node, size, index_put_node);
      dim_index_map.emplace(
          std::piecewise_construct,
          std::forward_as_tuple(dim),
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `kind`, `CreateSizeOfDim`, `ConvertSliceToIndex`, `emplace`, `forward_as_tuple`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`kind`, `CreateSizeOfDim`, `ConvertSliceToIndex`, `emplace`, `forward_as_tuple`。

### Lines 144-150
```cpp
          std::forward_as_tuple(index_tensor, aten::slice));
    } else if (node->kind() == aten::select) {
      auto index_tensor = ConvertSelectToIndex(node->input(2), index_put_node);
      dim_index_map.emplace(
          std::piecewise_construct,
          std::forward_as_tuple(dim),
          std::forward_as_tuple(index_tensor, aten::select));
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `forward_as_tuple`, `kind`, `ConvertSelectToIndex`, `input`, `emplace`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`forward_as_tuple`, `kind`, `ConvertSelectToIndex`, `input`, `emplace`。

### Lines 151-158
```cpp
      dim_offset++;
    } else {
      TORCH_CHECK(
          false,
          node->kind().toDisplayString(),
          " Expected aten::slice or aten::select.");
    }

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `kind`, `toDisplayString`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`kind`, `toDisplayString`。

### Lines 159-165
```cpp
    cur_dim++;
  }

  while (cur_dim - dim_offset < (int64_t)orig_tensor_indices.size()) {
    dim_index_map.emplace(
        std::piecewise_construct,
        std::forward_as_tuple(cur_dim),
```
- EN: This block iterates over collections or graph structures. Key symbols: `size`, `emplace`, `forward_as_tuple`.
- CN: 该代码块遍历集合或图结构。关键符号：`size`, `emplace`, `forward_as_tuple`。

### Lines 166-173
```cpp
        std::forward_as_tuple(
            orig_tensor_indices[cur_dim - dim_offset], aten::index));
    cur_dim++;
  }

  // Each dimension should have its associated index tensor.
  TORCH_INTERNAL_ASSERT((int64_t)dim_index_map.size() == cur_dim);
  return dim_index_map;
```
- EN: This block produces a result or forwards a computed value. Key symbols: `forward_as_tuple`, `size`.
- CN: 该代码块返回结果或转发已计算的值。关键符号：`forward_as_tuple`, `size`。

### Lines 174-186
```cpp
}

// Convert slice/select operators to tensor indices.
// Reshape the tensor indices according to their axis.
// E.g.                 x[1:3, 0, ind1, ind2] = y
//  slice index shape:   [2,   1, 1 ]
//  select index shape:  [     1, 1 ]
//  ind1 shape:          [        _ ]
//  ind2 shape:          [        _ ]
// where _ is the original size of ind1 and ind2.
// ind1 and ind2 are both 1-d tensors since currently we only supports 1-d
// tensor indices.
std::vector<Value*> ReshapeToAdvancedIndexingFormat(
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `ReshapeToAdvancedIndexingFormat`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`ReshapeToAdvancedIndexingFormat`。

### Lines 187-195
```cpp
    Graph* graph,
    Node* index_put_node,
    std::unordered_map<int64_t, ConvertedIndex>& dim_index_map) {
  std::vector<Value*> indices;

  size_t min_index_dim = dim_index_map.size();
  size_t max_index_dim = 0;
  size_t tensor_ind_count = 0;
  for (const auto i : c10::irange(dim_index_map.size())) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `size`, `irange`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`size`, `irange`。

### Lines 196-206
```cpp
    auto index_i = dim_index_map.find(i);
    TORCH_INTERNAL_ASSERT(index_i != dim_index_map.end());
    if (index_i->second.orig_node_kind == aten::index) {
      if (i < min_index_dim)
        min_index_dim = i;
      if (i > max_index_dim)
        max_index_dim = i;
      tensor_ind_count++;
    }
  }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `find`, `end`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`find`, `end`。

### Lines 207-214
```cpp
  if (((max_index_dim - min_index_dim + 1) != tensor_ind_count) &&
      tensor_ind_count != 0) {
    TORCH_CHECK(
        false,
        "Only consecutive 1-d tensor indices are supported in exporting aten::index_put to ONNX.",
        "Check https://pytorch.org/docs/stable/onnx.html#indexing for details");
  }

```
- EN: This block handles conditional branches. Key symbols: no dominant local symbol names.
- CN: 该代码块处理条件分支。关键符号：无明显局部符号。

### Lines 215-221
```cpp
  size_t tensor_ind_offset = tensor_ind_count == 0 ? 0 : tensor_ind_count - 1;
  WithInsertPoint guard(index_put_node);
  for (const auto i : c10::irange(dim_index_map.size())) {
    size_t ind_size = 0;
    auto index_i = dim_index_map.find(i);
    TORCH_INTERNAL_ASSERT(index_i != dim_index_map.end());
    Value* index = index_i->second.index;
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `guard`, `irange`, `size`, `find`, `end`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`guard`, `irange`, `size`, `find`, `end`。

### Lines 222-232
```cpp
    switch (index_i->second.orig_node_kind) {
      case aten::select:
      case aten::slice: {
        if (i < min_index_dim) {
          ind_size = dim_index_map.size() - tensor_ind_offset - i;
        } else {
          ind_size = dim_index_map.size() - i;
        }
        break;
      }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `size`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`size`。

### Lines 233-241
```cpp
      case aten::index: {
        ind_size = dim_index_map.size() - tensor_ind_offset - min_index_dim;
        break;
      }
      default:
        TORCH_CHECK(
            false, "Unexpected node kind ", index_i->second.orig_node_kind);
    }

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `size`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`size`。

### Lines 242-251
```cpp
    if (ind_size != 1) {
      std::vector<int64_t> view_shape(ind_size, 1);
      view_shape[0] = -1;
      auto unsqueezed_index = graph->insert(aten::view, {index, view_shape});
      indices.emplace_back(unsqueezed_index);
    } else {
      indices.emplace_back(index);
    }
  }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `view_shape`, `insert`, `emplace_back`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`view_shape`, `insert`, `emplace_back`。

### Lines 252-265
```cpp
  return indices;
}

// Trace back all the slice & select nodes associated with the index_put node,
// and convert them to associated indices.
// E.g. The IR for x[1:3, 0] = update
//    ...
//    %8 : Float(2, 4) = aten::slice(%0, %4, %5, %6, %7)
//    ...
//    %11 : Float(2) = aten::select(%8, %9, %10)
//    ...
//    %13 : Tensor?[] = prim::ListConstruct()
//    ...
//    %16 : Float(2) = aten::index_put(%11, %13, %14, %15)
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: no dominant local symbol names.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：无明显局部符号。

### Lines 266-279
```cpp
// The aten::index_put node alone does not contain any indices (%13 : Tensor?[]
// = prim::ListConstruct()).
//    ...
//    # Below constructs index from slice node.
//    %23 : Long() = aten::size(%0, %4)
//    %28 : Tensor = aten::arange(%23, %24, %25, %26, %27)
//    %33 : Tensor = aten::slice(%28, %4, %5, %6, %7)
//    %39 : int[] = prim::Constant[value=[-1, 1]]()
//    %40 : Tensor = aten::view(%33, %39)
//    ...
//    # Below constructs index from select node.
//    %36 : int = prim::Constant[value=0]()
//    %37 : Tensor = aten::unsqueeze(%10, %36)
//    %42 : int[] = prim::Constant[value=[-1]]()
```
- EN: Documents the intent, constraints, or compatibility notes that guide the surrounding implementation.
- CN: 说明周围实现所遵循的目标、限制条件或兼容性要求。

### Lines 280-287
```cpp
//    %43 : Tensor = aten::view(%37, %42)
//    ...
//    # Adding the above two indices to index_put
//    %44 : Tensor?[] = prim::ListConstruct(%40, %43)
//    %45 : Float(2, 5) = aten::index_put(%0, %44, %14, %15)
std::vector<Value*> ConvertIndexPutToONNX(
    Block* new_block,
    Node* old_node,
```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `ConvertIndexPutToONNX`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`ConvertIndexPutToONNX`。

### Lines 288-295
```cpp
    py::dict& env,
    py::set& values_in_env) {
  if (old_node->kind() != Symbol::fromQualString("onnx::Placeholder") ||
      (old_node->s(attr::name) != "index_put" &&
       old_node->s(attr::name) != "index_put_")) {
    return {};
  }

```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `kind`, `fromQualString`, `s`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`kind`, `fromQualString`, `s`。

### Lines 296-304
```cpp
  TORCH_INTERNAL_ASSERT(old_node->blocks().size() == 1);
  auto old_graph = old_node->owningGraph();
  auto subblock = old_node->blocks()[0];
  auto index_put_node = subblock->nodes().back()->prev();

  // Find slice and select operators that are associated with this index
  // operator. E.g. x[1:3, 0] = y will generate one slice operator(1:3) and one
  // select operator(0).
  std::vector<Node*> slice_and_select_nodes =
```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `blocks`, `size`, `owningGraph`, `nodes`, `back`, `prev`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`blocks`, `size`, `owningGraph`, `nodes`, `back`, `prev`。

### Lines 305-311
```cpp
      IndexingPatternFinder::FetchSliceAndSelect(index_put_node);
  Node* last_node = !slice_and_select_nodes.empty()
      ? slice_and_select_nodes.back()
      : index_put_node;
  // Update inner block input originates from outside.
  last_node->replaceInput(0, old_node->input(0));
  Value* orig_data = last_node->input(0);
```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `FetchSliceAndSelect`, `empty`, `back`, `replaceInput`, `input`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`FetchSliceAndSelect`, `empty`, `back`, `replaceInput`, `input`。

### Lines 312-319
```cpp

  // Convert slice and select operators to indices.
  std::unordered_map<int64_t, ConvertedIndex> dim_index_map =
      MergeSliceAndSelectToIndices(
          old_graph, index_put_node, slice_and_select_nodes, orig_data, env);

  // Reshape indices to advanced indexing format.
  std::vector<Value*> indices =
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `MergeSliceAndSelectToIndices`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`MergeSliceAndSelectToIndices`。

### Lines 320-327
```cpp
      ReshapeToAdvancedIndexingFormat(old_graph, index_put_node, dim_index_map);

  // Create new index_put node with converted indices.
  const auto list_indices =
      old_graph->createList(OptionalType::ofTensor(), indices)
          ->insertBefore(index_put_node)
          ->output();
  auto new_index_put_node = old_graph->create(
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `ReshapeToAdvancedIndexingFormat`, `createList`, `ofTensor`, `insertBefore`, `output`, `create`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`ReshapeToAdvancedIndexingFormat`, `createList`, `ofTensor`, `insertBefore`, `output`, `create`。

### Lines 328-335
```cpp
      aten::index_put,
      {orig_data,
       list_indices,
       index_put_node->input(2),
       index_put_node->input(3)});
  new_index_put_node->insertBefore(index_put_node);
  new_index_put_node->copyMetadata(index_put_node);
  auto new_index_put = new_index_put_node->output();
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `input`, `insertBefore`, `copyMetadata`, `output`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`input`, `insertBefore`, `copyMetadata`, `output`。

### Lines 336-345
```cpp
  new_index_put->copyMetadata(index_put_node->output());
  index_put_node->output()->replaceAllUsesWith(new_index_put);

  // Convert aten type to onnx type.
  EraseNumberTypesOnBlock(subblock);
  EliminateDeadCode(
      subblock,
      true,
      DCESideEffectPolicy::ALLOW_DELETING_NODES_WITH_SIDE_EFFECTS);

```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `copyMetadata`, `output`, `replaceAllUsesWith`, `EraseNumberTypesOnBlock`, `EliminateDeadCode`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`copyMetadata`, `output`, `replaceAllUsesWith`, `EraseNumberTypesOnBlock`, `EliminateDeadCode`。

### Lines 346-352
```cpp
  // Convert all the new aten nodes that were just created to onnx.
  // New onnx nodes are appended at the end of new_block.
  for (auto at_n : subblock->nodes()) {
    if (at_n == subblock->param_node() || at_n == subblock->return_node()) {
      continue;
    }

```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `nodes`, `param_node`, `return_node`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`nodes`, `param_node`, `return_node`。

### Lines 353-360
```cpp
    NodeToONNX(
        at_n,
        new_block,
        torch::onnx::OperatorExportTypes::ONNX,
        env,
        values_in_env);
  }

```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `NodeToONNX`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`NodeToONNX`。

### Lines 361-368
```cpp
  // Find onnx outputs corresponding to the aten outputs of index_put.
  std::vector<Value*> outs;
  for (auto o : subblock->return_node()->inputs()) {
    auto py_value = env[py::cast(o)];
    Value* value = py_value.cast<Value*>();
    outs.emplace_back(value);
  }
  return outs;
```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `return_node`, `inputs`, `cast`, `emplace_back`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`return_node`, `inputs`, `cast`, `emplace_back`。

### Lines 369-375
```cpp
}

} // namespace

std::vector<Value*> ConvertPatternFromSubblock(
    Block* new_block,
    Node* old_node,
```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `ConvertPatternFromSubblock`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`ConvertPatternFromSubblock`。

### Lines 376-383
```cpp
    py::dict& env,
    py::set& values_in_env) {
  std::vector<Value*> res;

  if (old_node->kind() != Symbol::fromQualString("onnx::Placeholder")) {
    return res;
  }

```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `kind`, `fromQualString`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`kind`, `fromQualString`。

### Lines 384-390
```cpp
  // The pattern conversion code should not alter nodes outside the Placeholder
  // subblock.
  auto op_name = old_node->s(attr::name);
  if (op_name == "index_put" || op_name == "index_put_") {
    res = ConvertIndexPutToONNX(new_block, old_node, env, values_in_env);
  }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `s`, `ConvertIndexPutToONNX`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`s`, `ConvertIndexPutToONNX`。

### Lines 391-394
```cpp
  return res;
}

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
- EN: Type and shape reasoning — the code propagates static information across the IR.
  CN: 类型与形状推理——代码会在 IR 中传播静态信息。
- EN: Export interop — the implementation helps bridge TorchScript graphs to ONNX-style representations.
  CN: 导出互操作——实现帮助 TorchScript 图与 ONNX 风格表示之间建立桥接。
- EN: Subsystem integration — the implementation collaborates with nearby JIT/frontend headers and runtime helpers.
  CN: 子系统集成——实现会与附近的 JIT/前端头文件以及运行时辅助模块协同工作。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `c10/util/irange.h`, `torch/csrc/jit/passes/dead_code_elimination.h`, `torch/csrc/jit/passes/erase_number_types.h`, `torch/csrc/jit/passes/onnx.h`, `torch/csrc/jit/passes/onnx/pattern_conversion/common.h`, `torch/csrc/jit/passes/onnx/pattern_conversion/pattern_conversion.h`, `ATen/ScalarOps.h`
- External includes / 外部头文件: `iostream`
- Namespaces / 命名空间: `torch::jit`
- Representative symbols / 代表性符号: `CreateSizeOfDim`, `owningGraph`, `guard`, `insert`, `ConvertSelectToIndex`, `ConvertSliceToIndex`, `inputs`, `size`, `NamedValue`, `create`, `...`
