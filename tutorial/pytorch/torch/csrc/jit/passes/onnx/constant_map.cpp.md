# constant_map.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/passes/onnx/constant_map.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements the TorchScript JIT pass logic for constant map, including graph analysis and rewrites.
- 用途 (CN): 实现与 constant map 相关的 TorchScript JIT pass 逻辑，包括图分析与改写。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7
```cpp
#include <c10/util/irange.h>
#include <torch/csrc/jit/passes/onnx/constant_map.h>
#include <iostream>
#include <sstream>
#include <string>
#include <unordered_map>

```
- EN: Pulls in the headers needed by the constant map logic. Internal dependencies: `c10/util/irange.h`, `torch/csrc/jit/passes/onnx/constant_map.h`; external dependencies: `iostream`, `sstream`, `string`, `unordered_map`.
- CN: 为 constant map 相关逻辑引入所需头文件。内部依赖：`c10/util/irange.h`, `torch/csrc/jit/passes/onnx/constant_map.h`；外部依赖：`iostream`, `sstream`, `string`, `unordered_map`。

### Lines 8-15
```cpp
namespace torch::jit {

// Meyer’s Singleton for C++ 14
ConstantValueMap& ConstantValueMap::getInstance() {
  static ConstantValueMap s;
  return s;
}

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `getInstance`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`getInstance`。

### Lines 16-22
```cpp
void ConstantValueMap::SetRank(
    const std::string& tensorName,
    size_t rankValue) {
  ConstantValueMap::getInstance().rankMap[tensorName] = rankValue;
  ConstantValueMap::getInstance().useInferredTypeMap[tensorName] = true;
}

```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `SetRank`, `getInstance`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`SetRank`, `getInstance`。

### Lines 23-29
```cpp
bool ConstantValueMap::HasRank(const std::string& tensorName) {
  return ConstantValueMap::getInstance().rankMap.find(tensorName) !=
      ConstantValueMap::getInstance().rankMap.end();
}

