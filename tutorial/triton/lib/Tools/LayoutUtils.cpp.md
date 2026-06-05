# LayoutUtils.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `lib/Tools/LayoutUtils.cpp`
- **Purpose / 作用:** **EN:** Provides reusable utility code for Layout Utils used across the Triton implementation. **CN:** 提供与 Layout Utils 相关的可复用工具代码，供 Triton 各模块共享。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2

```cpp
   1: #include "triton/Tools/LayoutUtils.h"
   2: #include "triton/Tools/GenericSwizzling.h"
```

- **EN:** Includes the interfaces this file depends on. Triton headers (`LayoutUtils.h`, `GenericSwizzling.h`) provide domain-specific IR/support, MLIR headers (None) provide rewriting and analysis infrastructure, LLVM headers (None) supply low-level utilities, and standard/library headers (None) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（`LayoutUtils.h`, `GenericSwizzling.h`）提供领域特定 IR/支持逻辑，MLIR 头文件（None）提供重写与分析基础设施，LLVM 头文件（None）提供底层工具，而标准/通用库头文件（None）提供通用能力。
### Lines 4-4

```cpp
   4: namespace mlir::triton {
```

- **EN:** Opens or closes the namespace nesting for mlir::triton, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 mlir::triton 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 6-23

```cpp
   6: static bool checkSquareSublayout(const LinearLayout &ll,
   7:                                  ArrayRef<StringAttr> dimNames,
   8:                                  function_ref<bool(int, int32_t)> checkBasis) {
   9:   // The empty layout is the identity
  10:   if (dimNames.size() == 0) {
  11:     return true;
  12:   }
  13:   // Check that the input-output sizes are the same
  14:   LinearLayout sl = ll.sublayout(dimNames, dimNames);
  15:   for (StringAttr dim : dimNames) {
  16:     if (ll.getInDimSize(dim) != ll.getOutDimSize(dim)) {
  17:       return false;
  18:     }
  19:   }
  20:   // Once the inputs and output dimensions are the same, we can just check
  21:   // that the basis for the single remaining dimension is the identity.
  22:   sl = sl.flattenIns().flattenOuts();
  23:   const auto &inDimBases = sl.getBases().begin()->second;
```

- **EN:** Defines `checkSquareSublayout`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `checkSquareSublayout`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 24-30

```cpp
  24:   for (auto [b, basis] : llvm::enumerate(inDimBases)) {
  25:     if (!checkBasis(b, basis[0])) {
  26:       return false;
  27:     }
  28:   }
  29:   return true;
  30: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 32-36

```cpp
  32: bool squareSublayoutIsIdentity(const LinearLayout &ll,
  33:                                ArrayRef<StringAttr> dimNames) {
  34:   return checkSquareSublayout(
  35:       ll, dimNames, [](int b, int32_t basis) { return basis == (1 << b); });
  36: }
