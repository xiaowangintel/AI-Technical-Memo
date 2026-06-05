# Graph.h — Code Analysis / 代码分析

## Source / 来源
- File: `torch/nativert/graph/Graph.h`
- Repository: `pytorch/pytorch`
- Purpose (EN): Declares Native Runtime runtime interfaces, helper types, and contracts for Graph.
- 用途 (CN): 声明 Native Runtime 中 runtime 子模块里与 Graph 相关的接口、辅助类型和约定。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9
```cpp
#pragma once

#include <memory>
#include <string>
#include <unordered_map>
#include <utility>
#include <variant>
#include <vector>

```
- EN: Pulls in compile-time configuration or headers required here. Internal includes: none; external includes: `memory`, `string`, `unordered_map`, `utility`, `variant`, `vector`.
- CN: 引入此处所需的编译期开关或头文件。内部依赖：无；外部依赖：`memory`, `string`, `unordered_map`, `utility`, `variant`, `vector`。

### Lines 10-18
```cpp
#include <ATen/core/ivalue.h>
#include <c10/util/IntrusiveList.h>
#include <c10/util/Logging.h>

#include <torch/csrc/utils/generated_serialization_types.h>
#include <torch/nativert/executor/Placement.h>
#include <torch/nativert/graph/GraphSignature.h>
#include <torch/nativert/graph/TensorMeta.h>

```
- EN: Pulls in compile-time configuration or headers required here. Internal includes: `ATen/core/ivalue.h`, `c10/util/IntrusiveList.h`, `c10/util/Logging.h`, `torch/csrc/utils/generated_serialization_types.h`, `torch/nativert/executor/Placement.h`, `torch/nativert/graph/GraphSignature.h`, `...`; external includes: none.
- CN: 引入此处所需的编译期开关或头文件。内部依赖：`ATen/core/ivalue.h`, `c10/util/IntrusiveList.h`, `c10/util/Logging.h`, `torch/csrc/utils/generated_serialization_types.h`, `torch/nativert/executor/Placement.h`, `torch/nativert/graph/GraphSignature.h`, `...`；外部依赖：无。

### Lines 19-27
```cpp
namespace torch::nativert {

using NodeIndex = size_t;

class Value;

class Type {
 public:
  enum class Kind {
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `NodeIndex`, `Value`, `Type`, `class`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`NodeIndex`, `Value`, `Type`, `class`。

### Lines 28-38
```cpp
    None,
    Tensor,
    TensorList,
    NestedTensorList,
    OptionalTensorList,
    SymInt,
    SymIntList,
    SymBool,
    SymFloat,
    CustomObj,
  };
```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 39-49
```cpp

  // For simple kinds without classFqn
  /*implicit*/ Type(Kind kind) : kind_(kind) {}

  // For CustomObj kind with classFqn
  explicit Type(Kind kind, const std::string& classFqn)
      : kind_(CustomObjData{classFqn}) {
    TORCH_CHECK(kind == Kind::CustomObj);
    TORCH_CHECK(!classFqn.empty());
  }

```
- EN: This block checks invariants or expected outcomes; iterates over collections or execution units. Key symbols: `Type`, `kind_`, `empty`.
- CN: 该代码块检查不变量或预期结果；遍历集合或执行单元。关键符号：`Type`, `kind_`, `empty`。

### Lines 50-59
```cpp
  Kind kind() const {
    if (std::holds_alternative<CustomObjData>(kind_)) {
      return Kind::CustomObj;
    }
    return std::get<Kind>(kind_);
  }

  friend std::ostream& operator<<(std::ostream& out, const Type& ty);
  friend bool operator==(const Type& left, const Type& right);

```
- EN: This block handles conditional control flow; returns results to callers or downstream stages. Key symbols: `kind`.
- CN: 该代码块处理条件控制流；向调用方或后续阶段返回结果。关键符号：`kind`。

### Lines 60-69
```cpp
  std::string classFqn() const {
    TORCH_CHECK(
        kind() == Kind::CustomObj, "Only CustomObj type can have classFqn");
    return std::get<CustomObjData>(kind_).classFqn;
  }

 private:
  struct CustomObjData {
    std::string classFqn;
  };
```
- EN: Declares C++ types or containers needed in this file. Representative symbols: `classFqn`, `kind`, `CustomObjData`.
- CN: 声明该文件需要的 C++ 类型或容器。代表性符号：`classFqn`, `kind`, `CustomObjData`。

### Lines 70-79
```cpp
  std::variant<Kind, CustomObjData> kind_;
};

// These are all the constant types that are allowed as attributes on Nodes.
struct None {};
// None always equals itself
inline bool operator==(const None& /*unused*/, const None& /*unused*/) {
  return true;
}

```
- EN: Declares C++ types or containers needed in this file. Representative symbols: `None`.
- CN: 声明该文件需要的 C++ 类型或容器。代表性符号：`None`。

### Lines 80-97
```cpp
class Graph;

/**
 * We distinguish between a symbolic value (Tensor, TensorList, SymInt, SymInts,
 * etc) and a constant value (int, bool, string, etc). Here Constant is the type
 * for all possible constant values. Along with a name, they are represented as
 * Attributes on a Node.
 */
using Constant = std::variant<
    None,
    int64_t,
    std::vector<int64_t>,
    double,
    std::vector<double>,
    std::string,
    c10::ScalarType,
    c10::MemoryFormat,
    c10::Layout,
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `Graph`, `value`, `Constant`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`Graph`, `value`, `Constant`。