std::optional<size_t> ConstantValueMap::GetRank(const std::string& tensorName) {
  if (!HasRank(tensorName)) {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `HasRank`, `getInstance`, `find`, `end`, `GetRank`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`HasRank`, `getInstance`, `find`, `end`, `GetRank`。

### Lines 30-38
```cpp
    return std::nullopt;
  }
  return ConstantValueMap::getInstance().rankMap[tensorName];
}

void ConstantValueMap::SetAllGraphInputsStatic(bool all_static) {
  ConstantValueMap::getInstance().allGraphInputsStatic = all_static;
}

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `getInstance`, `SetAllGraphInputsStatic`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`getInstance`, `SetAllGraphInputsStatic`。

### Lines 39-46
```cpp
std::optional<bool> ConstantValueMap::GetAllGraphInputsStatic() {
  return ConstantValueMap::getInstance().allGraphInputsStatic;
}

void ConstantValueMap::SetAllGraphInputsReliableComputed(bool computed) {
  ConstantValueMap::getInstance().allGraphInputsReliableComputed = computed;
}

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `GetAllGraphInputsStatic`, `getInstance`, `SetAllGraphInputsReliableComputed`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`GetAllGraphInputsStatic`, `getInstance`, `SetAllGraphInputsReliableComputed`。

### Lines 47-57
```cpp
bool ConstantValueMap::GetAllGraphInputsReliableComputed() {
  return ConstantValueMap::getInstance().allGraphInputsReliableComputed;
}

void ConstantValueMap::SetShape(
    const std::string& tensorName,
    const c10::SymbolicShape& shapeValue) {
  ConstantValueMap::getInstance().shapeMap[tensorName] = shapeValue;
  ConstantValueMap::getInstance().useInferredTypeMap[tensorName] = true;
}

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `GetAllGraphInputsReliableComputed`, `getInstance`, `SetShape`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`GetAllGraphInputsReliableComputed`, `getInstance`, `SetShape`。

### Lines 58-65
```cpp
bool ConstantValueMap::HasShape(const std::string& tensorName) {
  return ConstantValueMap::getInstance().shapeMap.find(tensorName) !=
      ConstantValueMap::getInstance().shapeMap.end();
}

std::optional<c10::SymbolicShape> ConstantValueMap::GetShape(
    const std::string& tensorName) {
  if (!HasShape(tensorName)) {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `HasShape`, `getInstance`, `find`, `end`, `GetShape`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`HasShape`, `getInstance`, `find`, `end`, `GetShape`。

### Lines 66-76
```cpp
    return std::nullopt;
  }
  return ConstantValueMap::getInstance().shapeMap[tensorName];
}

void ConstantValueMap::SetValue(
    const std::string& tensorName,
    const at::Tensor& value) {
  ConstantValueMap::getInstance().tensorValueMap[tensorName] = value;
}

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `getInstance`, `SetValue`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`getInstance`, `SetValue`。

### Lines 77-84
```cpp
bool ConstantValueMap::HasValue(const std::string& tensorName) {
  return ConstantValueMap::getInstance().tensorValueMap.find(tensorName) !=
      ConstantValueMap::getInstance().tensorValueMap.end();
}

std::optional<at::Tensor> ConstantValueMap::GetValue(
    const std::string& tensorName) {
  if (!HasValue(tensorName)) {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `HasValue`, `getInstance`, `find`, `end`, `GetValue`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`HasValue`, `getInstance`, `find`, `end`, `GetValue`。

### Lines 85-93
```cpp
    return std::nullopt;
  }
  return ConstantValueMap::getInstance().tensorValueMap[tensorName];
}

void ConstantValueMap::EraseValue(const std::string& tensorName) {
  ConstantValueMap::getInstance().tensorValueMap.erase(tensorName);
}

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `getInstance`, `EraseValue`, `erase`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`getInstance`, `EraseValue`, `erase`。

### Lines 94-100
```cpp
std::vector<int64_t> ConstantValueMap::GetCompleteShapeInto1DInt64Vector(
    const c10::SymbolicShape& shape) {
  TORCH_INTERNAL_ASSERT(shape.isComplete());
  std::vector<int64_t> shape_value;
  auto shape_symbol_list = shape.sizes().value();
  shape_value.reserve(shape_symbol_list.size());
  for (const auto& v : shape_symbol_list) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `GetCompleteShapeInto1DInt64Vector`, `isComplete`, `sizes`, `value`, `reserve`, `size`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`GetCompleteShapeInto1DInt64Vector`, `isComplete`, `sizes`, `value`, `reserve`, `size`。

### Lines 101-108
```cpp
    shape_value.emplace_back(v.static_size());
  }
  return shape_value;
}

std::optional<std::vector<int64_t>> ConstantValueMap::GetShapeInto1DInt64Vector(
    const std::string& value_name) {
  if (ConstantValueMap::HasShape(value_name)) {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `emplace_back`, `static_size`, `GetShapeInto1DInt64Vector`, `HasShape`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`emplace_back`, `static_size`, `GetShapeInto1DInt64Vector`, `HasShape`。

### Lines 109-116
```cpp
    auto shape_size = ConstantValueMap::GetShape(value_name).value();
    if (shape_size.isComplete()) {
      auto shape_value =
          ConstantValueMap::GetCompleteShapeInto1DInt64Vector(shape_size);
      return shape_value;
    }
  }
  return std::nullopt;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `GetShape`, `value`, `isComplete`, `GetCompleteShapeInto1DInt64Vector`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`GetShape`, `value`, `isComplete`, `GetCompleteShapeInto1DInt64Vector`。

### Lines 117-123
```cpp
}

std::optional<std::vector<int64_t>> ConstantValueMap::
    GetShapeInto1DInt64VectorWithOneUnknown(const std::string& value_name) {
  if (ConstantValueMap::HasShape(value_name)) {
    auto shape_size = ConstantValueMap::GetShape(value_name).value();
    if (shape_size.isComplete()) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `GetShapeInto1DInt64VectorWithOneUnknown`, `HasShape`, `GetShape`, `value`, `isComplete`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`GetShapeInto1DInt64VectorWithOneUnknown`, `HasShape`, `GetShape`, `value`, `isComplete`。

### Lines 124-130
```cpp
      return ConstantValueMap::GetCompleteShapeInto1DInt64Vector(shape_size);
    }
    size_t count_unknown = 0;
    auto shape_size_sizes = shape_size.sizes();
    if (shape_size_sizes.has_value()) {
      std::vector<int64_t> shape_value;
      auto shape_symbol_list = shape_size_sizes.value();
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `GetCompleteShapeInto1DInt64Vector`, `sizes`, `has_value`, `value`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`GetCompleteShapeInto1DInt64Vector`, `sizes`, `has_value`, `value`。

### Lines 131-139
```cpp
      for (const auto& v : shape_symbol_list) {
        if (v.is_static()) {
          shape_value.emplace_back(v.static_size());
        } else {
          shape_value.emplace_back(-1);
          count_unknown += 1;
        }
      }
      if (count_unknown == 1) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `is_static`, `emplace_back`, `static_size`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`is_static`, `emplace_back`, `static_size`。

### Lines 140-146
```cpp
        return shape_value;
      }
    }
  }
  return std::nullopt;
}

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: no dominant local symbol names.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：无明显局部符号。

### Lines 147-154
```cpp
// accessor<int64_t, 1> for 1DInt64 case.
std::vector<int64_t> ConstantValueMap::GetValueInto1DInt64Vector(
    const std::string& value_name) {
  auto value = ConstantValueMap::GetValue(value_name).value();
  auto value_int64_t = value.toType(at::ScalarType::Long);
  std::vector<int64_t> value_vector;
  value_vector.reserve(value_int64_t.size(0));
  auto value_size_a = value_int64_t.accessor<int64_t, 1>();
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `GetValueInto1DInt64Vector`, `GetValue`, `value`, `toType`, `reserve`, `size`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`GetValueInto1DInt64Vector`, `GetValue`, `value`, `toType`, `reserve`, `size`。

### Lines 155-161
```cpp
  for (const auto i : c10::irange(value_int64_t.size(0))) {
    value_vector.emplace_back(static_cast<int64_t>(value_size_a[i]));
  }
  return value_vector;
}

void ConstantValueMap::SetTypeReliable(
```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `irange`, `size`, `emplace_back`, `SetTypeReliable`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`irange`, `size`, `emplace_back`, `SetTypeReliable`。

### Lines 162-168
```cpp
    const std::string& tensorName,
    bool value) {
  ConstantValueMap::getInstance().typeReliableMap[tensorName] = value;
}

bool ConstantValueMap::HasTypeReliable(const std::string& tensorName) {
  return ConstantValueMap::getInstance().typeReliableMap.find(tensorName) !=
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `getInstance`, `HasTypeReliable`, `find`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`getInstance`, `HasTypeReliable`, `find`。

### Lines 169-175
```cpp
      ConstantValueMap::getInstance().typeReliableMap.end();
}

std::optional<bool> ConstantValueMap::GetTypeReliable(
    const std::string& tensorName) {
  if (!HasTypeReliable(tensorName)) {
    return std::nullopt;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `getInstance`, `end`, `GetTypeReliable`, `HasTypeReliable`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`getInstance`, `end`, `GetTypeReliable`, `HasTypeReliable`。

### Lines 176-182
```cpp
  }
  return ConstantValueMap::getInstance().typeReliableMap[tensorName];
}

