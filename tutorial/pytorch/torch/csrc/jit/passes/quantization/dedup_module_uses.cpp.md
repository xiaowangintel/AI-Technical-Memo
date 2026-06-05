# dedup_module_uses.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/passes/quantization/dedup_module_uses.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements the TorchScript JIT pass logic for dedup module uses, including graph analysis and rewrites.
- 用途 (CN): 实现与 dedup module uses 相关的 TorchScript JIT pass 逻辑，包括图分析与改写。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5
```cpp
#include <torch/csrc/jit/passes/quantization/dedup_module_uses.h>

#include <torch/csrc/jit/jit_log.h>
#include <torch/csrc/jit/passes/quantization/helper.h>

```
- EN: Pulls in the headers needed by the dedup module uses logic. Internal dependencies: `torch/csrc/jit/passes/quantization/dedup_module_uses.h`, `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/quantization/helper.h`; external dependencies: none.
- CN: 为 dedup module uses 相关逻辑引入所需头文件。内部依赖：`torch/csrc/jit/passes/quantization/dedup_module_uses.h`, `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/quantization/helper.h`；外部依赖：无。

### Lines 6-10
```cpp
#include <stack>

namespace torch::jit {
namespace {
class ModuleUseDeduper {
```
- EN: Opens or closes namespace scopes so the remaining declarations live in the expected JIT/frontend module.
- CN: 打开或关闭命名空间作用域，使后续声明归属于预期的 JIT/前端模块。

### Lines 11-20
```cpp
 public:
  ModuleUseDeduper(Module& module) : module_(module) {}
  void dedup() {
    for (auto& method : module_.get_methods()) {
      const auto& graph = method.graph();
      findModuleUses(graph.get());
    }
    dedupModuleUses();
  }

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `ModuleUseDeduper`, `module_`, `dedup`, `get_methods`, `graph`, `findModuleUses`, `...`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`ModuleUseDeduper`, `module_`, `dedup`, `get_methods`, `graph`, `findModuleUses`, `...`。

### Lines 21-26
```cpp
 private:
  // Analyze the code to record information represents
  // uses of the module, which we'll use later to actually perform the dedup
  // operation Please see the comments of member variables of the class for more
  // information
  void findModuleUses(Graph* graph) {
```
- EN: Declares core types or data containers for this file. Prominent symbols: `findModuleUses`.
- CN: 声明该文件使用的核心类型或数据容器。主要符号：`findModuleUses`。

### Lines 27-31
```cpp
    GRAPH_DUMP("Finding module uses for ", graph);

    std::stack<Block*> blocks_to_visit;
    blocks_to_visit.push(graph->block());
    Value* self = graph->inputs()[0];
```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `push`, `block`, `inputs`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`push`, `block`, `inputs`。

### Lines 32-36
```cpp
    while (!blocks_to_visit.empty()) {
      Block* b = blocks_to_visit.top();
      blocks_to_visit.pop();
      for (Node* n : b->nodes()) {
        for (Block* subblock : n->blocks()) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `empty`, `top`, `pop`, `nodes`, `blocks`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`empty`, `top`, `pop`, `nodes`, `blocks`。

### Lines 37-42
```cpp
          blocks_to_visit.push(subblock);
        }
        if (n->kind() != prim::CallMethod) {
          continue;
        }
        Value* instance = n->inputs()[0];
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `push`, `kind`, `inputs`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`push`, `kind`, `inputs`。

### Lines 43-47
```cpp
        // boundary_val is the value we get when we trace back
        // the GetAttr access chain until we hit the input of graph
        // or a node that is not prim::GetAttr
        auto path = getModuleAccessPath(instance, self);

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `getModuleAccessPath`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`getModuleAccessPath`。

### Lines 48-54
```cpp
        // path.size() == 0 means we're calling a method
        // on self, we don't need to dedup uses of self
        if (path.empty()) {
          continue;
        }
        value_to_path_map_[instance] = path;
        auto m = findChildModule(module_, path);
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `empty`, `findChildModule`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`empty`, `findChildModule`。

### Lines 55-64
```cpp
        // If we fail to insert the module to the unique_modules_ set,
        // which means there are uses of this module before this point,
        // we'll have to rewrite the use
        if (!unique_modules_.insert(m._ivalue()).second) {
          uses_to_rewrite_.push_back(instance);
          GRAPH_DEBUG("Found use to rewrite: ", instance->debugName());
        }
      }
    }
  }
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `insert`, `_ivalue`, `push_back`, `debugName`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`insert`, `_ivalue`, `push_back`, `debugName`。

