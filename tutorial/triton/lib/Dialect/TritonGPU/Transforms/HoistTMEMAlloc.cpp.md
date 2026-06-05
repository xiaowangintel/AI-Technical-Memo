# HoistTMEMAlloc.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `lib/Dialect/TritonGPU/Transforms/HoistTMEMAlloc.cpp`
- **Purpose / 作用:** **EN:** Implements the Hoist TMEM Alloc transformation or optimization pass for the TritonGPU pipeline. **CN:** 为 TritonGPU 编译流程实现与 Hoist TMEM Alloc 相关的变换或优化 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11

```cpp
   1: #include "mlir/Dialect/Arith/IR/Arith.h"
   2: #include "mlir/Dialect/SCF/IR/SCF.h"
   3: #include "mlir/Dialect/UB/IR/UBOps.h"
   4: #include "mlir/IR/Dominance.h"
   5: #include "mlir/Transforms/GreedyPatternRewriteDriver.h"
   6: #include "triton/Dialect/TritonGPU/IR/Dialect.h"
   7: #include "triton/Dialect/TritonGPU/Transforms/MMAv5PipelineUtility.h"
   8: #include "triton/Dialect/TritonGPU/Transforms/Passes.h"
   9: #include "triton/Dialect/TritonGPU/Transforms/PipeliningUtility.h"
  10: #include "triton/Dialect/TritonGPU/Transforms/Utility.h"
  11: #include "triton/Dialect/TritonNvidiaGPU/IR/Dialect.h"
```

- **EN:** Includes the interfaces this file depends on. Triton headers (`Dialect.h`, `MMAv5PipelineUtility.h`, `Passes.h`, `PipeliningUtility.h`, ... (+2 more)) provide domain-specific IR/support, MLIR headers (`Arith.h`, `SCF.h`, `UBOps.h`, `Dominance.h`, ... (+1 more)) provide rewriting and analysis infrastructure, LLVM headers (None) supply low-level utilities, and standard/library headers (None) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（`Dialect.h`, `MMAv5PipelineUtility.h`, `Passes.h`, `PipeliningUtility.h`, ... (+2 more)）提供领域特定 IR/支持逻辑，MLIR 头文件（`Arith.h`, `SCF.h`, `UBOps.h`, `Dominance.h`, ... (+1 more)）提供重写与分析基础设施，LLVM 头文件（None）提供底层工具，而标准/通用库头文件（None）提供通用能力。
### Lines 13-16

