# xnnpack_rewrite.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/passes/xnnpack_rewrite.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements the TorchScript JIT pass logic for xnnpack rewrite, including graph analysis and rewrites.
- 用途 (CN): 实现与 xnnpack rewrite 相关的 TorchScript JIT pass 逻辑，包括图分析与改写。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14
```cpp
#include <ATen/core/jit_type.h>
#include <ATen/native/xnnpack/OpContext.h>

#include <torch/csrc/jit/ir/ir.h>
#include <torch/csrc/jit/passes/constant_propagation.h>
#include <torch/csrc/jit/passes/fold_conv_bn.h>
#include <torch/csrc/jit/passes/freeze_module.h>
#include <torch/csrc/jit/passes/fuse_linear.h>
#include <torch/csrc/jit/passes/fuse_relu.h>
#include <torch/csrc/jit/passes/graph_rewrite_helper.h>
#include <torch/csrc/jit/passes/hoist_conv_packed_params.h>
#include <torch/csrc/jit/passes/mobile_optimizer_type.h>
#include <torch/csrc/jit/passes/prepack_folding.h>
#include <torch/csrc/jit/passes/remove_dropout.h>
```
- EN: Pulls in the headers needed by the xnnpack rewrite logic. Internal dependencies: `ATen/core/jit_type.h`, `ATen/native/xnnpack/OpContext.h`, `torch/csrc/jit/ir/ir.h`, `torch/csrc/jit/passes/constant_propagation.h`, `torch/csrc/jit/passes/fold_conv_bn.h`, `...`; external dependencies: none.
- CN: 为 xnnpack rewrite 相关逻辑引入所需头文件。内部依赖：`ATen/core/jit_type.h`, `ATen/native/xnnpack/OpContext.h`, `torch/csrc/jit/ir/ir.h`, `torch/csrc/jit/passes/constant_propagation.h`, `torch/csrc/jit/passes/fold_conv_bn.h`, `...`；外部依赖：无。

### Lines 15-21
```cpp
#include <torch/csrc/jit/passes/subgraph_rewrite.h>
#include <torch/csrc/jit/passes/xnnpack_rewrite.h>
#include <torch/csrc/jit/runtime/graph_executor_impl.h>

namespace torch::jit {

namespace {
```
- EN: This block manipulates TorchScript IR objects. Key symbols: no dominant local symbol names.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：无明显局部符号。

### Lines 22-28
```cpp

void replaceConv1dWithConv2d(std::shared_ptr<Graph>& graph) {
  std::string conv_1d_pattern = R"(
    graph(%input, %weight, %bias, %stride:int[], %padding:int[], %dilation:int[], %groups:int):
        %res = aten::conv1d(%input, %weight, %bias, %stride, %padding, %dilation, %groups)
        return (%res) )";

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `replaceConv1dWithConv2d`, `graph`, `conv1d`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`replaceConv1dWithConv2d`, `graph`, `conv1d`。

### Lines 29-42
```cpp
  std::string conv_2d_pattern = R"(
    graph(%input, %weight, %bias, %stride:int[], %padding:int[], %dilation:int[], %groups:int):
        %zero : int = prim::Constant[value=0]()
        %one : int = prim::Constant[value=1]()
        %stride_w : int = prim::ListUnpack(%stride)
        %stride_2d : int[] = prim::ListConstruct(%one, %stride_w)
        %padding_w : int = prim::ListUnpack(%padding)
        %padding_2d : int[] = prim::ListConstruct(%zero, %padding_w)
        %dilation_w : int = prim::ListUnpack(%dilation)
        %dilation_2d : int[] = prim::ListConstruct(%one, %dilation_w)
        %two : int = prim::Constant[value=2]()
        %input_2d : Tensor = aten::unsqueeze(%input, %two)
        %weight_2d : Tensor = aten::unsqueeze(%weight, %two)
        %output_2d = aten::conv2d(
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `graph`, `ListUnpack`, `ListConstruct`, `unsqueeze`, `conv2d`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`graph`, `ListUnpack`, `ListConstruct`, `unsqueeze`, `conv2d`。

### Lines 43-56
```cpp
            %input_2d, %weight_2d, %bias, %stride_2d, %padding_2d, %dilation_2d, %groups)
        %output : Tensor = aten::squeeze(%output_2d, %two)
        return (%output) )";

  std::vector<std::pair<std::string, std::string>> value_mappings(
      {{"zero", "res"},
       {"one", "res"},
       {"stride_w", "res"},
       {"stride_2d", "res"},
       {"padding_w", "res"},
       {"padding_2d", "res"},
       {"dilation_w", "res"},
       {"dilation_2d", "res"},
       {"two", "res"},
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `squeeze`, `value_mappings`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`squeeze`, `value_mappings`。

### Lines 57-67
```cpp
       {"input_2d", "res"},
       {"weight_2d", "res"},
       {"output_2d", "res"},
       {"output", "res"}});

  SubgraphRewriter rewriter;
  rewriter.RegisterRewritePattern(
      conv_1d_pattern, conv_2d_pattern, value_mappings);
  rewriter.runOnGraph(graph);
}

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `RegisterRewritePattern`, `runOnGraph`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`RegisterRewritePattern`, `runOnGraph`。

