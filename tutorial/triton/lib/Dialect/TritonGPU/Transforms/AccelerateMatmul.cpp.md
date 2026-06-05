# AccelerateMatmul.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `lib/Dialect/TritonGPU/Transforms/AccelerateMatmul.cpp`
- **Purpose / 作用:** **EN:** Implements the Accelerate Matmul transformation or optimization pass for the TritonGPU pipeline. **CN:** 为 TritonGPU 编译流程实现与 Accelerate Matmul 相关的变换或优化 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```cpp
   1: #include "mlir/Analysis/SliceAnalysis.h"
   2: #include "mlir/IR/BuiltinAttributes.h"
   3: #include "mlir/IR/BuiltinTypes.h"
   4: #include "mlir/IR/TypeUtilities.h"
   5: #include "mlir/IR/Types.h"
   6: #include "mlir/IR/Value.h"
   7: #include "mlir/Support/LLVM.h"
   8: #include "mlir/Support/LogicalResult.h"
   9: #include "mlir/Transforms/GreedyPatternRewriteDriver.h"
  10: #include "triton/Analysis/Utility.h"
  11: #include "triton/Conversion/MLIRTypes.h"
  12: #include "triton/Dialect/Triton/IR/Dialect.h"
  13: #include "triton/Dialect/Triton/IR/OpInterfaces.h"
  14: #include "triton/Dialect/Triton/IR/Utility.h"
  15: #include "triton/Dialect/TritonGPU/IR/Attributes.h"
  16: #include "triton/Dialect/TritonGPU/IR/Dialect.h"
  17: #include "triton/Dialect/TritonGPU/IR/LinearLayoutConversions.h"
  18: #include "triton/Dialect/TritonGPU/Transforms/DecomposeScaledBlocked.h"
```

- **EN:** Includes the interfaces this file depends on. Triton headers (`Utility.h`, `MLIRTypes.h`, `Dialect.h`, `OpInterfaces.h`, ... (+5 more)) provide domain-specific IR/support, MLIR headers (`SliceAnalysis.h`, `BuiltinAttributes.h`, `BuiltinTypes.h`, `TypeUtilities.h`, ... (+5 more)) provide rewriting and analysis infrastructure, LLVM headers (None) supply low-level utilities, and standard/library headers (None) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（`Utility.h`, `MLIRTypes.h`, `Dialect.h`, `OpInterfaces.h`, ... (+5 more)）提供领域特定 IR/支持逻辑，MLIR 头文件（`SliceAnalysis.h`, `BuiltinAttributes.h`, `BuiltinTypes.h`, `TypeUtilities.h`, ... (+5 more)）提供重写与分析基础设施，LLVM 头文件（None）提供底层工具，而标准/通用库头文件（None）提供通用能力。
### Lines 19-25

```cpp
  19: #include "triton/Dialect/TritonGPU/Transforms/Passes.h"
  20: #include "triton/Dialect/TritonGPU/Transforms/Utility.h"
  21: #include "triton/Tools/LayoutUtils.h"
  22: #include "triton/Tools/StrUtil.h"
  23: #include "llvm/ADT/ArrayRef.h"
  24: #include "llvm/ADT/SmallVector.h"
  25: #include "llvm/Support/Casting.h"
```

- **EN:** Includes the interfaces this file depends on. Triton headers (`Passes.h`, `Utility.h`, `LayoutUtils.h`, `StrUtil.h`) provide domain-specific IR/support, MLIR headers (None) provide rewriting and analysis infrastructure, LLVM headers (`ArrayRef.h`, `SmallVector.h`, `Casting.h`) supply low-level utilities, and standard/library headers (None) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（`Passes.h`, `Utility.h`, `LayoutUtils.h`, `StrUtil.h`）提供领域特定 IR/支持逻辑，MLIR 头文件（None）提供重写与分析基础设施，LLVM 头文件（`ArrayRef.h`, `SmallVector.h`, `Casting.h`）提供底层工具，而标准/通用库头文件（None）提供通用能力。
### Lines 27-29

```cpp
  27: namespace mlir {
  28: namespace triton {
  29: namespace gpu {
```

- **EN:** Opens or closes the namespace nesting for mlir -> triton -> gpu, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 mlir -> triton -> gpu 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 31-31

```cpp
  31: namespace {
```

- **EN:** Opens or closes the namespace nesting for (anonymous), keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 (anonymous) 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 33-39

```cpp
  33: static bool isUnsupportedMMAv5Int8Dot(int computeCapability, DotOp op) {
  34:   if (computeCapability != 103)
  35:     return false;
  36:   auto aElemTy = op.getA().getType().getElementType();
  37:   auto bElemTy = op.getB().getType().getElementType();
  38:   return aElemTy.isInteger(8) && bElemTy.isInteger(8);
  39: }
```

- **EN:** Defines `isUnsupportedMMAv5Int8Dot`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `isUnsupportedMMAv5Int8Dot`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 41-57

```cpp
  41: // Get the highest version supported for the hardware and the dot.
  42: static int getMMAVersionSafe(int computeCapability, DotOp op) {
  43:   // List supported mma version in order of preference.
  44:   SmallVector<int> versionsSupported;
  45:   if (computeCapability < 75) {
  46:     versionsSupported = {1};
  47:   } else if (computeCapability < 90) {
  48:     versionsSupported = {2};
  49:   } else if (computeCapability < 100) {
  50:     versionsSupported = {3, 2};
  51:   } else if (computeCapability < 120) {
  52:     // Exclude consumer Blackwell (sm120)
  53:     if (isUnsupportedMMAv5Int8Dot(computeCapability, op)) {
  54:       versionsSupported = {2};
  55:     } else {
  56:       versionsSupported = {5, 2};
  57:     }
```