```cpp
  13: using namespace mlir;
  14: namespace tt = mlir::triton;
  15: namespace ttg = mlir::triton::gpu;
  16: namespace ttng = mlir::triton::nvidia_gpu;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 18-20

```cpp
  18: namespace mlir {
  19: namespace triton {
  20: namespace gpu {
```

- **EN:** Opens or closes the namespace nesting for mlir -> triton -> gpu, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 mlir -> triton -> gpu 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 22-23

```cpp
  22: #define GEN_PASS_DEF_TRITONGPUHOISTTMEMALLOC
  23: #include "triton/Dialect/TritonGPU/Transforms/Passes.h.inc"
```

- **EN:** Pulls in generated definitions from TableGen/MLIR include fragments so the handwritten code can reuse auto-generated declarations.
- **CN:** 这里引入由 TableGen/MLIR 生成的定义片段，使手写代码能够复用自动生成的声明。
### Lines 25-25

```cpp
  25: namespace {
```

- **EN:** Opens or closes the namespace nesting for (anonymous), keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 (anonymous) 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 27-31

```cpp
  27: // This CRTP class is an operation type constraint that checks that it has TMEM
  28: // dependency tokens present. HoistTMEMAlloc requires that TMEM tokens are
  29: // present to check aliasing for its transformations.
  30: template <typename OpT> struct HasToken : public OpT {
  31:   using OpT::OpT;
```

- **EN:** Defines `HasToken`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `HasToken`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 33-38

```cpp
  33:   static bool classof(Operation *op) {
  34:     if (auto tmemOp = dyn_cast<OpT>(op))
  35:       return !!tmemOp.getToken();
  36:     return false;
  37:   }
  38: };
```

- **EN:** Defines `classof`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `classof`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 40-42

```cpp
  40: using TMEMTokenLoadOp = HasToken<ttng::TMEMLoadOp>;
  41: using TMEMTokenStoreOp = HasToken<ttng::TMEMStoreOp>;
  42: using TMEMTokenAllocOp = HasToken<ttng::TMEMAllocOp>;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 44-46

```cpp
  44: class CombineTMEMStoreAndSelect : public OpRewritePattern<ttng::TMEMStoreOp> {
  45: public:
  46:   using OpRewritePattern::OpRewritePattern;
```

- **EN:** Defines `CombineTMEMStoreAndSelect`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `CombineTMEMStoreAndSelect`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 48-63

```cpp
  48:   LogicalResult matchAndRewrite(ttng::TMEMStoreOp store,
  49:                                 PatternRewriter &rewriter) const override {
  50:     if (!store.getDep())
  51:       return failure();
  52:     Value src = store.getSrc();
  53:     auto select = src.getDefiningOp<arith::SelectOp>();
  54:     if (!select) {
  55:       return failure();
  56:     }
  57:     enum { kTrue, kFalse, kUnknown } valueFromTMEM = kUnknown;
  58:     Value trueSrc = select.getTrueValue();
  59:     Value falseSrc = select.getFalseValue();
  60:     if (auto load = trueSrc.getDefiningOp<TMEMTokenLoadOp>()) {
  61:       if (store.getDst() == load.getSrc() && load.getToken() == store.getDep())
  62:         valueFromTMEM = kTrue;
  63:     }
```

- **EN:** Defines `matchAndRewrite`, the core MLIR rewrite callback. It checks whether the source operation matches the pattern and then materializes the lowered replacement.
- **CN:** 这里定义 `matchAndRewrite`，即 MLIR 模式重写的核心回调。它先检查源操作是否匹配模式，再构造降级后的替代结果。
### Lines 64-79

```cpp
  64:     if (auto load = falseSrc.getDefiningOp<TMEMTokenLoadOp>()) {
  65:       if (store.getDst() == load.getSrc() && load.getToken() == store.getDep())
  66:         valueFromTMEM = valueFromTMEM == kTrue ? kUnknown : kFalse;
  67:     }
  68:     if (valueFromTMEM == kUnknown) {
  69:       return failure();
  70:     }
  71:     Value pred = select.getCondition();
  72:     // In case the false operand is overwriting, we need to negate the predicate
  73:     // (owerwrite when select would be false)
  74:     if (valueFromTMEM == kTrue) {
  75:       Value one = arith::ConstantIntOp::create(rewriter, select.getLoc(), 1, 1);
  76:       pred = arith::XOrIOp::create(rewriter, select.getLoc(), pred, one);
  77:     }
  78:     // Store the selected value with the updated predicate
  79:     Value overwritingValue = valueFromTMEM == kTrue ? falseSrc : trueSrc;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 80-85

```cpp
  80:     rewriter.replaceOpWithNewOp<ttng::TMEMStoreOp>(
  81:         store, rewriter.getType<AsyncTokenType>(), store.getDst(),
  82:         store.getDep(), overwritingValue, pred);
  83:     return success();
  84:   }
  85: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. It finishes by replacing the original operation with the lowered form.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 它会在转换成功后用降级后的形式替换原始操作。
### Lines 87-89

```cpp
  87: class RemoveUnusedTMEMLoad : public OpRewritePattern<ttng::TMEMLoadOp> {
  88: public:
  89:   using OpRewritePattern::OpRewritePattern;
```

- **EN:** Defines `RemoveUnusedTMEMLoad`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `RemoveUnusedTMEMLoad`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 91-100

```cpp
  91:   LogicalResult matchAndRewrite(ttng::TMEMLoadOp load,
  92:                                 PatternRewriter &rewriter) const override {
  93:     if (!load.getDep())
  94:       return failure();
  95:     if (!load.getResult().use_empty())
  96:       return failure();
  97:     rewriter.replaceAllUsesWith(load.getToken(), load.getDep());
  98:     return success();
  99:   }
 100: };
```

- **EN:** Defines `matchAndRewrite`, the core MLIR rewrite callback. It checks whether the source operation matches the pattern and then materializes the lowered replacement. The code uses `LogicalResult` to stay conservative when preconditions are not met.
- **CN:** 这里定义 `matchAndRewrite`，即 MLIR 模式重写的核心回调。它先检查源操作是否匹配模式，再构造降级后的替代结果。 代码通过 `LogicalResult` 在前提不满足时保持保守处理。
### Lines 102-105

```cpp
 102: // Load-store forwarding pattern.
 103: class CombineTMEMLoadAndStore : public OpRewritePattern<ttng::TMEMStoreOp> {
 104: public:
 105:   using OpRewritePattern::OpRewritePattern;
```

- **EN:** Defines `CombineTMEMLoadAndStore`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `CombineTMEMLoadAndStore`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 107-118

```cpp
 107:   LogicalResult matchAndRewrite(ttng::TMEMStoreOp store,
 108:                                 PatternRewriter &rewriter) const override {
 109:     if (!store.getDep())
 110:       return failure();
 111:     auto load = store.getDep().getDefiningOp<HasToken<ttng::TMEMLoadOp>>();
 112:     if (!load || load.getResult() != store.getSrc() ||
 113:         load.getSrc() != store.getDst())
 114:       return failure();
 115:     rewriter.replaceOp(store, load.getToken());
 116:     return success();
 117:   }
 118: };
```

- **EN:** Defines `matchAndRewrite`, the core MLIR rewrite callback. It checks whether the source operation matches the pattern and then materializes the lowered replacement. It finishes by replacing the original operation with the lowered form. The code uses `LogicalResult` to stay conservative when preconditions are not met.
- **CN:** 这里定义 `matchAndRewrite`，即 MLIR 模式重写的核心回调。它先检查源操作是否匹配模式，再构造降级后的替代结果。 它会在转换成功后用降级后的形式替换原始操作。 代码通过 `LogicalResult` 在前提不满足时保持保守处理。
### Lines 120-122

```cpp
 120: class SinkTMEMLoad : public OpRewritePattern<ttng::TMEMLoadOp> {
 121: public:
 122:   using OpRewritePattern::OpRewritePattern;
```

- **EN:** Defines `SinkTMEMLoad`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `SinkTMEMLoad`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 124-140

```cpp
 124:   LogicalResult matchAndRewrite(ttng::TMEMLoadOp load,
 125:                                 PatternRewriter &rewriter) const override {
 126:     if (!load.getDep())
 127:       return failure();
 128:     auto forOp = load->getParentOfType<scf::ForOp>();
 129:     if (!forOp) {
 130:       return failure();
 131:     }
 132:     DominanceInfo domInfo(forOp);
 133:     Operation *domOp = findNearestCommonDominator(
 134:         llvm::to_vector(load.getResult().getUsers()), domInfo);
 135:     if (!domOp || !domInfo.properlyDominates(load.getOperation(), domOp)) {
 136:       return failure();
 137:     }
 138:     // Don't sink past potentially aliasing ops.
 139:     PostDominanceInfo postDomInfo(forOp);
 140:     SmallVector<OpOperand *> uses;
```

- **EN:** Defines `matchAndRewrite`, the core MLIR rewrite callback. It checks whether the source operation matches the pattern and then materializes the lowered replacement.
- **CN:** 这里定义 `matchAndRewrite`，即 MLIR 模式重写的核心回调。它先检查源操作是否匹配模式，再构造降级后的替代结果。
### Lines 141-157

```cpp
 141:     for (OpOperand &use : load.getToken().getUses())
 142:       uses.push_back(&use);
 143:     if (!llvm::all_of(uses, [&](OpOperand *use) {
 144:           return postDomInfo.properlyPostDominates(use->getOwner(), domOp);
 145:         }))
 146:       return failure();
 147:     // In order to not re-ordering multiple tmem load in a loop, don't sink if
 148:     // all the ops between the load and the domOp are tmem loads.
 149:     Operation *nextNode = load->getNextNode();
 150:     while (auto tmemLoad = dyn_cast<ttng::TMEMLoadOp>(nextNode)) {
 151:       nextNode = tmemLoad->getNextNode();
 152:     }
 153:     if (domOp == nextNode) {
 154:       // The load wasn't moved.
 155:       return failure();
 156:     }
 157:     rewriter.moveOpBefore(load, domOp);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 158-166

```cpp
 158:     Value newToken = sinkValueRedefinition(rewriter, load.getDep(),
 159:                                            load.getToken(), domOp->getBlock());
 160:     if (newToken != load.getToken()) {
 161:       for (OpOperand *use : uses)
 162:         use->set(newToken);
 163:     }
 164:     return success();
 165:   }
 166: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 168-172

```cpp
 168: // Combine back TMEM alloc and store. This is equivalent but gives us a more
 169: // canonical form to do further optimizations.
 170: class CombineTMEMStoreAndAlloc : public OpRewritePattern<ttng::TMEMStoreOp> {
 171: public:
 172:   using OpRewritePattern::OpRewritePattern;
```

- **EN:** Defines `CombineTMEMStoreAndAlloc`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `CombineTMEMStoreAndAlloc`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 174-191

```cpp
 174:   LogicalResult matchAndRewrite(ttng::TMEMStoreOp store,
 175:                                 PatternRewriter &rewriter) const override {
 176:     if (!store.getDep())
 177:       return failure();
 178:     if (!matchPattern(store.getPred(), m_One()))
 179:       return failure();
 180:     auto alloc = store.getDep().getDefiningOp<TMEMTokenAllocOp>();
 181:     if (!alloc)
 182:       return failure();
 183:     if (store.getDst() != alloc.getResult())
 184:       return failure();
 185:     if (alloc->getBlock() != store->getBlock())
 186:       return failure();
 187:     if (auto srcDef = store.getSrc().getDefiningOp()) {
 188:       if (alloc->getBlock() == srcDef->getBlock() &&
 189:           alloc->isBeforeInBlock(srcDef))
 190:         return failure();
 191:     }
```

- **EN:** Defines `matchAndRewrite`, the core MLIR rewrite callback. It checks whether the source operation matches the pattern and then materializes the lowered replacement.
- **CN:** 这里定义 `matchAndRewrite`，即 MLIR 模式重写的核心回调。它先检查源操作是否匹配模式，再构造降级后的替代结果。
### Lines 192-196

```cpp
 192:     alloc.getSrcMutable().assign(store.getSrc());
 193:     rewriter.replaceOp(store, alloc.getToken());
 194:     return success();
 195:   }
 196: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. It finishes by replacing the original operation with the lowered form.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 它会在转换成功后用降级后的形式替换原始操作。
### Lines 198-215

```cpp
 198: // Hoists a tmem alloc outside an if op like this:
 199: // %0 = scf.if {
 200: //   %1, %token0 = tmem.alloc %init
 201: //   ...
 202: //   %2 = tmem.load %1, %token1
 203: //   scf.yield %2
 204: // } else {
 205: //   scf.yield %init
 206: // }
 207: // ->
 208: // %a, %token0 = tmem.alloc %init
 209: // %token2 = scf.if {
 210: //
 211: //   ...
 212: //   scf.yield %token1
 213: // } else {
 214: //   scf.yield %token0
 215: // }
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 216-219

```cpp
 216: // %2 = tmem.load %a, %token2
 217: class HoistTMEMAllocOutOfIf : public OpRewritePattern<ttng::TMEMAllocOp> {
 218: public:
 219:   using OpRewritePattern::OpRewritePattern;
```

- **EN:** Defines `HoistTMEMAllocOutOfIf`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `HoistTMEMAllocOutOfIf`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 221-238

```cpp
 221:   LogicalResult matchAndRewrite(ttng::TMEMAllocOp alloc,
 222:                                 PatternRewriter &rewriter) const override {
 223:     if (!alloc.getToken())
 224:       return failure();
 225:     Value init = alloc.getSrc();
 226:     if (!init)
 227:       return failure();
 228:     auto ifOp = dyn_cast<scf::IfOp>(alloc->getParentOp());
 229:     if (!ifOp || !ifOp.elseBlock())
 230:       return failure();
 231:     auto thenOp = ifOp.thenBlock()->getTerminator();
 232:     auto elseOp = ifOp.elseBlock()->getTerminator();
 233:     SmallVector<int> yieldArgs;
 234:     for (auto [thenOperand, elseOperand] :
 235:          llvm::zip(thenOp->getOpOperands(), elseOp->getOpOperands())) {
 236:       auto load = thenOperand.get().getDefiningOp<TMEMTokenLoadOp>();
 237:       if (!load || load.getSrc() != alloc.getResult())
 238:         continue;
```

- **EN:** Defines `matchAndRewrite`, the core MLIR rewrite callback. It checks whether the source operation matches the pattern and then materializes the lowered replacement. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `matchAndRewrite`，即 MLIR 模式重写的核心回调。它先检查源操作是否匹配模式，再构造降级后的替代结果。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 239-256

```cpp
 239:       if (elseOperand.get() != init)
 240:         continue;
 241:       yieldArgs.push_back(thenOperand.getOperandNumber());
 242:     }
 243:     if (yieldArgs.empty())
 244:       return failure();
 245:     // Since init is used in the else terminator we know that it dominates the
 246:     // if op.
 247:     alloc->moveBefore(ifOp);
 248:     rewriter.setInsertionPointAfter(ifOp);
 249:     for (int argNo : yieldArgs) {
 250:       auto load =
 251:           cast<TMEMTokenLoadOp>(thenOp->getOperand(argNo).getDefiningOp());
 252:       auto newLoad = cast<TMEMTokenLoadOp>(rewriter.clone(*load));
 253:       rewriter.modifyOpInPlace(ifOp, [&] {
 254:         ifOp->getResult(argNo).replaceAllUsesWith(newLoad.getResult());
 255:         newLoad.getDepMutable().assign(ifOp->getResult(argNo));
 256:         thenOp->setOperand(argNo, load.getToken());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 257-263

```cpp
 257:         elseOp->setOperand(argNo, alloc.getToken());
 258:         ifOp->getResult(argNo).setType(newLoad.getToken().getType());
 259:       });
 260:     }
 261:     return success();
 262:   }
 263: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 265-268

