# GenericSwizzling.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `lib/Tools/GenericSwizzling.cpp`
- **Purpose / 作用:** **EN:** Provides reusable utility code for Generic Swizzling used across the Triton implementation. **CN:** 提供与 Generic Swizzling 相关的可复用工具代码，供 Triton 各模块共享。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1

```cpp
   1: #include "triton/Tools/GenericSwizzling.h"
```

- **EN:** Includes the interfaces this file depends on. Triton headers (`GenericSwizzling.h`) provide domain-specific IR/support, MLIR headers (None) provide rewriting and analysis infrastructure, LLVM headers (None) supply low-level utilities, and standard/library headers (None) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（`GenericSwizzling.h`）提供领域特定 IR/支持逻辑，MLIR 头文件（None）提供重写与分析基础设施，LLVM 头文件（None）提供底层工具，而标准/通用库头文件（None）提供通用能力。
### Lines 3-10

```cpp
   3: #include "third_party/f2reduce/f2reduce.h"
   4: #include "triton/Tools/LayoutUtils.h"
   5: #include "triton/Tools/LinearLayout.h"
   6: #include "llvm/ADT/DenseSet.h"
   7: #include "llvm/ADT/STLExtras.h"
   8: #include "llvm/ADT/SmallVector.h"
   9: #include "llvm/Support/Debug.h"
  10: #include "llvm/Support/MathExtras.h"
```

- **EN:** Includes the interfaces this file depends on. Triton headers (`LayoutUtils.h`, `LinearLayout.h`) provide domain-specific IR/support, MLIR headers (None) provide rewriting and analysis infrastructure, LLVM headers (`DenseSet.h`, `STLExtras.h`, `SmallVector.h`, `Debug.h`, ... (+1 more)) supply low-level utilities, and standard/library headers (`third_party/f2reduce/f2reduce.h`) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（`LayoutUtils.h`, `LinearLayout.h`）提供领域特定 IR/支持逻辑，MLIR 头文件（None）提供重写与分析基础设施，LLVM 头文件（`DenseSet.h`, `STLExtras.h`, `SmallVector.h`, `Debug.h`, ... (+1 more)）提供底层工具，而标准/通用库头文件（`third_party/f2reduce/f2reduce.h`）提供通用能力。
### Lines 12-13

```cpp
  12: #define DEBUG_TYPE "generic-swizzling"
  13: #define DBGS() (llvm::dbgs() << "[" DEBUG_TYPE "]: ")
```

- **EN:** Defines debug logging helpers and compile-time tags used when tracing this pass or utility at runtime.
- **CN:** 这里定义调试日志辅助宏和编译期标签，用于在运行时跟踪该 pass 或工具。
### Lines 15-17

```cpp
  15: #if defined(_MSC_VER) && !defined(__clang__)
  16: // from https://gist.github.com/pps83/3210a2f980fd02bb2ba2e5a1fc4a2ef0
  17: #include <intrin.h>
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 19-23

```cpp
  19: static int __builtin_ctzll(unsigned long long x) {
  20:   unsigned long r;
  21:   _BitScanForward64(&r, x);
  22:   return static_cast<int>(r);
  23: }
```

- **EN:** Defines `__builtin_ctzll`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `__builtin_ctzll`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 25-25

```cpp
  25: #endif
```

- **EN:** Defines or closes a file guard so this header/TableGen fragment is included only once during compilation.
- **CN:** 这里定义或关闭文件保护宏，确保该头文件/TableGen 片段在编译过程中只被包含一次。
### Lines 27-28

```cpp
  27: using namespace mlir;
  28: using namespace mlir::triton;
