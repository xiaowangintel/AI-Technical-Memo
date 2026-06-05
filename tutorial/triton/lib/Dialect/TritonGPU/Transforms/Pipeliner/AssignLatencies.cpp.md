# AssignLatencies.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `lib/Dialect/TritonGPU/Transforms/Pipeliner/AssignLatencies.cpp`
- **Purpose / 作用:** **EN:** Implements the Assign Latencies transformation or optimization pass for the TritonGPU pipeline. **CN:** 为 TritonGPU 编译流程实现与 Assign Latencies 相关的变换或优化 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-10

```cpp
   1: #include "triton/Analysis/AxisInfo.h"
   2: #include "triton/Dialect/TritonGPU/IR/Dialect.h"
   3: #include "triton/Dialect/TritonGPU/Transforms/MMAv5PipelineUtility.h"
   4: #include "triton/Dialect/TritonGPU/Transforms/Passes.h"
   5: #include "triton/Dialect/TritonGPU/Transforms/PipeliningUtility.h"
   6: #include "triton/Dialect/TritonGPU/Transforms/Schedule.h"
   7: #include "triton/Dialect/TritonGPU/Transforms/Utility.h"
   8: #include "triton/Dialect/TritonNvidiaGPU/IR/Dialect.h"
   9: #include "triton/Tools/Sys/GetEnv.h"
  10: #include "llvm/Support/Debug.h"
```

- **EN:** Includes the interfaces this file depends on. Triton headers (`AxisInfo.h`, `Dialect.h`, `MMAv5PipelineUtility.h`, `Passes.h`, ... (+5 more)) provide domain-specific IR/support, MLIR headers (None) provide rewriting and analysis infrastructure, LLVM headers (`Debug.h`) supply low-level utilities, and standard/library headers (None) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（`AxisInfo.h`, `Dialect.h`, `MMAv5PipelineUtility.h`, `Passes.h`, ... (+5 more)）提供领域特定 IR/支持逻辑，MLIR 头文件（None）提供重写与分析基础设施，LLVM 头文件（`Debug.h`）提供底层工具，而标准/通用库头文件（None）提供通用能力。
### Lines 12-14

```cpp
  12: #define DEBUG_TYPE "triton-loop-pipeline"
  13: #define DBGS() (llvm::dbgs() << "[" DEBUG_TYPE "]: ")
  14: #define LDBG(X) LLVM_DEBUG(DBGS() << X << "\n")
```

- **EN:** Defines debug logging helpers and compile-time tags used when tracing this pass or utility at runtime.
- **CN:** 这里定义调试日志辅助宏和编译期标签，用于在运行时跟踪该 pass 或工具。
### Lines 16-19

