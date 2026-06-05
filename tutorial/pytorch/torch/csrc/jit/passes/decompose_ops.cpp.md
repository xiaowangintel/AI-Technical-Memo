# decompose_ops.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/passes/decompose_ops.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements the TorchScript JIT pass logic for decompose ops, including graph analysis and rewrites.
- 用途 (CN): 实现与 decompose ops 相关的 TorchScript JIT pass 逻辑，包括图分析与改写。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10
```cpp
#include <torch/csrc/jit/passes/decompose_ops.h>

#include <torch/csrc/jit/frontend/ir_emitter.h>
#include <torch/csrc/jit/passes/constant_propagation.h>
#include <torch/csrc/jit/passes/dead_code_elimination.h>
#include <torch/csrc/jit/passes/shape_analysis.h>
#include <torch/csrc/jit/passes/utils/subgraph_utils.h>
#include <torch/csrc/jit/runtime/custom_operator.h>
#include <torch/csrc/jit/runtime/operator.h>

```
- EN: Pulls in the headers needed by the decompose ops logic. Internal dependencies: `torch/csrc/jit/passes/decompose_ops.h`, `torch/csrc/jit/frontend/ir_emitter.h`, `torch/csrc/jit/passes/constant_propagation.h`, `torch/csrc/jit/passes/dead_code_elimination.h`, `torch/csrc/jit/passes/shape_analysis.h`, `...`; external dependencies: none.
- CN: 为 decompose ops 相关逻辑引入所需头文件。内部依赖：`torch/csrc/jit/passes/decompose_ops.h`, `torch/csrc/jit/frontend/ir_emitter.h`, `torch/csrc/jit/passes/constant_propagation.h`, `torch/csrc/jit/passes/dead_code_elimination.h`, `torch/csrc/jit/passes/shape_analysis.h`, `...`；外部依赖：无。

### Lines 11-17
```cpp
#include <ATen/core/symbol.h>

namespace torch::jit {

namespace {
c10::AliasAnalysisKind aliasAnalysisFromSchema() {
  return c10::AliasAnalysisKind::FROM_SCHEMA;
```
- EN: This block produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: `aliasAnalysisFromSchema`.
- CN: 该代码块返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：`aliasAnalysisFromSchema`。

### Lines 18-24
```cpp
}
} // namespace

// helper to determine if an optional tensor argument/value passed in is
// statically defined (neither a None constant nor a Optional[Tensor] type)
// return yes, no, or no value if we can't tell
static std::optional<bool> isDefined(Value* tensor) {
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `isDefined`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`isDefined`。

### Lines 25-31
```cpp
  if (tensor->type()->isSubtypeOf(*TensorType::get())) {
    return true;
  }
  if (tensor->node()->mustBeNone()) {
    return false;
  }
  return {};
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `type`, `isSubtypeOf`, `get`, `node`, `mustBeNone`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`type`, `isSubtypeOf`, `get`, `node`, `mustBeNone`。

### Lines 32-38
```cpp
}

static bool isDecomposableNorm(Node* normalize_op) {
  static const OperatorSet decomposable_normalization_ops = {
      "aten::batch_norm(Tensor input, Tensor? weight, Tensor? bias, Tensor? running_mean, Tensor? running_var, bool training, float momentum, float eps, bool cudnn_enabled) -> Tensor",
      "aten::layer_norm(Tensor input, int[] normalized_shape, Tensor? weight, Tensor? bias, float eps, bool cudnn_enable) -> Tensor",
  };
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `isDecomposableNorm`, `batch_norm`, `layer_norm`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`isDecomposableNorm`, `batch_norm`, `layer_norm`。

### Lines 39-46
```cpp
  Value* input = normalize_op->namedInput(attr::input);
  if (!input->type()->isSubtypeOf(*TensorType::get())) {
    return false;
  }
  auto device = input->type()->expectRef<TensorType>().device();
  // As of now, we do the decomposition for batchnorm/layernorm on GPU device
  // only
  if (!device || !(*device).is_cuda()) {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `namedInput`, `type`, `isSubtypeOf`, `get`, `device`, `is_cuda`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`namedInput`, `type`, `isSubtypeOf`, `get`, `device`, `is_cuda`。

### Lines 47-54
```cpp
    return false;
  }

  if (normalize_op->isMemberOf(decomposable_normalization_ops)) {
    // If we can't determine if weight and bias is defined statically there's
    // really no point in decomposing normalization into simpler ops, since it
    // won't get fused into a single kernel.
    return isDefined(normalize_op->namedInput(attr::weight)).has_value() &&
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `isMemberOf`, `isDefined`, `namedInput`, `has_value`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`isMemberOf`, `isDefined`, `namedInput`, `has_value`。

### Lines 55-65
```cpp
        isDefined(normalize_op->namedInput(attr::bias)).has_value();
  }
  return false;
}

