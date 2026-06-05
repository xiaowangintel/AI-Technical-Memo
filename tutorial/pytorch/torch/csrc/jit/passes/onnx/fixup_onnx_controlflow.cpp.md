# fixup_onnx_controlflow.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/passes/onnx/fixup_onnx_controlflow.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements the TorchScript JIT pass logic for fixup onnx controlflow, including graph analysis and rewrites.
- 用途 (CN): 实现与 fixup onnx controlflow 相关的 TorchScript JIT pass 逻辑，包括图分析与改写。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9
```cpp
#include <torch/csrc/jit/passes/onnx/fixup_onnx_controlflow.h>

#include <ATen/InitialTensorOptions.h>
#include <c10/util/irange.h>
#include <torch/csrc/jit/jit_log.h>
#include <torch/csrc/jit/passes/onnx/helper.h>
#include <torch/csrc/jit/passes/onnx/shape_type_inference.h>

namespace torch::jit {
```
- EN: This block implements local helper logic for fixup onnx controlflow. Key symbols: no dominant local symbol names.
- CN: 该代码块实现与 fixup onnx controlflow 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 10-20
```cpp

namespace {
const int ONNX_OPSET_13 = 13;
const int ONNX_TYPE_BOOL = 9;

Node* CreateCastToBoolNode(Value* val, Graph* graph) {
  Node* cast_node = graph->create(c10::onnx::Cast);
  cast_node->addInput(val);
  cast_node->i_(attr::to, ONNX_TYPE_BOOL);
  cast_node->output()->setType(BoolType::get());
  return cast_node;
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `CreateCastToBoolNode`, `create`, `addInput`, `i_`, `output`, `setType`, `...`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`CreateCastToBoolNode`, `create`, `addInput`, `i_`, `output`, `setType`, `...`。

### Lines 21-33
```cpp
}

Node* InsertCastForCond(
    Value* cond_val,
    Graph* graph,
    Node* consumer_node,
    int opset_version) {
  // prev:  cond_val -> consumer_node
  // after: cond_val -> cast -> consumer_node
  // NOTE: The cast is required because operators like PyTorch Greater/Less
  //       return tensor in type torch.uint8. However the type for condition
  //       input in ONNX Loop must be bool.
  Node* cast_node = CreateCastToBoolNode(cond_val, graph);
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `InsertCastForCond`, `CreateCastToBoolNode`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`InsertCastForCond`, `CreateCastToBoolNode`。

### Lines 34-42
```cpp
  cast_node->insertBefore(consumer_node);

  consumer_node->replaceInputWith(cond_val, cast_node->output());
  const ParamMap empty_params_dict = {};
  ONNXShapeTypeInference(cast_node, empty_params_dict, opset_version);
  return cast_node;
}

bool IsCondCastRequired(Value* cond_val) {
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `insertBefore`, `replaceInputWith`, `output`, `ONNXShapeTypeInference`, `IsCondCastRequired`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`insertBefore`, `replaceInputWith`, `output`, `ONNXShapeTypeInference`, `IsCondCastRequired`。

### Lines 43-51
```cpp
  const auto& type = cond_val->type();
  if (auto tt = type->cast<TensorType>()) {
    if (auto scalar_type = tt->scalarType()) {
      return *scalar_type != c10::kBool;
    }
  }
  return !type->isSubtypeOf(*BoolType::get());
}

```
- EN: This block handles conditional branches; produces a result or forwards a computed value. Key symbols: `type`, `scalarType`, `isSubtypeOf`, `get`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值。关键符号：`type`, `scalarType`, `isSubtypeOf`, `get`。

### Lines 52-61
```cpp
bool IsErasableSequence(const Node* loop_node, size_t i) {
  TORCH_INTERNAL_ASSERT(loop_node->blocks().size() == 1);
  auto* sub_block = loop_node->blocks()[0];
  auto* seq_node = sub_block->outputs()[i - 1]->node();
  auto* in_val = sub_block->inputs()[i];

  if (seq_node->kind() != ::c10::onnx::SequenceInsert) {
    return false;
  }

```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `IsErasableSequence`, `blocks`, `size`, `outputs`, `node`, `inputs`, `...`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`IsErasableSequence`, `blocks`, `size`, `outputs`, `node`, `inputs`, `...`。

### Lines 62-71
```cpp
  if (seq_node->inputs().size() == 3) {
    // Non-default insert position is not supported.
    return false;
  }

  if (seq_node->input(0) != in_val) {
    // Only SequenceInsert that applies on loop-carried sequence is supported.
    return false;
  }

```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `inputs`, `size`, `input`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`inputs`, `size`, `input`。

### Lines 72-80
```cpp
  const auto* init_seq_node = loop_node->inputs()[i]->node();
  const auto init_seq_node_kind = init_seq_node->kind();
  if ((init_seq_node_kind != ::c10::onnx::SequenceEmpty) &&
      (init_seq_node_kind != ::c10::prim::ListConstruct ||
       !init_seq_node->inputs().empty())) {
    // Initial sequence must be empty.
    return false;
  }

```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `inputs`, `node`, `kind`, `empty`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`inputs`, `node`, `kind`, `empty`。

### Lines 81-98
```cpp
  if (seq_node->output()->uses().size() != 1) {
    // The sequence is not supported to be used elsewhere inside the sub-block.
    return false;
  }

  return true;
}

// ONNX::Loop does not support Sequence type as loop-carried dependencies. Only
// tensors are supported. This pass converts Sequence loop-carried dependencies
// to scan_outputs. In opset 11, only the below pattern is supported.
//
// PTIR graph:
//  ...
//  %res.1 : Tensor[] = prim::ListConstruct()
//  %res : Tensor[] = prim::Loop(%11, %22, %res.1)
//    block0(%i.1 : Tensor, %res.6 : Tensor[]):
//      ...
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `output`, `uses`, `size`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`output`, `uses`, `size`。

