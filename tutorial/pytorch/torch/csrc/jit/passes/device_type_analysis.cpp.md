# device_type_analysis.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/passes/device_type_analysis.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements the TorchScript JIT pass logic for device type analysis, including graph analysis and rewrites.
- 用途 (CN): 实现与 device type analysis 相关的 TorchScript JIT pass 逻辑，包括图分析与改写。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12
```cpp
#include <ATen/core/interned_strings.h>
#include <ATen/core/jit_type.h>
#include <c10/core/Device.h>
#include <c10/util/ArrayRef.h>
#include <torch/csrc/jit/ir/ir.h>
#include <torch/csrc/jit/jit_log.h>
#include <torch/csrc/jit/passes/device_type_analysis.h>
#include <torch/csrc/jit/passes/shape_analysis.h>
#include <memory>
#include <optional>
#include <utility>

```
- EN: Pulls in the headers needed by the device type analysis logic. Internal dependencies: `ATen/core/interned_strings.h`, `ATen/core/jit_type.h`, `c10/core/Device.h`, `c10/util/ArrayRef.h`, `torch/csrc/jit/ir/ir.h`, `...`; external dependencies: `memory`, `optional`, `utility`.
- CN: 为 device type analysis 相关逻辑引入所需头文件。内部依赖：`ATen/core/interned_strings.h`, `ATen/core/jit_type.h`, `c10/core/Device.h`, `c10/util/ArrayRef.h`, `torch/csrc/jit/ir/ir.h`, `...`；外部依赖：`memory`, `optional`, `utility`。

### Lines 13-19
```cpp
namespace torch::jit {

namespace {

using Tensor = at::Tensor;
using Device = at::Device;

```
- EN: Opens or closes namespace scopes so the remaining declarations live in the expected JIT/frontend module.
- CN: 打开或关闭命名空间作用域，使后续声明归属于预期的 JIT/前端模块。

### Lines 20-28
```cpp
using PropRule = std::function<bool(Node*)>;
/*
A Propagation Rule takes the Node, and
applies the relevant properties to the Tensor outputs
of the Node (based on the rule itself)

Returns: Bool indicating if anything was changed
*/

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `bool`, `PropRule`, `Node`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`bool`, `PropRule`, `Node`。

### Lines 29-35
```cpp
bool setDeviceType(Value* value, std::optional<Device> device) {
  auto tensor_type = value->type()->expect<TensorType>();
  bool changed = tensor_type->device() != device;
  if (changed) {
    value->setType(tensor_type->withDevice(device));
  }
  return changed;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `setDeviceType`, `type`, `device`, `setType`, `withDevice`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`setDeviceType`, `type`, `device`, `setType`, `withDevice`。

### Lines 36-42
```cpp
}

bool setReturnsToDevice(Node* n, std::optional<Device> device) {
  bool changed = false;
  for (Value* out : n->outputs()) {
    auto tensor_type = out->type()->cast<TensorType>();
    if (!tensor_type) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `setReturnsToDevice`, `outputs`, `type`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`setReturnsToDevice`, `outputs`, `type`。

### Lines 43-49
```cpp
      continue;
    }
    changed |= setDeviceType(out, device);
  }
  return changed;
}

```
- EN: This block produces a result or forwards a computed value. Key symbols: `setDeviceType`.
- CN: 该代码块返回结果或转发已计算的值。关键符号：`setDeviceType`。

### Lines 50-57
```cpp
PropRule setReturnstoDeviceRule(DeviceType deviceType) {
  Device device = Device(deviceType);
  return [=](Node* n) { return setReturnsToDevice(n, device); };
}

bool returnFirstArgDeviceRule(Node* n) {
  // Custom Rule for when multiple args can have mismatched device types
  auto tensor_type = n->inputs()[0]->type()->cast<TensorType>();
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `setReturnstoDeviceRule`, `Device`, `setReturnsToDevice`, `returnFirstArgDeviceRule`, `inputs`, `type`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`setReturnstoDeviceRule`, `Device`, `setReturnsToDevice`, `returnFirstArgDeviceRule`, `inputs`, `type`。

### Lines 58-64
```cpp
  TORCH_INTERNAL_ASSERT(tensor_type, "Expecting a tensor type");
  return setReturnsToDevice(n, tensor_type->device());
}

bool returnSecondArgDeviceRule(Node* n) {
  // Custom Rule for when multiple args can have mismatched device types
  auto tensor_type = n->inputs()[1]->type()->cast<TensorType>();
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `setReturnsToDevice`, `device`, `returnSecondArgDeviceRule`, `inputs`, `type`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`setReturnsToDevice`, `device`, `returnSecondArgDeviceRule`, `inputs`, `type`。

### Lines 65-73
```cpp
  TORCH_INTERNAL_ASSERT(tensor_type, "Expecting a tensor type");
  return setReturnsToDevice(n, tensor_type->device());
}

