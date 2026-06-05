# PipelineExpander.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `lib/Dialect/TritonGPU/Transforms/Pipeliner/PipelineExpander.cpp`
- **Purpose / 作用:** **EN:** Implements the Pipeline Expander transformation or optimization pass for the TritonGPU pipeline. **CN:** 为 TritonGPU 编译流程实现与 Pipeline Expander 相关的变换或优化 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11

```cpp
   1: //===- LoopPipelining.cpp - Code to perform loop software pipelining-------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file implements loop software pipelining
  10: //
  11: //===----------------------------------------------------------------------===//
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 13-21

```cpp
  13: // Fork of upstream pipeliner. This will be merged upstream once things are
  14: // stable. Modifications so far are:
  15: // -Bug fix for def with a distance of 1 scheduled in stage 0.
  16: // -Support dynamic loops and predicate operations in the prologue.
  17: // -Support for non-index type for induction variable.
  18: // -Support source with distance of 1 used multiple stages later.
  19: // -Fix bug when a value yield is used outside the loop and the value def is not
  20: // in the last stage. If we are not peeling the epilgue we need to remap the
  21: // output correctly.
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 23-32

```cpp
  23: #include "mlir/Dialect/Arith/IR/Arith.h"
  24: #include "mlir/Dialect/SCF/IR/SCF.h"
  25: #include "mlir/Dialect/SCF/Transforms/Patterns.h"
  26: #include "mlir/Dialect/SCF/Utils/Utils.h"
  27: #include "mlir/IR/IRMapping.h"
  28: #include "mlir/IR/PatternMatch.h"
  29: #include "mlir/Transforms/RegionUtils.h"
  30: #include "llvm/ADT/MapVector.h"
  31: #include "llvm/Support/Debug.h"
  32: #include "llvm/Support/MathExtras.h"
```

- **EN:** Includes the interfaces this file depends on. Triton headers (None) provide domain-specific IR/support, MLIR headers (`Arith.h`, `SCF.h`, `Patterns.h`, `Utils.h`, ... (+3 more)) provide rewriting and analysis infrastructure, LLVM headers (`MapVector.h`, `Debug.h`, `MathExtras.h`) supply low-level utilities, and standard/library headers (None) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（None）提供领域特定 IR/支持逻辑，MLIR 头文件（`Arith.h`, `SCF.h`, `Patterns.h`, `Utils.h`, ... (+3 more)）提供重写与分析基础设施，LLVM 头文件（`MapVector.h`, `Debug.h`, `MathExtras.h`）提供底层工具，而标准/通用库头文件（None）提供通用能力。
### Lines 34-34

```cpp
  34: #include "triton/Dialect/TritonGPU/Transforms/PipelineExpander.h"
```

- **EN:** Includes the interfaces this file depends on. Triton headers (`PipelineExpander.h`) provide domain-specific IR/support, MLIR headers (None) provide rewriting and analysis infrastructure, LLVM headers (None) supply low-level utilities, and standard/library headers (None) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（`PipelineExpander.h`）提供领域特定 IR/支持逻辑，MLIR 头文件（None）提供重写与分析基础设施，LLVM 头文件（None）提供底层工具，而标准/通用库头文件（None）提供通用能力。
### Lines 36-38

```cpp
  36: // FIXME: PipelineExpander should not depend on Triton-specific headers!
  37: #include "triton/Dialect/TritonGPU/IR/Types.h"
  38: #include "triton/Dialect/TritonGPU/Transforms/PipeliningUtility.h"
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 40-42

```cpp
  40: #define DEBUG_TYPE "triton-loop-pipelining"
  41: #define DBGS() (llvm::dbgs() << "[" DEBUG_TYPE "]: ")
  42: #define LDBG(X) LLVM_DEBUG(DBGS() << X << "\n")
```

- **EN:** Defines debug logging helpers and compile-time tags used when tracing this pass or utility at runtime.
- **CN:** 这里定义调试日志辅助宏和编译期标签，用于在运行时跟踪该 pass 或工具。
### Lines 44-46

```cpp
  44: using namespace mlir;
  45: using namespace mlir::scf;
  46: using namespace mlir::triton;
