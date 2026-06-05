# PartitionScheduling.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `lib/Dialect/TritonGPU/Transforms/WarpSpecialization/PartitionScheduling.cpp`
- **Purpose / 作用:** **EN:** Implements the Partition Scheduling transformation or optimization pass for the TritonGPU pipeline. **CN:** 为 TritonGPU 编译流程实现与 Partition Scheduling 相关的变换或优化 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-10

```cpp
   1: #include "PartitionAttrs.h"
   2: #include "mlir/Pass/Pass.h"
   3: #include "mlir/Support/LLVM.h"
   4: #include "triton/Dialect/Triton/IR/Dialect.h"
   5: #include "triton/Dialect/TritonGPU/IR/Dialect.h"
   6: #include "triton/Dialect/TritonGPU/Transforms/Partition.h"
   7: #include "triton/Dialect/TritonGPU/Transforms/PartitionSchedulingUtility.h"
   8: #include "triton/Dialect/TritonGPU/Transforms/PipeliningUtility.h"
   9: #include "triton/Dialect/TritonNvidiaGPU/IR/Dialect.h"
  10: #include "llvm/Support/Debug.h"
```

- **EN:** Includes the interfaces this file depends on. Triton headers (`Dialect.h`, `Dialect.h`, `Partition.h`, `PartitionSchedulingUtility.h`, ... (+2 more)) provide domain-specific IR/support, MLIR headers (`Pass.h`, `LLVM.h`) provide rewriting and analysis infrastructure, LLVM headers (`Debug.h`) supply low-level utilities, and standard/library headers (`PartitionAttrs.h`) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（`Dialect.h`, `Dialect.h`, `Partition.h`, `PartitionSchedulingUtility.h`, ... (+2 more)）提供领域特定 IR/支持逻辑，MLIR 头文件（`Pass.h`, `LLVM.h`）提供重写与分析基础设施，LLVM 头文件（`Debug.h`）提供底层工具，而标准/通用库头文件（`PartitionAttrs.h`）提供通用能力。
### Lines 12-12

```cpp
  12: namespace mlir::triton::gpu {
```

- **EN:** Opens or closes the namespace nesting for mlir::triton::gpu, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 mlir::triton::gpu 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 14-31

```cpp
  14: // This pass assigns partitions to ops within each warp specialized loop.
  15: //
  16: // Ops are first categorized as either "data" ops (which operate on tiles of
  17: // data, for example load/store/mma ops) or "non-data" ops (for example index
  18: // calculations).
  19: //
  20: // A dataflow graph representation of the program is constructed: every edge in
  21: // the graph represents an MLIR value, and every node represents an MLIR
  22: // operation or block argument.
  23: //
  24: // Initially all nodes for "data" ops are assigned to a new partition. A set of
  25: // heuristics is then applied to every edge that crosses partitions (connects a
  26: // pair of nodes assigned to different partitions). When a heuristic matches,
  27: // the two partitions are merged into a single partition. This is done up until
  28: // a fixed point is reached. A second set of heuristics is run on every
  29: // pair of partitions, merging them until a fixed point is reached.
  30: //
  31: // After the heuristics have been applied, all data ops are assigned to a
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 32-37

```cpp
  32: // single partition. These partition assignments are then propagated to all
  33: // "non-data" ops. This pulls all of the necessary index calculations etc. into
  34: // the partitions that require them (possibly multiple).
  35: //
  36: // Finally the partition assignments in the dataflow graph are serialized to
  37: // attributes, and the temporary data structure is discarded.
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 39-40

```cpp
  39: #define GEN_PASS_DEF_TRITONGPUPARTITIONSCHEDULING
  40: #include "triton/Dialect/TritonGPU/Transforms/Passes.h.inc"
```

- **EN:** Pulls in generated definitions from TableGen/MLIR include fragments so the handwritten code can reuse auto-generated declarations.
- **CN:** 这里引入由 TableGen/MLIR 生成的定义片段，使手写代码能够复用自动生成的声明。
### Lines 42-44

```cpp
  42: #define DEBUG_TYPE "tritongpu-partition-scheduling"
  43: #define DBGS() (llvm::dbgs() << "[" DEBUG_TYPE "]: ")
  44: #define LDBG(X) LLVM_DEBUG(DBGS() << X << "\n")
```

- **EN:** Defines debug logging helpers and compile-time tags used when tracing this pass or utility at runtime.
- **CN:** 这里定义调试日志辅助宏和编译期标签，用于在运行时跟踪该 pass 或工具。
### Lines 46-46

```cpp
  46: namespace {
```

- **EN:** Opens or closes the namespace nesting for (anonymous), keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 (anonymous) 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 48-50

```cpp
  48: using namespace mlir;
  49: using namespace triton;
  50: using namespace partition_scheduling_detail;
```

- **EN:** Introduces namespace aliases/imports (`mlir`, `triton`, `partition_scheduling_detail`) so the rest of the file can use MLIR/Triton symbols without repetitive qualification.
- **CN:** 这里通过命名空间导入（`mlir`, `triton`, `partition_scheduling_detail`）减少后续代码中的重复限定，使 MLIR/Triton 符号使用更简洁。
### Lines 52-54

```cpp
  52: namespace tt = triton;
  53: namespace ttg = triton::gpu;
  54: namespace ttng = triton::nvidia_gpu;
```

- **EN:** Opens or closes the namespace nesting for tt -> ttg -> ttng, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 tt -> ttg -> ttng 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 56-56

```cpp
  56: using Partition = partition_scheduling_detail::Partition; // resolve ambiguity
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 58-60

```cpp
  58: template <typename... Args> bool node_isa(Node *node) {
  59:   return node->isOp() && isa<Args...>(node->getOp());
  60: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 62-65

```cpp
  62: std::unique_ptr<Graph> buildGraph(Operation *region) {
  63:   DenseMap<Operation *, Node *> nodes;
  64:   DenseMap<std::pair<Operation *, size_t>, InputPort> operands;
  65:   SmallVector<std::pair<OutputPort, Value>> values;
```

- **EN:** Defines helper `buildGraph` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `buildGraph`，用于计算或构造外围变换所需的中间数据。
### Lines 67-80

```cpp
  67:   std::function<void(Node * graph, Operation *)> visitOperation =
  68:       [&](Node *graph, Operation *op) {
  69:         if (auto funcOp = dyn_cast<FuncOp>(op)) {
  70:           auto node = graph->addNode(op, 0, 0);
  71:           nodes[op] = node;
  72:           for (size_t idx = 0; idx < funcOp.getNumArguments(); idx++) {
  73:             auto argNode = node->addNode(funcOp.getArgument(idx), 0, 1);
  74:             values.push_back(std::make_pair(OutputPort(argNode, 0),
  75:                                             funcOp.getArgument(idx)));
  76:           }
  77:           for (auto &region : op->getRegions())
  78:             for (auto &block : region)
  79:               for (auto &op : block)
  80:                 visitOperation(node, &op);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 82-84

```cpp
  82:         } else if (auto forOp = dyn_cast<scf::ForOp>(op)) {
  83:           auto node = graph->addNode(op, 3, 0);
  84:           nodes[op] = node;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 86-89

```cpp
  86:           // lb / ub / step
  87:           operands[std::make_pair(op, 0)] = InputPort(node, 0);
  88:           operands[std::make_pair(op, 1)] = InputPort(node, 1);
  89:           operands[std::make_pair(op, 2)] = InputPort(node, 2);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 91-105

```cpp
  91:           // iter args / results
  92:           auto ind_var = node->addNode(forOp.getInductionVar(), 0, 1);
  93:           node->addDefines(ind_var);
  94:           values.push_back(
  95:               std::make_pair(OutputPort(ind_var, 0), forOp.getInductionVar()));
  96:           size_t idx = 0;
  97:           for (auto iter_arg : forOp.getRegionIterArgs()) {
  98:             auto iter_arg_node = node->addNode(iter_arg, 2, 1);
  99:             node->addDefines(iter_arg_node);
 100:             values.push_back(
 101:                 std::make_pair(OutputPort(iter_arg_node, 0), iter_arg));
 102:             values.push_back(std::make_pair(OutputPort(iter_arg_node, 0),
 103:                                             forOp.getResult(idx)));
 104:             idx++;
 105:           }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 107-114

```cpp
 107:           // init iter args
 108:           {
 109:             for (size_t idx = 0; idx < forOp.getInitArgs().size(); ++idx) {
 110:               auto iter_arg_node = node->getDefines()[idx + 1];
 111:               operands[std::make_pair(op, idx + 3)] =
 112:                   InputPort(iter_arg_node, 0);
 113:             }
 114:           }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 116-119

```cpp
 116:           for (auto &region : op->getRegions())
 117:             for (auto &block : region)
 118:               for (auto &op : block)
 119:                 visitOperation(node, &op);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 121-123

```cpp
 121:         } else if (auto ifOp = dyn_cast<scf::IfOp>(op)) {
 122:           auto node = graph->addNode(op, 1, 0);
 123:           nodes[op] = node;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 125-126

```cpp
 125:           // cond
 126:           operands[std::make_pair(op, 0)] = InputPort(node, 0);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 128-134

```cpp
 128:           // results
 129:           for (auto result : ifOp.getResults()) {
 130:             auto result_node = node->addNode(result, 2, 1);
 131:             node->addDefines(result_node);
 132:             values.push_back(
 133:                 std::make_pair(OutputPort(result_node, 0), result));
 134:           }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 136-139

```cpp
 136:           for (auto &region : op->getRegions())
 137:             for (auto &block : region)
 138:               for (auto &op : block)
 139:                 visitOperation(node, &op);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 141-141

```cpp
 141:         } else if (auto reduceOp = dyn_cast<tt::ReduceOp>(op)) {
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 143-144

```cpp
 143:           auto node = graph->addNode(op, 1, 1);
 144:           nodes[op] = node;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 146-147

```cpp
 146:           // input
 147:           operands[std::make_pair(op, 0)] = InputPort(node, 0);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 149-152

```cpp
 149:           // result
 150:           assert(reduceOp.getResults().size() == 1);
 151:           auto result = reduceOp.getResults().front();
 152:           values.push_back(std::make_pair(OutputPort(node, 0), result));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 154-157

```cpp
 154:           for (auto &region : op->getRegions())
 155:             for (auto &block : region)
 156:               for (auto &op : block)
 157:                 visitOperation(node, &op);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 159-159

```cpp
 159:         } else if (isa<scf::YieldOp>(op)) {
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 161-168

```cpp
 161:           if (auto forOp = dyn_cast<scf::ForOp>(op->getParentOp())) {
 162:             // map operands to yield in a for op to the iter arg nodes
 163:             auto for_node = nodes[op->getParentOp()];
 164:             for (size_t idx = 0; idx < op->getNumOperands(); idx++) {
 165:               auto block_arg_node =
 166:                   for_node->getDefines()[idx + 1]; // skip iter arg
 167:               operands[std::make_pair(op, idx)] = InputPort(block_arg_node, 1);
 168:             }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 170-181

```cpp
 170:           } else if (auto ifOp = dyn_cast<scf::IfOp>(op->getParentOp())) {
 171:             // map operands to yield in an if op to the if results
 172:             auto if_node = nodes[op->getParentOp()];
 173:             for (size_t idx = 0; idx < op->getNumOperands(); idx++) {
 174:               auto result_node = if_node->getDefines()[idx];
 175:               operands[std::make_pair(op, idx)] = InputPort(
 176:                   result_node,
 177:                   (op->getParentRegion() == &ifOp.getThenRegion()) ? 0 : 1);
 178:             }
 179:           } else {
 180:             assert(false && "unsupported");
 181:           }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 183-184

