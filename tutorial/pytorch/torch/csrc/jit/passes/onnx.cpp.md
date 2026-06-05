# onnx.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/passes/onnx.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements the TorchScript JIT pass logic for onnx, including graph analysis and rewrites.
- 用途 (CN): 实现与 onnx 相关的 TorchScript JIT pass 逻辑，包括图分析与改写。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15
```cpp
#include <torch/csrc/jit/passes/onnx.h>

#include <c10/util/Exception.h>
#include <c10/util/irange.h>
#include <torch/csrc/autograd/symbolic.h>
#include <torch/csrc/jit/ir/constants.h>
#include <torch/csrc/jit/jit_log.h>
#include <torch/csrc/jit/passes/dead_code_elimination.h>
#include <torch/csrc/jit/passes/onnx/constant_map.h>
#include <torch/csrc/jit/passes/onnx/helper.h>
#include <torch/csrc/jit/passes/onnx/onnx_log.h>
#include <torch/csrc/jit/passes/onnx/shape_type_inference.h>
#include <torch/csrc/jit/python/python_ir.h>
#include <sstream>

```
- EN: Pulls in the headers needed by the onnx logic. Internal dependencies: `torch/csrc/jit/passes/onnx.h`, `c10/util/Exception.h`, `c10/util/irange.h`, `torch/csrc/autograd/symbolic.h`, `torch/csrc/jit/ir/constants.h`, `...`; external dependencies: `sstream`.
- CN: 为 onnx 相关逻辑引入所需头文件。内部依赖：`torch/csrc/jit/passes/onnx.h`, `c10/util/Exception.h`, `c10/util/irange.h`, `torch/csrc/autograd/symbolic.h`, `torch/csrc/jit/ir/constants.h`, `...`；外部依赖：`sstream`。

### Lines 16-24
```cpp
namespace torch::jit {

static void removePrintOps(Block* block) {
  for (auto it = block->nodes().begin(), end = block->nodes().end(); it != end;
       ++it) {
    for (auto b : it->blocks()) {
      removePrintOps(b);
    }
    if (it->kind() == prim::Print || it->kind() == aten::warn) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `removePrintOps`, `nodes`, `begin`, `end`, `blocks`, `kind`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`removePrintOps`, `nodes`, `begin`, `end`, `blocks`, `kind`。

### Lines 25-40
```cpp
      for (size_t i = 0; i < it->inputs().size();) {
        auto input = it->inputs().at(i);
        // only handling constants bc of potential side effects
        if (input->uses().size() == 1 &&
            input->node()->kind() == prim::Constant) {
          it->removeInput(i);
          input->node()->destroy();
        } else {
          ++i;
        }
      }
      it.destroyCurrent();
    }
  }
}

```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `inputs`, `size`, `uses`, `node`, `kind`, `removeInput`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`inputs`, `size`, `uses`, `node`, `kind`, `removeInput`, `...`。

### Lines 41-49
```cpp
void RemovePrintOps(std::shared_ptr<Graph>& graph) {
  removePrintOps(graph->block());
  GRAPH_DUMP("After RemovePrintOps: ", graph);
}

