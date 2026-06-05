# PartitionSchedulingUtility.h — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./include/triton/Dialect/TritonGPU/Transforms/PartitionSchedulingUtility.h`
- **EN:** Provides helper utilities that support transformations or analysis in this area.
- **CN:** 提供支撑该领域分析或变换的辅助工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```cpp
   1: #ifndef TRITON_TRITONGPU_TRANSFORMS_PARTITION_SCHEDULING_UTILITY_H_
   2: #define TRITON_TRITONGPU_TRANSFORMS_PARTITION_SCHEDULING_UTILITY_H_
```
**EN:** This block establishes the header guard so the declarations in this file are only processed once per translation unit.
**CN:** 该代码块建立头文件保护宏，避免同一翻译单元重复包含本文件时出现重复声明。

### Lines 4-7
```cpp
   4: #include "mlir/Support/LLVM.h"
   5: #include "triton/Dialect/Triton/IR/Dialect.h"
   6: #include "triton/Dialect/TritonGPU/IR/Dialect.h"
   7: #include "triton/Dialect/TritonNvidiaGPU/IR/Dialect.h"
```
**EN:** This block imports the direct dependencies needed here, including mlir/Support/LLVM.h, triton/Dialect/Triton/IR/Dialect.h, triton/Dialect/TritonGPU/IR/Dialect.h, and triton/Dialect/TritonNvidiaGPU/IR/Dialect.h.
**CN:** 该代码块引入后续声明直接依赖的头文件，例如 mlir/Support/LLVM.h, triton/Dialect/Triton/IR/Dialect.h, triton/Dialect/TritonGPU/IR/Dialect.h, and triton/Dialect/TritonNvidiaGPU/IR/Dialect.h。

### Lines 9-9
```cpp
   9: namespace mlir::triton::gpu::partition_scheduling_detail {
```
**EN:** This block opens or closes the namespace scope used to organize APIs under mlir::triton::gpu::partition_scheduling_detail.
**CN:** 该代码块打开或关闭命名空间作用域，用来把相关 API 组织在 mlir::triton::gpu::partition_scheduling_detail 下。

### Lines 11-13
```cpp
  11: namespace tt = triton;
  12: namespace ttg = triton::gpu;
  13: namespace ttng = triton::nvidia_gpu;
```
**EN:** This block opens or closes the namespace scope used to organize APIs under tt = triton;, ttg = triton::gpu;, and ttng = triton::nvidia_gpu;.
**CN:** 该代码块打开或关闭命名空间作用域，用来把相关 API 组织在 tt = triton;, ttg = triton::gpu;, and ttng = triton::nvidia_gpu; 下。

### Lines 15-16
```cpp
  15: class Graph;
  16: class Node;
```
**EN:** This block introduces `Graph`, the main class/struct defined here.
**CN:** 该代码块引入了此文件的核心类/结构体 `Graph`。

### Lines 18-27
```cpp
  18: enum Flags : uint8_t {
  19:   NONE = 0,
  20:   MANUAL = 1 << 0,
  21:   LOAD = 1 << 1,
  22:   STORE = 1 << 2,
  23:   MMA = 1 << 3,
  24:   TMEM = 1 << 4,
  25:   SFU = 1 << 5,
  26:   VIEW = 1 << 6,
  27: };
```
**EN:** This block defines an enumeration covering values such as Flags, uint8_t, NONE, MANUAL, LOAD, and STORE.
**CN:** 该代码块定义了一个枚举类型，覆盖了 Flags, uint8_t, NONE, MANUAL, LOAD, and STORE 等取值。

### Lines 29-31
```cpp
  29: inline Flags &operator|=(Flags &lhs, Flags rhs) {
  30:   return lhs = static_cast<Flags>(lhs | rhs);
  31: }
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 33-33
```cpp
  33: llvm::raw_ostream &operator<<(llvm::raw_ostream &stream, Flags flags);
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 35-35
```cpp
  35: Flags getNodeFlags(Node *node);
```
**EN:** This block declares or defines callable APIs such as getNodeFlags, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getNodeFlags 等可调用 API，用来封装这里提供的核心行为。

### Lines 37-37
```cpp
  37: size_t computeCost(Operation *op);
```
**EN:** This block declares or defines callable APIs such as computeCost, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 computeCost 等可调用 API，用来封装这里提供的核心行为。

### Lines 39-42
```cpp
  39: inline bool isViewOp(Operation *op) {
  40:   return isa<tt::BroadcastOp, tt::ExpandDimsOp, ttg::ConvertLayoutOp>(op) ||
  41:          op->hasTrait<OpTrait::MemDescViewTrait>();
  42: }
```
**EN:** This block declares or defines callable APIs such as isViewOp, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isViewOp 等可调用 API，用来封装这里提供的核心行为。