### Lines 99-111
```cpp
//      %res.3 : Tensor[] = aten::append(%res.6, %17)
//      -> (%22, %res.3)
//  return (%res.3)
//
// ONNX graph:
//  ...
//  %res : Tensor = onnx::Loop(%11, %22)
//    block0(%i.1 : Tensor):
//      ...
//      -> (%22, %17)
//  %res_seq : Tensor[] = onnx::SplitToSequence[keepdims=0](%res)
//  return (%res_seq)
std::vector<Value*> ConvertSequenceDependencies(Node* node, int opset_version) {
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `ConvertSequenceDependencies`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`ConvertSequenceDependencies`。

### Lines 112-121
```cpp
  if (node->kind() != ::c10::onnx::Loop) {
    return node->outputs().vec();
  }

  if (opset_version >= ONNX_OPSET_13) {
    // Sequence type as loop-carried dependencies should be supported by ONNX
    // ospet 13.
    return node->outputs().vec();
  }

```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `kind`, `outputs`, `vec`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`kind`, `outputs`, `vec`。

### Lines 122-134
```cpp
  auto* loop_node = node;

  TORCH_INTERNAL_ASSERT(loop_node->blocks().size() == 1);
  auto* sub_block = loop_node->blocks()[0];

  std::vector<size_t> idx_to_remove;
  std::vector<Value*> new_outputs;
  // ONNX Loop node:
  // sub-block inputs are  (iter, cond, loop-carried dependencies)
  // sub-block outputs are (      cond, loop-carried dependencies, scan outputs)
  // inputs are            (iter, cond, loop-carried dependencies)
  // outputs are           (            loop-carried dependencies, scan outputs)
  for (size_t i = 2; i < sub_block->inputs().size(); ++i) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `blocks`, `size`, `inputs`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`blocks`, `size`, `inputs`。

### Lines 135-143
```cpp
    if (IsErasableSequence(loop_node, i)) {
      auto* seq_node = sub_block->outputs()[i - 1]->node();
      // Replace sequence output with the inserted element.
      auto inserted_value = seq_node->input(1);
      sub_block->return_node()->replaceInputWith(
          seq_node->output(), inserted_value);

      // Split the added scan_output back to expected tensor sequence.
      auto loop_output = loop_node->output(i - 2);
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `IsErasableSequence`, `outputs`, `node`, `input`, `return_node`, `replaceInputWith`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`IsErasableSequence`, `outputs`, `node`, `input`, `return_node`, `replaceInputWith`, `...`。

### Lines 144-152
```cpp
      Node* split_node =
          loop_node->owningGraph()->create(c10::onnx::SplitToSequence);
      loop_output->replaceAllUsesWith(split_node->output());
      split_node->i_(attr::keepdims, 0);
      split_node->addInput(loop_output);
      split_node->insertAfter(loop_node);
      split_node->output()->setType(loop_output->type());
      split_node->copyMetadata(loop_node);

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `owningGraph`, `create`, `replaceAllUsesWith`, `output`, `i_`, `addInput`, `...`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`owningGraph`, `create`, `replaceAllUsesWith`, `output`, `i_`, `addInput`, `...`。