```

- **EN:** Introduces namespace aliases/imports (`mlir`, `mlir::scf`, `mlir::triton`) so the rest of the file can use MLIR/Triton symbols without repetitive qualification.
- **CN:** 这里通过命名空间导入（`mlir`, `mlir::scf`, `mlir::triton`）减少后续代码中的重复限定，使 MLIR/Triton 符号使用更简洁。
### Lines 48-48

```cpp
  48: namespace {
```

- **EN:** Opens or closes the namespace nesting for (anonymous), keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 (anonymous) 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 50-56

```cpp
  50: /// Helper to keep internal information during pipelining transformation.
  51: struct LoopPipelinerInternal {
  52:   /// Coarse liverange information for ops used across stages.
  53:   struct LiverangeInfo {
  54:     unsigned lastUseStage = 0;
  55:     unsigned defStage = 0;
  56:   };
```

- **EN:** Defines `LoopPipelinerInternal`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `LoopPipelinerInternal`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 58-71

```cpp
  58: protected:
  59:   ForOp forOp;
  60:   unsigned maxStage = 0;
  61:   DenseMap<Operation *, unsigned> stages;
  62:   std::vector<Operation *> opOrder;
  63:   Value ub;
  64:   Value lb;
  65:   Value step;
  66:   bool dynamicLoop;
  67:   triton::PipeliningOption::AnnotationlFnType annotateFn = nullptr;
  68:   bool peelEpilogue;
  69:   triton::PipeliningOption::PredicateOpFnType predicateFn = nullptr;
  70:   triton::PipeliningOption::EmitPredicateStageFnType emitPredicateStageFn =
  71:       nullptr;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 73-77

```cpp
  73:   // When peeling the kernel we generate several version of each value for
  74:   // different stage of the prologue. This map tracks the mapping between
  75:   // original Values in the loop and the different versions
  76:   // peeled from the loop.
  77:   DenseMap<Value, llvm::SmallVector<Value>> valueMapping;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 79-81

```cpp
  79:   /// Assign a value to `valueMapping`, this means `val` represents the version
  80:   /// `idx` of `key` in the epilogue.
  81:   void setValueMapping(Value key, Value el, int64_t idx);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 83-86

```cpp
  83:   /// Return the defining op of the given value, if the Value is an argument of
  84:   /// the loop return the associated defining op in the loop and its distance to
  85:   /// the Value.
  86:   std::pair<Operation *, int64_t> getDefiningOpAndDistance(Value value);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 88-90

```cpp
  88:   /// Return true if the schedule is possible and return false otherwise. A
  89:   /// schedule is correct if all definitions are scheduled before uses.
  90:   bool verifySchedule();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 92-107

```cpp
  92: public:
  93:   /// Initialize the information for the given `op`, return true if it
  94:   /// satisfies the pre-condition to apply pipelining.
  95:   bool initializeLoopInfo(ForOp op, const triton::PipeliningOption &options);
  96:   /// Emits the prologue, this creates `maxStage - 1` part which will contain
  97:   /// operations from stages [0; i], where i is the part index.
  98:   LogicalResult emitPrologue(RewriterBase &rewriter);
  99:   /// Gather liverange information for Values that are used in a different stage
 100:   /// than its definition.
 101:   llvm::MapVector<Value, LiverangeInfo> analyzeCrossStageValues();
 102:   scf::ForOp createKernelLoop(
 103:       const llvm::MapVector<Value, LiverangeInfo> &crossStageValues,
 104:       RewriterBase &rewriter,
 105:       llvm::DenseMap<std::pair<Value, unsigned>, unsigned> &loopArgMap);
 106:   /// Emits the pipelined kernel. This clones loop operations following user
 107:   /// order and remaps operands defined in a different stage as their use.
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 108-117

```cpp
 108:   LogicalResult createKernel(
 109:       scf::ForOp newForOp,
 110:       const llvm::MapVector<Value, LiverangeInfo> &crossStageValues,
 111:       const llvm::DenseMap<std::pair<Value, unsigned>, unsigned> &loopArgMap,
 112:       RewriterBase &rewriter);
 113:   /// Emits the epilogue, this creates `maxStage - 1` part which will contain
 114:   /// operations from stages [i; maxStage], where i is the part index.
 115:   LogicalResult emitEpilogue(RewriterBase &rewriter,
 116:                              llvm::SmallVector<Value> &returnValues);
 117: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 119-128

```cpp
 119: /// Find operands of all the nested operations within `op`.
 120: static SetVector<Value> getNestedOperands(Operation *op) {
 121:   SetVector<Value> operands;
 122:   op->walk([&](Operation *nestedOp) {
 123:     for (Value operand : nestedOp->getOperands()) {
 124:       operands.insert(operand);
 125:     }
 126:   });
 127:   return operands;
 128: }
```

- **EN:** Defines accessor/helper `getNestedOperands` that exposes or updates operation state in a compact, reusable way. The implementation traverses IR operations to collect facts or apply rewrites globally.
- **CN:** 这里定义访问器/辅助函数 `getNestedOperands`，以紧凑且可复用的方式读取或更新操作状态。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。
### Lines 130-136

```cpp
 130: bool LoopPipelinerInternal::initializeLoopInfo(
 131:     ForOp op, const triton::PipeliningOption &options) {
 132:   LDBG("Start initializeLoopInfo");
 133:   forOp = op;
 134:   ub = forOp.getUpperBound();
 135:   lb = forOp.getLowerBound();
 136:   step = forOp.getStep();
```

- **EN:** Defines `LoopPipelinerInternal::initializeLoopInfo`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `LoopPipelinerInternal::initializeLoopInfo`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 138-143

```cpp
 138:   std::vector<std::pair<Operation *, unsigned>> schedule;
 139:   options.getScheduleFn(forOp, schedule);
 140:   if (schedule.empty()) {
 141:     LDBG("--empty schedule -> BAIL");
 142:     return false;
 143:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 145-150

```cpp
 145:   opOrder.reserve(schedule.size());
 146:   for (auto &opSchedule : schedule) {
 147:     maxStage = std::max(maxStage, opSchedule.second);
 148:     stages[opSchedule.first] = opSchedule.second;
 149:     opOrder.push_back(opSchedule.first);
 150:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 152-169

```cpp
 152:   dynamicLoop = true;
 153:   auto upperBoundCst = ub.getDefiningOp<arith::ConstantIndexOp>();
 154:   auto lowerBoundCst = lb.getDefiningOp<arith::ConstantIndexOp>();
 155:   auto stepCst = step.getDefiningOp<arith::ConstantIndexOp>();
 156:   if (!upperBoundCst || !lowerBoundCst || !stepCst) {
 157:     if (!options.supportDynamicLoops) {
 158:       LDBG("--dynamic loop not supported -> BAIL");
 159:       return false;
 160:     }
 161:   } else {
 162:     int64_t ubImm = upperBoundCst.value();
 163:     int64_t lbImm = lowerBoundCst.value();
 164:     int64_t stepImm = stepCst.value();
 165:     int64_t numIteration = llvm::divideCeilSigned(ubImm - lbImm, stepImm);
 166:     if (numIteration >= maxStage) {
 167:       dynamicLoop = false;
 168:     } else if (!options.supportDynamicLoops) {
 169:       LDBG("--fewer loop iterations than pipeline stages -> BAIL");
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 170-182

```cpp
 170:       return false;
 171:     }
 172:   }
 173:   peelEpilogue = options.peelEpilogue;
 174:   predicateFn = options.predicateFn;
 175:   if ((!peelEpilogue || dynamicLoop) && predicateFn == nullptr) {
 176:     LDBG("--no epilogue or predicate set -> BAIL");
 177:     return false;
 178:   }
 179:   emitPredicateStageFn = options.emitPredicateStageFn;
 180:   if (emitPredicateStageFn == nullptr) {
 181:     emitPredicateStageFn = mlir::triton::emitPredicateForStage;
 182:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 184-191

```cpp
 184:   // All operations need to have a stage.
 185:   for (Operation &op : forOp.getBody()->without_terminator()) {
 186:     if (!stages.contains(&op)) {
 187:       op.emitOpError("not assigned a pipeline stage");
 188:       LDBG("--op not assigned a pipeline stage: " << op << " -> BAIL");
 189:       return false;
 190:     }
 191:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 193-196

```cpp
 193:   if (!verifySchedule()) {
 194:     LDBG("--invalid schedule: " << op << " -> BAIL");
 195:     return false;
 196:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 198-215

```cpp
 198:   // Currently, we do not support assigning stages to ops in nested regions. The
 199:   // block of all operations assigned a stage should be the single `scf.for`
 200:   // body block.
 201:   for (const auto &[op, stageNum] : stages) {
 202:     (void)stageNum;
 203:     if (op == forOp.getBody()->getTerminator()) {
 204:       op->emitError("terminator should not be assigned a stage");
 205:       LDBG("--terminator should not be assigned stage: " << *op << " -> BAIL");
 206:       return false;
 207:     }
 208:     if (op->getBlock() != forOp.getBody()) {
 209:       op->emitOpError("the owning Block of all operations assigned a stage "
 210:                       "should be the loop body block");
 211:       LDBG("--the owning Block of all operations assigned a stage "
 212:            "should be the loop body block: "
 213:            << *op << " -> BAIL");
 214:       return false;
 215:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 216-216

```cpp
 216:   }
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 218-235

```cpp
 218:   // Support only loop-carried dependencies with a distance of one iteration or
 219:   // those defined outside of the loop. This means that any dependency within a
 220:   // loop should either be on the immediately preceding iteration, the current
 221:   // iteration, or on variables whose values are set before entering the loop.
 222:   for (auto &op : forOp.getBody()->without_terminator()) {
 223:     for (auto operand : getNestedOperands(&op)) {
 224:       auto [def, distance] = getDefiningOpAndDistance(operand);
 225:       if (!def)
 226:         continue;
 227:       if (distance > 1) {
 228:         LDBG("--only support loop carried dependency with a distance of 1 or "
 229:              "defined outside of the loop -> BAIL");
 230:         return false;
 231:       }
 232:     }
 233:   }
 234:   annotateFn = options.annotateFn;
 235:   return true;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 236-236

```cpp
 236: }
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 238-255

```cpp
 238: /// Compute unrolled cycles of each op (consumer) and verify that each op is
 239: /// scheduled after its operands (producers) while adjusting for the distance
 240: /// between producer and consumer.
 241: bool LoopPipelinerInternal::verifySchedule() {
 242:   int64_t numCylesPerIter = opOrder.size();
 243:   // Pre-compute the unrolled cycle of each op.
 244:   DenseMap<Operation *, int64_t> unrolledCyles;
 245:   for (int64_t cycle = 0; cycle < numCylesPerIter; cycle++) {
 246:     Operation *def = opOrder[cycle];
 247:     auto it = stages.find(def);
 248:     assert(it != stages.end());
 249:     int64_t stage = it->second;
 250:     unrolledCyles[def] = cycle + stage * numCylesPerIter;
 251:   }
 252:   for (Operation *consumer : opOrder) {
 253:     int64_t consumerCycle = unrolledCyles[consumer];
 254:     for (Value operand : getNestedOperands(consumer)) {
 255:       auto [producer, distance] = getDefiningOpAndDistance(operand);
```

- **EN:** Defines `LoopPipelinerInternal::verifySchedule`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `LoopPipelinerInternal::verifySchedule`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 256-273

```cpp
 256:       if (!producer)
 257:         continue;
 258:       auto it = unrolledCyles.find(producer);
 259:       // Skip producer coming from outside the loop.
 260:       if (it == unrolledCyles.end())
 261:         continue;
 262:       int64_t producerCycle = it->second;
 263:       if (consumerCycle < producerCycle - numCylesPerIter * distance) {
 264:         InFlightDiagnostic diag =
 265:             consumer->emitWarning("operation scheduled before its operands. "
 266:                                   "Pipelining will be disabled.");
 267:         diag.attachNote(producer->getLoc())
 268:             .append("operand defined here: ")
 269:             .appendOp(*producer, OpPrintingFlags().printGenericOpForm());
 270:         return false;
 271:       }
 272:     }
 273:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 274-275

```cpp
 274:   return true;
 275: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 277-294

```cpp
 277: /// Clone `op` and call `callback` on the cloned op's operands as well as any
 278: /// operands of nested ops that:
 279: /// 1) aren't defined within the new op or
 280: /// 2) are block arguments.
 281: static Operation *
 282: cloneAndUpdateOperands(RewriterBase &rewriter, Operation *op,
 283:                        function_ref<void(OpOperand *newOperand)> callback) {
 284:   Operation *clone = rewriter.clone(*op);
 285:   clone->walk<WalkOrder::PreOrder>([&](Operation *nested) {
 286:     // 'clone' itself will be visited first.
 287:     for (OpOperand &operand : nested->getOpOperands()) {
 288:       Operation *def = operand.get().getDefiningOp();
 289:       if ((def && !clone->isAncestor(def)) || isa<BlockArgument>(operand.get()))
 290:         callback(&operand);
 291:     }
 292:   });
 293:   return clone;
 294: }
```

- **EN:** Defines `cloneAndUpdateOperands`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `cloneAndUpdateOperands`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 296-301

```cpp
 296: LogicalResult LoopPipelinerInternal::emitPrologue(RewriterBase &rewriter) {
 297:   // Initialize the iteration argument to the loop initiale values.
 298:   for (auto [arg, operand] :
 299:        llvm::zip(forOp.getRegionIterArgs(), forOp.getInitsMutable())) {
 300:     setValueMapping(arg, operand.get(), 0);
 301:   }
```

- **EN:** Defines `LoopPipelinerInternal::emitPrologue`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `LoopPipelinerInternal::emitPrologue`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 303-313

```cpp
 303:   // If the incoming value to an iter arg from the loop yield is defined outside
 304:   // the loop, then that means the iter arg takes that value for all stages
 305:   // after the first stage.
 306:   auto yield = cast<scf::YieldOp>(forOp.getBody()->getTerminator());
 307:   for (auto [arg, operand] :
 308:        llvm::zip(forOp.getRegionIterArgs(), yield->getOpOperands())) {
 309:     if (forOp.getBodyRegion().isAncestor(operand.get().getParentRegion()))
 310:       continue;
 311:     for (int64_t i = 1; i < maxStage; ++i)
 312:       setValueMapping(arg, operand.get(), i);
 313:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 315-327

```cpp
 315:   Location loc = forOp.getLoc();
 316:   SmallVector<Value> predicates(maxStage);
 317:   for (int64_t i = 0; i < maxStage; i++) {
 318:     // special handling for induction variable as the increment is implicit.
 319:     // iv = lb + i * step
 320:     Type t = lb.getType();
 321:     Value iv = arith::AddIOp::create(
 322:         rewriter, loc, lb,
 323:         arith::MulIOp::create(
 324:             rewriter, loc, step,
 325:             arith::ConstantOp::create(rewriter, loc,
 326:                                       rewriter.getIntegerAttr(t, i))));
 327:     setValueMapping(forOp.getInductionVar(), iv, i);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 329-333

```cpp
 329:     if (dynamicLoop) {
 330:       // pred = ub > lb + (i * step)
 331:       predicates[i] = arith::CmpIOp::create(rewriter, loc,
 332:                                             arith::CmpIPredicate::slt, iv, ub);
 333:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 335-352

```cpp
 335:     for (Operation *op : opOrder) {
 336:       if (stages[op] > i)
 337:         continue;
 338:       Operation *newOp =
 339:           cloneAndUpdateOperands(rewriter, op, [&](OpOperand *newOperand) {
 340:             auto it = valueMapping.find(newOperand->get());
 341:             if (it != valueMapping.end()) {
 342:               Value replacement = it->second[i - stages[op]];
 343:               newOperand->set(replacement);
 344:             }
 345:           });
 346:       int predicateIdx = i - stages[op];
 347:       if (predicates[predicateIdx]) {
 348:         OpBuilder::InsertionGuard insertGuard(rewriter);
 349:         newOp = predicateFn(rewriter, newOp, predicates[predicateIdx]);
 350:         if (newOp == nullptr)
 351:           return failure();
 352:       }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 353-370

```cpp
 353:       if (annotateFn)
 354:         annotateFn(newOp, triton::PipeliningOption::PipelinerPart::Prologue, i);
 355:       for (unsigned destId : llvm::seq(unsigned(0), op->getNumResults())) {
 356:         Value source = newOp->getResult(destId);
 357:         // If the value is a loop carried dependency update the loop argument
 358:         for (OpOperand &operand : yield->getOpOperands()) {
 359:           if (operand.get() != op->getResult(destId))
 360:             continue;
 361:           if (predicates[predicateIdx] &&
 362:               !forOp.getResult(operand.getOperandNumber()).use_empty()) {
 363:             // If the value is used outside the loop, we need to make sure we
 364:             // return the correct version of it.
 365:             Value prevValue = valueMapping
 366:                 [forOp.getRegionIterArgs()[operand.getOperandNumber()]]
 367:                 [i - stages[op]];
 368:             source = arith::SelectOp::create(
 369:                 rewriter, loc, predicates[predicateIdx], source, prevValue);
 370:           }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 371-380

```cpp
 371:           setValueMapping(forOp.getRegionIterArgs()[operand.getOperandNumber()],
 372:                           source, i - stages[op] + 1);
 373:         }
 374:         setValueMapping(op->getResult(destId), newOp->getResult(destId),
 375:                         i - stages[op]);
 376:       }
 377:     }
 378:   }
 379:   return success();
 380: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 382-386