### Lines 44-52
```cpp
  44: class Partition {
  45: public:
  46:   explicit Partition(Graph *graph) : graph(graph) {}
  47:   void add(Node *node);
  48:   void remove(Node *node) { nodes.remove(node); }
  49:   void addFlag(Flags flag) { flags |= flag; }
  50:   Flags getFlags() const { return flags; }
  51:   const SetVector<Node *> &getNodes() const { return nodes; }
  52:   bool empty() const { return nodes.empty(); }
```
**EN:** This block introduces `Partition`, the main class/struct defined here. Within the declaration, methods such as graph, add, remove, addFlag, getFlags, and getNodes expose its core API.
**CN:** 该代码块引入了此文件的核心类/结构体 `Partition`。 其中 graph, add, remove, addFlag, getFlags, and getNodes 等方法构成了它的主要接口。

### Lines 54-59
```cpp
  54:   size_t getStage() const {
  55:     if (flags & Flags::MMA)
  56:       return 1;
  57:     return 0;
  58:   }
  59:   size_t getCost() const { return cost; }
```
**EN:** This block declares or defines callable APIs such as getStage and getCost, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getStage and getCost 等可调用 API，用来封装这里提供的核心行为。

### Lines 61-61
```cpp
  61:   static void merge(Partition *lhs, Partition *rhs);
```
**EN:** This block declares or defines callable APIs such as merge, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 merge 等可调用 API，用来封装这里提供的核心行为。

### Lines 63-63
```cpp
  63:   void dump() const;
```
**EN:** This block declares or defines callable APIs such as dump, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 dump 等可调用 API，用来封装这里提供的核心行为。

### Lines 65-65
```cpp
  65:   std::optional<size_t> id;
```
**EN:** This block stores supporting state such as id, which other APIs in the file consume.
**CN:** 该代码块声明了 id 等支撑状态，供本文件中的其他 API 使用。

### Lines 67-72
```cpp
  67: private:
  68:   Graph *graph;
  69:   Flags flags = Flags::NONE;
  70:   size_t cost = 0;
  71:   SetVector<Node *> nodes;
  72: };
```
**EN:** This block stores supporting state such as graph, NONE, and nodes, which other APIs in the file consume.
**CN:** 该代码块声明了 graph, NONE, and nodes 等支撑状态，供本文件中的其他 API 使用。

### Lines 74-79
```cpp
  74: class Port {
  75: public:
  76:   Port() = default;
  77:   Port(Node *node, size_t idx) : node(node), idx(idx) {}
  78:   Node *getNode() const { return node; }
  79:   size_t getIdx() const { return idx; }
```
**EN:** This block introduces `Port`, the main class/struct defined here. Within the declaration, methods such as node, idx, getNode, and getIdx expose its core API.
**CN:** 该代码块引入了此文件的核心类/结构体 `Port`。 其中 node, idx, getNode, and getIdx 等方法构成了它的主要接口。

### Lines 81-83
```cpp
  81:   bool operator==(const Port &other) const {
  82:     return node == other.node && idx == other.idx;
  83:   }
```
**EN:** This block stores supporting state such as idx, which other APIs in the file consume.
**CN:** 该代码块声明了 idx 等支撑状态，供本文件中的其他 API 使用。

### Lines 85-88
```cpp
  85: private:
  86:   Node *node = nullptr;
  87:   size_t idx = 0;
  88: };
```
**EN:** This block stores supporting state such as nullptr, which other APIs in the file consume.
**CN:** 该代码块声明了 nullptr 等支撑状态，供本文件中的其他 API 使用。

### Lines 90-90
```cpp
  90: } // namespace mlir::triton::gpu::partition_scheduling_detail
```
**EN:** This block manages the namespace scope for the surrounding declarations.
**CN:** 该代码块负责管理周围声明所属的命名空间作用域。

### Lines 92-98
```cpp
  92: namespace llvm {
  93: template <>
  94: struct DenseMapInfo<mlir::triton::gpu::partition_scheduling_detail::Port> {
  95:   static inline mlir::triton::gpu::partition_scheduling_detail::Port
  96:   getEmptyKey() {
  97:     return {};
  98:   }
```
**EN:** This block declares or defines callable APIs such as getEmptyKey, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getEmptyKey 等可调用 API，用来封装这里提供的核心行为。

### Lines 100-103
```cpp
 100:   static inline mlir::triton::gpu::partition_scheduling_detail::Port
 101:   getTombstoneKey() {
 102:     return mlir::triton::gpu::partition_scheduling_detail::Port(0, 1);
 103:   }
```
**EN:** This block declares or defines callable APIs such as getTombstoneKey and Port, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getTombstoneKey and Port 等可调用 API，用来封装这里提供的核心行为。