### Lines 68-75
```cpp
} // namespace

void transformConv1dToConv2d(std::shared_ptr<Graph>& graph) {
  // Replace _convolution with conv1d and conv2d
  graph_rewrite_helper::replaceConvolutionWithAtenConv(graph);
  replaceConv1dWithConv2d(graph);
}

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `transformConv1dToConv2d`, `replaceConvolutionWithAtenConv`, `replaceConv1dWithConv2d`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`transformConv1dToConv2d`, `replaceConvolutionWithAtenConv`, `replaceConv1dWithConv2d`。

### Lines 76-85
```cpp
void transformConv1dToConv2d(script::Module& module) {
  for (auto& method : module.get_methods()) {
    auto graph = method.graph();
    transformConv1dToConv2d(graph);
  }
  for (script::Module m : module.children()) {
    transformConv1dToConv2d(m);
  }
}

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `transformConv1dToConv2d`, `get_methods`, `graph`, `children`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`transformConv1dToConv2d`, `get_methods`, `graph`, `children`。

### Lines 86-93
```cpp
#ifdef USE_XNNPACK

namespace {

void insertPrePackedLinearOp(std::shared_ptr<Graph>& graph) {
  // fuse decomposed linear into aten::linear
  FuseLinear(graph);

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `insertPrePackedLinearOp`, `FuseLinear`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`insertPrePackedLinearOp`, `FuseLinear`。

### Lines 94-104
```cpp
  std::string linear_pattern = R"(
    graph(%input, %weight, %bias):
        %res = aten::linear(%input, %weight, %bias)
        return (%res))";
  std::string prepacked_ops_pattern = R"(
    graph(%input, %weight, %bias):
        %output_min_max : None = prim::Constant()
        %packed_weight_bias = prepacked::linear_clamp_prepack(
            %weight, %bias, %output_min_max, %output_min_max)
        %res = prepacked::linear_clamp_run(%input, %packed_weight_bias)
        return (%res))";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `graph`, `linear`, `Constant`, `linear_clamp_prepack`, `linear_clamp_run`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`graph`, `linear`, `Constant`, `linear_clamp_prepack`, `linear_clamp_run`。