```cpp
 183:         } else if (isa<tt::ReturnOp>(op)) {
 184:           // omit
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 186-196

```cpp
 186:         } else {
 187:           auto node =
 188:               graph->addNode(op, op->getNumOperands(), op->getNumResults());
 189:           nodes[op] = node;
 190:           for (size_t idx = 0; idx < op->getNumOperands(); idx++)
 191:             operands[std::make_pair(op, idx)] = InputPort(node, idx);
 192:           for (const auto &result : op->getResults())
 193:             values.push_back(std::make_pair(
 194:                 OutputPort(node, result.getResultNumber()), result));
 195:         }
 196:       };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 198-199

```cpp
 198:   auto graph = std::make_unique<Graph>(region);
 199:   visitOperation(graph->getRoot(), region);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 201-210

```cpp
 201:   for (auto [outputPort, value] : values) {
 202:     for (auto &use : value.getUses()) {
 203:       auto op = use.getOwner();
 204:       auto key = std::make_pair(op, use.getOperandNumber());
 205:       if (operands.find(key) != operands.end()) {
 206:         auto inputPort = operands[key];
 207:         Node::addEdge(outputPort, inputPort);
 208:       }
 209:     }
 210:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 212-213

```cpp
 212:   return graph;
 213: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 215-232

```cpp
 215: SmallVector<OutputPort> initialDataValues(Graph *graph) {
 216:   SmallVector<OutputPort> values;
 217:   graph->walk([&](Node *node) {
 218:     if (node->isOp()) {
 219:       auto op = node->getOp();
 220:       if (isa<tt::DescriptorLoadLikeOpInterface>(op)) {
 221:         node->setDataValue(0);
 222:         values.push_back({node, 0});
 223:       }
 224:       if (isa<ttng::TMEMLoadOp>(op)) {
 225:         node->setDataValue(0);
 226:         values.push_back({node, 0});
 227:         node->setDataValue(1);
 228:         values.push_back({node, 1});
 229:       }
 230:       if (isa<nvidia_gpu::TCGen5MMAOp, nvidia_gpu::TCGen5MMAScaledOp>(op)) {
 231:         node->setDataValue(0);
 232:         values.push_back({node, 0});
```

- **EN:** Defines `initialDataValues`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. The implementation traverses IR operations to collect facts or apply rewrites globally.
- **CN:** 这里定义 `initialDataValues`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。
### Lines 233-245

```cpp
 233:       }
 234:       // if it is manually tagged with data attribute,
 235:       // all outputs are treated as data values
 236:       if (op->hasAttr("data")) {
 237:         for (size_t i = 0; i < node->getNumOutputs(); i++) {
 238:           node->setDataValue(i);
 239:           values.push_back({node, i});
 240:         }
 241:       }
 242:     }
 243:   });
 244:   return values;
 245: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 247-250

```cpp
 247: void propagateDataValues(const SmallVector<OutputPort> &values) {
 248:   SmallVector<OutputPort> stack = values;
 249:   DenseSet<OutputPort> seen;
 250:   seen.insert(values.begin(), values.end());
```

- **EN:** Defines `propagateDataValues`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `propagateDataValues`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 252-258

```cpp
 252:   auto add = [&](OutputPort value) {
 253:     value.getNode()->setDataValue(value.getIdx());
 254:     if (seen.find(value) == seen.end()) {
 255:       stack.push_back(value);
 256:       seen.insert(value);
 257:     }
 258:   };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 260-271

```cpp
 260:   while (!stack.empty()) {
 261:     auto value = stack.back();
 262:     stack.pop_back();
 263:     for (auto use : value.getNode()->getOutputsFromPort(value.getIdx())) {
 264:       auto use_node = use.getNode();
 265:       for (size_t idx = 0; idx < use_node->getNumOutputs(); idx++) {
 266:         OutputPort new_value{use_node, idx};
 267:         add(new_value);
 268:       }
 269:     }
 270:   }
 271: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 273-280

```cpp
 273: void initialPartitionAssignment(Graph *graph) {
 274:   graph->walk([&](Node *node) {
 275:     if (node->isData() && !node->hasPartition()) {
 276:       auto partition = graph->addPartition();
 277:       node->setPartition(partition);
 278:     }
 279:   });
 280: }
```

- **EN:** Defines `initialPartitionAssignment`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. The implementation traverses IR operations to collect facts or apply rewrites globally.
- **CN:** 这里定义 `initialPartitionAssignment`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。
### Lines 282-292

```cpp
 282: SmallVector<Edge> getCrossingEdges(Graph *graph) {
 283:   SmallVector<Edge> edges;
 284:   for (auto &partition : graph->getPartitions())
 285:     for (auto node : partition->getNodes())
 286:       for (auto edge : node->getOutEdges()) {
 287:         if (!edge.crossesPartitions())
 288:           continue;
 289:         edges.push_back(edge);
 290:       }
 291:   return edges;
 292: }
```

- **EN:** Defines accessor/helper `getCrossingEdges` that exposes or updates operation state in a compact, reusable way. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义访问器/辅助函数 `getCrossingEdges`，以紧凑且可复用的方式读取或更新操作状态。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 294-303

```cpp
 294: SmallVector<Edge> getOutCrossingEdges(Partition *partition) {
 295:   SmallVector<Edge> edges;
 296:   for (auto node : partition->getNodes())
 297:     for (auto edge : node->getOutEdges()) {
 298:       if (!edge.crossesPartitions())
 299:         continue;
 300:       edges.push_back(edge);
 301:     }
 302:   return edges;
 303: }
```

- **EN:** Defines accessor/helper `getOutCrossingEdges` that exposes or updates operation state in a compact, reusable way. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义访问器/辅助函数 `getOutCrossingEdges`，以紧凑且可复用的方式读取或更新操作状态。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 305-322

```cpp
 305: void deserializeManualPartitions(Operation *region, Graph *graph) {
 306:   std::map<int, Partition *> manual_partitions;
 307:   graph->walk([&](Node *node) {
 308:     if (node->isOp()) {
 309:       auto op = node->getOp();
 310:       if (op->hasAttr(kPartitionAttrName)) {
 311:         auto partitionIds =
 312:             cast<DenseI32ArrayAttr>(op->getAttr(kPartitionAttrName))
 313:                 .asArrayRef();
 314:         for (auto id : partitionIds) {
 315:           if (manual_partitions.find(id) == manual_partitions.end()) {
 316:             auto partition = graph->addPartition();
 317:             partition->addFlag(Flags::MANUAL);
 318:             manual_partitions[id] = partition;
 319:             LLVM_DEBUG({
 320:               llvm::errs() << "deserialize manual partition:";
 321:               partition->dump();
 322:             });
```

- **EN:** Defines debug logging helpers and compile-time tags used when tracing this pass or utility at runtime.
- **CN:** 这里定义调试日志辅助宏和编译期标签，用于在运行时跟踪该 pass 或工具。
### Lines 323-329

```cpp
 323:           }
 324:           node->addPartition(manual_partitions[id]);
 325:         }
 326:       }
 327:     }
 328:   });
 329: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 331-335

```cpp
 331: bool isNone(Node *node) {
 332:   auto partition = node->getPartition();
 333:   auto flags = partition->getFlags();
 334:   return flags == Flags::NONE || flags == Flags::MANUAL;
 335: }
```

- **EN:** Defines `isNone`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `isNone`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 337-341

```cpp
 337: bool isOnlyNone(Node *node) {
 338:   auto partition = node->getPartition();
 339:   auto flags = partition->getFlags();
 340:   return flags == Flags::NONE;
 341: }
```

- **EN:** Defines `isOnlyNone`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `isOnlyNone`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 343-347

```cpp
 343: bool isView(Node *node) {
 344:   auto partition = node->getPartition();
 345:   auto flags = partition->getFlags();
 346:   return flags & Flags::VIEW;
 347: }
```

- **EN:** Defines `isView`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `isView`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 349-353

```cpp
 349: bool isManual(Node *node) {
 350:   auto partition = node->getPartition();
 351:   auto flags = partition->getFlags();
 352:   return flags & Flags::MANUAL;
 353: }
```

- **EN:** Defines `isManual`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `isManual`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 355-359

```cpp
 355: bool isLoad(Node *node) {
 356:   auto partition = node->getPartition();
 357:   auto flags = partition->getFlags();
 358:   return flags & Flags::LOAD;
 359: }
```

- **EN:** Defines `isLoad`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `isLoad`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 361-365

```cpp
 361: bool isStore(Node *node) {
 362:   auto partition = node->getPartition();
 363:   auto flags = partition->getFlags();
 364:   return flags & Flags::STORE;
 365: }
```

- **EN:** Defines `isStore`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `isStore`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 367-371

```cpp
 367: bool isMMA(Node *node) {
 368:   auto partition = node->getPartition();
 369:   auto flags = partition->getFlags();
 370:   return flags & Flags::MMA;
 371: }
```

- **EN:** Defines `isMMA`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `isMMA`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 373-377

```cpp
 373: bool isTMEM(Node *node) {
 374:   auto partition = node->getPartition();
 375:   auto flags = partition->getFlags();
 376:   return flags & Flags::TMEM;
 377: }
```

- **EN:** Defines `isTMEM`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `isTMEM`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 379-390

```cpp
 379: bool hasEligibleMemoryOps(Graph *graph) {
 380:   bool found = false;
 381:   graph->walk([&](Node *node) {
 382:     if (!node->isOp() || found)
 383:       return;
 384:     auto flags = getNodeFlags(node);
 385:     // We cannot handle none descriptor memory operations
 386:     if (flags & (Flags::LOAD | Flags::STORE))
 387:       found = true;
 388:   });
 389:   return found;
 390: }
```

- **EN:** Defines `hasEligibleMemoryOps`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. The implementation traverses IR operations to collect facts or apply rewrites globally.
- **CN:** 这里定义 `hasEligibleMemoryOps`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。
### Lines 392-396

```cpp
 392: bool isSFU(Node *node) {
 393:   auto partition = node->getPartition();
 394:   auto flags = partition->getFlags();
 395:   return flags & Flags::SFU;
 396: }
```

- **EN:** Defines `isSFU`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `isSFU`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 398-402

```cpp
 398: bool isCostlySFU(Node *node) {
 399:   auto partition = node->getPartition();
 400:   auto flags = partition->getFlags();
 401:   return (flags & Flags::SFU) && partition->getCost() > 256;
 402: }
```

- **EN:** Defines `isCostlySFU`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `isCostlySFU`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 404-411

```cpp
 404: bool isForIterArg(Node *node) {
 405:   if (node->isOp())
 406:     return false;
 407:   auto blockArg = dyn_cast<BlockArgument>(node->getValue());
 408:   if (!blockArg)
 409:     return false;
 410:   return isa<scf::ForOp>(blockArg.getOwner()->getParentOp());
 411: }
```

- **EN:** Defines `isForIterArg`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `isForIterArg`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 413-420

```cpp
 413: bool isIfResult(Node *node) {
 414:   if (node->isOp())
 415:     return false;
 416:   auto result = dyn_cast<OpResult>(node->getValue());
 417:   if (!result)
 418:     return false;
 419:   return isa<scf::IfOp>(result.getOwner());
 420: }
```

- **EN:** Defines `isIfResult`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `isIfResult`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 422-428

```cpp
 422: SmallVector<std::pair<std::string, std::function<bool(Edge)>>> heuristics = {
 423:     // load followed by local alloc in same partition
 424:     {"load_local_alloc",
 425:      [](Edge edge) {
 426:        if (!node_isa<ttg::LocalAllocOp>(edge.getToNode())) {
 427:          return false;
 428:        }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 430-435

```cpp
 430:        if (node_isa<tt::DescriptorLoadLikeOpInterface>(edge.getFromNode())) {
 431:          // require layouts to match for TMA load + alloc
 432:          auto load = edge.getFromNode()->getOp();
 433:          auto alloc = cast<ttg::LocalAllocOp>(edge.getToNode()->getOp());
 434:          return getSharedEncoding(load) == alloc.getType().getEncoding();
 435:        }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 437-439

```cpp
 437:        if (node_isa<tt::LoadOp>(edge.getFromNode())) {
 438:          return true;
 439:        }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 441-442

```cpp
 441:        return false;
 442:      }},
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 444-452

```cpp
 444:     // sequence of view ops in same partition
 445:     // Note: view ops guaranteed to have been duplicated so there
 446:     // is one use/def for each
 447:     {"view_sequence",
 448:      [](Edge edge) {
 449:        auto from = getNodeFlags(edge.getFromNode());
 450:        auto to = getNodeFlags(edge.getToNode());
 451:        return (from & Flags::VIEW) && (to & Flags::VIEW);
 452:      }},
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 454-464

```cpp
 454:     // merge view op partition with producer if it involves fewer
 455:     // elements than merging with the consumer of the view partition
 456:     {"view_producer",
 457:      [](Edge edge) {
 458:        if (!isView(edge.getToNode())) {
 459:          return false;
 460:        }
 461:        auto to = getNodeFlags(edge.getToNode());
 462:        if (!(to & Flags::VIEW)) {
 463:          return false;
 464:        }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 466-469

```cpp
 466:        auto view_partition = edge.getToNode()->getPartition();
 467:        auto out_edges = getOutCrossingEdges(view_partition);
 468:        assert(out_edges.size() == 1);
 469:        auto out_edge = out_edges[0];
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 471-472

```cpp
 471:        auto in_size = edge.getSize();
 472:        auto out_size = out_edge.getSize();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 474-475

```cpp
 474:        return in_size > out_size;
 475:      }},
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 477-489

```cpp
 477:     // merge remaining view op partitions with consumer
 478:     // as that involves fewer elements being communicated via aref
 479:     {"view_consumer",
 480:      [](Edge edge) {
 481:        if (!isView(edge.getFromNode())) {
 482:          return false;
 483:        }
 484:        auto from = getNodeFlags(edge.getFromNode());
 485:        if (!(from & Flags::VIEW)) {
 486:          return false;
 487:        }
 488:        return true;
 489:      }},
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 491-507

```cpp
 491:     // for op iter arg placed in same partition as op that produces
 492:     // its value in the loop body (if it is not a token)
 493:     {"for_op_iter_arg",
 494:      [](Edge edge) {
 495:        auto from = edge.getFromNode();
 496:        auto to = edge.getToNode();
 497:        if (from->getParent() != to->getParent())
 498:          // skip if not both in the loop body
 499:          return false;
 500:        if (!isForIterArg(to))
 501:          // skip is not to an iter arg
 502:          return false;
 503:        if (isa<AsyncTokenType>(to->getValue().getType()))
 504:          // skip if a token type
 505:          return false;
 506:        return true;
 507:      }},
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 509-521

```cpp
 509:     // for op iter arg placed in same partition as op that consumes
 510:     // its value (if it is a token)
 511:     {"for_op_iter_arg_token",
 512:      [](Edge edge) {
 513:        auto from = edge.getFromNode();
 514:        if (!isForIterArg(from))
 515:          // skip if not from an iter arg
 516:          return false;
 517:        if (!isa<AsyncTokenType>(from->getValue().getType()))
 518:          // skip if not a token
 519:          return false;
 520:        return true;
 521:      }},
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 523-539

```cpp
 523:     // if op result placed in same partition as MMA op that produces it (if it
 524:     // is a token)
 525:     {"if_op_result_token",
 526:      [](Edge edge) {
 527:        auto from = edge.getFromNode();
 528:        auto to = edge.getToNode();
 529:        if (!isMMA(from)) {
 530:          // skip if not from an MMA
 531:        }
 532:        if (!isIfResult(to))
 533:          // skip if not to an if op result
 534:          return false;
 535:        if (!isa<AsyncTokenType>(to->getValue().getType()))
 536:          // skip if not a token
 537:          return false;
 538:        return true;
 539:      }},
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 541-548

```cpp
 541:     // merge expensive SFU ops with their dependencies (except MMA, STORE and
 542:     // other SFU)
 543:     {"sfu_consumer",
 544:      [](Edge edge) {
 545:        auto from = edge.getFromNode();
 546:        auto to = edge.getToNode();
 547:        return isCostlySFU(to) && !isMMA(from) && !isLoad(from) && !isSFU(from);
 548:      }},
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 550-558

```cpp
 550:     // straight sequence of NONE ops merges together
 551:     {"sequence",
 552:      [](Edge edge) {
 553:        auto from = edge.getFromNode();
 554:        auto to = edge.getToNode();
 555:        if (from->getNumOutDataEdges() > 1 || to->getNumInDataEdges() > 1)
 556:          return false;
 557:        return isNone(from) && isNone(to);
 558:      }},
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 560-568

```cpp
 560:     // straight sequence of NONE op to SFU op merges together
 561:     {"sequence_sfu",
 562:      [](Edge edge) {
 563:        auto from = edge.getFromNode();
 564:        auto to = edge.getToNode();
 565:        if (from->getNumOutDataEdges() > 1 || to->getNumInDataEdges() > 1)
 566:          return false;
 567:        return isNone(from) && isSFU(to);
 568:      }},
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 570-576

```cpp
 570:     // TMEM load merges with consumer
 571:     // FIXME: limit to single consumer?
 572:     {"tmem_load",
 573:      [](Edge edge) {
 574:        auto from = edge.getFromNode();
 575:        return node_isa<ttng::TMEMLoadOp>(from);
 576:      }},
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 578-584

```cpp
 578:     // TMEM and STORE groups merge
 579:     {"tmem_store",
 580:      [](Edge edge) {
 581:        auto from = edge.getFromNode();
 582:        auto to = edge.getToNode();
 583:        return isTMEM(from) && isStore(to);
 584:      }},
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 586-593

```cpp
 586:     // NONE/cheap SFU merges with consumer (except LOAD, MMA or costly SFU)
 587:     {"none_consumer",
 588:      [](Edge edge) {
 589:        auto from = edge.getFromNode();
 590:        auto to = edge.getToNode();
 591:        return (isNone(from) || (isSFU(from) && !isCostlySFU(from))) &&
 592:               !isNone(to) && !isMMA(to) && !isLoad(to) && !isCostlySFU(to);
 593:      }},
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 595-605

```cpp
 595:     // NONE merges with costly producer (except LOAD or MMA)
 596:     // This will prefer to merge NONE nodes into costly groups, rather than
 597:     // non-costly groups
 598:     // e.g. in the two SFU groups of attention kernels
 599:     {"none_producer_costly",
 600:      [](Edge edge) {
 601:        auto from = edge.getFromNode();
 602:        auto to = edge.getToNode();
 603:        return isNone(to) && !isNone(from) && !isMMA(from) && !isLoad(from) &&
 604:               from->getPartition()->getCost() > 256;
 605:      }},
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 607-613

```cpp
 607:     // NONE merges with producer (except LOAD or MMA)
 608:     {"none_producer",
 609:      [](Edge edge) {
 610:        auto from = edge.getFromNode();
 611:        auto to = edge.getToNode();
 612:        return isNone(to) && !isNone(from) && !isMMA(from) && !isLoad(from);
 613:      }},
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 615-623

```cpp
 615:     // merge connected STORE partitions together
 616:     // these are both using tt.descriptor_store and have a dataflow edge
 617:     // between, so avoid communicating between partitions via aref
 618:     {"connected_store",
 619:      [](Edge edge) {
 620:        auto from = edge.getFromNode();
 621:        auto to = edge.getToNode();
 622:        return isStore(from) && isStore(to);
 623:      }},
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 625-631

```cpp
 625:     // merge connected NONE partitions together
 626:     {"connected_none",
 627:      [](Edge edge) {
 628:        auto from = edge.getFromNode();
 629:        auto to = edge.getToNode();
 630:        return isOnlyNone(from) && isOnlyNone(to);
 631:      }},
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 633-640

```cpp
 633:     // merge connected NONE and MANUAL partitions together
 634:     {"connected_none_manual",
 635:      [](Edge edge) {
 636:        auto from = edge.getFromNode();
 637:        auto to = edge.getToNode();
 638:        return (isOnlyNone(from) && isManual(to)) ||
 639:               (isOnlyNone(to) && isManual(from));
 640:      }},
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 642-651

```cpp
 642:     // merge connected partitions together if edge between is expensive
 643:     // TODO: this might be better expressed as a horizontal rule,
 644:     // that aims to keep shmem usage under the limit
 645:     {"connected",
 646:      [](Edge edge) {
 647:        auto from = edge.getFromNode();
 648:        auto to = edge.getToNode();
 649:        return !isLoad(from) && !isLoad(to) && !isMMA(from) && !isMMA(to) &&
 650:               edge.getSize() > 16384; // FIXME: seemingly arbitrary size...
 651:      }},
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 653-658

```cpp
 653:     // store group not used by an mma/dot op should be merged
 654:     {"load_epilog",
 655:      [](Edge edge) {
 656:        auto from = edge.getFromNode();
 657:        if (!isLoad(from))
 658:          return false;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 660-663

```cpp
 660:        SmallVector<Node *> stack;
 661:        DenseSet<Node *> seen;
 662:        stack.push_back(from);
 663:        seen.insert(from);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 665-678

```cpp
 665:        while (!stack.empty()) {
 666:          auto node = stack.back();
 667:          stack.pop_back();
 668:          if (isMMA(node) || (node->isOp() && isa<tt::DotOp>(node->getOp()))) {
 669:            return false;
 670:          } else {
 671:            for (auto edge : node->getOutEdges()) {
 672:              if (!seen.contains(edge.getToNode())) {
 673:                stack.push_back(edge.getToNode());
 674:                seen.insert(edge.getToNode());
 675:              }
 676:            }
 677:          }
 678:        }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 680-682

```cpp
 680:        return true;
 681:      }},
 682: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 684-691

```cpp
 684: SmallVector<std::pair<std::string, std::function<bool(Edge)>>> constraints = {
 685:     // don't merge manual partitions
 686:     {"manual",
 687:      [](Edge edge) {
 688:        auto from = edge.getFromNode();
 689:        auto to = edge.getToNode();
 690:        return !(isManual(from) && isManual(to));
 691:      }},
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 693-699

```cpp
 693:     // don't merge partitions with tmem ops into mma partitions
 694:     {"tmem_mma",
 695:      [](Edge edge) {
 696:        auto from = edge.getFromNode();
 697:        auto to = edge.getToNode();
 698:        return !((isMMA(from) && isTMEM(to)) || (isMMA(to) && isTMEM(from)));
 699:      }},
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 701-708

```cpp
 701:     // don't merge tmem alloc (non-token form) into mma partition
 702:     {"tmem_alloc",
 703:      [](Edge edge) {
 704:        auto from = edge.getFromNode();
 705:        auto to = edge.getToNode();
 706:        return !(node_isa<ttng::TMEMAllocOp>(from) && isMMA(to));
 707:      }},
 708: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 710-727

```cpp
 710: DenseSet<Operation *> getTMEMAllocs(Partition *partition) {
 711:   // look for all tmem allocs used by the partition
 712:   DenseSet<Operation *> result;
 713:   for (auto node : partition->getNodes()) {
 714:     if (!node->isOp())
 715:       continue;
 716:     Operation *alloc = nullptr;
 717:     if (auto load = dyn_cast<ttng::TMEMLoadOp>(node->getOp())) {
 718:       alloc = load.getOperand(0).getDefiningOp();
 719:     }
 720:     if (auto store = dyn_cast<ttng::TMEMStoreOp>(node->getOp())) {
 721:       alloc = store.getOperand(0).getDefiningOp();
 722:     }
 723:     if (alloc) {
 724:       assert(isa<ttng::TMEMAllocOp>(alloc));
 725:       result.insert(alloc);
 726:     }
 727:   }
```

- **EN:** Defines accessor/helper `getTMEMAllocs` that exposes or updates operation state in a compact, reusable way. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义访问器/辅助函数 `getTMEMAllocs`，以紧凑且可复用的方式读取或更新操作状态。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 728-729

```cpp
 728:   return result;
 729: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 731-740

```cpp
 731: SmallVector<
 732:     std::pair<std::string, std::function<bool(Partition *, Partition *)>>>
 733:     partition_heuristics = {
 734:         // merge mma partitions
 735:         {"mma",
 736:          [](Partition *a, Partition *b) {
 737:            auto a_is_mma = (a->getFlags() == Flags::MMA);
 738:            auto b_is_mma = (b->getFlags() == Flags::MMA);
 739:            return a_is_mma && b_is_mma;
 740:          }},
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 742-748

```cpp
 742:         // merge load partitions
 743:         {"load",
 744:          [](Partition *a, Partition *b) {
 745:            auto a_is_load = (a->getFlags() == Flags::LOAD);
 746:            auto b_is_load = (b->getFlags() == Flags::LOAD);
 747:            return a_is_load && b_is_load;
 748:          }},
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 750-758

```cpp
 750:         // merge none with store partitions
 751:         {"none",
 752:          [](Partition *a, Partition *b) {
 753:            auto a_is_none = (a->getFlags() == Flags::NONE);
 754:            auto b_is_none = (b->getFlags() == Flags::NONE);
 755:            auto a_is_store = (a->getFlags() & Flags::STORE);
 756:            auto b_is_store = (b->getFlags() & Flags::STORE);
 757:            return (a_is_none && b_is_store) || (a_is_store && b_is_none);
 758:          }},
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 760-776

```cpp
 760:         // merge TMEM partitions together, if they use the same tmem alloc
 761:         // aref does not support tmem with more than 2 partitions
 762:         // and the tmem_alloc'd memory can maximally be used by an MMA
 763:         // partition and a TMEM partition
 764:         {"tmem",
 765:          [](Partition *a, Partition *b) {
 766:            auto a_is_tmem = (a->getFlags() & Flags::TMEM);
 767:            auto b_is_tmem = (b->getFlags() & Flags::TMEM);
 768:            if (!a_is_tmem || !b_is_tmem)
 769:              return false;
 770:            auto allocs_a = getTMEMAllocs(a);
 771:            auto allocs_b = getTMEMAllocs(b);
 772:            // if the sets are overlapping, alloc is used by both TMEM partitions
 773:            for (auto alloc_a : allocs_a)
 774:              if (allocs_b.contains(alloc_a))
 775:                return true;
 776:            return false;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 777-778

```cpp
 777:          }},
 778: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 780-782

```cpp
 780: void mergePartitions(Graph *graph, std::string funcName,
 781:                      VisualizationInfo &vis_info) {
 782:   LLVM_DEBUG({ llvm::errs() << "#### applying heuristics...\n"; });
```

- **EN:** Defines debug logging helpers and compile-time tags used when tracing this pass or utility at runtime.
- **CN:** 这里定义调试日志辅助宏和编译期标签，用于在运行时跟踪该 pass 或工具。
### Lines 784-792

```cpp
 784:   // initial worklist is list of all edges that cross partitions
 785:   auto crossingEdges = getCrossingEdges(graph);
 786:   bool changed = false;
 787:   do {
 788:     changed = false;
 789:     LLVM_DEBUG({
 790:       llvm::errs() << "\n"
 791:                    << crossingEdges.size() << " crossing edges remaining\n";
 792:     });
```

- **EN:** Defines debug logging helpers and compile-time tags used when tracing this pass or utility at runtime.
- **CN:** 这里定义调试日志辅助宏和编译期标签，用于在运行时跟踪该 pass 或工具。
### Lines 794-796

```cpp
 794:     for (auto [name, apply] : heuristics) {
 795:       for (auto it = crossingEdges.begin(); it != crossingEdges.end();) {
 796:         auto edge = *it;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 798-802

```cpp
 798:         // remove edges that no longer cross partitions from the worklist
 799:         if (!edge.crossesPartitions()) {
 800:           it = crossingEdges.erase(it);
 801:           continue;
 802:         }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 804-816

```cpp
 804:         if (apply(edge)) {
 805:           // check if applying the heuristic will satisfy the constraints
 806:           bool ok = true;
 807:           for (auto [name, constraint] : constraints) {
 808:             if (!constraint(edge)) {
 809:               ok = false;
 810:               break;
 811:             }
 812:           }
 813:           if (!ok) {
 814:             it++;
 815:             continue;
 816:           }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 818-829

```cpp
 818:           LLVM_DEBUG({
 819:             llvm::dbgs() << "\napply heuristic \"" << name << "\"\n";
 820:             llvm::dbgs() << edge.getFromNode()->getLabel() << " -> "
 821:                          << edge.getToNode()->getLabel() << "\n";
 822:             llvm::dbgs() << "partitions " << edge.getFromNode()->getPartition()
 823:                          << " -> " << edge.getToNode()->getPartition() << "\n";
 824:             llvm::dbgs() << "flags "
 825:                          << edge.getFromNode()->getPartition()->getFlags()
 826:                          << " -> "
 827:                          << edge.getToNode()->getPartition()->getFlags()
 828:                          << "\n";
 829:           });
```

- **EN:** Defines debug logging helpers and compile-time tags used when tracing this pass or utility at runtime.
- **CN:** 这里定义调试日志辅助宏和编译期标签，用于在运行时跟踪该 pass 或工具。
### Lines 831-834

```cpp
 831:           // merge the partitions
 832:           auto from_partition = edge.getFromNode()->getPartition();
 833:           auto to_partition = edge.getToNode()->getPartition();
 834:           Partition::merge(from_partition, to_partition);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 836-838

```cpp
 836:           visualize(funcName, "merge-step", std::string("merge: rule ") + name,
 837:                     graph, vis_info);
 838:           crossingEdges.erase(it);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 840-842

```cpp
 840:           changed = true;
 841:           break;
 842:         }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 844-849

```cpp
 844:         it++;
 845:       }
 846:       if (changed)
 847:         break;
 848:     }
 849:   } while (changed);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 851-852

```cpp
 851:   visualize(funcName, "merge-step", "edge based merge complete", graph,
 852:             vis_info);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 854-871

```cpp
 854:   {
 855:     // look at every pair of partitions and check if they should be merged
 856:     auto merge_partitions_step = [&]() {
 857:       SmallVector<Partition *> all_partitions;
 858:       for (auto partition : graph->getPartitions())
 859:         all_partitions.push_back(partition);
 860:       for (auto [name, apply] : partition_heuristics) {
 861:         for (auto partitionA : all_partitions) {
 862:           for (auto partitionB : all_partitions) {
 863:             if (partitionA == partitionB)
 864:               continue;
 865:             if (apply(partitionA, partitionB)) {
 866:               LLVM_DEBUG({
 867:                 llvm::errs() << "\nmerge \"" << name << "\" ----\n";
 868:                 partitionA->dump();
 869:                 partitionB->dump();
 870:               });
 871:               Partition::merge(partitionA, partitionB);
```

- **EN:** Defines debug logging helpers and compile-time tags used when tracing this pass or utility at runtime.
- **CN:** 这里定义调试日志辅助宏和编译期标签，用于在运行时跟踪该 pass 或工具。
### Lines 872-880

```cpp
 872:               visualize(funcName, "merge-step",
 873:                         std::string("merge: rule ") + name, graph, vis_info);
 874:               return false;
 875:             }
 876:           }
 877:         }
 878:       }
 879:       return true;
 880:     };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 882-886

```cpp
 882:     while (true) {
 883:       if (merge_partitions_step())
 884:         break;
 885:     }
 886:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 888-889

```cpp
 888:   visualize(funcName, "merge-step", "partition based merge complete", graph,
 889:             vis_info);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 891-892

```cpp
 891:   LLVM_DEBUG({ llvm::errs() << "\n#### heuristics done\n"; });
 892: }
```

- **EN:** Defines debug logging helpers and compile-time tags used when tracing this pass or utility at runtime.
- **CN:** 这里定义调试日志辅助宏和编译期标签，用于在运行时跟踪该 pass 或工具。
### Lines 894-896

```cpp
 894: void propagatePartitions(Graph *graph, std::string funcName,
 895:                          VisualizationInfo &vis_info) {
 896:   visualize(funcName, "propagate", "before propagate", graph, vis_info);
```

- **EN:** Defines `propagatePartitions`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `propagatePartitions`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 898-899

```cpp
 898:   // propagate partitions to parent ops
 899:   SmallVector<Node *> leaves;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 901-913

```cpp
 901:   graph->walk([&](Node *node) {
 902:     // node is a leaf if it has a region,
 903:     // and none of the ops in the region are leaves
 904:     bool is_leaf = !node->getNodes().empty();
 905:     for (auto &child : node->getNodes()) {
 906:       if (!child->getNodes().empty()) {
 907:         is_leaf = false;
 908:         break;
 909:       }
 910:     }
 911:     if (is_leaf)
 912:       leaves.push_back(node);
 913:   });
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. The implementation traverses IR operations to collect facts or apply rewrites globally.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。
### Lines 915-924

```cpp
 915:   bool changed = true;
 916:   while (changed) {
 917:     for (auto leaf : leaves) {
 918:       // partitions for leaf are union of partitions of all ops contained in
 919:       // the leaf
 920:       SetVector<Partition *> partitions;
 921:       for (auto &node : leaf->getNodes())
 922:         partitions.insert(node->getPartitions().begin(),
 923:                           node->getPartitions().end());
 924:       leaf->addPartitions(partitions);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 926-936

```cpp
 926:       // propagate to parent nodes
 927:       auto node = leaf->getParent();
 928:       while (node) {
 929:         // include union of partitions of ops in the parent
 930:         for (auto &child : node->getNodes())
 931:           partitions.insert(child->getPartitions().begin(),
 932:                             child->getPartitions().end());
 933:         node->addPartitions(partitions);
 934:         node = node->getParent();
 935:       }
 936:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 938-950

```cpp
 938:     // propagate partitions to non-data nodes
 939:     {
 940:       SmallVector<Node *> nodes;
 941:       // include nodes with regions
 942:       graph->walk([&](Node *node) {
 943:         if (!node->getNodes().empty())
 944:           nodes.push_back(node);
 945:       });
 946:       // include data nodes
 947:       for (auto &partition : graph->getPartitions())
 948:         for (auto &node : partition->getNodes())
 949:           if (node->isData())
 950:             nodes.push_back(node);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. The implementation traverses IR operations to collect facts or apply rewrites globally. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 952-958

```cpp
 952:       changed = false;
 953:       for (auto node : nodes) {
 954:         SmallVector<Node *> stack;
 955:         DenseSet<Node *> seen;
 956:         auto partitions = node->getPartitions();
 957:         stack.push_back(node);
 958:         seen.insert(node);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 960-962

```cpp
 960:         while (!stack.empty()) {
 961:           auto node = stack.back();
 962:           stack.pop_back();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 964-975

```cpp
 964:           auto propagate = [&](Edge edge, Node *node) {
 965:             if (!node || node->isData())
 966:               return;
 967:             auto numPartitionsBefore = node->getPartitions().size();
 968:             node->addPartitions(partitions);
 969:             auto numPartitionsAfter = node->getPartitions().size();
 970:             changed |= (numPartitionsBefore != numPartitionsAfter);
 971:             if (seen.count(node) == 0) {
 972:               stack.push_back(node);
 973:               seen.insert(node);
 974:             }
 975:           };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 977-982

```cpp
 977:           for (auto edge : node->getInEdges())
 978:             propagate(edge, edge.getFromNode());
 979:         }
 980:       }
 981:     }
 982:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 984-984

```cpp
 984:   visualize(funcName, "propagate", "after propagate", graph, vis_info);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 986-993

```cpp
 986:   // propagate partitions to non-data nodes (forward)
 987:   {
 988:     SmallVector<Node *> nodes;
 989:     // get nodes that have no partition assigned
 990:     graph->walk([&](Node *node) {
 991:       if (!node->hasPartition())
 992:         nodes.push_back(node);
 993:     });
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. The implementation traverses IR operations to collect facts or apply rewrites globally.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。
### Lines 995-1010

```cpp
 995:     changed = false;
 996:     while (!nodes.empty()) {
 997:       // try propagating partitions forward to nodes with no partition
 998:       int start_size = nodes.size();
 999:       for (auto node : nodes) {
1000:         for (auto edge : node->getInEdges()) {
1001:           if (!edge.getFromNode())
1002:             continue;
1003:           if (edge.getFromNode()->hasPartition()) {
1004:             for (auto partition : edge.getFromNode()->getPartitions())
1005:               node->setPartition(partition);
1006:             changed = true;
1007:           }
1008:         }
1009:       }
1010:       // remove all nodes that now have a partition
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1011-1021

```cpp
1011:       nodes.erase(
1012:           std::remove_if(nodes.begin(), nodes.end(),
1013:                          [](Node *node) { return node->hasPartition(); }),
1014:           nodes.end());
1015:       int end_size = nodes.size();
1016:       if (start_size == end_size) {
1017:         // no change -> exit
1018:         break;
1019:       }
1020:     }
1021:   }
```

- **EN:** Defines `std::remove_if`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `std::remove_if`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1023-1023

```cpp
1023:   visualize(funcName, "propagate", "propagate forward", graph, vis_info);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1025-1032

```cpp
1025:   // propagate partitions of tt.reduce into its body
1026:   graph->walk([&](Node *node) {
1027:     if (node->isOp() && isa<tt::ReduceOp>(node->getOp())) {
1028:       auto partitions = node->getPartitions();
1029:       node->walk(
1030:           [&](Node *child_node) { child_node->addPartitions(partitions); });
1031:     }
1032:   });
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. The implementation traverses IR operations to collect facts or apply rewrites globally.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。
### Lines 1034-1034

```cpp
1034:   visualize(funcName, "propagate", "propagate reduce", graph, vis_info);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1036-1041

```cpp
1036:   // Corner case: tmem store following tmem alloc should be in a warp
1037:   // partition with 4 warps (i.e. a non-mma partition)
1038:   // This fixes the case where in a tmem alloc + initial store that feeds into
1039:   // an mma, the store is propagated the partition of the mma. It should instead
1040:   // have the same partition as the alloc
1041:   SmallVector<Node *> patched_nodes;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1043-1047

```cpp
1043:   graph->walk([&](Node *node) {
1044:     if (node->isData() || !node->isOp() ||
1045:         !isa<ttng::TMEMStoreOp>(node->getOp())) {
1046:       return;
1047:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. The implementation traverses IR operations to collect facts or apply rewrites globally.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。
### Lines 1049-1057

```cpp
1049:     Node *alloc = nullptr;
1050:     for (auto edge : node->getInEdges()) {
1051:       if (edge.getToIdx() == 1) { // token edge
1052:         alloc = edge.getFromNode();
1053:         break;
1054:       }
1055:     }
1056:     if (!alloc || !alloc->isOp() || !isa<ttng::TMEMAllocOp>(alloc->getOp()))
1057:       return;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1059-1069

```cpp
1059:     // pick the first non-mma partition
1060:     // does nothing if the only partitions are mma
1061:     auto partitions = alloc->getPartitions();
1062:     for (auto partition : partitions) {
1063:       if (partition->getFlags() & MMA)
1064:         continue;
1065:       node->setPartition(partition);
1066:       patched_nodes.push_back(node);
1067:       break;
1068:     }
1069:   });
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1071-1071

```cpp
1071:   visualize(funcName, "propagate", "tmem store corner case", graph, vis_info);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1073-1079

```cpp
1073:   // propagate partitions for patched up nodes to non-data nodes
1074:   for (auto node : patched_nodes) {
1075:     SmallVector<Node *> stack;
1076:     DenseSet<Node *> seen;
1077:     auto partitions = node->getPartitions();
1078:     stack.push_back(node);
1079:     seen.insert(node);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 1081-1083

```cpp
1081:     while (!stack.empty()) {
1082:       auto node = stack.back();
1083:       stack.pop_back();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1085-1091

```cpp
1085:       for (auto edge : node->getInEdges()) {
1086:         if (edge.isDataValue())
1087:           continue;
1088:         auto fromNode = edge.getFromNode();
1089:         if (!fromNode)
1090:           continue;
1091:         fromNode->addPartitions(partitions);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1093-1100

```cpp
1093:         if (seen.count(edge.getFromNode()) == 0) {
1094:           stack.push_back(fromNode);
1095:           seen.insert(fromNode);
1096:         }
1097:       }
1098:     }
1099:   }
1100: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1102-1105

```cpp
1102: void duplicateCheapOps(Graph *graph, std::string funcName,
1103:                        VisualizationInfo &vis_info) {
1104:   visualize(funcName, "duplicate", "before duplicate cheap ops", graph,
1105:             vis_info);
```

- **EN:** Defines `duplicateCheapOps`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `duplicateCheapOps`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1107-1111

```cpp
1107:   // for each partition:
1108:   // look at all crossing edges leaving the partition
1109:   // do a depth first search through NONE nodes, if we hit the same partition
1110:   // assign all nodes on that path to the partition
1111:   for (auto partition : graph->getPartitions()) {
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1113-1113

```cpp
1113:     auto crossingEdges = getOutCrossingEdges(partition);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1115-1119

```cpp
1115:     for (auto edge : crossingEdges) {
1116:       // only handle start nodes with a single partition
1117:       if (edge.getFromNode()->getPartitions().size() != 1)
1118:         continue;
1119:       auto startPartition = edge.getFromNode()->getPartition();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1121-1125

```cpp
1121:       // only handle nodes with a single partition
1122:       auto start = edge.getToNode();
1123:       if (start->getPartitions().size() != 1)
1124:         continue;
1125:       auto partition = start->getPartition();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1127-1130

```cpp
1127:       auto isCandidate = [](Node *node) {
1128:         return (getNodeFlags(node) == Flags::NONE ||
1129:                 getNodeFlags(node) == Flags::SFU);
1130:       };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1132-1133

```cpp
1132:       if (!isCandidate(edge.getToNode()))
1133:         continue;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1135-1136

```cpp
1135:       auto update = [&]() {
1136:         std::map<Node *, Node *> parentMap;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1138-1140

```cpp
1138:         SmallVector<Node *> stack;
1139:         stack.push_back(start);
1140:         DenseSet<Node *> seen;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1142-1159

```cpp
1142:         while (!stack.empty()) {
1143:           auto node = stack.back();
1144:           stack.pop_back();
1145:           if (!seen.contains(node)) {
1146:             seen.insert(node);
1147:             for (auto edge : node->getOutEdges()) {
1148:               auto child = edge.getToNode();
1149:               if (!seen.contains(child)) {
1150:                 if (child->getPartitions().size() != 1 || !isCandidate(child)) {
1151:                   // do nothing
1152:                 } else if (child->getPartition() == partition) {
1153:                   parentMap.emplace(child, node);
1154:                   stack.push_back(child);
1155:                 } else if (child->getPartition() == startPartition) {
1156:                   // found a path, set all nodes on the path to the partition
1157:                   node->addPartition(startPartition);
1158:                   while (parentMap.find(node) != parentMap.end()) {
1159:                     node = parentMap[node];
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1160-1161

```cpp
1160:                     node->addPartition(startPartition);
1161:                   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1163-1164

```cpp
1163:                   visualize(funcName, "duplicate", "duplicate cheap ops", graph,
1164:                             vis_info);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1166-1175

```cpp
1166:                   return;
1167:                 }
1168:               }
1169:             }
1170:           }
1171:         }
1172:       };
1173:       update();
1174:     }
1175:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1177-1178