### Lines 98-107
```cpp
    c10::Device,
    bool,
    std::vector<bool>,
    std::vector<std::string>,
    std::vector<std::vector<int64_t>>,
    std::vector<std::vector<double>>,
    std::unique_ptr<Graph>>;

c10::IValue constantToIValue(const Constant& constant);

```
- EN: This block handles tensor metadata or sample values; manipulates graph-like program structures. Key symbols: `constantToIValue`.
- CN: 该代码块处理张量元数据或示例值；操作图状程序结构。关键符号：`constantToIValue`。

### Lines 108-124
```cpp
class Node;

/**
 * Represents a single symbolic value (tensor/symint/list of them). Values are
 * inputs and outputs of Nodes.
 */
using ValueId = int;
class Value {
 public:
  explicit Value(ValueId id, std::string name, Type t, Node* producer)
      : name_(std::move(name)),
        id_(id),
        type_(std::move(t)),
        producer_(producer) {
    TORCH_INTERNAL_ASSERT_DEBUG_ONLY(name_ == this->name());
  }

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `Node`, `value`, `ValueId`, `Value`, `name_`, `move`, `...`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`Node`, `value`, `ValueId`, `Value`, `name_`, `move`, `...`。

### Lines 125-133
```cpp
  // Each Value should be uniquely created and managed by a Graph. It's not
  // allowed to copy/move Value instances.
  Value(Value&&) = delete;
  Value& operator=(Value&&) = delete;
  Value(const Value&) = delete;
  Value& operator=(Value&) = delete;

  Type type() const {
    return type_;
```
- EN: This block manipulates graph-like program structures; returns results to callers or downstream stages. Key symbols: `Value`, `type`.
- CN: 该代码块操作图状程序结构；向调用方或后续阶段返回结果。关键符号：`Value`, `type`。

### Lines 134-143
```cpp
  }

  ValueId id() const {
    return id_;
  }

  std::string_view name() const {
    return name_;
  }

```
- EN: This block returns results to callers or downstream stages. Key symbols: `id`, `name`.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：`id`, `name`。

### Lines 144-152
```cpp
  const Node* producer(bool resolve_folded = false) const {
    return (!resolve_folded && isFolded()) ? nullptr : producer_;
  }

  Node* producer() {
    return producer_;
  }

  void addUser(Node* node);
```
- EN: This block manipulates graph-like program structures; returns results to callers or downstream stages. Key symbols: `producer`, `isFolded`, `addUser`.
- CN: 该代码块操作图状程序结构；向调用方或后续阶段返回结果。关键符号：`producer`, `isFolded`, `addUser`。

### Lines 153-162
```cpp
  void eraseUser(Node* node);
  void eraseAllUsers() {
    users_.clear();
  }

  // Throws an exception if the value is not a TensorList
  std::vector<const Value*> getListElements() const;

  const auto& users() const {
    return users_;
```
- EN: This block handles tensor metadata or sample values; manipulates graph-like program structures; reports or normalizes error conditions; handles conditional control flow; returns results to callers or downstream stages. Key symbols: `eraseUser`, `eraseAllUsers`, `clear`, `getListElements`, `users`.
- CN: 该代码块处理张量元数据或示例值；操作图状程序结构；报告或规范化错误情况；处理条件控制流；向调用方或后续阶段返回结果。关键符号：`eraseUser`, `eraseAllUsers`, `clear`, `getListElements`, `users`。

### Lines 163-173
```cpp
  }

  auto& users() {
    return users_;
  }

  void setId(ValueId newId) {
    // This should only be used inside the renumberValues pass
    id_ = newId;
  }

```
- EN: This block returns results to callers or downstream stages. Key symbols: `users`, `setId`.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：`users`, `setId`。

### Lines 174-185
```cpp
  void setIsFolded() {
    isFolded_ = true;
  }

  bool isFolded() const {
    return isFolded_;
  }

 private:
  friend std::ostream& operator<<(std::ostream& out, const Value& v);
  std::string name_;
  bool isFolded_{false};
```
- EN: This block returns results to callers or downstream stages. Key symbols: `setIsFolded`, `isFolded`.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：`setIsFolded`, `isFolded`。

### Lines 186-194
```cpp
  ValueId id_;
  Type type_;
  Node* producer_;
  // All nodes which have this value as input.
  // Note that this is a vector to avoid nondeterminism in iteration, but
  // probably should be an unordered set given usage patterns. If this becomes a
  // perf problem we should revise.
  std::vector<Node*> users_;
};
```
- EN: This block manipulates graph-like program structures; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块操作图状程序结构；处理条件控制流。关键符号：无明显局部符号。

### Lines 195-204
```cpp

struct NamedArgument {
  std::string name;
  Value* value;
};

struct Attribute {
  std::string name;
  Constant value;
};
```
- EN: Declares C++ types or containers needed in this file. Representative symbols: `NamedArgument`, `Attribute`.
- CN: 声明该文件需要的 C++ 类型或容器。代表性符号：`NamedArgument`, `Attribute`。

### Lines 205-213
```cpp

/**
 * Node represents a single unit of execution, typically a PyTorch operator.
 * Using an intrusive list allows us to allocate all the memory at once for a
 * node. This also allows us to track nodes safely without passing around the
 * list object, as an intrusive list maintains a stronger invariant that
 * expiration will always cause unlinking.
 */
