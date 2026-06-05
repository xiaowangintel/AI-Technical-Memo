# fuse_linear.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/passes/fuse_linear.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements the TorchScript JIT pass logic for fuse linear, including graph analysis and rewrites.
- 用途 (CN): 实现与 fuse linear 相关的 TorchScript JIT pass 逻辑，包括图分析与改写。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5
```cpp
#include <torch/csrc/jit/passes/fuse_linear.h>
#include <torch/csrc/jit/passes/graph_rewrite_helper.h>
#include <torch/csrc/jit/passes/quantization/helper.h>
#include <torch/csrc/jit/passes/subgraph_rewrite.h>

```
- EN: Pulls in the headers needed by the fuse linear logic. Internal dependencies: `torch/csrc/jit/passes/fuse_linear.h`, `torch/csrc/jit/passes/graph_rewrite_helper.h`, `torch/csrc/jit/passes/quantization/helper.h`, `torch/csrc/jit/passes/subgraph_rewrite.h`; external dependencies: none.
- CN: 为 fuse linear 相关逻辑引入所需头文件。内部依赖：`torch/csrc/jit/passes/fuse_linear.h`, `torch/csrc/jit/passes/graph_rewrite_helper.h`, `torch/csrc/jit/passes/quantization/helper.h`, `torch/csrc/jit/passes/subgraph_rewrite.h`；外部依赖：无。

### Lines 6-12
```cpp
namespace torch::jit {

void FuseLinear(std::shared_ptr<Graph>& graph) {
  std::string addmm_pattern = R"IR(
    graph(%input, %weight_t, %bias, %beta, %alpha):
        %res = aten::addmm(%bias, %input, %weight_t, %beta, %alpha)
        return (%res))IR";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `FuseLinear`, `graph`, `addmm`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`FuseLinear`, `graph`, `addmm`。

### Lines 13-17
```cpp
  std::string fused_linear_addmm = R"IR(
    graph(%input, %weight_t, %bias, %beta, %alpha):
        %weight = aten::t(%weight_t)
        %res = aten::linear(%input, %weight, %bias)
        return (%res))IR";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `t`, `linear`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `t`, `linear`。

### Lines 18-22
```cpp

  auto beta_is_one = [](const Match& match,
                        const std::unordered_map<std::string, Value*>& vmap) {
    return is_int_constant(match, vmap, "beta", 1);
  };
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `is_int_constant`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`is_int_constant`。

### Lines 23-30
```cpp

  // check %weight_t is produced by `aten::t` to make sure
  // we can transform the pattern to `aten::linear`
  auto weight_transposed =
      [](const Match& match,
         const std::unordered_map<std::string, Value*>& vmap) {
        const auto& match_vmap = match.values_map;
        auto v = match_vmap.at(vmap.at("weight_t"));
```
- EN: This block manipulates TorchScript IR objects. Key symbols: no dominant local symbol names.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：无明显局部符号。

### Lines 31-36
```cpp
        return v->node()->kind() == Symbol::aten("t");
      };

  // replace addmm pattern to linear
  SubgraphRewriter addmm_to_linear;
  std::vector<std::pair<std::string, std::string>> value_mappings(
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `node`, `kind`, `aten`, `value_mappings`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`node`, `kind`, `aten`, `value_mappings`。

### Lines 37-42
```cpp
      {{"weight", "res"}, {"res", "res"}});
  addmm_to_linear.RegisterRewritePattern(
      addmm_pattern, fused_linear_addmm, value_mappings);
  addmm_to_linear.runOnGraph(
      graph, {aten_add_alpha_is_one, beta_is_one, weight_transposed});

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `RegisterRewritePattern`, `runOnGraph`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`RegisterRewritePattern`, `runOnGraph`。

### Lines 43-47
```cpp
  std::string matmul_add_pattern = R"IR(
    graph(%input, %weight_t, %bias, %alpha):
        %output = aten::matmul(%input, %weight_t)
        %res = aten::add_(%output, %bias, %alpha)
        return (%res))IR";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `matmul`, `add_`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `matmul`, `add_`。

### Lines 48-52
```cpp
  std::string fused_linear_matmul = R"IR(
    graph(%input, %weight_t, %bias, %alpha):
        %weight = aten::t(%weight_t)
        %res = aten::linear(%input, %weight, %bias)
        return (%res))IR";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `t`, `linear`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `t`, `linear`。

### Lines 53-60
```cpp
  value_mappings = {{"weight", "output"}, {"res", "output"}};
  // replace matmul + add pattern to linear
  SubgraphRewriter matmuladd_to_linear;
  matmuladd_to_linear.RegisterRewritePattern(
      matmul_add_pattern, fused_linear_matmul, value_mappings);
  matmuladd_to_linear.runOnGraph(
      graph, {aten_add_alpha_is_one, weight_transposed});

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `RegisterRewritePattern`, `runOnGraph`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`RegisterRewritePattern`, `runOnGraph`。

