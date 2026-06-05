# PipeliningUtility.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `lib/Dialect/TritonGPU/Transforms/Pipeliner/PipeliningUtility.cpp`
- **Purpose / 作用:** **EN:** Implements the Pipelining Utility transformation or optimization pass for the TritonGPU pipeline. **CN:** 为 TritonGPU 编译流程实现与 Pipelining Utility 相关的变换或优化 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```cpp
   1: #include "triton/Dialect/TritonGPU/Transforms/PipeliningUtility.h"
   2: #include "mlir/Analysis/TopologicalSortUtils.h"
   3: #include "mlir/Dialect/LLVMIR/LLVMDialect.h"
   4: #include "mlir/Dialect/Tensor/IR/Tensor.h"
   5: #include "mlir/IR/ImplicitLocOpBuilder.h"
   6: #include "mlir/IR/TypeUtilities.h"
   7: #include "mlir/Interfaces/SideEffectInterfaces.h"
   8: #include "mlir/Support/LLVM.h"
   9: #include "mlir/Transforms/GreedyPatternRewriteDriver.h"
  10: #include "triton/Analysis/AxisInfo.h"
  11: #include "triton/Dialect/Triton/IR/Utility.h"
  12: #include "triton/Dialect/TritonGPU/IR/Dialect.h"
  13: #include "triton/Dialect/TritonGPU/Transforms/Passes.h"
  14: #include "triton/Dialect/TritonGPU/Transforms/Schedule.h"
  15: #include "triton/Dialect/TritonGPU/Transforms/Utility.h"
  16: #include "triton/Dialect/TritonNvidiaGPU/IR/Dialect.h"
  17: #include "triton/Dialect/TritonNvidiaGPU/Transforms/TMAUtilities.h"
  18: #include "triton/Tools/LayoutUtils.h"
```

- **EN:** Includes the interfaces this file depends on. Triton headers (`PipeliningUtility.h`, `AxisInfo.h`, `Utility.h`, `Dialect.h`, ... (+6 more)) provide domain-specific IR/support, MLIR headers (`TopologicalSortUtils.h`, `LLVMDialect.h`, `Tensor.h`, `ImplicitLocOpBuilder.h`, ... (+4 more)) provide rewriting and analysis infrastructure, LLVM headers (None) supply low-level utilities, and standard/library headers (None) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（`PipeliningUtility.h`, `AxisInfo.h`, `Utility.h`, `Dialect.h`, ... (+6 more)）提供领域特定 IR/支持逻辑，MLIR 头文件（`TopologicalSortUtils.h`, `LLVMDialect.h`, `Tensor.h`, `ImplicitLocOpBuilder.h`, ... (+4 more)）提供重写与分析基础设施，LLVM 头文件（None）提供底层工具，而标准/通用库头文件（None）提供通用能力。
### Lines 19-21

```cpp
  19: #include "llvm/Support/Casting.h"
  20: #include "llvm/Support/Debug.h"
  21: #include <queue>
```

- **EN:** Includes the interfaces this file depends on. Triton headers (None) provide domain-specific IR/support, MLIR headers (None) provide rewriting and analysis infrastructure, LLVM headers (`Casting.h`, `Debug.h`) supply low-level utilities, and standard/library headers (`queue`) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（None）提供领域特定 IR/支持逻辑，MLIR 头文件（None）提供重写与分析基础设施，LLVM 头文件（`Casting.h`, `Debug.h`）提供底层工具，而标准/通用库头文件（`queue`）提供通用能力。
### Lines 23-25

```cpp
  23: #define DEBUG_TYPE "triton-loop-pipeline"
  24: #define DBGS() (llvm::dbgs() << "[" DEBUG_TYPE "]: ")
  25: #define LDBG(X) LLVM_DEBUG(DBGS() << X << "\n")
```

- **EN:** Defines debug logging helpers and compile-time tags used when tracing this pass or utility at runtime.
- **CN:** 这里定义调试日志辅助宏和编译期标签，用于在运行时跟踪该 pass 或工具。
### Lines 27-30

```cpp
  27: using namespace mlir;
  28: namespace tt = mlir::triton;
  29: namespace ttg = mlir::triton::gpu;
  30: namespace ttng = mlir::triton::nvidia_gpu;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 32-34

```cpp
  32: //===----------------------------------------------------------------------===//
  33: // Hoisting Utilities
  34: //===----------------------------------------------------------------------===//
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 36-40

```cpp
  36: bool triton::isPureScalarOp(Operation *op) {
  37:   auto isScalar = [](Type type) { return type.isIntOrIndexOrFloat(); };
  38:   return isPure(op) && llvm::all_of(op->getOperandTypes(), isScalar) &&
  39:          llvm::all_of(op->getResultTypes(), isScalar);
  40: }
```

- **EN:** Defines `triton::isPureScalarOp`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `triton::isPureScalarOp`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 42-50

```cpp
  42: bool triton::getDominatingValueSetOpsToHoist(
  43:     DominanceInfo &domInfo, Operation *refOp, ArrayRef<Value> valueSet,
  44:     llvm::SetVector<Operation *> &toHoist,
  45:     function_ref<bool(Operation *)> canHoist,
  46:     function_ref<bool(BlockArgument)> canUseArg) {
  47:   // The set of operations below `refOp` that are being checked if they can be
  48:   // hoisted. This set prevents checking operations twice but also if the
  49:   // computation can be hoisted, this becomes the set of operations to hoist.
  50:   llvm::SetVector<Operation *> visited;
```

- **EN:** Defines accessor/helper `triton::getDominatingValueSetOpsToHoist` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `triton::getDominatingValueSetOpsToHoist`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 52-56

