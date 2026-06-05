# Ops.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `lib/Dialect/TritonInstrument/IR/Ops.cpp`
- **Purpose / 作用:** **EN:** Defines TritonInstrument operations, including builders, verifiers, folders, canonicalization hooks, and helper methods. **CN:** 定义 TritonInstrument 的操作，包括构造器、验证器、折叠逻辑、规范化钩子与辅助方法。

## Line-by-Line Analysis / 逐行分析

### Lines 1-5

```cpp
   1: #include "mlir/IR/PatternMatch.h"
   2: #include "triton/Dialect/Triton/IR/Dialect.h"
   3: #include "triton/Dialect/TritonGPU/IR/Dialect.h"
   4: #include "triton/Dialect/TritonInstrument/IR/Dialect.h"
   5: #include "triton/Dialect/TritonInstrument/IR/Utility.h"
```

- **EN:** Includes the interfaces this file depends on. Triton headers (`Dialect.h`, `Dialect.h`, `Dialect.h`, `Utility.h`) provide domain-specific IR/support, MLIR headers (`PatternMatch.h`) provide rewriting and analysis infrastructure, LLVM headers (None) supply low-level utilities, and standard/library headers (None) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（`Dialect.h`, `Dialect.h`, `Dialect.h`, `Utility.h`）提供领域特定 IR/支持逻辑，MLIR 头文件（`PatternMatch.h`）提供重写与分析基础设施，LLVM 头文件（None）提供底层工具，而标准/通用库头文件（None）提供通用能力。
### Lines 7-8

```cpp
   7: #define GET_OP_CLASSES
   8: #include "triton/Dialect/TritonInstrument/IR/Ops.cpp.inc"
```

- **EN:** Pulls in generated definitions from TableGen/MLIR include fragments so the handwritten code can reuse auto-generated declarations.
- **CN:** 这里引入由 TableGen/MLIR 生成的定义片段，使手写代码能够复用自动生成的声明。
### Lines 10-10

```cpp
  10: #include "triton/Dialect/TritonInstrument/IR/OpsEnums.cpp.inc"
```

- **EN:** Pulls in generated definitions from TableGen/MLIR include fragments so the handwritten code can reuse auto-generated declarations.
- **CN:** 这里引入由 TableGen/MLIR 生成的定义片段，使手写代码能够复用自动生成的声明。
### Lines 12-14

```cpp
  12: namespace mlir {
  13: namespace triton {
  14: namespace instrument {
```

- **EN:** Opens or closes the namespace nesting for mlir -> triton -> instrument, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 mlir -> triton -> instrument 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 16-17

```cpp
  16: namespace tt = mlir::triton;
  17: namespace ttg = mlir::triton::gpu;
```

- **EN:** Opens or closes the namespace nesting for tt -> ttg, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 tt -> ttg 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 19-21

```cpp
  19: template <typename ViewOp, typename FPSanOp>
  20: struct PushFPSanThroughViewPattern : public OpRewritePattern<ViewOp> {
  21:   using OpRewritePattern<ViewOp>::OpRewritePattern;
```

- **EN:** Defines `PushFPSanThroughViewPattern`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `PushFPSanThroughViewPattern`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 23-27

```cpp
  23:   LogicalResult matchAndRewrite(ViewOp view,
  24:                                 PatternRewriter &rewriter) const override {
  25:     auto fpsan = view->getOperand(0).template getDefiningOp<FPSanOp>();
  26:     if (!fpsan)
  27:       return failure();
```

- **EN:** Defines `matchAndRewrite`, the core MLIR rewrite callback. It checks whether the source operation matches the pattern and then materializes the lowered replacement.
- **CN:** 这里定义 `matchAndRewrite`，即 MLIR 模式重写的核心回调。它先检查源操作是否匹配模式，再构造降级后的替代结果。
### Lines 29-32

```cpp
  29:     auto resultTy = dyn_cast<RankedTensorType>(view->getResult(0).getType());
  30:     auto payloadTy = dyn_cast<RankedTensorType>(fpsan.getVal().getType());
  31:     if (!resultTy || !payloadTy)
  32:       return failure();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 34-45

```cpp
  34:     auto payloadViewTy = resultTy.clone(payloadTy.getElementType());
  35:     OperationState state(view.getLoc(), view->getName());
  36:     state.addOperands(fpsan.getVal());
  37:     state.addTypes(payloadViewTy);
  38:     state.addAttributes(view->getAttrs());
  39:     Operation *payloadView = rewriter.create(state);
  40:     auto moved = FPSanOp::create(rewriter, view.getLoc(), resultTy,
  41:                                  payloadView->getResult(0));
  42:     rewriter.replaceOp(view, moved->getResults());
  43:     return success();
  44:   }
  45: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. It finishes by replacing the original operation with the lowered form.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 它会在转换成功后用降级后的形式替换原始操作。
### Lines 47-55

```cpp
  47: template <typename FPSanOp>
  48: void addPushFPSanThroughViewPatterns(RewritePatternSet &patterns,
  49:                                      MLIRContext *context) {
  50:   patterns.add<PushFPSanThroughViewPattern<ttg::ConvertLayoutOp, FPSanOp>,
  51:                PushFPSanThroughViewPattern<tt::TransOp, FPSanOp>,
  52:                PushFPSanThroughViewPattern<tt::ReshapeOp, FPSanOp>,
  53:                PushFPSanThroughViewPattern<tt::BroadcastOp, FPSanOp>,
  54:                PushFPSanThroughViewPattern<tt::ExpandDimsOp, FPSanOp>>(context);
  55: }
```

