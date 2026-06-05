# PartitionSchedulingUtility.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `lib/Dialect/TritonGPU/Transforms/WarpSpecialization/PartitionSchedulingUtility.cpp`
- **Purpose / 作用:** **EN:** Implements the Partition Scheduling Utility transformation or optimization pass for the TritonGPU pipeline. **CN:** 为 TritonGPU 编译流程实现与 Partition Scheduling Utility 相关的变换或优化 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-3

```cpp
   1: #include "triton/Dialect/TritonGPU/Transforms/PartitionSchedulingUtility.h"
   2: #include "mlir/Support/LLVM.h"
   3: #include "triton/Tools/Sys/GetEnv.h"
```

- **EN:** Includes the interfaces this file depends on. Triton headers (`PartitionSchedulingUtility.h`, `GetEnv.h`) provide domain-specific IR/support, MLIR headers (`LLVM.h`) provide rewriting and analysis infrastructure, LLVM headers (None) supply low-level utilities, and standard/library headers (None) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（`PartitionSchedulingUtility.h`, `GetEnv.h`）提供领域特定 IR/支持逻辑，MLIR 头文件（`LLVM.h`）提供重写与分析基础设施，LLVM 头文件（None）提供底层工具，而标准/通用库头文件（None）提供通用能力。
### Lines 5-6

```cpp
   5: #include <iomanip>
   6: #include <sstream>
```

- **EN:** Includes the interfaces this file depends on. Triton headers (None) provide domain-specific IR/support, MLIR headers (None) provide rewriting and analysis infrastructure, LLVM headers (None) supply low-level utilities, and standard/library headers (`iomanip`, `sstream`) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（None）提供领域特定 IR/支持逻辑，MLIR 头文件（None）提供重写与分析基础设施，LLVM 头文件（None）提供底层工具，而标准/通用库头文件（`iomanip`, `sstream`）提供通用能力。
### Lines 8-8

```cpp
   8: namespace mlir::triton::gpu::partition_scheduling_detail {
```

- **EN:** Opens or closes the namespace nesting for mlir::triton::gpu::partition_scheduling_detail, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 mlir::triton::gpu::partition_scheduling_detail 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 10-26

```cpp
  10: llvm::raw_ostream &operator<<(llvm::raw_ostream &stream, Flags flags) {
  11:   std::vector<std::string> strs;
  12:   if (flags == Flags::NONE) {
  13:     strs.push_back("NONE");
  14:   } else {
  15:     if (flags & Flags::MANUAL)
  16:       strs.push_back("MANUAL");
  17:     if (flags & Flags::LOAD)
  18:       strs.push_back("LOAD");
  19:     if (flags & Flags::STORE)
  20:       strs.push_back("STORE");
  21:     if (flags & Flags::MMA)
  22:       strs.push_back("MMA");
  23:     if (flags & Flags::TMEM)
  24:       strs.push_back("TMEM");
  25:     if (flags & Flags::SFU)
  26:       strs.push_back("SFU");
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 27-36

```cpp
  27:     if (flags & Flags::VIEW)
  28:       strs.push_back("VIEW");
  29:   }
  30:   for (size_t i = 0; i < strs.size(); i++) {
  31:     if (i != 0)
  32:       stream << "|";
  33:     stream << strs[i];
  34:   }
  35:   return stream;
  36: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 38-40

```cpp
  38: Flags getNodeFlags(Node *node) {
  39:   if (node->isOp()) {
  40:     auto op = node->getOp();
```

- **EN:** Defines accessor/helper `getNodeFlags` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getNodeFlags`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 42-44

```cpp
  42:     // if it is manually tagged with a node type
  43:     if (op->hasAttr("store"))
  44:       return Flags::STORE;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 46-60

