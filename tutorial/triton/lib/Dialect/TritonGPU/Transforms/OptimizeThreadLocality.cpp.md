# OptimizeThreadLocality.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `lib/Dialect/TritonGPU/Transforms/OptimizeThreadLocality.cpp`
- **Purpose / 作用:** **EN:** Implements the Optimize Thread Locality transformation or optimization pass for the TritonGPU pipeline. **CN:** 为 TritonGPU 编译流程实现与 Optimize Thread Locality 相关的变换或优化 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2

```cpp
   1: #include <memory>
   2: #include <numeric>
```

- **EN:** Includes the interfaces this file depends on. Triton headers (None) provide domain-specific IR/support, MLIR headers (None) provide rewriting and analysis infrastructure, LLVM headers (None) supply low-level utilities, and standard/library headers (`memory`, `numeric`) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（None）提供领域特定 IR/支持逻辑，MLIR 头文件（None）提供重写与分析基础设施，LLVM 头文件（None）提供底层工具，而标准/通用库头文件（`memory`, `numeric`）提供通用能力。
### Lines 4-15

```cpp
   4: #include "mlir/IR/ImplicitLocOpBuilder.h"
   5: #include "mlir/Pass/Pass.h"
   6: #include "mlir/Pass/PassManager.h"
   7: #include "mlir/Support/LLVM.h"
   8: #include "mlir/Transforms/GreedyPatternRewriteDriver.h"
   9: #include "mlir/Transforms/Passes.h"
  10: #include "triton/Analysis/Utility.h"
  11: #include "triton/Dialect/Triton/IR/Utility.h"
  12: #include "triton/Dialect/TritonGPU/IR/Dialect.h"
  13: #include "triton/Dialect/TritonGPU/Transforms/Passes.h"
  14: #include "triton/Dialect/TritonGPU/Transforms/Utility.h"
  15: #include "triton/Tools/LayoutUtils.h"
```

- **EN:** Includes the interfaces this file depends on. Triton headers (`Utility.h`, `Utility.h`, `Dialect.h`, `Passes.h`, ... (+2 more)) provide domain-specific IR/support, MLIR headers (`ImplicitLocOpBuilder.h`, `Pass.h`, `PassManager.h`, `LLVM.h`, ... (+2 more)) provide rewriting and analysis infrastructure, LLVM headers (None) supply low-level utilities, and standard/library headers (None) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（`Utility.h`, `Utility.h`, `Dialect.h`, `Passes.h`, ... (+2 more)）提供领域特定 IR/支持逻辑，MLIR 头文件（`ImplicitLocOpBuilder.h`, `Pass.h`, `PassManager.h`, `LLVM.h`, ... (+2 more)）提供重写与分析基础设施，LLVM 头文件（None）提供底层工具，而标准/通用库头文件（None）提供通用能力。
### Lines 17-19

```cpp
  17: namespace mlir {
  18: namespace triton {
  19: namespace gpu {
```

- **EN:** Opens or closes the namespace nesting for mlir -> triton -> gpu, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 mlir -> triton -> gpu 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 21-22

```cpp
  21: #define GEN_PASS_DEF_TRITONGPUOPTIMIZETHREADLOCALITY
  22: #include "triton/Dialect/TritonGPU/Transforms/Passes.h.inc"
```

- **EN:** Pulls in generated definitions from TableGen/MLIR include fragments so the handwritten code can reuse auto-generated declarations.
- **CN:** 这里引入由 TableGen/MLIR 生成的定义片段，使手写代码能够复用自动生成的声明。
### Lines 24-30

```cpp
  24: namespace {
  25: // Change the destination layout of reshape ops allowing reorder when used by a
  26: // reduction in order to minimize the amount of cross thread communication for
  27: // the reduction.
  28: struct OptimizeReshapeLayoutPattern : public OpRewritePattern<ReshapeOp> {
  29:   OptimizeReshapeLayoutPattern(MLIRContext *context)
  30:       : OpRewritePattern<ReshapeOp>(context, 1) {}
```

- **EN:** Defines `OptimizeReshapeLayoutPattern`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `OptimizeReshapeLayoutPattern`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 32-49

```cpp
  32:   LogicalResult matchAndRewrite(ReshapeOp viewOp,
  33:                                 PatternRewriter &rewriter) const override {
  34:     if (!viewOp.getAllowReorder())
  35:       return failure();
  36:     std::optional<int> reductionAxis;
  37:     for (Operation *user : viewOp.getResult().getUsers()) {
  38:       if (auto reduceOp = dyn_cast<triton::ReduceOp>(user)) {
  39:         if (reductionAxis) {
  40:           if (reductionAxis != reduceOp.getAxis())
  41:             return failure();
  42:         } else {
  43:           reductionAxis = reduceOp.getAxis();
  44:         }
  45:       }
  46:     }
  47:     if (!reductionAxis)
  48:       return failure();
  49:     RankedTensorType tensorType = viewOp.getType();
```

