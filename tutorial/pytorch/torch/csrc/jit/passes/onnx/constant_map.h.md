# constant_map.h — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/passes/onnx/constant_map.h`
- Repository: `pytorch`
- Purpose (EN): Declares the TorchScript JIT pass interfaces, helpers, and data structures for constant map.
- 用途 (CN): 声明与 constant map 相关的 TorchScript JIT pass 接口、辅助函数和数据结构。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6
```cpp
#pragma once

#include <c10/macros/Macros.h>

#include <onnx/shape_inference/implementation.h>

```
- EN: Pulls in the headers needed by the constant map logic. Internal dependencies: `c10/macros/Macros.h`; external dependencies: `onnx/shape_inference/implementation.h`.
- CN: 为 constant map 相关逻辑引入所需头文件。内部依赖：`c10/macros/Macros.h`；外部依赖：`onnx/shape_inference/implementation.h`。

### Lines 7-11
```cpp
#include <torch/csrc/jit/ir/ir.h>
#include <torch/csrc/jit/serialization/export.h>
#include <unordered_map>

namespace torch::jit {
```
- EN: Opens or closes namespace scopes so the remaining declarations live in the expected JIT/frontend module.
- CN: 打开或关闭命名空间作用域，使后续声明归属于预期的 JIT/前端模块。

### Lines 12-16
```cpp

using ShapeDataMap =
    std::unordered_map<std::string, ::ONNX_NAMESPACE::TensorShapeProto>;

class ConstantValueMap {
```
- EN: Declares core types or data containers for this file. Prominent symbols: `ShapeDataMap`, `ConstantValueMap`.
- CN: 声明该文件使用的核心类型或数据容器。主要符号：`ShapeDataMap`, `ConstantValueMap`。

### Lines 17-21
```cpp
 public:
  static ConstantValueMap& getInstance();
  static void SetRank(const std::string& tensorName, size_t rankValue);
  static bool HasRank(const std::string& tensorName);
  static std::optional<size_t> GetRank(const std::string& tensorName);
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `getInstance`, `SetRank`, `HasRank`, `GetRank`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`getInstance`, `SetRank`, `HasRank`, `GetRank`。

### Lines 22-26
```cpp

  static void SetAllGraphInputsStatic(bool all_static);
  static std::optional<bool> GetAllGraphInputsStatic();

  static void SetAllGraphInputsReliableComputed(bool computed);
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `SetAllGraphInputsStatic`, `GetAllGraphInputsStatic`, `SetAllGraphInputsReliableComputed`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`SetAllGraphInputsStatic`, `GetAllGraphInputsStatic`, `SetAllGraphInputsReliableComputed`。

### Lines 27-32
```cpp
  static bool GetAllGraphInputsReliableComputed();

  static void SetShape(
      const std::string& tensorName,
      const c10::SymbolicShape& shapeValue);
  static bool HasShape(const std::string& tensorName);
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `GetAllGraphInputsReliableComputed`, `SetShape`, `HasShape`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`GetAllGraphInputsReliableComputed`, `SetShape`, `HasShape`。

### Lines 33-37
```cpp
  static std::optional<c10::SymbolicShape> GetShape(
      const std::string& tensorName);

  static void SetValue(const std::string& tensorName, const at::Tensor& value);
  static bool HasValue(const std::string& tensorName);
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `GetShape`, `SetValue`, `HasValue`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`GetShape`, `SetValue`, `HasValue`。

### Lines 38-43
```cpp
  static std::optional<at::Tensor> GetValue(const std::string& tensorName);
  static void EraseValue(const std::string& tensorName);

  static std::vector<int64_t> GetCompleteShapeInto1DInt64Vector(
      const c10::SymbolicShape& shape);
  static std::optional<std::vector<int64_t>> GetShapeInto1DInt64Vector(
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `GetValue`, `EraseValue`, `GetCompleteShapeInto1DInt64Vector`, `GetShapeInto1DInt64Vector`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`GetValue`, `EraseValue`, `GetCompleteShapeInto1DInt64Vector`, `GetShapeInto1DInt64Vector`。

### Lines 44-49
```cpp
      const std::string& value_name);
  static std::optional<std::vector<int64_t>>
  GetShapeInto1DInt64VectorWithOneUnknown(const std::string& value_name);
  static std::vector<int64_t> GetValueInto1DInt64Vector(
      const std::string& value_name);

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `GetShapeInto1DInt64VectorWithOneUnknown`, `GetValueInto1DInt64Vector`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`GetShapeInto1DInt64VectorWithOneUnknown`, `GetValueInto1DInt64Vector`。

### Lines 50-54
```cpp
  static void SetTypeReliable(const std::string& tensorName, bool reliable);
  static bool HasTypeReliable(const std::string& tensorName);
  static std::optional<bool> GetTypeReliable(const std::string& tensorName);

  static void SetUseInferredType(
```
- EN: This block implements local helper logic for constant map. Key symbols: `SetTypeReliable`, `HasTypeReliable`, `GetTypeReliable`, `SetUseInferredType`.
- CN: 该代码块实现与 constant map 相关的局部辅助逻辑。关键符号：`SetTypeReliable`, `HasTypeReliable`, `GetTypeReliable`, `SetUseInferredType`。