### Lines 105-110
```cpp
 105:   static unsigned getHashValue(
 106:       const mlir::triton::gpu::partition_scheduling_detail::Port &port) {
 107:     return std::hash<mlir::triton::gpu::partition_scheduling_detail::Node *>()(
 108:                port.getNode()) ^
 109:            std::hash<size_t>()(port.getIdx());
 110:   }
```
**EN:** This block declares or defines callable APIs such as getHashValue, getNode, and getIdx, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getHashValue, getNode, and getIdx 等可调用 API，用来封装这里提供的核心行为。

### Lines 112-118
```cpp
 112:   static bool
 113:   isEqual(const mlir::triton::gpu::partition_scheduling_detail::Port &lhs,
 114:           const mlir::triton::gpu::partition_scheduling_detail::Port &rhs) {
 115:     return lhs == rhs;
 116:   }
 117: };
 118: } // namespace llvm
```
**EN:** This block declares or defines callable APIs such as isEqual, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isEqual 等可调用 API，用来封装这里提供的核心行为。

### Lines 120-120
```cpp
 120: namespace mlir::triton::gpu::partition_scheduling_detail {
```
**EN:** This block opens or closes the namespace scope used to organize APIs under mlir::triton::gpu::partition_scheduling_detail.
**CN:** 该代码块打开或关闭命名空间作用域，用来把相关 API 组织在 mlir::triton::gpu::partition_scheduling_detail 下。

### Lines 122-123
```cpp
 122: using InputPort = Port;
 123: using OutputPort = Port;
```
**EN:** This block stores supporting state such as Port, which other APIs in the file consume.
**CN:** 该代码块声明了 Port 等支撑状态，供本文件中的其他 API 使用。

### Lines 125-128
```cpp
 125: class Edge {
 126: public:
 127:   Edge() = default;
 128:   Edge(OutputPort from, InputPort to) : from(from), to(to) {}
```
**EN:** This block introduces `Edge`, the main class/struct defined here. Within the declaration, methods such as from and to expose its core API.
**CN:** 该代码块引入了此文件的核心类/结构体 `Edge`。 其中 from and to 等方法构成了它的主要接口。

### Lines 130-131
```cpp
 130:   OutputPort getFrom() const { return from; }
 131:   InputPort getTo() const { return to; }
```
**EN:** This block declares or defines callable APIs such as getFrom and getTo, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getFrom and getTo 等可调用 API，用来封装这里提供的核心行为。

### Lines 133-134
```cpp
 133:   Node *getFromNode() const { return from.getNode(); }
 134:   size_t getFromIdx() const { return from.getIdx(); }
```
**EN:** This block declares or defines callable APIs such as getFromNode, getNode, getFromIdx, and getIdx, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getFromNode, getNode, getFromIdx, and getIdx 等可调用 API，用来封装这里提供的核心行为。

### Lines 136-137
```cpp
 136:   Node *getToNode() const { return to.getNode(); }
 137:   size_t getToIdx() const { return to.getIdx(); }
```
**EN:** This block declares or defines callable APIs such as getToNode, getNode, getToIdx, and getIdx, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getToNode, getNode, getToIdx, and getIdx 等可调用 API，用来封装这里提供的核心行为。

### Lines 139-142
```cpp
 139:   bool isDataValue() const;
 140:   bool crossesPartitions() const;
 141:   Type getType() const;
 142:   size_t getSize() const;
```
**EN:** This block declares or defines callable APIs such as isDataValue, crossesPartitions, getType, and getSize, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isDataValue, crossesPartitions, getType, and getSize 等可调用 API，用来封装这里提供的核心行为。

### Lines 144-147
```cpp
 144: private:
 145:   OutputPort from;
 146:   InputPort to;
 147: };
```
**EN:** This block stores supporting state such as from and to, which other APIs in the file consume.
**CN:** 该代码块声明了 from and to 等支撑状态，供本文件中的其他 API 使用。

### Lines 149-151
```cpp
 149: class Node {
 150: public:
 151:   explicit Node(Operation *op) : op(op), cost(computeCost(op)) {}
```
**EN:** This block introduces `Node`, the main class/struct defined here. Within the declaration, methods such as op, cost, and computeCost expose its core API.
**CN:** 该代码块引入了此文件的核心类/结构体 `Node`。 其中 op, cost, and computeCost 等方法构成了它的主要接口。