- **EN:** Defines `matchAndRewrite`, the core MLIR rewrite callback. It checks whether the source operation matches the pattern and then materializes the lowered replacement.
- **CN:** 这里定义 `matchAndRewrite`，即 MLIR 模式重写的核心回调。它先检查源操作是否匹配模式，再构造降级后的替代结果。
### Lines 50-67

```cpp
  50:     if (auto blocked =
  51:             mlir::dyn_cast<BlockedEncodingAttr>(tensorType.getEncoding())) {
  52:       // If the layout already has all the elements along the reduction
  53:       // dimension in the same thread we can skip.
  54:       if (blocked.getThreadsPerWarp()[*reductionAxis] == 1 &&
  55:           blocked.getWarpsPerCTA()[*reductionAxis] == 1 &&
  56:           blocked.getCGALayout().getCTAsPerCGA()[*reductionAxis] == 1)
  57:         return failure();
  58:     }
  59:     ArrayRef<int64_t> shape = tensorType.getShape();
  60:     SmallVector<unsigned> order;
  61:     for (int i : triton::gpu::getOrder(tensorType)) {
  62:       if (i != *reductionAxis)
  63:         order.push_back(i);
  64:     }
  65:     // Make the reduction axis last so that elements won't be distributed
  66:     // amongst threads along this dimension.
  67:     order.push_back(*reductionAxis);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 68-85

```cpp
  68:     SmallVector<unsigned> sizePerThread(shape.size(), 1);
  69:     auto mod = viewOp->getParentOfType<ModuleOp>();
  70:     int numWarps = lookupNumWarps(viewOp);
  71:     int threadsPerWarp = TritonGPUDialect::getThreadsPerWarp(mod);
  72:     int numCTAs = TritonGPUDialect::getNumCTAs(mod);
  73:     auto encoding =
  74:         BlockedEncodingAttr::get(viewOp.getContext(), shape, sizePerThread,
  75:                                  order, numWarps, threadsPerWarp, numCTAs);
  76:     if (encoding == tensorType.getEncoding())
  77:       return failure();
  78:     RankedTensorType newType =
  79:         RankedTensorType::get(shape, tensorType.getElementType(), encoding);
  80:     if (triton::gpu::isExpensiveView(viewOp.getSrc().getType(), newType))
  81:       return failure();
  82:     rewriter.setInsertionPointAfter(viewOp);
  83:     rewriter.modifyOpInPlace(viewOp, [&]() {
  84:       viewOp.getResult().setType(newType);
  85:       viewOp.setEfficientLayout(true);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 86-93

```cpp
  86:     });
  87:     auto cvt = ConvertLayoutOp::create(rewriter, viewOp.getLoc(), tensorType,
  88:                                        viewOp.getResult());
  89:     rewriter.replaceAllUsesExcept(viewOp.getResult(), cvt.getResult(), cvt);
  90:     return success();
  91:   }
  92: };
  93: } // namespace
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 95-99

```cpp
  95: // This function considers a gather op in isolation and attempts to determine
  96: // whether an optimized layout can be applied to the source and index tensors.
  97: static LogicalResult setOptimizedGatherLayout(GatherOp op, RewriterBase &b) {
  98:   RankedTensorType srcType = op.getSrc().getType();
  99:   RankedTensorType idxType = op.getIndices().getType();
```

- **EN:** Defines accessor/helper `setOptimizedGatherLayout` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `setOptimizedGatherLayout`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 101-104

```cpp
 101:   // Determine a warp-local gather layout that minimizes the number of emitted
 102:   // warp shuffles.
 103:   unsigned numThreadsPerWarp = lookupThreadsPerWarp(b);
 104:   unsigned numWarps = lookupNumWarps(op);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 106-123

```cpp
 106:   // If in a gather column, each thread owns `srcSizePerThread[axis]` elements
 107:   // in the source tensor and `idxSizePerThread[axis]` elements in the index
 108:   // tensor (including broadcasting), then the number of index shuffles per
 109:   // column is `srcSizePerThread[axis] * idxSizePerThread[axis]`. This is then
 110:   // replicated over the number of columns in which a thread owns (an equal
 111:   // number of) elements, which is `product(srcSizePerThread[i] for i != axis)`.
 112:   //
 113:   // Thus, the total number of index shuffles is `product(srcSizePerThread) *
 114:   // idxSizePerThread[axis]`. Since we cannot alter the number of threads per
 115:   // warp or the number of warps, `product(srcSizePerThread)` is just a function
 116:   // of the shape.
 117:   //
 118:   // So we want to minimize `idxSizePerThread[axis]`. Note that broadcasting is
 119:   // forbidden in the source tensor but allowed in the index tensor. Choose the
 120:   // smallest value while still ensuring that a warp spans whole columns.
 121:   //
 122:   // In order to prevent broadcasting in the source tensor layout, ensure
 123:   //
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 124-129

```cpp
 124:   //   sizePerThread(i) * threadsPerWarp(i) * warpsPerCTA(i) = shape(i)
 125:   //
 126:   // For all i != axis in the source tensor. The same relationship must hold for
 127:   // the index tensor. This means we can't just set `idxSizePerThread[axis]` to
 128:   // 1 and compute the rest from that. Find the smallest value where this
 129:   // relationship is still respected.
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 131-140

```cpp
 131:   // We know that the layouts will be the same between the two tensors except
 132:   // for `sizePerThread[axis]`.
 133:   unsigned axis = op.getAxis();
 134:   unsigned rank = srcType.getRank();
 135:   if (rank == 1)
 136:     return failure();
 137:   SmallVector<unsigned> threadsPerWarp(rank);
 138:   SmallVector<unsigned> warpsPerCTA(rank);
 139:   SmallVector<unsigned> order;
 140:   order.push_back(axis);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 142-146

```cpp
 142:   // Minimize `sizePerThread[axis]` by putting as many theads along the axis as
 143:   // possible, limited to the actual size of the dimension.
 144:   unsigned maxThreadsInAxis =
 145:       std::min<unsigned>(srcType.getDimSize(axis), numThreadsPerWarp);
 146:   threadsPerWarp[axis] = maxThreadsInAxis;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 148-161

```cpp
 148:   // Now spread them along the other dimensions. Do this according to order
 149:   // (arbitrary).
 150:   unsigned threadsToAlloc = numThreadsPerWarp / maxThreadsInAxis;
 151:   for (unsigned dim : getThreadOrder(srcType)) {
 152:     if (dim == axis)
 153:       continue;
 154:     // The gather axis is now the fastest-changing dimension.
 155:     order.push_back(dim);
 156:     unsigned nextThreadAlloc =
 157:         std::min<unsigned>(srcType.getDimSize(dim), threadsToAlloc);
 158:     threadsPerWarp[dim] = nextThreadAlloc;
 159:     threadsToAlloc /= nextThreadAlloc;
 160:   }
 161:   assert(llvm::none_of(threadsPerWarp, [](unsigned c) { return c == 0; }));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 163-176

```cpp
 163:   // There must be one warp along the gather axis.
 164:   warpsPerCTA[axis] = 1;
 165:   // Allocate the remaining warps in the same manner.
 166:   unsigned warpsToAlloc = numWarps;
 167:   for (unsigned dim : getWarpOrder(srcType)) {
 168:     if (dim == axis)
 169:       continue;
 170:     unsigned warpsCanFit = srcType.getDimSize(dim) / threadsPerWarp[dim];
 171:     assert(warpsCanFit != 0);
 172:     unsigned nextWarpAlloc = std::min<unsigned>(warpsCanFit, warpsToAlloc);
 173:     warpsPerCTA[dim] = nextWarpAlloc;
 174:     warpsToAlloc /= nextWarpAlloc;
 175:   }
 176:   assert(llvm::none_of(warpsPerCTA, [](unsigned c) { return c == 0; }));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 178-182

```cpp
 178:   // Just set `sizePerThread` to 1 along other dimensions and let broadcasting
 179:   // handling it. This also means we can use the same layout between the source
 180:   // and index tensors for simplicity.
 181:   SmallVector<unsigned> sizePerThread(rank, 1);
 182:   sizePerThread[axis] = srcType.getDimSize(axis) / threadsPerWarp[axis];
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 184-187

```cpp
 184:   // Overflow by broadcasting along the gather axis since this is the most
 185:   // predictable.
 186:   threadsPerWarp[axis] *= threadsToAlloc;
 187:   warpsPerCTA[axis] *= warpsToAlloc;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 189-190

```cpp
 189:   assert(product(threadsPerWarp) == numThreadsPerWarp);
 190:   assert(product(warpsPerCTA) == numWarps);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 192-197

```cpp
 192:   // Construct the new layout.
 193:   MLIRContext *ctx = srcType.getContext();
 194:   auto baseLayout = cast<LayoutEncodingTrait>(srcType.getEncoding());
 195:   auto cgaLayout = getCGALayout(baseLayout);
 196:   auto newLayout = BlockedEncodingAttr::get(ctx, sizePerThread, threadsPerWarp,
 197:                                             warpsPerCTA, order, cgaLayout);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 199-203

```cpp
 199:   // Update the layout on the gather op and insert conversions.
 200:   auto cvtSrc = ConvertLayoutOp::create(
 201:       b, op.getLoc(), srcType.cloneWithEncoding(newLayout), op.getSrc());
 202:   auto cvtIdx = ConvertLayoutOp::create(
 203:       b, op.getLoc(), idxType.cloneWithEncoding(newLayout), op.getIndices());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 205-208

```cpp
 205:   b.setInsertionPointAfter(op);
 206:   auto cvtOut =
 207:       ConvertLayoutOp::create(b, op.getLoc(), op.getType(), op.getResult());
 208:   b.replaceAllUsesExcept(op.getResult(), cvtOut, cvtOut);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 210-213

```cpp
 210:   b.modifyOpInPlace(op, [&] {
 211:     op.getSrcMutable().set(cvtSrc);
 212:     op.getIndicesMutable().set(cvtIdx);
 213:     op.getResult().setType(op.getType().cloneWithEncoding(newLayout));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 215-217

```cpp
 215:     // Mark the layout as optimized on the op to prevent it from being changed.
 216:     op.setEfficientLayout(true);
 217:   });
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 219-220

```cpp
 219:   // Make sure we did this right.
 220:   assert(GatherLoweringHelper(op).isWarpLocal());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 222-223

```cpp
 222:   return success();
 223: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 225-227

```cpp
 225: namespace {
 226: struct OptimizeGatherLayoutPattern : public mlir::OpRewritePattern<GatherOp> {
 227:   using OpRewritePattern::OpRewritePattern;
```

- **EN:** Defines `OptimizeGatherLayoutPattern`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `OptimizeGatherLayoutPattern`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 229-236

```cpp
 229:   LogicalResult matchAndRewrite(GatherOp op,
 230:                                 PatternRewriter &rewriter) const override {
 231:     if (op.getEfficientLayout())
 232:       return failure();
 233:     return setOptimizedGatherLayout(op, rewriter);
 234:   }
 235: };
 236: } // namespace
```

- **EN:** Defines `matchAndRewrite`, the core MLIR rewrite callback. It checks whether the source operation matches the pattern and then materializes the lowered replacement.
- **CN:** 这里定义 `matchAndRewrite`，即 MLIR 模式重写的核心回调。它先检查源操作是否匹配模式，再构造降级后的替代结果。
### Lines 238-243

```cpp
 238: namespace {
 239: class TritonGPUOptimizeThreadLocalityPass
 240:     : public impl::TritonGPUOptimizeThreadLocalityBase<
 241:           TritonGPUOptimizeThreadLocalityPass> {
 242:   void runOnOperation() override {
 243:     ModuleOp mod = getOperation();
```

- **EN:** Defines `TritonGPUOptimizeThreadLocalityPass`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `TritonGPUOptimizeThreadLocalityPass`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 245-251

```cpp
 245:     // First try to optimize the layout of views and gathers.
 246:     mlir::RewritePatternSet layoutPatterns(&getContext());
 247:     layoutPatterns.add<OptimizeReshapeLayoutPattern>(&getContext());
 248:     layoutPatterns.add<OptimizeGatherLayoutPattern>(&getContext());
 249:     if (mlir::applyPatternsGreedily(mod, std::move(layoutPatterns)).failed()) {
 250:       signalPassFailure();
 251:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 253-270

```cpp
 253:     DenseSet<triton::ReduceOp> reduceOps;
 254:     mod.walk([&](triton::ReduceOp reduce) -> void {
 255:       auto srcType = cast<RankedTensorType>(reduce.getOperands()[0].getType());
 256:       auto rank = srcType.getShape().size();
 257:       auto srcEncoding = srcType.getEncoding();
 258:       auto reductionOp = getReductionOp(reduce);
 259:       if (!reductionOp ||
 260:           !isa<arith::AddFOp, arith::MulFOp, arith::MaximumFOp,
 261:                arith::MaxNumFOp, arith::MinimumFOp, arith::MinNumFOp>(
 262:               reductionOp.value()))
 263:         return;
 264:       // TODO: relax this restriction
 265:       if (!(isa<triton::gpu::BlockedEncodingAttr>(srcEncoding) && rank > 1))
 266:         return;
 267:       // The code currently assumes that the reduction is happening on the most
 268:       // inner dim.
 269:       if (reduce.getAxis() != rank - 1)
 270:         return;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. The implementation traverses IR operations to collect facts or apply rewrites globally.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。
### Lines 271-287

```cpp
 271:       for (auto operand : reduce->getOperands()) {
 272:         if (!operand.getDefiningOp<triton::LoadOp>())
 273:           return;
 274:       }
 275:       auto elemsPerThread =
 276:           triton::gpu::getElemsPerThread(srcType)[reduce.getAxis()];
 277:       // Not worth applying this optimization if there is only one element per
 278:       // thread on the reduction axis
 279:       if (elemsPerThread == 1)
 280:         return;
 281:       if (!reduce->hasOneUse())
 282:         return;
 283:       Operation *user = *(reduce->getUsers().begin());
 284:       if (!user->hasOneUse())
 285:         return;
 286:       OpOperand &yieldOpOperand = *(user->getUses().begin());
 287:       auto yieldOp = dyn_cast<scf::YieldOp>(yieldOpOperand.getOwner());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 288-301

```cpp
 288:       if (!yieldOp)
 289:         return;
 290:       Block *block = reduce->getBlock();
 291:       Operation *parentOp = block->getParentOp();
 292:       auto forOp = dyn_cast<scf::ForOp>(parentOp);
 293:       if (!forOp)
 294:         return;
 295:       auto argNum = yieldOpOperand.getOperandNumber();
 296:       auto oldAccum = forOp.getInitArgs()[argNum];
 297:       auto cstOp = oldAccum.getDefiningOp<arith::ConstantOp>();
 298:       if (!cstOp)
 299:         return;
 300:       reduceOps.insert(reduce);
 301:     });
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 303-320

```cpp
 303:     IRRewriter builder(&getContext());
 304:     for (auto reduce : reduceOps) {
 305:       builder.setInsertionPoint(reduce);
 306:       auto srcType = cast<RankedTensorType>(reduce.getOperands()[0].getType());
 307:       auto srcShape = srcType.getShape();
 308:       auto srcEncoding = srcType.getEncoding();
 309:       assert(isa<triton::gpu::BlockedEncodingAttr>(srcEncoding) &&
 310:              "Thread locality optimization only supports blocked encoding");
 311:       auto rank = srcShape.size();
 312:       // create new layouts
 313:       auto blocked3d = getThreadLocalityOptimizedEncoding(reduce);
 314:       auto viewOpTensorShape = getThreadLocalityOptimizedShape(reduce);
 315:       auto viewOpTensorType = RankedTensorType::get(
 316:           viewOpTensorShape, srcType.getElementType(), blocked3d);
 317:       auto slice2d = triton::gpu::SliceEncodingAttr::get(mod.getContext(), rank,
 318:                                                          blocked3d);
 319:       // Get forOp
 320:       assert(reduce->hasOneUse());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 321-338

```cpp
 321:       OpOperand &use = *(reduce->getUses().begin());
 322:       auto operandNumber = use.getOperandNumber();
 323:       auto oldUpdate = use.getOwner();
 324:       assert(oldUpdate->getNumOperands() == 2);
 325:       auto accumOperandNumber = (operandNumber == 0) ? 1 : 0;
 326:       auto accumOperand = oldUpdate->getOperand(accumOperandNumber);
 327:       assert(isa<BlockArgument>(accumOperand));
 328:       auto blockArg = dyn_cast<BlockArgument>(accumOperand);
 329:       auto blockArgNum = blockArg.getArgNumber();
 330:       auto forOp = dyn_cast<scf::ForOp>(blockArg.getOwner()->getParentOp());
 331:       // get oldAccum
 332:       auto oldAccum =
 333:           forOp.getInitArgs()[blockArgNum - forOp.getNumInductionVars()];
 334:       // get old loop user
 335:       Value loopResult =
 336:           forOp.getResult(blockArgNum - forOp.getNumInductionVars());
 337:       assert(loopResult.hasOneUse());
 338:       OpOperand &loopUse = *(loopResult.getUses().begin());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 339-350

```cpp
 339:       Operation *loopUser = loopUse.getOwner();
 340:       // get old loop yield
 341:       auto oldYield = cast<scf::YieldOp>(forOp.getBody()->getTerminator());
 342:       // create newAccum initialization
 343:       auto newAccum =
 344:           createAccum(builder, reduce, oldAccum, viewOpTensorShape, slice2d);
 345:       // create new loop by copying the old for op signature and appending
 346:       // newAccum to the block arguments
 347:       auto newLoop = replaceForOpWithNewSignature(
 348:           builder, forOp, ValueRange{newAccum->getResult(0)});
 349:       // create thread local reduction (also adds viewOps)
 350:       auto newReduce = createReduce(builder, reduce, viewOpTensorType);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 352-367

```cpp
 352:       // create new accum update
 353:       auto newUpdate = createUpdate(builder, newLoop, newReduce, oldUpdate);
 354:       // create new yield
 355:       createYield(builder, newLoop, oldYield, newUpdate->getResult(0),
 356:                   blockArgNum);
 357:       // create post loop reduction on the original reduce axis
 358:       auto newReduce2 = createPostLoopReduce(builder, newLoop, reduce);
 359:       // add convert_layout to get back to original layout, the result layout
 360:       // should now match the layout of the old accumulator (%cst)
 361:       Type destType = loopResult.getType();
 362:       auto cvtLayout = createConvertLayout(builder, destType, newReduce2);
 363:       // incorporate the original accumulator value into the final result
 364:       auto finalOp = incorporateOriginalAccumulatorValue(builder, oldUpdate,
 365:                                                          cvtLayout, oldAccum);
 366:       // Replace the old loop user with the final result
 367:       loopUser->setOperand(loopUse.getOperandNumber(), finalOp->getResult(0));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 369-373

```cpp
 369:       // cleanup
 370:       oldYield.erase();
 371:       forOp.erase();
 372:     }
 373:   };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 375-392

```cpp
 375: private:
 376:   std::optional<Operation *> getReductionOp(triton::ReduceOp reduce) const {
 377:     auto numRegions = reduce->getNumRegions();
 378:     if (numRegions != 1)
 379:       return std::nullopt;
 380:     Region &region = reduce->getRegion(0);
 381:     auto numBlocks = region.getBlocks().size();
 382:     if (numBlocks != 1)
 383:       return std::nullopt;
 384:     Block &block = region.front();
 385:     auto blockWithoutTerminator = block.without_terminator();
 386:     auto blockSizeWithoutTerminator = std::distance(
 387:         blockWithoutTerminator.begin(), blockWithoutTerminator.end());
 388:     if (blockSizeWithoutTerminator != 1)
 389:       return std::nullopt;
 390:     Operation *op = &block.front();
 391:     return std::optional<Operation *>(op);
 392:   }
```

- **EN:** Defines accessor/helper `getReductionOp` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getReductionOp`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 393-410

```cpp
 393:   Operation *incorporateOriginalAccumulatorValue(OpBuilder &builder,
 394:                                                  Operation *oldUpdate,
 395:                                                  Operation *cvtLayout,
 396:                                                  Value oldAccum) const {
 397:     builder.setInsertionPointAfter(cvtLayout);
 398:     IRMapping mapping;
 399:     mapping.map(oldUpdate->getOperand(0), oldAccum);
 400:     mapping.map(oldUpdate->getOperand(1), cvtLayout->getResult(0));
 401:     auto finalOp = cloneWithInferType(builder, &(*oldUpdate), mapping);
 402:     return finalOp;
 403:   }
 404:   Operation *createConvertLayout(OpBuilder &builder, Type destType,
 405:                                  Operation *newReduce) const {
 406:     builder.setInsertionPointAfter(newReduce);
 407:     auto newCvt = triton::gpu::ConvertLayoutOp::create(
 408:         builder, newReduce->getLoc(), destType, newReduce->getResult(0));
 409:     return newCvt;
 410:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 412-422

```cpp
 412:   Operation *createPostLoopReduce(OpBuilder &builder, scf::ForOp &loop,
 413:                                   triton::ReduceOp &reduce) const {
 414:     auto resultIndex =
 415:         loop.getBody()->getNumArguments() - 1 - loop.getNumInductionVars();
 416:     auto newLoopResult = loop.getResult(resultIndex);
 417:     builder.setInsertionPointAfter(loop);
 418:     IRMapping mapping;
 419:     mapping.map(*(reduce.getOperands().begin()), newLoopResult);
 420:     auto newReduce2 = cloneWithInferType(builder, &(*reduce), mapping);
 421:     return newReduce2;
 422:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 424-435

```cpp
 424:   Operation *createYield(OpBuilder &builder, scf::ForOp &loop,
 425:                          scf::YieldOp &oldYield, Value newUpdate,
 426:                          int oldAccumBlockArgNum) const {
 427:     builder.setInsertionPoint(oldYield);
 428:     SmallVector<Value> yieldValues = llvm::to_vector(oldYield.getOperands());
 429:     yieldValues[oldAccumBlockArgNum - 1] =
 430:         loop.getBody()->getArgument(oldAccumBlockArgNum);
 431:     yieldValues.push_back(newUpdate);
 432:     auto newYield =
 433:         scf::YieldOp::create(builder, oldYield.getLoc(), yieldValues);
 434:     return newYield;
 435:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 437-447

```cpp
 437:   Operation *createUpdate(OpBuilder &builder, scf::ForOp &loop,
 438:                           Operation *newReduce, Operation *oldUpdate) const {
 439:     auto blockArgNum = loop.getBody()->getNumArguments() - 1;
 440:     auto newArg = loop.getBody()->getArgument(blockArgNum);
 441:     builder.setInsertionPointAfter(newReduce);
 442:     IRMapping mapping;
 443:     mapping.map(oldUpdate->getOperand(0), newArg);
 444:     mapping.map(oldUpdate->getOperand(1), newReduce->getResult(0));
 445:     auto newUpdate = cloneWithInferType(builder, oldUpdate, mapping);
 446:     return newUpdate;
 447:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 449-460

```cpp
 449:   Operation *createReduce(OpBuilder &builder, triton::ReduceOp reduce,
 450:                           Type viewOpTensorType) const {
 451:     auto srcType = cast<RankedTensorType>(reduce.getOperands()[0].getType());
 452:     auto rank = srcType.getShape().size();
 453:     builder.setInsertionPointAfter(reduce);
 454:     IRMapping mapping;
 455:     for (auto operand : reduce.getOperands()) {
 456:       auto viewOp = triton::ReshapeOp::create(
 457:           builder, reduce.getLoc(), viewOpTensorType, operand,
 458:           /*allowReorder=*/true, /*efficientLayout=*/true);
 459:       mapping.map(operand, viewOp);
 460:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 462-477

```cpp
 462:     auto newReduce = cloneWithInferType(builder, &(*reduce), mapping);
 463:     newReduce->setAttr("axis", builder.getI32IntegerAttr(rank));
 464:     auto typeInfer = dyn_cast<InferTypeOpInterface>(newReduce);
 465:     if (typeInfer) {
 466:       SmallVector<Type, 1> newTypes;
 467:       auto success = typeInfer.inferReturnTypes(
 468:           newReduce->getContext(), newReduce->getLoc(),
 469:           newReduce->getOperands(), newReduce->getAttrDictionary(),
 470:           newReduce->getPropertiesStorage(), newReduce->getRegions(), newTypes);
 471:       if (succeeded(success)) {
 472:         for (size_t i = 0; i < newTypes.size(); i++)
 473:           newReduce->getResult(i).setType(newTypes[i]);
 474:       }
 475:     }
 476:     return newReduce;
 477:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 479-483

```cpp
 479:   // Work around the lack of support for MaxNumFOp and MinNumFOp in
 480:   // arith::getNeutralElement.
 481:   std::optional<TypedAttr> getNeutralElement(Operation *op) const {
 482:     if (isa<arith::MaxNumFOp, arith::MinNumFOp>(op)) {
 483:       OpBuilder builder(op->getContext());
```

- **EN:** Defines accessor/helper `getNeutralElement` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getNeutralElement`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 485-501

```cpp
 485:       Type resultType = op->getResult(0).getType();
 486:       const llvm::fltSemantics &semantic =
 487:           llvm::cast<FloatType>(resultType).getFloatSemantics();
 488:       if (isa<arith::MaxNumFOp>(op)) {
 489:         return builder.getFloatAttr(
 490:             resultType, APFloat::getInf(semantic, /*Negative=*/true));
 491:       }
 492:       if (isa<arith::MinNumFOp>(op)) {
 493:         return builder.getFloatAttr(
 494:             resultType, APFloat::getInf(semantic, /*Negative=*/false));
 495:       }
 496:     } else {
 497:       return mlir::arith::getNeutralElement(op);
 498:     }
 499:     llvm_unreachable("Unhandled reduction op");
 500:     return std::nullopt;
 501:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 503-520

```cpp
 503:   Operation *createAccum(OpBuilder &builder, triton::ReduceOp reduce,
 504:                          Value &oldAccum, SmallVector<int64_t> &shape,
 505:                          Attribute &slice2d) const {
 506:     // Drop the last dimension (thread locality dimension)
 507:     SmallVector<int64_t> accumShape(shape.begin(), shape.end() - 1);
 508:     auto elemType = cast<RankedTensorType>(oldAccum.getType()).getElementType();
 509:     // Create tensor type for the new accumulator
 510:     auto accumType = RankedTensorType::get(accumShape, elemType, slice2d);
 511:     // Create new accumulator
 512:     builder.setInsertionPointAfter(oldAccum.getDefiningOp());
 513:     auto reductionOp = getReductionOp(reduce);
 514:     assert(reductionOp && "Processing a reduce that is not supported!");
 515:     auto neutralVal = getNeutralElement(reductionOp.value());
 516:     assert(neutralVal && "Could not find neutral value for reduction op!");
 517:     auto denseAttr = DenseElementsAttr::get(accumType, neutralVal.value());
 518:     auto newAccum = arith::ConstantOp::create(builder, oldAccum.getLoc(),
 519:                                               accumType, denseAttr);
 520:     return newAccum;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 521-521

```cpp
 521:   }
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 523-534

```cpp
 523:   SmallVector<int64_t>
 524:   getThreadLocalityOptimizedShape(triton::ReduceOp reduce) const {
 525:     auto srcType = cast<RankedTensorType>(reduce.getOperands()[0].getType());
 526:     auto srcShape = srcType.getShape();
 527:     auto rank = srcShape.size();
 528:     auto elemsPerThread =
 529:         triton::gpu::getElemsPerThread(srcType)[reduce.getAxis()];
 530:     auto viewOpTensorShape = insertValue(srcShape, rank, 1);
 531:     viewOpTensorShape[reduce.getAxis()] /= elemsPerThread;
 532:     viewOpTensorShape[rank] = elemsPerThread;
 533:     return viewOpTensorShape;
 534:   }
```

- **EN:** Defines accessor/helper `getThreadLocalityOptimizedShape` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getThreadLocalityOptimizedShape`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 536-553

```cpp
 536:   BlockedEncodingAttr
 537:   getThreadLocalityOptimizedEncoding(triton::ReduceOp reduce) const {
 538:     auto srcType = cast<RankedTensorType>(reduce.getOperands()[0].getType());
 539:     auto rank = srcType.getShape().size();
 540:     auto srcEncoding = srcType.getEncoding();
 541:     auto blocked = dyn_cast<triton::gpu::BlockedEncodingAttr>(srcEncoding);
 542:     auto sizePerThread3d =
 543:         insertValue(blocked.getSizePerThread(), rank,
 544:                     blocked.getSizePerThread()[reduce.getAxis()]);
 545:     sizePerThread3d[reduce.getAxis()] = 1;
 546:     auto threadsPerWarp3d = insertValue(blocked.getThreadsPerWarp(), rank, 1);
 547:     auto warsPerCTA3d = insertValue(blocked.getWarpsPerCTA(), rank, 1);
 548:     auto order3d = insertValue(blocked.getOrder(), 0, rank);
 549:     auto ctaLl = blocked.getCGALayout().getLinearLayout();
 550:     auto kBlocked = *ctaLl.getInDimNames().begin();
 551:     auto *ctx = kBlocked.getContext();
 552:     auto dim = standardOutDimNames(ctx, rank + 1)[rank];
 553:     ctaLl *= LinearLayout::identity1D(1, kBlocked, dim);
```

- **EN:** Defines accessor/helper `getThreadLocalityOptimizedEncoding` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getThreadLocalityOptimizedEncoding`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 554-559

```cpp
 554:     auto ctaLayout3d = CGAEncodingAttr::get(ctx, std::move(ctaLl));
 555:     auto blocked3d = triton::gpu::BlockedEncodingAttr::get(
 556:         reduce.getContext(), sizePerThread3d, threadsPerWarp3d, warsPerCTA3d,
 557:         order3d, ctaLayout3d);
 558:     return blocked3d;
 559:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 561-575

```cpp
 561:   template <typename T>
 562:   SmallVector<T> insertValue(ArrayRef<T> vec, unsigned index, int value) const {
 563:     SmallVector<T> res(vec.begin(), vec.end());
 564:     res.insert(res.begin() + index, static_cast<T>(value));
 565:     return res;
 566:   }
 567:   template <typename T>
 568:   SmallVector<T> insertValue(const SmallVector<T> &vec, unsigned index,
 569:                              int value) const {
 570:     SmallVector<T> res(vec.begin(), vec.end());
 571:     res.insert(res.begin() + index, static_cast<T>(value));
 572:     return res;
 573:   }
 574: };
 575: } // namespace
```

- **EN:** Defines `insertValue`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `insertValue`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 577-579

```cpp
 577: } // namespace gpu
 578: } // namespace triton
 579: } // namespace mlir
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。

## Key Concepts / 关键概念
- **EN:** The main concern is pass-driven transformation around optimize thread locality.
  **CN:** 核心关注点是围绕 Optimize Thread Locality 的 pass 驱动变换。
- **EN:** Pattern rewriting is the main mechanism for canonicalization and lowering in this file.
  **CN:** 模式重写是本文件进行规范化与降级转换的核心机制。
- **EN:** Tensor shape and element-type reasoning is central to the implementation.
  **CN:** 张量形状与元素类型推导是实现中的核心内容。
- **EN:** Linear layout utilities translate between logical tensor coordinates and physical placement.
  **CN:** 线性布局工具负责在逻辑张量坐标与物理放置之间进行转换。
- **EN:** Encoding attributes describe how distributed GPU data is laid out across threads and warps.
  **CN:** 编码属性描述 GPU 分布式数据在线程与 warp 之间的布局方式。
- **EN:** The code reasons at module scope rather than only on isolated operations.
  **CN:** 代码在模块范围内进行分析，而不只是处理单个操作。

## Dependencies / 依赖关系
- **Internal Triton headers / Triton 内部头文件:** `triton/Analysis/Utility.h`, `triton/Dialect/Triton/IR/Utility.h`, `triton/Dialect/TritonGPU/IR/Dialect.h`, `triton/Dialect/TritonGPU/Transforms/Passes.h`, `triton/Dialect/TritonGPU/Transforms/Utility.h`, `triton/Tools/LayoutUtils.h`, ... (+1 more)
- **MLIR headers / MLIR 头文件:** `mlir/IR/ImplicitLocOpBuilder.h`, `mlir/Pass/Pass.h`, `mlir/Pass/PassManager.h`, `mlir/Support/LLVM.h`, `mlir/Transforms/GreedyPatternRewriteDriver.h`, `mlir/Transforms/Passes.h`
- **LLVM headers / LLVM 头文件:** None
- **Standard/library headers / 标准或通用库头文件:** `memory`, `numeric`
- **Generated fragments / 生成片段:** `triton/Dialect/TritonGPU/Transforms/Passes.h.inc`
- **Primary APIs used / 主要 API:** `PatternRewriter`, `ModuleOp`, `RankedTensorType`, `LinearLayout`, `OpBuilder`
- **Pipeline role / 流程角色:** This file participates in Triton pass pipelines and usually expects upstream analyses or dialect invariants to have prepared the IR. / 本文件参与 Triton 的 pass 流程，通常依赖上游分析或方言不变量先把 IR 准备好。
