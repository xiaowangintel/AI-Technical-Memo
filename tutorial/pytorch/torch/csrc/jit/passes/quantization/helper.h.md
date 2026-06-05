# helper.h — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/passes/quantization/helper.h`
- Repository: `pytorch`
- Purpose (EN): Declares the TorchScript JIT pass interfaces, helpers, and data structures for helper.
- 用途 (CN): 声明与 helper 相关的 TorchScript JIT pass 接口、辅助函数和数据结构。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7
```cpp
#pragma once
#include <torch/csrc/jit/api/module.h>
#include <torch/csrc/jit/ir/ir.h>
#include <torch/csrc/jit/ir/subgraph_matcher.h>
#include <torch/csrc/jit/passes/graph_rewrite_helper.h>
#include <torch/csrc/jit/passes/quantization/quantization_type.h>

```
- EN: Pulls in the headers needed by the helper logic. Internal dependencies: `torch/csrc/jit/api/module.h`, `torch/csrc/jit/ir/ir.h`, `torch/csrc/jit/ir/subgraph_matcher.h`, `torch/csrc/jit/passes/graph_rewrite_helper.h`, `torch/csrc/jit/passes/quantization/quantization_type.h`; external dependencies: none.
- CN: 为 helper 相关逻辑引入所需头文件。内部依赖：`torch/csrc/jit/api/module.h`, `torch/csrc/jit/ir/ir.h`, `torch/csrc/jit/ir/subgraph_matcher.h`, `torch/csrc/jit/passes/graph_rewrite_helper.h`, `torch/csrc/jit/passes/quantization/quantization_type.h`；外部依赖：无。

### Lines 8-14
```cpp
#include <functional>
#include <regex>

namespace torch::jit {

using graph_rewrite_helper::getFuncName;

```
- EN: Opens or closes namespace scopes so the remaining declarations live in the expected JIT/frontend module.
- CN: 打开或关闭命名空间作用域，使后续声明归属于预期的 JIT/前端模块。

### Lines 15-21
```cpp
// Vector of a module and the name of its method
using ModuleMethodVector = std::vector<std::pair<Module, std::string>>;
// Map of quantization parameter name and value
// for example _scale, _zero_point,
// _scalar_type and _axis(for per channel quantization)
using QParamVector = std::vector<std::pair<std::string, IValue>>;

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `ModuleMethodVector`, `QParamVector`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`ModuleMethodVector`, `QParamVector`。

### Lines 22-29
```cpp
// =========== helper functions for Value =========
// Check if a value is weight, since we need to use weight observer
// for weight
TORCH_API bool isWeight(Value* v);

// Check if a value is bias for conv and linear, which we do not
// quantize
TORCH_API bool isBiasOfConvOrLinear(Value* v);
```
- EN: Registers callable entities, pass hooks, or exported interfaces so runtime code can discover helper behavior. Symbols: `isWeight`, `isBiasOfConvOrLinear`.
- CN: 注册可调用实体、pass 钩子或导出接口，使运行时能够发现 helper 的行为。符号：`isWeight`, `isBiasOfConvOrLinear`。

### Lines 30-40
```cpp

TORCH_API bool isEmbeddingBagNonInput(Value* v);

// Get the use as scalar input of clamp ops for the input value
std::optional<Use> getClampScalarInputUse(Value* v);

// For a given value `v`, get the list of values that we need to check
// if they are observed/quantized or not, if so, we can say the
// `v` is also observed/quantized, since we can derive
// the quantization parameters for `v` given the list of values
TORCH_API std::vector<Value*> getPassThroughInputs(Value* v);
```
- EN: Registers callable entities, pass hooks, or exported interfaces so runtime code can discover helper behavior. Symbols: `isEmbeddingBagNonInput`, `getClampScalarInputUse`, `getPassThroughInputs`.
- CN: 注册可调用实体、pass 钩子或导出接口，使运行时能够发现 helper 的行为。符号：`isEmbeddingBagNonInput`, `getClampScalarInputUse`, `getPassThroughInputs`。

### Lines 41-47
```cpp

// Clones the method by the name of orig_method_name into new_method_name method
TORCH_API void cloneMethod(
    Module& module,
    const std::string& orig_method_name,
    const std::string& new_method_name);

