# LoopInvariantCodeMotion.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `lib/Dialect/Triton/Transforms/LoopInvariantCodeMotion.cpp`
- **Purpose / 作用:** **EN:** Implements the Loop Invariant Code Motion transformation or optimization pass for the Triton pipeline. **CN:** 为 Triton 编译流程实现与 Loop Invariant Code Motion 相关的变换或优化 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-5

```cpp
   1: #include "mlir/Transforms/LoopInvariantCodeMotionUtils.h"
   2: #include "triton/Dialect/Triton/IR/Dialect.h"
   3: #include "triton/Dialect/Triton/IR/Utility.h"
   4: #include "triton/Dialect/Triton/Transforms/Passes.h"
   5: #include "llvm/Support/Debug.h"
```

- **EN:** Includes the interfaces this file depends on. Triton headers (`Dialect.h`, `Utility.h`, `Passes.h`) provide domain-specific IR/support, MLIR headers (`LoopInvariantCodeMotionUtils.h`) provide rewriting and analysis infrastructure, LLVM headers (`Debug.h`) supply low-level utilities, and standard/library headers (None) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（`Dialect.h`, `Utility.h`, `Passes.h`）提供领域特定 IR/支持逻辑，MLIR 头文件（`LoopInvariantCodeMotionUtils.h`）提供重写与分析基础设施，LLVM 头文件（`Debug.h`）提供底层工具，而标准/通用库头文件（None）提供通用能力。
### Lines 7-7

```cpp
   7: namespace mlir::triton {
```

- **EN:** Opens or closes the namespace nesting for mlir::triton, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 mlir::triton 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 9-10

```cpp
   9: #define GEN_PASS_DEF_TRITONLOOPINVARIANTCODEMOTION
  10: #include "triton/Dialect/Triton/Transforms/Passes.h.inc"
```

- **EN:** Pulls in generated definitions from TableGen/MLIR include fragments so the handwritten code can reuse auto-generated declarations.
- **CN:** 这里引入由 TableGen/MLIR 生成的定义片段，使手写代码能够复用自动生成的声明。
### Lines 12-14

```cpp
  12: #define DEBUG_TYPE "triton-licm"
  13: #define DBGS() (llvm::dbgs() << "[" DEBUG_TYPE "]: ")
  14: #define LDBG(X) LLVM_DEBUG(DBGS() << X << "\n")
```

- **EN:** Defines debug logging helpers and compile-time tags used when tracing this pass or utility at runtime.
- **CN:** 这里定义调试日志辅助宏和编译期标签，用于在运行时跟踪该 pass 或工具。
### Lines 16-18

```cpp
  16: class LoopInvariantCodeMotionPass
  17:     : public impl::TritonLoopInvariantCodeMotionBase<
  18:           LoopInvariantCodeMotionPass> {
```

- **EN:** Defines `LoopInvariantCodeMotionPass`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `LoopInvariantCodeMotionPass`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 20-20

```cpp
  20:   DenseMap<LoopLikeOpInterface, bool> isLoopMemoryEffectFreeOrOnlyRead;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 22-31

```cpp
  22:   bool isMemoryEffectFreeOrOnlyRead(Operation *op) {
  23:     std::optional<SmallVector<MemoryEffects::EffectInstance>> effects =
  24:         getEffectsRecursively(op);
  25:     if (!effects)
  26:       return false;
  27:     return llvm::all_of(*effects,
  28:                         [&](const MemoryEffects::EffectInstance &effect) {
  29:                           return isa<MemoryEffects::Read>(effect.getEffect());
  30:                         });
  31:   }
