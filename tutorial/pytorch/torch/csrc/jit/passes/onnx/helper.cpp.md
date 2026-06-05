# helper.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/passes/onnx/helper.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements the TorchScript JIT pass logic for helper, including graph analysis and rewrites.
- 用途 (CN): 实现与 helper 相关的 TorchScript JIT pass 逻辑，包括图分析与改写。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12
```cpp
#include <torch/csrc/jit/jit_log.h>
#include <torch/csrc/jit/passes/onnx/helper.h>
#include <torch/csrc/onnx/back_compat.h>

#include <ATen/ScalarOps.h>

#ifndef AT_PER_OPERATOR_HEADERS
#include <ATen/Functions.h>
#else
#include <ATen/ops/unsqueeze.h>
#endif

```
- EN: Pulls in the headers needed by the helper logic. Internal dependencies: `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/onnx/helper.h`, `torch/csrc/onnx/back_compat.h`, `ATen/ScalarOps.h`, `ATen/Functions.h`, `...`; external dependencies: none.
- CN: 为 helper 相关逻辑引入所需头文件。内部依赖：`torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/onnx/helper.h`, `torch/csrc/onnx/back_compat.h`, `ATen/ScalarOps.h`, `ATen/Functions.h`, `...`；外部依赖：无。

### Lines 13-23
```cpp
namespace torch::jit {
namespace onnx {
using namespace ::c10::onnx;

} // namespace onnx

ValueToParamPairMap buildValueToParamsMap(
    Block* b,
    const ParamMap& paramsDict) {
  ValueToParamPairMap valsToParamsMap;
  for (auto& input : b->inputs()) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `namespace`, `buildValueToParamsMap`, `inputs`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`namespace`, `buildValueToParamsMap`, `inputs`。

### Lines 24-31
```cpp
    auto it = paramsDict.find(input->debugName());
    if (it != paramsDict.end()) {
      valsToParamsMap.emplace(input, *it);
    }
  }
  return valsToParamsMap;
}

```
- EN: This block handles conditional branches; produces a result or forwards a computed value. Key symbols: `find`, `debugName`, `end`, `emplace`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值。关键符号：`find`, `debugName`, `end`, `emplace`。

### Lines 32-40
```cpp
void eraseUnusedBlockInputs(Block* b) {
  for (size_t i_1 = b->inputs().size(); i_1 > 0; --i_1) {
    size_t i = i_1 - 1;
    if (!b->inputs().at(i)->hasUses()) {
      b->eraseInput(i);
    }
  }
}

```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `eraseUnusedBlockInputs`, `inputs`, `size`, `hasUses`, `eraseInput`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`eraseUnusedBlockInputs`, `inputs`, `size`, `hasUses`, `eraseInput`。

### Lines 41-50
```cpp
void eraseUnusedValuesFromMap(ValueToParamPairMap& valsToParamsMap) {
  std::erase_if(
      valsToParamsMap, [](const auto& pr) { return !pr.first->hasUses(); });
}