### Lines 105-116
```cpp

  std::vector<std::pair<std::string, std::string>> value_mappings(
      {{"output_min_max", "res"},
       {"packed_weight_bias", "res"},
       {"res", "res"}});

  SubgraphRewriter linear_rewriter;
  linear_rewriter.RegisterRewritePattern(
      linear_pattern, prepacked_ops_pattern, value_mappings);
  linear_rewriter.runOnGraph(graph);
}

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `value_mappings`, `RegisterRewritePattern`, `runOnGraph`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`value_mappings`, `RegisterRewritePattern`, `runOnGraph`。

### Lines 117-124
```cpp
void insertPrePackedConv2dOp(std::shared_ptr<Graph>& graph) {
  // Replace _convolution with conv2d
  graph_rewrite_helper::replaceConvolutionWithAtenConv(graph);

  std::string conv_2d_pattern = R"(
    graph(%input, %weight, %bias, %stride:int[], %padding:int[], %dilation:int[], %groups:int):
        %res = aten::conv2d(%input, %weight, %bias, %stride, %padding, %dilation, %groups)
        return (%res) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `insertPrePackedConv2dOp`, `replaceConvolutionWithAtenConv`, `graph`, `conv2d`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`insertPrePackedConv2dOp`, `replaceConvolutionWithAtenConv`, `graph`, `conv2d`。

### Lines 125-133
```cpp

  std::string prepacked_ops_conv2d_pattern = R"(
    graph(%input, %weight, %bias, %stride:int[], %padding:int[], %dilation:int[], %groups:int):
        %output_min_max : None = prim::Constant()
        %packed_weight_bias = prepacked::conv2d_clamp_prepack(
            %weight, %bias, %stride, %padding, %dilation, %groups,
            %output_min_max, %output_min_max)
        %res = prepacked::conv2d_clamp_run(%input, %packed_weight_bias)
        return (%res) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `graph`, `Constant`, `conv2d_clamp_prepack`, `conv2d_clamp_run`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`graph`, `Constant`, `conv2d_clamp_prepack`, `conv2d_clamp_run`。

### Lines 134-144
```cpp

  std::vector<std::pair<std::string, std::string>> value_mappings(
      {{"output_min_max", "res"},
       {"packed_weight_bias", "res"},
       {"res", "res"}});

  SubgraphRewriter rewriter;
  rewriter.RegisterRewritePattern(
      conv_2d_pattern, prepacked_ops_conv2d_pattern, value_mappings);
  rewriter.runOnGraph(graph);

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `value_mappings`, `RegisterRewritePattern`, `runOnGraph`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`value_mappings`, `RegisterRewritePattern`, `runOnGraph`。

### Lines 145-151
```cpp
  std::string conv_2d_transpose_pattern = R"(
      graph(%input, %weight, %bias, %stride:int[], %padding:int[], %dilation:int[],
          %output_padding:int[], %groups:int):
        %res = aten::conv_transpose2d(%input, %weight, %bias, %stride, %padding, %output_padding, %groups, %dilation)
        return (%res) )";

  std::string prepacked_ops_conv2d_transpose_pattern = R"(
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `conv_transpose2d`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `conv_transpose2d`。

### Lines 152-158
```cpp
    graph(%input, %weight, %bias, %stride:int[], %padding:int[], %dilation:int[], %output_padding:int[], %groups:int):
        %output_min_max : None = prim::Constant()
        %packed_weight_bias = prepacked::conv2d_transpose_clamp_prepack(
            %weight, %bias, %stride, %padding, %output_padding, %dilation, %groups,
            %output_min_max, %output_min_max)
        %res = prepacked::conv2d_transpose_clamp_run(%input, %packed_weight_bias)
        return (%res) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `graph`, `Constant`, `conv2d_transpose_clamp_prepack`, `conv2d_transpose_clamp_run`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`graph`, `Constant`, `conv2d_transpose_clamp_prepack`, `conv2d_transpose_clamp_run`。

### Lines 159-170
```cpp

  value_mappings = {
      {"output_min_max", "res"}, {"packed_weight_bias", "res"}, {"res", "res"}};

  SubgraphRewriter transpose_rewriter;
  transpose_rewriter.RegisterRewritePattern(
      conv_2d_transpose_pattern,
      prepacked_ops_conv2d_transpose_pattern,
      value_mappings);
  transpose_rewriter.runOnGraph(graph);
}

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `RegisterRewritePattern`, `runOnGraph`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`RegisterRewritePattern`, `runOnGraph`。

### Lines 171-179
```cpp
void fuseHardtanhWithPackedOps(std::shared_ptr<Graph>& graph) {
  SubgraphRewriter rewriter;

  std::string linear_prepack_run_hardtanh_fused = R"(
    graph(%input, %weight, %bias, %output_min, %output_max, %dummy_min_max):
        %packed_weight_bias : __torch__.torch.classes.xnnpack.LinearOpContext = prepacked::linear_clamp_prepack(
            %weight, %bias, %output_min, %output_max)
        %res = prepacked::linear_clamp_run(%input, %packed_weight_bias)
        return (%res))";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `fuseHardtanhWithPackedOps`, `graph`, `linear_clamp_prepack`, `linear_clamp_run`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`fuseHardtanhWithPackedOps`, `graph`, `linear_clamp_prepack`, `linear_clamp_run`。

### Lines 180-188
```cpp

  std::string conv2d_prepack_run_hardtanh_fused = R"(
    graph(%input, %weight, %bias, %stride:int[], %padding:int[],
          %dilation:int[], %groups:int, %output_min, %output_max, %dummy_min_max):
        %packed_weight_bias : __torch__.torch.classes.xnnpack.Conv2dOpContext = prepacked::conv2d_clamp_prepack(
            %weight, %bias, %stride, %padding, %dilation, %groups,
            %output_min, %output_max)
        %res = prepacked::conv2d_clamp_run(%input, %packed_weight_bias)
        return (%res) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `conv2d_clamp_prepack`, `conv2d_clamp_run`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `conv2d_clamp_prepack`, `conv2d_clamp_run`。

### Lines 189-196
```cpp

  std::string linear_prepack_run_hardtanh = R"(
    graph(%input, %weight, %bias, %output_min, %output_max, %dummy_min_max):
        %packed_weight_bias = prepacked::linear_clamp_prepack(
            %weight, %bias, %dummy_min_max, %dummy_min_max)
        %linear_res = prepacked::linear_clamp_run(%input, %packed_weight_bias)
        %res = aten::hardtanh(%linear_res, %output_min, %output_max)
        return (%res))";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `linear_clamp_prepack`, `linear_clamp_run`, `hardtanh`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `linear_clamp_prepack`, `linear_clamp_run`, `hardtanh`。