bool isZerodimCPUTensor(const std::shared_ptr<TensorType>& tensor_type) {
  // CPU devices on zerodim tensors are the only device that can be
  // overwritten by another device. Therefore, to be conservative
  // assume that it is not a zerodim cpu tensor if something is not known.
  bool is_zerodim = tensor_type->symbolic_sizes().rank().value_or(-1) == 0;
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `setReturnsToDevice`, `device`, `isZerodimCPUTensor`, `symbolic_sizes`, `rank`, `value_or`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`setReturnsToDevice`, `device`, `isZerodimCPUTensor`, `symbolic_sizes`, `rank`, `value_or`。

### Lines 74-82
```cpp
  bool is_cpu = tensor_type->device() && tensor_type->device()->is_cpu();
  return is_zerodim && is_cpu;
}

bool propWithNoDevice(Node* n) {
  // Propagate if we can verify that all input devices match,
  // except CPU zerodim, which any other type can overwrite
  size_t input_num = 0;

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `device`, `is_cpu`, `propWithNoDevice`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`device`, `is_cpu`, `propWithNoDevice`。

### Lines 83-90
```cpp
  for (; input_num < n->inputs().size(); input_num++) {
    if (n->inputs()[input_num]->type()->cast<TensorType>()) {
      break;
    }
  }
  if (input_num == n->inputs().size()) {
    // No tensor found
    return setReturnsToDevice(n, std::nullopt);
```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value. Key symbols: `inputs`, `size`, `type`, `setReturnsToDevice`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值。关键符号：`inputs`, `size`, `type`, `setReturnsToDevice`。

### Lines 91-98
```cpp
  }

  auto tensor_type = n->inputs()[input_num]->type()->expect<TensorType>();
  bool only_seen_cpu_zerodim = isZerodimCPUTensor(tensor_type);
  std::optional<Device> device = tensor_type->device();

  // Now see if all inputs have a consistent device type
  for (input_num++; input_num < n->inputs().size(); input_num++) {
```
- EN: This block iterates over collections or graph structures. Key symbols: `inputs`, `type`, `isZerodimCPUTensor`, `device`, `size`.
- CN: 该代码块遍历集合或图结构。关键符号：`inputs`, `type`, `isZerodimCPUTensor`, `device`, `size`。

### Lines 99-105
```cpp
    auto tensor_type = n->inputs()[input_num]->type()->cast<TensorType>();
    if (!tensor_type || isZerodimCPUTensor(tensor_type)) {
      continue;
    }

    if (device != tensor_type->device()) {
      if (only_seen_cpu_zerodim) {
```
- EN: This block handles conditional branches. Key symbols: `inputs`, `type`, `isZerodimCPUTensor`, `device`.
- CN: 该代码块处理条件分支。关键符号：`inputs`, `type`, `isZerodimCPUTensor`, `device`。

### Lines 106-114
```cpp
        device = tensor_type->device();
        only_seen_cpu_zerodim = false;
      } else {
        // Bail on the type not match case
        return setReturnsToDevice(n, std::nullopt);
      }
    }
  }
  return setReturnsToDevice(n, device);
```
- EN: This block produces a result or forwards a computed value. Key symbols: `device`, `setReturnsToDevice`.
- CN: 该代码块返回结果或转发已计算的值。关键符号：`device`, `setReturnsToDevice`。

### Lines 115-121
```cpp
}

bool defaultDeviceProp(Node* n) {
  // Detecting if the op has a device object argument
  // as there is implicit string conversion to device
  auto schema = n->maybeSchema();
  if (!schema) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `defaultDeviceProp`, `maybeSchema`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`defaultDeviceProp`, `maybeSchema`。

### Lines 122-129
```cpp
    return false;
  }
  auto arguments = schema->arguments();
  for (size_t i = 0; i < arguments.size(); i++) {
    Argument& argument = arguments[i];
    if (DeviceObjType::get()->isSubtypeOf(argument.type())) {
      // Optional args are filled in by torchscript with default val
      auto input_val = toIValue(n->inputs().at(i));
```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `arguments`, `size`, `get`, `isSubtypeOf`, `type`, `toIValue`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`arguments`, `size`, `get`, `isSubtypeOf`, `type`, `toIValue`, `...`。

### Lines 130-137
```cpp
      if (!input_val.has_value()) {
        // Can't propagate if there is a dynamic device type
        return false;
      }
      if (input_val->isNone()) {
        continue;
      }
      if (!input_val->isDevice()) {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `has_value`, `isNone`, `isDevice`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`has_value`, `isNone`, `isDevice`。

### Lines 138-146
```cpp
        // Bail on union types
        return false;
      }
      TORCH_INTERNAL_ASSERT(input_val->isDevice())
      Device device = input_val->toDevice();
      return setReturnsToDevice(n, device);
    }
  }
  return propWithNoDevice(n);
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `isDevice`, `toDevice`, `setReturnsToDevice`, `propWithNoDevice`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`isDevice`, `toDevice`, `setReturnsToDevice`, `propWithNoDevice`。