### Lines 153-158
```cpp
 153:   Node(Node *parent, Operation *op, size_t numInputs, size_t numOutputs)
 154:       : parent(parent), op(op), cost(computeCost(op)) {
 155:     inputs.resize(numInputs);
 156:     outputs.resize(numOutputs);
 157:     dataOutputs.resize(numOutputs);
 158:   }
```
**EN:** This block declares or defines callable APIs such as Node, parent, op, cost, computeCost, and resize, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 Node, parent, op, cost, computeCost, and resize 等可调用 API，用来封装这里提供的核心行为。

### Lines 160-165
```cpp
 160:   Node(Node *parent, Value value, size_t numInputs, size_t numOutputs)
 161:       : parent(parent), value(value) {
 162:     inputs.resize(numInputs);
 163:     outputs.resize(numOutputs);
 164:     dataOutputs.resize(numOutputs);
 165:   }
```
**EN:** This block declares or defines callable APIs such as Node, parent, value, and resize, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 Node, parent, value, and resize 等可调用 API，用来封装这里提供的核心行为。

### Lines 167-169
```cpp
 167:   Node *addNode(Operation *op, size_t inputs, size_t outputs) {
 168:     return nodes.emplace_back(new Node(this, op, inputs, outputs)).get();
 169:   }
```
**EN:** This block declares or defines callable APIs such as addNode, emplace_back, Node, and get, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 addNode, emplace_back, Node, and get 等可调用 API，用来封装这里提供的核心行为。

### Lines 171-173
```cpp
 171:   Node *addNode(Value value, size_t inputs, size_t outputs) {
 172:     return nodes.emplace_back(new Node(this, value, inputs, outputs)).get();
 173:   }
```
**EN:** This block declares or defines callable APIs such as addNode, emplace_back, Node, and get, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 addNode, emplace_back, Node, and get 等可调用 API，用来封装这里提供的核心行为。

### Lines 175-183
```cpp
 175:   void walk(const std::function<void(Node *)> &fn) {
 176:     std::function<void(Node *)> do_walk = [&](Node *node) {
 177:       for (auto &child : node->getNodes()) {
 178:         fn(child.get());
 179:         do_walk(child.get());
 180:       }
 181:     };
 182:     do_walk(this);
 183:   }
```
**EN:** This block declares or defines callable APIs such as walk, void, getNodes, fn, get, and do_walk, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 walk, void, getNodes, fn, get, and do_walk 等可调用 API，用来封装这里提供的核心行为。

### Lines 185-188
```cpp
 185:   static void addEdge(OutputPort from, InputPort to) {
 186:     from.getNode()->addOutputEdge(from.getIdx(), to);
 187:     to.getNode()->addInputEdge(to.getIdx(), from);
 188:   }
```
**EN:** This block declares or defines callable APIs such as addEdge, getNode, addOutputEdge, getIdx, and addInputEdge, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 addEdge, getNode, addOutputEdge, getIdx, and addInputEdge 等可调用 API，用来封装这里提供的核心行为。

### Lines 190-193
```cpp
 190:   static void removeEdge(Edge edge) {
 191:     edge.getFromNode()->removeOutputEdge(edge.getFromIdx(), edge.getTo());
 192:     edge.getToNode()->removeInputEdge(edge.getToIdx(), edge.getFrom());
 193:   }
```
**EN:** This block declares or defines callable APIs such as removeEdge, getFromNode, removeOutputEdge, getFromIdx, getTo, getToNode, removeInputEdge, and getToIdx, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 removeEdge, getFromNode, removeOutputEdge, getFromIdx, getTo, getToNode, removeInputEdge, and getToIdx 等可调用 API，用来封装这里提供的核心行为。

### Lines 195-195
```cpp
 195:   void addDefines(Node *node) { defines.push_back(node); }
```
**EN:** This block declares or defines callable APIs such as addDefines and push_back, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 addDefines and push_back 等可调用 API，用来封装这里提供的核心行为。

### Lines 197-200
```cpp
 197:   void addInputEdge(size_t idx, OutputPort port) {
 198:     assert(idx < inputs.size());
 199:     inputs[idx] = port;
 200:   }
```
**EN:** This block declares or defines callable APIs such as addInputEdge and size, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 addInputEdge and size 等可调用 API，用来封装这里提供的核心行为。

### Lines 202-205
```cpp
 202:   void removeInputEdge(size_t idx, OutputPort port) {
 203:     assert(idx < inputs.size());
 204:     inputs[idx] = {};
 205:   }
```
**EN:** This block declares or defines callable APIs such as removeInputEdge and size, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 removeInputEdge and size 等可调用 API，用来封装这里提供的核心行为。