static void checkONNXCompatibility(const c10::FunctionSchema& schema) {
  // in ONNX, all inputs are tensors, no support for tensor list
  // so at most one input tensor list is supported
  bool has_tensor_list = false;
```
- EN: This block manipulates TorchScript IR objects; advances frontend parsing or lowering state; protects shared state or ordering assumptions. Key symbols: `RemovePrintOps`, `removePrintOps`, `block`, `checkONNXCompatibility`.
- CN: 该代码块操作 TorchScript IR 对象；推进前端解析或降级状态；保护共享状态或执行顺序假设。关键符号：`RemovePrintOps`, `removePrintOps`, `block`, `checkONNXCompatibility`。

### Lines 50-61
```cpp
  const auto& args = schema.arguments();
  for (const auto& arg : args) {
    if (arg.name() == "_caffe2_preallocated_outputs") {
      continue;
    }
    auto type = arg.type();
    if (type->kind() == TypeKind::OptionalType) {
      type = reinterpret_cast<OptionalType*>(type.get())->getElementType();
      // recursive optional type is not supported
      TORCH_INTERNAL_ASSERT(type->kind() != TypeKind::OptionalType);
    }
    if (type->kind() == TypeKind::ListType) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; advances frontend parsing or lowering state. Key symbols: `arguments`, `name`, `type`, `kind`, `get`, `getElementType`.
- CN: 该代码块处理条件分支；遍历集合或图结构；推进前端解析或降级状态。关键符号：`arguments`, `name`, `type`, `kind`, `get`, `getElementType`。

### Lines 62-73
```cpp
      const auto& elem_type =
          reinterpret_cast<ListType*>(type.get())->getElementType();
      if (elem_type->isSubtypeOf(*TensorType::get())) {
        TORCH_INTERNAL_ASSERT(
            !has_tensor_list,
            "ONNX export supports at most one TensorList as input.");
        has_tensor_list = true;
      }
    }
  }
}

```
- EN: This block handles conditional branches. Key symbols: `get`, `getElementType`, `isSubtypeOf`.
- CN: 该代码块处理条件分支。关键符号：`get`, `getElementType`, `isSubtypeOf`。

### Lines 74-83
```cpp
static void preprocessCaffe2Ops(Block* block) {
  for (auto it = block->nodes().begin(), end = block->nodes().end(); it != end;
       ++it) {
    for (auto b : it->blocks()) {
      preprocessCaffe2Ops(b);
    }
    if (it->kind().is_caffe2()) {
      const auto& schema = it->schema();
      checkONNXCompatibility(schema);
      std::vector<Value*> origin_inputs;
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; advances frontend parsing or lowering state; protects shared state or ordering assumptions. Key symbols: `preprocessCaffe2Ops`, `nodes`, `begin`, `end`, `blocks`, `kind`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；推进前端解析或降级状态；保护共享状态或执行顺序假设。关键符号：`preprocessCaffe2Ops`, `nodes`, `begin`, `end`, `blocks`, `kind`, `...`。

### Lines 84-94
```cpp
      for (Value* v : it->inputs()) {
        origin_inputs.push_back(v);
      }
      it->removeAllInputs();
      const auto& args = schema.arguments();
      size_t origin_inputs_index = 0;
      for (const auto& arg : args) {
        const auto& type = arg.type();
        TORCH_INTERNAL_ASSERT(origin_inputs_index < origin_inputs.size());
        const auto& origin_input = origin_inputs[origin_inputs_index++];
        if (type->kind() == TypeKind::OptionalType &&
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `inputs`, `push_back`, `removeAllInputs`, `arguments`, `type`, `size`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`inputs`, `push_back`, `removeAllInputs`, `arguments`, `type`, `size`, `...`。

### Lines 95-112
```cpp
            origin_input->mustBeNone()) {
          continue;
        }
        if (type->isSubtypeOf(*TensorType::get())) {
          it->addInput(origin_input);
        } else if (
            type->kind() == TypeKind::BoolType ||
            type->kind() == TypeKind::IntType) {
          const auto* constant_node = origin_input->node();
          TORCH_INTERNAL_ASSERT(constant_node->kind() == prim::Constant);
          it->i_(Symbol::attr(arg.name()), constant_node->i(attr::value));
        } else if (type->kind() == TypeKind::FloatType) {
          const auto* constant_node = origin_input->node();
          TORCH_INTERNAL_ASSERT(constant_node->kind() == prim::Constant);
          it->f_(Symbol::attr(arg.name()), constant_node->f(attr::value));
        } else if (type->kind() == TypeKind::StringType) {
          const auto* constant_node = origin_input->node();
          TORCH_INTERNAL_ASSERT(constant_node->kind() == prim::Constant);
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `mustBeNone`, `isSubtypeOf`, `get`, `addInput`, `kind`, `node`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`mustBeNone`, `isSubtypeOf`, `get`, `addInput`, `kind`, `node`, `...`。

### Lines 113-123
```cpp
          it->s_(Symbol::attr(arg.name()), constant_node->s(attr::value));
        } else if (type->kind() == TypeKind::ListType) {
          const auto& list_node = origin_input->node();
          const auto& elem_type = type->castRaw<ListType>()->getElementType();
          TORCH_INTERNAL_ASSERT(
              list_node->kind() == prim::ListConstruct ||
              list_node->kind() == prim::Constant);
          if (elem_type->isSubtypeOf(*TensorType::get())) {
            TORCH_INTERNAL_ASSERT(list_node->kind(), prim::ListConstruct);
            const auto& tensor_list = origin_input->node()->inputs();
            for (const auto& t : tensor_list) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `s_`, `attr`, `name`, `s`, `kind`, `node`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`s_`, `attr`, `name`, `s`, `kind`, `node`, `...`。

### Lines 124-141
```cpp
              it->addInput(t);
            }
          } else if (elem_type->kind() == TypeKind::FloatType) {
            std::vector<double> values;
            if (list_node->kind() == prim::ListConstruct) {
              for (const auto* elem_input : list_node->inputs()) {
                const auto* constant_node = elem_input->node();
                TORCH_INTERNAL_ASSERT(constant_node->kind() == prim::Constant);
                values.push_back(constant_node->f(attr::value));
              }
            } else { // is a constant list
              values = list_node->fs(attr::value);
            }
            it->fs_(Symbol::attr(arg.name()), values);
          } else {
            throw std::runtime_error(
                "Unhandled scalar arg: " + arg.name() +
                ", type: " + c10::typeKindToString(elem_type->kind()));
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `addInput`, `kind`, `inputs`, `node`, `push_back`, `f`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`addInput`, `kind`, `inputs`, `node`, `push_back`, `f`, `...`。

### Lines 142-154
```cpp
          }
        } else {
          throw std::runtime_error(
              "Unsupported input type of arg " + arg.name() +
              " in Caffe2 operator: " + c10::typeKindToString(type->kind()));
        }
      }
    }
  }
  EliminateDeadCode(
      block, true, DCESideEffectPolicy::ALLOW_DELETING_NODES_WITH_SIDE_EFFECTS);
}