```

- **EN:** Introduces namespace aliases/imports (`mlir`, `mlir::triton`) so the rest of the file can use MLIR/Triton symbols without repetitive qualification.
- **CN:** 这里通过命名空间导入（`mlir`, `mlir::triton`）减少后续代码中的重复限定，使 MLIR/Triton 符号使用更简洁。
### Lines 30-30

```cpp
  30: namespace {
```

- **EN:** Opens or closes the namespace nesting for (anonymous), keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 (anonymous) 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 32-38

```cpp
  32: void printBasis(const llvm::SmallVector<int32_t> &basis,
  33:                 const std::string &name) {
  34:   llvm::errs() << name << ": ";
  35:   for (int32_t b : basis)
  36:     llvm::errs() << b << " ";
  37:   llvm::errs() << "\n";
  38: }
```

- **EN:** Defines `printBasis`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `printBasis`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 40-48

```cpp
  40: // Goes from bases of the form [[1], [2], [4], [8]] to [1, 2, 4, 8]
  41: SmallVector<int32_t> flatten(const LinearLayout &ll, StringAttr dim) {
  42:   assert(ll.getNumOutDims() == 1);
  43:   auto outDim = *ll.getOutDimNames().begin();
  44:   SmallVector<int32_t> vec;
  45:   for (int i = 0; i < ll.getInDimSizeLog2(dim); ++i)
  46:     vec.push_back(ll.getBasis(dim, i, outDim));
  47:   return vec;
  48: };
```

- **EN:** Defines `flatten`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `flatten`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 50-58

```cpp
  50: SmallVector<int32_t> removeZeros(ArrayRef<int32_t> vec) {
  51:   SmallVector<int32_t> result;
  52:   for (int32_t r : vec) {
  53:     if (r != 0) {
  54:       result.push_back(r);
  55:     }
  56:   }
  57:   return result;
  58: }
```

- **EN:** Defines `removeZeros`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `removeZeros`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 60-66

```cpp
  60: // [1, 2, 4, 8] -> [[1], [2], [4], [8]]
  61: std::vector<std::vector<int32_t>> unflatten(ArrayRef<int32_t> basis) {
  62:   std::vector<std::vector<int32_t>> unflattened;
  63:   for (int32_t b : basis)
  64:     unflattened.push_back({b});
  65:   return unflattened;
  66: }
```

- **EN:** Defines `unflatten`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `unflatten`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 68-77

```cpp
  68: // Compute the nullspace basis of `vectors`
  69: SmallVector<int32_t> nullspaceBasis(ArrayRef<int32_t> vectors, int32_t dim) {
  70:   // Solve A^T x = 0, where A is the matrix of vectors
  71:   // To do this, we form a matrix where each vector is a row
  72:   const int32_t nRows = vectors.size();
  73:   auto mat = std::make_unique<uint64_t[]>(nRows);
  74:   for (int i = 0; i < nRows; ++i)
  75:     mat[i] = static_cast<uint64_t>(vectors[i]);
  76:   f2reduce::inplace_rref_strided(mat.get(), /*rows=*/nRows, /*cols=*/dim,
  77:                                  /*stride=*/1);
```

- **EN:** Defines `nullspaceBasis`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `nullspaceBasis`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 79-82

```cpp
  79:   llvm::SmallDenseSet<int32_t> pivotCols;
  80:   for (int32_t r = 0; r < nRows; ++r)
  81:     if (mat[r])
  82:       pivotCols.insert(__builtin_ctzll(mat[r]));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 84-97

```cpp
  84:   SmallVector<int32_t> basis;
  85:   for (int32_t freeCol = 0; freeCol < dim; ++freeCol) {
  86:     if (!pivotCols.contains(freeCol)) {
  87:       uint64_t vec = 1ull << freeCol;
  88:       for (int32_t r = 0; r < nRows; ++r)
  89:         if (mat[r] & (1ull << freeCol)) {
  90:           const int32_t pivot = __builtin_ctzll(mat[r]);
  91:           vec ^= (1ull << pivot);
  92:         }
  93:       basis.push_back(static_cast<int32_t>(vec));
  94:     }
  95:   }
  96:   return basis;
  97: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 99-116

```cpp
  99: // Find the smallest tile that we can read and write to smem
 100: // without sacrificing vectorisation and split it into its own
 101: // `reps` dimension
 102: LinearLayout buildReps(MLIRContext *ctx, const LinearLayout &src,
 103:                        const LinearLayout &dst, const LinearLayout &smem,
 104:                        int32_t leaveReps) {
 105:   auto kVec = StringAttr::get(ctx, "vector");
 106:   auto kBank = StringAttr::get(ctx, "bank");
 107:   auto kSegment = StringAttr::get(ctx, "segment");
 108:   auto kReps = StringAttr::get(ctx, "reps");
 109:   auto kBlock = StringAttr::get(ctx, "block");
 110:   auto kReg = StringAttr::get(ctx, "register");
 111:   // A basis is a rep if:
 112:   // 1) It is in registers in both src and dst
 113:   // 2) It is in the segment of smem (i.e., is not part of just one
 114:   //    load/store)
 115:   SetVector<int32_t> srcRegs(llvm::from_range_t{}, flatten(src, kReg));
 116:   SetVector<int32_t> dstRegs(llvm::from_range_t{}, flatten(dst, kReg));
```

- **EN:** Defines helper `buildReps` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `buildReps`，用于计算或构造外围变换所需的中间数据。
### Lines 117-134

```cpp
 117:   SetVector<int32_t> smemSegment(llvm::from_range_t{}, flatten(smem, kSegment));
 118:   SetVector<int32_t> segment;
 119:   SetVector<int32_t> reps;
 120:   for (auto s : smemSegment) {
 121:     // Do not move the first leaveReps bases from reps to segment
 122:     // as we need them to vectorise the instructions (think .x2 and .x4 in
 123:     // ldmatrix)
 124:     if (srcRegs.contains(s) && dstRegs.contains(s)) {
 125:       if (leaveReps > 0) {
 126:         leaveReps--;
 127:         segment.insert(s);
 128:       } else {
 129:         reps.insert(s);
 130:       }
 131:     } else {
 132:       segment.insert(s);
 133:     }
 134:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 136-144

```cpp
 136:   auto smemReps = LinearLayout({{kVec, smem.getBases().lookup(kVec)},
 137:                                 {kBank, smem.getBases().lookup(kBank)},
 138:                                 {kSegment, unflatten(to_vector(segment))},
 139:                                 {kBlock, smem.getBases().lookup(kBlock)},
 140:                                 {kReps, unflatten(to_vector(reps))}},
 141:                                smem.getOutDims(),
 142:                                /*requireSurjective=*/true);
 143:   return smemReps;
 144: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 146-156

```cpp
 146: SmallVector<int32_t> computeSegment(ArrayRef<int32_t> bankSrc,
 147:                                     ArrayRef<int32_t> bankDst,
 148:                                     ArrayRef<int32_t> blockBases, int32_t dim,
 149:                                     int32_t lenSegment) {
 150:   llvm::SmallDenseSet<int32_t> setSrc(bankSrc.begin(), bankSrc.end());
 151:   llvm::SmallDenseSet<int32_t> setDst(bankDst.begin(), bankDst.end());
 152:   setSrc.insert(blockBases.begin(), blockBases.end());
 153:   setDst.insert(blockBases.begin(), blockBases.end());
 154:   // Remove the 0 as it's not a basis
 155:   setSrc.erase(0);
 156:   setDst.erase(0);
```

- **EN:** Defines helper `computeSegment` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `computeSegment`，用于计算或构造外围变换所需的中间数据。
### Lines 158-165

```cpp
 158:   SmallVector<int32_t> segment;
 159:   for (int32_t b = 0; b < dim; ++b)
 160:     if (!setSrc.contains(1 << b) && !setDst.contains(1 << b))
 161:       segment.push_back(1 << b);
 162:   if (segment.size() >= lenSegment) {
 163:     segment.resize(lenSegment);
 164:     return segment;
 165:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 167-183

```cpp
 167:   // A and B are the difference sets
 168:   SmallVector<int32_t> A, B;
 169:   for (int32_t v : setSrc)
 170:     if (!setDst.contains(v))
 171:       A.push_back(v);
 172:   for (int32_t v : setDst)
 173:     if (!setSrc.contains(v))
 174:       B.push_back(v);
 175:   if (A.size() > B.size()) {
 176:     std::swap(A, B);
 177:   }
 178:   llvm::sort(A);
 179:   llvm::sort(B);
 180:   // A is the smaller set now
 181:   auto logBankConflicts = std::min<int32_t>(
 182:       std::max<int32_t>(0, lenSegment - A.size() - segment.size()), A.size());
 183:   // Conflict-free
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 184-189

```cpp
 184:   for (int i = logBankConflicts; i < A.size(); ++i)
 185:     segment.push_back(A[i] ^ B[i]);
 186:   // Write conflicts
 187:   segment.append(A.begin(), A.begin() + logBankConflicts);
 188:   // Read conflicts
 189:   segment.append(B.begin(), B.begin() + logBankConflicts);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 191-194

```cpp
 191:   if (segment.size() > lenSegment)
 192:     segment.resize(lenSegment);
 193:   return segment;
 194: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 196-200

```cpp
 196: SmallVector<int32_t> complementBasis(ArrayRef<int32_t> basis, int32_t dim) {
 197:   const int32_t nRows = basis.size();
 198:   auto mat = std::make_unique<uint64_t[]>(nRows);
 199:   for (int r = 0; r < nRows; ++r)
 200:     mat[r] = static_cast<uint64_t>(basis[r]);
```

- **EN:** Defines `complementBasis`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `complementBasis`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 202-203

```cpp
 202:   f2reduce::inplace_rref_strided(mat.get(), /*rows=*/nRows,
 203:                                  /*cols=*/dim, /*stride=*/1);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 205-210

```cpp
 205:   llvm::SmallDenseSet<int32_t> pivotCols;
 206:   for (int r = 0; r < nRows; ++r) {
 207:     if (mat[r]) {
 208:       pivotCols.insert(__builtin_ctzll(mat[r])); // leading-1 position
 209:     }
 210:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 212-215

```cpp
 212:   SmallVector<int32_t> comp;
 213:   for (int i = 0; i < dim; ++i)
 214:     if (!pivotCols.contains(i))
 215:       comp.push_back(1 << i);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 217-218

```cpp
 217:   return comp;
 218: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 220-236

```cpp
 220: SmallVector<int32_t> intersectionBasis(ArrayRef<int32_t> b1,
 221:                                        ArrayRef<int32_t> b2, int32_t dim) {
 222:   // If needed to be generic, this can be done computing
 223:   // nullspaceBasis(concat(nullspaceBasis(b1), nullspaceBasis(b2)))
 224:   // but doing this returns the bases in an arbitrary order!
 225:   auto isPowerOf2 = [](int32_t x) { return llvm::isPowerOf2_32(x); };
 226:   bool powerOf2 = llvm::all_of(b1, isPowerOf2) && llvm::all_of(b2, isPowerOf2);
 227:   if (powerOf2) {
 228:     SmallVector<int32_t> result;
 229:     // Heuristic: We choose to retain the order relative to b1
 230:     SetVector<int32_t> set2(b2.begin(), b2.end());
 231:     for (int32_t b : b1) {
 232:       if (b != 0 && set2.contains(b)) {
 233:         result.push_back(b);
 234:       }
 235:     }
 236:     return result;
```

- **EN:** Defines `intersectionBasis`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `intersectionBasis`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 237-243

```cpp
 237:   } else {
 238:     auto ns1 = nullspaceBasis(b1, dim);
 239:     auto ns2 = nullspaceBasis(b2, dim);
 240:     auto joint = llvm::to_vector(llvm::concat<int32_t>(ns1, ns2));
 241:     return nullspaceBasis(joint, dim);
 242:   }
 243: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 245-245

```cpp
 245: } // namespace
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 247-247

```cpp
 247: namespace mlir::triton::gpu {
```

- **EN:** Opens or closes the namespace nesting for mlir::triton::gpu, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 mlir::triton::gpu 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 249-259

```cpp
 249: std::pair<int, int> bankConflicts(ArrayRef<int32_t> tileSrc,
 250:                                   ArrayRef<int32_t> tileDst,
 251:                                   const LinearLayout &smem) {
 252:   auto *ctx = smem.getOutDimNames().begin()->getContext();
 253:   auto smemFlat = smem.flattenOuts();
 254:   // Look at the intersection between the segment bases and the tile bases
 255:   // We don't need to intersect with the bases that covert the bank (as in
 256:   // the first 32 / bitwidth bases) because if we hit any of those broadcasting
 257:   // will avoid the bank conflict
 258:   auto segment = StringAttr::get(ctx, "segment");
 259:   auto segmentBases = flatten(smemFlat, segment);
```

- **EN:** Defines `bankConflicts`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `bankConflicts`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 261-266

```cpp
 261:   int32_t rank = smem.getTotalOutDimSizeLog2();
 262:   // compute conflicts
 263:   int write = 1 << intersectionBasis(segmentBases, tileSrc, rank).size();
 264:   int read = 1 << intersectionBasis(segmentBases, tileDst, rank).size();
 265:   return {read - 1, write - 1};
 266: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 268-275

```cpp
 268: SmallVector<int32_t> LocalMemOpTile::getLaneAddr(ArrayRef<int32_t> lane) const {
 269:   SmallVector<int32_t> ret;
 270:   ret.reserve(laneAddr.size());
 271:   for (int32_t idx : laneAddr) {
 272:     ret.push_back(lane[idx]);
 273:   }
 274:   return ret;
 275: }
```

- **EN:** Defines accessor/helper `LocalMemOpTile::getLaneAddr` that exposes or updates operation state in a compact, reusable way. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义访问器/辅助函数 `LocalMemOpTile::getLaneAddr`，以紧凑且可复用的方式读取或更新操作状态。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 277-293

```cpp
 277: SmallVector<int32_t> getLaneTile(const LocalMemOpTile &tile,
 278:                                  ArrayRef<int32_t> lane, int32_t vecSize,
 279:                                  int32_t bitwidth, int32_t numBanks) {
 280:   auto log2Vec = llvm::Log2_32(std::max<int32_t>(1, (vecSize * bitwidth) / 32));
 281:   auto log2Bank = llvm::Log2_32(numBanks);
 282:   auto log2Phase = std::max<int32_t>(0, log2Vec + lane.size() - log2Bank);
 283:   SmallVector<int32_t> res;
 284:   if (!tile.laneAddr.empty()) {
 285:     // The laneAddr field explicitly defines the lane basis indices for
 286:     // load/store instructions with non-sequential lane IDs within a single
 287:     // phase, like ds_read_b128.
 288:     res = tile.getLaneAddr(lane);
 289:   } else {
 290:     // If laneAddr is empty, we fall back to the standard assumption for
 291:     // regular loads/stores: lane IDs are sequential within a single phase.
 292:     res = to_vector(lane.drop_back(log2Phase));
 293:   }
```

- **EN:** Defines accessor/helper `getLaneTile` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getLaneTile`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 294-297

```cpp
 294:   assert(res.size() == lane.size() - log2Phase &&
 295:          "Both paths should return the expected number of elements");
 296:   return res;
 297: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 299-314

```cpp
 299: std::pair<int, int>
 300: bankConflictsLdSt(const LinearLayout &src, const LinearLayout &dst,
 301:                   const LinearLayout &smem, int32_t bitwidth, int32_t numBanks,
 302:                   LocalMemOpTile srcTile, LocalMemOpTile dstTile) {
 303:   auto srcFlat = src.flattenOuts();
 304:   auto dstFlat = dst.flattenOuts();
 305:   auto *ctx = smem.getOutDimNames().begin()->getContext();
 306:   auto S = [ctx](StringRef str) { return StringAttr::get(ctx, str); };
 307:   auto kVec = S("vector");
 308:   auto srcLane = flatten(srcFlat, S("lane"));
 309:   auto dstLane = flatten(dstFlat, S("lane"));
 310:   auto vecSize = smem.getInDimSize(kVec);
 311:   auto tileSrc = getLaneTile(srcTile, srcLane, vecSize, bitwidth, numBanks);
 312:   auto tileDst = getLaneTile(dstTile, dstLane, vecSize, bitwidth, numBanks);
 313:   return bankConflicts(tileSrc, tileDst, smem);
 314: }
```

- **EN:** Defines `bankConflictsLdSt`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `bankConflictsLdSt`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 316-320

```cpp
 316: int bankConflictsMemDesc(const LinearLayout &reg, const LinearLayout &smem,
 317:                          int32_t bitwidth, int32_t numBanks,
 318:                          LocalMemOpTile laneTile) {
 319:   auto *ctx = smem.getInDimNames().begin()->getContext();
 320:   auto S = [ctx](StringRef str) { return StringAttr::get(ctx, str); };
```

- **EN:** Defines `bankConflictsMemDesc`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `bankConflictsMemDesc`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 322-329

```cpp
 322:   assert(smem.hasInDim(S("offset")) && "shared layout must have an offset dim");
 323:   assert(reg.hasInDim(S("register")) &&
 324:          "register layout must have a register dim");
 325:   auto regNoBroadcast = actionRemoveBroadcastedRegs(reg).apply(reg);
 326:   auto regToShared = regNoBroadcast.invertAndCompose(smem);
 327:   auto [elemsPerVec, permutation] =
 328:       largestVectorisation(ctx, regToShared, bitwidth);
 329:   regNoBroadcast = permutation.apply(regNoBroadcast);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 331-344

```cpp
 331:   int32_t vecSize = elemsPerVec;
 332:   int32_t bankSize =
 333:       std::min(numBanks * 32 / (vecSize * bitwidth), smem.getTotalInDimSize());
 334:   int32_t segmentSize = smem.getTotalInDimSize() / (bankSize * vecSize);
 335:   SmallVector<std::pair<StringAttr, int32_t>> newInDims = {
 336:       {S("vector"), vecSize},
 337:       {S("bank"), bankSize},
 338:       {S("segment"), segmentSize},
 339:   };
 340:   auto smemReshaped = smem.reshapeIns(newInDims);
 341:   return bankConflictsLdSt(regNoBroadcast, regNoBroadcast, smemReshaped,
 342:                            bitwidth, numBanks, laneTile, laneTile)
 343:       .first;
 344: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 346-356

```cpp
 346: static std::optional<SmallVector<int32_t>> optimalSwizzlingTile(
 347:     const LinearLayout &a, const LinearLayout &b, int32_t nRegA, int32_t nRegB,
 348:     ArrayRef<int32_t> laneIdTileA, ArrayRef<int32_t> laneIdTileB) {
 349:   // For now se just implement the .v4 variants for all the instructions
 350:   // We could generalise this in the future
 351:   assert(nRegA + laneIdTileA.size() == nRegB + laneIdTileB.size());
 352:   // normalise nRegA >= nRegB
 353:   if (nRegA < nRegB) {
 354:     return optimalSwizzlingTile(b, a, nRegB, nRegA, laneIdTileB, laneIdTileA);
 355:   }
 356:   assert(nRegA >= nRegB);
```

- **EN:** Defines `optimalSwizzlingTile`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `optimalSwizzlingTile`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 358-363

```cpp
 358:   auto *ctx = a.getInDimNames().begin()->getContext();
 359:   auto kReg = StringAttr::get(ctx, "register");
 360:   auto kLane = StringAttr::get(ctx, "lane");
 361:   auto dim = a.getTotalOutDimSizeLog2();
 362:   // map from b to a
 363:   LinearLayout cvt = b.invertAndCompose(a);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 365-371

```cpp
 365:   // The contiguous tile of ld.shared.b32.v4 for a packed element of size
 366:   // bitwidth is composed of 128/bitwidth register elements
 367:   // The contiguous tile of ldmatrix.v4 for a packed element of size bitwidth
 368:   // is composed of 32/bitwidth register elements and the bases 0, 1st as given
 369:   // by the laneAddr
 370:   // The contiguous tile of ldmatrix.v4.trans for a packed element of size 16
 371:   // is composed of the bases 2, 3, 4th as given by the laneAddr
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 373-374

```cpp
 373:   // Note that for register elements, we can choose any register basis we want,
 374:   // but the lane bases are fixed
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 376-377

```cpp
 376:   // In this function, we compute a tile (set of bases) such that it matches
 377:   // the tiles of A and B
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 379-382

```cpp
 379:   auto regA = flatten(a, kReg);
 380:   auto regB = flatten(b, kReg);
 381:   auto laneA = flatten(a, kLane);
 382:   auto laneB = flatten(b, kLane);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 384-390

```cpp
 384:   // Compute the number of registers that start the tile
 385:   SmallVector<int32_t> vbasis = intersectionBasis(regA, regB, dim);
 386:   // We need to have at least nRegB vectorisation
 387:   if (vbasis.size() < nRegB) {
 388:     return std::nullopt;
 389:   }
 390:   vbasis.resize(nRegB);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 392-400

```cpp
 392:   auto index = [](ArrayRef<int32_t> lane, ArrayRef<int32_t> laneIdTile) {
 393:     SmallVector<int32_t> ret;
 394:     for (auto id : laneIdTile) {
 395:       ret.push_back(lane[id]);
 396:     }
 397:     return ret;
 398:   };
 399:   auto laneTileA = index(laneA, laneIdTileA);
 400:   auto laneTileB = index(laneB, laneIdTileB);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 402-419

```cpp
 402:   // We need the tiles to be contiguous
 403:   auto isZero = [](int32_t b) { return b == 0; };
 404:   if (llvm::any_of(laneTileA, isZero) || llvm::any_of(laneTileB, isZero)) {
 405:     return std::nullopt;
 406:   }
 407:   // The first lanes must map to registers in A
 408:   for (int i = 0; i < nRegA - nRegB; ++i) {
 409:     if (cvt.getBasis(kLane, laneIdTileB[i], kReg) == 0) {
 410:       return std::nullopt;
 411:     }
 412:   }
 413:   // The rest of the lanes must map to each other
 414:   for (auto [idxA, idxB] :
 415:        llvm::zip(laneIdTileA, laneIdTileB.take_back(laneIdTileA.size()))) {
 416:     if (cvt.getBasis(kLane, idxB, kLane) != (1 << idxA)) {
 417:       return std::nullopt;
 418:     }
 419:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 420-422

```cpp
 420:   vbasis.append(laneTileB.begin(), laneTileB.end());
 421:   return vbasis;
 422: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 424-433

```cpp
 424: LinearLayout optimalSwizzling(const LinearLayout &src, const LinearLayout &dst,
 425:                               int32_t bitwidth, ArrayRef<int32_t> vbasis,
 426:                               ArrayRef<int32_t> tileSrc,
 427:                               ArrayRef<int32_t> tileDst,
 428:                               ArrayRef<int32_t> blockBases,
 429:                               ArrayRef<std::pair<StringAttr, int32_t>> outDims,
 430:                               int32_t leaveReps = 0, int32_t numBanks = 32) {
 431:   // We work on the flattened tensors as the tensor dimensions are not relevant
 432:   assert(src.getNumOutDims() == 1 && dst.getNumOutDims() == 1 &&
 433:          "src and dst must have a single output dimension");
```

- **EN:** Defines `optimalSwizzling`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `optimalSwizzling`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 435-437

```cpp
 435:   const int32_t dim = src.getTotalOutDimSizeLog2();
 436:   auto *ctx = src.getInDimNames().begin()->getContext();
 437:   auto kReg = StringAttr::get(ctx, "register");
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 439-450

```cpp
 439:   auto regsNotZero = [kReg](const LinearLayout &ll) {
 440:     return llvm::all_of(
 441:         ll.getBases().lookup(kReg),
 442:         [](const std::vector<int32_t> &basis) { return basis[0] != 0; });
 443:   };
 444:   assert(
 445:       regsNotZero(src) &&
 446:       "Remove register broadcasting from src. See actionRemoveBroadcastedRegs");
 447:   assert(
 448:       regsNotZero(dst) &&
 449:       "Remove register broadcasting from dst. See actionRemoveBroadcastedRegs");
 450:   auto nonZeroBlockBases = removeZeros(blockBases);
```

- **EN:** Defines `llvm::all_of`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `llvm::all_of`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 452-457

```cpp
 452:   llvm::SmallVector<int32_t> bankSrc;
 453:   bankSrc.append(vbasis.begin(), vbasis.end());
 454:   bankSrc.append(tileSrc.begin(), tileSrc.end());
 455:   llvm::SmallVector<int32_t> bankDst;
 456:   bankDst.append(vbasis.begin(), vbasis.end());
 457:   bankDst.append(tileDst.begin(), tileDst.end());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 459-467

```cpp
 459:   // Bits in a bank segment: num banks x 32 bits
 460:   int32_t bankBits = numBanks * 32;
 461:   // Bases needed to cover a whole bank segment
 462:   const int32_t lenBbasis = std::min<int32_t>(
 463:       llvm::Log2_32(bankBits / ((1 << vbasis.size()) * bitwidth)),
 464:       dim - vbasis.size() - nonZeroBlockBases.size());
 465:   // Bases to cover all the tensor
 466:   const int32_t lenSbasis =
 467:       dim - lenBbasis - vbasis.size() - nonZeroBlockBases.size();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 469-470

```cpp
 469:   auto sbasis =
 470:       computeSegment(bankSrc, bankDst, nonZeroBlockBases, dim, lenSbasis);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 472-478

```cpp
 472:   // The bank is the complement of the union of the vector and the start of the
 473:   // segments and the block bases
 474:   SmallVector<int32_t> unionBasis;
 475:   unionBasis.append(vbasis.begin(), vbasis.end());
 476:   unionBasis.append(sbasis.begin(), sbasis.end());
 477:   unionBasis.append(nonZeroBlockBases.begin(), nonZeroBlockBases.end());
 478:   SmallVector<int32_t> bbasis = complementBasis(unionBasis, dim);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 480-481

```cpp
 480:   assert(bbasis.size() == lenBbasis + (lenSbasis - sbasis.size()) &&
 481:          "bbasis size mismatch");
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 483-487

```cpp
 483:   // Build the 1D result layout
 484:   StringAttr vecAttr = StringAttr::get(ctx, "vector");
 485:   StringAttr bankAttr = StringAttr::get(ctx, "bank");
 486:   StringAttr segAttr = StringAttr::get(ctx, "segment");
 487:   StringAttr blockAttr = StringAttr::get(ctx, "block");
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 489-495

```cpp
 489:   // src has just 1 outDim
 490:   LinearLayout basis1D({{vecAttr, unflatten(vbasis)},
 491:                         {bankAttr, unflatten(bbasis)},
 492:                         {segAttr, unflatten(sbasis)},
 493:                         {blockAttr, unflatten(blockBases)}},
 494:                        src.getOutDims(), /*requireSurjective=*/true);
 495:   basis1D = buildReps(ctx, src, dst, basis1D, leaveReps);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 497-498

```cpp
 497:   return basis1D.reshapeOuts(outDims);
 498: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 500-508

```cpp
 500: std::pair<SmallVector<int32_t>, std::optional<bool>>
 501: getVecBasisLdSt(const LinearLayout &srcFlat, const LinearLayout &dstFlat,
 502:                 int32_t bitwidth) {
 503:   auto *ctx = srcFlat.getInDimNames().begin()->getContext();
 504:   auto kReg = StringAttr::get(ctx, "register");
 505:   auto kBlock = StringAttr::get(ctx, "block");
 506:   auto regSrc = flatten(srcFlat, kReg);
 507:   auto regDst = flatten(dstFlat, kReg);
 508:   auto blockSrc = flatten(srcFlat, kBlock);
```

- **EN:** Defines accessor/helper `getVecBasisLdSt` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getVecBasisLdSt`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 510-525

```cpp
 510:   auto dim = srcFlat.getTotalOutDimSizeLog2();
 511:   SmallVector<int32_t> vbasis = intersectionBasis(regSrc, regDst, dim);
 512:   // Restrict the vectorisation to the maximum we can use
 513:   auto maxVecBases = llvm::Log2_32(128 / bitwidth);
 514:   if (vbasis.size() > maxVecBases) {
 515:     vbasis.resize(maxVecBases);
 516:   }
 517:   // We fill-up vbasis until it has 32 bits as best we can
 518:   std::optional<bool> srcFillsBank = std::nullopt;
 519:   if ((1 << vbasis.size()) * bitwidth < 32) {
 520:     auto basesPerBank = llvm::Log2_32(32 / bitwidth);
 521:     auto kWarp = StringAttr::get(ctx, "warp");
 522:     auto warpSrc = removeZeros(flatten(srcFlat, kWarp));
 523:     auto warpDst = removeZeros(flatten(dstFlat, kWarp));
 524:     auto removeVec = [&vbasis](ArrayRef<int32_t> vec) {
 525:       SmallVector<int32_t> result;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 526-543

```cpp
 526:       for (int32_t r : vec) {
 527:         if (!llvm::is_contained(vbasis, r)) {
 528:           result.push_back(r);
 529:         }
 530:       }
 531:       return result;
 532:     };
 533:     auto regSrcWarp = intersectionBasis(removeVec(regSrc), warpDst, dim);
 534:     auto regDstWarp = intersectionBasis(removeVec(regDst), warpSrc, dim);
 535:     // Maximise vectorisation in the load or the store without creating
 536:     // conflicts
 537:     SmallVector<int32_t> largest;
 538:     if (regSrcWarp.size() == regDstWarp.size() && regSrcWarp.size() > 0) {
 539:       // We choose the one with the lowest basis in the hope that it will
 540:       // avoid PRMTs. The comparison of the mins will be strict as the sets
 541:       // removeVec(regSrc) and removeVec(regDst) don't intersect
 542:       if (*llvm::min_element(regSrcWarp) < *llvm::min_element(regDstWarp)) {
 543:         largest = regSrcWarp;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 544-553

```cpp
 544:         srcFillsBank = true;
 545:       } else {
 546:         largest = regDstWarp;
 547:         srcFillsBank = false;
 548:       }
 549:     } else {
 550:       srcFillsBank = regSrcWarp.size() > regDstWarp.size();
 551:       largest = srcFillsBank.value() ? regSrcWarp : regDstWarp;
 552:     }
 553:     vbasis.append(largest.begin(), largest.end());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 555-559

```cpp
 555:     if (vbasis.size() < basesPerBank) {
 556:       // Pad the vectorisation to 32 bits with warp bases
 557:       auto warpSrcWarp = intersectionBasis(warpSrc, warpDst, dim);
 558:       vbasis.append(warpSrcWarp.begin(), warpSrcWarp.end());
 559:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 561-576

```cpp
 561:     int i = 0;
 562:     while (vbasis.size() < basesPerBank &&
 563:            (i < warpSrc.size() || i < warpDst.size())) {
 564:       // If we have not filled up a whole bank, we add more warp bases
 565:       // until we have 32 bits. They will at least avoid bank conflicts in one
 566:       // direction
 567:       if (i < warpSrc.size() && !llvm::is_contained(vbasis, warpSrc[i])) {
 568:         vbasis.push_back(warpSrc[i]);
 569:       }
 570:       if (vbasis.size() < basesPerBank && i < warpDst.size() &&
 571:           !llvm::is_contained(vbasis, warpDst[i]) &&
 572:           !llvm::is_contained(blockSrc, warpDst[i])) {
 573:         vbasis.push_back(warpDst[i]);
 574:       }
 575:       ++i;
 576:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 578-590

```cpp
 578:     // Trim to basesPerBank if we have added more
 579:     // The idea here is that implementing asymmetric vectorisation without bank
 580:     // conflicts is a bit tricky. Basically, in this case, you need to use the
 581:     // vectorisation base in the swizzling pattern. As such, you would not be
 582:     // able to vectorise all the `ld.shared` instructions that you emit, but
 583:     // just about half of them (the ones that are not swizzled). We don't
 584:     // implement this yet
 585:     if (vbasis.size() > basesPerBank) {
 586:       vbasis.resize(basesPerBank);
 587:     }
 588:   }
 589:   return {vbasis, srcFillsBank};
 590: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 592-598

```cpp
 592: int32_t getVecBitwidthLdSt(const LinearLayout &src, const LinearLayout &dst,
 593:                            int32_t bitwidth) {
 594:   auto srcFlat = src.flattenOuts();
 595:   auto dstFlat = dst.flattenOuts();
 596:   auto vbasis = getVecBasisLdSt(srcFlat, dstFlat, bitwidth).first;
 597:   return (1 << vbasis.size()) * bitwidth;
 598: }
```

- **EN:** Defines accessor/helper `getVecBitwidthLdSt` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getVecBitwidthLdSt`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 600-614

```cpp
 600: LinearLayout optimalSwizzlingLdSt(const LinearLayout &src,
 601:                                   const LinearLayout &dst, int32_t bitwidth,
 602:                                   int32_t numBanks, LocalMemOpTile srcTile,
 603:                                   LocalMemOpTile dstTile) {
 604:   auto *ctx = src.getInDimNames().begin()->getContext();
 605:   auto kReg = StringAttr::get(ctx, "register");
 606:   auto kLane = StringAttr::get(ctx, "lane");
 607:   auto kBlock = StringAttr::get(ctx, "block");
 608:   auto srcFlat = src.flattenOuts();
 609:   auto dstFlat = dst.flattenOuts();
 610:   auto regSrc = flatten(srcFlat, kReg);
 611:   auto regDst = flatten(dstFlat, kReg);
 612:   auto laneSrc = flatten(srcFlat, kLane);
 613:   auto laneDst = flatten(dstFlat, kLane);
 614:   auto blockSrc = flatten(srcFlat, kBlock);
```

- **EN:** Defines `optimalSwizzlingLdSt`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `optimalSwizzlingLdSt`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 616-623

```cpp
 616:   auto [vbasis, srcFillsBank] = getVecBasisLdSt(srcFlat, dstFlat, bitwidth);
 617:   auto vecSize = 1 << vbasis.size();
 618:   auto log2Vec = llvm::Log2_32(std::max<int32_t>(1, (vecSize * bitwidth) / 32));
 619:   auto tileSrc = getLaneTile(srcTile, laneSrc, vecSize, bitwidth, numBanks);
 620:   auto tileDst = getLaneTile(dstTile, laneDst, vecSize, bitwidth, numBanks);
 621:   auto smem =
 622:       optimalSwizzling(srcFlat, dstFlat, bitwidth, vbasis, tileSrc, tileDst,
 623:                        blockSrc, src.getOutDims(), 0, numBanks);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 625-641

```cpp
 625:   // We might be able to vectorise a bit more the load or the store
 626:   // This may happen when there is broadcasting
 627:   // e.g for fp32
 628:   // src = {reg = [], lane = [1, 2, 4, 8, 16], warp = [32]}
 629:   // dst = {reg = [8, 32], lane = [0, 0, 1, 2, 4], warp = [16]}
 630:   if (log2Vec < 2) {
 631:     auto smemFlat = smem.flattenOuts();
 632:     // For every bank line, find if it is in regSrc or regDst
 633:     // and if so, store the index in the vector
 634:     SmallVector<size_t> idxBanksInRegSrc;
 635:     SmallVector<size_t> idxBanksInRegDst;
 636:     auto kBank = StringAttr::get(ctx, "bank");
 637:     const auto &banks = flatten(smemFlat, kBank);
 638:     for (auto [i, r] : llvm::enumerate(banks)) {
 639:       if (llvm::is_contained(regSrc, r)) {
 640:         idxBanksInRegSrc.push_back(i);
 641:       }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 642-645

```cpp
 642:       if (llvm::is_contained(regDst, r)) {
 643:         idxBanksInRegDst.push_back(i);
 644:       }
 645:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 647-663

```cpp
 647:     // Choose src/dst if we used them to fill the bank
 648:     // Otherwise choose the max vectorisation
 649:     SmallVector<size_t> bBasisOrder;
 650:     if (srcFillsBank.has_value() && srcFillsBank.value()) {
 651:       bBasisOrder = std::move(idxBanksInRegSrc);
 652:     } else if (srcFillsBank.has_value() && !srcFillsBank.value()) {
 653:       bBasisOrder = std::move(idxBanksInRegDst);
 654:     } else {
 655:       bBasisOrder = idxBanksInRegSrc.size() > idxBanksInRegDst.size()
 656:                         ? std::move(idxBanksInRegSrc)
 657:                         : std::move(idxBanksInRegDst);
 658:     }
 659:     for (int i = 0; i < banks.size(); ++i) {
 660:       if (!llvm::is_contained(bBasisOrder, i)) {
 661:         bBasisOrder.push_back(i);
 662:       }
 663:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 664-666

```cpp
 664:     smem = ColumnAction(bBasisOrder, kBank, smem.getInDimSizeLog2(kBank))
 665:                .apply(smem);
 666:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 668-669

```cpp
 668:   return smem;
 669: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 671-680

```cpp
 671: std::pair<LinearLayout, std::pair<int32_t, int32_t>>
 672: optimalSwizzling(const LinearLayout &src, const LinearLayout &dst,
 673:                  ArrayRef<LocalMemOpTile> srcTiles,
 674:                  ArrayRef<LocalMemOpTile> dstTiles, int32_t bitwidth) {
 675:   assert(bitwidth <= 128 && "bitwidth must be <= 128");
 676:   auto srcFlat = src.flattenOuts();
 677:   auto dstFlat = dst.flattenOuts();
 678:   // Number of total bases needed to cover the necessary contiguous tile
 679:   // We assume using ld.shared.b32.v4 in the case of ld/st ops
 680:   const auto totalBases = llvm::Log2_32(128 / bitwidth);
```

- **EN:** Defines `optimalSwizzling`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `optimalSwizzling`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 682-683

```cpp
 682:   auto *ctx = src.getInDimNames().begin()->getContext();
 683:   auto kReg = StringAttr::get(ctx, "register");
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 685-699

```cpp
 685:   // Find the pairs of instructions that we can use to lower this converet
 686:   SmallVector<std::tuple<std::pair<int32_t, int32_t>, SmallVector<int32_t>>>
 687:       instr;
 688:   for (const auto &[idxSrc, instrSrc] : llvm::enumerate(srcTiles)) {
 689:     auto logRegSrc = totalBases - instrSrc.laneContig.size();
 690:     for (const auto &[idxDst, instrDst] : llvm::enumerate(dstTiles)) {
 691:       auto logRegDst = totalBases - instrDst.laneContig.size();
 692:       auto maybeTile =
 693:           optimalSwizzlingTile(srcFlat, dstFlat, logRegSrc, logRegDst,
 694:                                instrSrc.laneContig, instrDst.laneContig);
 695:       if (maybeTile.has_value()) {
 696:         instr.push_back({{idxSrc, idxDst}, std::move(*maybeTile)});
 697:       }
 698:     }
 699:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 700-717

```cpp
 700:   auto getTile =
 701:       [](const LocalMemOpTile &instr, ArrayRef<int32_t> regs,
 702:          ArrayRef<int32_t> lane,
 703:          ArrayRef<int32_t> vbasis) -> std::optional<SmallVector<int32_t>> {
 704:     // pick the first 3 - laneAddr.size() registers that are not in vbasis
 705:     SmallVector<int32_t> tile;
 706:     auto regNeeded = 3 - instr.laneAddr.size();
 707:     assert(regNeeded >= 0 && "laneAddr.size() must be <= 3");
 708:     for (int32_t r : regs) {
 709:       if (regNeeded == 0) {
 710:         break;
 711:       }
 712:       if (!llvm::is_contained(vbasis, r)) {
 713:         tile.push_back(r);
 714:         regNeeded--;
 715:       }
 716:     }
 717:     // Not enough registers to fill in the tile
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 718-724

```cpp
 718:     if (regNeeded > 0) {
 719:       return std::nullopt;
 720:     }
 721:     auto laneBases = instr.getLaneAddr(lane);
 722:     tile.append(laneBases.begin(), laneBases.end());
 723:     return tile;
 724:   };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 726-743

```cpp
 726:   auto kLane = StringAttr::get(ctx, "lane");
 727:   auto kBlock = StringAttr::get(ctx, "block");
 728:   auto regSrc = flatten(srcFlat, kReg);
 729:   auto regDst = flatten(dstFlat, kReg);
 730:   auto laneSrc = flatten(srcFlat, kLane);
 731:   auto laneDst = flatten(dstFlat, kLane);
 732:   auto blockBases = flatten(srcFlat, kBlock);
 733:   // Get the associated src/dst tiles for each instruction if they exist
 734:   SmallVector<std::tuple<std::pair<int32_t, int32_t>, SmallVector<int32_t>,
 735:                          SmallVector<int32_t>, SmallVector<int32_t>, int32_t>>
 736:       tiles;
 737:   for (auto [instrs, vbasis] : instr) {
 738:     auto maybeTileSrc =
 739:         getTile(srcTiles[instrs.first], regSrc, laneSrc, vbasis);
 740:     auto maybeTileDst =
 741:         getTile(dstTiles[instrs.second], regDst, laneDst, vbasis);
 742:     if (!maybeTileSrc.has_value() || !maybeTileDst.has_value()) {
 743:       continue;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 744-756

```cpp
 744:     }
 745:     // Regs bases missing to get full vectorisation
 746:     auto regsMissing = [](const LocalMemOpTile &instr) {
 747:       return instr.laneContig.size() + instr.laneAddr.size() - 3;
 748:     };
 749:     // We leave 2 reps for combinations of ldmatrix/stmatrix instructions
 750:     // to be able to fully vectorise them
 751:     int32_t leaveReps = std::min(regsMissing(srcTiles[instrs.first]),
 752:                                  regsMissing(dstTiles[instrs.second]));
 753:     assert((leaveReps == 0 || leaveReps == 2) && "leaveReps must be 0 or 2");
 754:     tiles.push_back({instrs, std::move(vbasis), std::move(*maybeTileSrc),
 755:                      std::move(*maybeTileDst), leaveReps});
 756:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 758-775

```cpp
 758:   if (tiles.empty()) {
 759:     // We lower to an ld / st, but can't use LDS128/STS128
 760:     auto smem = optimalSwizzlingLdSt(src, dst, bitwidth);
 761:     return {smem, {0, 0}};
 762:   } else {
 763:     SmallVector<std::tuple<int, LinearLayout, std::pair<int32_t, int32_t>>>
 764:         smems;
 765:     // We choose the pair of instructions that minimises the total bank
 766:     // conflicts
 767:     for (auto [instrs, vbasis, tileSrc, tileDst, leaveReps] : tiles) {
 768:       auto smem =
 769:           optimalSwizzling(srcFlat, dstFlat, bitwidth, vbasis, tileSrc, tileDst,
 770:                            blockBases, src.getOutDims(), leaveReps);
 771:       auto [read, write] = bankConflicts(tileSrc, tileDst, smem);
 772:       smems.push_back({read + write, smem, {instrs.first, instrs.second}});
 773:     }
 774:     // Current heuristic: Minimise total bank conflicts
 775:     // We break ties looking at the number of rounds we do to move the data
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 776-785

```cpp
 776:     auto kReps = StringAttr::get(ctx, "reps");
 777:     auto it = llvm::min_element(smems, [kReps](const auto &a, const auto &b) {
 778:       return std::get<0>(a) < std::get<0>(b) ||
 779:              (std::get<0>(a) == std::get<0>(b) &&
 780:               std::get<1>(a).getInDimSize(kReps) >
 781:                   std::get<1>(b).getInDimSize(kReps));
 782:     });
 783:     return {std::get<1>(*it), std::get<2>(*it)};
 784:   }
 785: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 787-787

```cpp
 787: } // namespace mlir::triton::gpu
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。

## Key Concepts / 关键概念
- **EN:** The file packages reusable support around generic swizzling.
  **CN:** 该文件围绕 Generic Swizzling 封装了可复用的支持逻辑。
- **EN:** Linear layout utilities translate between logical tensor coordinates and physical placement.
  **CN:** 线性布局工具负责在逻辑张量坐标与物理放置之间进行转换。
- **EN:** Layout and encoding decisions are first-class because they determine how work and memory are distributed on the GPU.
  **CN:** 布局与编码决策是第一类问题，因为它们决定了 GPU 上工作与内存的分布方式。

## Dependencies / 依赖关系
- **Internal Triton headers / Triton 内部头文件:** `triton/Tools/GenericSwizzling.h`, `triton/Tools/LayoutUtils.h`, `triton/Tools/LinearLayout.h`
- **MLIR headers / MLIR 头文件:** None
- **LLVM headers / LLVM 头文件:** `llvm/ADT/DenseSet.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallVector.h`, `llvm/Support/Debug.h`, `llvm/Support/MathExtras.h`
- **Standard/library headers / 标准或通用库头文件:** `third_party/f2reduce/f2reduce.h`, `intrin.h`
- **Primary APIs used / 主要 API:** `LinearLayout`
