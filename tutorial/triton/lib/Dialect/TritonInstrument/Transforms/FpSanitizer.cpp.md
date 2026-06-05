# FpSanitizer.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `lib/Dialect/TritonInstrument/Transforms/FpSanitizer.cpp`
- **Purpose / 作用:** **EN:** Implements theFP sanitizeritizer transformation or optimization pass for the TritonInstrument pipeline. **CN:** 为 TritonInstrument 编译流程实现与 FP Sanitizer 相关的变换或优化 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15

```cpp
   1: #include "mlir/IR/Diagnostics.h"
   2: #include "mlir/IR/PatternMatch.h"
   3: #include "mlir/IR/Types.h"
   4: #include "mlir/Transforms/GreedyPatternRewriteDriver.h"
   5: #include "mlir/Transforms/RegionUtils.h"
   6: #include "triton/Dialect/Triton/IR/Dialect.h"
   7: #include "triton/Dialect/TritonGPU/IR/Dialect.h"
   8: #include "triton/Dialect/TritonGPU/Transforms/Utility.h"
   9: #include "triton/Dialect/TritonInstrument/IR/Dialect.h"
  10: #include "triton/Dialect/TritonInstrument/IR/Utility.h"
  11: #include "triton/Dialect/TritonInstrument/Transforms/Passes.h"
  12: #include "triton/Dialect/TritonNvidiaGPU/IR/Dialect.h"
  13: #include "llvm/ADT/DenseSet.h"
  14: #include "llvm/ADT/STLExtras.h"
  15: #include <cassert>
```

- **EN:** Includes the interfaces this file depends on. Triton headers (`Dialect.h`, `Dialect.h`, `Utility.h`, `Dialect.h`, ... (+3 more)) provide domain-specific IR/support, MLIR headers (`Diagnostics.h`, `PatternMatch.h`, `Types.h`, `GreedyPatternRewriteDriver.h`, ... (+1 more)) provide rewriting and analysis infrastructure, LLVM headers (`DenseSet.h`, `STLExtras.h`) supply low-level utilities, and standard/library headers (`cassert`) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（`Dialect.h`, `Dialect.h`, `Utility.h`, `Dialect.h`, ... (+3 more)）提供领域特定 IR/支持逻辑，MLIR 头文件（`Diagnostics.h`, `PatternMatch.h`, `Types.h`, `GreedyPatternRewriteDriver.h`, ... (+1 more)）提供重写与分析基础设施，LLVM 头文件（`DenseSet.h`, `STLExtras.h`）提供底层工具，而标准/通用库头文件（`cassert`）提供通用能力。
### Lines 17-19

```cpp
  17: namespace mlir {
  18: namespace triton {
  19: namespace instrument {
```

- **EN:** Opens or closes the namespace nesting for mlir -> triton -> instrument, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 mlir -> triton -> instrument 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 21-23

```cpp
  21: namespace tt = mlir::triton;
  22: namespace ttg = mlir::triton::gpu;
  23: namespace ttng = mlir::triton::nvidia_gpu;
```

- **EN:** Opens or closes the namespace nesting for tt -> ttg -> ttng, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 tt -> ttg -> ttng 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 25-26

```cpp
  25: #define GEN_PASS_DEF_TRITONINSTRUMENTFPSANITIZER
  26: #include "triton/Dialect/TritonInstrument/Transforms/Passes.h.inc"
```

- **EN:** Pulls in generated definitions from TableGen/MLIR include fragments so the handwritten code can reuse auto-generated declarations.
- **CN:** 这里引入由 TableGen/MLIR 生成的定义片段，使手写代码能够复用自动生成的声明。
### Lines 28-28

```cpp
  28: namespace {
```

- **EN:** Opens or closes the namespace nesting for (anonymous), keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 (anonymous) 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 30-32

```cpp
  30: Type getIntTypeLike(Type ty);
  31: bool isFloatLike(Type ty) { return isa<FloatType>(getElementTypeOrSelf(ty)); }
  32: bool isIntLike(Type ty) { return isa<IntegerType>(getElementTypeOrSelf(ty)); }
```

- **EN:** Defines `isFloatLike`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `isFloatLike`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 34-37

```cpp
  34: bool isNumericLike(Type ty) {
  35:   Type elemTy = getElementTypeOrSelf(ty);
  36:   return isa<FloatType>(elemTy) || isa<IntegerType>(elemTy);
  37: }
```

- **EN:** Defines `isNumericLike`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `isNumericLike`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 39-51

```cpp
  39: static bool isValueAvailableInScope(Value value, Region *scope) {
  40:   if (!scope)
  41:     return false;
  42:   if (auto arg = dyn_cast<BlockArgument>(value)) {
  43:     Region *argRegion = arg.getOwner()->getParent();
  44:     return argRegion == scope || scope->isAncestor(argRegion);
  45:   }
  46:   if (Operation *def = value.getDefiningOp()) {
  47:     Region *defRegion = def->getParentRegion();
  48:     return defRegion == scope || scope->isAncestor(defRegion);
  49:   }
  50:   return false;
  51: }
```

- **EN:** Defines `isValueAvailableInScope`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `isValueAvailableInScope`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 53-54

```cpp
  53: constexpr int64_t kTileM = 8;
  54: constexpr int64_t kTileN = 8;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 56-59

```cpp
  56: void createGlobalScratchBarrier(PatternRewriter &rewriter, Location loc) {
  57:   ttg::BarrierOp::create(
  58:       rewriter, loc, ttg::AddrSpace::GlobalRead | ttg::AddrSpace::GlobalWrite);
  59: }
```

- **EN:** Defines helper `createGlobalScratchBarrier` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createGlobalScratchBarrier`，用于计算或构造外围变换所需的中间数据。
### Lines 61-74

```cpp
  61: enum class UnaryOpId : uint64_t {
  62:   Exp = 0,
  63:   Log,
  64:   Exp2,
  65:   Log2,
  66:   Cos,
  67:   Sin,
  68:   Sqrt,
  69:   Rsqrt,
  70:   Erf,
  71:   Floor,
  72:   Ceil,
  73:   PreciseSqrt,
  74: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 76-78

```cpp
  76: constexpr uint64_t getUnaryOpId(UnaryOpId opId) {
  77:   return static_cast<uint64_t>(opId);
  78: }
```

- **EN:** Defines accessor/helper `getUnaryOpId` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getUnaryOpId`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 80-82