```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `runtime_error`, `name`, `typeKindToString`, `kind`, `EliminateDeadCode`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`runtime_error`, `name`, `typeKindToString`, `kind`, `EliminateDeadCode`。

### Lines 155-165
```cpp
void PreprocessCaffe2Ops(std::shared_ptr<Graph>& graph) {
  preprocessCaffe2Ops(graph->block());
  GRAPH_DUMP("After PreprocessCaffe2Ops: ", graph);
}

// Transform PythonOps into Nodes that match ONNX semantics.
std::shared_ptr<Graph> ToONNX(
    std::shared_ptr<Graph>& graph,
    ::torch::onnx::OperatorExportTypes operator_export_type) {
  ConstantValueMap::ClearMaps();
  auto new_graph = std::make_shared<Graph>(graph->current_scope());
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `PreprocessCaffe2Ops`, `preprocessCaffe2Ops`, `block`, `ToONNX`, `ClearMaps`, `current_scope`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`PreprocessCaffe2Ops`, `preprocessCaffe2Ops`, `block`, `ToONNX`, `ClearMaps`, `current_scope`。

### Lines 166-183
```cpp
  py::dict env;
  // Kept identical to values in env. Used for constant-time existence check.
  py::set values_in_env;
  try {
    BlockToONNX(
        graph->block(),
        new_graph->block(),
        operator_export_type,
        env,
        values_in_env);
  } catch (std::runtime_error&) {
    ONNX_LOG(
        "ONNX graph being constructed during exception:\n",
        new_graph->toString());
    throw;
  }
  GRAPH_DUMP("after ToONNX: ", new_graph);
  ConstantValueMap::ClearMaps();
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `BlockToONNX`, `block`, `toString`, `ClearMaps`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`BlockToONNX`, `block`, `toString`, `ClearMaps`。

### Lines 184-198
```cpp
  return new_graph;
}

// BlockToONNX.
// is_sub_block = true means the old_block (aten graph) is in the sub block
// (e.g., if sub block), and we want to convert it into its parent block in onnx
// graph. In this case, we don't register the input/output or eliminate the dead
// code.
py::dict BlockToONNX(
    Block* old_block,
    Block* new_block,
    ::torch::onnx::OperatorExportTypes operator_export_type,
    py::dict& env,
    py::set& values_in_env,
    bool is_sub_block) {
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `BlockToONNX`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`BlockToONNX`。

### Lines 199-207
```cpp
  torch::autograd::SymbolicContext ctx{};
  ctx.block = new_block;

  GRAPH_DEBUG(
      "BlockToONNX: graph of old block: ",
      old_block->owningGraph()->toString());

  // Initialize context and environment
  if (!is_sub_block) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `owningGraph`, `toString`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`owningGraph`, `toString`。

### Lines 208-218
```cpp
    for (auto input : old_block->inputs()) {
      auto n = ctx.block->addInput()->copyMetadata(input);
      auto py_n = py::cast(n);
      env[py::cast(input)] = py_n;
      values_in_env.add(py_n);
    }
  }

  // Determine if all inputs are static. This is used for each node to
  // determine whether or not to propagate shapes.
  if (!is_sub_block) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `inputs`, `addInput`, `copyMetadata`, `cast`, `add`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`inputs`, `addInput`, `copyMetadata`, `cast`, `add`。