class Node : public c10::IntrusiveListHook {
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `Node`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`Node`。

### Lines 214-222
```cpp
 public:
  Node(
      Graph* owningGraph,
      std::string target,
      std::vector<NamedArgument> inputs,
      std::unordered_map<std::string, std::string> metadata);

  std::string_view target() const {
    return target_;
```
- EN: This block manipulates graph-like program structures; returns results to callers or downstream stages. Key symbols: `Node`, `target`.
- CN: 该代码块操作图状程序结构；向调用方或后续阶段返回结果。关键符号：`Node`, `target`。

### Lines 223-232
```cpp
  }

  void setTarget(std::string_view target) {
    target_ = target;
  }

  const auto& inputs() const {
    return inputs_;
  }

```
- EN: This block returns results to callers or downstream stages. Key symbols: `setTarget`, `inputs`.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：`setTarget`, `inputs`。

### Lines 233-242
```cpp
  auto& inputs() {
    return inputs_;
  }

  // NOTE: this invalidates spans given out by inputs()
  Value* addInput(NamedArgument input);
  void addInputs(const std::vector<NamedArgument>& inputs);

  // NOTE: this invalidates spans given out by attributes()
  void addAttribute(Attribute attr);
```
- EN: This block checks invariants or expected outcomes; returns results to callers or downstream stages. Key symbols: `inputs`, `addInput`, `addInputs`, `addAttribute`.
- CN: 该代码块检查不变量或预期结果；向调用方或后续阶段返回结果。关键符号：`inputs`, `addInput`, `addInputs`, `addAttribute`。

### Lines 243-251
```cpp

  // NOTE: this is ONLY for graph's constant inputs and NOT the common case
  void addOutput();

  Value* addOutput(const Type& type);

  // NOTE: this invalidates spans given out by outputs()
  Value* addOutput(std::string_view name, const Type& type);

```
- EN: This block checks invariants or expected outcomes; manipulates graph-like program structures; iterates over collections or execution units. Key symbols: `addOutput`.
- CN: 该代码块检查不变量或预期结果；操作图状程序结构；遍历集合或执行单元。关键符号：`addOutput`。

### Lines 252-264
```cpp
  size_t numInputs() const {
    return inputs_.size();
  }

  size_t numOutputs() const {
    return outputs_.size();
  }

  // Return the next node in the Graph's node ordering.
  // NOTE: Calling next on the last node (prim.Output) returns nullptr.
  Node* next();
  const Node* next() const;

```
- EN: This block manipulates graph-like program structures; returns results to callers or downstream stages. Key symbols: `numInputs`, `size`, `numOutputs`, `next`.
- CN: 该代码块操作图状程序结构；向调用方或后续阶段返回结果。关键符号：`numInputs`, `size`, `numOutputs`, `next`。

### Lines 265-274
```cpp
  // Return the previous node in the Graph's node ordering.
  // NOTE: Calling prev on the first node (prim.Input) returns nullptr.
  Node* prev();
  const Node* prev() const;

  bool isBefore(const Node* n) const;

  std::vector<Node*> producers() const;
  std::vector<Node*> users() const;

```
- EN: This block manipulates graph-like program structures; returns results to callers or downstream stages. Key symbols: `prev`, `isBefore`, `producers`, `users`.
- CN: 该代码块操作图状程序结构；向调用方或后续阶段返回结果。关键符号：`prev`, `isBefore`, `producers`, `users`。

### Lines 275-283
```cpp
  // Returns nullptr if `name` is not an input
  const NamedArgument* tryGetInput(std::string_view name) const;
  // Throws an exception if `name` is not an input
  const NamedArgument& getInput(std::string_view name) const;

  const auto& attributes() const {
    return attributes_;
  }

```
- EN: This block reports or normalizes error conditions; handles conditional control flow; returns results to callers or downstream stages. Key symbols: `tryGetInput`, `getInput`, `attributes`.
- CN: 该代码块报告或规范化错误情况；处理条件控制流；向调用方或后续阶段返回结果。关键符号：`tryGetInput`, `getInput`, `attributes`。

### Lines 284-292
```cpp
  // Returns nullptr if `name` is not an attribute
  const Attribute* tryGetAttribute(std::string_view name) const;
  // Throws an exception if `name` is not an attribute
  const Attribute& getAttribute(std::string_view name) const;

  const auto& outputs() const {
    return outputs_;
  }

```
- EN: This block reports or normalizes error conditions; handles conditional control flow; returns results to callers or downstream stages. Key symbols: `tryGetAttribute`, `getAttribute`, `outputs`.
- CN: 该代码块报告或规范化错误情况；处理条件控制流；向调用方或后续阶段返回结果。关键符号：`tryGetAttribute`, `getAttribute`, `outputs`。

### Lines 293-302
```cpp
  void applyDevicePlacement(const Placement& placement);

  std::optional<std::string_view> getMetadata(std::string_view key) const {
    return metadata_.find(std::string{key}) != metadata_.end()
        ? std::optional(std::string_view{metadata_.at(std::string{key})})
        : std::nullopt;
  }

  Graph* owningGraph() {
    return owningGraph_;
```
- EN: This block handles tensor metadata or sample values; manipulates graph-like program structures; returns results to callers or downstream stages. Key symbols: `applyDevicePlacement`, `getMetadata`, `find`, `end`, `optional`, `owningGraph`.
- CN: 该代码块处理张量元数据或示例值；操作图状程序结构；向调用方或后续阶段返回结果。关键符号：`applyDevicePlacement`, `getMetadata`, `find`, `end`, `optional`, `owningGraph`。

### Lines 303-312
```cpp
  }

  const Graph* owningGraph() const {
    return owningGraph_;
  }

  void destroy();

  const std::unordered_map<std::string, std::string>& metadata() const {
    return metadata_;
```
- EN: This block manipulates graph-like program structures; returns results to callers or downstream stages. Key symbols: `owningGraph`, `destroy`, `metadata`.
- CN: 该代码块操作图状程序结构；向调用方或后续阶段返回结果。关键符号：`owningGraph`, `destroy`, `metadata`。

### Lines 313-321
```cpp
  }

  std::string toString() const {
    std::stringstream ss;
    ss << *this;
    return ss.str();
  }

  void updateInputName(std::string_view oldName, std::string_view newName) {
```
- EN: This block returns results to callers or downstream stages. Key symbols: `toString`, `str`, `updateInputName`.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：`toString`, `str`, `updateInputName`。

### Lines 322-330
```cpp
    for (auto& input : inputs_) {
      if (input.name == oldName) {
        input.name = newName;
        break;
      }
    }
  }

  void updateAttributeName(std::string_view oldName, std::string_view newName) {
```
- EN: This block handles conditional control flow; iterates over collections or execution units. Key symbols: `updateAttributeName`.
- CN: 该代码块处理条件控制流；遍历集合或执行单元。关键符号：`updateAttributeName`。

### Lines 331-342
```cpp
    for (auto& attr : attributes_) {
      if (attr.name == oldName) {
        attr.name = newName;
        break;
      }
    }
  }

 private:
  friend std::ostream& operator<<(std::ostream& out, const Node& n);
  Graph* owningGraph_;

```
- EN: This block manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块操作图状程序结构；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 343-357
```cpp
  // Target used to retrieve the actual thing to execute.
  // If an aten operator, we expect this to be fully qualified, including an
  // overload name, e.g. "aten.unsqueeze.default"
  std::string target_;
  // *Symbolic* inputs to this node. NOTE: this does not match the ATen operator
  // schema inputs directly. It only represents things that actually participate
  // in dataflow, like tensors/symints and lists thereof.
  //
  // The "name" of the NamedArgument refers to the name of the parameter.
  std::vector<NamedArgument> inputs_;
  // Constant inputs to the node. The "name" of the Attribute refers to the
  // name of the parameter.
  std::vector<Attribute> attributes_;
  std::vector<Value*> outputs_;

```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; advances ONNX export translation; manipulates graph-like program structures; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；推进 ONNX 导出翻译流程；操作图状程序结构；处理条件控制流。关键符号：无明显局部符号。

### Lines 358-375
```cpp
  // Extra bits of info added to the node. Contents that are guaranteed will be
  // eventually moved to a first-class field on the json struct of schema.
  std::unordered_map<std::string, std::string> metadata_;
};

/**
 * Graph represents a model's computation graph, which is designed to
 * facilitate transformation and analysis.
 *
 * Ownership semantics:
 *  - Graph owns Nodes and Values
 *  - Nodes own their constant attributes (which we treat as value types)
 *  - Nodes have non-owning pointers back to the graph.
 *
 * NOTE: this class is marked noncopyable/nonmovable and only can be
 * heap-allocated via `createGraph()`. This is to ensure stability of
 * back-pointers held by Nodes/Values.
 */
```
- EN: Declares C++ types or containers needed in this file. Representative symbols: `attributes`, `is`, `createGraph`.
- CN: 声明该文件需要的 C++ 类型或容器。代表性符号：`attributes`, `is`, `createGraph`。

### Lines 376-387
```cpp
class Graph {
 public:
  static std::unique_ptr<Graph> createGraph() {
    return std::unique_ptr<Graph>(new Graph());
  }

  Graph(const Graph&) = delete;
  Graph& operator=(const Graph&) = delete;
  Graph(Graph&&) = delete;
  Graph& operator=(Graph&&) = delete;
  ~Graph() = default;

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `Graph`, `createGraph`, `~Graph`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`Graph`, `createGraph`, `~Graph`。

### Lines 388-396
```cpp
  // NOTE: this invalidates spans given out by inputs()
  Value* addInput(std::string_view name, const Type& type);

