# shape_type_inference.h — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/passes/onnx/shape_type_inference.h`
- Repository: `pytorch`
- Purpose (EN): Declares the TorchScript JIT pass interfaces, helpers, and data structures for shape type inference.
- 用途 (CN): 声明与 shape type inference 相关的 TorchScript JIT pass 接口、辅助函数和数据结构。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6
```cpp
#pragma once

#include <torch/csrc/jit/ir/ir.h>
#include <torch/csrc/jit/passes/onnx/helper.h>
#include <torch/csrc/jit/python/python_arg_flatten.h>

```
- EN: Pulls in the headers needed by the shape type inference logic. Internal dependencies: `torch/csrc/jit/ir/ir.h`, `torch/csrc/jit/passes/onnx/helper.h`, `torch/csrc/jit/python/python_arg_flatten.h`; external dependencies: none.
- CN: 为 shape type inference 相关逻辑引入所需头文件。内部依赖：`torch/csrc/jit/ir/ir.h`, `torch/csrc/jit/passes/onnx/helper.h`, `torch/csrc/jit/python/python_arg_flatten.h`；外部依赖：无。

### Lines 7-16
```cpp
#include <utility>

namespace torch::jit {

// Merges existing_type and inferred_type.
// Returns {merged type, whether or not inferred_type was used}.
//
// The inferred type will take higher precedence, since it is produced by ONNX
// shape inference, and is more compatible with ONNX. In cases where ONNX shape
// inference fails to produce an inferred type, or produces an inferred type
```
- EN: This block implements local helper logic for shape type inference. Key symbols: no dominant local symbol names.
- CN: 该代码块实现与 shape type inference 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 17-26
```cpp
// that is incomplete, refer to existing type and fill in the gap that is
// missing. Currently the following cases are supported.
//  1. existing type: Tensor[], inferred type: Tensor[]
//    For list of tensors, existing type does not store datatype nor shape for
//    inner tensor. Thus inferred type always contain more information, and is
//    returned.
//  2. existing type: Tensor, inferred type: Tensor
//    Fill in missing info (shape, data type) for inferred type from existing
//    type.
//  3. existing type: Scalar[], inferred type: Tensor
```
- EN: Documents the intent, constraints, or compatibility notes that guide the surrounding implementation.
- CN: 说明周围实现所遵循的目标、限制条件或兼容性要求。

### Lines 27-32
```cpp
//    ONNX represents list of scalars by 1-d Tensor. Return inferred type since
//    it is more compatible with ONNX.
std::pair<TypePtr, bool> MergeInferredType(
    const TypePtr& existing_type,
    const TypePtr& inferred_type);

```
- EN: This block implements local helper logic for shape type inference. Key symbols: `MergeInferredType`.
- CN: 该代码块实现与 shape type inference 相关的局部辅助逻辑。关键符号：`MergeInferredType`。

### Lines 33-37
```cpp
void MergeInferredTypeAndSetMap(
    Value* dest_v,
    const TypePtr& existing_type,
    const TypePtr& inferred_type);

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `MergeInferredTypeAndSetMap`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`MergeInferredTypeAndSetMap`。

### Lines 38-42
```cpp
// Update graph input types with dynamic axes info.
// Axes that are marked as dynamic will be assigned as dynamic ShapeSymbol.
// Note it is possible for multiple axes to share the same ShapeSymbol,
// if they are defined as such in dynamic_axes.
TORCH_API void ONNXSetDynamicInputShape(
```
- EN: Registers callable entities, pass hooks, or exported interfaces so runtime code can discover shape type inference behavior. Symbols: `ONNXSetDynamicInputShape`.
- CN: 注册可调用实体、pass 钩子或导出接口，使运行时能够发现 shape type inference 的行为。符号：`ONNXSetDynamicInputShape`。

### Lines 43-48
```cpp
    std::shared_ptr<Graph>& graph,
    const std::unordered_map<
        std::string,
        std::unordered_map<int64_t, std::string>>& dynamic_axes,
    const std::vector<std::string>& input_names);

