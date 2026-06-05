# lower_graph.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/passes/lower_graph.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements the TorchScript JIT pass logic for lower graph, including graph analysis and rewrites.
- 用途 (CN): 实现与 lower graph 相关的 TorchScript JIT pass 逻辑，包括图分析与改写。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8
```cpp
#include <torch/csrc/jit/passes/lower_graph.h>

#include <torch/csrc/jit/api/object.h>
#include <torch/csrc/jit/frontend/error_report.h>
#include <torch/csrc/jit/passes/inliner.h>
#include <torch/custom_class.h>
#include <unordered_map>

```
- EN: Pulls in the headers needed by the lower graph logic. Internal dependencies: `torch/csrc/jit/passes/lower_graph.h`, `torch/csrc/jit/api/object.h`, `torch/csrc/jit/frontend/error_report.h`, `torch/csrc/jit/passes/inliner.h`, `torch/custom_class.h`; external dependencies: `unordered_map`.
- CN: 为 lower graph 相关逻辑引入所需头文件。内部依赖：`torch/csrc/jit/passes/lower_graph.h`, `torch/csrc/jit/api/object.h`, `torch/csrc/jit/frontend/error_report.h`, `torch/csrc/jit/passes/inliner.h`, `torch/custom_class.h`；外部依赖：`unordered_map`。

### Lines 9-14
```cpp
namespace torch::jit {

struct Slot {
  c10::intrusive_ptr<c10::ivalue::Object> obj;
  size_t offset;
  bool operator==(const Slot& other) const {
```
- EN: Declares core types or data containers for this file. Prominent symbols: `Slot`.
- CN: 声明该文件使用的核心类型或数据容器。主要符号：`Slot`。

### Lines 15-23
```cpp
    return (this->obj == other.obj && this->offset == other.offset);
  }
};

// remove the first module argument, replacing any access of its
// parameters/attributes with extra_ivalue input Slots that hold what value to
// pass into the graph. Used for ONNX export to remove first-class modules
// so it can deal purely with parameters and inputs
static std::pair<std::shared_ptr<Graph>, std::vector<Slot>> lower_graph(
```
- EN: Declares core types or data containers for this file. Prominent symbols: `lower_graph`.
- CN: 声明该文件使用的核心类型或数据容器。主要符号：`lower_graph`。

### Lines 24-30
```cpp
    const ModulePtr& self,
    Graph& g_,
    size_t self_offset = 0) {
  std::shared_ptr<Graph> g = g_.copy();
  // Inline to remove method/function calls
  Inline(*g);

```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `copy`, `Inline`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`copy`, `Inline`。

### Lines 31-35
```cpp
  std::vector<Slot> extra_ivalues;

  struct SlotHash {
    std::size_t operator()(const Slot& slot) const {
      auto obj_hash = std::hash<c10::ivalue::Object*>{}(slot.obj.get());
```
- EN: Declares core types or data containers for this file. Prominent symbols: `SlotHash`, `get`.
- CN: 声明该文件使用的核心类型或数据容器。主要符号：`SlotHash`, `get`。

### Lines 36-40
```cpp
      auto offset_hash = std::hash<size_t>{}(slot.offset);
      return c10::hash_combine(obj_hash, offset_hash);
    }
  };
  std::unordered_map<Slot, size_t, SlotHash> slot_to_offset;
```
- EN: This block produces a result or forwards a computed value. Key symbols: `hash_combine`.
- CN: 该代码块返回结果或转发已计算的值。关键符号：`hash_combine`。

### Lines 41-45
```cpp
  struct ToScan {
    ModulePtr mod;
    Node* n;
    size_t offset;
  };
```
- EN: Declares core types or data containers for this file. Prominent symbols: `ToScan`.
- CN: 声明该文件使用的核心类型或数据容器。主要符号：`ToScan`。

### Lines 46-50
```cpp
  std::vector<ToScan> to_scan;
  std::vector<Node*> to_clean; // nodes that should be dead at the end

  auto getOrAddSlot = [&](const Slot& slot) -> Value* {
    auto it = slot_to_offset.find(slot);
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `find`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`find`。

### Lines 51-57
```cpp
    if (it != slot_to_offset.end()) {
      size_t ivalues_start = g->inputs().size() - extra_ivalues.size();
      return g->inputs().at(ivalues_start + it->second);
    }
    extra_ivalues.emplace_back(slot);
    slot_to_offset[slot] = extra_ivalues.size() - 1;
    return g->addInput()->setType(slot.obj->getSlot(slot.offset).type());
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `end`, `inputs`, `size`, `emplace_back`, `addInput`, `setType`, `...`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`end`, `inputs`, `size`, `emplace_back`, `addInput`, `setType`, `...`。