  // NOTE: this is ONLY for graph's constant inputs and NOT the common case
  void addInput();

  // NOTE: this invalidates spans given out by outputs()
  Value* addOutput(Value* v);

```
- EN: This block checks invariants or expected outcomes; manipulates graph-like program structures; iterates over collections or execution units. Key symbols: `addInput`, `addOutput`.
- CN: 该代码块检查不变量或预期结果；操作图状程序结构；遍历集合或执行单元。关键符号：`addInput`, `addOutput`。

### Lines 397-411
```cpp
  void addConstantOutput(Constant c);

  // Create and insert a node at insertionPoint_
  Node* insertNode(
      std::string target,
      std::vector<NamedArgument> inputs = {},
      std::unordered_map<std::string, std::string> metadata = {});

  // Returns the inserted node.
  Node* insertBefore(Node* toInsert, Node* insertionPoint);
  // Returns the inserted node.
  Node* insertAfter(Node* toInsert, Node* insertionPoint);
  // Insert at the insertionPoint. Returns the inserted node.
  Node* insert(Node* toInsert);

```
- EN: This block manipulates graph-like program structures. Key symbols: `addConstantOutput`, `insertNode`, `insertBefore`, `insertAfter`, `insert`.
- CN: 该代码块操作图状程序结构。关键符号：`addConstantOutput`, `insertNode`, `insertBefore`, `insertAfter`, `insert`。

### Lines 412-420
```cpp
  // Create a node without inserting it into the execution graph.
  // A raw pointer to the node is created when `createNode()` on the
  // owner Graph object is called. It is guranateed that to be valid
  // until the Graph object is destructed.
  Node* createNode(
      std::string target,
      std::vector<NamedArgument> inputs = {},
      std::unordered_map<std::string, std::string> metadata = {});

```
- EN: This block manipulates graph-like program structures. Key symbols: `createNode`.
- CN: 该代码块操作图状程序结构。关键符号：`createNode`。

### Lines 421-430
```cpp
  Value* createConstantSymIntValue(int value);