### Lines 55-59
```cpp
      const std::string& tensorName,
      bool useInferredType);
  static bool HasUseInferredType(const std::string& tensorName);
  static std::optional<bool> GetUseInferredType(const std::string& tensorName);

```
- EN: This block implements local helper logic for constant map. Key symbols: `HasUseInferredType`, `GetUseInferredType`.
- CN: 该代码块实现与 constant map 相关的局部辅助逻辑。关键符号：`HasUseInferredType`, `GetUseInferredType`。

### Lines 60-64
```cpp
  static void SetShapeValue(
      const std::string& tensorName,
      const c10::SymbolicShape& shapeValue);
  static bool HasShapeValue(const std::string& tensorName);
  static std::optional<c10::SymbolicShape> GetShapeValue(
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `SetShapeValue`, `HasShapeValue`, `GetShapeValue`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`SetShapeValue`, `HasShapeValue`, `GetShapeValue`。

### Lines 65-69
```cpp
      const std::string& tensorName);

  static ShapeDataMap& GetInferredShapeData();

  static SymbolDimMap& GetSymbolDimMap();
```
- EN: This block implements local helper logic for constant map. Key symbols: `GetInferredShapeData`, `GetSymbolDimMap`.
- CN: 该代码块实现与 constant map 相关的局部辅助逻辑。关键符号：`GetInferredShapeData`, `GetSymbolDimMap`。

### Lines 70-75
```cpp
  static DimSymbolMap& GetDimSymbolMap();

  static void UpdateValueName(
      const std::string& old_name,
      const std::string& new_name);

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `GetDimSymbolMap`, `UpdateValueName`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`GetDimSymbolMap`, `UpdateValueName`。

### Lines 76-81
```cpp
  static void PrintMaps();
  static void ClearMaps();
  ~ConstantValueMap() = default;

  ConstantValueMap& operator=(const ConstantValueMap&) = delete;

```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `PrintMaps`, `ClearMaps`, `~ConstantValueMap`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`PrintMaps`, `ClearMaps`, `~ConstantValueMap`。

### Lines 82-86
```cpp
 private:
  ConstantValueMap() = default;

  std::unordered_map<std::string, size_t> rankMap;
  std::unordered_map<std::string, c10::SymbolicShape> shapeMap;
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `ConstantValueMap`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`ConstantValueMap`。

### Lines 87-92
```cpp
  std::unordered_map<std::string, at::Tensor> tensorValueMap;
  // This map indicates whether the current type is reliably estimated or not.
  std::unordered_map<std::string, bool> typeReliableMap;
  // This map indicates whether the current type is estimated through inference
  // or tracer.
  std::unordered_map<std::string, bool> useInferredTypeMap;
```
- EN: This block manipulates TorchScript IR objects. Key symbols: no dominant local symbol names.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：无明显局部符号。

### Lines 93-100
```cpp
  // This map indicates a tensor value which represents a shape.
  // We assume that the rank of the tensor value <= 1, and we ensure this when
  // we write the processing logic for the operators. When the rank > 1, we
  // should be able to rewrite the model so that the rank <= 1. The difference
  // between shapeMap and shapeValueMap: shapeMap stores the shape of the tensor
  // from a node. shapeValueMap stores the value of the tensor from a node when
  // this tensor represents a shape.
  std::unordered_map<std::string, c10::SymbolicShape> shapeValueMap;
```
- EN: This block manipulates TorchScript IR objects. Key symbols: no dominant local symbol names.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：无明显局部符号。

### Lines 101-107
```cpp
  // Stores earlier data propagation results so that they are accessible
  // during future node-level shape inference.
  ShapeDataMap inferredShapeData;
  SymbolDimMap symbolDimMap;
  DimSymbolMap dimSymbolMap;
  // Stores if all graph-level inputs have static shape
  std::optional<bool> allGraphInputsStatic;
```
- EN: This block manipulates TorchScript IR objects. Key symbols: no dominant local symbol names.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：无明显局部符号。

### Lines 108-112
```cpp
  // True if reliable has been computed for all graph inputs
  bool allGraphInputsReliableComputed{};
};

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
- Internal includes / 内部头文件: `c10/macros/Macros.h`, `torch/csrc/jit/ir/ir.h`, `torch/csrc/jit/serialization/export.h`
- External includes / 外部头文件: `onnx/shape_inference/implementation.h`, `unordered_map`
- Namespaces / 命名空间: `torch::jit`
- Representative symbols / 代表性符号: `ShapeDataMap`, `ConstantValueMap`, `getInstance`, `SetRank`, `HasRank`, `GetRank`, `SetAllGraphInputsStatic`, `GetAllGraphInputsStatic`, `SetAllGraphInputsReliableComputed`, `GetAllGraphInputsReliableComputed`, `...`