```cpp
 382: llvm::MapVector<Value, LoopPipelinerInternal::LiverangeInfo>
 383: LoopPipelinerInternal::analyzeCrossStageValues() {
 384:   llvm::MapVector<Value, LoopPipelinerInternal::LiverangeInfo> crossStageValues;
 385:   for (Operation *op : opOrder) {
 386:     unsigned stage = stages[op];
```

- **EN:** Defines `LoopPipelinerInternal::analyzeCrossStageValues`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `LoopPipelinerInternal::analyzeCrossStageValues`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 388-400

```cpp
 388:     auto analyzeOperand = [&](OpOperand &operand) {
 389:       auto [def, distance] = getDefiningOpAndDistance(operand.get());
 390:       if (!def)
 391:         return;
 392:       auto defStage = stages.find(def);
 393:       if (defStage == stages.end() || defStage->second == stage ||
 394:           defStage->second == stage + distance)
 395:         return;
 396:       assert(stage > defStage->second);
 397:       LiverangeInfo &info = crossStageValues[operand.get()];
 398:       info.defStage = defStage->second;
 399:       info.lastUseStage = std::max(info.lastUseStage, stage);
 400:     };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 402-409

```cpp
 402:     for (OpOperand &operand : op->getOpOperands())
 403:       analyzeOperand(operand);
 404:     visitUsedValuesDefinedAbove(op->getRegions(), [&](OpOperand *operand) {
 405:       analyzeOperand(*operand);
 406:     });
 407:   }
 408:   return crossStageValues;
 409: }