  Node* createListPack(std::vector<Value*> inputs, const Type& inputType);

  Node* createOptionalListPack(std::vector<Value*> inputs);

  size_t numValues() const {
    return values_.size();
  }

```
- EN: This block manipulates graph-like program structures; returns results to callers or downstream stages. Key symbols: `createConstantSymIntValue`, `createListPack`, `createOptionalListPack`, `numValues`, `size`.
- CN: 该代码块操作图状程序结构；向调用方或后续阶段返回结果。关键符号：`createConstantSymIntValue`, `createListPack`, `createOptionalListPack`, `numValues`, `size`。

### Lines 431-439
```cpp
  // throws on missing name
  Value* getValue(std::string_view name) const;
  // returns nullptr on missing name
  Value* tryGetValue(std::string_view name) const;

  const std::unordered_map<ValueId, int> getConstantSymIntValues() const {
    return constantSymIntValues_;
  }

```
- EN: This block returns results to callers or downstream stages. Key symbols: `getValue`, `tryGetValue`, `getConstantSymIntValues`.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：`getValue`, `tryGetValue`, `getConstantSymIntValues`。

### Lines 440-448
```cpp
  Value* addValue(
      const std::optional<std::string>& name,
      const Type& type,
      Node* producer);
  void removeValue(Value* value);

  void replaceAllUses(Value* old, Value* replacement);
  void replaceAllUsesAfterNode(Value* old, Value* replacement, Node* afterThis);
  void removeNode(Node* node);
```
- EN: This block manipulates graph-like program structures. Key symbols: `addValue`, `removeValue`, `replaceAllUses`, `replaceAllUsesAfterNode`, `removeNode`.
- CN: 该代码块操作图状程序结构。关键符号：`addValue`, `removeValue`, `replaceAllUses`, `replaceAllUsesAfterNode`, `removeNode`。

### Lines 449-458
```cpp

  void applyDevicePlacement(const Placement& placement);

  // Override all weights in the graph if matching name is found in the map.
  void overrideWeightsDevice(
      const std::unordered_map<std::string, std::optional<c10::Device>>&
          submodNameToDevice);

  std::string getUniqueValueName();

```
- EN: This block handles tensor metadata or sample values; manipulates graph-like program structures; handles conditional control flow. Key symbols: `applyDevicePlacement`, `overrideWeightsDevice`, `getUniqueValueName`.
- CN: 该代码块处理张量元数据或示例值；操作图状程序结构；处理条件控制流。关键符号：`applyDevicePlacement`, `overrideWeightsDevice`, `getUniqueValueName`。

### Lines 459-467
```cpp
  ValueId getNextValueId() {
    return uniqueValueId_++;
  }

  // NOTE: this range can be invalidated by mutations to the graph.
  const auto& inputs() const {
    return inputNode_->outputs();
  }

```
- EN: This block checks invariants or expected outcomes; manipulates graph-like program structures; returns results to callers or downstream stages. Key symbols: `getNextValueId`, `inputs`, `outputs`.
- CN: 该代码块检查不变量或预期结果；操作图状程序结构；向调用方或后续阶段返回结果。关键符号：`getNextValueId`, `inputs`, `outputs`。

### Lines 468-479
```cpp
  c10::ArrayRef<const Value*> userInputs() const {
    size_t offset = signature().inputsToWeights().size() +
        signature().inputsToCustomObjs().size();
    return {inputs().data() + offset, inputs().data() + inputs().size()};
  }

  c10::ArrayRef<const Value*> weightValues() const {
    return {
        inputs().data(),
        inputs().data() + signature().inputsToWeights().size()};
  }

```
- EN: This block returns results to callers or downstream stages. Key symbols: `userInputs`, `signature`, `inputsToWeights`, `size`, `inputsToCustomObjs`, `inputs`, `...`.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：`userInputs`, `signature`, `inputsToWeights`, `size`, `inputsToCustomObjs`, `inputs`, `...`。

### Lines 480-488
```cpp
  // Return a bidirectional range over `const Value*`
  // NOTE: this range can be invalidated by mutations to the graph.
  auto outputs() const {
    std::vector<const Value*> ret;
    ret.reserve(outputNode_->inputs().size());
    for (const auto& namedArg : outputNode_->inputs()) {
      ret.push_back(namedArg.value);
    }
    return ret;
```
- EN: This block checks invariants or expected outcomes; manipulates graph-like program structures; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: `outputs`, `reserve`, `inputs`, `size`, `push_back`.
- CN: 该代码块检查不变量或预期结果；操作图状程序结构；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：`outputs`, `reserve`, `inputs`, `size`, `push_back`。

### Lines 489-499
```cpp
  }

