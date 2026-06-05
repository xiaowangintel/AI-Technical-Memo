# LowerLoops.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `lib/Dialect/TritonGPU/Transforms/Pipeliner/LowerLoops.cpp`
- **Purpose / 作用:** **EN:** Implements the Lower Loops transformation or optimization pass for the TritonGPU pipeline. **CN:** 为 TritonGPU 编译流程实现与 Lower Loops 相关的变换或优化 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```cpp
   1: #include "mlir/Dialect/UB/IR/UBOps.h"
   2: #include "mlir/IR/Dominance.h"
   3: #include "triton/Analysis/AxisInfo.h"
   4: #include "triton/Analysis/Utility.h"
   5: #include "triton/Dialect/Triton/IR/Dialect.h"
   6: #include "triton/Dialect/Triton/IR/Types.h"
   7: #include "triton/Dialect/Triton/IR/Utility.h"
   8: #include "triton/Dialect/TritonGPU/IR/Dialect.h"
   9: #include "triton/Dialect/TritonGPU/IR/TritonGPUInterfaces.h"
  10: #include "triton/Dialect/TritonGPU/Transforms/MMAv5PipelineUtility.h"
  11: #include "triton/Dialect/TritonGPU/Transforms/PipeliningUtility.h"
  12: #include "triton/Dialect/TritonGPU/Transforms/Schedule.h"
  13: #include "triton/Dialect/TritonGPU/Transforms/Utility.h"
  14: #include "triton/Dialect/TritonNvidiaGPU/IR/Dialect.h"
  15: #include "triton/Dialect/TritonNvidiaGPU/Transforms/TMAUtilities.h"
  16: #include "triton/Tools/StrUtil.h"
  17: #include "llvm/Support/Debug.h"
  18: #include "llvm/Support/ErrorHandling.h"
```

- **EN:** Includes the interfaces this file depends on. Triton headers (`AxisInfo.h`, `Utility.h`, `Dialect.h`, `Types.h`, ... (+10 more)) provide domain-specific IR/support, MLIR headers (`UBOps.h`, `Dominance.h`) provide rewriting and analysis infrastructure, LLVM headers (`Debug.h`, `ErrorHandling.h`) supply low-level utilities, and standard/library headers (None) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（`AxisInfo.h`, `Utility.h`, `Dialect.h`, `Types.h`, ... (+10 more)）提供领域特定 IR/支持逻辑，MLIR 头文件（`UBOps.h`, `Dominance.h`）提供重写与分析基础设施，LLVM 头文件（`Debug.h`, `ErrorHandling.h`）提供底层工具，而标准/通用库头文件（None）提供通用能力。
### Lines 20-22

```cpp
  20: #define DEBUG_TYPE "triton-loop-pipeline"
  21: #define DBGS() (llvm::dbgs() << "[" DEBUG_TYPE "]: ")
  22: #define LDBG(X) LLVM_DEBUG(DBGS() << X << "\n")
```

- **EN:** Defines debug logging helpers and compile-time tags used when tracing this pass or utility at runtime.
- **CN:** 这里定义调试日志辅助宏和编译期标签，用于在运行时跟踪该 pass 或工具。
### Lines 24-27