### Lines 153-165
```cpp
      // Update loop output type.
      loop_output->setType(c10::unshapedType(inserted_value->type()));

      // The node that produces sequence should be safe to remove now.
      seq_node->destroy();

      idx_to_remove.push_back(i);
      new_outputs.push_back(split_node->output());
    } else {
      new_outputs.push_back(loop_node->output(i - 2));
    }
  }

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `setType`, `unshapedType`, `type`, `destroy`, `push_back`, `output`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`setType`, `unshapedType`, `type`, `destroy`, `push_back`, `output`。

### Lines 166-177
```cpp
  // Remove sequence outputs, and replace with scan outputs.
  for (const auto i : c10::irange(idx_to_remove.size())) {
    size_t idx = idx_to_remove[i] - i;

    sub_block->eraseInput(idx);
    loop_node->removeInput(idx);

    // Swap output order. Move all scan outputs to the back.
    sub_block->return_node()->addInput(
        sub_block->return_node()->inputs().at(idx - 1));
    sub_block->return_node()->removeInput(idx - 1);

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `irange`, `size`, `eraseInput`, `removeInput`, `return_node`, `addInput`, `...`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`irange`, `size`, `eraseInput`, `removeInput`, `return_node`, `addInput`, `...`。

### Lines 178-186
```cpp
    auto loop_out = loop_node->addOutput();
    loop_out->copyMetadata(loop_node->outputs().at(idx - 2));
    loop_node->outputs().at(idx - 2)->replaceAllUsesWith(loop_out);
    loop_node->eraseOutput(idx - 2);
  }

  return new_outputs;
}

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `addOutput`, `copyMetadata`, `outputs`, `replaceAllUsesWith`, `eraseOutput`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`addOutput`, `copyMetadata`, `outputs`, `replaceAllUsesWith`, `eraseOutput`。

### Lines 187-195
```cpp
Node* ONNXOptionalNode(const OptionalTypePtr& opt_type, Graph* g) {
  TORCH_INTERNAL_ASSERT(opt_type);
  TypePtr elem_type = opt_type->getElementType();
  Node* opt_node = g->create(::c10::onnx::Optional, 1);
  opt_node->ty_(Symbol::attr("type"), elem_type);
  opt_node->output()->setType(OptionalType::create(elem_type));
  return opt_node;
}

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `ONNXOptionalNode`, `getElementType`, `create`, `ty_`, `attr`, `output`, `...`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`ONNXOptionalNode`, `getElementType`, `create`, `ty_`, `attr`, `output`, `...`。

### Lines 196-207
```cpp
// Replaces block output i with an onnx::Optional
// with `type` taken from opt_type. If and Loop Ops shares this function.
// 1. If Op: Needed when control flow has multiple branches, one of which
// is defined by `block` and returns a None and another branch
// returns not-None. The passed-in opt_type should be from the other branch.
// 2. Loop Op: insert Optional node before output, if input is Optional type
// or output type is None.
void ReplaceBlockOutputWithOptional(
    const OptionalTypePtr& opt_type,
    Block* block,
    size_t i) {
  Node* opt_node = ONNXOptionalNode(opt_type, block->owningGraph());
```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `ReplaceBlockOutputWithOptional`, `ONNXOptionalNode`, `owningGraph`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`ReplaceBlockOutputWithOptional`, `ONNXOptionalNode`, `owningGraph`。

### Lines 208-218
```cpp
  opt_node->insertBefore(block->return_node());
  Value* block_output = block->outputs().at(i);
  // replace only the last value as Optional type only affects
  // the value right before output
  block_output->replaceAllUsesAfterNodeWith(opt_node, opt_node->output());
  if (!block_output->type()->cast<NoneType>()) {
    opt_node->addInput(block_output);
    opt_node->copyMetadata(block_output->node());
  }
}

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `insertBefore`, `return_node`, `outputs`, `replaceAllUsesAfterNodeWith`, `output`, `type`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`insertBefore`, `return_node`, `outputs`, `replaceAllUsesAfterNodeWith`, `output`, `type`, `...`。

### Lines 219-230
```cpp
// Resolving limitation from ONNX that the block output can not be
// a value from outside the block. Inserting an Identity node inside
// the block, linking with the value outside as workaround.
void FixupONNXSubblockOutputs(Node* n) {
  for (Block* block : n->blocks()) {
    for (Value* output : block->outputs()) {
      if (output->node()->owningBlock() != block) {
        Node* id_node = nullptr;
        // Simplify graph by creating an empty optional rather than
        // Identity(None). Also enables shape inference later on, since
        // ONNX shape inference doesn't handle None.
        if (output->type()->cast<NoneType>()) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `FixupONNXSubblockOutputs`, `blocks`, `outputs`, `node`, `owningBlock`, `type`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`FixupONNXSubblockOutputs`, `blocks`, `outputs`, `node`, `owningBlock`, `type`。

### Lines 231-244
```cpp
          id_node = block->owningGraph()->create(c10::onnx::Optional);
        } else {
          id_node = block->owningGraph()->create(c10::onnx::Identity);
          id_node->addInput(output);
        }
        id_node->insertBefore(block->return_node());
        id_node->output()->copyMetadata(output);
        id_node->copyMetadata(n);
        block->return_node()->replaceInputWith(output, id_node->output());
      }
    }
  }
}

```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `owningGraph`, `create`, `addInput`, `insertBefore`, `return_node`, `output`, `...`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`owningGraph`, `create`, `addInput`, `insertBefore`, `return_node`, `output`, `...`。

### Lines 245-253
```cpp
// Infer type of optional inputs from outputs.
void FixupONNXLoopBlockInputs(Node* n) {
  for (Block* block : n->blocks()) {
    for (const auto i : c10::irange(1, block->inputs().size())) {
      // input i corresponds to output i until we run FixupONNXLoopNodeInputs.
      Value* input_i = block->inputs().at(i);
      if (input_i->type()->cast<OptionalType>() &&
          !block->outputs().at(i)->type()->cast<OptionalType>()) {
        auto [merged_type, inferred] = MergeInferredType(
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `FixupONNXLoopBlockInputs`, `blocks`, `irange`, `inputs`, `size`, `type`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`FixupONNXLoopBlockInputs`, `blocks`, `irange`, `inputs`, `size`, `type`, `...`。

### Lines 254-263
```cpp
            input_i->type()->cast<OptionalType>()->getElementType(),
            block->outputs().at(i)->type());
        if (inferred) {
          input_i->setType(OptionalType::create(merged_type));
        }
      }
    }
  }
}

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `type`, `getElementType`, `outputs`, `setType`, `create`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`type`, `getElementType`, `outputs`, `setType`, `create`。

### Lines 264-272
```cpp
// Replace None in outputs with Optional.
void FixupONNXLoopBlockOutputs(Node* n) {
  for (Block* block : n->blocks()) {
    // output 0 is continue_condition, never None.
    for (const auto i : c10::irange(1, block->outputs().size())) {
      // Two conditions that we need to replace block output with optional
      // 1. output is NoneType
      // 2. input is optional but output type is not
      if ((block->outputs().at(i)->type()->cast<NoneType>()) ||
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `FixupONNXLoopBlockOutputs`, `blocks`, `irange`, `outputs`, `size`, `type`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`FixupONNXLoopBlockOutputs`, `blocks`, `irange`, `outputs`, `size`, `type`。

### Lines 273-287
```cpp
          (block->inputs().at(i + 1)->type()->cast<OptionalType>() &&
           !block->outputs().at(i)->type()->cast<OptionalType>())) {
        ReplaceBlockOutputWithOptional(
            // Output 0 is continue_condition.
            // Inputs (0, 1) are (loop_counter, cond). So input i + 1
            // corresponds to output i.
            block->inputs().at(i + 1)->type()->cast<OptionalType>(),
            block,
            i);
      }
    }
  }
  FixupONNXSubblockOutputs(n);
}