static RegisterOperators reg_ops(
    {Operator(
         "aten::_ncf_unsqueeze(Tensor(a) self, int ndim) -> Tensor(a)",
         [](Stack& stack) {
           const int64_t ndim = pop(stack).toInt();
           auto self = pop(stack).toTensor();
```
- EN: Registers callable entities, pass hooks, or exported interfaces so runtime code can discover decompose ops behavior. Symbols: `isDefined`, `namedInput`, `has_value`, `reg_ops`, `Operator`, `_ncf_unsqueeze`, `...`.
- CN: 注册可调用实体、pass 钩子或导出接口，使运行时能够发现 decompose ops 的行为。符号：`isDefined`, `namedInput`, `has_value`, `reg_ops`, `Operator`, `_ncf_unsqueeze`, `...`。

### Lines 66-76
```cpp
           c10::SmallVector<int64_t, 8> sizes(ndim, 1);
           AT_ASSERT(self.dim() == 1);
           sizes.at(1) = self.size(0);
           push(stack, self.reshape(sizes));
         },
         aliasAnalysisFromSchema()),
     Operator(
         "aten::_ncf_view(Tensor(a) self, int[] input_shape, int normalized_ndim) -> Tensor(a)",
         [](Stack& stack) {
           const int64_t normalized_ndim = pop(stack).toInt();
           auto input_shape = pop(stack).toIntList();
```
- EN: Registers callable entities, pass hooks, or exported interfaces so runtime code can discover decompose ops behavior. Symbols: `sizes`, `dim`, `size`, `push`, `reshape`, `aliasAnalysisFromSchema`, `...`.
- CN: 注册可调用实体、pass 钩子或导出接口，使运行时能够发现 decompose ops 的行为。符号：`sizes`, `dim`, `size`, `push`, `reshape`, `aliasAnalysisFromSchema`, `...`。

### Lines 77-86
```cpp
           auto self = pop(stack).toTensor();
           const int64_t input_ndim = input_shape.size();
           c10::SmallVector<int64_t, 8> sizes(input_ndim, 1);
           for (int i = 0; i < input_ndim - normalized_ndim; ++i) {
             sizes.at(i) = input_shape.get(i);
           }
           push(stack, self.reshape(sizes));
         },
         aliasAnalysisFromSchema())});