```
- EN: This block manipulates TorchScript IR objects. Key symbols: no dominant local symbol names.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：无明显局部符号。

### Lines 49-53
```cpp
// Update graph output with types of output Tensors.
// If onnx_shape_inference is true, types of output Tensors will be compared and
// merged with inferred types. It is possible that inferred types contain
// dynamic axes, hence it takes precedence over types of output Tensors.
TORCH_API void ONNXAssignOutputShape(
```
- EN: Registers callable entities, pass hooks, or exported interfaces so runtime code can discover shape type inference behavior. Symbols: `ONNXAssignOutputShape`.
- CN: 注册可调用实体、pass 钩子或导出接口，使运行时能够发现 shape type inference 的行为。符号：`ONNXAssignOutputShape`。

### Lines 54-58
```cpp
    std::shared_ptr<Graph>& graph,
    at::ArrayRef<at::Tensor> outputs,
    const python::IODescriptor& desc,
    bool onnx_shape_inference,
    bool is_script,
```
- EN: This block manipulates TorchScript IR objects. Key symbols: no dominant local symbol names.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：无明显局部符号。

### Lines 59-65
```cpp
    int opset_version);

// Replace None in output with Optional node (opset > 15) if it's
// script model. This helps align the output format in ONNX internal tests
// when comparing pytorch results with ONNX results, as they have different
// process for None in output.
void ReplaceGraphOutputNoneWithOptional(
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `ReplaceGraphOutputNoneWithOptional`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`ReplaceGraphOutputNoneWithOptional`。

### Lines 66-74
```cpp
    std::shared_ptr<Graph>& graph,
    size_t outputs_index);
Node* ONNXOptionalNodeForNone(std::shared_ptr<Graph>& graph);

// Utilize ONNX Shape Inference for node.
// The node must have ONNX namespace, and is valid ONNX node according to spec.
// On successful ONNX shape inference runs, the function updates output types of
// n with inferred shape and type. Otherwise n is unchanged.
TORCH_API void ONNXShapeTypeInference(
```
- EN: Registers callable entities, pass hooks, or exported interfaces so runtime code can discover shape type inference behavior. Symbols: `ONNXOptionalNodeForNone`, `ONNXShapeTypeInference`.
- CN: 注册可调用实体、pass 钩子或导出接口，使运行时能够发现 shape type inference 的行为。符号：`ONNXOptionalNodeForNone`, `ONNXShapeTypeInference`。

### Lines 75-83
```cpp
    Node* n,
    const ParamMap& params_dict,
    int opset_version);

// Utilize ONNX Shape Inference for graph.
// Internally calls ONNXShapeTypeInference for each node, to achieve more
// coverage that skips only individual nodes if illegal, instead of skipping for
// the entire graph.
TORCH_API void ONNXShapeTypeInference(
```
- EN: Registers callable entities, pass hooks, or exported interfaces so runtime code can discover shape type inference behavior. Symbols: `ONNXShapeTypeInference`.
- CN: 注册可调用实体、pass 钩子或导出接口，使运行时能够发现 shape type inference 的行为。符号：`ONNXShapeTypeInference`。

### Lines 84-88
```cpp
    std::shared_ptr<Graph>& g,
    const ParamMap& params_dict,
    int opset_version);

bool AllGraphInputsStatic(const Graph* g);
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `AllGraphInputsStatic`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`AllGraphInputsStatic`。

### Lines 89-93
```cpp
std::pair<bool, bool> AreInputsReliableOrStatic(Node* n);
void UpdateReliable(
    torch::jit::Value* output,
    const std::pair<bool, bool>& input_reliable,
    bool no_type_warning = false);
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `AreInputsReliableOrStatic`, `UpdateReliable`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`AreInputsReliableOrStatic`, `UpdateReliable`。

### Lines 94-98
```cpp

void UpdateReliable(torch::jit::Node* n);
void UpdateShapeConstantIfReliable(torch::jit::Value* output);

} // namespace torch::jit
```
- EN: Opens or closes namespace scopes so the remaining declarations live in the expected JIT/frontend module.
- CN: 打开或关闭命名空间作用域，使后续声明归属于预期的 JIT/前端模块。


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
- Internal includes / 内部头文件: `torch/csrc/jit/ir/ir.h`, `torch/csrc/jit/passes/onnx/helper.h`, `torch/csrc/jit/python/python_arg_flatten.h`
- External includes / 外部头文件: `utility`
- Namespaces / 命名空间: `torch::jit`
- Representative symbols / 代表性符号: `MergeInferredType`, `MergeInferredTypeAndSetMap`, `ONNXSetDynamicInputShape`, `ONNXAssignOutputShape`, `ReplaceGraphOutputNoneWithOptional`, `ONNXOptionalNodeForNone`, `ONNXShapeTypeInference`, `AllGraphInputsStatic`, `AreInputsReliableOrStatic`, `UpdateReliable`, `...`