void buildParamsMapFromValueToParamsMap(
    const ValueToParamPairMap& valsToParamsMap,
    ParamMap& paramsDict) {
  paramsDict.clear();
  for (const auto& nameTensorParamPair : valsToParamsMap) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `eraseUnusedValuesFromMap`, `erase_if`, `hasUses`, `buildParamsMapFromValueToParamsMap`, `clear`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`eraseUnusedValuesFromMap`, `erase_if`, `hasUses`, `buildParamsMapFromValueToParamsMap`, `clear`。

### Lines 51-58
```cpp
    paramsDict.insert(nameTensorParamPair.second);
  }
}

std::optional<at::ScalarType> ONNXTypeToATenType(int32_t onnx_type) {
  switch (onnx_type) {
    case ::ONNX_NAMESPACE::TensorProto_DataType_UNDEFINED:
      return at::ScalarType::Undefined;
```
- EN: This block produces a result or forwards a computed value. Key symbols: `insert`, `ONNXTypeToATenType`.
- CN: 该代码块返回结果或转发已计算的值。关键符号：`insert`, `ONNXTypeToATenType`。

### Lines 59-66
```cpp
    case ::ONNX_NAMESPACE::TensorProto_DataType_FLOAT:
      return at::kFloat;
    case ::ONNX_NAMESPACE::TensorProto_DataType_UINT8:
      return at::kByte;
    case ::ONNX_NAMESPACE::TensorProto_DataType_INT8:
      return at::kChar;
    case ::ONNX_NAMESPACE::TensorProto_DataType_INT16:
      return at::kShort;
```
- EN: This block produces a result or forwards a computed value. Key symbols: no dominant local symbol names.
- CN: 该代码块返回结果或转发已计算的值。关键符号：无明显局部符号。

### Lines 67-74
```cpp
    case ::ONNX_NAMESPACE::TensorProto_DataType_INT32:
      return at::kInt;
    case ::ONNX_NAMESPACE::TensorProto_DataType_INT64:
      return at::kLong;
    case ::ONNX_NAMESPACE::TensorProto_DataType_BOOL:
      return at::kBool;
    case ::ONNX_NAMESPACE::TensorProto_DataType_FLOAT16:
      return at::kHalf;
```
- EN: This block produces a result or forwards a computed value. Key symbols: no dominant local symbol names.
- CN: 该代码块返回结果或转发已计算的值。关键符号：无明显局部符号。

### Lines 75-82
```cpp
    case ::ONNX_NAMESPACE::TensorProto_DataType_DOUBLE:
      return at::kDouble;
    case ::ONNX_NAMESPACE::TensorProto_DataType_COMPLEX64:
      return at::kComplexFloat;
    case ::ONNX_NAMESPACE::TensorProto_DataType_COMPLEX128:
      return at::kComplexDouble;
    case ::ONNX_NAMESPACE::TensorProto_DataType_BFLOAT16:
      return at::kBFloat16;
```
- EN: This block produces a result or forwards a computed value. Key symbols: no dominant local symbol names.
- CN: 该代码块返回结果或转发已计算的值。关键符号：无明显局部符号。

### Lines 83-90
```cpp
    case ::torch::onnx::TensorProto_DataType_FLOAT8E5M2:
      return at::kFloat8_e5m2;
    case ::torch::onnx::TensorProto_DataType_FLOAT8E5M2FNUZ:
      return at::kFloat8_e5m2fnuz;
    case ::torch::onnx::TensorProto_DataType_FLOAT8E4M3FN:
      return at::kFloat8_e4m3fn;
    case ::torch::onnx::TensorProto_DataType_FLOAT8E4M3FNUZ:
      return at::kFloat8_e4m3fnuz;
```
- EN: This block produces a result or forwards a computed value. Key symbols: no dominant local symbol names.
- CN: 该代码块返回结果或转发已计算的值。关键符号：无明显局部符号。

### Lines 91-99
```cpp
    default:
      TORCH_CHECK(
          false,
          "ONNX type ",
          onnx_type,
          " is an unexpected tensor scalar type");
  }
}

```
- EN: This block implements local helper logic for helper. Key symbols: no dominant local symbol names.
- CN: 该代码块实现与 helper 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 100-107
```cpp
Node* addNodeToBlock(Block* block, Symbol kind, ArrayRef<Value*> inputs) {
  auto new_node = block->appendNode(block->owningGraph()->create(kind));
  for (auto input : inputs) {
    new_node->addInput(input);
  }
  return new_node;
}

```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `addNodeToBlock`, `appendNode`, `owningGraph`, `create`, `addInput`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`addNodeToBlock`, `appendNode`, `owningGraph`, `create`, `addInput`。

### Lines 108-117
```cpp
Value* addInputToBlock(Block* block) {
  return block->addInput();
}

namespace {
::ONNX_NAMESPACE::TensorProto_DataType ATenTypeToOnnxType_aux(
    at::ScalarType at_type) {
  switch (at_type) {
    case at::kDouble:
      return ::ONNX_NAMESPACE::TensorProto_DataType_DOUBLE;
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `addInputToBlock`, `addInput`, `ATenTypeToOnnxType_aux`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`addInputToBlock`, `addInput`, `ATenTypeToOnnxType_aux`。

### Lines 118-125
```cpp
    case at::kFloat:
      return ::ONNX_NAMESPACE::TensorProto_DataType_FLOAT;
    case at::kHalf:
      return ::ONNX_NAMESPACE::TensorProto_DataType_FLOAT16;
    case at::kByte:
      return ::ONNX_NAMESPACE::TensorProto_DataType_UINT8;
    case at::kChar:
      return ::ONNX_NAMESPACE::TensorProto_DataType_INT8;
```
- EN: This block produces a result or forwards a computed value. Key symbols: no dominant local symbol names.
- CN: 该代码块返回结果或转发已计算的值。关键符号：无明显局部符号。

### Lines 126-133
```cpp
    case at::kShort:
      return ::ONNX_NAMESPACE::TensorProto_DataType_INT16;
    case at::kInt:
      return ::ONNX_NAMESPACE::TensorProto_DataType_INT32;
    case at::kLong:
      return ::ONNX_NAMESPACE::TensorProto_DataType_INT64;
    case at::kBool:
      return ::ONNX_NAMESPACE::TensorProto_DataType_BOOL;
```
- EN: This block produces a result or forwards a computed value. Key symbols: no dominant local symbol names.
- CN: 该代码块返回结果或转发已计算的值。关键符号：无明显局部符号。

### Lines 134-147
```cpp
    case at::kQInt8:
      return ::ONNX_NAMESPACE::TensorProto_DataType_INT8;
    case at::kQUInt8:
      return ::ONNX_NAMESPACE::TensorProto_DataType_UINT8;
    case at::kQInt32:
      return ::ONNX_NAMESPACE::TensorProto_DataType_INT32;
    default:
      TORCH_CHECK(
          false,
          "ScalarType ",
          toString(at_type),
          " is an unexpected tensor scalar type");
  }
}
```
- EN: This block produces a result or forwards a computed value. Key symbols: `toString`.
- CN: 该代码块返回结果或转发已计算的值。关键符号：`toString`。

### Lines 148-154
```cpp
} // namespace

