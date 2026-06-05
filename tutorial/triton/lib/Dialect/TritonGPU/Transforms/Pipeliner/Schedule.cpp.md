# Schedule.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `lib/Dialect/TritonGPU/Transforms/Pipeliner/Schedule.cpp`
- **Purpose / 作用:** **EN:** Implements the Schedule transformation or optimization pass for the TritonGPU pipeline. **CN:** 为 TritonGPU 编译流程实现与 Schedule 相关的变换或优化 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2

```cpp
   1: #include "triton/Dialect/TritonGPU/Transforms/Schedule.h"
   2: #include "triton/Dialect/TritonGPU/Transforms/PipeliningUtility.h"
```

- **EN:** Includes the interfaces this file depends on. Triton headers (`Schedule.h`, `PipeliningUtility.h`) provide domain-specific IR/support, MLIR headers (None) provide rewriting and analysis infrastructure, LLVM headers (None) supply low-level utilities, and standard/library headers (None) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（`Schedule.h`, `PipeliningUtility.h`）提供领域特定 IR/支持逻辑，MLIR 头文件（None）提供重写与分析基础设施，LLVM 头文件（None）提供底层工具，而标准/通用库头文件（None）提供通用能力。
### Lines 4-11

```cpp
   4: #include "mlir/Dialect/Tensor/IR/Tensor.h"
   5: #include "mlir/IR/TypeUtilities.h"
   6: #include "mlir/Interfaces/SideEffectInterfaces.h"
   7: #include "mlir/Support/LLVM.h"
   8: #include "triton/Dialect/TritonGPU/IR/Dialect.h"
   9: #include "triton/Dialect/TritonGPU/Transforms/Passes.h"
  10: #include "triton/Dialect/TritonGPU/Transforms/Utility.h"
  11: #include "llvm/Support/Debug.h"
```

- **EN:** Includes the interfaces this file depends on. Triton headers (`Dialect.h`, `Passes.h`, `Utility.h`) provide domain-specific IR/support, MLIR headers (`Tensor.h`, `TypeUtilities.h`, `SideEffectInterfaces.h`, `LLVM.h`) provide rewriting and analysis infrastructure, LLVM headers (`Debug.h`) supply low-level utilities, and standard/library headers (None) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（`Dialect.h`, `Passes.h`, `Utility.h`）提供领域特定 IR/支持逻辑，MLIR 头文件（`Tensor.h`, `TypeUtilities.h`, `SideEffectInterfaces.h`, `LLVM.h`）提供重写与分析基础设施，LLVM 头文件（`Debug.h`）提供底层工具，而标准/通用库头文件（None）提供通用能力。
### Lines 13-14