- **EN:** Defines accessor/helper `getMMAVersionSafe` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getMMAVersionSafe`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 58-72

```cpp
  58:   } else if (computeCapability < 130) {
  59:     versionsSupported = {2};
  60:   } else {
  61:     assert(false && "computeCapability not supported");
  62:   }
  63:   for (int baseVersion : versionsSupported) {
  64:     if (supportMMA(op, baseVersion))
  65:       return baseVersion;
  66:     if (baseVersion == 3) {
  67:       auto remark = op.emitRemark()
  68:                     << "MMA version 3 acceleration not applied due to "
  69:                        "unsupported shapes or data types.";
  70:       remark.attachNote() << "Target compute capability (" << computeCapability
  71:                           << ") supports MMA v3.";
  72:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 74-83

```cpp
  74:     if (baseVersion == 5) {
  75:       auto remark = op.emitRemark()
  76:                     << "MMA version 5 acceleration not applied due to "
  77:                        "unsupported shapes or data types.";
  78:       remark.attachNote() << "Target compute capability (" << computeCapability
  79:                           << ") supports MMA v5.";
  80:     }
  81:   }
  82:   return 0;
  83: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 85-91

```cpp
  85: SmallVector<unsigned> warpsPerTileV2(DotOpInterface dotOp,
  86:                                      const ArrayRef<int64_t> shape,
  87:                                      int numWarps) {
  88:   auto rank = shape.size();
  89:   // Early exit for batched matmul
  90:   if (rank == 3)
  91:     return {(unsigned)numWarps, 1, 1};
```

- **EN:** Defines `warpsPerTileV2`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `warpsPerTileV2`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 93-110

```cpp
  93:   auto filter = [&dotOp](Operation *op) {
  94:     return op->getParentRegion() == dotOp->getParentRegion() &&
  95:            !isa<TransOp>(op);
  96:   };
  97:   auto slices = mlir::getSlice(dotOp, {filter}, {filter});
  98:   bool hasChainedDot = false;
  99:   for (Operation *op : slices) {
 100:     if (isa<DotOp, DotScaledOp>(op) && (op != dotOp)) {
 101:       auto resTy = cast<RankedTensorType>(op->getResult(0).getType());
 102:       if (resTy.getRank() != rank) {
 103:         continue;
 104:       }
 105:       if (auto mmaEncoding =
 106:               dyn_cast<NvidiaMmaEncodingAttr>(resTy.getEncoding())) {
 107:         return to_vector(mmaEncoding.getWarpsPerCTA());
 108:       }
 109:       hasChainedDot = true;
 110:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 111-118

```cpp
 111:   }
 112:   if (hasChainedDot) {
 113:     if (shape[0] >= shape[1]) {
 114:       return {(unsigned)numWarps, 1};
 115:     } else {
 116:       return {1, (unsigned)numWarps};
 117:     }
 118:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 120-137

```cpp
 120:   assert(rank == 2);
 121:   SmallVector<int64_t> shapePerWarp = {16, 8};
 122:   SmallVector<int64_t> warps = {1, 1};
 123:   // Compute repM and repN
 124:   SmallVector<int64_t> reps = {ceil(shape[0], shapePerWarp[0]),
 125:                                ceil(shape[1], shapePerWarp[1])};
 126:   // The formula for the number of registers given the reps is
 127:   // repM * 4 * repK + repN * 2 * repK + regsC
 128:   // where regsC = repM * repN * 4, which does not depend on the warp shape
 129:   //
 130:   // As such, to minimize the register pressure, we need to balance
 131:   // repM and repN. We then untie towards M, as the lhs tile has 4 elements,
 132:   // and the rhs tile has just 2.
 133:   while (product(warps) < numWarps) {
 134:     if (reps[0] >= reps[1]) {
 135:       warps[0] *= 2;
 136:       // Too many warps for this mma (repM == repN == 1).
 137:       // We allocate the remaining warps to the left (arbitrary choice)
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 138-155

```cpp
 138:       if (reps[0] != 1) {
 139:         reps[0] /= 2;
 140:       }
 141:     } else {
 142:       warps[1] *= 2;
 143:       reps[1] /= 2;
 144:     }
 145:   }
 146:   return {(unsigned)warps[0], (unsigned)warps[1]};
 147: }
 148: SmallVector<unsigned, 2>
 149: warpsPerTileV3(DotOpInterface dotOp, const ArrayRef<int64_t> shape,
 150:                int numWarps, const SmallVector<unsigned, 3> &instrShape) {
 151:   SetVector<Operation *> slices;
 152:   mlir::getForwardSlice(dotOp.getD(), &slices);
 153:   // Contains a chained dot. We prefer to assign warps to one axis
 154:   // to facilitate use cases like flash attention, allowing reductions within
 155:   // the same warp.
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 156-159

```cpp
 156:   if (llvm::find_if(slices, [](Operation *op) {
 157:         return isa<mlir::triton::DotOpInterface>(op);
 158:       }) != slices.end())
 159:     return {(unsigned)numWarps, 1};
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 161-174

```cpp
 161:   // For MMAv3, the smallest indivisible unit of warp shape is (4, 1).
 162:   SmallVector<unsigned, 2> ret = {4, 1};
 163:   SmallVector<int64_t, 2> shapePerWarp = {16, instrShape[1]};
 164:   do {
 165:     if (ret[0] * ret[1] >= numWarps)
 166:       break;
 167:     if (shape[0] > shapePerWarp[0] * ret[0]) {
 168:       ret[0] *= 2;
 169:     } else {
 170:       ret[1] *= 2;
 171:     }
 172:   } while (true);
 173:   return ret;
 174: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 176-189

```cpp
 176: // Returns a shared memory allocation that can be used by a dotMMA op for the
 177: // given value.
 178: static Value
 179: getSharedMemoryMMAOperand(Value v, mlir::PatternRewriter &rewriter, int opIdx,
 180:                           bool allowTranspose, bool isMMAv5Fp4Padded = false,
 181:                           bool forceTranspose = false,
 182:                           Operation *op = nullptr /*only for diagnostic*/) {
 183:   OpBuilder::InsertionGuard g(rewriter);
 184:   Value arg = v;
 185:   while (auto cvtOp = arg.getDefiningOp<ConvertLayoutOp>())
 186:     arg = cvtOp.getSrc();
 187:   auto argType = cast<RankedTensorType>(arg.getType());
 188:   assert(argType.getEncoding() && "unexpected tensor type");
 189:   auto order = getOrderForMemory(argType);
```

- **EN:** Defines accessor/helper `getSharedMemoryMMAOperand` that exposes or updates operation state in a compact, reusable way. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义访问器/辅助函数 `getSharedMemoryMMAOperand`，以紧凑且可复用的方式读取或更新操作状态。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 191-202

```cpp
 191:   // If the MMA op doesn't support transpose pick the layout expected by the MMA
 192:   // op.
 193:   llvm::SmallVector<unsigned> newOrder = order;
 194:   if (!allowTranspose) {
 195:     if (opIdx == 1) {
 196:       newOrder = {0, 1};
 197:     } else {
 198:       newOrder = {1, 0};
 199:     }
 200:     if (forceTranspose)
 201:       std::swap(newOrder[0], newOrder[1]);
 202:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 204-213

```cpp
 204:   if (newOrder != order && op) {
 205:     op->emitWarning("Warning: Forcing a different order [")
 206:         << newOrder[0] << ", " << newOrder[1]
 207:         << "] on SMEM than the register order for the operand " << opIdx
 208:         << ". Registers will be transposed before SMEM store and the pipelined "
 209:            "load for this operand will be disabled, so poor performance is "
 210:            "expected. Recommendation: consider transposing the operand in "
 211:            "global "
 212:            "memory to remove the need to transpose the tensor in registers.";
 213:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 215-225

```cpp
 215:   Attribute SharedMemorySpace =
 216:       SharedMemorySpaceAttr::get(argType.getContext());
 217:   auto CGALayout = getCGALayout(argType.getEncoding());
 218:   auto newLayout = NVMMASharedEncodingAttr::get(
 219:       argType.getContext(), argType.getShape(), newOrder, CGALayout,
 220:       argType.getElementType(), isMMAv5Fp4Padded);
 221:   auto newType = MemDescType::get(argType.getShape(), argType.getElementType(),
 222:                                   newLayout, SharedMemorySpace);
 223:   rewriter.setInsertionPointAfterValue(arg);
 224:   return LocalAllocOp::create(rewriter, arg.getLoc(), newType, arg);
 225: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。
### Lines 227-232

```cpp
 227: static LocalAllocOp
 228: getSharedMemoryScale(Value arg, mlir::PatternRewriter &rewriter, Location loc) {
 229:   OpBuilder::InsertionGuard g(rewriter);
 230:   auto argType = cast<RankedTensorType>(arg.getType());
 231:   assert(argType.getEncoding() && "unexpected tensor type");
 232:   auto newOrder = getOrderForMemory(argType);
```

- **EN:** Defines accessor/helper `getSharedMemoryScale` that exposes or updates operation state in a compact, reusable way. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义访问器/辅助函数 `getSharedMemoryScale`，以紧凑且可复用的方式读取或更新操作状态。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 234-247

```cpp
 234:   Attribute SharedMemorySpace =
 235:       SharedMemorySpaceAttr::get(argType.getContext());
 236:   auto CGALayout = getCGALayout(argType.getEncoding());
 237:   // No swizzling for scale for now
 238:   auto newLayout = NVMMASharedEncodingAttr::get(
 239:       argType.getContext(), /*swizzlingByteWidth=*/0,
 240:       /*transposed=*/false,
 241:       /*elementBitWidth=*/argType.getElementType().getIntOrFloatBitWidth(),
 242:       /*fp4Padded=*/false, CGALayout);
 243:   auto newType = MemDescType::get(argType.getShape(), argType.getElementType(),
 244:                                   newLayout, SharedMemorySpace);
 245:   rewriter.setInsertionPointAfterValue(arg);
 246:   return LocalAllocOp::create(rewriter, loc, newType, arg);
 247: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。
### Lines 249-262

```cpp
 249: SmallVector<unsigned, 3>
 250: getWarpsPerTile(DotOpInterface dotOp, const ArrayRef<int64_t> shape,
 251:                 int version, int numWarps,
 252:                 const SmallVector<unsigned, 3> &instrShape) {
 253:   switch (version) {
 254:   case 2:
 255:     return warpsPerTileV2(dotOp, shape, numWarps);
 256:   case 3:
 257:     return warpsPerTileV3(dotOp, shape, numWarps, instrShape);
 258:   default:
 259:     assert(false && "not supported version");
 260:     return {0, 0};
 261:   }
 262: }
```

- **EN:** Defines accessor/helper `getWarpsPerTile` that exposes or updates operation state in a compact, reusable way. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义访问器/辅助函数 `getWarpsPerTile`，以紧凑且可复用的方式读取或更新操作状态。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 264-269

```cpp
 264: static bool bwdFilter(Operation *op) {
 265:   return (op->hasTrait<OpTrait::Elementwise>() && isMemoryEffectFree(op)) ||
 266:          isView(op) ||
 267:          isa<Fp4ToFpOp, LoadOp, DescriptorLoadLikeOpInterface, BroadcastOp,
 268:              ConvertLayoutOp>(op);
 269: }
```

- **EN:** Defines `bwdFilter`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `bwdFilter`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 271-277

```cpp
 271: // Finds the bitwidth with which the value x is loaded
 272: static int computeOrigBitWidth(Value x) {
 273:   SetVector<Operation *> slice;
 274:   mlir::BackwardSliceOptions opt;
 275:   opt.omitBlockArguments = true;
 276:   opt.filter = bwdFilter;
 277:   (void)getBackwardSlice(x, &slice, opt);
```

- **EN:** Defines helper `computeOrigBitWidth` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `computeOrigBitWidth`，用于计算或构造外围变换所需的中间数据。
### Lines 279-283

```cpp
 279:   // TODO: This heuristic may be a bit too coarse and may need improving
 280:   // If the chain contains a fp4 to fp16/bf16 conversion, then the original
 281:   // bitwidth is 4.
 282:   if (llvm::any_of(slice, [](Operation *op) { return isa<Fp4ToFpOp>(op); }))
 283:     return 4;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 285-294

```cpp
 285:   int origBitWidth = getElementTypeOrSelf(x).getIntOrFloatBitWidth();
 286:   for (auto op : slice) {
 287:     if (isa<LoadOp, DescriptorLoadLikeOpInterface>(op)) {
 288:       if (auto tensorTy =
 289:               dyn_cast<RankedTensorType>(op->getResultTypes().front())) {
 290:         origBitWidth =
 291:             std::min<int>(origBitWidth, tensorTy.getElementTypeBitWidth());
 292:       }
 293:     }
 294:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 296-307

```cpp
 296:   // If JoinOp occurred at least once, in backward layout propagation,
 297:   // the kWidth will be split in half as we pass through the JoinOp.
 298:   // Hence we divide origBitWidth by 2 here to compensate for that and
 299:   // improve our load width.
 300:   // This won't be optimal if there is a tree of multiple JoinOps, which
 301:   // would require counting the max number of JoinOp's along any path.
 302:   //
 303:   // In the future we might want to do something like trying a large kWidth,
 304:   // run layout backpropagation and see what's the contiguity that you
 305:   // get at the loads that feed into it.
 306:   if (llvm::any_of(slice, [](Operation *op) { return isa<JoinOp>(op); }))
 307:     origBitWidth /= 2;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 309-310

```cpp
 309:   return origBitWidth;
 310: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 312-312

```cpp
 312: namespace {
```

- **EN:** Opens or closes the namespace nesting for (anonymous), keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 (anonymous) 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 314-321

```cpp
 314: // Common MMA encoding creation
 315: struct MMAEncodingResult {
 316:   NvidiaMmaEncodingAttr mmaEnc;
 317:   RankedTensorType newRetType;
 318:   Value newAcc;
 319:   int versionMajor;
 320:   int versionMinor;
 321: };
```

- **EN:** Defines `MMAEncodingResult`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `MMAEncodingResult`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 323-329

```cpp
 323: // Unified implementation for DotOpInterface
 324: static MMAEncodingResult createMMAEncodingForDot(DotOpInterface dotOp,
 325:                                                  PatternRewriter &rewriter,
 326:                                                  int computeCapability,
 327:                                                  int versionMajor) {
 328:   auto oldRetType = cast<RankedTensorType>(dotOp.getD().getType());
 329:   auto oldAType = cast<RankedTensorType>(dotOp.getA().getType());
```

- **EN:** Defines helper `createMMAEncodingForDot` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createMMAEncodingForDot`，用于计算或构造外围变换所需的中间数据。
### Lines 331-331

```cpp
 331:   int numWarps = lookupNumWarps(dotOp);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 333-337

```cpp
 333:   int versionMinor = computeCapability == 75 ? 1 : 0;
 334:   // Only MMAv2 and MMAv3 rely on computing instrShape/warpsPerTile here.
 335:   if (!(versionMajor == 2 || versionMajor == 3)) {
 336:     return {nullptr, RankedTensorType(), Value(), versionMajor, versionMinor};
 337:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 339-344

```cpp
 339:   auto CGALayout = getCGALayout(oldRetType.getEncoding());
 340:   auto retShapePerCTA = getShapePerCTA(oldRetType);
 341:   auto instrShape = mmaVersionToInstrShape(versionMajor, retShapePerCTA,
 342:                                            oldAType.getElementType(), numWarps);
 343:   auto warpsPerTile = getWarpsPerTile(dotOp, retShapePerCTA, versionMajor,
 344:                                       numWarps, instrShape);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 346-349

```cpp
 346:   auto mmaEnc = NvidiaMmaEncodingAttr::get(oldRetType.getContext(),
 347:                                            versionMajor, versionMinor,
 348:                                            warpsPerTile, CGALayout, instrShape);
 349:   auto newRetType = oldRetType.cloneWithEncoding(mmaEnc);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 351-353

```cpp
 351:   auto oldAcc = dotOp->getOperand(2);
 352:   auto newAcc =
 353:       ConvertLayoutOp::create(rewriter, oldAcc.getLoc(), newRetType, oldAcc);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 355-356

```cpp
 355:   return {mmaEnc, newRetType, newAcc, versionMajor, versionMinor};
 356: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 358-368

```cpp
 358: // Common operand conversion
 359: static Value convertDotOperandForMMA(Value v, int opIdx, int bitwidth,
 360:                                      RankedTensorType newRetType,
 361:                                      PatternRewriter &rewriter) {
 362:   auto minType = bitwidth > 0 ? rewriter.getIntegerType(bitwidth) : v.getType();
 363:   auto vType = cast<RankedTensorType>(v.getType());
 364:   auto newVEncoding = DotOperandEncodingAttr::get(
 365:       v.getContext(), opIdx, newRetType.getEncoding(), minType);
 366:   auto newVType = vType.cloneWithEncoding(newVEncoding);
 367:   return ConvertLayoutOp::create(rewriter, v.getLoc(), newVType, v);
 368: }
```

- **EN:** Defines helper `convertDotOperandForMMA` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `convertDotOperandForMMA`，用于计算或构造外围变换所需的中间数据。
### Lines 370-370

```cpp
 370: } // namespace
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 372-374

```cpp
 372: class BlockedToMMA : public mlir::OpRewritePattern<DotOp> {
 373:   int computeCapability;
 374:   mutable llvm::DenseMap<Operation *, unsigned> dotOpInstNs;
```

- **EN:** Defines `BlockedToMMA`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `BlockedToMMA`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 376-379

```cpp
 376: public:
 377:   BlockedToMMA(mlir::MLIRContext *context, int computeCapability, int benefit)
 378:       : OpRewritePattern<DotOp>(context, benefit),
 379:         computeCapability(computeCapability) {}
```

- **EN:** Defines `BlockedToMMA`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `BlockedToMMA`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 381-396

```cpp
 381:   mlir::LogicalResult
 382:   matchAndRewrite(triton::DotOp dotOp,
 383:                   mlir::PatternRewriter &rewriter) const override {
 384:     if (computeCapability < 70)
 385:       return failure();
 386:     if (computeCapability < 80) {
 387:       dotOp.emitRemark()
 388:           << "Dot op using MMA for compute capability " << computeCapability
 389:           << " has been deprecated. It falls back to the FMA path.";
 390:       return failure();
 391:     }
 392:     // TODO: Check data-types and SM compatibility
 393:     auto retType = dotOp.getType();
 394:     if (!retType.getEncoding() ||
 395:         mlir::isa<NvidiaMmaEncodingAttr>(retType.getEncoding()))
 396:       return failure();
```

- **EN:** Defines `matchAndRewrite`, the core MLIR rewrite callback. It checks whether the source operation matches the pattern and then materializes the lowered replacement.
- **CN:** 这里定义 `matchAndRewrite`，即 MLIR 模式重写的核心回调。它先检查源操作是否匹配模式，再构造降级后的替代结果。
### Lines 398-402

```cpp
 398:     Value a = dotOp.getA();
 399:     Value b = dotOp.getB();
 400:     auto oldAType = cast<RankedTensorType>(a.getType());
 401:     auto oldBType = cast<RankedTensorType>(b.getType());
 402:     auto oldRetType = cast<RankedTensorType>(dotOp.getType());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 404-411

```cpp
 404:     // Enable F64 MMA only on SM80/SM90 with high performance F64 tensorcore.
 405:     // Otherwise, fallback to F64 FMA for better performance.
 406:     if ((oldAType.getElementType().isF64() ||
 407:          oldBType.getElementType().isF64() ||
 408:          oldRetType.getElementType().isF64()) &&
 409:         !(computeCapability == 80 || computeCapability == 90)) {
 410:       return failure();
 411:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 413-417

```cpp
 413:     auto mmaVersion = getMMAVersionSafe(computeCapability, dotOp);
 414:     auto mmaResult =
 415:         createMMAEncodingForDot(dotOp, rewriter, computeCapability, mmaVersion);
 416:     if (!(mmaResult.versionMajor >= 1 && mmaResult.versionMajor <= 3))
 417:       return failure();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 419-420

```cpp
 419:     Operation *newDot = nullptr;
 420:     bool aFromLoad = comesFromLoadOrBlockArg(a);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 422-436

```cpp
 422:     if (mmaResult.versionMajor == 3) {
 423:       auto eltType = cast<RankedTensorType>(a.getType()).getElementType();
 424:       bool allowTranspose = eltType.isF16() || eltType.isBF16();
 425:       if (!aFromLoad) {
 426:         int bitwidth = getElementTypeOrSelf(a).getIntOrFloatBitWidth();
 427:         a = convertDotOperandForMMA(a, 0, bitwidth, mmaResult.newRetType,
 428:                                     rewriter);
 429:       } else {
 430:         a = getSharedMemoryMMAOperand(a, rewriter, 0, allowTranspose,
 431:                                       /*isMMAv5Fp4Padded=*/false,
 432:                                       /*forceTranspose=*/false, dotOp);
 433:       }
 434:       b = getSharedMemoryMMAOperand(b, rewriter, 1, allowTranspose,
 435:                                     /*isMMAv5Fp4Padded=*/false,
 436:                                     /*forceTranspose=*/false, dotOp);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 438-452

```cpp
 438:       newDot = triton::nvidia_gpu::WarpGroupDotOp::create(
 439:           rewriter, dotOp.getLoc(), mmaResult.newRetType, a, b,
 440:           mmaResult.newAcc, nullptr, dotOp.getInputPrecision(),
 441:           dotOp.getMaxNumImpreciseAcc(), false);
 442:     } else {
 443:       int minBitwidth =
 444:           std::min(computeOrigBitWidth(a), computeOrigBitWidth(b));
 445:       a = convertDotOperandForMMA(a, 0, minBitwidth, mmaResult.newRetType,
 446:                                   rewriter);
 447:       b = convertDotOperandForMMA(b, 1, minBitwidth, mmaResult.newRetType,
 448:                                   rewriter);
 449:       newDot = DotOp::create(rewriter, dotOp.getLoc(), mmaResult.newRetType, a,
 450:                              b, mmaResult.newAcc, dotOp.getInputPrecision(),
 451:                              dotOp.getMaxNumImpreciseAcc());
 452:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 454-458

```cpp
 454:     rewriter.replaceOpWithNewOp<ConvertLayoutOp>(dotOp, dotOp.getType(),
 455:                                                  newDot->getResult(0));
 456:     return success();
 457:   }
 458: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. It finishes by replacing the original operation with the lowered form.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 它会在转换成功后用降级后的形式替换原始操作。
### Lines 460-476

```cpp
 460: static DistributedEncodingTrait
 461: replaceCGALayout(DistributedEncodingTrait layout,
 462:                  const triton::gpu::CGAEncodingAttr &newCGALayout) {
 463:   if (auto blockedLayout = mlir::dyn_cast<BlockedEncodingAttr>(layout)) {
 464:     return BlockedEncodingAttr::get(
 465:         layout.getContext(), blockedLayout.getSizePerThread(),
 466:         blockedLayout.getThreadsPerWarp(), blockedLayout.getWarpsPerCTA(),
 467:         blockedLayout.getOrder(), newCGALayout);
 468:   } else if (auto sliceLayout = mlir::dyn_cast<SliceEncodingAttr>(layout)) {
 469:     return SliceEncodingAttr::get(
 470:         layout.getContext(), sliceLayout.getDim(),
 471:         replaceCGALayout(sliceLayout.getParent(), newCGALayout));
 472:   } else {
 473:     llvm::report_fatal_error("not implemented");
 474:     return layout;
 475:   }
 476: }
```

- **EN:** Defines `replaceCGALayout`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `replaceCGALayout`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 478-495

```cpp
 478: static Value splitBOperand(Value b, mlir::PatternRewriter &rewriter) {
 479:   OpBuilder::InsertionGuard g(rewriter);
 480:   MLIRContext *ctx = b.getContext();
 481:   while (auto cvtOp = b.getDefiningOp<ConvertLayoutOp>())
 482:     b = cvtOp.getSrc();
 483:   auto loadOp = b.getDefiningOp();
 484:   assert((isa<triton::LoadOp, triton::DescriptorLoadLikeOpInterface>(loadOp)) &&
 485:          "expected LoadOp");
 486:   RankedTensorType bType = cast<RankedTensorType>(b.getType());
 487:   auto currentLayout = cast<DistributedEncodingTrait>(bType.getEncoding());
 488:   auto kBlock = StringAttr::get(ctx, "block");
 489:   auto dims = standardOutDimNames(ctx, 2);
 490:   auto newCGALayout =
 491:       CGAEncodingAttr::get(ctx, LinearLayout({{kBlock, {{0, 1}}}}, dims));
 492:   Attribute newLayout = replaceCGALayout(currentLayout, newCGALayout);
 493:   rewriter.setInsertionPoint(loadOp);
 494:   for (OpOperand &operand : loadOp->getOpOperands()) {
 495:     auto tensorType = dyn_cast<RankedTensorType>(operand.get().getType());
```

- **EN:** Defines `splitBOperand`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `splitBOperand`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 496-509

```cpp
 496:     if (!tensorType)
 497:       continue;
 498:     Value newOperand = ConvertLayoutOp::create(
 499:         rewriter, operand.get().getLoc(),
 500:         tensorType.cloneWithEncoding(newLayout), operand.get());
 501:     loadOp->setOperand(operand.getOperandNumber(), newOperand);
 502:   }
 503:   loadOp->getResult(0).setType(bType.cloneWithEncoding(newLayout));
 504:   Value newB = loadOp->getResult(0);
 505:   rewriter.setInsertionPointAfter(loadOp);
 506:   auto cvt = ConvertLayoutOp::create(rewriter, b.getLoc(), bType, newB);
 507:   rewriter.replaceAllUsesExcept(newB, cvt.getResult(), cvt);
 508:   return newB;
 509: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 511-512

```cpp
 511: class BlockedToMMAv5 : public mlir::OpRewritePattern<DotOp> {
 512:   int computeCapability;
```

- **EN:** Defines `BlockedToMMAv5`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `BlockedToMMAv5`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 514-517

```cpp
 514: public:
 515:   BlockedToMMAv5(mlir::MLIRContext *context, int computeCapability, int benefit)
 516:       : OpRewritePattern<DotOp>(context, benefit),
 517:         computeCapability(computeCapability) {}
```

- **EN:** Defines `BlockedToMMAv5`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `BlockedToMMAv5`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 519-525

```cpp
 519:   mlir::LogicalResult
 520:   matchAndRewrite(triton::DotOp dotOp,
 521:                   mlir::PatternRewriter &rewriter) const override {
 522:     RankedTensorType oldRetType = dotOp.getType();
 523:     if (!oldRetType.getEncoding() ||
 524:         mlir::isa<NvidiaMmaEncodingAttr>(oldRetType.getEncoding()))
 525:       return failure();
```

- **EN:** Defines `matchAndRewrite`, the core MLIR rewrite callback. It checks whether the source operation matches the pattern and then materializes the lowered replacement.
- **CN:** 这里定义 `matchAndRewrite`，即 MLIR 模式重写的核心回调。它先检查源操作是否匹配模式，再构造降级后的替代结果。
### Lines 527-530

```cpp
 527:     // get MMA encoding for the given number of warps
 528:     auto retShapePerCTA = getShapePerCTA(oldRetType);
 529:     int numWarps = lookupNumWarps(dotOp);
 530:     auto CGALayout = getCGALayout(oldRetType.getEncoding());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 532-549

```cpp
 532:     int versionMajor = getMMAVersionSafe(computeCapability, dotOp);
 533:     if (versionMajor != 5)
 534:       return failure();
 535:     Location loc = dotOp.getLoc();
 536:     // operands
 537:     Value a = dotOp.getA();
 538:     Value b = dotOp.getB();
 539:     if (std::min(computeOrigBitWidth(a), computeOrigBitWidth(b)) >= 32 &&
 540:         dotOp.getInputPrecision() != InputPrecision::TF32)
 541:       return failure();
 542:     auto oldAType = dotOp.getA().getType();
 543:     // NYI: PTX 13+ requires all tcgen instructions in a kernel to have a
 544:     // consistent CTA mode, disabling 2CTA mode for now. To re-enable,
 545:     // change the line below to: bool useTwoCTAs = canUseTwoCTAs(dotOp);
 546:     bool useTwoCTAs = false;
 547:     if (useTwoCTAs) {
 548:       b = splitBOperand(b, rewriter);
 549:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 550-565

```cpp
 550:     // TF32 transpose is only supported with 128 swizzle mode with 32B
 551:     // atomicity. As we currently don't support this layout we disallow
 552:     // transpose for TF32 inputs.
 553:     bool allowTranspose = !dotOp.getA().getType().getElementType().isF32();
 554:     a = getSharedMemoryMMAOperand(a, rewriter, 0, allowTranspose);
 555:     b = getSharedMemoryMMAOperand(b, rewriter, 1, allowTranspose);
 556:     MLIRContext *context = dotOp->getContext();
 557:     auto instrShape = mmaVersionToInstrShape(
 558:         versionMajor, retShapePerCTA, oldAType.getElementType(), numWarps);
 559:     auto bitwidth = oldRetType.getElementType().getIntOrFloatBitWidth();
 560:     unsigned colStride = 32 / bitwidth;
 561:     Attribute accEncoding = triton::nvidia_gpu::TensorMemoryEncodingAttr::get(
 562:         context, instrShape[0], instrShape[1], colStride, CGALayout,
 563:         useTwoCTAs);
 564:     Attribute tensorMemorySpace =
 565:         triton::nvidia_gpu::TensorMemorySpaceAttr::get(context);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 566-582

```cpp
 566:     MemDescType accMemDescType =
 567:         MemDescType::get(oldRetType.getShape(), oldRetType.getElementType(),
 568:                          accEncoding, tensorMemorySpace,
 569:                          /*mutableMemory=*/true);
 570:     auto newDistributedEncoding =
 571:         nvidia_gpu::getDefaultLayoutForTmemLdSt(accMemDescType, numWarps);
 572:     auto newAccType = oldRetType.cloneWithEncoding(newDistributedEncoding);
 573:     Value cvtAcc =
 574:         ConvertLayoutOp::create(rewriter, loc, newAccType, dotOp.getOperand(2));
 575:     auto tokType = rewriter.getType<AsyncTokenType>();
 576:     auto acc = triton::nvidia_gpu::TMEMAllocOp::create(
 577:         rewriter, loc, accMemDescType, tokType, cvtAcc);
 578:     auto vTrue = arith::ConstantIntOp::create(rewriter, dotOp.getLoc(), 1, 1);
 579:     auto mma = triton::nvidia_gpu::TCGen5MMAOp::create(
 580:         rewriter, loc, tokType, a, b, acc, acc.getToken(), /*useD=*/vTrue,
 581:         /*pred=*/vTrue);
 582:     mma.setTwoCtas(useTwoCTAs);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。
### Lines 584-589

```cpp
 584:     auto ld = triton::nvidia_gpu::TMEMLoadOp::create(
 585:         rewriter, loc, newAccType, tokType, acc, /*dep=*/mma.getToken());
 586:     rewriter.replaceOpWithNewOp<ConvertLayoutOp>(dotOp, oldRetType, ld);
 587:     return success();
 588:   }
 589: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. It finishes by replacing the original operation with the lowered form.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 它会在转换成功后用降级后的形式替换原始操作。
### Lines 591-595

```cpp
 591: Value addSmemStageToScaleLoad(Value scale, mlir::PatternRewriter &rewriter) {
 592:   /*
 593:     Rewrite load(scale) -> local_load(local_alloc(load(scale))).
 594:     This function does not add anything to the final IR when num_stages > 1,
 595:     but it makes it easy to apply TMEM copy rewriting later.
```

- **EN:** Defines `addSmemStageToScaleLoad`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `addSmemStageToScaleLoad`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 597-604

```cpp
 597:     Since scales are stored in TMEM for MMAv5 scaled dot, loading of scales do
 598:     not needs to be put into SMEM. But in practice, the software pipeliner puts
 599:     loading of scales into multi-buffered SMEM. At that point, the SMEM
 600:     allocation created here is eliminated.
 601:    */
 602:   OpBuilder::InsertionGuard g(rewriter);
 603:   auto op = scale.getDefiningOp();
 604:   Operation *loadConsumer = nullptr;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 606-607

```cpp
 606:   if (!op)
 607:     return scale;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 609-625

```cpp
 609:   while (!isa<LoadOp, DescriptorLoadLikeOpInterface>(op)) {
 610:     if (auto reshape = dyn_cast<ReshapeOp>(op)) {
 611:       op = reshape.getSrc().getDefiningOp();
 612:       loadConsumer = reshape;
 613:     } else if (auto trans = dyn_cast<TransOp>(op)) {
 614:       op = trans.getSrc().getDefiningOp();
 615:       loadConsumer = trans;
 616:     } else if (auto cvt = dyn_cast<ConvertLayoutOp>(op)) {
 617:       op = cvt.getSrc().getDefiningOp();
 618:       loadConsumer = cvt;
 619:     } else {
 620:       // Unrecognized pattern, bail out. In practice, this implies that MMA
 621:       // pipelining will not apply to the scaled dot op, since scales will not
 622:       // be in passed through SMEM to tc_gen5_mma_scaled.
 623:       return scale;
 624:     }
 625:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 627-629

```cpp
 627:   auto scaleAfterLoad = op->getResult(0);
 628:   auto scaleSmemAlloc =
 629:       getSharedMemoryScale(scaleAfterLoad, rewriter, op->getLoc());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 631-633

```cpp
 631:   rewriter.setInsertionPointAfterValue(scaleSmemAlloc);
 632:   auto localLoad = LocalLoadOp::create(
 633:       rewriter, op->getLoc(), scaleAfterLoad.getType(), scaleSmemAlloc);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 635-636

```cpp
 635:   rewriter.replaceAllUsesExcept(scaleAfterLoad, localLoad.getResult(),
 636:                                 scaleSmemAlloc);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 638-643

```cpp
 638:   if (loadConsumer) {
 639:     return scale;
 640:   } else {
 641:     return localLoad;
 642:   }
 643: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 645-646

```cpp
 645: class ScaledBlockedToMMA : public mlir::OpRewritePattern<triton::DotScaledOp> {
 646:   int computeCapability;
```

- **EN:** Defines `ScaledBlockedToMMA`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `ScaledBlockedToMMA`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 648-652

```cpp
 648: public:
 649:   ScaledBlockedToMMA(mlir::MLIRContext *context, int computeCapability,
 650:                      int benefit)
 651:       : mlir::OpRewritePattern<triton::DotScaledOp>(context, benefit),
 652:         computeCapability(computeCapability) {}
```

- **EN:** Defines `ScaledBlockedToMMA`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `ScaledBlockedToMMA`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 654-658

```cpp
 654:   mlir::LogicalResult
 655:   matchAndRewrite(triton::DotScaledOp dotOp,
 656:                   mlir::PatternRewriter &rewriter) const override {
 657:     if (computeCapability / 10 != 12)
 658:       return failure();
```

- **EN:** Defines `matchAndRewrite`, the core MLIR rewrite callback. It checks whether the source operation matches the pattern and then materializes the lowered replacement.
- **CN:** 这里定义 `matchAndRewrite`，即 MLIR 模式重写的核心回调。它先检查源操作是否匹配模式，再构造降级后的替代结果。
### Lines 660-666

```cpp
 660:     auto numCTAs = lookupNumCTAs(rewriter);
 661:     if (numCTAs != 1) {
 662:       return failure();
 663:     }
 664:     // Skip if any scale is missing. This pattern requires both scales.
 665:     if (!dotOp.getAScale() || !dotOp.getBScale())
 666:       return failure();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 668-669

```cpp
 668:     auto aScaleType = dotOp.getAScale().getType();
 669:     auto bScaleType = dotOp.getBScale().getType();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 671-683

```cpp
 671:     if (mlir::isa<LinearEncodingAttr>(aScaleType.getEncoding()) ||
 672:         mlir::isa<LinearEncodingAttr>(bScaleType.getEncoding())) {
 673:       return failure();
 674:     }
 675:     auto aElemType = dotOp.getAElemType();
 676:     auto bElemType = dotOp.getBElemType();
 677:     auto isFP8 = [&](ScaleDotElemType elemType) -> bool {
 678:       return elemType == ScaleDotElemType::E4M3 ||
 679:              elemType == ScaleDotElemType::E5M2;
 680:     };
 681:     auto isFP4 = [&](ScaleDotElemType elemType) -> bool {
 682:       return elemType == ScaleDotElemType::E2M1;
 683:     };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 685-690

```cpp
 685:     // TODO: Enable mixed-precision mxfp for sm120
 686:     if (!((isFP8(aElemType) && isFP8(bElemType)) ||
 687:           (isFP4(aElemType) && isFP4(bElemType)))) {
 688:       return rewriter.notifyMatchFailure(
 689:           dotOp, "only FP8xFP8 and FP4xFP4 are supported on sm120");
 690:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 692-695

```cpp
 692:     auto scaleElemType = dotOp.getAScale().getType().getElementType();
 693:     if (scaleElemType != dotOp.getBScale().getType().getElementType()) {
 694:       return failure();
 695:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 697-699

```cpp
 697:     // Common MMA encoding creation
 698:     auto mmaResult =
 699:         createMMAEncodingForDot(dotOp, rewriter, computeCapability, 2);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 701-705

```cpp
 701:     // Operand processing
 702:     Value a = dotOp.getA();
 703:     Value b = dotOp.getB();
 704:     auto oldAType = cast<RankedTensorType>(a.getType());
 705:     auto oldBType = cast<RankedTensorType>(b.getType());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 707-707

```cpp
 707:     Operation *newDot = nullptr;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 709-712

```cpp
 709:     // ScaledBlockedToMMA logic
 710:     int bitwidthA = oldAType.getElementType().getIntOrFloatBitWidth();
 711:     int bitwidthB = oldBType.getElementType().getIntOrFloatBitWidth();
 712:     int minBitwidth = std::min(bitwidthA, bitwidthB);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 714-724

```cpp
 714:     Value newA = convertDotOperandForMMA(a, 0, minBitwidth,
 715:                                          mmaResult.newRetType, rewriter);
 716:     Value newB = convertDotOperandForMMA(b, 1, minBitwidth,
 717:                                          mmaResult.newRetType, rewriter);
 718:     const auto mmaWarps = mmaResult.mmaEnc.getWarpsPerCTA(); // [wM, wN]
 719:     // Convert scales to Linear layout
 720:     auto convertScale = [&](Value scale, int opIdx) -> Value {
 721:       auto ty = cast<RankedTensorType>(scale.getType());
 722:       SmallVector<int64_t> shape = llvm::to_vector(ty.getShape());
 723:       MLIRContext *ctx = ty.getContext();
 724:       auto blocked = cast<triton::gpu::BlockedEncodingAttr>(ty.getEncoding());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 726-733

```cpp
 726:       auto ll = triton::gpu::getSM120DotScaledScaleLayout(
 727:           ctx, shape, opIdx, mmaWarps, blocked.getCGALayout());
 728:       auto newEnc = triton::gpu::LinearEncodingAttr::get(ctx, std::move(ll));
 729:       auto newTy = RankedTensorType::get(shape, ty.getElementType(), newEnc);
 730:       return ConvertLayoutOp::create(rewriter, scale.getLoc(), newTy, scale);
 731:     };
 732:     Value aScale = convertScale(dotOp.getAScale(), /*opIdx=*/0);
 733:     Value bScale = convertScale(dotOp.getBScale(), /*opIdx=*/1);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 735-744

```cpp
 735:     newDot = triton::DotScaledOp::create(
 736:         rewriter, dotOp.getLoc(), mmaResult.newRetType, newA, newB,
 737:         mmaResult.newAcc, aScale, bScale, dotOp.getAElemType(),
 738:         dotOp.getBElemType(), dotOp.getFastMath(), dotOp.getLhsKPack(),
 739:         dotOp.getRhsKPack());
 740:     rewriter.replaceOpWithNewOp<ConvertLayoutOp>(dotOp, dotOp.getType(),
 741:                                                  newDot->getResult(0));
 742:     return success();
 743:   }
 744: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. It finishes by replacing the original operation with the lowered form.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 它会在转换成功后用降级后的形式替换原始操作。
### Lines 746-748

```cpp
 746: class ScaledBlockedToMMAv5
 747:     : public mlir::OpRewritePattern<triton::DotScaledOp> {
 748:   int computeCapability;
```

- **EN:** Defines `ScaledBlockedToMMAv5`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `ScaledBlockedToMMAv5`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 750-754

```cpp
 750: public:
 751:   ScaledBlockedToMMAv5(mlir::MLIRContext *context, int computeCapability,
 752:                        int benefit)
 753:       : mlir::OpRewritePattern<triton::DotScaledOp>(context, benefit),
 754:         computeCapability(computeCapability) {}
```

- **EN:** Defines `ScaledBlockedToMMAv5`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `ScaledBlockedToMMAv5`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 756-762

```cpp
 756:   mlir::LogicalResult
 757:   matchAndRewrite(triton::DotScaledOp dotOp,
 758:                   mlir::PatternRewriter &rewriter) const override {
 759:     RankedTensorType oldRetType = dotOp.getType();
 760:     if (!oldRetType.getEncoding() ||
 761:         mlir::isa<NvidiaMmaEncodingAttr>(oldRetType.getEncoding()))
 762:       return failure();
```

- **EN:** Defines `matchAndRewrite`, the core MLIR rewrite callback. It checks whether the source operation matches the pattern and then materializes the lowered replacement.
- **CN:** 这里定义 `matchAndRewrite`，即 MLIR 模式重写的核心回调。它先检查源操作是否匹配模式，再构造降级后的替代结果。
### Lines 764-766

```cpp
 764:     if (dotOp.getAScale() == nullptr || dotOp.getBScale() == nullptr) {
 765:       return failure();
 766:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 768-781

```cpp
 768:     // get MMA encoding for the given number of warps
 769:     auto retShapePerCTA = getShapePerCTA(oldRetType);
 770:     int numWarps = lookupNumWarps(dotOp);
 771:     auto CGALayout = getCGALayout(oldRetType.getEncoding());
 772:     if (computeCapability < 100 || computeCapability >= 120)
 773:       return failure();
 774:     if (numWarps != 4 && numWarps != 8)
 775:       return failure();
 776:     if (retShapePerCTA[0] < 128 || retShapePerCTA[1] < 16)
 777:       return failure();
 778:     Location loc = dotOp.getLoc();
 779:     // operands
 780:     Value a = dotOp.getA();
 781:     Value b = dotOp.getB();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 783-786

```cpp
 783:     bool IsAMixedPrecFp4 = false;
 784:     bool IsBMixedPrecFp4 = false;
 785:     bool isAFP4 = dotOp.getAElemType() == ScaleDotElemType::E2M1;
 786:     bool isBFP4 = dotOp.getBElemType() == ScaleDotElemType::E2M1;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 788-804

```cpp
 788:     if (dotOp.getAElemType() != dotOp.getBElemType()) {
 789:       if (isAFP4)
 790:         IsAMixedPrecFp4 = true;
 791:       else if (isBFP4)
 792:         IsBMixedPrecFp4 = true;
 793:     }
 794:     // If we use txgen05.mma.kind.mxf864 we need to padd the fp4 operands:
 795:     // https://docs.nvidia.com/cuda/parallel-thread-execution/#tcgen05-packing-formats-mxf8f6f4-smem
 796:     bool isMMAv5Fp4PaddedLhs = IsAMixedPrecFp4 || !dotOp.getLhsKPack();
 797:     bool isMMAv5Fp4PaddedRhs = IsBMixedPrecFp4 || !dotOp.getRhsKPack();
 798:     // For mixed-precision fp4 operands, set allowTranspose = false, to force
 799:     // the packed axis, K, to be contiguous in SMEM
 800:     a = getSharedMemoryMMAOperand(a, rewriter, 0,
 801:                                   /*allowTranspose=*/!isAFP4,
 802:                                   /*isMMAv5Fp4Padded=*/isMMAv5Fp4PaddedLhs,
 803:                                   /*forceTranspose=*/!dotOp.getLhsKPack(),
 804:                                   dotOp);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 805-809

```cpp
 805:     b = getSharedMemoryMMAOperand(b, rewriter, 1,
 806:                                   /*allowTranspose=*/!isBFP4,
 807:                                   /*isMMAv5Fp4Padded=*/isMMAv5Fp4PaddedRhs,
 808:                                   /*forceTranspose=*/!dotOp.getRhsKPack(),
 809:                                   dotOp);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 811-813

```cpp
 811:     MLIRContext *context = dotOp->getContext();
 812:     unsigned m = 128;
 813:     unsigned n = retShapePerCTA[1] >= 256 ? 256 : retShapePerCTA[1];
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 815-832

```cpp
 815:     auto bitwidth = oldRetType.getElementType().getIntOrFloatBitWidth();
 816:     unsigned colStride = 32 / bitwidth;
 817:     Attribute accEncoding = triton::nvidia_gpu::TensorMemoryEncodingAttr::get(
 818:         context, m, n, colStride, CGALayout, false);
 819:     Attribute tensorMemorySpace =
 820:         triton::nvidia_gpu::TensorMemorySpaceAttr::get(context);
 821:     MemDescType accMemDescType =
 822:         MemDescType::get(oldRetType.getShape(), oldRetType.getElementType(),
 823:                          accEncoding, tensorMemorySpace,
 824:                          /*mutableMemory=*/true);
 825:     auto newDistributedEncoding =
 826:         nvidia_gpu::getDefaultLayoutForTmemLdSt(accMemDescType, numWarps);
 827:     auto newAccType = oldRetType.cloneWithEncoding(newDistributedEncoding);
 828:     Value cvtAcc =
 829:         ConvertLayoutOp::create(rewriter, loc, newAccType, dotOp.getOperand(2));
 830:     auto tokType = rewriter.getType<AsyncTokenType>();
 831:     auto acc = triton::nvidia_gpu::TMEMAllocOp::create(
 832:         rewriter, loc, accMemDescType, tokType, cvtAcc);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。
### Lines 834-835

```cpp
 834:     RankedTensorType oldScaleAType = dotOp.getAScale().getType();
 835:     RankedTensorType oldScaleBType = dotOp.getBScale().getType();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 837-853

```cpp
 837:     Attribute scaleEncoding =
 838:         triton::nvidia_gpu::TensorMemoryScalesEncodingAttr::get(context,
 839:                                                                 CGALayout);
 840:     MemDescType scaleAType = triton::gpu::MemDescType::get(
 841:         oldScaleAType.getShape(), oldScaleAType.getElementType(), scaleEncoding,
 842:         tensorMemorySpace,
 843:         /*mutableMemory=*/false);
 844:     MemDescType scaleBType = triton::gpu::MemDescType::get(
 845:         oldScaleBType.getShape(), oldScaleBType.getElementType(), scaleEncoding,
 846:         tensorMemorySpace,
 847:         /*mutableMemory=*/false);
 848:     Attribute scaleALayout =
 849:         nvidia_gpu::getDefaultLayoutForTmemLdSt(scaleAType, numWarps);
 850:     Attribute scaleBLayout =
 851:         nvidia_gpu::getDefaultLayoutForTmemLdSt(scaleBType, numWarps);
 852:     RankedTensorType newScaleAType =
 853:         oldScaleAType.cloneWithEncoding(scaleALayout);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。
### Lines 854-855

```cpp
 854:     RankedTensorType newScaleBType =
 855:         oldScaleBType.cloneWithEncoding(scaleBLayout);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 857-858

```cpp
 857:     auto lhsScale = addSmemStageToScaleLoad(dotOp.getAScale(), rewriter);
 858:     auto rhsScale = addSmemStageToScaleLoad(dotOp.getBScale(), rewriter);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 860-863

```cpp
 860:     Value newScaleA =
 861:         ConvertLayoutOp::create(rewriter, loc, newScaleAType, lhsScale);
 862:     Value newScaleB =
 863:         ConvertLayoutOp::create(rewriter, loc, newScaleBType, rhsScale);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 865-870

```cpp
 865:     // We don't need to track memory dependencies for the scale operands since
 866:     // they are not pipelined.
 867:     auto scaleA = triton::nvidia_gpu::TMEMAllocOp::create(
 868:         rewriter, loc, scaleAType, /*token=*/Type(), newScaleA);
 869:     auto scaleB = triton::nvidia_gpu::TMEMAllocOp::create(
 870:         rewriter, loc, scaleBType, /*token=*/Type(), newScaleB);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 872-877

```cpp
 872:     auto vTrue = arith::ConstantIntOp::create(rewriter, dotOp.getLoc(), 1, 1);
 873:     auto mmaOp = triton::nvidia_gpu::TCGen5MMAScaledOp::create(
 874:         rewriter, loc, tokType, a, b, acc.getResult(), acc.getToken(),
 875:         scaleA.getResult(), scaleB.getResult(), dotOp.getAElemType(),
 876:         dotOp.getBElemType(),
 877:         /*useD=*/vTrue, /*pred=*/vTrue);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 879-885

```cpp
 879:     auto ld = triton::nvidia_gpu::TMEMLoadOp::create(
 880:         rewriter, loc, newAccType, tokType, acc, mmaOp.getToken());
 881:     rewriter.replaceOpWithNewOp<ConvertLayoutOp>(dotOp, oldRetType, ld);
 882:     return success();
 883:   }
 884: };
 885: } // namespace
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. It finishes by replacing the original operation with the lowered form.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 它会在转换成功后用降级后的形式替换原始操作。
### Lines 887-897

```cpp
 887: static Value promoteOperand(OpBuilder &builder, Location loc, Value operand,
 888:                             Type promotedType) {
 889:   Type tensorPromotedType = cast<RankedTensorType>(operand.getType())
 890:                                 .cloneWith(std::nullopt, promotedType);
 891:   Type operandElType =
 892:       cast<RankedTensorType>(operand.getType()).getElementType();
 893:   if (type::isFloat8(operandElType)) {
 894:     return FpToFpOp::create(builder, loc, tensorPromotedType, operand);
 895:   }
 896:   return arith::ExtFOp::create(builder, loc, tensorPromotedType, operand);
 897: }
```

- **EN:** Defines `promoteOperand`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `promoteOperand`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 899-905

```cpp
 899: static bool mmav2SupportsFp8Operands(int computeCapability) {
 900:   // promote operands for sm < 89 since fp8 mma is not natively supported
 901:   // although PTX instructions for mma v2 w/ fp8 operands exist for sm90 and
 902:   // sm100, they are emulated as fp16 upcasts + fp16 HMMA in SASS. sm120 has
 903:   // hardware support for fp8 operands w/ mmav2.
 904:   return computeCapability == 89 || computeCapability / 10 == 12;
 905: }
```

- **EN:** Defines `mmav2SupportsFp8Operands`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `mmav2SupportsFp8Operands`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 907-924

```cpp
 907: // promote operands of dot op if the existing combination is not natively
 908: // supported.
 909: static void decomposeMixedModeDotOp(ModuleOp mod, int computeCapability) {
 910:   mod.walk([=](DotOp dotOp) -> void {
 911:     auto D = dotOp.getD();
 912:     OpBuilder builder(dotOp);
 913:     Type AElType = dotOp.getA().getType().getElementType();
 914:     Type promoteType;
 915:     NvidiaMmaEncodingAttr mmaLayout =
 916:         dyn_cast<NvidiaMmaEncodingAttr>(D.getType().getEncoding());
 917:     if (mmaLayout) {
 918:       bool isNativeFP8 = llvm::isa<Float8E5M2Type, Float8E4M3FNType>(AElType);
 919:       // promote to f16 unless there's hardware support for fp8 operands
 920:       if (!isNativeFP8 ||
 921:           (isNativeFP8 && (mmav2SupportsFp8Operands(computeCapability) ||
 922:                            mmaLayout.isHopper())))
 923:         return;
 924:       promoteType = builder.getF16Type();
```

- **EN:** Defines `decomposeMixedModeDotOp`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `decomposeMixedModeDotOp`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 925-939

```cpp
 925:     } else {
 926:       // FMA case.
 927:       Type AElType = dotOp.getA().getType().getElementType();
 928:       Type DElType = D.getType().getElementType();
 929:       if (AElType == DElType)
 930:         return;
 931:       promoteType = DElType;
 932:     }
 933:     Location loc = dotOp.getLoc();
 934:     Value promotedA = promoteOperand(builder, loc, dotOp.getA(), promoteType);
 935:     Value promotedB = promoteOperand(builder, loc, dotOp.getB(), promoteType);
 936:     dotOp.setOperand(0, promotedA);
 937:     dotOp.setOperand(1, promotedB);
 938:   });
 939: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 941-958

```cpp
 941: // Transpose scaled_dot ops that have a scale on lhs.
 942: static void transposeDotOp(DotScaledOp dotOp) {
 943:   OpBuilder builder(dotOp);
 944:   Value lhs = dotOp.getA();
 945:   std::array<int, 2> transOrder = {1, 0};
 946:   Value lhsTransposed = TransOp::create(builder, lhs.getLoc(), lhs, transOrder);
 947:   Value rhs = dotOp.getB();
 948:   Value rhsTransposed = TransOp::create(builder, rhs.getLoc(), rhs, transOrder);
 949:   Value c = dotOp.getC();
 950:   Value cTransposed = TransOp::create(builder, c.getLoc(), c, transOrder);
 951:   Value result = DotScaledOp::create(
 952:       builder, dotOp.getLoc(), cTransposed.getType(), rhsTransposed,
 953:       lhsTransposed, cTransposed, dotOp.getBScale(), dotOp.getAScale(),
 954:       dotOp.getBElemType(), dotOp.getAElemType(), dotOp.getFastMath());
 955:   Operation *transposedResult =
 956:       TransOp::create(builder, result.getLoc(), result, transOrder);
 957:   dotOp.replaceAllUsesWith(transposedResult);
 958:   dotOp.erase();
```

- **EN:** Defines `transposeDotOp`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `transposeDotOp`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 959-959

```cpp
 959: }
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 961-972

```cpp
 961: static void transposeDots(ModuleOp m) {
 962:   // TODO: extend to regular dot when it is profitable. For instance when we may
 963:   // want to use rhs from register for mmav3.
 964:   SmallVector<DotScaledOp> toTranspose;
 965:   m.walk([&](DotScaledOp dotOp) -> void {
 966:     if (dotOp.getAScale() == nullptr && dotOp.getBScale() != nullptr)
 967:       toTranspose.push_back(dotOp);
 968:   });
 969:   for (DotScaledOp dotOp : toTranspose) {
 970:     transposeDotOp(dotOp);
 971:   }
 972: }
```

- **EN:** Defines `transposeDots`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. The implementation traverses IR operations to collect facts or apply rewrites globally. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `transposeDots`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 974-975

```cpp
 974: #define GEN_PASS_DEF_TRITONGPUACCELERATEMATMUL
 975: #include "triton/Dialect/TritonGPU/Transforms/Passes.h.inc"
```

- **EN:** Pulls in generated definitions from TableGen/MLIR include fragments so the handwritten code can reuse auto-generated declarations.
- **CN:** 这里引入由 TableGen/MLIR 生成的定义片段，使手写代码能够复用自动生成的声明。
### Lines 977-982

```cpp
 977: class TritonGPUAccelerateMatmulPass
 978:     : public impl::TritonGPUAccelerateMatmulBase<
 979:           TritonGPUAccelerateMatmulPass> {
 980: public:
 981:   using impl::TritonGPUAccelerateMatmulBase<
 982:       TritonGPUAccelerateMatmulPass>::TritonGPUAccelerateMatmulBase;
```

- **EN:** Defines `TritonGPUAccelerateMatmulPass`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `TritonGPUAccelerateMatmulPass`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 984-986

```cpp
 984:   void runOnOperation() override {
 985:     MLIRContext *context = &getContext();
 986:     ModuleOp m = getOperation();
```

- **EN:** Defines `runOnOperation`, the pass entry point. It gathers analysis information and applies the file's transformation logic to the current operation.
- **CN:** 这里定义 `runOnOperation`，即 pass 的入口函数。它会收集分析信息，并把本文件的变换逻辑应用到当前操作上。
### Lines 988-992

```cpp
 988:     auto computeCapability = getNVIDIAComputeCapability(m);
 989:     // We could do this generically if we manage to improve the heuristics
 990:     // reverted in these two PRs https://github.com/triton-lang/triton/pull/5834
 991:     // https://github.com/triton-lang/triton/pull/5837
 992:     transposeDots(m);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 994-997

```cpp
 994:     mlir::RewritePatternSet patterns(context);
 995:     constexpr int benefitDefault = 1;
 996:     constexpr int benefitMMAv5 = 10;
 997:     constexpr int benefitSM120 = 10;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 999-1003

```cpp
 999:     patterns.add<BlockedToMMA>(context, computeCapability, benefitDefault);
1000:     patterns.add<ScaledBlockedToMMA>(context, computeCapability, benefitSM120);
1001:     populateDecomposeScaledBlockedPatterns(patterns, benefitDefault);
1002:     patterns.add<BlockedToMMAv5, ScaledBlockedToMMAv5>(
1003:         context, computeCapability, benefitMMAv5);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1005-1011

```cpp
1005:     if (applyPatternsGreedily(m, std::move(patterns)).failed())
1006:       return signalPassFailure();
1007:     // Now that we have picked the mma type, decompose dot that are not natively
1008:     // supported.
1009:     decomposeMixedModeDotOp(m, computeCapability);
1010:   }
1011: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1013-1015

```cpp
1013: } // namespace gpu
1014: } // namespace triton
1015: } // namespace mlir
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。

## Key Concepts / 关键概念
- **EN:** The main concern is pass-driven transformation around accelerate matmul.
  **CN:** 核心关注点是围绕 Accelerate Matmul 的 pass 驱动变换。
- **EN:** Pattern rewriting is the main mechanism for canonicalization and lowering in this file.
  **CN:** 模式重写是本文件进行规范化与降级转换的核心机制。
- **EN:** Memory descriptor types carry layout, rank, and address-space information.
  **CN:** 内存描述符类型携带布局、维度和地址空间信息。
- **EN:** Tensor shape and element-type reasoning is central to the implementation.
  **CN:** 张量形状与元素类型推导是实现中的核心内容。
- **EN:** Linear layout utilities translate between logical tensor coordinates and physical placement.
  **CN:** 线性布局工具负责在逻辑张量坐标与物理放置之间进行转换。
- **EN:** Encoding attributes describe how distributed GPU data is laid out across threads and warps.
  **CN:** 编码属性描述 GPU 分布式数据在线程与 warp 之间的布局方式。

## Dependencies / 依赖关系
- **Internal Triton headers / Triton 内部头文件:** `triton/Analysis/Utility.h`, `triton/Conversion/MLIRTypes.h`, `triton/Dialect/Triton/IR/Dialect.h`, `triton/Dialect/Triton/IR/OpInterfaces.h`, `triton/Dialect/Triton/IR/Utility.h`, `triton/Dialect/TritonGPU/IR/Attributes.h`, ... (+8 more)
- **MLIR headers / MLIR 头文件:** `mlir/Analysis/SliceAnalysis.h`, `mlir/IR/BuiltinAttributes.h`, `mlir/IR/BuiltinTypes.h`, `mlir/IR/TypeUtilities.h`, `mlir/IR/Types.h`, `mlir/IR/Value.h`, ... (+3 more)
- **LLVM headers / LLVM 头文件:** `llvm/ADT/ArrayRef.h`, `llvm/ADT/SmallVector.h`, `llvm/Support/Casting.h`
- **Standard/library headers / 标准或通用库头文件:** None
- **Generated fragments / 生成片段:** `triton/Dialect/TritonGPU/Transforms/Passes.h.inc`
- **Primary APIs used / 主要 API:** `PatternRewriter`, `ModuleOp`, `RankedTensorType`, `MemDescType`, `LinearLayout`, `OpBuilder`
- **Pipeline role / 流程角色:** This file participates in Triton pass pipelines and usually expects upstream analyses or dialect invariants to have prepared the IR. / 本文件参与 Triton 的 pass 流程，通常依赖上游分析或方言不变量先把 IR 准备好。