### Lines 207-210
```cpp
 207:   void addOutputEdge(size_t idx, InputPort port) {
 208:     assert(idx < outputs.size());
 209:     outputs[idx].push_back(port);
 210:   }
```
**EN:** This block declares or defines callable APIs such as addOutputEdge, size, and push_back, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 addOutputEdge, size, and push_back 等可调用 API，用来封装这里提供的核心行为。

### Lines 212-220
```cpp
 212:   void removeOutputEdge(size_t idx, InputPort port) {
 213:     assert(idx < outputs.size());
 214:     for (auto it = outputs[idx].begin(); it != outputs[idx].end(); it++) {
 215:       if (*it == port) {
 216:         outputs[idx].erase(it);
 217:         break;
 218:       }
 219:     }
 220:   }
```
**EN:** This block declares or defines callable APIs such as removeOutputEdge, size, begin, end, and erase, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 removeOutputEdge, size, begin, end, and erase 等可调用 API，用来封装这里提供的核心行为。

### Lines 222-230
```cpp
 222:   Node *getParent() const { return parent; }
 223:   bool isOp() const { return op; }
 224:   bool isValue() const { return !op; }
 225:   Operation *getOp() { return op; }
 226:   Value &getValue() {
 227:     assert(isValue());
 228:     return value;
 229:   }
 230:   const SmallVector<Node *> &getDefines() const { return defines; }
```
**EN:** This block declares or defines callable APIs such as getParent, isOp, isValue, getOp, getValue, and getDefines, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getParent, isOp, isValue, getOp, getValue, and getDefines 等可调用 API，用来封装这里提供的核心行为。

### Lines 232-232
```cpp
 232:   const SmallVector<std::unique_ptr<Node>> &getNodes() const { return nodes; }
```
**EN:** This block declares or defines callable APIs such as getNodes, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getNodes 等可调用 API，用来封装这里提供的核心行为。

### Lines 234-235
```cpp
 234:   size_t getNumInputs() const { return inputs.size(); }
 235:   size_t getNumOutputs() const { return outputs.size(); }
```
**EN:** This block declares or defines callable APIs such as getNumInputs, size, and getNumOutputs, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getNumInputs, size, and getNumOutputs 等可调用 API，用来封装这里提供的核心行为。

### Lines 237-243
```cpp
 237:   const SmallVector<OutputPort> &getInputs() const { return inputs; }
 238:   const SmallVector<SmallVector<InputPort>> &getOutputs() const {
 239:     return outputs;
 240:   }
 241:   SmallVector<InputPort> getOutputsFromPort(size_t idx) const {
 242:     return outputs[idx];
 243:   }
```
**EN:** This block declares or defines callable APIs such as getInputs, getOutputs, and getOutputsFromPort, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getInputs, getOutputs, and getOutputsFromPort 等可调用 API，用来封装这里提供的核心行为。

### Lines 245-253
```cpp
 245:   SmallVector<Edge> getInEdges() {
 246:     SmallVector<Edge> result;
 247:     size_t idx = 0;
 248:     for (auto input : inputs) {
 249:       result.push_back(Edge(input, InputPort(this, idx)));
 250:       idx++;
 251:     }
 252:     return result;
 253:   }
```
**EN:** This block declares or defines callable APIs such as getInEdges, push_back, Edge, and InputPort, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getInEdges, push_back, Edge, and InputPort 等可调用 API，用来封装这里提供的核心行为。

### Lines 255-265
```cpp
 255:   size_t getNumInDataEdges() {
 256:     size_t count = 0;
 257:     size_t idx = 0;
 258:     for (auto input : inputs) {
 259:       Edge edge(input, InputPort(this, idx));
 260:       if (edge.isDataValue())
 261:         count++;
 262:       idx++;
 263:     }
 264:     return count;
 265:   }
```
**EN:** This block declares or defines callable APIs such as getNumInDataEdges, edge, InputPort, and isDataValue, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getNumInDataEdges, edge, InputPort, and isDataValue 等可调用 API，用来封装这里提供的核心行为。

### Lines 267-276
```cpp
 267:   SmallVector<Edge> getOutEdges() {
 268:     SmallVector<Edge> result;
 269:     size_t idx = 0;
 270:     for (auto outputs : this->outputs) {
 271:       for (auto output : outputs)
 272:         result.push_back(Edge(OutputPort(this, idx), output));
 273:       idx++;
 274:     }
 275:     return result;
 276:   }
```
**EN:** This block declares or defines callable APIs such as getOutEdges, push_back, Edge, and OutputPort, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getOutEdges, push_back, Edge, and OutputPort 等可调用 API，用来封装这里提供的核心行为。