```cpp
  16: using namespace mlir;
  17: namespace tt = mlir::triton;
  18: namespace ttg = mlir::triton::gpu;
  19: namespace ttng = mlir::triton::nvidia_gpu;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 21-22

```cpp
  21: namespace mlir::triton::gpu {
  22: namespace {
```

- **EN:** Opens or closes the namespace nesting for mlir::triton::gpu -> (anonymous), keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 mlir::triton::gpu -> (anonymous) 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 24-26

```cpp
  24: //===----------------------------------------------------------------------===//
  25: // assignLatencies
  26: //===----------------------------------------------------------------------===//
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 28-38

```cpp
  28: // Return true if the preconditions for pipelining the loop are met.
  29: bool preCondition(scf::ForOp forOp) {
  30:   // Skip loop with distance > 1 for now.
  31:   // TODO: relax the constraint in the expander.
  32:   if (loopHasDistGreaterThanOne(forOp))
  33:     return false;
  34:   // Don't pipeline outer loops.
  35:   if (isOuterLoop(forOp))
  36:     return false;
  37:   return true;
  38: }
```

- **EN:** Defines `preCondition`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `preCondition`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 40-47

```cpp
  40: bool hasLatenciesAssigned(scf::ForOp forOp) {
  41:   auto helper = TritonDialect::getLoaded(forOp)->getLatencyAttrHelper();
  42:   for (auto &op : forOp.getBody()->without_terminator()) {
  43:     if (helper.getAttr(&op))
  44:       return true;
  45:   }
  46:   return false;
  47: }
```

- **EN:** Defines `hasLatenciesAssigned`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `hasLatenciesAssigned`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 49-57

```cpp
  49: void assignUserProvidedLatencies(scf::ForOp forOp,
  50:                                  DenseMap<Operation *, int> &opLatency) {
  51:   auto helper = TritonDialect::getLoaded(forOp)->getLatencyAttrHelper();
  52:   for (auto &op : forOp.getBody()->without_terminator()) {
  53:     if (auto latencyAttr = helper.getAttr(&op)) {
  54:       opLatency[&op] = latencyAttr.getInt();
  55:     }
  56:   }
  57: }
```

- **EN:** Defines `assignUserProvidedLatencies`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `assignUserProvidedLatencies`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 59-63

```cpp
  59: class AssignLoadLatencies {
  60: public:
  61:   AssignLoadLatencies(scf::ForOp forOp, int numStages,
  62:                       DenseMap<Operation *, int> &opLatency)
  63:       : forOp(forOp), numStages(numStages), opLatency(opLatency) {};
```

- **EN:** Defines `AssignLoadLatencies`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `AssignLoadLatencies`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 65-68

```cpp
  65:   void run() {
  66:     bool pipelineWithoutDot = forOp->hasAttr(mlir::triton::kNumStagesAttrName);
  67:     ModuleOp moduleOp = forOp->getParentOfType<ModuleOp>();
  68:     tt::ModuleAxisInfoAnalysis axisInfoAnalysis(moduleOp);
```

- **EN:** Defines `run`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `run`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 70-74

```cpp
  70:     llvm::MapVector<Operation *, std::pair<int, Operation *>> loadOpToIndLevel =
  71:         loadOpsToIndirectionLevel(forOp, pipelineWithoutDot, axisInfoAnalysis,
  72:                                   numStages);
  73:     if (loadOpToIndLevel.empty())
  74:       return;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 76-80

```cpp
  76:     // Calculate the stage distance between applicable loads.
  77:     int maxIndirectionLevel = 0;
  78:     for (auto &[loadOp, info] : loadOpToIndLevel)
  79:       maxIndirectionLevel = std::max(maxIndirectionLevel, info.first);
  80:     unsigned loadLatency = (numStages - 1) / (maxIndirectionLevel + 1);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 82-85

```cpp
  82:     for (auto [loadOp, dist] : loadOpToIndLevel) {
  83:       opLatency[loadOp] = loadLatency;
  84:     }
  85:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 87-90

```cpp
  87: private:
  88:   scf::ForOp forOp;
  89:   int numStages;
  90:   DenseMap<Operation *, int> &opLatency;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 92-98

```cpp
  92: public:
  93:   static bool canHaveSharedEncoding(tt::LoadOp op) {
  94:     // If used by an user with DotOp encoding, all the uses must be compatible.
  95:     bool incompatible = false;
  96:     getSharedEncIfAllUsersAreDotEnc(op.getResult(), incompatible);
  97:     return !incompatible;
  98:   }
```

- **EN:** Defines `canHaveSharedEncoding`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `canHaveSharedEncoding`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 100-115

```cpp
 100:   static bool
 101:   isPipeliningBeneficial(Operation *op, Operation *finalUser,
 102:                          tt::ModuleAxisInfoAnalysis &axisInfoAnalysis,
 103:                          bool filterSmall) {
 104:     if (auto loadOp = dyn_cast<tt::LoadOp>(op)) {
 105:       if (filterSmall && !canBeConvertedToAsyncLoad(loadOp, axisInfoAnalysis)) {
 106:         LDBG("Load " << *loadOp << " is too small for pipelining");
 107:         return false;
 108:       }
 109:     }
 110:     if (isa<tt::DescriptorLoadLikeOpInterface>(op))
 111:       return true;
 112:     if (!canHaveSharedEncoding(cast<tt::LoadOp>(op))) {
 113:       LDBG("Load " << *op << " cannot have shared encoding");
 114:       return false;
 115:     }
```

- **EN:** Defines `isPipeliningBeneficial`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `isPipeliningBeneficial`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 117-134

```cpp
 117:     ttg::SharedEncodingTrait localAllocEnc;
 118:     if (llvm::any_of(op->getUsers(), [&](Operation *user) {
 119:           return isa<ttg::LocalAllocOp>(user);
 120:         })) {
 121:       for (auto user : op->getUsers()) {
 122:         auto localAlloc = dyn_cast<ttg::LocalAllocOp>(user);
 123:         if (!localAlloc)
 124:           continue;
 125:         auto enc = mlir::cast<ttg::SharedEncodingTrait>(
 126:             localAlloc.getType().getEncoding());
 127:         if (!localAllocEnc) {
 128:           localAllocEnc = enc;
 129:         }
 130:         if (enc != localAllocEnc) {
 131:           // If the load is used by a LocalAllocOp, all the users need to have
 132:           // the same encoding.
 133:           return false;
 134:         }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 135-136

```cpp
 135:       }
 136:     }
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 138-145

