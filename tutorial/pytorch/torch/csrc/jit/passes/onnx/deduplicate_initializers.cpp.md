# deduplicate_initializers.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/passes/onnx/deduplicate_initializers.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements the TorchScript JIT pass logic for deduplicate initializers, including graph analysis and rewrites.
- 用途 (CN): 实现与 deduplicate initializers 相关的 TorchScript JIT pass 逻辑，包括图分析与改写。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7
```cpp
#include <torch/csrc/jit/jit_log.h>
#include <torch/csrc/jit/passes/onnx/deduplicate_initializers.h>
#include <torch/csrc/jit/passes/onnx/helper.h>

#include <c10/util/hash.h>
#include <c10/util/irange.h>

```
- EN: Pulls in the headers needed by the deduplicate initializers logic. Internal dependencies: `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/onnx/deduplicate_initializers.h`, `torch/csrc/jit/passes/onnx/helper.h`, `c10/util/hash.h`, `c10/util/irange.h`; external dependencies: none.
- CN: 为 deduplicate initializers 相关逻辑引入所需头文件。内部依赖：`torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/onnx/deduplicate_initializers.h`, `torch/csrc/jit/passes/onnx/helper.h`, `c10/util/hash.h`, `c10/util/irange.h`；外部依赖：无。

### Lines 8-12
```cpp
#include <functional>
#include <unordered_set>

namespace torch::jit {

```
- EN: Opens or closes namespace scopes so the remaining declarations live in the expected JIT/frontend module.
- CN: 打开或关闭命名空间作用域，使后续声明归属于预期的 JIT/前端模块。

### Lines 13-17
```cpp
namespace onnx {
using namespace ::c10::onnx;
}

struct HashValue {
```
- EN: Opens or closes namespace scopes so the remaining declarations live in the expected JIT/frontend module.
- CN: 打开或关闭命名空间作用域，使后续声明归属于预期的 JIT/前端模块。

### Lines 18-22
```cpp
  HashValue(ValueToParamPairMap& valsToParamsMap, bool compare_by_ptr)
      : valsToParamsMap_(valsToParamsMap), compare_by_ptr_(compare_by_ptr) {}

  size_t operator()(Value* v) const {
    auto t = valsToParamsMap_.find(v)->second.second.toTensor();
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `HashValue`, `valsToParamsMap_`, `compare_by_ptr_`, `find`, `toTensor`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`HashValue`, `valsToParamsMap_`, `compare_by_ptr_`, `find`, `toTensor`。

### Lines 23-29
```cpp

    if (compare_by_ptr_) {
      // Hash by metadata + data pointer
      return at::get_hash(
          t.sizes(), t.strides(), t.has_storage() ? t.data_ptr() : 0);
    }

```
- EN: This block handles conditional branches; produces a result or forwards a computed value. Key symbols: `get_hash`, `sizes`, `strides`, `has_storage`, `data_ptr`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值。关键符号：`get_hash`, `sizes`, `strides`, `has_storage`, `data_ptr`。

### Lines 30-36
```cpp
    // Hash by metadata + first element value (if exists). This is a fast
    // approximation of hashing by the whole tensor value, which can be
    // expensive for large tensors.
    double first_elem_double = 0.0;
    int64_t first_elem_int64 = 0;
    uint64_t first_elem_uint64 = 0;

```
- EN: This block manipulates TorchScript IR objects. Key symbols: no dominant local symbol names.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：无明显局部符号。

### Lines 37-43
```cpp
    if (t.numel() > 0 && t.has_storage()) {
      auto scalar = t.reshape(-1)[0].item();

      if (scalar.isFloatingPoint()) {
        first_elem_double = scalar.to<double>();
      } else if (scalar.isIntegral(/*includeBool=*/true)) {
        if (scalar.isUnsigned()) {
```
- EN: This block handles conditional branches. Key symbols: `numel`, `has_storage`, `reshape`, `item`, `isFloatingPoint`, `isIntegral`, `...`.
- CN: 该代码块处理条件分支。关键符号：`numel`, `has_storage`, `reshape`, `item`, `isFloatingPoint`, `isIntegral`, `...`。

### Lines 44-50
```cpp
          first_elem_uint64 = scalar.to<uint64_t>();
        } else {
          first_elem_int64 = scalar.to<int64_t>();
        }
      }
    }

```
- EN: This block implements local helper logic for deduplicate initializers. Key symbols: no dominant local symbol names.
- CN: 该代码块实现与 deduplicate initializers 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 51-58
```cpp
    return at::get_hash(
        first_elem_double,
        first_elem_int64,
        first_elem_uint64,
        t.sizes(),
        t.strides());
  }

```
- EN: This block produces a result or forwards a computed value. Key symbols: `get_hash`, `sizes`, `strides`.
- CN: 该代码块返回结果或转发已计算的值。关键符号：`get_hash`, `sizes`, `strides`。

### Lines 59-63
```cpp
 private:
  ValueToParamPairMap& valsToParamsMap_;
  bool compare_by_ptr_;
};