```
- EN: This block iterates over collections or graph structures; advances frontend parsing or lowering state. Key symbols: `pop`, `toTensor`, `size`, `sizes`, `get`, `push`, `...`.
- CN: 该代码块遍历集合或图结构；推进前端解析或降级状态。关键符号：`pop`, `toTensor`, `size`, `sizes`, `get`, `push`, `...`。

### Lines 87-94
```cpp
static bool DecomposeOps(Block* block, CompilationUnit& decompose_funcs) {
  bool decomposed = false;
  for (auto it = block->nodes().begin(), end = block->nodes().end(); it != end;
       ++it) {
    for (auto sub : it->blocks()) {
      DecomposeOps(sub, decompose_funcs);
    }

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `DecomposeOps`, `nodes`, `begin`, `end`, `blocks`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`DecomposeOps`, `nodes`, `begin`, `end`, `blocks`。

### Lines 95-102
```cpp
    if (it->matches(
            "aten::addmm(Tensor self, Tensor mat1, Tensor mat2, *, Scalar beta, Scalar alpha) -> Tensor",
            /*const_inputs=*/{attr::beta, attr::alpha})) {
      // For the case where we have an addmm where alpha and beta are Attributes
      // and both of those scalars are equal to 1.0, decompose this into an mm
      // followed by an add so that it can go through the existing optimization
      // (batchmm)
      if (it->get<at::Scalar>(attr::alpha)->toComplexDouble() != 1.0 ||
```
- EN: This block handles conditional branches. Key symbols: `matches`, `addmm`, `toComplexDouble`.
- CN: 该代码块处理条件分支。关键符号：`matches`, `addmm`, `toComplexDouble`。

### Lines 103-109
```cpp
          it->get<at::Scalar>(attr::beta)->toComplexDouble() != 1.0) {
        continue;
      }

      decomposed = true;
      WithInsertPoint guard(*it);
      std::shared_ptr<Graph> d_graph =
```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `toComplexDouble`, `guard`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`toComplexDouble`, `guard`。

### Lines 110-122
```cpp
          toGraphFunction(decompose_funcs.get_function("addmm")).graph();
      Value* new_output =
          insertGraph(*it->owningGraph(), *d_graph, it->inputs()).at(0);
      // Set the output of the decomposed graph to have the same output type as
      // the original op otherwise the canonicalized graph will have TensorType
      // as the output of this node which is incorrect
      new_output->setType(it->output()->type());
      it->output()->replaceAllUsesWith(new_output);
      it.destroyCurrent();
    } else if (
        it->matches(
            "aten::batch_norm(Tensor input, Tensor? weight, Tensor? bias, Tensor? running_mean, Tensor? running_var, bool training, float momentum, float eps, bool cudnn_enabled) -> Tensor")) {
      if (!isDecomposableNorm(*it)) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `toGraphFunction`, `get_function`, `graph`, `insertGraph`, `owningGraph`, `inputs`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`toGraphFunction`, `get_function`, `graph`, `insertGraph`, `owningGraph`, `inputs`, `...`。

### Lines 123-129
```cpp
        continue;
      }
      decomposed = true;
      WithInsertPoint insert_guard{*it};
      Graph* graph = it->owningGraph();
      Value* input = it->namedInput(attr::input);
      Value* input_dim = graph->insert(aten::dim, {input});
```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `owningGraph`, `namedInput`, `insert`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`owningGraph`, `namedInput`, `insert`。

### Lines 130-137
```cpp
      std::vector<Value*> inputs{
          input,
          it->namedInput(attr::running_mean),
          it->namedInput(attr::running_var),
          it->namedInput(attr::training),
          it->namedInput(attr::momentum),
          it->namedInput(attr::eps)};

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `namedInput`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`namedInput`。

### Lines 138-144
```cpp
      // inline the compiled decomposed batchnorm
      std::shared_ptr<Graph> d_graph =
          toGraphFunction(decompose_funcs.get_function("batch_norm")).graph();
      Value* new_output = insertGraph(*graph, *d_graph, inputs).at(0);

      // post processing the graph
      Value* weight = it->namedInput(attr::weight);
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `toGraphFunction`, `get_function`, `graph`, `insertGraph`, `namedInput`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`toGraphFunction`, `get_function`, `graph`, `insertGraph`, `namedInput`。

### Lines 145-151
```cpp
      Value* bias = it->namedInput(attr::bias);
      if (isDefined(weight).value()) {
        Value* expanded_weight =
            graph->insert(aten::_ncf_unsqueeze, {weight, input_dim});
        new_output = graph->insert(aten::mul, {new_output, expanded_weight});
      }
      if (isDefined(bias).value()) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `namedInput`, `isDefined`, `value`, `insert`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`namedInput`, `isDefined`, `value`, `insert`。

### Lines 152-161
```cpp
        Value* expanded_bias =
            graph->insert(aten::_ncf_unsqueeze, {bias, input_dim});
        new_output = graph->insert(aten::add, {new_output, expanded_bias});
      }
      it->output()->replaceAllUsesWith(new_output);
      it.destroyCurrent();
    } else if (
        it->matches(
            "aten::layer_norm(Tensor input, int[] normalized_shape, Tensor? weight, Tensor? bias, float eps, bool cudnn_enable) -> Tensor")) {
      if (!isDecomposableNorm(*it)) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `insert`, `output`, `replaceAllUsesWith`, `destroyCurrent`, `matches`, `layer_norm`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`insert`, `output`, `replaceAllUsesWith`, `destroyCurrent`, `matches`, `layer_norm`, `...`。

### Lines 162-172
```cpp
        continue;
      }
      decomposed = true;
      WithInsertPoint insert_guard{*it};
      Graph* graph = it->owningGraph();
      std::vector<Value*> inputs{
          it->namedInput(attr::input),
          it->namedInput(attr::normalized_shape),
          it->namedInput(attr::eps),
          it->namedInput(attr::cudnn_enable)};