```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `inputs`, `type`, `outputs`, `ReplaceBlockOutputWithOptional`, `FixupONNXSubblockOutputs`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`inputs`, `type`, `outputs`, `ReplaceBlockOutputWithOptional`, `FixupONNXSubblockOutputs`。

### Lines 288-296
```cpp
void FixupONNXLoopNodeInputs(Node* node, int opset_version) {
  if (node->kind() != ::c10::onnx::Loop) {
    return;
  }

  auto* graph = node->owningGraph();

  // add cast to condition input outside the loop.
  Value* cond_val = node->input(1);
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `FixupONNXLoopNodeInputs`, `kind`, `owningGraph`, `input`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`FixupONNXLoopNodeInputs`, `kind`, `owningGraph`, `input`。

### Lines 297-305
```cpp
  if (IsCondCastRequired(cond_val)) {
    auto* cast_node = InsertCastForCond(cond_val, graph, node, opset_version);
    cast_node->copyMetadata(node);
  }

  // Setup Loop input cond and i.
  TORCH_INTERNAL_ASSERT(node->blocks().size() == 1);
  auto* sub_block = node->blocks().at(0);
  Value* cond = sub_block->insertInput(1, "cond");
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `IsCondCastRequired`, `InsertCastForCond`, `copyMetadata`, `blocks`, `size`, `insertInput`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`IsCondCastRequired`, `InsertCastForCond`, `copyMetadata`, `blocks`, `size`, `insertInput`。

### Lines 306-318
```cpp
  cond->setType(BoolType::get());

  Value* i = sub_block->inputs().at(0);
  i->setType(TensorType::fromNumberType(*IntType::get()));

  // add cast to condition input inside the loop.
  Value* next_cond_val = sub_block->outputs().at(0);
  if (IsCondCastRequired(next_cond_val)) {
    auto* cast_node = InsertCastForCond(
        next_cond_val, graph, sub_block->return_node(), opset_version);
    cast_node->copyMetadata(node);
  }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `setType`, `get`, `inputs`, `fromNumberType`, `outputs`, `IsCondCastRequired`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`setType`, `get`, `inputs`, `fromNumberType`, `outputs`, `IsCondCastRequired`, `...`。

### Lines 319-328
```cpp
  // Inputs (0, 1) are (max_trip_count, start_condition). Skip them
  // since they're never None or Optional.
  for (const auto i : c10::irange(2, node->inputs().size())) {
    Value* input = node->inputs().at(i);
    OptionalTypePtr sub_block_input_optional =
        sub_block->inputs().at(i)->type()->cast<OptionalType>();
    // If loop input is not optional but block input is, wrap loop input with
    // Optional. Happens when the loop takes in None and outputs not-None, or
    // vice-versa.
    if (!input->type()->cast<OptionalType>() && sub_block_input_optional) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `irange`, `inputs`, `size`, `type`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`irange`, `inputs`, `size`, `type`。

### Lines 329-337
```cpp
      if (!input->type()->cast<NoneType>()) {
        auto [merged_type, inferred] = MergeInferredType(
            sub_block_input_optional->getElementType(), input->type());
        if (inferred) {
          sub_block_input_optional = OptionalType::create(merged_type);
          sub_block->inputs().at(i)->setType(sub_block_input_optional);
        }
      }
      Node* opt_node = ONNXOptionalNode(sub_block_input_optional, graph);
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `type`, `MergeInferredType`, `getElementType`, `create`, `inputs`, `setType`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`type`, `MergeInferredType`, `getElementType`, `create`, `inputs`, `setType`, `...`。

### Lines 338-347
```cpp
      if (!input->type()->cast<NoneType>()) {
        opt_node->addInput(input);
      }
      opt_node->insertBefore(node);
      node->replaceInputWith(input, opt_node->output());
    }
  }
}
} // anonymous namespace

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `type`, `addInput`, `insertBefore`, `replaceInputWith`, `output`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`type`, `addInput`, `insertBefore`, `replaceInputWith`, `output`。

### Lines 348-359
```cpp
static std::vector<Value*> FixupONNXLoopNode(Node* node, int opset_version) {
  auto output_size = node->outputs().size();
  GRAPH_DEBUG("before FixupONNXLoopBlockInputs: ", *node->owningGraph());
  FixupONNXLoopBlockInputs(node);
  GRAPH_DEBUG("after FixupONNXLoopBlockInputs: ", *node->owningGraph());
  FixupONNXLoopNodeInputs(node, opset_version);
  GRAPH_DEBUG("after FixupONNXLoopNodeInputs: ", *node->owningGraph());
  FixupONNXLoopBlockOutputs(node);
  GRAPH_DEBUG("after FixupONNXLoopBlockOutputs: ", *node->owningGraph());
  // NOTE: the output order is deliberately changed to match expected order
  //       since onnx loop requires scan outputs to be the last outputs.
  auto new_outputs = ConvertSequenceDependencies(node, opset_version);
```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `FixupONNXLoopNode`, `outputs`, `size`, `owningGraph`, `FixupONNXLoopBlockInputs`, `FixupONNXLoopNodeInputs`, `...`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`FixupONNXLoopNode`, `outputs`, `size`, `owningGraph`, `FixupONNXLoopBlockInputs`, `FixupONNXLoopNodeInputs`, `...`。

### Lines 360-369
```cpp
  // Copy type of block output to node output.
  FixupONNXControlflowNodeOutputs(node);
  GRAPH_DEBUG("after FixupONNXControlflowNodeOutputs: ", *node->owningGraph());
  TORCH_INTERNAL_ASSERT(output_size == new_outputs.size());
  return new_outputs;
}

// Check if node is prim::Uninitialized,
// or output of prim::Uninitialized->onnx::Identity
static bool IsUninitializedNode(Node* n) {
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `FixupONNXControlflowNodeOutputs`, `owningGraph`, `size`, `IsUninitializedNode`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`FixupONNXControlflowNodeOutputs`, `owningGraph`, `size`, `IsUninitializedNode`。

### Lines 370-381
```cpp
  if (n->kind() == ::c10::onnx::Identity &&
      n->inputs()[0]->node()->kind() == prim::Uninitialized)
    return true;
  if (n->kind() == prim::Uninitialized)
    return true;
  return false;
}

// Infer shape and type of the uninitialized_output from the corresponding
// output of the other subblock. prim::Uninitialized node is proven to be
// unused. So replace this node with one of the inferred shape and type.
static void InferShapeTypeForUninitializedOutput(
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `kind`, `inputs`, `node`, `InferShapeTypeForUninitializedOutput`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`kind`, `inputs`, `node`, `InferShapeTypeForUninitializedOutput`。

### Lines 382-392
```cpp
    Graph* graph,
    Block* block,
    Value* uninitialized_output,
    Value* other_output,
    int opset_version) {
  Node* const_node = nullptr;
  if (auto output_type = other_output->type()->cast<TensorType>()) {
    auto elem_type =
        at::initialTensorOptions().dtype(output_type->scalarType());
    const_node = graph->create(::c10::onnx::Constant, 1);

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `type`, `initialTensorOptions`, `dtype`, `scalarType`, `create`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`type`, `initialTensorOptions`, `dtype`, `scalarType`, `create`。

### Lines 393-405
```cpp
    if (output_type->sizes().concrete_sizes().has_value()) {
      auto size = output_type->sizes().concrete_sizes().value();
      const_node->t_(attr::value, at::zeros(size, elem_type));
      const_node->output()->setType(other_output->type());
    } else {
      const_node->t_(attr::value, at::zeros({}, elem_type));
      const_node->output()->setType(
          TensorType::create(output_type->scalarType(), at::kCPU, {}, {}));
    }
  } else if (auto output_type = other_output->type()->cast<ListType>()) {
    TypePtr elem = output_type->getElementType();
    const_node = graph->create(::c10::onnx::SequenceEmpty, 1);
    if (elem->cast<TensorType>() &&
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `sizes`, `concrete_sizes`, `has_value`, `value`, `t_`, `zeros`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`sizes`, `concrete_sizes`, `has_value`, `value`, `t_`, `zeros`, `...`。

### Lines 406-423
```cpp
        elem->cast<TensorType>()->scalarType().has_value()) {
      auto scalar_type = elem->cast<TensorType>()->scalarType().value();
      auto onnx_type = ATenTypeToOnnxType(scalar_type);
      const_node->i_(attr::dtype, onnx_type);
      const_node->output()->setType(other_output->type());
    } else if (elem->cast<IntType>()) {
      auto scalar_type = at::kLong;
      auto onnx_type = ATenTypeToOnnxType(scalar_type);
      const_node->i_(attr::dtype, onnx_type);
      const_node->output()->setType(other_output->type());
    } else {
      TORCH_WARN(
          "UninitializedOutput - Invalid elem Type of ListTensor found.");
      const_node->output()->setType(other_output->type());
    }
  } else if (auto output_type = other_output->type()->cast<OptionalType>()) {
    const_node = ONNXOptionalNode(output_type, graph);
  }
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `scalarType`, `has_value`, `value`, `ATenTypeToOnnxType`, `i_`, `output`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`scalarType`, `has_value`, `value`, `ATenTypeToOnnxType`, `i_`, `output`, `...`。

### Lines 424-435
```cpp
  TORCH_CHECK(
      const_node,
      "Inferring type for prim::Uninitialized node from " +
          other_output->type()->repr_str() + " not supported.")
  const ParamMap empty_params_dict = {};
  ONNXShapeTypeInference(const_node, empty_params_dict, opset_version);
  const_node->insertBefore(block->return_node());
  const_node->copyMetadata(block->return_node());
  uninitialized_output->replaceAllUsesWith(const_node->output());
  uninitialized_output->node()->destroy();
}

```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `type`, `repr_str`, `ONNXShapeTypeInference`, `insertBefore`, `return_node`, `copyMetadata`, `...`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`type`, `repr_str`, `ONNXShapeTypeInference`, `insertBefore`, `return_node`, `copyMetadata`, `...`。

### Lines 436-453
```cpp
// Corresponding outputs for ONNX If then and else subblocks should have
// same shape and type. This pass detects if prim::Uninitialized node
// appears as part of outputs of either of the subblocks, and infers
// shape and type from the corresponding output of the other subblock
// In the example graph below, shape and type of the subblock output %7
// for subblock 1 is inferred from %y.1. Shape and type of Subblock
// output %7 is inferred from %y.5.
//
// graph(%y.1 : Int(3:4, 4:1, requires_grad=0, device=cpu)):
//   ...
//   %7 : Tensor = prim::Uninitialized()
//   %16 : bool, %17 : Tensor, %y.14 : Tensor = prim::If(%15) #
//   test/onnx/test_pytorch_onnx_onnxruntime.py:614:20
//     block0():
//       %y.5 : Tensor = aten::add(%y.1, %3, %6) #
//       test/onnx/test_pytorch_onnx_onnxruntime.py:615:28
//       -> (%2, %7, %y.5)
//     block1():
```
- EN: Documents the intent, constraints, or compatibility notes that guide the surrounding implementation.
- CN: 说明周围实现所遵循的目标、限制条件或兼容性要求。

### Lines 454-465
```cpp
//       -> (%1, %y.1, %7)
//   ...

static void ONNXFixupUninitializedOutput(Node* node, int opset_version) {
  if (node->kind() != ::c10::onnx::If) {
    return;
  }

  GRAPH_DUMP("Graph before fixing If shape type: ", node->owningGraph());
  auto* if_node = node;
  auto* graph = if_node->owningGraph();

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `ONNXFixupUninitializedOutput`, `kind`, `owningGraph`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`ONNXFixupUninitializedOutput`, `kind`, `owningGraph`。

### Lines 466-476
```cpp
  // Check if the input to ONNX If node is node Bool, and insert
  // cast to Bool if needed.
  if (!if_node->input()->type()->isSubtypeOf(*BoolType::get())) {
    Node* cast_node =
        InsertCastForCond(if_node->input(), graph, if_node, opset_version);
    cast_node->copyMetadata(if_node);
  }

  Block* then_block = if_node->blocks()[0];
  Block* else_block = if_node->blocks()[1];

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `input`, `type`, `isSubtypeOf`, `get`, `InsertCastForCond`, `copyMetadata`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`input`, `type`, `isSubtypeOf`, `get`, `InsertCastForCond`, `copyMetadata`, `...`。

### Lines 477-490
```cpp
  // Infer shape and type for subblock outputs
  TORCH_INTERNAL_ASSERT(
      then_block->outputs().size() == else_block->outputs().size())
  for (const auto i : c10::irange(else_block->outputs().size())) {
    Value* then_block_output = then_block->outputs()[i];
    Value* else_block_output = else_block->outputs()[i];

    // If both subblocks have an uninitialized output, shape and type cannot
    // be inferred.
    TORCH_CHECK(
        !(IsUninitializedNode(then_block_output->node()) &&
          IsUninitializedNode(else_block_output->node())),
        "Cannot infer shape and type for ONNX If with uninitialized output in both subblocks. Please check the model graph.");

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `outputs`, `size`, `irange`, `IsUninitializedNode`, `node`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`outputs`, `size`, `irange`, `IsUninitializedNode`, `node`。

### Lines 491-508
```cpp
    if (IsUninitializedNode(then_block_output->node())) {
      InferShapeTypeForUninitializedOutput(
          graph,
          then_block,
          then_block_output,
          else_block_output,
          opset_version);
      if_node->outputs()[i]->setType(then_block->outputs()[i]->type());
    } else if (IsUninitializedNode(else_block_output->node())) {
      InferShapeTypeForUninitializedOutput(
          graph,
          else_block,
          else_block_output,
          then_block_output,
          opset_version);
      if_node->outputs()[i]->setType(else_block->outputs()[i]->type());
    }
  }
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `IsUninitializedNode`, `node`, `InferShapeTypeForUninitializedOutput`, `outputs`, `setType`, `type`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`IsUninitializedNode`, `node`, `InferShapeTypeForUninitializedOutput`, `outputs`, `setType`, `type`。