void ConstantValueMap::SetUseInferredType(
    const std::string& tensorName,
    bool value) {
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `getInstance`, `SetUseInferredType`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`getInstance`, `SetUseInferredType`。

### Lines 183-190
```cpp
  ConstantValueMap::getInstance().useInferredTypeMap[tensorName] = value;
}

bool ConstantValueMap::HasUseInferredType(const std::string& tensorName) {
  return ConstantValueMap::getInstance().useInferredTypeMap.find(tensorName) !=
      ConstantValueMap::getInstance().useInferredTypeMap.end();
}

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `getInstance`, `HasUseInferredType`, `find`, `end`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`getInstance`, `HasUseInferredType`, `find`, `end`。

### Lines 191-198
```cpp
std::optional<bool> ConstantValueMap::GetUseInferredType(
    const std::string& tensorName) {
  if (!HasUseInferredType(tensorName)) {
    return std::nullopt;
  }
  return ConstantValueMap::getInstance().useInferredTypeMap[tensorName];
}

```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `GetUseInferredType`, `HasUseInferredType`, `getInstance`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`GetUseInferredType`, `HasUseInferredType`, `getInstance`。

### Lines 199-205
```cpp
void ConstantValueMap::SetShapeValue(
    const std::string& tensorName,
    const c10::SymbolicShape& shapeValue) {
  ConstantValueMap::getInstance().shapeValueMap[tensorName] = shapeValue;
}

bool ConstantValueMap::HasShapeValue(const std::string& tensorName) {
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `SetShapeValue`, `getInstance`, `HasShapeValue`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`SetShapeValue`, `getInstance`, `HasShapeValue`。

### Lines 206-212
```cpp
  return ConstantValueMap::getInstance().shapeValueMap.find(tensorName) !=
      ConstantValueMap::getInstance().shapeValueMap.end();
}

std::optional<c10::SymbolicShape> ConstantValueMap::GetShapeValue(
    const std::string& tensorName) {
  if (!HasShapeValue(tensorName)) {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `getInstance`, `find`, `end`, `GetShapeValue`, `HasShapeValue`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`getInstance`, `find`, `end`, `GetShapeValue`, `HasShapeValue`。

### Lines 213-220
```cpp
    return std::nullopt;
  }
  return ConstantValueMap::getInstance().shapeValueMap[tensorName];
}