```

- **EN:** Defines `visitUsedValuesDefinedAbove`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `visitUsedValuesDefinedAbove`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 411-414

```cpp
 411: std::pair<Operation *, int64_t>
 412: LoopPipelinerInternal::getDefiningOpAndDistance(Value value) {
 413:   return triton::getDefiningOpAndDistance(forOp, value);
 414: }
```

- **EN:** Defines accessor/helper `LoopPipelinerInternal::getDefiningOpAndDistance` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `LoopPipelinerInternal::getDefiningOpAndDistance`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 416-431

```cpp
 416: scf::ForOp LoopPipelinerInternal::createKernelLoop(
 417:     const llvm::MapVector<Value, LoopPipelinerInternal::LiverangeInfo>
 418:         &crossStageValues,
 419:     RewriterBase &rewriter,
 420:     llvm::DenseMap<std::pair<Value, unsigned>, unsigned> &loopArgMap) {
 421:   // Creates the list of initial values associated to values used across
 422:   // stages. The initial values come from the prologue created above.
 423:   // Keep track of the kernel argument associated to each version of the
 424:   // values passed to the kernel.
 425:   llvm::SmallVector<Value> newLoopArg;
 426:   // For existing loop argument initialize them with the right version from the
 427:   // prologue.
 428:   for (const auto &retVal :
 429:        llvm::enumerate(forOp.getBody()->getTerminator()->getOperands())) {
 430:     Operation *def = retVal.value().getDefiningOp();
 431:     auto defStage = stages.find(def);
```

- **EN:** Defines helper `LoopPipelinerInternal::createKernelLoop` that computes or constructs intermediate data used by the surrounding transformation. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义辅助函数 `LoopPipelinerInternal::createKernelLoop`，用于计算或构造外围变换所需的中间数据。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 432-449

```cpp
 432:     if (defStage != stages.end()) {
 433:       Value valueVersion =
 434:           valueMapping[forOp.getRegionIterArgs()[retVal.index()]]
 435:                       [maxStage - defStage->second];
 436:       assert(valueVersion);
 437:       newLoopArg.push_back(valueVersion);
 438:     } else
 439:       newLoopArg.push_back(forOp.getInitArgs()[retVal.index()]);
 440:   }
 441:   for (auto escape : crossStageValues) {
 442:     LiverangeInfo &info = escape.second;
 443:     Value value = escape.first;
 444:     for (unsigned stageIdx = 0; stageIdx < info.lastUseStage - info.defStage;
 445:          stageIdx++) {
 446:       Value valueVersion =
 447:           valueMapping[value][maxStage - info.lastUseStage + stageIdx];
 448:       assert(valueVersion);
 449:       newLoopArg.push_back(valueVersion);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 450-453

```cpp
 450:       loopArgMap[std::make_pair(value, info.lastUseStage - info.defStage -
 451:                                            stageIdx)] = newLoopArg.size() - 1;
 452:     }
 453:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 455-472

```cpp
 455:   // Create the new kernel loop. When we peel the epilgue we need to peel
 456:   // `numStages - 1` iterations. Then we adjust the upper bound to remove those
 457:   // iterations.
 458:   Value newUb = forOp.getUpperBound();
 459:   if (peelEpilogue) {
 460:     Type t = ub.getType();
 461:     Location loc = forOp.getLoc();
 462:     // newUb = ub - maxStage * step
 463:     Value maxStageValue = arith::ConstantOp::create(
 464:         rewriter, loc, rewriter.getIntegerAttr(t, maxStage));
 465:     Value maxStageByStep =
 466:         arith::MulIOp::create(rewriter, loc, step, maxStageValue);
 467:     newUb = arith::SubIOp::create(rewriter, loc, ub, maxStageByStep);
 468:   }
 469:   auto newForOp =
 470:       scf::ForOp::create(rewriter, forOp.getLoc(), forOp.getLowerBound(), newUb,
 471:                          forOp.getStep(), newLoopArg);
 472:   newForOp->setAttrs(forOp->getAttrs());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 473-478

```cpp
 473:   // When there are no iter args, the loop body terminator will be created.
 474:   // Since we always create it below, remove the terminator if it was created.
 475:   if (!newForOp.getBody()->empty())
 476:     rewriter.eraseOp(newForOp.getBody()->getTerminator());
 477:   return newForOp;
 478: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. When the lowered form has no direct result, the original op is erased.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 当降级后的形式不再需要直接结果时，原始操作会被删除。
### Lines 480-486

```cpp
 480: LogicalResult LoopPipelinerInternal::createKernel(
 481:     scf::ForOp newForOp,
 482:     const llvm::MapVector<Value, LoopPipelinerInternal::LiverangeInfo>
 483:         &crossStageValues,
 484:     const llvm::DenseMap<std::pair<Value, unsigned>, unsigned> &loopArgMap,
 485:     RewriterBase &rewriter) {
 486:   valueMapping.clear();
```

- **EN:** Defines helper `LoopPipelinerInternal::createKernel` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `LoopPipelinerInternal::createKernel`，用于计算或构造外围变换所需的中间数据。
### Lines 488-504

```cpp
 488:   // Create the kernel, we clone instruction based on the order given by
 489:   // user and remap operands coming from a previous stages.
 490:   rewriter.setInsertionPoint(newForOp.getBody(), newForOp.getBody()->begin());
 491:   IRMapping mapping;
 492:   mapping.map(forOp.getInductionVar(), newForOp.getInductionVar());
 493:   for (const auto &arg : llvm::enumerate(forOp.getRegionIterArgs())) {
 494:     mapping.map(arg.value(), newForOp.getRegionIterArgs()[arg.index()]);
 495:   }
 496:   SmallVector<Value> predicates(maxStage + 1, nullptr);
 497:   if (!peelEpilogue) {
 498:     // Create a predicate for each stage except the last stage.
 499:     for (unsigned i = 0; i < maxStage; i++) {
 500:       // c = ub - (maxStage - i) * step
 501:       predicates[i] = emitPredicateStageFn(rewriter, newForOp.getInductionVar(),
 502:                                            ub, step, maxStage, i);
 503:     }
 504:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 505-520

```cpp
 505:   for (Operation *op : opOrder) {
 506:     int64_t useStage = stages[op];
 507:     auto *newOp = rewriter.clone(*op, mapping);
 508:     SmallVector<OpOperand *> operands;
 509:     // Collect all the operands for the cloned op and its nested ops.
 510:     op->walk([&operands](Operation *nestedOp) {
 511:       for (OpOperand &operand : nestedOp->getOpOperands()) {
 512:         operands.push_back(&operand);
 513:       }
 514:     });
 515:     for (OpOperand *operand : operands) {
 516:       Operation *nestedNewOp = mapping.lookup(operand->getOwner());
 517:       // Special case for the induction variable uses. We replace it with a
 518:       // version incremented based on the stage where it is used.
 519:       if (operand->get() == forOp.getInductionVar()) {
 520:         rewriter.setInsertionPoint(newOp);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 522-539

```cpp
 522:         // offset = (maxStage - stages[op]) * step
 523:         Type t = step.getType();
 524:         Value offset = arith::MulIOp::create(
 525:             rewriter, forOp.getLoc(), step,
 526:             arith::ConstantOp::create(
 527:                 rewriter, forOp.getLoc(),
 528:                 rewriter.getIntegerAttr(t, maxStage - stages[op])));
 529:         Value iv = arith::AddIOp::create(rewriter, forOp.getLoc(),
 530:                                          newForOp.getInductionVar(), offset);
 531:         nestedNewOp->setOperand(operand->getOperandNumber(), iv);
 532:         rewriter.setInsertionPointAfter(newOp);
 533:         continue;
 534:       }
 535:       Value source = operand->get();
 536:       auto arg = dyn_cast<BlockArgument>(source);
 537:       if (arg && arg.getOwner() == forOp.getBody()) {
 538:         Value ret = forOp.getBody()->getTerminator()->getOperand(
 539:             arg.getArgNumber() - 1);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 540-555

```cpp
 540:         if (forOp.isDefinedOutsideOfLoop(ret)) {
 541:           // Special case for values defined outside the loop accessed with
 542:           // distance 1.
 543:           if (useStage != maxStage) {
 544:             nestedNewOp->setOperand(operand->getOperandNumber(), ret);
 545:           }
 546:           continue;
 547:         }
 548:         Operation *dep = ret.getDefiningOp();
 549:         if (!dep)
 550:           continue;
 551:         auto stageDep = stages.find(dep);
 552:         if (stageDep == stages.end() || stageDep->second == useStage)
 553:           continue;
 554:         // If the value is a loop carried value coming from stage N + 1 remap,
 555:         // it will become a direct use.
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 556-573

```cpp
 556:         if (stageDep->second == useStage + 1) {
 557:           nestedNewOp->setOperand(operand->getOperandNumber(),
 558:                                   mapping.lookupOrDefault(ret));
 559:           continue;
 560:         }
 561:         source = ret;
 562:       }
 563:       // For operands defined in a previous stage we need to remap it to use
 564:       // the correct region argument. We look for the right version of the
 565:       // Value based on the stage where it is used.
 566:       Operation *def = source.getDefiningOp();
 567:       if (!def)
 568:         continue;
 569:       auto stageDef = stages.find(def);
 570:       if (stageDef == stages.end() || stageDef->second == useStage)
 571:         continue;
 572:       auto remap = loopArgMap.find(
 573:           std::make_pair(operand->get(), useStage - stageDef->second));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 574-577

```cpp
 574:       assert(remap != loopArgMap.end());
 575:       nestedNewOp->setOperand(operand->getOperandNumber(),
 576:                               newForOp.getRegionIterArgs()[remap->second]);
 577:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 579-590

```cpp
 579:     if (predicates[useStage]) {
 580:       OpBuilder::InsertionGuard insertGuard(rewriter);
 581:       newOp = predicateFn(rewriter, newOp, predicates[useStage]);
 582:       if (!newOp)
 583:         return failure();
 584:       // Remap the results to the new predicated one.
 585:       for (auto values : llvm::zip(op->getResults(), newOp->getResults()))
 586:         mapping.map(std::get<0>(values), std::get<1>(values));
 587:     }
 588:     if (annotateFn)
 589:       annotateFn(newOp, triton::PipeliningOption::PipelinerPart::Kernel, 0);
 590:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 592-608

```cpp
 592:   // Collect the Values that need to be returned by the forOp. For each
 593:   // value we need to have `LastUseStage - DefStage` number of versions
 594:   // returned.
 595:   // We create a mapping between original values and the associated loop
 596:   // returned values that will be needed by the epilogue.
 597:   llvm::SmallVector<Value> yieldOperands;
 598:   for (OpOperand &yieldOperand :
 599:        forOp.getBody()->getTerminator()->getOpOperands()) {
 600:     Value source = mapping.lookupOrDefault(yieldOperand.get());
 601:     // When we don't peel the epilogue and the yield value is used outside the
 602:     // loop we need to make sure we return the version from numStages -
 603:     // defStage.
 604:     if (!peelEpilogue &&
 605:         !forOp.getResult(yieldOperand.getOperandNumber()).use_empty()) {
 606:       Operation *def = getDefiningOpAndDistance(yieldOperand.get()).first;
 607:       if (def) {
 608:         auto defStage = stages.find(def);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 609-619

```cpp
 609:         if (defStage != stages.end() && defStage->second < maxStage) {
 610:           Value pred = predicates[defStage->second];
 611:           source = arith::SelectOp::create(
 612:               rewriter, pred.getLoc(), pred, source,
 613:               newForOp.getBody()
 614:                   ->getArguments()[yieldOperand.getOperandNumber() + 1]);
 615:         }
 616:       }
 617:     }
 618:     yieldOperands.push_back(source);
 619:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 621-638

```cpp
 621:   for (auto &it : crossStageValues) {
 622:     int64_t version = maxStage - it.second.lastUseStage + 1;
 623:     unsigned numVersionReturned = it.second.lastUseStage - it.second.defStage;
 624:     // add the original version to yield ops.
 625:     // If there is a live range spanning across more than 2 stages we need to
 626:     // add extra arg.
 627:     for (unsigned i = 1; i < numVersionReturned; i++) {
 628:       setValueMapping(it.first, newForOp->getResult(yieldOperands.size()),
 629:                       version++);
 630:       yieldOperands.push_back(
 631:           newForOp.getBody()->getArguments()[yieldOperands.size() + 1 +
 632:                                              newForOp.getNumInductionVars()]);
 633:     }
 634:     setValueMapping(it.first, newForOp->getResult(yieldOperands.size()),
 635:                     version++);
 636:     yieldOperands.push_back(mapping.lookupOrDefault(it.first));
 637:   }
 638:   // Map the yield operand to the forOp returned value.
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 639-655

```cpp
 639:   for (const auto &retVal :
 640:        llvm::enumerate(forOp.getBody()->getTerminator()->getOperands())) {
 641:     Operation *def = retVal.value().getDefiningOp();
 642:     auto defStage = stages.find(def);
 643:     if (defStage == stages.end()) {
 644:       for (unsigned int stage = 1; stage <= maxStage; stage++)
 645:         setValueMapping(forOp.getRegionIterArgs()[retVal.index()],
 646:                         retVal.value(), stage);
 647:     } else if (defStage->second > 0) {
 648:       setValueMapping(forOp.getRegionIterArgs()[retVal.index()],
 649:                       newForOp->getResult(retVal.index()),
 650:                       maxStage - defStage->second + 1);
 651:     }
 652:   }
 653:   scf::YieldOp::create(rewriter, forOp.getLoc(), yieldOperands);
 654:   return success();
 655: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 657-663

```cpp
 657: LogicalResult
 658: LoopPipelinerInternal::emitEpilogue(RewriterBase &rewriter,
 659:                                     llvm::SmallVector<Value> &returnValues) {
 660:   Location loc = forOp.getLoc();
 661:   Type t = lb.getType();
 662:   // Emit different versions of the induction variable. They will be
 663:   // removed by dead code if not used.
```

- **EN:** Defines `LoopPipelinerInternal::emitEpilogue`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `LoopPipelinerInternal::emitEpilogue`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 665-668

```cpp
 665:   auto createConst = [&](int v) {
 666:     return arith::ConstantOp::create(rewriter, loc,
 667:                                      rewriter.getIntegerAttr(t, v));
 668:   };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 670-678

```cpp
 670:   // total_iterations = cdiv(range_diff, step);
 671:   // - range_diff = ub - lb
 672:   // - total_iterations = (range_diff + step + (step < 0 ? 1 : -1)) / step
 673:   Value zero = createConst(0);
 674:   Value one = createConst(1);
 675:   Value stepLessZero = arith::CmpIOp::create(
 676:       rewriter, loc, arith::CmpIPredicate::slt, step, zero);
 677:   Value stepDecr = arith::SelectOp::create(rewriter, loc, stepLessZero, one,
 678:                                            createConst(-1));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 680-685

```cpp
 680:   Value rangeDiff = arith::SubIOp::create(rewriter, loc, ub, lb);
 681:   Value rangeIncrStep = arith::AddIOp::create(rewriter, loc, rangeDiff, step);
 682:   Value rangeDecr =
 683:       arith::AddIOp::create(rewriter, loc, rangeIncrStep, stepDecr);
 684:   Value totalIterations =
 685:       arith::DivSIOp::create(rewriter, loc, rangeDecr, step);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 687-692

```cpp
 687:   // If total_iters < max_stage, start the epilogue at zero to match the
 688:   // ramp-up in the prologue.
 689:   // start_iter = max(0, total_iters - max_stage)
 690:   Value iterI = arith::SubIOp::create(rewriter, loc, totalIterations,
 691:                                       createConst(maxStage));
 692:   iterI = arith::MaxSIOp::create(rewriter, loc, zero, iterI);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 694-695

```cpp
 694:   // Capture predicates for dynamic loops.
 695:   SmallVector<Value> predicates(maxStage + 1);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 697-700

```cpp
 697:   for (int64_t i = 1; i <= maxStage; i++) {
 698:     // newLastIter = lb + step * iterI
 699:     Value newlastIter = arith::AddIOp::create(
 700:         rewriter, loc, lb, arith::MulIOp::create(rewriter, loc, step, iterI));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 702-702

```cpp
 702:     setValueMapping(forOp.getInductionVar(), newlastIter, i);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 704-705

```cpp
 704:     // increment to next iterI
 705:     iterI = arith::AddIOp::create(rewriter, loc, iterI, one);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 707-714

```cpp
 707:     if (dynamicLoop) {
 708:       // Disable stages when `i` is greater than total_iters.
 709:       // pred = total_iters >= i
 710:       predicates[i] =
 711:           arith::CmpIOp::create(rewriter, loc, arith::CmpIPredicate::sge,
 712:                                 totalIterations, createConst(i));
 713:     }
 714:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 716-732

```cpp
 716:   // Emit `maxStage - 1` epilogue part that includes operations from stages
 717:   // [i; maxStage].
 718:   for (int64_t i = 1; i <= maxStage; i++) {
 719:     SmallVector<std::pair<Value, unsigned>> returnMap(returnValues.size());
 720:     for (Operation *op : opOrder) {
 721:       if (stages[op] < i)
 722:         continue;
 723:       unsigned currentVersion = maxStage - stages[op] + i;
 724:       unsigned nextVersion = currentVersion + 1;
 725:       Operation *newOp =
 726:           cloneAndUpdateOperands(rewriter, op, [&](OpOperand *newOperand) {
 727:             auto it = valueMapping.find(newOperand->get());
 728:             if (it != valueMapping.end()) {
 729:               Value replacement = it->second[currentVersion];
 730:               newOperand->set(replacement);
 731:             }
 732:           });
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 733-747

```cpp
 733:       if (dynamicLoop) {
 734:         OpBuilder::InsertionGuard insertGuard(rewriter);
 735:         newOp = predicateFn(rewriter, newOp, predicates[currentVersion]);
 736:         if (!newOp)
 737:           return failure();
 738:       }
 739:       if (annotateFn)
 740:         annotateFn(newOp, triton::PipeliningOption::PipelinerPart::Epilogue,
 741:                    i - 1);
 742:       for (auto [opRes, newRes] :
 743:            llvm::zip(op->getResults(), newOp->getResults())) {
 744:         setValueMapping(opRes, newRes, currentVersion);
 745:         // If the value is a loop carried dependency update the loop argument
 746:         // mapping and keep track of the last version to replace the original
 747:         // forOp uses.
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 748-765

```cpp
 748:         for (OpOperand &operand :
 749:              forOp.getBody()->getTerminator()->getOpOperands()) {
 750:           if (operand.get() != opRes)
 751:             continue;
 752:           // If the version is greater than maxStage it means it maps to the
 753:           // original forOp returned value.
 754:           unsigned ri = operand.getOperandNumber();
 755:           returnValues[ri] = newRes;
 756:           Value mapVal = forOp.getRegionIterArgs()[ri];
 757:           returnMap[ri] = std::make_pair(mapVal, currentVersion);
 758:           if (nextVersion <= maxStage)
 759:             setValueMapping(mapVal, newRes, nextVersion);
 760:         }
 761:       }
 762:     }
 763:     if (dynamicLoop) {
 764:       // Select return values from this stage (live outs) based on predication.
 765:       // If the stage is valid select the peeled value, else use previous stage
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 766-783

```cpp
 766:       // value.
 767:       for (auto pair : llvm::enumerate(returnValues)) {
 768:         unsigned ri = pair.index();
 769:         auto [mapVal, currentVersion] = returnMap[ri];
 770:         if (mapVal) {
 771:           unsigned nextVersion = currentVersion + 1;
 772:           Value pred = predicates[currentVersion];
 773:           Value prevValue = valueMapping[mapVal][currentVersion];
 774:           auto selOp = arith::SelectOp::create(rewriter, loc, pred,
 775:                                                pair.value(), prevValue);
 776:           returnValues[ri] = selOp;
 777:           if (nextVersion <= maxStage)
 778:             setValueMapping(mapVal, selOp, nextVersion);
 779:         }
 780:       }
 781:     }
 782:   }
 783:   return success();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 784-784

```cpp
 784: }
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 786-796

```cpp
 786: void LoopPipelinerInternal::setValueMapping(Value key, Value el, int64_t idx) {
 787:   auto it = valueMapping.find(key);
 788:   // If the value is not in the map yet add a vector big enough to store all
 789:   // versions.
 790:   if (it == valueMapping.end())
 791:     it =
 792:         valueMapping
 793:             .insert(std::make_pair(key, llvm::SmallVector<Value>(maxStage + 1)))
 794:             .first;
 795:   it->second[idx] = el;
 796: }
```

- **EN:** Defines accessor/helper `LoopPipelinerInternal::setValueMapping` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `LoopPipelinerInternal::setValueMapping`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 798-798

```cpp
 798: } // namespace
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 800-808

```cpp
 800: FailureOr<ForOp>
 801: mlir::triton::pipelineForLoop(RewriterBase &rewriter, ForOp forOp,
 802:                               const triton::PipeliningOption &options,
 803:                               bool *modifiedIR) {
 804:   if (modifiedIR)
 805:     *modifiedIR = false;
 806:   LoopPipelinerInternal pipeliner;
 807:   if (!pipeliner.initializeLoopInfo(forOp, options))
 808:     return failure();
```

- **EN:** Defines `mlir::triton::pipelineForLoop`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `mlir::triton::pipelineForLoop`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 810-811

```cpp
 810:   if (modifiedIR)
 811:     *modifiedIR = true;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 813-815

```cpp
 813:   // 1. Emit prologue.
 814:   if (failed(pipeliner.emitPrologue(rewriter)))
 815:     return failure();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 817-822

```cpp
 817:   // 2. Track values used across stages. When a value cross stages it will
 818:   // need to be passed as loop iteration arguments.
 819:   // We first collect the values that are used in a different stage than where
 820:   // they are defined.
 821:   llvm::MapVector<Value, LoopPipelinerInternal::LiverangeInfo>
 822:       crossStageValues = pipeliner.analyzeCrossStageValues();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 824-835

```cpp
 824:   // Mapping between original loop values used cross stage and the block
 825:   // arguments associated after pipelining. A Value may map to several
 826:   // arguments if its liverange spans across more than 2 stages.
 827:   llvm::DenseMap<std::pair<Value, unsigned>, unsigned> loopArgMap;
 828:   // 3. Create the new kernel loop and return the block arguments mapping.
 829:   ForOp newForOp =
 830:       pipeliner.createKernelLoop(crossStageValues, rewriter, loopArgMap);
 831:   // Create the kernel block, order ops based on user choice and remap
 832:   // operands.
 833:   if (failed(pipeliner.createKernel(newForOp, crossStageValues, loopArgMap,
 834:                                     rewriter)))
 835:     return failure();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 837-849

```cpp
 837:   llvm::SmallVector<Value> returnValues =
 838:       newForOp.getResults().take_front(forOp->getNumResults());
 839:   if (options.peelEpilogue) {
 840:     // 4. Emit the epilogue after the new forOp.
 841:     rewriter.setInsertionPointAfter(newForOp);
 842:     if (failed(pipeliner.emitEpilogue(rewriter, returnValues)))
 843:       return failure();
 844:   }
 845:   // 5. Erase the original loop and replace the uses with the epilogue output.
 846:   if (forOp->getNumResults() > 0)
 847:     rewriter.replaceOp(forOp, returnValues);
 848:   else
 849:     rewriter.eraseOp(forOp);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. It finishes by replacing the original operation with the lowered form. When the lowered form has no direct result, the original op is erased.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 它会在转换成功后用降级后的形式替换原始操作。 当降级后的形式不再需要直接结果时，原始操作会被删除。
### Lines 851-852

```cpp
 851:   return newForOp;
 852: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 854-868

```cpp
 854: Value mlir::triton::emitPredicateForStage(RewriterBase &rewriter,
 855:                                           Value inductionVar, Value upperBound,
 856:                                           Value step, uint64_t maxStage,
 857:                                           uint64_t stage) {
 858:   auto loc = inductionVar.getLoc();
 859:   auto type = inductionVar.getType();
 860:   Value c = arith::SubIOp::create(
 861:       rewriter, loc, upperBound,
 862:       arith::MulIOp::create(
 863:           rewriter, loc, step,
 864:           arith::ConstantOp::create(
 865:               rewriter, loc, rewriter.getIntegerAttr(type, maxStage - stage))));
 866:   return arith::CmpIOp::create(rewriter, loc, arith::CmpIPredicate::slt,
 867:                                inductionVar, c);
 868: }
```

- **EN:** Defines `mlir::triton::emitPredicateForStage`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `mlir::triton::emitPredicateForStage`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。

## Key Concepts / 关键概念
- **EN:** The main concern is pass-driven transformation around pipeline expander.
  **CN:** 核心关注点是围绕 Pipeline Expander 的 pass 驱动变换。

## Dependencies / 依赖关系
- **Internal Triton headers / Triton 内部头文件:** `triton/Dialect/TritonGPU/Transforms/PipelineExpander.h`, `triton/Dialect/TritonGPU/IR/Types.h`, `triton/Dialect/TritonGPU/Transforms/PipeliningUtility.h`
- **MLIR headers / MLIR 头文件:** `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/SCF/IR/SCF.h`, `mlir/Dialect/SCF/Transforms/Patterns.h`, `mlir/Dialect/SCF/Utils/Utils.h`, `mlir/IR/IRMapping.h`, `mlir/IR/PatternMatch.h`, ... (+1 more)
- **LLVM headers / LLVM 头文件:** `llvm/ADT/MapVector.h`, `llvm/Support/Debug.h`, `llvm/Support/MathExtras.h`
- **Standard/library headers / 标准或通用库头文件:** None
- **Primary APIs used / 主要 API:** `OpBuilder`
- **Pipeline role / 流程角色:** This file participates in Triton pass pipelines and usually expects upstream analyses or dialect invariants to have prepared the IR. / 本文件参与 Triton 的 pass 流程，通常依赖上游分析或方言不变量先把 IR 准备好。