```

- **EN:** Defines `isMemoryEffectFreeOrOnlyRead`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `isMemoryEffectFreeOrOnlyRead`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 33-50

```cpp
  33:   void runOnOperation() override {
  34:     // Walk through all loops in a function in innermost-loop-first order.
  35:     // This way, we first LICM from the inner loop, and place the ops in the
  36:     // outer loop, which in turn can be further LICM'ed.
  37:     getOperation()->walk([&](LoopLikeOpInterface loopLike) {
  38:       moveLoopInvariantCode(
  39:           loopLike.getLoopRegions(),
  40:           // isDefinedOutsideOfRegion
  41:           [&](Value value, Region *region) {
  42:             return loopLike.isDefinedOutsideOfLoop(value);
  43:           },
  44:           // shouldMoveOutOfRegion
  45:           [&](Operation *op, Region *region) {
  46:             if (!isa<LoadOp>(op))
  47:               return isSpeculatable(op) && isMemoryEffectFree(op);
  48:             if (!isLoopMemoryEffectFreeOrOnlyRead.contains(loopLike))
  49:               isLoopMemoryEffectFreeOrOnlyRead[loopLike] =
  50:                   isMemoryEffectFreeOrOnlyRead(loopLike);
```

- **EN:** Defines `runOnOperation`, the pass entry point. It gathers analysis information and applies the file's transformation logic to the current operation. The implementation traverses IR operations to collect facts or apply rewrites globally.
- **CN:** 这里定义 `runOnOperation`，即 pass 的入口函数。它会收集分析信息，并把本文件的变换逻辑应用到当前操作上。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。
### Lines 51-68

```cpp
  51:             return isMemoryEffectFreeOrOnlyRead(op) &&
  52:                    isLoopMemoryEffectFreeOrOnlyRead[loopLike];
  53:           },
  54:           // moveOutOfRegion
  55:           [&](Operation *op, Region *) {
  56:             // Create the new mask for load op.
  57:             if (auto loadOp = dyn_cast<LoadOp>(op)) {
  58:               IRRewriter rewriter(loopLike);
  59:               Location loc = loopLike->getLoc();
  60:               Value cond;
  61:               if (auto forOp = dyn_cast<scf::ForOp>(loopLike.getOperation())) {
  62:                 cond = arith::CmpIOp::create(
  63:                     rewriter, loc, arith::CmpIPredicate::slt,
  64:                     forOp.getLowerBound(), forOp.getUpperBound());
  65:               } else if (auto whileOp =
  66:                              dyn_cast<scf::WhileOp>(loopLike.getOperation())) {
  67:                 // TODO: Support Load Op hoisting for while loop.
  68:                 return;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 69-80

```cpp
  69:               } else {
  70:                 return;
  71:               }
  72:               Value newMask = getPredMask(rewriter, loadOp.getPtr().getType(),
  73:                                           loadOp.getMask(), cond);
  74:               loadOp.getMaskMutable().assign(newMask);
  75:             }
  76:             loopLike.moveOutOfLoop(op);
  77:           });
  78:     });
  79:   }
  80: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 82-82

```cpp
  82: } // namespace mlir::triton
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。

## Key Concepts / 关键概念
- **EN:** The main concern is pass-driven transformation around loop invariant code motion.
  **CN:** 核心关注点是围绕 Loop Invariant Code Motion 的 pass 驱动变换。

## Dependencies / 依赖关系
- **Internal Triton headers / Triton 内部头文件:** `triton/Dialect/Triton/IR/Dialect.h`, `triton/Dialect/Triton/IR/Utility.h`, `triton/Dialect/Triton/Transforms/Passes.h`, `triton/Dialect/Triton/Transforms/Passes.h.inc`
- **MLIR headers / MLIR 头文件:** `mlir/Transforms/LoopInvariantCodeMotionUtils.h`
- **LLVM headers / LLVM 头文件:** `llvm/Support/Debug.h`
- **Standard/library headers / 标准或通用库头文件:** None
- **Generated fragments / 生成片段:** `triton/Dialect/Triton/Transforms/Passes.h.inc`
- **Pipeline role / 流程角色:** This file participates in Triton pass pipelines and usually expects upstream analyses or dialect invariants to have prepared the IR. / 本文件参与 Triton 的 pass 流程，通常依赖上游分析或方言不变量先把 IR 准备好。