### Lines 197-205
```cpp

  std::vector<std::pair<std::string, std::string>> value_mappings(
      {{"packed_weight_bias", "packed_weight_bias"}, {"res", "res"}});

  rewriter.RegisterRewritePattern(
      linear_prepack_run_hardtanh,
      linear_prepack_run_hardtanh_fused,
      value_mappings);

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `value_mappings`, `RegisterRewritePattern`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`value_mappings`, `RegisterRewritePattern`。

### Lines 206-214
```cpp
  std::string conv2d_prepack_run_hardtanh = R"(
    graph(%input, %weight, %bias, %stride:int[], %padding:int[],
          %dilation:int[], %groups:int, %output_min, %output_max, %dummy_min_max):
        %packed_weight_bias = prepacked::conv2d_clamp_prepack(
            %weight, %bias, %stride, %padding, %dilation, %groups,
            %dummy_min_max, %dummy_min_max)
        %conv2d_res = prepacked::conv2d_clamp_run(%input, %packed_weight_bias)
        %res = aten::hardtanh(%conv2d_res, %output_min, %output_max)
        return (%res) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `conv2d_clamp_prepack`, `conv2d_clamp_run`, `hardtanh`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `conv2d_clamp_prepack`, `conv2d_clamp_run`, `hardtanh`。

### Lines 215-223
```cpp

  value_mappings = {
      {"packed_weight_bias", "packed_weight_bias"}, {"res", "res"}};

  rewriter.RegisterRewritePattern(
      conv2d_prepack_run_hardtanh,
      conv2d_prepack_run_hardtanh_fused,
      value_mappings);

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `RegisterRewritePattern`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`RegisterRewritePattern`。

### Lines 224-230
```cpp
  std::string linear_prepack_run_hardtanh_inplace = R"(
    graph(%input, %weight, %bias, %output_min, %output_max, %dummy_min_max):
        %packed_weight_bias = prepacked::linear_clamp_prepack(
            %weight, %bias, %dummy_min_max, %dummy_min_max)
        %linear_res = prepacked::linear_clamp_run(%input, %packed_weight_bias)
        %res = aten::hardtanh_(%linear_res, %output_min, %output_max)
        return (%res))";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `linear_clamp_prepack`, `linear_clamp_run`, `hardtanh_`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `linear_clamp_prepack`, `linear_clamp_run`, `hardtanh_`。

### Lines 231-240
```cpp

  std::string conv2d_prepack_run_hardtanh_inplace = R"(
    graph(%input, %weight, %bias, %stride:int[], %padding:int[],
          %dilation:int[], %groups:int, %output_min, %output_max, %dummy_min_max):
        %packed_weight_bias = prepacked::conv2d_clamp_prepack(
            %weight, %bias, %stride, %padding, %dilation, %groups,
            %dummy_min_max, %dummy_min_max)
        %conv2d_res = prepacked::conv2d_clamp_run(%input, %packed_weight_bias)
        %res = aten::hardtanh_(%conv2d_res, %output_min, %output_max)
        return (%res) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `conv2d_clamp_prepack`, `conv2d_clamp_run`, `hardtanh_`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `conv2d_clamp_prepack`, `conv2d_clamp_run`, `hardtanh_`。

### Lines 241-249
```cpp

  value_mappings = {
      {"packed_weight_bias", "packed_weight_bias"}, {"res", "res"}};

  rewriter.RegisterRewritePattern(
      linear_prepack_run_hardtanh_inplace,
      linear_prepack_run_hardtanh_fused,
      value_mappings);

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `RegisterRewritePattern`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`RegisterRewritePattern`。

### Lines 250-257
```cpp
  value_mappings = {
      {"packed_weight_bias", "packed_weight_bias"}, {"res", "res"}};

  rewriter.RegisterRewritePattern(
      conv2d_prepack_run_hardtanh_inplace,
      conv2d_prepack_run_hardtanh_fused,
      value_mappings);

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `RegisterRewritePattern`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`RegisterRewritePattern`。

### Lines 258-264
```cpp
  rewriter.runOnGraph(graph, torch::jit::graph_rewrite_helper::isClampFusable);
}

