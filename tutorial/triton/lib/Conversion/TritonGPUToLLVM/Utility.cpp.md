# Utility.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `lib/Conversion/TritonGPUToLLVM/Utility.cpp`
- **Purpose / 作用:** **EN:** Lowers TritonGPU constructs related to Utility into LLVM-compatible IR and rewrite patterns. **CN:** 把与 Utility 相关的 TritonGPU 构造降级为 LLVM 兼容 IR 与重写模式。

## Line-by-Line Analysis / 逐行分析

### Lines 1-17

```cpp
   1: #include "triton/Conversion/TritonGPUToLLVM/Utility.h"
   2: #include "mlir/Dialect/LLVMIR/LLVMDialect.h"
   3: #include "mlir/IR/Attributes.h"
   4: #include "mlir/Transforms/RegionUtils.h"
   5: #include "triton/Analysis/Allocation.h"
   6: #include "triton/Analysis/Utility.h"
   7: #include "triton/Conversion/TritonGPUToLLVM/TargetInfoBase.h"
   8: #include "triton/Dialect/Triton/IR/Dialect.h"
   9: #include "triton/Dialect/TritonGPU/IR/Attributes.h"
  10: #include "triton/Dialect/TritonGPU/IR/LinearLayoutConversions.h"
  11: #include "triton/Dialect/TritonGPU/Transforms/Utility.h"
  12: #include "triton/Dialect/TritonNvidiaGPU/IR/Dialect.h"
  13: #include "triton/Tools/GenericSwizzling.h"
  14: #include "triton/Tools/LayoutUtils.h"
  15: #include "triton/Tools/LinearLayout.h"
  16: #include "llvm/ADT/STLExtras.h"
  17: #include "llvm/Support/MathExtras.h"
```

- **EN:** Includes the interfaces this file depends on. Triton headers (`Utility.h`, `Allocation.h`, `Utility.h`, `TargetInfoBase.h`, ... (+8 more)) provide domain-specific IR/support, MLIR headers (`LLVMDialect.h`, `Attributes.h`, `RegionUtils.h`) provide rewriting and analysis infrastructure, LLVM headers (`STLExtras.h`, `MathExtras.h`) supply low-level utilities, and standard/library headers (None) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（`Utility.h`, `Allocation.h`, `Utility.h`, `TargetInfoBase.h`, ... (+8 more)）提供领域特定 IR/支持逻辑，MLIR 头文件（`LLVMDialect.h`, `Attributes.h`, `RegionUtils.h`）提供重写与分析基础设施，LLVM 头文件（`STLExtras.h`, `MathExtras.h`）提供底层工具，而标准/通用库头文件（None）提供通用能力。
### Lines 19-19

```cpp
  19: #include <functional>
```

- **EN:** Includes the interfaces this file depends on. Triton headers (None) provide domain-specific IR/support, MLIR headers (None) provide rewriting and analysis infrastructure, LLVM headers (None) supply low-level utilities, and standard/library headers (`functional`) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（None）提供领域特定 IR/支持逻辑，MLIR 头文件（None）提供重写与分析基础设施，LLVM 头文件（None）提供底层工具，而标准/通用库头文件（`functional`）提供通用能力。
### Lines 21-23

```cpp
  21: #if defined(_MSC_VER) && !defined(__clang__)
  22: // from https://gist.github.com/pps83/3210a2f980fd02bb2ba2e5a1fc4a2ef0
  23: #include <intrin.h>
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 25-29

```cpp
  25: static int __builtin_clz(unsigned x) {
  26:   unsigned long r;
  27:   _BitScanReverse(&r, x);
  28:   return static_cast<int>(r ^ 31);
  29: }
```

- **EN:** Defines `__builtin_clz`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `__builtin_clz`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 31-35

```cpp
  31: static int __builtin_ctz(unsigned x) {
  32:   unsigned long r;
  33:   _BitScanForward(&r, x);
  34:   return static_cast<int>(r);
  35: }
```

- **EN:** Defines `__builtin_ctz`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `__builtin_ctz`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 37-37

```cpp
  37: #endif
```

- **EN:** Defines or closes a file guard so this header/TableGen fragment is included only once during compilation.
- **CN:** 这里定义或关闭文件保护宏，确保该头文件/TableGen 片段在编译过程中只被包含一次。
### Lines 39-39

```cpp
  39: namespace mlir {
```

- **EN:** Opens or closes the namespace nesting for mlir, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 mlir 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 41-41

```cpp
  41: namespace triton::gpu {
```

- **EN:** Opens or closes the namespace nesting for triton::gpu, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 triton::gpu 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 43-49

```cpp
  43: std::pair<SmallVector<LocalMemOpTile>, SmallVector<LocalMemOpTile>>
  44: getSrcDstTiles(const TargetInfoBase &targetInfo, int bitwidth,
  45:                bool crossCTALoads) {
  46:   assert(bitwidth <= 128 && "bitwidth must be <= 128");
  47:   assert(llvm::isPowerOf2_32(bitwidth) && "bitwidth must be a power of two");
  48:   SmallVector<LocalMemOpTile> src;
  49:   SmallVector<LocalMemOpTile> dst;
```

- **EN:** Defines accessor/helper `getSrcDstTiles` that exposes or updates operation state in a compact, reusable way. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义访问器/辅助函数 `getSrcDstTiles`，以紧凑且可复用的方式读取或更新操作状态。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 51-54

```cpp
  51:   // ld.shared/st.shared
  52:   auto ldstshared = LocalMemOpTile{{}, {0, 1, 2}};
  53:   src.push_back(ldstshared);
  54:   dst.push_back(ldstshared);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 56-73

```cpp
  56:   if (targetInfo.supportLdMatrix() || targetInfo.supportStMatrix()) {
  57:     // ldmatrix/stmatrix
  58:     if (bitwidth <= 32) {
  59:       auto ldstmatrix = LocalMemOpTile{{0, 1}, {2, 3, 4}};
  60:       if (targetInfo.supportStMatrix()) {
  61:         src.push_back(ldstmatrix);
  62:       }
  63:       // We do cross-CTA reads but in-CTA writes
  64:       // ldmatrix/stmatrix do not support cross-CTA transfers.
  65:       if (!crossCTALoads && targetInfo.supportLdMatrix()) {
  66:         dst.push_back(ldstmatrix);
  67:       }
  68:     }
  69:     // ldmatrix.trans/stmatrix.trans
  70:     if (bitwidth == 16) {
  71:       auto ldstmatrixtrans = LocalMemOpTile{{2, 3, 4}, {0, 1}};
  72:       if (targetInfo.supportStMatrix()) {
  73:         src.push_back(ldstmatrixtrans);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 74-81

```cpp
  74:       }
  75:       if (!crossCTALoads && targetInfo.supportLdMatrix()) {
  76:         dst.push_back(ldstmatrixtrans);
  77:       }
  78:     }
  79:   }
  80:   return {std::move(src), std::move(dst)};
  81: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 83-86

```cpp
  83: Type getFunctionType(Type resultType, ValueRange operands) {
  84:   SmallVector<Type> operandTypes(operands.getTypes());
  85:   return LLVM::LLVMFunctionType::get(resultType, operandTypes);
  86: }
```