### Lines 278-287
```cpp
 278:   size_t getNumOutDataEdges() {
 279:     size_t count = 0;
 280:     size_t idx = 0;
 281:     for (auto output : dataOutputs) {
 282:       if (output)
 283:         count += outputs[idx].size();
 284:       idx++;
 285:     }
 286:     return count;
 287:   }
```
**EN:** This block declares or defines callable APIs such as getNumOutDataEdges and size, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getNumOutDataEdges and size 等可调用 API，用来封装这里提供的核心行为。

### Lines 289-292
```cpp
 289:   void setDataValue(size_t idx) {
 290:     assert(idx < dataOutputs.size());
 291:     dataOutputs[idx] = true;
 292:   }
```
**EN:** This block declares or defines callable APIs such as setDataValue and size, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 setDataValue and size 等可调用 API，用来封装这里提供的核心行为。

### Lines 294-297
```cpp
 294:   bool isDataValue(size_t idx) {
 295:     assert(idx < dataOutputs.size());
 296:     return dataOutputs[idx];
 297:   }
```
**EN:** This block declares or defines callable APIs such as isDataValue and size, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isDataValue and size 等可调用 API，用来封装这里提供的核心行为。

### Lines 299-309
```cpp
 299:   bool isData() {
 300:     // node is data if it consumes/produces a data value
 301:     if (std::any_of(dataOutputs.begin(), dataOutputs.end(),
 302:                     [](bool x) { return x; })) {
 303:       return true;
 304:     }
 305:     for (auto input : inputs)
 306:       if (input.getNode() && input.getNode()->isDataValue(input.getIdx()))
 307:         return true;
 308:     return false;
 309:   }
```
**EN:** This block declares or defines callable APIs such as isData, any_of, begin, end, getNode, isDataValue, and getIdx, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isData, any_of, begin, end, getNode, isDataValue, and getIdx 等可调用 API，用来封装这里提供的核心行为。

### Lines 311-320
```cpp
 311:   bool containsData() {
 312:     // node contains data if a data op appears in its region
 313:     for (auto &node : getNodes()) {
 314:       if (node->isData())
 315:         return true;
 316:       if (node->containsData())
 317:         return true;
 318:     }
 319:     return false;
 320:   }
```
**EN:** This block declares or defines callable APIs such as containsData, getNodes, and isData, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 containsData, getNodes, and isData 等可调用 API，用来封装这里提供的核心行为。

### Lines 322-333
```cpp
 322:   bool inLoopBody() {
 323:     if (op)
 324:       return op->getParentOfType<scf::ForOp>();
 325:     if (auto blockArg = dyn_cast<BlockArgument>(value)) {
 326:       auto parentOp = blockArg.getOwner()->getParentOp();
 327:       return isa<scf::ForOp>(parentOp) ||
 328:              parentOp->getParentOfType<scf::ForOp>();
 329:     }
 330:     auto result = cast<OpResult>(value);
 331:     auto op = result.getOwner();
 332:     return isa<scf::ForOp>(op) || op->getParentOfType<scf::ForOp>();
 333:   }
```
**EN:** This block declares or defines callable APIs such as inLoopBody, getOwner, and getParentOp, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 inLoopBody, getOwner, and getParentOp 等可调用 API，用来封装这里提供的核心行为。

### Lines 335-343
```cpp
 335:   bool containsLoopBody() {
 336:     for (auto &node : getNodes()) {
 337:       if (node->inLoopBody())
 338:         return true;
 339:       if (node->containsLoopBody())
 340:         return true;
 341:     }
 342:     return false;
 343:   }
```
**EN:** This block declares or defines callable APIs such as containsLoopBody, getNodes, and inLoopBody, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 containsLoopBody, getNodes, and inLoopBody 等可调用 API，用来封装这里提供的核心行为。

### Lines 345-361
```cpp
 345:   std::string getLabel() {
 346:     if (op)
 347:       return op->getName().getStringRef().str();
 348:     if (auto blockArg = dyn_cast<BlockArgument>(value)) {
 349:       auto parentOp = blockArg.getOwner()->getParentOp();
 350:       if (isa<tt::FuncOp>(parentOp))
 351:         return "arg " + std::to_string(blockArg.getArgNumber());
 352:       if (isa<scf::ForOp>(parentOp)) {
 353:         if (blockArg.getArgNumber() == 0)
 354:           return "ind var";
 355:         return "iter arg " + std::to_string(blockArg.getArgNumber() - 1);
 356:       }
 357:       return "?";
 358:     }
 359:     auto result = cast<OpResult>(value);
 360:     return "result " + std::to_string(result.getResultNumber());
 361:   }
```
**EN:** This block declares or defines callable APIs such as getLabel, getName, getStringRef, str, getOwner, getParentOp, to_string, and getArgNumber, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getLabel, getName, getStringRef, str, getOwner, getParentOp, to_string, and getArgNumber 等可调用 API，用来封装这里提供的核心行为。