// Gets the inferredShapeData which is obtained by ONNX data propagation
ShapeDataMap& ConstantValueMap::GetInferredShapeData() {
  return ConstantValueMap::getInstance().inferredShapeData;
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `getInstance`, `GetInferredShapeData`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`getInstance`, `GetInferredShapeData`。

### Lines 221-228
```cpp
}

SymbolDimMap& ConstantValueMap::GetSymbolDimMap() {
  return ConstantValueMap::getInstance().symbolDimMap;
}

DimSymbolMap& ConstantValueMap::GetDimSymbolMap() {
  return ConstantValueMap::getInstance().dimSymbolMap;
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `GetSymbolDimMap`, `getInstance`, `GetDimSymbolMap`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`GetSymbolDimMap`, `getInstance`, `GetDimSymbolMap`。

### Lines 229-237
```cpp
}

template <typename Map>
static void UpdateStrKey(
    Map& map,
    const std::string& old_key,
    const std::string& new_key) {
  TORCH_INTERNAL_ASSERT(old_key != new_key);
  if (map.find(old_key) == map.end()) {
```
- EN: Builds a reusable template/helper layer around constant map. Symbols: `UpdateStrKey`, `find`, `end`.
- CN: 围绕 constant map 构建可复用的模板或辅助层。符号：`UpdateStrKey`, `find`, `end`。

### Lines 238-244
```cpp
    return;
  }
  map[new_key] = map[old_key];
  map.erase(old_key);
}