int ATenTypeToOnnxType(at::ScalarType at_type) {
  return static_cast<int>(ATenTypeToOnnxType_aux(at_type));
}

Node* createONNXUnsqueeze(
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `ATenTypeToOnnxType`, `ATenTypeToOnnxType_aux`, `createONNXUnsqueeze`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`ATenTypeToOnnxType`, `ATenTypeToOnnxType_aux`, `createONNXUnsqueeze`。

### Lines 155-163
```cpp
    Graph* graph,
    Node* n_to_insert_before,
    Value* input,
    int axis,
    int opset_version) {
  Node* unsqueeze_node = graph->create(onnx::Unsqueeze, 1);
  unsqueeze_node->addInput(input);
  unsqueeze_node->insertBefore(n_to_insert_before);
  if (opset_version >= OPSET_VERSION_13) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `create`, `addInput`, `insertBefore`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`create`, `addInput`, `insertBefore`。

### Lines 164-174
```cpp
    // ONNX spec sets `axes` as input for opset >= 13.
    Node* unsqueeze_axes = graph->create(onnx::Constant, 1);
    unsqueeze_axes->insertBefore(unsqueeze_node);
    unsqueeze_axes->t_(
        attr::value, at::unsqueeze(at::scalar_to_tensor(at::Scalar(axis)), 0));
    unsqueeze_node->addInput(unsqueeze_axes->output());
  } else {
    // ONNX spec sets `axes` as attribute for opset < 13.
    unsqueeze_node->is_(attr::axes, {0});
  }
  return unsqueeze_node;
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `create`, `insertBefore`, `t_`, `unsqueeze`, `scalar_to_tensor`, `Scalar`, `...`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`create`, `insertBefore`, `t_`, `unsqueeze`, `scalar_to_tensor`, `Scalar`, `...`。