```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `owningGraph`, `namedInput`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`owningGraph`, `namedInput`。

### Lines 173-179
```cpp
      // inline the compiled decomposed layernorm
      std::shared_ptr<Graph> d_graph =
          toGraphFunction(decompose_funcs.get_function("layer_norm")).graph();
      Value* new_output = insertGraph(*graph, *d_graph, inputs).at(0);

      // post processing the graph
      Value* weight = it->namedInput(attr::weight);
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `toGraphFunction`, `get_function`, `graph`, `insertGraph`, `namedInput`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`toGraphFunction`, `get_function`, `graph`, `insertGraph`, `namedInput`。

### Lines 180-191
```cpp
      Value* bias = it->namedInput(attr::bias);
      if (isDefined(weight).value()) {
        new_output = graph->insert(aten::mul, {new_output, weight});
      }
      if (isDefined(bias).value()) {
        new_output = graph->insert(aten::add, {new_output, bias});
      }
      it->output()->replaceAllUsesWith(new_output);
      it.destroyCurrent();
    }
  }
  return decomposed;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `namedInput`, `isDefined`, `value`, `insert`, `output`, `replaceAllUsesWith`, `...`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`namedInput`, `isDefined`, `value`, `insert`, `output`, `replaceAllUsesWith`, `...`。

### Lines 192-198
```cpp
}