### Lines 147-154
```cpp
}

struct DeviceTypePropagationPass : public PropertyPropBase {
  explicit DeviceTypePropagationPass(std::shared_ptr<Graph> graph)
      : PropertyPropBase(std::move(graph)) {
    buildRuleRegistry();
  }

```
- EN: Declares core types or data containers for this file. Prominent symbols: `DeviceTypePropagationPass`, `PropertyPropBase`, `move`, `buildRuleRegistry`.
- CN: 声明该文件使用的核心类型或数据容器。主要符号：`DeviceTypePropagationPass`, `PropertyPropBase`, `move`, `buildRuleRegistry`。

### Lines 155-162
```cpp
  // returns true if at least one node has its scalar type set on a tensor node
  bool run() {
    propagateBlock(graph_->block(), false);
    return changed_;
  }

 private:
  void propagateNode(Node* n, bool _ = true) override {
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `run`, `propagateBlock`, `block`, `propagateNode`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`run`, `propagateBlock`, `block`, `propagateNode`。

### Lines 163-175
```cpp
    GRAPH_DEBUG("processNode");
    switch (n->kind()) {
      case prim::If:
        return processIf(n);
      case prim::Loop:
        return processLoop(n);
      case prim::CallMethod:
      case prim::CallFunction:
        return; // Not handled for now
      default:
        break;
    }

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `kind`, `processIf`, `processLoop`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`kind`, `processIf`, `processLoop`。

### Lines 176-185
```cpp
    bool has_tensor_output =
        std::any_of(n->outputs().begin(), n->outputs().end(), [](Value* v) {
          return (bool)v->type()->cast<TensorType>();
        });

    if (!has_tensor_output) {
      // if output contains no tensor, nothing to propagate
      return;
    }

```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `any_of`, `outputs`, `begin`, `end`, `type`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`any_of`, `outputs`, `begin`, `end`, `type`。

### Lines 186-193
```cpp
    switch (n->kind()) {
      case prim::Constant:
        // This is already been propagated by something else
      case prim::ListConstruct:
      case prim::ListUnpack:
        return; // Not handled for now
      default:
        if (n->kind().is_aten()) {
```
- EN: This block handles conditional branches; performs optimization-oriented rewriting. Key symbols: `kind`, `is_aten`.
- CN: 该代码块处理条件分支；执行面向优化的改写。关键符号：`kind`, `is_aten`。

### Lines 194-200
```cpp
          return processAtenOps(n);
        } else {
          return; // Not handled for now
        }
    }
  }

```
- EN: This block produces a result or forwards a computed value. Key symbols: `processAtenOps`.
- CN: 该代码块返回结果或转发已计算的值。关键符号：`processAtenOps`。

### Lines 201-209
```cpp
  void processAtenOps(Node* n) {
    GRAPH_DEBUG("processAtenOps");
    GRAPH_DEBUG("case = ", n->kind(), " ", *n);
    // Custom Rule Matching
    auto op = n->maybeOperator();
    if (!op) {
      return;
    }
    auto prop_fn = device_prop_registry_->find(*op);
```
- EN: Registers callable entities, pass hooks, or exported interfaces so runtime code can discover device type analysis behavior. Symbols: `processAtenOps`, `kind`, `maybeOperator`, `find`.
- CN: 注册可调用实体、pass 钩子或导出接口，使运行时能够发现 device type analysis 的行为。符号：`processAtenOps`, `kind`, `maybeOperator`, `find`。

### Lines 210-217
```cpp
    if (prop_fn) {
      PropRule rule = *prop_fn;
      changed_ |= rule(n);
      return;
    }
    changed_ |= defaultDeviceProp(n);
  }

```
- EN: This block handles conditional branches. Key symbols: `rule`, `defaultDeviceProp`.
- CN: 该代码块处理条件分支。关键符号：`rule`, `defaultDeviceProp`。