### Lines 509-518
```cpp
}

static void ONNXMergeIfBlockOutputShapes(Node* node) {
  TORCH_INTERNAL_ASSERT(node->kind() == ::c10::onnx::If);
  Block* then_block = node->blocks().at(0);
  Block* else_block = node->blocks().at(1);

  TORCH_INTERNAL_ASSERT(
      then_block->outputs().size() == else_block->outputs().size())

```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `ONNXMergeIfBlockOutputShapes`, `kind`, `blocks`, `outputs`, `size`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`ONNXMergeIfBlockOutputShapes`, `kind`, `blocks`, `outputs`, `size`。

### Lines 519-531
```cpp
  auto findCommonShape =
      [](const ::c10::SymbolicShape& a,
         const ::c10::SymbolicShape& b) -> ::c10::SymbolicShape {
    std::vector<::c10::ShapeSymbol> dims;
    if (a.rank() && b.rank() && a.rank() == b.rank()) {
      for (const auto j : c10::irange(a.rank().value())) {
        if (a[j] == b[j]) {
          dims.emplace_back(a[j]);
        } else {
          dims.emplace_back(::c10::ShapeSymbol::newSymbol());
        }
      }
      return ::c10::SymbolicShape(dims);
```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `rank`, `irange`, `value`, `emplace_back`, `newSymbol`, `SymbolicShape`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`rank`, `irange`, `value`, `emplace_back`, `newSymbol`, `SymbolicShape`。