```cpp
  80: // ------------------------------------------------------------
  81: // Scratch memory management
  82: // ------------------------------------------------------------
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 84-100

```cpp
  84: ttg::BlockedEncodingAttr getOptimizedBlockedEncoding(PatternRewriter &rewriter,
  85:                                                      ArrayRef<int64_t> shape,
  86:                                                      Type elemType) {
  87:   int numWarps = ttg::lookupNumWarps(rewriter.getInsertionBlock()->getParent());
  88:   int threadsPerWarp = ttg::lookupThreadsPerWarp(rewriter);
  89:   int numCTAs = ttg::lookupNumCTAs(rewriter.getInsertionBlock()->getParentOp());
  90:   auto base = ttg::getDefaultBlockedEncoding(rewriter.getContext(), shape,
  91:                                              numWarps, threadsPerWarp, numCTAs);
  92:   SmallVector<unsigned> order = llvm::to_vector(base.getOrder());
  93:   SmallVector<unsigned> sizePerThread(shape.size(), 1);
  94:   unsigned elemBits = elemType.getIntOrFloatBitWidth();
  95:   unsigned maxElems = std::max(128u / elemBits, 1u);
  96:   if (!order.empty()) {
  97:     unsigned dim = order.front();
  98:     sizePerThread[dim] =
  99:         static_cast<unsigned>(std::min<int64_t>(shape[dim], maxElems));
 100:   }
```

- **EN:** Defines accessor/helper `getOptimizedBlockedEncoding` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getOptimizedBlockedEncoding`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 101-104

```cpp
 101:   return ttg::BlockedEncodingAttr::get(
 102:       rewriter.getContext(), sizePerThread, base.getThreadsPerWarp(),
 103:       base.getWarpsPerCTA(), order, base.getCGALayout());
 104: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 106-109

```cpp
 106: struct ScratchInfo {
 107:   Value ptr;
 108:   RankedTensorType tensorType;
 109: };
```

- **EN:** Defines `ScratchInfo`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `ScratchInfo`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 111-114

```cpp
 111: struct ScratchState {
 112:   std::optional<ScratchInfo> canonical;
 113:   DenseMap<Region *, ScratchInfo> byScope;
 114: };
```

- **EN:** Defines `ScratchState`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `ScratchState`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 116-120

```cpp
 116: Type getScratchStorageElementType(Type elemTy) {
 117:   if (auto floatTy = dyn_cast<FloatType>(elemTy))
 118:     return IntegerType::get(elemTy.getContext(), floatTy.getWidth());
 119:   return elemTy;
 120: }
```

- **EN:** Defines accessor/helper `getScratchStorageElementType` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getScratchStorageElementType`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 122-125

```cpp
 122: RankedTensorType getScratchStorageType(RankedTensorType tensorTy) {
 123:   auto elemTy = getScratchStorageElementType(tensorTy.getElementType());
 124:   return tensorTy.clone(elemTy);
 125: }
```

- **EN:** Defines accessor/helper `getScratchStorageType` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getScratchStorageType`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 127-132

```cpp
 127: Value embedToInt(PatternRewriter &rewriter, Location loc, Value v) {
 128:   if (isa<IntegerType>(getElementTypeOrSelf(v.getType())))
 129:     return v;
 130:   return ExperimentalFPSanEmbedOp::create(rewriter, loc,
 131:                                           getIntTypeLike(v.getType()), v);
 132: }
```

- **EN:** Defines `embedToInt`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `embedToInt`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 134-137

```cpp
 134: Value unembedToFloat(PatternRewriter &rewriter, Location loc, Value v,
 135:                      Type floatTy) {
 136:   return ExperimentalFPSanUnembedOp::create(rewriter, loc, floatTy, v);
 137: }
```

- **EN:** Defines `unembedToFloat`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `unembedToFloat`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 139-146

```cpp
 139: Value loadFpSanScratchMemory(PatternRewriter &rewriter, Location loc,
 140:                              Value alloc, RankedTensorType tensorTy) {
 141:   auto storageTy = getScratchStorageType(tensorTy);
 142:   Value stored = createLoadScratchMemory(rewriter, loc, alloc, storageTy);
 143:   if (isFloatLike(tensorTy))
 144:     return unembedToFloat(rewriter, loc, stored, tensorTy);
 145:   return stored;
 146: }
```

- **EN:** Defines `loadFpSanScratchMemory`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `loadFpSanScratchMemory`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 148-156

```cpp
 148: Operation *storeFpSanScratchMemory(PatternRewriter &rewriter, Location loc,
 149:                                    Value alloc, Value tensor,
 150:                                    RankedTensorType tensorTy) {
 151:   auto storageTy = getScratchStorageType(tensorTy);
 152:   Value stored = tensor;
 153:   if (isFloatLike(tensorTy))
 154:     stored = embedToInt(rewriter, loc, tensor);
 155:   return createStoreScratchMemory(rewriter, loc, alloc, stored, storageTy);
 156: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 158-165

```cpp
 158: class TmemScratchManager {
 159: public:
 160:   ttg::BlockedEncodingAttr getScratchEncoding(PatternRewriter &rewriter,
 161:                                               Value memdesc,
 162:                                               ttg::MemDescType memTy) {
 163:     return getOptimizedBlockedEncoding(rewriter, memTy.getShape(),
 164:                                        memTy.getElementType());
 165:   }
```

- **EN:** Defines `TmemScratchManager`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic.
- **CN:** 这里定义 `TmemScratchManager`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。
### Lines 167-180

```cpp
 167:   static Value castToI32(PatternRewriter &rewriter, Location loc, Value value) {
 168:     auto i32Ty = rewriter.getI32Type();
 169:     auto ty = value.getType();
 170:     if (ty == i32Ty)
 171:       return value;
 172:     if (ty.isIndex())
 173:       return arith::IndexCastOp::create(rewriter, loc, i32Ty, value);
 174:     if (auto intTy = dyn_cast<IntegerType>(ty)) {
 175:       if (intTy.getWidth() > 32)
 176:         return arith::TruncIOp::create(rewriter, loc, i32Ty, value);
 177:       return arith::ExtSIOp::create(rewriter, loc, i32Ty, value);
 178:     }
 179:     return Value();
 180:   }
```

- **EN:** Defines `castToI32`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `castToI32`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 182-198

```cpp
 182:   std::optional<ScratchInfo>
 183:   getOrCreate(Value memdesc, PatternRewriter &rewriter, Region *scope) {
 184:     if (auto arg = dyn_cast<BlockArgument>(memdesc)) {
 185:       if (auto wsPartitions = dyn_cast<ttg::WarpSpecializePartitionsOp>(
 186:               arg.getOwner()->getParentOp())) {
 187:         auto capture = wsPartitions.getExplicitCaptures()[arg.getArgNumber()];
 188:         return getOrCreate(capture, rewriter, scope);
 189:       }
 190:       if (auto forOp = dyn_cast<scf::ForOp>(arg.getOwner()->getParentOp())) {
 191:         unsigned argNum = arg.getArgNumber();
 192:         if (argNum == 0)
 193:           return std::nullopt;
 194:         Value init = forOp.getInitArgs()[argNum - 1];
 195:         return getOrCreate(init, rewriter, scope);
 196:       }
 197:       return std::nullopt;
 198:     }
```

- **EN:** Defines accessor/helper `getOrCreate` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getOrCreate`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 200-203

```cpp
 200:     auto memTy = dyn_cast<ttg::MemDescType>(memdesc.getType());
 201:     if (!memTy || !isa<ttng::TensorMemorySpaceAttr>(memTy.getMemorySpace())) {
 202:       return std::nullopt;
 203:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。
### Lines 205-219

```cpp
 205:     if (auto alloc = memdesc.getDefiningOp<ttng::TMEMAllocOp>()) {
 206:       ScratchState &state = scratchMap[memdesc];
 207:       auto itRegion = state.byScope.find(scope);
 208:       if (itRegion != state.byScope.end()) {
 209:         if (itRegion->second.ptr && itRegion->second.ptr.getType())
 210:           return itRegion->second;
 211:         state.byScope.erase(itRegion);
 212:       }
 213:       if (state.canonical) {
 214:         Value ptr =
 215:             remapToScope(state.canonical->ptr, rewriter, scope, alloc.getLoc());
 216:         ScratchInfo info{ptr, state.canonical->tensorType};
 217:         state.byScope[scope] = info;
 218:         return info;
 219:       }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 221-227

```cpp
 221:       OpBuilder::InsertionGuard guard(rewriter);
 222:       rewriter.setInsertionPointAfter(alloc);
 223:       auto loc = alloc.getLoc();
 224:       auto layout = getScratchEncoding(rewriter, memdesc, memTy);
 225:       auto tensorTy = RankedTensorType::get(memTy.getShape(),
 226:                                             memTy.getElementType(), layout);
 227:       auto storageElemTy = getScratchStorageElementType(memTy.getElementType());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 229-235

```cpp
 229:       int64_t elSize = memTy.getElementType().getIntOrFloatBitWidth() / 8;
 230:       int64_t alignment = std::max<int64_t>(elSize, 16);
 231:       int64_t sizeInBytes = product(memTy.getShape()) * elSize;
 232:       auto ptrTy = triton::getPointerType(storageElemTy);
 233:       auto allocOp = createThirdPartyScratchAlloc(rewriter, loc, ptrTy,
 234:                                                   sizeInBytes, alignment);
 235:       Value ptr = allocOp.getResult();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 237-241

```cpp
 237:       if (Value init = alloc.getSrc()) {
 238:         auto initTy = cast<RankedTensorType>(init.getType());
 239:         if (!storeFpSanScratchMemory(rewriter, loc, ptr, init, initTy))
 240:           return std::nullopt;
 241:       }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 243-243

```cpp
 243:       state.canonical = ScratchInfo{ptr, tensorTy};
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 245-249

```cpp
 245:       ptr = remapToScope(ptr, rewriter, scope, loc);
 246:       ScratchInfo info{ptr, tensorTy};
 247:       state.byScope[scope] = info;
 248:       return info;
 249:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 251-254

```cpp
 251:     if (auto subslice = memdesc.getDefiningOp<ttng::TMEMSubSliceOp>()) {
 252:       auto baseInfo = getOrCreate(subslice.getSrc(), rewriter, scope);
 253:       if (!baseInfo)
 254:         return std::nullopt;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 256-258

```cpp
 256:       auto baseTy = cast<ttg::MemDescType>(subslice.getSrc().getType());
 257:       if (baseTy.getRank() < 2 || memTy.getRank() != 2)
 258:         return std::nullopt;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。
### Lines 260-276

```cpp
 260:       OpBuilder::InsertionGuard guard(rewriter);
 261:       rewriter.setInsertionPoint(subslice);
 262:       auto loc = subslice.getLoc();
 263:       int64_t stride = baseTy.getShape().front();
 264:       if (baseTy.getRank() > 2)
 265:         stride = product(baseTy.getShape().drop_front(1));
 266:       int64_t offset = subslice.getN();
 267:       auto offsetVal = arith::ConstantOp::create(
 268:           rewriter, loc, rewriter.getI32IntegerAttr(offset));
 269:       auto strideVal = arith::ConstantOp::create(
 270:           rewriter, loc, rewriter.getI32IntegerAttr(stride));
 271:       auto offsetEls = arith::MulIOp::create(
 272:           rewriter, loc, rewriter.getI32Type(), offsetVal, strideVal);
 273:       Value ptr = tt::AddPtrOp::create(rewriter, loc, baseInfo->ptr.getType(),
 274:                                        baseInfo->ptr, offsetEls);
 275:       ptr = remapToScope(ptr, rewriter, scope, loc);
 276:       auto layout = getScratchEncoding(rewriter, memdesc, memTy);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 277-278

```cpp
 277:       auto tensorTy = RankedTensorType::get(memTy.getShape(),
 278:                                             memTy.getElementType(), layout);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 280-282

```cpp
 280:       ScratchInfo info{ptr, tensorTy};
 281:       return info;
 282:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 284-287

```cpp
 284:     if (auto view = memdesc.getDefiningOp<ttg::MemDescIndexOp>()) {
 285:       auto baseInfo = getOrCreate(view.getSrc(), rewriter, scope);
 286:       if (!baseInfo)
 287:         return std::nullopt;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 289-291

```cpp
 289:       auto baseTy = cast<ttg::MemDescType>(view.getSrc().getType());
 290:       if (baseTy.getRank() < 2)
 291:         return std::nullopt;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。
### Lines 293-308

```cpp
 293:       OpBuilder::InsertionGuard guard(rewriter);
 294:       rewriter.setInsertionPoint(view);
 295:       auto loc = view.getLoc();
 296:       Value idx = view.getIndex();
 297:       idx = castToI32(rewriter, loc, idx);
 298:       int64_t stride = product(baseTy.getShape().drop_front(1));
 299:       auto strideVal = arith::ConstantOp::create(
 300:           rewriter, loc, rewriter.getI32IntegerAttr(stride));
 301:       auto offset = arith::MulIOp::create(rewriter, loc, rewriter.getI32Type(),
 302:                                           idx, strideVal);
 303:       Value ptr = tt::AddPtrOp::create(rewriter, loc, baseInfo->ptr.getType(),
 304:                                        baseInfo->ptr, offset);
 305:       ptr = remapToScope(ptr, rewriter, scope, loc);
 306:       auto layout = getScratchEncoding(rewriter, memdesc, memTy);
 307:       auto tensorTy = RankedTensorType::get(memTy.getShape(),
 308:                                             memTy.getElementType(), layout);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 310-312

```cpp
 310:       ScratchInfo info{ptr, tensorTy};
 311:       return info;
 312:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 314-317

```cpp
 314:     if (auto view = memdesc.getDefiningOp<ttg::MemDescReinterpretOp>()) {
 315:       auto baseInfo = getOrCreate(view.getSrc(), rewriter, scope);
 316:       if (!baseInfo)
 317:         return std::nullopt;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 319-328

```cpp
 319:       OpBuilder::InsertionGuard guard(rewriter);
 320:       rewriter.setInsertionPoint(view);
 321:       auto loc = view.getLoc();
 322:       Value ptr = baseInfo->ptr;
 323:       auto ptrTy = triton::getPointerType(
 324:           getScratchStorageElementType(memTy.getElementType()));
 325:       if (ptr.getType() != ptrTy) {
 326:         ptr = tt::BitcastOp::create(rewriter, loc, ptrTy, ptr);
 327:       }
 328:       ptr = remapToScope(ptr, rewriter, scope, loc);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 330-332

```cpp
 330:       auto layout = getScratchEncoding(rewriter, memdesc, memTy);
 331:       auto tensorTy = RankedTensorType::get(memTy.getShape(),
 332:                                             memTy.getElementType(), layout);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 334-336

```cpp
 334:       ScratchInfo info{ptr, tensorTy};
 335:       return info;
 336:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 338-339

```cpp
 338:     return std::nullopt;
 339:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 341-345

```cpp
 341: private:
 342:   Value remapToScope(Value value, PatternRewriter &rewriter, Region *scope,
 343:                      Location loc) {
 344:     if (!scope || isValueAvailableInScope(value, scope))
 345:       return value;
```

- **EN:** Defines `remapToScope`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `remapToScope`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 347-351

```cpp
 347:     auto *parentOp = scope->getParentOp();
 348:     auto partitions = dyn_cast_or_null<ttg::WarpSpecializePartitionsOp>(
 349:         parentOp ? parentOp : nullptr);
 350:     if (!partitions)
 351:       return value;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 353-360

```cpp
 353:     unsigned captureIdx = partitions.getNumOperands();
 354:     for (auto [i, capture] :
 355:          llvm::enumerate(partitions.getExplicitCaptures())) {
 356:       if (capture == value) {
 357:         captureIdx = i;
 358:         break;
 359:       }
 360:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 362-367

```cpp
 362:     if (captureIdx == partitions.getNumOperands()) {
 363:       partitions->insertOperands(captureIdx, value);
 364:       for (Region &region : partitions.getPartitionRegions()) {
 365:         region.addArgument(value.getType(), loc);
 366:       }
 367:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 369-370

```cpp
 369:     return scope->getArgument(captureIdx);
 370:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 372-373

```cpp
 372:   DenseMap<Value, ScratchState> scratchMap;
 373: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 375-388

```cpp
 375: Value createScratchAndStore(PatternRewriter &rewriter, Location loc, Value val,
 376:                             RankedTensorType tensorTy) {
 377:   auto storageTy = getScratchStorageType(tensorTy);
 378:   int64_t elSize = tensorTy.getElementType().getIntOrFloatBitWidth() / 8;
 379:   int64_t alignment = std::max<int64_t>(elSize, 16);
 380:   int64_t sizeInBytes = product(tensorTy.getShape()) * elSize;
 381:   auto ptrTy = triton::getPointerType(storageTy.getElementType());
 382:   auto allocOp = createThirdPartyScratchAlloc(rewriter, loc, ptrTy, sizeInBytes,
 383:                                               alignment);
 384:   if (!storeFpSanScratchMemory(rewriter, loc, allocOp.getResult(), val,
 385:                                tensorTy))
 386:     return Value();
 387:   return allocOp.getResult();
 388: }
```

- **EN:** Defines helper `createScratchAndStore` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createScratchAndStore`，用于计算或构造外围变换所需的中间数据。
### Lines 390-407

```cpp
 390: Region *getScratchScopeRegion(Operation *anchor) {
 391:   Region *region = anchor->getParentRegion();
 392:   while (region) {
 393:     if (auto wsOp = dyn_cast<ttg::WarpSpecializeOp>(region->getParentOp())) {
 394:       if (region == &wsOp.getDefaultRegion()) {
 395:         region = wsOp->getParentRegion();
 396:         continue;
 397:       }
 398:       return region;
 399:     }
 400:     if (isa<ttg::WarpSpecializePartitionsOp>(region->getParentOp()))
 401:       return region;
 402:     if (isa<tt::FuncOp>(region->getParentOp()))
 403:       return region;
 404:     region = region->getParentRegion();
 405:   }
 406:   llvm::report_fatal_error("getScratchScopeRegion called on an op that is not "
 407:                            "contained in a function");
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 408-408

```cpp
 408: }
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 410-412

```cpp
 410: // ------------------------------------------------------------
 411: // Utility functions
 412: // ------------------------------------------------------------
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 414-417

```cpp
 414: LogicalResult emitFpSanUnsupported(Operation *op) {
 415:   op->emitOpError() << "unsupported by fpsan";
 416:   return failure();
 417: }
```

- **EN:** Defines `emitFpSanUnsupported`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `emitFpSanUnsupported`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 419-422

```cpp
 419: LogicalResult emitFpSanCodegenError(Operation *op) {
 420:   op->emitOpError() << "fpsan codegen error";
 421:   return failure();
 422: }
```

- **EN:** Defines `emitFpSanCodegenError`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `emitFpSanCodegenError`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 424-428

```cpp
 424: LogicalResult emitFpSanInvariantError(Operation *op) {
 425:   assert(false && "unexpected invalid IR in FpSanitizer");
 426:   op->emitOpError() << "fpsan invariant violation";
 427:   return failure();
 428: }
```

- **EN:** Defines `emitFpSanInvariantError`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `emitFpSanInvariantError`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 430-433

```cpp
 430: Type getIntTypeLike(Type ty) {
 431:   auto elem = dyn_cast<FloatType>(getElementTypeOrSelf(ty));
 432:   if (!elem)
 433:     return Type();
```

- **EN:** Defines accessor/helper `getIntTypeLike` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getIntTypeLike`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 435-444

```cpp
 435:   auto *ctx = ty.getContext();
 436:   auto intElem = IntegerType::get(ctx, elem.getWidth());
 437:   if (auto ranked = dyn_cast<RankedTensorType>(ty)) {
 438:     return RankedTensorType::get(ranked.getShape(), intElem,
 439:                                  ranked.getEncoding());
 440:   }
 441:   if (isa<FloatType>(ty))
 442:     return intElem;
 443:   llvm::report_fatal_error("expected FloatType or RankedTensorType");
 444: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 446-449

```cpp
 446: unsigned getIntBitwidth(Type ty) {
 447:   auto elem = cast<IntegerType>(getElementTypeOrSelf(ty));
 448:   return elem.getWidth();
 449: }
```

- **EN:** Defines accessor/helper `getIntBitwidth` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getIntBitwidth`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 451-453

```cpp
 451: Type getTypeWithElement(Type ty, Type elemTy) {
 452:   return cast<RankedTensorType>(ty).clone(elemTy);
 453: }
```

- **EN:** Defines accessor/helper `getTypeWithElement` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getTypeWithElement`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 455-470

```cpp
 455: Value getIntConstantLike(PatternRewriter &rewriter, Location loc, Type targetTy,
 456:                          int64_t value) {
 457:   if (auto shaped = dyn_cast<ShapedType>(targetTy)) {
 458:     auto elem = cast<IntegerType>(shaped.getElementType());
 459:     auto intAttr = IntegerAttr::get(
 460:         elem, APInt(elem.getWidth(), static_cast<uint64_t>(value),
 461:                     /*isSigned=*/true, /*implicitTrunc=*/true));
 462:     auto attr = DenseElementsAttr::get(shaped, intAttr);
 463:     return arith::ConstantOp::create(rewriter, loc, attr);
 464:   }
 465:   auto intTy = cast<IntegerType>(targetTy);
 466:   auto attr = IntegerAttr::get(
 467:       intTy, APInt(intTy.getWidth(), static_cast<uint64_t>(value),
 468:                    /*isSigned=*/true, /*implicitTrunc=*/true));
 469:   return arith::ConstantOp::create(rewriter, loc, attr);
 470: }
```

- **EN:** Defines accessor/helper `getIntConstantLike` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getIntConstantLike`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 472-487

```cpp
 472: Value getUIntConstantLike(PatternRewriter &rewriter, Location loc,
 473:                           Type targetTy, uint64_t value) {
 474:   if (auto shaped = dyn_cast<ShapedType>(targetTy)) {
 475:     auto elem = cast<IntegerType>(shaped.getElementType());
 476:     auto intAttr = IntegerAttr::get(elem, APInt(elem.getWidth(), value,
 477:                                                 /*isSigned=*/false,
 478:                                                 /*implicitTrunc=*/true));
 479:     auto attr = DenseElementsAttr::get(shaped, intAttr);
 480:     return arith::ConstantOp::create(rewriter, loc, attr);
 481:   }
 482:   auto intTy = cast<IntegerType>(targetTy);
 483:   auto attr = IntegerAttr::get(intTy, APInt(intTy.getWidth(), value,
 484:                                             /*isSigned=*/false,
 485:                                             /*implicitTrunc=*/true));
 486:   return arith::ConstantOp::create(rewriter, loc, attr);
 487: }
```

- **EN:** Defines accessor/helper `getUIntConstantLike` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getUIntConstantLike`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 489-492

```cpp
 489: Value getU32ConstantLike(PatternRewriter &rewriter, Location loc, Type targetTy,
 490:                          uint32_t value) {
 491:   return getUIntConstantLike(rewriter, loc, targetTy, value);
 492: }
```

- **EN:** Defines accessor/helper `getU32ConstantLike` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getU32ConstantLike`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 494-497

```cpp
 494: Value castSignedIntValueToType(PatternRewriter &rewriter, Location loc, Value v,
 495:                                Type targetTy) {
 496:   if (v.getType() == targetTy)
 497:     return v;
```

- **EN:** Defines `castSignedIntValueToType`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `castSignedIntValueToType`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 499-508

```cpp
 499:   unsigned srcWidth = getIntBitwidth(v.getType());
 500:   unsigned dstWidth = getIntBitwidth(targetTy);
 501:   if (dstWidth > srcWidth) {
 502:     return arith::ExtSIOp::create(rewriter, loc, targetTy, v);
 503:   }
 504:   if (srcWidth > dstWidth) {
 505:     return arith::TruncIOp::create(rewriter, loc, targetTy, v);
 506:   }
 507:   return v;
 508: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 510-518

```cpp
 510: Value castScalarIntToIntLike(PatternRewriter &rewriter, Location loc,
 511:                              Value scalar, Type targetTy) {
 512:   auto elemTy = cast<IntegerType>(getElementTypeOrSelf(targetTy));
 513:   if (scalar.getType() != elemTy)
 514:     scalar = castSignedIntValueToType(rewriter, loc, scalar, elemTy);
 515:   if (isa<ShapedType>(targetTy))
 516:     return tt::SplatOp::create(rewriter, loc, targetTy, scalar);
 517:   return scalar;
 518: }
```

- **EN:** Defines `castScalarIntToIntLike`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `castScalarIntToIntLike`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 520-525

```cpp
 520: uint64_t getLowBitsMask(unsigned bitWidth) {
 521:   assert(bitWidth > 0 && bitWidth <= 64);
 522:   if (bitWidth == 64)
 523:     return ~uint64_t{0};
 524:   return (uint64_t{1} << bitWidth) - 1;
 525: }
```

- **EN:** Defines accessor/helper `getLowBitsMask` that exposes or updates operation state in a compact, reusable way. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义访问器/辅助函数 `getLowBitsMask`，以紧凑且可复用的方式读取或更新操作状态。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 527-533

```cpp
 527: uint64_t invOddU64(uint64_t a) {
 528:   assert((a & 1) == 1);
 529:   uint64_t x = 2 - a;
 530:   for (unsigned correctBits = 2; correctBits < 64; correctBits *= 2)
 531:     x *= 2 - a * x;
 532:   return x;
 533: }
```

- **EN:** Defines `invOddU64`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `invOddU64`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 535-542

```cpp
 535: Value embedFloatBitsToInt(PatternRewriter &rewriter, Location loc,
 536:                           Value rawBits, FloatType floatElemTy) {
 537:   Type floatTy = floatElemTy;
 538:   if (auto ranked = dyn_cast<RankedTensorType>(rawBits.getType()))
 539:     floatTy = ranked.clone(floatElemTy);
 540:   Value rawFloat = tt::BitcastOp::create(rewriter, loc, floatTy, rawBits);
 541:   return embedToInt(rewriter, loc, rawFloat);
 542: }
```

- **EN:** Defines `embedFloatBitsToInt`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `embedFloatBitsToInt`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 544-551

```cpp
 544: uint64_t stableStringHash(StringRef str) {
 545:   uint64_t h = 14695981039346656037ull;
 546:   for (uint8_t c : str.bytes()) {
 547:     h ^= c;
 548:     h *= 1099511628211ull;
 549:   }
 550:   return h;
 551: }
```

- **EN:** Defines `stableStringHash`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `stableStringHash`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 553-560

```cpp
 553: uint64_t murmur64Mixer(uint64_t h) {
 554:   h ^= h >> 33;
 555:   h *= 0xff51afd7ed558ccd;
 556:   h ^= h >> 33;
 557:   h *= 0xc4ceb9fe1a85ec53;
 558:   h ^= h >> 33;
 559:   return h;
 560: }
```

- **EN:** Defines `murmur64Mixer`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `murmur64Mixer`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 562-562

```cpp
 562: constexpr uint32_t kUnaryTagMultiplier = 314159u;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 564-576

```cpp
 564: Value fpsanUnaryTagged(PatternRewriter &rewriter, Location loc, Value input,
 565:                        UnaryOpId opId) {
 566:   auto inI = embedToInt(rewriter, loc, input);
 567:   uint64_t opIdHash = murmur64Mixer(getUnaryOpId(opId));
 568:   auto opIdVal = getIntConstantLike(rewriter, loc, inI.getType(),
 569:                                     static_cast<int64_t>(opIdHash));
 570:   auto multiplier =
 571:       getU32ConstantLike(rewriter, loc, inI.getType(), kUnaryTagMultiplier);
 572:   auto mixedIn = arith::MulIOp::create(rewriter, loc, inI, multiplier);
 573:   auto tagged = arith::XOrIOp::create(rewriter, loc, mixedIn, opIdVal);
 574:   auto outI = arith::MulIOp::create(rewriter, loc, tagged, multiplier);
 575:   return unembedToFloat(rewriter, loc, outI, input.getType());
 576: }
```

- **EN:** Defines `fpsanUnaryTagged`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `fpsanUnaryTagged`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 578-581

```cpp
 578: Value fpsanIntInv(PatternRewriter &rewriter, Location loc, Value u) {
 579:   auto one = getU32ConstantLike(rewriter, loc, u.getType(), 1u);
 580:   auto two = getU32ConstantLike(rewriter, loc, u.getType(), 2u);
 581:   auto evenMask = getIntConstantLike(rewriter, loc, u.getType(), -2);
```

- **EN:** Defines `fpsanIntInv`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `fpsanIntInv`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 583-590

```cpp
 583:   Value a = arith::OrIOp::create(rewriter, loc, u, one);
 584:   Value x = arith::SubIOp::create(rewriter, loc, two, a);
 585:   for (unsigned correctBits = 2; correctBits < getIntBitwidth(u.getType());
 586:        correctBits *= 2) {
 587:     Value ax = arith::MulIOp::create(rewriter, loc, a, x);
 588:     Value factor = arith::SubIOp::create(rewriter, loc, two, ax);
 589:     x = arith::MulIOp::create(rewriter, loc, x, factor);
 590:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 592-595

```cpp
 592:   Value evenPart = arith::AndIOp::create(rewriter, loc, x, evenMask);
 593:   Value originalParity = arith::AndIOp::create(rewriter, loc, u, one);
 594:   return arith::OrIOp::create(rewriter, loc, evenPart, originalParity);
 595: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 597-603

```cpp
 597: Value fpsanFDiv(PatternRewriter &rewriter, Location loc, Value num, Value den) {
 598:   auto numI = embedToInt(rewriter, loc, num);
 599:   auto denI = embedToInt(rewriter, loc, den);
 600:   auto inv = fpsanIntInv(rewriter, loc, denI);
 601:   auto resI = arith::MulIOp::create(rewriter, loc, numI, inv);
 602:   return unembedToFloat(rewriter, loc, resI, num.getType());
 603: }
```

- **EN:** Defines `fpsanFDiv`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `fpsanFDiv`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 605-612

```cpp
 605: Value fpsanSRem(PatternRewriter &rewriter, Location loc, Value num, Value den) {
 606:   auto numI = embedToInt(rewriter, loc, num);
 607:   auto denI = embedToInt(rewriter, loc, den);
 608:   auto one = getIntConstantLike(rewriter, loc, denI.getType(), 1);
 609:   auto denSafe = arith::OrIOp::create(rewriter, loc, denI, one);
 610:   auto resI = arith::RemSIOp::create(rewriter, loc, numI, denSafe);
 611:   return unembedToFloat(rewriter, loc, resI, num.getType());
 612: }
```

- **EN:** Defines `fpsanSRem`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `fpsanSRem`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 614-621

```cpp
 614: // Modular exponentiation in payload space; this preserves
 615: // exp2(a + b) = exp2(a) * exp2(b) under the integer rewrite.
 616: Value fpsanExp2FromInt(PatternRewriter &rewriter, Location loc, Value xI,
 617:                        Type floatTy) {
 618:   unsigned bitWidth = getIntBitwidth(xI.getType());
 619:   auto one = getIntConstantLike(rewriter, loc, xI.getType(), 1);
 620:   auto zero = getIntConstantLike(rewriter, loc, xI.getType(), 0);
 621:   auto c = getIntConstantLike(rewriter, loc, xI.getType(), 0xa343836d);
```

- **EN:** Defines `fpsanExp2FromInt`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `fpsanExp2FromInt`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 623-632

```cpp
 623:   auto lower =
 624:       arith::ConstantOp::create(rewriter, loc, rewriter.getI32IntegerAttr(0));
 625:   auto upper = arith::ConstantOp::create(rewriter, loc,
 626:                                          rewriter.getI32IntegerAttr(bitWidth));
 627:   auto step =
 628:       arith::ConstantOp::create(rewriter, loc, rewriter.getI32IntegerAttr(1));
 629:   auto topBit = arith::ConstantOp::create(
 630:       rewriter, loc, rewriter.getI32IntegerAttr(bitWidth - 1));
 631:   auto loop = scf::ForOp::create(rewriter, loc, lower, upper, step, one);
 632:   rewriter.setInsertionPointToStart(loop.getBody());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 634-647

```cpp
 634:   Value i = loop.getInductionVar();
 635:   Value y = loop.getRegionIterArgs()[0];
 636:   y = arith::MulIOp::create(rewriter, loc, y, y);
 637:   Value bitIndex =
 638:       arith::SubIOp::create(rewriter, loc, rewriter.getI32Type(), topBit, i);
 639:   Value shift = castScalarIntToIntLike(rewriter, loc, bitIndex, xI.getType());
 640:   Value bit = arith::ShLIOp::create(rewriter, loc, one, shift);
 641:   auto masked = arith::AndIOp::create(rewriter, loc, xI, bit);
 642:   auto isZero = arith::CmpIOp::create(rewriter, loc, arith::CmpIPredicate::eq,
 643:                                       masked, zero);
 644:   auto factor = arith::SelectOp::create(rewriter, loc, isZero, one, c);
 645:   y = arith::MulIOp::create(rewriter, loc, y, factor);
 646:   scf::YieldOp::create(rewriter, loc, y);
 647:   rewriter.setInsertionPointAfter(loop);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 649-650

```cpp
 649:   return unembedToFloat(rewriter, loc, loop.getResult(0), floatTy);
 650: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 652-658

```cpp
 652: Value fpsanExp2(PatternRewriter &rewriter, Location loc, Value input) {
 653:   auto elemTy = dyn_cast<FloatType>(getElementTypeOrSelf(input.getType()));
 654:   if (!elemTy)
 655:     return Value();
 656:   return fpsanExp2FromInt(rewriter, loc, embedToInt(rewriter, loc, input),
 657:                           input.getType());
 658: }
```

- **EN:** Defines `fpsanExp2`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `fpsanExp2`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 660-663

```cpp
 660: Value fpsanExp(PatternRewriter &rewriter, Location loc, Value input) {
 661:   auto elemTy = dyn_cast<FloatType>(getElementTypeOrSelf(input.getType()));
 662:   if (!elemTy)
 663:     return Value();
```

- **EN:** Defines `fpsanExp`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `fpsanExp`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 665-670

```cpp
 665:   auto inputI = embedToInt(rewriter, loc, input);
 666:   auto rcpLog2 =
 667:       getU32ConstantLike(rewriter, loc, inputI.getType(), 0x236ee9bfu);
 668:   auto scaledI = arith::MulIOp::create(rewriter, loc, inputI, rcpLog2);
 669:   return fpsanExp2FromInt(rewriter, loc, scaledI, input.getType());
 670: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 672-675

```cpp
 672: struct FpSanCosSin {
 673:   Value cos;
 674:   Value sin;
 675: };
```

- **EN:** Defines `FpSanCosSin`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `FpSanCosSin`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 677-684

```cpp
 677: FpSanCosSin fpsanCosSinPayload(PatternRewriter &rewriter, Location loc,
 678:                                Value xI) {
 679:   Type intTy = xI.getType();
 680:   unsigned bitWidth = getIntBitwidth(intTy);
 681:   uint64_t mask = getLowBitsMask(bitWidth);
 682:   uint64_t rcp5 = invOddU64(5) & mask;
 683:   uint64_t aValue = (uint64_t{0} - ((uint64_t{3} * rcp5) & mask)) & mask;
 684:   uint64_t bValue = (uint64_t{4} * rcp5) & mask;
```

- **EN:** Defines `fpsanCosSinPayload`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `fpsanCosSinPayload`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 686-690

```cpp
 686:   auto zero = getUIntConstantLike(rewriter, loc, intTy, 0);
 687:   auto one = getUIntConstantLike(rewriter, loc, intTy, 1);
 688:   auto two = getUIntConstantLike(rewriter, loc, intTy, 2);
 689:   auto a = getUIntConstantLike(rewriter, loc, intTy, aValue);
 690:   auto b = getUIntConstantLike(rewriter, loc, intTy, bValue);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 692-702

```cpp
 692:   auto lower =
 693:       arith::ConstantOp::create(rewriter, loc, rewriter.getI32IntegerAttr(0));
 694:   auto upper = arith::ConstantOp::create(rewriter, loc,
 695:                                          rewriter.getI32IntegerAttr(bitWidth));
 696:   auto step =
 697:       arith::ConstantOp::create(rewriter, loc, rewriter.getI32IntegerAttr(1));
 698:   auto topBit = arith::ConstantOp::create(
 699:       rewriter, loc, rewriter.getI32IntegerAttr(bitWidth - 1));
 700:   SmallVector<Value> initArgs{one, zero};
 701:   auto loop = scf::ForOp::create(rewriter, loc, lower, upper, step, initArgs);
 702:   rewriter.setInsertionPointToStart(loop.getBody());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 704-711

```cpp
 704:   Value bit = loop.getInductionVar();
 705:   Value c = loop.getRegionIterArgs()[0];
 706:   Value s = loop.getRegionIterArgs()[1];
 707:   Value cc = arith::MulIOp::create(rewriter, loc, c, c);
 708:   Value ss = arith::MulIOp::create(rewriter, loc, s, s);
 709:   Value cDouble = arith::SubIOp::create(rewriter, loc, cc, ss);
 710:   Value cs = arith::MulIOp::create(rewriter, loc, c, s);
 711:   Value sDouble = arith::MulIOp::create(rewriter, loc, two, cs);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 713-718

```cpp
 713:   Value ac = arith::MulIOp::create(rewriter, loc, a, cDouble);
 714:   Value bs = arith::MulIOp::create(rewriter, loc, b, sDouble);
 715:   Value cInc = arith::SubIOp::create(rewriter, loc, ac, bs);
 716:   Value as = arith::MulIOp::create(rewriter, loc, a, sDouble);
 717:   Value bc = arith::MulIOp::create(rewriter, loc, b, cDouble);
 718:   Value sInc = arith::AddIOp::create(rewriter, loc, as, bc);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 720-730

```cpp
 720:   Value bitIndex =
 721:       arith::SubIOp::create(rewriter, loc, rewriter.getI32Type(), topBit, bit);
 722:   Value shift = castScalarIntToIntLike(rewriter, loc, bitIndex, intTy);
 723:   Value bitMask = arith::ShLIOp::create(rewriter, loc, one, shift);
 724:   auto masked = arith::AndIOp::create(rewriter, loc, xI, bitMask);
 725:   auto isZero = arith::CmpIOp::create(rewriter, loc, arith::CmpIPredicate::eq,
 726:                                       masked, zero);
 727:   c = arith::SelectOp::create(rewriter, loc, isZero, cDouble, cInc);
 728:   s = arith::SelectOp::create(rewriter, loc, isZero, sDouble, sInc);
 729:   scf::YieldOp::create(rewriter, loc, ValueRange{c, s});
 730:   rewriter.setInsertionPointAfter(loop);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 732-733

```cpp
 732:   return {loop.getResult(0), loop.getResult(1)};
 733: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 735-741

```cpp
 735: Value fpsanCos(PatternRewriter &rewriter, Location loc, Value input) {
 736:   if (!isFloatLike(input.getType()))
 737:     return Value();
 738:   auto cosSin =
 739:       fpsanCosSinPayload(rewriter, loc, embedToInt(rewriter, loc, input));
 740:   return unembedToFloat(rewriter, loc, cosSin.cos, input.getType());
 741: }
```

- **EN:** Defines `fpsanCos`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `fpsanCos`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 743-749

```cpp
 743: Value fpsanSin(PatternRewriter &rewriter, Location loc, Value input) {
 744:   if (!isFloatLike(input.getType()))
 745:     return Value();
 746:   auto cosSin =
 747:       fpsanCosSinPayload(rewriter, loc, embedToInt(rewriter, loc, input));
 748:   return unembedToFloat(rewriter, loc, cosSin.sin, input.getType());
 749: }
```

- **EN:** Defines `fpsanSin`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `fpsanSin`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 751-755

```cpp
 751: bool externHasNumericOperands(tt::ExternElementwiseOp op) {
 752:   return llvm::all_of(op.getOperands(), [](Value operand) {
 753:     return isNumericLike(operand.getType());
 754:   });
 755: }
```

- **EN:** Defines `externHasNumericOperands`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `externHasNumericOperands`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 757-767

```cpp
 757: Value castExternOperandToResultInt(PatternRewriter &rewriter, Location loc,
 758:                                    Value operand, Type resultIntTy) {
 759:   if (isFloatLike(operand.getType())) {
 760:     return castSignedIntValueToType(
 761:         rewriter, loc, embedToInt(rewriter, loc, operand), resultIntTy);
 762:   }
 763:   if (isIntLike(operand.getType())) {
 764:     return castSignedIntValueToType(rewriter, loc, operand, resultIntTy);
 765:   }
 766:   return Value();
 767: }
```

- **EN:** Defines `castExternOperandToResultInt`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `castExternOperandToResultInt`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 769-776

```cpp
 769: Value rotateLeftIntByAmount(PatternRewriter &rewriter, Location loc,
 770:                             Value value, unsigned amount) {
 771:   unsigned bitWidth = getIntBitwidth(value.getType());
 772:   if (bitWidth == 0)
 773:     return value;
 774:   amount %= bitWidth;
 775:   if (amount == 0)
 776:     return value;
```

- **EN:** Defines `rotateLeftIntByAmount`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `rotateLeftIntByAmount`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 778-785

```cpp
 778:   auto leftShift = getIntConstantLike(rewriter, loc, value.getType(),
 779:                                       static_cast<int64_t>(amount));
 780:   auto rightShift = getIntConstantLike(rewriter, loc, value.getType(),
 781:                                        static_cast<int64_t>(bitWidth - amount));
 782:   auto left = arith::ShLIOp::create(rewriter, loc, value, leftShift);
 783:   auto right = arith::ShRUIOp::create(rewriter, loc, value, rightShift);
 784:   return arith::OrIOp::create(rewriter, loc, left, right);
 785: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 787-790

```cpp
 787: Value fpsanVariadicExternTagged(PatternRewriter &rewriter, Location loc,
 788:                                 tt::ExternElementwiseOp op, uint64_t hash) {
 789:   Type resultTy = op.getType();
 790:   Type resultIntTy = getIntTypeLike(resultTy);
```

- **EN:** Defines `fpsanVariadicExternTagged`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `fpsanVariadicExternTagged`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 792-801

```cpp
 792:   Value sumI = getIntConstantLike(rewriter, loc, resultIntTy, 0);
 793:   for (auto [argIdx, operand] : llvm::enumerate(op.getOperands())) {
 794:     Value operandI =
 795:         castExternOperandToResultInt(rewriter, loc, operand, resultIntTy);
 796:     if (!operandI)
 797:       return Value();
 798:     Value rotated = rotateLeftIntByAmount(rewriter, loc, operandI,
 799:                                           static_cast<unsigned>(argIdx));
 800:     sumI = arith::AddIOp::create(rewriter, loc, sumI, rotated);
 801:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 803-807

```cpp
 803:   auto hashVal = getIntConstantLike(rewriter, loc, resultIntTy,
 804:                                     static_cast<int64_t>(hash));
 805:   auto outI = arith::XOrIOp::create(rewriter, loc, sumI, hashVal);
 806:   return unembedToFloat(rewriter, loc, outI, resultTy);
 807: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 809-823

```cpp
 809: std::optional<ScratchInfo>
 810: createOperandScratch(PatternRewriter &rewriter, Location loc,
 811:                      TmemScratchManager &scratch, Value memdesc,
 812:                      ttg::MemDescType memTy, bool isTmem, Region *scope) {
 813:   auto layout = scratch.getScratchEncoding(rewriter, memdesc, memTy);
 814:   auto tensorTy =
 815:       RankedTensorType::get(memTy.getShape(), memTy.getElementType(), layout);
 816:   Value fullVal;
 817:   if (isTmem) {
 818:     auto info = scratch.getOrCreate(memdesc, rewriter, scope);
 819:     if (!info)
 820:       return std::nullopt;
 821:     fullVal = loadFpSanScratchMemory(rewriter, loc, info->ptr, tensorTy);
 822:     if (!fullVal)
 823:       return std::nullopt;
```

- **EN:** Defines helper `createOperandScratch` that computes or constructs intermediate data used by the surrounding transformation. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic.
- **CN:** 这里定义辅助函数 `createOperandScratch`，用于计算或构造外围变换所需的中间数据。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。
### Lines 824-840

```cpp
 824:   } else {
 825:     fullVal =
 826:         ttg::LocalLoadOp::create(rewriter, loc, tensorTy, memdesc, Value())
 827:             .getResult();
 828:   }
 829:   int64_t elSize = memTy.getElementType().getIntOrFloatBitWidth() / 8;
 830:   int64_t alignment = std::max<int64_t>(elSize, 16);
 831:   int64_t sizeInBytes = product(memTy.getShape()) * elSize;
 832:   auto ptrTy = triton::getPointerType(
 833:       getScratchStorageElementType(memTy.getElementType()));
 834:   auto allocOp = createThirdPartyScratchAlloc(rewriter, loc, ptrTy, sizeInBytes,
 835:                                               alignment);
 836:   Value ptr = allocOp.getResult();
 837:   if (!storeFpSanScratchMemory(rewriter, loc, ptr, fullVal, tensorTy))
 838:     return std::nullopt;
 839:   return ScratchInfo{ptr, tensorTy};
 840: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 842-856

```cpp
 842: std::optional<ScratchInfo> createWGMMAScratch(PatternRewriter &rewriter,
 843:                                               Location loc, Value operand) {
 844:   if (auto memTy = dyn_cast<ttg::MemDescType>(operand.getType())) {
 845:     auto layout = getOptimizedBlockedEncoding(rewriter, memTy.getShape(),
 846:                                               memTy.getElementType());
 847:     auto tensorTy =
 848:         RankedTensorType::get(memTy.getShape(), memTy.getElementType(), layout);
 849:     Value fullVal =
 850:         ttg::LocalLoadOp::create(rewriter, loc, tensorTy, operand, Value())
 851:             .getResult();
 852:     Value ptr = createScratchAndStore(rewriter, loc, fullVal, tensorTy);
 853:     if (!ptr)
 854:       return std::nullopt;
 855:     return ScratchInfo{ptr, tensorTy};
 856:   }
```

- **EN:** Defines helper `createWGMMAScratch` that computes or constructs intermediate data used by the surrounding transformation. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic.
- **CN:** 这里定义辅助函数 `createWGMMAScratch`，用于计算或构造外围变换所需的中间数据。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。
### Lines 858-865

```cpp
 858:   auto tensorTy = dyn_cast<RankedTensorType>(operand.getType());
 859:   if (!tensorTy)
 860:     return std::nullopt;
 861:   Value ptr = createScratchAndStore(rewriter, loc, operand, tensorTy);
 862:   if (!ptr)
 863:     return std::nullopt;
 864:   return ScratchInfo{ptr, tensorTy};
 865: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 867-870

```cpp
 867: Value createAsyncToken(PatternRewriter &rewriter, Location loc,
 868:                        ValueRange deps) {
 869:   return ttg::AsyncCommitGroupOp::create(rewriter, loc, deps).getResult();
 870: }
```

- **EN:** Defines helper `createAsyncToken` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createAsyncToken`，用于计算或构造外围变换所需的中间数据。
### Lines 872-882

```cpp
 872: Value expandAllSlicedDims(PatternRewriter &rewriter, Location loc,
 873:                           Value tensor) {
 874:   auto type = cast<RankedTensorType>(tensor.getType());
 875:   auto sliceEncoding = dyn_cast<ttg::SliceEncodingAttr>(type.getEncoding());
 876:   while (sliceEncoding) {
 877:     tensor = expandOuterSlicedDim(rewriter, loc, tensor);
 878:     type = cast<RankedTensorType>(tensor.getType());
 879:     sliceEncoding = dyn_cast<ttg::SliceEncodingAttr>(type.getEncoding());
 880:   }
 881:   return tensor;
 882: }
```

- **EN:** Defines `expandAllSlicedDims`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `expandAllSlicedDims`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 884-893

```cpp
 884: Value createPointerTensorStrided2D(PatternRewriter &rewriter, Location loc,
 885:                                    Value base, RankedTensorType resultTy,
 886:                                    int64_t stride0, int64_t stride1) {
 887:   auto shape = resultTy.getShape();
 888:   auto encoding = cast<ttg::DistributedEncodingTrait>(resultTy.getEncoding());
 889:   auto ptrTy = base.getType();
 890:   auto ptrTensorTy = RankedTensorType::get(shape, ptrTy, encoding);
 891:   Value ptrTensor = tt::SplatOp::create(rewriter, loc, ptrTensorTy, base);
 892:   auto i32Ty = rewriter.getI32Type();
 893:   auto offsetsTy = RankedTensorType::get(shape, i32Ty, encoding);
```

- **EN:** Defines helper `createPointerTensorStrided2D` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createPointerTensorStrided2D`，用于计算或构造外围变换所需的中间数据。
### Lines 895-906

```cpp
 895:   auto dim0Enc = getSingleDimSliceEncoding(encoding, 0);
 896:   auto dim0Ty = RankedTensorType::get({shape[0]}, i32Ty, dim0Enc);
 897:   auto range0 = tt::MakeRangeOp::create(rewriter, loc, dim0Ty, 0, shape[0]);
 898:   auto stride0Const = createConstIntTensor(rewriter, loc, stride0, dim0Ty);
 899:   auto off0 =
 900:       arith::MulIOp::create(rewriter, loc, dim0Ty, range0, stride0Const);
 901:   auto off0Exp = expandAllSlicedDims(rewriter, loc, off0);
 902:   if (cast<RankedTensorType>(off0Exp.getType()).getShape() != shape) {
 903:     off0Exp = tt::BroadcastOp::create(rewriter, loc, offsetsTy, off0Exp);
 904:   }
 905:   ptrTensor =
 906:       tt::AddPtrOp::create(rewriter, loc, ptrTensorTy, ptrTensor, off0Exp);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 908-919

```cpp
 908:   auto dim1Enc = getSingleDimSliceEncoding(encoding, 1);
 909:   auto dim1Ty = RankedTensorType::get({shape[1]}, i32Ty, dim1Enc);
 910:   auto range1 = tt::MakeRangeOp::create(rewriter, loc, dim1Ty, 0, shape[1]);
 911:   auto stride1Const = createConstIntTensor(rewriter, loc, stride1, dim1Ty);
 912:   auto off1 =
 913:       arith::MulIOp::create(rewriter, loc, dim1Ty, range1, stride1Const);
 914:   auto off1Exp = expandAllSlicedDims(rewriter, loc, off1);
 915:   if (cast<RankedTensorType>(off1Exp.getType()).getShape() != shape) {
 916:     off1Exp = tt::BroadcastOp::create(rewriter, loc, offsetsTy, off1Exp);
 917:   }
 918:   ptrTensor =
 919:       tt::AddPtrOp::create(rewriter, loc, ptrTensorTy, ptrTensor, off1Exp);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 921-922

```cpp
 921:   return ptrTensor;
 922: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 924-936

```cpp
 924: Value loadScratchStrided2D(PatternRewriter &rewriter, Location loc, Value base,
 925:                            RankedTensorType tensorTy, int64_t stride0,
 926:                            int64_t stride1) {
 927:   auto storageTy = getScratchStorageType(tensorTy);
 928:   auto ptrTensor = createPointerTensorStrided2D(rewriter, loc, base, storageTy,
 929:                                                 stride0, stride1);
 930:   Value stored =
 931:       tt::LoadOp::create(rewriter, loc, ptrTensor, CacheModifier::NONE,
 932:                          EvictionPolicy::NORMAL, false);
 933:   if (isFloatLike(tensorTy))
 934:     return unembedToFloat(rewriter, loc, stored, tensorTy);
 935:   return stored;
 936: }
```

- **EN:** Defines `loadScratchStrided2D`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `loadScratchStrided2D`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 938-942

```cpp
 938: Value loadScratchStrided2D(PatternRewriter &rewriter, Location loc, Value base,
 939:                            RankedTensorType tensorTy, int64_t stride1) {
 940:   return loadScratchStrided2D(rewriter, loc, base, tensorTy, /*stride0=*/1,
 941:                               stride1);
 942: }
```

- **EN:** Defines `loadScratchStrided2D`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `loadScratchStrided2D`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 944-956

```cpp
 944: Operation *storeScratchStrided2D(PatternRewriter &rewriter, Location loc,
 945:                                  Value base, Value tensor,
 946:                                  RankedTensorType tensorTy, int64_t stride0,
 947:                                  int64_t stride1) {
 948:   auto storageTy = getScratchStorageType(tensorTy);
 949:   auto ptrTensor = createPointerTensorStrided2D(rewriter, loc, base, storageTy,
 950:                                                 stride0, stride1);
 951:   Value stored = tensor;
 952:   if (isFloatLike(tensorTy))
 953:     stored = embedToInt(rewriter, loc, tensor);
 954:   return tt::StoreOp::create(rewriter, loc, ptrTensor, stored,
 955:                              CacheModifier::NONE, EvictionPolicy::NORMAL);
 956: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 958-961

```cpp
 958: Value unpackPackedFp4Slice(PatternRewriter &rewriter, Location loc,
 959:                            Value packedSlice, Value kI32) {
 960:   Value packedI = embedToInt(rewriter, loc, packedSlice);
 961:   auto intTy = packedI.getType();
```

- **EN:** Defines `unpackPackedFp4Slice`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `unpackPackedFp4Slice`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 963-971

```cpp
 963:   Value one =
 964:       arith::ConstantOp::create(rewriter, loc, rewriter.getI32IntegerAttr(1));
 965:   Value four =
 966:       arith::ConstantOp::create(rewriter, loc, rewriter.getI32IntegerAttr(4));
 967:   Value isOdd = arith::AndIOp::create(rewriter, loc, kI32, one);
 968:   Value shiftI32 = arith::MulIOp::create(rewriter, loc, isOdd, four);
 969:   Value shiftI8 =
 970:       arith::TruncIOp::create(rewriter, loc, rewriter.getI8Type(), shiftI32);
 971:   Value shiftTensor = tt::SplatOp::create(rewriter, loc, intTy, shiftI8);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 973-976

```cpp
 973:   Value shifted = arith::ShRUIOp::create(rewriter, loc, packedI, shiftTensor);
 974:   Value mask = getIntConstantLike(rewriter, loc, intTy, 0x0F);
 975:   return arith::AndIOp::create(rewriter, loc, shifted, mask);
 976: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 978-985

```cpp
 978: FloatType getDotScaledComputeFloatType(PatternRewriter &rewriter,
 979:                                        tt::ScaleDotElemType aElemType,
 980:                                        tt::ScaleDotElemType bElemType) {
 981:   if (aElemType == tt::ScaleDotElemType::FP16 ||
 982:       bElemType == tt::ScaleDotElemType::FP16)
 983:     return Float16Type::get(rewriter.getContext());
 984:   return BFloat16Type::get(rewriter.getContext());
 985: }
```

- **EN:** Defines accessor/helper `getDotScaledComputeFloatType` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getDotScaledComputeFloatType`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 987-1002

```cpp
 987: FloatType getDotScaledStorageFloatType(PatternRewriter &rewriter,
 988:                                        tt::ScaleDotElemType elemType) {
 989:   MLIRContext *ctx = rewriter.getContext();
 990:   switch (elemType) {
 991:   case tt::ScaleDotElemType::E4M3:
 992:     return Float8E4M3FNType::get(ctx);
 993:   case tt::ScaleDotElemType::E5M2:
 994:     return Float8E5M2Type::get(ctx);
 995:   case tt::ScaleDotElemType::BF16:
 996:     return BFloat16Type::get(ctx);
 997:   case tt::ScaleDotElemType::FP16:
 998:     return Float16Type::get(ctx);
 999:   default:
1000:     return {};
1001:   }
1002: }
```

- **EN:** Defines accessor/helper `getDotScaledStorageFloatType` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getDotScaledStorageFloatType`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 1004-1010

```cpp
1004: Value castDotScaledOperandToComputePayload(PatternRewriter &rewriter,
1005:                                            Location loc, Value slice,
1006:                                            tt::ScaleDotElemType elemType,
1007:                                            FloatType computeElem) {
1008:   Type computeIntTy = getTypeWithElement(
1009:       slice.getType(), IntegerType::get(rewriter.getContext(),
1010:                                         computeElem.getIntOrFloatBitWidth()));
```

- **EN:** Defines `castDotScaledOperandToComputePayload`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `castDotScaledOperandToComputePayload`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1012-1025

```cpp
1012:   if (auto storageFloat = getDotScaledStorageFloatType(rewriter, elemType)) {
1013:     Value payload;
1014:     if (isFloatLike(slice.getType())) {
1015:       payload = embedToInt(rewriter, loc, slice);
1016:     } else {
1017:       Value raw = castSignedIntValueToType(
1018:           rewriter, loc, slice,
1019:           getTypeWithElement(slice.getType(),
1020:                              IntegerType::get(rewriter.getContext(),
1021:                                               storageFloat.getWidth())));
1022:       payload = embedFloatBitsToInt(rewriter, loc, raw, storageFloat);
1023:     }
1024:     return castSignedIntValueToType(rewriter, loc, payload, computeIntTy);
1025:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1027-1032

```cpp
1027:   // Match ttg.fp4_to_fp sanitization: unpacked e2m1 nibbles are payloads in
1028:   // the destination floating type.  The 6-bit formats are not packed here, but
1029:   // use the same payload-preserving integer cast until we add a float6 mixer.
1030:   Value rawPayload = embedToInt(rewriter, loc, slice);
1031:   return castSignedIntValueToType(rewriter, loc, rawPayload, computeIntTy);
1032: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1034-1042

```cpp
1034: Value scaleI8ToF32Payload(PatternRewriter &rewriter, Location loc,
1035:                           Value scaleI) {
1036:   auto i32Elem = rewriter.getI32Type();
1037:   auto i32Ty = getTypeWithElement(scaleI.getType(), i32Elem);
1038:   Value scaleI32 = arith::ExtUIOp::create(rewriter, loc, i32Ty, scaleI);
1039:   auto shift = getUIntConstantLike(rewriter, loc, i32Ty, 23);
1040:   Value rawF32 = arith::ShLIOp::create(rewriter, loc, scaleI32, shift);
1041:   return embedFloatBitsToInt(rewriter, loc, rawF32, rewriter.getF32Type());
1042: }
```

- **EN:** Defines `scaleI8ToF32Payload`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `scaleI8ToF32Payload`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1044-1048

```cpp
1044: Value scaleI8ToComputePayload(PatternRewriter &rewriter, Location loc,
1045:                               Value scaleI, FloatType computeElem) {
1046:   unsigned computeWidth = computeElem.getIntOrFloatBitWidth();
1047:   Type computeIntTy = getTypeWithElement(
1048:       scaleI.getType(), IntegerType::get(rewriter.getContext(), computeWidth));
```

- **EN:** Defines `scaleI8ToComputePayload`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `scaleI8ToComputePayload`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1050-1055

```cpp
1050:   if (computeElem == rewriter.getF16Type()) {
1051:     // The real decomposition builds an f32 E8M0 scale and truncates it to f16.
1052:     // Under FPSan, truncf means mix-f32, signed-truncate, unmix-f16.
1053:     Value payloadF32 = scaleI8ToF32Payload(rewriter, loc, scaleI);
1054:     return castSignedIntValueToType(rewriter, loc, payloadF32, computeIntTy);
1055:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1057-1063

```cpp
1057:   Value scaleComputeI =
1058:       arith::ExtUIOp::create(rewriter, loc, computeIntTy, scaleI);
1059:   unsigned shiftValue = computeElem.getFPMantissaWidth() - 1;
1060:   auto shift = getUIntConstantLike(rewriter, loc, computeIntTy, shiftValue);
1061:   Value rawCompute = arith::ShLIOp::create(rewriter, loc, scaleComputeI, shift);
1062:   return embedFloatBitsToInt(rewriter, loc, rawCompute, computeElem);
1063: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1065-1078

```cpp
1065: Value castDotScaledScaleToComputePayload(PatternRewriter &rewriter,
1066:                                          Location loc, Value scaleSlice,
1067:                                          FloatType computeElem) {
1068:   Type computeIntTy =
1069:       getTypeWithElement(scaleSlice.getType(),
1070:                          IntegerType::get(rewriter.getContext(),
1071:                                           computeElem.getIntOrFloatBitWidth()));
1072:   if (isFloatLike(scaleSlice.getType())) {
1073:     Value payload = embedToInt(rewriter, loc, scaleSlice);
1074:     return castSignedIntValueToType(rewriter, loc, payload, computeIntTy);
1075:   }
1076:   return scaleI8ToComputePayload(
1077:       rewriter, loc, embedToInt(rewriter, loc, scaleSlice), computeElem);
1078: }
```

- **EN:** Defines `castDotScaledScaleToComputePayload`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `castDotScaledScaleToComputePayload`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1080-1094

```cpp
1080: struct DotScaleConfig {
1081:   Value aScalePtr;
1082:   Value bScalePtr;
1083:   RankedTensorType aScaleTileTy;
1084:   RankedTensorType bScaleTileTy;
1085:   int64_t aScaleStride = 0;
1086:   int64_t bScaleStride = 0;
1087:   int64_t aKPackFactor = 1;
1088:   int64_t bKPackFactor = 1;
1089:   int64_t aScaleFactor = 0;
1090:   int64_t bScaleFactor = 0;
1091:   tt::ScaleDotElemType aElemType;
1092:   tt::ScaleDotElemType bElemType;
1093:   FloatType computeElem;
1094: };
```

- **EN:** Defines `DotScaleConfig`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `DotScaleConfig`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1096-1113

```cpp
1096: Value loadScaleSlice(PatternRewriter &rewriter, Location loc, bool isLhs,
1097:                      const DotScaleConfig &scale, Value tileIdx, Value kI32) {
1098:   Value ptr = isLhs ? scale.aScalePtr : scale.bScalePtr;
1099:   int64_t sFactor = isLhs ? scale.aScaleFactor : scale.bScaleFactor;
1100:   int64_t sStride = isLhs ? scale.aScaleStride : scale.bScaleStride;
1101:   int64_t loadStride = isLhs ? scale.aScaleStride : 1;
1102:   auto tileTy = isLhs ? scale.aScaleTileTy : scale.bScaleTileTy;
1103:   Value tilePtr =
1104:       tt::AddPtrOp::create(rewriter, loc, ptr.getType(), ptr, tileIdx);
1105:   Value sFactorConst = arith::ConstantOp::create(
1106:       rewriter, loc, rewriter.getI32IntegerAttr(sFactor));
1107:   Value kGrp = arith::DivUIOp::create(rewriter, loc, kI32, sFactorConst);
1108:   Value sStrideConst = arith::ConstantOp::create(
1109:       rewriter, loc, rewriter.getI32IntegerAttr(sStride));
1110:   Value offset = arith::MulIOp::create(rewriter, loc, kGrp, sStrideConst);
1111:   Value slicePtr =
1112:       tt::AddPtrOp::create(rewriter, loc, ptr.getType(), tilePtr, offset);
1113:   return loadScratchStrided2D(rewriter, loc, slicePtr, tileTy, loadStride);
```

- **EN:** Defines `loadScaleSlice`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `loadScaleSlice`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1114-1114

```cpp
1114: }
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 1116-1122

```cpp
1116: Value emulateDotStep(PatternRewriter &rewriter, Location loc, Value aSlice,
1117:                      Value bSlice, Value aScaleSlice, Value bScaleSlice,
1118:                      int64_t m, int64_t n,
1119:                      ttg::DistributedEncodingTrait accLayout,
1120:                      IntegerType accElem, const DotScaleConfig &scale = {}) {
1121:   OpBuilder::InsertionGuard guard(rewriter);
1122:   auto fullTy = RankedTensorType::get({m, n}, accElem, accLayout);
```

- **EN:** Defines `emulateDotStep`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `emulateDotStep`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1124-1140

```cpp
1124:   Value aI;
1125:   Value bI;
1126:   if (scale.computeElem) {
1127:     aI = castDotScaledOperandToComputePayload(
1128:         rewriter, loc, aSlice, scale.aElemType, scale.computeElem);
1129:     bI = castDotScaledOperandToComputePayload(
1130:         rewriter, loc, bSlice, scale.bElemType, scale.computeElem);
1131:     if (aScaleSlice) {
1132:       auto aScaleI = castDotScaledScaleToComputePayload(
1133:           rewriter, loc, aScaleSlice, scale.computeElem);
1134:       aI = arith::MulIOp::create(rewriter, loc, aI, aScaleI);
1135:     }
1136:     if (bScaleSlice) {
1137:       auto bScaleI = castDotScaledScaleToComputePayload(
1138:           rewriter, loc, bScaleSlice, scale.computeElem);
1139:       bI = arith::MulIOp::create(rewriter, loc, bI, bScaleI);
1140:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1141-1152

```cpp
1141:   } else {
1142:     aI = embedToInt(rewriter, loc, aSlice);
1143:     bI = embedToInt(rewriter, loc, bSlice);
1144:   }
1145:   aI = castSignedIntValueToType(rewriter, loc, aI,
1146:                                 getTypeWithElement(aI.getType(), accElem));
1147:   bI = castSignedIntValueToType(rewriter, loc, bI,
1148:                                 getTypeWithElement(bI.getType(), accElem));
1149:   Value aFull = tt::BroadcastOp::create(rewriter, loc, fullTy, aI);
1150:   Value bFull = tt::BroadcastOp::create(rewriter, loc, fullTy, bI);
1151:   return arith::MulIOp::create(rewriter, loc, aFull, bFull);
1152: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1154-1163

```cpp
1154: std::optional<scf::ForOp> emitMmaEmulationLoops(
1155:     PatternRewriter &rewriter, Location loc, Value aPtr, Value bPtr, Value dPtr,
1156:     int64_t m, int64_t n, int64_t k, int64_t tileM, int64_t tileN,
1157:     RankedTensorType aTileTy, RankedTensorType bTileTy,
1158:     RankedTensorType accTileTy, ttg::DistributedEncodingTrait accLayout,
1159:     IntegerType accElem, Value useDInt, Value predInt, int64_t aStride,
1160:     int64_t bStride, int64_t dStride, const DotScaleConfig &scale = {},
1161:     int64_t aRowStride = 1, int64_t bRowStride = 1, int64_t dRowStride = 1) {
1162:   if ((m % tileM) != 0 || (n % tileN) != 0)
1163:     return std::nullopt;
```

- **EN:** Defines `emitMmaEmulationLoops`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `emitMmaEmulationLoops`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1165-1175

```cpp
1165:   OpBuilder::InsertionGuard guard(rewriter);
1166:   Value zero =
1167:       arith::ConstantOp::create(rewriter, loc, rewriter.getI32IntegerAttr(0));
1168:   Value mUpper =
1169:       arith::ConstantOp::create(rewriter, loc, rewriter.getI32IntegerAttr(m));
1170:   Value nUpper =
1171:       arith::ConstantOp::create(rewriter, loc, rewriter.getI32IntegerAttr(n));
1172:   Value mStep = arith::ConstantOp::create(rewriter, loc,
1173:                                           rewriter.getI32IntegerAttr(tileM));
1174:   Value nStep = arith::ConstantOp::create(rewriter, loc,
1175:                                           rewriter.getI32IntegerAttr(tileN));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1177-1182

```cpp
1177:   auto mLoop = scf::ForOp::create(rewriter, loc, zero, mUpper, mStep);
1178:   rewriter.setInsertionPointToStart(mLoop.getBody());
1179:   Value mIdx = mLoop.getInductionVar();
1180:   auto nLoop = scf::ForOp::create(rewriter, loc, zero, nUpper, nStep);
1181:   rewriter.setInsertionPointToStart(nLoop.getBody());
1182:   Value nIdx = nLoop.getInductionVar();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1184-1196

```cpp
1184:   auto i32Ty = rewriter.getI32Type();
1185:   Value mIdxI32 = arith::IndexCastOp::create(rewriter, loc, i32Ty, mIdx);
1186:   Value nIdxI32 = arith::IndexCastOp::create(rewriter, loc, i32Ty, nIdx);
1187:   Value dRowStrideConst = arith::ConstantOp::create(
1188:       rewriter, loc, rewriter.getI32IntegerAttr(dRowStride));
1189:   Value dStrideConst = arith::ConstantOp::create(
1190:       rewriter, loc, rewriter.getI32IntegerAttr(dStride));
1191:   Value aRowStrideConst = arith::ConstantOp::create(
1192:       rewriter, loc, rewriter.getI32IntegerAttr(aRowStride));
1193:   Value bStrideConst = arith::ConstantOp::create(
1194:       rewriter, loc, rewriter.getI32IntegerAttr(bStride));
1195:   Value bRowStrideConst = arith::ConstantOp::create(
1196:       rewriter, loc, rewriter.getI32IntegerAttr(bRowStride));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1198-1206

```cpp
1198:   Value mDOffset =
1199:       arith::MulIOp::create(rewriter, loc, mIdxI32, dRowStrideConst);
1200:   Value nDOffset = arith::MulIOp::create(rewriter, loc, nIdxI32, dStrideConst);
1201:   Value dOffset = arith::AddIOp::create(rewriter, loc, mDOffset, nDOffset);
1202:   Value dTilePtr =
1203:       tt::AddPtrOp::create(rewriter, loc, dPtr.getType(), dPtr, dOffset);
1204:   Value accTile = loadScratchStrided2D(rewriter, loc, dTilePtr, accTileTy,
1205:                                        dRowStride, dStride);
1206:   Value accTileI = embedToInt(rewriter, loc, accTile);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1208-1214

```cpp
1208:   Value aMOffset =
1209:       arith::MulIOp::create(rewriter, loc, mIdxI32, aRowStrideConst);
1210:   Value aTilePtr =
1211:       tt::AddPtrOp::create(rewriter, loc, aPtr.getType(), aPtr, aMOffset);
1212:   Value bOffset = arith::MulIOp::create(rewriter, loc, nIdxI32, bStrideConst);
1213:   Value bTilePtr =
1214:       tt::AddPtrOp::create(rewriter, loc, bPtr.getType(), bPtr, bOffset);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1216-1221

```cpp
1216:   auto aSliceTy =
1217:       RankedTensorType::get({tileM, 1}, aTileTy.getElementType(), accLayout);
1218:   auto bSliceTy =
1219:       RankedTensorType::get({1, tileN}, bTileTy.getElementType(), accLayout);
1220:   Value aStrideVal = arith::ConstantOp::create(
1221:       rewriter, loc, rewriter.getI32IntegerAttr(aStride));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1223-1238

```cpp
1223:   Value zeroSum = getIntConstantLike(rewriter, loc, accTileI.getType(), 0);
1224:   Value kUpper =
1225:       arith::ConstantOp::create(rewriter, loc, rewriter.getI32IntegerAttr(k));
1226:   Value kStep =
1227:       arith::ConstantOp::create(rewriter, loc, rewriter.getI32IntegerAttr(1));
1228:   auto kLoop = scf::ForOp::create(rewriter, loc, zero, kUpper, kStep, zeroSum);
1229:   rewriter.setInsertionPointToStart(kLoop.getBody());
1230:   Value kIdx = kLoop.getInductionVar();
1231:   Value kI32 = arith::IndexCastOp::create(rewriter, loc, i32Ty, kIdx);
1232:   Value aKIdx = kI32;
1233:   Value bKIdx = kI32;
1234:   if (scale.aKPackFactor == 2) {
1235:     Value aPackFactor = arith::ConstantOp::create(
1236:         rewriter, loc, rewriter.getI32IntegerAttr(scale.aKPackFactor));
1237:     aKIdx = arith::DivUIOp::create(rewriter, loc, kI32, aPackFactor);
1238:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1239-1255

```cpp
1239:   if (scale.bKPackFactor == 2) {
1240:     Value bPackFactor = arith::ConstantOp::create(
1241:         rewriter, loc, rewriter.getI32IntegerAttr(scale.bKPackFactor));
1242:     bKIdx = arith::DivUIOp::create(rewriter, loc, kI32, bPackFactor);
1243:   }
1244:   Value aOffset =
1245:       arith::MulIOp::create(rewriter, loc, i32Ty, aKIdx, aStrideVal);
1246:   Value aSlicePtr =
1247:       tt::AddPtrOp::create(rewriter, loc, aPtr.getType(), aTilePtr, aOffset);
1248:   Value aSlice = loadScratchStrided2D(rewriter, loc, aSlicePtr, aSliceTy,
1249:                                       aRowStride, aStride);
1250:   Value bKOffset = arith::MulIOp::create(rewriter, loc, bKIdx, bRowStrideConst);
1251:   Value bSlicePtr =
1252:       tt::AddPtrOp::create(rewriter, loc, bPtr.getType(), bTilePtr, bKOffset);
1253:   Value bSlice = loadScratchStrided2D(rewriter, loc, bSlicePtr, bSliceTy,
1254:                                       bRowStride, bStride);
1255:   Value aScaleSlice;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1256-1273

```cpp
1256:   if (scale.aScalePtr) {
1257:     if (scale.aKPackFactor == 2)
1258:       aSlice = unpackPackedFp4Slice(rewriter, loc, aSlice, kI32);
1259:     aScaleSlice =
1260:         loadScaleSlice(rewriter, loc, /*isLhs=*/true, scale, mIdxI32, kI32);
1261:   }
1262:   Value bScaleSlice;
1263:   if (scale.bScalePtr) {
1264:     if (scale.bKPackFactor == 2)
1265:       bSlice = unpackPackedFp4Slice(rewriter, loc, bSlice, kI32);
1266:     bScaleSlice =
1267:         loadScaleSlice(rewriter, loc, /*isLhs=*/false, scale, nIdxI32, kI32);
1268:   }
1269:   Value partial =
1270:       emulateDotStep(rewriter, loc, aSlice, bSlice, aScaleSlice, bScaleSlice,
1271:                      tileM, tileN, accLayout, accElem, scale);
1272:   Value acc = kLoop.getRegionIterArgs()[0];
1273:   Value next = arith::AddIOp::create(rewriter, loc, acc, partial);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1274-1276

```cpp
1274:   scf::YieldOp::create(rewriter, loc, next);
1275:   rewriter.setInsertionPointAfter(kLoop);
1276:   Value sum = kLoop.getResult(0);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1278-1281

```cpp
1278:   Value useDMask =
1279:       tt::SplatOp::create(rewriter, loc, accTileI.getType(), useDInt);
1280:   Value accInitI = arith::MulIOp::create(rewriter, loc, accTileI, useDMask);
1281:   Value outI = arith::AddIOp::create(rewriter, loc, sum, accInitI);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1283-1293

```cpp
1283:   Value predMask =
1284:       tt::SplatOp::create(rewriter, loc, accTileI.getType(), predInt);
1285:   Value oneI = getIntConstantLike(rewriter, loc, accTileI.getType(), 1);
1286:   Value predInv = arith::SubIOp::create(rewriter, loc, oneI, predMask);
1287:   Value outMasked = arith::MulIOp::create(rewriter, loc, outI, predMask);
1288:   Value accMasked = arith::MulIOp::create(rewriter, loc, accTileI, predInv);
1289:   Value outSelI = arith::AddIOp::create(rewriter, loc, outMasked, accMasked);
1290:   Value out = unembedToFloat(rewriter, loc, outSelI, accTileTy);
1291:   createGlobalScratchBarrier(rewriter, loc);
1292:   storeScratchStrided2D(rewriter, loc, dTilePtr, out, accTileTy, dRowStride,
1293:                         dStride);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1295-1296

```cpp
1295:   return mLoop;
1296: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1298-1300

```cpp
1298: //----------------------------------------
1299: // Patterns
1300: //----------------------------------------
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 1302-1317

```cpp
1302: template <typename OpF, typename OpI>
1303: struct BinaryFloatToIntPattern : public OpRewritePattern<OpF> {
1304:   using OpRewritePattern<OpF>::OpRewritePattern;
1305:   LogicalResult matchAndRewrite(OpF op,
1306:                                 PatternRewriter &rewriter) const override {
1307:     if (!isFloatLike(op.getType()))
1308:       return failure();
1309:     auto loc = op.getLoc();
1310:     auto lhsI = embedToInt(rewriter, loc, op.getLhs());
1311:     auto rhsI = embedToInt(rewriter, loc, op.getRhs());
1312:     auto resI = OpI::create(rewriter, loc, lhsI, rhsI);
1313:     auto resF = unembedToFloat(rewriter, loc, resI, op.getType());
1314:     rewriter.replaceOp(op, resF);
1315:     return success();
1316:   }
1317: };
```

- **EN:** Defines `BinaryFloatToIntPattern`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. It finishes by replacing the original operation with the lowered form. The code uses `LogicalResult` to stay conservative when preconditions are not met.
- **CN:** 这里定义 `BinaryFloatToIntPattern`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 它会在转换成功后用降级后的形式替换原始操作。 代码通过 `LogicalResult` 在前提不满足时保持保守处理。
### Lines 1319-1320

```cpp
1319: struct NegFOpPattern : public OpRewritePattern<arith::NegFOp> {
1320:   using OpRewritePattern::OpRewritePattern;
```

- **EN:** Defines `NegFOpPattern`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `NegFOpPattern`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1322-1325

```cpp
1322:   LogicalResult matchAndRewrite(arith::NegFOp op,
1323:                                 PatternRewriter &rewriter) const override {
1324:     if (!isFloatLike(op.getType()))
1325:       return failure();
```

- **EN:** Defines `matchAndRewrite`, the core MLIR rewrite callback. It checks whether the source operation matches the pattern and then materializes the lowered replacement.
- **CN:** 这里定义 `matchAndRewrite`，即 MLIR 模式重写的核心回调。它先检查源操作是否匹配模式，再构造降级后的替代结果。
### Lines 1327-1335

```cpp
1327:     auto loc = op.getLoc();
1328:     auto inputI = embedToInt(rewriter, loc, op.getOperand());
1329:     auto zeroI = getIntConstantLike(rewriter, loc, inputI.getType(), 0);
1330:     auto resI = arith::SubIOp::create(rewriter, loc, zeroI, inputI);
1331:     auto resF = unembedToFloat(rewriter, loc, resI, op.getType());
1332:     rewriter.replaceOp(op, resF);
1333:     return success();
1334:   }
1335: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. It finishes by replacing the original operation with the lowered form.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 它会在转换成功后用降级后的形式替换原始操作。
### Lines 1337-1347

```cpp
1337: struct DivFOpPattern : public OpRewritePattern<arith::DivFOp> {
1338:   using OpRewritePattern::OpRewritePattern;
1339:   LogicalResult matchAndRewrite(arith::DivFOp op,
1340:                                 PatternRewriter &rewriter) const override {
1341:     if (!isFloatLike(op.getType()))
1342:       return failure();
1343:     rewriter.replaceOp(
1344:         op, fpsanFDiv(rewriter, op.getLoc(), op.getLhs(), op.getRhs()));
1345:     return success();
1346:   }
1347: };
```

- **EN:** Defines `DivFOpPattern`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. It finishes by replacing the original operation with the lowered form. The code uses `LogicalResult` to stay conservative when preconditions are not met.
- **CN:** 这里定义 `DivFOpPattern`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 它会在转换成功后用降级后的形式替换原始操作。 代码通过 `LogicalResult` 在前提不满足时保持保守处理。
### Lines 1349-1359

```cpp
1349: struct PreciseDivFOpPattern : public OpRewritePattern<tt::PreciseDivFOp> {
1350:   using OpRewritePattern::OpRewritePattern;
1351:   LogicalResult matchAndRewrite(tt::PreciseDivFOp op,
1352:                                 PatternRewriter &rewriter) const override {
1353:     if (!isFloatLike(op.getType()))
1354:       return failure();
1355:     rewriter.replaceOp(op,
1356:                        fpsanFDiv(rewriter, op.getLoc(), op.getX(), op.getY()));
1357:     return success();
1358:   }
1359: };
```

- **EN:** Defines `PreciseDivFOpPattern`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. It finishes by replacing the original operation with the lowered form. The code uses `LogicalResult` to stay conservative when preconditions are not met.
- **CN:** 这里定义 `PreciseDivFOpPattern`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 它会在转换成功后用降级后的形式替换原始操作。 代码通过 `LogicalResult` 在前提不满足时保持保守处理。
### Lines 1361-1371

```cpp
1361: struct RemFOpPattern : public OpRewritePattern<arith::RemFOp> {
1362:   using OpRewritePattern::OpRewritePattern;
1363:   LogicalResult matchAndRewrite(arith::RemFOp op,
1364:                                 PatternRewriter &rewriter) const override {
1365:     if (!isFloatLike(op.getType()))
1366:       return failure();
1367:     rewriter.replaceOp(
1368:         op, fpsanSRem(rewriter, op.getLoc(), op.getLhs(), op.getRhs()));
1369:     return success();
1370:   }
1371: };
```

- **EN:** Defines `RemFOpPattern`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. It finishes by replacing the original operation with the lowered form. The code uses `LogicalResult` to stay conservative when preconditions are not met.
- **CN:** 这里定义 `RemFOpPattern`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 它会在转换成功后用降级后的形式替换原始操作。 代码通过 `LogicalResult` 在前提不满足时保持保守处理。
### Lines 1373-1389

```cpp
1373: struct FmaPattern : public OpRewritePattern<math::FmaOp> {
1374:   using OpRewritePattern::OpRewritePattern;
1375:   LogicalResult matchAndRewrite(math::FmaOp op,
1376:                                 PatternRewriter &rewriter) const override {
1377:     if (!isFloatLike(op.getType()))
1378:       return failure();
1379:     auto loc = op.getLoc();
1380:     auto aI = embedToInt(rewriter, loc, op.getA());
1381:     auto bI = embedToInt(rewriter, loc, op.getB());
1382:     auto cI = embedToInt(rewriter, loc, op.getC());
1383:     auto mul = arith::MulIOp::create(rewriter, loc, aI, bI);
1384:     auto sum = arith::AddIOp::create(rewriter, loc, mul, cI);
1385:     auto resF = unembedToFloat(rewriter, loc, sum, op.getType());
1386:     rewriter.replaceOp(op, resF);
1387:     return success();
1388:   }
1389: };
```

- **EN:** Defines `FmaPattern`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. It finishes by replacing the original operation with the lowered form. The code uses `LogicalResult` to stay conservative when preconditions are not met.
- **CN:** 这里定义 `FmaPattern`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 它会在转换成功后用降级后的形式替换原始操作。 代码通过 `LogicalResult` 在前提不满足时保持保守处理。
### Lines 1391-1404

```cpp
1391: struct ExpOpPattern : public OpRewritePattern<math::ExpOp> {
1392:   using OpRewritePattern::OpRewritePattern;
1393:   LogicalResult matchAndRewrite(math::ExpOp op,
1394:                                 PatternRewriter &rewriter) const override {
1395:     if (!isFloatLike(op.getType()))
1396:       return failure();
1397:     Value result = fpsanExp(rewriter, op.getLoc(), op.getOperand());
1398:     if (!result)
1399:       result = fpsanUnaryTagged(rewriter, op.getLoc(), op.getOperand(),
1400:                                 UnaryOpId::Exp);
1401:     rewriter.replaceOp(op, result);
1402:     return success();
1403:   }
1404: };
```

- **EN:** Defines `ExpOpPattern`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. It finishes by replacing the original operation with the lowered form. The code uses `LogicalResult` to stay conservative when preconditions are not met.
- **CN:** 这里定义 `ExpOpPattern`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 它会在转换成功后用降级后的形式替换原始操作。 代码通过 `LogicalResult` 在前提不满足时保持保守处理。
### Lines 1406-1419

```cpp
1406: struct Exp2OpPattern : public OpRewritePattern<math::Exp2Op> {
1407:   using OpRewritePattern::OpRewritePattern;
1408:   LogicalResult matchAndRewrite(math::Exp2Op op,
1409:                                 PatternRewriter &rewriter) const override {
1410:     if (!isFloatLike(op.getType()))
1411:       return failure();
1412:     Value result = fpsanExp2(rewriter, op.getLoc(), op.getOperand());
1413:     if (!result)
1414:       result = fpsanUnaryTagged(rewriter, op.getLoc(), op.getOperand(),
1415:                                 UnaryOpId::Exp2);
1416:     rewriter.replaceOp(op, result);
1417:     return success();
1418:   }
1419: };
```

- **EN:** Defines `Exp2OpPattern`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. It finishes by replacing the original operation with the lowered form. The code uses `LogicalResult` to stay conservative when preconditions are not met.
- **CN:** 这里定义 `Exp2OpPattern`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 它会在转换成功后用降级后的形式替换原始操作。 代码通过 `LogicalResult` 在前提不满足时保持保守处理。
### Lines 1421-1434

```cpp
1421: struct CosOpPattern : public OpRewritePattern<math::CosOp> {
1422:   using OpRewritePattern::OpRewritePattern;
1423:   LogicalResult matchAndRewrite(math::CosOp op,
1424:                                 PatternRewriter &rewriter) const override {
1425:     if (!isFloatLike(op.getType()))
1426:       return failure();
1427:     Value result = fpsanCos(rewriter, op.getLoc(), op.getOperand());
1428:     if (!result)
1429:       result = fpsanUnaryTagged(rewriter, op.getLoc(), op.getOperand(),
1430:                                 UnaryOpId::Cos);
1431:     rewriter.replaceOp(op, result);
1432:     return success();
1433:   }
1434: };
```

- **EN:** Defines `CosOpPattern`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. It finishes by replacing the original operation with the lowered form. The code uses `LogicalResult` to stay conservative when preconditions are not met.
- **CN:** 这里定义 `CosOpPattern`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 它会在转换成功后用降级后的形式替换原始操作。 代码通过 `LogicalResult` 在前提不满足时保持保守处理。
### Lines 1436-1449

```cpp
1436: struct SinOpPattern : public OpRewritePattern<math::SinOp> {
1437:   using OpRewritePattern::OpRewritePattern;
1438:   LogicalResult matchAndRewrite(math::SinOp op,
1439:                                 PatternRewriter &rewriter) const override {
1440:     if (!isFloatLike(op.getType()))
1441:       return failure();
1442:     Value result = fpsanSin(rewriter, op.getLoc(), op.getOperand());
1443:     if (!result)
1444:       result = fpsanUnaryTagged(rewriter, op.getLoc(), op.getOperand(),
1445:                                 UnaryOpId::Sin);
1446:     rewriter.replaceOp(op, result);
1447:     return success();
1448:   }
1449: };
```

- **EN:** Defines `SinOpPattern`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. It finishes by replacing the original operation with the lowered form. The code uses `LogicalResult` to stay conservative when preconditions are not met.
- **CN:** 这里定义 `SinOpPattern`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 它会在转换成功后用降级后的形式替换原始操作。 代码通过 `LogicalResult` 在前提不满足时保持保守处理。
### Lines 1451-1465

```cpp
1451: struct ExtFOpPattern : public OpRewritePattern<arith::ExtFOp> {
1452:   using OpRewritePattern::OpRewritePattern;
1453:   LogicalResult matchAndRewrite(arith::ExtFOp op,
1454:                                 PatternRewriter &rewriter) const override {
1455:     if (!isFloatLike(op.getType()))
1456:       return failure();
1457:     auto loc = op.getLoc();
1458:     auto inI = embedToInt(rewriter, loc, op.getIn());
1459:     auto outI = castSignedIntValueToType(rewriter, loc, inI,
1460:                                          getIntTypeLike(op.getType()));
1461:     auto outF = unembedToFloat(rewriter, loc, outI, op.getType());
1462:     rewriter.replaceOp(op, outF);
1463:     return success();
1464:   }
1465: };
```

- **EN:** Defines `ExtFOpPattern`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. It finishes by replacing the original operation with the lowered form. The code uses `LogicalResult` to stay conservative when preconditions are not met.
- **CN:** 这里定义 `ExtFOpPattern`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 它会在转换成功后用降级后的形式替换原始操作。 代码通过 `LogicalResult` 在前提不满足时保持保守处理。
### Lines 1467-1481

```cpp
1467: struct TruncFOpPattern : public OpRewritePattern<arith::TruncFOp> {
1468:   using OpRewritePattern::OpRewritePattern;
1469:   LogicalResult matchAndRewrite(arith::TruncFOp op,
1470:                                 PatternRewriter &rewriter) const override {
1471:     if (!isFloatLike(op.getType()))
1472:       return failure();
1473:     auto loc = op.getLoc();
1474:     auto inI = embedToInt(rewriter, loc, op.getIn());
1475:     auto outI = castSignedIntValueToType(rewriter, loc, inI,
1476:                                          getIntTypeLike(op.getType()));
1477:     auto outF = unembedToFloat(rewriter, loc, outI, op.getType());
1478:     rewriter.replaceOp(op, outF);
1479:     return success();
1480:   }
1481: };
```

- **EN:** Defines `TruncFOpPattern`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. It finishes by replacing the original operation with the lowered form. The code uses `LogicalResult` to stay conservative when preconditions are not met.
- **CN:** 这里定义 `TruncFOpPattern`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 它会在转换成功后用降级后的形式替换原始操作。 代码通过 `LogicalResult` 在前提不满足时保持保守处理。
### Lines 1483-1497

```cpp
1483: struct FpToFpPattern : public OpRewritePattern<tt::FpToFpOp> {
1484:   using OpRewritePattern::OpRewritePattern;
1485:   LogicalResult matchAndRewrite(tt::FpToFpOp op,
1486:                                 PatternRewriter &rewriter) const override {
1487:     if (!isFloatLike(op.getType()))
1488:       return failure();
1489:     auto loc = op.getLoc();
1490:     auto inI = embedToInt(rewriter, loc, op.getSrc());
1491:     auto outI = castSignedIntValueToType(rewriter, loc, inI,
1492:                                          getIntTypeLike(op.getType()));
1493:     auto outF = unembedToFloat(rewriter, loc, outI, op.getType());
1494:     rewriter.replaceOp(op, outF);
1495:     return success();
1496:   }
1497: };
```

- **EN:** Defines `FpToFpPattern`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. It finishes by replacing the original operation with the lowered form. The code uses `LogicalResult` to stay conservative when preconditions are not met.
- **CN:** 这里定义 `FpToFpPattern`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 它会在转换成功后用降级后的形式替换原始操作。 代码通过 `LogicalResult` 在前提不满足时保持保守处理。
### Lines 1499-1511

```cpp
1499: struct Fp4ToFpPattern : public OpRewritePattern<ttg::Fp4ToFpOp> {
1500:   using OpRewritePattern::OpRewritePattern;
1501:   LogicalResult matchAndRewrite(ttg::Fp4ToFpOp op,
1502:                                 PatternRewriter &rewriter) const override {
1503:     if (!isFloatLike(op.getType()))
1504:       return failure();
1505:     auto srcTy = dyn_cast<RankedTensorType>(op.getSrc().getType());
1506:     auto dstTy = dyn_cast<RankedTensorType>(op.getType());
1507:     if (!srcTy || !dstTy)
1508:       return emitFpSanInvariantError(op.getOperation());
1509:     auto srcElemTy = dyn_cast<IntegerType>(srcTy.getElementType());
1510:     if (!srcElemTy || srcElemTy.getWidth() != 8)
1511:       return emitFpSanInvariantError(op.getOperation());
```

- **EN:** Defines `Fp4ToFpPattern`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `Fp4ToFpPattern`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1513-1516

```cpp
1513:     int64_t axis = op.getAxis();
1514:     int64_t rank = srcTy.getRank();
1515:     auto dstIntTy = cast<RankedTensorType>(getIntTypeLike(dstTy));
1516:     auto halfIntTy = srcTy.clone(dstIntTy.getElementType());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1518-1525

```cpp
1518:     auto loc = op.getLoc();
1519:     auto mask = getIntConstantLike(rewriter, loc, srcTy, 0x0F);
1520:     auto four = getIntConstantLike(rewriter, loc, srcTy, 4);
1521:     Value lo = arith::AndIOp::create(rewriter, loc, op.getSrc(), mask);
1522:     Value hi = arith::ShRUIOp::create(rewriter, loc, op.getSrc(), four);
1523:     auto loI = castSignedIntValueToType(rewriter, loc, lo, halfIntTy);
1524:     auto hiI = castSignedIntValueToType(rewriter, loc, hi, halfIntTy);
1525:     Value joined = tt::JoinOp::create(rewriter, loc, loI, hiI);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1527-1530

```cpp
1527:     auto order = llvm::to_vector(llvm::seq<int32_t>(axis + 1));
1528:     order.push_back(rank);
1529:     llvm::append_range(order, llvm::seq<int32_t>(axis + 1, rank));
1530:     auto transposed = tt::TransOp::create(rewriter, loc, joined, order);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1532-1535

```cpp
1532:     Value result =
1533:         tt::ReshapeOp::create(rewriter, loc, dstTy.getShape(), transposed);
1534:     if (result.getType() != dstIntTy)
1535:       result = ttg::ConvertLayoutOp::create(rewriter, loc, dstIntTy, result);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1537-1540

```cpp
1537:     rewriter.replaceOp(op, unembedToFloat(rewriter, loc, result, dstTy));
1538:     return success();
1539:   }
1540: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. It finishes by replacing the original operation with the lowered form.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 它会在转换成功后用降级后的形式替换原始操作。
### Lines 1542-1557

```cpp
1542: struct DotPattern : public OpRewritePattern<tt::DotOp> {
1543:   using OpRewritePattern::OpRewritePattern;
1544:   LogicalResult matchAndRewrite(tt::DotOp op,
1545:                                 PatternRewriter &rewriter) const override {
1546:     if (!isFloatLike(op.getType()))
1547:       return failure();
1548:     auto aTy = dyn_cast<RankedTensorType>(op.getA().getType());
1549:     auto bTy = dyn_cast<RankedTensorType>(op.getB().getType());
1550:     auto cTy = dyn_cast<RankedTensorType>(op.getC().getType());
1551:     if (!aTy || !bTy || !cTy)
1552:       return emitFpSanInvariantError(op.getOperation());
1553:     if (aTy.getRank() != bTy.getRank() || aTy.getRank() != cTy.getRank() ||
1554:         (aTy.getRank() != 2 && aTy.getRank() != 3))
1555:       return emitFpSanUnsupported(op.getOperation());
1556:     if (!aTy.getEncoding() || !bTy.getEncoding() || !cTy.getEncoding())
1557:       return emitFpSanUnsupported(op.getOperation());
```

- **EN:** Defines `DotPattern`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `DotPattern`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1559-1575

```cpp
1559:     auto aShape = aTy.getShape();
1560:     auto bShape = bTy.getShape();
1561:     auto cShape = cTy.getShape();
1562:     auto loc = op.getLoc();
1563:     int64_t batch = 1;
1564:     int64_t m;
1565:     int64_t k;
1566:     int64_t n;
1567:     int64_t aBatchStride = 0;
1568:     int64_t bBatchStride = 0;
1569:     int64_t dBatchStride = 0;
1570:     int64_t aRowStride;
1571:     int64_t aKStride;
1572:     int64_t bKStride;
1573:     int64_t bNStride;
1574:     int64_t dRowStride;
1575:     int64_t dNStride;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1576-1593

```cpp
1576:     if (aTy.getRank() == 2) {
1577:       if (aShape[1] != bShape[0] || aShape[0] != cShape[0] ||
1578:           bShape[1] != cShape[1])
1579:         return emitFpSanInvariantError(op.getOperation());
1580:       m = aShape[0];
1581:       k = aShape[1];
1582:       n = bShape[1];
1583:       aRowStride = 1;
1584:       aKStride = m;
1585:       bKStride = 1;
1586:       bNStride = k;
1587:       dRowStride = 1;
1588:       dNStride = m;
1589:     } else {
1590:       if (aShape[0] != bShape[0] || aShape[0] != cShape[0] ||
1591:           aShape[2] != bShape[1] || aShape[1] != cShape[1] ||
1592:           bShape[2] != cShape[2])
1593:         return emitFpSanInvariantError(op.getOperation());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1594-1607

```cpp
1594:       batch = aShape[0];
1595:       m = aShape[1];
1596:       k = aShape[2];
1597:       n = bShape[2];
1598:       aBatchStride = 1;
1599:       bBatchStride = 1;
1600:       dBatchStride = 1;
1601:       aRowStride = batch;
1602:       aKStride = batch * m;
1603:       bKStride = batch;
1604:       bNStride = batch * k;
1605:       dRowStride = batch;
1606:       dNStride = batch * m;
1607:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1609-1614

```cpp
1609:     auto accElem = IntegerType::get(
1610:         rewriter.getContext(), cTy.getElementType().getIntOrFloatBitWidth());
1611:     Value useDInt = arith::ConstantOp::create(
1612:         rewriter, loc, rewriter.getIntegerAttr(accElem, 1));
1613:     Value predInt = arith::ConstantOp::create(
1614:         rewriter, loc, rewriter.getIntegerAttr(accElem, 1));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1616-1617

```cpp
1616:     int64_t tileM = std::min<int64_t>(kTileM, m);
1617:     int64_t tileN = std::min<int64_t>(kTileN, n);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1619-1628

```cpp
1619:     // Use optimized blocked layouts for emulation tiles instead of the
1620:     // original dot encodings.  Encodings like AMDWmmaEncodingAttr impose
1621:     // minimum shape requirements (e.g. >= 16x16) that the small emulation
1622:     // tiles (kTileM x kTileN = 8x8) cannot satisfy.
1623:     auto accLayout = getOptimizedBlockedEncoding(rewriter, {tileM, tileN},
1624:                                                  cTy.getElementType());
1625:     auto aLayout =
1626:         getOptimizedBlockedEncoding(rewriter, {tileM, k}, aTy.getElementType());
1627:     auto bLayout =
1628:         getOptimizedBlockedEncoding(rewriter, {k, tileN}, bTy.getElementType());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1630-1635

```cpp
1630:     auto accTileTy =
1631:         RankedTensorType::get({tileM, tileN}, cTy.getElementType(), accLayout);
1632:     auto aTileTy =
1633:         RankedTensorType::get({tileM, k}, aTy.getElementType(), aLayout);
1634:     auto bTileTy =
1635:         RankedTensorType::get({k, tileN}, bTy.getElementType(), bLayout);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1637-1641

```cpp
1637:     Value aPtr = createScratchAndStore(rewriter, loc, op.getA(), aTy);
1638:     Value bPtr = createScratchAndStore(rewriter, loc, op.getB(), bTy);
1639:     Value dPtr = createScratchAndStore(rewriter, loc, op.getC(), cTy);
1640:     if (!aPtr || !bPtr || !dPtr)
1641:       return emitFpSanCodegenError(op.getOperation());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1643-1645

```cpp
1643:     // Each warp may only store a subset of each tile's rows, so a barrier is
1644:     // needed to make all scratch stores visible before the loops read them.
1645:     createGlobalScratchBarrier(rewriter, loc);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1647-1664

```cpp
1647:     if (batch == 1) {
1648:       auto mLoop = emitMmaEmulationLoops(
1649:           rewriter, loc, aPtr, bPtr, dPtr, m, n, k, tileM, tileN, aTileTy,
1650:           bTileTy, accTileTy, accLayout, accElem, useDInt, predInt,
1651:           /*aStride=*/aKStride, /*bStride=*/bNStride, /*dStride=*/dNStride,
1652:           /*scale=*/{}, aRowStride, bKStride, dRowStride);
1653:       if (!mLoop)
1654:         return emitFpSanUnsupported(op.getOperation());
1655:       rewriter.setInsertionPointAfter(*mLoop);
1656:     } else {
1657:       Value zero = arith::ConstantOp::create(rewriter, loc,
1658:                                              rewriter.getI32IntegerAttr(0));
1659:       Value batchUpper = arith::ConstantOp::create(
1660:           rewriter, loc, rewriter.getI32IntegerAttr(batch));
1661:       Value one = arith::ConstantOp::create(rewriter, loc,
1662:                                             rewriter.getI32IntegerAttr(1));
1663:       auto batchLoop = scf::ForOp::create(rewriter, loc, zero, batchUpper, one);
1664:       rewriter.setInsertionPointToStart(batchLoop.getBody());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1665-1682

```cpp
1665:       Value batchIdx = arith::IndexCastOp::create(
1666:           rewriter, loc, rewriter.getI32Type(), batchLoop.getInductionVar());
1667:       Value aBatchOffset = arith::MulIOp::create(
1668:           rewriter, loc, batchIdx,
1669:           arith::ConstantOp::create(rewriter, loc,
1670:                                     rewriter.getI32IntegerAttr(aBatchStride)));
1671:       Value bBatchOffset = arith::MulIOp::create(
1672:           rewriter, loc, batchIdx,
1673:           arith::ConstantOp::create(rewriter, loc,
1674:                                     rewriter.getI32IntegerAttr(bBatchStride)));
1675:       Value dBatchOffset = arith::MulIOp::create(
1676:           rewriter, loc, batchIdx,
1677:           arith::ConstantOp::create(rewriter, loc,
1678:                                     rewriter.getI32IntegerAttr(dBatchStride)));
1679:       Value aBatchPtr = tt::AddPtrOp::create(rewriter, loc, aPtr.getType(),
1680:                                              aPtr, aBatchOffset);
1681:       Value bBatchPtr = tt::AddPtrOp::create(rewriter, loc, bPtr.getType(),
1682:                                              bPtr, bBatchOffset);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1683-1693

```cpp
1683:       Value dBatchPtr = tt::AddPtrOp::create(rewriter, loc, dPtr.getType(),
1684:                                              dPtr, dBatchOffset);
1685:       auto mLoop = emitMmaEmulationLoops(
1686:           rewriter, loc, aBatchPtr, bBatchPtr, dBatchPtr, m, n, k, tileM, tileN,
1687:           aTileTy, bTileTy, accTileTy, accLayout, accElem, useDInt, predInt,
1688:           /*aStride=*/aKStride, /*bStride=*/bNStride,
1689:           /*dStride=*/dNStride, /*scale=*/{}, aRowStride, bKStride, dRowStride);
1690:       if (!mLoop)
1691:         return emitFpSanUnsupported(op.getOperation());
1692:       rewriter.setInsertionPointAfter(batchLoop);
1693:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1695-1697

```cpp
1695:     // Same reason: each warp may only write a subset of D's rows in the loop,
1696:     // so synchronize before the final load.
1697:     createGlobalScratchBarrier(rewriter, loc);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1699-1708

```cpp
1699:     Value out = aTy.getRank() == 2
1700:                     ? loadScratchStrided2D(rewriter, loc, dPtr, cTy,
1701:                                            /*stride1=*/m)
1702:                     : loadFpSanScratchMemory(rewriter, loc, dPtr, cTy);
1703:     if (!out)
1704:       return emitFpSanCodegenError(op.getOperation());
1705:     rewriter.replaceOp(op, out);
1706:     return success();
1707:   }
1708: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. It finishes by replacing the original operation with the lowered form.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 它会在转换成功后用降级后的形式替换原始操作。
### Lines 1710-1726

```cpp
1710: struct DotScaledPattern : public OpRewritePattern<tt::DotScaledOp> {
1711:   using OpRewritePattern::OpRewritePattern;
1712:   LogicalResult matchAndRewrite(tt::DotScaledOp op,
1713:                                 PatternRewriter &rewriter) const override {
1714:     if (!isFloatLike(op.getType()))
1715:       return failure();
1716:     auto aScale = op.getAScale();
1717:     auto bScale = op.getBScale();
1718:     auto aTy = dyn_cast<RankedTensorType>(op.getA().getType());
1719:     auto bTy = dyn_cast<RankedTensorType>(op.getB().getType());
1720:     auto cTy = dyn_cast<RankedTensorType>(op.getC().getType());
1721:     auto aScaleTy = aScale ? dyn_cast<RankedTensorType>(aScale.getType())
1722:                            : RankedTensorType();
1723:     auto bScaleTy = bScale ? dyn_cast<RankedTensorType>(bScale.getType())
1724:                            : RankedTensorType();
1725:     if (!aTy || !bTy || !cTy || (aScale && !aScaleTy) || (bScale && !bScaleTy))
1726:       return emitFpSanInvariantError(op.getOperation());
```

- **EN:** Defines `DotScaledPattern`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `DotScaledPattern`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1727-1737

```cpp
1727:     if (aTy.getRank() != 2 || bTy.getRank() != 2 || cTy.getRank() != 2 ||
1728:         (aScale && aScaleTy.getRank() != 2) ||
1729:         (bScale && bScaleTy.getRank() != 2))
1730:       return emitFpSanUnsupported(op.getOperation());
1731:     if (!aTy.getEncoding() || !bTy.getEncoding() || !cTy.getEncoding() ||
1732:         (aScale && !aScaleTy.getEncoding()) ||
1733:         (bScale && !bScaleTy.getEncoding()))
1734:       return emitFpSanUnsupported(op.getOperation());
1735:     // TODO: Support M/N packing.
1736:     if (!op.getLhsKPack() || !op.getRhsKPack())
1737:       return emitFpSanUnsupported(op.getOperation());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1739-1743

```cpp
1739:     auto aShape = aTy.getShape();
1740:     auto bShape = bTy.getShape();
1741:     auto cShape = cTy.getShape();
1742:     if (aShape[0] != cShape[0] || bShape[1] != cShape[1])
1743:       return emitFpSanInvariantError(op.getOperation());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1745-1755

```cpp
1745:     int64_t aKPackFactor = 1;
1746:     int64_t bKPackFactor = 1;
1747:     if (op.getAElemType() == tt::ScaleDotElemType::E2M1)
1748:       aKPackFactor = 2;
1749:     if (op.getBElemType() == tt::ScaleDotElemType::E2M1)
1750:       bKPackFactor = 2;
1751:     int64_t aPackedK = aShape[1];
1752:     int64_t bPackedK = bShape[0];
1753:     int64_t k = aPackedK * aKPackFactor;
1754:     if (k != bPackedK * bKPackFactor)
1755:       return emitFpSanInvariantError(op.getOperation());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1757-1762

```cpp
1757:     auto loc = op.getLoc();
1758:     int64_t m = cShape[0];
1759:     int64_t n = cShape[1];
1760:     if ((aScale && aScaleTy.getShape()[0] != m) ||
1761:         (bScale && bScaleTy.getShape()[0] != n))
1762:       return emitFpSanInvariantError(op.getOperation());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1764-1769

```cpp
1764:     auto accElem = IntegerType::get(
1765:         rewriter.getContext(), cTy.getElementType().getIntOrFloatBitWidth());
1766:     Value useDInt = arith::ConstantOp::create(
1767:         rewriter, loc, rewriter.getIntegerAttr(accElem, 1));
1768:     Value predInt = arith::ConstantOp::create(
1769:         rewriter, loc, rewriter.getIntegerAttr(accElem, 1));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1771-1772

```cpp
1771:     int64_t tileM = std::min<int64_t>(kTileM, m);
1772:     int64_t tileN = std::min<int64_t>(kTileN, n);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1774-1779

```cpp
1774:     auto accLayout = getOptimizedBlockedEncoding(rewriter, {tileM, tileN},
1775:                                                  cTy.getElementType());
1776:     auto aLayout = getOptimizedBlockedEncoding(rewriter, {tileM, aPackedK},
1777:                                                aTy.getElementType());
1778:     auto bLayout = getOptimizedBlockedEncoding(rewriter, {bPackedK, tileN},
1779:                                                bTy.getElementType());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1781-1786

```cpp
1781:     auto accTileTy =
1782:         RankedTensorType::get({tileM, tileN}, cTy.getElementType(), accLayout);
1783:     auto aTileTy =
1784:         RankedTensorType::get({tileM, aPackedK}, aTy.getElementType(), aLayout);
1785:     auto bTileTy =
1786:         RankedTensorType::get({bPackedK, tileN}, bTy.getElementType(), bLayout);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1788-1792

```cpp
1788:     auto aPtr = createScratchAndStore(rewriter, loc, op.getA(), aTy);
1789:     auto bPtr = createScratchAndStore(rewriter, loc, op.getB(), bTy);
1790:     auto dPtr = createScratchAndStore(rewriter, loc, op.getC(), cTy);
1791:     if (!aPtr || !bPtr || !dPtr)
1792:       return emitFpSanCodegenError(op.getOperation());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1794-1799

```cpp
1794:     auto aElemType = op.getAElemType();
1795:     auto bElemType = op.getBElemType();
1796:     bool skipAScale = aElemType == tt::ScaleDotElemType::BF16 ||
1797:                       aElemType == tt::ScaleDotElemType::FP16;
1798:     bool skipBScale = bElemType == tt::ScaleDotElemType::BF16 ||
1799:                       bElemType == tt::ScaleDotElemType::FP16;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1801-1818

```cpp
1801:     DotScaleConfig scale;
1802:     scale.aElemType = aElemType;
1803:     scale.bElemType = bElemType;
1804:     scale.computeElem =
1805:         getDotScaledComputeFloatType(rewriter, aElemType, bElemType);
1806:     scale.aKPackFactor = aKPackFactor;
1807:     scale.bKPackFactor = bKPackFactor;
1808:     if (aScale && !skipAScale) {
1809:       scale.aScalePtr = createScratchAndStore(rewriter, loc, aScale, aScaleTy);
1810:       if (!scale.aScalePtr)
1811:         return emitFpSanCodegenError(op.getOperation());
1812:       scale.aScaleStride = aScaleTy.getShape()[0];
1813:       scale.aScaleFactor = op.deduceScaleFactor();
1814:       scale.aScaleTileTy = RankedTensorType::get(
1815:           {tileM, 1}, aScaleTy.getElementType(), accLayout);
1816:     }
1817:     if (bScale && !skipBScale) {
1818:       scale.bScalePtr = createScratchAndStore(rewriter, loc, bScale, bScaleTy);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1819-1825

```cpp
1819:       if (!scale.bScalePtr)
1820:         return emitFpSanCodegenError(op.getOperation());
1821:       scale.bScaleStride = bScaleTy.getShape()[0];
1822:       scale.bScaleFactor = op.deduceScaleFactor();
1823:       scale.bScaleTileTy = RankedTensorType::get(
1824:           {1, tileN}, bScaleTy.getElementType(), accLayout);
1825:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1827-1827

```cpp
1827:     createGlobalScratchBarrier(rewriter, loc);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1829-1835

```cpp
1829:     auto mLoop = emitMmaEmulationLoops(
1830:         rewriter, loc, aPtr, bPtr, dPtr, m, n, k, tileM, tileN, aTileTy,
1831:         bTileTy, accTileTy, accLayout, accElem, useDInt, predInt,
1832:         /*aStride=*/m, /*bStride=*/bPackedK, /*dStride=*/m, scale);
1833:     if (!mLoop)
1834:       return emitFpSanUnsupported(op.getOperation());
1835:     rewriter.setInsertionPointAfter(*mLoop);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1837-1837

```cpp
1837:     createGlobalScratchBarrier(rewriter, loc);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1839-1845

```cpp
1839:     Value out = loadScratchStrided2D(rewriter, loc, dPtr, cTy, /*stride1=*/m);
1840:     if (!out)
1841:       return emitFpSanCodegenError(op.getOperation());
1842:     rewriter.replaceOp(op, out);
1843:     return success();
1844:   }
1845: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. It finishes by replacing the original operation with the lowered form.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 它会在转换成功后用降级后的形式替换原始操作。
### Lines 1847-1849

```cpp
1847: struct TMEMLoadPattern : public OpRewritePattern<ttng::TMEMLoadOp> {
1848:   TMEMLoadPattern(MLIRContext *ctx, TmemScratchManager *scratch)
1849:       : OpRewritePattern(ctx), scratch(scratch) {}
```

- **EN:** Defines `TMEMLoadPattern`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `TMEMLoadPattern`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1851-1857

```cpp
1851:   LogicalResult matchAndRewrite(ttng::TMEMLoadOp op,
1852:                                 PatternRewriter &rewriter) const override {
1853:     Region *scope = getScratchScopeRegion(op);
1854:     std::optional<ScratchInfo> info =
1855:         scratch->getOrCreate(op.getSrc(), rewriter, scope);
1856:     if (!info)
1857:       return emitFpSanCodegenError(op.getOperation());
```

- **EN:** Defines `matchAndRewrite`, the core MLIR rewrite callback. It checks whether the source operation matches the pattern and then materializes the lowered replacement.
- **CN:** 这里定义 `matchAndRewrite`，即 MLIR 模式重写的核心回调。它先检查源操作是否匹配模式，再构造降级后的替代结果。
### Lines 1859-1865

```cpp
1859:     Location loc = op.getLoc();
1860:     auto resultTy = cast<RankedTensorType>(op.getResult().getType());
1861:     if (!resultTy.getEncoding())
1862:       return emitFpSanUnsupported(op.getOperation());
1863:     Value result = loadFpSanScratchMemory(rewriter, loc, info->ptr, resultTy);
1864:     if (!result)
1865:       return emitFpSanCodegenError(op.getOperation());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1867-1867

```cpp
1867:     createGlobalScratchBarrier(rewriter, loc);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1869-1879

```cpp
1869:     if (op.getNumResults() == 1) {
1870:       rewriter.replaceOp(op, result);
1871:       return success();
1872:     }
1873:     SmallVector<Value> deps;
1874:     if (op.getDep())
1875:       deps.push_back(op.getDep());
1876:     Value token = createAsyncToken(rewriter, loc, deps);
1877:     rewriter.replaceOp(op, {result, token});
1878:     return success();
1879:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. It finishes by replacing the original operation with the lowered form.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 它会在转换成功后用降级后的形式替换原始操作。
### Lines 1881-1883

```cpp
1881: private:
1882:   TmemScratchManager *scratch;
1883: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1885-1887

```cpp
1885: struct TMEMStorePattern : public OpRewritePattern<ttng::TMEMStoreOp> {
1886:   TMEMStorePattern(MLIRContext *ctx, TmemScratchManager *scratch)
1887:       : OpRewritePattern(ctx), scratch(scratch) {}
```

- **EN:** Defines `TMEMStorePattern`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `TMEMStorePattern`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1889-1894

```cpp
1889:   LogicalResult matchAndRewrite(ttng::TMEMStoreOp op,
1890:                                 PatternRewriter &rewriter) const override {
1891:     auto scope = getScratchScopeRegion(op);
1892:     auto info = scratch->getOrCreate(op.getDst(), rewriter, scope);
1893:     if (!info)
1894:       return emitFpSanCodegenError(op.getOperation());
```

- **EN:** Defines `matchAndRewrite`, the core MLIR rewrite callback. It checks whether the source operation matches the pattern and then materializes the lowered replacement.
- **CN:** 这里定义 `matchAndRewrite`，即 MLIR 模式重写的核心回调。它先检查源操作是否匹配模式，再构造降级后的替代结果。
### Lines 1896-1901

```cpp
1896:     auto loc = op.getLoc();
1897:     auto srcTy = cast<RankedTensorType>(op.getSrc().getType());
1898:     if (!srcTy.getEncoding())
1899:       return emitFpSanUnsupported(op.getOperation());
1900:     if (!storeFpSanScratchMemory(rewriter, loc, info->ptr, op.getSrc(), srcTy))
1901:       return emitFpSanCodegenError(op.getOperation());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1903-1903

```cpp
1903:     createGlobalScratchBarrier(rewriter, loc);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1905-1915

```cpp
1905:     if (op.getNumResults() == 0) {
1906:       rewriter.eraseOp(op);
1907:       return success();
1908:     }
1909:     SmallVector<Value> deps;
1910:     if (op.getDep())
1911:       deps.push_back(op.getDep());
1912:     Value token = createAsyncToken(rewriter, loc, deps);
1913:     rewriter.replaceOp(op, token);
1914:     return success();
1915:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. It finishes by replacing the original operation with the lowered form. When the lowered form has no direct result, the original op is erased.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 它会在转换成功后用降级后的形式替换原始操作。 当降级后的形式不再需要直接结果时，原始操作会被删除。
### Lines 1917-1919

```cpp
1917: private:
1918:   TmemScratchManager *scratch;
1919: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1921-1923

```cpp
1921: struct TMEMCopyPattern : public OpRewritePattern<ttng::TMEMCopyOp> {
1922:   TMEMCopyPattern(MLIRContext *ctx, TmemScratchManager *scratch)
1923:       : OpRewritePattern(ctx), scratch(scratch) {}
```

- **EN:** Defines `TMEMCopyPattern`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `TMEMCopyPattern`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1925-1930

```cpp
1925:   LogicalResult matchAndRewrite(ttng::TMEMCopyOp op,
1926:                                 PatternRewriter &rewriter) const override {
1927:     auto scope = getScratchScopeRegion(op);
1928:     auto info = scratch->getOrCreate(op.getDst(), rewriter, scope);
1929:     if (!info)
1930:       return emitFpSanCodegenError(op.getOperation());
```

- **EN:** Defines `matchAndRewrite`, the core MLIR rewrite callback. It checks whether the source operation matches the pattern and then materializes the lowered replacement.
- **CN:** 这里定义 `matchAndRewrite`，即 MLIR 模式重写的核心回调。它先检查源操作是否匹配模式，再构造降级后的替代结果。
### Lines 1932-1943

```cpp
1932:     auto loc = op.getLoc();
1933:     auto srcMemTy = cast<ttg::MemDescType>(op.getSrc().getType());
1934:     auto dstMemTy = cast<ttg::MemDescType>(op.getDst().getType());
1935:     auto srcEncoding =
1936:         scratch->getScratchEncoding(rewriter, op.getDst(), dstMemTy);
1937:     auto srcRegTy = RankedTensorType::get(
1938:         srcMemTy.getShape(), srcMemTy.getElementType(), srcEncoding);
1939:     Value srcReg =
1940:         ttg::LocalLoadOp::create(rewriter, loc, srcRegTy, op.getSrc(), Value())
1941:             .getResult();
1942:     if (!storeFpSanScratchMemory(rewriter, loc, info->ptr, srcReg, srcRegTy))
1943:       return emitFpSanCodegenError(op.getOperation());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。
### Lines 1945-1945

```cpp
1945:     createGlobalScratchBarrier(rewriter, loc);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1947-1949

```cpp
1947:     rewriter.eraseOp(op);
1948:     return success();
1949:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. When the lowered form has no direct result, the original op is erased.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 当降级后的形式不再需要直接结果时，原始操作会被删除。
### Lines 1951-1953

```cpp
1951: private:
1952:   TmemScratchManager *scratch;
1953: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1955-1964

```cpp
1955: struct TCGen5CommitPattern : public OpRewritePattern<ttng::TCGen5CommitOp> {
1956:   using OpRewritePattern::OpRewritePattern;
1957:   LogicalResult matchAndRewrite(ttng::TCGen5CommitOp op,
1958:                                 PatternRewriter &rewriter) const override {
1959:     ttng::ArriveBarrierOp::create(rewriter, op.getLoc(), op.getBarrier(), 1,
1960:                                   op.getPred());
1961:     rewriter.eraseOp(op);
1962:     return success();
1963:   }
1964: };
```

- **EN:** Defines `TCGen5CommitPattern`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. When the lowered form has no direct result, the original op is erased.
- **CN:** 这里定义 `TCGen5CommitPattern`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 当降级后的形式不再需要直接结果时，原始操作会被删除。
### Lines 1966-1967

```cpp
1966: struct WarpGroupDotPattern : public OpRewritePattern<ttng::WarpGroupDotOp> {
1967:   using OpRewritePattern::OpRewritePattern;
```

- **EN:** Defines `WarpGroupDotPattern`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `WarpGroupDotPattern`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1969-1975

```cpp
1969:   LogicalResult matchAndRewrite(ttng::WarpGroupDotOp op,
1970:                                 PatternRewriter &rewriter) const override {
1971:     auto aTy = dyn_cast<ttg::TensorOrMemDesc>(op.getA().getType());
1972:     auto bMemTy = dyn_cast<ttg::MemDescType>(op.getB().getType());
1973:     auto cTy = dyn_cast<RankedTensorType>(op.getC().getType());
1974:     if (!aTy || !bMemTy || !cTy)
1975:       return emitFpSanInvariantError(op.getOperation());
```

- **EN:** Defines `matchAndRewrite`, the core MLIR rewrite callback. It checks whether the source operation matches the pattern and then materializes the lowered replacement. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic.
- **CN:** 这里定义 `matchAndRewrite`，即 MLIR 模式重写的核心回调。它先检查源操作是否匹配模式，再构造降级后的替代结果。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。
### Lines 1977-1982

```cpp
1977:     if (auto aMemTy = dyn_cast<ttg::MemDescType>(op.getA().getType())) {
1978:       if (!isa<ttg::SharedMemorySpaceAttr>(aMemTy.getMemorySpace()))
1979:         return emitFpSanInvariantError(op.getOperation());
1980:     }
1981:     if (!isa<ttg::SharedMemorySpaceAttr>(bMemTy.getMemorySpace()))
1982:       return emitFpSanInvariantError(op.getOperation());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。
### Lines 1984-1990

```cpp
1984:     bool aIsFloat = isa<FloatType>(aTy.getElementType());
1985:     bool bIsFloat = isa<FloatType>(bMemTy.getElementType());
1986:     bool cIsFloat = isa<FloatType>(cTy.getElementType());
1987:     if (!aIsFloat && !bIsFloat && !cIsFloat)
1988:       return failure();
1989:     if (!aIsFloat || !bIsFloat || !cIsFloat)
1990:       return emitFpSanUnsupported(op.getOperation());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1992-1999

```cpp
1992:     if (aTy.getRank() != 2 || bMemTy.getRank() != 2 || cTy.getRank() != 2)
1993:       return emitFpSanUnsupported(op.getOperation());
1994:     auto aShape = aTy.getShape();
1995:     auto bShape = bMemTy.getShape();
1996:     auto cShape = cTy.getShape();
1997:     if (aShape[1] != bShape[0] || aShape[0] != cShape[0] ||
1998:         bShape[1] != cShape[1])
1999:       return emitFpSanInvariantError(op.getOperation());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2001-2004

```cpp
2001:     auto loc = op.getLoc();
2002:     int64_t m = aShape[0];
2003:     int64_t k = aShape[1];
2004:     int64_t n = bShape[1];
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2006-2017

```cpp
2006:     auto *ctx = rewriter.getContext();
2007:     auto accElem =
2008:         IntegerType::get(ctx, cTy.getElementType().getIntOrFloatBitWidth());
2009:     Value useCInt;
2010:     if (op.getUseC()) {
2011:       useCInt = arith::ExtUIOp::create(rewriter, loc, accElem, op.getUseC());
2012:     } else {
2013:       useCInt = arith::ConstantOp::create(rewriter, loc,
2014:                                           rewriter.getIntegerAttr(accElem, 1));
2015:     }
2016:     Value predInt = arith::ConstantOp::create(
2017:         rewriter, loc, rewriter.getIntegerAttr(accElem, 1));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2019-2024

```cpp
2019:     rewriter.setInsertionPoint(op);
2020:     auto aScratch = createWGMMAScratch(rewriter, loc, op.getA());
2021:     auto bScratch = createWGMMAScratch(rewriter, loc, op.getB());
2022:     Value dPtr = createScratchAndStore(rewriter, loc, op.getC(), cTy);
2023:     if (!aScratch || !bScratch || !dPtr)
2024:       return emitFpSanCodegenError(op.getOperation());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2026-2027

```cpp
2026:     int64_t tileM = std::min<int64_t>(kTileM, m);
2027:     int64_t tileN = std::min<int64_t>(kTileN, n);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2029-2040

```cpp
2029:     auto accTileLayout = getOptimizedBlockedEncoding(rewriter, {tileM, tileN},
2030:                                                      cTy.getElementType());
2031:     auto accTileTy = RankedTensorType::get({tileM, tileN}, cTy.getElementType(),
2032:                                            accTileLayout);
2033:     auto aTileLayout =
2034:         getOptimizedBlockedEncoding(rewriter, {tileM, k}, aTy.getElementType());
2035:     auto aTileTy =
2036:         RankedTensorType::get({tileM, k}, aTy.getElementType(), aTileLayout);
2037:     auto bTileLayout = getOptimizedBlockedEncoding(rewriter, {k, tileN},
2038:                                                    bMemTy.getElementType());
2039:     auto bTileTy =
2040:         RankedTensorType::get({k, tileN}, bMemTy.getElementType(), bTileLayout);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2042-2042

```cpp
2042:     createGlobalScratchBarrier(rewriter, loc);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2044-2050

```cpp
2044:     auto mLoop = emitMmaEmulationLoops(
2045:         rewriter, loc, aScratch->ptr, bScratch->ptr, dPtr, m, n, k, tileM,
2046:         tileN, aTileTy, bTileTy, accTileTy, accTileLayout, accElem, useCInt,
2047:         predInt, /*aStride=*/m, /*bStride=*/k, /*dStride=*/m);
2048:     if (!mLoop)
2049:       return emitFpSanUnsupported(op.getOperation());
2050:     rewriter.setInsertionPointAfter(*mLoop);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2052-2052

```cpp
2052:     createGlobalScratchBarrier(rewriter, loc);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2054-2060

```cpp
2054:     Value out = loadScratchStrided2D(rewriter, loc, dPtr, cTy, /*stride1=*/m);
2055:     if (!out)
2056:       return emitFpSanCodegenError(op.getOperation());
2057:     rewriter.replaceOp(op, out);
2058:     return success();
2059:   }
2060: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. It finishes by replacing the original operation with the lowered form.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 它会在转换成功后用降级后的形式替换原始操作。
### Lines 2062-2064

```cpp
2062: struct TCGen5MMAPattern : public OpRewritePattern<ttng::TCGen5MMAOp> {
2063:   TCGen5MMAPattern(MLIRContext *ctx, TmemScratchManager *scratch)
2064:       : OpRewritePattern(ctx), scratch(scratch) {}
```

- **EN:** Defines `TCGen5MMAPattern`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `TCGen5MMAPattern`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 2066-2070

```cpp
2066:   LogicalResult matchAndRewrite(ttng::TCGen5MMAOp op,
2067:                                 PatternRewriter &rewriter) const override {
2068:     auto aMemTy = cast<ttg::MemDescType>(op.getA().getType());
2069:     auto bMemTy = cast<ttg::MemDescType>(op.getB().getType());
2070:     auto dMemTy = cast<ttg::MemDescType>(op.getD().getType());
```

- **EN:** Defines `matchAndRewrite`, the core MLIR rewrite callback. It checks whether the source operation matches the pattern and then materializes the lowered replacement. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic.
- **CN:** 这里定义 `matchAndRewrite`，即 MLIR 模式重写的核心回调。它先检查源操作是否匹配模式，再构造降级后的替代结果。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。
### Lines 2072-2078

```cpp
2072:     bool aIsFloat = isa<FloatType>(aMemTy.getElementType());
2073:     bool bIsFloat = isa<FloatType>(bMemTy.getElementType());
2074:     bool dIsFloat = isa<FloatType>(dMemTy.getElementType());
2075:     if (!aIsFloat && !bIsFloat && !dIsFloat)
2076:       return failure();
2077:     if (!aIsFloat || !bIsFloat || !dIsFloat)
2078:       return emitFpSanUnsupported(op.getOperation());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2080-2083

```cpp
2080:     auto scope = getScratchScopeRegion(op);
2081:     auto dInfo = scratch->getOrCreate(op.getD(), rewriter, scope);
2082:     if (!dInfo)
2083:       return emitFpSanCodegenError(op.getOperation());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2085-2085

```cpp
2085:     auto loc = op.getLoc();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2087-2088

```cpp
2087:     bool aIsTmem = isa<ttng::TensorMemorySpaceAttr>(aMemTy.getMemorySpace());
2088:     bool bIsTmem = isa<ttng::TensorMemorySpaceAttr>(bMemTy.getMemorySpace());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2090-2092

```cpp
2090:     if ((aIsTmem && aMemTy.getRank() != 2) ||
2091:         (bIsTmem && bMemTy.getRank() != 2) || dMemTy.getRank() != 2)
2092:       return emitFpSanUnsupported(op.getOperation());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2094-2102

```cpp
2094:     auto aShape = aMemTy.getShape();
2095:     auto bShape = bMemTy.getShape();
2096:     if (aShape.size() != 2 || bShape.size() != 2)
2097:       return emitFpSanInvariantError(op.getOperation());
2098:     if (aShape[1] != bShape[0])
2099:       return emitFpSanInvariantError(op.getOperation());
2100:     int64_t m = aShape[0];
2101:     int64_t k = aShape[1];
2102:     int64_t n = bShape[1];
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2104-2110

```cpp
2104:     auto *ctx = rewriter.getContext();
2105:     auto accElem =
2106:         IntegerType::get(ctx, dMemTy.getElementType().getIntOrFloatBitWidth());
2107:     Value useDInt =
2108:         arith::ExtUIOp::create(rewriter, loc, accElem, op.getUseD());
2109:     Value predInt =
2110:         arith::ExtUIOp::create(rewriter, loc, accElem, op.getPred());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2112-2120

```cpp
2112:     rewriter.setInsertionPoint(op);
2113:     auto aScratch = createOperandScratch(rewriter, loc, *scratch, op.getA(),
2114:                                          aMemTy, aIsTmem, scope);
2115:     if (!aScratch)
2116:       return emitFpSanCodegenError(op.getOperation());
2117:     auto bScratch = createOperandScratch(rewriter, loc, *scratch, op.getB(),
2118:                                          bMemTy, bIsTmem, scope);
2119:     if (!bScratch)
2120:       return emitFpSanCodegenError(op.getOperation());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2122-2123

```cpp
2122:     int64_t tileM = std::min<int64_t>(kTileM, m);
2123:     int64_t tileN = std::min<int64_t>(kTileN, n);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2125-2136

```cpp
2125:     auto accTileLayout = getOptimizedBlockedEncoding(rewriter, {tileM, tileN},
2126:                                                      dMemTy.getElementType());
2127:     auto accTileTy = RankedTensorType::get(
2128:         {tileM, tileN}, dMemTy.getElementType(), accTileLayout);
2129:     auto aTileLayout = getOptimizedBlockedEncoding(rewriter, {tileM, k},
2130:                                                    aMemTy.getElementType());
2131:     auto aTileTy =
2132:         RankedTensorType::get({tileM, k}, aMemTy.getElementType(), aTileLayout);
2133:     auto bTileLayout = getOptimizedBlockedEncoding(rewriter, {k, tileN},
2134:                                                    bMemTy.getElementType());
2135:     auto bTileTy =
2136:         RankedTensorType::get({k, tileN}, bMemTy.getElementType(), bTileLayout);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2138-2140

```cpp
2138:     // Each warp may only populate a subset of the operand scratch tiles, so
2139:     // synchronize before the emulation loops start reading them.
2140:     createGlobalScratchBarrier(rewriter, loc);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2142-2148

```cpp
2142:     auto mLoop = emitMmaEmulationLoops(
2143:         rewriter, loc, aScratch->ptr, bScratch->ptr, dInfo->ptr, m, n, k, tileM,
2144:         tileN, aTileTy, bTileTy, accTileTy, accTileLayout, accElem, useDInt,
2145:         predInt, /*aStride=*/m, /*bStride=*/k, /*dStride=*/m);
2146:     if (!mLoop)
2147:       return emitFpSanUnsupported(op.getOperation());
2148:     rewriter.setInsertionPointAfter(*mLoop);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2150-2154

```cpp
2150:     // The emulation loop also writes D through scratch memory from multiple
2151:     // warps, so make those stores visible before signaling completion.
2152:     auto postLoopBarrier = ttg::BarrierOp::create(
2153:         rewriter, loc,
2154:         ttg::AddrSpace::GlobalRead | ttg::AddrSpace::GlobalWrite);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2156-2166

```cpp
2156:     if (!op.getBarriers().empty()) {
2157:       OpBuilder::InsertionGuard guard(rewriter);
2158:       rewriter.setInsertionPointAfter(postLoopBarrier);
2159:       auto barriers = op.getBarriers();
2160:       auto barrierPreds = op.getBarrierPreds();
2161:       for (size_t i = 0; i < barriers.size(); ++i) {
2162:         Value pred =
2163:             arith::AndIOp::create(rewriter, loc, op.getPred(), barrierPreds[i]);
2164:         ttng::ArriveBarrierOp::create(rewriter, loc, barriers[i], 1, pred);
2165:       }
2166:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2168-2178

```cpp
2168:     if (op.getNumResults() == 0) {
2169:       rewriter.eraseOp(op);
2170:       return success();
2171:     }
2172:     SmallVector<Value> deps;
2173:     if (op.getAccDep())
2174:       deps.push_back(op.getAccDep());
2175:     Value token = createAsyncToken(rewriter, loc, deps);
2176:     rewriter.replaceOp(op, token);
2177:     return success();
2178:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. It finishes by replacing the original operation with the lowered form. When the lowered form has no direct result, the original op is erased.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 它会在转换成功后用降级后的形式替换原始操作。 当降级后的形式不再需要直接结果时，原始操作会被删除。
### Lines 2180-2182

```cpp
2180: private:
2181:   TmemScratchManager *scratch;
2182: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2184-2187

```cpp
2184: struct TCGen5MMAScaledPattern
2185:     : public OpRewritePattern<ttng::TCGen5MMAScaledOp> {
2186:   TCGen5MMAScaledPattern(MLIRContext *ctx, TmemScratchManager *scratch)
2187:       : OpRewritePattern(ctx), scratch(scratch) {}
```

- **EN:** Defines `TCGen5MMAScaledPattern`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `TCGen5MMAScaledPattern`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 2189-2195

```cpp
2189:   LogicalResult matchAndRewrite(ttng::TCGen5MMAScaledOp op,
2190:                                 PatternRewriter &rewriter) const override {
2191:     auto aMemTy = cast<ttg::MemDescType>(op.getA().getType());
2192:     auto bMemTy = cast<ttg::MemDescType>(op.getB().getType());
2193:     auto dMemTy = cast<ttg::MemDescType>(op.getD().getType());
2194:     auto aScaleMemTy = cast<ttg::MemDescType>(op.getAScale().getType());
2195:     auto bScaleMemTy = cast<ttg::MemDescType>(op.getBScale().getType());
```

- **EN:** Defines `matchAndRewrite`, the core MLIR rewrite callback. It checks whether the source operation matches the pattern and then materializes the lowered replacement. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic.
- **CN:** 这里定义 `matchAndRewrite`，即 MLIR 模式重写的核心回调。它先检查源操作是否匹配模式，再构造降级后的替代结果。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。
### Lines 2197-2198

```cpp
2197:     bool aIsTmem = isa<ttng::TensorMemorySpaceAttr>(aMemTy.getMemorySpace());
2198:     bool bIsTmem = isa<ttng::TensorMemorySpaceAttr>(bMemTy.getMemorySpace());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2200-2204

```cpp
2200:     if ((aIsTmem && aMemTy.getRank() != 2) ||
2201:         (bIsTmem && bMemTy.getRank() != 2) || (aScaleMemTy.getRank() != 2) ||
2202:         (bScaleMemTy.getRank() != 2) || dMemTy.getRank() != 2) {
2203:       return emitFpSanUnsupported(op.getOperation());
2204:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2206-2213

```cpp
2206:     auto aShape = aMemTy.getShape();
2207:     auto bShape = bMemTy.getShape();
2208:     auto dShape = dMemTy.getShape();
2209:     auto aScaleShape = aScaleMemTy.getShape();
2210:     auto bScaleShape = bScaleMemTy.getShape();
2211:     if (aShape.size() != 2 || bShape.size() != 2 || dShape.size() != 2 ||
2212:         aScaleShape.size() != 2 || bScaleShape.size() != 2)
2213:       return emitFpSanInvariantError(op.getOperation());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2215-2232

```cpp
2215:     int64_t m = dShape[0];
2216:     int64_t n = dShape[1];
2217:     int64_t aPackedK = aShape[1];
2218:     int64_t bPackedK = bShape[0];
2219:     int64_t aKPackFactor = 1;
2220:     int64_t bKPackFactor = 1;
2221:     if (op.getAType() == tt::ScaleDotElemType::E2M1) {
2222:       if (op.getBlockK() == aPackedK * 2) {
2223:         aKPackFactor = 2;
2224:       } else {
2225:         return emitFpSanInvariantError(op.getOperation());
2226:       }
2227:     }
2228:     if (op.getBType() == tt::ScaleDotElemType::E2M1) {
2229:       if (op.getBlockK() == bPackedK * 2) {
2230:         bKPackFactor = 2;
2231:       } else {
2232:         return emitFpSanInvariantError(op.getOperation());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2233-2234

```cpp
2233:       }
2234:     }
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 2236-2238

```cpp
2236:     int64_t k = aPackedK * aKPackFactor;
2237:     if (aShape[0] != m || bShape[1] != n || k != bPackedK * bKPackFactor)
2238:       return emitFpSanInvariantError(op.getOperation());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2240-2250

```cpp
2240:     auto deduceScaleFactor = [&](ArrayRef<int64_t> scaleShape,
2241:                                  int64_t rows) -> std::optional<int64_t> {
2242:       if (scaleShape[0] != rows || scaleShape[1] <= 0 ||
2243:           (k % scaleShape[1]) != 0)
2244:         return std::nullopt;
2245:       return k / scaleShape[1];
2246:     };
2247:     auto aScaleFactor = deduceScaleFactor(aScaleShape, m);
2248:     auto bScaleFactor = deduceScaleFactor(bScaleShape, n);
2249:     if (!aScaleFactor || !bScaleFactor)
2250:       return emitFpSanInvariantError(op.getOperation());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2252-2255

```cpp
2252:     auto scope = getScratchScopeRegion(op);
2253:     auto dInfo = scratch->getOrCreate(op.getD(), rewriter, scope);
2254:     if (!dInfo)
2255:       return emitFpSanCodegenError(op.getOperation());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2257-2264

```cpp
2257:     auto loc = op.getLoc();
2258:     auto *ctx = rewriter.getContext();
2259:     auto accElem =
2260:         IntegerType::get(ctx, dMemTy.getElementType().getIntOrFloatBitWidth());
2261:     Value useDInt =
2262:         arith::ExtUIOp::create(rewriter, loc, accElem, op.getUseD());
2263:     Value predInt =
2264:         arith::ExtUIOp::create(rewriter, loc, accElem, op.getPred());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2266-2282

```cpp
2266:     rewriter.setInsertionPoint(op);
2267:     auto aScratch = createOperandScratch(rewriter, loc, *scratch, op.getA(),
2268:                                          aMemTy, aIsTmem, scope);
2269:     if (!aScratch)
2270:       return emitFpSanCodegenError(op.getOperation());
2271:     auto bScratch = createOperandScratch(rewriter, loc, *scratch, op.getB(),
2272:                                          bMemTy, bIsTmem, scope);
2273:     if (!bScratch)
2274:       return emitFpSanCodegenError(op.getOperation());
2275:     auto aScaleScratch = createOperandScratch(
2276:         rewriter, loc, *scratch, op.getAScale(), aScaleMemTy, true, scope);
2277:     if (!aScaleScratch)
2278:       return emitFpSanCodegenError(op.getOperation());
2279:     auto bScaleScratch = createOperandScratch(
2280:         rewriter, loc, *scratch, op.getBScale(), bScaleMemTy, true, scope);
2281:     if (!bScaleScratch)
2282:       return emitFpSanCodegenError(op.getOperation());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2284-2285

```cpp
2284:     int64_t tileM = std::min<int64_t>(kTileM, m);
2285:     int64_t tileN = std::min<int64_t>(kTileN, n);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2287-2298

```cpp
2287:     auto accTileLayout = getOptimizedBlockedEncoding(rewriter, {tileM, tileN},
2288:                                                      dMemTy.getElementType());
2289:     auto accTileTy = RankedTensorType::get(
2290:         {tileM, tileN}, dMemTy.getElementType(), accTileLayout);
2291:     auto aTileLayout = getOptimizedBlockedEncoding(rewriter, {tileM, aPackedK},
2292:                                                    aMemTy.getElementType());
2293:     auto aTileTy = RankedTensorType::get({tileM, aPackedK},
2294:                                          aMemTy.getElementType(), aTileLayout);
2295:     auto bTileLayout = getOptimizedBlockedEncoding(rewriter, {bPackedK, tileN},
2296:                                                    bMemTy.getElementType());
2297:     auto bTileTy = RankedTensorType::get({bPackedK, tileN},
2298:                                          bMemTy.getElementType(), bTileLayout);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2300-2316

```cpp
2300:     DotScaleConfig scale;
2301:     scale.aElemType = op.getAType();
2302:     scale.bElemType = op.getBType();
2303:     scale.computeElem = getDotScaledComputeFloatType(rewriter, scale.aElemType,
2304:                                                      scale.bElemType);
2305:     scale.aScalePtr = aScaleScratch->ptr;
2306:     scale.bScalePtr = bScaleScratch->ptr;
2307:     scale.aScaleTileTy = RankedTensorType::get(
2308:         {tileM, 1}, aScaleMemTy.getElementType(), accTileLayout);
2309:     scale.bScaleTileTy = RankedTensorType::get(
2310:         {1, tileN}, bScaleMemTy.getElementType(), accTileLayout);
2311:     scale.aScaleStride = aScaleShape[0];
2312:     scale.bScaleStride = bScaleShape[0];
2313:     scale.aKPackFactor = aKPackFactor;
2314:     scale.bKPackFactor = bKPackFactor;
2315:     scale.aScaleFactor = *aScaleFactor;
2316:     scale.bScaleFactor = *bScaleFactor;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2318-2320

```cpp
2318:     // The operand and scale scratch buffers are written cooperatively, so all
2319:     // warps must finish those stores before the emulation loop reads them.
2320:     createGlobalScratchBarrier(rewriter, loc);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2322-2328

```cpp
2322:     auto mLoop = emitMmaEmulationLoops(
2323:         rewriter, loc, aScratch->ptr, bScratch->ptr, dInfo->ptr, m, n, k, tileM,
2324:         tileN, aTileTy, bTileTy, accTileTy, accTileLayout, accElem, useDInt,
2325:         predInt, /*aStride=*/m, /*bStride=*/bPackedK, /*dStride=*/m, scale);
2326:     if (!mLoop)
2327:       return emitFpSanUnsupported(op.getOperation());
2328:     rewriter.setInsertionPointAfter(*mLoop);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2330-2334

```cpp
2330:     // The emulated MMA updates the accumulator scratch cooperatively as well.
2331:     // Flush those stores before completion barriers or later TMEM loads.
2332:     auto postLoopBarrier = ttg::BarrierOp::create(
2333:         rewriter, loc,
2334:         ttg::AddrSpace::GlobalRead | ttg::AddrSpace::GlobalWrite);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2336-2346

```cpp
2336:     if (!op.getBarriers().empty()) {
2337:       OpBuilder::InsertionGuard guard(rewriter);
2338:       rewriter.setInsertionPointAfter(postLoopBarrier);
2339:       auto barriers = op.getBarriers();
2340:       auto barrierPreds = op.getBarrierPreds();
2341:       for (size_t i = 0; i < barriers.size(); ++i) {
2342:         Value pred =
2343:             arith::AndIOp::create(rewriter, loc, op.getPred(), barrierPreds[i]);
2344:         ttng::ArriveBarrierOp::create(rewriter, loc, barriers[i], 1, pred);
2345:       }
2346:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2348-2358

```cpp
2348:     if (op.getNumResults() == 0) {
2349:       rewriter.eraseOp(op);
2350:       return success();
2351:     }
2352:     SmallVector<Value> deps;
2353:     if (op.getAccDep())
2354:       deps.push_back(op.getAccDep());
2355:     Value token = createAsyncToken(rewriter, loc, deps);
2356:     rewriter.replaceOp(op, token);
2357:     return success();
2358:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. It finishes by replacing the original operation with the lowered form. When the lowered form has no direct result, the original op is erased.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 它会在转换成功后用降级后的形式替换原始操作。 当降级后的形式不再需要直接结果时，原始操作会被删除。
### Lines 2360-2362

```cpp
2360: private:
2361:   TmemScratchManager *scratch;
2362: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2364-2366

```cpp
2364: template <typename OpTy> struct UnaryPattern : public OpRewritePattern<OpTy> {
2365:   UnaryPattern(MLIRContext *context, UnaryOpId unaryOpId)
2366:       : OpRewritePattern<OpTy>(context), unaryOpId(unaryOpId) {}
```

- **EN:** Defines `UnaryPattern`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `UnaryPattern`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 2368-2375

```cpp
2368:   LogicalResult matchAndRewrite(OpTy op,
2369:                                 PatternRewriter &rewriter) const override {
2370:     if (!isFloatLike(op.getType()))
2371:       return failure();
2372:     rewriter.replaceOp(op, fpsanUnaryTagged(rewriter, op.getLoc(),
2373:                                             op.getOperand(), unaryOpId));
2374:     return success();
2375:   }
```

- **EN:** Defines `matchAndRewrite`, the core MLIR rewrite callback. It checks whether the source operation matches the pattern and then materializes the lowered replacement. It finishes by replacing the original operation with the lowered form. The code uses `LogicalResult` to stay conservative when preconditions are not met.
- **CN:** 这里定义 `matchAndRewrite`，即 MLIR 模式重写的核心回调。它先检查源操作是否匹配模式，再构造降级后的替代结果。 它会在转换成功后用降级后的形式替换原始操作。 代码通过 `LogicalResult` 在前提不满足时保持保守处理。
### Lines 2377-2379

```cpp
2377: private:
2378:   UnaryOpId unaryOpId;
2379: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2381-2383

```cpp
2381: struct ExternElementwisePattern
2382:     : public OpRewritePattern<tt::ExternElementwiseOp> {
2383:   using OpRewritePattern::OpRewritePattern;
```

- **EN:** Defines `ExternElementwisePattern`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `ExternElementwisePattern`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 2385-2389

```cpp
2385:   LogicalResult matchAndRewrite(tt::ExternElementwiseOp op,
2386:                                 PatternRewriter &rewriter) const override {
2387:     if (!op.getPure() || !isFloatLike(op.getType()) ||
2388:         op.getNumOperands() == 0 || !externHasNumericOperands(op))
2389:       return failure();
```

- **EN:** Defines `matchAndRewrite`, the core MLIR rewrite callback. It checks whether the source operation matches the pattern and then materializes the lowered replacement.
- **CN:** 这里定义 `matchAndRewrite`，即 MLIR 模式重写的核心回调。它先检查源操作是否匹配模式，再构造降级后的替代结果。
### Lines 2391-2398

```cpp
2391:     uint64_t hash = stableStringHash(op.getSymbol());
2392:     Value result = fpsanVariadicExternTagged(rewriter, op.getLoc(), op, hash);
2393:     if (!result)
2394:       return emitFpSanCodegenError(op.getOperation());
2395:     rewriter.replaceOp(op, result);
2396:     return success();
2397:   }
2398: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. It finishes by replacing the original operation with the lowered form.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 它会在转换成功后用降级后的形式替换原始操作。
### Lines 2400-2410

```cpp
2400: class FpSanitizerPass
2401:     : public impl::TritonInstrumentFpSanitizerBase<FpSanitizerPass> {
2402: public:
2403:   void runOnOperation() override {
2404:     bool fpSanErrorEmitted = false;
2405:     ScopedDiagnosticHandler diagnosticHandler(
2406:         &getContext(), [&](Diagnostic &diagnostic) {
2407:           if (diagnostic.getSeverity() == DiagnosticSeverity::Error)
2408:             fpSanErrorEmitted = true;
2409:           return failure();
2410:         });
```

- **EN:** Defines `FpSanitizerPass`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `FpSanitizerPass`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 2412-2429

```cpp
2412:     TmemScratchManager scratch;
2413:     RewritePatternSet patterns(&getContext());
2414:     patterns.add<BinaryFloatToIntPattern<arith::AddFOp, arith::AddIOp>,
2415:                  BinaryFloatToIntPattern<arith::SubFOp, arith::SubIOp>,
2416:                  BinaryFloatToIntPattern<arith::MulFOp, arith::MulIOp>,
2417:                  BinaryFloatToIntPattern<arith::MinimumFOp, arith::MinSIOp>,
2418:                  BinaryFloatToIntPattern<arith::MaximumFOp, arith::MaxSIOp>,
2419:                  BinaryFloatToIntPattern<arith::MinNumFOp, arith::MinSIOp>,
2420:                  BinaryFloatToIntPattern<arith::MaxNumFOp, arith::MaxSIOp>,
2421:                  NegFOpPattern, DivFOpPattern, PreciseDivFOpPattern,
2422:                  RemFOpPattern, FmaPattern, ExpOpPattern, Exp2OpPattern,
2423:                  CosOpPattern, SinOpPattern, ExtFOpPattern, TruncFOpPattern,
2424:                  FpToFpPattern, Fp4ToFpPattern, DotPattern, DotScaledPattern>(
2425:         &getContext());
2426:     patterns.add<UnaryPattern<math::LogOp>>(&getContext(), UnaryOpId::Log);
2427:     patterns.add<UnaryPattern<math::Log2Op>>(&getContext(), UnaryOpId::Log2);
2428:     patterns.add<UnaryPattern<math::SqrtOp>>(&getContext(), UnaryOpId::Sqrt);
2429:     patterns.add<UnaryPattern<math::RsqrtOp>>(&getContext(), UnaryOpId::Rsqrt);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2430-2440

```cpp
2430:     patterns.add<UnaryPattern<math::ErfOp>>(&getContext(), UnaryOpId::Erf);
2431:     patterns.add<UnaryPattern<math::FloorOp>>(&getContext(), UnaryOpId::Floor);
2432:     patterns.add<UnaryPattern<math::CeilOp>>(&getContext(), UnaryOpId::Ceil);
2433:     patterns.add<UnaryPattern<tt::PreciseSqrtOp>>(&getContext(),
2434:                                                   UnaryOpId::PreciseSqrt);
2435:     patterns.add<ExternElementwisePattern>(&getContext());
2436:     patterns.add<TMEMLoadPattern, TMEMStorePattern, TMEMCopyPattern,
2437:                  TCGen5MMAPattern, TCGen5MMAScaledPattern>(&getContext(),
2438:                                                            &scratch);
2439:     patterns.add<WarpGroupDotPattern>(&getContext());
2440:     patterns.add<TCGen5CommitPattern>(&getContext());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2442-2449

```cpp
2442:     LogicalResult result =
2443:         applyPatternsGreedily(getOperation(), std::move(patterns));
2444:     if (failed(result)) {
2445:       llvm::errs() << "FpSanitizer error: Failed to apply patterns\n";
2446:       signalPassFailure();
2447:     }
2448:     if (fpSanErrorEmitted)
2449:       signalPassFailure();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2451-2454

```cpp
2451:     // TODO: Remove unused tmem usages. This requires unwiring them from the
2452:     // warp specialize partitions.
2453:   }
2454: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2456-2456

```cpp
2456: } // namespace
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 2458-2460

```cpp
2458: } // namespace instrument
2459: } // namespace triton
2460: } // namespace mlir
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。

## Key Concepts / 关键概念
- **EN:** The main concern is pass-driven transformation around fp sanitizer.
  **CN:** 核心关注点是围绕 FP Sanitizer 的 pass 驱动变换。
- **EN:** Pattern rewriting is the main mechanism for canonicalization and lowering in this file.
  **CN:** 模式重写是本文件进行规范化与降级转换的核心机制。
- **EN:** Memory descriptor types carry layout, rank, and address-space information.
  **CN:** 内存描述符类型携带布局、维度和地址空间信息。
- **EN:** Tensor shape and element-type reasoning is central to the implementation.
  **CN:** 张量形状与元素类型推导是实现中的核心内容。
- **EN:** Encoding attributes describe how distributed GPU data is laid out across threads and warps.
  **CN:** 编码属性描述 GPU 分布式数据在线程与 warp 之间的布局方式。
- **EN:** Tensor memory specific allocation, layout, or synchronization rules are important here.
  **CN:** 这里重点处理张量内存（TMEM）的分配、布局或同步规则。

## Dependencies / 依赖关系
- **Internal Triton headers / Triton 内部头文件:** `triton/Dialect/Triton/IR/Dialect.h`, `triton/Dialect/TritonGPU/IR/Dialect.h`, `triton/Dialect/TritonGPU/Transforms/Utility.h`, `triton/Dialect/TritonInstrument/IR/Dialect.h`, `triton/Dialect/TritonInstrument/IR/Utility.h`, `triton/Dialect/TritonInstrument/Transforms/Passes.h`, ... (+2 more)
- **MLIR headers / MLIR 头文件:** `mlir/IR/Diagnostics.h`, `mlir/IR/PatternMatch.h`, `mlir/IR/Types.h`, `mlir/Transforms/GreedyPatternRewriteDriver.h`, `mlir/Transforms/RegionUtils.h`
- **LLVM headers / LLVM 头文件:** `llvm/ADT/DenseSet.h`, `llvm/ADT/STLExtras.h`
- **Standard/library headers / 标准或通用库头文件:** `cassert`
- **Generated fragments / 生成片段:** `triton/Dialect/TritonInstrument/Transforms/Passes.h.inc`
- **Primary APIs used / 主要 API:** `PatternRewriter`, `RankedTensorType`, `MemDescType`, `OpBuilder`
- **Pipeline role / 流程角色:** This file participates in Triton pass pipelines and usually expects upstream analyses or dialect invariants to have prepared the IR. / 本文件参与 Triton 的 pass 流程，通常依赖上游分析或方言不变量先把 IR 准备好。