```
- EN: This block manipulates TorchScript IR objects. Key symbols: no dominant local symbol names.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：无明显局部符号。

### Lines 64-68
```cpp
struct CompareValue {
  CompareValue(std::function<bool(Value*, Value*)> is_same_tensor_as)
      : is_same_tensor_as_(is_same_tensor_as) {}

  bool operator()(Value* v1, Value* v2) const {
```
- EN: Declares core types or data containers for this file. Prominent symbols: `CompareValue`, `bool`, `is_same_tensor_as_`.
- CN: 声明该文件使用的核心类型或数据容器。主要符号：`CompareValue`, `bool`, `is_same_tensor_as_`。

### Lines 69-73
```cpp
    return is_same_tensor_as_(v1, v2);
  }

 private:
  std::function<bool(Value*, Value*)> is_same_tensor_as_;
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `is_same_tensor_as_`, `bool`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`is_same_tensor_as_`, `bool`。

### Lines 74-78
```cpp
};

// forward declaration
static bool DeduplicateInitializersByDataPtr(at::Tensor& t1, at::Tensor& t2);

```
- EN: This block implements local helper logic for deduplicate initializers. Key symbols: `DeduplicateInitializersByDataPtr`.
- CN: 该代码块实现与 deduplicate initializers 相关的局部辅助逻辑。关键符号：`DeduplicateInitializersByDataPtr`。

### Lines 79-83
```cpp
static void DeduplicateInitializers(
    std::shared_ptr<Graph>& g,
    ValueToParamPairMap& valsToParamsMap,
    bool (*comp)(at::Tensor&, at::Tensor&)) {
  auto is_same_tensor_as = [&valsToParamsMap, comp](Value* v1, Value* v2) {
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `DeduplicateInitializers`, `bool`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`DeduplicateInitializers`, `bool`。

### Lines 84-88
```cpp
    auto t1 = valsToParamsMap.find(v1)->second.second.toTensor();
    auto t2 = valsToParamsMap.find(v2)->second.second.toTensor();
    return comp(t1, t2);
  };

```
- EN: This block produces a result or forwards a computed value. Key symbols: `find`, `toTensor`, `comp`.
- CN: 该代码块返回结果或转发已计算的值。关键符号：`find`, `toTensor`, `comp`。

### Lines 89-94
```cpp
  bool compare_by_ptr = comp == &DeduplicateInitializersByDataPtr;
  std::unordered_set<Value*, HashValue, CompareValue> uniqueVals(
      0,
      HashValue(valsToParamsMap, compare_by_ptr),
      CompareValue(is_same_tensor_as));
  std::vector<size_t> inputsIndicesToRemove;
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `uniqueVals`, `HashValue`, `CompareValue`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`uniqueVals`, `HashValue`, `CompareValue`。

### Lines 95-99
```cpp
  auto b = g->block();

  for (auto i : c10::irange(b->inputs().size())) {
    auto v = g->inputs().at(i);
    auto vals_to_param_it = valsToParamsMap.find(v);
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `block`, `irange`, `inputs`, `size`, `find`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`block`, `irange`, `inputs`, `size`, `find`。

### Lines 100-105
```cpp

    // Skip parameters without initializers
    if (vals_to_param_it == valsToParamsMap.end()) {
      continue;
    }

```
- EN: This block handles conditional branches. Key symbols: `end`.
- CN: 该代码块处理条件分支。关键符号：`end`。

### Lines 106-110
```cpp
    // Skip non-tensors
    if (!vals_to_param_it->second.second.isTensor()) {
      continue;
    }

```
- EN: This block handles conditional branches. Key symbols: `isTensor`.
- CN: 该代码块处理条件分支。关键符号：`isTensor`。

### Lines 111-115
```cpp
    auto it = uniqueVals.insert(v);
    if (!it.second) {
      // Same value already exists
      inputsIndicesToRemove.emplace_back(i);
      auto id_node = g->create(onnx::Identity);
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `insert`, `emplace_back`, `create`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`insert`, `emplace_back`, `create`。

### Lines 116-123
```cpp
      id_node->insertAfter(g->block()->param_node());
      id_node->addInput(*it.first);
      id_node->output()->copyMetadata(v);
      id_node->copyMetadata(g->block()->param_node());
      v->replaceAllUsesWith(id_node->output());
    }
  }
  for (auto it = inputsIndicesToRemove.rbegin();
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `insertAfter`, `block`, `param_node`, `addInput`, `output`, `copyMetadata`, `...`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`insertAfter`, `block`, `param_node`, `addInput`, `output`, `copyMetadata`, `...`。

### Lines 124-130
```cpp
       it != inputsIndicesToRemove.rend();
       ++it) {
    valsToParamsMap.erase(g->inputs().at(*it));
    g->eraseInput(*it);
  }
}