```
- EN: Registers callable entities, pass hooks, or exported interfaces so runtime code can discover helper behavior. Symbols: `cloneMethod`.
- CN: 注册可调用实体、pass 钩子或导出接口，使运行时能够发现 helper 的行为。符号：`cloneMethod`。

### Lines 48-58
```cpp
// Check if a value in the graph is a Scalar value
TORCH_API bool isScalar(Value* v);

// Check if value is the input of the graph
TORCH_API bool hitGraphInput(Value* value);

// Converts a mangled name, such as
//   __torch__.torch.ao.nn.quantized.modules.conv.___torch_mangle_7.Conv2d
// into an unmangled name, such as
//   __torch__.torch.ao.nn.quantized.modules.conv.Conv2d
TORCH_API std::string removeTorchMangle(const std::string& orig_name);
```
- EN: Registers callable entities, pass hooks, or exported interfaces so runtime code can discover helper behavior. Symbols: `isScalar`, `hitGraphInput`, `removeTorchMangle`.
- CN: 注册可调用实体、pass 钩子或导出接口，使运行时能够发现 helper 的行为。符号：`isScalar`, `hitGraphInput`, `removeTorchMangle`。

### Lines 59-65
```cpp

// Return the module name that corresponds to the value.
TORCH_API std::optional<std::string> getModuleName(Value* value);

// =========== helper functions for Node =========
TORCH_API bool isSingleInputGeneralShapeAtenFunction(Node* n);

```
- EN: Registers callable entities, pass hooks, or exported interfaces so runtime code can discover helper behavior. Symbols: `getModuleName`, `isSingleInputGeneralShapeAtenFunction`.
- CN: 注册可调用实体、pass 钩子或导出接口，使运行时能够发现 helper 的行为。符号：`getModuleName`, `isSingleInputGeneralShapeAtenFunction`。

### Lines 66-72
```cpp
TORCH_API bool isSingleInputGeneralValueAtenFunction(Node* n);

TORCH_API bool isSingleInputGeneralCallFunction(Node* n);

TORCH_API bool isSingleInputGeneralAtenFunction(Node* n);

TORCH_API bool isClamp(Node* n);
```
- EN: Registers callable entities, pass hooks, or exported interfaces so runtime code can discover helper behavior. Symbols: `isSingleInputGeneralValueAtenFunction`, `isSingleInputGeneralCallFunction`, `isSingleInputGeneralAtenFunction`, `isClamp`.
- CN: 注册可调用实体、pass 钩子或导出接口，使运行时能够发现 helper 的行为。符号：`isSingleInputGeneralValueAtenFunction`, `isSingleInputGeneralCallFunction`, `isSingleInputGeneralAtenFunction`, `isClamp`。

### Lines 73-79
```cpp

// Check if the node will produce the same result regardless of whether
// the input tensor is quantized or not, example: aten::size
TORCH_API bool isTensorInfoNode(Node* n);

// Check if this the propagate op that has single input, e.g. aten::cat
TORCH_API bool isPropagateQuantSingleInputOp(Node* n);
```
- EN: Registers callable entities, pass hooks, or exported interfaces so runtime code can discover helper behavior. Symbols: `isTensorInfoNode`, `isPropagateQuantSingleInputOp`.
- CN: 注册可调用实体、pass 钩子或导出接口，使运行时能够发现 helper 的行为。符号：`isTensorInfoNode`, `isPropagateQuantSingleInputOp`。

### Lines 80-86
```cpp

// Check if this is the propagate op that has two inputs, e.g. aten::add
TORCH_API bool isPropagateQuantBinaryOp(Node* n);

// Check if this is the node that we'll quantize or not quantize depending on
// whether the input of the node is quantized, example: aten::cat
TORCH_API bool isPropagateQuantOp(Node* n);
```
- EN: Registers callable entities, pass hooks, or exported interfaces so runtime code can discover helper behavior. Symbols: `isPropagateQuantBinaryOp`, `isPropagateQuantOp`.
- CN: 注册可调用实体、pass 钩子或导出接口，使运行时能够发现 helper 的行为。符号：`isPropagateQuantBinaryOp`, `isPropagateQuantOp`。

### Lines 87-93
```cpp

// Check if the node is a binary op like aten::add and aten::mul and
// if the input 1 is a scalar, these ops will be quantized to
// quantized::{op}_scalar
TORCH_API bool isBinaryOpWithScalarInput(Node* n);