### Lines 219-227
```cpp
    bool static_input_shape = AllGraphInputsStatic(ctx.block->owningGraph());
    ConstantValueMap::SetAllGraphInputsStatic(static_input_shape);
  }

  // Finally, visit all nodes in the graph
  for (auto node : old_block->nodes()) {
    NodeToONNX(node, ctx.block, operator_export_type, env, values_in_env);
  }

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `AllGraphInputsStatic`, `owningGraph`, `SetAllGraphInputsStatic`, `nodes`, `NodeToONNX`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`AllGraphInputsStatic`, `owningGraph`, `SetAllGraphInputsStatic`, `nodes`, `NodeToONNX`。

### Lines 228-242
```cpp
  if (is_sub_block) {
    return env;
  }

  for (auto output : old_block->outputs()) {
    auto py_value = env[py::cast(output)];
    Value* value = py_value.cast<Value*>();
    ctx.block->registerOutput(value);
  }
  // Run dce to clean-up unused functional and inplace ops.
  EliminateDeadCode(
      ctx.block,
      true,
      DCESideEffectPolicy::ALLOW_DELETING_NODES_WITH_SIDE_EFFECTS);

```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `outputs`, `cast`, `registerOutput`, `EliminateDeadCode`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`outputs`, `cast`, `registerOutput`, `EliminateDeadCode`。

### Lines 243-251
```cpp
  return py::dict();
}