- **EN:** Defines `addPushFPSanThroughViewPatterns`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `addPushFPSanThroughViewPatterns`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 57-61

```cpp
  57: void ExperimentalFPSanEmbedOp::getCanonicalizationPatterns(
  58:     RewritePatternSet &patterns, MLIRContext *context) {
  59:   // view(embed(x)) -> embed(view(x))
  60:   addPushFPSanThroughViewPatterns<ExperimentalFPSanEmbedOp>(patterns, context);
  61: }
```

- **EN:** Defines `ExperimentalFPSanEmbedOp::getCanonicalizationPatterns` to register canonicalization patterns that simplify this IR before later passes.
- **CN:** 这里定义 `ExperimentalFPSanEmbedOp::getCanonicalizationPatterns`，用于注册规范化模式，在后续 pass 前先简化该 IR。
### Lines 63-68

```cpp
  63: OpFoldResult ExperimentalFPSanEmbedOp::fold(FoldAdaptor adaptor) {
  64:   if (auto unembed = getVal().getDefiningOp<ExperimentalFPSanUnembedOp>())
  65:     if (unembed.getVal().getType() == getType())
  66:       return unembed.getVal();
  67:   return {};
  68: }
```

- **EN:** Defines `ExperimentalFPSanEmbedOp::fold` to simplify the IR by folding away redundant or constant-computable behavior.
- **CN:** 这里定义 `ExperimentalFPSanEmbedOp::fold`，通过折叠冗余或可常量计算的行为来简化 IR。
### Lines 70-75

```cpp
  70: void ExperimentalFPSanUnembedOp::getCanonicalizationPatterns(
  71:     RewritePatternSet &patterns, MLIRContext *context) {
  72:   // view(unembed(x)) -> unembed(view(x))
  73:   addPushFPSanThroughViewPatterns<ExperimentalFPSanUnembedOp>(patterns,
  74:                                                               context);
  75: }
```

- **EN:** Defines `ExperimentalFPSanUnembedOp::getCanonicalizationPatterns` to register canonicalization patterns that simplify this IR before later passes.
- **CN:** 这里定义 `ExperimentalFPSanUnembedOp::getCanonicalizationPatterns`，用于注册规范化模式，在后续 pass 前先简化该 IR。
### Lines 77-82

```cpp
  77: OpFoldResult ExperimentalFPSanUnembedOp::fold(FoldAdaptor adaptor) {
  78:   if (auto embed = getVal().getDefiningOp<ExperimentalFPSanEmbedOp>())
  79:     if (embed.getVal().getType() == getType())
  80:       return embed.getVal();
  81:   return {};
  82: }
```

- **EN:** Defines `ExperimentalFPSanUnembedOp::fold` to simplify the IR by folding away redundant or constant-computable behavior.
- **CN:** 这里定义 `ExperimentalFPSanUnembedOp::fold`，通过折叠冗余或可常量计算的行为来简化 IR。
### Lines 84-86

```cpp
  84: } // namespace instrument
  85: } // namespace triton
  86: } // namespace mlir
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。

## Key Concepts / 关键概念
- **EN:** The file defines IR semantics for ops in its dialect layer.
  **CN:** 本文件在方言层为 Ops 定义 IR 语义。
- **EN:** Pattern rewriting is the main mechanism for canonicalization and lowering in this file.
  **CN:** 模式重写是本文件进行规范化与降级转换的核心机制。
- **EN:** Tensor shape and element-type reasoning is central to the implementation.
  **CN:** 张量形状与元素类型推导是实现中的核心内容。
- **EN:** Folding/canonicalization simplifies IR before heavier passes run.
  **CN:** 折叠/规范化会在更重的优化之前先简化 IR。
- **EN:** Layout and encoding decisions are first-class because they determine how work and memory are distributed on the GPU.
  **CN:** 布局与编码决策是第一类问题，因为它们决定了 GPU 上工作与内存的分布方式。

## Dependencies / 依赖关系
- **Internal Triton headers / Triton 内部头文件:** `triton/Dialect/Triton/IR/Dialect.h`, `triton/Dialect/TritonGPU/IR/Dialect.h`, `triton/Dialect/TritonInstrument/IR/Dialect.h`, `triton/Dialect/TritonInstrument/IR/Utility.h`, `triton/Dialect/TritonInstrument/IR/Ops.cpp.inc`, `triton/Dialect/TritonInstrument/IR/OpsEnums.cpp.inc`
- **MLIR headers / MLIR 头文件:** `mlir/IR/PatternMatch.h`
- **LLVM headers / LLVM 头文件:** None
- **Standard/library headers / 标准或通用库头文件:** None
- **Generated fragments / 生成片段:** `triton/Dialect/TritonInstrument/IR/Ops.cpp.inc`, `triton/Dialect/TritonInstrument/IR/OpsEnums.cpp.inc`
- **Primary APIs used / 主要 API:** `PatternRewriter`, `RankedTensorType`, `OperationState`
- **IR role / IR 角色:** The code is consumed by parsers, printers, verifiers, folders, and downstream passes that need stable dialect semantics. / 这些代码会被解析器、打印器、验证器、折叠逻辑以及需要稳定方言语义的下游 pass 使用。