void ConstantValueMap::UpdateValueName(
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `erase`, `UpdateValueName`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`erase`, `UpdateValueName`。

### Lines 245-258
```cpp
    const std::string& old_name,
    const std::string& new_name) {
  if (old_name == new_name) {
    return;
  }
  UpdateStrKey<decltype(rankMap)>(
      ConstantValueMap::getInstance().rankMap, old_name, new_name);
  UpdateStrKey<decltype(shapeMap)>(
      ConstantValueMap::getInstance().shapeMap, old_name, new_name);
  UpdateStrKey<decltype(tensorValueMap)>(
      ConstantValueMap::getInstance().tensorValueMap, old_name, new_name);
  UpdateStrKey<decltype(typeReliableMap)>(
      ConstantValueMap::getInstance().typeReliableMap, old_name, new_name);
  UpdateStrKey<decltype(useInferredTypeMap)>(
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `getInstance`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`getInstance`。

### Lines 259-265
```cpp
      ConstantValueMap::getInstance().useInferredTypeMap, old_name, new_name);
  UpdateStrKey<decltype(shapeValueMap)>(
      ConstantValueMap::getInstance().shapeValueMap, old_name, new_name);
  UpdateStrKey<decltype(inferredShapeData)>(
      ConstantValueMap::getInstance().inferredShapeData, old_name, new_name);
}

```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `getInstance`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`getInstance`。

### Lines 266-279
```cpp
void ConstantValueMap::ClearMaps() {
  ConstantValueMap::getInstance().rankMap.clear();
  ConstantValueMap::getInstance().shapeMap.clear();
  ConstantValueMap::getInstance().tensorValueMap.clear();
  ConstantValueMap::getInstance().typeReliableMap.clear();
  ConstantValueMap::getInstance().useInferredTypeMap.clear();
  ConstantValueMap::getInstance().shapeValueMap.clear();
  ConstantValueMap::getInstance().inferredShapeData.clear();
  ConstantValueMap::getInstance().symbolDimMap.clear();
  ConstantValueMap::getInstance().dimSymbolMap.clear();
  ConstantValueMap::getInstance().allGraphInputsStatic = std::nullopt;
  ConstantValueMap::getInstance().allGraphInputsReliableComputed = false;
}

```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `ClearMaps`, `getInstance`, `clear`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`ClearMaps`, `getInstance`, `clear`。

### Lines 280-287
```cpp
// For debug only.
void ConstantValueMap::PrintMaps() {
  std::cout << "Rank/Shape Map:" << '\n';
  for (const auto& x : ConstantValueMap::getInstance().rankMap) {
    std::stringstream ss;
    if (ConstantValueMap::getInstance().shapeMap.find(x.first) !=
        ConstantValueMap::getInstance().shapeMap.end()) {
      auto shape_symbols =
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `PrintMaps`, `getInstance`, `find`, `end`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`PrintMaps`, `getInstance`, `find`, `end`。

### Lines 288-300
```cpp
          ConstantValueMap::getInstance().shapeMap[x.first].sizes();
      if (shape_symbols.has_value()) {
        for (const auto& shape_symbol : shape_symbols.value()) {
          if (shape_symbol.is_static()) {
            ss << shape_symbol.static_size() << ", ";
          } else {
            ss << "*, ";
          }
        }
      }
    }
    ss << " (rank = " << x.second << ')';
    std::cout << "node " << x.first << ": " << ss.str() << '\n';
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `getInstance`, `sizes`, `has_value`, `value`, `is_static`, `static_size`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`getInstance`, `sizes`, `has_value`, `value`, `is_static`, `static_size`, `...`。

### Lines 301-307
```cpp
  }
  std::cout << '\n';
  std::cout << "Value Map:" << '\n';
  for (const auto& x : ConstantValueMap::getInstance().tensorValueMap) {
    std::cout << "node " << x.first << ": " << x.second << '\n';
  }
  std::cout << '\n';
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `getInstance`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`getInstance`。

### Lines 308-314
```cpp
  std::cout << "TypeReliable Map:" << '\n';
  size_t count = 0;
  for (const auto& x : ConstantValueMap::getInstance().typeReliableMap) {
    std::cout << "(node " << x.first << ": " << x.second << "), ";
    count++;
    if (count % 10 == 0) {
      std::cout << '\n';
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `getInstance`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`getInstance`。

### Lines 315-321
```cpp
    }
  }
  std::cout << '\n';
  std::cout << "UseInferredType Map:" << '\n';
  count = 0;
  for (const auto& x : ConstantValueMap::getInstance().useInferredTypeMap) {
    std::cout << "(node " << x.first << ": " << x.second << "), ";
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `getInstance`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`getInstance`。

### Lines 322-328
```cpp
    count++;
    if (count % 10 == 0) {
      std::cout << '\n';
    }
  }
  std::cout << '\n';
  std::cout << "ShapeValue Map:" << '\n';
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: no dominant local symbol names.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：无明显局部符号。

### Lines 329-337
```cpp
  count = 0;
  for (const auto& x : ConstantValueMap::getInstance().shapeValueMap) {
    std::cout << "(node " << x.first << ": " << x.second << "), ";
    count++;
    if (count % 10 == 0) {
      std::cout << '\n';
    }
  }
  std::cout << '\n';
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `getInstance`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`getInstance`。

### Lines 338-344
```cpp
  std::cout << "InferredShape Map:" << '\n';
  count = 0;
  for (const auto& x : ConstantValueMap::getInstance().inferredShapeData) {
    std::cout << "(node " << x.first << ": ";
    for (const auto& dim : x.second.dim()) {
      if (dim.has_dim_param()) {
        std::cout << dim.dim_param() << ' ';
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `getInstance`, `dim`, `has_dim_param`, `dim_param`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`getInstance`, `dim`, `has_dim_param`, `dim_param`。

### Lines 345-351
```cpp
      } else {
        std::cout << dim.dim_value() << ' ';
      }
    }
    std::cout << "), ";
    count++;
    if (count % 10 == 0) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `dim_value`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`dim_value`。