```cpp
 138:     if (localAllocEnc) {
 139:       auto registerTy = cast<RankedTensorType>(op->getResultTypes()[0]);
 140:       auto vecBytes = getCopyVecBytes(registerTy, localAllocEnc);
 141:       if (filterSmall && vecBytes < 4) {
 142:         // At least 4 bytes need to be consecutive for cp.async
 143:         return false;
 144:       }
 145:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 147-149

```cpp
 147:     return true;
 148:   }
 149: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 151-154

```cpp
 151: class AssignMMALatencies {
 152: public:
 153:   AssignMMALatencies(scf::ForOp forOp, DenseMap<Operation *, int> &opLatency)
 154:       : forOp(forOp), opLatency(opLatency) {};
```

- **EN:** Defines `AssignMMALatencies`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `AssignMMALatencies`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 156-168

```cpp
 156:   void run() {
 157:     DenseMap<Operation *, int> mmaSelfLatency;
 158:     // Check if the load op (mma operand) is pipelineable.
 159:     auto isLoadToBePipelined = [&](Operation *op) {
 160:       return opLatency.count(op) && opLatency[op] > 0;
 161:     };
 162:     for (auto &op : forOp.getBody()->without_terminator()) {
 163:       // If the acc can not be multibuffered, do not pipeline the uses of
 164:       // the MMA to later stages.
 165:       if (auto mma = dyn_cast<ttng::MMAv5OpInterface>(&op)) {
 166:         // Try to push out the wait by one stage even if the operands are not
 167:         // pipelineable, but we know where the loads are scheduled, so we can
 168:         // place the wait right before the loads.
```

- **EN:** Defines `run`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `run`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 170-187