```cpp
 265: // Forward a TMEM load into the user allocation.
 266: class TMEMLoadForwarding : public OpRewritePattern<ttng::TMEMAllocOp> {
 267: public:
 268:   using OpRewritePattern::OpRewritePattern;
```

- **EN:** Defines `TMEMLoadForwarding`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `TMEMLoadForwarding`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 270-285

```cpp
 270:   LogicalResult matchAndRewrite(ttng::TMEMAllocOp alloc,
 271:                                 PatternRewriter &rewriter) const override {
 272:     if (!alloc.getToken())
 273:       return failure();
 274:     Value init = alloc.getSrc();
 275:     if (!init)
 276:       return failure();
 277:     auto load = init.getDefiningOp<TMEMTokenLoadOp>();
 278:     if (!load || !load->hasOneUse() || !load.getDep().hasOneUse())
 279:       return failure();
 280:     if (alloc.getType() != load.getSrc().getType())
 281:       return failure();
 282:     rewriter.replaceOp(alloc, {load.getSrc(), load.getDep()});
 283:     return success();
 284:   }
 285: };
```

- **EN:** Defines `matchAndRewrite`, the core MLIR rewrite callback. It checks whether the source operation matches the pattern and then materializes the lowered replacement. It finishes by replacing the original operation with the lowered form. The code uses `LogicalResult` to stay conservative when preconditions are not met.
- **CN:** 这里定义 `matchAndRewrite`，即 MLIR 模式重写的核心回调。它先检查源操作是否匹配模式，再构造降级后的替代结果。 它会在转换成功后用降级后的形式替换原始操作。 代码通过 `LogicalResult` 在前提不满足时保持保守处理。
### Lines 287-291

