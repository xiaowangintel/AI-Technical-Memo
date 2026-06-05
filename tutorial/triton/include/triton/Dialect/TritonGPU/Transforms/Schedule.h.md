# Schedule.h — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./include/triton/Dialect/TritonGPU/Transforms/Schedule.h`
- **EN:** Declares transformation support utilities centered on `Schedule`.
- **CN:** 声明围绕 `Schedule` 的变换支持工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```cpp
   1: #ifndef TRITON_TRITONGPU_TRANSFORM_PIPELINE_SCHEDULE_H_
   2: #define TRITON_TRITONGPU_TRANSFORM_PIPELINE_SCHEDULE_H_
```
**EN:** This block establishes the header guard so the declarations in this file are only processed once per translation unit.
**CN:** 该代码块建立头文件保护宏，避免同一翻译单元重复包含本文件时出现重复声明。

### Lines 4-11
```cpp
   4: #include "mlir/Dialect/SCF/IR/SCF.h"
   5: #include "mlir/IR/ImplicitLocOpBuilder.h"
   6: #include "mlir/Support/LLVM.h"
   7: #include "triton/Analysis/AxisInfo.h"
   8: #include "triton/Dialect/TritonGPU/Transforms/PipelineExpander.h"
   9: #include "llvm/ADT/ArrayRef.h"
  10: #include <list>
  11: #include <vector>
```
**EN:** This block imports the direct dependencies needed here, including mlir/Dialect/SCF/IR/SCF.h, mlir/IR/ImplicitLocOpBuilder.h, mlir/Support/LLVM.h, triton/Analysis/AxisInfo.h, triton/Dialect/TritonGPU/Transforms/PipelineExpander.h, and llvm/ADT/ArrayRef.h.
**CN:** 该代码块引入后续声明直接依赖的头文件，例如 mlir/Dialect/SCF/IR/SCF.h, mlir/IR/ImplicitLocOpBuilder.h, mlir/Support/LLVM.h, triton/Analysis/AxisInfo.h, triton/Dialect/TritonGPU/Transforms/PipelineExpander.h, and llvm/ADT/ArrayRef.h。

### Lines 13-14
```cpp
  13: namespace mlir {
  14: namespace triton {
```
**EN:** This block opens or closes the namespace scope used to organize APIs under mlir and triton.
**CN:** 该代码块打开或关闭命名空间作用域，用来把相关 API 组织在 mlir and triton 下。

### Lines 16-16
```cpp
  16: namespace gpu {
```
**EN:** This block opens or closes the namespace scope used to organize APIs under gpu.
**CN:** 该代码块打开或关闭命名空间作用域，用来把相关 API 组织在 gpu 下。

### Lines 18-19
```cpp
  18: /// Lower the loops to prepare them for pipeline expansion.
  19: void lowerLoops(ModuleOp moduleOp);
```
**EN:** This block declares or defines callable APIs such as lowerLoops, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 lowerLoops 等可调用 API，用来封装这里提供的核心行为。

### Lines 21-26
```cpp
  21: bool hasGpuBarriers(scf::ForOp forOp);
  22: bool isSafeToPipeline(scf::ForOp forOp);
  23: llvm::MapVector<Operation *, std::pair<int, Operation *>>
  24: loadOpsToIndirectionLevel(scf::ForOp forOp, bool pipelineWithoutDot,
  25:                           triton::ModuleAxisInfoAnalysis &axisInfoAnalysis,
  26:                           int numStages, bool filterSmall = true);
```
**EN:** This block declares or defines callable APIs such as hasGpuBarriers, isSafeToPipeline, and loadOpsToIndirectionLevel, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 hasGpuBarriers, isSafeToPipeline, and loadOpsToIndirectionLevel 等可调用 API，用来封装这里提供的核心行为。

### Lines 28-28
```cpp
  28: }; // namespace gpu
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 30-31
```cpp
  30: /// Pipeline the TMA stores in the loop.
  31: bool pipelineTMAStores(scf::ForOp forOp);