```cpp
  52:   // Climb the use-def chain breadth-first so that operations can be hoisted in
  53:   // the reverse visitation order.
  54:   std::queue<Value> queue;
  55:   for (Value value : valueSet)
  56:     queue.push(value);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 58-60

```cpp
  58:   while (!queue.empty()) {
  59:     Value value = queue.front();
  60:     queue.pop();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 62-71

```cpp
  62:     // If the value properly dominates the outer loop, then it must be invariant
  63:     // to it.
  64:     if (domInfo.properlyDominates(value, refOp))
  65:       continue;
  66:     // If the value is a block argument, check if it can be used.
  67:     if (auto arg = dyn_cast<BlockArgument>(value)) {
  68:       if (!canUseArg(arg))
  69:         return false;
  70:       continue;
  71:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 73-85

```cpp
  73:     Operation *op = value.getDefiningOp();
  74:     // Check if the op was already visited.
  75:     if (visited.contains(op))
  76:       continue;
  77:     // If the defining op cannot be hoisted, then the value cannot be made loop
  78:     // invariant.
  79:     if (!canHoist(op))
  80:       return false;
  81:     visited.insert(op);
  82:     // Recurse on the operands of the op.
  83:     for (Value operand : op->getOperands())
  84:       queue.push(operand);
  85:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 87-91

```cpp
  87:   // The operations in `visited` must be hoisted. Note that operations are not
  88:   // added to `toHoist` unless all of `values` can be hoisted. This is to avoid
  89:   // hoisting operations for loops that don't end up getting fused if one of
  90:   // their bounds operands cannot be hoisted.
  91:   toHoist.insert(visited.begin(), visited.end());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 93-94

```cpp
  93:   return true;
  94: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 96-105

```cpp
  96: void triton::hoistOpsBefore(Operation *refOp,
  97:                             const llvm::SetVector<Operation *> &toHoist) {
  98:   return hoistOpsBefore(refOp->getBlock(), refOp->getIterator(), toHoist);
  99: }
 100: void triton::hoistOpsBefore(Block *block, Block::iterator it,
 101:                             const llvm::SetVector<Operation *> &toHoist) {
 102:   for (Operation *op : topologicalSort(toHoist)) {
 103:     op->moveBefore(block, it);
 104:   }
 105: }
```

- **EN:** Defines `triton::hoistOpsBefore`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `triton::hoistOpsBefore`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 107-109

```cpp
 107: //===----------------------------------------------------------------------===//
 108: // Sinking Utilities
 109: //===----------------------------------------------------------------------===//
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 111-117

```cpp
 111: Value triton::sinkValueRedefinition(RewriterBase &rewriter, Value in, Value out,
 112:                                     Block *block) {
 113:   OpBuilder::InsertionGuard guard(rewriter);
 114:   for (; block != in.getParentBlock();
 115:        block = block->getParentOp()->getBlock()) {
 116:     Operation *op = block->getParentOp();
 117:     rewriter.setInsertionPoint(op);
```

- **EN:** Defines `triton::sinkValueRedefinition`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `triton::sinkValueRedefinition`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 119-126

```cpp
 119:     // `in` is live into the loop body. `out` becomes the live-out if the
 120:     // loop executes at least once.
 121:     if (auto forOp = dyn_cast<scf::ForOp>(op)) {
 122:       forOp = addIterArgsToLoop(rewriter, forOp, in);
 123:       appendToForOpYield(forOp, out);
 124:       out = forOp.getResults().back();
 125:       continue;
 126:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 128-142

```cpp
 128:     // `in` is live into both branches. `out` becomes the live-out if the
 129:     // particular branch is taken.
 130:     if (auto ifOp = dyn_cast<scf::IfOp>(op)) {
 131:       scf::IfOp newIfOp =
 132:           replaceIfOpWithNewSignature(rewriter, ifOp, out.getType());
 133:       scf::YieldOp taken = newIfOp.thenYield();
 134:       scf::YieldOp other = newIfOp.elseYield();
 135:       if (block == newIfOp.elseBlock())
 136:         std::swap(taken, other);
 137:       taken->insertOperands(taken.getNumOperands(), out);
 138:       other->insertOperands(other.getNumOperands(), in);
 139:       out = newIfOp.getResults().back();
 140:       rewriter.eraseOp(ifOp);
 141:       continue;
 142:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. When the lowered form has no direct result, the original op is erased.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 当降级后的形式不再需要直接结果时，原始操作会被删除。
### Lines 144-147

```cpp
 144:     // TODO: Handle `scf.while`, etc.
 145:     llvm::report_fatal_error("FIXME: sinking into unhandled control flow op: " +
 146:                              op->getName().getStringRef());
 147:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 149-150

```cpp
 149:   return out;
 150: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 152-154

```cpp
 152: //===----------------------------------------------------------------------===//
 153: // Loop Pipelining Utilities
 154: //===----------------------------------------------------------------------===//
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 156-162

```cpp
 156: bool mlir::triton::loopHasDistGreaterThanOne(scf::ForOp forOp) {
 157:   return llvm::any_of(forOp.getBody()->getTerminator()->getOperands(),
 158:                       [](Value operand) {
 159:                         Operation *def = operand.getDefiningOp();
 160:                         return !def;
 161:                       });
 162: }
```

- **EN:** Defines `mlir::triton::loopHasDistGreaterThanOne`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `mlir::triton::loopHasDistGreaterThanOne`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 164-168

```cpp
 164: bool mlir::triton::isOuterLoop(scf::ForOp forOp) {
 165:   return llvm::any_of(forOp.getBody()->getOperations(), [](Operation &op) {
 166:     return isa<scf::ForOp, scf::WhileOp>(op);
 167:   });
 168: }
```

- **EN:** Defines `mlir::triton::isOuterLoop`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `mlir::triton::isOuterLoop`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 170-187

```cpp
 170: // Function to mask operations during scheduling.
 171: Operation *mlir::triton::predicateOp(RewriterBase &rewriter, Operation *op,
 172:                                      Value pred) {
 173:   OpBuilder::InsertionGuard guard(rewriter);
 174:   if (mlir::isMemoryEffectFree(op))
 175:     return op;
 176:   if (isConstantIntValue(pred, 1))
 177:     return op;
 178:   if (isa<LLVM::AssumeOp, ttng::FenceAsyncSharedOp>(op))
 179:     return op;
 180:   if (isa<ttg::AsyncCommitGroupOp, ttg::AsyncWaitOp>(op))
 181:     return op;
 182:   if (op->hasTrait<OpTrait::LocalLoadTrait>())
 183:     return op;
 184:   if (isa<ttg::LocalStoreOp>(op))
 185:     return op;
 186:   if (isa<ttng::TMEMAllocOp, ttng::TMEMLoadOp>(op))
 187:     return op;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 188-205

```cpp
 188:   if (auto ifOp = dyn_cast<scf::IfOp>(op)) {
 189:     rewriter.setInsertionPoint(op);
 190:     Value cnd = getPredMask(rewriter, ifOp.getCondition().getType(),
 191:                             ifOp.getCondition(), pred);
 192:     ifOp.getConditionMutable().assign(cnd);
 193:     return op;
 194:   }
 195:   if (auto predicatedOp = dyn_cast<tt::PredicatedOpInterface>(op)) {
 196:     rewriter.setInsertionPoint(op);
 197:     Value mask =
 198:         getPredMask(rewriter, predicatedOp.getPredicateOperandTypeLike(),
 199:                     predicatedOp.getPredicateOperand(), pred);
 200:     predicatedOp.setPredicateOperand(mask);
 201:     return op;
 202:   }
 203:   if (!op->isRegistered()) {
 204:     // Skip ops from unregistered dialects to make writing lit tests easier.
 205:     return op;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 206-206

```cpp
 206:   }
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 208-211

```cpp
 208:   op->emitOpError("pipeliner doesn't know how to predicate this op.");
 209:   llvm::report_fatal_error("Fatal pipeliner error");
 210:   return op;
 211: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 213-224

```cpp
 213: Operation *mlir::triton::wrapInMaskOp(RewriterBase &rewriter, Operation *op,
 214:                                       Value pred) {
 215:   auto mask =
 216:       ttg::MaskOp::create(rewriter, op->getLoc(), op->getResultTypes(), pred);
 217:   rewriter.createBlock(&mask->getRegion(0));
 218:   rewriter.setInsertionPointToStart(&mask->getRegion(0).front());
 219:   auto newOp = rewriter.clone(*op);
 220:   ttg::MaskReturnOp::create(rewriter, op->getLoc(), newOp->getResults());
 221:   op->replaceAllUsesWith(mask->getResults());
 222:   rewriter.eraseOp(op);
 223:   return mask;
 224: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. When the lowered form has no direct result, the original op is erased.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 当降级后的形式不再需要直接结果时，原始操作会被删除。
### Lines 226-227

```cpp
 226: void mlir::triton::resolveMaskOp(ModuleOp moduleOp) {
 227:   IRRewriter rewriter(moduleOp);
```

- **EN:** Defines `mlir::triton::resolveMaskOp`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `mlir::triton::resolveMaskOp`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 229-235

```cpp
 229:   // Canonicalize the IR to simplify the arithmetic ops defining the mask
 230:   auto arithDialect =
 231:       moduleOp.getContext()->getLoadedDialect<arith::ArithDialect>();
 232:   RewritePatternSet patterns(moduleOp.getContext());
 233:   arithDialect->getCanonicalizationPatterns(patterns);
 234:   if (mlir::applyPatternsGreedily(moduleOp, std::move(patterns)).failed())
 235:     return llvm::report_fatal_error("Failed to canonicalize the IR");
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 237-250

```cpp
 237:   SmallVector<ttg::MaskOp> maskOps;
 238:   moduleOp->walk([&](ttg::MaskOp maskOp) { maskOps.push_back(maskOp); });
 239:   for (auto maskOp : maskOps) {
 240:     rewriter.setInsertionPoint(maskOp);
 241:     while (&maskOp.getBody()->front() != maskOp.getBody()->getTerminator()) {
 242:       Operation *op = &maskOp.getBody()->front();
 243:       rewriter.moveOpBefore(op, maskOp);
 244:       op = triton::predicateOp(rewriter, op, maskOp.getPred());
 245:     }
 246:     maskOp->replaceAllUsesWith(
 247:         maskOp.getBody()->getTerminator()->getOperands());
 248:     maskOp->erase();
 249:   }
 250: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. The implementation traverses IR operations to collect facts or apply rewrites globally. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 252-256

```cpp
 252: // Return true if the given ForOp has the attribute
 253: // `tt.disallow_acc_multi_buffer` set to true.
 254: bool mlir::triton::getDisallowAccMultiBuffer(scf::ForOp forOp) {
 255:   return forOp->hasAttr(mlir::triton::kDisallowAccMultiBufferAttrName);
 256: }
```

- **EN:** Defines accessor/helper `mlir::triton::getDisallowAccMultiBuffer` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `mlir::triton::getDisallowAccMultiBuffer`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 258-275

```cpp
 258: std::pair<OpResult, int64_t>
 259: mlir::triton::getDefinitionAndDistance(scf::ForOp forOp, Value value) {
 260:   int64_t distance = 0;
 261:   DenseSet<Value> seen;
 262:   while (auto arg = dyn_cast<BlockArgument>(value)) {
 263:     // Ignore implicit captures.
 264:     if (arg.getOwner() != forOp.getBody())
 265:       return {nullptr, 0};
 266:     // Ignore induction variable.
 267:     if (arg.getArgNumber() == 0)
 268:       return {nullptr, 0};
 269:     ++distance;
 270:     value = forOp.getYieldedValues()[arg.getArgNumber() - 1];
 271:     if (!seen.insert(value).second)
 272:       return {nullptr, 0};
 273:   }
 274:   return {cast<OpResult>(value), distance};
 275: }
```

- **EN:** Defines accessor/helper `mlir::triton::getDefinitionAndDistance` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `mlir::triton::getDefinitionAndDistance`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 277-281

```cpp
 277: std::pair<Operation *, int64_t>
 278: mlir::triton::getDefiningOpAndDistance(scf::ForOp forOp, Value value) {
 279:   auto [definition, distance] = getDefinitionAndDistance(forOp, value);
 280:   return {definition ? definition.getDefiningOp() : nullptr, distance};
 281: }
```

- **EN:** Defines accessor/helper `mlir::triton::getDefiningOpAndDistance` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `mlir::triton::getDefiningOpAndDistance`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 283-293

```cpp
 283: int mlir::triton::getCopyVecBytes(RankedTensorType registerTy,
 284:                                   ttg::SharedEncodingTrait sharedEnc) {
 285:   auto shape = registerTy.getShape();
 286:   auto regLayout = triton::gpu::toLinearLayout(shape, registerTy.getEncoding());
 287:   // FIXME: Here we should pass a MemDescType instead of a SharedEncodingTrait!!
 288:   // This is currently broken for memdesc_subslice!
 289:   auto sharedLayout = triton::gpu::toLinearLayout(shape, sharedEnc);
 290:   auto regToSharedLayout = regLayout.invertAndCompose(sharedLayout);
 291:   const int vecElems = regToSharedLayout.getNumConsecutiveInOut();
 292:   return vecElems * registerTy.getElementTypeBitWidth() / 8;
 293: }
```

- **EN:** Defines accessor/helper `mlir::triton::getCopyVecBytes` that exposes or updates operation state in a compact, reusable way. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic.
- **CN:** 这里定义访问器/辅助函数 `mlir::triton::getCopyVecBytes`，以紧凑且可复用的方式读取或更新操作状态。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。
### Lines 295-300

```cpp
 295: bool mlir::triton::canBeConvertedToAsyncLoad(
 296:     tt::LoadOp loadOp, tt::ModuleAxisInfoAnalysis &axisInfoAnalysis) {
 297:   auto ptr = loadOp.getPtr();
 298:   unsigned vec = axisInfoAnalysis.getContiguity(ptr);
 299:   if (auto mask = loadOp.getMask())
 300:     vec = std::min<unsigned>(vec, axisInfoAnalysis.getMaskAlignment(mask));
```

- **EN:** Defines `mlir::triton::canBeConvertedToAsyncLoad`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `mlir::triton::canBeConvertedToAsyncLoad`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 302-311

```cpp
 302:   auto tensorTy = dyn_cast<RankedTensorType>(ptr.getType());
 303:   unsigned width = 0;
 304:   if (tensorTy) {
 305:     auto ty = cast<tt::PointerType>(tensorTy.getElementType()).getPointeeType();
 306:     width = vec * ty.getIntOrFloatBitWidth();
 307:   } else {
 308:     width = cast<tt::PointerType>(ptr.getType())
 309:                 .getPointeeType()
 310:                 .getIntOrFloatBitWidth();
 311:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 313-320

```cpp
 313:   // We do not pipeline all loads for the following reasons:
 314:   // 1. On nvidia GPUs, cp.async's cp-size can only be 4, 8, or 16.
 315:   // 2. It's likely that pipling small loads won't offer much performance
 316:   //    improvement and may even hurt performance by increasing register
 317:   //    pressure.
 318:   LDBG("Load " << *loadOp << " has width " << width);
 319:   return width >= 32;
 320: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 322-329

```cpp
 322: void mlir::triton::serializeLatencies(ModuleOp module,
 323:                                       DenseMap<Operation *, int> &opLatency) {
 324:   auto helper = TritonDialect::getLoaded(module)->getLatencyAttrHelper();
 325:   auto builder = Builder(module);
 326:   for (auto &[op, latency] : opLatency) {
 327:     helper.setAttr(op, builder.getI32IntegerAttr(latency));
 328:   }
 329: }
```

- **EN:** Defines `mlir::triton::serializeLatencies`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `mlir::triton::serializeLatencies`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 331-338

```cpp
 331: void mlir::triton::serializeSelfLatencies(
 332:     ModuleOp module, DenseMap<Operation *, int> &opSelfLatency) {
 333:   auto helper = TritonDialect::getLoaded(module)->getSelfLatencyAttrHelper();
 334:   auto builder = Builder(module);
 335:   for (auto &[op, latency] : opSelfLatency) {
 336:     helper.setAttr(op, builder.getI32IntegerAttr(latency));
 337:   }
 338: }
```

- **EN:** Defines `mlir::triton::serializeSelfLatencies`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `mlir::triton::serializeSelfLatencies`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 340-350

```cpp
 340: DenseMap<Operation *, int> mlir::triton::deserializeLatencies(Operation *op) {
 341:   DenseMap<Operation *, int> opLatency;
 342:   auto latencyHelper = TritonDialect::getLoaded(op)->getLatencyAttrHelper();
 343:   op->walk([&](Operation *op) {
 344:     if (auto attr = latencyHelper.getAttr(op)) {
 345:       opLatency[op] = attr.getInt();
 346:       latencyHelper.removeAttr(op);
 347:     }
 348:   });
 349:   return opLatency;
 350: }
```

- **EN:** Defines `mlir::triton::deserializeLatencies`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. The implementation traverses IR operations to collect facts or apply rewrites globally.
- **CN:** 这里定义 `mlir::triton::deserializeLatencies`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。
### Lines 352-366

```cpp
 352: Value mlir::triton::createScalarAlloc(ImplicitLocOpBuilder &rewriter, Type type,
 353:                                       unsigned numBuffers) {
 354:   MLIRContext *ctx = rewriter.getContext();
 355:   unsigned numCTAs = triton::gpu::TritonGPUDialect::getNumCTAs(
 356:       rewriter.getBlock()->getParentOp()->getParentOfType<ModuleOp>());
 357:   Attribute sharedMemorySpace =
 358:       ttg::SharedMemorySpaceAttr::get(rewriter.getContext());
 359:   auto barrierCGALayout = ttg::CGAEncodingAttr::get1DLayout(ctx, numCTAs);
 360:   auto barrierEncoding =
 361:       ttg::SwizzledSharedEncodingAttr::get(ctx, 1, 1, 1, {0}, barrierCGALayout);
 362:   ttg::MemDescType memDescType = ttg::MemDescType::get(
 363:       {numBuffers, numCTAs}, type, barrierEncoding, sharedMemorySpace,
 364:       /*mutableMemory=*/true);
 365:   return ttg::LocalAllocOp::create(rewriter, memDescType, Value());
 366: }
```

- **EN:** Defines helper `mlir::triton::createScalarAlloc` that computes or constructs intermediate data used by the surrounding transformation. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic.
- **CN:** 这里定义辅助函数 `mlir::triton::createScalarAlloc`，用于计算或构造外围变换所需的中间数据。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。
### Lines 368-371

```cpp
 368: // Create an allocation and init the mbarriers.
 369: Value mlir::triton::createBarrierAlloc(Operation *op, int numBarriers,
 370:                                        int arriveCount) {
 371:   ImplicitLocOpBuilder rewriter(op->getLoc(), op);
```

- **EN:** Defines helper `mlir::triton::createBarrierAlloc` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `mlir::triton::createBarrierAlloc`，用于计算或构造外围变换所需的中间数据。
### Lines 373-387

```cpp
 373:   Value barrierAlloc =
 374:       createScalarAlloc(rewriter, rewriter.getI64Type(), numBarriers);
 375:   for (unsigned i = 0; i < numBarriers; i++) {
 376:     Value barrierView = createSingleBufferView(rewriter, barrierAlloc, i);
 377:     ttng::InitBarrierOp::create(rewriter, barrierView, arriveCount);
 378:   }
 379:   // Invalidate and deallocate the barriers.
 380:   rewriter.setInsertionPointAfter(op);
 381:   for (unsigned i = 0; i < numBarriers; i++) {
 382:     Value barrierView = createSingleBufferView(rewriter, barrierAlloc, i);
 383:     ttng::InvalBarrierOp::create(rewriter, barrierView);
 384:   }
 385:   ttg::LocalDeallocOp::create(rewriter, barrierAlloc);
 386:   return barrierAlloc;
 387: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 389-401

```cpp
 389: Value mlir::triton::createAlloc(Operation *insertBefore, RankedTensorType ty,
 390:                                 Location loc,
 391:                                 gpu::SharedEncodingTrait sharedEnc,
 392:                                 unsigned distance) {
 393:   OpBuilder builder(insertBefore);
 394:   Attribute sharedMemorySpace =
 395:       ttg::SharedMemorySpaceAttr::get(insertBefore->getContext());
 396:   SmallVector<int64_t> bufferShape(ty.getShape().begin(), ty.getShape().end());
 397:   bufferShape.insert(bufferShape.begin(), distance);
 398:   Type memdescType = ttg::MemDescType::get(bufferShape, ty.getElementType(),
 399:                                            sharedEnc, sharedMemorySpace,
 400:                                            /*mutableMemory=*/true);
 401:   Value alloc = ttg::LocalAllocOp::create(builder, loc, memdescType);
```

- **EN:** Defines helper `mlir::triton::createAlloc` that computes or constructs intermediate data used by the surrounding transformation. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic.
- **CN:** 这里定义辅助函数 `mlir::triton::createAlloc`，用于计算或构造外围变换所需的中间数据。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。
### Lines 403-406

```cpp
 403:   builder.setInsertionPointAfter(insertBefore);
 404:   ttg::LocalDeallocOp::create(builder, insertBefore->getLoc(), alloc);
 405:   return alloc;
 406: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 408-422

```cpp
 408: bool mlir::triton::canBeAsyncLoad(Operation *op) {
 409:   if (mlir::triton::isTMALoad(op)) {
 410:     return true;
 411:   }
 412:   assert(isa<tt::LoadOp>(op));
 413:   ttg::SharedEncodingTrait sharedEncoding = mlir::triton::getSharedEncoding(op);
 414:   // Do not create async loads for small loads (cp.async requires at least 4
 415:   // bytes)
 416:   int copyVecBytes = mlir::triton::getCopyVecBytes(
 417:       cast<RankedTensorType>(op->getResultTypes()[0]), sharedEncoding);
 418:   if (copyVecBytes >= 4) {
 419:     return true;
 420:   }
 421:   return false;
 422: }
```

- **EN:** Defines `mlir::triton::canBeAsyncLoad`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `mlir::triton::canBeAsyncLoad`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 424-441

```cpp
 424: void mlir::triton::combineRedundantWaitOps(
 425:     llvm::SmallSetVector<Operation *, 8> &waitOps,
 426:     llvm::function_ref<bool(Operation *)> isCounterBarrier,
 427:     llvm::function_ref<Operation *(OpBuilder &, Location, ValueRange, unsigned)>
 428:         createWait) {
 429:   llvm::MapVector<Operation *, Operation *> toDelete;
 430:   for (Operation *waitOp : waitOps) {
 431:     if (toDelete.count(waitOp))
 432:       continue;
 433:     StringRef waitName = waitOp->getName().getStringRef();
 434:     auto getNum = [](Operation *op) {
 435:       return static_cast<unsigned>(
 436:           op->getAttrOfType<IntegerAttr>("num").getInt());
 437:     };
 438:     SmallVector<Operation *> waitGroup = {waitOp};
 439:     SmallVector<Value> depTokens(waitOp->getOperands().begin(),
 440:                                  waitOp->getOperands().end());
 441:     unsigned minWaitNumber = getNum(waitOp);
```

- **EN:** Defines `mlir::triton::combineRedundantWaitOps`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `mlir::triton::combineRedundantWaitOps`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 442-459

```cpp
 442:     Operation *next = waitOp->getNextNode();
 443:     // Stop at the end of the block, at any branching op (forOp, ifOp, whileOp),
 444:     // or at any caller-declared counter barrier.
 445:     while (next && !isa<RegionBranchOpInterface>(next) &&
 446:            !isCounterBarrier(next)) {
 447:       if (next->getName().getStringRef() == waitName) {
 448:         waitGroup.push_back(next);
 449:         minWaitNumber = std::min(minWaitNumber, getNum(next));
 450:         depTokens.append(next->getOperands().begin(),
 451:                          next->getOperands().end());
 452:       }
 453:       next = next->getNextNode();
 454:     }
 455:     if (waitGroup.size() == 1)
 456:       continue;
 457:     OpBuilder builder(waitGroup.front());
 458:     Operation *newWaitOp =
 459:         createWait(builder, waitOp->getLoc(), depTokens, minWaitNumber);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 460-468

```cpp
 460:     for (Operation *wo : waitGroup) {
 461:       toDelete[wo] = newWaitOp;
 462:     }
 463:   }
 464:   for (auto entry : toDelete) {
 465:     entry.first->replaceAllUsesWith(entry.second);
 466:     entry.first->erase();
 467:   }
 468: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 470-476

```cpp
 470: ttg::MemDescType mlir::triton::getBufferViewType(ttg::MemDescType allocTy,
 471:                                                  bool mutableMemory) {
 472:   return ttg::MemDescType::get(allocTy.getShape().drop_front(),
 473:                                allocTy.getElementType(), allocTy.getEncoding(),
 474:                                allocTy.getMemorySpace(), mutableMemory,
 475:                                /*allocShape=*/allocTy.getAllocShape());
 476: }
```

- **EN:** Defines accessor/helper `mlir::triton::getBufferViewType` that exposes or updates operation state in a compact, reusable way. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic.
- **CN:** 这里定义访问器/辅助函数 `mlir::triton::getBufferViewType`，以紧凑且可复用的方式读取或更新操作状态。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。
### Lines 478-487

```cpp
 478: ttg::MemDescType
 479: mlir::triton::getMultiBufferedType(ttg::MemDescType memDescType,
 480:                                    int32_t depth) {
 481:   auto shape = memDescType.getShape();
 482:   SmallVector<int64_t> bufferShape(shape.begin(), shape.end());
 483:   bufferShape.insert(bufferShape.begin(), depth);
 484:   return ttg::MemDescType::get(
 485:       bufferShape, memDescType.getElementType(), memDescType.getEncoding(),
 486:       memDescType.getMemorySpace(), /*mutableMemory*/ true);
 487: }
```

- **EN:** Defines accessor/helper `mlir::triton::getMultiBufferedType` that exposes or updates operation state in a compact, reusable way. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic.
- **CN:** 这里定义访问器/辅助函数 `mlir::triton::getMultiBufferedType`，以紧凑且可复用的方式读取或更新操作状态。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。
### Lines 489-495

```cpp
 489: ttg::SharedEncodingTrait mlir::triton::getSharedEncoding(RankedTensorType ty) {
 490:   auto cgaLayout = ttg::getCGALayout(ty.getEncoding());
 491:   auto order = ttg::getOrder(ty);
 492:   // Use generic layout. This won't be optimal for 2D tensors.
 493:   return ttg::SwizzledSharedEncodingAttr::get(ty.getContext(), 1, 1, 1, order,
 494:                                               cgaLayout);
 495: }
```

- **EN:** Defines accessor/helper `mlir::triton::getSharedEncoding` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `mlir::triton::getSharedEncoding`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 497-514

```cpp
 497: ttg::SharedEncodingTrait mlir::triton::getSharedEncoding(Operation *op) {
 498:   // Try to use local alloc encoding if possible.
 499:   ttg::SharedEncodingTrait localAllocEnc;
 500:   if (llvm::any_of(op->getUsers(), [&](Operation *user) {
 501:         return isa<ttg::LocalAllocOp>(user);
 502:       })) {
 503:     for (auto user : op->getUsers()) {
 504:       auto localAlloc = dyn_cast<ttg::LocalAllocOp>(user);
 505:       if (!localAlloc)
 506:         continue;
 507:       auto enc = mlir::cast<ttg::SharedEncodingTrait>(
 508:           localAlloc.getType().getEncoding());
 509:       if (!localAllocEnc) {
 510:         localAllocEnc = enc;
 511:       }
 512:       if (enc != localAllocEnc) {
 513:         // Some users have different encoding than others.
 514:         // Use one of the encodings, and warn about the performance issue.
```

- **EN:** Defines accessor/helper `mlir::triton::getSharedEncoding` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `mlir::triton::getSharedEncoding`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 515-521

```cpp
 515:         op->emitRemark()
 516:             << "Pipelining load with different use encodings. This will lead "
 517:                "to layout conversions and performance degradation.";
 518:         continue;
 519:       }
 520:     }
 521:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 523-529

```cpp
 523:   auto ty = cast<RankedTensorType>(op->getResultTypes()[0]);
 524:   auto cgaLayout = ttg::getCGALayout(ty.getEncoding());
 525:   auto order = ttg::getOrder(ty);
 526:   if (auto load = dyn_cast<tt::DescriptorLoadLikeOpInterface>(op)) {
 527:     // TMA encoding is set on the descriptor type
 528:     return ttng::getEncodingFromDescriptor(op, ty, load.getDesc());
 529:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 531-532

```cpp
 531:   if (localAllocEnc)
 532:     return localAllocEnc;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 534-538

```cpp
 534:   // Try to use dot encoding if possible.
 535:   bool incompatible = false;
 536:   localAllocEnc =
 537:       getSharedEncIfAllUsersAreDotEnc(op->getResult(0), incompatible)
 538:           .value_or(nullptr);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 540-541

```cpp
 540:   if (localAllocEnc)
 541:     return localAllocEnc;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 543-546

```cpp
 543:   // Use generic layout. This won't be optimal for 2D tensors.
 544:   return ttg::SwizzledSharedEncodingAttr::get(ty.getContext(), 1, 1, 1, order,
 545:                                               cgaLayout);
 546: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 548-556

```cpp
 548: int mlir::triton::getNumStagesOrDefault(scf::ForOp forOp,
 549:                                         int defaultNumStages) {
 550:   // Use the attribute attached to the loop if it exists otherwise use the
 551:   // global control.
 552:   auto helper = TritonDialect::getLoaded(forOp)->getNumStagesAttrHelper();
 553:   if (auto attr = helper.getAttr(forOp))
 554:     return attr.getInt();
 555:   return defaultNumStages;
 556: }
```

- **EN:** Defines accessor/helper `mlir::triton::getNumStagesOrDefault` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `mlir::triton::getNumStagesOrDefault`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 558-572

```cpp
 558: TypedValue<ttg::MemDescType>
 559: triton::createSingleBufferView(OpBuilder &builder, Value alloc, Value idx) {
 560:   assert(isa<ttg::MemDescType>(alloc.getType()) && "Expected MemDescType");
 561:   auto allocDescType = cast<ttg::MemDescType>(alloc.getType());
 562:   SmallVector<int64_t> shape;
 563:   assert(allocDescType.getShape().size() > 1 &&
 564:          "Expected multi-dimensional memdesc (e.g., Nx...) for subview");
 565:   shape.insert(shape.end(), allocDescType.getShape().begin() + 1,
 566:                allocDescType.getShape().end());
 567:   auto viewDescType = ttg::MemDescType::get(
 568:       shape, allocDescType.getElementType(), allocDescType.getEncoding(),
 569:       allocDescType.getMemorySpace(), allocDescType.getMutableMemory());
 570:   return ttg::MemDescIndexOp::create(builder, alloc.getLoc(), viewDescType,
 571:                                      alloc, idx);
 572: }
```

- **EN:** Defines helper `triton::createSingleBufferView` that computes or constructs intermediate data used by the surrounding transformation. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义辅助函数 `triton::createSingleBufferView`，用于计算或构造外围变换所需的中间数据。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 574-578

```cpp
 574: TypedValue<ttg::MemDescType>
 575: triton::createSingleBufferView(OpBuilder &builder, Value alloc, int idx) {
 576:   Value idxVal = arith::ConstantIntOp::create(builder, alloc.getLoc(), idx, 32);
 577:   return createSingleBufferView(builder, alloc, idxVal);
 578: }
```

- **EN:** Defines helper `triton::createSingleBufferView` that computes or constructs intermediate data used by the surrounding transformation. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic.
- **CN:** 这里定义辅助函数 `triton::createSingleBufferView`，用于计算或构造外围变换所需的中间数据。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。
### Lines 580-589

```cpp
 580: Value triton::createIncrementModulo(OpBuilder &builder, Location loc,
 581:                                     Value counter, Value modulus, Value zero,
 582:                                     Value one, Value *outWrapCond) {
 583:   Value addOne = arith::AddIOp::create(builder, loc, counter, one);
 584:   Value outOfRangeCond = arith::CmpIOp::create(
 585:       builder, loc, arith::CmpIPredicate::sge, addOne, modulus);
 586:   if (outWrapCond)
 587:     *outWrapCond = outOfRangeCond;
 588:   return arith::SelectOp::create(builder, loc, outOfRangeCond, zero, addOne);
 589: }
```

- **EN:** Defines helper `triton::createIncrementModulo` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `triton::createIncrementModulo`，用于计算或构造外围变换所需的中间数据。
### Lines 591-593

```cpp
 591: /////////////////////////////
 592: // LOWER TMA DESCRIPTORS
 593: /////////////////////////////
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 595-599

```cpp
 595: static void
 596: allocTMABuffers(scf::ForOp forOp,
 597:                 llvm::MapVector<Operation *, Value> &tmaBufferMapping,
 598:                 int maxStage) {
 599:   IRRewriter rewriter(forOp);
```

- **EN:** Defines `allocTMABuffers`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `allocTMABuffers`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 601-613

```cpp
 601:   // Create a multi-buffered allocation for each MakeTensorDescOp call in the
 602:   // loop
 603:   forOp.walk([&](tt::MakeTensorDescOp op) {
 604:     // TODO peter: walk to loop yield to find the init value if this is a
 605:     // loop-carried value. That would save us from allocating another buffer
 606:     // just for the init value
 607:     auto loc = op.getLoc();
 608:     Value alloc = triton::gpu::GlobalScratchAllocOp::create(
 609:         rewriter, loc, triton::getPointerType(rewriter.getI8Type()),
 610:         maxStage * ttng::TMA_SIZE_BYTES, ttng::TMA_ALIGN, UnitAttr());
 611:     tmaBufferMapping[op.getOperation()] = alloc;
 612:   });
 613: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. The implementation traverses IR operations to collect facts or apply rewrites globally.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。
### Lines 615-621

```cpp
 615: static Value subviewTMADescriptor(OpBuilder &builder, Location loc, Value alloc,
 616:                                   Value counter) {
 617:   Value tmaSizeVal =
 618:       arith::ConstantIntOp::create(builder, loc, ttng::TMA_SIZE_BYTES, 32);
 619:   Value offset = arith::MulIOp::create(builder, loc, tmaSizeVal, counter);
 620:   return triton::AddPtrOp::create(builder, loc, alloc.getType(), alloc, offset);
 621: }
```

- **EN:** Defines `subviewTMADescriptor`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `subviewTMADescriptor`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 623-628

```cpp
 623: static LogicalResult rewriteTMABufferUpdates(
 624:     scf::ForOp forOp,
 625:     const llvm::MapVector<Operation *, Value> &tmaBufferMapping,
 626:     ArrayRef<BlockArgument> tmaCounters, int numBuffers, Value one, Value zero,
 627:     triton::CoarseSchedule &schedule) {
 628:   assert(tmaBufferMapping.size() == tmaCounters.size());
```

- **EN:** Defines `rewriteTMABufferUpdates`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `rewriteTMABufferUpdates`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 630-632

```cpp
 630:   auto auxBuilder = mlir::OpBuilder(forOp);
 631:   Value numBuffersVal =
 632:       arith::ConstantIntOp::create(auxBuilder, forOp.getLoc(), numBuffers, 32);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 634-635

```cpp
 634:   for (auto [iOp, pair] : llvm::enumerate(tmaBufferMapping)) {
 635:     auto &[op, alloc] = pair;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 637-638

```cpp
 637:     // Rewriter MakeTensorDescOp as writing a TMA descriptor
 638:     auto makeDescOp = cast<tt::MakeTensorDescOp>(op);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 640-641

```cpp
 640:     triton::OpBuilderForStage builder(makeDescOp.getLoc(), makeDescOp,
 641:                                       schedule);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 643-651

```cpp
 643:     BlockArgument counter = tmaCounters[iOp];
 644:     Value nextBuf =
 645:         subviewTMADescriptor(builder, builder.getLoc(), alloc, counter);
 646:     if (failed(ttng::createTMADesc(nextBuf, makeDescOp, builder))) {
 647:       return failure();
 648:     }
 649:     ttng::TensormapFenceproxyAcquireOp::create(builder, nextBuf);
 650:     Value nextDesc = ttng::ReinterpretTensorDescOp::create(
 651:         builder, makeDescOp.getType(), nextBuf);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 653-653

```cpp
 653:     makeDescOp.getResult().replaceAllUsesWith(nextDesc);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 655-657

```cpp
 655:     // Increment the buffer index counter
 656:     Value nextCounter = createIncrementModulo(
 657:         builder, builder.getLoc(), counter, numBuffersVal, zero, one);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 659-663

```cpp
 659:     // If we are in a (potentially nested) if region, propagate the counter
 660:     // up to the main for op body scope
 661:     IRRewriter rewriter(forOp);
 662:     nextCounter = triton::sinkValueRedefinition(rewriter, counter, nextCounter,
 663:                                                 op->getBlock());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 665-671

```cpp
 665:     // Finally, rewrite the loop level yield
 666:     auto forYield = cast<scf::YieldOp>(forOp.getBody()->getTerminator());
 667:     forYield.setOperand(counter.getArgNumber() - 1, nextCounter);
 668:     makeDescOp.erase();
 669:   }
 670:   return success();
 671: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 673-688

```cpp
 673: scf::ForOp triton::lowerTMADescriptors(scf::ForOp forOp,
 674:                                        CoarseSchedule &schedule) {
 675:   llvm::MapVector<Operation *, Value> tmaBufferMapping;
 676:   int maxStage = schedule.getNumStages() - 1;
 677:   for (auto &op : forOp.getBody()->without_terminator()) {
 678:     if (auto wgMmaOp = dyn_cast<ttng::WarpGroupDotOp>(&op)) {
 679:       // Hopper only: Add one more buffer slice if there is a WarpGroupDotOp,
 680:       // as if it will be pipelined, we will effectively make the pipeline
 681:       // one stage longer.
 682:       maxStage += 1;
 683:       break;
 684:     }
 685:   }
 686:   allocTMABuffers(forOp, tmaBufferMapping, maxStage);
 687:   if (tmaBufferMapping.empty())
 688:     return forOp;
```

- **EN:** Defines helper `triton::lowerTMADescriptors` that performs a lowering step from Triton/MLIR semantics to a lower-level representation.
- **CN:** 这里定义辅助函数 `triton::lowerTMADescriptors`，负责把 Triton/MLIR 语义中的一个步骤降级到更底层表示。
### Lines 690-702

```cpp
 690:   IRRewriter builder(forOp);
 691:   Location loc = forOp.getLoc();
 692:   Value zero = arith::ConstantIntOp::create(builder, loc, 0, 32);
 693:   Value one = arith::ConstantIntOp::create(builder, loc, 1, 32);
 694:   SmallVector<Value> newOperands;
 695:   unsigned newOperandIndex = forOp.getBody()->getNumArguments();
 696:   // Create one counter per TMA buffer. This allows the descriptors to be
 697:   // updated independently without needing to write duplicate of existing tma
 698:   // descriptors.
 699:   unsigned tmaCounterArgsStartIdx = newOperandIndex + newOperands.size();
 700:   for (int i = 0; i < tmaBufferMapping.size(); ++i) {
 701:     newOperands.push_back(zero);
 702:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 704-704

```cpp
 704:   forOp = addIterArgsToLoop(builder, forOp, newOperands);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 706-707

```cpp
 706:   auto tmaCounters = ArrayRef<BlockArgument>(forOp.getBody()->getArguments())
 707:                          .slice(tmaCounterArgsStartIdx);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 709-713

```cpp
 709:   // Update yield op with temporary yield values
 710:   auto forYield = cast<scf::YieldOp>(forOp.getBody()->getTerminator());
 711:   for (unsigned i = 0; i < newOperands.size(); ++i) {
 712:     forYield.getResultsMutable().append(newOperands[i]);
 713:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 715-720

```cpp
 715:   if (failed(rewriteTMABufferUpdates(forOp, tmaBufferMapping, tmaCounters,
 716:                                      maxStage, one, zero, schedule))) {
 717:     llvm_unreachable("Failed to rewrite TMA ops");
 718:   }
 719:   return forOp;
 720: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 722-739

```cpp
 722: DenseSet<Operation *>
 723: triton::getTopLevelUsersInLoop(Operation *op, scf::ForOp forOp,
 724:                                std::function<bool(Operation *)> filter) {
 725:   DenseSet<Operation *> topLevelUsers;
 726:   SmallVector<OpOperand *> q;
 727:   for (auto &use : op->getUses())
 728:     q.push_back(&use);
 729:   while (!q.empty()) {
 730:     auto use = q.pop_back_val();
 731:     auto yieldOp = dyn_cast<scf::YieldOp>(use->getOwner());
 732:     if (yieldOp && yieldOp->getParentOp() == forOp) {
 733:       for (auto &use :
 734:            forOp.getRegionIterArgs()[use->getOperandNumber()].getUses())
 735:         q.push_back(&use);
 736:       continue;
 737:     }
 738:     // Don't count view operations as uses. Follow them through to their
 739:     // users.
```

- **EN:** Defines accessor/helper `triton::getTopLevelUsersInLoop` that exposes or updates operation state in a compact, reusable way. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义访问器/辅助函数 `triton::getTopLevelUsersInLoop`，以紧凑且可复用的方式读取或更新操作状态。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 740-752

```cpp
 740:     if (use->getOwner()->hasTrait<OpTrait::MemDescViewTrait>()) {
 741:       for (auto &use : use->getOwner()->getUses())
 742:         q.push_back(&use);
 743:       continue;
 744:     }
 745:     if (filter && !filter(use->getOwner()))
 746:       continue;
 747:     Operation *topLevelUser =
 748:         forOp.getBody()->findAncestorOpInBlock(*use->getOwner());
 749:     topLevelUsers.insert(topLevelUser);
 750:   }
 751:   return topLevelUsers;
 752: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 754-771

```cpp
 754: // Helper function that finds an operation based on a comparison predicate
 755: static Operation *getUseOfPipelinedOp(
 756:     ArrayRef<Operation *> ops, scf::ForOp forOp,
 757:     triton::CoarseSchedule &schedule,
 758:     std::function<bool(Operation *)> filterUse,
 759:     std::function<bool(Operation *, Operation *)> shouldPrefer) {
 760:   DenseSet<Operation *> topLevelUsers;
 761:   Operation *selectedUser = nullptr;
 762:   for (Operation *op : ops) {
 763:     auto users = triton::getTopLevelUsersInLoop(op, forOp, filterUse);
 764:     topLevelUsers.insert(users.begin(), users.end());
 765:   }
 766:   for (Operation *topLevelUser : topLevelUsers) {
 767:     assert(schedule.count(topLevelUser) && "op user not found in the schedule");
 768:     if (!selectedUser || shouldPrefer(topLevelUser, selectedUser)) {
 769:       selectedUser = topLevelUser;
 770:     }
 771:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 772-773

```cpp
 772:   return selectedUser;
 773: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 775-783

```cpp
 775: Operation *
 776: triton::getFirstUseOfPipelinedOp(ArrayRef<Operation *> ops, scf::ForOp forOp,
 777:                                  triton::CoarseSchedule &schedule,
 778:                                  std::function<bool(Operation *)> filterUse) {
 779:   return getUseOfPipelinedOp(
 780:       ops, forOp, schedule, filterUse,
 781:       [&](Operation *candidate, Operation *current) {
 782:         auto [candidateStage, candidateCluster] = schedule[candidate];
 783:         auto [currentStage, currentCluster] = schedule[current];
```

- **EN:** Defines accessor/helper `triton::getFirstUseOfPipelinedOp` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `triton::getFirstUseOfPipelinedOp`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 785-792

```cpp
 785:         return candidateStage < currentStage ||
 786:                (candidateStage == currentStage &&
 787:                 schedule.clusters.isBefore(candidateCluster, currentCluster)) ||
 788:                (candidateStage == currentStage &&
 789:                 candidateCluster == currentCluster &&
 790:                 candidate->isBeforeInBlock(current));
 791:       });
 792: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 794-802

```cpp
 794: Operation *
 795: triton::getLastUseOfPipelinedOp(ArrayRef<Operation *> ops, scf::ForOp forOp,
 796:                                 triton::CoarseSchedule &schedule,
 797:                                 std::function<bool(Operation *)> filterUse) {
 798:   return getUseOfPipelinedOp(
 799:       ops, forOp, schedule, filterUse,
 800:       [&](Operation *candidate, Operation *current) {
 801:         auto [candidateStage, candidateCluster] = schedule[candidate];
 802:         auto [currentStage, currentCluster] = schedule[current];
```

- **EN:** Defines accessor/helper `triton::getLastUseOfPipelinedOp` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `triton::getLastUseOfPipelinedOp`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 804-811

```cpp
 804:         return candidateStage > currentStage ||
 805:                (candidateStage == currentStage &&
 806:                 schedule.clusters.isBefore(currentCluster, candidateCluster)) ||
 807:                (candidateStage == currentStage &&
 808:                 candidateCluster == currentCluster &&
 809:                 current->isBeforeInBlock(candidate));
 810:       });
 811: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 813-819

```cpp
 813: void triton::removePipeliningAttributes(ModuleOp moduleOp) {
 814:   moduleOp->walk([&](Operation *op) {
 815:     op->removeAttr(mlir::triton::kLoopStageAttrName);
 816:     op->removeAttr(mlir::triton::kLoopClusterAttrName);
 817:     op->removeAttr(mlir::triton::kScheduledMaxStageAttrName);
 818:   });
 819: }
```

- **EN:** Defines `triton::removePipeliningAttributes`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. The implementation traverses IR operations to collect facts or apply rewrites globally.
- **CN:** 这里定义 `triton::removePipeliningAttributes`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。

## Key Concepts / 关键概念
- **EN:** The main concern is pass-driven transformation around pipelining utility.
  **CN:** 核心关注点是围绕 Pipelining Utility 的 pass 驱动变换。
- **EN:** Memory descriptor types carry layout, rank, and address-space information.
  **CN:** 内存描述符类型携带布局、维度和地址空间信息。
- **EN:** Tensor shape and element-type reasoning is central to the implementation.
  **CN:** 张量形状与元素类型推导是实现中的核心内容。
- **EN:** Linear layout utilities translate between logical tensor coordinates and physical placement.
  **CN:** 线性布局工具负责在逻辑张量坐标与物理放置之间进行转换。
- **EN:** Encoding attributes describe how distributed GPU data is laid out across threads and warps.
  **CN:** 编码属性描述 GPU 分布式数据在线程与 warp 之间的布局方式。
- **EN:** The code reasons at module scope rather than only on isolated operations.
  **CN:** 代码在模块范围内进行分析，而不只是处理单个操作。

## Dependencies / 依赖关系
- **Internal Triton headers / Triton 内部头文件:** `triton/Dialect/TritonGPU/Transforms/PipeliningUtility.h`, `triton/Analysis/AxisInfo.h`, `triton/Dialect/Triton/IR/Utility.h`, `triton/Dialect/TritonGPU/IR/Dialect.h`, `triton/Dialect/TritonGPU/Transforms/Passes.h`, `triton/Dialect/TritonGPU/Transforms/Schedule.h`, ... (+4 more)
- **MLIR headers / MLIR 头文件:** `mlir/Analysis/TopologicalSortUtils.h`, `mlir/Dialect/LLVMIR/LLVMDialect.h`, `mlir/Dialect/Tensor/IR/Tensor.h`, `mlir/IR/ImplicitLocOpBuilder.h`, `mlir/IR/TypeUtilities.h`, `mlir/Interfaces/SideEffectInterfaces.h`, ... (+2 more)
- **LLVM headers / LLVM 头文件:** `llvm/Support/Casting.h`, `llvm/Support/Debug.h`
- **Standard/library headers / 标准或通用库头文件:** `queue`
- **Primary APIs used / 主要 API:** `ModuleOp`, `RankedTensorType`, `MemDescType`, `LinearLayout`, `OpBuilder`
- **Pipeline role / 流程角色:** This file participates in Triton pass pipelines and usually expects upstream analyses or dialect invariants to have prepared the IR. / 本文件参与 Triton 的 pass 流程，通常依赖上游分析或方言不变量先把 IR 准备好。
