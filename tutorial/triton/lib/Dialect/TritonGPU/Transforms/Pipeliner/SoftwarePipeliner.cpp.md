# SoftwarePipeliner.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `lib/Dialect/TritonGPU/Transforms/Pipeliner/SoftwarePipeliner.cpp`
- **Purpose / 作用:** **EN:** Implements the Software Pipeliner transformation or optimization pass for the TritonGPU pipeline. **CN:** 为 TritonGPU 编译流程实现与 Software Pipeliner 相关的变换或优化 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```cpp
   1: #include "mlir/Dialect/Tensor/IR/Tensor.h"
   2: #include "mlir/Dialect/UB/IR/UBOps.h"
   3: #include "mlir/IR/TypeUtilities.h"
   4: #include "mlir/IR/Verifier.h"
   5: #include "mlir/Interfaces/SideEffectInterfaces.h"
   6: #include "mlir/Support/LLVM.h"
   7: #include "mlir/Transforms/GreedyPatternRewriteDriver.h"
   8: #include "triton/Analysis/AxisInfo.h"
   9: #include "triton/Analysis/Utility.h"
  10: #include "triton/Dialect/Triton/IR/Utility.h"
  11: #include "triton/Dialect/Triton/Transforms/LoopPeeling.h"
  12: #include "triton/Dialect/TritonGPU/IR/Dialect.h"
  13: #include "triton/Dialect/TritonGPU/Transforms/Passes.h"
  14: #include "triton/Dialect/TritonGPU/Transforms/PipelineExpander.h"
  15: #include "triton/Dialect/TritonGPU/Transforms/PipeliningUtility.h"
  16: #include "triton/Dialect/TritonGPU/Transforms/Schedule.h"
  17: #include "triton/Dialect/TritonGPU/Transforms/Utility.h"
  18: #include "triton/Dialect/TritonNvidiaGPU/IR/Dialect.h"
```

- **EN:** Includes the interfaces this file depends on. Triton headers (`AxisInfo.h`, `Utility.h`, `Utility.h`, `LoopPeeling.h`, ... (+7 more)) provide domain-specific IR/support, MLIR headers (`Tensor.h`, `UBOps.h`, `TypeUtilities.h`, `Verifier.h`, ... (+3 more)) provide rewriting and analysis infrastructure, LLVM headers (None) supply low-level utilities, and standard/library headers (None) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（`AxisInfo.h`, `Utility.h`, `Utility.h`, `LoopPeeling.h`, ... (+7 more)）提供领域特定 IR/支持逻辑，MLIR 头文件（`Tensor.h`, `UBOps.h`, `TypeUtilities.h`, `Verifier.h`, ... (+3 more)）提供重写与分析基础设施，LLVM 头文件（None）提供底层工具，而标准/通用库头文件（None）提供通用能力。
### Lines 19-28