### Lines 363-369
```cpp
 363:   void setPartition(Partition *partition) {
 364:     for (auto current_partition : partitions)
 365:       current_partition->remove(this);
 366:     partitions.clear();
 367:     partitions.insert(partition);
 368:     partition->add(this);
 369:   }
```
**EN:** This block declares or defines callable APIs such as setPartition, remove, clear, insert, and add, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 setPartition, remove, clear, insert, and add 等可调用 API，用来封装这里提供的核心行为。

### Lines 371-374
```cpp
 371:   void addPartition(Partition *partition) {
 372:     partitions.insert(partition);
 373:     partition->add(this);
 374:   }
```
**EN:** This block declares or defines callable APIs such as addPartition, insert, and add, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 addPartition, insert, and add 等可调用 API，用来封装这里提供的核心行为。

### Lines 376-380
```cpp
 376:   void addPartitions(const SetVector<Partition *> &partitions) {
 377:     this->partitions.insert(partitions.begin(), partitions.end());
 378:     for (auto partition : partitions)
 379:       partition->add(this);
 380:   }
```
**EN:** This block declares or defines callable APIs such as addPartitions, insert, begin, end, and add, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 addPartitions, insert, begin, end, and add 等可调用 API，用来封装这里提供的核心行为。

### Lines 382-382
```cpp
 382:   bool hasPartition() const { return !partitions.empty(); }
```
**EN:** This block declares or defines callable APIs such as hasPartition and empty, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 hasPartition and empty 等可调用 API，用来封装这里提供的核心行为。

### Lines 384-387
```cpp
 384:   Partition *getPartition() const {
 385:     assert(partitions.size() == 1);
 386:     return *(partitions.begin());
 387:   }
```
**EN:** This block declares or defines callable APIs such as getPartition, size, and begin, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getPartition, size, and begin 等可调用 API，用来封装这里提供的核心行为。

### Lines 389-389
```cpp
 389:   const SetVector<Partition *> &getPartitions() const { return partitions; }
```
**EN:** This block declares or defines callable APIs such as getPartitions, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getPartitions 等可调用 API，用来封装这里提供的核心行为。

### Lines 391-395
```cpp
 391:   bool hasCost() const { return cost > 0; }
 392:   size_t getCost() const {
 393:     assert(hasCost());
 394:     return cost;
 395:   }
```
**EN:** This block declares or defines callable APIs such as hasCost and getCost, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 hasCost and getCost 等可调用 API，用来封装这里提供的核心行为。

### Lines 397-397
```cpp
 397:   void dump() { llvm::errs() << "node '" << getLabel() << "'\n"; }
```
**EN:** This block declares or defines callable APIs such as dump, errs, and getLabel, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 dump, errs, and getLabel 等可调用 API，用来封装这里提供的核心行为。

### Lines 399-403
```cpp
 399: private:
 400:   Node *parent = nullptr;
 401:   Operation *op = nullptr;
 402:   Value value;
 403:   size_t cost = 0;
```
**EN:** This block stores supporting state such as nullptr and value, which other APIs in the file consume.
**CN:** 该代码块声明了 nullptr and value 等支撑状态，供本文件中的其他 API 使用。

### Lines 405-406
```cpp
 405:   SmallVector<std::unique_ptr<Node>> nodes;
 406:   SmallVector<Node *> defines;
```
**EN:** This block stores supporting state such as nodes and defines, which other APIs in the file consume.
**CN:** 该代码块声明了 nodes and defines 等支撑状态，供本文件中的其他 API 使用。

### Lines 408-410
```cpp
 408:   SmallVector<OutputPort> inputs;
 409:   SmallVector<SmallVector<InputPort>> outputs;
 410:   SmallVector<bool> dataOutputs;
```
**EN:** This block stores supporting state such as inputs, outputs, and dataOutputs, which other APIs in the file consume.
**CN:** 该代码块声明了 inputs, outputs, and dataOutputs 等支撑状态，供本文件中的其他 API 使用。

### Lines 412-413
```cpp
 412:   SetVector<Partition *> partitions;
 413: };
```
**EN:** This block stores supporting state such as partitions, which other APIs in the file consume.
**CN:** 该代码块声明了 partitions 等支撑状态，供本文件中的其他 API 使用。

### Lines 415-417
```cpp
 415: class Graph {
 416: public:
 417:   explicit Graph(Operation *op) : root(new Node(op)) {}
```
**EN:** This block introduces `Graph`, the main class/struct defined here. Within the declaration, methods such as root and Node expose its core API.
**CN:** 该代码块引入了此文件的核心类/结构体 `Graph`。 其中 root and Node 等方法构成了它的主要接口。

