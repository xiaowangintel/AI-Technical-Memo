# LoopAwareCSE.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `lib/Dialect/Triton/Transforms/LoopAwareCSE.cpp`
- **Purpose / 作用:** **EN:** Implements the Loop Aware CSE transformation or optimization pass for the Triton pipeline. **CN:** 为 Triton 编译流程实现与 Loop Aware CSE 相关的变换或优化 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
   1: #include "mlir/Dialect/SCF/IR/SCF.h"
   2: #include "mlir/IR/Dominance.h"
   3: #include "mlir/Pass/Pass.h"
   4: #include "mlir/Transforms/CSE.h"
   5: #include "mlir/Transforms/GreedyPatternRewriteDriver.h"
   6: #include "llvm/ADT/EquivalenceClasses.h"
```

- **EN:** Includes the interfaces this file depends on. Triton headers (None) provide domain-specific IR/support, MLIR headers (`SCF.h`, `Dominance.h`, `Pass.h`, `CSE.h`, ... (+1 more)) provide rewriting and analysis infrastructure, LLVM headers (`EquivalenceClasses.h`) supply low-level utilities, and standard/library headers (None) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（None）提供领域特定 IR/支持逻辑，MLIR 头文件（`SCF.h`, `Dominance.h`, `Pass.h`, `CSE.h`, ... (+1 more)）提供重写与分析基础设施，LLVM 头文件（`EquivalenceClasses.h`）提供底层工具，而标准/通用库头文件（None）提供通用能力。
### Lines 8-8

```cpp
   8: using namespace mlir;
```

- **EN:** Introduces namespace aliases/imports (`mlir`) so the rest of the file can use MLIR/Triton symbols without repetitive qualification.
- **CN:** 这里通过命名空间导入（`mlir`）减少后续代码中的重复限定，使 MLIR/Triton 符号使用更简洁。
### Lines 10-13

```cpp
  10: namespace mlir::triton {
  11: #define GEN_PASS_DEF_TRITONLOOPAWARECSE
  12: #include "triton/Dialect/Triton/Transforms/Passes.h.inc"
  13: } // namespace mlir::triton
