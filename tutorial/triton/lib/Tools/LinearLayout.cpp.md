# LinearLayout.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `lib/Tools/LinearLayout.cpp`
- **Purpose / 作用:** **EN:** Provides reusable utility code for Linear Layout used across the Triton implementation. **CN:** 提供与 Linear Layout 相关的可复用工具代码，供 Triton 各模块共享。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1

```cpp
   1: #include "triton/Tools/LinearLayout.h"
```

- **EN:** Includes the interfaces this file depends on. Triton headers (`LinearLayout.h`) provide domain-specific IR/support, MLIR headers (None) provide rewriting and analysis infrastructure, LLVM headers (None) supply low-level utilities, and standard/library headers (None) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（`LinearLayout.h`）提供领域特定 IR/支持逻辑，MLIR 头文件（None）提供重写与分析基础设施，LLVM 头文件（None）提供底层工具，而标准/通用库头文件（None）提供通用能力。
### Lines 3-5

```cpp
   3: #include <cstdint>
   4: #include <set>
   5: #include <vector>
```

- **EN:** Includes the interfaces this file depends on. Triton headers (None) provide domain-specific IR/support, MLIR headers (None) provide rewriting and analysis infrastructure, LLVM headers (None) supply low-level utilities, and standard/library headers (`cstdint`, `set`, `vector`) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（None）提供领域特定 IR/支持逻辑，MLIR 头文件（None）提供重写与分析基础设施，LLVM 头文件（None）提供底层工具，而标准/通用库头文件（`cstdint`, `set`, `vector`）提供通用能力。
### Lines 7-16

```cpp
   7: #include "mlir/IR/BuiltinAttributes.h"
   8: #include "third_party/f2reduce/f2reduce.h"
   9: #include "triton/Tools/LayoutUtils.h"
  10: #include "triton/Tools/StrUtil.h"
  11: #include "llvm/ADT/STLExtras.h"
  12: #include "llvm/ADT/SetOperations.h"
  13: #include "llvm/ADT/StringRef.h"
  14: #include "llvm/Support/Debug.h"
  15: #include "llvm/Support/ErrorHandling.h"
  16: #include "llvm/Support/MathExtras.h"
```

- **EN:** Includes the interfaces this file depends on. Triton headers (`LayoutUtils.h`, `StrUtil.h`) provide domain-specific IR/support, MLIR headers (`BuiltinAttributes.h`) provide rewriting and analysis infrastructure, LLVM headers (`STLExtras.h`, `SetOperations.h`, `StringRef.h`, `Debug.h`, ... (+2 more)) supply low-level utilities, and standard/library headers (`third_party/f2reduce/f2reduce.h`) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（`LayoutUtils.h`, `StrUtil.h`）提供领域特定 IR/支持逻辑，MLIR 头文件（`BuiltinAttributes.h`）提供重写与分析基础设施，LLVM 头文件（`STLExtras.h`, `SetOperations.h`, `StringRef.h`, `Debug.h`, ... (+2 more)）提供底层工具，而标准/通用库头文件（`third_party/f2reduce/f2reduce.h`）提供通用能力。
### Lines 18-20

```cpp
  18: #define DEBUG_TYPE "linear_layout"
  19: #define DBGS() (llvm::dbgs() << "[" DEBUG_TYPE "]: ")
  20: #define LDBG(X) LLVM_DEBUG(DBGS() << X << "\n")
```

- **EN:** Defines debug logging helpers and compile-time tags used when tracing this pass or utility at runtime.
- **CN:** 这里定义调试日志辅助宏和编译期标签，用于在运行时跟踪该 pass 或工具。
### Lines 22-24

```cpp
  22: #if defined(_MSC_VER) && !defined(__clang__)
  23: // from https://gist.github.com/pps83/3210a2f980fd02bb2ba2e5a1fc4a2ef0
  24: #include <intrin.h>
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 26-30

```cpp
  26: static int __builtin_ctz(unsigned x) {
  27:   unsigned long r;
  28:   _BitScanForward(&r, x);
  29:   return static_cast<int>(r);
  30: }
```

- **EN:** Defines `__builtin_ctz`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `__builtin_ctz`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 32-36

```cpp
  32: static int __builtin_ctzll(unsigned long long x) {
  33:   unsigned long r;
  34:   _BitScanForward64(&r, x);
  35:   return static_cast<int>(r);
  36: }
```

- **EN:** Defines `__builtin_ctzll`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `__builtin_ctzll`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 38-38

```cpp
  38: #endif