void fuseReluWithPackedOps(std::shared_ptr<Graph>& graph) {
  SubgraphRewriter rewriter;

  std::string linear_prepack_run_relu_fused = R"(
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `runOnGraph`, `fuseReluWithPackedOps`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`runOnGraph`, `fuseReluWithPackedOps`。

### Lines 265-271
```cpp
    graph(%input, %weight, %bias, %dummy_min_max):
        %output_min: float = prim::Constant[value=0.0]()
        %output_max: None = prim::Constant()
        %packed_weight_bias : __torch__.torch.classes.xnnpack.LinearOpContext = prepacked::linear_clamp_prepack(
            %weight, %bias, %output_min, %output_max)
        %res = prepacked::linear_clamp_run(%input, %packed_weight_bias)
        return (%res))";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `graph`, `Constant`, `linear_clamp_prepack`, `linear_clamp_run`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`graph`, `Constant`, `linear_clamp_prepack`, `linear_clamp_run`。

### Lines 272-282
```cpp

  std::string conv2d_prepack_run_relu_fused = R"(
    graph(%input, %weight, %bias, %stride:int[], %padding:int[],
          %dilation:int[], %groups:int, %dummy_min_max):
        %output_min: float = prim::Constant[value=0.0]()
        %output_max: None = prim::Constant()
        %packed_weight_bias : __torch__.torch.classes.xnnpack.Conv2dOpContext = prepacked::conv2d_clamp_prepack(
            %weight, %bias, %stride, %padding, %dilation, %groups,
            %output_min, %output_max)
        %res = prepacked::conv2d_clamp_run(%input, %packed_weight_bias)
        return (%res) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `graph`, `Constant`, `conv2d_clamp_prepack`, `conv2d_clamp_run`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`graph`, `Constant`, `conv2d_clamp_prepack`, `conv2d_clamp_run`。

### Lines 283-290
```cpp

  std::string linear_prepack_run_relu = R"(
    graph(%input, %weight, %bias, %dummy_min_max):
        %packed_weight_bias = prepacked::linear_clamp_prepack(
            %weight, %bias, %dummy_min_max, %dummy_min_max)
        %linear_res = prepacked::linear_clamp_run(%input, %packed_weight_bias)
        %res = aten::relu(%linear_res)
        return (%res))";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `linear_clamp_prepack`, `linear_clamp_run`, `relu`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `linear_clamp_prepack`, `linear_clamp_run`, `relu`。

### Lines 291-297
```cpp

  std::vector<std::pair<std::string, std::string>> value_mappings(
      {{"output_min", "packed_weight_bias"},
       {"output_max", "packed_weight_bias"},
       {"packed_weight_bias", "packed_weight_bias"},
       {"res", "res"}});

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `value_mappings`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`value_mappings`。

### Lines 298-309
```cpp
  rewriter.RegisterRewritePattern(
      linear_prepack_run_relu, linear_prepack_run_relu_fused, value_mappings);

  std::string conv2d_prepack_run_relu = R"(
    graph(%input, %weight, %bias, %stride:int[], %padding:int[],
          %dilation:int[], %groups:int, %dummy_min_max):
        %packed_weight_bias = prepacked::conv2d_clamp_prepack(
            %weight, %bias, %stride, %padding, %dilation, %groups,
            %dummy_min_max, %dummy_min_max)
        %conv2d_res = prepacked::conv2d_clamp_run(%input, %packed_weight_bias)
        %res = aten::relu(%conv2d_res)
        return (%res) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `RegisterRewritePattern`, `graph`, `conv2d_clamp_prepack`, `conv2d_clamp_run`, `relu`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`RegisterRewritePattern`, `graph`, `conv2d_clamp_prepack`, `conv2d_clamp_run`, `relu`。

### Lines 310-316
```cpp

  value_mappings = {
      {"output_min", "packed_weight_bias"},
      {"output_max", "packed_weight_bias"},
      {"packed_weight_bias", "packed_weight_bias"},
      {"res", "res"}};

```
- EN: This block manipulates TorchScript IR objects. Key symbols: no dominant local symbol names.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：无明显局部符号。

### Lines 317-326
```cpp
  rewriter.RegisterRewritePattern(
      conv2d_prepack_run_relu, conv2d_prepack_run_relu_fused, value_mappings);

  std::string linear_prepack_run_relu_inplace = R"(
    graph(%input, %weight, %bias, %dummy_min_max):
        %packed_weight_bias = prepacked::linear_clamp_prepack(
            %weight, %bias, %dummy_min_max, %dummy_min_max)
        %linear_res = prepacked::linear_clamp_run(%input, %packed_weight_bias)
        %res = aten::relu_(%linear_res)
        return (%res))";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `RegisterRewritePattern`, `graph`, `linear_clamp_prepack`, `linear_clamp_run`, `relu_`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`RegisterRewritePattern`, `graph`, `linear_clamp_prepack`, `linear_clamp_run`, `relu_`。