### Lines 61-65
```cpp
  std::string matmul_pattern = R"IR(
    graph(%input, %weight_t):
        %output = aten::matmul(%input, %weight_t)
        return (%output))IR";
  std::string fused_linear_bias_none = R"IR(
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `matmul`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `matmul`。

### Lines 66-70
```cpp
    graph(%input, %weight_t):
        %weight = aten::t(%weight_t)
        %bias: Tensor? = prim::Constant()
        %res = aten::linear(%input, %weight, %bias)
        return (%res))IR";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `graph`, `t`, `Constant`, `linear`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`graph`, `t`, `Constant`, `linear`。

### Lines 71-77
```cpp

  // replace matmul with bias=None pattern to linear
  SubgraphRewriter matmul_to_linear;
  matmul_to_linear.RegisterRewritePattern(
      matmul_pattern, fused_linear_bias_none, value_mappings);
  matmul_to_linear.runOnGraph(graph, weight_transposed);

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `RegisterRewritePattern`, `runOnGraph`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`RegisterRewritePattern`, `runOnGraph`。

### Lines 78-84
```cpp
  // clean up extra transpose for the weight of aten::linear
  std::string linear_weight_extra_transpose = R"IR(
    graph(%input, %weight, %bias):
        %weight_t1 = aten::t(%weight)
        %weight_t2 = aten::t(%weight_t1)
        %res = aten::linear(%input, %weight_t2, %bias)
        return (%res))IR";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `t`, `linear`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `t`, `linear`。

### Lines 85-89
```cpp

  std::string linear_weight_no_transpose = R"IR(
    graph(%input, %weight, %bias):
        %res = aten::linear(%input, %weight, %bias)
        return (%res))IR";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `linear`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `linear`。

### Lines 90-98
```cpp

  value_mappings = {{"res", "res"}};
  SubgraphRewriter cleanup;
  cleanup.RegisterRewritePattern(
      linear_weight_extra_transpose,
      linear_weight_no_transpose,
      value_mappings);
  cleanup.runOnGraph(graph);

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `RegisterRewritePattern`, `runOnGraph`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`RegisterRewritePattern`, `runOnGraph`。

### Lines 99-103
```cpp
  SwapFunctionalLinear(graph);
}

void SwapFunctionalLinear(Module& module) {
  for (auto& method : module.get_methods()) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `SwapFunctionalLinear`, `get_methods`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`SwapFunctionalLinear`, `get_methods`。

### Lines 104-111
```cpp
    std::shared_ptr<Graph> g = method.graph();
    SwapFunctionalLinear(g);
  }
  for (Module m : module.children()) {
    SwapFunctionalLinear(m);
  }
}

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `graph`, `SwapFunctionalLinear`, `children`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`graph`, `SwapFunctionalLinear`, `children`。

### Lines 112-116
```cpp
void SwapFunctionalLinear(std::shared_ptr<Graph>& graph) {
  std::string functional_linear = R"(
graph(%linear, %input, %weight, %bias):
  %r = prim::CallFunction(%linear, %input, %weight, %bias)
  return (%r) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `SwapFunctionalLinear`, `graph`, `CallFunction`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`SwapFunctionalLinear`, `graph`, `CallFunction`。

### Lines 117-121
```cpp
  std::string aten_linear = R"(
graph(%linear, %input, %weight, %bias):
  %r = aten::linear(%input, %weight, %bias)
  return (%r) )";

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `linear`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `linear`。

### Lines 122-126
```cpp
  auto filter = [](const Match& match,
                   const std::unordered_map<std::string, Value*>& vmap) {
    const auto& match_vmap = match.values_map;
    auto linear = graph_rewrite_helper::getValue("linear", match_vmap, vmap);
    auto func_name = graph_rewrite_helper::getFuncName(linear);
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `getValue`, `getFuncName`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`getValue`, `getFuncName`。

### Lines 127-133
```cpp
    return func_name == "linear";
  };
  SubgraphRewriter rewriter;
  rewriter.RegisterRewritePattern(functional_linear, aten_linear);
  rewriter.runOnGraph(graph, filter);
}

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `RegisterRewritePattern`, `runOnGraph`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`RegisterRewritePattern`, `runOnGraph`。

### Lines 134-134
```cpp
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
- EN: Subsystem integration — the implementation collaborates with nearby JIT/frontend headers and runtime helpers.
  CN: 子系统集成——实现会与附近的 JIT/前端头文件以及运行时辅助模块协同工作。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `torch/csrc/jit/passes/fuse_linear.h`, `torch/csrc/jit/passes/graph_rewrite_helper.h`, `torch/csrc/jit/passes/quantization/helper.h`, `torch/csrc/jit/passes/subgraph_rewrite.h`
- External includes / 外部头文件: 无
- Namespaces / 命名空间: `torch::jit`
- Representative symbols / 代表性符号: `FuseLinear`, `graph`, `addmm`, `t`, `linear`, `is_int_constant`, `node`, `kind`, `aten`, `value_mappings`, `...`
