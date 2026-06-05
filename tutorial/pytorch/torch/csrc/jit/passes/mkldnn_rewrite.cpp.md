# mkldnn_rewrite.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/passes/mkldnn_rewrite.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements the TorchScript JIT pass logic for mkldnn rewrite, including graph analysis and rewrites.
- 用途 (CN): 实现与 mkldnn rewrite 相关的 TorchScript JIT pass 逻辑，包括图分析与改写。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9
```cpp
#include <ATen/Config.h>
#include <ATen/code_template.h>
#include <torch/csrc/jit/ir/ir.h>
#include <torch/csrc/jit/jit_log.h>
#include <torch/csrc/jit/passes/constant_propagation.h>
#include <torch/csrc/jit/passes/dead_code_elimination.h>
#include <torch/csrc/jit/passes/mkldnn_rewrite.h>
#include <torch/csrc/jit/tensorexpr/kernel.h>

```
- EN: Pulls in the headers needed by the mkldnn rewrite logic. Internal dependencies: `ATen/Config.h`, `ATen/code_template.h`, `torch/csrc/jit/ir/ir.h`, `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/constant_propagation.h`, `...`; external dependencies: none.
- CN: 为 mkldnn rewrite 相关逻辑引入所需头文件。内部依赖：`ATen/Config.h`, `ATen/code_template.h`, `torch/csrc/jit/ir/ir.h`, `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/constant_propagation.h`, `...`；外部依赖：无。

### Lines 10-16
```cpp
namespace torch::jit {

#if AT_MKLDNN_ENABLED()

static c10::VaryingShape<int64_t> getSizesOf(Node* n, size_t idx) {
  auto tt = n->input(idx)->type()->cast<TensorType>();
  return tt->sizes();
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `getSizesOf`, `input`, `type`, `sizes`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`getSizesOf`, `input`, `type`, `sizes`。

### Lines 17-28
```cpp
}