### Lines 327-336
```cpp

  std::string conv2d_prepack_run_relu_inplace = R"(
    graph(%input, %weight, %bias, %stride:int[], %padding:int[],
          %dilation:int[], %groups:int, %dummy_min_max):
        %packed_weight_bias = prepacked::conv2d_clamp_prepack(
            %weight, %bias, %stride, %padding, %dilation, %groups,
            %dummy_min_max, %dummy_min_max)
        %conv2d_res = prepacked::conv2d_clamp_run(%input, %packed_weight_bias)
        %res = aten::relu_(%conv2d_res)
        return (%res) )";
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `conv2d_clamp_prepack`, `conv2d_clamp_run`, `relu_`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `conv2d_clamp_prepack`, `conv2d_clamp_run`, `relu_`。

### Lines 337-343
```cpp

  value_mappings = {
      {"output_min", "packed_weight_bias"},
      {"output_max", "packed_weight_bias"},
      {"packed_weight_bias", "packed_weight_bias"},
      {"res", "res"}};

```
- EN: This block manipulates TorchScript IR objects. Key symbols: no dominant local symbol names.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：无明显局部符号。

### Lines 344-354
```cpp
  rewriter.RegisterRewritePattern(
      linear_prepack_run_relu_inplace,
      linear_prepack_run_relu_fused,
      value_mappings);

  value_mappings = {
      {"output_min", "packed_weight_bias"},
      {"output_max", "packed_weight_bias"},
      {"packed_weight_bias", "packed_weight_bias"},
      {"res", "res"}};

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `RegisterRewritePattern`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`RegisterRewritePattern`。

### Lines 355-361
```cpp
  rewriter.RegisterRewritePattern(
      conv2d_prepack_run_relu_inplace,
      conv2d_prepack_run_relu_fused,
      value_mappings);
  rewriter.runOnGraph(graph, torch::jit::graph_rewrite_helper::isClampFusable);
}

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `RegisterRewritePattern`, `runOnGraph`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`RegisterRewritePattern`, `runOnGraph`。

### Lines 362-370
```cpp
void runCanonicalOptimizations(script::Module& module) {
  for (const auto& method : module.get_methods()) {
    auto graph = method.graph();
    // Not sure if we have models running on mobile that require loop unrolling.
    // Perhaps language/speech models? Conservatively setting that to false.
    runOptimization(graph, false /* no loop unrolling */);
  }
}

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `runCanonicalOptimizations`, `get_methods`, `graph`, `runOptimization`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`runCanonicalOptimizations`, `get_methods`, `graph`, `runOptimization`。

### Lines 371-377
```cpp
} // namespace

void insertPrePackedOps(std::shared_ptr<Graph>& graph) {
  insertPrePackedLinearOp(graph);
  insertPrePackedConv2dOp(graph);
}

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `insertPrePackedOps`, `insertPrePackedLinearOp`, `insertPrePackedConv2dOp`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`insertPrePackedOps`, `insertPrePackedLinearOp`, `insertPrePackedConv2dOp`。

### Lines 378-387
```cpp
void insertPrePackedOps(script::Module& module) {
  for (auto& method : module.get_methods()) {
    auto graph = method.graph();
    insertPrePackedOps(graph);
  }
  for (script::Module m : module.children()) {
    insertPrePackedOps(m);
  }
}

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `insertPrePackedOps`, `get_methods`, `graph`, `children`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`insertPrePackedOps`, `get_methods`, `graph`, `children`。