```cpp
  46:     if (isa<tt::DescriptorLoadLikeOpInterface>(op))
  47:       return Flags::LOAD;
  48:     if (isa<tt::DescriptorStoreLikeOpInterface>(op))
  49:       return Flags::STORE;
  50:     if (isa<ttng::MMAv5OpInterface>(op) || op->hasAttr("mma"))
  51:       return Flags::MMA;
  52:     if (isa<ttng::TMEMLoadOp, ttng::TMEMStoreOp>(op))
  53:       return Flags::TMEM;
  54:     if (isa<math::Exp2Op>(op))
  55:       return Flags::SFU;
  56:     if (isViewOp(op))
  57:       return Flags::VIEW;
  58:   }
  59:   return Flags::NONE;
  60: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 62-75

```cpp
  62: size_t computeCost(Operation *op) {
  63:   if (auto mma = dyn_cast<nvidia_gpu::MMAv5OpInterface>(op)) {
  64:     auto a = mma.getA();
  65:     auto b = mma.getB();
  66:     auto a_shape = a.getType().getShape();
  67:     auto b_shape = b.getType().getShape();
  68:     assert(a_shape.size() == 2);
  69:     assert(b_shape.size() == 2);
  70:     auto M = a_shape[0];
  71:     auto N = b_shape[0];
  72:     auto K = a_shape[1];
  73:     auto cycles = M * N * K / 8192;
  74:     return cycles;
  75:   }
```

- **EN:** Defines helper `computeCost` that computes or constructs intermediate data used by the surrounding transformation. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义辅助函数 `computeCost`，用于计算或构造外围变换所需的中间数据。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 77-84

```cpp
  77:   if (isa<math::Exp2Op, ElementwiseInlineAsmOp>(op)) {
  78:     int elementCount = 0;
  79:     for (Type type : op->getResultTypes()) {
  80:       if (auto tensorTy = dyn_cast<RankedTensorType>(type))
  81:         elementCount += tensorTy.getNumElements();
  82:     }
  83:     return elementCount;
  84:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 86-87

```cpp
  86:   return 0;
  87: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 89-90

```cpp
  89: void Partition::add(Node *node) {
  90:   auto node_flags = getNodeFlags(node);
```

- **EN:** Defines `Partition::add`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `Partition::add`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 92-99

```cpp
  92:   // Note: only set view flag for partition,
  93:   // if it consists of all view ops
  94:   // FIXME: have a set kinds of flag to make this generic?
  95:   bool all_view = true;
  96:   if (!nodes.empty() && !(flags & Flags::VIEW))
  97:     all_view = false;
  98:   if (!(node_flags & Flags::VIEW))
  99:     all_view = false;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 101-101

```cpp
 101:   nodes.insert(node);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 103-105

```cpp
 103:   flags |= node_flags;
 104:   if (!all_view)
 105:     flags = static_cast<Flags>(flags & ~Flags::VIEW);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 107-109

```cpp
 107:   if (node->hasCost())
 108:     cost += node->getCost();
 109: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 111-112

```cpp
 111: void Partition::merge(Partition *lhs, Partition *rhs) {
 112:   assert(lhs != rhs);
```

- **EN:** Defines `Partition::merge`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `Partition::merge`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 114-116

```cpp
 114:   // Should never be merging MANUAL partitions
 115:   assert(!((lhs->getFlags() & Flags::MANUAL) &&
 116:            (rhs->getFlags() & Flags::MANUAL)));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 118-121

```cpp
 118:   // Always keep the MANUAL partition,
 119:   // and prefer emptying the NONE partition
 120:   if (lhs->getFlags() & Flags::MANUAL || rhs->getFlags() == Flags::NONE)
 121:     std::swap(lhs, rhs);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 123-126

```cpp
 123:   auto nodes = lhs->getNodes();
 124:   for (auto node : nodes) {
 125:     node->setPartition(rhs);
 126:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 128-130

```cpp
 128:   // remove the now empty partition
 129:   lhs->graph->erasePartition(lhs);
 130: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 132-139

```cpp
 132: void Partition::dump() const {
 133:   llvm::errs() << "Partition@" << this << " {\n"
 134:                << "  id=" << id << "\n"
 135:                << "  size=" << nodes.size() << "\n"
 136:                << "  cost=" << cost << "\n"
 137:                << "  flags=" << flags << "\n"
 138:                << "}\n";
 139: }
```

- **EN:** Defines `Partition::dump`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `Partition::dump`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 141-145

```cpp
 141: bool Edge::isDataValue() const {
 142:   if (!from.getNode())
 143:     return false;
 144:   return from.getNode()->isDataValue(from.getIdx());
 145: }
```

- **EN:** Defines `Edge::isDataValue`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `Edge::isDataValue`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 147-158

```cpp
 147: bool Edge::crossesPartitions() const {
 148:   if (!isDataValue())
 149:     return false;
 150:   if (!from.getNode()->hasPartition() || !to.getNode()->hasPartition())
 151:     return false;
 152:   // FIXME: only considers edges between nodes assigned to single partitions
 153:   // as crossing a boundary
 154:   if (from.getNode()->getPartitions().size() != 1 ||
 155:       to.getNode()->getPartitions().size() != 1)
 156:     return false;
 157:   return from.getNode()->getPartition() != to.getNode()->getPartition();
 158: }
```

- **EN:** Defines `Edge::crossesPartitions`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `Edge::crossesPartitions`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 160-165

```cpp
 160: Type Edge::getType() const {
 161:   auto fromNode = from.getNode();
 162:   if (fromNode->isOp())
 163:     return fromNode->getOp()->getResult(from.getIdx()).getType();
 164:   return fromNode->getValue().getType();
 165: }
```

- **EN:** Defines accessor/helper `Edge::getType` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `Edge::getType`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 167-168

```cpp
 167: size_t Edge::getSize() const {
 168:   auto type = getType();
```

- **EN:** Defines accessor/helper `Edge::getSize` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `Edge::getSize`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 170-175

```cpp
 170:   if (auto tensor = dyn_cast<TensorType>(type)) {
 171:     size_t size = 1;
 172:     for (auto x : tensor.getShape())
 173:       size *= x;
 174:     return size;
 175:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 177-182

```cpp
 177:   if (auto memdesc = dyn_cast<MemDescType>(type)) {
 178:     size_t size = 1;
 179:     for (auto x : memdesc.getShape())
 180:       size *= x;
 181:     return size;
 182:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。
### Lines 184-185

```cpp
 184:   return 1;
 185: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 187-188

```cpp
 187: void visualize(std::string key, std::string filename, std::string title,
 188:                Graph *graph, VisualizationInfo &info) {
```

- **EN:** Defines `visualize`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `visualize`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 190-191

```cpp
 190:   if (!tools::getBoolEnv("TRITON_PARTITION_SCHEDULING_ENABLE_DUMP_DOT"))
 191:     return;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 193-196

```cpp
 193:   const auto dump_data_only =
 194:       tools::getBoolEnv("TRITON_PARTITION_SCHEDULING_DUMP_DATA_ONLY");
 195:   const auto dump_loop_only =
 196:       tools::getBoolEnv("TRITON_PARTITION_SCHEDULING_DUMP_LOOP_ONLY");
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 198-203

```cpp
 198:   static std::map<std::string, int> keys;
 199:   if (keys.find(key) == keys.end()) {
 200:     keys[key] = 0;
 201:   }
 202:   auto idx = keys[key];
 203:   keys[key]++;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 205-207

```cpp
 205:   std::stringstream path;
 206:   path << "graph-" << key << "-" << std::setfill('0') << std::setw(4) << idx
 207:        << "-" << filename << ".dot";
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 209-211

```cpp
 209:   std::error_code err;
 210:   llvm::raw_fd_ostream dot(path.str(), err);
 211:   assert(!err);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 213-216

```cpp
 213:   dot << "digraph G {\n";
 214:   dot << "label = \"" << title << "\";\n";
 215:   dot << "labelloc=\"t\";\n";
 216:   dot << "labeljust=\"c\";\n";
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 218-218

```cpp
 218:   DenseMap<Node *, size_t> node_ids;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 220-224

```cpp
 220:   auto getPartitionId = [&](Partition *partition) {
 221:     if (info.partition_ids.count(partition) == 0)
 222:       info.partition_ids[partition] = info.partition_ids.size();
 223:     return info.partition_ids[partition];
 224:   };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 226-234

```cpp
 226:   auto getPartitionColor = [&](Partition *partition) {
 227:     if (info.partition_colors.count(partition) == 0) {
 228:       size_t color = info.partition_colors.size() + 1;
 229:       color = (color % 12) + 1;
 230:       info.partition_colors[partition] =
 231:           std::string("/set312/") + std::to_string(color);
 232:     }
 233:     return info.partition_colors[partition];
 234:   };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 236-239

```cpp
 236:   // add nodes
 237:   std::function<void(Node *)> visitNodes = [&](Node *graph) {
 238:     for (auto &node_obj : graph->getNodes()) {
 239:       auto node = node_obj.get();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 241-247

```cpp
 241:       if (dump_data_only && !node->isData() && !node->containsData())
 242:         // skip if dumping data nodes only, and this op is non-data or doesn't
 243:         // contain a data node
 244:         continue;
 245:       if (dump_loop_only && !node->inLoopBody() && !node->containsLoopBody())
 246:         // skip if dumping loop body nodes only
 247:         continue;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 249-249

```cpp
 249:       node_ids[node] = node_ids.size();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 251-268

```cpp
 251:       if (!node->getNodes().empty())
 252:         dot << "subgraph cluster_cx" << node_ids[node] << " {\n"
 253:             << "label=\"\"\n";
 254:       dot << "x" << node_ids[node] << "[shape=plaintext, ";
 255:       if (node->isData())
 256:         dot << "color=blue, ";
 257:       dot << "label=<";
 258:       dot << "<TABLE BORDER=\"0\" CELLBORDER=\"1\" CELLSPACING=\"0\">";
 259:       if (node->getNumInputs() > 1) {
 260:         dot << "<TR>";
 261:         for (size_t idx = 0; idx < node->getNumInputs(); idx++)
 262:           dot << "<TD PORT=\"in" << idx << "\">" << idx << "</TD>";
 263:         dot << "</TR>";
 264:       }
 265:       dot << "<TR><TD PORT=\"inout\"";
 266:       size_t colspan = std::max(node->getNumInputs(), node->getNumOutputs());
 267:       if (colspan > 0)
 268:         dot << " COLSPAN=\"" << colspan << "\"";
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 269-269

```cpp
 269:       dot << ">";
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 271-284

```cpp
 271:       dot << "<TABLE BORDER=\"0\" CELLBORDER=\"0\" CELLSPACING=\"0\"><TR>";
 272:       if (node->hasPartition()) {
 273:         for (auto partition : node->getPartitions()) {
 274:           auto name = std::to_string(getPartitionId(partition));
 275:           dot << "<TD BGCOLOR=\"" << getPartitionColor(partition) << "\">"
 276:               << name << "{" << partition->getCost() << "}"
 277:               << "[" << partition->getFlags() << "]</TD>";
 278:         }
 279:       }
 280:       dot << "<TD>" << node->getLabel();
 281:       if (node->isData())
 282:         dot << " [" << getNodeFlags(node) << "]";
 283:       dot << "</TD></TR></TABLE>";
 284:       dot << "</TD></TR>";
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 286-293

```cpp
 286:       if (node->hasCost()) {
 287:         dot << "<TR><TD";
 288:         if (colspan > 0)
 289:           dot << " COLSPAN=\"" << colspan << "\"";
 290:         dot << ">";
 291:         dot << "cost:" << node->getCost();
 292:         dot << "</TD></TR>";
 293:       }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 295-308

```cpp
 295:       if (node->getNumOutputs() > 1) {
 296:         dot << "<TR>";
 297:         for (size_t idx = 0; idx < node->getNumOutputs(); idx++)
 298:           dot << "<TD PORT=\"out" << idx << "\">" << idx << "</TD>";
 299:         dot << "</TR>";
 300:       }
 301:       dot << "</TABLE>>];\n";
 302:       if (!node->getNodes().empty()) {
 303:         visitNodes(node);
 304:         dot << "}\n";
 305:       }
 306:     }
 307:   };
 308:   visitNodes(graph->getRoot());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 310-327

```cpp
 310:   // add edges
 311:   std::function<void(Node *)> visitEdges = [&](Node *node) {
 312:     size_t idx = 0;
 313:     for (auto inputPorts : node->getOutputs()) {
 314:       OutputPort outputPort{node, idx};
 315:       for (auto inputPort : inputPorts) {
 316:         Edge edge(outputPort, inputPort);
 317:         if (node_ids.count(outputPort.getNode()) == 0 ||
 318:             node_ids.count(inputPort.getNode()) == 0)
 319:           continue;
 320:         dot << "x" << node_ids[outputPort.getNode()];
 321:         dot << ":";
 322:         if (outputPort.getNode()->getNumOutputs() == 1)
 323:           dot << "inout";
 324:         else
 325:           dot << "out" << outputPort.getIdx();
 326:         dot << " -> ";
 327:         dot << "x" << node_ids[inputPort.getNode()];
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 328-345

```cpp
 328:         dot << ":";
 329:         if (inputPort.getNode()->getNumInputs() == 1)
 330:           dot << "inout";
 331:         else
 332:           dot << "in" << inputPort.getIdx();
 333:         std::vector<std::string> attrs;
 334:         if (edge.isDataValue()) {
 335:           if (edge.getFromNode()->getPartitions().size() > 1 ||
 336:               edge.getToNode()->getPartitions().size() > 1)
 337:             // invalid edge, should only have one partition
 338:             attrs.push_back("color=\"green\"");
 339:           else if (edge.crossesPartitions())
 340:             attrs.push_back("color=\"red\"");
 341:           else
 342:             attrs.push_back("color=\"blue\"");
 343:           auto size = edge.getSize();
 344:           if (size != 1) {
 345:             attrs.push_back("label=\"" + std::to_string(size) + "\"");
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 346-363

```cpp
 346:           }
 347:         }
 348:         if (!attrs.empty()) {
 349:           dot << "[";
 350:           for (auto attr = attrs.begin(); attr != attrs.end(); attr++) {
 351:             if (attr != attrs.begin()) {
 352:               dot << ",";
 353:             }
 354:             dot << *attr;
 355:           }
 356:           dot << "]";
 357:         }
 358:         dot << ";\n";
 359:       }
 360:       idx++;
 361:     }
 362:     for (auto &node : node->getNodes())
 363:       visitEdges(node.get());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 364-365

```cpp
 364:   };
 365:   visitEdges(graph->getRoot());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 367-368

```cpp
 367:   dot << "}\n";
 368: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 370-370

```cpp
 370: } // namespace mlir::triton::gpu::partition_scheduling_detail
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。

## Key Concepts / 关键概念
- **EN:** The main concern is pass-driven transformation around partition scheduling utility.
  **CN:** 核心关注点是围绕 Partition Scheduling Utility 的 pass 驱动变换。
- **EN:** Memory descriptor types carry layout, rank, and address-space information.
  **CN:** 内存描述符类型携带布局、维度和地址空间信息。
- **EN:** Tensor shape and element-type reasoning is central to the implementation.
  **CN:** 张量形状与元素类型推导是实现中的核心内容。
- **EN:** Tensor memory specific allocation, layout, or synchronization rules are important here.
  **CN:** 这里重点处理张量内存（TMEM）的分配、布局或同步规则。

## Dependencies / 依赖关系
- **Internal Triton headers / Triton 内部头文件:** `triton/Dialect/TritonGPU/Transforms/PartitionSchedulingUtility.h`, `triton/Tools/Sys/GetEnv.h`
- **MLIR headers / MLIR 头文件:** `mlir/Support/LLVM.h`
- **LLVM headers / LLVM 头文件:** None
- **Standard/library headers / 标准或通用库头文件:** `iomanip`, `sstream`
- **Primary APIs used / 主要 API:** `RankedTensorType`, `MemDescType`
- **Pipeline role / 流程角色:** This file participates in Triton pass pipelines and usually expects upstream analyses or dialect invariants to have prepared the IR. / 本文件参与 Triton 的 pass 流程，通常依赖上游分析或方言不变量先把 IR 准备好。