static void insertPrePackedConvOpForNode(Node* n) {
  constexpr int POS_INPUT = 0;
  constexpr int POS_WEIGHT = 1;
  if (!tensorexpr::isContiguous(
          n->input(POS_INPUT), at::MemoryFormat::ChannelsLast)) {
    GRAPH_DEBUG(
        "insertPrePackedConvOpForNode: input is not ChannelsLast contiguous");
    return;
  }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `insertPrePackedConvOpForNode`, `isContiguous`, `input`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`insertPrePackedConvOpForNode`, `isContiguous`, `input`。

### Lines 29-35
```cpp
  if (!tensorexpr::isContiguous(
          n->input(POS_WEIGHT), at::MemoryFormat::ChannelsLast)) {
    GRAPH_DEBUG(
        "insertPrePackedConvOpForNode: weight is not ChannelsLast contiguous");
    return;
  }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `isContiguous`, `input`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`isContiguous`, `input`。

### Lines 36-43
```cpp
  // Leave depthwise conv2d to NNC
  if (tensorexpr::conv2dIsSupportedJit(n)) {
    GRAPH_DEBUG("insertPrePackedConvOpForNode: leave depthwise conv2d to NNC");
    return;
  }

  WithInsertPoint guard(n);
  auto graph = n->owningGraph();
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `conv2dIsSupportedJit`, `guard`, `owningGraph`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`conv2dIsSupportedJit`, `guard`, `owningGraph`。

### Lines 44-51
```cpp

  auto input_sizes = getSizesOf(n, POS_INPUT);
  IValue input_size_value(*input_sizes.concrete_sizes());
  auto input_size = graph->insertConstant(input_size_value);

  auto prepack_node = graph->create(
      Symbol::fromQualString("mkldnn_prepacked::conv2d_prepack"), 1);

```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `getSizesOf`, `input_size_value`, `concrete_sizes`, `insertConstant`, `create`, `fromQualString`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`getSizesOf`, `input_size_value`, `concrete_sizes`, `insertConstant`, `create`, `fromQualString`。

### Lines 52-58
```cpp
  // skip input value
  for (const auto i : c10::irange(1, n->inputs().size())) {
    Value* v = n->input(i);
    prepack_node->addInput(v);
  }
  prepack_node->addInput(input_size);
  auto attr = graph->insertConstant(IValue("none"));
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `irange`, `inputs`, `size`, `input`, `addInput`, `insertConstant`, `...`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`irange`, `inputs`, `size`, `input`, `addInput`, `insertConstant`, `...`。

### Lines 59-69
```cpp
  prepack_node->addInput(attr);
  prepack_node->output()->setType(
      getCustomClass("__torch__.torch.classes.mkldnn.ConvOpContext"));
  graph->insertNode(prepack_node);

  auto prepack_conv = graph->insertNode(
      graph->create(Symbol::fromQualString("mkldnn_prepacked::conv2d_run"), 1));
  prepack_conv->addInput(n->input(0));
  prepack_conv->addInput(prepack_node->output());
  prepack_conv->output()->setType(n->output()->type()->cast<TensorType>());

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `addInput`, `output`, `setType`, `getCustomClass`, `insertNode`, `create`, `...`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`addInput`, `output`, `setType`, `getCustomClass`, `insertNode`, `create`, `...`。

### Lines 70-76
```cpp
  n->output()->replaceAllUsesWith(prepack_conv->output());
}

static bool isTensorTypeCPU(Node* node) {
  for (const auto& input : node->inputs()) {
    auto type = input->type()->cast<TensorType>();
    if (!type) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `output`, `replaceAllUsesWith`, `isTensorTypeCPU`, `inputs`, `type`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`output`, `replaceAllUsesWith`, `isTensorTypeCPU`, `inputs`, `type`。

### Lines 77-83
```cpp
      continue;
    }
    auto device = type->device();
    if (!device) {
      return false;
    }
    if (!device->is_cpu()) {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value. Key symbols: `device`, `is_cpu`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值。关键符号：`device`, `is_cpu`。

### Lines 84-90
```cpp
      return false;
    }
  }
  return true;
}

static void insertPrePackedConvOp(Block* b) {
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `insertPrePackedConvOp`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`insertPrePackedConvOp`。

### Lines 91-97
```cpp
  for (Node* n : b->nodes()) {
    for (Block* b : n->blocks()) {
      insertPrePackedConvOp(b);
    }

    if (n->kind() == aten::conv2d) {
      if (isTensorTypeCPU(n)) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `nodes`, `blocks`, `insertPrePackedConvOp`, `kind`, `isTensorTypeCPU`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`nodes`, `blocks`, `insertPrePackedConvOp`, `kind`, `isTensorTypeCPU`。

### Lines 98-104
```cpp
        insertPrePackedConvOpForNode(n);
      }
    }
  }
  EliminateDeadCode(b);
}

```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `insertPrePackedConvOpForNode`, `EliminateDeadCode`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`insertPrePackedConvOpForNode`, `EliminateDeadCode`。

### Lines 105-112
```cpp
static void insertMkldnnPrePackedConv2dOp(std::shared_ptr<Graph>& graph) {
  insertPrePackedConvOp(graph->block());
}

static void insertMkldnnPrePackedOps(std::shared_ptr<Graph>& graph) {
  insertMkldnnPrePackedConv2dOp(graph);
}

```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `insertMkldnnPrePackedConv2dOp`, `insertPrePackedConvOp`, `block`, `insertMkldnnPrePackedOps`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`insertMkldnnPrePackedConv2dOp`, `insertPrePackedConvOp`, `block`, `insertMkldnnPrePackedOps`。

### Lines 113-122
```cpp
static void FuseReluWithPackedOps(std::shared_ptr<Graph>& graph) {
  auto conv_op_rstring = at::jit::CodeTemplate(R"(
    graph(%input, %weight, %bias, %stride:int[], %padding:int[],
          %dilation:int[], %groups:int, %input_size:int[], %dummy_attr:str):
        %packed_weight_bias = mkldnn_prepacked::conv2d_prepack(
            %weight, %bias, %stride, %padding, %dilation, %groups,
            %input_size, %dummy_attr)
        %conv2d_res = mkldnn_prepacked::conv2d_run(%input, %packed_weight_bias)
        %res = aten::${op}(%conv2d_res)
        return (%res))");
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `FuseReluWithPackedOps`, `CodeTemplate`, `graph`, `conv2d_prepack`, `conv2d_run`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`FuseReluWithPackedOps`, `CodeTemplate`, `graph`, `conv2d_prepack`, `conv2d_run`。

### Lines 123-132
```cpp

  auto conv_op_fused_rstring = at::jit::CodeTemplate(R"(
    graph(%input, %weight, %bias, %stride:int[], %padding:int[],
          %dilation:int[], %groups:int, %input_size:int[], %dummy_attr:str):
        %attr: str = prim::Constant[value="${op_attr}"]()
        %packed_weight_bias : __torch__.torch.classes.mkldnn.ConvOpContext = mkldnn_prepacked::conv2d_prepack(
            %weight, %bias, %stride, %padding, %dilation, %groups,
            %input_size, %attr)
        %res = mkldnn_prepacked::conv2d_run(%input, %packed_weight_bias)
        return (%res))");
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `CodeTemplate`, `graph`, `conv2d_prepack`, `conv2d_run`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`CodeTemplate`, `graph`, `conv2d_prepack`, `conv2d_run`。

### Lines 133-139
```cpp

  for (auto const& it : mkldnn::fusion_rewrite_map) {
    std::string op = it.first;
    if (op == std::string("none")) {
      continue;
    }

```
- EN: This block handles conditional branches; iterates over collections or graph structures. Key symbols: `string`.
- CN: 该代码块处理条件分支；遍历集合或图结构。关键符号：`string`。

### Lines 140-149
```cpp
    at::jit::TemplateEnv env;
    env.s("op", op);

    at::jit::TemplateEnv env_fused;
    env_fused.s("op_attr", op);

    SubgraphRewriter rewriter;
    rewriter.RegisterRewritePattern(
        conv_op_rstring.format(env), conv_op_fused_rstring.format(env_fused));

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `s`, `RegisterRewritePattern`, `format`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`s`, `RegisterRewritePattern`, `format`。

### Lines 150-156
```cpp
    auto filters = it.second;
    rewriter.runOnGraph(graph, filters);
  }
}

static void PrePackingOpsFolder(Block* b) {
  auto is_foldable_op = [](const Node* n) -> bool {
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `runOnGraph`, `PrePackingOpsFolder`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`runOnGraph`, `PrePackingOpsFolder`。

### Lines 157-163
```cpp
    return (
        n->kind() ==
        Symbol::fromQualString("mkldnn_prepacked::conv2d_prepack"));
  };

  std::unordered_set<Node*> nodes_to_delete;
  for (Node* n : b->nodes()) {
```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `kind`, `fromQualString`, `nodes`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`kind`, `fromQualString`, `nodes`。

### Lines 164-170
```cpp
    for (Block* block : n->blocks()) {
      PrePackingOpsFolder(block);
    }
    if (is_foldable_op(n)) {
      auto optional_outputs = torch::jit::runNodeIfInputsAreConstant(n);
      if (optional_outputs) {
        auto outputs = optional_outputs.value();
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `blocks`, `PrePackingOpsFolder`, `is_foldable_op`, `runNodeIfInputsAreConstant`, `value`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`blocks`, `PrePackingOpsFolder`, `is_foldable_op`, `runNodeIfInputsAreConstant`, `value`。

### Lines 171-177
```cpp
        TORCH_CHECK(outputs.size() == 1, "Prepack ops have single output");
        Value* prepack_op_value = n->output(0);
        auto graph = n->owningGraph();
        WithInsertPoint ins(prepack_op_value->node());
        auto weak_class_obj =
            outputs[0].toObject()->copy_to_weak_compilation_ref();
        Value* packed_weight = graph->insertConstant(weak_class_obj)
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `size`, `output`, `owningGraph`, `ins`, `node`, `toObject`, `...`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`size`, `output`, `owningGraph`, `ins`, `node`, `toObject`, `...`。

### Lines 178-184
```cpp
                                   ->setType(n->output(0)->type());
        prepack_op_value->replaceAllUsesWith(packed_weight);
        nodes_to_delete.insert(n);
      }
    }
  }
  for (auto n : nodes_to_delete) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `setType`, `output`, `type`, `replaceAllUsesWith`, `insert`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`setType`, `output`, `type`, `replaceAllUsesWith`, `insert`。

### Lines 185-191
```cpp
    n->removeAllInputs();
  }
  for (auto n : nodes_to_delete) {
    n->destroy();
  }
}

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `removeAllInputs`, `destroy`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`removeAllInputs`, `destroy`。

### Lines 192-205
```cpp
static void FoldPrePackingOps(std::shared_ptr<Graph>& graph) {
  PrePackingOpsFolder(graph->block());
}

void FuseConvWithEltwise(std::shared_ptr<Graph>& graph) {
  GRAPH_DEBUG(
      "Before insertMkldnnPrePackedOps. Beginning of FuseConvWithEltwise\n",
      *graph);
  insertMkldnnPrePackedOps(graph);
  GRAPH_DEBUG(
      "After insertMkldnnPrePackedOps, before FuseReluWithPackedOps\n", *graph);
  FuseReluWithPackedOps(graph);
  GRAPH_DEBUG(
      "After FuseReluWithPackedOps, before FoldPrePackingOps\n", *graph);
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `FoldPrePackingOps`, `PrePackingOpsFolder`, `block`, `FuseConvWithEltwise`, `insertMkldnnPrePackedOps`, `FuseReluWithPackedOps`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`FoldPrePackingOps`, `PrePackingOpsFolder`, `block`, `FuseConvWithEltwise`, `insertMkldnnPrePackedOps`, `FuseReluWithPackedOps`。

### Lines 206-212
```cpp
  FoldPrePackingOps(graph);
  GRAPH_DEBUG("After FoldPrePackingOps. End of FuseConvWithEltwise\n", *graph);
}