### Lines 532-540
```cpp
    }
    if (a.rank() && a.rank().value() > 0) {
      return a;
    }
    if (b.rank() && b.rank().value() > 0) {
      return b;
    }

    return ::c10::SymbolicShape();
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `rank`, `value`, `SymbolicShape`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`rank`, `value`, `SymbolicShape`。

### Lines 541-549
```cpp
  };

  auto mergeTensorType =
      [&findCommonShape](TensorTypePtr a, TensorTypePtr b) -> TensorTypePtr {
    if (a && b) {
      const auto& a_shape = a->symbolic_sizes();
      const auto& b_shape = b->symbolic_sizes();
      auto commonShape = findCommonShape(a_shape, b_shape);
      return a->withSymbolicShapes(commonShape);
```
- EN: This block handles conditional branches; produces a result or forwards a computed value. Key symbols: `symbolic_sizes`, `findCommonShape`, `withSymbolicShapes`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值。关键符号：`symbolic_sizes`, `findCommonShape`, `withSymbolicShapes`。

### Lines 550-558
```cpp
    } else if (a) {
      return a;
    } else if (b) {
      return b;
    }
    return nullptr;
  };

  auto mergeListType = [&mergeTensorType](
```
- EN: This block handles conditional branches; produces a result or forwards a computed value. Key symbols: no dominant local symbol names.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值。关键符号：无明显局部符号。

### Lines 559-568
```cpp
                           ListTypePtr a, ListTypePtr b) -> ListTypePtr {
    if (a && b) {
      auto a_tensor_type = a->getElementType()->cast<TensorType>();
      auto b_tensor_type = b->getElementType()->cast<TensorType>();
      auto tensor_type = mergeTensorType(a_tensor_type, b_tensor_type);
      if (tensor_type) {
        return a->withContained({tensor_type})->cast<ListType>();
      }
      // Both branches produce ListType without tensor shape.
      return a;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value. Key symbols: `getElementType`, `mergeTensorType`, `withContained`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值。关键符号：`getElementType`, `mergeTensorType`, `withContained`。

### Lines 569-577
```cpp
    } else if (a) {
      return a;
    } else if (b) {
      return b;
    }
    return nullptr;
  };

  auto mergeOptionalType = [&mergeTensorType, &mergeListType](
```
- EN: This block handles conditional branches; produces a result or forwards a computed value. Key symbols: no dominant local symbol names.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值。关键符号：无明显局部符号。

### Lines 578-586
```cpp
                               OptionalTypePtr a,
                               OptionalTypePtr b) -> OptionalTypePtr {
    if (a && b) {
      if (a->getElementType()->cast<TensorType>()) {
        auto a_tensor_type = a->getElementType()->cast<TensorType>();
        auto b_tensor_type = b->getElementType()->cast<TensorType>();
        auto tensor_type = mergeTensorType(a_tensor_type, b_tensor_type);
        if (tensor_type) {
          return a->withContained({tensor_type})->cast<OptionalType>();
```
- EN: This block handles conditional branches; produces a result or forwards a computed value. Key symbols: `getElementType`, `mergeTensorType`, `withContained`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值。关键符号：`getElementType`, `mergeTensorType`, `withContained`。

### Lines 587-595
```cpp
        }
        // Both branches produce OptionalType without tensor shape.
        return a;
      } else if (a->getElementType()->cast<ListType>()) {
        auto a_list_type = a->getElementType()->cast<ListType>();
        auto b_list_type = b->getElementType()->cast<ListType>();
        auto list_type = mergeListType(a_list_type, b_list_type);
        if (list_type) {
          return a->withContained({list_type})->cast<OptionalType>();
```
- EN: This block handles conditional branches; produces a result or forwards a computed value. Key symbols: `getElementType`, `mergeListType`, `withContained`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值。关键符号：`getElementType`, `mergeListType`, `withContained`。

### Lines 596-605
```cpp
        }
        // Both branches produce OptionalType without tensor shape.
        return a;
      }
    } else if (a) {
      return a;
    } else if (b) {
      return b;
    }
    return nullptr;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value. Key symbols: no dominant local symbol names.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值。关键符号：无明显局部符号。

### Lines 606-614
```cpp
  };

  for (const auto i : c10::irange(else_block->outputs().size())) {
    Value* output_i = node->output(i);
    auto then_type = then_block->outputs().at(i)->type();
    auto else_type = else_block->outputs().at(i)->type();
    auto then_tensor_type = then_type->cast<TensorType>();
    auto else_tensor_type = else_type->cast<TensorType>();
    auto then_list_type = then_type->cast<ListType>();
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `irange`, `outputs`, `size`, `output`, `type`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`irange`, `outputs`, `size`, `output`, `type`。

### Lines 615-623
```cpp
    auto else_list_type = else_type->cast<ListType>();
    auto then_optional_type = then_type->cast<OptionalType>();
    auto else_optional_type = else_type->cast<OptionalType>();
    auto then_none_type = then_type->cast<NoneType>();
    auto else_none_type = else_type->cast<NoneType>();
    if (then_tensor_type || else_tensor_type) {
      if (TypePtr merged_type =
              mergeTensorType(then_tensor_type, else_tensor_type)) {
        if (else_optional_type || else_none_type || then_optional_type ||
```
- EN: This block handles conditional branches. Key symbols: `mergeTensorType`.
- CN: 该代码块处理条件分支。关键符号：`mergeTensorType`。

### Lines 624-638
```cpp
            then_none_type) {
          merged_type = OptionalType::create(merged_type);
        }
        output_i->setType(merged_type);
      }
    } else if (then_list_type || else_list_type) {
      if (TypePtr merged_type = mergeListType(then_list_type, else_list_type)) {
        if (else_optional_type || else_none_type || then_optional_type ||
            then_none_type) {
          merged_type = OptionalType::create(merged_type);
        }
        output_i->setType(merged_type);
      }
    }

```
- EN: This block handles conditional branches. Key symbols: `create`, `setType`, `mergeListType`.
- CN: 该代码块处理条件分支。关键符号：`create`, `setType`, `mergeListType`。

### Lines 639-651
```cpp
    if (then_optional_type || else_optional_type) {
      if (auto optional_type =
              mergeOptionalType(then_optional_type, else_optional_type)) {
        output_i->setType(optional_type);
        // Both branches output types must match.
        if (!then_optional_type) {
          ReplaceBlockOutputWithOptional(optional_type, then_block, i);
        } else if (!else_optional_type) {
          ReplaceBlockOutputWithOptional(optional_type, else_block, i);
        }
      }
    }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `mergeOptionalType`, `setType`, `ReplaceBlockOutputWithOptional`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`mergeOptionalType`, `setType`, `ReplaceBlockOutputWithOptional`。

### Lines 652-663
```cpp
    if (then_none_type && !else_optional_type) {
      ReplaceBlockOutputWithOptional(
          output_i->type()->cast<OptionalType>(), then_block, i);
    }

    if (else_none_type && !then_optional_type) {
      ReplaceBlockOutputWithOptional(
          output_i->type()->cast<OptionalType>(), else_block, i);
    }
  }
}

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `ReplaceBlockOutputWithOptional`, `type`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`ReplaceBlockOutputWithOptional`, `type`。