```cpp
1177:   visualize(funcName, "duplicate", "duplicate cheap ops done", graph, vis_info);
1178: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1180-1180

```cpp
1180: void serialize(size_t idx, Operation *region, Graph *graph) {
```

- **EN:** Defines `serialize`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `serialize`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1182-1182

```cpp
1182:   SetVector<Operation *> alreadyWritten;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1184-1185

```cpp
1184:   auto context = graph->getRoot()->getOp()->getContext();
1185:   Builder b(context);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1187-1188

```cpp
1187:   // annotate loop with index
1188:   region->setAttr(kWarpSpecializeTagAttrName, b.getI32IntegerAttr(idx));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1190-1193

```cpp
1190:   auto setPartitionsAttr = [&](Operation *op, Node *node) {
1191:     // not for func op
1192:     if (isa<tt::FuncOp>(op))
1193:       return;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1195-1209

```cpp
1195:     // Note: we may have multiple nodes per op, so we merge the partition
1196:     // ids for all nodes of the op
1197:     SetVector<int> partitionIds;
1198:     if (alreadyWritten.contains(op)) {
1199:       // if we already serialized a node to this op, merge those partition ids
1200:       // with the node being serialized
1201:       partitionIds = getPartitionIds(op);
1202:     }
1203:     alreadyWritten.insert(op);
1204:     for (auto partition : node->getPartitions())
1205:       partitionIds.insert(*partition->id);
1206:     auto partitionIdsList = partitionIds.takeVector();
1207:     std::sort(partitionIdsList.begin(), partitionIdsList.end());
1208:     auto partitionsAttr = b.getDenseI32ArrayAttr(partitionIdsList);
1209:     op->setAttr(kPartitionAttrName, partitionsAttr);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1211-1221

```cpp
1211:     // set same paritions in yield ops
1212:     if (auto forOp = dyn_cast<scf::ForOp>(op)) {
1213:       cast<scf::YieldOp>(forOp.getBody()->getTerminator())
1214:           ->setAttr(kPartitionAttrName, partitionsAttr);
1215:     } else if (auto ifOp = dyn_cast<scf::IfOp>(op)) {
1216:       ifOp.thenYield()->setAttr(kPartitionAttrName, partitionsAttr);
1217:       if (!ifOp.getElseRegion().empty()) {
1218:         ifOp.elseYield()->setAttr(kPartitionAttrName, partitionsAttr);
1219:       }
1220:     }
1221:   };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1223-1237

```cpp
1223:   auto setPartitionOutputsAttr = [&](Operation *op, size_t idx, size_t size,
1224:                                      Node *node) {
1225:     llvm::SmallVector<Attribute> partitionAttrs;
1226:     if (op->hasAttr(kPartitionOutputsAttrName)) {
1227:       // get existing partitions
1228:       for (auto attr :
1229:            op->getAttrOfType<ArrayAttr>(kPartitionOutputsAttrName)) {
1230:         partitionAttrs.push_back(attr);
1231:       }
1232:       assert(partitionAttrs.size() == size);
1233:     } else {
1234:       // initialize to no partitions
1235:       for (size_t i = 0; i < size; i++)
1236:         partitionAttrs.push_back(b.getDenseI32ArrayAttr({}));
1237:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 1239-1247

```cpp
1239:     // update partitions for this output
1240:     SmallVector<int> partitions;
1241:     for (auto partition : node->getPartitions())
1242:       partitions.push_back(*partition->id);
1243:     std::sort(partitions.begin(), partitions.end());
1244:     partitionAttrs[idx] = b.getDenseI32ArrayAttr(partitions);
1245:     op->setAttr(kPartitionOutputsAttrName,
1246:                 ArrayAttr::get(context, partitionAttrs));
1247:   };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 1249-1251

```cpp
1249:   graph->walk([&](Node *node) {
1250:     if (node->isOp()) {
1251:       setPartitionsAttr(node->getOp(), node);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. The implementation traverses IR operations to collect facts or apply rewrites globally.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。
### Lines 1253-1257

```cpp
1253:       if (auto ret = dyn_cast<tt::ReduceReturnOp>(node->getOp())) {
1254:         // result of a reduce
1255:         auto reduce = node->getParent()->getOp();
1256:         setPartitionOutputsAttr(reduce, 0, 1, node);
1257:       }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1259-1275

```cpp
1259:     } else {
1260:       auto value = node->getValue();
1261:       if (auto blockArg = dyn_cast<BlockArgument>(value)) {
1262:         auto parentOp = blockArg.getOwner()->getParentOp();
1263:         if (isa<tt::FuncOp>(parentOp)) {
1264:           // nothing for func ops
1265:         } else if (auto forOp = dyn_cast<scf::ForOp>(parentOp)) {
1266:           if (blockArg.getArgNumber() == 0) {
1267:             // nothing for induction variable
1268:           } else {
1269:             // for op iter args
1270:             setPartitionOutputsAttr(parentOp, blockArg.getArgNumber() - 1,
1271:                                     forOp.getResultTypes().size(), node);
1272:           }
1273:         } else {
1274:           assert(false);
1275:         }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 1276-1291

```cpp
1276:       } else if (auto result = dyn_cast<OpResult>(value)) {
1277:         auto op = result.getOwner();
1278:         if (isa<scf::ForOp>(op)) {
1279:           // do nothing (handled by block arg)
1280:         } else if (auto ifOp = dyn_cast<scf::IfOp>(op)) {
1281:           // result of an if
1282:           setPartitionOutputsAttr(op, result.getResultNumber(),
1283:                                   ifOp.getResultTypes().size(), node);
1284:         } else {
1285:           assert(false);
1286:         }
1287:       } else {
1288:         assert(false);
1289:       }
1290:     }
1291:   });
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 1293-1302

```cpp
1293:   // set stages
1294:   SmallVector<Attribute> stages;
1295:   for (auto &partition : graph->getPartitions()) {
1296:     auto id = *partition->id;
1297:     while (id >= stages.size())
1298:       stages.push_back(b.getI32IntegerAttr(0));
1299:     stages[id] = b.getI32IntegerAttr(partition->getStage());
1300:   }
1301:   region->setAttr(kPartitionStagesAttrName, b.getArrayAttr(stages));
1302: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 1304-1306

```cpp
1304: void duplicateViewOps(Graph *graph) {
1305:   // Ensure all view ops (e.g. broadcast/expand dims) have a single user,
1306:   // by duplicating nodes where necessary
```

- **EN:** Defines `duplicateViewOps`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `duplicateViewOps`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1308-1308

```cpp
1308:   SmallVector<Node *> viewOps;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1310-1313

```cpp
1310:   graph->walk([&](Node *node) {
1311:     if (node->isData() && node->isOp() && isViewOp(node->getOp()))
1312:       viewOps.push_back(node);
1313:   });
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. The implementation traverses IR operations to collect facts or apply rewrites globally.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。
### Lines 1315-1317

```cpp
1315:   while (!viewOps.empty()) {
1316:     auto node = viewOps.pop_back_val();
1317:     auto op = node->getOp();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1319-1319

```cpp
1319:     assert(op->getResults().size() == 1);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 1321-1321

```cpp
1321:     auto outEdges = node->getOutEdges();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1323-1327

```cpp
1323:     bool first = true;
1324:     for (auto edge : outEdges) {
1325:       if (!first) {
1326:         auto newNode = node->getParent()->addNode(op, op->getNumOperands(),
1327:                                                   op->getNumResults());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1329-1330

```cpp
1329:         // remove old edge
1330:         Node::removeEdge(edge);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1332-1335

```cpp
1332:         // add new edge
1333:         OutputPort outputPort(newNode, 0);
1334:         OutputPort inputPort(edge.getToNode(), edge.getToIdx());
1335:         Node::addEdge(outputPort, inputPort);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1337-1341

```cpp
1337:         // add operands of new node
1338:         for (auto inEdge : node->getInEdges()) {
1339:           Node::addEdge(inEdge.getFrom(),
1340:                         InputPort(newNode, inEdge.getToIdx()));
1341:         }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1343-1353

```cpp
1343:         // copy data values
1344:         for (auto idx = 0; idx < op->getNumResults(); idx++) {
1345:           if (node->isDataValue(idx)) {
1346:             newNode->setDataValue(idx);
1347:           }
1348:         }
1349:       }
1350:       first = false;
1351:     }
1352:   }
1353: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1355-1356

```cpp
1355: void assignPartitionIds(Graph *graph) {
1356:   size_t idx = 0;
```

- **EN:** Defines `assignPartitionIds`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `assignPartitionIds`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1358-1361

```cpp
1358:   SmallVector<Partition *> store_partitions;
1359:   SmallVector<Partition *> mma_partitions;
1360:   SmallVector<Partition *> load_partitions;
1361:   SmallVector<Partition *> other_partitions;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1363-1372

```cpp
1363:   for (auto partition : graph->getPartitions()) {
1364:     if (partition->getFlags() & Flags::STORE)
1365:       store_partitions.push_back(partition);
1366:     else if (partition->getFlags() & Flags::MMA)
1367:       mma_partitions.push_back(partition);
1368:     else if (partition->getFlags() & Flags::LOAD)
1369:       load_partitions.push_back(partition);
1370:     else
1371:       other_partitions.push_back(partition);
1372:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1374-1391

```cpp
1374:   for (auto partition : other_partitions) {
1375:     partition->id = idx;
1376:     idx++;
1377:   }
1378:   for (auto partition : store_partitions) {
1379:     partition->id = idx;
1380:     idx++;
1381:   }
1382:   // ensure MMA and LOAD partitions are never the same as the default
1383:   // partition
1384:   if (idx == 0)
1385:     idx++;
1386:   for (auto partition : mma_partitions) {
1387:     partition->id = idx;
1388:     idx++;
1389:   }
1390:   for (auto partition : load_partitions) {
1391:     partition->id = idx;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1392-1394

```cpp
1392:     idx++;
1393:   }
1394: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1396-1413

```cpp
1396: void assignPartitionsForOpsWithNoUse(Graph *graph) {
1397:   // nodes with no partition placed in same partition as other ops in the
1398:   // region or default partition if none. Note: we can't just use partitions
1399:   // of parent op, as this includes things like tmem tokens
1400:   Partition *defaultPartition = nullptr;
1401:   for (auto partition : graph->getPartitions())
1402:     if (partition->id && *partition->id == 0)
1403:       defaultPartition = partition;
1404:   graph->walk([&](Node *node) {
1405:     if (node->getPartitions().empty()) {
1406:       bool done = false;
1407:       auto parent = node->getParent();
1408:       if (parent && parent->isOp()) {
1409:         for (auto &otherNode : parent->getNodes()) {
1410:           if (node == otherNode.get())
1411:             continue;
1412:           if (otherNode->isOp() && otherNode->hasPartition()) {
1413:             node->addPartitions(otherNode->getPartitions());
```

- **EN:** Defines `assignPartitionsForOpsWithNoUse`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. The implementation traverses IR operations to collect facts or apply rewrites globally.
- **CN:** 这里定义 `assignPartitionsForOpsWithNoUse`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。
### Lines 1414-1428

```cpp
1414:             done = true;
1415:           }
1416:         }
1417:       }
1418:       if (!done) {
1419:         if (defaultPartition == nullptr) {
1420:           // default partition doesn't exist, create one
1421:           defaultPartition = graph->addPartition();
1422:           defaultPartition->id = 0;
1423:         }
1424:         node->setPartition(defaultPartition);
1425:       }
1426:     }
1427:   });
1428: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1430-1430

```cpp
1430: } // namespace
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 1432-1434

```cpp
1432: //===----------------------------------------------------------------------===//
1433: // Pass Definition
1434: //===----------------------------------------------------------------------===//
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 1436-1438

```cpp
1436: struct PartitionScheduling
1437:     : public impl::TritonGPUPartitionSchedulingBase<PartitionScheduling> {
1438:   using TritonGPUPartitionSchedulingBase::TritonGPUPartitionSchedulingBase;
```

- **EN:** Defines `PartitionScheduling`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `PartitionScheduling`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1440-1446

```cpp
1440:   void runOnOperation() override {
1441:     // find ops to partition
1442:     SmallVector<Operation *> ops;
1443:     getOperation().walk([&](scf::ForOp op) {
1444:       if (op->hasAttr(kWarpSpecializeAttrName))
1445:         ops.push_back(op);
1446:     });
```

- **EN:** Defines `runOnOperation`, the pass entry point. It gathers analysis information and applies the file's transformation logic to the current operation. The implementation traverses IR operations to collect facts or apply rewrites globally.
- **CN:** 这里定义 `runOnOperation`，即 pass 的入口函数。它会收集分析信息，并把本文件的变换逻辑应用到当前操作上。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。
### Lines 1448-1462

```cpp
1448:     // run partitioner on each op
1449:     size_t idx = 0;
1450:     for (auto op : ops) {
1451:       analyze(idx, op);
1452:       if (hasPartition(op))
1453:         cloneMultiPartitionDataOps(op);
1454:       if (auto loop = dyn_cast<scf::ForOp>(op);
1455:           loop && loop->hasAttr(kPartitionStagesAttrName) &&
1456:           failed(verifyPartitionedLoop(loop))) {
1457:         signalPassFailure();
1458:         return;
1459:       }
1460:       idx++;
1461:     }
1462:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1464-1466

```cpp
1464: private:
1465:   void analyze(size_t idx, Operation *op) {
1466:     using namespace partition_scheduling_detail;
```

- **EN:** Defines `analyze`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `analyze`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1468-1468

```cpp
1468:     auto func = op->getParentOfType<FuncOp>();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1470-1471

```cpp
1470:     VisualizationInfo vis_info;
1471:     auto key = func.getSymName().str() + "_" + std::to_string(idx);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1473-1481

```cpp
1473:     auto graph = buildGraph(op);
1474:     visualize(key, "input", "input", graph.get(), vis_info);
1475:     auto initValues = initialDataValues(graph.get());
1476:     propagateDataValues(initValues);
1477:     visualize(key, "input", "after data values", graph.get(), vis_info);
1478:     duplicateViewOps(graph.get());
1479:     visualize(key, "input", "after duplicate view ops", graph.get(), vis_info);
1480:     deserializeManualPartitions(op, graph.get());
1481:     visualize(key, "input", "final", graph.get(), vis_info);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1483-1488

```cpp
1483:     initialPartitionAssignment(graph.get());
1484:     visualize(key, "initial", "initial partitions", graph.get(), vis_info);
1485:     mergePartitions(graph.get(), key, vis_info);
1486:     visualize(key, "merge", "merged", graph.get(), vis_info);
1487:     propagatePartitions(graph.get(), key, vis_info);
1488:     visualize(key, "propagate", "propagated", graph.get(), vis_info);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1490-1504

```cpp
1490:     assignPartitionIds(graph.get());
1491:     visualize(key, "assign-partition-ids", "assign partition ids", graph.get(),
1492:               vis_info);
1493:     // Handle case where ops with no uses (like llvm.intr.assume) get no
1494:     // partition assigned
1495:     assignPartitionsForOpsWithNoUse(graph.get());
1496:     visualize(key, "assign-no-use", "assign no use", graph.get(), vis_info);
1497:     propagatePartitions(graph.get(), key, vis_info);
1498:     visualize(key, "propagate", "propagated", graph.get(), vis_info);
1499:     // Optimization: looks for paths of NONE ops with low cost, from one
1500:     // partition, through another partition, and back to the same partition.
1501:     // Duplicates these to avoid the aref involved (i.e. assign to both
1502:     // partitions)
1503:     duplicateCheapOps(graph.get(), key, vis_info);
1504:     visualize(key, "final", "final", graph.get(), vis_info);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1506-1510

```cpp
1506:     LLVM_DEBUG({
1507:       llvm::errs() << "\nfinal partitions:\n";
1508:       for (auto &partition : graph->getPartitions())
1509:         partition->dump();
1510:     });
```

- **EN:** Defines debug logging helpers and compile-time tags used when tracing this pass or utility at runtime.
- **CN:** 这里定义调试日志辅助宏和编译期标签，用于在运行时跟踪该 pass 或工具。
### Lines 1512-1513

```cpp
1512:     if (!hasEligibleMemoryOps(graph.get()))
1513:       return;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1515-1516

```cpp
1515:     serialize(idx, op, graph.get());
1516:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1518-1523

```cpp
1518:   void cloneMultiPartitionDataOps(Operation *region) {
1519:     // FIXME: this transformation runs after the partition scheduling is
1520:     // complete It clones "data" ops with multiple partitions assigned, as
1521:     // insert-aref pass cannot currently handly these. E.g. an op assigned to
1522:     // partitions 0,1 will be cloned into two ops, one in partition 0 and the
1523:     // other in partition 1 and all uses are updated correctly.
```

- **EN:** Defines `cloneMultiPartitionDataOps`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `cloneMultiPartitionDataOps`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1525-1525

```cpp
1525:     using namespace partition_scheduling_detail;
```

- **EN:** Introduces namespace aliases/imports (`partition_scheduling_detail`) so the rest of the file can use MLIR/Triton symbols without repetitive qualification.
- **CN:** 这里通过命名空间导入（`partition_scheduling_detail`）减少后续代码中的重复限定，使 MLIR/Triton 符号使用更简洁。
### Lines 1527-1537

```cpp
1527:     // build data flow graph to find all data ops
1528:     DenseSet<Operation *> dataOps;
1529:     {
1530:       auto graph = buildGraph(region);
1531:       auto initValues = initialDataValues(graph.get());
1532:       propagateDataValues(initValues);
1533:       graph->walk([&](Node *node) {
1534:         if (node->isOp() && node->isData())
1535:           dataOps.insert(node->getOp());
1536:       });
1537:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. The implementation traverses IR operations to collect facts or apply rewrites globally.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。
### Lines 1539-1543

```cpp
1539:     // for each partition, find all data ops that are in that partition,
1540:     // and in another partition
1541:     for (auto partition : getPartitionIds(region)) {
1542:       SetVector<int> partitionSet;
1543:       partitionSet.insert(partition);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1545-1551

```cpp
1545:       SmallVector<Operation *> ops;
1546:       region->walk([&](Operation *op) {
1547:         auto partitions = getPartitionIds(op);
1548:         if (partitions.contains(partition) && partitions.size() > 1 &&
1549:             dataOps.contains(op))
1550:           ops.push_back(op);
1551:       });
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. The implementation traverses IR operations to collect facts or apply rewrites globally.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。
### Lines 1553-1563

```cpp
1553:       SmallVector<Operation *> oldOps;
1554:       SetVector<Operation *> newOps;
1555:       DenseMap<Operation *, Operation *> mapping;
1556:       for (auto op : ops) {
1557:         auto newOp = OpBuilder(op).clone(*op);
1558:         setPartition(newOp, partitionSet);
1559:         oldOps.push_back(op);
1560:         newOps.insert(newOp);
1561:         mapping[newOp] = op;
1562:         mapping[op] = newOp;
1563:       }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 1565-1582

```cpp
1565:       // rewrite operands
1566:       // if op that produces operand of new op is has a duplicated op,
1567:       // rewrite the operand to use that op
1568:       for (auto newOp : newOps) {
1569:         for (auto &operand : newOp->getOpOperands()) {
1570:           auto value = operand.get();
1571:           if (isa<OpResult>(value)) {
1572:             auto result = cast<OpResult>(value);
1573:             auto producerOp = result.getOwner();
1574:             if (mapping.contains(producerOp)) {
1575:               auto newProducerOp = mapping[producerOp];
1576:               auto newValue =
1577:                   newProducerOp->getResult(result.getResultNumber());
1578:               auto idx = operand.getOperandNumber();
1579:               newOp->setOperand(idx, newValue);
1580:             }
1581:           }
1582:         }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1583-1583

```cpp
1583:       }
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 1585-1600

```cpp
1585:       // rewrite results
1586:       for (auto newOp : newOps) {
1587:         auto oldOp = mapping[newOp];
1588:         for (auto &use : oldOp->getUses()) {
1589:           auto user = use.getOwner();
1590:           assert(user);
1591:           auto userPartitions = getPartitionIds(user);
1592:           // skip if use is not in same partition as new op
1593:           if (userPartitions != partitionSet)
1594:             continue;
1595:           // update the use to use the new op
1596:           auto result = cast<OpResult>(use.get());
1597:           auto idx = result.getResultNumber();
1598:           use.set(newOp->getResult(idx));
1599:         }
1600:       }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 1602-1618

```cpp
1602:       // remove dead code
1603:       bool done = false;
1604:       while (!done) {
1605:         done = true;
1606:         auto op = oldOps.begin();
1607:         for (; op != oldOps.end(); op++) {
1608:           if ((*op)->getUses().empty()) {
1609:             (*op)->erase();
1610:             oldOps.erase(op);
1611:             done = false;
1612:             break;
1613:           }
1614:         }
1615:       }
1616:     }
1617:   }
1618: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1620-1620

```cpp
1620: } // namespace mlir::triton::gpu
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。

## Key Concepts / 关键概念
- **EN:** The main concern is pass-driven transformation around partition scheduling.
  **CN:** 核心关注点是围绕 Partition Scheduling 的 pass 驱动变换。
- **EN:** Dataflow analysis tracks facts across operations to make conservative optimization decisions.
  **CN:** 数据流分析会跨操作跟踪事实，以做出保守但有效的优化决策。
- **EN:** Tensor Memory Accelerator related logic appears in this implementation.
  **CN:** 此实现涉及 Tensor Memory Accelerator（TMA）相关逻辑。
- **EN:** Tensor memory specific allocation, layout, or synchronization rules are important here.
  **CN:** 这里重点处理张量内存（TMEM）的分配、布局或同步规则。
- **EN:** Layout and encoding decisions are first-class because they determine how work and memory are distributed on the GPU.
  **CN:** 布局与编码决策是第一类问题，因为它们决定了 GPU 上工作与内存的分布方式。

## Dependencies / 依赖关系
- **Internal Triton headers / Triton 内部头文件:** `triton/Dialect/Triton/IR/Dialect.h`, `triton/Dialect/TritonGPU/IR/Dialect.h`, `triton/Dialect/TritonGPU/Transforms/Partition.h`, `triton/Dialect/TritonGPU/Transforms/PartitionSchedulingUtility.h`, `triton/Dialect/TritonGPU/Transforms/PipeliningUtility.h`, `triton/Dialect/TritonNvidiaGPU/IR/Dialect.h`, ... (+1 more)
- **MLIR headers / MLIR 头文件:** `mlir/Pass/Pass.h`, `mlir/Support/LLVM.h`
- **LLVM headers / LLVM 头文件:** `llvm/Support/Debug.h`
- **Standard/library headers / 标准或通用库头文件:** `PartitionAttrs.h`
- **Generated fragments / 生成片段:** `triton/Dialect/TritonGPU/Transforms/Passes.h.inc`
- **Primary APIs used / 主要 API:** `OpBuilder`
- **Pipeline role / 流程角色:** This file participates in Triton pass pipelines and usually expects upstream analyses or dialect invariants to have prepared the IR. / 本文件参与 Triton 的 pass 流程，通常依赖上游分析或方言不变量先把 IR 准备好。