```
- EN: This block implements local helper logic for deduplicate initializers. Key symbols: `rend`, `erase`, `inputs`, `eraseInput`.
- CN: 该代码块实现与 deduplicate initializers 相关的局部辅助逻辑。关键符号：`rend`, `erase`, `inputs`, `eraseInput`。

### Lines 131-135
```cpp
static bool DeduplicateInitializersByDataPtr(at::Tensor& t1, at::Tensor& t2) {
  return t1.sizes().equals(t2.sizes()) && t1.strides().equals(t2.strides()) &&
      (t1.has_storage() && t2.has_storage() && t1.data_ptr() == t2.data_ptr());
}

```
- EN: This block produces a result or forwards a computed value. Key symbols: `DeduplicateInitializersByDataPtr`, `sizes`, `equals`, `strides`, `has_storage`, `data_ptr`.
- CN: 该代码块返回结果或转发已计算的值。关键符号：`DeduplicateInitializersByDataPtr`, `sizes`, `equals`, `strides`, `has_storage`, `data_ptr`。

### Lines 136-141
```cpp
static bool DeduplicateInitializersByValue(at::Tensor& t1, at::Tensor& t2) {
  if (t1.dtype() != t2.dtype() || !t1.sizes().equals(t2.sizes()) ||
      !t1.strides().equals(t2.strides())) {
    return false;
  }

```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `DeduplicateInitializersByValue`, `dtype`, `sizes`, `equals`, `strides`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`DeduplicateInitializersByValue`, `dtype`, `sizes`, `equals`, `strides`。

### Lines 142-146
```cpp
  if (t1.device() != t2.device()) {
    return t1.to("cpu").equal(t2.to("cpu"));
  }

  return t1.equal(t2);
```
- EN: This block handles conditional branches; produces a result or forwards a computed value. Key symbols: `device`, `to`, `equal`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值。关键符号：`device`, `to`, `equal`。

### Lines 147-151
```cpp
}

void DeduplicateInitializers(
    std::shared_ptr<Graph>& g,
    std::map<std::string, IValue>& paramsDict,
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `DeduplicateInitializers`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`DeduplicateInitializers`。

### Lines 152-157
```cpp
    bool is_train) {
  auto valsToParamsMap = buildValueToParamsMap(g->block(), paramsDict);
  // ONNX spec does not support parameters with shared memory.
  // This pass de-duplicate those parameters. Training is not affected.
  DeduplicateInitializers(g, valsToParamsMap, DeduplicateInitializersByDataPtr);
  if (!is_train) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `buildValueToParamsMap`, `block`, `DeduplicateInitializers`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`buildValueToParamsMap`, `block`, `DeduplicateInitializers`。

### Lines 158-166
```cpp
    // More aggressive parameters de-duplication based on tensor values.
    // Producing more compact model for inference.
    // For training, this pass is disabled,
    // because parameters may be updated differently.
    DeduplicateInitializers(g, valsToParamsMap, DeduplicateInitializersByValue);
  }
  buildParamsMapFromValueToParamsMap(valsToParamsMap, paramsDict);
}

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `DeduplicateInitializers`, `buildParamsMapFromValueToParamsMap`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`DeduplicateInitializers`, `buildParamsMapFromValueToParamsMap`。

### Lines 167-167
```cpp
} // namespace torch::jit
```
- EN: Opens or closes namespace scopes so the remaining declarations live in the expected JIT/frontend module.
- CN: 打开或关闭命名空间作用域，使后续声明归属于预期的 JIT/前端模块。


## Key Concepts / 关键概念
- EN: IR graph manipulation — the file inspects or rewrites TorchScript graphs and nodes.
  CN: IR 图操作——该文件会检查或改写 TorchScript 的图、节点与块。
- EN: Fusion support — the file prepares groups of operations for fused execution.
  CN: 融合支持——该文件会为融合执行准备算子分组。
- EN: Type and shape reasoning — the code propagates static information across the IR.
  CN: 类型与形状推理——代码会在 IR 中传播静态信息。
- EN: Export interop — the implementation helps bridge TorchScript graphs to ONNX-style representations.
  CN: 导出互操作——实现帮助 TorchScript 图与 ONNX 风格表示之间建立桥接。
- EN: Pass orchestration — the file plugs logic into the TorchScript optimization pipeline.
  CN: Pass 编排——该文件把相关逻辑接入 TorchScript 优化流水线。
- EN: Subsystem integration — the implementation collaborates with nearby JIT/frontend headers and runtime helpers.
  CN: 子系统集成——实现会与附近的 JIT/前端头文件以及运行时辅助模块协同工作。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/onnx/deduplicate_initializers.h`, `torch/csrc/jit/passes/onnx/helper.h`, `c10/util/hash.h`, `c10/util/irange.h`
- External includes / 外部头文件: `functional`, `unordered_set`
- Namespaces / 命名空间: `torch::jit`, `onnx`
- Representative symbols / 代表性符号: `namespace`, `HashValue`, `valsToParamsMap_`, `compare_by_ptr_`, `find`, `toTensor`, `get_hash`, `sizes`, `strides`, `has_storage`, `...`