### Lines 58-62
```cpp
  };

  auto self_value = g->inputs().at(self_offset);

  for (Use use : self_value->uses()) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `inputs`, `uses`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`inputs`, `uses`。

### Lines 63-68
```cpp
    to_scan.emplace_back(ToScan{self, use.user, use.offset});
  }
  while (!to_scan.empty()) {
    auto e = to_scan.back();
    to_scan.pop_back();

```
- EN: This block iterates over collections or graph structures. Key symbols: `emplace_back`, `empty`, `back`, `pop_back`.
- CN: 该代码块遍历集合或图结构。关键符号：`emplace_back`, `empty`, `back`, `pop_back`。

### Lines 69-73
```cpp
    // when we lambda lift forks, first-class modules may be passed across
    // forks. This code recursively lowers the module in the fork call.
    if (e.n->kind() == prim::fork) {
      auto subgraph = e.n->g(attr::Subgraph);
      std::vector<Slot> new_slots;
```
- EN: Declares core types or data containers for this file. Prominent symbols: `kind`, `g`.
- CN: 声明该文件使用的核心类型或数据容器。主要符号：`kind`, `g`。

### Lines 74-82
```cpp
      std::tie(subgraph, new_slots) = lower_graph(e.mod, *subgraph, e.offset);
      e.n->g_(attr::Subgraph, subgraph);
      for (const Slot& slot : new_slots) {
        e.n->addInput(getOrAddSlot(slot));
      }
      e.n->removeInput(e.offset);
      continue;
    }
    if (e.n->kind() == prim::PythonOp) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `tie`, `lower_graph`, `g_`, `addInput`, `getOrAddSlot`, `removeInput`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`tie`, `lower_graph`, `g_`, `addInput`, `getOrAddSlot`, `removeInput`, `...`。

### Lines 83-92
```cpp
      throw ErrorReport(e.n->sourceRange()) << "Couldn't export Python method.";
    }
    if (e.n->kind() != prim::GetAttr) {
      throw ErrorReport(e.n->sourceRange())
          << "temporary: the only valid use of a module is looking up an "
             "attribute but found "
          << *e.n;
    }
    size_t slot_idx = e.mod->type()->getAttributeSlot(e.n->s(attr::name));
    auto iv = e.mod->getSlot(slot_idx);
```
- EN: This block handles conditional branches. Key symbols: `ErrorReport`, `sourceRange`, `kind`, `type`, `getAttributeSlot`, `s`, `...`.
- CN: 该代码块处理条件分支。关键符号：`ErrorReport`, `sourceRange`, `kind`, `type`, `getAttributeSlot`, `s`, `...`。

### Lines 93-102
```cpp
    if (ClassTypePtr c = e.n->output()->type()->cast<ClassType>()) {
      if (c->is_module()) {
        for (Use use : e.n->output()->uses()) {
          to_scan.emplace_back(ToScan{iv.toObject(), use.user, use.offset});
        }
        to_clean.emplace_back(e.n);
        continue;
      }
    }
    e.n->output()->replaceAllUsesWith(getOrAddSlot({e.mod, slot_idx}));
```
- EN: This block handles conditional branches; iterates over collections or graph structures. Key symbols: `output`, `type`, `is_module`, `uses`, `emplace_back`, `toObject`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构。关键符号：`output`, `type`, `is_module`, `uses`, `emplace_back`, `toObject`, `...`。

### Lines 103-107
```cpp
    e.n->destroy();
  }

  while (!to_clean.empty()) {
    Node* n = to_clean.back();
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `destroy`, `empty`, `back`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`destroy`, `empty`, `back`。

### Lines 108-114
```cpp
    AT_ASSERT(!n->hasUses());
    n->destroy();
    to_clean.pop_back();
  }
  AT_ASSERT(!self_value->hasUses());
  g->eraseInput(self_offset);

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `hasUses`, `destroy`, `pop_back`, `eraseInput`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`hasUses`, `destroy`, `pop_back`, `eraseInput`。