### Lines 664-672
```cpp
static std::vector<Value*> FixupONNXIfNode(Node* node, int opset_version) {
  if (node->kind() != ::c10::onnx::If) {
    return node->outputs().vec();
  }
  GRAPH_DUMP("Graph before fixing controlflow: ", node->owningGraph());
  FixupONNXSubblockOutputs(node);
  ONNXFixupUninitializedOutput(node, opset_version);
  ONNXMergeIfBlockOutputShapes(node);

```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `FixupONNXIfNode`, `kind`, `outputs`, `vec`, `owningGraph`, `FixupONNXSubblockOutputs`, `...`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`FixupONNXIfNode`, `kind`, `outputs`, `vec`, `owningGraph`, `FixupONNXSubblockOutputs`, `...`。

### Lines 673-683
```cpp
  GRAPH_DUMP("Graph after fixing controlflow: ", node->owningGraph());
  return node->outputs().vec();
}

std::vector<Value*> FixupONNXControlflowNode(Node* n, int opset_version) {
  switch (n->kind()) {
    case ::c10::onnx::Loop: {
      return FixupONNXLoopNode(n, opset_version);
    }
    case ::c10::onnx::If: {
      return FixupONNXIfNode(n, opset_version);
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `owningGraph`, `outputs`, `vec`, `FixupONNXControlflowNode`, `kind`, `FixupONNXLoopNode`, `...`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`owningGraph`, `outputs`, `vec`, `FixupONNXControlflowNode`, `kind`, `FixupONNXLoopNode`, `...`。

### Lines 684-696
```cpp
    }
    default:
      return n->outputs().vec();
  }
}