TORCH_API std::optional<std::tuple<c10::QScheme, QParamVector>> getFixedQParams(
```
- EN: Registers callable entities, pass hooks, or exported interfaces so runtime code can discover helper behavior. Symbols: `isBinaryOpWithScalarInput`, `getFixedQParams`.
- CN: 注册可调用实体、pass 钩子或导出接口，使运行时能够发现 helper 的行为。符号：`isBinaryOpWithScalarInput`, `getFixedQParams`。

### Lines 94-101
```cpp
    Node* n);

// We don't want to analyze the graph for some `builtin` CallFunctions
// like `linear` because we want to preserve the op boundary
TORCH_API bool userDefinedCallFunction(Node* n);

// Check if the node has scalar input
TORCH_API bool hasScalarInput(Node* n);
```
- EN: Registers callable entities, pass hooks, or exported interfaces so runtime code can discover helper behavior. Symbols: `userDefinedCallFunction`, `hasScalarInput`.
- CN: 注册可调用实体、pass 钩子或导出接口，使运行时能够发现 helper 的行为。符号：`userDefinedCallFunction`, `hasScalarInput`。

### Lines 102-109
```cpp

// Check if a node is quantizable
TORCH_API bool nodeQuantizable(
    Node* n,
    QuantType quant_type = QuantType::STATIC);

// Nodes which only require quantization of weight value, eg. embedding_bag
bool isWeightOnlyStaticQuantOp(Node* n);
```
- EN: Registers callable entities, pass hooks, or exported interfaces so runtime code can discover helper behavior. Symbols: `nodeQuantizable`, `isWeightOnlyStaticQuantOp`.
- CN: 注册可调用实体、pass 钩子或导出接口，使运行时能够发现 helper 的行为。符号：`nodeQuantizable`, `isWeightOnlyStaticQuantOp`。

### Lines 110-116
```cpp

// Check if a use of the value is quantizable, this depends on
// both the use node and the offset
TORCH_API bool useQuantizable(const Use& use, QuantType quant_type);

// Given a CallFunction node, extract the graph of the called function
TORCH_API std::shared_ptr<Graph> getCallFunctionGraph(Node* n);
```
- EN: Registers callable entities, pass hooks, or exported interfaces so runtime code can discover helper behavior. Symbols: `useQuantizable`, `getCallFunctionGraph`.
- CN: 注册可调用实体、pass 钩子或导出接口，使运行时能够发现 helper 的行为。符号：`useQuantizable`, `getCallFunctionGraph`。

### Lines 117-123
```cpp

// Check if `use` is a CallFunction of name `func_name` and if value
// `v` is the nth argument (if provided) of the function
bool matchCallFuncToUse(
    const Use& use,
    const std::string& func_name,
    std::optional<int> nth_arg);
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `matchCallFuncToUse`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`matchCallFuncToUse`。

### Lines 124-130
```cpp

// Check if `use` is a AtenFunction of name `func_name` and if value
// `v` is the nth argument (if provided) of the function
bool matchAtenFuncToUse(
    const Use& use,
    const std::string& func_name,
    std::optional<int> nth_arg);
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `matchAtenFuncToUse`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`matchAtenFuncToUse`。

### Lines 131-138
```cpp

// =========== helper functions for Block =========
// checks if a block will always raise an Exception
TORCH_API bool alwaysRaisesException(Block* block);

// =========== helper functions for Module  ==========
// TODO: remove
TORCH_API std::vector<std::string> getModuleAccessPath(
```
- EN: Registers callable entities, pass hooks, or exported interfaces so runtime code can discover helper behavior. Symbols: `alwaysRaisesException`, `getModuleAccessPath`.
- CN: 注册可调用实体、pass 钩子或导出接口，使运行时能够发现 helper 的行为。符号：`alwaysRaisesException`, `getModuleAccessPath`。

### Lines 139-148
```cpp
    Value* instance,
    Value* self);
// TODO: remove
TORCH_API Module
findChildModule(const Module& module, const std::vector<std::string>& path);

// Given an CallMethod node, get the module instance corresponding
// to the instance Value
// TODO: refactor all current uses of this function to the Opt one
TORCH_API Module getInvokedModule(Module& module, Node* n, Value* self);
```
- EN: Registers callable entities, pass hooks, or exported interfaces so runtime code can discover helper behavior. Symbols: `findChildModule`, `getInvokedModule`.
- CN: 注册可调用实体、pass 钩子或导出接口，使运行时能够发现 helper 的行为。符号：`findChildModule`, `getInvokedModule`。

### Lines 149-155
```cpp