### Lines 352-358
```cpp
      std::cout << '\n';
    }
  }
  std::cout << '\n';
  std::cout << "SymbolDim Map:" << '\n';
  count = 0;
  for (const auto& x : ConstantValueMap::getInstance().symbolDimMap) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `getInstance`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`getInstance`。

### Lines 359-365
```cpp
    std::cout << '(' << x.first << ": " << x.second << "), ";
    count++;
    if (count % 10 == 0) {
      std::cout << '\n';
    }
  }
  std::cout << "DimSymbol Map:" << '\n';
```
- EN: This block handles conditional branches. Key symbols: no dominant local symbol names.
- CN: 该代码块处理条件分支。关键符号：无明显局部符号。

### Lines 366-375
```cpp
  count = 0;
  for (const auto& x : ConstantValueMap::getInstance().dimSymbolMap) {
    std::cout << '(' << x.first << ": " << x.second << "), ";
    count++;
    if (count % 10 == 0) {
      std::cout << '\n';
    }
  }
}

```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `getInstance`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`getInstance`。

### Lines 376-376
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
- EN: Type and shape reasoning — the code propagates static information across the IR.
  CN: 类型与形状推理——代码会在 IR 中传播静态信息。
- EN: Export interop — the implementation helps bridge TorchScript graphs to ONNX-style representations.
  CN: 导出互操作——实现帮助 TorchScript 图与 ONNX 风格表示之间建立桥接。
- EN: Subsystem integration — the implementation collaborates with nearby JIT/frontend headers and runtime helpers.
  CN: 子系统集成——实现会与附近的 JIT/前端头文件以及运行时辅助模块协同工作。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `c10/util/irange.h`, `torch/csrc/jit/passes/onnx/constant_map.h`
- External includes / 外部头文件: `iostream`, `sstream`, `string`, `unordered_map`
- Namespaces / 命名空间: `torch::jit`
- Representative symbols / 代表性符号: `getInstance`, `SetRank`, `HasRank`, `find`, `end`, `GetRank`, `SetAllGraphInputsStatic`, `GetAllGraphInputsStatic`, `SetAllGraphInputsReliableComputed`, `GetAllGraphInputsReliableComputed`, `...`