void FixupONNXControlflowNodeOutputs(Node* n) {
  switch (n->kind()) {
    case ::c10::onnx::Loop: {
      Block* loop_block = n->blocks().at(0);
      // inputs (0, 1) are (i, cond), remainder are carried outputs.
      size_t loop_carried_output_size = loop_block->inputs().size() - 2;

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `outputs`, `vec`, `FixupONNXControlflowNodeOutputs`, `kind`, `blocks`, `inputs`, `...`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`outputs`, `vec`, `FixupONNXControlflowNodeOutputs`, `kind`, `blocks`, `inputs`, `...`。

### Lines 697-707
```cpp
      for (auto i : c10::irange(n->outputs().size())) {
        if (i < loop_carried_output_size) {
          const TypePtr block_input_type =
              loop_block->inputs().at(i + 2)->type();
          const TypePtr block_output_type =
              loop_block->outputs().at(i + 1)->type();
          TypePtr type = block_output_type;
          // Handle the case where a block input is Optional but the
          // output is not (i.e. if the loop executes > 0 times, the
          // output will not be None).
          if (block_input_type->cast<OptionalType>() &&
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `irange`, `outputs`, `size`, `inputs`, `type`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`irange`, `outputs`, `size`, `inputs`, `type`。

### Lines 708-716
```cpp
              !block_output_type->cast<OptionalType>()) {
            type = OptionalType::create(block_output_type);
          }
          n->output(i)->setType(type);
        } else {
          // scan output, should be a Tensor type
          TypePtr type = loop_block->outputs().at(i + 1)->type();
          if (auto t_type = type->cast<TensorType>()) {
            auto sizes = t_type->symbolic_sizes().sizes();
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `create`, `output`, `setType`, `outputs`, `type`, `symbolic_sizes`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`create`, `output`, `setType`, `outputs`, `type`, `symbolic_sizes`, `...`。

### Lines 717-734
```cpp
            if (sizes.has_value()) {
              sizes.value().emplace(
                  sizes.value().begin(), c10::ShapeSymbol::newSymbol());
              type = t_type->withSymbolicShapes(sizes.value());
            }
          }
          n->output(i)->setType(type);
        }
      }
      break;
    }
    case ::c10::onnx::If: {
      ONNXMergeIfBlockOutputShapes(n);
      break;
    }
    default:
      break;
  }
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `has_value`, `value`, `emplace`, `begin`, `newSymbol`, `withSymbolicShapes`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`has_value`, `value`, `emplace`, `begin`, `newSymbol`, `withSymbolicShapes`, `...`。

### Lines 735-737
```cpp
}

} // namespace torch::jit
```
- EN: Opens or closes namespace scopes so the remaining declarations live in the expected JIT/frontend module.
- CN: 打开或关闭命名空间作用域，使后续声明归属于预期的 JIT/前端模块。


## Key Concepts / 关键概念
- EN: IR graph manipulation — the file inspects or rewrites TorchScript graphs and nodes.
  CN: IR 图操作——该文件会检查或改写 TorchScript 的图、节点与块。
- EN: Optimization pass — the code simplifies graphs by removing redundant work or folding values.
  CN: 优化 pass——代码通过删除冗余计算或常量折叠来简化图。
- EN: Fusion support — the file prepares groups of operations for fused execution.
  CN: 融合支持——该文件会为融合执行准备算子分组。
- EN: Type and shape reasoning — the code propagates static information across the IR.
  CN: 类型与形状推理——代码会在 IR 中传播静态信息。
- EN: Export interop — the implementation helps bridge TorchScript graphs to ONNX-style representations.
  CN: 导出互操作——实现帮助 TorchScript 图与 ONNX 风格表示之间建立桥接。
- EN: Subsystem integration — the implementation collaborates with nearby JIT/frontend headers and runtime helpers.
  CN: 子系统集成——实现会与附近的 JIT/前端头文件以及运行时辅助模块协同工作。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `torch/csrc/jit/passes/onnx/fixup_onnx_controlflow.h`, `ATen/InitialTensorOptions.h`, `c10/util/irange.h`, `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/onnx/helper.h`, `torch/csrc/jit/passes/onnx/shape_type_inference.h`
- External includes / 外部头文件: 无
- Namespaces / 命名空间: `torch::jit`
- Representative symbols / 代表性符号: `CreateCastToBoolNode`, `create`, `addInput`, `i_`, `output`, `setType`, `get`, `InsertCastForCond`, `insertBefore`, `replaceInputWith`, `...`