### Lines 388-398
```cpp
void fusePrePackedLinearConvWithClamp(script::Module& module) {
  for (auto& method : module.get_methods()) {
    auto graph = method.graph();
    fuseReluWithPackedOps(graph);
    fuseHardtanhWithPackedOps(graph);

    // Ignore user defined classes for later passes
    ConstantPropagation(graph, true);
  }
}

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `fusePrePackedLinearConvWithClamp`, `get_methods`, `graph`, `fuseReluWithPackedOps`, `fuseHardtanhWithPackedOps`, `ConstantPropagation`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`fusePrePackedLinearConvWithClamp`, `get_methods`, `graph`, `fuseReluWithPackedOps`, `fuseHardtanhWithPackedOps`, `ConstantPropagation`。

### Lines 399-409
```cpp
void FoldPrePackingOps(script::Module& m) {
  PrePackingOpsFilterFn filter_fn = [](const Node* n) -> bool {
    return (
        (n->kind() ==
         Symbol::fromQualString("prepacked::linear_clamp_prepack")) ||
        n->kind() ==
            Symbol::fromQualString("prepacked::conv2d_clamp_prepack") ||
        n->kind() ==
            Symbol::fromQualString(
                "prepacked::conv2d_transpose_clamp_prepack"));
  };
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `FoldPrePackingOps`, `kind`, `fromQualString`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`FoldPrePackingOps`, `kind`, `fromQualString`。

### Lines 410-417
```cpp
  PrePackingOpsFolder(m, filter_fn, "prepack_folding");
  for (auto& method : m.get_methods()) {
    auto graph = method.graph();
    // Folding requires a const propagation through user defined classes
    ConstantPropagation(graph, false);
  }
}

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `PrePackingOpsFolder`, `get_methods`, `graph`, `ConstantPropagation`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`PrePackingOpsFolder`, `get_methods`, `graph`, `ConstantPropagation`。

### Lines 418-424
```cpp
script::Module optimizeForMobile(
    const script::Module& m,
    const std::set<MobileOptimizerType>& optimization_blocklist,
    const std::vector<std::string>& preserved_methods) {
  auto cloned_module = m.clone();
  cloned_module.eval();

```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `optimizeForMobile`, `clone`, `eval`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`optimizeForMobile`, `clone`, `eval`。

### Lines 425-432
```cpp
  if (!optimization_blocklist.count(MobileOptimizerType::CONV_1D_TO_2D)) {
    transformConv1dToConv2d(cloned_module);
  }

  if (!optimization_blocklist.count(MobileOptimizerType::CONV_BN_FUSION)) {
    cloned_module = FoldConvBatchNorm(cloned_module);
  }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `count`, `transformConv1dToConv2d`, `FoldConvBatchNorm`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`count`, `transformConv1dToConv2d`, `FoldConvBatchNorm`。

### Lines 433-446
```cpp
  // Many optimizations require a frozen module, but ConvBatchNorm requires
  // an unfrozen module
  cloned_module = freeze_module(cloned_module, preserved_methods);

  if (!optimization_blocklist.count(
          MobileOptimizerType::INSERT_FOLD_PREPACK_OPS)) {
    // TODO fix duplication caused by referencing same op across multiple
    // functions
    insertPrePackedOps(cloned_module);
    cloned_module = freeze_module(cloned_module, preserved_methods);
    fusePrePackedLinearConvWithClamp(cloned_module);
    FoldPrePackingOps(cloned_module);
  }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `freeze_module`, `count`, `insertPrePackedOps`, `fusePrePackedLinearConvWithClamp`, `FoldPrePackingOps`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`freeze_module`, `count`, `insertPrePackedOps`, `fusePrePackedLinearConvWithClamp`, `FoldPrePackingOps`。

### Lines 447-456
```cpp
  if (!optimization_blocklist.count(
          MobileOptimizerType::HOIST_CONV_PACKED_PARAMS) &&
      cloned_module.find_method("forward")) {
    // freeze again in case it was not done in previous optional passes
    cloned_module = freeze_module(cloned_module, preserved_methods);
    HoistConvPackedParams(cloned_module);
    // and freeze yet again to remove the empty QuantizedConv modules
    cloned_module = freeze_module(cloned_module, preserved_methods);
  }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `count`, `find_method`, `freeze_module`, `HoistConvPackedParams`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`count`, `find_method`, `freeze_module`, `HoistConvPackedParams`。

### Lines 457-463
```cpp
  // Run canonical optimizations post freezing
  // since freezing inlines the graph. Otherwise we
  // will have to explicitly call Inlining pass.
  runCanonicalOptimizations(cloned_module);

  if (!optimization_blocklist.count(MobileOptimizerType::REMOVE_DROPOUT)) {
    for (const auto& method : cloned_module.get_methods()) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `runCanonicalOptimizations`, `count`, `get_methods`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`runCanonicalOptimizations`, `count`, `get_methods`。

### Lines 464-470
```cpp
      auto graph = method.graph();
      // Module must be not be in training mode but optimize calls eval()
      removeDropout(graph);
    }
  }

  if (!optimization_blocklist.count(MobileOptimizerType::FUSE_ADD_RELU)) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `graph`, `removeDropout`, `count`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`graph`, `removeDropout`, `count`。

