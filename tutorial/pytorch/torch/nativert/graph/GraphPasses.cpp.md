# GraphPasses.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/nativert/graph/GraphPasses.cpp`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements Native Runtime runtime behavior for GraphPasses, including execution, delegation, or low-level utilities.
- 用途 (CN): 实现 Native Runtime 中 runtime 子模块里与 GraphPasses 相关的行为，包括执行、delegate 协调或底层工具逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5
```cpp
#include <torch/nativert/graph/GraphPasses.h>

#include <unordered_set>

#include <fmt/format.h>
```
- EN: Pulls in compile-time configuration or headers required here. Internal includes: `torch/nativert/graph/GraphPasses.h`; external includes: `unordered_set`, `fmt/format.h`.
- CN: 引入此处所需的编译期开关或头文件。内部依赖：`torch/nativert/graph/GraphPasses.h`；外部依赖：`unordered_set`, `fmt/format.h`。

### Lines 6-10
```cpp

#include <ATen/core/dispatch/Dispatcher.h>
#include <ATen/core/function_schema.h>

#include <c10/util/StringUtil.h>
```
- EN: Pulls in compile-time configuration or headers required here. Internal includes: `ATen/core/dispatch/Dispatcher.h`, `ATen/core/function_schema.h`, `c10/util/StringUtil.h`; external includes: none.
- CN: 引入此处所需的编译期开关或头文件。内部依赖：`ATen/core/dispatch/Dispatcher.h`, `ATen/core/function_schema.h`, `c10/util/StringUtil.h`；外部依赖：无。

### Lines 11-15
```cpp

namespace torch::nativert {
namespace {
bool isScalar(const Constant& c) {
  return std::holds_alternative<int64_t>(c) ||
```
- EN: This block returns results to callers or downstream stages. Key symbols: `isScalar`.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：`isScalar`。

### Lines 16-20
```cpp
      std::holds_alternative<double>(c);
}

bool isScalar(const Value& v) {
  return v.type() == Type::Kind::SymInt || v.type() == Type::Kind::SymFloat;
```
- EN: This block returns results to callers or downstream stages. Key symbols: `isScalar`, `type`.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：`isScalar`, `type`。

### Lines 21-25
```cpp
}

bool schemaTypeMatch(const c10::FunctionSchema& schema, const Node& node) {
  std::unordered_set<std::string> inputNames;
  for (const auto& input : node.inputs()) {
```
- EN: This block manipulates graph-like program structures; iterates over collections or execution units. Key symbols: `schemaTypeMatch`, `inputs`.
- CN: 该代码块操作图状程序结构；遍历集合或执行单元。关键符号：`schemaTypeMatch`, `inputs`。