```

- **EN:** Defines or closes a file guard so this header/TableGen fragment is included only once during compilation.
- **CN:** 这里定义或关闭文件保护宏，确保该头文件/TableGen 片段在编译过程中只被包含一次。
### Lines 40-40

```cpp
  40: namespace mlir::triton {
```

- **EN:** Opens or closes the namespace nesting for mlir::triton, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 mlir::triton 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 42-45

```cpp
  42: namespace {
  43: using BasesT = LinearLayout::BasesT;
  44: using llvm::SmallDenseSet;
  45: using llvm::Twine;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 47-54

```cpp
  47: BasesT makeBasesMap(
  48:     ArrayRef<std::pair<StringAttr, std::vector<std::vector<int32_t>>>> bases) {
  49:   BasesT ret;
  50:   for (const auto &[inDim, inDimBases] : bases) {
  51:     ret[inDim] = inDimBases;
  52:   }
  53:   return ret;
  54: }
```

- **EN:** Defines `makeBasesMap`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `makeBasesMap`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 56-66

```cpp
  56: // Dump the matrix to stderr in a human-readable format for debugging.
  57: void dumpMatrix(uint64_t *m, int numRows, int numCols) {
  58:   assert(numCols <= 64);
  59:   for (int r = 0; r < numRows; r++) {
  60:     llvm::errs() << "0b";
  61:     for (int c = 0; c < numCols; c++) {
  62:       llvm::errs() << ((m[r] & (1 << c)) != 0 ? "1" : "0");
  63:     }
  64:     llvm::errs() << "\n";
  65:   }
  66: }
```

- **EN:** Defines `dumpMatrix`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `dumpMatrix`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 68-75

```cpp
  68: // Compute the rank of the matrix formed by taking the bases for the given
  69: // outDim as columns.  In other words, finds the number of linearly-independent
  70: // bases for this output dimension.
  71: int getMatrixRank(std::unique_ptr<uint64_t[]> m, int numRows, int numCols) {
  72:   // stride is specified in number of 64-bit words per row, and we pack our
  73:   // matrix so that there's only one uint64_t per row.
  74:   assert(numCols <= 64);
  75:   f2reduce::inplace_rref_strided(m.get(), numRows, numCols, /*stride=*/1);
```

- **EN:** Defines accessor/helper `getMatrixRank` that exposes or updates operation state in a compact, reusable way. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义访问器/辅助函数 `getMatrixRank`，以紧凑且可复用的方式读取或更新操作状态。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 77-84

```cpp
  77:   // The rank of the reduced matrix is simply the number of nonzero rows.
  78:   int rank = 0;
  79:   for (int i = 0; i < numRows; i++) {
  80:     if (m[i] != 0)
  81:       rank++;
  82:   }
  83:   return rank;
  84: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 86-96

```cpp
  86: template <typename T, typename U>
  87: void assertDimsEqualIgnoringOrder(T &&a, U &&b) {
  88:   SmallDenseSet<StringAttr> as(a.begin(), a.end());
  89:   SmallDenseSet<StringAttr> bs(b.begin(), b.end());
  90:   if (as != bs) {
  91:     llvm::report_fatal_error("Dimensions must match, ignoring order, but they "
  92:                              "don't.  Got dims: [" +
  93:                              Twine(triton::join(a, ", ")) + "] and [" +
  94:                              triton::join(b, ", ") + "]");
  95:   }
  96: }
```

- **EN:** Defines `assertDimsEqualIgnoringOrder`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `assertDimsEqualIgnoringOrder`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 98-109

```cpp
  98: template <typename T, typename U>
  99: void assertDimsSubsetIgnoringOrder(T &&small, U &&big) {
 100:   SmallDenseSet<StringAttr> smallSet(small.begin(), small.end());
 101:   SmallDenseSet<StringAttr> bigSet(big.begin(), big.end());
 102:   if (!llvm::set_is_subset(smallSet, bigSet)) {
 103:     llvm::report_fatal_error("Dimensions must be a subset, ignoring order, but "
 104:                              "they aren't.  Got dims: [" +
 105:                              Twine(triton::join(small, ", ")) + "] and [" +
 106:                              triton::join(big, ", ") + "]");
 107:   }
 108: }
 109: } // anonymous namespace
```

- **EN:** Defines `assertDimsSubsetIgnoringOrder`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `assertDimsSubsetIgnoringOrder`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 111-121

```cpp
 111: /*static*/ std::optional<LinearLayout>
 112: LinearLayout::tryCreate(BasesT bases,
 113:                         ArrayRef<std::pair<StringAttr, int32_t>> outDims,
 114:                         bool requireSurjective) {
 115:   LinearLayout ll(std::move(bases), std::move(outDims), NoCheckInvariants{});
 116:   std::optional<std::string> error = ll.checkInvariants(requireSurjective);
 117:   if (error) {
 118:     return std::nullopt;
 119:   }
 120:   return ll;
 121: }
```

- **EN:** Defines `LinearLayout::tryCreate`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `LinearLayout::tryCreate`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 123-130

```cpp
 123: LinearLayout::LinearLayout(BasesT bases,
 124:                            ArrayRef<std::pair<StringAttr, int32_t>> outDims,
 125:                            NoCheckInvariants)
 126:     : bases(std::move(bases)) {
 127:   for (auto [outDim, size] : outDims) {
 128:     this->outDims[outDim] = size;
 129:   }
 130: }
```

- **EN:** Defines `LinearLayout::LinearLayout`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `LinearLayout::LinearLayout`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 132-145

```cpp
 132: LinearLayout::LinearLayout(BasesT bases, ArrayRef<StringAttr> outDimNames)
 133:     : bases(std::move(bases)) {
 134:   // Infer out-dim sizes.
 135:   for (StringAttr outDim : outDimNames) {
 136:     outDims[outDim] = 1;
 137:   }
 138:   for (const auto &[inDim, inDimBases] : this->bases) {
 139:     for (const auto &basis : inDimBases) {
 140:       for (int i = 0; i < basis.size(); i++) {
 141:         int32_t &size = outDims[outDimNames[i]];
 142:         size = std::max<int32_t>(size, llvm::NextPowerOf2(basis[i]));
 143:       }
 144:     }
 145:   }
```

- **EN:** Defines `LinearLayout::LinearLayout`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `LinearLayout::LinearLayout`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 147-152

```cpp
 147:   std::optional<std::string> error =
 148:       checkInvariants(/*requireSurjective=*/true);
 149:   if (error.has_value()) {
 150:     llvm::report_fatal_error(StringRef(*error));
 151:   }
 152: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 154-162

```cpp
 154: LinearLayout::LinearLayout(BasesT bases,
 155:                            ArrayRef<std::pair<StringAttr, int32_t>> outDims,
 156:                            bool requireSurjective)
 157:     : LinearLayout(std::move(bases), std::move(outDims), NoCheckInvariants{}) {
 158:   std::optional<std::string> error = checkInvariants(requireSurjective);
 159:   if (error.has_value()) {
 160:     llvm::report_fatal_error(StringRef(*error));
 161:   }
 162: }
```

- **EN:** Defines `LinearLayout::LinearLayout`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `LinearLayout::LinearLayout`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 164-176

```cpp
 164: std::optional<std::string>
 165: LinearLayout::checkInvariants(bool requireSurjective) {
 166:   // Check that basis values are non-negative.
 167:   for (const auto &[inDim, inDimBases] : bases) {
 168:     for (const auto &basis : inDimBases) {
 169:       if (llvm::any_of(basis, [](int32_t b) { return b < 0; })) {
 170:         return "Invalid bases passed to LinearLayout.  Expected all basis "
 171:                "values to be non-negative, but found a negative value for "
 172:                "in dimension '" +
 173:                inDim.str() + "'.  Full list of bases:" + toString() + "\n";
 174:       }
 175:     }
 176:   }
```

- **EN:** Defines `LinearLayout::checkInvariants`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `LinearLayout::checkInvariants`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 178-189

```cpp
 178:   // Check that the bases all have length equal to outDimNames.size().
 179:   for (const auto &[inDim, inDimBases] : bases) {
 180:     for (const auto &basis : inDimBases) {
 181:       if (basis.size() != outDims.size()) {
 182:         return "Invalid bases passed to LinearLayout.  Expect all bases to "
 183:                "have the same size, equal to outDimNames.size() (" +
 184:                std::to_string(outDims.size()) +
 185:                ").  But this failed for in dimension '" + inDim.str() +
 186:                "'.  Full list of bases:" + toString() + "\n";
 187:       }
 188:     }
 189:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 191-197

```cpp
 191:   // Check that the out-dim sizes are powers of 2.
 192:   for (const auto &[outDim, size] : outDims) {
 193:     if (!llvm::isPowerOf2_32(size)) {
 194:       return "Invalid out-dim size " + std::to_string(size) + " for out-dim '" +
 195:              outDim.str() + "'.  Out-dim sizes must be powers of 2.\n";
 196:     }
 197:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 199-211

```cpp
 199:   // Check that the bases are smaller than the out-dim sizes.
 200:   SmallVector<StringAttr> outDimNames = llvm::to_vector(getOutDimNames());
 201:   for (const auto &[inDim, inDimBases] : this->bases) {
 202:     for (const auto &basis : inDimBases) {
 203:       for (int i = 0; i < basis.size(); i++) {
 204:         if (basis[i] >= outDims[outDimNames[i]]) {
 205:           return "Invalid basis " + std::to_string(basis[i]) + " for in-dim '" +
 206:                  inDim.str() + "' and out-dim '" + outDimNames[i].str() +
 207:                  "'.  Basis must be less than the out-dim size.\n";
 208:         }
 209:       }
 210:     }
 211:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 213-225

```cpp
 213:   // Determine whether the this layout is surjective, i.e. that every `out`
 214:   // coordinate can be reached by some `in` coordinate.
 215:   //
 216:   // It's prohibitively slow to calculate this naively, but thankfully, this
 217:   // is equivalent to checking that the number of linearly-independent bases
 218:   // is equal to sum(getOutDimSizeLog2).  This can be computed by finding
 219:   // the rank of the matrix whose columns are those bases.  We can compute
 220:   // the rank of our matrix using Gaussian elimination, which runs in O(n^3)
 221:   // for an n x n matrix.  Our matrix size is sum(inDimSizeLog2) x
 222:   // sum(outDimSizeLog2), so this should be plenty fast.
 223:   this->rank =
 224:       getMatrixRank(getMatrix(*this), /*numRows=*/getTotalOutDimSizeLog2(),
 225:                     /*numCols=*/getTotalInDimSizeLog2());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 227-231

```cpp
 227:   if (requireSurjective && !isSurjective()) {
 228:     return "Layout is expected to be surjective, i.e. every `out` coordinate "
 229:            "can be reached by some `in` coordinate, but was not:" +
 230:            toString();
 231:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 233-234

```cpp
 233:   return std::nullopt;
 234: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 236-239

```cpp
 236: LinearLayout::LinearLayout(
 237:     ArrayRef<std::pair<StringAttr, std::vector<std::vector<int32_t>>>> bases,
 238:     ArrayRef<StringAttr> outDimNames)
 239:     : LinearLayout(makeBasesMap(bases), outDimNames) {}
```

- **EN:** Defines `LinearLayout::LinearLayout`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `LinearLayout::LinearLayout`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 241-244

```cpp
 241: LinearLayout::LinearLayout(
 242:     ArrayRef<std::pair<StringAttr, std::vector<std::vector<int32_t>>>> bases,
 243:     ArrayRef<std::pair<StringAttr, int32_t>> outDims, bool requireSurjective)
 244:     : LinearLayout(makeBasesMap(bases), outDims, requireSurjective) {}
```

- **EN:** Defines `LinearLayout::LinearLayout`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `LinearLayout::LinearLayout`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 246-250

```cpp
 246: /*static*/ LinearLayout LinearLayout::strided1D(int32_t size, int32_t stride,
 247:                                                 StringAttr inDimName,
 248:                                                 StringAttr outDimName) {
 249:   if (size == 0)
 250:     return LinearLayout::empty();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 252-260

```cpp
 252:   assert(llvm::isPowerOf2_32(size));
 253:   std::vector<std::vector<int32_t>> bases;
 254:   for (int32_t i = 1; i < size; i *= 2) {
 255:     bases.emplace_back(std::vector<int32_t>{i * stride});
 256:   }
 257:   bool requiresSurjective = (stride == 1);
 258:   return LinearLayout({{inDimName, std::move(bases)}},
 259:                       {{outDimName, stride * size}}, requiresSurjective);
 260: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 262-267

```cpp
 262: /*static*/ LinearLayout LinearLayout::zeros1D(int32_t size,
 263:                                               StringAttr inDimName,
 264:                                               StringAttr outDimName,
 265:                                               int32_t outDimSize) {
 266:   if (size == 0)
 267:     return LinearLayout::empty();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 269-276

```cpp
 269:   assert(llvm::isPowerOf2_32(size));
 270:   std::vector<std::vector<int32_t>> zeros;
 271:   for (int i = 1; i < size; i *= 2) {
 272:     zeros.emplace_back(std::vector<int32_t>{0});
 273:   }
 274:   return LinearLayout({{inDimName, zeros}}, {{outDimName, outDimSize}},
 275:                       /*requiresSurjective=*/outDimSize == 1);
 276: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 278-288

```cpp
 278: int32_t LinearLayout::getOutDimIndex(StringAttr outDim) const {
 279:   int i = 0;
 280:   for (auto [name, _] : outDims) {
 281:     if (name == outDim) {
 282:       return i;
 283:     }
 284:     i++;
 285:   }
 286:   llvm::report_fatal_error("outDim " + Twine(outDim) + " is not in layout" +
 287:                            toString());
 288: }
```

- **EN:** Defines accessor/helper `LinearLayout::getOutDimIndex` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `LinearLayout::getOutDimIndex`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 290-294

```cpp
 290: int32_t LinearLayout::getInDimSizeLog2(StringAttr inDim) const {
 291:   auto it = bases.find(inDim);
 292:   assert(it != bases.end() && "inDim not found in layout");
 293:   return it->second.size();
 294: }
```

- **EN:** Defines accessor/helper `LinearLayout::getInDimSizeLog2` that exposes or updates operation state in a compact, reusable way. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义访问器/辅助函数 `LinearLayout::getInDimSizeLog2`，以紧凑且可复用的方式读取或更新操作状态。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 296-301

```cpp
 296: int32_t LinearLayout::getTotalInDimSizeLog2() const {
 297:   return std::accumulate(getInDimNames().begin(), getInDimNames().end(), 0,
 298:                          [&](int32_t acc, StringAttr inDim) {
 299:                            return acc + getInDimSizeLog2(inDim);
 300:                          });
 301: }
```

- **EN:** Defines accessor/helper `LinearLayout::getTotalInDimSizeLog2` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `LinearLayout::getTotalInDimSizeLog2`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 303-307

```cpp
 303: int32_t LinearLayout::getOutDimSizeLog2(StringAttr outDim) const {
 304:   auto it = outDims.find(outDim);
 305:   assert(it != outDims.end() && "outDim not found in layout");
 306:   return llvm::Log2_32(it->second);
 307: }
```

- **EN:** Defines accessor/helper `LinearLayout::getOutDimSizeLog2` that exposes or updates operation state in a compact, reusable way. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义访问器/辅助函数 `LinearLayout::getOutDimSizeLog2`，以紧凑且可复用的方式读取或更新操作状态。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 309-314

```cpp
 309: int32_t LinearLayout::getTotalOutDimSizeLog2() const {
 310:   return std::accumulate(getOutDimNames().begin(), getOutDimNames().end(), 0,
 311:                          [&](int32_t acc, StringAttr outDim) {
 312:                            return acc + getOutDimSizeLog2(outDim);
 313:                          });
 314: }
```

- **EN:** Defines accessor/helper `LinearLayout::getTotalOutDimSizeLog2` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `LinearLayout::getTotalOutDimSizeLog2`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 316-318

```cpp
 316: int32_t LinearLayout::getNumConsecutiveInOut() const {
 317:   if (bases.empty() || getNumOutDims() == 0)
 318:     return 1;
```

- **EN:** Defines accessor/helper `LinearLayout::getNumConsecutiveInOut` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `LinearLayout::getNumConsecutiveInOut`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 320-331

```cpp
 320:   // Count how many of the initial bases for the first in-dim are
 321:   // (2^i, 0, ..., 0).
 322:   const auto &firstInDimBases = bases.begin()->second;
 323:   int consec = 0;
 324:   for (; consec < firstInDimBases.size(); consec++) {
 325:     const auto &basis = firstInDimBases[consec];
 326:     if (basis[0] != (1 << consec) ||
 327:         !std::all_of(basis.begin() + 1, basis.end(),
 328:                      [](int32_t x) { return x == 0; })) {
 329:       break;
 330:     }
 331:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 333-342

```cpp
 333:   // `or` together all other bases' first out-dim.
 334:   int32_t otherBits = 0;
 335:   for (const auto &[inDim, inDimBases] : bases) {
 336:     for (int i = 0; i < inDimBases.size(); i++) {
 337:       if (inDim != bases.begin()->first || i >= consec) {
 338:         otherBits |= inDimBases[i][0];
 339:       }
 340:     }
 341:   }
 342:   int32_t trailingZeros = otherBits != 0 ? __builtin_ctz(otherBits) : 31;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 344-345

```cpp
 344:   return 1 << std::min(consec, trailingZeros);
 345: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 347-348

```cpp
 347: LinearLayout LinearLayout::transposeIns(ArrayRef<StringAttr> newInDims) const {
 348:   assertDimsEqualIgnoringOrder(newInDims, getInDimNames());
```

- **EN:** Defines `LinearLayout::transposeIns`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `LinearLayout::transposeIns`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 350-356

```cpp
 350:   BasesT newBases;
 351:   for (const auto &inDim : newInDims) {
 352:     newBases[inDim] = bases.find(inDim)->second;
 353:   }
 354:   return LinearLayout(std::move(newBases), llvm::to_vector(outDims),
 355:                       isSurjective());
 356: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 358-360

```cpp
 358: LinearLayout
 359: LinearLayout::transposeOuts(ArrayRef<StringAttr> newOutDims) const {
 360:   assertDimsEqualIgnoringOrder(newOutDims, getOutDimNames());
```

- **EN:** Defines `LinearLayout::transposeOuts`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `LinearLayout::transposeOuts`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 362-365

```cpp
 362:   std::vector<int32_t> permutation;
 363:   for (const auto &outDim : newOutDims) {
 364:     permutation.push_back(getOutDimIndex(outDim));
 365:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 367-377

```cpp
 367:   BasesT newBases;
 368:   for (const auto &[inDim, inDimBases] : bases) {
 369:     auto &newInDimBases = newBases[inDim];
 370:     for (const auto &basis : inDimBases) {
 371:       std::vector<int32_t> newBasis;
 372:       for (int32_t i : permutation) {
 373:         newBasis.push_back(basis[i]);
 374:       }
 375:       newInDimBases.push_back(std::move(newBasis));
 376:     }
 377:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 379-384

```cpp
 379:   SmallVector<std::pair<StringAttr, int32_t>> newOutDimSizes;
 380:   for (auto outDim : newOutDims) {
 381:     newOutDimSizes.push_back({outDim, getOutDimSize(outDim)});
 382:   }
 383:   return LinearLayout(std::move(newBases), newOutDimSizes, isSurjective());
 384: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 386-395

```cpp
 386: LinearLayout LinearLayout::reshapeIns(
 387:     ArrayRef<std::pair<StringAttr, int32_t>> newInDims) const {
 388:   assert(llvm::all_of(newInDims, [&](auto &inDim) {
 389:     return llvm::isPowerOf2_32(inDim.second);
 390:   }));
 391:   assert(getTotalInDimSize() == std::accumulate(newInDims.begin(),
 392:                                                 newInDims.end(), 1,
 393:                                                 [&](int32_t acc, auto &inDim) {
 394:                                                   return acc * inDim.second;
 395:                                                 }));
```

- **EN:** Defines `LinearLayout::reshapeIns`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `LinearLayout::reshapeIns`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 397-404

```cpp
 397:   // First flatten into a single in-dimension.  Then split it up according
 398:   // to `newInDims`.
 399:   SmallVector<std::vector<int32_t>> flatBases;
 400:   for (const auto &[inDim, inDimBases] : bases) {
 401:     for (const auto &basis : inDimBases) {
 402:       flatBases.push_back(basis);
 403:     }
 404:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 406-416

```cpp
 406:   BasesT newBases;
 407:   int i = 0;
 408:   for (const auto &[inDim, inDimSize] : newInDims) {
 409:     auto &newInDimBases = newBases[inDim];
 410:     for (int j = 1; j < inDimSize; j *= 2) {
 411:       newInDimBases.push_back(flatBases[i++]);
 412:     }
 413:   }
 414:   return LinearLayout(std::move(newBases), llvm::to_vector(outDims),
 415:                       isSurjective());
 416: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 418-426

```cpp
 418: LinearLayout LinearLayout::reshapeOuts(
 419:     ArrayRef<std::pair<StringAttr, int32_t>> newOutDims) const {
 420:   assert(llvm::all_of(newOutDims, [&](auto &outDim) {
 421:     return llvm::isPowerOf2_32(outDim.second);
 422:   }));
 423:   assert(getTotalOutDimSize() ==
 424:          std::accumulate(
 425:              newOutDims.begin(), newOutDims.end(), 1,
 426:              [&](int32_t acc, auto &outDim) { return acc * outDim.second; }));
```

- **EN:** Defines `LinearLayout::reshapeOuts`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `LinearLayout::reshapeOuts`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 428-432

```cpp
 428:   SmallVector<int32_t> shifts;
 429:   shifts.push_back(0);
 430:   for (StringAttr outDim : getOutDimNames()) {
 431:     shifts.push_back(shifts.back() + getOutDimSizeLog2(outDim));
 432:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 434-446

```cpp
 434:   // Flatten into a single out-dimension.  Then split it up according to
 435:   // `newOutDims`.
 436:   llvm::MapVector<StringAttr, std::vector<int32_t>> flatBases;
 437:   for (const auto &[inDim, inDimBases] : bases) {
 438:     auto &flatInBases = flatBases[inDim];
 439:     for (const auto &basis : inDimBases) {
 440:       int b = 0;
 441:       for (int i = 0; i < basis.size(); i++) {
 442:         b += basis[i] << shifts[i];
 443:       }
 444:       flatInBases.push_back(b);
 445:     }
 446:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 448-459

```cpp
 448:   BasesT newBases;
 449:   for (const auto &[inDim, flatInBases] : flatBases) {
 450:     std::vector<std::vector<int32_t>> &newInDimBases = newBases[inDim];
 451:     for (int32_t b : flatInBases) {
 452:       std::vector<int32_t> multiDimBasis;
 453:       for (int32_t newSize : llvm::make_second_range(newOutDims)) {
 454:         multiDimBasis.push_back(b % newSize);
 455:         b /= newSize;
 456:       }
 457:       newInDimBases.push_back(std::move(multiDimBasis));
 458:     }
 459:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 461-462

```cpp
 461:   return LinearLayout(std::move(newBases), newOutDims, isSurjective());
 462: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 464-472

```cpp
 464: LinearLayout LinearLayout::resizeInDim(StringAttr inDim,
 465:                                        int32_t newSize) const {
 466:   assert(llvm::isPowerOf2_32(newSize));
 467:   assert(newSize <= getInDimSize(inDim));
 468:   auto newBases = bases;
 469:   newBases[inDim].resize(llvm::Log2_32(newSize));
 470:   return LinearLayout(std::move(newBases), getOutDims(),
 471:                       /*requiresSurjective=*/false);
 472: }
```

- **EN:** Defines `LinearLayout::resizeInDim`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `LinearLayout::resizeInDim`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 474-491

```cpp
 474: LinearLayout LinearLayout::resizeOutDim(StringAttr outDim,
 475:                                         int32_t newSize) const {
 476:   assert(llvm::isPowerOf2_32(newSize));
 477:   assert(newSize <= getOutDimSize(outDim));
 478:   auto newBases = bases;
 479:   // Zero-out the basis vectors that are greater than or equal to the new size
 480:   for (auto &[inDim, inDimBases] : newBases) {
 481:     for (auto &basis : inDimBases) {
 482:       auto &b = basis[getOutDimIndex(outDim)];
 483:       if (b >= newSize) {
 484:         b = 0;
 485:       }
 486:     }
 487:   }
 488:   auto outDims = getOutDims();
 489:   for (auto &[dim, size] : outDims) {
 490:     if (dim == outDim) {
 491:       size = newSize;
```

- **EN:** Defines `LinearLayout::resizeOutDim`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `LinearLayout::resizeOutDim`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 492-496

```cpp
 492:     }
 493:   }
 494:   return LinearLayout(std::move(newBases), outDims,
 495:                       /*requiresSurjective=*/false);
 496: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 498-505

```cpp
 498: LinearLayout LinearLayout::concatIns(const LinearLayout &other) const {
 499:   assert(llvm::to_vector(getOutDimNames()) ==
 500:              llvm::to_vector(other.getOutDimNames()) &&
 501:          "layouts must have the same output dimensions");
 502:   for (StringAttr outDim : getOutDimNames()) {
 503:     assert(getOutDimSize(outDim) == other.getOutDimSize(outDim) &&
 504:            "layouts must have the same output dimension sizes");
 505:   }
```

- **EN:** Defines `LinearLayout::concatIns`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `LinearLayout::concatIns`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 507-515

```cpp
 507:   LinearLayout::BasesT resultBases = getBases();
 508:   for (auto &bases : other.getBases())
 509:     resultBases.insert(bases);
 510:   SmallVector<std::pair<StringAttr, int32_t>> newOutDims;
 511:   for (auto &[outDim, outDimSize] : outDims)
 512:     newOutDims.emplace_back(outDim, outDimSize);
 513:   return LinearLayout(std::move(resultBases), newOutDims,
 514:                       /*requiresSurjective=*/false);
 515: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 517-524

```cpp
 517: LinearLayout LinearLayout::concatOuts(const LinearLayout &other) const {
 518:   assert(llvm::to_vector(getInDimNames()) ==
 519:              llvm::to_vector(other.getInDimNames()) &&
 520:          "layouts must have the same input dimensions");
 521:   for (StringAttr inDim : getInDimNames()) {
 522:     assert(getInDimSize(inDim) == other.getInDimSize(inDim) &&
 523:            "layouts must have the same input dimension sizes");
 524:   }
```

- **EN:** Defines `LinearLayout::concatOuts`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `LinearLayout::concatOuts`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 526-542

```cpp
 526:   LinearLayout::BasesT result;
 527:   for (auto [lhsBases, rhsBases] : llvm::zip(getBases(), other.getBases())) {
 528:     auto &resultBases = result[lhsBases.first];
 529:     assert(lhsBases.first == rhsBases.first);
 530:     for (auto [lhsBasis, rhsBasis] :
 531:          llvm::zip(lhsBases.second, rhsBases.second)) {
 532:       std::vector<int32_t> resultBasis;
 533:       llvm::append_range(resultBasis, lhsBasis);
 534:       llvm::append_range(resultBasis, rhsBasis);
 535:       resultBases.push_back(std::move(resultBasis));
 536:     }
 537:   }
 538:   SmallVector<std::pair<StringAttr, int32_t>> newOutDims;
 539:   for (auto &[outDim, outDimSize] : outDims)
 540:     newOutDims.emplace_back(outDim, outDimSize);
 541:   for (auto &[outDim, outDimSize] : other.outDims)
 542:     newOutDims.emplace_back(outDim, outDimSize);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 543-545

```cpp
 543:   return LinearLayout(std::move(result), newOutDims,
 544:                       /*requiresSurjective=*/false);
 545: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 547-564

```cpp
 547: std::optional<LinearLayout> divideLeft(const LinearLayout &A,
 548:                                        const LinearLayout &B) {
 549:   // Compute a C such that A = B * C if it exists.
 550:   // Note that such a C exists iff (every pair of input/output dim of) A is of
 551:   // the form
 552:   // [[B, 0],
 553:   //  [0, C]]
 554:   // as a matrix, whenever those dimensions are present in B.
 555:   for (StringAttr dim : B.getInDimNames()) {
 556:     if (!llvm::is_contained(A.getInDimNames(), dim))
 557:       return std::nullopt;
 558:   }
 559:   for (StringAttr dim : B.getOutDimNames()) {
 560:     if (!llvm::is_contained(A.getOutDimNames(), dim))
 561:       return std::nullopt;
 562:   }
 563:   // Compute candidate C's log-sizes for output dimensions.
 564:   llvm::MapVector<StringAttr, int32_t> cOutDimSizes;
```

- **EN:** Defines `divideLeft`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `divideLeft`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 565-572

```cpp
 565:   for (StringAttr outDim : A.getOutDimNames()) {
 566:     int outA = A.getOutDimSizeLog2(outDim);
 567:     int outB = B.hasOutDim(outDim) ? B.getOutDimSizeLog2(outDim) : 0;
 568:     int outC = outA - outB;
 569:     if (outC < 0)
 570:       return std::nullopt;
 571:     cOutDimSizes[outDim] = 1 << outC;
 572:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 574-580

```cpp
 574:   LinearLayout::BasesT cBases;
 575:   for (StringAttr inDim : A.getInDimNames()) {
 576:     int inA = A.getInDimSizeLog2(inDim);
 577:     int inB = B.hasInDim(inDim) ? B.getInDimSizeLog2(inDim) : 0;
 578:     int inC = inA - inB;
 579:     if (inC < 0)
 580:       return std::nullopt;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 582-591

```cpp
 582:     std::vector<std::vector<int32_t>> basesForDim;
 583:     // Check that A’s first inB entries agree with B.
 584:     for (int i = 0; i < inB; ++i) {
 585:       for (StringAttr outDim : A.getOutDimNames()) {
 586:         int expected = B.hasOutDim(outDim) ? B.getBasis(inDim, i, outDim) : 0;
 587:         int actual = A.getBasis(inDim, i, outDim);
 588:         if (actual != expected)
 589:           return std::nullopt;
 590:       }
 591:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 593-598

```cpp
 593:     // Extract the candidate C bases from the remaining (shifted) entries in A.
 594:     for (int i = inB; i < inA; ++i) {
 595:       std::vector<int32_t> candidateBasis;
 596:       for (StringAttr outDim : llvm::make_first_range(cOutDimSizes)) {
 597:         int outB = B.hasOutDim(outDim) ? B.getOutDimSizeLog2(outDim) : 0;
 598:         int v = A.getBasis(inDim, i, outDim);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 600-608

```cpp
 600:         // The lower outB bits must be zero.
 601:         if ((v & ((1 << outB) - 1)) != 0)
 602:           return std::nullopt;
 603:         candidateBasis.push_back(v >> outB);
 604:       }
 605:       basesForDim.push_back(std::move(candidateBasis));
 606:     }
 607:     cBases[inDim] = basesForDim;
 608:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 610-619

```cpp
 610:   SmallVector<std::pair<StringAttr, int32_t>> COutDims;
 611:   for (auto [outDim, outC] : cOutDimSizes) {
 612:     COutDims.push_back({outDim, outC});
 613:   }
 614:   // If the layout A and B are surjective, then C should also be surjective.
 615:   LinearLayout C(std::move(cBases), COutDims,
 616:                  /*requireSurjective=*/A.isSurjective() && B.isSurjective());
 617:   assert(B * C == A);
 618:   return C;
 619: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 621-628

```cpp
 621: std::optional<LinearLayout> divideRight(const LinearLayout &A,
 622:                                         const LinearLayout &B) {
 623:   // Compute a C such that A = C * B if it exists.
 624:   // Note that such a C exists iff (every pair of input/output dim of) A is of
 625:   // the form
 626:   // [[C, 0],
 627:   //  [0, B]]
 628:   // as a matrix, whenever those dimensions are present in B.
```

- **EN:** Defines `divideRight`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `divideRight`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 630-638

```cpp
 630:   // Check that B's in-dimensions and out-dimensions are contained in A.
 631:   for (StringAttr dim : B.getInDimNames()) {
 632:     if (!llvm::is_contained(A.getInDimNames(), dim))
 633:       return std::nullopt;
 634:   }
 635:   for (StringAttr dim : B.getOutDimNames()) {
 636:     if (!llvm::is_contained(A.getOutDimNames(), dim))
 637:       return std::nullopt;
 638:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 640-649

```cpp
 640:   // Compute candidate C's log-sizes for output dimensions.
 641:   llvm::MapVector<StringAttr, int32_t> cOutDimSizes;
 642:   for (StringAttr outDim : A.getOutDimNames()) {
 643:     int outA = A.getOutDimSizeLog2(outDim);
 644:     int outB = B.hasOutDim(outDim) ? B.getOutDimSizeLog2(outDim) : 0;
 645:     int outC = outA - outB;
 646:     if (outC < 0)
 647:       return std::nullopt;
 648:     cOutDimSizes[outDim] = 1 << outC;
 649:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 651-659

```cpp
 651:   // For candidate C, its in-dim sizes come from subtracting B's in-dim sizes
 652:   // from A's.
 653:   LinearLayout::BasesT cBases;
 654:   for (StringAttr inDim : A.getInDimNames()) {
 655:     int inA = A.getInDimSizeLog2(inDim);
 656:     int inB = B.hasInDim(inDim) ? B.getInDimSizeLog2(inDim) : 0;
 657:     int inC = inA - inB;
 658:     if (inC < 0)
 659:       return std::nullopt;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 661-669

```cpp
 661:     std::vector<std::vector<int32_t>> basesForDim;
 662:     // The first inC basis vectors come directly from C.
 663:     for (int i = 0; i < inC; ++i) {
 664:       std::vector<int32_t> candidate;
 665:       for (StringAttr outDim : llvm::make_first_range(cOutDimSizes)) {
 666:         candidate.push_back(A.getBasis(inDim, i, outDim));
 667:       }
 668:       basesForDim.push_back(std::move(candidate));
 669:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 671-688

```cpp
 671:     // The remaining inB basis vectors in A should correspond to B after being
 672:     // shifted.
 673:     for (int i = inC; i < inA; ++i) {
 674:       int j = i - inC; // Index into B's basis vectors for this inDim.
 675:       for (StringAttr outDim : B.getOutDimNames()) {
 676:         int outA = A.getOutDimSizeLog2(outDim);
 677:         int outB = B.getOutDimSizeLog2(outDim);
 678:         int outC = outA - outB; // Expected log2 size for C in this output.
 679:         int shift = outC;
 680:         int v = A.getBasis(inDim, i, outDim);
 681:         // The lower shift bits must be zero.
 682:         if ((v & ((1 << shift) - 1)) != 0)
 683:           return std::nullopt;
 684:         int recovered = v >> shift;
 685:         int expected = B.getBasis(inDim, j, outDim);
 686:         if (recovered != expected)
 687:           return std::nullopt;
 688:       }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 689-691

```cpp
 689:     }
 690:     cBases[inDim] = basesForDim;
 691:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 693-701

```cpp
 693:   SmallVector<std::pair<StringAttr, int32_t>> COutDims;
 694:   for (auto [outDim, size] : cOutDimSizes)
 695:     COutDims.push_back({outDim, size});
 696:   // If A and B are surjective, then C should also be surjective.
 697:   LinearLayout C(std::move(cBases), COutDims,
 698:                  /*requireSurjective=*/A.isSurjective() && B.isSurjective());
 699:   assert(C * B == A);
 700:   return C;
 701: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 703-708

```cpp
 703: LinearLayout operator*(LinearLayout inner, LinearLayout outer) {
 704:   // Check that dims common to outer and inner have the same relative order.
 705:   auto inDims = supremum(llvm::to_vector(inner.getInDimNames()),
 706:                          llvm::to_vector(outer.getInDimNames()));
 707:   auto outDims = supremum(llvm::to_vector(inner.getOutDimNames()),
 708:                           llvm::to_vector(outer.getOutDimNames()));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 710-726

```cpp
 710:   // Get the sizeLog2 of all input and output dimensions we're going to
 711:   // consider, in order.  `inner` is more minor, so its dimensions come
 712:   // first.
 713:   llvm::MapVector<StringAttr, int32_t> inDimSizesLog2;
 714:   llvm::MapVector<StringAttr, int32_t> outDimSizesLog2;
 715:   for (const auto &dim : inDims)
 716:     inDimSizesLog2.insert({dim, 0});
 717:   for (const auto &dim : outDims)
 718:     outDimSizesLog2.insert({dim, 0});
 719:   for (const auto &layout : {inner, outer}) {
 720:     for (StringAttr inDim : layout.getInDimNames()) {
 721:       inDimSizesLog2[inDim] += layout.getInDimSizeLog2(inDim);
 722:     }
 723:     for (StringAttr outDim : layout.getOutDimNames()) {
 724:       outDimSizesLog2[outDim] += layout.getOutDimSizeLog2(outDim);
 725:     }
 726:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 728-730

```cpp
 728:   BasesT allBases;
 729:   for (auto [inDimName, inDimSizeLog2] : inDimSizesLog2) {
 730:     std::vector<std::vector<int32_t>> &inDimBases = allBases[inDimName];
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 732-734

```cpp
 732:     // Fill with zeros.
 733:     inDimBases = std::vector<std::vector<int32_t>>(
 734:         inDimSizeLog2, std::vector<int32_t>(outDimSizesLog2.size(), 0));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 736-753

```cpp
 736:     for (auto [outDimIdx, outDimNameAndSize] :
 737:          llvm::enumerate(outDimSizesLog2)) {
 738:       auto [outDimName, outDimSize] = outDimNameAndSize;
 739:       if (inner.hasInDim(inDimName) && inner.hasOutDim(outDimName)) {
 740:         for (int i = 0; i < inner.getInDimSizeLog2(inDimName); i++) {
 741:           inDimBases[i][outDimIdx] = inner.getBasis(inDimName, i, outDimName);
 742:         }
 743:       }
 744:       if (outer.hasInDim(inDimName) && outer.hasOutDim(outDimName)) {
 745:         int offset =
 746:             inner.hasInDim(inDimName) ? inner.getInDimSizeLog2(inDimName) : 0;
 747:         int shift = inner.hasOutDim(outDimName)
 748:                         ? inner.getOutDimSizeLog2(outDimName)
 749:                         : 0;
 750:         for (int i = 0; i < outer.getInDimSizeLog2(inDimName); i++) {
 751:           inDimBases[offset + i][outDimIdx] =
 752:               outer.getBasis(inDimName, i, outDimName) << shift;
 753:         }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 754-756

```cpp
 754:       }
 755:     }
 756:   }
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 758-764

```cpp
 758:   llvm::SmallVector<std::pair<StringAttr, int32_t>> outDimSizes;
 759:   for (auto [outDim, sizeLog2] : outDimSizesLog2) {
 760:     outDimSizes.push_back({outDim, 1 << sizeLog2});
 761:   }
 762:   return LinearLayout(std::move(allBases), outDimSizes,
 763:                       inner.isSurjective() && outer.isSurjective());
 764: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 766-772

```cpp
 766: bool LinearLayout::isTrivialOver(ArrayRef<StringAttr> dimNames) const {
 767:   for (StringAttr dim : dimNames) {
 768:     if (!hasInDim(dim) || !hasOutDim(dim)) {
 769:       llvm::report_fatal_error(
 770:           ("dim " + dim.str() + " must be present in the layout").c_str());
 771:     }
 772:   }
```

- **EN:** Defines `LinearLayout::isTrivialOver`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `LinearLayout::isTrivialOver`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 774-786

```cpp
 774:   auto getRemainingDimNames = [&](auto allDimNames) {
 775:     SmallVector<StringAttr> remainingDimNames;
 776:     for (StringAttr dim : allDimNames) {
 777:       if (!llvm::is_contained(dimNames, dim)) {
 778:         remainingDimNames.push_back(dim);
 779:       }
 780:     }
 781:     return remainingDimNames;
 782:   };
 783:   SmallVector<StringAttr> remainingInDimNames =
 784:       getRemainingDimNames(getInDimNames());
 785:   SmallVector<StringAttr> remainingOutDimNames =
 786:       getRemainingDimNames(getOutDimNames());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 788-798

```cpp
 788:   // Think of this as a block-matrix multiplying a vector:
 789:   // [[A, B],  *  [v_1,
 790:   //  [C, D]]      v_2]
 791:   // where v_2 is the dimNames and v_1 is the remainingInDimNames
 792:   // We can quotient out dimNames iff they don't affect the remainingInDimNames
 793:   // in the result. In other words, we want to check that B is zero, and C is
 794:   // zero, and D is the identity
 795:   return squareSublayoutIsIdentity(*this, dimNames) &&
 796:          sublayoutIsZero(remainingInDimNames, dimNames) &&
 797:          sublayoutIsZero(dimNames, remainingOutDimNames);
 798: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 800-808

```cpp
 800: std::optional<LinearLayout>
 801: LinearLayout::quotient(ArrayRef<StringAttr> dimNames) const {
 802:   if (llvm::any_of(dimNames,
 803:                    [this](StringAttr dim) { return !hasInDim(dim); })) {
 804:     return std::nullopt;
 805:   }
 806:   if (!isTrivialOver(dimNames)) {
 807:     return std::nullopt;
 808:   }
```

- **EN:** Defines `LinearLayout::quotient`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `LinearLayout::quotient`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 810-820

```cpp
 810:   // This should probably be even less general, where we ask inDimNames ==
 811:   // outDimNames
 812:   auto getRemainingDimNames = [&](auto allDimNames) {
 813:     SmallVector<StringAttr> remainingDimNames;
 814:     for (StringAttr dim : allDimNames) {
 815:       if (!llvm::is_contained(dimNames, dim)) {
 816:         remainingDimNames.push_back(dim);
 817:       }
 818:     }
 819:     return remainingDimNames;
 820:   };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 822-823

```cpp
 822:   SmallVector<StringAttr> inDimNames = getRemainingDimNames(getInDimNames());
 823:   SmallVector<StringAttr> outDimNames = getRemainingDimNames(getOutDimNames());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 825-826

```cpp
 825:   return sublayout(inDimNames, outDimNames);
 826: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 828-833

```cpp
 828: LinearLayout LinearLayout::sublayout(ArrayRef<StringAttr> inDimNames,
 829:                                      ArrayRef<StringAttr> outDimNames) const {
 830:   assertDimsSubsetIgnoringOrder(inDimNames, getInDimNames());
 831:   assertDimsSubsetIgnoringOrder(outDimNames, getOutDimNames());
 832:   SmallDenseSet<StringAttr> inDimSet(inDimNames.begin(), inDimNames.end());
 833:   SmallDenseSet<StringAttr> outDimSet(outDimNames.begin(), outDimNames.end());
```

- **EN:** Defines `LinearLayout::sublayout`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `LinearLayout::sublayout`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 835-852

```cpp
 835:   SmallVector<int> outDimIndicesToKeep;
 836:   for (auto [i, outDim] : llvm::enumerate(getOutDimNames())) {
 837:     if (outDimSet.contains(outDim)) {
 838:       outDimIndicesToKeep.push_back(i);
 839:     }
 840:   }
 841:   BasesT newBases;
 842:   for (auto [inDim, inDimBases] : bases) {
 843:     if (!inDimSet.contains(inDim)) {
 844:       continue;
 845:     }
 846:     auto &newInDimBases = newBases[inDim];
 847:     for (auto &basis : inDimBases) {
 848:       auto &newBasis = newInDimBases.emplace_back();
 849:       for (int i : outDimIndicesToKeep) {
 850:         newBasis.push_back(basis[i]);
 851:       }
 852:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 853-853

```cpp
 853:   }
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 855-863

```cpp
 855:   SmallVector<std::pair<StringAttr, int32_t>> newOutDims;
 856:   for (auto [outDim, outDimSize] : outDims) {
 857:     if (outDimSet.contains(outDim)) {
 858:       newOutDims.push_back({outDim, outDimSize});
 859:     }
 860:   }
 861:   return LinearLayout(std::move(newBases), std::move(newOutDims),
 862:                       /*requireSurjective=*/false);
 863: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 865-876

```cpp
 865: bool LinearLayout::sublayoutIsZero(ArrayRef<StringAttr> inDimNames,
 866:                                    ArrayRef<StringAttr> outDimNames) const {
 867:   LinearLayout ss = sublayout(inDimNames, outDimNames);
 868:   for (auto [inDim, inDimBases] : ss.bases) {
 869:     for (auto basis : inDimBases) {
 870:       if (!llvm::all_of(basis, [](int32_t b) { return b == 0; })) {
 871:         return false;
 872:       }
 873:     }
 874:   }
 875:   return true;
 876: }
```

- **EN:** Defines `LinearLayout::sublayoutIsZero`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `LinearLayout::sublayoutIsZero`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 878-880

```cpp
 878: SmallVector<std::pair<StringAttr, int32_t>>
 879: LinearLayout::apply(ArrayRef<std::pair<StringAttr, int32_t>> ins) const {
 880:   assertDimsEqualIgnoringOrder(llvm::make_first_range(ins), getInDimNames());
```

- **EN:** Defines `LinearLayout::apply`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `LinearLayout::apply`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 882-894

```cpp
 882:   SmallVector<std::pair<StringAttr, int32_t>> ret;
 883:   for (StringAttr outDim : getOutDimNames()) {
 884:     int32_t outVal = 0;
 885:     for (auto &[inDim, val] : ins) {
 886:       for (int i = 0; i < getInDimSizeLog2(inDim); i++) {
 887:         if (val & (1 << i))
 888:           outVal ^= getBasis(inDim, i, outDim);
 889:       }
 890:     }
 891:     ret.push_back({outDim, outVal});
 892:   }
 893:   return ret;
 894: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 896-900

```cpp
 896: LinearLayout LinearLayout::compose(const LinearLayout &outer) const {
 897:   assertDimsEqualIgnoringOrder(getOutDimNames(), outer.getInDimNames());
 898:   for (StringAttr outDim : getOutDimNames()) {
 899:     assert(getOutDimSize(outDim) <= outer.getInDimSize(outDim));
 900:   }
```

- **EN:** Defines `LinearLayout::compose`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `LinearLayout::compose`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 902-915

```cpp
 902:   BasesT newBases;
 903:   for (const auto &[inDim, inDimBases] : bases) {
 904:     auto &newInDimBases = newBases[inDim];
 905:     for (const auto &basis : inDimBases) {
 906:       SmallVector<std::pair<StringAttr, int32_t>> bases;
 907:       for (auto [outDim, b] : llvm::zip(getOutDimNames(), basis)) {
 908:         bases.push_back({outDim, b});
 909:       }
 910:       auto newBases = outer.apply(bases);
 911:       auto newBasesRange = llvm::make_second_range(newBases);
 912:       newInDimBases.push_back(
 913:           std::vector<int32_t>(newBasesRange.begin(), newBasesRange.end()));
 914:     }
 915:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 917-924

```cpp
 917:   bool compositionIsSurjective =
 918:       isSurjective() && outer.isSurjective() &&
 919:       llvm::all_of(getOutDimNames(), [&](StringAttr outDim) {
 920:         return getOutDimSize(outDim) == outer.getInDimSize(outDim);
 921:       });
 922:   return LinearLayout(std::move(newBases), llvm::to_vector(outer.outDims),
 923:                       compositionIsSurjective);
 924: }
```

- **EN:** Defines `isSurjective`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `isSurjective`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 926-932

```cpp
 926: namespace {
 927: std::unique_ptr<uint64_t[]> concatMatrices(const LinearLayout &A,
 928:                                            const LinearLayout &B) {
 929:   // conv
 930:   assert(A.getTotalOutDimSizeLog2() >= B.getTotalOutDimSizeLog2() &&
 931:          "A must have at least as many output bits as B");
 932:   int numColsA = A.getTotalInDimSizeLog2();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 934-949

```cpp
 934:   // rref expects the lower bits to be the lower indices of the matrix
 935:   auto concat = getMatrix(A);
 936:   auto BMat = getMatrix(B);
 937:   int rowA = 0;
 938:   int rowB = 0;
 939:   for (auto [outDim, outDimSize] : A.getOutDims()) {
 940:     for (int r = 0; r < llvm::Log2_32(outDimSize); r++) {
 941:       if (r < llvm::Log2_32(B.getOutDimSize(outDim))) {
 942:         concat[rowA] |= BMat[rowB] << numColsA;
 943:         rowB++;
 944:       }
 945:       rowA++;
 946:     }
 947:   }
 948:   return concat;
 949: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 951-966

```cpp
 951: LinearLayout lstsq(const LinearLayout &A, const LinearLayout &B) {
 952:   // Solve the least square system AX = B
 953:   // and return the least square solution X by computing RREF and setting
 954:   // the free variables to zero.
 955:   // A and B may not be surjective, but we assume that Im(B) \subset Im(A)
 956:   // Sketch of the algorithm:
 957:   // https://github.com/triton-lang/triton/pull/5309#discussion_r1869084111
 958:   int numRows = A.getTotalOutDimSizeLog2();
 959:   assert(numRows >= B.getTotalOutDimSizeLog2() &&
 960:          "A.lstsq(B) called with incompatible output shapes");
 961:   int numColsA = A.getTotalInDimSizeLog2();
 962:   int numColsB = B.getTotalInDimSizeLog2();
 963:   int numCols = numColsA + numColsB;
 964:   std::unique_ptr<uint64_t[]> combinedMat = concatMatrices(A, B);
 965:   f2reduce::inplace_rref_strided(combinedMat.get(), numRows, numCols,
 966:                                  /*stride=*/1);
```

- **EN:** Defines `lstsq`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `lstsq`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 968-982

```cpp
 968:   // Compute the pivot columns
 969:   // Since A and B have the same image, each row will either have a pivot
 970:   // or will be all zeros
 971:   SmallVector<int32_t> pivotRowOfCol(numColsA, -1);
 972:   for (int r = 0; r < numRows; r++) {
 973:     auto row = combinedMat[r];
 974:     if (row == 0) {
 975:       continue;
 976:     }
 977:     int c = __builtin_ctzll(row);
 978:     assert(c < numColsA && "Precondition broken. Im(B) not contained in Im(A)");
 979:     assert(pivotRowOfCol[c] == -1 &&
 980:            "duplicate pivot => matrix not in RREF or A not injective");
 981:     pivotRowOfCol[c] = r;
 982:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 984-989

```cpp
 984:   // Extract A^{-1}B and complete the matrix using zeros
 985:   std::unique_ptr<uint64_t[]> retMat(new uint64_t[numColsA]());
 986:   for (int c = 0; c < numColsA; ++c) {
 987:     int row = pivotRowOfCol[c];
 988:     retMat[c] = (row == -1) ? 0 : (combinedMat[row] >> numColsA);
 989:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 991-996

```cpp
 991:   // We need names for the in/out dim of the flattened layout we're going to
 992:   // read off from `m`.  These could be anything, doesn't matter.
 993:   assert(!A.getInDimNames().empty() &&
 994:          "attempt to solve lstsq for empty layout");
 995:   StringAttr inDim1D = *A.getInDimNames().begin();
 996:   StringAttr outDim1D = *A.getOutDimNames().begin();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 998-1007

```cpp
 998:   // Read off the new bases.  These are for a flattened 1D -> 1D
 999:   LinearLayout::BasesT retBases;
1000:   auto &bs = retBases[inDim1D];
1001:   for (int c = 0; c < numColsB; c++) {
1002:     int32_t basis = 0;
1003:     for (int r = 0; r < numColsA; r++) {
1004:       basis |= (retMat[r] >> c & 1) << r;
1005:     }
1006:     bs.push_back({basis});
1007:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1009-1011

```cpp
1009:   LinearLayout retFlattened(std::move(retBases),
1010:                             {{outDim1D, A.getTotalInDimSize()}},
1011:                             /*requireSurjective=*/false);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1013-1022

```cpp
1013:   SmallVector<std::pair<StringAttr, int32_t>> retInDims;
1014:   SmallVector<std::pair<StringAttr, int32_t>> retOutDims;
1015:   for (StringAttr dim : B.getInDimNames()) {
1016:     retInDims.push_back({dim, B.getInDimSize(dim)});
1017:   }
1018:   for (StringAttr dim : A.getInDimNames()) {
1019:     retOutDims.push_back({dim, A.getInDimSize(dim)});
1020:   }
1021:   return retFlattened.reshapeIns(retInDims).reshapeOuts(retOutDims);
1022: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 1024-1024

```cpp
1024: } // namespace
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 1026-1032

```cpp
1026: LinearLayout LinearLayout::invertAndCompose(const LinearLayout &outer) const {
1027:   // TODO(Lezcano) Make friend and perhaps rename to `convertFrom` or `lstsq`
1028:   // For this, we need to implement our LLVM lowerings by inverting the "outer"
1029:   // layout, and then iterating over the elements from the "this" layout and
1030:   // fetching the corresponding element from the "outer" layout. This exercises
1031:   // the broadcasting that we incentivise via choosing the minimum norm solution
1032:   // in lstsq.
```

- **EN:** Defines `LinearLayout::invertAndCompose`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `LinearLayout::invertAndCompose`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1034-1045

```cpp
1034:   // The order of dims does not matter. We choose to transpose outer
1035:   auto outDims = llvm::to_vector(getOutDimNames());
1036:   assertDimsEqualIgnoringOrder(outDims, outer.getOutDimNames());
1037:   const auto &B = *this;
1038:   const auto A = outer.transposeOuts(outDims);
1039:   for (auto dim : outDims) {
1040:     assert(A.getOutDimSize(dim) >= B.getOutDimSize(dim) &&
1041:            ("A.invertAndCompose(B) called with incompatible output shapes in " +
1042:             dim.str() + ": " + std::to_string(A.getOutDimSize(dim)) +
1043:             " >= " + std::to_string(B.getOutDimSize(dim)))
1044:                .c_str());
1045:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 1047-1058

```cpp
1047:   // Broadcasting heuristic
1048:   // Imagine we have two layouts with `warps = [[0, 0],  [0, 0]]`
1049:   // (broadcasting) on both layouts. We could map any warp to any warp in the
1050:   // conversion. Now, we want to map them as the identity map, to mark that
1051:   // nothing needs to be done there (`lstsq` would map all the warps to the
1052:   // zero warp, minimum norm solution). The heuristic here is as follows:
1053:   // - If a dimension is the same for both layouts, we want to map it as the
1054:   // identity
1055:   //   Equivalently, we don't add it to the conversion
1056:   // - Otherwise, we just call lstsq (i.e. map all the equivalent elements
1057:   //   to the same input element) to take advantage of broadcasting in shared
1058:   //   memory and avoid saving repeated elements in shared memory
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 1060-1077

```cpp
1060:   // FIXME: We should check that the other dimensions don't touch the image of
1061:   // this dimension.
1062:   SmallVector<StringAttr> identityDims;
1063:   for (auto dim : A.getInDimNames()) {
1064:     if (B.hasInDim(dim)) {
1065:       auto aSub = A.sublayout(dim, outDims);
1066:       auto bSub = B.sublayout(dim, outDims);
1067:       if (aSub.equalIgnoringOutDimSizes(bSub))
1068:         identityDims.push_back(dim);
1069:     }
1070:   }
1071:   SmallVector<StringAttr> ANonIdentityInDims;
1072:   SmallVector<StringAttr> BNonIdentityInDims;
1073:   for (auto dim : A.getInDimNames()) {
1074:     if (!llvm::is_contained(identityDims, dim)) {
1075:       ANonIdentityInDims.push_back(dim);
1076:     }
1077:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 1078-1082

```cpp
1078:   for (auto dim : B.getInDimNames()) {
1079:     if (!llvm::is_contained(identityDims, dim)) {
1080:       BNonIdentityInDims.push_back(dim);
1081:     }
1082:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1084-1085

```cpp
1084:   auto AReduced = A.sublayout(ANonIdentityInDims, outDims);
1085:   auto BReduced = B.sublayout(BNonIdentityInDims, outDims);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1087-1089

```cpp
1087:   // If one is empty, the other must be empty as well
1088:   assert((ANonIdentityInDims.empty()) == (BNonIdentityInDims.empty()));
1089:   bool isEmpty = ANonIdentityInDims.empty();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 1091-1091

```cpp
1091:   auto ret = isEmpty ? LinearLayout::empty() : lstsq(AReduced, BReduced);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1093-1094

```cpp
1093:   // TODO(Lezcano): We should return the reduced layout instead of re-adding the
1094:   // identity maps. With this, we'll be able to kill `minimalCvtLayout`
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 1096-1099

```cpp
1096:   // Add the identity maps for the dimensions that are the same for both layouts
1097:   for (auto dim : identityDims) {
1098:     ret *= LinearLayout::identity1D(A.getInDimSize(dim), dim, dim);
1099:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1101-1105

```cpp
1101:   // Reorder the dimensions in the result to match the order expected by the
1102:   // current and outer layouts.
1103:   return ret.transposeIns(llvm::to_vector(B.getInDimNames()))
1104:       .transposeOuts(llvm::to_vector(A.getInDimNames()));
1105: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1107-1111

```cpp
1107: LinearLayout LinearLayout::invert() const {
1108:   assert(isInvertible() &&
1109:          "A linear layout must be surjective and square to be invertible");
1110:   return pseudoinvert();
1111: }
```

- **EN:** Defines `LinearLayout::invert`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `LinearLayout::invert`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 1113-1119

```cpp
1113: LinearLayout LinearLayout::pseudoinvert() const {
1114:   LinearLayout identity = LinearLayout::empty();
1115:   for (auto outDim : getOutDimNames()) {
1116:     identity *= LinearLayout::identity1D(getOutDimSize(outDim), outDim, outDim);
1117:   }
1118:   return identity.invertAndCompose(*this);
1119: }
```

- **EN:** Defines `LinearLayout::pseudoinvert`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `LinearLayout::pseudoinvert`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1121-1130

```cpp
1121: LinearLayout LinearLayout::squeezeIns(StringAttr dim) const {
1122:   assert(getInDimSize(dim) == 1);
1123:   SmallVector<std::pair<StringAttr, int32_t>> newInDims;
1124:   for (auto inDim : getInDimNames()) {
1125:     if (inDim != dim) {
1126:       newInDims.push_back({inDim, getInDimSize(inDim)});
1127:     }
1128:   }
1129:   return reshapeIns(newInDims);
1130: }
```

- **EN:** Defines `LinearLayout::squeezeIns`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `LinearLayout::squeezeIns`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 1132-1141

```cpp
1132: LinearLayout LinearLayout::squeezeOuts(StringAttr dim) const {
1133:   assert(getOutDimSize(dim) == 1);
1134:   SmallVector<std::pair<StringAttr, int32_t>> newOutDims;
1135:   for (auto [outDim, outDimSize] : getOutDims()) {
1136:     if (outDim != dim) {
1137:       newOutDims.push_back({outDim, outDimSize});
1138:     }
1139:   }
1140:   return reshapeOuts(newOutDims);
1141: }
```

- **EN:** Defines `LinearLayout::squeezeOuts`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `LinearLayout::squeezeOuts`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 1143-1147

```cpp
1143: llvm::MapVector<StringAttr, int32_t>
1144: LinearLayout::getFreeVariableMasks() const {
1145:   std::unique_ptr<uint64_t[]> mat = getMatrix(*this);
1146:   int numRows = getTotalOutDimSizeLog2();
1147:   int numCols = getTotalInDimSizeLog2();
```

- **EN:** Defines accessor/helper `LinearLayout::getFreeVariableMasks` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `LinearLayout::getFreeVariableMasks`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 1149-1152

```cpp
1149:   // stride is specified in number of 64-bit words per row, and we pack our
1150:   // matrix so that there's only one uint64_t per row.
1151:   assert(numCols <= 64);
1152:   f2reduce::inplace_rref_strided(mat.get(), numRows, numCols, /*stride=*/1);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 1154-1162

```cpp
1154:   // For each row in the RREF matrix, identify the column with the first "1".
1155:   // These columns correspond to the basic (i.e. non-free) variables.
1156:   std::set<int32_t> basicVars;
1157:   for (int r = 0; r < numRows; r++) {
1158:     if (mat[r] == 0) {
1159:       continue;
1160:     }
1161:     basicVars.insert(__builtin_ctzll(mat[r]));
1162:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1164-1176

```cpp
1164:   llvm::MapVector<StringAttr, int32_t> ret;
1165:   int c = 0;
1166:   for (StringAttr dim : getInDimNames()) {
1167:     int32_t mask = 0;
1168:     for (int i = 0; i < getInDimSizeLog2(dim); i++, c++) {
1169:       if (basicVars.count(c) == 0) {
1170:         mask |= (1 << i);
1171:       }
1172:     }
1173:     ret[dim] = mask;
1174:   }
1175:   return ret;
1176: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1178-1195

```cpp
1178: LinearLayout LinearLayout::removeZeroBasesAlongDim(StringAttr stripDim) const {
1179:   LinearLayout::BasesT result;
1180:   for (auto &[inDim, inDimBases] : getBases()) {
1181:     auto &newInDimBases = result[inDim];
1182:     if (inDim != stripDim) {
1183:       newInDimBases = inDimBases;
1184:       continue;
1185:     }
1186:     for (auto &basis : inDimBases) {
1187:       if (llvm::any_of(basis, [](int32_t val) { return val != 0; })) {
1188:         newInDimBases.push_back(basis);
1189:       }
1190:     }
1191:   }
1192:   SmallVector<std::pair<StringAttr, int32_t>> newOutDimSizes;
1193:   for (auto outDim : getOutDimNames()) {
1194:     newOutDimSizes.push_back({outDim, getOutDimSize(outDim)});
1195:   }
```

- **EN:** Defines `LinearLayout::removeZeroBasesAlongDim`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `LinearLayout::removeZeroBasesAlongDim`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 1196-1199

```cpp
1196:   auto newLayout = LinearLayout(std::move(result), ArrayRef(newOutDimSizes),
1197:                                 this->isSurjective());
1198:   return newLayout;
1199: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1201-1202

```cpp
1201: size_t hash_value(const LinearLayout &layout) {
1202:   size_t seed = 0;
```

- **EN:** Defines `hash_value`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `hash_value`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1204-1207

```cpp
1204:   // Hash the bases
1205:   for (const auto &base : layout.getBases()) {
1206:     // Hash the input dimension name
1207:     seed = llvm::hash_combine(seed, base.first);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1209-1215

```cpp
1209:     // Hash the vectors in bases
1210:     for (const auto &vec : base.second) {
1211:       for (int32_t val : vec) {
1212:         seed = llvm::hash_combine(seed, val);
1213:       }
1214:     }
1215:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1217-1223

```cpp
1217:   // Hash the output dimensions and their sizes
1218:   for (const auto &outDim : layout.getOutDimNames()) {
1219:     seed = llvm::hash_combine(seed, outDim, layout.getOutDimSize(outDim));
1220:   }
1221:   // Don't hash the surjective flag as it's a cached property
1222:   return seed;
1223: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1225-1227

```cpp
1225: bool operator==(const LinearLayout &lhs, const LinearLayout &rhs) {
1226:   if (!lhs.equalIgnoringOutDimSizes(rhs))
1227:     return false;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1229-1235

```cpp
1229:   for (const auto &[lhsOutDimAndSize, rhsOutDimAndSize] :
1230:        llvm::zip(lhs.outDims, rhs.outDims)) {
1231:     if (lhsOutDimAndSize.second != rhsOutDimAndSize.second)
1232:       return false;
1233:   }
1234:   return true;
1235: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1237-1250

```cpp
1237: bool LinearLayout::equalIgnoringOutDimSizes(const LinearLayout &other) const {
1238:   // llvm::MapVector doesn't have an operator== :(.
1239:   if (llvm::to_vector(this->getOutDimNames()) !=
1240:       llvm::to_vector(other.getOutDimNames()))
1241:     return false;
1242:   if (this->bases.size() != other.bases.size())
1243:     return false;
1244:   for (auto it1 = this->bases.begin(), it2 = other.bases.begin();
1245:        it1 != this->bases.end(); ++it1, ++it2) {
1246:     if (*it1 != *it2)
1247:       return false;
1248:   }
1249:   return true;
1250: }
```

- **EN:** Defines `LinearLayout::equalIgnoringOutDimSizes`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `LinearLayout::equalIgnoringOutDimSizes`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1252-1264

```cpp
1252: std::string LinearLayout::toString() const {
1253:   // Start with a newline because we print out a bulleted list; it doesn't
1254:   // make sense for the first line of this list to be on the same line as
1255:   // any previous text.
1256:   std::string ret = "\n";
1257:   std::string outDimsStr =
1258:       "[" +
1259:       join(outDims, ", ",
1260:            [](auto dimAndSize) {
1261:              auto [outDim, size] = dimAndSize;
1262:              return outDim.str() + " (size " + std::to_string(size) + ")";
1263:            }) +
1264:       "]";
```

- **EN:** Defines `LinearLayout::toString`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `LinearLayout::toString`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1266-1272

```cpp
1266:   if (bases.empty()) {
1267:     if (outDims.empty()) {
1268:       return "\n(empty layout)";
1269:     } else {
1270:       return "\n(empty layout with out-dims " + outDimsStr + ")";
1271:     }
1272:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1274-1279

```cpp
1274:   // TODO: Add spaces for alignment.
1275:   for (const auto &[inDim, inDimBases] : bases) {
1276:     if (inDimBases.empty()) {
1277:       ret += " - " + inDim.str() + " is a size 1 dimension\n";
1278:       continue;
1279:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1281-1291

```cpp
1281:     ret += " - " +
1282:            join(llvm::seq(inDimBases.size()), "\n   ",
1283:                 [&, &inDim = inDim, &inDimBases = inDimBases](int i) {
1284:                   return inDim.str() + "=" + std::to_string(1 << i) + " -> (" +
1285:                          join(inDimBases[i], ", ") + ")";
1286:                 }) +
1287:            "\n";
1288:   }
1289:   ret += "where out dims are: " + outDimsStr;
1290:   return ret;
1291: }
```

- **EN:** Defines `join` to merge information from multiple analysis states into one conservative result.
- **CN:** 这里定义 `join`，把多个分析状态合并为一个保守的结果。
### Lines 1293-1299

```cpp
1293: LinearLayout ColumnAction::apply(const LinearLayout &layout) const {
1294:   assert(layout.hasInDim(inDim));
1295:   assert(layout.getInDimSizeLog2(inDim) == inSizeLog2 &&
1296:          "Layout has a different size than the ColumnAction");
1297:   if (m_isIdentity) {
1298:     return layout;
1299:   }
```

- **EN:** Defines `ColumnAction::apply`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `ColumnAction::apply`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 1301-1308

```cpp
1301:   auto bases = layout.getBases();
1302:   const auto &basesInDim = bases[inDim];
1303:   std::vector<std::vector<int32_t>> newBases;
1304:   newBases.reserve(action.size());
1305:   for (size_t a : action) {
1306:     newBases.push_back(basesInDim[a]);
1307:   }
1308:   bases[inDim] = std::move(newBases);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1310-1316

```cpp
1310:   SmallVector<std::pair<StringAttr, int32_t>> outDims;
1311:   for (auto outDim : layout.getOutDimNames()) {
1312:     outDims.emplace_back(outDim, layout.getOutDimSize(outDim));
1313:   }
1314:   return LinearLayout(std::move(bases), std::move(outDims),
1315:                       /*requireSurjective=*/false);
1316: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 1318-1334

```cpp
1318: SmallVector<Value> ColumnAction::apply(ValueRange values) const {
1319:   assert(values.size() == (1 << inSizeLog2) &&
1320:          "Values have a different size than the ColumnAction");
1321:   assert(inDim.str() == "register" && "Values are in registers, so we can only "
1322:                                       "apply ColumnAction to registers");
1323:   if (m_isIdentity) {
1324:     return values;
1325:   }
1326:   auto permLL = apply(LinearLayout::identity1D(values.size(), inDim, inDim));
1327:   SmallVector<Value> ret;
1328:   ret.reserve(permLL.getInDimSize(inDim));
1329:   for (int i = 0; i < permLL.getInDimSize(inDim); i++) {
1330:     int32_t srcIdx = permLL.apply({{inDim, i}}).begin()->second;
1331:     ret.push_back(values[srcIdx]);
1332:   }
1333:   return ret;
1334: }
```

- **EN:** Defines `ColumnAction::apply`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `ColumnAction::apply`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 1336-1345

```cpp
1336: ColumnAction ColumnAction::leftCompose(const ColumnAction &other) const {
1337:   assert(inDim == other.inDim);
1338:   assert(inSizeLog2 == other.inSizeLog2);
1339:   assert(action.size() == other.action.size());
1340:   auto newAction = SmallVector<size_t>(action.size());
1341:   for (size_t i = 0; i < action.size(); i++) {
1342:     newAction[i] = action[other.action[i]];
1343:   }
1344:   return ColumnAction(newAction, inDim, inSizeLog2);
1345: }
```

- **EN:** Defines `ColumnAction::leftCompose`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `ColumnAction::leftCompose`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 1347-1353

```cpp
1347: ColumnAction ColumnAction::inverse() const {
1348:   auto invPerm = SmallVector<size_t>(action.size());
1349:   for (size_t i = 0; i < action.size(); i++) {
1350:     invPerm[action[i]] = i;
1351:   }
1352:   return ColumnAction(invPerm, inDim, inSizeLog2);
1353: }
```

- **EN:** Defines `ColumnAction::inverse`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `ColumnAction::inverse`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 1355-1360

```cpp
1355: std::string ColumnAction::toString() const {
1356:   std::string ret = "ColumnAction([";
1357:   ret += join(action, ", ");
1358:   ret += "], " + inDim.str() + ", " + std::to_string(inSizeLog2) + ")";
1359:   return ret;
1360: }
```

- **EN:** Defines `ColumnAction::toString`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `ColumnAction::toString`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1362-1369

```cpp
1362: // Build a matrix of size sum(outDimSizeLog2) x sum(inDimSizeLog2) representing
1363: // the bases of the given layout.  This can then be used by f2reduce.
1364: //
1365: // This function is called from the constructor of LinearLayout, so be careful
1366: // not to use any functions that create LLs in here.
1367: std::unique_ptr<uint64_t[]> getMatrix(const LinearLayout &layout) {
1368:   int numRows = layout.getTotalOutDimSizeLog2();
1369:   int numCols = layout.getTotalInDimSizeLog2();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1371-1374

```cpp
1371:   // Don't handle giant LLs.  This makes some things easier; for example, each
1372:   // row can be a single uint64_t.
1373:   assert(numCols <= 64 && "LinearLayout too large");
1374:   assert(numRows <= 64 && "LinearLayout too large");
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 1376-1393

```cpp
1376:   // Suppose we have a layout specified by the following values.
1377:   //
1378:   //   L(0,1) = (0b01, 0b1)
1379:   //   L(0,2) = (0b10, 0b0)
1380:   //   L(1,0) = (0b10, 0b0)
1381:   //   L(2,0) = (0b11, 0b0)
1382:   //
1383:   // We will create one column per entry above.  The max bit width of the
1384:   // codomain is (2,1), so our matrix will have 2+1=3 rows.  The final matrix
1385:   // will be
1386:   //
1387:   //  | L(0,1)[0] L(0,2)[0] L(1,0)[0] L(2,0)[0] |   | 0b1001 |
1388:   //  |    ↓         ↓         ↓         ↓      |   | 0b0111 |
1389:   //  | L(0,1)[1] L(0,2)[1] L(1,0)[1] L(2,0)[1] | = | 0b1000 |
1390:   //  |    ↓         ↓         ↓         ↓      |
1391:   //
1392:   // Note `new uint64_t[n]()` is zero-initialized, but `new uint64_t[n]` is not.
1393:   std::unique_ptr<uint64_t[]> m(new uint64_t[numRows]());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1394-1407

```cpp
1394:   int r = 0;
1395:   for (StringAttr outDim : layout.getOutDimNames()) {
1396:     int c = 0;
1397:     for (StringAttr inDim : layout.getInDimNames()) {
1398:       for (int i = 0; i < layout.getInDimSizeLog2(inDim); i++) {
1399:         uint64_t basis = layout.getBasis(inDim, i, outDim);
1400:         for (int j = 0; j < layout.getOutDimSizeLog2(outDim); j++) {
1401:           m[r + j] |= ((basis >> j) & 1) << c;
1402:         }
1403:         c++;
1404:       }
1405:     }
1406:     r += layout.getOutDimSizeLog2(outDim);
1407:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1409-1410

```cpp
1409:   return m;
1410: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1412-1412

```cpp
1412: } // namespace mlir::triton
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。

## Key Concepts / 关键概念
- **EN:** The file packages reusable support around linear layout.
  **CN:** 该文件围绕 Linear Layout 封装了可复用的支持逻辑。
- **EN:** Linear layout utilities translate between logical tensor coordinates and physical placement.
  **CN:** 线性布局工具负责在逻辑张量坐标与物理放置之间进行转换。
- **EN:** Layout and encoding decisions are first-class because they determine how work and memory are distributed on the GPU.
  **CN:** 布局与编码决策是第一类问题，因为它们决定了 GPU 上工作与内存的分布方式。

## Dependencies / 依赖关系
- **Internal Triton headers / Triton 内部头文件:** `triton/Tools/LinearLayout.h`, `triton/Tools/LayoutUtils.h`, `triton/Tools/StrUtil.h`
- **MLIR headers / MLIR 头文件:** `mlir/IR/BuiltinAttributes.h`
- **LLVM headers / LLVM 头文件:** `llvm/ADT/STLExtras.h`, `llvm/ADT/SetOperations.h`, `llvm/ADT/StringRef.h`, `llvm/Support/Debug.h`, `llvm/Support/ErrorHandling.h`, `llvm/Support/MathExtras.h`
- **Standard/library headers / 标准或通用库头文件:** `cstdint`, `set`, `vector`, `third_party/f2reduce/f2reduce.h`, `intrin.h`
- **Primary APIs used / 主要 API:** `LinearLayout`