```cpp
  24: using namespace mlir;
  25: namespace tt = mlir::triton;
  26: namespace ttg = mlir::triton::gpu;
  27: namespace ttng = mlir::triton::nvidia_gpu;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 29-31

```cpp
  29: namespace mlir {
  30: namespace triton {
  31: namespace gpu {
```

- **EN:** Opens or closes the namespace nesting for mlir -> triton -> gpu, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 mlir -> triton -> gpu 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 33-33

```cpp
  33: namespace {
```

- **EN:** Opens or closes the namespace nesting for (anonymous), keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 (anonymous) 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 35-37

```cpp
  35: /////////////////////////////
  36: // UTILS
  37: /////////////////////////////
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 39-47

```cpp
  39: int getSelfLatencyFromAttr(Operation *op) {
  40:   auto module = op->getParentOfType<ModuleOp>();
  41:   auto helper = TritonDialect::getLoaded(module)->getSelfLatencyAttrHelper();
  42:   if (!helper.isAttrPresent(op))
  43:     return 0;
  44:   int val = helper.getAttr(op).getInt();
  45:   helper.removeAttr(op);
  46:   return val;
  47: }
```

- **EN:** Defines accessor/helper `getSelfLatencyFromAttr` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getSelfLatencyFromAttr`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 49-58

```cpp
  49: // Check if the load can be pipelined entirely in shared memory,
  50: // or if we need to load to registers.
  51: bool mustLoadToRegisters(Operation *op) {
  52:   if (auto loadOp = dyn_cast<tt::LoadOp>(op)) {
  53:     // AsyncCopyGlobalToLocalOp does not support the non-zero "other" value.
  54:     // With consumer consuming directly the shared memory, there would be no way
  55:     // to replace masked values with the "other" value.
  56:     if (loadOp.getOther() && !isZeroConst(loadOp.getOther()))
  57:       return true;
  58:   }
```

- **EN:** Defines `mustLoadToRegisters`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `mustLoadToRegisters`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 60-65

```cpp
  60:   if (!op->hasOneUse())
  61:     return true;
  62:   Operation *user = *op->getUsers().begin();
  63:   auto alloc = dyn_cast<ttg::LocalAllocOp>(user);
  64:   if (!alloc)
  65:     return true;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 67-74

```cpp
  67:   Attribute loadEncoding;
  68:   if (auto descLoad = dyn_cast<DescriptorLoadLikeOpInterface>(op)) {
  69:     auto tensorType = cast<RankedTensorType>(op->getResult(0).getType());
  70:     loadEncoding = nvidia_gpu::getEncodingFromDescriptor(op, tensorType,
  71:                                                          descLoad.getDesc());
  72:   }
  73:   return loadEncoding && (loadEncoding != alloc.getType().getEncoding());
  74: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 76-90

```cpp
  76: int getDefUseStageDiff(Operation *op, scf::ForOp forOp,
  77:                        CoarseSchedule &schedule) {
  78:   assert(schedule.count(op) && "Op not found in the schedule");
  79:   int defStage = schedule[op].first;
  80:   CoarseSchedule::Cluster defCluster = schedule[op].second;
  81:   std::optional<int> useStage;
  82:   DenseSet<Operation *> topLevelUsers =
  83:       triton::getTopLevelUsersInLoop(op, forOp);
  84:   // Special case for loads used by local_alloc:
  85:   // we must consider the uses of the local_alloc, as it may be removed and its
  86:   // uses will become direct uses of the async load.
  87:   // TODO: This is overly conservative, we may need to restrict to cases where
  88:   // local_alloc is used by a dot product and has correct encoding.
  89:   if (isa<tt::LoadOp, tt::DescriptorLoadLikeOpInterface>(op)) {
  90:     DenseSet<Operation *> allocUsers;
```

- **EN:** Defines accessor/helper `getDefUseStageDiff` that exposes or updates operation state in a compact, reusable way. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义访问器/辅助函数 `getDefUseStageDiff`，以紧凑且可复用的方式读取或更新操作状态。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 91-108

```cpp
  91:     for (Operation *topLevelUser : topLevelUsers) {
  92:       if (auto localAlloc = dyn_cast<ttg::LocalAllocOp>(topLevelUser)) {
  93:         DenseSet<Operation *> users =
  94:             triton::getTopLevelUsersInLoop(localAlloc, forOp);
  95:         allocUsers.insert(users.begin(), users.end());
  96:       }
  97:     }
  98:     topLevelUsers.insert(allocUsers.begin(), allocUsers.end());
  99:   }
 100:   DenseSet<Operation *> topLevelWaitUsers;
 101:   for (Operation *topLevelUser : topLevelUsers) {
 102:     if (isa<ttng::WaitBarrierOp>(topLevelUser)) {
 103:       topLevelWaitUsers.insert(topLevelUser);
 104:     }
 105:   }
 106:   for (Operation *topLevelUser : topLevelUsers) {
 107:     int _useStage = schedule[topLevelUser].first;
 108:     CoarseSchedule::Cluster _useCluster = schedule[topLevelUser].second;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 109-125

```cpp
 109:     if (*_useCluster > *defCluster) {
 110:       // Check if we need extra buffer due to unusual execution order
 111:       // The issue occurs when users of the load are scheduled in a later
 112:       // cluster, which happens when conditional code gets moved to epilogue
 113:       // cluster. This creates a race condition where the local load happens
 114:       // after the global-to-local copy for the next pipeline stage starts.
 115:       _useStage++;
 116:     }
 117:     useStage = std::min(_useStage, useStage.value_or(_useStage));
 118:   }
 119:   // Waits tells us the buffer is still in use until the wait completes, we
 120:   // can't simply load from the buffer and replace the uses of the buffer with
 121:   // the load. The stage diff needs to account for the furthest wait.
 122:   for (Operation *topLevelUser : topLevelWaitUsers) {
 123:     int _useStage = schedule[topLevelUser].first;
 124:     useStage = std::max(_useStage, useStage.value_or(_useStage));
 125:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 126-130

```cpp
 126:   if (!useStage)
 127:     return 0;
 128:   assert(useStage >= defStage && "Op used before defined");
 129:   return useStage.value() - defStage;
 130: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 132-139

```cpp
 132: void replaceAllUsesDominatedBy(Operation *domOp, Value newValue, Value oldValue,
 133:                                DominanceInfo &domInfo) {
 134:   if (newValue == oldValue)
 135:     return;
 136:   oldValue.replaceUsesWithIf(newValue, [&](OpOperand &use) {
 137:     return domInfo.properlyDominates(domOp, use.getOwner());
 138:   });
 139: }
```

- **EN:** Defines `replaceAllUsesDominatedBy`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `replaceAllUsesDominatedBy`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 141-143

```cpp
 141: /////////////////////////////
 142: // LOWER LOADS
 143: /////////////////////////////
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 145-152

```cpp
 145: // Create an allocation that can hold distance number of loadOp shapes.
 146: static Value createAlloc(scf::ForOp &forOp, Operation *loadOp,
 147:                          ttg::SharedEncodingTrait sharedEnc,
 148:                          unsigned distance) {
 149:   return triton::createAlloc(
 150:       forOp, cast<RankedTensorType>(loadOp->getResultTypes().front()),
 151:       loadOp->getLoc(), sharedEnc, distance);
 152: }
```

- **EN:** Defines helper `createAlloc` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createAlloc`，用于计算或构造外围变换所需的中间数据。
### Lines 154-157

```cpp
 154: void createAsyncCopy(scf::ForOp forOp, tt::LoadOp loadOp, Value alloc,
 155:                      Value insertIdx, Value extractIdx, int contiguity,
 156:                      CoarseSchedule &schedule) {
 157:   OpBuilderForStage builder(loadOp.getLoc(), forOp, schedule);
```

- **EN:** Defines helper `createAsyncCopy` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createAsyncCopy`，用于计算或构造外围变换所需的中间数据。
### Lines 159-167

```cpp
 159:   Operation *firstUse = getFirstUseOfPipelinedOp({loadOp}, forOp, schedule);
 160:   assert(firstUse && "LoadOp has no users");
 161:   // Replace the load with async copy, wait and loal_load.
 162:   OpBuilder::InsertionGuard guard(builder);
 163:   builder.setInsertionPoint(loadOp);
 164:   builder.setStageCluster(schedule[loadOp]);
 165:   Value src = loadOp.getPtr();
 166:   Value mask = loadOp.getMask();
 167:   Value other = loadOp.getOther();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 169-175

```cpp
 169:   // Create async copy
 170:   Value view = createSingleBufferView(builder, alloc, insertIdx);
 171:   Operation *copy = ttg::AsyncCopyGlobalToLocalOp::create(
 172:       builder, src, view, mask, other, loadOp.getCache(), loadOp.getEvict(),
 173:       loadOp.getIsVolatile(), contiguity);
 174:   Operation *commit =
 175:       ttg::AsyncCommitGroupOp::create(builder, copy->getResult(0));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 177-180

```cpp
 177:   // Create wait and local load
 178:   builder.setStageCluster(schedule[firstUse]);
 179:   auto wait = ttg::AsyncWaitOp::create(builder, commit->getResult(0), 0);
 180:   auto viewLoad = createSingleBufferView(builder, alloc, extractIdx);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 182-199

```cpp
 182:   if (!loadOp.getOther() || isZeroConst(loadOp.getOther())) {
 183:     // If masking isn't required, load directly from shared
 184:     replaceUsesWithLocalLoad(builder, loadOp->getResult(0), viewLoad,
 185:                              wait.getResult());
 186:   } else if (loadOp->use_begin() != loadOp->use_end()) {
 187:     // Otherwise, create a select for non-zero other values as they are not
 188:     // handled by AsyncCopyGlobalToLocalOp for now.
 189:     auto sharedLoad = ttg::LocalLoadOp::create(builder, loadOp.getType(),
 190:                                                viewLoad, wait.getResult());
 191:     auto select = arith::SelectOp::create(
 192:         builder, loadOp.getType(),
 193:         // Use the mask operand from the original load, not the one with a
 194:         // potentially transformed layout.
 195:         loadOp.getMask(), sharedLoad.getResult(), other);
 196:     loadOp->replaceAllUsesWith(select->getResults());
 197:   }
 198:   schedule.erase(loadOp);
 199:   loadOp->erase();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 200-200

```cpp
 200: }
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 202-208

```cpp
 202: void createTMAAsyncCopy(
 203:     scf::ForOp forOp, Operation *loadOp, Value desc, Value alloc,
 204:     Value insertIdx, Value extractIdx, Value barrier, Operation *waitOp,
 205:     CoarseSchedule &schedule,
 206:     function_ref<void(OpBuilderForStage &, Value, Value, Value, Value)>
 207:         createCopy) {
 208:   OpBuilderForStage builder(loadOp->getLoc(), forOp, schedule);
```

- **EN:** Defines helper `createTMAAsyncCopy` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createTMAAsyncCopy`，用于计算或构造外围变换所需的中间数据。
### Lines 210-211

```cpp
 210:   Operation *firstUse = getFirstUseOfPipelinedOp({loadOp}, forOp, schedule);
 211:   assert(firstUse && "LoadOp has no users");
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 213-214

```cpp
 213:   builder.setInsertionPoint(loadOp);
 214:   builder.setStageCluster(schedule[loadOp]);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 216-217

```cpp
 216:   // Create async copy
 217:   Value view = createSingleBufferView(builder, alloc, insertIdx);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 219-220

```cpp
 219:   Value pred = arith::ConstantIntOp::create(builder, 1, 1);
 220:   createCopy(builder, desc, barrier, view, pred);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 222-226

```cpp
 222:   // Create local load after the wait
 223:   builder.setInsertionPointAfter(waitOp);
 224:   builder.setStageCluster(schedule[firstUse]);
 225:   auto viewLoad = createSingleBufferView(builder, alloc, extractIdx);
 226:   replaceUsesWithLocalLoad(builder, loadOp->getResult(0), viewLoad);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 228-230

```cpp
 228:   schedule.erase(loadOp);
 229:   loadOp->erase();
 230: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 232-244

```cpp
 232: void createTMAAsyncLoad(scf::ForOp forOp, tt::DescriptorLoadOp loadOp,
 233:                         Value alloc, Value insertIdx, Value extractIdx,
 234:                         Value barrier, Operation *waitOp,
 235:                         CoarseSchedule &schedule) {
 236:   return createTMAAsyncCopy(forOp, loadOp, loadOp.getDesc(), alloc, insertIdx,
 237:                             extractIdx, barrier, waitOp, schedule,
 238:                             [&](OpBuilderForStage &builder, Value desc,
 239:                                 Value barrier, Value view, Value pred) {
 240:                               ttng::AsyncTMACopyGlobalToLocalOp::create(
 241:                                   builder, loadOp.getLoc(), desc,
 242:                                   loadOp.getIndices(), barrier, view, pred);
 243:                             });
 244: }
```

- **EN:** Defines helper `createTMAAsyncLoad` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createTMAAsyncLoad`，用于计算或构造外围变换所需的中间数据。
### Lines 246-261

```cpp
 246: void createTMAAsyncGather(scf::ForOp forOp, tt::DescriptorGatherOp gatherOp,
 247:                           Value alloc, Value insertIdx, Value extractIdx,
 248:                           Value barrier, Operation *waitOp,
 249:                           CoarseSchedule &schedule) {
 250:   return createTMAAsyncCopy(
 251:       forOp, gatherOp, gatherOp.getDesc(), alloc, insertIdx, extractIdx,
 252:       barrier, waitOp, schedule,
 253:       [&](OpBuilderForStage &builder, Value desc, Value barrier, Value view,
 254:           Value pred) {
 255:         Value xOffsets = ttng::sextI16ToI32Indices(gatherOp.getXOffsets(),
 256:                                                    builder, gatherOp.getLoc());
 257:         ttng::AsyncTMAGatherOp::create(builder, gatherOp.getLoc(), desc,
 258:                                        xOffsets, gatherOp.getYOffset(), barrier,
 259:                                        view, pred);
 260:       });
 261: }
```

- **EN:** Defines helper `createTMAAsyncGather` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createTMAAsyncGather`，用于计算或构造外围变换所需的中间数据。
### Lines 263-276

```cpp
 263: struct AsyncLoad {
 264:   int stageDiff;
 265:   int contiguity = 1;
 266:   Value alloc;
 267:   Value barrier;
 268:   Operation *waitOp;
 269:   SharedEncodingTrait sharedEncoding;
 270: };
 271: struct LoadGroupInfo {
 272:   Value insertIdx;
 273:   Value extractIdx;
 274:   Value phase;
 275:   bool hasTMALoad = false;
 276: };
```

- **EN:** Defines `AsyncLoad`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `AsyncLoad`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 278-295

```cpp
 278: // Convert a scalar load to a load of a tensor of shape <1>.
 279: void convertScalarToTensorLoad(Operation *op, CoarseSchedule &schedule,
 280:                                scf::ForOp forOp) {
 281:   auto scalarLoad = cast<tt::LoadOp>(op);
 282:   Type scalarTy = scalarLoad.getType();
 283:   OpBuilderForStage builder(op->getLoc(), op, schedule);
 284:   builder.setInsertionPoint(op);
 285:   MLIRContext *ctx = op->getContext();
 286:   auto nWarps = lookupNumWarps(op);
 287:   ModuleOp mod = forOp->getParentOfType<ModuleOp>();
 288:   auto threadsPerWarp = TritonGPUDialect::getThreadsPerWarp(mod);
 289:   auto numCTAs = TritonGPUDialect::getNumCTAs(mod);
 290:   auto blockedEnc =
 291:       getDefaultBlockedEncoding(ctx, {1}, nWarps, threadsPerWarp, numCTAs);
 292:   auto newPtrTy =
 293:       RankedTensorType::get({1}, scalarLoad.getPtr().getType(), blockedEnc);
 294:   auto newPtr =
 295:       tt::SplatOp::create(builder, op->getLoc(), newPtrTy, scalarLoad.getPtr());
```

- **EN:** Defines helper `convertScalarToTensorLoad` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `convertScalarToTensorLoad`，用于计算或构造外围变换所需的中间数据。
### Lines 296-313

```cpp
 296:   scalarLoad.getPtrMutable().assign(newPtr);
 297:   if (scalarLoad.getMask()) {
 298:     auto newMaskTy =
 299:         RankedTensorType::get({1}, scalarLoad.getMask().getType(), blockedEnc);
 300:     auto newMask = tt::SplatOp::create(builder, op->getLoc(), newMaskTy,
 301:                                        scalarLoad.getMask());
 302:     scalarLoad.getMaskMutable().assign(newMask);
 303:   }
 304:   if (scalarLoad.getOther()) {
 305:     auto newOtherTy =
 306:         RankedTensorType::get({1}, scalarLoad.getOther().getType(), blockedEnc);
 307:     auto newOther = tt::SplatOp::create(builder, op->getLoc(), newOtherTy,
 308:                                         scalarLoad.getOther());
 309:     scalarLoad.getOtherMutable().assign(newOther);
 310:   }
 311:   auto newDstTy = RankedTensorType::get({1}, scalarLoad.getType(), blockedEnc);
 312:   scalarLoad.getResult().setType(newDstTy);
 313:   builder.setInsertionPointAfter(op);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 314-319

```cpp
 314:   Operation *firstUse = getFirstUseOfPipelinedOp({op}, forOp, schedule);
 315:   builder.setStageCluster(schedule[firstUse]);
 316:   Operation *unsplat = tt::UnsplatOp::create(builder, op->getLoc(), scalarTy,
 317:                                              scalarLoad.getResult());
 318:   scalarLoad.getResult().replaceAllUsesExcept(unsplat->getResult(0), unsplat);
 319: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 321-337

```cpp
 321: void createTMABarrierAndWait(
 322:     scf::ForOp forOp, llvm::MapVector<Operation *, AsyncLoad> &asyncLoads,
 323:     const llvm::MapVector<int, LoadGroupInfo> &loadGroups,
 324:     CoarseSchedule &schedule) {
 325:   SmallVector<SmallVector<Operation *>> commonWaitGroups;
 326:   llvm::SmallDenseSet<Operation *> visited;
 327:   // Find groups of loads that can share the same barrier. We look consecutive
 328:   // loads and check that there are uses in between.
 329:   for (auto &[loadOp, asyncLoad] : asyncLoads) {
 330:     if (!isTMALoad(loadOp) || visited.count(loadOp))
 331:       continue;
 332:     llvm::SmallDenseSet<Operation *> users;
 333:     SmallVector<Operation *> group;
 334:     Block *loadBlock = loadOp->getBlock();
 335:     auto addToGroup = [&](Operation *loadOp) {
 336:       group.push_back(loadOp);
 337:       visited.insert(loadOp);
```

- **EN:** Defines helper `createTMABarrierAndWait` that computes or constructs intermediate data used by the surrounding transformation. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义辅助函数 `createTMABarrierAndWait`，用于计算或构造外围变换所需的中间数据。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 338-355

```cpp
 338:       for (Operation *user : loadOp->getUsers()) {
 339:         // Special case for MMAv3 loads, we can ignore the alloc and only
 340:         // consider uses of the alloc op since it will be removed.
 341:         if (!mustLoadToRegisters(loadOp)) {
 342:           assert(loadOp->hasOneUse());
 343:           auto alloc = cast<ttg::LocalAllocOp>(*loadOp->getUsers().begin());
 344:           if (alloc->getBlock() == loadBlock) {
 345:             users.insert(alloc->getUsers().begin(), alloc->getUsers().end());
 346:             continue;
 347:           }
 348:         }
 349:         Operation *userInBlock = loadBlock->findAncestorOpInBlock(*user);
 350:         if (userInBlock)
 351:           users.insert(userInBlock);
 352:       }
 353:     };
 354:     addToGroup(loadOp);
 355:     Operation *nextOp = loadOp->getNextNode();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 356-370

```cpp
 356:     int numBuffers = asyncLoad.stageDiff;
 357:     while (nextOp) {
 358:       if (users.count(nextOp) || visited.count(nextOp))
 359:         break;
 360:       if (isTMALoad(nextOp) && asyncLoads.count(nextOp)) {
 361:         if (asyncLoads[nextOp].stageDiff != numBuffers)
 362:           break;
 363:         if (group.size() > 0 && schedule[group[0]] == schedule[nextOp]) {
 364:           addToGroup(nextOp);
 365:         }
 366:       }
 367:       nextOp = nextOp->getNextNode();
 368:     }
 369:     commonWaitGroups.push_back(group);
 370:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 372-382

```cpp
 372:   // For each group calculate the size and insert the barrier after the last
 373:   // load.
 374:   for (SmallVector<Operation *> &group : commonWaitGroups) {
 375:     int sizeInBytes = 0;
 376:     int numBuffers = asyncLoads[group[0]].stageDiff;
 377:     const LoadGroupInfo loadGroup = loadGroups.find(numBuffers)->second;
 378:     for (Operation *op : group) {
 379:       auto tensorTy = cast<RankedTensorType>(op->getResultTypes()[0]);
 380:       int loadSize = product(getShapePerCTA(tensorTy));
 381:       sizeInBytes += loadSize * tensorTy.getElementTypeBitWidth() / 8;
 382:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 384-389

```cpp
 384:     Value barrierAlloc = triton::createBarrierAlloc(forOp, numBuffers);
 385:     OpBuilderForStage builder(forOp.getLoc(), group[0], schedule);
 386:     Value barrier = triton::createSingleBufferView(builder, barrierAlloc,
 387:                                                    loadGroup.insertIdx);
 388:     Value pred = arith::ConstantIntOp::create(builder, 1, 1);
 389:     ttng::BarrierExpectOp::create(builder, barrier, sizeInBytes, pred);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 391-397

```cpp
 391:     builder.setInsertionPointAfter(group.back());
 392:     Operation *firstUse = getFirstUseOfPipelinedOp(group, forOp, schedule);
 393:     builder.setStageCluster(schedule[firstUse]);
 394:     Value barrierViewWait = triton::createSingleBufferView(
 395:         builder, barrierAlloc, loadGroup.extractIdx);
 396:     auto wait =
 397:         ttng::WaitBarrierOp::create(builder, barrierViewWait, loadGroup.phase);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 399-405

```cpp
 399:     // Update the async loads info.
 400:     for (Operation *op : group) {
 401:       asyncLoads[op].barrier = barrier;
 402:       asyncLoads[op].waitOp = wait;
 403:     }
 404:   }
 405: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 407-424

```cpp
 407: // Check if load requires additional buffer for a mma pipelining
 408: bool loadRequiresAdditionalBuffer(Operation *loadOp) {
 409:   std::function<void(Operation *, SmallVector<Operation *> & out)>
 410:       collectNonViewUsers = [&](Operation *op, SmallVector<Operation *> &out) {
 411:         for (Operation *user : op->getUsers()) {
 412:           if (user->hasTrait<OpTrait::MemDescViewTrait>())
 413:             collectNonViewUsers(user, out);
 414:           else
 415:             out.push_back(user);
 416:         }
 417:       };
 418:   // Pattern match the op sequence used for loading mmav3 operands
 419:   if (!mustLoadToRegisters(loadOp)) {
 420:     assert(loadOp->hasOneUse());
 421:     ttg::LocalAllocOp alloc =
 422:         dyn_cast<ttg::LocalAllocOp>(*loadOp->getUsers().begin());
 423:     if (alloc) {
 424:       SmallVector<Operation *> nonViewUsers;
```

- **EN:** Defines `loadRequiresAdditionalBuffer`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `loadRequiresAdditionalBuffer`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 425-432

```cpp
 425:       collectNonViewUsers(alloc, nonViewUsers);
 426:       return llvm::any_of(nonViewUsers, [&](Operation *op) {
 427:         return isa<ttng::WarpGroupDotOp>(op);
 428:       });
 429:     }
 430:   }
 431:   return false;
 432: }
```

- **EN:** Defines `llvm::any_of`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `llvm::any_of`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 434-451

```cpp
 434: scf::ForOp lowerLoads(scf::ForOp forOp, CoarseSchedule &schedule,
 435:                       triton::ModuleAxisInfoAnalysis &axisInfoAnalysis) {
 436:   llvm::MapVector<Operation *, AsyncLoad> asyncLoads;
 437:   llvm::MapVector<int, LoadGroupInfo> loadGroups;
 438:   llvm::SmallVector<Operation *> scalarLoads;
 439:   // Only visit the top level ops, we do not support pipelining conditional
 440:   // loads for now
 441:   for (auto &op : forOp.getBody()->without_terminator()) {
 442:     if (isa<tt::LoadOp, tt::DescriptorLoadLikeOpInterface>(op)) {
 443:       int stageDiff = getDefUseStageDiff(&op, forOp, schedule);
 444:       if (stageDiff == 0) {
 445:         // Don't care about non-pipelined loads. Scalar loads will be converted
 446:         // to tensor loads if they are pipelined.
 447:         continue;
 448:       }
 449:       SharedEncodingTrait sharedEncoding;
 450:       bool canUseAsyncCp = false;
 451:       int contiguity = 1;
```

- **EN:** Defines helper `lowerLoads` that performs a lowering step from Triton/MLIR semantics to a lower-level representation. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义辅助函数 `lowerLoads`，负责把 Triton/MLIR 语义中的一个步骤降级到更底层表示。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 452-469

```cpp
 452:       if (!isa<RankedTensorType>(op.getResultTypes()[0])) {
 453:         canUseAsyncCp = op.getResultTypes()[0].getIntOrFloatBitWidth() >= 32;
 454:         auto numCTAs = lookupNumCTAs(forOp);
 455:         sharedEncoding = ttg::SwizzledSharedEncodingAttr::get(
 456:             forOp.getContext(), 1, 1, 1, {0},
 457:             ttg::CGAEncodingAttr::get1DLayout(forOp.getContext(), numCTAs));
 458:         if (canUseAsyncCp) {
 459:           scalarLoads.push_back(&op);
 460:         }
 461:       } else {
 462:         sharedEncoding = getSharedEncoding(&op);
 463:         // Do not create async loads for small loads (cp.async requires at least
 464:         // 4 bytes)
 465:         canUseAsyncCp =
 466:             isa<tt::LoadOp>(op) &&
 467:             canBeConvertedToAsyncLoad(cast<tt::LoadOp>(op), axisInfoAnalysis);
 468:         int copyVecBytes = getCopyVecBytes(
 469:             cast<RankedTensorType>(op.getResultTypes()[0]), sharedEncoding);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 471-488

```cpp
 471:         canUseAsyncCp &= copyVecBytes >= 4;
 472:         if (canUseAsyncCp) {
 473:           auto loadOp = cast<tt::LoadOp>(op);
 474:           auto ptr = loadOp.getPtr();
 475:           unsigned vec = axisInfoAnalysis.getContiguity(ptr);
 476:           if (auto mask = loadOp.getMask())
 477:             vec = std::min<unsigned>(vec,
 478:                                      axisInfoAnalysis.getMaskAlignment(mask));
 479:           contiguity = vec;
 480:         }
 481:       }
 482:       if (canUseAsyncCp || isTMALoad(&op)) {
 483:         if (loadRequiresAdditionalBuffer(&op)) {
 484:           // Allocate additional buffer required by the wgmma pipelining.
 485:           stageDiff += 1;
 486:         }
 487:         auto &asyncLoad = asyncLoads[&op];
 488:         asyncLoad.stageDiff = stageDiff;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 489-502

```cpp
 489:         asyncLoad.contiguity = contiguity;
 490:         asyncLoad.sharedEncoding = sharedEncoding;
 491:       } else if (stageDiff > 1) {
 492:         // Distance-1 loads can in most cases be pipelined in registers without
 493:         // any performance degradation, as the schedule will usually reorder the
 494:         // user and the producer so there is no liverange overlap, and no copy
 495:         // needed.
 496:         op.emitRemark() << "Pipelining load that cannot use vectorized "
 497:                            "copy. This will likely "
 498:                            "lead to pipelining in registers and severe "
 499:                            "performance degradation.";
 500:       }
 501:     }
 502:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 504-507

```cpp
 504:   // Convert scalar loads to be able to use async copy.
 505:   for (auto op : scalarLoads) {
 506:     convertScalarToTensorLoad(op, schedule, forOp);
 507:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 509-510

```cpp
 509:   if (asyncLoads.empty())
 510:     return forOp;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 512-520

```cpp
 512:   for (auto &[loadOp, asyncLoad] : asyncLoads) {
 513:     Value alloc = createAlloc(forOp, loadOp, asyncLoad.sharedEncoding,
 514:                               asyncLoad.stageDiff);
 515:     asyncLoad.alloc = alloc;
 516:     loadGroups.insert({asyncLoad.stageDiff, {}});
 517:     if (isTMALoad(loadOp)) {
 518:       loadGroups[asyncLoad.stageDiff].hasTMALoad = true;
 519:     }
 520:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 522-539

```cpp
 522:   IRRewriter builder(forOp);
 523:   builder.setInsertionPoint(forOp);
 524:   Location loc = forOp.getLoc();
 525:   // Create a counter to index into the allocations per loop iteration.
 526:   // NOTE: We create two duplicates values, insertIdx and extractIdx so that the
 527:   // pipeliner will re-materialize the value in later stages of the pipeline
 528:   // instead of carrying it as a dependency across multiple iterations.
 529:   Value minusOne = arith::ConstantIntOp::create(builder, loc, -1, 32);
 530:   Value zero = arith::ConstantIntOp::create(builder, loc, 0, 32);
 531:   Value one = arith::ConstantIntOp::create(builder, loc, 1, 32);
 532:   SmallVector<Value> newOperands;
 533:   unsigned newOperandIndex = forOp.getBody()->getNumArguments();
 534:   for (auto [_, loadGroup] : loadGroups) {
 535:     newOperands.push_back(minusOne); // insertIdx
 536:     newOperands.push_back(minusOne); // extractIdx
 537:     if (loadGroup.hasTMALoad) {
 538:       // A single barrier arrival sequence is a "phase" and two phases can
 539:       // overlap, provided the phases are differentiated with an alternating
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 540-543

```cpp
 540:       // boolean value.
 541:       newOperands.push_back(zero); // phase
 542:     }
 543:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 545-546

```cpp
 545:   // Patch the loop to add the new loop carried dependencies.
 546:   forOp = addIterArgsToLoop(builder, forOp, newOperands);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 548-552

```cpp
 548:   // Update yield op with temporary yield values
 549:   auto forYield = cast<scf::YieldOp>(forOp.getBody()->getTerminator());
 550:   for (unsigned i = 0; i < newOperands.size(); ++i) {
 551:     forYield.getResultsMutable().append(newOperands[i]);
 552:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 554-566

```cpp
 554:   builder.setInsertionPoint(forOp);
 555:   loc = forOp.getLoc();
 556:   int argIdx = newOperandIndex;
 557:   for (auto &[numBuffers, loadGroup] : loadGroups) {
 558:     Value insertIdx = forOp.getBody()->getArgument(argIdx);
 559:     argIdx++;
 560:     Value extractIdx = forOp.getBody()->getArgument(argIdx);
 561:     argIdx++;
 562:     Value phase = nullptr;
 563:     if (loadGroup.hasTMALoad) {
 564:       phase = forOp.getBody()->getArgument(argIdx);
 565:       argIdx++;
 566:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 568-570

```cpp
 568:     // Create two counters for the insert and extract indices to avoid creating
 569:     // long liverange.
 570:     builder.setInsertionPoint(forOp.getBody(), forOp.getBody()->begin());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 572-584

```cpp
 572:     Value numBuffersVal =
 573:         arith::ConstantIntOp::create(builder, loc, numBuffers, 32);
 574:     loadGroup.insertIdx = createIncrementModulo(builder, loc, insertIdx,
 575:                                                 numBuffersVal, zero, one);
 576:     Value cndExt = nullptr;
 577:     loadGroup.extractIdx = createIncrementModulo(
 578:         builder, loc, extractIdx, numBuffersVal, zero, one, &cndExt);
 579:     if (phase) {
 580:       Value nextPhase = arith::XOrIOp::create(builder, loc, phase, one);
 581:       phase = arith::SelectOp::create(builder, loc, cndExt, nextPhase, phase);
 582:       loadGroup.phase = phase;
 583:     }
 584:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 586-586

```cpp
 586:   createTMABarrierAndWait(forOp, asyncLoads, loadGroups, schedule);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 588-605

```cpp
 588:   bool hasAsyncLoads = false;
 589:   for (auto [op, asyncLoad] : asyncLoads) {
 590:     auto [insertIdx, extractIdx, phase, _] = loadGroups[asyncLoad.stageDiff];
 591:     if (auto loadOp = dyn_cast<tt::LoadOp>(op)) {
 592:       createAsyncCopy(forOp, loadOp, asyncLoad.alloc, insertIdx, extractIdx,
 593:                       asyncLoad.contiguity, schedule);
 594:       hasAsyncLoads = true;
 595:     } else if (auto loadOp = dyn_cast<tt::DescriptorLoadOp>(op)) {
 596:       createTMAAsyncLoad(forOp, loadOp, asyncLoad.alloc, insertIdx, extractIdx,
 597:                          asyncLoad.barrier, asyncLoad.waitOp, schedule);
 598:     } else if (auto loadOp = dyn_cast<tt::DescriptorGatherOp>(op)) {
 599:       createTMAAsyncGather(forOp, loadOp, asyncLoad.alloc, insertIdx,
 600:                            extractIdx, asyncLoad.barrier, asyncLoad.waitOp,
 601:                            schedule);
 602:     }
 603:   }
 604:   // Patch the yield with the updated counters. Subtract to account for the loop
 605:   // counter.
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 606-612

```cpp
 606:   argIdx = newOperandIndex - 1;
 607:   for (auto &[numBuffers, loadGroup] : loadGroups) {
 608:     forYield.setOperand(argIdx++, loadGroup.insertIdx);
 609:     forYield.setOperand(argIdx++, loadGroup.extractIdx);
 610:     if (loadGroup.phase)
 611:       forYield.setOperand(argIdx++, loadGroup.phase);
 612:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 614-616

```cpp
 614:   // Automatically discover dependencies and schedule new insert/extract ops to
 615:   // correct stages.
 616:   scheduleDependencies(forOp, schedule);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 618-623

```cpp
 618:   if (hasAsyncLoads) {
 619:     // Insert sync point for any possibly outstanding loads after the loop. This
 620:     // can happen as we speculatively execute loads in the loop.
 621:     builder.setInsertionPointAfter(forOp);
 622:     ttg::AsyncWaitOp::create(builder, loc, ValueRange({}), 0);
 623:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 625-633

```cpp
 625:   // Make sure all ops have attributes.
 626:   for (Operation &op : forOp.getBody()->without_terminator()) {
 627:     if (!schedule.count(&op)) {
 628:       op.emitError() << "op not found in the schedule";
 629:     }
 630:     assert(schedule.count(&op) && "op not found in the schedule");
 631:   }
 632:   return forOp;
 633: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 635-637

```cpp
 635: /////////////////////////////
 636: // LOWER MMA
 637: /////////////////////////////
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 639-656

```cpp
 639: std::pair<Operation *, Operation *>
 640: getTmemUseStageBoundOps(Value alloc, scf::ForOp forOp,
 641:                         CoarseSchedule &schedule) {
 642:   std::pair<Operation *, Operation *> bounds = {nullptr, nullptr};
 643:   for (auto user : alloc.getUsers()) {
 644:     if (!forOp->isAncestor(user->getParentOp())) {
 645:       continue;
 646:     }
 647:     auto topLevelUser = forOp.getBody()->findAncestorOpInBlock(*user);
 648:     if (!bounds.first) {
 649:       bounds.first = topLevelUser;
 650:     }
 651:     if (!bounds.second) {
 652:       bounds.second = topLevelUser;
 653:     }
 654:     if (schedule.isOpBefore(topLevelUser, bounds.first)) {
 655:       bounds.first = topLevelUser;
 656:     }
```

- **EN:** Defines accessor/helper `getTmemUseStageBoundOps` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getTmemUseStageBoundOps`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 657-662

```cpp
 657:     if (schedule.isOpBefore(bounds.second, topLevelUser)) {
 658:       bounds.second = topLevelUser;
 659:     }
 660:   }
 661:   return bounds;
 662: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 664-680

```cpp
 664: Operation *hoistBufferOutOfLoop(scf::ForOp forOp, Operation *op,
 665:                                 CoarseSchedule &schedule) {
 666:   Operation *newStore = nullptr;
 667:   if (!isa<ttng::TMEMAllocOp, ttg::LocalAllocOp>(op))
 668:     return nullptr;
 669:   // If the alloc is already out of the loop, there is nothing to do.
 670:   if (!forOp->isAncestor(op))
 671:     return nullptr;
 672:   OpBuilderForStage builder(op->getLoc(), forOp, schedule);
 673:   auto allocType = dyn_cast<MemDescType>(op->getResult(0).getType());
 674:   auto newType = triton::gpu::MemDescType::get(
 675:       allocType.getShape(), allocType.getElementType(), allocType.getEncoding(),
 676:       allocType.getMemorySpace(),
 677:       /*mutableMemory=*/true);
 678:   auto newAlloc = builder.clone(*op);
 679:   newAlloc->getResult(0).setType(newType);
 680:   builder.setStageCluster(schedule[op]);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。
### Lines 681-697

```cpp
 681:   if (auto tmemAlloc = dyn_cast<ttng::TMEMAllocOp>(newAlloc)) {
 682:     tmemAlloc.getSrcMutable().clear();
 683:     builder.setInsertionPointAfter(op);
 684:     Value trueVal = arith::ConstantIntOp::create(builder, 1, 1);
 685:     newStore = ttng::TMEMStoreOp::create(builder, tmemAlloc.getResult(),
 686:                                          op->getOperand(0), trueVal);
 687:   } else {
 688:     auto localAlloc = cast<ttg::LocalAllocOp>(newAlloc);
 689:     localAlloc.getSrcMutable().clear();
 690:     builder.setInsertionPointAfter(op);
 691:     newStore = ttg::LocalStoreOp::create(builder, op->getOperand(0),
 692:                                          localAlloc.getResult());
 693:   }
 694:   replaceUsesAndPropagateType(builder, op, newAlloc->getResult(0));
 695:   op->erase();
 696:   return newStore;
 697: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 699-705

```cpp
 699: void createBarrierAndWaitOps(scf::ForOp forOp, CoarseSchedule &schedule,
 700:                              ttng::MMAv5OpInterface mma, int mmaSelfLatency,
 701:                              Value alloc, int phaseArgIdx,
 702:                              int barrierIdxArgIdx) {
 703:   auto isLoadToBePipelined = [&](Operation *op) {
 704:     return schedule[mma].first > schedule[op].first;
 705:   };
```

- **EN:** Defines helper `createBarrierAndWaitOps` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createBarrierAndWaitOps`，用于计算或构造外围变换所需的中间数据。
### Lines 707-707

```cpp
 707:   llvm::SmallDenseSet<Operation *> syncCandidates;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 709-716

```cpp
 709:   for (auto user : alloc.getUsers()) {
 710:     if (auto load = dyn_cast<ttng::TMEMLoadOp>(user)) {
 711:       if (load->getBlock() != mma->getBlock()) {
 712:         continue;
 713:       }
 714:       syncCandidates.insert(load);
 715:     }
 716:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 718-719

```cpp
 718:   ttng::MMAv5PipelineableOperandsHelper mmaPipeHelper(mma, forOp,
 719:                                                       isLoadToBePipelined);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 721-733

```cpp
 721:   for (auto def : mmaPipeHelper.unpipelineableOperandDefs) {
 722:     auto newStore = hoistBufferOutOfLoop(forOp, def, schedule);
 723:     // If the operands are not pipelineable, we need to consider the stores as
 724:     // well.
 725:     if (!mmaPipeHelper.isPipelineable &&
 726:         mmaPipeHelper.isOperandsStateDetermined) {
 727:       if (newStore) {
 728:         syncCandidates.insert(newStore);
 729:       } else {
 730:         syncCandidates.insert(def);
 731:       }
 732:     }
 733:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 735-740

```cpp
 735:   // Find the first sync candidate that appears after the MMA
 736:   // in the linearized schedule. This is either the first op to appear
 737:   // after the MMA or the first op
 738:   auto linearizedSchedule = schedule.linearized(forOp, mma);
 739:   std::optional<Operation *> latestSyncPoint = linearizedSchedule.findNext(
 740:       [&](Operation *op) { return syncCandidates.contains(op); });
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 742-754

```cpp
 742:   int mainWaitStage = schedule[mma].first + mmaSelfLatency;
 743:   CoarseSchedule::Cluster mainWaitCluster = schedule[mma].second;
 744:   if (latestSyncPoint && mmaPipeHelper.isOperandsStateDetermined) {
 745:     if (schedule.isOpBefore(*latestSyncPoint, mma)) {
 746:       mainWaitStage = schedule[mma].first + 1;
 747:       mainWaitCluster = schedule.clusters.newBefore(
 748:           schedule.splitClusterBefore(*latestSyncPoint, forOp));
 749:     } else {
 750:       mainWaitStage = schedule[*latestSyncPoint].first;
 751:       mainWaitCluster = schedule.clusters.newBefore(
 752:           schedule.splitClusterBefore(*latestSyncPoint, forOp));
 753:     }
 754:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 756-756

```cpp
 756:   int numStages = mainWaitStage - schedule[mma].first + 1;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 758-771

```cpp
 758:   OpBuilderForStage builder(mma.getLoc(), mma, schedule);
 759:   Value barrierAlloc = createBarrierAlloc(forOp, numStages);
 760:   Value vTrue = arith::ConstantIntOp::create(builder, 1, 1);
 761:   Value phase = forOp.getRegionIterArg(phaseArgIdx);
 762:   Value zero = arith::ConstantIntOp::create(builder, forOp.getLoc(), 0, 32);
 763:   Value barrierIdx;
 764:   if (numStages > 1) {
 765:     barrierIdx = forOp.getRegionIterArg(barrierIdxArgIdx);
 766:   } else {
 767:     barrierIdx = zero;
 768:   }
 769:   Value one = arith::ConstantIntOp::create(builder, forOp.getLoc(), 1, 32);
 770:   Value numStagesVal =
 771:       arith::ConstantIntOp::create(builder, forOp.getLoc(), numStages, 32);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 773-776

```cpp
 773:   Value barrierSlice =
 774:       triton::createSingleBufferView(builder, barrierAlloc, barrierIdx);
 775:   mma.addCompletionBarrier(barrierSlice, vTrue);
 776:   mma.setIsAsync(true);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 778-787

```cpp
 778:   // List of buffers that may be used until wait completes
 779:   SmallVector<Value> waitBuffers;
 780:   auto mmaAsDotOp = cast<DotOpInterface>(mma.getOperation());
 781:   waitBuffers.push_back(mmaAsDotOp.getA());
 782:   waitBuffers.push_back(mmaAsDotOp.getB());
 783:   if (auto mmaAsScaledDotOp =
 784:           dyn_cast<ttng::TCGen5MMAScaledOp>(mma.getOperation())) {
 785:     waitBuffers.push_back(mmaAsScaledDotOp.getAScale());
 786:     waitBuffers.push_back(mmaAsScaledDotOp.getBScale());
 787:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 789-791

```cpp
 789:   builder.setInsertionPointAfter(mma);
 790:   builder.setStageCluster({mainWaitStage, mainWaitCluster});
 791:   ttng::WaitBarrierOp::create(builder, barrierSlice, phase, waitBuffers);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 793-810

```cpp
 793:   // Add waits before loads in conditional blocks
 794:   for (auto user : alloc.getUsers()) {
 795:     if (auto load = dyn_cast<ttng::TMEMLoadOp>(user)) {
 796:       if (load->getBlock() == mma->getBlock()) {
 797:         continue;
 798:       }
 799:       auto topLevelUser = forOp.getBody()->findAncestorOpInBlock(*load);
 800:       if (!topLevelUser) {
 801:         continue;
 802:       }
 803:       auto [loadStage, loadCluster] = schedule[topLevelUser];
 804:       if (loadStage < mainWaitStage) {
 805:         builder.setStageCluster({loadStage, loadCluster});
 806:         builder.setInsertionPoint(load);
 807:         ttng::WaitBarrierOp::create(builder, barrierSlice, phase, waitBuffers);
 808:       }
 809:     }
 810:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 812-826

```cpp
 812:   builder.setStageCluster(schedule[mma]);
 813:   auto yieldOp = cast<scf::YieldOp>(forOp.getBody()->getTerminator());
 814:   builder.setInsertionPoint(yieldOp);
 815:   Value newPhase = arith::XOrIOp::create(builder, phase, one);
 816:   Value newBarrierIdx = barrierIdx;
 817:   if (numStages > 1) {
 818:     Value barWrap;
 819:     newBarrierIdx = createIncrementModulo(builder, builder.getLoc(), barrierIdx,
 820:                                           numStagesVal, zero, one, &barWrap);
 821:     newPhase = arith::SelectOp::create(builder, phase.getType(), barWrap,
 822:                                        newPhase, phase);
 823:   }
 824:   yieldOp->replaceUsesOfWith(phase, newPhase);
 825:   yieldOp->replaceUsesOfWith(barrierIdx, newBarrierIdx);
 826: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 828-842

```cpp
 828: void multibufferTensorMemory(scf::ForOp forOp, CoarseSchedule &schedule,
 829:                              ttng::TMEMAllocOp alloc, int bufIdxArgIdx,
 830:                              int tmemUseNumStages) {
 831:   DominanceInfo domInfo(forOp);
 832:   Value bufIdx = forOp.getRegionIterArg(bufIdxArgIdx);
 833:   SmallVector<std::pair<Operation *, Value>> bufIdxDefs;
 834:   auto getCurrBufIdx = [&](Operation *op) {
 835:     for (auto [_op, _val] : llvm::reverse(bufIdxDefs)) {
 836:       if (domInfo.properlyDominates(_op, op)) {
 837:         return _val;
 838:       }
 839:     }
 840:     return Value();
 841:   };
 842:   bufIdxDefs.push_back({&forOp.getBody()->front(), bufIdx});
```

- **EN:** Defines `multibufferTensorMemory`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `multibufferTensorMemory`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 844-849

```cpp
 844:   OpBuilderForStage builder(alloc.getLoc(), alloc, schedule);
 845:   auto newAlloc = createTMemAlloc(builder, alloc, true, tmemUseNumStages);
 846:   Value numStagesVal =
 847:       arith::ConstantIntOp::create(builder, tmemUseNumStages, 32);
 848:   Value zero = arith::ConstantIntOp::create(builder, 0, 32);
 849:   Value one = arith::ConstantIntOp::create(builder, 1, 32);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 851-851

```cpp
 851:   bool multibufferingIsValid = false;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 853-870

```cpp
 853:   SmallVector<Operation *> allocUsers =
 854:       llvm::to_vector(alloc.getResult().getUsers());
 855:   auto auxBuilder = OpBuilder(forOp);
 856:   Value replTok = ub::PoisonOp::create(auxBuilder, forOp.getLoc(),
 857:                                        builder.getType<AsyncTokenType>());
 858:   if (newAlloc.getToken()) {
 859:     newAlloc.getToken().replaceAllUsesWith(replTok);
 860:   }
 861:   for (auto user : allocUsers) {
 862:     if (auto store = dyn_cast<ttng::TMEMStoreOp>(user)) {
 863:       store.getDepMutable().clear();
 864:       store.getToken().replaceAllUsesWith(replTok);
 865:       if (forOp->isAncestor(store)) {
 866:         // We can multibuffer, since the store is a point where we can
 867:         // change the buffer index
 868:         multibufferingIsValid = true;
 869:         builder.setStageCluster(schedule[store]);
 870:         builder.setInsertionPoint(store);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 871-887

```cpp
 871:         // Change the buffer index to the new buffer index on store.
 872:         Value curBufIdx = getCurrBufIdx(store);
 873:         Value newBufIdx = createIncrementModulo(
 874:             builder, forOp.getLoc(), curBufIdx, numStagesVal, zero, one);
 875:         if (Value pred = store.getPred()) {
 876:           newBufIdx = arith::SelectOp::create(builder, newBufIdx.getType(),
 877:                                               pred, newBufIdx, curBufIdx);
 878:         }
 879:         replaceAllUsesDominatedBy(store, newBufIdx, curBufIdx, domInfo);
 880:         bufIdxDefs.push_back({store, newBufIdx});
 881:         auto tmemSlice =
 882:             triton::createSingleBufferView(builder, newAlloc, newBufIdx);
 883:         store.getDstMutable().assign(tmemSlice);
 884:       } else {
 885:         // Store before the loop
 886:         assert(store->isBeforeInBlock(forOp) && "Store is not before the loop");
 887:         builder.setInsertionPoint(store);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 888-905

```cpp
 888:         auto tmemSlice =
 889:             triton::createSingleBufferView(builder, newAlloc, zero);
 890:         store.getDstMutable().assign(tmemSlice);
 891:       }
 892:     } else if (auto load = dyn_cast<ttng::TMEMLoadOp>(user)) {
 893:       load.getDepMutable().clear();
 894:       load.getToken().replaceAllUsesWith(replTok);
 895:       if (forOp->isAncestor(load)) {
 896:         builder.setStageCluster(schedule[load]);
 897:         builder.setInsertionPoint(load);
 898:         Value curBufIdx = getCurrBufIdx(load);
 899:         auto tmemSlice =
 900:             triton::createSingleBufferView(builder, newAlloc, curBufIdx);
 901:         load.getSrcMutable().assign(tmemSlice);
 902:       } else {
 903:         // Load after the loop
 904:         assert(forOp->isBeforeInBlock(load) && "Load is not after the loop");
 905:         builder.setInsertionPoint(load);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 906-923

```cpp
 906:         auto tmemSlice = triton::createSingleBufferView(
 907:             builder, newAlloc, forOp->getResult(bufIdxArgIdx));
 908:         load.getSrcMutable().assign(tmemSlice);
 909:       }
 910:     } else if (auto mma = dyn_cast<ttng::MMAv5OpInterface>(user)) {
 911:       mma.getAccDepMutable().clear();
 912:       mma.getToken().replaceAllUsesWith(replTok);
 913:       builder.setStageCluster(schedule[mma]);
 914:       builder.setInsertionPoint(mma);
 915:       // We can legally switch to next buffer index if the mma does not use the
 916:       // accumulator
 917:       auto isConstTrue = [](Value v) {
 918:         if (auto constOp = v.getDefiningOp<arith::ConstantOp>()) {
 919:           if (auto attr = dyn_cast<BoolAttr>(constOp.getValueAttr())) {
 920:             return attr.getValue();
 921:           }
 922:         }
 923:         return false;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 924-941

```cpp
 924:       };
 925:       multibufferingIsValid = !isConstTrue(mma.useAccumulator());
 926:       Value curBufIdx = getCurrBufIdx(mma.getOperation());
 927:       Value newBufIdx = createIncrementModulo(
 928:           builder, forOp.getLoc(), curBufIdx, numStagesVal, zero, one);
 929:       newBufIdx =
 930:           arith::SelectOp::create(builder, newBufIdx.getType(),
 931:                                   mma.useAccumulator(), curBufIdx, newBufIdx);
 932:       replaceAllUsesDominatedBy(mma.getOperation(), newBufIdx, curBufIdx,
 933:                                 domInfo);
 934:       bufIdxDefs.push_back({mma.getOperation(), newBufIdx});
 935:       auto tmemSlice =
 936:           triton::createSingleBufferView(builder, newAlloc, newBufIdx);
 937:       mma.setAccumulator(tmemSlice);
 938:     } else {
 939:       llvm::errs() << "Unsupported user of the accumulator: " << *user << "\n";
 940:       llvm::report_fatal_error("Unsupported user of the accumulator");
 941:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 942-949

```cpp
 942:   }
 943:   if (!multibufferingIsValid) {
 944:     llvm::report_fatal_error(
 945:         "Trying to multibuffer TMEM while there is no store to the "
 946:         "accumulator, and the mma uses the accumulator all the time.");
 947:   }
 948:   alloc.getToken().replaceAllUsesWith(newAlloc.getToken());
 949:   alloc->erase();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 951-954

```cpp
 951:   Value newBufIdx = bufIdxDefs.back().second;
 952:   replaceAllUsesDominatedBy(newBufIdx.getDefiningOp(), newBufIdx, bufIdx,
 953:                             domInfo);
 954: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 956-958

```cpp
 956: scf::ForOp lowerMMA(ttng::MMAv5OpInterface mma, scf::ForOp forOp,
 957:                     CoarseSchedule &schedule) {
 958:   Value alloc = mma.getAccumulator();
```

- **EN:** Defines helper `lowerMMA` that performs a lowering step from Triton/MLIR semantics to a lower-level representation.
- **CN:** 这里定义辅助函数 `lowerMMA`，负责把 Triton/MLIR 语义中的一个步骤降级到更底层表示。
### Lines 960-963

```cpp
 960:   int mmaSelfLatency = getSelfLatencyFromAttr(mma.getOperation());
 961:   if (mmaSelfLatency == 0) {
 962:     return forOp;
 963:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 965-979

```cpp
 965:   // Create barrier and wait ops
 966:   std::pair<Operation *, Operation *> tmemUseStageBoundOps =
 967:       getTmemUseStageBoundOps(alloc, forOp, schedule);
 968:   int tmemUseNumStages = schedule[tmemUseStageBoundOps.second].first -
 969:                          schedule[tmemUseStageBoundOps.first].first;
 970:   // If def is in the earlier cluster than the use, we will have a liverange
 971:   // overlap and need to add an extra buffer.
 972:   if (schedule.isOpInEarlierCluster(tmemUseStageBoundOps.first,
 973:                                     tmemUseStageBoundOps.second) ||
 974:       (schedule.isOpInSameCluster(tmemUseStageBoundOps.first,
 975:                                   tmemUseStageBoundOps.second) &&
 976:        tmemUseStageBoundOps.first->isBeforeInBlock(
 977:            tmemUseStageBoundOps.second))) {
 978:     tmemUseNumStages += 1;
 979:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 981-984

```cpp
 981:   // If the accumulator needs to be double-buffered but we can't find the alloc
 982:   // op, then bail out.
 983:   if (tmemUseNumStages > 1 && !alloc.getDefiningOp<ttng::TMEMAllocOp>())
 984:     return forOp;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 986-989

```cpp
 986:   OpBuilder builder(forOp);
 987:   Value minusOne =
 988:       arith::ConstantIntOp::create(builder, forOp.getLoc(), -1, 32);
 989:   Value zero = arith::ConstantIntOp::create(builder, forOp.getLoc(), 0, 32);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 991-1003

```cpp
 991:   // Add arguments to the forOp
 992:   unsigned newOperandIndex = forOp.getInitArgs().size();
 993:   SmallVector<Value> newOperands = {
 994:       zero, // phase
 995:       zero, // barrierIdx
 996:   };
 997:   if (tmemUseNumStages > 1) {
 998:     newOperands.push_back(minusOne); // bufIdx
 999:   }
1000:   scf::ForOp newForOp =
1001:       replaceForOpWithNewSignature(builder, forOp, newOperands);
1002:   forOp.erase();
1003:   forOp = newForOp;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1005-1009

```cpp
1005:   int phaseArgIdx = newOperandIndex + 0;
1006:   int barrierIdxArgIdx = newOperandIndex + 1;
1007:   int bufIdxArgIdx = newOperandIndex + 2;
1008:   Value phase = forOp.getRegionIterArg(phaseArgIdx);
1009:   Value barrierIdx = forOp.getRegionIterArg(barrierIdxArgIdx);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1011-1018

```cpp
1011:   SmallVector<Value> newYieldOperands = {phase, barrierIdx};
1012:   if (tmemUseNumStages > 1) {
1013:     Value bufIdx = forOp.getRegionIterArg(bufIdxArgIdx);
1014:     newYieldOperands.push_back(bufIdx);
1015:   }
1016:   cast<scf::YieldOp>(forOp.getBody()->getTerminator())
1017:       .getResultsMutable()
1018:       .append(newYieldOperands);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1020-1021

```cpp
1020:   createBarrierAndWaitOps(forOp, schedule, mma, mmaSelfLatency, alloc,
1021:                           phaseArgIdx, barrierIdxArgIdx);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1023-1027

```cpp
1023:   if (tmemUseNumStages > 1) {
1024:     multibufferTensorMemory(forOp, schedule,
1025:                             alloc.getDefiningOp<ttng::TMEMAllocOp>(),
1026:                             bufIdxArgIdx, tmemUseNumStages);
1027:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1029-1030

```cpp
1029:   return forOp;
1030: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1032-1039

```cpp
1032: scf::ForOp lowerMMAs(scf::ForOp forOp, CoarseSchedule &schedule) {
1033:   SmallVector<ttng::MMAv5OpInterface> mmas;
1034:   forOp.walk([&](ttng::MMAv5OpInterface mma) { mmas.push_back(mma); });
1035:   for (auto mma : mmas) {
1036:     forOp = lowerMMA(mma, forOp, schedule);
1037:   }
1038:   return forOp;
1039: }
```

- **EN:** Defines helper `lowerMMAs` that performs a lowering step from Triton/MLIR semantics to a lower-level representation. The implementation traverses IR operations to collect facts or apply rewrites globally. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义辅助函数 `lowerMMAs`，负责把 Triton/MLIR 语义中的一个步骤降级到更底层表示。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 1041-1043

```cpp
1041: /////////////////////////////
1042: // LOWER LOOP
1043: /////////////////////////////
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 1045-1055

```cpp
1045: void lowerLoop(scf::ForOp forOp,
1046:                triton::ModuleAxisInfoAnalysis &axisInfoAnalysis) {
1047:   CoarseSchedule schedule;
1048:   if (failed(schedule.deSerialize(forOp))) {
1049:     return;
1050:   }
1051:   scf::ForOp newForOp = lowerMMAs(forOp, schedule);
1052:   newForOp = lowerLoads(newForOp, schedule, axisInfoAnalysis);
1053:   newForOp = lowerTMADescriptors(newForOp, schedule);
1054:   schedule.serialize(newForOp);
1055: }
```

- **EN:** Defines helper `lowerLoop` that performs a lowering step from Triton/MLIR semantics to a lower-level representation.
- **CN:** 这里定义辅助函数 `lowerLoop`，负责把 Triton/MLIR 语义中的一个步骤降级到更底层表示。
### Lines 1057-1057

```cpp
1057: } // namespace
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 1059-1068

```cpp
1059: void lowerLoops(ModuleOp moduleOp) {
1060:   triton::ModuleAxisInfoAnalysis axisInfoAnalysis(moduleOp);
1061:   SmallVector<scf::ForOp> loops;
1062:   moduleOp->walk([&](scf::ForOp forOp) { loops.push_back(forOp); });
1063:   if (loops.empty())
1064:     return;
1065:   for (auto forOp : loops) {
1066:     lowerLoop(forOp, axisInfoAnalysis);
1067:   }
1068: }
```

- **EN:** Defines helper `lowerLoops` that performs a lowering step from Triton/MLIR semantics to a lower-level representation. The implementation traverses IR operations to collect facts or apply rewrites globally. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义辅助函数 `lowerLoops`，负责把 Triton/MLIR 语义中的一个步骤降级到更底层表示。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 1070-1072

```cpp
1070: } // namespace gpu
1071: } // namespace triton
1072: } // namespace mlir
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。

## Key Concepts / 关键概念
- **EN:** The main concern is pass-driven transformation around lower loops.
  **CN:** 核心关注点是围绕 Lower Loops 的 pass 驱动变换。
- **EN:** Memory descriptor types carry layout, rank, and address-space information.
  **CN:** 内存描述符类型携带布局、维度和地址空间信息。
- **EN:** Tensor shape and element-type reasoning is central to the implementation.
  **CN:** 张量形状与元素类型推导是实现中的核心内容。
- **EN:** Encoding attributes describe how distributed GPU data is laid out across threads and warps.
  **CN:** 编码属性描述 GPU 分布式数据在线程与 warp 之间的布局方式。
- **EN:** The code reasons at module scope rather than only on isolated operations.
  **CN:** 代码在模块范围内进行分析，而不只是处理单个操作。
- **EN:** Axis information captures per-dimension contiguity, divisibility, or constancy facts.
  **CN:** AxisInfo 记录逐维的连续性、可整除性或常量性等信息。

## Dependencies / 依赖关系
- **Internal Triton headers / Triton 内部头文件:** `triton/Analysis/AxisInfo.h`, `triton/Analysis/Utility.h`, `triton/Dialect/Triton/IR/Dialect.h`, `triton/Dialect/Triton/IR/Types.h`, `triton/Dialect/Triton/IR/Utility.h`, `triton/Dialect/TritonGPU/IR/Dialect.h`, ... (+8 more)
- **MLIR headers / MLIR 头文件:** `mlir/Dialect/UB/IR/UBOps.h`, `mlir/IR/Dominance.h`
- **LLVM headers / LLVM 头文件:** `llvm/Support/Debug.h`, `llvm/Support/ErrorHandling.h`
- **Standard/library headers / 标准或通用库头文件:** None
- **Primary APIs used / 主要 API:** `ModuleOp`, `RankedTensorType`, `MemDescType`, `OpBuilder`
- **Pipeline role / 流程角色:** This file participates in Triton pass pipelines and usually expects upstream analyses or dialect invariants to have prepared the IR. / 本文件参与 Triton 的 pass 流程，通常依赖上游分析或方言不变量先把 IR 准备好。
