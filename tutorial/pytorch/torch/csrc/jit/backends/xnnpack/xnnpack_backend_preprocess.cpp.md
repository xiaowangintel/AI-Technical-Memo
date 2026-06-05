# xnnpack_backend_preprocess.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/csrc/jit/backends/xnnpack/xnnpack_backend_preprocess.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements the extension points, compilation flow, and runtime helpers used by custom JIT backends. This specific file centers on `xnnpack_backend_preprocess.cpp`.
- **Purpose (CN)**: 实现自定义 JIT 后端所需的扩展点、编译流程和运行时辅助逻辑。 该文件具体围绕 `xnnpack_backend_preprocess.cpp` 展开。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

```cpp
#include <torch/csrc/jit/backends/backend.h>
#include <torch/csrc/jit/backends/backend_preprocess.h>

#include <torch/csrc/jit/tensorexpr/graph_opt.h>
#include <torch/torch.h>
#include <xnnpack.h>

#include <ATen/core/List.h>
#include <torch/csrc/jit/backends/xnnpack/xnnpack_graph_builder.h>

namespace torch::jit::xnnpack::delegate {

// Expected method_compile_spec should look something like this:
// {
//     "forward" : {"inputs" : at::Tensor}
// }
// or
// {
//     "forward" : {
//                  "inputs" : c10::List<at::Tensor>,
```

- **EN:** It enters or references namespace scopes such as torch::jit::xnnpack::delegate, which positions the code inside the JIT subsystem hierarchy.
- **CN:** 这里进入或引用了 torch::jit::xnnpack::delegate 等命名空间作用域，用于把代码放入 JIT 子系统层级中。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Concepts touched here: Graph IR / 图中间表示, Module API / 模块 API, Backend integration / 后端集成, Header composition / 头文件组织, Namespace scoping / 命名空间作用域.
- **CN:** 这里涉及的概念包括：Graph IR / 图中间表示, Module API / 模块 API, Backend integration / 后端集成, Header composition / 头文件组织, Namespace scoping / 命名空间作用域。

### Lines 21-40 / 第 21-40 行

```cpp
//                  "outputs" : c10::List<at::Tensor>
//                  }
// }
// in which the value for "inputs" is the input shape to the module.
// The module fed to the xnnpack backend must first be traced in order
// to propagate input shapes through the module. This is important
// for building the xnnpack_subgraph_t object.
c10::IValue preprocess(
    const Module& mod,
    const c10::Dict<c10::IValue, c10::IValue>& method_compile_spec,
    const BackendDebugHandleGenerator& generate_debug_handles) {
  auto eval_mod = mod.clone();
  eval_mod.eval();
  eval_mod = torch::jit::freeze(eval_mod);

  c10::Dict<IValue, IValue> compiled(StringType::get(), TensorType::get());

  c10::IValue inp;
  c10::IValue out;

```

- **EN:** Important callable entry points in this range include preprocess, compiled.
- **CN:** 这一段的重要可调用入口包括 preprocess, compiled。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Concepts touched here: Graph IR / 图中间表示, Nodes and values / 节点与值, Type system / 类型系统, Module API / 模块 API, Backend integration / 后端集成, Shape/resource guard / 形状或资源保护.
- **CN:** 这里涉及的概念包括：Graph IR / 图中间表示, Nodes and values / 节点与值, Type system / 类型系统, Module API / 模块 API, Backend integration / 后端集成, Shape/resource guard / 形状或资源保护。

### Lines 41-60 / 第 41-60 行

```cpp
  TORCH_CHECK(
      method_compile_spec.contains("forward"),
      "method_compile_spec does not contain the \"forward\" key.");
  auto innerDict = method_compile_spec.at("forward");

  TORCH_CHECK(
      innerDict.isGenericDict() &&
          innerDict.toGenericDict().contains("inputs") &&
          innerDict.toGenericDict().contains("outputs"),
      "method_compile_spec does not contain a dictionary with an \"inputs\" key, under \"forward\" key.");

  inp = innerDict.toGenericDict().at("inputs");
  out = innerDict.toGenericDict().at("outputs");

  TORCH_CHECK(
      inp.isTensor() || inp.isTensorList(),
      "method_compile_spec does not contain either a Tensor or TensorList, under it's \"inputs\" key.");
  TORCH_CHECK(
      out.isTensor() || out.isTensorList(),
      "method_compile_spec does not contain either a Tensor or TensorList, under it's \"outputs\" key.");
```

- **EN:** Important callable entry points in this range include TORCH_CHECK.
- **CN:** 这一段的重要可调用入口包括 TORCH_CHECK。
- **EN:** Concepts touched here: Module API / 模块 API, Backend integration / 后端集成, Declared symbols / 声明的符号.
- **CN:** 这里涉及的概念包括：Module API / 模块 API, Backend integration / 后端集成, Declared symbols / 声明的符号。

### Lines 61-80 / 第 61-80 行