```cpp
 287: // Remove loop-carried tensor dependencies if they are fed immediately into a
 288: // TMEM store by pulling the store into the previous iteration.
 289: class RotateTMEMStoreInLoop : public OpRewritePattern<ttng::TMEMStoreOp> {
 290: public:
 291:   using OpRewritePattern::OpRewritePattern;
```

- **EN:** Defines `RotateTMEMStoreInLoop`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `RotateTMEMStoreInLoop`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 293-310

```cpp
 293:   LogicalResult matchAndRewrite(ttng::TMEMStoreOp store,
 294:                                 PatternRewriter &rewriter) const override {
 295:     if (!store.getDep())
 296:       return failure();
 297:     // Pattern match stores whose source comes from a loop region argument and
 298:     // whose predicate is loop-invariant.
 299:     scf::ForOp forOp = dyn_cast<scf::ForOp>(store->getParentOp());
 300:     if (!forOp || !forOp.isDefinedOutsideOfLoop(store.getPred()) ||
 301:         !forOp.isDefinedOutsideOfLoop(store.getDst())) {
 302:       return failure();
 303:     }
 304:     auto getAsLoopArg = [&](Value v) -> BlockArgument {
 305:       auto arg = dyn_cast<BlockArgument>(v);
 306:       if (arg && arg.getOwner() == forOp.getBody())
 307:         return arg;
 308:       return {};
 309:     };
 310:     BlockArgument src = getAsLoopArg(store.getSrc());
```

- **EN:** Defines `matchAndRewrite`, the core MLIR rewrite callback. It checks whether the source operation matches the pattern and then materializes the lowered replacement.
- **CN:** 这里定义 `matchAndRewrite`，即 MLIR 模式重写的核心回调。它先检查源操作是否匹配模式，再构造降级后的替代结果。
### Lines 311-313

```cpp
 311:     if (!src || !src.hasOneUse()) {
 312:       return failure();
 313:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 315-320

```cpp
 315:     // Check that rotating the store into the past won't violate any
 316:     // write-after-read dependencies.
 317:     BlockArgument storeTok = getAsLoopArg(store.getDep());
 318:     if (!storeTok)
 319:       return failure();
 320:     int tokArgNo = storeTok.getArgNumber() - 1;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 322-332

```cpp
 322:     // Create two copies of the store: one before the loop, storing the initial
 323:     // value, and one before the yield, storing the value carried by the loop
 324:     // arg.
 325:     int argNo = src.getArgNumber() - 1;
 326:     Value initVal = forOp.getInitArgs()[argNo];
 327:     rewriter.setInsertionPoint(forOp);
 328:     auto tokType = rewriter.getType<AsyncTokenType>();
 329:     auto initStore = ttng::TMEMStoreOp::create(
 330:         rewriter, store.getLoc(), tokType, store.getDst(),
 331:         forOp.getInitArgs()[tokArgNo], initVal, store.getPred());
 332:     forOp.getInitArgsMutable()[tokArgNo].assign(initStore.getToken());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 334-339

```cpp
 334:     auto yield = cast<scf::YieldOp>(forOp.getBody()->getTerminator());
 335:     store.getToken().replaceAllUsesWith(forOp.getRegionIterArg(tokArgNo));
 336:     rewriter.moveOpBefore(store, yield);
 337:     store.getDepMutable().assign(yield.getOperand(tokArgNo));
 338:     yield.setOperand(tokArgNo, store.getToken());
 339:     store.getSrcMutable().assign(yield.getOperand(argNo));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 341-352