#else

void FuseConvWithEltwise(std::shared_ptr<Graph>& graph) {
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `FoldPrePackingOps`, `FuseConvWithEltwise`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`FoldPrePackingOps`, `FuseConvWithEltwise`。

### Lines 213-218
```cpp
  GRAPH_DEBUG("MKLDNN Not enabled");
}

#endif // AT_MKLDNN_ENABLED()

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
- EN: Subsystem integration — the implementation collaborates with nearby JIT/frontend headers and runtime helpers.
  CN: 子系统集成——实现会与附近的 JIT/前端头文件以及运行时辅助模块协同工作。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `ATen/Config.h`, `ATen/code_template.h`, `torch/csrc/jit/ir/ir.h`, `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/constant_propagation.h`, `torch/csrc/jit/passes/dead_code_elimination.h`, `torch/csrc/jit/passes/mkldnn_rewrite.h`, `torch/csrc/jit/tensorexpr/kernel.h`
- External includes / 外部头文件: 无
- Namespaces / 命名空间: `torch::jit`
- Representative symbols / 代表性符号: `getSizesOf`, `input`, `type`, `sizes`, `insertPrePackedConvOpForNode`, `isContiguous`, `conv2dIsSupportedJit`, `guard`, `owningGraph`, `input_size_value`, `...`