### Lines 471-477
```cpp
    for (const auto& method : cloned_module.get_methods()) {
      auto graph = method.graph();
      FuseAddRelu(graph);
    }
  }
  cloned_module.register_attribute("mobile_optimized", BoolType::get(), true);
  return cloned_module;
```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `get_methods`, `graph`, `FuseAddRelu`, `register_attribute`, `get`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`get_methods`, `graph`, `FuseAddRelu`, `register_attribute`, `get`。

### Lines 478-486
```cpp
}

#else

void insertPrePackedOps(std::shared_ptr<Graph>& graph) {
  TORCH_INTERNAL_ASSERT(
      false, "XNNPACK is not enabled. Please build with USE_XNNPACK=1");
}

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `insertPrePackedOps`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`insertPrePackedOps`。

### Lines 487-496
```cpp
void insertPrePackedOps(script::Module& module) {
  TORCH_INTERNAL_ASSERT(
      false, "XNNPACK is not enabled. Please build with USE_XNNPACK=1");
}

void fusePrePackedLinearConvWithClamp(script::Module& module) {
  TORCH_INTERNAL_ASSERT(
      false, "XNNPACK is not enabled. Please build with USE_XNNPACK=1");
}

```
- EN: This block implements local helper logic for xnnpack rewrite. Key symbols: `insertPrePackedOps`, `fusePrePackedLinearConvWithClamp`.
- CN: 该代码块实现与 xnnpack rewrite 相关的局部辅助逻辑。关键符号：`insertPrePackedOps`, `fusePrePackedLinearConvWithClamp`。

### Lines 497-510
```cpp
void FoldPrePackingOps(script::Module& m) {
  TORCH_INTERNAL_ASSERT(
      false, "XNNPACK is not enabled. Please build with USE_XNNPACK=1");
}

script::Module optimizeForMobile(
    const script::Module& module,
    const std::set<MobileOptimizerType>& blocklist,
    const std::vector<std::string>& preserved_methods) {
  TORCH_INTERNAL_ASSERT(
      false,
      "Mobile optimization only available with XNNPACK at the moment. "
      "XNNPACK is not enabled. Please build with USE_XNNPACK=1");
  return module;
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `FoldPrePackingOps`, `optimizeForMobile`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`FoldPrePackingOps`, `optimizeForMobile`。

### Lines 511-514
```cpp
}

#endif
} // namespace torch::jit
```
- EN: Opens or closes namespace scopes so the remaining declarations live in the expected JIT/frontend module.
- CN: 打开或关闭命名空间作用域，使后续声明归属于预期的 JIT/前端模块。


## Key Concepts / 关键概念
- EN: IR graph manipulation — the file inspects or rewrites TorchScript graphs and nodes.
  CN: IR 图操作——该文件会检查或改写 TorchScript 的图、节点与块。
- EN: Optimization pass — the code simplifies graphs by removing redundant work or folding values.
  CN: 优化 pass——代码通过删除冗余计算或常量折叠来简化图。
- EN: Inlining — the implementation merges callees or blocks into their call sites.
  CN: 内联——实现会把被调用者或代码块并入调用位置。
- EN: Fusion support — the file prepares groups of operations for fused execution.
  CN: 融合支持——该文件会为融合执行准备算子分组。
- EN: Type and shape reasoning — the code propagates static information across the IR.
  CN: 类型与形状推理——代码会在 IR 中传播静态信息。
- EN: Subsystem integration — the implementation collaborates with nearby JIT/frontend headers and runtime helpers.
  CN: 子系统集成——实现会与附近的 JIT/前端头文件以及运行时辅助模块协同工作。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `ATen/core/jit_type.h`, `ATen/native/xnnpack/OpContext.h`, `torch/csrc/jit/ir/ir.h`, `torch/csrc/jit/passes/constant_propagation.h`, `torch/csrc/jit/passes/fold_conv_bn.h`, `torch/csrc/jit/passes/freeze_module.h`, `torch/csrc/jit/passes/fuse_linear.h`, `torch/csrc/jit/passes/fuse_relu.h`, `torch/csrc/jit/passes/graph_rewrite_helper.h`, `torch/csrc/jit/passes/hoist_conv_packed_params.h`, `...`
- External includes / 外部头文件: 无
- Namespaces / 命名空间: `torch::jit`
- Representative symbols / 代表性符号: `replaceConv1dWithConv2d`, `graph`, `conv1d`, `ListUnpack`, `ListConstruct`, `unsqueeze`, `conv2d`, `squeeze`, `value_mappings`, `RegisterRewritePattern`, `...`