static bool ConstantFoldCondition(torch::jit::Value* output) {
  auto fold_condition = output->node()->kind() != c10::onnx::Constant &&
      ConstantValueMap::HasValue(output->debugName());
  auto reliable_value =
      ConstantValueMap::GetTypeReliable(output->debugName()).value_or(false);
  return fold_condition && reliable_value;
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `dict`, `ConstantFoldCondition`, `node`, `kind`, `HasValue`, `debugName`, `...`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`dict`, `ConstantFoldCondition`, `node`, `kind`, `HasValue`, `debugName`, `...`。

### Lines 252-266
```cpp
}

void NodeToONNX(
    Node* old_node,
    Block* new_block,
    ::torch::onnx::OperatorExportTypes operator_export_type,
    py::dict& env,
    py::set& values_in_env) {
  py::object onnx_utils =
      py::module::import("torch.onnx._internal.torchscript_exporter.utils");
  py::object onnx_globals =
      py::module::import("torch.onnx._internal.torchscript_exporter._globals");
  py::object onnx_registration = py::module::import(
      "torch.onnx._internal.torchscript_exporter.registration");

```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `NodeToONNX`, `import`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`NodeToONNX`, `import`。

### Lines 267-275
```cpp
  // Setup all the lambda helper functions.

  // Returns a node that n maps to in the new graph
  auto envFn = [&env](Value* n) -> Value* {
    auto py_n = py::cast(n);
    TORCH_CHECK(env.contains(py_n), "Dangling node reference");
    auto py_value = env[py_n];
    TORCH_CHECK(!py_value.is_none(), "Unused node was subsequently used");
    Value* value = py_value.cast<Value*>();
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `cast`, `contains`, `is_none`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`cast`, `contains`, `is_none`。

### Lines 276-285
```cpp
    return value;
  };

  // Put the new outputs in our environment map, and copy the type from the
  // input graph if they were not set by the symbolic. This is called only
  // with results of symbolic call (not for nodes that are just cloned).
  auto setOutputs = [&](const std::string& op_name,
                        Node* node,
                        const value_list& outputs) {
    auto old_outputs = node->outputs();
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `outputs`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`outputs`。

### Lines 286-297
```cpp
    // Count all outputs, excluding Handles
    auto num_old_outputs = old_outputs.size();
    if (outputs.size() != num_old_outputs) {
      std::ostringstream ss;
      ss << "symbolic for " << op_name
         << " produced an incorrect number of outputs (expected ";
      ss << num_old_outputs << ", but got " << outputs.size() << ')';
      throw std::runtime_error(ss.str());
    }
    // For const node, it does not need params_dict info, so set it to {}.
    const ParamMap empty_params_dict = {};
    auto opset_version = py::cast<int>(
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `size`, `outputs`, `runtime_error`, `str`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`size`, `outputs`, `runtime_error`, `str`。

### Lines 298-307
```cpp
        onnx_globals.attr("GLOBALS").attr("export_onnx_opset_version"));
    for (const auto i : c10::irange(num_old_outputs)) {
      auto old = old_outputs[i];
      if (outputs[i]) {
        bool exist_in_env = values_in_env.contains(py::cast(outputs[i]));
        // Update ONNX value debug name with ATen value debug name if existed.
        // Skip if ONNX value already exist in environment.
        // This implies the op is a noop, and the value is owned by
        // other node created elsewhere.
        if (old->hasDebugName() && !exist_in_env) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `attr`, `irange`, `contains`, `cast`, `hasDebugName`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`attr`, `irange`, `contains`, `cast`, `hasDebugName`。

### Lines 308-320
```cpp
          auto old_name = outputs[i]->debugName();
          auto new_name = old->debugNameBase();
          Value* found_value = nullptr;
          bool exists = false;
          // In this scope, we fetch debug_names as a const reference and then
          // construct an iterator exist_name based on it. This iterator will
          // be corrupted if the underlying map of debug_names changes. This
          // will happen as a side-effect of setDebugName. For these reasons,
          // we make an explicit scope for exist_name and make sure that
          // setDebugName is never called with this scope.
          {
            const auto& debug_names = new_block->owningGraph()->debugNames();
            auto exist_name = debug_names.find(new_name);
```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `debugName`, `debugNameBase`, `owningGraph`, `debugNames`, `find`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`debugName`, `debugNameBase`, `owningGraph`, `debugNames`, `find`。

### Lines 321-338
```cpp
            exists = exist_name != debug_names.end();
            if (exists) {
              found_value = exist_name->second;
            }
          }
          outputs[i]->setDebugName(new_name);
          if (exists) {
            found_value->setDebugName(new_name);
          }
          ConstantValueMap::UpdateValueName(old_name, outputs[i]->debugName());
        }
        // Allow symbolic() to skip specifying the type of the return node.
        // Unfortunately, they are on the hook for all internal nodes
        // (though in practice, the types are not computed.)
        //
        // If onnx shape inference is turned on, the new outputs will have
        // types inferred, and they will be merged with the old types.
        if (ConstantFoldCondition(outputs[i])) {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `end`, `setDebugName`, `UpdateValueName`, `debugName`, `ConstantFoldCondition`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`end`, `setDebugName`, `UpdateValueName`, `debugName`, `ConstantFoldCondition`。

### Lines 339-347
```cpp
          // Create a const node if the node output value is in
          // ConstantValueMap.
          auto value =
              ConstantValueMap::GetValue(outputs[i]->debugName()).value();
          Node* const_node =
              new_block->owningGraph()->create(c10::onnx::Constant);
          const_node->t_(attr::value, value);
          const_node->output()->setType(TensorType::create(value));

```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `GetValue`, `debugName`, `value`, `owningGraph`, `create`, `t_`, `...`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`GetValue`, `debugName`, `value`, `owningGraph`, `create`, `t_`, `...`。

### Lines 348-365
```cpp
          // Copy over source location and scope information to all nodes
          // created by the symbolic
          const_node->copyMetadata(node);
          new_block->appendNode(const_node);
          ONNXShapeTypeInference(const_node, empty_params_dict, opset_version);
          auto py_output = py::cast(const_node->output());
          env[py::cast(old)] = py_output;
          values_in_env.add(py_output);
        } else {
          // An update in ConstantValueMap is also needed here, since
          // the user setType can be only accessed in this step, and it
          // should be reliable.
          MergeInferredTypeAndSetMap(
              outputs[i], old->type(), outputs[i]->type());
          // non ONNX node with no type given will throw out the warnings here.
          UpdateReliable(
              outputs[i],
              AreInputsReliableOrStatic(outputs[i]->node()),
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `copyMetadata`, `appendNode`, `ONNXShapeTypeInference`, `cast`, `output`, `add`, `...`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`copyMetadata`, `appendNode`, `ONNXShapeTypeInference`, `cast`, `output`, `add`, `...`。

### Lines 366-375
```cpp
              /*no_type_warning=*/true);
          // For the node type that does not have ComputeConstant logic, it may
          // have reliable shape but its shape is not in ConstantValueMap. So we
          // need to update ConstantValueMap.
          UpdateShapeConstantIfReliable(outputs[i]);

          // Copy over source location and scope information to all nodes
          // created by the symbolic
          // Do not set metadata if outputs[i] is already in env.
          if (!exist_in_env) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `UpdateShapeConstantIfReliable`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`UpdateShapeConstantIfReliable`。

### Lines 376-386
```cpp
            outputs[i]->node()->copyMetadata(node);
          }
          auto py_output = py::cast(outputs[i]);
          env[py::cast(old)] = py_output;
          values_in_env.add(py_output);
        }
      } else {
        // Null output means that the ONNX op doesn't have outputs corresponding
        // to certain PyTorch outputs
        env[py::cast(old)] = py::none();
        if (!old->uses().empty()) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `node`, `copyMetadata`, `cast`, `add`, `none`, `uses`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`node`, `copyMetadata`, `cast`, `add`, `none`, `uses`, `...`。

### Lines 387-397
```cpp
          std::ostringstream ss;
          ss << "symbolic for " << op_name << " returned None for the output "
             << i;
          ss << " (indicating conversion for that particular output is not supported), ";
          ss << "but the network uses this output later";
          // TODO: Say what actually used it
          throw std::runtime_error(ss.str());
        }
      }
    }
  };
```
- EN: This block implements local helper logic for onnx. Key symbols: `runtime_error`, `str`.
- CN: 该代码块实现与 onnx 相关的局部辅助逻辑。关键符号：`runtime_error`, `str`。

### Lines 398-409
```cpp

  // Clone the node and add it to the new graph
  auto cloneNode = [&](Node* node) {
    auto n_ = new_block->appendNode(
        new_block->owningGraph()->createClone(node, envFn));
    for (const auto i : c10::irange(node->outputs().size())) {
      // n_->outputs()[i]->setType(node->outputs()[i]->type());
      auto py_output = py::cast(n_->output(i));
      env[py::cast(node->output(i))] = py_output;
      values_in_env.add(py_output);
    }
  };
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `appendNode`, `owningGraph`, `createClone`, `irange`, `outputs`, `size`, `...`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`appendNode`, `owningGraph`, `createClone`, `irange`, `outputs`, `size`, `...`。

### Lines 410-419
```cpp

  // Inline the prim::PythonOp sub-block nodes and append them to the onnx graph
  auto inlineAutograd = [&](Node* PythonOpNode) {
    for (auto subblock : PythonOpNode->blocks()) {
      for (const auto i : c10::irange(PythonOpNode->inputs().size())) {
        auto py_value = env[py::cast(PythonOpNode->inputs()[i])];
        env[py::cast(subblock->inputs()[i])] = py_value;
        values_in_env.add(py_value);
      }
      for (auto* node : subblock->nodes()) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `blocks`, `irange`, `inputs`, `size`, `cast`, `add`, `...`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`blocks`, `irange`, `inputs`, `size`, `cast`, `add`, `...`。

### Lines 420-428
```cpp
        NodeToONNX(node, new_block, operator_export_type, env, values_in_env);
      }
      for (const auto i : c10::irange(PythonOpNode->outputs().size())) {
        auto py_value = env[py::cast(subblock->outputs()[i])];
        env[py::cast(PythonOpNode->outputs()[i])] = py_value;
        values_in_env.add(py_value);
      }
    }
  };
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `NodeToONNX`, `irange`, `outputs`, `size`, `cast`, `add`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`NodeToONNX`, `irange`, `outputs`, `size`, `cast`, `add`。

### Lines 429-439
```cpp

  // Cast output of symbolic() python implementation
  auto processSymbolicOutput = [&](const std::string& op_name,
                                   Node* n,
                                   const py::object& raw_output) {
    if (Py_IsNone(raw_output.ptr())) {
      cloneNode(n);
      return;
    }
    // Cast the outputs back to C++ and put them in the new graph
    std::vector<Value*> outputs;
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `Py_IsNone`, `ptr`, `cloneNode`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`Py_IsNone`, `ptr`, `cloneNode`。

### Lines 440-454
```cpp
    try {
      if (py::isinstance<Value>(raw_output)) {
        outputs = value_list{py::cast<Value*>(raw_output)};
      } else {
        outputs = py::cast<std::vector<Value*>>(raw_output);
      }
    } catch (const std::exception&) {
      std::ostringstream ss;
      ss << "Error casting results of symbolic for " << op_name
         << ": expected to return list of op nodes, instead received type ''"
         << py::str(py::type::handle_of(raw_output))
         << "': " << py::str(raw_output);
      throw std::runtime_error(ss.str());
    }

```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `str`, `handle_of`, `runtime_error`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`str`, `handle_of`, `runtime_error`。

### Lines 455-464
```cpp
    setOutputs(op_name, n, outputs);
  };

  auto callPySymbolicFunction = [&](Node* n) {
    // The idea is delegate as much of the actual argument massaging to
    // Python as possible

    py::tuple py_inputs(n->inputs().size());
    Py_ssize_t input_nr = 0;
    for (auto* input : n->inputs()) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `setOutputs`, `py_inputs`, `inputs`, `size`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`setOutputs`, `py_inputs`, `inputs`, `size`。

### Lines 465-482
```cpp
      py_inputs[input_nr++] = py::cast(envFn(input));
    }

    Graph* g = new_block->owningGraph();

    WithInsertPoint insert_point_guard(new_block);
    WithCurrentScope scope_guard(*g, n->scope());

    // IMPORTANT: NEVER pass raw pointer of smart pointer managed objects to
    // Python. Check #87343 for details.
    py::list new_nodes = py::list();
    py::object raw_output = onnx_utils.attr("_run_symbolic_function")(
        g->shared_from_this(),
        new_block,
        n,
        py_inputs,
        env,
        values_in_env,
```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `cast`, `envFn`, `owningGraph`, `insert_point_guard`, `scope_guard`, `scope`, `...`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`cast`, `envFn`, `owningGraph`, `insert_point_guard`, `scope_guard`, `scope`, `...`。

### Lines 483-492
```cpp
        new_nodes,
        operator_export_type);

    // Find new nodes that have been created by _run_symbolic_function and
    // propagate metadata
    for (py::handle py_node : new_nodes) {
      Node* node = py_node.cast<Node*>();
      node->copyMetadata(n);
    }

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `copyMetadata`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`copyMetadata`。

### Lines 493-501
```cpp
    // TODO: Assert it's an ATen identifier???
    // (Sometimes it's not...)
    processSymbolicOutput(n->kind().toUnqualString(), n, raw_output);
    GRAPH_DUMP("after processSymbolicOutput: ", g);
  };

  auto callPySymbolicMethod = [&](ConcretePythonOp* op) {
    // Test if there is a symbolic function; bail if there is not
    auto pyobj = py::handle(op->pyobj.get());
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `processSymbolicOutput`, `kind`, `toUnqualString`, `handle`, `get`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`processSymbolicOutput`, `kind`, `toUnqualString`, `handle`, `get`。

### Lines 502-511
```cpp
    auto func = op->autogradFunction();
    if (func) {
      pyobj = func->get();
    }

    py::object opset_version =
        onnx_globals.attr("GLOBALS").attr("export_onnx_opset_version");
    // NOTE(justinchuby): Call the internal registry to register the symbolic
    // method defined in the module.
    bool is_registered_op =
```
- EN: This block handles conditional branches. Key symbols: `autogradFunction`, `get`, `attr`.
- CN: 该代码块处理条件分支。关键符号：`autogradFunction`, `get`, `attr`。

### Lines 512-523
```cpp
        onnx_registration.attr("registry")
            .attr("is_registered_op")("prim::PythonOp", opset_version)
            .cast<bool>();
    py::bool_ is_autograd_inlining_enabled =
        py::cast<bool>(onnx_globals.attr("GLOBALS").attr("autograd_inlining"));
    if (!py::hasattr(pyobj, "symbolic") && !is_registered_op) {
      // Inline the subgraph within the prim::PythonOp unless
      // either of these conditions are satisfied
      // 1. The torch.autograd.Function class of this node object has `symbolic`
      // method defined.
      // 2. Custom export symbolic is registered for prim::PythonOp.
      if ((operator_export_type == ::torch::onnx::OperatorExportTypes::ONNX ||
```
- EN: Declares core types or data containers for this file. Prominent symbols: `attr`, `hasattr`.
- CN: 声明该文件使用的核心类型或数据容器。主要符号：`attr`, `hasattr`。

### Lines 524-541
```cpp
           operator_export_type ==
               ::torch::onnx::OperatorExportTypes::ONNX_ATEN_FALLBACK) &&
          (py::cast<bool>(is_autograd_inlining_enabled))) {
        try {
          inlineAutograd(op);
        } catch (const std::exception& ex) {
          TORCH_WARN(
              "Unable to inline PythonOp: ",
              op->name(),
              " due to the following exception\n",
              ex.what(),
              "prim::PythonOp will be exported as is and without being inlined\n",
              "Try exporting with the following alternatives: \n",
              "1) Set operator_export_type to ONNX_FALLTHROUGH mode\n",
              "2) Register a symbolic method for the prim::PythonOp ",
              op->name());
          cloneNode(op);
        }
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `inlineAutograd`, `name`, `what`, `cloneNode`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`inlineAutograd`, `name`, `what`, `cloneNode`。

### Lines 542-552
```cpp
      } else {
        cloneNode(op);
      }
      return;
    }

    // Prepare args for Python. First one is the graph, and is followed
    // by regular args, with Variables replaced by corresponding nodes.
    Py_ssize_t input_nr = 0;
    py::tuple py_symbolic_args(op->cconv.size());
    auto inputs = op->inputs();
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `cloneNode`, `py_symbolic_args`, `size`, `inputs`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`cloneNode`, `py_symbolic_args`, `size`, `inputs`。

### Lines 553-570
```cpp
    auto node_it = inputs.begin();
    auto scalar_it = op->scalar_args.begin();
    for (auto arg_type : op->cconv) {
      py::object obj;
      if (arg_type == 'c') {
        TORCH_CHECK(
            scalar_it != op->scalar_args.end(),
            "expected too many scalar args");
        obj = py::reinterpret_borrow<py::object>(
            py::handle((scalar_it++)->get()));
      } else if (arg_type == 'd') {
        TORCH_CHECK(node_it != inputs.end(), "expected too many inputs");
        obj = py::cast(envFn(*node_it++));
      } else {
        throw std::runtime_error("unexpected calling convention");
      }
      py_symbolic_args[input_nr++] = obj;
    }
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `begin`, `end`, `handle`, `get`, `cast`, `envFn`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`begin`, `end`, `handle`, `get`, `cast`, `envFn`, `...`。

### Lines 571-588
```cpp

    WithInsertPoint insert_point_guard(new_block);
    WithCurrentScope scope_guard(*new_block->owningGraph(), op->scope());

    if (py::hasattr(pyobj, "symbolic")) {
      // Call the symbolic function
      // Use a little trampoline function so we can give good error messages
      // upon argument mismatch
      // Register as a custom operator
      // TODO: Find a more elegant way to do this without having to touch
      // internal Python modules.
      // TODO(justinchuby): Define a namespace for these Python Ops.
      onnx_registration.attr("registry")
          .attr("register")(
              "::" + op->name(),
              opset_version,
              pyobj.attr("symbolic"),
              /* custom */ true);
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `insert_point_guard`, `scope_guard`, `owningGraph`, `scope`, `hasattr`, `attr`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`insert_point_guard`, `scope_guard`, `owningGraph`, `scope`, `hasattr`, `attr`, `...`。

### Lines 589-597
```cpp

      // IMPORTANT: NEVER pass raw pointer of smart pointer managed objects to
      // Python. Check #87343 for details.
      py::object raw_output = onnx_utils.attr("_run_symbolic_method")(
          new_block->owningGraph()->shared_from_this(),
          op->name(),
          pyobj.attr("symbolic"),
          py_symbolic_args);

```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `attr`, `owningGraph`, `shared_from_this`, `name`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`attr`, `owningGraph`, `shared_from_this`, `name`。

### Lines 598-615
```cpp
      processSymbolicOutput(op->name(), op, raw_output);
    } else {
      TORCH_INTERNAL_ASSERT(is_registered_op);
      Node* n = static_cast<Node*>(op);
      n->s_(attr::name, op->name());
      // Call symbolic function
      // IMPORTANT: NEVER pass raw pointer of smart pointer managed objects to
      // Python. Check #87343 for details.
      py::list new_nodes = py::list();
      py::object raw_output = onnx_utils.attr("_run_symbolic_function")(
          new_block->owningGraph()->shared_from_this(),
          new_block,
          n,
          py_symbolic_args,
          env,
          values_in_env,
          new_nodes,
          operator_export_type);
```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `processSymbolicOutput`, `name`, `s_`, `list`, `attr`, `owningGraph`, `...`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`processSymbolicOutput`, `name`, `s_`, `list`, `attr`, `owningGraph`, `...`。

### Lines 616-631
```cpp

      processSymbolicOutput(op->kind().toUnqualString(), n, raw_output);
    }
  };

  auto k = old_node->kind();
  if (k.is_caffe2()) {
    // Pass on Caffe2 operator, since we already preprocess it
    cloneNode(old_node);
  } else if (k == prim::PythonOp) {
    callPySymbolicMethod(static_cast<ConcretePythonOp*>(old_node));
  } else {
    callPySymbolicFunction(old_node);
  }
}

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `processSymbolicOutput`, `kind`, `toUnqualString`, `is_caffe2`, `cloneNode`, `callPySymbolicMethod`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`processSymbolicOutput`, `kind`, `toUnqualString`, `is_caffe2`, `cloneNode`, `callPySymbolicMethod`, `...`。

### Lines 632-632
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
- EN: Type and shape reasoning — the code propagates static information across the IR.
  CN: 类型与形状推理——代码会在 IR 中传播静态信息。
- EN: Export interop — the implementation helps bridge TorchScript graphs to ONNX-style representations.
  CN: 导出互操作——实现帮助 TorchScript 图与 ONNX 风格表示之间建立桥接。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `torch/csrc/jit/passes/onnx.h`, `c10/util/Exception.h`, `c10/util/irange.h`, `torch/csrc/autograd/symbolic.h`, `torch/csrc/jit/ir/constants.h`, `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/dead_code_elimination.h`, `torch/csrc/jit/passes/onnx/constant_map.h`, `torch/csrc/jit/passes/onnx/helper.h`, `torch/csrc/jit/passes/onnx/onnx_log.h`, `...`
- External includes / 外部头文件: `sstream`
- Namespaces / 命名空间: `torch::jit`
- Representative symbols / 代表性符号: `removePrintOps`, `nodes`, `begin`, `end`, `blocks`, `kind`, `inputs`, `size`, `uses`, `node`, `...`