  // Return a bidirectional range over `Value*`
  // NOTE: this range can be invalidated by mutations to the graph.
  auto outputs() {
    std::vector<Value*> ret;
    ret.reserve(outputNode_->inputs().size());
    for (const auto& namedArg : outputNode_->inputs()) {
      ret.push_back(namedArg.value);
    }
    return ret;
```
- EN: This block checks invariants or expected outcomes; manipulates graph-like program structures; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: `outputs`, `reserve`, `inputs`, `size`, `push_back`.
- CN: 该代码块检查不变量或预期结果；操作图状程序结构；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：`outputs`, `reserve`, `inputs`, `size`, `push_back`。

### Lines 500-509
```cpp
  }

  const auto& userOutputs() const {
    return userOutputs_;
  }

  // Return a list over `const Node&`.
  // NOTE: this can be invalidated by mutations to the graph.
  const auto& nodes() const {
    return nodes_;
```
- EN: This block checks invariants or expected outcomes; manipulates graph-like program structures; returns results to callers or downstream stages. Key symbols: `userOutputs`, `nodes`.
- CN: 该代码块检查不变量或预期结果；操作图状程序结构；向调用方或后续阶段返回结果。关键符号：`userOutputs`, `nodes`。

### Lines 510-518
```cpp
  }

  auto& nodes() {
    return nodes_;
  }

  // Return a forward range over `const Value*`.
  // NOTE: this range can be invalidated by mutations to the graph.
  auto values() const {
```
- EN: This block checks invariants or expected outcomes; manipulates graph-like program structures; returns results to callers or downstream stages. Key symbols: `nodes`, `values`.
- CN: 该代码块检查不变量或预期结果；操作图状程序结构；向调用方或后续阶段返回结果。关键符号：`nodes`, `values`。

### Lines 519-528
```cpp
    std::vector<const Value*> ret;
    ret.reserve(values_.size());
    for (const auto& [_, value] : values_) {
      ret.push_back(value.get());
    }
    return ret;
  }

  Node* inputNode() {
    return inputNode_;
```
- EN: This block manipulates graph-like program structures; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: `reserve`, `size`, `push_back`, `get`, `inputNode`.
- CN: 该代码块操作图状程序结构；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：`reserve`, `size`, `push_back`, `get`, `inputNode`。

### Lines 529-538
```cpp
  }

  Node* outputNode() {
    return outputNode_;
  }

  const Node* outputNode() const {
    return outputNode_;
  }

```
- EN: This block manipulates graph-like program structures; returns results to callers or downstream stages. Key symbols: `outputNode`.
- CN: 该代码块操作图状程序结构；向调用方或后续阶段返回结果。关键符号：`outputNode`。

### Lines 539-551
```cpp
  // Assert various graph invariants
  void lint() const;

  bool /* removed > 0? */ cleanupDeadNodes();

  void finalize();

  Node* insertionPoint() {
    // This should never happen, since the last-most insertion point is the
    // prim.Outputs node, not end().
    TORCH_CHECK(insertBefore_ != nodes_.end());
    auto& node = *insertBefore_;
    return &node;
```
- EN: This block checks invariants or expected outcomes; manipulates graph-like program structures; returns results to callers or downstream stages. Key symbols: `lint`, `cleanupDeadNodes`, `finalize`, `insertionPoint`, `end`.
- CN: 该代码块检查不变量或预期结果；操作图状程序结构；向调用方或后续阶段返回结果。关键符号：`lint`, `cleanupDeadNodes`, `finalize`, `insertionPoint`, `end`。

### Lines 552-561
```cpp
  }

  void setInsertionPoint(Node* n) {
    TORCH_CHECK(n != inputNode_, "can't insert before prim.Input");
    insertBefore_ = nodes_.iterator_to(*n);
  }

  void setInsertionPointAfter(Node* n) {
    TORCH_CHECK(n != outputNode_, "can't insert after prim.Output");
    auto it = nodes_.iterator_to(*n);
```
- EN: This block checks invariants or expected outcomes; manipulates graph-like program structures. Key symbols: `setInsertionPoint`, `iterator_to`, `setInsertionPointAfter`.
- CN: 该代码块检查不变量或预期结果；操作图状程序结构。关键符号：`setInsertionPoint`, `iterator_to`, `setInsertionPointAfter`。

### Lines 562-570
```cpp
    ++it;
    insertBefore_ = it;
  }

  // Return the next node in the Graph's node ordering.
  // NOTE: Calling on the last node (prim.Output) returns nullptr.
  Node* nodeAfter(Node* n);
  const Node* nodeAfter(const Node* n) const;

```
- EN: This block manipulates graph-like program structures; returns results to callers or downstream stages. Key symbols: `nodeAfter`.
- CN: 该代码块操作图状程序结构；向调用方或后续阶段返回结果。关键符号：`nodeAfter`。

### Lines 571-587
```cpp
  // Return the previous node in the Graph's node ordering.
  // NOTE: Calling on the first node (prim.Input) returns nullptr.
  Node* nodeBefore(Node* n);
  const Node* nodeBefore(const Node* n) const;

  // Clone each node from subgraph (except prim.Input/prim.Output) into current
  // graph.
  // @param subgraph: the subgraph to be cloned
  // @param inputs: values from the target graph that will serve as the
  // subgraph's inputs
  // @param valueMap: a map from the cloned subgraph's values to the target
  // graph's values
  std::vector<Value*> insertGraph(
      const Graph& subgraph,
      std::vector<Value*> inputs,
      std::unordered_map<const Value*, Value*>& valueMap);

```
- EN: This block manipulates graph-like program structures; returns results to callers or downstream stages. Key symbols: `nodeBefore`, `insertGraph`.
- CN: 该代码块操作图状程序结构；向调用方或后续阶段返回结果。关键符号：`nodeBefore`, `insertGraph`。

### Lines 588-596
```cpp
  const GraphSignature& signature() const {
    return signature_;
  }