```

- **EN:** Defines `squareSublayoutIsIdentity`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `squareSublayoutIsIdentity`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 38-48

```cpp
  38: LinearLayout
  39: ensureLayoutNotLargerThan(const LinearLayout &layout,
  40:                           const llvm::SmallDenseMap<StringAttr, int64_t> &shape,
  41:                           bool broadcastRegisters) {
  42:   assert(shape.size() == layout.getNumOutDims());
  43:   if (shape.empty()) {
  44:     return layout;
  45:   }
  46:   MLIRContext *ctx = shape.begin()->first.getContext();
  47:   auto bases = layout.getBases();
  48:   auto kRegister = StringAttr::get(ctx, "register");
```

- **EN:** Defines `ensureLayoutNotLargerThan`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `ensureLayoutNotLargerThan`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 50-56

```cpp
  50:   auto outDims = layout.getOutDims();
  51:   for (auto &[outDim, outDimSize] : outDims) {
  52:     auto newOutDim = shape.lookup(outDim);
  53:     // Shape should be a non-zero power of 2
  54:     assert(llvm::isPowerOf2_32(newOutDim) && newOutDim != 0);
  55:     outDimSize = std::min<int32_t>(outDimSize, newOutDim);
  56:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 58-75

```cpp
  58:   // Ensure no base exceeds the resized out-dim sizes.
  59:   SmallVector<int32_t> outDimSizes;
  60:   for (auto &pair : outDims) {
  61:     outDimSizes.push_back(pair.second);
  62:   }
  63:   for (auto &[inDimName, inDimBases] : bases) {
  64:     bool dropBroadcasting = (!broadcastRegisters && inDimName == kRegister);
  65:     std::vector<std::vector<int32_t>> newBasesRegister;
  66:     for (auto &basis : inDimBases) {
  67:       bool wasZero = true;
  68:       bool isZero = true;
  69:       for (size_t i = 0; i < basis.size(); ++i) {
  70:         int32_t original = basis[i];
  71:         if (original != 0) {
  72:           wasZero = false;
  73:         }
  74:         if (original >= outDimSizes[i]) {
  75:           basis[i] = 0;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 76-90

```cpp
  76:         }
  77:         if (basis[i] != 0) {
  78:           isZero = false;
  79:         }
  80:       }
  81:       if (dropBroadcasting) {
  82:         if (wasZero || !isZero) {
  83:           newBasesRegister.push_back(std::move(basis));
  84:         }
  85:       }
  86:     }
  87:     if (dropBroadcasting) {
  88:       inDimBases = std::move(newBasesRegister);
  89:     }
  90:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 92-94