### Lines 419-419
```cpp
 419:   Node *getRoot() { return root.get(); }
```
**EN:** This block declares or defines callable APIs such as getRoot and get, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getRoot and get 等可调用 API，用来封装这里提供的核心行为。

### Lines 421-425
```cpp
 421:   Partition *addPartition() {
 422:     auto partition = partition_storage.emplace_back(new Partition(this)).get();
 423:     partitions.insert(partition);
 424:     return partition;
 425:   }
```
**EN:** This block declares or defines callable APIs such as addPartition, emplace_back, Partition, get, and insert, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 addPartition, emplace_back, Partition, get, and insert 等可调用 API，用来封装这里提供的核心行为。

### Lines 427-430
```cpp
 427:   void erasePartition(Partition *partition) {
 428:     assert(partition->empty());
 429:     partitions.remove(partition);
 430:   }
```
**EN:** This block declares or defines callable APIs such as erasePartition, empty, and remove, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 erasePartition, empty, and remove 等可调用 API，用来封装这里提供的核心行为。

### Lines 432-432
```cpp
 432:   const SetVector<Partition *> &getPartitions() const { return partitions; }
```
**EN:** This block declares or defines callable APIs such as getPartitions, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getPartitions 等可调用 API，用来封装这里提供的核心行为。

### Lines 434-442
```cpp
 434:   void walk(const std::function<void(Node *)> &fn) {
 435:     std::function<void(Node *)> do_walk = [&](Node *node) {
 436:       for (auto &child : node->getNodes()) {
 437:         fn(child.get());
 438:         do_walk(child.get());
 439:       }
 440:     };
 441:     do_walk(root.get());
 442:   }
```
**EN:** This block declares or defines callable APIs such as walk, void, getNodes, fn, get, and do_walk, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 walk, void, getNodes, fn, get, and do_walk 等可调用 API，用来封装这里提供的核心行为。

### Lines 444-448
```cpp
 444: private:
 445:   std::unique_ptr<Node> root;
 446:   SetVector<Partition *> partitions;
 447:   SmallVector<std::unique_ptr<Partition>> partition_storage;
 448: };
```
**EN:** This block stores supporting state such as root, partitions, and partition_storage, which other APIs in the file consume.
**CN:** 该代码块声明了 root, partitions, and partition_storage 等支撑状态，供本文件中的其他 API 使用。

### Lines 450-453
```cpp
 450: struct VisualizationInfo {
 451:   DenseMap<Partition *, size_t> partition_ids;
 452:   DenseMap<Partition *, std::string> partition_colors;
 453: };
```
**EN:** This block introduces `VisualizationInfo`, the main class/struct defined here.
**CN:** 该代码块引入了此文件的核心类/结构体 `VisualizationInfo`。

### Lines 455-456
```cpp
 455: void visualize(std::string key, std::string filename, std::string title,
 456:                Graph *graph, VisualizationInfo &info);
```
**EN:** This block declares or defines callable APIs such as visualize, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 visualize 等可调用 API，用来封装这里提供的核心行为。

### Lines 458-458
```cpp
 458: } // namespace mlir::triton::gpu::partition_scheduling_detail
```
**EN:** This block manages the namespace scope for the surrounding declarations.
**CN:** 该代码块负责管理周围声明所属的命名空间作用域。

### Lines 460-460
```cpp
 460: #endif // TRITON_TRITONGPU_TRANSFORMS_PARTITION_SCHEDULING_UTILITY_H_
```
**EN:** This block closes the header guard started at the top of the file.
**CN:** 该代码块结束文件顶部开始的头文件保护宏。

## Key Concepts / 关键概念
- **EN:** layout conversion and reasoning  
  **CN:** 布局转换与推理
- **EN:** matrix-multiply acceleration  
  **CN:** 矩阵乘加加速
- **EN:** LLVM lowering  
  **CN:** LLVM 降级
- **EN:** MLIR dialect definition  
  **CN:** MLIR 方言定义
- **EN:** operation definitions  
  **CN:** 操作定义
- **EN:** types  
  **CN:** 类型
- **EN:** traits  
  **CN:** 特征约束
- **EN:** scheduling  
  **CN:** 调度

## Dependencies / 依赖关系
- **Project includes / 项目内依赖:**
  - `mlir/Support/LLVM.h`
  - `triton/Dialect/Triton/IR/Dialect.h`
  - `triton/Dialect/TritonGPU/IR/Dialect.h`
  - `triton/Dialect/TritonNvidiaGPU/IR/Dialect.h`