void DecomposeOps(std::shared_ptr<Graph>& graph) {
  static CompilationUnit decompose_funcs(R"SCRIPT(
      def addmm(self: Tensor, mat1: Tensor, mat2: Tensor, beta: number = 1.0, alpha: number = 1.0):
          return self + mat1.mm(mat2)

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `DecomposeOps`, `decompose_funcs`, `addmm`, `mm`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`DecomposeOps`, `decompose_funcs`, `addmm`, `mm`。

### Lines 199-208
```cpp
      def batch_norm(input : Tensor, running_mean : Optional[Tensor], running_var : Optional[Tensor], training : bool, momentum : float, eps : float) -> Tensor:
          if training:
              norm_mean, norm_var = torch.batch_norm_update_stats(input, running_mean, running_var, momentum)
          else:
              norm_mean = torch._unwrap_optional(running_mean)
              norm_var = torch._unwrap_optional(running_var)
          norm_mean = torch._ncf_unsqueeze(norm_mean, input.dim())
          norm_var = torch._ncf_unsqueeze(norm_var, input.dim())
          norm_invstd = 1 / (torch.sqrt(norm_var + eps))
          return ((input - norm_mean) * norm_invstd)
```
- EN: This block handles conditional branches; produces a result or forwards a computed value. Key symbols: `batch_norm`, `batch_norm_update_stats`, `_unwrap_optional`, `_ncf_unsqueeze`, `dim`, `sqrt`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值。关键符号：`batch_norm`, `batch_norm_update_stats`, `_unwrap_optional`, `_ncf_unsqueeze`, `dim`, `sqrt`。

### Lines 209-221
```cpp

      def layer_norm(input : Tensor, normalized_shape : List[int], eps : float, cudnn_enable : bool) -> Tensor:
          input_ndim = input.dim()
          normalized_ndim = len(normalized_shape)
          n = 1
          for i in range(input_ndim - normalized_ndim):
              n *= input.size(i)
          input_reshape = input.contiguous().view(1, n, -1)
          mean, invstd = torch.batch_norm_stats(input_reshape, eps)
          input_shape = input.size()
          mean = torch._ncf_view(mean, input_shape, normalized_ndim)
          invstd = torch._ncf_view(invstd, input_shape, normalized_ndim)

```
- EN: This block implements local helper logic for decompose ops. Key symbols: `layer_norm`, `dim`, `len`, `range`, `size`, `contiguous`, `...`.
- CN: 该代码块实现与 decompose ops 相关的局部辅助逻辑。关键符号：`layer_norm`, `dim`, `len`, `range`, `size`, `contiguous`, `...`。

### Lines 222-232
```cpp
          return (input - mean) * invstd
      )SCRIPT");
  bool is_decomposed = DecomposeOps(graph->block(), decompose_funcs);
  if (is_decomposed) {
    // we only re-run those passes when the graph get decomposed
    PropagateInputShapes(graph);
    ConstantPropagation(graph);
    EliminateDeadCode(graph);
  }
}

```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `DecomposeOps`, `block`, `PropagateInputShapes`, `ConstantPropagation`, `EliminateDeadCode`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`DecomposeOps`, `block`, `PropagateInputShapes`, `ConstantPropagation`, `EliminateDeadCode`。

### Lines 233-233
```cpp
} // namespace torch::jit
```
- EN: Opens or closes namespace scopes so the remaining declarations live in the expected JIT/frontend module.
- CN: 打开或关闭命名空间作用域，使后续声明归属于预期的 JIT/前端模块。


## Key Concepts / 关键概念
- EN: IR graph manipulation — the file inspects or rewrites TorchScript graphs and nodes.
  CN: IR 图操作——该文件会检查或改写 TorchScript 的图、节点与块。
- EN: Alias and mutation reasoning — the code tracks side effects and write/read interactions.
  CN: 别名与变更分析——代码会跟踪副作用以及读写关系。
- EN: Schema handling — the implementation parses or normalizes operator/function signatures.
  CN: 模式处理——实现会解析或规范化算子/函数签名。
- EN: Frontend lowering — the file resolves symbols and lowers frontend constructs into IR.
  CN: 前端降级——该文件负责符号解析，并将前端结构降为 IR。
- EN: Optimization pass — the code simplifies graphs by removing redundant work or folding values.
  CN: 优化 pass——代码通过删除冗余计算或常量折叠来简化图。
- EN: Inlining — the implementation merges callees or blocks into their call sites.
  CN: 内联——实现会把被调用者或代码块并入调用位置。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `torch/csrc/jit/passes/decompose_ops.h`, `torch/csrc/jit/frontend/ir_emitter.h`, `torch/csrc/jit/passes/constant_propagation.h`, `torch/csrc/jit/passes/dead_code_elimination.h`, `torch/csrc/jit/passes/shape_analysis.h`, `torch/csrc/jit/passes/utils/subgraph_utils.h`, `torch/csrc/jit/runtime/custom_operator.h`, `torch/csrc/jit/runtime/operator.h`, `ATen/core/symbol.h`
- External includes / 外部头文件: 无
- Namespaces / 命名空间: `torch::jit`
- Representative symbols / 代表性符号: `aliasAnalysisFromSchema`, `isDefined`, `type`, `isSubtypeOf`, `get`, `node`, `mustBeNone`, `isDecomposableNorm`, `batch_norm`, `layer_norm`, `...`
