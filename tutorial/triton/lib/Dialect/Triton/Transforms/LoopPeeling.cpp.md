# LoopPeeling.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `lib/Dialect/Triton/Transforms/LoopPeeling.cpp`
- **Purpose / 作用:** **EN:** Implements the Loop Peeling transformation or optimization pass for the Triton pipeline. **CN:** 为 Triton 编译流程实现与 Loop Peeling 相关的变换或优化 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-4

```cpp
   1: #include "triton/Dialect/Triton/Transforms/LoopPeeling.h"
   2: #include "mlir/Dialect/SCF/IR/SCF.h"
   3: #include "mlir/Pass/Pass.h"
   4: #include "triton/Dialect/Triton/IR/Utility.h"
```

- **EN:** Includes the interfaces this file depends on. Triton headers (`LoopPeeling.h`, `Utility.h`) provide domain-specific IR/support, MLIR headers (`SCF.h`, `Pass.h`) provide rewriting and analysis infrastructure, LLVM headers (None) supply low-level utilities, and standard/library headers (None) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（`LoopPeeling.h`, `Utility.h`）提供领域特定 IR/支持逻辑，MLIR 头文件（`SCF.h`, `Pass.h`）提供重写与分析基础设施，LLVM 头文件（None）提供底层工具，而标准/通用库头文件（None）提供通用能力。
### Lines 6-6

```cpp
   6: using namespace mlir;
```

- **EN:** Introduces namespace aliases/imports (`mlir`) so the rest of the file can use MLIR/Triton symbols without repetitive qualification.
- **CN:** 这里通过命名空间导入（`mlir`）减少后续代码中的重复限定，使 MLIR/Triton 符号使用更简洁。
### Lines 8-9

```cpp
   8: namespace mlir {
   9: namespace triton {
```

- **EN:** Opens or closes the namespace nesting for mlir -> triton, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 mlir -> triton 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 11-17

```cpp
  11: void peelLoopEpilogue(
  12:     scf::ForOp forOp,
  13:     function_ref<Operation *(RewriterBase &, Operation *, bool)>
  14:         processPeeledOp) {
  15:   SmallVector<Operation *> loopBodyOps;
  16:   IRRewriter rewriter(forOp);
  17:   Location loc = forOp.getLoc();
```

- **EN:** Defines `peelLoopEpilogue`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `peelLoopEpilogue`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 19-23

```cpp
  19:   // Fetch loop bounds and step
  20:   Value lowerBound = forOp.getLowerBound();
  21:   Value upperBound = forOp.getUpperBound();
  22:   Value step = forOp.getStep();
  23:   Value newUpperBound = arith::SubIOp::create(rewriter, loc, upperBound, step);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 25-26

```cpp
  25:   rewriter.setInsertionPointAfter(forOp);
  26:   Value lastIV = getLastInductionValue(rewriter, forOp);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 28-29

```cpp
  28:   auto cond = arith::CmpIOp::create(rewriter, loc, arith::CmpIPredicate::slt,
  29:                                     lowerBound, upperBound);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 31-39

```cpp
  31:   // Create an if op to execute the peeled iteration
  32:   IRMapping map;
  33:   map.map(forOp.getRegionIterArgs(), forOp.getResults());
  34:   map.map(forOp.getInductionVar(), lastIV);
  35:   auto ifOp = scf::IfOp::create(rewriter, loc, forOp.getResultTypes(), cond);
  36:   forOp.getBodyRegion().cloneInto(&ifOp.getThenRegion(), map);
  37:   auto newElseBlock = rewriter.createBlock(&ifOp.getElseRegion());
  38:   rewriter.setInsertionPointToStart(newElseBlock);
  39:   scf::YieldOp::create(rewriter, loc, forOp.getResults());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 41-43

```cpp
  41:   forOp->replaceUsesWithIf(ifOp, [&](OpOperand &operand) {
  42:     return !ifOp->isAncestor(operand.getOwner());
  43:   });
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 45-45

```cpp
  45:   forOp.getUpperBoundMutable().assign(newUpperBound);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 47-63

```cpp
  47:   if (processPeeledOp) {
  48:     for (auto &op :
  49:          llvm::make_early_inc_range(forOp.getBody()->without_terminator())) {
  50:       Operation *newOp = processPeeledOp(rewriter, &op, /*isEpilogue=*/false);
  51:       if (newOp && newOp != &op) {
  52:         op.replaceAllUsesWith(newOp);
  53:       }
  54:     }
  55:     for (auto &op : llvm::make_early_inc_range(
  56:              ifOp.getThenRegion().front().without_terminator())) {
  57:       Operation *newOp = processPeeledOp(rewriter, &op, /*isEpilogue=*/true);
  58:       if (newOp && newOp != &op) {
  59:         op.replaceAllUsesWith(newOp);
  60:       }
  61:     }
  62:   }
  63: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 65-66

```cpp
  65: } // namespace triton
  66: } // namespace mlir
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。

## Key Concepts / 关键概念
- **EN:** The main concern is pass-driven transformation around loop peeling.
  **CN:** 核心关注点是围绕 Loop Peeling 的 pass 驱动变换。

## Dependencies / 依赖关系
- **Internal Triton headers / Triton 内部头文件:** `triton/Dialect/Triton/Transforms/LoopPeeling.h`, `triton/Dialect/Triton/IR/Utility.h`
- **MLIR headers / MLIR 头文件:** `mlir/Dialect/SCF/IR/SCF.h`, `mlir/Pass/Pass.h`
- **LLVM headers / LLVM 头文件:** None
- **Standard/library headers / 标准或通用库头文件:** None
- **Pipeline role / 流程角色:** This file participates in Triton pass pipelines and usually expects upstream analyses or dialect invariants to have prepared the IR. / 本文件参与 Triton 的 pass 流程，通常依赖上游分析或方言不变量先把 IR 准备好。