### Lines 218-231
```cpp
  void buildRuleRegistry() {
    // building a registry for all of the custom Device Type rules
    if (device_prop_registry_)
      return;

    static OperatorMap<PropRule> temp_registry{
        {"aten::cpu(Tensor self) -> Tensor",
         setReturnstoDeviceRule(DeviceType::CPU)},
        {"aten::cuda(Tensor self) -> Tensor",
         setReturnstoDeviceRule(DeviceType::CUDA)},
        {"aten::to_mkldnn(Tensor self, ScalarType? dtype) -> Tensor",
         setReturnstoDeviceRule(DeviceType::MKLDNN)},
        {"aten::reshape_as(Tensor self, Tensor other) -> Tensor",
         returnFirstArgDeviceRule},
```
- EN: This block handles conditional branches. Key symbols: `buildRuleRegistry`, `cpu`, `setReturnstoDeviceRule`, `cuda`, `to_mkldnn`, `reshape_as`.
- CN: 该代码块处理条件分支。关键符号：`buildRuleRegistry`, `cpu`, `setReturnstoDeviceRule`, `cuda`, `to_mkldnn`, `reshape_as`。

### Lines 232-238
```cpp
        {"aten::view_as(Tensor self, Tensor other) -> Tensor",
         returnFirstArgDeviceRule},
        {"aten::expand_as(Tensor self, Tensor other) -> Tensor",
         returnFirstArgDeviceRule},
        {"aten::type_as(Tensor self, Tensor other) -> Tensor",
         returnSecondArgDeviceRule},
    };
```
- EN: This block implements local helper logic for device type analysis. Key symbols: `view_as`, `expand_as`, `type_as`.
- CN: 该代码块实现与 device type analysis 相关的局部辅助逻辑。关键符号：`view_as`, `expand_as`, `type_as`。

### Lines 239-245
```cpp
    device_prop_registry_ =
        std::make_unique<OperatorMap<PropRule>>(std::move(temp_registry));
  }

  static std::unique_ptr<OperatorMap<PropRule>> device_prop_registry_;
  bool changed_ = false;
};
```
- EN: This block implements local helper logic for device type analysis. Key symbols: `move`.
- CN: 该代码块实现与 device type analysis 相关的局部辅助逻辑。关键符号：`move`。

### Lines 246-254
```cpp

std::unique_ptr<OperatorMap<PropRule>>
    DeviceTypePropagationPass::device_prop_registry_ = nullptr;

} // anonymous namespace

// This analysis propagates input device types (if any) throughout the
// graph.
bool DeviceTypePropagation(std::shared_ptr<Graph>& graph) {
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `DeviceTypePropagation`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`DeviceTypePropagation`。

### Lines 255-262
```cpp
  auto tp = std::make_unique<DeviceTypePropagationPass>(graph);
  bool changed = tp->run();
  if (changed) {
    GRAPH_DUMP("After TensorPropertyPropagation pass:", graph);
  }
  return changed;
}

```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `run`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`run`。

### Lines 263-263
```cpp
} // namespace torch::jit
```
- EN: Opens or closes namespace scopes so the remaining declarations live in the expected JIT/frontend module.
- CN: 打开或关闭命名空间作用域，使后续声明归属于预期的 JIT/前端模块。


## Key Concepts / 关键概念
- EN: IR graph manipulation — the file inspects or rewrites TorchScript graphs and nodes.
  CN: IR 图操作——该文件会检查或改写 TorchScript 的图、节点与块。
- EN: Schema handling — the implementation parses or normalizes operator/function signatures.
  CN: 模式处理——实现会解析或规范化算子/函数签名。
- EN: Optimization pass — the code simplifies graphs by removing redundant work or folding values.
  CN: 优化 pass——代码通过删除冗余计算或常量折叠来简化图。
- EN: Fusion support — the file prepares groups of operations for fused execution.
  CN: 融合支持——该文件会为融合执行准备算子分组。
- EN: Type and shape reasoning — the code propagates static information across the IR.
  CN: 类型与形状推理——代码会在 IR 中传播静态信息。
- EN: Subsystem integration — the implementation collaborates with nearby JIT/frontend headers and runtime helpers.
  CN: 子系统集成——实现会与附近的 JIT/前端头文件以及运行时辅助模块协同工作。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `ATen/core/interned_strings.h`, `ATen/core/jit_type.h`, `c10/core/Device.h`, `c10/util/ArrayRef.h`, `torch/csrc/jit/ir/ir.h`, `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/device_type_analysis.h`, `torch/csrc/jit/passes/shape_analysis.h`
- External includes / 外部头文件: `memory`, `optional`, `utility`
- Namespaces / 命名空间: `torch::jit`
- Representative symbols / 代表性符号: `Tensor`, `Device`, `bool`, `PropRule`, `Node`, `setDeviceType`, `type`, `device`, `setType`, `withDevice`, `...`