```cpp
  13: using namespace mlir;
  14: namespace tt = mlir::triton;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 16-21

```cpp
  16: bool tt::CoarseSchedule::insertMinimum(Operation *op, int stage,
  17:                                        Cluster cluster) {
  18:   auto res = opToStageAndCluster.insert({op, {stage, cluster}});
  19:   if (res.second) {
  20:     return true;
  21:   }
```

- **EN:** Defines `tt::CoarseSchedule::insertMinimum`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `tt::CoarseSchedule::insertMinimum`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 23-23

```cpp
  23:   auto &[existingStage, existingCluster] = res.first->second;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 25-30

```cpp
  25:   // Always insert if the stage is earlier.
  26:   if (stage < existingStage) {
  27:     existingStage = stage;
  28:     existingCluster = cluster;
  29:     return true;
  30:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 32-35

```cpp
  32:   // If the stage is later, no change.
  33:   if (stage > existingStage) {
  34:     return false;
  35:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 37-46

```cpp
  37:   // If existingCluster is reachable from cluster,
  38:   // then cluster is earlier in the list
  39:   for (auto it = std::next(cluster); it != clusters.end(); ++it) {
  40:     if (it == existingCluster) {
  41:       if (existingCluster == cluster)
  42:         return false;
  43:       existingCluster = cluster;
  44:       return true;
  45:     }
  46:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 48-50

```cpp
  48:   // Didn't change the cluster.
  49:   return false;
  50: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 52-60

```cpp
  52: bool tt::CoarseSchedule::insertDepsOfOp(Operation *op, int stage,
  53:                                         tt::CoarseSchedule::Cluster cluster,
  54:                                         bool includeArg, bool insertIfEarlier) {
  55:   auto tryInsert = [&](Operation *op, int stage,
  56:                        tt::CoarseSchedule::Cluster cluster) {
  57:     if (!insertIfEarlier)
  58:       return insertIfAbsent(op, stage, cluster);
  59:     return insertMinimum(op, stage, cluster);
  60:   };
```

- **EN:** Defines `tt::CoarseSchedule::insertDepsOfOp`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `tt::CoarseSchedule::insertDepsOfOp`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 62-78

```cpp
  62:   bool inserted = false;
  63:   for (Value operand : getNestedOperands(op)) {
  64:     Value v = operand;
  65:     llvm::SmallDenseSet<Value> seen;
  66:     while (auto arg = dyn_cast<BlockArgument>(v)) {
  67:       if (!includeArg)
  68:         break;
  69:       if (!seen.insert(v).second)
  70:         break;
  71:       if (arg.getArgNumber() > 0 && arg.getOwner() == op->getBlock()) {
  72:         auto yieldOp = op->getBlock()->getTerminator();
  73:         v = yieldOp->getOperand(arg.getArgNumber() - 1);
  74:         continue;
  75:       }
  76:       break;
  77:     }
  78:     Operation *defOp = v.getDefiningOp();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 79-87

```cpp
  79:     if (defOp && defOp->getBlock() == op->getBlock()) {
  80:       if (tryInsert(defOp, stage, cluster)) {
  81:         inserted = true;
  82:         insertDepsOfOp(defOp, stage, cluster, includeArg, insertIfEarlier);
  83:       }
  84:     }
  85:   }
  86:   return inserted;
  87: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 89-100

```cpp
  89: void tt::CoarseSchedule::shrinkToFit() {
  90:   int minStage = std::numeric_limits<int>::max();
  91:   int maxStage = std::numeric_limits<int>::min();
  92:   for (auto &[op, stageAndCluster] : opToStageAndCluster) {
  93:     auto [stage, cluster] = stageAndCluster;
  94:     minStage = std::min(minStage, stage);
  95:     maxStage = std::max(maxStage, stage);
  96:   }
  97:   for (auto &[op, stageAndCluster] : opToStageAndCluster)
  98:     stageAndCluster.first -= minStage;
  99:   numStages = maxStage - minStage + 1;
 100: }
```

- **EN:** Defines `tt::CoarseSchedule::shrinkToFit`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `tt::CoarseSchedule::shrinkToFit`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 102-117

```cpp
 102: // Split the cluster containing op into two clusters, one containing all
 103: // operations before the op and one containing op and all operations after the
 104: // op. Return the cluster containing op and all operations after the op. Do not
 105: // split if the op is the first operation in the cluster.
 106: tt::CoarseSchedule::Cluster
 107: tt::CoarseSchedule::splitClusterBefore(Operation *op, scf::ForOp forOp) {
 108:   auto it = opToStageAndCluster.find(op);
 109:   assert(it != opToStageAndCluster.end() &&
 110:          "Operation must be in the schedule!");
 111:   auto cluster = it->second.second;
 112:   std::optional<tt::CoarseSchedule::Cluster> newCluster = std::nullopt;
 113:   for (auto &_op : forOp.getBody()->without_terminator()) {
 114:     if (&_op == op) {
 115:       break;
 116:     }
 117:     auto it_op = opToStageAndCluster.find(&_op);
```

- **EN:** Defines `tt::CoarseSchedule::splitClusterBefore`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `tt::CoarseSchedule::splitClusterBefore`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 118-126

```cpp
 118:     if (it_op != opToStageAndCluster.end() && it_op->second.second == cluster) {
 119:       if (!newCluster) {
 120:         newCluster = clusters.newBefore(cluster);
 121:       }
 122:       it_op->second.second = *newCluster;
 123:     }
 124:   }
 125:   return cluster;
 126: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 128-141

```cpp
 128: // Check if op a will show up before op b in the final unrolled code.
 129: bool tt::CoarseSchedule::isOpBefore(Operation *a, Operation *b) const {
 130:   assert(opToStageAndCluster.count(a) && opToStageAndCluster.count(b) &&
 131:          "Operations must be in the schedule");
 132:   auto [aStage, aCluster] = opToStageAndCluster.lookup(a);
 133:   auto [bStage, bCluster] = opToStageAndCluster.lookup(b);
 134:   if (aStage != bStage) {
 135:     return aStage < bStage;
 136:   }
 137:   if (aCluster != bCluster) {
 138:     return clusters.isBefore(aCluster, bCluster);
 139:   }
 140:   return a->isBeforeInBlock(b);
 141: }
```

- **EN:** Defines `tt::CoarseSchedule::isOpBefore`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `tt::CoarseSchedule::isOpBefore`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 143-149

```cpp
 143: bool tt::CoarseSchedule::isOpInEarlierCluster(Operation *a,
 144:                                               Operation *b) const {
 145:   assert(opToStageAndCluster.count(a) && opToStageAndCluster.count(b) &&
 146:          "Operations must be in the schedule");
 147:   return clusters.isBefore(opToStageAndCluster.lookup(a).second,
 148:                            opToStageAndCluster.lookup(b).second);
 149: }
```

- **EN:** Defines `tt::CoarseSchedule::isOpInEarlierCluster`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `tt::CoarseSchedule::isOpInEarlierCluster`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 151-156

```cpp
 151: bool tt::CoarseSchedule::isOpInSameCluster(Operation *a, Operation *b) const {
 152:   assert(opToStageAndCluster.count(a) && opToStageAndCluster.count(b) &&
 153:          "Operations must be in the schedule");
 154:   return opToStageAndCluster.lookup(a).second ==
 155:          opToStageAndCluster.lookup(b).second;
 156: }
```

- **EN:** Defines `tt::CoarseSchedule::isOpInSameCluster`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `tt::CoarseSchedule::isOpInSameCluster`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 158-175

```cpp
 158: SmallVector<std::tuple<Operation *, int, tt::CoarseSchedule::Cluster>>
 159: tt::CoarseSchedule::getOpsInOrder(scf::ForOp forOp) const {
 160:   SmallVector<SmallVector<std::tuple<Operation *, int, Cluster>>, 8>
 161:       orderClusters(clusters.size());
 162:   for (auto &op : forOp.getBody()->without_terminator()) {
 163:     auto it = opToStageAndCluster.find(&op);
 164:     if (it == opToStageAndCluster.end()) {
 165:       continue;
 166:     }
 167:     auto [stage, cluster] = it->second;
 168:     assert(cluster != Cluster{} && "Op with invalid cluster!");
 169:     assert(stage < numStages && "Op with invalid stage!");
 170:     int clusterId = *cluster;
 171:     assert(clusterId == std::distance(clusters.begin(),
 172:                                       ClusterList::const_iterator(cluster)) &&
 173:            "Cluster ID mismatch!");
 174:     orderClusters[clusterId].push_back(make_tuple(&op, stage, cluster));
 175:   }
```

- **EN:** Defines accessor/helper `tt::CoarseSchedule::getOpsInOrder` that exposes or updates operation state in a compact, reusable way. Assertions document invariants that should already hold at this stage. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义访问器/辅助函数 `tt::CoarseSchedule::getOpsInOrder`，以紧凑且可复用的方式读取或更新操作状态。 断言用于说明在当前阶段本就应该成立的不变量。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 176-181

```cpp
 176:   SmallVector<std::tuple<Operation *, int, Cluster>> opsInOrder;
 177:   for (int i = 0; i < orderClusters.size(); i++) {
 178:     for (auto [op, stage, cluster] : orderClusters[i]) {
 179:       opsInOrder.push_back({op, stage, cluster});
 180:     }
 181:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 183-184

```cpp
 183:   return opsInOrder;
 184: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 186-194

```cpp
 186: std::vector<std::pair<Operation *, unsigned>>
 187: tt::CoarseSchedule::createFinalSchedule(scf::ForOp forOp) const {
 188:   SmallVector<std::tuple<Operation *, int, tt::CoarseSchedule::Cluster>>
 189:       opsInOrder = getOpsInOrder(forOp);
 190:   std::vector<std::pair<Operation *, unsigned>> schedule;
 191:   for (auto [op, stage, cluster] : opsInOrder)
 192:     schedule.push_back({op, stage});
 193:   return schedule;
 194: }
```

- **EN:** Defines helper `tt::CoarseSchedule::createFinalSchedule` that computes or constructs intermediate data used by the surrounding transformation. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义辅助函数 `tt::CoarseSchedule::createFinalSchedule`，用于计算或构造外围变换所需的中间数据。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 196-207

```cpp
 196: void tt::CoarseSchedule::dump() {
 197:   assert(numStages > 0 && "Invalid number of stages");
 198:   for (int i = 0; i < numStages; i++) {
 199:     llvm::dbgs() << "\n---- Ops in stage " << i << "\n";
 200:     for (auto &[op, stageAndCluster] : opToStageAndCluster) {
 201:       if (i == stageAndCluster.first) {
 202:         llvm::dbgs() << "        cluster: " << *stageAndCluster.second
 203:                      << ":\n\t" << *op << "\n";
 204:       }
 205:     }
 206:   }
 207: }
```

- **EN:** Defines debug logging helpers and compile-time tags used when tracing this pass or utility at runtime.
- **CN:** 这里定义调试日志辅助宏和编译期标签，用于在运行时跟踪该 pass 或工具。
### Lines 209-215

```cpp
 209: static void setStageCluster(Operation *op, int stage, int cluster) {
 210:   auto ctx = op->getContext();
 211:   op->setAttr(mlir::triton::kLoopStageAttrName,
 212:               IntegerAttr::get(IntegerType::get(ctx, 32), stage));
 213:   op->setAttr(mlir::triton::kLoopClusterAttrName,
 214:               IntegerAttr::get(IntegerType::get(ctx, 32), cluster));
 215: }
```

- **EN:** Defines accessor/helper `setStageCluster` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `setStageCluster`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 217-223

```cpp
 217: static std::pair<int, int> getStageCluster(Operation *op) {
 218:   auto stage = op->getAttrOfType<IntegerAttr>(tt::kLoopStageAttrName);
 219:   auto clusterId = op->getAttrOfType<IntegerAttr>(tt::kLoopClusterAttrName);
 220:   assert(stage && clusterId &&
 221:          "Operation is missing stage & cluster attribute");
 222:   return {stage.getValue().getSExtValue(), clusterId.getValue().getSExtValue()};
 223: }
```

- **EN:** Defines accessor/helper `getStageCluster` that exposes or updates operation state in a compact, reusable way. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义访问器/辅助函数 `getStageCluster`，以紧凑且可复用的方式读取或更新操作状态。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 225-241

```cpp
 225: static std::pair<int, int> getMinMaxCluster(scf::ForOp &forOp) {
 226:   int minClusterId = -1, maxClusterId = -1;
 227:   for (auto &op : forOp.getBody()->without_terminator()) {
 228:     if (!op.hasAttr(mlir::triton::kLoopStageAttrName) ||
 229:         !op.hasAttr(mlir::triton::kLoopClusterAttrName))
 230:       continue;
 231:     auto [_, cluster] = getStageCluster(&op);
 232:     if (maxClusterId < 0) {
 233:       minClusterId = cluster;
 234:       maxClusterId = cluster;
 235:       continue;
 236:     }
 237:     maxClusterId = cluster > maxClusterId ? cluster : maxClusterId;
 238:     minClusterId = cluster < minClusterId ? cluster : minClusterId;
 239:   }
 240:   return std::make_pair(minClusterId, maxClusterId);
 241: }
```

- **EN:** Defines accessor/helper `getMinMaxCluster` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getMinMaxCluster`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 243-252

```cpp
 243: static std::optional<int> tryGetMaxStage(scf::ForOp &forOp) {
 244:   std::optional<int> maxStage = std::nullopt;
 245:   if (forOp->hasAttr(mlir::triton::kScheduledMaxStageAttrName)) {
 246:     return forOp
 247:         ->getAttrOfType<IntegerAttr>(mlir::triton::kScheduledMaxStageAttrName)
 248:         .getValue()
 249:         .getSExtValue();
 250:   }
 251:   return maxStage;
 252: }
```

- **EN:** Defines `tryGetMaxStage`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `tryGetMaxStage`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 254-258

```cpp
 254: // Set <stage, cluster> based on CoarseSchedule.
 255: void tt::CoarseSchedule::serialize(scf::ForOp &forOp) const {
 256:   for (auto [op, stage, cluster] : getOpsInOrder(forOp)) {
 257:     setStageCluster(op, stage, *cluster);
 258:   }
```

- **EN:** Defines `tt::CoarseSchedule::serialize`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `tt::CoarseSchedule::serialize`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 260-266

```cpp
 260:   Builder b(forOp.getContext());
 261:   int maxStages = numStages - 1;
 262:   if (auto maxStageAttr = tryGetMaxStage(forOp))
 263:     maxStages = std::max(maxStages, *maxStageAttr);
 264:   forOp->setAttr(mlir::triton::kScheduledMaxStageAttrName,
 265:                  b.getI32IntegerAttr(maxStages));
 266: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 268-276

```cpp
 268: // Create a CoarseSchedule based on forOp's <stage, cluster>.
 269: LogicalResult tt::CoarseSchedule::deSerialize(scf::ForOp &forOp,
 270:                                               bool normalizeClusterId) {
 271:   auto [minClusterId, maxClusterId] = getMinMaxCluster(forOp);
 272:   std::optional<int> maxStage = tryGetMaxStage(forOp);
 273:   if (!maxStage) {
 274:     return failure();
 275:   }
 276:   numStages = *maxStage + 1;
```

- **EN:** Defines `tt::CoarseSchedule::deSerialize`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `tt::CoarseSchedule::deSerialize`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 278-287

```cpp
 278:   DenseMap<int, tt::CoarseSchedule::Cluster> clustersMap;
 279:   if (normalizeClusterId) {
 280:     for (int i = minClusterId; i < maxClusterId + 1; i++) {
 281:       clustersMap.insert({i, clusters.newAtBack()});
 282:     }
 283:   } else {
 284:     for (int i = 0; i < maxClusterId + 1; i++) {
 285:       clustersMap.insert({i, clusters.newAtBack()});
 286:     }
 287:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 289-296

```cpp
 289:   for (Operation &op : forOp.getBody()->without_terminator()) {
 290:     if (!op.hasAttr(mlir::triton::kLoopStageAttrName))
 291:       continue;
 292:     auto [stage, clusterId] = getStageCluster(&op);
 293:     insert(&op, stage, clustersMap[clusterId]);
 294:   }
 295:   return success();
 296: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 298-303

```cpp
 298: // TODO: Should this be moved somewhere else?
 299: // Add dependencies of anchor ops to the coarse schedule. Schedule them to
 300: // the same stage and ordering cluster as the anchor op.
 301: // ============================================================
 302: // LinearizedIterator Implementation
 303: // ============================================================
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 305-312

```cpp
 305: tt::CoarseSchedule::LinearizedIterator::LinearizedIterator(
 306:     scf::ForOp forOp, const CoarseSchedule &schedule, Operation *initialOp)
 307:     : forOp(forOp), schedule(&schedule), initialOp(initialOp), atEnd(false),
 308:       maxStages(schedule.getNumStages()) {
 309:   clusterBegin = schedule.clusters.begin();
 310:   clusterEnd = schedule.clusters.end();
 311:   opIt = forOp.getBody()->without_terminator().begin();
 312:   opEnd = forOp.getBody()->without_terminator().end();
```

- **EN:** Defines `tt::CoarseSchedule::LinearizedIterator::LinearizedIterator`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `tt::CoarseSchedule::LinearizedIterator::LinearizedIterator`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 314-330

```cpp
 314:   // Find the cluster containing initialOp and its stage
 315:   auto it = schedule.opToStageAndCluster.find(initialOp);
 316:   if (it != schedule.opToStageAndCluster.end()) {
 317:     auto [stage, cluster] = it->second;
 318:     clusterIt = cluster;
 319:     currStageLimit = stage;
 320:     // Find initialOp within its cluster
 321:     while (opIt != opEnd) {
 322:       Operation *op = &*opIt;
 323:       if (op == initialOp) {
 324:         break;
 325:       }
 326:       ++opIt;
 327:     }
 328:     // Move past initialOp to start iteration from the next op
 329:     ++opIt;
 330:     advanceToNextScheduledOp();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 331-335

```cpp
 331:   } else {
 332:     atEnd = true;
 333:     currentOp = nullptr;
 334:   }
 335: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 337-354

```cpp
 337: void tt::CoarseSchedule::LinearizedIterator::advanceToNextScheduledOp() {
 338:   while (true) {
 339:     while (opIt != opEnd) {
 340:       Operation *op = &*opIt;
 341:       auto it = schedule->opToStageAndCluster.find(op);
 342:       if (it != schedule->opToStageAndCluster.end()) {
 343:         auto [stage, cluster] = it->second;
 344:         if (cluster == clusterIt) {
 345:           // Check if we've come back to initialOp
 346:           if (op == initialOp) {
 347:             // Check termination condition
 348:             if (currStageLimit >= maxStages) {
 349:               atEnd = true;
 350:               currentOp = nullptr;
 351:               return;
 352:             }
 353:           }
 354:           // Only yield if stage <= currStageLimit
```

- **EN:** Defines `tt::CoarseSchedule::LinearizedIterator::advanceToNextScheduledOp`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `tt::CoarseSchedule::LinearizedIterator::advanceToNextScheduledOp`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 355-365

```cpp
 355:           if (stage <= currStageLimit) {
 356:             currentOp = op;
 357:             return;
 358:           }
 359:         }
 360:       }
 361:       ++opIt;
 362:     }
 363:     // Move to next cluster
 364:     ++clusterIt;
 365:     opIt = forOp.getBody()->without_terminator().begin();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 367-374

```cpp
 367:     // Wrap around to the beginning if we've reached the end
 368:     if (clusterIt == clusterEnd) {
 369:       clusterIt = clusterBegin;
 370:       // Increment stage limit as we are in the next iteration.
 371:       currStageLimit++;
 372:     }
 373:   }
 374: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 376-383

```cpp
 376: tt::CoarseSchedule::LinearizedIterator &
 377: tt::CoarseSchedule::LinearizedIterator::operator++() {
 378:   if (atEnd)
 379:     return *this;
 380:   ++opIt;
 381:   advanceToNextScheduledOp();
 382:   return *this;
 383: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 385-390

```cpp
 385: tt::CoarseSchedule::LinearizedIterator
 386: tt::CoarseSchedule::LinearizedIterator::operator++(int) {
 387:   LinearizedIterator tmp = *this;
 388:   ++(*this);
 389:   return tmp;
 390: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 392-394

```cpp
 392: Operation *tt::CoarseSchedule::LinearizedIterator::operator*() const {
 393:   return currentOp;
 394: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 396-403

```cpp
 396: bool tt::CoarseSchedule::LinearizedIterator::operator==(
 397:     const LinearizedIterator &other) const {
 398:   if (atEnd && other.atEnd)
 399:     return true;
 400:   if (atEnd != other.atEnd)
 401:     return false;
 402:   return currentOp == other.currentOp;
 403: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 405-408

```cpp
 405: bool tt::CoarseSchedule::LinearizedIterator::operator!=(
 406:     const LinearizedIterator &other) const {
 407:   return !(*this == other);
 408: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 410-423

```cpp
 410: void tt::scheduleDependencies(scf::ForOp forOp, tt::CoarseSchedule &schedule) {
 411:   int numStages = schedule.getNumStages();
 412:   SmallVector<std::tuple<Operation *, int, tt::CoarseSchedule::Cluster>>
 413:       opsInOrder = schedule.getOpsInOrder(forOp);
 414:   // Schedule dependencies stage by stage.
 415:   for (int stage = 0; stage < numStages; stage++) {
 416:     for (auto [op, stage_, cluster] : opsInOrder) {
 417:       if (stage_ != stage)
 418:         continue;
 419:       schedule.insertDepsOfOp(op, stage, cluster, /*includeArg=*/false,
 420:                               /*insertIfEarlier=*/true);
 421:     }
 422:   }
 423: }
```

- **EN:** Defines `tt::scheduleDependencies`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `tt::scheduleDependencies`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。

## Key Concepts / 关键概念
- **EN:** The main concern is pass-driven transformation around schedule.
  **CN:** 核心关注点是围绕 Schedule 的 pass 驱动变换。

## Dependencies / 依赖关系
- **Internal Triton headers / Triton 内部头文件:** `triton/Dialect/TritonGPU/Transforms/Schedule.h`, `triton/Dialect/TritonGPU/Transforms/PipeliningUtility.h`, `triton/Dialect/TritonGPU/IR/Dialect.h`, `triton/Dialect/TritonGPU/Transforms/Passes.h`, `triton/Dialect/TritonGPU/Transforms/Utility.h`
- **MLIR headers / MLIR 头文件:** `mlir/Dialect/Tensor/IR/Tensor.h`, `mlir/IR/TypeUtilities.h`, `mlir/Interfaces/SideEffectInterfaces.h`, `mlir/Support/LLVM.h`
- **LLVM headers / LLVM 头文件:** `llvm/Support/Debug.h`
- **Standard/library headers / 标准或通用库头文件:** None
- **Pipeline role / 流程角色:** This file participates in Triton pass pipelines and usually expects upstream analyses or dialect invariants to have prepared the IR. / 本文件参与 Triton 的 pass 流程，通常依赖上游分析或方言不变量先把 IR 准备好。