```cpp
 341:     // Load from the tmem after the loop, and use it instead of the loop carried
 342:     // value.
 343:     rewriter.setInsertionPointAfter(forOp);
 344:     auto load = ttng::TMEMLoadOp::create(
 345:         rewriter, store.getLoc(), store.getSrc().getType(), tokType,
 346:         store.getDst(), forOp.getResult(tokArgNo));
 347:     forOp->getResult(argNo).replaceAllUsesWith(load.getResult());
 348:     // Loop carried value is no longer used, short-circuit it.
 349:     yield.setOperand(argNo, forOp.getRegionIterArg(argNo));
 350:     return success();
 351:   }
 352: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 354-358

```cpp
 354: // Remove loop-carried tensor dependencies if they are the result of TMEM loads
 355: // at the end of the loop by pushing the load into the next iteration.
 356: class RotateTMEMLoadInLoop : public OpRewritePattern<ttng::TMEMLoadOp> {
 357: public:
 358:   using OpRewritePattern::OpRewritePattern;
```

- **EN:** Defines `RotateTMEMLoadInLoop`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `RotateTMEMLoadInLoop`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 360-374

```cpp
 360:   LogicalResult matchAndRewrite(ttng::TMEMLoadOp load,
 361:                                 PatternRewriter &rewriter) const override {
 362:     if (!load.getDep())
 363:       return failure();
 364:     // Pattern match loads whose results are only passed into the next iteration
 365:     // of a loop.
 366:     scf::ForOp forOp = dyn_cast<scf::ForOp>(load->getParentOp());
 367:     if (!forOp || !forOp.isDefinedOutsideOfLoop(load.getSrc()) ||
 368:         !load.getResult().hasOneUse()) {
 369:       return failure();
 370:     }
 371:     OpOperand &use = *load.getResult().use_begin();
 372:     auto yield = dyn_cast<scf::YieldOp>(use.getOwner());
 373:     if (!yield)
 374:       return failure();
```

- **EN:** Defines `matchAndRewrite`, the core MLIR rewrite callback. It checks whether the source operation matches the pattern and then materializes the lowered replacement.
- **CN:** 这里定义 `matchAndRewrite`，即 MLIR 模式重写的核心回调。它先检查源操作是否匹配模式，再构造降级后的替代结果。
### Lines 376-379

```cpp
 376:     // By rotating the load into the future, we are essentially merging the
 377:     // loop-carried tensor value into the same TMEM allocation as the load.
 378:     // Thus, they cannot be live at the same time. Check this by ensuring we
 379:     // won't clobber the memory.
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 381-390

```cpp
 381:     // 1. There are no aliasing stores between the load and the end of the loop.
 382:     if (!llvm::is_contained(load.getToken().getUsers(), yield))
 383:       return failure();
 384:     // 2. The TMEM variable is live into the loop with an undefined value.
 385:     int tokArgNo = load.getToken().use_begin()->getOperandNumber();
 386:     Value initTok = forOp.getInitArgs()[tokArgNo];
 387:     auto initAlloc = initTok.getDefiningOp<TMEMTokenAllocOp>();
 388:     if (!initAlloc || initAlloc.getSrc())
 389:       return failure();
 390:     // TODO: 3. The live-in value of the TMEM variable is never read.
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 392-401

```cpp
 392:     // Create a store before the loop to write the initial value.
 393:     int argNo = use.getOperandNumber();
 394:     Value initVal = forOp.getInitArgs()[argNo];
 395:     rewriter.setInsertionPoint(forOp);
 396:     auto vTrue = arith::ConstantIntOp::create(rewriter, load.getLoc(), 1, 1);
 397:     auto tokType = rewriter.getType<AsyncTokenType>();
 398:     auto initStore = ttng::TMEMStoreOp::create(
 399:         rewriter, load.getLoc(), tokType, load.getSrc(), initAlloc.getToken(),
 400:         initVal, vTrue);
 401:     forOp.getInitArgsMutable()[tokArgNo].assign(initStore.getToken());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 403-409

```cpp
 403:     // Move the load to the beginning of the loop to load the tensor value.
 404:     yield.setOperand(tokArgNo, load.getDep());
 405:     rewriter.moveOpBefore(load, &forOp.getBody()->front());
 406:     Value tokArg = forOp.getRegionIterArg(tokArgNo);
 407:     load.getDepMutable().assign(tokArg);
 408:     tokArg.replaceAllUsesExcept(load.getToken(), load);
 409:     forOp.getRegionIterArg(argNo).replaceAllUsesWith(load.getResult());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 411-422

```cpp
 411:     // Load from the tmem after the loop, and use it instead of the loop carried
 412:     // value.
 413:     rewriter.setInsertionPointAfter(forOp);
 414:     auto loadAfterLoop = ttng::TMEMLoadOp::create(
 415:         rewriter, load.getLoc(), load.getResult().getType(), tokType,
 416:         load.getSrc(), forOp.getResult(tokArgNo));
 417:     forOp->getResult(argNo).replaceAllUsesWith(loadAfterLoop.getResult());
 418:     // Loop carried value is no longer used, short-circuit it.
 419:     yield.setOperand(argNo, forOp.getRegionIterArg(argNo));
 420:     return success();
 421:   }
 422: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 424-432

```cpp
 424: // Helper: return true if the value is a constant boolean with the expected
 425: // value.
 426: static bool isConstBool(Value v, bool expected) {
 427:   if (auto cst = v.getDefiningOp<arith::ConstantOp>()) {
 428:     if (auto attr = dyn_cast<BoolAttr>(cst.getValueAttr()))
 429:       return attr.getValue() == expected;
 430:   }
 431:   return false;
 432: }
```

- **EN:** Defines `isConstBool`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `isConstBool`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 434-438

```cpp
 434: // Helper to detect if the use of an alloc fully overwrites it.
 435: struct AllocUse {
 436:   Operation *useOp;
 437:   MutableOperandRange dep;
 438: };
```

- **EN:** Defines `AllocUse`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `AllocUse`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 440-457

```cpp
 440: static FailureOr<AllocUse> useOverwritesAlloc(ttng::TMEMAllocOp alloc,
 441:                                               BlockArgument tokArg) {
 442:   if (!tokArg.hasOneUse())
 443:     return failure();
 444:   OpOperand &onlyUse = *tokArg.use_begin();
 445:   Operation *useOp = onlyUse.getOwner();
 446:   if (auto store = dyn_cast<ttng::TMEMStoreOp>(useOp)) {
 447:     if (store.getDst() != alloc.getResult() ||
 448:         !isConstBool(store.getPred(), true))
 449:       return failure();
 450:     return AllocUse{useOp, store.getDepMutable()};
 451:   }
 452:   if (auto mma = dyn_cast<ttng::MMAv5OpInterface>(useOp)) {
 453:     if (onlyUse.get() == mma.getAccDep() &&
 454:         mma.getAccumulator() == alloc.getResult() &&
 455:         isConstBool(mma.useAccumulator(), false) &&
 456:         isConstBool(mma.getPredicate(), true))
 457:       return AllocUse{useOp, mma.getAccDepMutable()};
```

- **EN:** Defines `useOverwritesAlloc`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `useOverwritesAlloc`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 458-460

```cpp
 458:   }
 459:   return failure();
 460: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 462-464

```cpp
 462: class SinkTMemAlloc : public OpRewritePattern<ttng::TMEMAllocOp> {
 463: public:
 464:   using OpRewritePattern::OpRewritePattern;
```

- **EN:** Defines `SinkTMemAlloc`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `SinkTMemAlloc`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 466-470

```cpp
 466:   LogicalResult matchAndRewrite(ttng::TMEMAllocOp alloc,
 467:                                 PatternRewriter &rewriter) const override {
 468:     // Only handle allocs that produce a token.
 469:     if (!alloc.getToken())
 470:       return failure();
```

- **EN:** Defines `matchAndRewrite`, the core MLIR rewrite callback. It checks whether the source operation matches the pattern and then materializes the lowered replacement.
- **CN:** 这里定义 `matchAndRewrite`，即 MLIR 模式重写的核心回调。它先检查源操作是否匹配模式，再构造降级后的替代结果。
### Lines 472-486

```cpp
 472:     // Find a forOp that takes the alloc token as an init arg.
 473:     // Limit to the simple case where the token has exactly one use.
 474:     if (!alloc.getToken().hasOneUse())
 475:       return failure();
 476:     OpOperand &tokUse = *alloc.getToken().use_begin();
 477:     auto userFor = dyn_cast<scf::ForOp>(tokUse.getOwner());
 478:     if (!userFor)
 479:       return failure();
 480:     scf::ForOp forOp = userFor;
 481:     if (forOp.getInitArgs().empty())
 482:       return failure();
 483:     auto firstInitIt = forOp.getInitArgsMutable().begin();
 484:     int baseOpNo = firstInitIt->getOperandNumber();
 485:     int tokIdx = tokUse.getOperandNumber() - baseOpNo;
 486:     assert(tokIdx >= 0 && tokIdx < (int)forOp.getInitArgs().size());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 488-493

```cpp
 488:     // Ensure the memory descriptor result of the alloc is only used inside the
 489:     // loop. Otherwise sinking would break users after the loop.
 490:     for (Operation *user : alloc.getResult().getUsers()) {
 491:       if (!forOp->isProperAncestor(user))
 492:         return failure();
 493:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 495-495

```cpp
 495:     BlockArgument tokArg = forOp.getRegionIterArg(tokIdx);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 497-502

```cpp
 497:     // Check if the op consuming the tocken fully overwrites the alloc. This
 498:     // means there is no loop carried values.
 499:     auto sinkable = useOverwritesAlloc(alloc, tokArg);
 500:     if (failed(sinkable))
 501:       return failure();
 502:     Operation *useOp = sinkable->useOp;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 504-506

```cpp
 504:     // Since the alloc is not used outside the loop its token should not be
 505:     // used.
 506:     assert(forOp.getResult(tokIdx).use_empty());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 508-511

```cpp
 508:     // Move the alloc just before the store to minimize live range inside the
 509:     // loop.
 510:     rewriter.moveOpBefore(alloc, useOp);
 511:     sinkable->dep.assign(alloc.getToken());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 513-519

```cpp
 513:     // Leave the token loop arguments to dead code.
 514:     auto yield = cast<scf::YieldOp>(forOp.getBody()->getTerminator());
 515:     yield.setOperand(tokIdx, forOp.getRegionIterArg(tokIdx));
 516:     rewriter.setInsertionPoint(forOp);
 517:     auto poisonTok = ub::PoisonOp::create(
 518:         rewriter, forOp.getLoc(), forOp.getInitArgs()[tokIdx].getType());
 519:     forOp.getInitArgsMutable()[tokIdx].assign(poisonTok);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 521-523

```cpp
 521:     return success();
 522:   }
 523: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 525-537

```cpp
 525: // Given an operation that uses a token, return its forwarded token. This
 526: // assumes the memory variable is not loop carried.
 527: static Value getTokenFromOp(Operation *op) {
 528:   if (auto mmaOp = dyn_cast<HasToken<ttng::MMAv5OpInterface>>(op)) {
 529:     return mmaOp.getToken();
 530:   } else if (auto loadOp = dyn_cast<TMEMTokenLoadOp>(op)) {
 531:     return loadOp.getToken();
 532:   } else if (auto storeOp = dyn_cast<TMEMTokenStoreOp>(op)) {
 533:     return storeOp.getToken();
 534:   }
 535:   assert(!isa<scf::YieldOp>(op) && "unexpected loop carried token");
 536:   llvm_unreachable("unknown TMEM memory user");
 537: }
```

- **EN:** Defines accessor/helper `getTokenFromOp` that exposes or updates operation state in a compact, reusable way. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义访问器/辅助函数 `getTokenFromOp`，以紧凑且可复用的方式读取或更新操作状态。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 539-552

```cpp
 539: // Find all the last uses of a memory variable in a loop body. This traces the
 540: // token lattice to its leaves.
 541: static void findLastMemoryUses(OpResult token,
 542:                                SmallVectorImpl<OpResult> &lastUses,
 543:                                DenseSet<Value> &seen) {
 544:   if (!seen.insert(token).second)
 545:     return;
 546:   if (token.use_empty()) {
 547:     lastUses.push_back(token);
 548:     return;
 549:   }
 550:   for (Operation *user : token.getUsers())
 551:     findLastMemoryUses(cast<OpResult>(getTokenFromOp(user)), lastUses, seen);
 552: }
```

- **EN:** Defines `findLastMemoryUses`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `findLastMemoryUses`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 554-560

```cpp
 554: // Find the last uses of a memory variable, joining them into a single token if
 555: // necessary. This token can be carried into the next loop iteration.
 556: static Value joinLastMemoryUses(OpBuilder &b, Value token) {
 557:   SmallVector<OpResult> lastUses;
 558:   DenseSet<Value> seenTokens;
 559:   findLastMemoryUses(cast<OpResult>(token), lastUses, seenTokens);
 560:   assert(!lastUses.empty());
```

- **EN:** Defines `joinLastMemoryUses`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `joinLastMemoryUses`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 562-568

```cpp
 562:   if (lastUses.size() == 1 && lastUses.front().getDefiningOp()->getBlock() ==
 563:                                   token.getDefiningOp()->getBlock())
 564:     return lastUses.front();
 565:   // We can handle this case as needed. Right now it never happens.
 566:   llvm::report_fatal_error(
 567:       "FIXME: can't hoist TMEM alloc with multiple or conditional uses");
 568: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 570-576

```cpp
 570: ttng::TMEMAllocOp hoistTMEMAlloc(TMEMTokenAllocOp alloc, scf::ForOp &forOp) {
 571:   OpBuilder builder(alloc);
 572:   builder.setInsertionPoint(forOp);
 573:   Value vTrue = arith::ConstantIntOp::create(builder, alloc.getLoc(), 1, 1);
 574:   auto src = alloc.getSrc();
 575:   auto newAlloc = cast<ttng::TMEMAllocOp>(builder.clone(*alloc));
 576:   newAlloc.getSrcMutable().clear();
```

- **EN:** Defines `hoistTMEMAlloc`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `hoistTMEMAlloc`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 578-583

```cpp
 578:   // By hoisting the allocation out of the loop, we need to turn the underlying
 579:   // memory variable into a loop-carried depdendency.
 580:   auto tokType = builder.getType<AsyncTokenType>();
 581:   forOp = addIterArgsToLoop(builder, forOp, newAlloc.getToken());
 582:   Value newTok = forOp.getRegionIterArgs().back();
 583:   appendToForOpYield(forOp, joinLastMemoryUses(builder, alloc.getToken()));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 585-594

```cpp
 585:   if (src != nullptr) {
 586:     builder.setInsertionPoint(alloc);
 587:     // Write the initial value of the allocation and replace the token.
 588:     auto initStoreOp =
 589:         ttng::TMEMStoreOp::create(builder, alloc.getLoc(), tokType,
 590:                                   newAlloc.getResult(), newTok, src, vTrue);
 591:     newTok = initStoreOp.getToken();
 592:   }
 593:   alloc.replaceAllUsesWith(ValueRange{newAlloc.getResult(), newTok});
 594:   alloc.erase();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 596-597

```cpp
 596:   return newAlloc;
 597: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 599-616

```cpp
 599: // Hoist invariant tmem_alloc. This could technically be done as general LICM
 600: // but controlling tmem liveranga more precisley is likely to be important.
 601: static void hoistInvariantInputs(Operation *mmaOp, scf::ForOp forOp) {
 602:   for (auto operand : mmaOp->getOperands()) {
 603:     if (forOp.isDefinedOutsideOfLoop(operand))
 604:       continue;
 605:     auto tmemAllocOp = operand.getDefiningOp<ttng::TMEMAllocOp>();
 606:     if (!tmemAllocOp || tmemAllocOp.getType().getMutableMemory())
 607:       continue;
 608:     assert(tmemAllocOp.getSrc());
 609:     Value src = tmemAllocOp.getSrc();
 610:     SmallVector<Operation *> opToHoist = {tmemAllocOp.getOperation()};
 611:     // Also hoist simple unary elementwise that may have sinked into the loop.
 612:     while (Operation *defOp = src.getDefiningOp()) {
 613:       if (forOp.isDefinedOutsideOfLoop(src))
 614:         break;
 615:       if (!(isPure(defOp) && defOp->getNumOperands() == 1))
 616:         break;
```

- **EN:** Defines `hoistInvariantInputs`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `hoistInvariantInputs`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 617-627

```cpp
 617:       opToHoist.push_back(defOp);
 618:       src = defOp->getOperand(0);
 619:     }
 620:     if (!forOp.isDefinedOutsideOfLoop(src))
 621:       continue;
 622:     for (auto op : llvm::reverse(opToHoist)) {
 623:       forOp.moveOutOfLoop(op);
 624:     }
 625:   }
 626: }
 627: } // namespace
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 629-632

```cpp
 629: struct HoistTMEMAlloc
 630:     : public impl::TritonGPUHoistTMEMAllocBase<HoistTMEMAlloc> {
 631:   using impl::TritonGPUHoistTMEMAllocBase<
 632:       HoistTMEMAlloc>::TritonGPUHoistTMEMAllocBase;
```

- **EN:** Defines `HoistTMEMAlloc`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `HoistTMEMAlloc`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 634-644

```cpp
 634:   void runOnOperation() override {
 635:     ModuleOp m = getOperation();
 636:     if (!postPipeline) {
 637:       SmallVector<ttng::MMAv5OpInterface> mmaOps;
 638:       m.walk([&](ttng::MMAv5OpInterface mmaOp) { mmaOps.push_back(mmaOp); });
 639:       for (auto mmaOp : mmaOps) {
 640:         auto forOp = dyn_cast<scf::ForOp>(mmaOp->getParentOp());
 641:         if (!forOp) {
 642:           continue;
 643:         }
 644:         hoistInvariantInputs(mmaOp, forOp);
```

- **EN:** Defines `runOnOperation`, the pass entry point. It gathers analysis information and applies the file's transformation logic to the current operation. The implementation traverses IR operations to collect facts or apply rewrites globally. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `runOnOperation`，即 pass 的入口函数。它会收集分析信息，并把本文件的变换逻辑应用到当前操作上。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 646-654

```cpp
 646:         // Only hoist the TMEM alloc feeding into the accumulator. Leave the
 647:         // ones for the scales in the loop.
 648:         auto alloc = mmaOp.getAccumulator().getDefiningOp<TMEMTokenAllocOp>();
 649:         if (!alloc || alloc->getParentRegion() != mmaOp->getParentRegion()) {
 650:           continue;
 651:         }
 652:         hoistTMEMAlloc(alloc, forOp);
 653:       }
 654:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 656-667

```cpp
 656:     mlir::RewritePatternSet patterns(&getContext());
 657:     patterns.add<RotateTMEMStoreInLoop, RotateTMEMLoadInLoop,
 658:                  CombineTMEMLoadAndStore, CombineTMEMStoreAndSelect,
 659:                  SinkTMEMLoad, RemoveUnusedTMEMLoad>(&getContext());
 660:     if (postPipeline) {
 661:       patterns.add<CombineTMEMStoreAndAlloc, HoistTMEMAllocOutOfIf,
 662:                    TMEMLoadForwarding>(&getContext());
 663:     }
 664:     scf::ForOp::getCanonicalizationPatterns(patterns, &getContext());
 665:     if (failed(applyPatternsGreedily(getOperation(), std::move(patterns)))) {
 666:       llvm_unreachable("Failed to hoist tmem_store");
 667:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 669-677

```cpp
 669:     // Finally try to sink the alloc that can be to reduce tmem liveranges.
 670:     if (postPipeline) {
 671:       mlir::RewritePatternSet postPatterns(&getContext());
 672:       postPatterns.add<SinkTMemAlloc>(&getContext());
 673:       if (failed(
 674:               applyPatternsGreedily(getOperation(), std::move(postPatterns)))) {
 675:         signalPassFailure();
 676:       }
 677:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 679-696

```cpp
 679:     // TODO: currently some code assumes that a mutable tmem alloc doesn't have
 680:     // an initial value. As a workaround we break up the op in order to keep
 681:     // this form for the downstream passes. We should remove this once the
 682:     // downstread passes are fixed.
 683:     m.walk([&](ttng::TMEMAllocOp alloc) {
 684:       if (alloc.getType().getMutableMemory() && alloc.getSrc()) {
 685:         OpBuilder builder(alloc);
 686:         builder.setInsertionPointAfter(alloc);
 687:         auto store = ttng::TMEMStoreOp::create(
 688:             builder, alloc.getLoc(), builder.getType<AsyncTokenType>(),
 689:             alloc.getResult(), alloc.getToken(), alloc.getSrc(),
 690:             arith::ConstantIntOp::create(builder, alloc.getLoc(), 1, 1));
 691:         alloc.getToken().replaceAllUsesExcept(store.getToken(), store);
 692:         alloc.getSrcMutable().clear();
 693:       }
 694:     });
 695:   }
 696: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. The implementation traverses IR operations to collect facts or apply rewrites globally.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。
### Lines 698-700

```cpp
 698: } // namespace gpu
 699: } // namespace triton
 700: } // namespace mlir
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。

## Key Concepts / 关键概念
- **EN:** The main concern is pass-driven transformation around hoist tmem alloc.
  **CN:** 核心关注点是围绕 Hoist TMEM Alloc 的 pass 驱动变换。
- **EN:** Pattern rewriting is the main mechanism for canonicalization and lowering in this file.
  **CN:** 模式重写是本文件进行规范化与降级转换的核心机制。
- **EN:** The code reasons at module scope rather than only on isolated operations.
  **CN:** 代码在模块范围内进行分析，而不只是处理单个操作。
- **EN:** Tensor memory specific allocation, layout, or synchronization rules are important here.
  **CN:** 这里重点处理张量内存（TMEM）的分配、布局或同步规则。

## Dependencies / 依赖关系
- **Internal Triton headers / Triton 内部头文件:** `triton/Dialect/TritonGPU/IR/Dialect.h`, `triton/Dialect/TritonGPU/Transforms/MMAv5PipelineUtility.h`, `triton/Dialect/TritonGPU/Transforms/Passes.h`, `triton/Dialect/TritonGPU/Transforms/PipeliningUtility.h`, `triton/Dialect/TritonGPU/Transforms/Utility.h`, `triton/Dialect/TritonNvidiaGPU/IR/Dialect.h`, ... (+1 more)
- **MLIR headers / MLIR 头文件:** `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/SCF/IR/SCF.h`, `mlir/Dialect/UB/IR/UBOps.h`, `mlir/IR/Dominance.h`, `mlir/Transforms/GreedyPatternRewriteDriver.h`
- **LLVM headers / LLVM 头文件:** None
- **Standard/library headers / 标准或通用库头文件:** None
- **Generated fragments / 生成片段:** `triton/Dialect/TritonGPU/Transforms/Passes.h.inc`
- **Primary APIs used / 主要 API:** `PatternRewriter`, `ModuleOp`, `OpBuilder`
- **Pipeline role / 流程角色:** This file participates in Triton pass pipelines and usually expects upstream analyses or dialect invariants to have prepared the IR. / 本文件参与 Triton 的 pass 流程，通常依赖上游分析或方言不变量先把 IR 准备好。