```
**EN:** This block declares or defines callable APIs such as pipelineTMAStores, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 pipelineTMAStores 等可调用 API，用来封装这里提供的核心行为。

### Lines 33-37
```cpp
  33: /// This does post-processing on the pipelined loop to try to pipeline wgmma
  34: /// ops.
  35: // TODO: this should be included as part of the pipeline but currently the wgmma
  36: // wait modeling is problematic.
  37: void asyncLaunchDots(scf::ForOp forOp);
```
**EN:** This block declares or defines callable APIs such as asyncLaunchDots, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 asyncLaunchDots 等可调用 API，用来封装这里提供的核心行为。

### Lines 39-41
```cpp
  39: /// Post process the pipelined loop by updating the wait ops with the right
  40: /// number of groups in flight.
  41: void updateWaits(ModuleOp module);
```
**EN:** This block declares or defines callable APIs such as updateWaits, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 updateWaits 等可调用 API，用来封装这里提供的核心行为。

### Lines 43-46
```cpp
  43: class CoarseSchedule {
  44: public:
  45:   class ClusterList {
  46:     std::list<int> orderClusters;
```
**EN:** This block introduces `CoarseSchedule`, the main class/struct defined here.
**CN:** 该代码块引入了此文件的核心类/结构体 `CoarseSchedule`。

### Lines 48-75
```cpp
  48:   public:
  49:     using iterator = decltype(orderClusters)::iterator;
  50:     using const_iterator = decltype(orderClusters)::const_iterator;
  51:     ClusterList() = default;
  52:     iterator begin() { return orderClusters.begin(); }
  53:     const_iterator begin() const { return orderClusters.begin(); }
  54:     iterator end() { return orderClusters.end(); }
  55:     const_iterator end() const { return orderClusters.end(); }
  56:     size_t size() const { return orderClusters.size(); }
  57:     void clear() { orderClusters.clear(); }
  58:     iterator newAtBack() {
  59:       orderClusters.push_back(orderClusters.size());
  60:       return std::prev(orderClusters.end());
  61:     }
  62:     iterator newAtFront() {
  63:       orderClusters.push_front(-1);
  64:       for (auto &clusterId : orderClusters) {
  65:         clusterId++;
  66:       }
  67:       return orderClusters.begin();
  68:     }
  69:     iterator newBefore(iterator cluster) {
  70:       auto ret = orderClusters.insert(cluster, *cluster);
  71:       for (auto &clusterId : llvm::make_range(cluster, orderClusters.end())) {
  72:         clusterId++;
  73:       }
  74:       return ret;
  75:     }
```
**EN:** This block declares or defines callable APIs such as ClusterList, begin, end, size, clear, newAtBack, push_back, and prev, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 ClusterList, begin, end, size, clear, newAtBack, push_back, and prev 等可调用 API，用来封装这里提供的核心行为。

### Lines 77-89
```cpp
  77:     bool isBefore(iterator a, iterator b) const {
  78:       if (a == b)
  79:         return false;
  80:       for (auto it = begin(); it != end(); ++it) {
  81:         if (it == a)
  82:           return true;
  83:         if (it == b)
  84:           return false;
  85:       }
  86:       llvm::report_fatal_error(
  87:           "One or both clusters not found in clusters list!");
  88:     }
  89:   };
```
**EN:** This block declares or defines callable APIs such as isBefore, begin, end, and report_fatal_error, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isBefore, begin, end, and report_fatal_error 等可调用 API，用来封装这里提供的核心行为。

### Lines 91-95
```cpp
  91:   CoarseSchedule() = default;
  92:   CoarseSchedule(int numStages) : numStages(numStages) {}
  93:   ClusterList clusters;
  94:   using Cluster = ClusterList::iterator;
  95:   using ClusterHash = size_t;
```
**EN:** This block declares or defines callable APIs such as CoarseSchedule and numStages, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 CoarseSchedule and numStages 等可调用 API，用来封装这里提供的核心行为。

### Lines 97-97
```cpp
  97:   llvm::MapVector<Operation *, std::pair<int, Cluster>> opToStageAndCluster;
```
**EN:** This block stores supporting state such as opToStageAndCluster, which other APIs in the file consume.
**CN:** 该代码块声明了 opToStageAndCluster 等支撑状态，供本文件中的其他 API 使用。

### Lines 99-100
```cpp
  99:   void setNumStages(int numStages) { this->numStages = numStages; }
 100:   int getNumStages() const { return numStages; }
```
**EN:** This block declares or defines callable APIs such as setNumStages and getNumStages, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 setNumStages and getNumStages 等可调用 API，用来封装这里提供的核心行为。

### Lines 102-107
```cpp
 102:   void insert(Operation *op, int stage, Cluster cluster) {
 103:     if (stage >= numStages) {
 104:       numStages = stage + 1;
 105:     }
 106:     opToStageAndCluster[op] = {stage, cluster};
 107:   }
```
**EN:** This block declares or defines callable APIs such as insert, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 insert 等可调用 API，用来封装这里提供的核心行为。

### Lines 109-114
```cpp
 109:   bool insertIfAbsent(Operation *op, int stage, Cluster cluster) {
 110:     if (opToStageAndCluster.count(op))
 111:       return false;
 112:     insert(op, stage, cluster);
 113:     return true;
 114:   }
```
**EN:** This block declares or defines callable APIs such as insertIfAbsent, count, and insert, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 insertIfAbsent, count, and insert 等可调用 API，用来封装这里提供的核心行为。

### Lines 116-116
```cpp
 116:   bool insertMinimum(Operation *op, int stage, Cluster cluster);
```
**EN:** This block declares or defines callable APIs such as insertMinimum, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 insertMinimum 等可调用 API，用来封装这里提供的核心行为。

### Lines 118-119
```cpp
 118:   bool insertDepsOfOp(Operation *op, int stage, CoarseSchedule::Cluster cluster,
 119:                       bool includeArg, bool insertIfEarlier = false);
```
**EN:** This block declares or defines callable APIs such as insertDepsOfOp, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 insertDepsOfOp 等可调用 API，用来封装这里提供的核心行为。

### Lines 121-123
```cpp
 121:   // Remove empty stages and clusters from the schedule, adjusting the maximum
 122:   // number of stages as appropriate.
 123:   void shrinkToFit();
```
**EN:** This block declares or defines callable APIs such as shrinkToFit, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 shrinkToFit 等可调用 API，用来封装这里提供的核心行为。

### Lines 125-125
```cpp
 125:   void erase(Operation *op) { opToStageAndCluster.erase(op); }
```
**EN:** This block declares or defines callable APIs such as erase, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 erase 等可调用 API，用来封装这里提供的核心行为。

### Lines 127-127
```cpp
 127:   int count(Operation *op) const { return opToStageAndCluster.count(op); }
```
**EN:** This block declares or defines callable APIs such as count, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 count 等可调用 API，用来封装这里提供的核心行为。

### Lines 129-131
```cpp
 129:   std::pair<int, Cluster> operator[](Operation *op) {
 130:     return opToStageAndCluster[op];
 131:   }
```
**EN:** This block stores supporting state such as opToStageAndCluster, which other APIs in the file consume.
**CN:** 该代码块声明了 opToStageAndCluster 等支撑状态，供本文件中的其他 API 使用。

### Lines 133-133
```cpp
 133:   auto find(Operation *op) const { return opToStageAndCluster.find(op); }
```
**EN:** This block declares or defines callable APIs such as find, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 find 等可调用 API，用来封装这里提供的核心行为。

### Lines 135-138
```cpp
 135:   // Split the cluster containing op into two clusters, one containing all
 136:   // operations before the op and one containing op and all operations after the
 137:   // op. Return the cluster containing op and all operations after the op.
 138:   Cluster splitClusterBefore(Operation *op, scf::ForOp forOp);
```
**EN:** This block declares or defines callable APIs such as splitClusterBefore, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 splitClusterBefore 等可调用 API，用来封装这里提供的核心行为。

### Lines 140-141
```cpp
 140:   // Check if op a will show up before op b in the final unrolled code.
 141:   bool isOpBefore(Operation *a, Operation *b) const;
```
**EN:** This block declares or defines callable APIs such as isOpBefore, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isOpBefore 等可调用 API，用来封装这里提供的核心行为。

### Lines 143-144
```cpp
 143:   // Check if op a is in earlier cluster than op b.
 144:   bool isOpInEarlierCluster(Operation *a, Operation *b) const;
```
**EN:** This block declares or defines callable APIs such as isOpInEarlierCluster, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isOpInEarlierCluster 等可调用 API，用来封装这里提供的核心行为。

### Lines 146-147
```cpp
 146:   // Check if op a is in the same cluster as op b.
 147:   bool isOpInSameCluster(Operation *a, Operation *b) const;
```
**EN:** This block declares or defines callable APIs such as isOpInSameCluster, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isOpInSameCluster 等可调用 API，用来封装这里提供的核心行为。

### Lines 149-152
```cpp
 149:   SmallVector<std::tuple<Operation *, int, Cluster>>
 150:   getOpsInOrder(scf::ForOp forOp) const;
 151:   std::vector<std::pair<Operation *, unsigned>>
 152:   createFinalSchedule(scf::ForOp forOp) const;
```
**EN:** This block declares or defines callable APIs such as getOpsInOrder and createFinalSchedule, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getOpsInOrder and createFinalSchedule 等可调用 API，用来封装这里提供的核心行为。

### Lines 154-156
```cpp
 154:   bool empty() const { return opToStageAndCluster.size() == 0; }
 155:   auto end() const { return opToStageAndCluster.end(); }
 156:   auto begin() const { return opToStageAndCluster.begin(); }
```
**EN:** This block declares or defines callable APIs such as empty, size, end, and begin, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 empty, size, end, and begin 等可调用 API，用来封装这里提供的核心行为。

### Lines 158-164
```cpp
 158:   // Set <stage, cluster> based on CoarseSchedule.
 159:   void serialize(scf::ForOp &forOp) const;
 160:   // Create a CoarseSchedule based on forOp's <stage, cluster>.
 161:   // If normalizeClusterId is true, clusters [minClusterId, maxClusterId] will
 162:   // be remapped to [0, maxClusterId - minClusterId].
 163:   // If false, it won't remap and clusters [0, maxClusterId] will be created.
 164:   LogicalResult deSerialize(scf::ForOp &forOp, bool normalizeClusterId = true);
```
**EN:** This block declares or defines callable APIs such as serialize and deSerialize, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 serialize and deSerialize 等可调用 API，用来封装这里提供的核心行为。

### Lines 166-168
```cpp
 166:   static ClusterHash hashCluster(Cluster cluster) {
 167:     return reinterpret_cast<ClusterHash>(&*cluster);
 168:   }
```
**EN:** This block declares or defines callable APIs such as hashCluster, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 hashCluster 等可调用 API，用来封装这里提供的核心行为。

### Lines 170-170
```cpp
 170:   LLVM_DUMP_METHOD void dump();
```
**EN:** This block declares or defines callable APIs such as dump, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 dump 等可调用 API，用来封装这里提供的核心行为。

### Lines 172-174
```cpp
 172:   // ============================================================
 173:   // Linearized Schedule Iterator API
 174:   // ============================================================
```
**EN:** This comment block records the intent and constraints of the surrounding code: ============================================================ Linearized Schedule Iterator API ============================================================.
**CN:** 该注释块解释了周边代码的设计背景、意图或约束条件，帮助读者理解后续实现。

### Lines 176-191
```cpp
 176:   /// A stateful iterator over operations in linearized schedule order.
 177:   /// Operations are yielded lazily in order: (stage, cluster,
 178:   /// IR-order-within-cluster).
 179:   ///
 180:   /// The iterator is circular and stage-aware: it starts from initialOp at its
 181:   /// stage, traverses to the end of clusters, wraps around to the beginning,
 182:   /// and when it reaches initialOp again, increments the stage limit. An op is
 183:   /// only yielded if its stage <= currStageLimit. The iterator stops when it
 184:   /// reaches initialOp and currStageLimit >= numStages.
 185:   class LinearizedIterator {
 186:   public:
 187:     /// Construct an iterator for the given forOp and schedule.
 188:     /// The iterator starts at initialOp and wraps around circularly with
 189:     /// stage-based filtering.
 190:     LinearizedIterator(scf::ForOp forOp, const CoarseSchedule &schedule,
 191:                        Operation *initialOp);
```
**EN:** This block introduces `LinearizedIterator`, the main class/struct defined here.
**CN:** 该代码块引入了此文件的核心类/结构体 `LinearizedIterator`。

### Lines 193-198
```cpp
 193:     // Standard iterator operations
 194:     LinearizedIterator &operator++();
 195:     LinearizedIterator operator++(int);
 196:     Operation *operator*() const;
 197:     bool operator==(const LinearizedIterator &other) const;
 198:     bool operator!=(const LinearizedIterator &other) const;
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 200-200
```cpp
 200:     bool isEnd() const { return atEnd; }
```
**EN:** This block declares or defines callable APIs such as isEnd, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isEnd 等可调用 API，用来封装这里提供的核心行为。

### Lines 202-215
```cpp
 202:     /// Advance the iterator to the next operation that satisfies the optional
 203:     /// predicate. Returns the found operation, or std::nullopt if not found.
 204:     /// The iterator position is updated to the found operation (or end).
 205:     std::optional<Operation *>
 206:     findNext(std::function<bool(Operation *)> predicate = nullptr) {
 207:       while (!isEnd()) {
 208:         Operation *op = *(*this);
 209:         ++(*this);
 210:         if (!predicate || predicate(op)) {
 211:           return op;
 212:         }
 213:       }
 214:       return std::nullopt;
 215:     }
```
**EN:** This block declares or defines callable APIs such as findNext, bool, isEnd, and predicate, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 findNext, bool, isEnd, and predicate 等可调用 API，用来封装这里提供的核心行为。

### Lines 217-219
```cpp
 217:   private:
 218:     /// Advance to the next valid operation in the schedule.
 219:     void advanceToNextScheduledOp();
```
**EN:** This block declares or defines callable APIs such as advanceToNextScheduledOp, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 advanceToNextScheduledOp 等可调用 API，用来封装这里提供的核心行为。

### Lines 221-233
```cpp
 221:     scf::ForOp forOp;
 222:     const CoarseSchedule *schedule;
 223:     ClusterList::const_iterator clusterIt;
 224:     ClusterList::const_iterator clusterBegin;
 225:     ClusterList::const_iterator clusterEnd;
 226:     Block::iterator opIt;
 227:     Block::iterator opEnd;
 228:     Operation *currentOp = nullptr;
 229:     Operation *initialOp = nullptr;
 230:     int currStageLimit = 0;
 231:     int maxStages = 0;
 232:     bool atEnd = false;
 233:   };
```
**EN:** This block stores supporting state such as forOp, schedule, clusterIt, clusterBegin, clusterEnd, and opIt, which other APIs in the file consume.
**CN:** 该代码块声明了 forOp, schedule, clusterIt, clusterBegin, clusterEnd, and opIt 等支撑状态，供本文件中的其他 API 使用。

### Lines 235-240
```cpp
 235:   /// Get a circular iterator over the linearized schedule starting from
 236:   /// initialOp. The iterator will traverse from initialOp to the end, wrap
 237:   /// around to the beginning, and stop when it reaches initialOp again.
 238:   LinearizedIterator linearized(scf::ForOp forOp, Operation *initialOp) const {
 239:     return LinearizedIterator(forOp, *this, initialOp);
 240:   }
```
**EN:** This block declares or defines callable APIs such as linearized and LinearizedIterator, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 linearized and LinearizedIterator 等可调用 API，用来封装这里提供的核心行为。

### Lines 242-244
```cpp
 242: private:
 243:   int numStages = 0;
 244: };
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 246-248
```cpp
 246: // Add dependencies of anchor ops to the coarse schedule. Schedule them to
 247: // the same stage and ordering cluster as the anchor op.
 248: void scheduleDependencies(scf::ForOp forOp, CoarseSchedule &schedule);
```
**EN:** This block declares or defines callable APIs such as scheduleDependencies, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 scheduleDependencies 等可调用 API，用来封装这里提供的核心行为。

### Lines 250-258
```cpp
 250: class OpBuilderForStage : public mlir::ImplicitLocOpBuilder,
 251:                           public OpBuilder::Listener {
 252: public:
 253:   explicit OpBuilderForStage(Location loc, Operation *op,
 254:                              CoarseSchedule &schedule)
 255:       : ImplicitLocOpBuilder(loc, op, this), schedule(schedule) {
 256:     if (auto it = schedule.find(op); it != schedule.end())
 257:       std::tie(stage, cluster) = it->second;
 258:   }
```
**EN:** This block introduces `OpBuilderForStage`, the main class/struct defined here. Within the declaration, methods such as ImplicitLocOpBuilder, schedule, find, end, and tie expose its core API. It also inherits behavior from a base type.
**CN:** 该代码块引入了此文件的核心类/结构体 `OpBuilderForStage`。 其中 ImplicitLocOpBuilder, schedule, find, end, and tie 等方法构成了它的主要接口。 它还通过继承复用基类能力。

### Lines 260-263
```cpp
 260:   void setStageCluster(std::pair<int, CoarseSchedule::Cluster> stageCluster) {
 261:     stage = stageCluster.first;
 262:     cluster = stageCluster.second;
 263:   }
```
**EN:** This block declares or defines callable APIs such as setStageCluster, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 setStageCluster 等可调用 API，用来封装这里提供的核心行为。

### Lines 265-268
```cpp
 265:   void notifyOperationInserted(Operation *op, InsertPoint previous) {
 266:     if (stage && cluster)
 267:       schedule.insert(op, *stage, *cluster);
 268:   }
```
**EN:** This block declares or defines callable APIs such as notifyOperationInserted and insert, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 notifyOperationInserted and insert 等可调用 API，用来封装这里提供的核心行为。

### Lines 270-274
```cpp
 270: private:
 271:   std::optional<int> stage;
 272:   std::optional<CoarseSchedule::Cluster> cluster;
 273:   CoarseSchedule &schedule;
 274: };
```
**EN:** This block stores supporting state such as stage, cluster, and schedule, which other APIs in the file consume.
**CN:** 该代码块声明了 stage, cluster, and schedule 等支撑状态，供本文件中的其他 API 使用。

### Lines 276-281
```cpp
 276: namespace gpu {
 277: void scheduleDistanceOneDependencies(scf::ForOp forOp,
 278:                                      CoarseSchedule &schedule);
 279: void scheduleRemainingToLastStage(scf::ForOp forOp, CoarseSchedule &schedule,
 280:                                   CoarseSchedule::Cluster afterPrologue);
 281: } // namespace gpu
```
**EN:** This block declares or defines callable APIs such as scheduleDistanceOneDependencies and scheduleRemainingToLastStage, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 scheduleDistanceOneDependencies and scheduleRemainingToLastStage 等可调用 API，用来封装这里提供的核心行为。

### Lines 283-285
```cpp
 283: } // namespace triton
 284: } // namespace mlir
 285: #endif // TRITON_TRITONGPU_TRANSFORM_PIPELINE_SCHEDULE_H_
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

## Key Concepts / 关键概念
- **EN:** axis reasoning  
  **CN:** 轴信息推理
- **EN:** dot-product lowering  
  **CN:** 点积降级
- **EN:** LLVM lowering  
  **CN:** LLVM 降级
- **EN:** MLIR dialect definition  
  **CN:** MLIR 方言定义
- **EN:** operation definitions  
  **CN:** 操作定义
- **EN:** software pipelining  
  **CN:** 软件流水化
- **EN:** scheduling  
  **CN:** 调度
- **EN:** MLIR integration  
  **CN:** MLIR 集成

## Dependencies / 依赖关系
- **Project includes / 项目内依赖:**
  - `mlir/Dialect/SCF/IR/SCF.h`
  - `mlir/IR/ImplicitLocOpBuilder.h`
  - `mlir/Support/LLVM.h`
  - `triton/Analysis/AxisInfo.h`
  - `triton/Dialect/TritonGPU/Transforms/PipelineExpander.h`
  - `llvm/ADT/ArrayRef.h`
- **System or external includes / 系统或外部依赖:**
  - `<list>`
  - `<vector>`