```cpp
  19: #include "triton/Tools/Sys/Dump.h"
  20: //===----------------------------------------------------------------------===//
  21: // This file will create a schedule that will be handed over to the pipeline
  22: // expander.
  23: // Software pipeliners are usually separated into two pieces, one that create a
  24: // modulo schedule and an expander that rewrites the loop and emits a prologue
  25: // and epilogue. This pass first calls a helper that will pre-process the IR
  26: // to create async operations and create a modulo schedule. Then we call the
  27: // expander to generate the prologue and new loop.
  28: //===----------------------------------------------------------------------===//
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 30-32

```cpp
  30: namespace mlir {
  31: namespace triton {
  32: namespace gpu {
```

- **EN:** Opens or closes the namespace nesting for mlir -> triton -> gpu, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 mlir -> triton -> gpu 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 34-35

```cpp
  34: #define GEN_PASS_DEF_TRITONGPUPIPELINE
  35: #include "triton/Dialect/TritonGPU/Transforms/Passes.h.inc"
```

- **EN:** Pulls in generated definitions from TableGen/MLIR include fragments so the handwritten code can reuse auto-generated declarations.
- **CN:** 这里引入由 TableGen/MLIR 生成的定义片段，使手写代码能够复用自动生成的声明。
### Lines 37-39

```cpp
  37: static void pipelineWgmma(ModuleOp moduleOp, unsigned numStages) {
  38:   SmallVector<scf::ForOp> loops;
  39:   moduleOp->walk([&](scf::ForOp forOp) { loops.push_back(forOp); });
```

- **EN:** Defines `pipelineWgmma`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. The implementation traverses IR operations to collect facts or apply rewrites globally.
- **CN:** 这里定义 `pipelineWgmma`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。
### Lines 41-45

```cpp
  41:   for (scf::ForOp forOp : loops) {
  42:     if (getNumStagesOrDefault(forOp, numStages) >= 1)
  43:       mlir::triton::asyncLaunchDots(forOp);
  44:   }
  45: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 47-62

```cpp
  47: static bool hasMMAv5WaitsInLastStage(scf::ForOp forOp,
  48:                                      CoarseSchedule &schedule) {
  49:   int maxStage = schedule.getNumStages() - 1;
  50:   bool hasMMAv5 = false;
  51:   bool hasWaitInLastStage = false;
  52:   for (auto &op : forOp.getBody()->without_terminator()) {
  53:     if (isa<triton::nvidia_gpu::WaitBarrierOp>(op) &&
  54:         schedule[&op].first == maxStage) {
  55:       hasWaitInLastStage = true;
  56:     }
  57:     if (isa<triton::nvidia_gpu::MMAv5OpInterface>(op)) {
  58:       hasMMAv5 = true;
  59:     }
  60:   }
  61:   return hasMMAv5 && hasWaitInLastStage;
  62: }
```

- **EN:** Defines `hasMMAv5WaitsInLastStage`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `hasMMAv5WaitsInLastStage`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 64-79

```cpp
  64: static void expandLoops(ModuleOp moduleOp) {
  65:   DenseSet<MaskOp> peeledMaskOps;
  66:   auto processPeeledEpilogueOp = [&](RewriterBase &rewriter, Operation *op,
  67:                                      bool isEpilogue) -> Operation * {
  68:     OpBuilder::InsertionGuard guard(rewriter);
  69:     rewriter.setInsertionPoint(op);
  70:     if (auto predOp = dyn_cast<triton::gpu::PredicateStageOp>(op)) {
  71:       if (isEpilogue) {
  72:         // Return false for the predicate of the peeled iteration
  73:         return mlir::arith::ConstantIntOp::create(
  74:             rewriter, predOp.getLoc(), predOp.getResult().getType(), 0);
  75:       }
  76:       if (predOp.getStage() == predOp.getMaxStage() - 1) {
  77:         return mlir::arith::ConstantIntOp::create(
  78:             rewriter, predOp.getLoc(), predOp.getResult().getType(), 1);
  79:       }
```

- **EN:** Defines `expandLoops`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `expandLoops`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 80-91

```cpp
  80:       return triton::emitPredicateForStage(
  81:                  rewriter, predOp.getIv(), predOp.getUb(), predOp.getStep(),
  82:                  predOp.getMaxStage(), predOp.getStage())
  83:           .getDefiningOp();
  84:     }
  85:     if (auto maskOp = dyn_cast<triton::gpu::MaskOp>(op)) {
  86:       if (isEpilogue) {
  87:         peeledMaskOps.insert(maskOp);
  88:       }
  89:     }
  90:     return op;
  91:   };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 93-99

```cpp
  93:   SmallVector<scf::ForOp> loops;
  94:   moduleOp->walk([&](scf::ForOp forOp) { loops.push_back(forOp); });
  95:   for (scf::ForOp forOp : loops) {
  96:     CoarseSchedule schedule;
  97:     if (failed(schedule.deSerialize(forOp))) {
  98:       continue;
  99:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. The implementation traverses IR operations to collect facts or apply rewrites globally. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 101-111

```cpp
 101:     std::vector<std::pair<Operation *, unsigned>> finalSchedule =
 102:         schedule.createFinalSchedule(forOp);
 103:     triton::PipeliningOption options;
 104:     options.supportDynamicLoops = true;
 105:     options.peelEpilogue = false;
 106:     options.predicateFn = wrapInMaskOp;
 107:     options.getScheduleFn =
 108:         [&](scf::ForOp forOp,
 109:             std::vector<std::pair<Operation *, unsigned>> &schedule) {
 110:           schedule = finalSchedule;
 111:         };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 113-123

```cpp
 113:     // Testing feature: allow for unresolved predicate stage ops
 114:     // in the loop body.
 115:     bool keepPredicateStage = forOp->hasAttr("__test_keep_predicate_stage");
 116:     // TODO: Enable epilogue peeling for warp specialized loops
 117:     // Heuristic: only peel epilogue for MMAv5 loops with waits in the last
 118:     // stage
 119:     bool customEpiloguePeeling =
 120:         hasMMAv5WaitsInLastStage(forOp, schedule) &&
 121:         !forOp->getParentOfType<triton::gpu::WarpSpecializeOp>() &&
 122:         !keepPredicateStage; // do not peel if we are testing the stage
 123:                              // predication
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 125-136

```cpp
 125:     if (keepPredicateStage || customEpiloguePeeling) {
 126:       options.emitPredicateStageFn =
 127:           [](RewriterBase &rewriter, Value inductionVar, Value upperBound,
 128:              Value step, uint64_t maxStage, uint64_t stage) {
 129:             return triton::gpu::PredicateStageOp::create(
 130:                 rewriter, inductionVar.getLoc(), inductionVar, upperBound, step,
 131:                 maxStage, stage);
 132:           };
 133:     }
 134:     IRRewriter rewriter(forOp);
 135:     FailureOr<scf::ForOp> newForOp =
 136:         triton::pipelineForLoop(rewriter, forOp, options);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 138-144

```cpp
 138:     if (failed(newForOp)) {
 139:       continue;
 140:     }
 141:     forOp = *newForOp;
 142:     if (customEpiloguePeeling) {
 143:       mlir::triton::peelLoopEpilogue(forOp, processPeeledEpilogueOp);
 144:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 146-163

```cpp
 146:     // Prune all the statically dead mask ops in the epilogue. This is a
 147:     // hack, ideally we should do it for all the mask ops, but it is incorrect
 148:     // if we have speculatively executed async cp operations that will store to
 149:     // shmem even if the mask is false.
 150:     for (auto maskOp : peeledMaskOps) {
 151:       rewriter.setInsertionPoint(maskOp);
 152:       if (isConstantIntValue(maskOp.getPred(), 0)) {
 153:         SmallVector<Value> results;
 154:         for (auto result : maskOp->getResults()) {
 155:           auto poisonOp = mlir::ub::PoisonOp::create(rewriter, maskOp->getLoc(),
 156:                                                      result.getType());
 157:           results.push_back(poisonOp);
 158:         }
 159:         maskOp->replaceAllUsesWith(results);
 160:         maskOp->erase();
 161:       }
 162:     }
 163:     peeledMaskOps.clear();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 164-169

```cpp
 164:   }
 165:   assert(moduleOp.getOps<triton::gpu::PredicateStageOp>().empty() &&
 166:          "PredicateStageOp should be resolved after the pipeline expansion");
 167:   assert(verify(moduleOp).succeeded());
 168:   resolveMaskOp(moduleOp);
 169: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 171-171

```cpp
 171: struct PipelinePass : public impl::TritonGPUPipelineBase<PipelinePass> {
```

- **EN:** Defines `PipelinePass`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `PipelinePass`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 173-173

```cpp
 173:   using impl::TritonGPUPipelineBase<PipelinePass>::TritonGPUPipelineBase;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 175-184

```cpp
 175:   void runOnOperation() override {
 176:     ModuleOp moduleOp = getOperation();
 177:     // Transform the loop by introducing async operations to prepare it for
 178:     // pipeline expansion.
 179:     lowerLoops(moduleOp);
 180:     if (dumpIntermediateSteps) {
 181:       ::mlir::triton::tools::mlirDumpsOrDbgs()
 182:           << "// -----// SoftwarePipeliner internal IR Dump After: LowerLoops\n"
 183:           << moduleOp << "\n\n\n";
 184:     }
```

- **EN:** Defines `runOnOperation`, the pass entry point. It gathers analysis information and applies the file's transformation logic to the current operation.
- **CN:** 这里定义 `runOnOperation`，即 pass 的入口函数。它会收集分析信息，并把本文件的变换逻辑应用到当前操作上。
### Lines 186-193

```cpp
 186:     // Apply the pipeline expansion.
 187:     expandLoops(moduleOp);
 188:     if (dumpIntermediateSteps) {
 189:       ::mlir::triton::tools::mlirDumpsOrDbgs()
 190:           << "// -----// SoftwarePipeliner internal IR Dump After: "
 191:              "ExpandLoops\n"
 192:           << moduleOp << "\n\n\n";
 193:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 195-196

```cpp
 195:     // Cleanup the IR from the pipeline attributes.
 196:     removePipeliningAttributes(moduleOp);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 198-198

```cpp
 198:     pipelineWgmma(moduleOp, numStages);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 200-201

```cpp
 200:     // schedule the waits
 201:     mlir::triton::updateWaits(getOperation());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 203-210

```cpp
 203:     // Clean up arithmetic before applying the next level of pipelining to
 204:     // simplify the IR.
 205:     auto arithDialect =
 206:         getOperation().getContext()->getLoadedDialect<arith::ArithDialect>();
 207:     RewritePatternSet patterns(getOperation().getContext());
 208:     arithDialect->getCanonicalizationPatterns(patterns);
 209:     if (applyPatternsGreedily(getOperation(), std::move(patterns)).failed())
 210:       return signalPassFailure();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 212-218

```cpp
 212:     {
 213:       SmallVector<scf::ForOp> loops;
 214:       getOperation()->walk([&](scf::ForOp forOp) {
 215:         // Bail out for loops with num_stage <= 1.
 216:         if (getNumStagesOrDefault(forOp, numStages) > 1)
 217:           loops.push_back(forOp);
 218:       });
```

- **EN:** Defines accessor/helper `getOperation` that exposes or updates operation state in a compact, reusable way. The implementation traverses IR operations to collect facts or apply rewrites globally.
- **CN:** 这里定义访问器/辅助函数 `getOperation`，以紧凑且可复用的方式读取或更新操作状态。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。
### Lines 220-225

```cpp
 220:       for (scf::ForOp forOp : loops) {
 221:         mlir::triton::pipelineTMAStores(forOp);
 222:       }
 223:     }
 224:   }
 225: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 227-229

```cpp
 227: } // namespace gpu
 228: } // namespace triton
 229: } // namespace mlir
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。

## Key Concepts / 关键概念
- **EN:** The main concern is pass-driven transformation around software pipeliner.
  **CN:** 核心关注点是围绕 Software Pipeliner 的 pass 驱动变换。
- **EN:** The code reasons at module scope rather than only on isolated operations.
  **CN:** 代码在模块范围内进行分析，而不只是处理单个操作。
- **EN:** Verification logic enforces structural invariants early in the pipeline.
  **CN:** 验证逻辑在编译流程早期强制检查结构不变量。
- **EN:** Axis information captures per-dimension contiguity, divisibility, or constancy facts.
  **CN:** AxisInfo 记录逐维的连续性、可整除性或常量性等信息。
- **EN:** Tensor Memory Accelerator related logic appears in this implementation.
  **CN:** 此实现涉及 Tensor Memory Accelerator（TMA）相关逻辑。
- **EN:** Synchronization and ordering constraints matter to preserve correctness across threads, warps, or memory spaces.
  **CN:** 同步与顺序约束很重要，它们保证跨线程、warp 或内存空间的正确性。

## Dependencies / 依赖关系
- **Internal Triton headers / Triton 内部头文件:** `triton/Analysis/AxisInfo.h`, `triton/Analysis/Utility.h`, `triton/Dialect/Triton/IR/Utility.h`, `triton/Dialect/Triton/Transforms/LoopPeeling.h`, `triton/Dialect/TritonGPU/IR/Dialect.h`, `triton/Dialect/TritonGPU/Transforms/Passes.h`, ... (+7 more)
- **MLIR headers / MLIR 头文件:** `mlir/Dialect/Tensor/IR/Tensor.h`, `mlir/Dialect/UB/IR/UBOps.h`, `mlir/IR/TypeUtilities.h`, `mlir/IR/Verifier.h`, `mlir/Interfaces/SideEffectInterfaces.h`, `mlir/Support/LLVM.h`, ... (+1 more)
- **LLVM headers / LLVM 头文件:** None
- **Standard/library headers / 标准或通用库头文件:** None
- **Generated fragments / 生成片段:** `triton/Dialect/TritonGPU/Transforms/Passes.h.inc`
- **Primary APIs used / 主要 API:** `ModuleOp`, `OpBuilder`
- **Pipeline role / 流程角色:** This file participates in Triton pass pipelines and usually expects upstream analyses or dialect invariants to have prepared the IR. / 本文件参与 Triton 的 pass 流程，通常依赖上游分析或方言不变量先把 IR 准备好。
