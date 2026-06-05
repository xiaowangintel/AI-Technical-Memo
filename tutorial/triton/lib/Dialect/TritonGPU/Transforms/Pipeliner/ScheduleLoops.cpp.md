# ScheduleLoops.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `lib/Dialect/TritonGPU/Transforms/Pipeliner/ScheduleLoops.cpp`
- **Purpose / 作用:** **EN:** Implements the Schedule Loops transformation or optimization pass for the TritonGPU pipeline. **CN:** 为 TritonGPU 编译流程实现与 Schedule Loops 相关的变换或优化 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-10

```cpp
   1: #include "mlir/IR/Dominance.h"
   2: #include "triton/Analysis/Utility.h"
   3: #include "triton/Dialect/TritonGPU/IR/Dialect.h"
   4: #include "triton/Dialect/TritonGPU/Transforms/MMAv5PipelineUtility.h"
   5: #include "triton/Dialect/TritonGPU/Transforms/Passes.h"
   6: #include "triton/Dialect/TritonGPU/Transforms/PipeliningUtility.h"
   7: #include "triton/Dialect/TritonGPU/Transforms/Schedule.h"
   8: #include "triton/Dialect/TritonGPU/Transforms/Utility.h"
   9: #include "triton/Dialect/TritonNvidiaGPU/IR/Dialect.h"
  10: #include "llvm/Support/Debug.h"
```

- **EN:** Includes the interfaces this file depends on. Triton headers (`Utility.h`, `Dialect.h`, `MMAv5PipelineUtility.h`, `Passes.h`, ... (+4 more)) provide domain-specific IR/support, MLIR headers (`Dominance.h`) provide rewriting and analysis infrastructure, LLVM headers (`Debug.h`) supply low-level utilities, and standard/library headers (None) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（`Utility.h`, `Dialect.h`, `MMAv5PipelineUtility.h`, `Passes.h`, ... (+4 more)）提供领域特定 IR/支持逻辑，MLIR 头文件（`Dominance.h`）提供重写与分析基础设施，LLVM 头文件（`Debug.h`）提供底层工具，而标准/通用库头文件（None）提供通用能力。
### Lines 12-14

```cpp
  12: #define DEBUG_TYPE "triton-loop-pipeline"
  13: #define DBGS() (llvm::dbgs() << "[" DEBUG_TYPE "]: ")
  14: #define LDBG(X) LLVM_DEBUG(DBGS() << X << "\n")
```

- **EN:** Defines debug logging helpers and compile-time tags used when tracing this pass or utility at runtime.
- **CN:** 这里定义调试日志辅助宏和编译期标签，用于在运行时跟踪该 pass 或工具。
### Lines 16-20

```cpp
  16: using namespace mlir;
  17: namespace tt = mlir::triton;
  18: namespace ttg = mlir::triton::gpu;
  19: namespace ttng = mlir::triton::nvidia_gpu;
  20: namespace mlir::triton::gpu {
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 22-24

```cpp
  22: //===----------------------------------------------------------------------===//
  23: // scheduleLoops
  24: //===----------------------------------------------------------------------===//
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 26-33