```cpp
 170:         if (hasSyncDots(forOp)) {
 171:           // Skip pipelining MMA in the loops where sync dots are used. This
 172:           // is a dirty heuristic for performance drops in kernels where we
 173:           // would rather want to have last iteration peeled instead of having a
 174:           // full iteration of masked operations only to execute single wait.
 175:           continue;
 176:         }
 177:         auto pipeHelper = ttng::MMAv5PipelineableOperandsHelper(
 178:             mma, forOp, isLoadToBePipelined);
 179:         if (pipeHelper.isPipelineable ||
 180:             (pipeHelper.isOperandsStateDetermined &&
 181:              !ttng::hasLoadsAfterMMA(mma, forOp))) {
 182:           // MMA can be overlapped with itself
 183:           mmaSelfLatency[mma] = 1;
 184:           if (!ttng::requiresAccMultiBuffering(mma, forOp) ||
 185:               (ttng::isAccMultibufferingPossible(mma, forOp) &&
 186:                !getDisallowAccMultiBuffer(forOp))) {
 187:             // MMA's users can be pushed to the next stage
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 188-204

```cpp
 188:             opLatency[&op] = 1;
 189:           }
 190:           // HACK: A pipelined MMA's latency should equal the number of buffers
 191:           // for the accumulator, but when the user is in an `scf.if` in SWP,
 192:           // the `scf.if` is pushed to the end of the loop rather than peeled
 193:           // before the MMA op, requiring an extra buffer due to liverange
 194:           // overlap. WS does not have this problem because the MMA is placed in
 195:           // a different partition than the MMA, so we can correctly set the
 196:           // latency.
 197:           if (isWarpSpecialized(forOp)) {
 198:             if (ttng::hasAccReadModifyWrite(mma, forOp))
 199:               opLatency.erase(&op); // can't pipeline the MMA
 200:             else
 201:               opLatency[&op] += 1;
 202:             // If all inputs to the MMA are warp specialized, set the self
 203:             // latency to 0 since the MMA won't need to wait on itself.
 204:             auto cantWarpSpec = [](Operation *op) { return isa<LoadOp>(op); };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 205-219

```cpp
 205:             auto warpSpecHelper = ttng::MMAv5PipelineableOperandsHelper(
 206:                 mma, forOp, [&](Operation *op) {
 207:                   return isLoadToBePipelined(op) && !cantWarpSpec(op);
 208:                 });
 209:             if (warpSpecHelper.isPipelineable ||
 210:                 (warpSpecHelper.isOperandsStateDetermined &&
 211:                  llvm::none_of(warpSpecHelper.unpipelineableOperandDefs,
 212:                                cantWarpSpec)))
 213:               mmaSelfLatency[mma] = 0;
 214:           }
 215:         }
 216:       }
 217:     }
 218:     serializeSelfLatencies(forOp->getParentOfType<ModuleOp>(), mmaSelfLatency);
 219:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 221-223

```cpp
 221: private:
 222:   scf::ForOp forOp;
 223:   DenseMap<Operation *, int> &opLatency;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 225-231

```cpp
 225:   bool hasSyncDots(scf::ForOp forOp) {
 226:     for (auto &op : forOp.getBody()->without_terminator()) {
 227:       if (isa<mlir::triton::DotOp>(op))
 228:         return true;
 229:     }
 230:     return false;
 231:   }
```

- **EN:** Defines `hasSyncDots`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `hasSyncDots`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 233-243

```cpp
 233:   bool isWarpSpecialized(scf::ForOp forOp) {
 234:     scf::ForOp current = forOp;
 235:     do {
 236:       if (current->hasAttr(kWarpSpecializeAttrName)) {
 237:         return true;
 238:       }
 239:       current = current->getParentOfType<scf::ForOp>();
 240:     } while (current);
 241:     return false;
 242:   };
 243: };
```

- **EN:** Defines `isWarpSpecialized`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `isWarpSpecialized`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 245-261

```cpp
 245: // Discover operations that should become async and assign latencies to them
 246: // based on the numStages value provided by the user.
 247: //
 248: // Look for load ops that directly or indirectly feed into dot ops. Based on the
 249: // requested number of stages assign the latencies in a way that cover all the
 250: // stages with the sum of latencies in the chain from the first load to the
 251: // final dot op.
 252: void assignLatencies(ModuleOp moduleOp, int defaultNumStages) {
 253:   SmallVector<scf::ForOp> loops;
 254:   moduleOp->walk([&](scf::ForOp forOp) {
 255:     // Bail out for loops with num_stage <= 1.
 256:     if (preCondition(forOp) &&
 257:         getNumStagesOrDefault(forOp, defaultNumStages) > 1)
 258:       loops.push_back(forOp);
 259:   });
 260:   if (loops.empty())
 261:     return;
```

- **EN:** Defines `assignLatencies`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. The implementation traverses IR operations to collect facts or apply rewrites globally.
- **CN:** 这里定义 `assignLatencies`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。
### Lines 263-274

```cpp
 263:   DenseMap<Operation *, int> opLatency;
 264:   for (auto forOp : loops) {
 265:     if (hasLatenciesAssigned(forOp)) {
 266:       assignUserProvidedLatencies(forOp, opLatency);
 267:       continue;
 268:     }
 269:     int numStages = getNumStagesOrDefault(forOp, defaultNumStages);
 270:     AssignLoadLatencies(forOp, numStages, opLatency).run();
 271:     AssignMMALatencies(forOp, opLatency).run();
 272:   }
 273:   serializeLatencies(moduleOp, opLatency);
 274: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 276-276

```cpp
 276: } // namespace
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 278-288