  void setSignature(GraphSignature signature) {
    signature_ = std::move(signature);
  }

  void setWeightsMeta(
```
- EN: This block manipulates graph-like program structures; returns results to callers or downstream stages. Key symbols: `signature`, `setSignature`, `move`, `setWeightsMeta`.
- CN: 该代码块操作图状程序结构；向调用方或后续阶段返回结果。关键符号：`signature`, `setSignature`, `move`, `setWeightsMeta`。

### Lines 597-605
```cpp
      const std::unordered_map<std::string, torch::_export::TensorMeta>&
          tensorsMeta) {
    TORCH_CHECK(!placementApplied_);

    for (auto [name, tensorMeta] : tensorsMeta) {
      weightsMeta_.emplace(name, TensorMeta{tensorMeta});
    }
  }

```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; advances ONNX export translation; iterates over collections or execution units. Key symbols: `emplace`.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；推进 ONNX 导出翻译流程；遍历集合或执行单元。关键符号：`emplace`。

### Lines 606-616
```cpp
  const std::unordered_map<std::string, TensorMeta>& weightsMeta() const {
    return weightsMeta_;
  }

  std::vector<TensorMeta> userInputsMeta() const {
    std::vector<TensorMeta> userInputsMeta;
    userInputsMeta.reserve(signature_.userInputs().size());
    for (auto inputName : signature_.userInputs()) {
      userInputsMeta.push_back(tensorValuesMeta_.at(inputName));
    }
    return userInputsMeta;
```
- EN: This block handles tensor metadata or sample values; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: `weightsMeta`, `userInputsMeta`, `reserve`, `userInputs`, `size`, `push_back`.
- CN: 该代码块处理张量元数据或示例值；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：`weightsMeta`, `userInputsMeta`, `reserve`, `userInputs`, `size`, `push_back`。

### Lines 617-628
```cpp
  }

  void setTensorValuesMeta(
      const std::unordered_map<std::string, torch::_export::TensorMeta>&
          tensorsMeta) {
    TORCH_CHECK(!placementApplied_);

    for (auto [name, tensorMeta] : tensorsMeta) {
      tensorValuesMeta_.emplace(name, TensorMeta{tensorMeta});
    }
  }

```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; advances ONNX export translation; iterates over collections or execution units. Key symbols: `setTensorValuesMeta`, `emplace`.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；推进 ONNX 导出翻译流程；遍历集合或执行单元。关键符号：`setTensorValuesMeta`, `emplace`。

### Lines 629-638
```cpp
  const std::unordered_map<std::string, TensorMeta>& tensorValuesMeta() const {
    return tensorValuesMeta_;
  }

  std::string toString() const {
    std::stringstream ss;
    ss << *this;
    return ss.str();
  }

```
- EN: This block handles tensor metadata or sample values; returns results to callers or downstream stages. Key symbols: `tensorValuesMeta`, `toString`, `str`.
- CN: 该代码块处理张量元数据或示例值；向调用方或后续阶段返回结果。关键符号：`tensorValuesMeta`, `toString`, `str`。

### Lines 639-648
```cpp
  /* Reassigns IDs to every Value in this Graph so that they are contiguous from
   * 0..(numValues()-1). Should be used after values are removed
   */
  void renumberValues();

 private:
  Graph();
  friend std::ostream& operator<<(std::ostream& out, const Graph& g);
  GraphSignature signature_;

```
- EN: This block manipulates graph-like program structures. Key symbols: `numValues`, `renumberValues`, `Graph`.
- CN: 该代码块操作图状程序结构。关键符号：`numValues`, `renumberValues`, `Graph`。

### Lines 649-665
```cpp
  bool placementApplied_ = false;

  // keys are parameters, buffers, tensor_constants' names
  std::unordered_map<std::string, TensorMeta> weightsMeta_;

  // keys are tensor_values' names
  std::unordered_map<std::string, TensorMeta> tensorValuesMeta_;

  // Node lifetime is managed by nodesOwner_, but the actual ordering is
  // maintained intrusively using nodes_.
  // This is to facilitate quick insertion before/after a given Node*.
  std::vector<std::unique_ptr<Node>> nodesOwner_;
  c10::IntrusiveList<Node> nodes_;
  // The current insertion point. New nodes are inserted before this node.
  // Defaults to prim.Output.
  c10::IntrusiveList<Node>::iterator insertBefore_;

```
- EN: This block handles tensor metadata or sample values; manipulates graph-like program structures. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；操作图状程序结构。关键符号：无明显局部符号。

### Lines 666-681
```cpp
  // Graphs always start with an input and output node.
  // "prim.input() -> Value[]" take no input, and produces some outputs. AKA
  // "source“ of a graph.
  Node* inputNode_; // target: prim.Input
  // "prim.output(Value[]) -> None", take some inputs, but produce no output.
  // AKA "sink" of a graph.
  Node* outputNode_; // target: prim.Output

  std::unordered_map<std::string, std::unique_ptr<Value>> values_;
  // constantSymIntValues_ is a subset of values_
  std::unordered_map<ValueId, int> constantSymIntValues_;
  // Output values of the graph, which is a subset of values_.
  std::vector<std::variant<Value*, Constant>> userOutputs_;
  // Output constant values of the graph
  std::vector<Constant> constantOutputs_;

```
- EN: This block manipulates graph-like program structures. Key symbols: no dominant local symbols.
- CN: 该代码块操作图状程序结构。关键符号：无明显局部符号。

### Lines 682-696
```cpp
  size_t uniqueValueName_ = 0;