// Given an CallMethod node, get the module instance corresponding
// to the instance Value if the instance is a module, otherwise return
// std::nullopt
std::optional<Module> getInvokedModuleOpt(
    const Module& module,
    Node* n,
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `getInvokedModuleOpt`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`getInvokedModuleOpt`。

### Lines 156-165
```cpp
    Value* self);

// ==================== filter functions for matches ==============
// filter to check Value `vname` is a constant of int value `value`
bool is_int_constant(
    const Match& match,
    const std::unordered_map<std::string, Value*>& vmap,
    const std::string& vname,
    int value);

```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `is_int_constant`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`is_int_constant`。

### Lines 166-172
```cpp
// filter to check if the %alpha argument of aten::add is constant 1
bool aten_add_alpha_is_one(
    const Match& match,
    const std::unordered_map<std::string, Value*>& vmap);

// filter to check if the functional in CallFunction is relu
bool is_functional_relu(
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `aten_add_alpha_is_one`, `is_functional_relu`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`aten_add_alpha_is_one`, `is_functional_relu`。

### Lines 173-180
```cpp
    const Match& match,
    const std::unordered_map<std::string, Value*>& vmap);

// filter to check if the module is torch.nn.ReLU
bool is_relu_module(
    const Match& match,
    const std::unordered_map<std::string, Value*>& vmap);

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `is_relu_module`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`is_relu_module`。

### Lines 181-189
```cpp
bool is_linear_module(
    const Match& match,
    const std::unordered_map<std::string, Value*>& vmap);

// TODO: add a macro to declare the filters
bool is_conv1d_module(
    const Match& match,
    const std::unordered_map<std::string, Value*>& vmap);

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `is_linear_module`, `is_conv1d_module`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`is_linear_module`, `is_conv1d_module`。

### Lines 190-197
```cpp
bool is_conv2d_module(
    const Match& match,
    const std::unordered_map<std::string, Value*>& vmap);

bool is_conv3d_module(
    const Match& match,
    const std::unordered_map<std::string, Value*>& vmap);

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `is_conv2d_module`, `is_conv3d_module`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`is_conv2d_module`, `is_conv3d_module`。

### Lines 198-205
```cpp
bool is_conv_transpose1d_module(
    const Match& match,
    const std::unordered_map<std::string, Value*>& vmap);

bool is_conv_transpose2d_module(
    const Match& match,
    const std::unordered_map<std::string, Value*>& vmap);

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `is_conv_transpose1d_module`, `is_conv_transpose2d_module`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`is_conv_transpose1d_module`, `is_conv_transpose2d_module`。

### Lines 206-213
```cpp
bool is_batchnorm2d_module(
    const Match& match,
    const std::unordered_map<std::string, Value*>& vmap);

bool is_batchnorm3d_module(
    const Match& match,
    const std::unordered_map<std::string, Value*>& vmap);

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `is_batchnorm2d_module`, `is_batchnorm3d_module`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`is_batchnorm2d_module`, `is_batchnorm3d_module`。

### Lines 214-214
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
- EN: Type and shape reasoning — the code propagates static information across the IR.
  CN: 类型与形状推理——代码会在 IR 中传播静态信息。
- EN: Subsystem integration — the implementation collaborates with nearby JIT/frontend headers and runtime helpers.
  CN: 子系统集成——实现会与附近的 JIT/前端头文件以及运行时辅助模块协同工作。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `torch/csrc/jit/api/module.h`, `torch/csrc/jit/ir/ir.h`, `torch/csrc/jit/ir/subgraph_matcher.h`, `torch/csrc/jit/passes/graph_rewrite_helper.h`, `torch/csrc/jit/passes/quantization/quantization_type.h`
- External includes / 外部头文件: `functional`, `regex`
- Namespaces / 命名空间: `torch::jit`
- Representative symbols / 代表性符号: `graph_rewrite_helper`, `ModuleMethodVector`, `QParamVector`, `isWeight`, `isBiasOfConvOrLinear`, `isEmbeddingBagNonInput`, `getClampScalarInputUse`, `getPassThroughInputs`, `cloneMethod`, `isScalar`, `...`