### Lines 26-31
```cpp
    // The number of arguments is always O(10), so we can just do a linear scan.
    for (const auto& schemaArg : schema.arguments()) {
      if (schemaArg.name() == input.name) {
        if (schemaArg.type() == c10::TensorType::get() && input.value &&
            isScalar(*input.value)) {
          return false;
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: `arguments`, `name`, `type`, `get`, `isScalar`.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：`arguments`, `name`, `type`, `get`, `isScalar`。

### Lines 32-38
```cpp
        }
        break;
      }
    }
    inputNames.insert(input.name);
  }
  for (const auto& constant : node.attributes()) {
```
- EN: This block manipulates graph-like program structures; iterates over collections or execution units. Key symbols: `insert`, `attributes`.
- CN: 该代码块操作图状程序结构；遍历集合或执行单元。关键符号：`insert`, `attributes`。

### Lines 39-43
```cpp
    for (const auto& schemaArg : schema.arguments()) {
      if (schemaArg.name() == constant.name) {
        if (schemaArg.type() == c10::TensorType::get() &&
            isScalar(constant.value)) {
          return false;
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: `arguments`, `name`, `type`, `get`, `isScalar`.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：`arguments`, `name`, `type`, `get`, `isScalar`。

### Lines 44-50
```cpp
        }
        break;
      }
    }
    inputNames.insert(constant.name);
  }

```
- EN: This block implements local helper logic for GraphPasses. Key symbols: `insert`.
- CN: 该代码块实现与 GraphPasses 相关的局部辅助逻辑。关键符号：`insert`。

### Lines 51-55
```cpp
  // Make sure we have all the required arguments.
  for (const auto& schemaArg : schema.arguments()) {
    if (!schemaArg.default_value()) {
      if (inputNames.find(schemaArg.name()) == inputNames.end()) {
        return false;
```
- EN: This block handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: `arguments`, `default_value`, `find`, `name`, `end`.
- CN: 该代码块处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：`arguments`, `default_value`, `find`, `name`, `end`。

### Lines 56-61
```cpp
      }
    }
  }
  return true;
}

```
- EN: This block returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 62-70
```cpp
} // namespace

// PT2 intentionally broadcast things like aten.sub.Scalar
// to aten.sub.Tensor. https://github.com/pytorch/pytorch/issues/90923.
std::string selectScalarOverloadName(const Node& node) {
  // Copied from torch/csrc/utils/python_arg_parser.cpp
  // torch::should_allow_numbers_as_tensors() to workaround
  // some linking issues.
  static std::unordered_set<std::string> allowed = {
```
- EN: This block handles tensor metadata or sample values; manipulates graph-like program structures. Key symbols: `selectScalarOverloadName`.
- CN: 该代码块处理张量元数据或示例值；操作图状程序结构。关键符号：`selectScalarOverloadName`。

### Lines 71-80
```cpp
      "add",
      "add_",
      "add_out",
      "div",
      "div_",
      "div_out",
      "divide",
      "divide_",
      "divide_out", // alias of div
      "mul",
```
- EN: This block implements local helper logic for GraphPasses. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 GraphPasses 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 81-90
```cpp
      "mul_",
      "mul_out",
      "multiply",
      "multiply_",
      "multiply_out", // alias of mul
      "sub",
      "sub_",
      "sub_out",
      "subtract",
      "subtract_",
```
- EN: This block implements local helper logic for GraphPasses. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 GraphPasses 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 91-100
```cpp
      "subtract_out", // alias of sub
      "true_divide",
      "true_divide_",
      "true_divide_out",
      "to",
      "_to_copy",
      "copy_",
      "copy",
      "floor_divide",
      "floor_divide_",
```
- EN: This block implements local helper logic for GraphPasses. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 GraphPasses 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 101-105
```cpp
      "floor_divide_out",
      "_conj"};
  std::vector<std::string_view> atoms = c10::split(node.target(), '.');

  if (atoms.size() < 3) {
```
- EN: This block manipulates graph-like program structures; handles conditional control flow. Key symbols: `split`, `target`, `size`.
- CN: 该代码块操作图状程序结构；处理条件控制流。关键符号：`split`, `target`, `size`。

### Lines 106-112
```cpp
    return "";
  }

  std::string ns = std::string{atoms[atoms.size() - 3]};
  std::string opName = std::string{atoms[atoms.size() - 2]};
  std::string overloadName = std::string{atoms[atoms.size() - 1]};
  if (overloadName != "Tensor" && overloadName != "Tensor_Tensor" &&
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; returns results to callers or downstream stages. Key symbols: `size`.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；向调用方或后续阶段返回结果。关键符号：`size`。

### Lines 113-117
```cpp
      overloadName != "Tensor_mode") {
    return overloadName;
  }
  if (allowed.find(opName) == allowed.end()) {
    return overloadName;
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; returns results to callers or downstream stages. Key symbols: `find`, `end`.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；向调用方或后续阶段返回结果。关键符号：`find`, `end`。

### Lines 118-122
```cpp
  }
  auto op = c10::Dispatcher::singleton().findSchemaOrThrow(
      fmt::format("{}::{}", ns, opName.c_str()).c_str(), overloadName.c_str());
  if (schemaTypeMatch(op.schema(), node)) {
    return overloadName;
```
- EN: This block manipulates graph-like program structures; handles conditional control flow; returns results to callers or downstream stages. Key symbols: `singleton`, `findSchemaOrThrow`, `format`, `c_str`, `schemaTypeMatch`, `schema`.
- CN: 该代码块操作图状程序结构；处理条件控制流；向调用方或后续阶段返回结果。关键符号：`singleton`, `findSchemaOrThrow`, `format`, `c_str`, `schemaTypeMatch`, `schema`。

### Lines 123-128
```cpp
  }
  for (const auto& variant :
       {"Scalar_mode", "Scalar", "Scalar_Tensor", "Tensor_Scalar"}) {
    if (auto schema = c10::Dispatcher::singleton().findSchema(
            {fmt::format("{}::{}", ns, opName.c_str()), variant})) {
      if (schemaTypeMatch(schema->schema(), node)) {
```
- EN: This block handles tensor metadata or sample values; manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units. Key symbols: `singleton`, `findSchema`, `format`, `c_str`, `schemaTypeMatch`, `schema`.
- CN: 该代码块处理张量元数据或示例值；操作图状程序结构；处理条件控制流；遍历集合或执行单元。关键符号：`singleton`, `findSchema`, `format`, `c_str`, `schemaTypeMatch`, `schema`。

### Lines 129-133
```cpp
        return variant;
      }
    }
  }
  return overloadName;
```
- EN: This block returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 134-138
```cpp
}

void selectScalarOverload(Graph* graph) {
  for (auto& node : graph->nodes()) {
    for (auto& attr : node.attributes()) {
```
- EN: This block manipulates graph-like program structures; iterates over collections or execution units. Key symbols: `selectScalarOverload`, `nodes`, `attributes`.
- CN: 该代码块操作图状程序结构；遍历集合或执行单元。关键符号：`selectScalarOverload`, `nodes`, `attributes`。

### Lines 139-144
```cpp
      if (std::holds_alternative<std::unique_ptr<Graph>>(attr.value)) {
        selectScalarOverload(
            std::get<std::unique_ptr<Graph>>(attr.value).get());
      }
    }

```
- EN: This block manipulates graph-like program structures; handles conditional control flow. Key symbols: `selectScalarOverload`, `get`.
- CN: 该代码块操作图状程序结构；处理条件控制流。关键符号：`selectScalarOverload`, `get`。

### Lines 145-149
```cpp
    auto target = node.target();
    std::vector<std::string_view> atoms = c10::split(target, '.');

    size_t numAtoms = atoms.size();
    if (numAtoms != 5) {
```
- EN: This block manipulates graph-like program structures; handles conditional control flow. Key symbols: `target`, `split`, `size`.
- CN: 该代码块操作图状程序结构；处理条件控制流。关键符号：`target`, `split`, `size`。

### Lines 150-155
```cpp
      continue;
    }

    const std::string_view ns = atoms[numAtoms - 3];
    const std::string_view opName = atoms[numAtoms - 2];
    if (atoms[0] != "torch" || atoms[1] != "ops" || ns != "aten") {
```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 156-160
```cpp
      continue;
    }

    auto overloadName = selectScalarOverloadName(node);
    if (overloadName != atoms[numAtoms - 1]) {
```
- EN: This block manipulates graph-like program structures; handles conditional control flow. Key symbols: `selectScalarOverloadName`.
- CN: 该代码块操作图状程序结构；处理条件控制流。关键符号：`selectScalarOverloadName`。

### Lines 161-165
```cpp
      node.setTarget(
          fmt::format("torch.ops.{}.{}.{}", ns, opName, overloadName));
    } else if (ns == "aten" && opName == "sub" && overloadName == "Tensor") {
      // Special case for aten.sub.Tensor.
      if (auto i = node.tryGetInput("self")) {
```
- EN: This block handles tensor metadata or sample values; manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units. Key symbols: `setTarget`, `format`, `tryGetInput`.
- CN: 该代码块处理张量元数据或示例值；操作图状程序结构；处理条件控制流；遍历集合或执行单元。关键符号：`setTarget`, `format`, `tryGetInput`。

### Lines 166-172
```cpp
        if (isScalar(*i->value)) {
          node.updateInputName("self", "other");
          node.updateInputName("other", "self");
          node.setTarget("torch.ops.aten.rsub.Scalar");
        }
      }
      if (auto a = node.tryGetAttribute("self")) {
```
- EN: This block manipulates graph-like program structures; handles conditional control flow. Key symbols: `isScalar`, `updateInputName`, `setTarget`, `tryGetAttribute`.
- CN: 该代码块操作图状程序结构；处理条件控制流。关键符号：`isScalar`, `updateInputName`, `setTarget`, `tryGetAttribute`。

### Lines 173-182
```cpp
        if (isScalar(a->value)) {
          node.updateAttributeName("self", "other");
          node.updateInputName("other", "self");
          node.setTarget("torch.ops.aten.rsub.Scalar");
        }
      }
    }
  }
}

```
- EN: This block manipulates graph-like program structures; handles conditional control flow. Key symbols: `isScalar`, `updateAttributeName`, `updateInputName`, `setTarget`.
- CN: 该代码块操作图状程序结构；处理条件控制流。关键符号：`isScalar`, `updateAttributeName`, `updateInputName`, `setTarget`。

### Lines 183-183
```cpp
} // namespace torch::nativert
```
- EN: This block implements local helper logic for GraphPasses. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 GraphPasses 相关的局部辅助逻辑。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- EN: Tensor construction and metadata — the code manages sample tensors, dtypes, layouts, or device placement.
  CN: 张量构造与元数据——代码会管理示例张量、数据类型、布局或设备放置。
- EN: Graph capture and transformation — the implementation works with program graphs and rewrite passes.
  CN: 图捕获与变换——实现会处理程序图及其改写 pass。
- EN: Runtime composition — the file participates in assembling native runtime data flow, execution state, or backend handoff.
  CN: 运行时组合——该文件参与组织原生运行时的数据流、执行状态或后端交接。
- EN: Header-level integration — the translation unit depends on nearby runtime and framework declarations.
  CN: 头文件级集成——该翻译单元依赖周边运行时与框架声明。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `torch/nativert/graph/GraphPasses.h`, `ATen/core/dispatch/Dispatcher.h`, `ATen/core/function_schema.h`, `c10/util/StringUtil.h`
- External includes / 外部头文件: `unordered_set`, `fmt/format.h`
- Namespaces / 命名空间: `torch::nativert`
- Representative symbols / 代表性符号: `isScalar`, `type`, `schemaTypeMatch`, `inputs`, `arguments`, `name`, `get`, `insert`, `attributes`, `default_value`, `...`