- **EN:** Defines accessor/helper `getFunctionType` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getFunctionType`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 88-92

```cpp
  88: LLVM::LLVMFuncOp appendOrGetExternFuncOp(RewriterBase &rewriter, Operation *op,
  89:                                          StringRef funcName, Type funcType,
  90:                                          StringRef libname /*= ""*/,
  91:                                          StringRef libpath /*= ""*/) {
  92:   using LLVM::LLVMFuncOp;
```

- **EN:** Defines `appendOrGetExternFuncOp`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `appendOrGetExternFuncOp`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 94-97

```cpp
  94:   auto funcAttr = StringAttr::get(op->getContext(), funcName);
  95:   Operation *funcOp = SymbolTable::lookupNearestSymbolFrom(op, funcAttr);
  96:   if (funcOp)
  97:     return cast<LLVMFuncOp>(*funcOp);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 99-109

```cpp
  99:   Operation *parent = op;
 100:   if (!isa<LLVM::LLVMFuncOp>(op))
 101:     parent = op->getParentOfType<LLVM::LLVMFuncOp>();
 102:   OpBuilder b(parent);
 103:   auto ret = LLVMFuncOp::create(b, op->getLoc(), funcName, funcType);
 104:   ret.getOperation()->setAttr("libname",
 105:                               StringAttr::get(op->getContext(), libname));
 106:   ret.getOperation()->setAttr("libpath",
 107:                               StringAttr::get(op->getContext(), libpath));
 108:   return ret;
 109: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 111-124

```cpp
 111: Value matrixVectorProd(TritonLLVMOpBuilder &b, const LinearLayout &A, Value x) {
 112:   assert(A.getNumInDims() == 1);
 113:   assert(A.getNumOutDims() == 1);
 114:   auto flatten = [](const std::vector<std::vector<int32_t>> &matrix) {
 115:     SmallVector<int32_t> ret;
 116:     for (const auto &row : matrix) {
 117:       ret.push_back(row[0]);
 118:     }
 119:     return ret;
 120:   };
 121:   auto nCol = A.getTotalInDimSizeLog2();
 122:   auto nRow = A.getTotalOutDimSizeLog2();
 123:   SmallVector<int32_t> matrix = flatten(A.getBases().begin()->second);
 124:   assert(matrix.size() == nCol);
```

- **EN:** Defines `matrixVectorProd`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `matrixVectorProd`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 126-141

```cpp
 126:   // Row-wise popcount to detect rows that appear exactly once across columns.
 127:   uint32_t rowsUnique = 0;
 128:   {
 129:     SmallVector<int> rowPopCnt(nRow, 0);
 130:     for (int c = 0; c < nCol; ++c) {
 131:       uint32_t colBits = matrix[c];
 132:       for (int r = 0; r < nRow; ++r) {
 133:         if (colBits & (1u << r))
 134:           ++rowPopCnt[r];
 135:       }
 136:     }
 137:     for (int r = 0; r < nRow; ++r) {
 138:       if (rowPopCnt[r] == 1)
 139:         rowsUnique |= 1u << r;
 140:     }
 141:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 143-159

```cpp
 143:   // We iterate the matrix following the diagonals and build
 144:   // (x & mask_i) << s_i terms. Prefer OR for diagonals whose rows are unique,
 145:   // then XOR everything else. This tends to encourage mad.lo codegen.
 146:   auto getMaskAndAllRowsUnique = [&](int i) -> std::pair<uint32_t, bool> {
 147:     uint32_t mask = 0;
 148:     int row = i < 0 ? -i : 0;
 149:     int col = i < 0 ? 0 : i;
 150:     bool allRowsUnique = true;
 151:     while (row < nRow && col < nCol) {
 152:       uint32_t bitValue = (matrix[col] >> row) & 1u;
 153:       mask |= bitValue << col;
 154:       allRowsUnique &= ((rowsUnique >> row) & 1u) == 1u;
 155:       ++row;
 156:       ++col;
 157:     }
 158:     return {mask, allRowsUnique};
 159:   };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 161-161

```cpp
 161:   uint32_t explicitCols = 0;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 163-180

```cpp
 163:   {
 164:     SmallVector<uint32_t> masks;
 165:     for (int i = -nRow + 1; i < nCol; i++) {
 166:       masks.push_back(std::get<0>(getMaskAndAllRowsUnique(i)));
 167:     }
 168:     bool reachedFixedPoint = false;
 169:     while (!reachedFixedPoint) {
 170:       reachedFixedPoint = true;
 171:       for (uint32_t m : masks) {
 172:         uint32_t c = m & ~explicitCols;
 173:         if (llvm::isPowerOf2_32(c)) {
 174:           // found a single-element diagonal
 175:           explicitCols |= c;
 176:           reachedFixedPoint = false;
 177:         }
 178:       }
 179:     }
 180:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 182-198

```cpp
 182:   // handle any diagonals that have survived
 183:   SmallVector<Value> ors;
 184:   SmallVector<Value> xors;
 185:   for (int i = -nRow + 1; i < nCol; i++) {
 186:     auto [mask, allRowsUnique] = getMaskAndAllRowsUnique(i);
 187:     mask &= ~explicitCols;
 188:     if (mask == 0)
 189:       continue;
 190:     auto masked = b.and_(x, b.i32_val(mask));
 191:     auto shifted = i >= 0 ? Value(b.lshr(masked, b.i32_val(i)))
 192:                           : Value(b.shl(masked, b.i32_val(-i)));
 193:     if (allRowsUnique) {
 194:       ors.push_back(shifted);
 195:     } else {
 196:       xors.push_back(shifted);
 197:     }
 198:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 200-216

```cpp
 200:   // handle any explicit columns:
 201:   Value zero = b.i32_val(0);
 202:   for (int i = 0; i < nCol; i++) {
 203:     if ((explicitCols >> i) & 1) {
 204:       int32_t basis = matrix[i];
 205:       if (basis == 0)
 206:         continue;
 207:       Value term;
 208:       Value bit = b.and_(x, b.i32_val(1 << i));
 209:       uint32_t basisBits = basis;
 210:       if (llvm::isPowerOf2_32(basisBits)) {
 211:         unsigned row = llvm::countr_zero(basisBits);
 212:         unsigned col = i;
 213:         if (row == col)
 214:           term = bit;
 215:         else if (row > col)
 216:           term = b.shl(bit, b.i32_val(row - col));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 217-229

```cpp
 217:         else
 218:           term = b.lshr(bit, b.i32_val(col - row));
 219:       } else {
 220:         Value bit_is_zero = b.icmp_eq(bit, zero);
 221:         term = b.select(bit_is_zero, zero, b.i32_val(basis));
 222:       }
 223:       if ((rowsUnique & basis) == basis) {
 224:         ors.push_back(term);
 225:       } else {
 226:         xors.push_back(term);
 227:       }
 228:     }
 229:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 231-244

```cpp
 231:   auto treeReduce = [&](SmallVector<Value> &terms,
 232:                         std::function<Value(Value, Value)> op) -> Value {
 233:     if (terms.empty())
 234:       return b.i32_val(0);
 235:     while (terms.size() > 1) {
 236:       SmallVector<Value> next;
 237:       for (size_t i = 0; i + 1 < terms.size(); i += 2)
 238:         next.push_back(op(terms[i], terms[i + 1]));
 239:       if (terms.size() % 2 == 1)
 240:         next.push_back(terms.back());
 241:       terms = std::move(next);
 242:     }
 243:     return terms[0];
 244:   };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 246-251

```cpp
 246:   auto orPart = treeReduce(
 247:       ors, [&b](Value x, Value y) { return b.or_(x, y, /*disjoint=*/true); });
 248:   auto xorPart =
 249:       treeReduce(xors, [&b](Value x, Value y) { return b.xor_(x, y); });
 250:   return b.or_(orPart, xorPart, /*disjoint=*/true);
 251: }
```

- **EN:** Defines `treeReduce`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `treeReduce`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 253-255

```cpp
 253: bool cvtAlwaysUseWarpShuffle(ConvertLayoutOp cvt) {
 254:   return cvt->getParentOp()->hasAttrOfType<UnitAttr>("always_use_warp_shuffle");
 255: }
```

- **EN:** Defines `cvtAlwaysUseWarpShuffle`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `cvtAlwaysUseWarpShuffle`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 257-263

```cpp
 257: Value maybeAnd(OpBuilder &builder, Location loc, Value a, Value b) {
 258:   auto tb = TritonLLVMOpBuilder(loc, builder);
 259:   if (a && b) {
 260:     return tb.and_(a, b);
 261:   }
 262:   return a ? a : b;
 263: }
```

- **EN:** Defines `maybeAnd`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `maybeAnd`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 265-273

```cpp
 265: Value emitRedundantThreadPredicate(
 266:     const llvm::MapVector<StringAttr, int32_t> &freeVarMasks,
 267:     ConversionPatternRewriter &rewriter, Location loc,
 268:     const TargetInfoBase &targetInfo) {
 269:   auto b = TritonLLVMOpBuilder(loc, rewriter);
 270:   auto ctx = rewriter.getContext();
 271:   auto kLane = str_attr("lane");
 272:   auto kWarp = str_attr("warp");
 273:   auto kBlock = str_attr("block");
```

- **EN:** Defines `emitRedundantThreadPredicate`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `emitRedundantThreadPredicate`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 275-279

```cpp
 275:   Value zero = b.i32_val(0);
 276:   auto [laneId, warpId] = getLaneAndWarpId(rewriter, loc);
 277:   Value blockId = freeVarMasks.lookup(kBlock) == 0
 278:                       ? zero
 279:                       : targetInfo.getClusterCTAId(rewriter, loc);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 281-292

```cpp
 281:   Value pred;
 282:   auto dimNames = {kLane, kWarp, kBlock};
 283:   auto dimIds = {laneId, warpId, blockId};
 284:   for (auto [dimName, dimId] : llvm::zip(dimNames, dimIds)) {
 285:     int32_t mask = freeVarMasks.lookup(dimName);
 286:     if (mask != 0) {
 287:       auto dimPred = b.icmp_eq(b.and_(dimId, b.i32_val(mask)), zero);
 288:       pred = maybeAnd(rewriter, loc, pred, dimPred);
 289:     }
 290:   }
 291:   return pred;
 292: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 294-294

```cpp
 294: } // namespace triton::gpu
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 296-306

```cpp
 296: SmallVector<std::pair<StringAttr, Value>>
 297: applyLinearLayout(Location loc, RewriterBase &rewriter,
 298:                   const LinearLayout &layout,
 299:                   ArrayRef<std::pair<StringAttr, Value>> indices) {
 300:   auto b = TritonLLVMOpBuilder(loc, rewriter);
 301:   assert(layout.getNumInDims() == indices.size());
 302:   assert(llvm::equal(layout.getInDimNames(), llvm::make_first_range(indices)));
 303:   // Trivial layout
 304:   if (layout.getNumOutDims() == 0) {
 305:     return {};
 306:   }
```

- **EN:** Defines `applyLinearLayout`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `applyLinearLayout`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 308-314

```cpp
 308:   // This function can emit a lot of MLIR code, which ultimately makes
 309:   // compilation slow.  (We think this shouldn't be the case -- it's not *that*
 310:   // much code -- but we're not clear on how to fix the slowness, which happens
 311:   // in the bowels of MLIR.)
 312:   //
 313:   // As a result we go through some contortions to avoid emitting code where
 314:   // possible.
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 316-327

```cpp
 316:   // Manually constant-fold the layout where possible.
 317:   SmallVector<std::pair<StringAttr, int32_t>> constantIns;
 318:   SmallVector<std::pair<StringAttr, Value>> nonConstantIns;
 319:   for (auto [inDimName, idx] : indices) {
 320:     APInt constant;
 321:     if (matchPattern(idx, m_ConstantInt(&constant))) {
 322:       constantIns.push_back({inDimName, constant.getSExtValue()});
 323:     } else {
 324:       constantIns.push_back({inDimName, 0});
 325:       nonConstantIns.push_back({inDimName, idx});
 326:     }
 327:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 329-337

```cpp
 329:   // Compute constant part of the output and wrap it as values
 330:   Value zero = b.i32_val(0);
 331:   SmallVector<std::pair<StringAttr, Value>> outIndices;
 332:   for (auto [outDimName, constant] : layout.apply(constantIns)) {
 333:     if (constant == 0)
 334:       outIndices.push_back({outDimName, zero});
 335:     else
 336:       outIndices.push_back({outDimName, b.i32_val(constant)});
 337:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 339-341

```cpp
 339:   if (nonConstantIns.size() == 0) {
 340:     return outIndices;
 341:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 343-351

```cpp
 343:   SmallVector<StringAttr> inDimNames;
 344:   // Concatenate input
 345:   Value x = b.i32_val(0);
 346:   int shift = 0;
 347:   for (auto [inDimName, idx] : nonConstantIns) {
 348:     inDimNames.push_back(inDimName);
 349:     x = b.or_(x, b.shl(idx, b.i32_val(shift)));
 350:     shift += layout.getInDimSizeLog2(inDimName);
 351:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 353-358

```cpp
 353:   for (auto &[outDimName, outIdx] : outIndices) {
 354:     // Apply flattened sublayout for this output
 355:     auto matrix = layout.sublayout(inDimNames, outDimName).flattenIns();
 356:     auto out = triton::gpu::matrixVectorProd(b, matrix, x);
 357:     outIdx = b.xor_(outIdx, out);
 358:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 360-361

```cpp
 360:   return outIndices;
 361: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 363-364

```cpp
 363: std::optional<int> getWarpGroupStartWarpId(Block *block) {
 364:   using namespace triton::gpu;
```

- **EN:** Defines accessor/helper `getWarpGroupStartWarpId` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getWarpGroupStartWarpId`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 366-371

```cpp
 366:   // Look for an enclosing `ttg.warp_specialize` op.
 367:   while (block && block->getParentOp() &&
 368:          !isa<WarpSpecializePartitionsOp>(block->getParentOp()))
 369:     block = block->getParentOp()->getBlock();
 370:   if (!block || !block->getParentOp())
 371:     return {};
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 373-380

```cpp
 373:   auto partitions = cast<WarpSpecializePartitionsOp>(block->getParentOp());
 374:   unsigned idx = block->getParent()->getRegionNumber();
 375:   WarpSpecializeOp ws = partitions.getParentOp();
 376:   std::optional<ArrayRef<int32_t>> startIds = ws.getWarpGroupStartIds();
 377:   assert(startIds && "cannot get warp group ID before warp group allocation");
 378:   int32_t warpStartId = (*startIds)[idx];
 379:   return warpStartId;
 380: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 382-383

```cpp
 382: std::optional<int> getWarpGroupStartThreadId(Block *block) {
 383:   using namespace triton::gpu;
```

- **EN:** Defines accessor/helper `getWarpGroupStartThreadId` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getWarpGroupStartThreadId`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 385-387

```cpp
 385:   std::optional<int> warpStartId = getWarpGroupStartWarpId(block);
 386:   if (!warpStartId)
 387:     return {};
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 389-392

```cpp
 389:   int threadsPerWarp = TritonGPUDialect::getThreadsPerWarp(
 390:       block->getParentOp()->getParentOfType<ModuleOp>());
 391:   return *warpStartId * threadsPerWarp;
 392: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 394-397

```cpp
 394: Value getThreadId(OpBuilder &rewriter, Location loc) {
 395:   Value tid =
 396:       ::mlir::gpu::ThreadIdOp::create(rewriter, loc, ::mlir::gpu::Dimension::x);
 397:   tid = arith::IndexCastOp::create(rewriter, loc, i32_ty, tid);
```

- **EN:** Defines accessor/helper `getThreadId` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getThreadId`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 399-402

```cpp
 399:   Operation *lookupPt = &rewriter.getInsertionBlock()->front();
 400:   int threadsPerWarp = triton::gpu::lookupThreadsPerWarp(rewriter);
 401:   int numWarps = triton::gpu::lookupNumWarps(lookupPt);
 402:   int upperBound = numWarps * threadsPerWarp;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 404-404

```cpp
 404:   TritonLLVMOpBuilder b(loc, rewriter);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 406-411

```cpp
 406:   // If this is being created inside a warp specialize op, compute the relative
 407:   // thread ID within the warp group.
 408:   if (std::optional<int> startId =
 409:           getWarpGroupStartThreadId(rewriter.getInsertionBlock())) {
 410:     tid = arith::SubIOp::create(rewriter, loc, tid, b.i32_val(*startId));
 411:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 413-415

```cpp
 413:   assert(llvm::isPowerOf2_32(upperBound));
 414:   // help LLVM's known bits analysis:
 415:   tid = b.and_(tid, b.i32_val(upperBound - 1));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 417-418

```cpp
 417:   return tid;
 418: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 420-424

```cpp
 420: std::pair<Value, Value> getLaneAndWarpId(OpBuilder &rewriter, Location loc) {
 421:   TritonLLVMOpBuilder b(loc, rewriter);
 422:   Value tid = getThreadId(rewriter, loc);
 423:   int threadsPerWarp = triton::gpu::lookupThreadsPerWarp(rewriter);
 424:   Value warpSizeVal = b.i32_val(threadsPerWarp);
```

- **EN:** Defines accessor/helper `getLaneAndWarpId` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getLaneAndWarpId`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 426-437

```cpp
 426:   // If there is only one warp, the warp ID is always 0.
 427:   Operation *lookupPt = &rewriter.getInsertionBlock()->front();
 428:   Value laneId;
 429:   Value warpId;
 430:   if (triton::gpu::lookupNumWarps(lookupPt) == 1) {
 431:     laneId = tid;
 432:     warpId = b.i32_val(0);
 433:   } else {
 434:     laneId = b.urem(tid, warpSizeVal);
 435:     warpId = mlir::triton::gpu::WarpIdOp::create(rewriter, loc,
 436:                                                  /*omitUniformHint=*/true);
 437:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 439-440

```cpp
 439:   return {laneId, warpId};
 440: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 442-444

```cpp
 442: Value getLaneId(OpBuilder &rewriter, Location loc) {
 443:   return getLaneAndWarpId(rewriter, loc).first;
 444: }
```

- **EN:** Defines accessor/helper `getLaneId` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getLaneId`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 446-453

```cpp
 446: // Helper function: applies linear layout vectorized over register indices
 447: static SmallVector<SmallVector<std::pair<StringAttr, Value>>>
 448: applyLinearLayoutVec(Location loc, RewriterBase &rewriter,
 449:                      const LinearLayout &layout,
 450:                      ArrayRef<std::pair<StringAttr, Value>> indices,
 451:                      ArrayRef<uint32_t> registers) {
 452:   auto b = TritonLLVMOpBuilder(loc, rewriter);
 453:   MLIRContext *ctx = rewriter.getContext();
```

- **EN:** Defines `applyLinearLayoutVec`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `applyLinearLayoutVec`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 455-455

```cpp
 455:   StringAttr kRegister = str_attr("register");
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 457-464

```cpp
 457:   // Precompute the base (with register = 0)
 458:   SmallVector<std::pair<StringAttr, Value>> indicesWithZeroReg;
 459:   for (const auto &[attr, val] : indices) {
 460:     if (attr == kRegister)
 461:       indicesWithZeroReg.emplace_back(attr, b.i32_val(0));
 462:     else
 463:       indicesWithZeroReg.emplace_back(attr, val);
 464:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 466-467

```cpp
 466:   auto baseIndices =
 467:       applyLinearLayout(loc, rewriter, layout, indicesWithZeroReg);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 469-469

```cpp
 469:   SmallVector<SmallVector<std::pair<StringAttr, Value>>> ret;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 471-477

```cpp
 471:   // Iterate over registers, applying XOR trick
 472:   for (auto reg : registers) {
 473:     SmallVector<std::pair<StringAttr, int32_t>> constRegIndices;
 474:     for (const auto &[attr, val] : indices) {
 475:       constRegIndices.emplace_back(attr, attr == kRegister ? reg : 0);
 476:     }
 477:     auto regIndices = layout.apply(constRegIndices);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 479-484

```cpp
 479:     SmallVector<std::pair<StringAttr, Value>> combinedIndices;
 480:     for (auto [base, regIdx] : llvm::zip(baseIndices, regIndices)) {
 481:       assert(base.first == regIdx.first);
 482:       Value combined = b.xor_(base.second, b.i32_val(regIdx.second));
 483:       combinedIndices.emplace_back(base.first, combined);
 484:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 486-487

```cpp
 486:     ret.push_back(combinedIndices);
 487:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 489-490

```cpp
 489:   return ret;
 490: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 492-498

```cpp
 492: // Refactored emitIndices function using applyLinearLayoutVec
 493: SmallVector<SmallVector<Value>>
 494: emitIndices(Location loc, RewriterBase &rewriter, const TargetInfoBase &target,
 495:             const LinearLayout &ll, RankedTensorType type, bool withCTAOffset) {
 496:   auto b = TritonLLVMOpBuilder(loc, rewriter);
 497:   MLIRContext *ctx = rewriter.getContext();
 498:   auto shape = type.getShape();
```

- **EN:** Defines `emitIndices`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `emitIndices`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 500-503

```cpp
 500:   StringAttr kRegister = str_attr("register");
 501:   StringAttr kLane = str_attr("lane");
 502:   StringAttr kWarp = str_attr("warp");
 503:   StringAttr kBlock = str_attr("block");
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 505-507

```cpp
 505:   auto [laneId, warpId] = getLaneAndWarpId(rewriter, loc);
 506:   Value blockId =
 507:       withCTAOffset ? target.getClusterCTAId(rewriter, loc) : b.i32_val(0);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 509-513

```cpp
 509:   SmallVector<std::pair<StringAttr, Value>> commonIndices = {
 510:       {kRegister, b.i32_val(0)},
 511:       {kLane, laneId},
 512:       {kWarp, warpId},
 513:       {kBlock, blockId}};
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 515-518

```cpp
 515:   // Vectorize over registers
 516:   SmallVector<uint32_t> registerIndices;
 517:   for (unsigned reg = 0; reg < ll.getInDimSize(kRegister); ++reg)
 518:     registerIndices.push_back(reg);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 520-521

```cpp
 520:   auto vecIndices =
 521:       applyLinearLayoutVec(loc, rewriter, ll, commonIndices, registerIndices);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 523-531

```cpp
 523:   unsigned rank = shape.size();
 524:   SmallVector<SmallVector<Value>> ret;
 525:   for (auto &indices : vecIndices) {
 526:     SmallVector<Value> vals;
 527:     assert(indices.size() == rank);
 528:     for (auto &idx : indices)
 529:       vals.push_back(idx.second);
 530:     ret.push_back(vals);
 531:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 533-534

```cpp
 533:   return ret;
 534: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 536-541

```cpp
 536: SmallVector<SmallVector<Value>>
 537: emitIndices(Location loc, RewriterBase &rewriter, const TargetInfoBase &target,
 538:             Attribute layout, RankedTensorType type, bool withCTAOffset) {
 539:   LinearLayout ll = triton::gpu::toLinearLayout(type.getShape(), layout);
 540:   return emitIndices(loc, rewriter, target, ll, type, withCTAOffset);
 541: }
```

- **EN:** Defines `emitIndices`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `emitIndices`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 543-550

```cpp
 543: SmallVector<Value> computeLocalPtrs(Location loc,
 544:                                     triton::gpu::MemDescType memDescTy,
 545:                                     SharedMemoryObject smemObj, Type llvmElemTy,
 546:                                     ArrayRef<Value> idxValues,
 547:                                     ArrayRef<SmallVector<Value>> coords,
 548:                                     unsigned axis, RewriterBase &rewriter) {
 549:   MLIRContext *ctx = memDescTy.getContext();
 550:   auto b = TritonLLVMOpBuilder(loc, rewriter);
```

- **EN:** Defines helper `computeLocalPtrs` that computes or constructs intermediate data used by the surrounding transformation. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic.
- **CN:** 这里定义辅助函数 `computeLocalPtrs`，用于计算或构造外围变换所需的中间数据。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。
### Lines 552-556

```cpp
 552:   // Get the shared memory layout (linear component for padded layouts)
 553:   auto sharedLayout = triton::gpu::isPaddedEncoding(memDescTy.getEncoding())
 554:                           ? paddedLinearLayout(memDescTy)
 555:                           : toLinearLayout(memDescTy);
 556:   LinearLayout invSharedLayout = sharedLayout.pseudoinvert();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 558-561

```cpp
 558:   // Get layout dimension names for all dims
 559:   SmallVector<StringAttr> allDims;
 560:   for (unsigned dim = 0, rank = memDescTy.getRank(); dim < rank; ++dim)
 561:     allDims.push_back(str_attr("dim" + Twine(dim)));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 563-566

```cpp
 563:   auto kOffset = str_attr("offset");
 564:   // Get the subslice affine offset (non-zero for memdesc subslices)
 565:   Value affineOffset = smemObj.getShmemOffset(loc, rewriter, memDescTy);
 566:   auto bitwidth = getIntOrFloatOrPtrBitWidth(llvmElemTy);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 568-569

```cpp
 568:   SmallVector<Value> ptrs;
 569:   ptrs.reserve(coords.size());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 571-579

```cpp
 571:   for (auto [i, idxVal] : llvm::enumerate(idxValues)) {
 572:     Value idx = idxVal;
 573:     unsigned idxWidth = idx.getType().getIntOrFloatBitWidth();
 574:     // Convert index to i32 if needed
 575:     if (idxWidth > 32) {
 576:       idx = b.trunc(i32_ty, idx);
 577:     } else if (idxWidth < 32) {
 578:       idx = b.zext(i32_ty, idx);
 579:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 581-583

```cpp
 581:     // Copy coordinates and replace the axis coordinate with the index value
 582:     SmallVector<Value> indices(coords[i]);
 583:     indices[axis] = idx;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 585-588

```cpp
 585:     // Apply inverted shared layout to compute offset
 586:     SmallVector<std::pair<StringAttr, Value>> inputs;
 587:     for (unsigned dim = 0; dim < indices.size(); ++dim)
 588:       inputs.push_back({allDims[dim], indices[dim]});
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 590-590

```cpp
 590:     auto outputs = applyLinearLayout(loc, rewriter, invSharedLayout, inputs);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 592-600

```cpp
 592:     // Extract the offset value
 593:     Value offset = nullptr;
 594:     for (auto [name, value] : outputs) {
 595:       if (name == kOffset) {
 596:         offset = value;
 597:         break;
 598:       }
 599:     }
 600:     assert(offset && "expected offset output from inverted shared layout");
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 602-612

```cpp
 602:     // For subslices, the physical offset is computed as:
 603:     //   physical_offset = L⁻¹(coords) ⊕ L⁻¹(subslice_logical_offset)
 604:     //
 605:     // We use XOR for consistency with lowerLdSt. MemDescSubsliceOp::verify()
 606:     // enforces:
 607:     // 1. Subslice offsets must be multiples of the tile size
 608:     // 2. Subslice offsets must map to power-of-2 physical offsets
 609:     //
 610:     // These constraints ensure the bit ranges of L⁻¹(coords) and
 611:     // L⁻¹(subslice_offset) are disjoint, so XOR and addition are equivalent.
 612:     offset = b.xor_(offset, affineOffset);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 614-630

```cpp
 614:     // Add padding offset for padded layouts (non-linear component)
 615:     Value ptr;
 616:     if (triton::gpu::isPaddedEncoding(memDescTy.getEncoding())) {
 617:       // Convert offset to bytes for padding calculation
 618:       Value offsetBytes = b.mul(offset, b.i32_val(bitwidth / 8));
 619:       auto shifts = getPaddedSharedShifts(memDescTy.getEncoding(), bitwidth,
 620:                                           /*offsetInBytes=*/true);
 621:       // GEP in bytes: base + offset*elemSize + padOffset
 622:       Value totalOffset = applyPadding(loc, rewriter, offsetBytes, shifts);
 623:       ptr = b.gep(smemObj.getBase().getType(), i8_ty, smemObj.getBase(),
 624:                   totalOffset);
 625:     } else {
 626:       ptr = b.gep(smemObj.getBase().getType(), llvmElemTy, smemObj.getBase(),
 627:                   offset);
 628:     }
 629:     ptrs.push_back(ptr);
 630:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 632-633

```cpp
 632:   return ptrs;
 633: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 635-639

```cpp
 635: SmallVector<std::pair<unsigned, unsigned>>
 636: getPaddedSharedShifts(Attribute enc, unsigned bitwidth, bool offsetInBytes) {
 637:   auto padded = triton::gpu::getPaddedEncoding(enc);
 638:   if (!padded)
 639:     return {};
```

- **EN:** Defines accessor/helper `getPaddedSharedShifts` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getPaddedSharedShifts`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 641-655

```cpp
 641:   SmallVector<std::pair<unsigned, unsigned>> shifts;
 642:   assert(bitwidth >= 8 && (bitwidth % 8 == 0) &&
 643:          "bitwidth must be a positive multiple of 8 for padding");
 644:   uint64_t offScale = offsetInBytes ? (bitwidth / 8) : 1;
 645:   for (auto [interval, padding] :
 646:        llvm::zip_equal(padded.getIntervals(), padded.getPaddings())) {
 647:     uint64_t intervalScaled = static_cast<uint64_t>(interval) * offScale;
 648:     uint64_t paddingScaled = static_cast<uint64_t>(padding) * offScale;
 649:     unsigned i = llvm::Log2_64(intervalScaled);
 650:     unsigned p = llvm::Log2_64(paddingScaled);
 651:     assert(i < 32 && p < 32 && "shift amount must be < 32 for i32 offsets");
 652:     shifts.push_back({i, p});
 653:   }
 654:   return shifts;
 655: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 657-666

```cpp
 657: Value applyPadding(Location loc, RewriterBase &rewriter, Value baseOffset,
 658:                    ArrayRef<std::pair<unsigned, unsigned>> shifts) {
 659:   if (shifts.empty())
 660:     return baseOffset;
 661:   auto b = TritonLLVMOpBuilder(loc, rewriter);
 662:   Value pad = b.i32_val(0);
 663:   for (auto [i, p] : shifts)
 664:     pad = b.add(pad, b.shl(b.lshr(baseOffset, b.i32_val(i)), b.i32_val(p)));
 665:   return b.add(baseOffset, pad);
 666: }
```

- **EN:** Defines `applyPadding`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `applyPadding`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 668-677

```cpp
 668: uint32_t applyPadding(uint32_t baseOffset,
 669:                       ArrayRef<std::pair<unsigned, unsigned>> shifts) {
 670:   uint64_t pad = 0;
 671:   for (auto [i, p] : shifts)
 672:     pad += (static_cast<uint64_t>(baseOffset) >> i) << p;
 673:   uint64_t out = baseOffset + pad;
 674:   assert(out <= std::numeric_limits<uint32_t>::max() &&
 675:          "padded offset must be within 32-bit range");
 676:   return static_cast<uint32_t>(out);
 677: }
```

- **EN:** Defines `applyPadding`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `applyPadding`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 679-686

```cpp
 679: SmallVector<Value>
 680: lowerLdStShared(Location loc, MLIRContext *ctx, LinearLayout cvt,
 681:                 ArrayRef<Value> valsArray, // Input for store, output for load
 682:                 Type llvmElemTy, ArrayRef<Value> smemBases,
 683:                 ArrayRef<std::pair<unsigned, unsigned>> paddingShifts,
 684:                 Value affineOffset, uint64_t maskSpanAffineOffset,
 685:                 RewriterBase &rewriter, const TargetInfoBase &targetInfo,
 686:                 std::optional<int> maybeMaxVecElems, Operation *localLoadOp) {
```

- **EN:** Defines helper `lowerLdStShared` that performs a lowering step from Triton/MLIR semantics to a lower-level representation.
- **CN:** 这里定义辅助函数 `lowerLdStShared`，负责把 Triton/MLIR 语义中的一个步骤降级到更底层表示。
### Lines 688-689

```cpp
 688:   bool isStore = !valsArray.empty();
 689:   auto b = TritonLLVMOpBuilder(loc, rewriter);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 691-708

```cpp
 691:   auto emitLdSt = [&](RewriterBase &rewriter, Location loc,
 692:                       ArrayRef<Value> vals, Value shmemAddr, int idx,
 693:                       VectorType vecTy,
 694:                       std::optional<Value> ctaId) -> SmallVector<Value> {
 695:     auto length = vecTy.getNumElements();
 696:     if (isStore) {
 697:       Value valsVec =
 698:           packLLVector(loc, ArrayRef<Value>(vals).slice(idx, length), rewriter);
 699:       targetInfo.storeDShared(rewriter, loc, shmemAddr, ctaId, valsVec,
 700:                               /*pred=*/b.true_val());
 701:       return {};
 702:     } else {
 703:       assert(vals.empty());
 704:       Value valsVec =
 705:           targetInfo.loadDShared(rewriter, loc, shmemAddr, ctaId, vecTy,
 706:                                  /*pred=*/b.true_val(), localLoadOp);
 707:       return unpackLLVector(loc, valsVec, rewriter);
 708:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 709-714

```cpp
 709:   };
 710:   auto [laneId, warpId] = getLaneAndWarpId(rewriter, loc);
 711:   return lowerLdSt(loc, ctx, cvt, valsArray, llvmElemTy, smemBases,
 712:                    paddingShifts, affineOffset, maskSpanAffineOffset, laneId,
 713:                    warpId, rewriter, targetInfo, maybeMaxVecElems, emitLdSt);
 714: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 716-733

```cpp
 716: SmallVector<Value>
 717: lowerLdSt(Location loc, MLIRContext *ctx, LinearLayout cvt,
 718:           ArrayRef<Value> valsArray, // Input for store, output for load
 719:           Type llvmElemTy, ArrayRef<Value> smemBases,
 720:           ArrayRef<std::pair<unsigned, unsigned>> paddingShifts,
 721:           Value affineOffset, uint64_t maskSpanAffineOffset, Value laneId,
 722:           Value warpId, RewriterBase &rewriter,
 723:           const TargetInfoBase &targetInfo, std::optional<int> maybeMaxVecElems,
 724:           std::function<SmallVector<Value>(RewriterBase &, Location,
 725:                                            ArrayRef<Value>, Value, int,
 726:                                            VectorType, std::optional<Value>)>
 727:               lowerInst) {
 728:   assert(!smemBases.empty() && "smemBases cannot be empty");
 729:   auto vals = to_vector(valsArray);
 730:   bool isStore = !vals.empty();
 731:   auto b = TritonLLVMOpBuilder(loc, rewriter);
 732:   auto smemPtrTy = ptr_ty(ctx, targetInfo.getSharedAddressSpace());
 733:   auto kReg = str_attr("register");
```

- **EN:** Defines helper `lowerLdSt` that performs a lowering step from Triton/MLIR semantics to a lower-level representation. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义辅助函数 `lowerLdSt`，负责把 Triton/MLIR 语义中的一个步骤降级到更底层表示。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 734-739

```cpp
 734:   auto kLane = str_attr("lane");
 735:   auto kWarp = str_attr("warp");
 736:   auto kBlock = str_attr("block");
 737:   auto kOffset = str_attr("offset");
 738:   auto kPartition = str_attr("partition");
 739:   auto bitwidth = getIntOrFloatOrPtrBitWidth(llvmElemTy);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 741-747

```cpp
 741:   // Either we have multiple bases with a matching partition dimension,
 742:   // or we have a single base.
 743:   assert((smemBases.size() == 1 ||
 744:           (cvt.hasOutDim(kPartition) &&
 745:            cvt.getOutDimSize(kPartition) == smemBases.size())) &&
 746:          "smemBases size must match partition dimension size");
 747:   bool isPartitioned = smemBases.size() > 1;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 749-756

```cpp
 749:   // Extract the partition sublayout before stripping it for vectorization.
 750:   auto inDimNames = to_vector(cvt.getInDimNames());
 751:   LinearLayout partitionLayout;
 752:   Value basesVec;
 753:   if (isPartitioned) {
 754:     partitionLayout = cvt.sublayout(inDimNames, {kPartition});
 755:     basesVec = LLVM::buildBasePtrVector(loc, rewriter, smemBases);
 756:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 758-761

```cpp
 758:   // Strip kPartition output for vectorization analysis.
 759:   SmallVector<StringAttr> outDims = to_vector(cvt.getOutDimNames());
 760:   llvm::erase(outDims, kPartition);
 761:   cvt = cvt.sublayout(inDimNames, outDims);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 763-764

```cpp
 763:   auto [elemsPerVec, permutation] =
 764:       largestVectorisation(ctx, cvt, bitwidth, maybeMaxVecElems);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 766-769

```cpp
 766:   cvt = permutation.apply(cvt);
 767:   if (isStore) {
 768:     vals = permutation.apply(vals);
 769:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 771-783

```cpp
 771:   auto tile = LinearLayout::identity1D(elemsPerVec, kReg, kOffset);
 772:   auto quot = divideLeft(cvt, tile);
 773:   assert(quot.has_value() && "cvt must be divisible by tile");
 774:   LinearLayout reps = zerosLike(tile) * *quot;
 775:   assert(reps.hasInDim(kBlock));
 776:   LinearLayout addrLayout =
 777:       LinearLayout({{kLane, reps.getBases().lookup(kLane)},
 778:                     {kWarp, reps.getBases().lookup(kWarp)},
 779:                     {kBlock, reps.getBases().lookup(kBlock)}},
 780:                    reps.getOutDims(), false);
 781:   auto [nAdditive, permStrides] = actionAdditiveStrides(
 782:       reps, addrLayout, maskSpanAffineOffset, elemsPerVec);
 783:   reps = permStrides.apply(reps);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 785-791

```cpp
 785:   if (isPartitioned) {
 786:     partitionLayout = permutation.apply(partitionLayout);
 787:     partitionLayout = permStrides.apply(partitionLayout);
 788:   }
 789:   if (isStore) {
 790:     vals = permStrides.apply(vals);
 791:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 793-799

```cpp
 793:   // PTX expects the address increments to be done in bytes
 794:   // If we don't perform the computations in i8, the compiler would
 795:   // have to divide the computation by bitwdith / 8 and then lift this
 796:   // shl, which often it's not able to do.
 797:   auto i8Tile =
 798:       LinearLayout::zeros1D(bitwidth / 8, kReg, kOffset, bitwidth / 8);
 799:   auto i8AddrLayout = i8Tile * addrLayout;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 801-805

```cpp
 801:   Value blockId = b.i32_val(0);
 802:   bool useBlockId = !reps.isTrivialOver({kBlock});
 803:   if (useBlockId) {
 804:     blockId = targetInfo.getClusterCTAId(rewriter, loc);
 805:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 807-816

```cpp
 807:   auto baseI8AndCTA = applyLinearLayout(loc, rewriter, i8AddrLayout,
 808:                                         {{kReg, b.i32_val(0)},
 809:                                          {kLane, laneId},
 810:                                          {kWarp, warpId},
 811:                                          {kBlock, blockId}});
 812:   auto regBaseI8 = baseI8AndCTA[0].second;
 813:   Value targetCtaId;
 814:   if (useBlockId) {
 815:     targetCtaId = baseI8AndCTA[1].second;
 816:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 818-832

```cpp
 818:   // It's fine that we don't compute the offset in bytes as affineOffset
 819:   // will be folded into a constant
 820:   auto affineOffsetI8 = b.mul(affineOffset, b.i32_val(bitwidth / 8));
 821:   bool hasPadding = !paddingShifts.empty();
 822:   Value paddedAffineOffsetI8 = b.i32_val(0);
 823:   if (hasPadding && maskSpanAffineOffset != 0) {
 824:     // `maskSpanAffineOffset != 0` indicates the affine offsets come from
 825:     // MemDescSubsliceOp, whose verifier guarantees that the affine offsets are
 826:     // bitwise disjoint from other offset contributors. Padding can thus be
 827:     // applied separately. This helps LLVM reuse base pointers.
 828:     paddedAffineOffsetI8 =
 829:         applyPadding(loc, rewriter, affineOffsetI8, paddingShifts);
 830:   } else {
 831:     regBaseI8 = b.xor_(regBaseI8, affineOffsetI8);
 832:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 834-851

```cpp
 834:   SmallVector<Value> outVals;
 835:   auto vecTy = vec_ty(llvmElemTy, elemsPerVec);
 836:   for (int i = 0; i < cvt.getInDimSize(kReg); i += nAdditive) {
 837:     auto idxAndBlock =
 838:         reps.apply({{kReg, i}, {kLane, 0}, {kWarp, 0}, {kBlock, 0}});
 839:     auto regIdxI8 = idxAndBlock[0].second * (bitwidth / 8);
 840:     Value offset = b.xor_(regBaseI8, b.i32_val(regIdxI8));
 841:     if (hasPadding) {
 842:       offset = applyPadding(loc, rewriter, offset, paddingShifts);
 843:       if (maskSpanAffineOffset != 0)
 844:         offset = b.add(offset, paddedAffineOffsetI8);
 845:     }
 846:     Value ctaOffset = b.i32_val(0);
 847:     if (useBlockId) {
 848:       ctaOffset = b.xor_(targetCtaId, b.i32_val(idxAndBlock[1].second));
 849:     }
 850:     for (int j = 0; j < nAdditive; j += elemsPerVec) {
 851:       // all these constants will go as immediate values to LDS/STS
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 852-858

```cpp
 852:       auto idxAndBlockAdd =
 853:           reps.apply({{kReg, j}, {kLane, 0}, {kWarp, 0}, {kBlock, 0}});
 854:       auto regIdxAddI8 = idxAndBlockAdd[0].second * (bitwidth / 8);
 855:       // `actionAdditiveStrides` forces `regIdxAddI8` and `offset` to be bitwise
 856:       // disjoint, so we can calculate their padding contributions separately.
 857:       regIdxAddI8 = applyPadding(regIdxAddI8, paddingShifts);
 858:       Value innerOffset = b.add(offset, b.i32_val(regIdxAddI8));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 860-871

```cpp
 860:       // Select the appropriate base pointer for partitioned tensors
 861:       Value smemBase = smemBases[0];
 862:       if (isPartitioned) {
 863:         // Compute the partition index dynamically.
 864:         auto partitionResult = applyLinearLayout(loc, rewriter, partitionLayout,
 865:                                                  {{kReg, b.i32_val(i + j)},
 866:                                                   {kLane, laneId},
 867:                                                   {kWarp, warpId},
 868:                                                   {kBlock, blockId}});
 869:         Value partitionIdx = partitionResult[0].second;
 870:         smemBase = b.extract_element(basesVec, partitionIdx);
 871:       }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 873-882

```cpp
 873:       std::optional<Value> innerCtaOffset;
 874:       if (useBlockId) {
 875:         innerCtaOffset = b.add(ctaOffset, b.i32_val(idxAndBlockAdd[1].second));
 876:       }
 877:       auto vecAddr = b.gep(smemPtrTy, i8_ty, smemBase, innerOffset,
 878:                            LLVM::GEPNoWrapFlags::inbounds);
 879:       llvm::append_range(outVals, lowerInst(rewriter, loc, vals, vecAddr, i + j,
 880:                                             vecTy, innerCtaOffset));
 881:     }
 882:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 884-892

```cpp
 884:   // Permute the values back if we are loading
 885:   if (!isStore) {
 886:     auto invPermStrides = permStrides.inverse();
 887:     outVals = invPermStrides.apply(outVals);
 888:     auto invPerm = permutation.inverse();
 889:     outVals = invPerm.apply(outVals);
 890:   }
 891:   return outVals;
 892: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 894-900

```cpp
 894: SmallVector<Value>
 895: lowerLocalLdSt(Location loc, MLIRContext *ctx,
 896:                LinearLayout cvt, // Map from registers to offset[, partition]
 897:                ArrayRef<Value> valsArray, // Input for store, empty for load
 898:                Type llvmElemTy, triton::gpu::MemDescType srcTy,
 899:                SharedMemoryObject smemObj, RewriterBase &rewriter,
 900:                const TargetInfoBase &targetInfo, Operation *localLoadOp) {
```

- **EN:** Defines helper `lowerLocalLdSt` that performs a lowering step from Triton/MLIR semantics to a lower-level representation. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic.
- **CN:** 这里定义辅助函数 `lowerLocalLdSt`，负责把 Triton/MLIR 语义中的一个步骤降级到更底层表示。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。
### Lines 902-919

```cpp
 902:   auto isStore = !valsArray.empty();
 903:   // Remove broadcasting in the registers
 904:   auto removeBroadcastSrc = actionRemoveBroadcastedRegs(cvt);
 905:   if (!removeBroadcastSrc.isIdentity()) {
 906:     auto prmtCvt = removeBroadcastSrc.apply(cvt);
 907:     auto inVals = to_vector(valsArray);
 908:     if (isStore) {
 909:       inVals = removeBroadcastSrc.apply(inVals);
 910:     }
 911:     auto outVals = lowerLocalLdSt(loc, ctx, prmtCvt, inVals, llvmElemTy, srcTy,
 912:                                   smemObj, rewriter, targetInfo, localLoadOp);
 913:     if (!isStore) {
 914:       outVals = broadcastAs(outVals, cvt);
 915:     }
 916:     return outVals;
 917:   }
 918:   auto affineOffset = smemObj.getShmemOffset(loc, rewriter, srcTy);
 919:   auto maskSpanAffineOffset = smemObj.getMaskSpanOffsets(srcTy);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 921-930

```cpp
 921:   // Extract padding info from padded encoding (standalone or inside
 922:   // partitioned)
 923:   std::optional<int> maybeMaxVecElems;
 924:   SmallVector<std::pair<unsigned, unsigned>> paddingShifts;
 925:   if (triton::gpu::isPaddedEncoding(srcTy.getEncoding())) {
 926:     maybeMaxVecElems = triton::gpu::getMinInterval(srcTy.getEncoding());
 927:     auto bitwidth = getIntOrFloatOrPtrBitWidth(llvmElemTy);
 928:     paddingShifts = getPaddedSharedShifts(srcTy.getEncoding(), bitwidth,
 929:                                           /*offsetInBytes=*/true);
 930:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 932-940

```cpp
 932:   // Get shared memory bases from the SharedMemoryObject.
 933:   // For non-partitioned tensors, this returns a single base.
 934:   // For partitioned tensors, this returns all bases (one per partition).
 935:   SmallVector<Value> smemBases(smemObj.getBases().begin(),
 936:                                smemObj.getBases().end());
 937:   return lowerLdStShared(loc, ctx, cvt, valsArray, llvmElemTy, smemBases,
 938:                          paddingShifts, affineOffset, maskSpanAffineOffset,
 939:                          rewriter, targetInfo, maybeMaxVecElems, localLoadOp);
 940: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。
### Lines 942-958

```cpp
 942: SmallVector<Value> unpackLLElements(Location loc, Value llvmStruct,
 943:                                     RewriterBase &rewriter) {
 944:   assert(bool(llvmStruct) && "can not unpack null values");
 945:   if (llvmStruct.getType().isIntOrIndexOrFloat() ||
 946:       isa<triton::PointerType>(llvmStruct.getType()) ||
 947:       isa<LLVM::LLVMPointerType>(llvmStruct.getType()))
 948:     return {llvmStruct};
 949:   ArrayRef<Type> types =
 950:       cast<LLVM::LLVMStructType>(llvmStruct.getType()).getBody();
 951:   SmallVector<Value> results(types.size());
 952:   auto b = TritonLLVMOpBuilder(loc, rewriter);
 953:   for (unsigned i = 0; i < types.size(); ++i) {
 954:     Type type = types[i];
 955:     results[i] = b.extract_val(type, llvmStruct, i);
 956:   }
 957:   return results;
 958: }
```

- **EN:** Defines `unpackLLElements`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `unpackLLElements`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 960-967

```cpp
 960: Value packLLElements(Location loc, const LLVMTypeConverter *typeConverter,
 961:                      ValueRange resultVals, RewriterBase &rewriter, Type type) {
 962:   auto structType =
 963:       dyn_cast<LLVM::LLVMStructType>(typeConverter->convertType(type));
 964:   if (!structType) {
 965:     assert(resultVals.size() == 1);
 966:     return *resultVals.begin();
 967:   }
```

- **EN:** Defines `packLLElements`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Type mapping is delegated to the LLVM type converter or related conversion helpers. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `packLLElements`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 类型映射委托给 LLVM 类型转换器或相关转换辅助函数。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 969-985

```cpp
 969:   auto elementTypes = structType.getBody();
 970:   if (elementTypes.size() != resultVals.size()) {
 971:     emitError(loc) << " size mismatch when packing elements for LLVM struct"
 972:                    << " expected " << elementTypes.size() << " but got "
 973:                    << resultVals.size();
 974:     llvm::report_fatal_error(
 975:         "size mismatch when packing elements for LLVM struct");
 976:   }
 977:   Value llvmStruct = LLVM::UndefOp::create(rewriter, loc, structType);
 978:   auto b = TritonLLVMOpBuilder(loc, rewriter);
 979:   for (auto [i, value] : llvm::enumerate(resultVals)) {
 980:     assert(value && "unexpected null value");
 981:     if (value.getType() != elementTypes[i]) {
 982:       LDBG("type " << type << " structType " << structType);
 983:       LDBG("value " << value);
 984:       emitError(loc) << "invalid element type in packLLElements. Expected "
 985:                      << elementTypes[i] << " but got " << value.getType();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 986-992

```cpp
 986:       llvm::report_fatal_error(
 987:           "element type mismatch when packing elements for LLVM struct");
 988:     }
 989:     llvmStruct = b.insert_val(structType, llvmStruct, value, i);
 990:   }
 991:   return llvmStruct;
 992: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 994-1000

```cpp
 994: SmallVector<Value> unpackLLVector(Location loc, Value llvmVec,
 995:                                   RewriterBase &rewriter) {
 996:   assert(bool(llvmVec) && "cannot unpack null value");
 997:   if (llvmVec.getType().isIntOrIndexOrFloat() ||
 998:       isa<triton::PointerType>(llvmVec.getType()) ||
 999:       isa<LLVM::LLVMPointerType>(llvmVec.getType()))
1000:     return {llvmVec};
```

- **EN:** Defines `unpackLLVector`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `unpackLLVector`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 1002-1009

```cpp
1002:   auto b = TritonLLVMOpBuilder(loc, rewriter);
1003:   SmallVector<Value> results;
1004:   for (int i = 0; i < cast<VectorType>(llvmVec.getType()).getNumElements();
1005:        i++) {
1006:     results.push_back(b.extract_element(llvmVec, b.i32_val(i)));
1007:   }
1008:   return results;
1009: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 1011-1020

```cpp
1011: Value packLLVector(Location loc, ValueRange vals, RewriterBase &rewriter) {
1012:   assert(vals.size() > 0);
1013:   auto vecType = vec_ty(vals[0].getType(), vals.size());
1014:   auto b = TritonLLVMOpBuilder(loc, rewriter);
1015:   Value vec = b.undef(vecType);
1016:   for (int i = 0; i < vals.size(); i++) {
1017:     vec = b.insert_element(vec, vals[i], b.i32_val(i));
1018:   }
1019:   return vec;
1020: }
```

- **EN:** Defines `packLLVector`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `packLLVector`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 1022-1039

```cpp
1022: std::optional<LLVM::AtomicBinOp> matchAtomicOp(RMWOp atomicOp) {
1023:   switch (atomicOp) {
1024:   case RMWOp::AND:
1025:     return LLVM::AtomicBinOp::_and;
1026:   case RMWOp::OR:
1027:     return LLVM::AtomicBinOp::_or;
1028:   case RMWOp::XOR:
1029:     return LLVM::AtomicBinOp::_xor;
1030:   case RMWOp::ADD:
1031:     return LLVM::AtomicBinOp::add;
1032:   case RMWOp::FADD:
1033:     return LLVM::AtomicBinOp::fadd;
1034:   case RMWOp::MAX:
1035:     return LLVM::AtomicBinOp::max;
1036:   case RMWOp::MIN:
1037:     return LLVM::AtomicBinOp::min;
1038:   case RMWOp::UMAX:
1039:     return LLVM::AtomicBinOp::umax;
```

- **EN:** Defines `matchAtomicOp`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `matchAtomicOp`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1040-1047

```cpp
1040:   case RMWOp::UMIN:
1041:     return LLVM::AtomicBinOp::umin;
1042:   case RMWOp::XCHG:
1043:     return LLVM::AtomicBinOp::xchg;
1044:   default:
1045:     return {};
1046:   }
1047: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1049-1062

```cpp
1049: std::optional<LLVM::AtomicOrdering> getMemoryOrdering(MemSemantic memOrdering) {
1050:   switch (memOrdering) {
1051:   case MemSemantic::RELAXED:
1052:     return LLVM::AtomicOrdering::monotonic;
1053:   case MemSemantic::ACQUIRE:
1054:     return LLVM::AtomicOrdering::acquire;
1055:   case MemSemantic::RELEASE:
1056:     return LLVM::AtomicOrdering::release;
1057:   case MemSemantic::ACQUIRE_RELEASE:
1058:     return LLVM::AtomicOrdering::acq_rel;
1059:   default:
1060:     return {};
1061:   }
1062: }
```

- **EN:** Defines accessor/helper `getMemoryOrdering` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getMemoryOrdering`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 1064-1069

```cpp
1064: llvm::MapVector<StringAttr, int32_t> getAllFreeVarMasks(MLIRContext *ctx) {
1065:   // Mask where all elements are redundant
1066:   auto kReg = str_attr("reg");
1067:   auto kLane = str_attr("lane");
1068:   auto kWarp = str_attr("warp");
1069:   auto kBlock = str_attr("block");
```

- **EN:** Defines accessor/helper `getAllFreeVarMasks` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getAllFreeVarMasks`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 1071-1077

```cpp
1071:   int32_t fullMask = -1;
1072:   llvm::MapVector<StringAttr, int32_t> ret;
1073:   for (auto dimName : {kReg, kLane, kWarp, kBlock}) {
1074:     ret[dimName] = fullMask;
1075:   }
1076:   return ret;
1077: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1079-1087

```cpp
1079: llvm::MapVector<StringAttr, int32_t> getFreeVariableMasks(Type type) {
1080:   auto ctx = type.getContext();
1081:   auto tensorTy = dyn_cast<RankedTensorType>(type);
1082:   if (!tensorTy) {
1083:     return getAllFreeVarMasks(ctx);
1084:   }
1085:   auto ll = triton::gpu::toLinearLayout(tensorTy);
1086:   return ll.getFreeVariableMasks();
1087: }
```

- **EN:** Defines accessor/helper `getFreeVariableMasks` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getFreeVariableMasks`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 1089-1093

```cpp
1089: SmallVector<SmallVector<unsigned>> emitOffsetForLayout(Attribute layout,
1090:                                                        RankedTensorType type) {
1091:   MLIRContext *ctx = layout.getContext();
1092:   auto shape = type.getShape();
1093:   unsigned rank = shape.size();
```

- **EN:** Defines `emitOffsetForLayout`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `emitOffsetForLayout`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1095-1095

```cpp
1095:   auto ll = triton::gpu::toLinearLayout(type);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1097-1100

```cpp
1097:   StringAttr kRegister = str_attr("register");
1098:   StringAttr kLane = str_attr("lane");
1099:   StringAttr kWarp = str_attr("warp");
1100:   StringAttr kBlock = str_attr("block");
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1102-1113

```cpp
1102:   SmallVector<SmallVector<unsigned>> offsets;
1103:   for (int i = 0; i < ll.getInDimSize(str_attr("register")); i++) {
1104:     auto idxs = ll.apply({{kRegister, i}, {kLane, 0}, {kWarp, 0}, {kBlock, 0}});
1105:     assert(idxs.size() == rank);
1106:     for (unsigned k = 0; k < rank; ++k) {
1107:       assert(idxs[k].first == str_attr("dim" + std::to_string(k)));
1108:     }
1109:     offsets.push_back(
1110:         llvm::to_vector_of<unsigned>(llvm::make_second_range(idxs)));
1111:   }
1112:   return offsets;
1113: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 1115-1117

```cpp
1115: namespace LLVM {
1116: using namespace mlir::triton;
1117: using mlir::triton::gpu::getOrder;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1119-1123

```cpp
1119: Value createConstantI1(Location loc, OpBuilder &rewriter, bool v) {
1120:   auto i1ty = rewriter.getIntegerType(1);
1121:   return LLVM::ConstantOp::create(rewriter, loc, i1ty,
1122:                                   IntegerAttr::get(i1ty, v));
1123: }
```

- **EN:** Defines helper `createConstantI1` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createConstantI1`，用于计算或构造外围变换所需的中间数据。
### Lines 1125-1129

```cpp
1125: Value createConstantI32(Location loc, OpBuilder &rewriter, int32_t v) {
1126:   auto i32ty = rewriter.getIntegerType(32);
1127:   return LLVM::ConstantOp::create(rewriter, loc, i32ty,
1128:                                   IntegerAttr::get(i32ty, v));
1129: }
```

- **EN:** Defines helper `createConstantI32` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createConstantI32`，用于计算或构造外围变换所需的中间数据。
### Lines 1131-1135

```cpp
1131: Value createConstantI64(Location loc, OpBuilder &rewriter, int64_t v) {
1132:   auto i64ty = rewriter.getIntegerType(64);
1133:   return LLVM::ConstantOp::create(rewriter, loc, i64ty,
1134:                                   IntegerAttr::get(i64ty, v));
1135: }
```

- **EN:** Defines helper `createConstantI64` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createConstantI64`，用于计算或构造外围变换所需的中间数据。
### Lines 1137-1141

```cpp
1137: Value createConstantF16(Location loc, OpBuilder &rewriter, float v) {
1138:   auto type = type::f16Ty(rewriter.getContext());
1139:   return LLVM::ConstantOp::create(rewriter, loc, type,
1140:                                   rewriter.getF16FloatAttr(v));
1141: }
```

- **EN:** Defines helper `createConstantF16` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createConstantF16`，用于计算或构造外围变换所需的中间数据。
### Lines 1143-1150

```cpp
1143: Value createConstantBF16(Location loc, OpBuilder &rewriter, float v) {
1144:   APFloat apf(v);
1145:   bool ignored;
1146:   apf.convert(APFloat::BFloat(), APFloat::rmNearestTiesToEven, &ignored);
1147:   auto type = type::bf16Ty(rewriter.getContext());
1148:   auto attr = FloatAttr::get(type, apf);
1149:   return LLVM::ConstantOp::create(rewriter, loc, type, attr);
1150: }
```

- **EN:** Defines helper `createConstantBF16` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createConstantBF16`，用于计算或构造外围变换所需的中间数据。
### Lines 1152-1156

```cpp
1152: Value createConstantF32(Location loc, OpBuilder &rewriter, float v) {
1153:   auto type = type::f32Ty(rewriter.getContext());
1154:   return LLVM::ConstantOp::create(rewriter, loc, type,
1155:                                   rewriter.getF32FloatAttr(v));
1156: }
```

- **EN:** Defines helper `createConstantF32` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createConstantF32`，用于计算或构造外围变换所需的中间数据。
### Lines 1158-1162

```cpp
1158: Value createConstantF64(Location loc, OpBuilder &rewriter, double v) {
1159:   auto type = type::f64Ty(rewriter.getContext());
1160:   return LLVM::ConstantOp::create(rewriter, loc, type,
1161:                                   rewriter.getF64FloatAttr(v));
1162: }
```

- **EN:** Defines helper `createConstantF64` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createConstantF64`，用于计算或构造外围变换所需的中间数据。
### Lines 1164-1171

```cpp
1164: Value createNaNConstant(Location loc, OpBuilder &rewriter, Type type) {
1165:   if (!isa<FloatType>(type)) {
1166:     llvm::report_fatal_error("Creating NaN constant for non-float type!");
1167:   }
1168:   return LLVM::ConstantOp::create(
1169:       rewriter, loc, type,
1170:       APFloat::getNaN(cast<FloatType>(type).getFloatSemantics()));
1171: }
```

- **EN:** Defines helper `createNaNConstant` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createNaNConstant`，用于计算或构造外围变换所需的中间数据。
### Lines 1173-1179

```cpp
1173: // Create an index type constant.
1174: Value createIndexConstant(OpBuilder &builder, Location loc,
1175:                           const TypeConverter *converter, int64_t value) {
1176:   Type ty = converter->convertType(builder.getIndexType());
1177:   return LLVM::ConstantOp::create(builder, loc, ty,
1178:                                   builder.getIntegerAttr(ty, value));
1179: }
```

- **EN:** Defines helper `createIndexConstant` that computes or constructs intermediate data used by the surrounding transformation. Type mapping is delegated to the LLVM type converter or related conversion helpers.
- **CN:** 这里定义辅助函数 `createIndexConstant`，用于计算或构造外围变换所需的中间数据。 类型映射委托给 LLVM 类型转换器或相关转换辅助函数。
### Lines 1181-1187

```cpp
1181: // Create an integer constant of \param width bits.
1182: Value createLLVMIntegerConstant(OpBuilder &builder, Location loc, short width,
1183:                                 int64_t value) {
1184:   Type ty = builder.getIntegerType(width);
1185:   return LLVM::ConstantOp::create(builder, loc, ty,
1186:                                   builder.getIntegerAttr(ty, value));
1187: }
```

- **EN:** Defines helper `createLLVMIntegerConstant` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createLLVMIntegerConstant`，用于计算或构造外围变换所需的中间数据。
### Lines 1189-1195

```cpp
1189: LLVM::CallOp createLLVMCallOp(OpBuilder &builder, Location loc,
1190:                               LLVMFuncOp funcOp, ValueRange args) {
1191:   auto op = LLVM::CallOp::create(builder, loc, funcOp, args);
1192:   op.getProperties().setOpBundleSizes(builder.getDenseI32ArrayAttr({}));
1193:   op.getProperties().setOperandSegmentSizes({static_cast<int>(args.size()), 0});
1194:   return op;
1195: }
```

- **EN:** Defines helper `createLLVMCallOp` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createLLVMCallOp`，用于计算或构造外围变换所需的中间数据。
### Lines 1197-1205

```cpp
1197: LLVM::CallIntrinsicOp
1198: createLLVMIntrinsicCallOp(OpBuilder &builder, Location loc, StringRef intrinsic,
1199:                           TypeRange types, ValueRange args) {
1200:   auto op = LLVM::CallIntrinsicOp::create(builder, loc, types, args);
1201:   op.getProperties().setIntrin(builder.getStringAttr(intrinsic));
1202:   op.getProperties().setOpBundleSizes(builder.getDenseI32ArrayAttr({}));
1203:   op.getProperties().setOperandSegmentSizes({static_cast<int>(args.size()), 0});
1204:   return op;
1205: }
```

- **EN:** Defines helper `createLLVMIntrinsicCallOp` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createLLVMIntrinsicCallOp`，用于计算或构造外围变换所需的中间数据。
### Lines 1207-1210

```cpp
1207: SharedMemoryObject::SharedMemoryObject(Value base, Type baseElemType,
1208:                                        ArrayRef<Value> offsets)
1209:     : bases({base}), baseElemType(baseElemType),
1210:       offsets(offsets.begin(), offsets.end()) {}
```

- **EN:** Defines `SharedMemoryObject::SharedMemoryObject`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic.
- **CN:** 这里定义 `SharedMemoryObject::SharedMemoryObject`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。
### Lines 1212-1217

```cpp
1212: SharedMemoryObject::SharedMemoryObject(ArrayRef<Value> bases, Type baseElemType,
1213:                                        ArrayRef<Value> offsets)
1214:     : bases(bases.begin(), bases.end()), baseElemType(baseElemType),
1215:       offsets(offsets.begin(), offsets.end()) {
1216:   assert(!bases.empty() && "SharedMemoryObject must have at least one base");
1217: }
```

- **EN:** Defines `SharedMemoryObject::SharedMemoryObject`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `SharedMemoryObject::SharedMemoryObject`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 1219-1225

```cpp
1219: SharedMemoryObject::SharedMemoryObject(Value base, Type baseElemType,
1220:                                        int64_t rank, Location loc,
1221:                                        RewriterBase &rewriter)
1222:     : bases({base}), baseElemType(baseElemType) {
1223:   auto b = TritonLLVMOpBuilder(loc, rewriter);
1224:   offsets.append(rank, b.i32_val(0));
1225: }
```

- **EN:** Defines `SharedMemoryObject::SharedMemoryObject`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic.
- **CN:** 这里定义 `SharedMemoryObject::SharedMemoryObject`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。
### Lines 1227-1234

```cpp
1227: SharedMemoryObject::SharedMemoryObject(ArrayRef<Value> bases, Type baseElemType,
1228:                                        int64_t rank, Location loc,
1229:                                        RewriterBase &rewriter)
1230:     : bases(bases.begin(), bases.end()), baseElemType(baseElemType) {
1231:   assert(!bases.empty() && "SharedMemoryObject must have at least one base");
1232:   auto b = TritonLLVMOpBuilder(loc, rewriter);
1233:   offsets.append(rank, b.i32_val(0));
1234: }
```

- **EN:** Defines `SharedMemoryObject::SharedMemoryObject`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `SharedMemoryObject::SharedMemoryObject`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 1236-1242

```cpp
1236: SmallVector<Value> SharedMemoryObject::getElems() const {
1237:   SmallVector<Value> elems;
1238:   // All bases come first, then offsets
1239:   elems.append(bases.begin(), bases.end());
1240:   elems.append(offsets.begin(), offsets.end());
1241:   return elems;
1242: }
```

- **EN:** Defines accessor/helper `SharedMemoryObject::getElems` that exposes or updates operation state in a compact, reusable way. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic.
- **CN:** 这里定义访问器/辅助函数 `SharedMemoryObject::getElems`，以紧凑且可复用的方式读取或更新操作状态。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。
### Lines 1244-1253

```cpp
1244: SmallVector<Type> SharedMemoryObject::getTypes() const {
1245:   SmallVector<Type> types;
1246:   // All base pointer types first
1247:   for (Value base : bases) {
1248:     types.push_back(base.getType());
1249:   }
1250:   // Then all offset types (i32)
1251:   types.append(offsets.size(), IntegerType::get(bases[0].getContext(), 32));
1252:   return types;
1253: }
```

- **EN:** Defines accessor/helper `SharedMemoryObject::getTypes` that exposes or updates operation state in a compact, reusable way. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义访问器/辅助函数 `SharedMemoryObject::getTypes`，以紧凑且可复用的方式读取或更新操作状态。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 1255-1262

```cpp
1255: uint64_t
1256: SharedMemoryObject::getMaskSpanOffsets(triton::gpu::MemDescType srcTy) {
1257:   auto ctx = srcTy.getContext();
1258:   auto shape = srcTy.getShape();
1259:   auto allocShape = srcTy.getAllocShape();
1260:   assert(allocShape.size() >= shape.size());
1261:   assert(allocShape.size() - shape.size() <= 1);
1262:   allocShape = allocShape.take_back(shape.size());
```

- **EN:** Defines accessor/helper `SharedMemoryObject::getMaskSpanOffsets` that exposes or updates operation state in a compact, reusable way. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义访问器/辅助函数 `SharedMemoryObject::getMaskSpanOffsets`，以紧凑且可复用的方式读取或更新操作状态。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 1264-1278

```cpp
1264:   // Early exist when there is no subview
1265:   if (allocShape == shape) {
1266:     return 0;
1267:   }
1268:   auto totalLl =
1269:       triton::gpu::isPaddedEncoding(srcTy.getEncoding())
1270:           ? triton::gpu::paddedLinearLayout(allocShape, srcTy.getEncoding())
1271:           : triton::gpu::toLinearLayout(allocShape, srcTy.getEncoding());
1272:   auto dimNames = standardOutDimNames(ctx, shape.size());
1273:   // Map from dimNames to offset, block
1274:   auto invLl = totalLl.pseudoinvert();
1275:   SmallVector<std::pair<StringAttr, int32_t>> logicalOffsets;
1276:   for (auto dim : standardOutDimNames(srcTy.getContext(), shape.size())) {
1277:     logicalOffsets.push_back({dim, 0});
1278:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 1280-1293

```cpp
1280:   auto ret = 0;
1281:   for (auto [dim, shapes] : llvm::enumerate(llvm::zip(shape, allocShape))) {
1282:     auto [shape, allocShape] = shapes;
1283:     for (int j = llvm::Log2_32(shape); j < llvm::Log2_32(allocShape); ++j) {
1284:       logicalOffsets[dim].second = 1 << j;
1285:       auto offsetAndBlock = invLl.apply(logicalOffsets);
1286:       ret |= offsetAndBlock[0].second;
1287:       assert(offsetAndBlock[1].second == 0);
1288:     }
1289:     // Reset the offset for the next dimension
1290:     logicalOffsets[dim].second = 0;
1291:   }
1292:   return ret;
1293: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 1295-1298

```cpp
1295: Value SharedMemoryObject::getShmemOffset(Location loc, RewriterBase &rewriter,
1296:                                          triton::gpu::MemDescType srcTy) const {
1297:   auto ctx = srcTy.getContext();
1298:   auto b = TritonLLVMOpBuilder(loc, rewriter);
```

- **EN:** Defines accessor/helper `SharedMemoryObject::getShmemOffset` that exposes or updates operation state in a compact, reusable way. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic.
- **CN:** 这里定义访问器/辅助函数 `SharedMemoryObject::getShmemOffset`，以紧凑且可复用的方式读取或更新操作状态。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。
### Lines 1300-1304

```cpp
1300:   // If it did not have a memdesc_subslice we don't need to compute the offset
1301:   // as it is zero
1302:   if (!isAffineSharedMemoryAccess(srcTy)) {
1303:     return b.i32_val(0);
1304:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1306-1313

```cpp
1306:   // We return the offset without the padding. The padding will be added in the
1307:   // lowering
1308:   LinearLayout ll;
1309:   if (triton::gpu::isPaddedEncoding(srcTy.getEncoding())) {
1310:     ll = triton::gpu::paddedLinearLayout(srcTy);
1311:   } else {
1312:     ll = triton::gpu::toLinearLayout(srcTy);
1313:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1315-1319

```cpp
1315:   auto dimNames = standardOutDimNames(ctx, offsets.size());
1316:   SmallVector<std::pair<StringAttr, Value>> logicalOffsets;
1317:   for (auto [dim, offset] : llvm::zip(dimNames, offsets)) {
1318:     logicalOffsets.push_back({dim, offset});
1319:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 1321-1328

```cpp
1321:   // We don't allow for non-trivial block dimensions in the shared memory
1322:   // layout. We have in practice that offsetAndBlock[1].second is zero, but we
1323:   // cannot assert that without constant propagation so we just discard it.
1324:   auto offset =
1325:       applyLinearLayout(loc, rewriter, ll.pseudoinvert(), logicalOffsets)[0]
1326:           .second;
1327:   return offset;
1328: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1330-1338

```cpp
1330: Value SharedMemoryObject::getShmemAffineBase(
1331:     Location loc, RewriterBase &rewriter,
1332:     triton::gpu::MemDescType srcTy) const {
1333:   assert(bases.size() == 1 &&
1334:          "getShmemAffineBase does not support partitioned shared memory");
1335:   auto b = TritonLLVMOpBuilder(loc, rewriter);
1336:   Value offset = getShmemOffset(loc, rewriter, srcTy);
1337:   return b.gep(bases[0].getType(), baseElemType, bases[0], offset);
1338: }
```

- **EN:** Defines accessor/helper `SharedMemoryObject::getShmemAffineBase` that exposes or updates operation state in a compact, reusable way. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义访问器/辅助函数 `SharedMemoryObject::getShmemAffineBase`，以紧凑且可复用的方式读取或更新操作状态。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 1340-1355

```cpp
1340: Value getStructFromSharedMemoryObject(Location loc,
1341:                                       const SharedMemoryObject &smemObj,
1342:                                       RewriterBase &rewriter) {
1343:   auto b = TritonLLVMOpBuilder(loc, rewriter);
1344:   auto elems = smemObj.getElems();
1345:   auto types = smemObj.getTypes();
1346:   auto structTy =
1347:       LLVM::LLVMStructType::getLiteral(rewriter.getContext(), types);
1348:   // pack into struct
1349:   Value llvmStruct = LLVM::UndefOp::create(rewriter, loc, structTy);
1350:   for (const auto &v : llvm::enumerate(elems)) {
1351:     assert(v.value() && "can not insert null values");
1352:     llvmStruct = b.insert_val(structTy, llvmStruct, v.value(), v.index());
1353:   }
1354:   return llvmStruct;
1355: }
```

- **EN:** Defines accessor/helper `getStructFromSharedMemoryObject` that exposes or updates operation state in a compact, reusable way. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义访问器/辅助函数 `getStructFromSharedMemoryObject`，以紧凑且可复用的方式读取或更新操作状态。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 1357-1368

```cpp
1357: SharedMemoryObject getSharedMemoryObjectFromStruct(Location loc,
1358:                                                    Value llvmStruct,
1359:                                                    Type elemTy,
1360:                                                    RewriterBase &rewriter) {
1361:   auto b = TritonLLVMOpBuilder(loc, rewriter);
1362:   ArrayRef<Type> types =
1363:       cast<LLVM::LLVMStructType>(llvmStruct.getType()).getBody();
1364:   SmallVector<Value> elems(types.size());
1365:   for (unsigned i = 0; i < types.size(); ++i) {
1366:     Type type = types[i];
1367:     elems[i] = b.extract_val(type, llvmStruct, i);
1368:   }
```

- **EN:** Defines accessor/helper `getSharedMemoryObjectFromStruct` that exposes or updates operation state in a compact, reusable way. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义访问器/辅助函数 `getSharedMemoryObjectFromStruct`，以紧凑且可复用的方式读取或更新操作状态。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 1370-1376

```cpp
1370:   // Determine how many bases there are by counting pointer types at the start.
1371:   // The struct layout is: [base0, base1, ..., baseN-1, offset0, offset1, ...]
1372:   // All bases are pointer types, all offsets are i32.
1373:   size_t numBases = llvm::count_if(
1374:       types, [](Type t) { return isa<LLVM::LLVMPointerType>(t); });
1375:   assert(numBases > 0 &&
1376:          "SharedMemoryObject struct must have at least one base");
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 1378-1381

```cpp
1378:   return {/*bases=*/{elems.begin(), elems.begin() + numBases},
1379:           /*baseElemType=*/elemTy,
1380:           /*offsets=*/{elems.begin() + numBases, elems.end()}};
1381: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1383-1395

```cpp
1383: // Build a vector containing multiple base pointers for dynamic indexing.
1384: Value buildBasePtrVector(Location loc, RewriterBase &rewriter,
1385:                          ArrayRef<Value> smemBases) {
1386:   assert(smemBases.size() > 1 && "Need multiple bases to build a vector");
1387:   auto b = TritonLLVMOpBuilder(loc, rewriter);
1388:   auto ptrTy = smemBases[0].getType();
1389:   auto vecTy = VectorType::get({static_cast<int64_t>(smemBases.size())}, ptrTy);
1390:   Value basesVec = b.undef(vecTy);
1391:   for (size_t i = 0; i < smemBases.size(); ++i) {
1392:     basesVec = b.insert_element(basesVec, smemBases[i], b.i32_val(i));
1393:   }
1394:   return basesVec;
1395: }
```

- **EN:** Defines helper `buildBasePtrVector` that computes or constructs intermediate data used by the surrounding transformation. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义辅助函数 `buildBasePtrVector`，用于计算或构造外围变换所需的中间数据。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 1397-1401

```cpp
1397: Value getStackPointer(RewriterBase &rewriter, FunctionOpInterface funcOp) {
1398:   // See NOTE: [Additional Function Arguments]
1399:   if (!isKernel(funcOp)) {
1400:     return funcOp.getArgument(funcOp.getNumArguments() + kSharedMemoryOffset);
1401:   }
```

- **EN:** Defines accessor/helper `getStackPointer` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getStackPointer`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 1403-1407

```cpp
1403:   auto mod = funcOp->getParentOfType<ModuleOp>();
1404:   auto globalBase = dyn_cast<LLVM::GlobalOp>(mod.lookupSymbol("global_smem"));
1405:   assert(globalBase);
1406:   return LLVM::AddressOfOp::create(rewriter, funcOp.getLoc(), globalBase);
1407: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 1409-1412

```cpp
1409: static Value getScratchPtrWithOffset(Location loc, RewriterBase &rewriter,
1410:                                      Value gmemBase, Value allocOffset) {
1411:   if (!allocOffset)
1412:     return gmemBase;
```

- **EN:** Defines accessor/helper `getScratchPtrWithOffset` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getScratchPtrWithOffset`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 1414-1417

```cpp
1414:   auto ptrTy = mlir::LLVM::LLVMPointerType::get(rewriter.getContext(), 1);
1415:   auto b = TritonLLVMOpBuilder(loc, rewriter);
1416:   return b.gep(ptrTy, i8_ty, gmemBase, allocOffset);
1417: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1419-1427

```cpp
1419: static Value getScratchPtrImpl(Location loc, RewriterBase &rewriter,
1420:                                const TargetInfoBase &targetInfo,
1421:                                FunctionOpInterface funcOp, Value allocOffset,
1422:                                int32_t bufferArgOffset, StringRef allocSizeAttr,
1423:                                bool addOffsetIfNoAllocSizeAttr,
1424:                                bool currentCTA = true) {
1425:   // See NOTE: [Additional Function Arguments]
1426:   auto gmemBase =
1427:       funcOp.getArgument(funcOp.getNumArguments() + bufferArgOffset);
```

- **EN:** Defines accessor/helper `getScratchPtrImpl` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getScratchPtrImpl`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 1429-1430

```cpp
1429:   if (!isKernel(funcOp))
1430:     return getScratchPtrWithOffset(loc, rewriter, gmemBase, allocOffset);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1432-1439

```cpp
1432:   ModuleOp mod = funcOp.getOperation()->getParentOfType<ModuleOp>();
1433:   auto allocSizeAttrVal =
1434:       mod.getOperation()->getAttrOfType<mlir::IntegerAttr>(allocSizeAttr);
1435:   if (!allocSizeAttrVal || allocSizeAttrVal.getValue().isZero()) {
1436:     return addOffsetIfNoAllocSizeAttr
1437:                ? getScratchPtrWithOffset(loc, rewriter, gmemBase, allocOffset)
1438:                : gmemBase;
1439:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1441-1446

```cpp
1441:   Value gridIdx[3];
1442:   Value gridDim[2];
1443:   for (int k = 0; k < 3; ++k)
1444:     gridIdx[k] = GetProgramIdOp::create(rewriter, loc, k);
1445:   for (int k = 0; k < 2; ++k)
1446:     gridDim[k] = GetNumProgramsOp::create(rewriter, loc, k);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1448-1457

```cpp
1448:   auto b = TritonLLVMOpBuilder(loc, rewriter);
1449:   auto zextToI64 = [&](Value value) {
1450:     if (value.getType() == i64_ty)
1451:       return value;
1452:     return b.zext(i64_ty, value).getResult();
1453:   };
1454:   Value linearId = zextToI64(gridIdx[2]);
1455:   for (int k = 0; k < 2; ++k)
1456:     linearId = b.add(zextToI64(gridIdx[1 - k]),
1457:                      b.mul(linearId, zextToI64(gridDim[1 - k])));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1459-1467

```cpp
1459:   auto numCTAs = triton::gpu::TritonGPUDialect::getNumCTAs(mod);
1460:   if (numCTAs > 1) {
1461:     linearId = b.mul(linearId, b.i64_val(numCTAs));
1462:     // currentCTA sets whether to rebase the linearId to the CTA id or
1463:     // just keep the pointer to the whole tensor
1464:     if (currentCTA)
1465:       linearId =
1466:           b.add(linearId, zextToI64(targetInfo.getClusterCTAId(rewriter, loc)));
1467:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1469-1472

```cpp
1469:   auto allocSize = allocSizeAttrVal.getValue().getZExtValue();
1470:   Value offset = b.mul(linearId, b.i64_val(allocSize));
1471:   if (allocOffset)
1472:     offset = b.add(offset, zextToI64(allocOffset));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1474-1477

```cpp
1474:   auto *ctx = rewriter.getContext();
1475:   return b.gep(mlir::LLVM::LLVMPointerType::get(ctx, 1), i8_ty, gmemBase,
1476:                offset);
1477: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1479-1486

```cpp
1479: Value getGlobalScratchPtr(Location loc, RewriterBase &rewriter,
1480:                           const TargetInfoBase &targetInfo,
1481:                           FunctionOpInterface funcOp, Value allocOffset) {
1482:   return getScratchPtrImpl(loc, rewriter, targetInfo, funcOp, allocOffset,
1483:                            kGlobalScratchBufferOffset,
1484:                            "ttg.global_scratch_memory_size",
1485:                            /*addOffsetIfNoAllocSizeAttr=*/false);
1486: }
```

- **EN:** Defines accessor/helper `getGlobalScratchPtr` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getGlobalScratchPtr`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 1488-1496

```cpp
1488: Value getProfileScratchPtr(Location loc, RewriterBase &rewriter,
1489:                            const TargetInfoBase &targetInfo,
1490:                            FunctionOpInterface funcOp, Value allocOffset,
1491:                            bool currentCTA) {
1492:   return getScratchPtrImpl(loc, rewriter, targetInfo, funcOp, allocOffset,
1493:                            kProfileScratchBufferOffset,
1494:                            "ttg.profile_scratch_memory_size",
1495:                            /*addOffsetIfNoAllocSizeAttr=*/true, currentCTA);
1496: }
```

- **EN:** Defines accessor/helper `getProfileScratchPtr` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getProfileScratchPtr`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 1498-1504

```cpp
1498: Value getSharedMemoryBase(Location loc, RewriterBase &rewriter,
1499:                           const TargetInfoBase &target, Operation *op) {
1500:   auto ptrTy = LLVM::LLVMPointerType::get(rewriter.getContext(),
1501:                                           target.getSharedAddressSpace());
1502:   auto func = op->template getParentOfType<FunctionOpInterface>();
1503:   if (!func)
1504:     func = cast<FunctionOpInterface>(op);
```

- **EN:** Defines accessor/helper `getSharedMemoryBase` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getSharedMemoryBase`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 1506-1511

```cpp
1506:   assert(op->hasAttr("allocation.offset"));
1507:   auto offsetAttr = op->getAttr("allocation.offset");
1508:   assert(isa<IntegerAttr>(offsetAttr) &&
1509:          "getSharedMemoryBase() called on partitioned tensor. "
1510:          "Use getSharedMemoryBases() instead.");
1511:   size_t offset = cast<IntegerAttr>(offsetAttr).getValue().getZExtValue();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 1513-1518

```cpp
1513:   auto b = TritonLLVMOpBuilder(loc, rewriter);
1514:   Value offVal = b.i32_val(offset);
1515:   Value base =
1516:       b.gep(ptrTy, i8_ty, LLVM::getStackPointer(rewriter, func), offVal);
1517:   return base;
1518: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1520-1523

```cpp
1520: SmallVector<int64_t> getPartitionOffsets(Operation *op) {
1521:   SmallVector<int64_t> offsets;
1522:   if (!op->hasAttr("allocation.offset"))
1523:     return offsets;
```

- **EN:** Defines accessor/helper `getPartitionOffsets` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getPartitionOffsets`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 1525-1536

```cpp
1525:   auto offsetAttr = op->getAttr("allocation.offset");
1526:   if (auto intAttr = dyn_cast<IntegerAttr>(offsetAttr)) {
1527:     // Single offset (non-partitioned tensor)
1528:     offsets.push_back(intAttr.getValue().getZExtValue());
1529:   } else if (auto arrayAttr = dyn_cast<ArrayAttr>(offsetAttr)) {
1530:     // Multiple offsets (partitioned tensor)
1531:     for (auto elem : arrayAttr) {
1532:       offsets.push_back(cast<IntegerAttr>(elem).getValue().getZExtValue());
1533:     }
1534:   }
1535:   return offsets;
1536: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1538-1546

```cpp
1538: SmallVector<Value> getSharedMemoryBases(Location loc, RewriterBase &rewriter,
1539:                                         const TargetInfoBase &target,
1540:                                         Operation *op) {
1541:   SmallVector<Value> bases;
1542:   auto ptrTy = LLVM::LLVMPointerType::get(rewriter.getContext(),
1543:                                           target.getSharedAddressSpace());
1544:   auto func = op->template getParentOfType<FunctionOpInterface>();
1545:   if (!func)
1546:     func = cast<FunctionOpInterface>(op);
```

- **EN:** Defines accessor/helper `getSharedMemoryBases` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getSharedMemoryBases`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 1548-1550

```cpp
1548:   auto offsets = getPartitionOffsets(op);
1549:   auto b = TritonLLVMOpBuilder(loc, rewriter);
1550:   Value stackPtr = LLVM::getStackPointer(rewriter, func);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1552-1558

```cpp
1552:   for (int64_t offset : offsets) {
1553:     Value offVal = b.i32_val(offset);
1554:     Value base = b.gep(ptrTy, i8_ty, stackPtr, offVal);
1555:     bases.push_back(base);
1556:   }
1557:   return bases;
1558: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1560-1560

```cpp
1560: namespace {
```

- **EN:** Opens or closes the namespace nesting for (anonymous), keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 (anonymous) 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 1562-1568

```cpp
1562: // Extract the bits of `a` that are set in `mask`
1563: Value pext_i32(RewriterBase &rewriter, Location loc, Value a, uint32_t mask) {
1564:   auto b = TritonLLVMOpBuilder(loc, rewriter);
1565:   assert(a.getType() == i32_ty && "a must be i32");
1566:   // Handle width = 32 to avoid doing 1 << 32
1567:   if (mask == 0xFFFFFFFF)
1568:     return a;
```

- **EN:** Defines `pext_i32`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `pext_i32`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 1570-1587

```cpp
1570:   // Implements the blocked algorithm from
1571:   // https://forums.developer.nvidia.com/t/pdep-and-pext-functionality-for-cuda/270973
1572:   uint32_t mskConst = mask;
1573:   uint32_t extcnt = 0;
1574:   Value result = b.i32_val(0);
1575:   while (mskConst) {
1576:     uint32_t oldmsk = mskConst;
1577:     uint32_t bitgrplsb = mskConst & (-mskConst);
1578:     mskConst &= bitgrplsb + mskConst;
1579:     uint32_t bitgrp = mskConst ^ oldmsk;
1580:     uint32_t lsbpos = 31 - __builtin_clz(bitgrplsb);
1581:     // like popcount for a number 0..01..1..0 but portable
1582:     uint32_t grplen = __builtin_ctz(~(bitgrp >> lsbpos));
1583:     uint32_t shift = lsbpos - extcnt;
1584:     extcnt += grplen;
1585:     result =
1586:         b.or_(result, b.lshr(b.and_(b.i32_val(bitgrp), a), b.i32_val(shift)));
1587:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1588-1589

```cpp
1588:   return result;
1589: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1591-1594

```cpp
1591: // Puts the bits of `a` that are set in `mask` into the bits of `result`
1592: Value pdep_i32(RewriterBase &rewriter, Location loc, Value a, uint32_t mask) {
1593:   auto b = TritonLLVMOpBuilder(loc, rewriter);
1594:   assert(a.getType() == i32_ty && "a must be i32");
```

- **EN:** Defines `pdep_i32`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `pdep_i32`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 1596-1598

```cpp
1596:   if (mask == 0)
1597:     return b.i32_val(0);
1598:   assert(mask < 64 && "mask must be less than 64");
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 1600-1603

```cpp
1600:   // Blocked algorithm (same grouping trick as the pext example).
1601:   uint32_t mskConst = mask;
1602:   uint32_t depcnt = 0; // how many source bits from `a` we've consumed
1603:   Value result = b.i32_val(0);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1605-1606

```cpp
1605:   while (mskConst) {
1606:     uint32_t oldmsk = mskConst;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1608-1611

```cpp
1608:     // Isolate lsb set bit, then clear the lowest contiguous run of 1s.
1609:     uint32_t bitgrplsb = mskConst & (~mskConst + 1); // m & -m
1610:     mskConst &= (bitgrplsb + mskConst);
1611:     uint32_t bitgrp = mskConst ^ oldmsk; // the cleared run (contiguous 1s)
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1613-1615

```cpp
1613:     // Group start position and length.
1614:     uint32_t lsbpos = __builtin_ctz(bitgrplsb);
1615:     uint32_t grplen = __builtin_ctz(~(bitgrp >> lsbpos));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1617-1621

```cpp
1617:     // Align the next grplen bits of `a` to the group's lsb, then mask to the
1618:     // group.
1619:     uint32_t shift =
1620:         lsbpos - depcnt; // non-negative invariant for this traversal order
1621:     depcnt += grplen;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1623-1625

```cpp
1623:     Value deposited = b.and_(b.shl(a, b.i32_val(shift)), b.i32_val(bitgrp));
1624:     result = b.or_(result, deposited);
1625:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1627-1628

```cpp
1627:   return result;
1628: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1630-1630

```cpp
1630: } // namespace
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 1632-1647

```cpp
1632: std::tuple<SmallVector<Value>, Value>
1633: delinearize(RewriterBase &rewriter, Location loc,
1634:             triton::gpu::DistributedEncodingTrait layout,
1635:             ArrayRef<int64_t> shape, StringAttr dimName, Value linear) {
1636:   auto b = TritonLLVMOpBuilder(loc, rewriter);
1637:   auto ll = triton::gpu::toLinearLayout(shape, layout);
1638:   assert(ll.hasInDim(dimName));
1639:   int32_t freeVarMask = ll.getFreeVariableMasks()[dimName];
1640:   auto isRepresentative = b.true_val();
1641:   if (freeVarMask != 0) {
1642:     isRepresentative =
1643:         b.icmp_eq(b.and_(b.i32_val(freeVarMask), linear), b.i32_val(0));
1644:     // We remove the bits of linear that are set to one in freeVarMask
1645:     int32_t nonFreeVarMask = ~freeVarMask & (ll.getInDimSize(dimName) - 1);
1646:     linear = pext_i32(rewriter, loc, linear, nonFreeVarMask);
1647:   }
```

- **EN:** Defines `delinearize`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `delinearize`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 1649-1653

```cpp
1649:   auto linearLayout = triton::gpu::LinearEncodingAttr::get(
1650:       rewriter.getContext(), std::move(ll));
1651:   auto orderDim = linearLayout.orderPerDim(dimName, linearLayout.getOrder());
1652:   auto shapeDim = linearLayout.basesPerDim(dimName, /*skipBroadcast=*/true);
1653:   auto multiDim = delinearize(rewriter, loc, linear, shapeDim, orderDim);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1655-1656

```cpp
1655:   return std::make_tuple(std::move(multiDim), isRepresentative);
1656: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1658-1675

```cpp
1658: // Convert an \param index to a multi-dim coordinate given \param shape and
1659: // \param order.
1660: SmallVector<Value> delinearize(RewriterBase &rewriter, Location loc,
1661:                                Value linear, ArrayRef<unsigned> shape,
1662:                                ArrayRef<unsigned> order) {
1663:   unsigned rank = shape.size();
1664:   assert(rank == order.size());
1665:   auto reordered = applyPermutation(shape, order);
1666:   SmallVector<Value> reorderedMultiDim(rank);
1667:   if (auto constantOp = linear.getDefiningOp<arith::ConstantOp>()) {
1668:     unsigned intVal = mlir::cast<IntegerAttr>(constantOp.getValue())
1669:                           .getValue()
1670:                           .getSExtValue();
1671:     reorderedMultiDim = delinearize(rewriter, loc, intVal, reordered);
1672:   } else {
1673:     reorderedMultiDim = delinearize(rewriter, loc, linear, reordered);
1674:   }
1675:   SmallVector<Value> multiDim(rank);
```

- **EN:** Defines `delinearize`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `delinearize`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 1676-1680

```cpp
1676:   for (unsigned i = 0; i < rank; ++i) {
1677:     multiDim[order[i]] = reorderedMultiDim[i];
1678:   }
1679:   return multiDim;
1680: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1682-1695

```cpp
1682: SmallVector<Value> delinearize(RewriterBase &rewriter, Location loc,
1683:                                unsigned linear, ArrayRef<unsigned> shape) {
1684:   auto b = TritonLLVMOpBuilder(loc, rewriter);
1685:   unsigned rank = shape.size();
1686:   assert(rank > 0);
1687:   SmallVector<Value> multiDim(rank);
1688:   unsigned remained = linear;
1689:   for (auto &&en : llvm::enumerate(shape)) {
1690:     unsigned dimSize = en.value();
1691:     multiDim[en.index()] = b.i32_val(remained % dimSize);
1692:     remained = remained / dimSize;
1693:   }
1694:   return multiDim;
1695: }
```

- **EN:** Defines `delinearize`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `delinearize`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 1697-1710

```cpp
1697: SmallVector<Value> delinearize(RewriterBase &rewriter, Location loc,
1698:                                Value linear, ArrayRef<unsigned> shape) {
1699:   auto b = TritonLLVMOpBuilder(loc, rewriter);
1700:   unsigned rank = shape.size();
1701:   assert(rank > 0);
1702:   SmallVector<Value> multiDim(rank);
1703:   Value remained = linear;
1704:   for (auto &&en : llvm::enumerate(shape)) {
1705:     Value dimSize = b.i32_val(en.value());
1706:     multiDim[en.index()] = b.urem(remained, dimSize);
1707:     remained = b.udiv(remained, dimSize);
1708:   }
1709:   return multiDim;
1710: }
```

- **EN:** Defines `delinearize`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `delinearize`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 1712-1723

```cpp
1712: SmallVector<unsigned> delinearize(unsigned linear, ArrayRef<unsigned> shape,
1713:                                   ArrayRef<unsigned> order) {
1714:   auto rank = shape.size();
1715:   assert(order.size() == rank);
1716:   SmallVector<unsigned> multiDim(rank);
1717:   for (auto dim : order) {
1718:     multiDim[dim] = linear % shape[dim];
1719:     linear /= shape[dim];
1720:   }
1721:   assert(linear == 0);
1722:   return multiDim;
1723: }
```

- **EN:** Defines `delinearize`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `delinearize`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 1725-1729

```cpp
1725: Value linearize(RewriterBase &rewriter, Location loc, ArrayRef<Value> multiDim,
1726:                 ArrayRef<unsigned> shape, ArrayRef<unsigned> order) {
1727:   return linearize(rewriter, loc, applyPermutation(multiDim, order),
1728:                    applyPermutation(shape, order));
1729: }
```

- **EN:** Defines `linearize`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `linearize`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1731-1745

```cpp
1731: Value linearize(RewriterBase &rewriter, Location loc, ArrayRef<Value> multiDim,
1732:                 ArrayRef<unsigned> shape) {
1733:   auto b = TritonLLVMOpBuilder(loc, rewriter);
1734:   auto rank = multiDim.size();
1735:   Value linear = b.i32_val(0);
1736:   if (rank > 0) {
1737:     linear = multiDim.back();
1738:     for (auto [dim, dimShape] :
1739:          llvm::reverse(llvm::zip(multiDim.drop_back(), shape.drop_back()))) {
1740:       Value dimSize = b.i32_val(dimShape);
1741:       linear = b.add(b.mul(linear, dimSize), dim);
1742:     }
1743:   }
1744:   return linear;
1745: }
```

- **EN:** Defines `linearize`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `linearize`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1747-1759

```cpp
1747: Value linearize(RewriterBase &rewriter, Location loc, ArrayRef<Value> multiDim,
1748:                 triton::gpu::LinearEncodingAttr encoding, StringAttr dimName) {
1749:   auto orderDim = encoding.orderPerDim(dimName, encoding.getOrder());
1750:   auto shapeDim = encoding.basesPerDim(dimName, /*skipBroadcast=*/true);
1751:   auto linear = linearize(rewriter, loc, multiDim, shapeDim, orderDim);
1752:   auto ll = encoding.getLinearLayout();
1753:   int32_t freeVarMask = ll.getFreeVariableMasks().lookup(dimName);
1754:   if (freeVarMask != 0) {
1755:     int32_t nonFreeVarMask = ~freeVarMask & (ll.getInDimSize(dimName) - 1);
1756:     linear = pdep_i32(rewriter, loc, linear, nonFreeVarMask);
1757:   }
1758:   return linear;
1759: }
```

- **EN:** Defines `linearize`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `linearize`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1761-1767

```cpp
1761: size_t linearize(ArrayRef<unsigned> multiDim, ArrayRef<unsigned> shape,
1762:                  ArrayRef<unsigned> order) {
1763:   size_t linear = 0;
1764:   for (unsigned dim : llvm::reverse(order))
1765:     linear = linear * shape[dim] + multiDim[dim];
1766:   return linear;
1767: }
```

- **EN:** Defines `linearize`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `linearize`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1769-1776

```cpp
1769: LLVM::GlobalOp getOrInsertGlobalConstant(RewriterBase &rewriter,
1770:                                          ModuleOp module, Type type,
1771:                                          Attribute content, StringRef key) {
1772:   for (auto op : module.getOps<LLVM::GlobalOp>()) {
1773:     if (op.getConstant() && op.getLinkage() == LLVM::Linkage::Internal &&
1774:         op.getType() == type && op.getValueAttr() == content)
1775:       return op;
1776:   }
```

- **EN:** Defines accessor/helper `getOrInsertGlobalConstant` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getOrInsertGlobalConstant`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 1778-1789

```cpp
1778:   unsigned stringNumber = 0;
1779:   SmallString<16> name;
1780:   do {
1781:     name.clear();
1782:     (key + Twine(stringNumber++)).toStringRef(name);
1783:   } while (module.lookupSymbol(name));
1784:   RewriterBase::InsertionGuard guard(rewriter);
1785:   rewriter.setInsertionPointToStart(module.getBody());
1786:   return LLVM::GlobalOp::create(rewriter, UnknownLoc::get(type.getContext()),
1787:                                 type, /*isConstant=*/true,
1788:                                 LLVM::Linkage::Internal, name, content);
1789: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1791-1795

```cpp
1791: Value addStringToModule(Location loc, RewriterBase &rewriter, StringRef key,
1792:                         StringRef content) {
1793:   auto b = TritonLLVMOpBuilder(loc, rewriter);
1794:   auto moduleOp = rewriter.getBlock()->getParent()->getParentOfType<ModuleOp>();
1795:   auto ctx = moduleOp.getContext();
```

- **EN:** Defines `addStringToModule`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `addStringToModule`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1797-1802

```cpp
1797:   llvm::SmallString<64> contentStr(content);
1798:   size_t contentSize = contentStr.size_in_bytes();
1799:   auto globalType = LLVM::LLVMArrayType::get(i8_ty, contentSize);
1800:   auto contentAttr = rewriter.getStringAttr(contentStr);
1801:   LLVM::GlobalOp global = getOrInsertGlobalConstant(
1802:       rewriter, moduleOp, globalType, contentAttr, key);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1804-1811

```cpp
1804:   Value zero = b.i32_val(0);
1805:   Type globalPtrType = LLVM::LLVMPointerType::get(ctx, global.getAddrSpace());
1806:   Value globalPtr = LLVM::AddressOfOp::create(
1807:       rewriter, UnknownLoc::get(ctx), globalPtrType, global.getSymName());
1808:   Value stringStart =
1809:       b.gep(ptr_ty(ctx), i8_ty, globalPtr, SmallVector<Value>({zero}));
1810:   return stringStart;
1811: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1813-1813

```cpp
1813: } // namespace LLVM
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 1815-1824

```cpp
1815: Value dot(RewriterBase &rewriter, Location loc, ArrayRef<Value> offsets,
1816:           ArrayRef<Value> strides) {
1817:   assert(offsets.size() == strides.size());
1818:   auto b = TritonLLVMOpBuilder(loc, rewriter);
1819:   Value ret = b.i32_val(0);
1820:   for (auto [offset, stride] : llvm::zip(offsets, strides)) {
1821:     ret = b.add(ret, b.mul(offset, stride));
1822:   }
1823:   return ret;
1824: }
```

- **EN:** Defines `dot`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `dot`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 1826-1840

```cpp
1826: // Isolated a single warp specialize op from above.
1827: static void
1828: makeWarpGroupsIsolatedFromAbove(triton::gpu::WarpSpecializeOp wsOp) {
1829:   SetVector<Value> captures;
1830:   auto partOp = wsOp.getPartitionOp();
1831:   getUsedValuesDefinedAbove(partOp.getPartitionRegions(), captures);
1832:   for (Value capture : captures) {
1833:     partOp->insertOperands(partOp.getNumOperands(), capture);
1834:     for (Region &region : partOp.getPartitionRegions()) {
1835:       BlockArgument arg =
1836:           region.addArgument(capture.getType(), capture.getLoc());
1837:       replaceAllUsesInRegionWith(capture, arg, region);
1838:     }
1839:   }
1840: }
```

- **EN:** Defines `makeWarpGroupsIsolatedFromAbove`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `makeWarpGroupsIsolatedFromAbove`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1842-1846

```cpp
1842: void makeAllWarpGroupsIsolatedFromAbove(Operation *op) {
1843:   op->walk([](triton::gpu::WarpSpecializeOp wsOp) {
1844:     makeWarpGroupsIsolatedFromAbove(wsOp);
1845:   });
1846: }
```

- **EN:** Defines `makeAllWarpGroupsIsolatedFromAbove`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `makeAllWarpGroupsIsolatedFromAbove`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1848-1865

```cpp
1848: // TODO: Is there a better way to do this? This needs to be fixed upstream.
1849: void fixUpLoopAnnotation(ModuleOp mod) {
1850:   mod->walk([](Operation *op) {
1851:     if (isa<LLVM::BrOp, LLVM::CondBrOp>(op)) {
1852:       if (op->hasAttr("llvm.loop_annotation")) {
1853:         auto loopMD = dyn_cast<LLVM::LoopAnnotationAttr>(
1854:             op->getAttr("llvm.loop_annotation"));
1855:         if (loopMD) {
1856:           if (auto brOp = dyn_cast<LLVM::BrOp>(op)) {
1857:             brOp.setLoopAnnotationAttr(loopMD);
1858:           } else if (auto condBrOp = dyn_cast<LLVM::CondBrOp>(op)) {
1859:             condBrOp.setLoopAnnotationAttr(loopMD);
1860:           }
1861:         }
1862:       }
1863:     }
1864:   });
1865: }
```

- **EN:** Defines `fixUpLoopAnnotation`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `fixUpLoopAnnotation`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1867-1884

```cpp
1867: SmallVector<Value> inlineRegionImpl(RewriterBase &rewriter, Region &region,
1868:                                     ArrayRef<Value> args,
1869:                                     mlir::TypeID terminatorTypeId,
1870:                                     Location loc) {
1871:   // Inline regions with multiple blocks
1872:   //
1873:   //        Before                                   After
1874:   //                                              ┌─────────┐
1875:   //                                              │ op1     │
1876:   //                    ┌──────────┐              │ cf.br   │
1877:   //                    │region[0] │              └────┬────┘
1878:   //                    │cf.cond_br├─┐            ┌────▼─────┐
1879:   //                    └────┬─────┘ │            │region[0] │
1880:   //                         │       │            │cf.cond_br├─┐
1881:   // ┌───────┐          ┌────▼────┐  │            └────┬─────┘ │
1882:   // │  op1  │  IP      │region[1]│  │            ┌────▼────┐  │
1883:   // │       │◄───      │yield ...│  │            │region[1]│  │
1884:   // │  op2  │          └─────────┘  │          ┌─┤cf.br    │  │
```

- **EN:** Defines `inlineRegionImpl`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `inlineRegionImpl`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1885-1895

```cpp
1885:   // └───────┘                       │          │ └─────────┘  │
1886:   //                    ┌─────────┐  │          │ ┌─────────┐  │
1887:   //                    │region[2]│◄─┘          │ │region[2]│◄─┘
1888:   //                    │yield    │             │ │cf.br    │
1889:   //                    └─────────┘             │ └────┬────┘
1890:   //                                            │ ┌────▼────┐
1891:   //                                            └►│op2      │
1892:   //                                              └─────────┘
1893:   auto *curBlock = rewriter.getInsertionBlock();
1894:   auto opPosition = rewriter.getInsertionPoint();
1895:   auto *remainingOpsBlock = rewriter.splitBlock(curBlock, opPosition);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1897-1901

```cpp
1897:   IRMapping regionMap;
1898:   Region &parent = *curBlock->getParent();
1899:   rewriter.cloneRegionBefore(region, parent, parent.end(), regionMap);
1900:   rewriter.setInsertionPointToEnd(curBlock);
1901:   LLVM::BrOp::create(rewriter, loc, args, regionMap.lookup(&region.front()));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1903-1906

```cpp
1903:   ValueRange terminatorOperands;
1904:   for (Block &origBlock : region) {
1905:     Block *newBlock = regionMap.lookup(&origBlock);
1906:     rewriter.moveBlockBefore(newBlock, remainingOpsBlock);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1908-1915

```cpp
1908:     auto terminator = newBlock->getTerminator();
1909:     if (terminator->getRegisteredInfo()->getTypeID() == terminatorTypeId) {
1910:       terminatorOperands = terminator->getOperands();
1911:       rewriter.setInsertionPointAfter(terminator);
1912:       rewriter.replaceOpWithNewOp<LLVM::BrOp>(terminator, terminatorOperands,
1913:                                               remainingOpsBlock);
1914:     }
1915:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. It finishes by replacing the original operation with the lowered form.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 它会在转换成功后用降级后的形式替换原始操作。
### Lines 1917-1924

```cpp
1917:   rewriter.setInsertionPointToStart(remainingOpsBlock);
1918:   SmallVector<Value> vals;
1919:   for (auto resultTy : terminatorOperands.getType()) {
1920:     auto val = remainingOpsBlock->addArgument(resultTy, loc);
1921:     vals.push_back(val);
1922:   }
1923:   return vals;
1924: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 1926-1929

```cpp
1926: std::tuple<Block *, Block *, Block *> createIfBlock(RewriterBase &b,
1927:                                                     Location loc, Value cnd) {
1928:   Block *prevBlock = b.getInsertionBlock();
1929:   Block *ifBlock = b.splitBlock(prevBlock, b.getInsertionPoint());
```

- **EN:** Defines helper `createIfBlock` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createIfBlock`，用于计算或构造外围变换所需的中间数据。
### Lines 1931-1937

```cpp
1931:   // Split a block after the call.
1932:   Block *thenBlock = b.splitBlock(ifBlock, ifBlock->begin());
1933:   b.setInsertionPointToEnd(ifBlock);
1934:   LLVM::BrOp::create(b, loc, thenBlock);
1935:   b.setInsertionPointToEnd(prevBlock);
1936:   LLVM::CondBrOp::create(b, loc, cnd, ifBlock, thenBlock);
1937:   b.setInsertionPointToStart(thenBlock);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1939-1940

```cpp
1939:   return {prevBlock, ifBlock, thenBlock};
1940: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1942-1958

```cpp
1942: void finalizeTensorAtomicResults(Operation *op, RankedTensorType tensorTy,
1943:                                  ConversionPatternRewriter &rewriter,
1944:                                  SmallVector<Value> &resultVals,
1945:                                  Type valueElemTy, TritonLLVMOpBuilder &b,
1946:                                  Value threadPred,
1947:                                  const TargetInfoBase &targetInfo,
1948:                                  const LLVMTypeConverter *typeConverter) {
1949:   auto *ctx = rewriter.getContext();
1950:   auto loc = op->getLoc();
1951:   Type structTy = typeConverter->convertType(tensorTy);
1952:   if (!op->hasAttr("allocation.offset")) {
1953:     // No broadcasting, just pack the values into a struct
1954:     Value resultStruct =
1955:         packLLElements(loc, typeConverter, resultVals, rewriter, structTy);
1956:     rewriter.replaceOp(op, {resultStruct});
1957:     return;
1958:   }
```

- **EN:** Defines `finalizeTensorAtomicResults`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. It finishes by replacing the original operation with the lowered form. Type mapping is delegated to the LLVM type converter or related conversion helpers.
- **CN:** 这里定义 `finalizeTensorAtomicResults`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 它会在转换成功后用降级后的形式替换原始操作。 类型映射委托给 LLVM 类型转换器或相关转换辅助函数。
### Lines 1960-1972

```cpp
1960:   // Yanky way of "composing with the associated contiguous shmem layout"
1961:   auto kOffset = str_attr("offset");
1962:   auto kBlock = str_attr("block");
1963:   auto dstLayout = triton::gpu::toLinearLayout(tensorTy);
1964:   auto dimOut = dstLayout.getTotalOutDimSize();
1965:   auto dimBlock = dstLayout.getInDimSize(kBlock);
1966:   // You should create a Shared linear layout with kBlock bases equal to the
1967:   // kBlock of dstLayout and then put all the other bases in order in the
1968:   // offsets.
1969:   assert(dimBlock == 1 && "NYI");
1970:   dstLayout = dstLayout.flattenOuts().reshapeOuts(
1971:       {{kOffset, dimOut / dimBlock}, {kBlock, dimBlock}});
1972:   auto smemBase = LLVM::getSharedMemoryBase(loc, rewriter, targetInfo, op);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 1974-1983

```cpp
1974:   auto emitSt = [&](RewriterBase &rewriter, Location loc, ArrayRef<Value> vals,
1975:                     Value shmemAddr, int idx, VectorType vecTy,
1976:                     std::optional<Value> ctaId) -> SmallVector<Value> {
1977:     auto length = vecTy.getNumElements();
1978:     Value valsVec =
1979:         packLLVector(loc, ArrayRef<Value>(vals).slice(idx, length), rewriter);
1980:     targetInfo.storeDShared(rewriter, loc, shmemAddr, ctaId, valsVec,
1981:                             threadPred);
1982:     return {};
1983:   };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1985-1991

```cpp
1985:   auto emitLd = [&](RewriterBase &rewriter, Location loc, ArrayRef<Value> vals,
1986:                     Value shmemAddr, int idx, VectorType vecTy,
1987:                     std::optional<Value> ctaId) -> SmallVector<Value> {
1988:     Value loadedVec = targetInfo.loadDShared(rewriter, loc, shmemAddr, ctaId,
1989:                                              vecTy, b.true_val());
1990:     return unpackLLVector(loc, loadedVec, rewriter);
1991:   };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1993-1999

```cpp
1993:   auto [laneId, warpId] = getLaneAndWarpId(rewriter, loc);
1994:   SmallVector<Value> smemBases = {smemBase};
1995:   lowerLdSt(loc, ctx, dstLayout, resultVals, valueElemTy, smemBases,
1996:             /*paddingShifts=*/{}, /*affineOffset=*/b.i32_val(0),
1997:             /*maskSpanAffineOffset=*/0, laneId, warpId, rewriter, targetInfo,
1998:             /*maybeMaxVecElems=*/{}, emitSt);
1999:   b.barrier(triton::gpu::AddrSpace::Local);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2001-2005

```cpp
2001:   resultVals =
2002:       lowerLdSt(loc, ctx, dstLayout, resultVals, valueElemTy, smemBases,
2003:                 /*paddingShifts=*/{}, /*affineOffset=*/b.i32_val(0),
2004:                 /*maskSpanAffineOffset=*/0, laneId, warpId, rewriter,
2005:                 targetInfo, /*maybeMaxVecElems=*/{}, emitLd);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2007-2011

```cpp
2007:   // Create the result struct and replace the operation
2008:   Value resultStruct =
2009:       packLLElements(loc, typeConverter, resultVals, rewriter, structTy);
2010:   rewriter.replaceOp(op, {resultStruct});
2011: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. It finishes by replacing the original operation with the lowered form.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 它会在转换成功后用降级后的形式替换原始操作。
### Lines 2013-2017

```cpp
2013: // Only retain those attributes that are not constructed by
2014: // `LLVMFuncOp::build`. If `filterArgAttrs` is set, also filter out argument
2015: // attributes.
2016: void filterFuncAttributes(triton::FuncOp op, bool filterArgAttrs,
2017:                           SmallVectorImpl<NamedAttribute> &result) {
```

- **EN:** Defines `filterFuncAttributes`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `filterFuncAttributes`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 2019-2028

```cpp
2019:   for (const auto &attr : op->getAttrs()) {
2020:     if (attr.getName() == SymbolTable::getSymbolAttrName() ||
2021:         attr.getName() == op.getFunctionTypeAttrName() ||
2022:         attr.getName() == "std.varargs" ||
2023:         attr.getName() == triton::gpu::AttrNumWarpsName ||
2024:         (filterArgAttrs && attr.getName() == op.getArgAttrsAttrName()))
2025:       continue;
2026:     result.push_back(attr);
2027:   }
2028: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2030-2040

```cpp
2030: triton::FuncOp amendFuncOp(triton::FuncOp funcOp,
2031:                            ConversionPatternRewriter &rewriter,
2032:                            const TargetInfoBase &targetInfo) {
2033:   // Push back two new arguments that indicate the current pointer to shared
2034:   // memory and global scratch memory.
2035:   auto loc = funcOp.getLoc();
2036:   auto ctx = funcOp->getContext();
2037:   auto sharedPtrTy =
2038:       LLVM::LLVMPointerType::get(ctx, targetInfo.getSharedAddressSpace());
2039:   auto globalPtrTy = LLVM::LLVMPointerType::get(ctx, 1);
2040:   auto profilePtrTy = LLVM::LLVMPointerType::get(ctx, 1);
```

- **EN:** Defines `amendFuncOp`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `amendFuncOp`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 2042-2058

```cpp
2042:   // 1. Modify the function type to add the new arguments.
2043:   auto funcTy = funcOp.getFunctionType();
2044:   auto amendedInputTy = llvm::to_vector<4>(funcTy.getInputs());
2045:   bool isKernel = triton::isKernel(funcOp);
2046:   if (isKernel && targetInfo.isCuda()) {
2047:     for (auto i : llvm::seq(amendedInputTy.size())) {
2048:       if (isa<triton::TensorDescInterface>(amendedInputTy[i])) {
2049:         funcOp.setArgAttr(i, "tt.nv_tma_desc",
2050:                           mlir::IntegerAttr::get(i32_ty, 1));
2051:       }
2052:     }
2053:   }
2054:   if (!isKernel) {
2055:     amendedInputTy.push_back(sharedPtrTy);
2056:   }
2057:   amendedInputTy.push_back(globalPtrTy);
2058:   amendedInputTy.push_back(profilePtrTy);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2059-2072

```cpp
2059:   auto amendedFuncTy =
2060:       FunctionType::get(ctx, amendedInputTy, funcTy.getResults());
2061:   // 2. Modify the argument attributes to add the new argument.
2062:   SmallVector<NamedAttribute> amendedAttrs;
2063:   filterFuncAttributes(funcOp, /*filterArgAttrs=*/true, amendedAttrs);
2064:   if (auto argAttrs = funcOp.getAllArgAttrs()) {
2065:     llvm::SmallVector<mlir::Attribute> amendedArgAttrs(argAttrs.begin(),
2066:                                                        argAttrs.end());
2067:     while (amendedArgAttrs.size() < amendedInputTy.size()) {
2068:       amendedArgAttrs.emplace_back(DictionaryAttr::get(ctx));
2069:     }
2070:     amendedAttrs.push_back(rewriter.getNamedAttr(
2071:         funcOp.getArgAttrsAttrName(), rewriter.getArrayAttr(amendedArgAttrs)));
2072:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2074-2086

```cpp
2074:   // 3. Add the new arguments to the region
2075:   auto amendedFuncOp = triton::FuncOp::create(
2076:       rewriter, funcOp.getLoc(), funcOp.getName(), amendedFuncTy, amendedAttrs);
2077:   auto &region = funcOp.getBody();
2078:   if (!isKernel) {
2079:     region.addArgument(sharedPtrTy, loc);
2080:   }
2081:   region.addArgument(globalPtrTy, loc);
2082:   region.addArgument(profilePtrTy, loc);
2083:   rewriter.inlineRegionBefore(region, amendedFuncOp.getBody(),
2084:                               amendedFuncOp.end());
2085:   return amendedFuncOp;
2086: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2088-2101

```cpp
2088: void handleArgPtrDatatype(triton::FuncOp funcOp, LLVM::LLVMFuncOp &llvmFuncOp) {
2089:   // The convertion from triton::PointerType to LLVM::LLVMPointerType losts
2090:   // the pointee datatype information.
2091:   // This function add back the pointee datatype information to arg attribute.
2092:   FunctionType fty = funcOp.getFunctionType();
2093:   for (unsigned i = 0; i < fty.getNumInputs(); ++i) {
2094:     auto argType = fty.getInput(i);
2095:     if (auto argPtrType = dyn_cast<triton::PointerType>(argType)) {
2096:       auto argDType = argPtrType.getPointeeType();
2097:       llvmFuncOp.setArgAttr(i, "tt.pointee_type",
2098:                             mlir::TypeAttr::get(argDType));
2099:     }
2100:   }
2101: }
```

- **EN:** Defines `handleArgPtrDatatype`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `handleArgPtrDatatype`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 2103-2103

```cpp
2103: } // namespace mlir
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。

## Key Concepts / 关键概念
- **EN:** The file focuses on lowering utility related IR into a more target-oriented representation.
  **CN:** 本文件重点是把与 Utility 相关的 IR 降级为更面向目标的表示。
- **EN:** Type conversion bridges Triton/MLIR types to LLVM-level data layouts.
  **CN:** 类型转换负责把 Triton/MLIR 类型映射到 LLVM 层的数据布局。
- **EN:** Pattern rewriting is the main mechanism for canonicalization and lowering in this file.
  **CN:** 模式重写是本文件进行规范化与降级转换的核心机制。
- **EN:** Conversion rewrites replace source dialect ops with target dialect values while preserving semantics.
  **CN:** 转换式重写通过替换源方言操作来保持语义并生成目标方言值。
- **EN:** Shared-memory objects model on-chip scratchpad allocation and addressing.
  **CN:** SharedMemoryObject 用于建模片上共享内存的分配与寻址。
- **EN:** Memory descriptor types carry layout, rank, and address-space information.
  **CN:** 内存描述符类型携带布局、维度和地址空间信息。

## Dependencies / 依赖关系
- **Internal Triton headers / Triton 内部头文件:** `triton/Conversion/TritonGPUToLLVM/Utility.h`, `triton/Analysis/Allocation.h`, `triton/Analysis/Utility.h`, `triton/Conversion/TritonGPUToLLVM/TargetInfoBase.h`, `triton/Dialect/Triton/IR/Dialect.h`, `triton/Dialect/TritonGPU/IR/Attributes.h`, ... (+6 more)
- **MLIR headers / MLIR 头文件:** `mlir/Dialect/LLVMIR/LLVMDialect.h`, `mlir/IR/Attributes.h`, `mlir/Transforms/RegionUtils.h`
- **LLVM headers / LLVM 头文件:** `llvm/ADT/STLExtras.h`, `llvm/Support/MathExtras.h`
- **Standard/library headers / 标准或通用库头文件:** `functional`, `intrin.h`
- **Primary APIs used / 主要 API:** `PatternRewriter`, `ConversionPatternRewriter`, `LLVMTypeConverter`, `ModuleOp`, `RankedTensorType`, `MemDescType`, ... (+3 more)
- **Lowering role / 降级角色:** The implementation depends on source-dialect semantics being valid and emits forms expected by later LLVM or GPU stages. / 该实现依赖源方言语义已经正确，并输出供后续 LLVM 或 GPU 阶段使用的形式。
