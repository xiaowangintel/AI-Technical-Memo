# function_extraction.h — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/passes/onnx/function_extraction.h`
- Repository: `pytorch`
- Purpose (EN): Declares the TorchScript JIT pass interfaces, helpers, and data structures for function extraction.
- 用途 (CN): 声明与 function extraction 相关的 TorchScript JIT pass 接口、辅助函数和数据结构。

## Line-by-Line Analysis / 逐行分析
### Lines 1-4
```cpp
#pragma once

#include <torch/csrc/jit/ir/ir.h>

```
- EN: Pulls in the headers needed by the function extraction logic. Internal dependencies: `torch/csrc/jit/ir/ir.h`; external dependencies: none.
- CN: 为 function extraction 相关逻辑引入所需头文件。内部依赖：`torch/csrc/jit/ir/ir.h`；外部依赖：无。

### Lines 5-10
```cpp
// This api will be used by serialization/export.cpp to extract function
// information. It should do conversion on graph to
//    1. Extract subgraph pattern of functions and define as local function
//    node.
//    2. Replace subgraph pattern of functions with a single node reflecting
//    that local function node type.
```
- EN: Documents the intent, constraints, or compatibility notes that guide the surrounding implementation.
- CN: 说明周围实现所遵循的目标、限制条件或兼容性要求。

### Lines 11-15
```cpp
// Function attribute map information is also returned, as Torch IR cannot
// represent these info inside Graph object.
// export.cpp will serialize the ONNX model with function_proto with
// above information.
namespace torch::jit::onnx {
```
- EN: This block manipulates TorchScript IR objects. Key symbols: no dominant local symbol names.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：无明显局部符号。

### Lines 16-21
```cpp

// The following return types are used to track information regarding function
// attributes, that are unable to be traced through Torch IR.
// NodeAttrNameMap tracks mapping from attribute name of IR Node inside function
// subgraph, to function attribute name. Here's an example of exporting CELU and
// LayerNorm.
```
- EN: Documents the intent, constraints, or compatibility notes that guide the surrounding implementation.
- CN: 说明周围实现所遵循的目标、限制条件或兼容性要求。

### Lines 22-27
```cpp
//
// clang-format off
// class M(torch.nn.Module):
//     def __init__(self) -> None:
//         super().__init__()
//         self.lns = torch.nn.ModuleList([torch.nn.LayerNorm(3, eps = i) for i in range(2)])
```
- EN: Documents the intent, constraints, or compatibility notes that guide the surrounding implementation.
- CN: 说明周围实现所遵循的目标、限制条件或兼容性要求。

### Lines 28-30
```cpp
//         self.celu1 = torch.nn.CELU(1.0)
//         self.celu2 = torch.nn.CELU(2.0)

```
- EN: Documents the intent, constraints, or compatibility notes that guide the surrounding implementation.
- CN: 说明周围实现所遵循的目标、限制条件或兼容性要求。

### Lines 31-36
```cpp
//     def forward(self, x: torch.Tensor, y: torch.Tensor, z: torch.Tensor) -> torch.Tensor:
//         res1 = self.celu1(x)
//         res2 = self.celu2(y)
//         for ln in self.lns:
//             z = ln(z)
//         return res1 + res2 + z
```
- EN: Documents the intent, constraints, or compatibility notes that guide the surrounding implementation.
- CN: 说明周围实现所遵循的目标、限制条件或兼容性要求。

### Lines 37-42
```cpp
// clang-format on
//
// Returning
//
// NodeAttrNameMap:
// {
```
- EN: Documents the intent, constraints, or compatibility notes that guide the surrounding implementation.
- CN: 说明周围实现所遵循的目标、限制条件或兼容性要求。

### Lines 43-48
```cpp
//    %1 : Float(2, 3) = onnx::Celu[alpha=2.](%y) : {
//      'alpha' : 'Celu_alpha'
//    }
// }
//
// The info here helps graph._export_onnx to construct function attributes for
```
- EN: Documents the intent, constraints, or compatibility notes that guide the surrounding implementation.
- CN: 说明周围实现所遵循的目标、限制条件或兼容性要求。

### Lines 49-52
```cpp
// onnx local FunctionProto.
using NodeAttrNameMap = std::
    unordered_map<const Node*, std::unordered_map<std::string, std::string>>;

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `NodeAttrNameMap`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`NodeAttrNameMap`。

### Lines 53-57
```cpp
TORCH_API NodeAttrNameMap ONNXFunctionExtraction(
    std::shared_ptr<Graph>& graph,
    const std::unordered_set<std::string>& module_names,
    const std::vector<std::string>& param_names);

```
- EN: Registers callable entities, pass hooks, or exported interfaces so runtime code can discover function extraction behavior. Symbols: `ONNXFunctionExtraction`.
- CN: 注册可调用实体、pass 钩子或导出接口，使运行时能够发现 function extraction 的行为。符号：`ONNXFunctionExtraction`。

### Lines 58-60
```cpp
TORCH_API void ONNXClearScopeRecords();

TORCH_API void ONNXTrackScopeAttributes(
```
- EN: Registers callable entities, pass hooks, or exported interfaces so runtime code can discover function extraction behavior. Symbols: `ONNXClearScopeRecords`, `ONNXTrackScopeAttributes`.
- CN: 注册可调用实体、pass 钩子或导出接口，使运行时能够发现 function extraction 的行为。符号：`ONNXClearScopeRecords`, `ONNXTrackScopeAttributes`。

### Lines 61-63
```cpp
    std::shared_ptr<Graph>& graph,
    std::map<std::string, IValue>& attributes);

```
- EN: This block manipulates TorchScript IR objects. Key symbols: no dominant local symbol names.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：无明显局部符号。

### Lines 64-64
```cpp
} // namespace torch::jit::onnx
```
- EN: Opens or closes namespace scopes so the remaining declarations live in the expected JIT/frontend module.
- CN: 打开或关闭命名空间作用域，使后续声明归属于预期的 JIT/前端模块。


## Key Concepts / 关键概念
- EN: IR graph manipulation — the file inspects or rewrites TorchScript graphs and nodes.
  CN: IR 图操作——该文件会检查或改写 TorchScript 的图、节点与块。
- EN: Type and shape reasoning — the code propagates static information across the IR.
  CN: 类型与形状推理——代码会在 IR 中传播静态信息。
- EN: Export interop — the implementation helps bridge TorchScript graphs to ONNX-style representations.
  CN: 导出互操作——实现帮助 TorchScript 图与 ONNX 风格表示之间建立桥接。
- EN: Pass orchestration — the file plugs logic into the TorchScript optimization pipeline.
  CN: Pass 编排——该文件把相关逻辑接入 TorchScript 优化流水线。
- EN: Subsystem integration — the implementation collaborates with nearby JIT/frontend headers and runtime helpers.
  CN: 子系统集成——实现会与附近的 JIT/前端头文件以及运行时辅助模块协同工作。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `torch/csrc/jit/ir/ir.h`
- External includes / 外部头文件: 无
- Namespaces / 命名空间: `torch::jit::onnx`
- Representative symbols / 代表性符号: `NodeAttrNameMap`, `ONNXFunctionExtraction`, `ONNXClearScopeRecords`, `ONNXTrackScopeAttributes`