  ValueId uniqueValueId_ = 0;
};

/**
 * Scoped utility class for setting temporary insertion points.
 *
 * Use like:
 *   {
 *       InsertingAfter guard(node)
 *       graph.insertNode(...)  // this will be inserted after `node`.
 *   }
 */
class InsertingAfter {
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `for`, `guard`, `insertNode`, `InsertingAfter`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`for`, `guard`, `insertNode`, `InsertingAfter`。

### Lines 697-705
```cpp
 public:
  explicit InsertingAfter(Node* n)
      : insertAfter_(n), prev_(n->owningGraph()->insertionPoint()) {
    insertAfter_->owningGraph()->setInsertionPointAfter(insertAfter_);
  }
  ~InsertingAfter() {
    insertAfter_->owningGraph()->setInsertionPoint(prev_);
  }

```
- EN: This block manipulates graph-like program structures. Key symbols: `InsertingAfter`, `insertAfter_`, `prev_`, `owningGraph`, `insertionPoint`, `setInsertionPointAfter`, `...`.
- CN: 该代码块操作图状程序结构。关键符号：`InsertingAfter`, `insertAfter_`, `prev_`, `owningGraph`, `insertionPoint`, `setInsertionPointAfter`, `...`。

### Lines 706-714
```cpp
 private:
  Node* insertAfter_;
  Node* prev_;
};

inline constexpr std::string_view kMemoryFormatPrefix = "MemoryFormat::";
inline constexpr std::string_view kLayoutPrefix = "Layout::";
inline constexpr std::string_view kDevicePrefix = "Device";
inline constexpr std::string_view kScalarTypePrefix = "ScalarType::";
```
- EN: This block handles tensor metadata or sample values; manipulates graph-like program structures; reuses computed state to reduce repeated work. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；操作图状程序结构；复用已计算状态以减少重复工作。关键符号：无明显局部符号。

### Lines 715-725
```cpp

/**
 * Debug format serialization. The format here is intended to be human readable
 * and easy to work with, and is intended for debugging and testing only.
 * If you want stable serialization, use the json conversion utils.
 *
 * NOTE: node metadata currently not serialized
 */
std::string graphToString(const Graph& g, bool include_signature = false);
std::unique_ptr<Graph> stringToGraph(std::string_view source);

```
- EN: This block manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units. Key symbols: `graphToString`, `stringToGraph`.
- CN: 该代码块操作图状程序结构；处理条件控制流；遍历集合或执行单元。关键符号：`graphToString`, `stringToGraph`。

### Lines 726-734
```cpp
// Standalone functions to parse common constructs
// Parse something that looks like `Device{cuda:1}` to a device in json format.
c10::Device convertDevice(std::string_view symbol);
// We have separate functions for parsing atomic and list constants because
// there are restrictive rules about which constants can go in lists (i.e.
// it's not recursive).
Constant convertAtomicConstant(std::string_view symbol);
Constant convertListConstant(std::string_view symbol);

```
- EN: This block handles tensor metadata or sample values; iterates over collections or execution units; protects shared state or ordering guarantees. Key symbols: `convertDevice`, `convertAtomicConstant`, `convertListConstant`.
- CN: 该代码块处理张量元数据或示例值；遍历集合或执行单元；保护共享状态或执行顺序保证。关键符号：`convertDevice`, `convertAtomicConstant`, `convertListConstant`。

### Lines 735-735
```cpp
} // namespace torch::nativert
```
- EN: This block implements local helper logic for Graph. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 Graph 相关的局部辅助逻辑。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- EN: Testing assertions — the file focuses on checking values, errors, or textual expectations.
  CN: 测试断言——该文件重点检查数值、异常或文本期望是否满足要求。
- EN: Tensor construction and metadata — the code manages sample tensors, dtypes, layouts, or device placement.
  CN: 张量构造与元数据——代码会管理示例张量、数据类型、布局或设备放置。
- EN: Reusable test infrastructure — the module provides shared helpers, fixtures, or data for many test suites.
  CN: 可复用测试基础设施——该模块为多个测试套件提供共享辅助函数、夹具或数据。
- EN: Backend coverage — the file contains backend-specific behavior or validation paths.
  CN: 后端覆盖——该文件包含特定后端的行为或校验路径。
- EN: ONNX export pipeline — the code translates PyTorch programs into ONNX-friendly representations.
  CN: ONNX 导出流水线——代码把 PyTorch 程序转换为适合 ONNX 的表示。
- EN: Graph capture and transformation — the implementation works with program graphs and rewrite passes.
  CN: 图捕获与变换——实现会处理程序图及其改写 pass。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `ATen/core/ivalue.h`, `c10/util/IntrusiveList.h`, `c10/util/Logging.h`, `torch/csrc/utils/generated_serialization_types.h`, `torch/nativert/executor/Placement.h`, `torch/nativert/graph/GraphSignature.h`, `torch/nativert/graph/TensorMeta.h`
- External includes / 外部头文件: `memory`, `string`, `unordered_map`, `utility`, `variant`, `vector`
- Namespaces / 命名空间: `torch::nativert`
- Representative symbols / 代表性符号: `NodeIndex`, `Value`, `Type`, `class`, `kind_`, `empty`, `kind`, `classFqn`, `CustomObjData`, `None`, `...`