### Lines 115-119
```cpp
  return std::make_pair(std::move(g), std::move(extra_ivalues));
}

static std::vector<IValue> loadTensors(const std::vector<Slot>& slots) {
  std::vector<IValue> result;
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `make_pair`, `move`, `loadTensors`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`make_pair`, `move`, `loadTensors`。

### Lines 120-127
```cpp
  result.reserve(slots.size());
  for (const Slot& slot : slots) {
    auto obj = slot.obj->getSlot(slot.offset);
    if (obj.isTensor()) {
      result.emplace_back(obj.toTensor());
    } else {
      // Unpack quantization packed tensor
      auto type = obj.type();
```
- EN: This block handles conditional branches; iterates over collections or graph structures. Key symbols: `reserve`, `size`, `getSlot`, `isTensor`, `emplace_back`, `toTensor`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构。关键符号：`reserve`, `size`, `getSlot`, `isTensor`, `emplace_back`, `toTensor`, `...`。

### Lines 128-137
```cpp
      TORCH_CHECK(
          (type ==
           getCustomClass(
               "__torch__.torch.classes.quantized.Conv2dPackedParamsBase")) ||
              (type ==
               getCustomClass(
                   "__torch__.torch.classes.quantized.Conv3dPackedParamsBase")) ||
              (type ==
               getCustomClass(
                   "__torch__.torch.classes.quantized.LinearPackedParamsBase")),
```
- EN: This block implements local helper logic for lower graph. Key symbols: `getCustomClass`.
- CN: 该代码块实现与 lower graph 相关的局部辅助逻辑。关键符号：`getCustomClass`。

### Lines 138-145
```cpp
          "Unknown type ",
          type->repr_str(),
          " encountered in graph lowering. This type is not supported in ONNX export.");
      result.emplace_back(
          script::Object(obj.toObject()).run_method("__getstate__"));
    }
  }
  return result;
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `repr_str`, `emplace_back`, `Object`, `toObject`, `run_method`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`repr_str`, `emplace_back`, `Object`, `toObject`, `run_method`。

### Lines 146-151
```cpp
}

std::pair<std::shared_ptr<Graph>, std::vector<IValue>> LowerGraph(
    Graph& graph,
    const ModulePtr& self) {
  auto result = lower_graph(self, graph);
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `LowerGraph`, `lower_graph`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`LowerGraph`, `lower_graph`。

### Lines 152-155
```cpp
  return std::make_pair(result.first, loadTensors(result.second));
}

} // namespace torch::jit
```
- EN: Opens or closes namespace scopes so the remaining declarations live in the expected JIT/frontend module.
- CN: 打开或关闭命名空间作用域，使后续声明归属于预期的 JIT/前端模块。


## Key Concepts / 关键概念
- EN: IR graph manipulation — the file inspects or rewrites TorchScript graphs and nodes.
  CN: IR 图操作——该文件会检查或改写 TorchScript 的图、节点与块。
- EN: Inlining — the implementation merges callees or blocks into their call sites.
  CN: 内联——实现会把被调用者或代码块并入调用位置。
- EN: Fusion support — the file prepares groups of operations for fused execution.
  CN: 融合支持——该文件会为融合执行准备算子分组。
- EN: Type and shape reasoning — the code propagates static information across the IR.
  CN: 类型与形状推理——代码会在 IR 中传播静态信息。
- EN: Export interop — the implementation helps bridge TorchScript graphs to ONNX-style representations.
  CN: 导出互操作——实现帮助 TorchScript 图与 ONNX 风格表示之间建立桥接。
- EN: Pass orchestration — the file plugs logic into the TorchScript optimization pipeline.
  CN: Pass 编排——该文件把相关逻辑接入 TorchScript 优化流水线。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `torch/csrc/jit/passes/lower_graph.h`, `torch/csrc/jit/api/object.h`, `torch/csrc/jit/frontend/error_report.h`, `torch/csrc/jit/passes/inliner.h`, `torch/custom_class.h`
- External includes / 外部头文件: `unordered_map`
- Namespaces / 命名空间: `torch::jit`
- Representative symbols / 代表性符号: `Slot`, `lower_graph`, `copy`, `Inline`, `SlotHash`, `get`, `hash_combine`, `ToScan`, `find`, `end`, `...`