```

- **EN:** Pulls in generated definitions from TableGen/MLIR include fragments so the handwritten code can reuse auto-generated declarations.
- **CN:** 这里引入由 TableGen/MLIR 生成的定义片段，使手写代码能够复用自动生成的声明。
### Lines 15-17

```cpp
  15: namespace {
  16: struct LoopCSEDriver {
  17:   LoopCSEDriver(scf::ForOp loop) : loop(loop) {}
```

- **EN:** Defines `LoopCSEDriver`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `LoopCSEDriver`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 19-20

```cpp
  19:   bool areIterArgsEqual(int i, int j);
  20:   bool areEqualInLoop(Value a, Value b);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 22-25

```cpp
  22:   scf::ForOp loop;
  23:   SmallVector<std::pair<int, int>> argStack;
  24: };
  25: } // namespace
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 27-33

```cpp
  27: bool LoopCSEDriver::areIterArgsEqual(int i, int j) {
  28:   if (i == j)
  29:     return true;
  30:   if (loop.getInitArgs()[i] != loop.getInitArgs()[j])
  31:     return false;
  32:   if (llvm::is_contained(argStack, std::make_pair(i, j)))
  33:     return true;
```

- **EN:** Defines `LoopCSEDriver::areIterArgsEqual`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `LoopCSEDriver::areIterArgsEqual`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 35-41

```cpp
  35:   // First, assume the arguments are equal. This is how recursion is broken.
  36:   argStack.push_back({i, j});
  37:   bool result =
  38:       areEqualInLoop(loop.getYieldedValues()[i], loop.getYieldedValues()[j]);
  39:   argStack.pop_back();
  40:   return result;
  41: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 43-48

```cpp
  43: bool LoopCSEDriver::areEqualInLoop(Value a, Value b) {
  44:   // Check trivial case.
  45:   if (a == b)
  46:     return true;
  47:   if (a.getType() != b.getType())
  48:     return false;
```

- **EN:** Defines `LoopCSEDriver::areEqualInLoop`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `LoopCSEDriver::areEqualInLoop`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 50-61

```cpp
  50:   Block *aBlock = a.getParentBlock();
  51:   Block *bBlock = b.getParentBlock();
  52:   // Values from outside the loop must have been equal.
  53:   if (aBlock != loop.getBody() || bBlock != loop.getBody()) {
  54:     return false;
  55:   }
  56:   // Both must be block arguments or not.
  57:   if (isa<BlockArgument>(a) != isa<BlockArgument>(b))
  58:     return false;
  59:   // Both must be the inductor var or not.
  60:   if (a == loop.getInductionVar() || b == loop.getInductionVar())
  61:     return false;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 63-68

```cpp
  63:   if (auto aArg = dyn_cast<BlockArgument>(a)) {
  64:     auto bArg = cast<BlockArgument>(b);
  65:     bool result =
  66:         areIterArgsEqual(aArg.getArgNumber() - 1, bArg.getArgNumber() - 1);
  67:     return result;
  68:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 70-81

```cpp
  70:   Operation *aDef = a.getDefiningOp();
  71:   Operation *bDef = b.getDefiningOp();
  72:   if (cast<OpResult>(a).getResultNumber() !=
  73:       cast<OpResult>(b).getResultNumber())
  74:     return false;
  75:   // For it to be known that the operation results have the same value, they
  76:   // must be side effect free.
  77:   if (!isMemoryEffectFree(aDef) || !isMemoryEffectFree(bDef))
  78:     return false;
  79:   // Don't bother with operations with regions.
  80:   if (aDef->getNumRegions() || bDef->getNumRegions())
  81:     return false;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 83-88

```cpp
  83:   bool result = OperationEquivalence::isEquivalentTo(
  84:       aDef, bDef,
  85:       [&](Value a, Value b) { return success(areEqualInLoop(a, b)); },
  86:       /*markEquivalent=*/nullptr, OperationEquivalence::IgnoreLocations);
  87:   return result;
  88: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 90-100

```cpp
  90: static void loopCSE(scf::ForOp loop) {
  91:   int numIterArgs = loop.getNumRegionIterArgs();
  92:   // Group equivalent iter args together.
  93:   llvm::EquivalenceClasses<int> equivalentArgs;
  94:   LoopCSEDriver driver(loop);
  95:   for (int i = 0; i != numIterArgs; ++i) {
  96:     for (int j = i + 1; j != numIterArgs; ++j) {
  97:       if (driver.areIterArgsEqual(i, j))
  98:         equivalentArgs.unionSets(i, j);
  99:     }
 100:   }
```

- **EN:** Defines `loopCSE`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `loopCSE`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 102-119

```cpp
 102:   // For each equivalence class, replace all other args in the class with one.
 103:   for (auto it = equivalentArgs.begin(), end = equivalentArgs.end(); it != end;
 104:        ++it) {
 105:     if (!(*it)->isLeader())
 106:       continue;
 107:     SmallVector<int> eqArgs;
 108:     for (auto mIt = equivalentArgs.member_begin(**it);
 109:          mIt != equivalentArgs.member_end(); ++mIt)
 110:       eqArgs.push_back(*mIt);
 111:     assert(eqArgs.size() > 1);
 112:     // Sort the indices so the pass is deterministic.
 113:     llvm::sort(eqArgs);
 114:     BlockArgument unique = loop.getRegionIterArg(eqArgs.front());
 115:     Value uniqueResult = loop.getResult(eqArgs.front());
 116:     for (int j : llvm::drop_begin(eqArgs)) {
 117:       BlockArgument other = loop.getRegionIterArg(j);
 118:       other.replaceAllUsesWith(unique);
 119:       // Short-circuit the value. The canonicalizer will clean this up. Leftover
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 120-125

```cpp
 120:       // subcomputations can now be removed by normal CSE.
 121:       (*loop.getYieldedValuesMutable())[j].set(other);
 122:       loop.getResult(j).replaceAllUsesWith(uniqueResult);
 123:     }
 124:   }
 125: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 127-130

```cpp
 127: namespace {
 128: struct LoopAwareCSE
 129:     : public triton::impl::TritonLoopAwareCSEBase<LoopAwareCSE> {
 130:   using TritonLoopAwareCSEBase::TritonLoopAwareCSEBase;
```

- **EN:** Defines `LoopAwareCSE`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `LoopAwareCSE`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 132-138

```cpp
 132:   void runOnOperation() override {
 133:     // LoopAwareCSE doesn't recursively CSE ops outside of loops, so run CSE
 134:     // first to make sure values from outside loops that are equivalent are made
 135:     // pointer equal.
 136:     IRRewriter rewriter(&getContext());
 137:     auto &domInfo = getAnalysis<DominanceInfo>();
 138:     eliminateCommonSubExpressions(rewriter, domInfo, getOperation());
```

- **EN:** Defines `runOnOperation`, the pass entry point. It gathers analysis information and applies the file's transformation logic to the current operation.
- **CN:** 这里定义 `runOnOperation`，即 pass 的入口函数。它会收集分析信息，并把本文件的变换逻辑应用到当前操作上。
### Lines 140-141

```cpp
 140:     // CSE region iter args within loop bodies.
 141:     getOperation().walk(loopCSE);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 143-145

```cpp
 143:     // Now that equivalent iter args have been made pointer equal, run CSE again
 144:     // to clean up the loop body.
 145:     eliminateCommonSubExpressions(rewriter, domInfo, getOperation());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 147-155

```cpp
 147:     // Run the `scf.for` canonicalizer to clean up the loops (short-circuited
 148:     // values, unused results, etc.).
 149:     RewritePatternSet patterns(&getContext());
 150:     scf::ForOp::getCanonicalizationPatterns(patterns, &getContext());
 151:     if (failed(applyPatternsGreedily(getOperation(), std::move(patterns))))
 152:       return signalPassFailure();
 153:   }
 154: };
 155: } // namespace
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。

## Key Concepts / 关键概念
- **EN:** The main concern is pass-driven transformation around loop aware cse.
  **CN:** 核心关注点是围绕 Loop Aware CSE 的 pass 驱动变换。

## Dependencies / 依赖关系
- **Internal Triton headers / Triton 内部头文件:** `triton/Dialect/Triton/Transforms/Passes.h.inc`
- **MLIR headers / MLIR 头文件:** `mlir/Dialect/SCF/IR/SCF.h`, `mlir/IR/Dominance.h`, `mlir/Pass/Pass.h`, `mlir/Transforms/CSE.h`, `mlir/Transforms/GreedyPatternRewriteDriver.h`
- **LLVM headers / LLVM 头文件:** `llvm/ADT/EquivalenceClasses.h`
- **Standard/library headers / 标准或通用库头文件:** None
- **Generated fragments / 生成片段:** `triton/Dialect/Triton/Transforms/Passes.h.inc`
- **Pipeline role / 流程角色:** This file participates in Triton pass pipelines and usually expects upstream analyses or dialect invariants to have prepared the IR. / 本文件参与 Triton 的 pass 流程，通常依赖上游分析或方言不变量先把 IR 准备好。