```cpp
  26: template <typename... OpTypes> static bool containsAny(scf::ForOp forOp) {
  27:   WalkResult result = forOp.walk([&](Operation *op) {
  28:     if (isa<OpTypes...>(op))
  29:       return WalkResult::interrupt();
  30:     return WalkResult::advance();
  31:   });
  32:   return result.wasInterrupted();
  33: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. The implementation traverses IR operations to collect facts or apply rewrites globally.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。
### Lines 35-45

```cpp
  35: // Return true if the preconditions for pipelining the loop are met.
  36: bool isSafeToPipeline(scf::ForOp forOp) {
  37:   // Skip loop with distance > 1.
  38:   if (loopHasDistGreaterThanOne(forOp))
  39:     return false;
  40:   // Don't pipeline outer loops.
  41:   if (isOuterLoop(forOp))
  42:     return false;
  43:   // Skip loops with barriers, asserts or prints
  44:   if (containsAny<ttg::BarrierOp, tt::AssertOp, tt::PrintOp>(forOp))
  45:     return false;
```

- **EN:** Defines `isSafeToPipeline`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `isSafeToPipeline`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 47-48

```cpp
  47:   return true;
  48: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 50-54

```cpp
  50: // Find dependencies with distance of 1. They will go to the next stage,
  51: // but in the cluster before the current op.
  52: void scheduleDistanceOneDependencies(scf::ForOp forOp,
  53:                                      CoarseSchedule &schedule) {
  54:   int numStages = schedule.getNumStages();
```

- **EN:** Defines `scheduleDistanceOneDependencies`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `scheduleDistanceOneDependencies`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 56-73

```cpp
  56:   // Mapping from the cluster to the cluster before it.
  57:   DenseMap<CoarseSchedule::ClusterHash, CoarseSchedule::Cluster> dist1Cluster;
  58:   for (auto &op : forOp.getBody()->without_terminator()) {
  59:     if (schedule.count(&op) == 0)
  60:       continue;
  61:     auto [stage, cluster] = schedule[&op];
  62:     // Can't schedule past the last stage.
  63:     if (stage == numStages - 1)
  64:       continue;
  65:     for (Value operand : getNestedOperands(&op)) {
  66:       if (auto arg = dyn_cast<BlockArgument>(operand)) {
  67:         if (arg.getArgNumber() > 0 && arg.getOwner() == op.getBlock()) {
  68:           auto yieldOp = op.getBlock()->getTerminator();
  69:           Value v = yieldOp->getOperand(arg.getArgNumber() - 1);
  70:           Operation *defOp = v.getDefiningOp();
  71:           if (defOp && schedule.count(defOp) == 0) {
  72:             if (isa<tt::LoadOp>(defOp)) {
  73:               // Exception: Schedule loads with a distance of 1 together
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 74-91

```cpp
  74:               // with the current op.
  75:               schedule.insertIfAbsent(defOp, stage, cluster);
  76:               schedule.insertDepsOfOp(defOp, stage, cluster,
  77:                                       /*includeArg=*/true,
  78:                                       /*insertIfEarlier=*/true);
  79:             } else {
  80:               CoarseSchedule::ClusterHash clusterHash =
  81:                   CoarseSchedule::hashCluster(cluster);
  82:               if (dist1Cluster.count(clusterHash) == 0) {
  83:                 dist1Cluster[clusterHash] =
  84:                     schedule.clusters.newBefore(cluster);
  85:               }
  86:               schedule.insertIfAbsent(defOp, stage + 1,
  87:                                       dist1Cluster[clusterHash]);
  88:               schedule.insertDepsOfOp(defOp, stage + 1,
  89:                                       dist1Cluster[clusterHash],
  90:                                       /*includeArg=*/true,
  91:                                       /*includeIfEarlier=*/true);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 92-98

```cpp
  92:             }
  93:           }
  94:         }
  95:       }
  96:     }
  97:   }
  98: }
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 100-117

```cpp
 100: void scheduleRemainingToLastStage(scf::ForOp forOp, CoarseSchedule &schedule,
 101:                                   CoarseSchedule::Cluster afterPrologue) {
 102:   int numStages = schedule.getNumStages();
 103:   // Assign the rest of the ops to the last stage.
 104:   // Take care of the ordering of the ops - uses cannot be scheduled to the
 105:   // cluster before the definition.
 106:   DenseMap<Operation *, CoarseSchedule::Cluster> opToCluster;
 107:   for (auto &op : forOp.getBody()->without_terminator()) {
 108:     if (schedule.count(&op) == 0) {
 109:       opToCluster[&op] = afterPrologue;
 110:     }
 111:   }
 112:   SmallVector<Operation *> queue;
 113:   for (auto [op, stage, cluster] : schedule.getOpsInOrder(forOp)) {
 114:     // We really only care about the producers from the last stage.
 115:     // Others will be scheduled before these ops anyway.
 116:     if (stage == numStages - 1) {
 117:       queue.push_back(op);
```

- **EN:** Defines `scheduleRemainingToLastStage`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `scheduleRemainingToLastStage`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 118-135

```cpp
 118:     }
 119:   }
 120:   while (!queue.empty()) {
 121:     Operation *op = queue.pop_back_val();
 122:     for (auto user : op->getUsers()) {
 123:       if (opToCluster.count(user)) {
 124:         CoarseSchedule::Cluster userCluster = opToCluster[user];
 125:         CoarseSchedule::Cluster opCluster;
 126:         if (schedule.count(op))
 127:           opCluster = schedule[op].second;
 128:         else
 129:           opCluster = opToCluster[op];
 130:         if (*userCluster < *opCluster) {
 131:           opToCluster[user] = opCluster;
 132:           queue.push_back(user);
 133:         }
 134:       }
 135:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 136-140

```cpp
 136:   }
 137:   for (auto [op, cluster] : opToCluster) {
 138:     schedule.insert(op, numStages - 1, cluster);
 139:   }
 140: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 142-150

```cpp
 142: namespace {
 143: bool hasLatenciesAssigned(scf::ForOp forOp,
 144:                           const DenseMap<Operation *, int> &opLatency) {
 145:   for (auto &op : forOp.getBody()->without_terminator()) {
 146:     if (opLatency.count(&op))
 147:       return true;
 148:   }
 149:   return false;
 150: }
```

- **EN:** Defines `hasLatenciesAssigned`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `hasLatenciesAssigned`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 152-165

```cpp
 152: CoarseSchedule scheduleKeyOps(scf::ForOp forOp,
 153:                               const DenseMap<Operation *, int> &opLatency) {
 154:   llvm::MapVector<Operation *, int> opToStage;
 155:   // Find terminator for later reference
 156:   auto terminator = cast<scf::YieldOp>(forOp.getBody()->getTerminator());
 157:   // Determine all operations that have a non-zero latency
 158:   SmallVector<Operation *> latOps;
 159:   for (auto &op : forOp.getBody()->without_terminator()) {
 160:     if (opLatency.count(&op))
 161:       latOps.push_back(&op);
 162:   }
 163:   // If no latency ops, nothing to schedule
 164:   if (latOps.empty())
 165:     return CoarseSchedule(0);
```

- **EN:** Defines `scheduleKeyOps`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `scheduleKeyOps`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 167-184

```cpp
 167:   DominanceInfo domInfo(forOp);
 168:   // Compute the longest path to the yield for each operation reachable
 169:   // from any latency operation.
 170:   DenseMap<Operation *, int> distance;
 171:   std::function<int(Operation *)> computeDistance = [&](Operation *op) -> int {
 172:     auto it = distance.find(op);
 173:     if (it != distance.end())
 174:       return it->second;
 175:     // Compute max distance among all users that are inside the loop body
 176:     int maxDist = -1;
 177:     for (Operation *user : op->getUsers()) {
 178:       // Only consider users inside the same block and not the terminator
 179:       Operation *inBlockUser = forOp.getBody()->findAncestorOpInBlock(*user);
 180:       if (!inBlockUser || inBlockUser == terminator)
 181:         continue;
 182:       int distUser = computeDistance(inBlockUser);
 183:       if (distUser > maxDist)
 184:         maxDist = distUser;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 185-194

```cpp
 185:     }
 186:     int lat = 0;
 187:     if (opLatency.count(op))
 188:       lat = opLatency.lookup(op);
 189:     // If an op has no users (maxDist == -1) but has latency, we include its
 190:     // latency otherwise it contributes 0 to the distance.
 191:     int d = lat + (maxDist < 0 ? 0 : maxDist);
 192:     distance[op] = d;
 193:     return d;
 194:   };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 196-202

```cpp
 196:   // Compute distances for all latency-starting ops
 197:   int maxDistance = 0;
 198:   for (Operation *latOp : latOps) {
 199:     int d = computeDistance(latOp);
 200:     if (d > maxDistance)
 201:       maxDistance = d;
 202:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 204-210

```cpp
 204:   // Assign stage to each op reachable from a latency op
 205:   for (auto [op, dist] : distance) {
 206:     // We only schedule ops that are downstream of a latency op
 207:     // (had a non-negative distance due to a latency op).
 208:     if (dist >= 0)
 209:       opToStage[op] = maxDistance - dist;
 210:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 212-223

```cpp
 212:   auto stages = llvm::make_second_range(opToStage);
 213:   int maxStage = *llvm::max_element(stages);
 214:   CoarseSchedule schedule(maxStage + 1);
 215:   SmallVector<CoarseSchedule::Cluster> clusters(maxStage + 1);
 216:   for (int i = 0; i <= maxStage; i++) {
 217:     clusters[i] = schedule.clusters.newAtBack();
 218:   }
 219:   // Assign ops to the clusters in reverse-stage order;
 220:   // ops with higher stage numbers are assigned first. This way we will
 221:   // end up with roughly reverse program order in the clusters.
 222:   for (auto [op, stage] : opToStage)
 223:     schedule.insert(op, stage, clusters[maxStage - stage]);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 225-242

```cpp
 225:   // Move `scf.if` ops in the current schedule (forward slice of the latency
 226:   // ops) into a new epilogue cluster at the end of the schedule, pushing them
 227:   // as close to the end of the loop body as possible.
 228:   CoarseSchedule::Cluster epilogue = schedule.clusters.newAtBack();
 229:   for (auto [op, stage] : opToStage) {
 230:     auto ifOp = dyn_cast<scf::IfOp>(op);
 231:     if (!ifOp)
 232:       continue;
 233:     // If the `scf.if` op itself is a latency op, skip it.
 234:     if (opLatency.contains(ifOp))
 235:       continue;
 236:     // Ensure this does not create scheduling conflicts by ensuring the forward
 237:     // slice of the `scf.if` does not contain ops that are already scheduled, as
 238:     // this will cause the `scf.if` to be scheduled after its dependents.
 239:     SetVector<Operation *> slice;
 240:     getForwardSlice(ifOp, &slice);
 241:     if (llvm::any_of(slice, [&](Operation *op) { return opToStage.count(op); }))
 242:       continue;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 243-244

```cpp
 243:     schedule.insert(ifOp, stage, epilogue);
 244:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 246-247

```cpp
 246:   return schedule;
 247: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 249-254

```cpp
 249: // Get an initial schedule for the loop. This is the base schedule from which
 250: // the rest of the pass will backward propagate dependencies.
 251: CoarseSchedule getInitialSchedule(scf::ForOp forOp,
 252:                                   const DenseMap<Operation *, int> &opLatency) {
 253:   if (!isSafeToPipeline(forOp))
 254:     return CoarseSchedule(0);
```

- **EN:** Defines accessor/helper `getInitialSchedule` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getInitialSchedule`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 256-259

```cpp
 256:   // If the loop has assigned latencies, use them to determine the initial
 257:   // schedule.
 258:   if (hasLatenciesAssigned(forOp, opLatency))
 259:     return scheduleKeyOps(forOp, opLatency);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 261-275

```cpp
 261:   // If the loop has an existing schedule, use it as the base schedule.
 262:   CoarseSchedule schedule;
 263:   if (forOp->hasAttr(kWarpSpecializeAttrName) &&
 264:       succeeded(schedule.deSerialize(forOp))) {
 265:     // The loop was partitioned from a warp-specialized loop, meaning it can
 266:     // have a partial view of the original loop stages. Re-schedule the loop
 267:     // root at the stages of the latency ops to prune unnecessary stages.
 268:     auto isLatencyOp = [&](Operation &op) {
 269:       return opLatency.count(&op) ||
 270:              isa<LoadOp, DescriptorLoadLikeOpInterface, LocalStoreOp,
 271:                  LocalLoadOp, ttng::TMEMLoadOp, ttng::TMEMStoreOp,
 272:                  AsyncCopyGlobalToLocalOp, ttng::TMAOpInterface,
 273:                  ttng::MMAv5OpInterface, ttng::WaitBarrierOp,
 274:                  ttng::ArriveBarrierOp>(op);
 275:     };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 277-293

```cpp
 277:     // If there are no latency ops or all latency ops are in the same stage, we
 278:     // don't need to pipeline the loop. Return a new schedule with everything
 279:     // assigned to the same stage.
 280:     DenseSet<int> latencyStages;
 281:     auto ops = forOp.getBody()->without_terminator();
 282:     for (Operation &op : llvm::make_filter_range(ops, isLatencyOp)) {
 283:       // FIXME: This should assert all latency ops have an assigned stage.
 284:       if (schedule.count(&op))
 285:         latencyStages.insert(schedule[&op].first);
 286:     }
 287:     if (latencyStages.size() <= 1) {
 288:       CoarseSchedule normalized(/*numStages=*/1);
 289:       auto cluster = normalized.clusters.newAtFront();
 290:       for (Operation &op : ops)
 291:         normalized.insert(&op, 0, cluster);
 292:       return normalized;
 293:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 295-297

```cpp
 295:     schedule.shrinkToFit();
 296:     return schedule;
 297:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 299-300

```cpp
 299:   return CoarseSchedule(0);
 300: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 302-308

```cpp
 302: // Schedule the prologue and epilogue `if` ops in the loop, pushing them as
 303: // close to the loop boundaries as possible. Return the cluster after the
 304: // prologue (or the beginning of the loop if there is no prologue).
 305: CoarseSchedule::Cluster schedulePrologueAndEpilogue(scf::ForOp forOp,
 306:                                                     CoarseSchedule &schedule) {
 307:   int numStages = schedule.getNumStages();
 308:   CoarseSchedule::Cluster afterPrologue = schedule.clusters.begin();
```

- **EN:** Defines `schedulePrologueAndEpilogue`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `schedulePrologueAndEpilogue`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 310-322

```cpp
 310:   // Look for the IfOp that is in the backward slice any of the currently
 311:   // scheduled ops and put it at the beginning of the loop.
 312:   DenseMap<scf::IfOp, int> ifsToStage;
 313:   // Go stage by stage.
 314:   for (int stage = 0; stage < numStages; stage++) {
 315:     for (auto [op, stage_, cluster] : schedule.getOpsInOrder(forOp)) {
 316:       if (stage_ != stage)
 317:         continue;
 318:       SetVector<Operation *> backwardSlice;
 319:       BackwardSliceOptions opt;
 320:       opt.omitBlockArguments = true;
 321:       opt.omitUsesFromAbove = false;
 322:       (void)getBackwardSlice((Operation *)op, &backwardSlice, opt);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 324-336

```cpp
 324:       for (auto op : backwardSlice) {
 325:         if (auto ifOp = dyn_cast<scf::IfOp>(op)) {
 326:           ifsToStage.insert({ifOp, stage});
 327:         }
 328:       }
 329:     }
 330:   }
 331:   if (!ifsToStage.empty()) {
 332:     CoarseSchedule::Cluster prologueCluster = schedule.clusters.newAtFront();
 333:     for (auto [ifOp, stage] : ifsToStage) {
 334:       schedule.insertIfAbsent(ifOp, stage, prologueCluster);
 335:     }
 336:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 338-349

```cpp
 338:   // Other IfOps should be pushed to the end.
 339:   CoarseSchedule::Cluster epilogueCluster = schedule.clusters.newAtBack();
 340:   for (auto &op : forOp.getBody()->without_terminator()) {
 341:     if (auto ifOp = dyn_cast<scf::IfOp>(op)) {
 342:       if (ifsToStage.count(ifOp) == 0) {
 343:         schedule.insertIfAbsent(ifOp, numStages - 1,
 344:                                 epilogueCluster); // after prefetch extracts
 345:       }
 346:     }
 347:   }
 348:   return afterPrologue;
 349: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 351-368

```cpp
 351: void scheduleLoop(scf::ForOp forOp,
 352:                   const DenseMap<Operation *, int> &opLatency) {
 353:   // Based on the latencies, schedule the key ops to the stages.
 354:   CoarseSchedule schedule = getInitialSchedule(forOp, opLatency);
 355:   if (schedule.empty())
 356:     return;
 357:   LLVM_DEBUG({
 358:     schedule.serialize(forOp);
 359:     DBGS() << "Initial coarse schedule:\n" << forOp << "\n";
 360:   });
 361:   // Schedule the dependencies
 362:   CoarseSchedule::Cluster afterPrologue =
 363:       schedulePrologueAndEpilogue(forOp, schedule);
 364:   LLVM_DEBUG({
 365:     schedule.serialize(forOp);
 366:     DBGS() << "Coarse schedule with prologue and epilogue:\n" << forOp << "\n";
 367:   });
 368:   scheduleDependencies(forOp, schedule);
```

- **EN:** Defines debug logging helpers and compile-time tags used when tracing this pass or utility at runtime.
- **CN:** 这里定义调试日志辅助宏和编译期标签，用于在运行时跟踪该 pass 或工具。
### Lines 369-382

```cpp
 369:   LLVM_DEBUG({
 370:     schedule.serialize(forOp);
 371:     DBGS() << "Coarse schedule with dependencies:\n" << forOp << "\n";
 372:   });
 373:   scheduleDistanceOneDependencies(forOp, schedule);
 374:   LLVM_DEBUG({
 375:     schedule.serialize(forOp);
 376:     DBGS() << "Coarse schedule with dist 1:\n" << forOp << "\n";
 377:   });
 378:   scheduleRemainingToLastStage(forOp, schedule, afterPrologue);
 379:   LLVM_DEBUG({
 380:     schedule.serialize(forOp);
 381:     DBGS() << "Final coarse schedule:\n" << forOp << "\n";
 382:   });
```

- **EN:** Defines debug logging helpers and compile-time tags used when tracing this pass or utility at runtime.
- **CN:** 这里定义调试日志辅助宏和编译期标签，用于在运行时跟踪该 pass 或工具。
### Lines 384-386

```cpp
 384:   // Write the schedule to the IR
 385:   schedule.serialize(forOp);
 386: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 388-398

```cpp
 388: /// Schedule the loops based on the latencies assigned to the operations.
 389: void scheduleLoops(ModuleOp moduleOp) {
 390:   DenseMap<Operation *, int> opLatency = deserializeLatencies(moduleOp);
 391:   SmallVector<scf::ForOp> loops;
 392:   moduleOp->walk([&](scf::ForOp forOp) { loops.push_back(forOp); });
 393:   if (loops.empty())
 394:     return;
 395:   for (auto forOp : loops) {
 396:     scheduleLoop(forOp, opLatency);
 397:   }
 398: }
```

- **EN:** Defines `scheduleLoops`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. The implementation traverses IR operations to collect facts or apply rewrites globally. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `scheduleLoops`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 400-400

```cpp
 400: } // namespace
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 402-404

```cpp
 402: //===----------------------------------------------------------------------===//
 403: // Pass Definition
 404: //===----------------------------------------------------------------------===//
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 406-407

```cpp
 406: #define GEN_PASS_DEF_TRITONGPUSCHEDULELOOPS
 407: #include "triton/Dialect/TritonGPU/Transforms/Passes.h.inc"
```

- **EN:** Pulls in generated definitions from TableGen/MLIR include fragments so the handwritten code can reuse auto-generated declarations.
- **CN:** 这里引入由 TableGen/MLIR 生成的定义片段，使手写代码能够复用自动生成的声明。
### Lines 409-410

```cpp
 409: struct ScheduleLoops : public impl::TritonGPUScheduleLoopsBase<ScheduleLoops> {
 410:   using TritonGPUScheduleLoopsBase::TritonGPUScheduleLoopsBase;
```

- **EN:** Defines `ScheduleLoops`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `ScheduleLoops`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 412-413

```cpp
 412:   void runOnOperation() override { scheduleLoops(getOperation()); }
 413: };
```

- **EN:** Defines `runOnOperation`, the pass entry point. It gathers analysis information and applies the file's transformation logic to the current operation.
- **CN:** 这里定义 `runOnOperation`，即 pass 的入口函数。它会收集分析信息，并把本文件的变换逻辑应用到当前操作上。
### Lines 415-415

```cpp
 415: } // namespace mlir::triton::gpu
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。

## Key Concepts / 关键概念
- **EN:** The main concern is pass-driven transformation around schedule loops.
  **CN:** 核心关注点是围绕 Schedule Loops 的 pass 驱动变换。
- **EN:** The code reasons at module scope rather than only on isolated operations.
  **CN:** 代码在模块范围内进行分析，而不只是处理单个操作。
- **EN:** Tensor Memory Accelerator related logic appears in this implementation.
  **CN:** 此实现涉及 Tensor Memory Accelerator（TMA）相关逻辑。
- **EN:** Tensor memory specific allocation, layout, or synchronization rules are important here.
  **CN:** 这里重点处理张量内存（TMEM）的分配、布局或同步规则。
- **EN:** Synchronization and ordering constraints matter to preserve correctness across threads, warps, or memory spaces.
  **CN:** 同步与顺序约束很重要，它们保证跨线程、warp 或内存空间的正确性。

## Dependencies / 依赖关系
- **Internal Triton headers / Triton 内部头文件:** `triton/Analysis/Utility.h`, `triton/Dialect/TritonGPU/IR/Dialect.h`, `triton/Dialect/TritonGPU/Transforms/MMAv5PipelineUtility.h`, `triton/Dialect/TritonGPU/Transforms/Passes.h`, `triton/Dialect/TritonGPU/Transforms/PipeliningUtility.h`, `triton/Dialect/TritonGPU/Transforms/Schedule.h`, ... (+3 more)
- **MLIR headers / MLIR 头文件:** `mlir/IR/Dominance.h`
- **LLVM headers / LLVM 头文件:** `llvm/Support/Debug.h`
- **Standard/library headers / 标准或通用库头文件:** None
- **Generated fragments / 生成片段:** `triton/Dialect/TritonGPU/Transforms/Passes.h.inc`
- **Primary APIs used / 主要 API:** `ModuleOp`
- **Pipeline role / 流程角色:** This file participates in Triton pass pipelines and usually expects upstream analyses or dialect invariants to have prepared the IR. / 本文件参与 Triton 的 pass 流程，通常依赖上游分析或方言不变量先把 IR 准备好。