```cpp
 278: // Create a map from load ops to their indirection level and the
 279: // final use of the load op (another load op, or a dot op).
 280: // Indirection level is "0" for the load op directly used by the dot op,
 281: // "1" for the load op used by the load op used by the dot op, and so on.
 282: llvm::MapVector<Operation *, std::pair<int, Operation *>>
 283: loadOpsToIndirectionLevel(scf::ForOp forOp, bool pipelineWithoutDot,
 284:                           tt::ModuleAxisInfoAnalysis &axisInfoAnalysis,
 285:                           int numStages, bool filterSmall) {
 286:   llvm::MapVector<Operation *, std::pair<int, Operation *>> loadOpToIndLevel;
 287:   DenseSet<Operation *> seen;
 288:   DenseSet<Operation *> excluded;
```

- **EN:** Defines `loadOpsToIndirectionLevel`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `loadOpsToIndirectionLevel`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 290-307

```cpp
 290:   std::function<void(Operation *, Operation *, int)> dfs =
 291:       [&](Operation *op, Operation *finalUser, int distance) {
 292:         if (!seen.insert(op).second || excluded.count(op))
 293:           return;
 294:         if (isa<tt::LoadOp, tt::DescriptorLoadLikeOpInterface>(op)) {
 295:           if (!AssignLoadLatencies::isPipeliningBeneficial(
 296:                   op, finalUser, axisInfoAnalysis, filterSmall))
 297:             return;
 298:           if (loadOpToIndLevel.count(op)) {
 299:             int level = loadOpToIndLevel[op].first;
 300:             if (level != distance) {
 301:               // If we have multiple uses at different distances, we don't
 302:               // know which one to pick.
 303:               LDBG("Load " << *op
 304:                            << " has multiple uses at different distances:"
 305:                            << level << " and " << distance);
 306:               loadOpToIndLevel.erase(op);
 307:               excluded.insert(op);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 308-325

```cpp
 308:               return;
 309:             }
 310:           } else {
 311:             LDBG("Load " << *op << " considered for pipelining with distance "
 312:                          << distance);
 313:             loadOpToIndLevel[op] = {distance, finalUser};
 314:           }
 315:           finalUser = op;
 316:           distance++;
 317:         }
 318:         for (Value operand : getNestedOperands(op)) {
 319:           if (isa<mlir::triton::DotOpInterface>(op)) {
 320:             // Heuristic: only pipeline A and B operands of the dot op.
 321:             if (operand == op->getOperand(2))
 322:               continue;
 323:           }
 324:           Value v = operand;
 325:           Operation *defOp = v.getDefiningOp();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 326-330

```cpp
 326:           if (defOp && defOp->getBlock() == op->getBlock()) {
 327:             dfs(defOp, finalUser, distance);
 328:           }
 329:         }
 330:       };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 332-339

```cpp
 332:   for (Operation &op : forOp.getBody()->without_terminator()) {
 333:     // Arbitrary heuristic. TMEMStoreOp is included to keep logic consistent
 334:     // with legacy code when we weren't hoisting tmem allocas.
 335:     if (!isa<mlir::triton::DotOpInterface, ttng::TMEMStoreOp>(op))
 336:       continue;
 337:     seen.clear();
 338:     dfs(&op, &op, 0);
 339:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 341-348

```cpp
 341:   // If the loop has numStages attribute, also consider pipelining other loads
 342:   // that are not directly used by dot ops.
 343:   if (pipelineWithoutDot) {
 344:     for (Operation &op : forOp.getBody()->without_terminator()) {
 345:       if (!isa<tt::LoadOp, tt::DescriptorLoadLikeOpInterface>(op))
 346:         dfs(&op, &op, 0);
 347:     }
 348:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 350-359

```cpp
 350:   // We assume loads with different dist are assigned to different stages.
 351:   // If numStages is 2, we will have no stage available for indirect loads
 352:   // with dist >= 1. In general, when dist is equal to numStages - 1, we
 353:   // should not pipeline it.
 354:   for (auto iter = loadOpToIndLevel.begin(); iter != loadOpToIndLevel.end();) {
 355:     if (iter->second.first >= numStages - 1)
 356:       iter = loadOpToIndLevel.erase(iter);
 357:     else
 358:       ++iter;
 359:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 361-362

```cpp
 361:   return loadOpToIndLevel;
 362: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 364-366

```cpp
 364: //===----------------------------------------------------------------------===//
 365: // Pass Definition
 366: //===----------------------------------------------------------------------===//
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 368-369

```cpp
 368: #define GEN_PASS_DEF_TRITONGPUASSIGNLATENCIES
 369: #include "triton/Dialect/TritonGPU/Transforms/Passes.h.inc"
```

- **EN:** Pulls in generated definitions from TableGen/MLIR include fragments so the handwritten code can reuse auto-generated declarations.
- **CN:** 这里引入由 TableGen/MLIR 生成的定义片段，使手写代码能够复用自动生成的声明。
### Lines 371-373

```cpp
 371: struct AssignLatencies
 372:     : public impl::TritonGPUAssignLatenciesBase<AssignLatencies> {
 373:   using TritonGPUAssignLatenciesBase::TritonGPUAssignLatenciesBase;
```

- **EN:** Defines `AssignLatencies`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `AssignLatencies`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 375-376

```cpp
 375:   void runOnOperation() override { assignLatencies(getOperation(), numStages); }
 376: };
```

- **EN:** Defines `runOnOperation`, the pass entry point. It gathers analysis information and applies the file's transformation logic to the current operation.
- **CN:** 这里定义 `runOnOperation`，即 pass 的入口函数。它会收集分析信息，并把本文件的变换逻辑应用到当前操作上。
### Lines 378-378

```cpp
 378: } // namespace mlir::triton::gpu
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。

## Key Concepts / 关键概念
- **EN:** The main concern is pass-driven transformation around assign latencies.
  **CN:** 核心关注点是围绕 Assign Latencies 的 pass 驱动变换。
- **EN:** Tensor shape and element-type reasoning is central to the implementation.
  **CN:** 张量形状与元素类型推导是实现中的核心内容。
- **EN:** The code reasons at module scope rather than only on isolated operations.
  **CN:** 代码在模块范围内进行分析，而不只是处理单个操作。
- **EN:** Axis information captures per-dimension contiguity, divisibility, or constancy facts.
  **CN:** AxisInfo 记录逐维的连续性、可整除性或常量性等信息。
- **EN:** Tensor memory specific allocation, layout, or synchronization rules are important here.
  **CN:** 这里重点处理张量内存（TMEM）的分配、布局或同步规则。

## Dependencies / 依赖关系
- **Internal Triton headers / Triton 内部头文件:** `triton/Analysis/AxisInfo.h`, `triton/Dialect/TritonGPU/IR/Dialect.h`, `triton/Dialect/TritonGPU/Transforms/MMAv5PipelineUtility.h`, `triton/Dialect/TritonGPU/Transforms/Passes.h`, `triton/Dialect/TritonGPU/Transforms/PipeliningUtility.h`, `triton/Dialect/TritonGPU/Transforms/Schedule.h`, ... (+4 more)
- **MLIR headers / MLIR 头文件:** None
- **LLVM headers / LLVM 头文件:** `llvm/Support/Debug.h`
- **Standard/library headers / 标准或通用库头文件:** None
- **Generated fragments / 生成片段:** `triton/Dialect/TritonGPU/Transforms/Passes.h.inc`
- **Primary APIs used / 主要 API:** `ModuleOp`, `RankedTensorType`
- **Pipeline role / 流程角色:** This file participates in Triton pass pipelines and usually expects upstream analyses or dialect invariants to have prepared the IR. / 本文件参与 Triton 的 pass 流程，通常依赖上游分析或方言不变量先把 IR 准备好。