### Lines 65-74
```cpp

  // Deduplicate module uses given the information we recorded before
  void dedupModuleUses() {
    for (Value* v : uses_to_rewrite_) {
      const auto& path = value_to_path_map_.at(v);
      const auto& m = findChildModule(module_, path);
      // add a clone of the child module to the parent of the duplicated module
      const auto& child_name = addChildModule(module_, m, path);
      TORCH_INTERNAL_ASSERT(v->node()->kind() == prim::GetAttr);
      // change the name in GetAttr call
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `dedupModuleUses`, `findChildModule`, `addChildModule`, `node`, `kind`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`dedupModuleUses`, `findChildModule`, `addChildModule`, `node`, `kind`。

### Lines 75-84
```cpp
      auto original_name = v->node()->s(attr::name);
      v->node()->s_(attr::name, child_name);
      GRAPH_UPDATE(
          "Module use dedup: changing use of original module ",
          original_name,
          " to ",
          child_name);
    }
  }

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `node`, `s`, `s_`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`node`, `s`, `s_`。

### Lines 85-93
```cpp
  std::string addChildModule(
      Module& module,
      const Module& child_module,
      const std::vector<std::string>& path) {
    TORCH_INTERNAL_ASSERT(
        !path.empty(), "path must have at least one element.");
    // Parent module of the leaf child module corresponding to
    // the path
    auto parent_of_leaf = findChildModule(
```
- EN: This block implements local helper logic for dedup module uses. Key symbols: `addChildModule`, `empty`, `findChildModule`.
- CN: 该代码块实现与 dedup module uses 相关的局部辅助逻辑。关键符号：`addChildModule`, `empty`, `findChildModule`。

### Lines 94-99
```cpp
        module, std::vector<std::string>(path.begin(), path.end() - 1));

    // Original name of the child module
    const std::string& original_name = path[path.size() - 1];
    int uid = 0;
    std::string child_name = original_name + "_" + std::to_string(uid++);
```
- EN: This block implements local helper logic for dedup module uses. Key symbols: `begin`, `end`, `size`, `to_string`.
- CN: 该代码块实现与 dedup module uses 相关的局部辅助逻辑。关键符号：`begin`, `end`, `size`, `to_string`。

### Lines 100-104
```cpp
    while (parent_of_leaf.hasattr(child_name)) {
      child_name = original_name + "_" + std::to_string(uid++);
    }
    parent_of_leaf.register_module(child_name, child_module.deepcopy());
    return child_name;
```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value. Key symbols: `hasattr`, `to_string`, `register_module`, `deepcopy`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值。关键符号：`hasattr`, `to_string`, `register_module`, `deepcopy`。

### Lines 105-111
```cpp
  }

  Module module_;
  // Map from value of module instance to the list of names of submodules
  // starting from the top level module, e.g. ["sub1", "sub2", "relu"]
  // Also this is a cache of calling `getModuleAccessPath` of the value
  std::unordered_map<Value*, std::vector<std::string>> value_to_path_map_;
```
- EN: This block manipulates TorchScript IR objects. Key symbols: no dominant local symbol names.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：无明显局部符号。

### Lines 112-117
```cpp
  // Set of unique modules that are used in the graphs
  std::unordered_set<ModulePtr> unique_modules_;
  // Values that represent the module instance(the use of the module)
  // that we'll need to rewrite as a use of a cloned module
  // instance
  std::vector<Value*> uses_to_rewrite_;
```
- EN: This block manipulates TorchScript IR objects. Key symbols: no dominant local symbol names.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：无明显局部符号。

### Lines 118-122
```cpp
};

} // namespace

void DedupModuleUses(Module& module) {
```
- EN: This block implements local helper logic for dedup module uses. Key symbols: `DedupModuleUses`.
- CN: 该代码块实现与 dedup module uses 相关的局部辅助逻辑。关键符号：`DedupModuleUses`。

### Lines 123-127
```cpp
  ModuleUseDeduper d(module);
  d.dedup();
}

} // namespace torch::jit
```
- EN: Opens or closes namespace scopes so the remaining declarations live in the expected JIT/frontend module.
- CN: 打开或关闭命名空间作用域，使后续声明归属于预期的 JIT/前端模块。


## Key Concepts / 关键概念
- EN: IR graph manipulation — the file inspects or rewrites TorchScript graphs and nodes.
  CN: IR 图操作——该文件会检查或改写 TorchScript 的图、节点与块。
- EN: Fusion support — the file prepares groups of operations for fused execution.
  CN: 融合支持——该文件会为融合执行准备算子分组。
- EN: Pass orchestration — the file plugs logic into the TorchScript optimization pipeline.
  CN: Pass 编排——该文件把相关逻辑接入 TorchScript 优化流水线。
- EN: Subsystem integration — the implementation collaborates with nearby JIT/frontend headers and runtime helpers.
  CN: 子系统集成——实现会与附近的 JIT/前端头文件以及运行时辅助模块协同工作。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `torch/csrc/jit/passes/quantization/dedup_module_uses.h`, `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/quantization/helper.h`
- External includes / 外部头文件: `stack`
- Namespaces / 命名空间: `torch::jit`
- Representative symbols / 代表性符号: `ModuleUseDeduper`, `module_`, `dedup`, `get_methods`, `graph`, `findModuleUses`, `get`, `dedupModuleUses`, `push`, `block`, `...`