### Lines 175-181
```cpp
}

Node* createONNXConstant(
    Graph* graph,
    Node* n_to_insert_before,
    at::Tensor value) {
  Node* constant_node = graph->create(onnx::Constant, 1);
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `createONNXConstant`, `create`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`createONNXConstant`, `create`。

### Lines 182-188
```cpp
  constant_node->insertBefore(n_to_insert_before);
  constant_node->t_(attr::value, std::move(value));
  return constant_node;
}

bool isValidToTransformToONNXConcatNode(Node* lc_node) {
  return !lc_node->inputs().empty();
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `insertBefore`, `t_`, `move`, `isValidToTransformToONNXConcatNode`, `inputs`, `empty`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`insertBefore`, `t_`, `move`, `isValidToTransformToONNXConcatNode`, `inputs`, `empty`。

### Lines 189-199
```cpp
}

Node* transformToONNXConcatNode(
    Graph* g,
    Node* lc_node,
    bool need_new_input,
    int opset_version) {
  // ListConstruct Int[] output case, we need to transform to ONNX
  // Concat to ensure the output is a single tensor(dynamic) type in
  // order to be consumed as inputs
  std::vector<Value*> unsqueezed;
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `transformToONNXConcatNode`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`transformToONNXConcatNode`。

### Lines 200-211
```cpp
  auto new_node = need_new_input ? g->return_node() : lc_node;

  for (auto* input : lc_node->inputs()) {
    auto new_input =
        need_new_input ? g->addInput()->copyMetadata(input) : input;
    // This particular Concat operation concats along axis=0 and this requires
    // inputs to the node to have the same shape along dim-0. To ensure this,
    // unsqueeze nodes are added such that all shapes along dim-0 are 1.
    // Certain inputs from ListConstruct Int[] could be combinations of scalars
    // and 1-D tensors, For inputs that are already 1-D tensors, we skip the
    // step of creating a corresponding unsqueeze node.
    if (auto type = new_input->type()->cast<TensorType>()) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `return_node`, `inputs`, `addInput`, `copyMetadata`, `type`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`return_node`, `inputs`, `addInput`, `copyMetadata`, `type`。

### Lines 212-222
```cpp
      if (type->dim() && type->dim() == 1U) {
        unsqueezed.emplace_back(new_input);
        continue;
      }
    }
    Node* unsqueezed_node =
        createONNXUnsqueeze(g, new_node, new_input, 0, opset_version);
    unsqueezed_node->copyMetadata(lc_node);
    unsqueezed.emplace_back(unsqueezed_node->output());
  }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `dim`, `emplace_back`, `createONNXUnsqueeze`, `copyMetadata`, `output`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`dim`, `emplace_back`, `createONNXUnsqueeze`, `copyMetadata`, `output`。

### Lines 223-230
```cpp
  Node* concat_node = need_new_input
      ? g->insertNode(g->create(onnx::Concat, 1))
      : g->create(onnx::Concat, 1)->insertBefore(lc_node);
  concat_node->i_(attr::axis, 0);
  for (auto v : unsqueezed) {
    concat_node->addInput(v);
  }

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `insertNode`, `create`, `insertBefore`, `i_`, `addInput`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`insertNode`, `create`, `insertBefore`, `i_`, `addInput`。

### Lines 231-237
```cpp
  return concat_node;
}

static void ONNXLintGraph(
    const Block* b,
    std::vector<NodeKind>& n_miss_source_range,
    std::vector<NodeKind>& n_miss_scope) {
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `ONNXLintGraph`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`ONNXLintGraph`。

### Lines 238-247
```cpp
  for (const auto* n : b->nodes()) {
    for (const auto* sub_b : n->blocks()) {
      ONNXLintGraph(sub_b, n_miss_source_range, n_miss_scope);
    }

    if (nullptr == n->sourceRange().source()) {
      GRAPH_DEBUG("Node does not set sourceRange:", *n);
      n_miss_source_range.emplace_back(n->kind());
    }
    if (n->scopeName().empty()) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `nodes`, `blocks`, `ONNXLintGraph`, `sourceRange`, `source`, `emplace_back`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`nodes`, `blocks`, `ONNXLintGraph`, `sourceRange`, `source`, `emplace_back`, `...`。

### Lines 248-254
```cpp
      GRAPH_DEBUG("Node does not set scope:", *n);
      n_miss_scope.emplace_back(n->kind());
    }
  }
}

void ONNXLintGraph(const std::shared_ptr<Graph>& graph) {
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `emplace_back`, `kind`, `ONNXLintGraph`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`emplace_back`, `kind`, `ONNXLintGraph`。

### Lines 255-268
```cpp
  // Print nodes that do not have scope/source range covered.
  std::vector<NodeKind> n_miss_source_range, n_miss_scope;
  ONNXLintGraph(graph->block(), n_miss_source_range, n_miss_scope);
  auto count_const = [](const std::vector<NodeKind>& vec) -> size_t {
    size_t count = 0;
    for (auto k : vec) {
      switch (k) {
        case prim::Constant:
        case prim::ListConstruct:
        case onnx::Constant:
          count++;
          break;
      }
    }
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `ONNXLintGraph`, `block`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`ONNXLintGraph`, `block`。

### Lines 269-282
```cpp
    return count;
  };
  auto const_count_src = count_const(n_miss_source_range);
  auto const_count_scope = count_const(n_miss_scope);
  GRAPH_UPDATE(
      "Missing source range.\n",
      "Total ",
      n_miss_source_range.size(),
      " nodes. Including ",
      const_count_src,
      " constants.");
  GRAPH_UPDATE(
      "Missing scope.\n",
      "Total ",
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `count_const`, `size`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`count_const`, `size`。

### Lines 283-289
```cpp
      n_miss_scope.size(),
      " nodes. Including ",
      const_count_scope,
      " constants.");
}

} // namespace torch::jit
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `size`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`size`。


## Key Concepts / 关键概念
- EN: IR graph manipulation — the file inspects or rewrites TorchScript graphs and nodes.
  CN: IR 图操作——该文件会检查或改写 TorchScript 的图、节点与块。
- EN: Optimization pass — the code simplifies graphs by removing redundant work or folding values.
  CN: 优化 pass——代码通过删除冗余计算或常量折叠来简化图。
- EN: Type and shape reasoning — the code propagates static information across the IR.
  CN: 类型与形状推理——代码会在 IR 中传播静态信息。
- EN: Export interop — the implementation helps bridge TorchScript graphs to ONNX-style representations.
  CN: 导出互操作——实现帮助 TorchScript 图与 ONNX 风格表示之间建立桥接。
- EN: Subsystem integration — the implementation collaborates with nearby JIT/frontend headers and runtime helpers.
  CN: 子系统集成——实现会与附近的 JIT/前端头文件以及运行时辅助模块协同工作。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/onnx/helper.h`, `torch/csrc/onnx/back_compat.h`, `ATen/ScalarOps.h`, `ATen/Functions.h`, `ATen/ops/unsqueeze.h`
- External includes / 外部头文件: 无
- Namespaces / 命名空间: `torch::jit`, `onnx`
- Representative symbols / 代表性符号: `namespace`, `buildValueToParamsMap`, `inputs`, `find`, `debugName`, `end`, `emplace`, `eraseUnusedBlockInputs`, `size`, `hasUses`, `...`