```cpp

  // Graph preprocessing
  const auto& forward_method = eval_mod.get_method("forward");

  auto graph = toGraphFunction(forward_method.function()).graph()->copy();
  graph = tensorexpr::removeUnusedSelfArgument(graph);
  std::vector<c10::IValue> example_inputs;
  if (inp.isTensorList()) {
    c10::List<at::Tensor> inp_list = inp.toTensorList();
    TORCH_CHECK(
        graph->inputs().size() == inp_list.size(),
        "method_compile_spec inputs do not match expected number of forward inputs");

    example_inputs.reserve(inp_list.size());
    for (const auto i : c10::irange(inp_list.size())) {
      example_inputs.emplace_back(inp_list[i]);
    }
  } else {
    TORCH_CHECK(
        graph->inputs().size() == 1,
```

- **EN:** Important callable entry points in this range include TORCH_CHECK.
- **CN:** 这一段的重要可调用入口包括 TORCH_CHECK。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 81-100 / 第 81-100 行

```cpp
        "method_compile_spec inputs do not match expected number of forward inputs");

    example_inputs.emplace_back(inp.toTensor());
  }

  // inp above has been confirmed to be either Tensor or TensorList
  XNNGraph graph_builder;
  graph_builder.buildXNNGraph(graph, example_inputs);
  // at this point graph is complete, for the sake of testing preprocess at this
  // point we will do runtime setup and run with some default values

  // grabbing the inputs from compile spec for testing

  // gather sample inputs from compile spec
  std::vector<at::Tensor> inputs;
  auto input_list = inp.toList();

  for (int i = 0; i < input_list.size(); i++) {
    inputs.push_back(input_list.get(i).toTensor());
  }
```

- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Comments in this range document the intent of the API or the invariants expected by later code.
- **CN:** 这一段中的注释说明了 API 意图，或描述了后续代码依赖的不变式。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。
- **EN:** Concepts touched here: Graph IR / 图中间表示, Nodes and values / 节点与值, Module API / 模块 API, Backend integration / 后端集成, Iteration / 迭代处理.
- **CN:** 这里涉及的概念包括：Graph IR / 图中间表示, Nodes and values / 节点与值, Module API / 模块 API, Backend integration / 后端集成, Iteration / 迭代处理。

### Lines 101-120 / 第 101-120 行

```cpp
  std::vector<at::Tensor> outputs;
  auto output_list = out.toList();
  std::vector<c10::IntList> output_shapes;

  // gather sample outputs from compile spec
  for (int i = 0; i < output_list.size(); i++) {
    auto sample_output = output_list.get(i).toTensor();
    outputs.push_back(sample_output);
    // also gather output shapes to forward along to device
    output_shapes.push_back(sample_output.sizes());
  }

  // sample run on sample inputs
  graph_builder.runGraphOnInputs(inputs, outputs);
  c10::List<c10::IntList> shapes_list(output_shapes);

  compiled.insert("ser_model", graph_builder.serializedXNNGraph());
  compiled.insert("outputs", shapes_list);
  compiled.insert("Answer", outputs);

```

- **EN:** Important callable entry points in this range include shapes_list.
- **CN:** 这一段的重要可调用入口包括 shapes_list。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。
- **EN:** Concepts touched here: Graph IR / 图中间表示, Serialization / 序列化, Backend integration / 后端集成, Shape/resource guard / 形状或资源保护, Iteration / 迭代处理.
- **CN:** 这里涉及的概念包括：Graph IR / 图中间表示, Serialization / 序列化, Backend integration / 后端集成, Shape/resource guard / 形状或资源保护, Iteration / 迭代处理。

### Lines 121-126 / 第 121-126 行

```cpp
  return compiled;
}
constexpr auto backend_name = "xnnpack";
static auto pre_reg = backend_preprocess_register(backend_name, preprocess);

} // namespace torch::jit::xnnpack::delegate
```

- **EN:** Registration-related statements connect the implementation to operators, schemas, or runtime dispatch tables.
- **CN:** 与注册相关的语句把实现连接到算子、模式或运行时分发表。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。
- **EN:** Concepts touched here: Operator schema / 算子模式, Backend integration / 后端集成, Registration / 注册机制, Namespace scoping / 命名空间作用域, Result propagation / 结果传递.
- **CN:** 这里涉及的概念包括：Operator schema / 算子模式, Backend integration / 后端集成, Registration / 注册机制, Namespace scoping / 命名空间作用域, Result propagation / 结果传递。

## Key Concepts / 关键概念

- **Custom backend pipeline** — 自定义后端流水线
- **Graph IR** — 图中间表示
- **Nodes and values** — 节点与值
- **Type system** — 类型系统
- **Operator schema** — 算子模式
- **Module API** — 模块 API
- **Serialization** — 序列化
- **Backend integration** — 后端集成
- **Registration** — 注册机制
- **Shape/resource guard** — 形状或资源保护

## Dependencies / 依赖关系

- `torch/csrc/jit/backends/backend.h`
- `torch/csrc/jit/backends/backend_preprocess.h`
- `torch/csrc/jit/tensorexpr/graph_opt.h`
- `torch/torch.h`
- `ATen/core/List.h`
- `torch/csrc/jit/backends/xnnpack/xnnpack_graph_builder.h`