```cpp
  92:   return LinearLayout(std::move(bases), std::move(outDims),
  93:                       /*requireSurjective=*/false);
  94: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 96-109

```cpp
  96: // For each out-dim d, ensure the layout's out-size (i.e. its codomain) is no
  97: // smaller than shape[d].  Do this by increasing the size of the layout's inputs
  98: // along its most-minor dimension ("register" for register layouts, "offset" for
  99: // shared layouts).
 100: //
 101: // This function is invariant to the order of the layout's input dimensions, but
 102: // it cares about the order of the output dims, which should be minor-to-major.
 103: LinearLayout ensureLayoutNotSmallerThan(
 104:     const LinearLayout &layout,
 105:     const llvm::SmallDenseMap<StringAttr, int64_t> &shape) {
 106:   assert(shape.size() == layout.getNumOutDims());
 107:   if (shape.empty()) {
 108:     return layout;
 109:   }
```

- **EN:** Defines `ensureLayoutNotSmallerThan`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `ensureLayoutNotSmallerThan`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 111-112

```cpp
 111:   StringAttr kDim = *layout.getInDimNames().begin();
 112:   assert(kDim == "register" || kDim == "offset");
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 114-123

```cpp
 114:   LinearLayout ret = layout;
 115:   for (StringAttr outDimName : layout.getOutDimNames()) {
 116:     int32_t actualSize = layout.getOutDimSize(outDimName);
 117:     int32_t desiredSize = shape.lookup(outDimName);
 118:     assert(actualSize > desiredSize || desiredSize % actualSize == 0);
 119:     ret *= LinearLayout::identity1D(desiredSize / actualSize, kDim, outDimName);
 120:     assert(ret.getOutDimSize(outDimName) >= desiredSize);
 121:   }
 122:   return ret;
 123: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 125-132

```cpp
 125: // Returns ["dim0", "dim1", ..., "dim<rank-1>"].
 126: SmallVector<StringAttr> standardOutDimNames(MLIRContext *ctx, int rank) {
 127:   SmallVector<StringAttr> ret;
 128:   for (int i = 0; i < rank; i++) {
 129:     ret.push_back(StringAttr::get(ctx, "dim" + llvm::Twine(i)));
 130:   }
 131:   return ret;
 132: }
```

- **EN:** Defines `standardOutDimNames`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `standardOutDimNames`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 134-145

```cpp
 134: // Returns [("dim0", dstShape[0]), ("dim1", dstShape[1]), ...,
 135: // ("dim<rank-1>", dstShape[rank-1])].
 136: SmallVector<std::pair<StringAttr, int32_t>>
 137: standardOutDimPairs(MLIRContext *ctx, ArrayRef<int64_t> dstShape) {
 138:   auto newRank = dstShape.size();
 139:   SmallVector<std::pair<StringAttr, int32_t>> newOutDims;
 140:   for (auto [dim, size] :
 141:        llvm::zip(standardOutDimNames(ctx, newRank), dstShape)) {
 142:     newOutDims.emplace_back(dim, size);
 143:   }
 144:   return newOutDims;
 145: }
```

- **EN:** Defines `standardOutDimPairs`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `standardOutDimPairs`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 147-154

```cpp
 147: // Returns a 1D -> ND layout into [dim0, dim1, ...] that's equivalent to
 148: // creating a 1D -> 1D mapping of size product(shape) and then reshaping to
 149: // permute(shape, order).
 150: LinearLayout identityStandardND(StringAttr inDimName, ArrayRef<unsigned> shape,
 151:                                 ArrayRef<unsigned> order) {
 152:   assert(shape.size() == order.size());
 153:   MLIRContext *ctx = inDimName.getContext();
 154:   auto rank = shape.size();
```

- **EN:** Defines `identityStandardND`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `identityStandardND`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 156-157

```cpp
 156:   // The order in triton is written wrt. [dim0, dim1, ...].
 157:   SmallVector<StringAttr> outDimNames = standardOutDimNames(ctx, rank);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 159-166

```cpp
 159:   LinearLayout ret = LinearLayout::empty();
 160:   for (int i = 0; i < shape.size(); i++) {
 161:     // Start with the most-minor dimension, which is order[0].
 162:     int dim = order[i];
 163:     ret *= LinearLayout::identity1D(shape[dim], inDimName, outDimNames[dim]);
 164:   }
 165:   return ret;
 166: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 168-176

```cpp
 168: LinearLayout zerosLike(const LinearLayout &layout) {
 169:   auto bases = layout.getBases();
 170:   for (auto &basis : bases) {
 171:     for (auto &vec : basis.second) {
 172:       for (auto &val : vec) {
 173:         val = 0;
 174:       }
 175:     }
 176:   }
```

- **EN:** Defines `zerosLike`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `zerosLike`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 178-180

```cpp
 178:   return LinearLayout(std::move(bases), layout.getOutDims(),
 179:                       /*requireSurjective=*/false);
 180: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 182-190

```cpp
 182: std::optional<ColumnAction> regPermForDivide(const LinearLayout &A,
 183:                                              const LinearLayout &B, bool left) {
 184:   // We can implement this generically for any dimension, but for now we only do
 185:   // it for regs to keep the API simpler
 186:   assert(A.getNumInDims() != 0);
 187:   auto kReg = *A.getInDimNames().begin();
 188:   assert(kReg.str() == "register");
 189:   assert(B.getNumInDims() != 0);
 190:   assert(kReg == *B.getInDimNames().begin());
```

- **EN:** Defines `regPermForDivide`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `regPermForDivide`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 192-197

```cpp
 192:   // We broadcast B to have the same number of out dims as A.
 193:   LinearLayout broadcast;
 194:   for (StringAttr out : A.getOutDimNames()) {
 195:     broadcast *= LinearLayout::identity1D(1, kReg, out);
 196:   }
 197:   auto BBroadcast = broadcast * B;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 199-201

```cpp
 199:   // Retrieve the register bases from A and B.
 200:   const auto &ARegBases = A.getBases().lookup(kReg);
 201:   const auto &BRegBases = BBroadcast.getBases().lookup(kReg);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 203-209

```cpp
 203:   llvm::DenseMap<StringAttr, unsigned> log2QuotSize;
 204:   for (StringAttr out : A.getOutDimNames()) {
 205:     log2QuotSize[out] =
 206:         A.getOutDimSizeLog2(out) - BBroadcast.getOutDimSizeLog2(out);
 207:     if (log2QuotSize[out] < 0)
 208:       return std::nullopt;
 209:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 211-219

```cpp
 211:   auto multiplyByTileSize =
 212:       [&](ArrayRef<int32_t> bBasis) -> std::vector<int32_t> {
 213:     std::vector<int32_t> result;
 214:     assert(bBasis.size() == A.getNumOutDims());
 215:     for (auto [dim, b] : llvm::zip(A.getOutDimNames(), bBasis)) {
 216:       result.push_back(b << log2QuotSize.lookup(dim));
 217:     }
 218:     return result;
 219:   };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 221-230

```cpp
 221:   // Compute the permutation order:
 222:   // For each basis in B (in order), find its index in A (using each index at
 223:   // most once). We make sure we use each index at most once in case B
 224:   // broadcasts (weird case, but better safe than sorry).
 225:   SmallVector<size_t> bIndices;
 226:   SmallVector<bool> used(ARegBases.size(), false);
 227:   for (auto bB : BRegBases) {
 228:     bool found = false;
 229:     if (!left)
 230:       bB = multiplyByTileSize(bB);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 232-248

```cpp
 232:     for (size_t j = 0; j < ARegBases.size(); ++j) {
 233:       found = !used[j] && (ARegBases[j] == bB);
 234:       if (found) {
 235:         bIndices.push_back(j);
 236:         used[j] = true;
 237:         break;
 238:       }
 239:     }
 240:     if (!found)
 241:       return std::nullopt; // A basis from B not found in A.
 242:   }
 243:   // Append remaining indices from A (preserving their original order).
 244:   SmallVector<size_t> remainingIndices;
 245:   for (size_t i = 0; i < ARegBases.size(); ++i) {
 246:     if (!used[i])
 247:       remainingIndices.push_back(i);
 248:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 249-252

```cpp
 249:   SmallVector<size_t> permOrder = to_vector(llvm::concat<size_t>(
 250:       left ? bIndices : remainingIndices, left ? remainingIndices : bIndices));
 251:   return ColumnAction(permOrder, kReg, ARegBases.size());
 252: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 254-257

```cpp
 254: ColumnAction actionRemoveBroadcastedRegs(const LinearLayout &layout) {
 255:   assert(layout.getNumInDims() != 0);
 256:   auto kReg = *layout.getInDimNames().begin();
 257:   assert(kReg.str() == "register");
```

- **EN:** Defines `actionRemoveBroadcastedRegs`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `actionRemoveBroadcastedRegs`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 259-276

```cpp
 259:   // Drop the bases that are zero
 260:   const auto &bases = layout.getBases().lookup(kReg);
 261:   SmallVector<size_t> permOrder;
 262:   for (size_t i = 0; i < bases.size(); ++i) {
 263:     if (!llvm::all_of(bases[i], [](size_t x) { return x == 0; })) {
 264:       permOrder.push_back(i);
 265:     }
 266:   }
 267:   return ColumnAction(permOrder, kReg, bases.size());
 268: }
 269: std::pair<int64_t, ColumnAction>
 270: actionAdditiveStrides(const LinearLayout &layout, const LinearLayout addrLayout,
 271:                       uint64_t maskSpanOffsets, int64_t regsPerInst) {
 272:   // General idea:
 273:   // We want to swap an xor into an addition when computing the register
 274:   // offsets. We can do this if the output bits of this register are disjoint
 275:   // from those from lanes/warps/blocks or any affine offset (i.e.,
 276:   // maskSpanOffsets).
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 277-293

```cpp
 277:   //
 278:   // Additionally, the lowering only ever evaluates register offsets at indices
 279:   // that are multiples of `regsPerInst` (the inner-loop stride, matching one
 280:   // vectorised instruction). The first `log2(regsPerInst)` register bases are
 281:   // therefore never selected by any computed index and are trivially additive,
 282:   // independent of their basis value. We force them into the "additive" group
 283:   // unconditionally so that the returned `nAdditive` is always at least
 284:   // `regsPerInst`. In particular, callers do not need to pre-zero those bases
 285:   // for the invariant to hold.
 286:   assert(layout.getNumInDims() != 0);
 287:   assert(llvm::isPowerOf2_64(regsPerInst) &&
 288:          "regsPerInst must be a power of two");
 289:   auto kReg = *layout.getInDimNames().begin();
 290:   assert(kReg.str() == "register");
 291:   const size_t regBasisPerVec = llvm::Log2_64(regsPerInst);
 292:   uint32_t bits = maskSpanOffsets;
 293:   auto addrNamedBases = addrLayout.flattenOuts().getBases();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 294-310

```cpp
 294:   for (auto bases : llvm::make_second_range(addrNamedBases)) {
 295:     for (auto basis : bases) {
 296:       bits |= basis[0];
 297:     }
 298:   }
 299:   SmallVector<size_t> front, back;
 300:   auto layoutNamedBases = layout.flattenOuts().getBases();
 301:   assert(layoutNamedBases.lookup(kReg).size() >= regBasisPerVec &&
 302:          "layout must have at least log2(regsPerInst) register bases");
 303:   for (auto [idx, basis] : llvm::enumerate(layoutNamedBases.lookup(kReg))) {
 304:     if (idx < regBasisPerVec || (basis[0] & bits) == 0) {
 305:       front.push_back(idx);
 306:     } else {
 307:       back.push_back(idx);
 308:     }
 309:   }
 310:   auto permOrder = to_vector(llvm::concat<size_t>(front, back));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 311-313

```cpp
 311:   return {1 << front.size(),
 312:           ColumnAction(permOrder, kReg, layout.getInDimSizeLog2(kReg))};
 313: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 315-322

```cpp
 315: SmallVector<Value> broadcastAs(const SmallVector<Value> &values,
 316:                                const LinearLayout &layout) {
 317:   assert(layout.getNumInDims() != 0);
 318:   auto kReg = *layout.getInDimNames().begin();
 319:   assert(kReg.str() == "register");
 320:   uint32_t broadcastMask = layout.getFreeVariableMasks().lookup(kReg);
 321:   assert((layout.getInDimSize(kReg) / (1 << llvm::popcount(broadcastMask))) ==
 322:          values.size());
```

- **EN:** Defines `broadcastAs`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `broadcastAs`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 324-335

```cpp
 324:   std::vector<std::vector<int32_t>> newBases;
 325:   int i = 0;
 326:   for (int j = 0; j < layout.getInDimSizeLog2(kReg); j++) {
 327:     if (broadcastMask & (1 << j)) {
 328:       newBases.push_back({0});
 329:     } else {
 330:       newBases.push_back({1 << i});
 331:       i++;
 332:     }
 333:   }
 334:   auto newLayout = LinearLayout({{kReg, std::move(newBases)}}, {kReg});
 335:   SmallVector<Value> ret;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 337-343

```cpp
 337:   ret.reserve(newLayout.getInDimSize(kReg));
 338:   for (int i = 0; i < newLayout.getInDimSize(kReg); i++) {
 339:     int32_t srcIdx = newLayout.apply({{kReg, i}}).begin()->second;
 340:     ret.push_back(values[srcIdx]);
 341:   }
 342:   return ret;
 343: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 345-362

```cpp
 345: // Compute the supremum of two lists.
 346: // If the supremum is not unique, we return the first list first
 347: // Error out if the supremum does not exist
 348: // e.g. sup([a, b], [a, c]) = [a, b, c], sup([a, b], [b, c]) = [a, b, c]
 349: //      sup([a, b], [b, a]) = error! Supremum does not exist.
 350: SmallVector<StringAttr> supremum(const SmallVector<StringAttr> &x,
 351:                                  const SmallVector<StringAttr> &y) {
 352:   llvm::SetVector<StringAttr> result;
 353:   DenseMap<StringAttr, int> posX, posY;
 354:   for (auto [idx, elem] : llvm::enumerate(x))
 355:     posX[elem] = idx;
 356:   for (auto [idx, elem] : llvm::enumerate(y))
 357:     posY[elem] = idx;
 358:   int i = 0, j = 0;
 359:   const int INF = std::numeric_limits<int>::max();
 360:   while (i < x.size() || j < y.size()) {
 361:     while (i < x.size() && result.contains(x[i]))
 362:       ++i;
```

- **EN:** Defines `supremum`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `supremum`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 363-378

```cpp
 363:     while (j < y.size() && result.contains(y[j]))
 364:       ++j;
 365:     if (i >= x.size() && j >= y.size())
 366:       break;
 367:     if (i < x.size() && j < y.size() && x[i] == y[j]) {
 368:       if (posY[x[i]] < j)
 369:         llvm_unreachable("Supremum does not exist");
 370:       result.insert(x[i]);
 371:       ++i, ++j;
 372:       continue;
 373:     }
 374:     int candX = INF, candY = INF;
 375:     if (i < x.size()) {
 376:       if (posY.count(x[i]) && posY[x[i]] >= j)
 377:         candX = posY[x[i]];
 378:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 379-396

```cpp
 379:     if (j < y.size()) {
 380:       if (posX.count(y[j]) && posX[y[j]] >= i)
 381:         candY = posX[y[j]];
 382:     }
 383:     if (i < x.size() && candX == INF) {
 384:       result.insert(x[i]);
 385:       ++i;
 386:       continue;
 387:     }
 388:     if (j < y.size() && candY == INF) {
 389:       result.insert(y[j]);
 390:       ++j;
 391:       continue;
 392:     }
 393:     if (candX <= candY) {
 394:       if (posY[x[i]] < j)
 395:         llvm_unreachable("Supremum does not exist");
 396:       result.insert(x[i]);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 397-406

```cpp
 397:       ++i;
 398:     } else {
 399:       if (posX[y[j]] < i)
 400:         llvm_unreachable("Supremum does not exist");
 401:       result.insert(y[j]);
 402:       ++j;
 403:     }
 404:   }
 405:   return to_vector(result);
 406: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 408-418

```cpp
 408: LinearLayout reshapeLayout(MLIRContext *ctx, LinearLayout layout,
 409:                            ArrayRef<int64_t> shape) {
 410:   int rank = shape.size();
 411:   auto srcOutDims = to_vector(layout.getOutDimNames());
 412:   std::reverse(srcOutDims.begin(), srcOutDims.end());
 413:   auto newOutDims = standardOutDimPairs(ctx, shape);
 414:   std::reverse(newOutDims.begin(), newOutDims.end());
 415:   return layout.transposeOuts(srcOutDims)
 416:       .reshapeOuts(newOutDims)
 417:       .transposeOuts(standardOutDimNames(ctx, rank));
 418: }
```

- **EN:** Defines `reshapeLayout`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `reshapeLayout`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 420-423

```cpp
 420: LinearLayout transposeLinearLayout(LinearLayout layout, ArrayRef<int> order) {
 421:   // Transpose the tile layout.
 422:   auto namedBases = layout.getBases();
 423:   // move the most outer dimensions to the inner most position.
```

- **EN:** Defines `transposeLinearLayout`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `transposeLinearLayout`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 425-436

```cpp
 425:   for (auto &bases : llvm::make_second_range(namedBases)) {
 426:     for (auto &b : bases) {
 427:       std::vector<int32_t> newB;
 428:       for (auto i : order) {
 429:         newB.push_back(b[i]);
 430:       }
 431:       b = std::move(newB);
 432:     }
 433:   }
 434:   return LinearLayout(std::move(namedBases),
 435:                       to_vector(layout.getOutDimNames()));
 436: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 438-454

```cpp
 438: std::pair<int, ColumnAction>
 439: largestVectorisation(MLIRContext *ctx, const LinearLayout &cvt, int bitwidth,
 440:                      std::optional<int> maybeMaxVecElems) {
 441:   // Find the largest vectorisation we can use:
 442:   auto S = [ctx](StringRef str) { return StringAttr::get(ctx, str); };
 443:   StringAttr kReg = S("register");
 444:   StringAttr kOffset = S("offset");
 445:   LinearLayout quot;
 446:   LinearLayout tile;
 447:   ColumnAction permutation;
 448:   // If there are restrictions on the vectorisation, we don't allow
 449:   // permutations.
 450:   auto allowPerm = !maybeMaxVecElems.has_value();
 451:   auto maxVecElems = maybeMaxVecElems.value_or(128 / bitwidth);
 452:   for (int v = maxVecElems; v >= 1; v /= 2) {
 453:     tile = LinearLayout::identity1D(v, kReg, kOffset);
 454:     auto maybePerm = regPermForDivide(cvt, tile, /*left=*/true);
```

- **EN:** Defines `largestVectorisation`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `largestVectorisation`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 455-470

```cpp
 455:     if (!maybePerm) {
 456:       continue;
 457:     }
 458:     permutation = *maybePerm;
 459:     if (!allowPerm && !permutation.isIdentity()) {
 460:       continue;
 461:     }
 462:     auto newCvt = permutation.apply(cvt);
 463:     auto maybeQuot = divideLeft(newCvt, tile);
 464:     if (!maybeQuot) {
 465:       continue;
 466:     }
 467:     return {v, permutation};
 468:   }
 469:   llvm_unreachable("Vectorization < 1 is not valid");
 470: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 472-473

```cpp
 472: std::optional<LinearLayout> getReps(const LinearLayout &cvt,
 473:                                     const LinearLayout &tile) {
```

- **EN:** Defines accessor/helper `getReps` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getReps`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 475-477

```cpp
 475:   // Ensure tile out-dims are subset of cvt out-dims.
 476:   for (auto od : tile.getOutDimNames())
 477:     assert(cvt.hasOutDim(od) && "tile out-dims must be contained in cvt");
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 479-482

```cpp
 479:   // Precompute tile out-dim bit-widths.
 480:   llvm::SmallDenseMap<StringAttr, int> outBLog2;
 481:   for (StringAttr od : cvt.getOutDimNames())
 482:     outBLog2[od] = tile.hasOutDim(od) ? tile.getOutDimSizeLog2(od) : 0;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 484-496

```cpp
 484:   // Build a per-out-dimension mask by OR-ing all tile bases that touch it.
 485:   llvm::SmallDenseMap<StringAttr, int32_t> tileMaskPerOutDim;
 486:   for (StringAttr od : cvt.getOutDimNames())
 487:     tileMaskPerOutDim[od] = 0;
 488:   for (auto &[inDim, inBases] : tile.getBases()) {
 489:     (void)inDim;
 490:     for (auto &basis : inBases) {
 491:       int idx = 0;
 492:       for (StringAttr od : tile.getOutDimNames()) {
 493:         tileMaskPerOutDim[od] |= basis[idx++];
 494:       }
 495:     }
 496:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 498-506

```cpp
 498:   // Build reps with the same in/out dims as cvt, but zeroing out the leading
 499:   // inB bases (per in-dim) and keeping the remainder bases unchanged from cvt.
 500:   LinearLayout::BasesT repsBases;
 501:   for (StringAttr id : cvt.getInDimNames()) {
 502:     int inA = cvt.getInDimSizeLog2(id);
 503:     int inB = tile.hasInDim(id) ? tile.getInDimSizeLog2(id) : 0;
 504:     if (inB > inA) {
 505:       return std::nullopt;
 506:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 508-509

```cpp
 508:     std::vector<std::vector<int32_t>> basesForDim;
 509:     basesForDim.reserve(inA);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 511-520

```cpp
 511:     // 1) Validate the starting bases match exactly.
 512:     for (int i = 0; i < inB; ++i) {
 513:       for (StringAttr od : cvt.getOutDimNames()) {
 514:         int a = cvt.getBasis(id, i, od);
 515:         int b = tile.getBasis(id, i, od);
 516:         if (a != b) {
 517:           return std::nullopt;
 518:         }
 519:       }
 520:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 522-535

```cpp
 522:     // 2) Validate no overlap: the remaining cvt bases must have zeros in all
 523:     //    tile-bit positions (computed as OR of all tile bases) for each
 524:     //    out-dim.
 525:     for (int i = inB; i < inA; ++i) {
 526:       for (StringAttr od : cvt.getOutDimNames()) {
 527:         int32_t mask = tileMaskPerOutDim.lookup(od);
 528:         if (mask == 0)
 529:           continue;
 530:         int v = cvt.getBasis(id, i, od);
 531:         if ((v & mask) != 0) {
 532:           return std::nullopt;
 533:         }
 534:       }
 535:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 537-548

```cpp
 537:     // 3) Emit reps bases: first inB as all-zeros; remainder copied from cvt.
 538:     for (int i = 0; i < inB; ++i) {
 539:       std::vector<int32_t> zero(cvt.getNumOutDims(), 0);
 540:       basesForDim.push_back(std::move(zero));
 541:     }
 542:     for (int i = inB; i < inA; ++i) {
 543:       std::vector<int32_t> keep;
 544:       keep.reserve(cvt.getNumOutDims());
 545:       for (StringAttr od : cvt.getOutDimNames())
 546:         keep.push_back(cvt.getBasis(id, i, od));
 547:       basesForDim.push_back(std::move(keep));
 548:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 550-551

```cpp
 550:     repsBases[id] = std::move(basesForDim);
 551:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 553-555

```cpp
 553:   return LinearLayout(std::move(repsBases), cvt.getOutDims(),
 554:                       /*requireSurjective=*/false);
 555: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 557-572

```cpp
 557: LinearLayout removeStandardDim(const LinearLayout &layout, int dim) {
 558:   auto rank = layout.getNumOutDims();
 559:   assert(rank > 0);
 560:   assert(dim < rank);
 561:   auto *ctx = layout.getOutDimNames().begin()->getContext();
 562:   auto dims = to_vector(layout.getOutDimNames());
 563:   assert(dims == standardOutDimNames(ctx, rank));
 564:   dims.erase(dims.begin() + dim);
 565:   auto newLayout = layout.sublayout(to_vector(layout.getInDimNames()), dims);
 566:   auto dimSizes = newLayout.getOutDims();
 567:   auto newDims = standardOutDimNames(ctx, rank - 1);
 568:   for (auto [i, newDim] : llvm::enumerate(newDims)) {
 569:     dimSizes[i].first = newDim;
 570:   }
 571:   return LinearLayout(newLayout.getBases(), dimSizes, /*isSurjective*/ false);
 572: }
```

- **EN:** Defines `removeStandardDim`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `removeStandardDim`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 574-574

```cpp
 574: } // namespace mlir::triton
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。

## Key Concepts / 关键概念
- **EN:** The file packages reusable support around layout utils.
  **CN:** 该文件围绕 Layout Utils 封装了可复用的支持逻辑。
- **EN:** Linear layout utilities translate between logical tensor coordinates and physical placement.
  **CN:** 线性布局工具负责在逻辑张量坐标与物理放置之间进行转换。
- **EN:** Layout and encoding decisions are first-class because they determine how work and memory are distributed on the GPU.
  **CN:** 布局与编码决策是第一类问题，因为它们决定了 GPU 上工作与内存的分布方式。

## Dependencies / 依赖关系
- **Internal Triton headers / Triton 内部头文件:** `triton/Tools/LayoutUtils.h`, `triton/Tools/GenericSwizzling.h`
- **MLIR headers / MLIR 头文件:** None
- **LLVM headers / LLVM 头文件:** None
- **Standard/library headers / 标准或通用库头文件:** None
- **Primary APIs used / 主要 API:** `LinearLayout`
