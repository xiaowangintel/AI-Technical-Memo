# Utility.h — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./include/triton/Dialect/Triton/IR/Utility.h`
- **EN:** Provides helper utilities that support transformations or analysis in this area.
- **CN:** 提供支撑该领域分析或变换的辅助工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```cpp
   1: #ifndef TRITON_IR_UTILITY_H_
   2: #define TRITON_IR_UTILITY_H_
```
**EN:** This block establishes the header guard so the declarations in this file are only processed once per translation unit.
**CN:** 该代码块建立头文件保护宏，避免同一翻译单元重复包含本文件时出现重复声明。

### Lines 4-8
```cpp
   4: #include "mlir/Dialect/LLVMIR/LLVMDialect.h"
   5: #include "mlir/IR/BuiltinTypes.h"
   6: #include "triton/Dialect/Triton/IR/Dialect.h"
   7: #include <algorithm>
   8: #include <numeric>
```
**EN:** This block imports the direct dependencies needed here, including mlir/Dialect/LLVMIR/LLVMDialect.h, mlir/IR/BuiltinTypes.h, triton/Dialect/Triton/IR/Dialect.h, <algorithm>, and <numeric>.
**CN:** 该代码块引入后续声明直接依赖的头文件，例如 mlir/Dialect/LLVMIR/LLVMDialect.h, mlir/IR/BuiltinTypes.h, triton/Dialect/Triton/IR/Dialect.h, <algorithm>, and <numeric>。

### Lines 10-10
```cpp
  10: namespace mlir {
```
**EN:** This block opens or closes the namespace scope used to organize APIs under mlir.
**CN:** 该代码块打开或关闭命名空间作用域，用来把相关 API 组织在 mlir 下。

### Lines 12-13
```cpp
  12: // Bitwidth of pointers
  13: constexpr int kPtrBitWidth = 64;
```
**EN:** This block defines named compile-time constants such as Bitwidth, of, pointers, int, and kPtrBitWidth.
**CN:** 该代码块定义了 Bitwidth, of, pointers, int, and kPtrBitWidth 等具名编译期常量。

### Lines 15-21
```cpp
  15: // Returns the bit width of a type, treating pointer-like types as 64-bit.
  16: // This handles LLVM dialect pointer types.
  17: inline int getIntOrFloatOrPtrBitWidth(Type type) {
  18:   if (isa<LLVM::LLVMPointerType, triton::PointerType>(type))
  19:     return kPtrBitWidth;
  20:   return type.getIntOrFloatBitWidth();
  21: }
```
**EN:** This block declares or defines callable APIs such as getIntOrFloatOrPtrBitWidth and getIntOrFloatBitWidth, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getIntOrFloatOrPtrBitWidth and getIntOrFloatBitWidth 等可调用 API，用来封装这里提供的核心行为。

### Lines 23-28
```cpp
  23: template <typename T, typename U> SmallVector<T> convertType(ArrayRef<U> in) {
  24:   SmallVector<T> out;
  25:   for (const auto &i : in)
  26:     out.push_back(T(i));
  27:   return out;
  28: }
```
**EN:** This block declares or defines callable APIs such as convertType, push_back, and T, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 convertType, push_back, and T 等可调用 API，用来封装这里提供的核心行为。

### Lines 30-33
```cpp
  30: template <typename T, typename VecU>
  31: SmallVector<T> convertType(const VecU &in) {
  32:   return convertType<T>(ArrayRef(in));
  33: }
```
**EN:** This block declares or defines callable APIs such as convertType and ArrayRef, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 convertType and ArrayRef 等可调用 API，用来封装这里提供的核心行为。

### Lines 35-40
```cpp
  35: template <typename Int> Int product(llvm::ArrayRef<Int> arr) {
  36:   return std::accumulate(arr.begin(), arr.end(), 1, std::multiplies<Int>());
  37: }
  38: template <typename VecT> auto product(const VecT &vec) {
  39:   return product(llvm::ArrayRef(vec));
  40: }
```
**EN:** This block declares or defines callable APIs such as product, accumulate, begin, end, and ArrayRef, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 product, accumulate, begin, end, and ArrayRef 等可调用 API，用来封装这里提供的核心行为。

### Lines 42-43
```cpp
  42: // TODO(jlebar): Rename to ceilOfRatio.
  43: template <typename Int> Int ceil(Int m, Int n) { return (m + n - 1) / n; }
```
**EN:** This block declares or defines callable APIs such as ceil, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 ceil 等可调用 API，用来封装这里提供的核心行为。

### Lines 45-55
```cpp
  45: /// Get the highest power of 2 divisor of an integer.
  46: template <typename T> constexpr T highestPowOf2Divisor(T n) {
  47:   // When n is 0 or min, return the highest power of 2. The min case is handled
  48:   // separately to avoid underflow when T is a signed integer. Technically
  49:   // in that case the correct divisor is -n, but this value is outside the
  50:   // range of possible values, so we take the next best alternative.
  51:   if (n == 0 || n == std::numeric_limits<T>::min()) {
  52:     return (static_cast<T>(1) << (sizeof(T) * 8 - 2));
  53:   }
  54:   return (n & (~(n - 1)));
  55: }
```
**EN:** This block declares or defines callable APIs such as highestPowOf2Divisor and ~, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 highestPowOf2Divisor and ~ 等可调用 API，用来封装这里提供的核心行为。

### Lines 57-68
```cpp
  57: /// Get the next power of 2 for an integer (or the integer itself if it is a
  58: /// power of 2).
  59: template <typename T> T nextPowOf2(T n) {
  60:   if (n == 0) {
  61:     return 1;
  62:   }
  63:   n--;
  64:   for (unsigned i = 1; i < sizeof(T) * 8; i <<= 1) {
  65:     n |= n >> i;
  66:   }
  67:   return n + 1;
  68: }
```
**EN:** This block declares or defines callable APIs such as nextPowOf2, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 nextPowOf2 等可调用 API，用来封装这里提供的核心行为。

### Lines 70-70
```cpp
  70: namespace triton {
```
**EN:** This block opens or closes the namespace scope used to organize APIs under triton.
**CN:** 该代码块打开或关闭命名空间作用域，用来把相关 API 组织在 triton 下。

### Lines 72-75
```cpp
  72: // Many functions here have two overloads, fn(ArrayRef<T>) and fn(const VecT&).
  73: // This is helpful because C++ won't both convert a vector to ArrayRef *and*
  74: // infer the proper type T in one step.  So without the second overload, we
  75: // would have to explicitly convert most arguments to ArrayRef at the callsite.
```
**EN:** This comment block records the intent and constraints of the surrounding code: Many functions here have two overloads, fn(ArrayRef<T>) and fn(const VecT&). This is helpful because C++ won't both convert a vector to ArrayRef *and* infer the proper type T in....
**CN:** 该注释块解释了周边代码的设计背景、意图或约束条件，帮助读者理解后续实现。

### Lines 77-80
```cpp
  77: template <typename T, typename U>
  78: SmallVector<T> applyPermutation(ArrayRef<T> vec, ArrayRef<U> permutation) {
  79:   static_assert(std::is_integral_v<U>);
  80:   assert(vec.size() == permutation.size());
```
**EN:** This block declares or defines callable APIs such as applyPermutation, static_assert, and size, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 applyPermutation, static_assert, and size 等可调用 API，用来封装这里提供的核心行为。

### Lines 82-89
```cpp
  82:   // Check that `permutation` is actually a permutation.
  83: #ifndef NDEBUG
  84:   SmallVector<U> sortedPerm(permutation);
  85:   llvm::sort(sortedPerm);
  86:   for (U i = 0; i < static_cast<U>(sortedPerm.size()); i++) {
  87:     assert(sortedPerm[i] == i);
  88:   }
  89: #endif
```
**EN:** This block declares or defines callable APIs such as sortedPerm, sort, and size, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 sortedPerm, sort, and size 等可调用 API，用来封装这里提供的核心行为。

### Lines 91-97
```cpp
  91:   SmallVector<T> ret;
  92:   ret.reserve(vec.size());
  93:   for (const U &i : permutation) {
  94:     ret.push_back(vec[i]);
  95:   }
  96:   return ret;
  97: }
```
**EN:** This block declares or defines callable APIs such as reserve, size, and push_back, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 reserve, size, and push_back 等可调用 API，用来封装这里提供的核心行为。

### Lines 99-102
```cpp
  99: template <typename VecT, typename PermT>
 100: auto applyPermutation(const VecT &vec, const PermT &permutation) {
 101:   return applyPermutation(ArrayRef(vec), ArrayRef(permutation));
 102: }
```
**EN:** This block declares or defines callable APIs such as applyPermutation and ArrayRef, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 applyPermutation and ArrayRef 等可调用 API，用来封装这里提供的核心行为。

### Lines 104-113
```cpp
 104: template <typename T>
 105: [[nodiscard]] SmallVector<T> inversePermutation(ArrayRef<T> permutation) {
 106:   // Check that `permutation` is actually a permutation.
 107: #ifndef NDEBUG
 108:   SmallVector<T> sortedPerm(permutation);
 109:   llvm::sort(sortedPerm);
 110:   for (int i = 0; i < sortedPerm.size(); ++i) {
 111:     assert(sortedPerm[i] == i);
 112:   }
 113: #endif
```
**EN:** This block declares or defines callable APIs such as inversePermutation, sortedPerm, sort, and size, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 inversePermutation, sortedPerm, sort, and size 等可调用 API，用来封装这里提供的核心行为。

### Lines 115-120
```cpp
 115:   SmallVector<T> ret(permutation.size());
 116:   for (int i = 0; i < permutation.size(); ++i) {
 117:     ret[permutation[i]] = i;
 118:   }
 119:   return ret;
 120: }
```
**EN:** This block declares or defines callable APIs such as ret and size, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 ret and size 等可调用 API，用来封装这里提供的核心行为。

### Lines 122-125
```cpp
 122: template <typename VecT>
 123: [[nodiscard]] auto inversePermutation(const VecT &permutation) {
 124:   return inversePermutation(ArrayRef(permutation));
 125: }
```
**EN:** This block declares or defines callable APIs such as inversePermutation and ArrayRef, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 inversePermutation and ArrayRef 等可调用 API，用来封装这里提供的核心行为。

### Lines 127-135
```cpp
 127: template <typename T, typename U>
 128: [[nodiscard]] SmallVector<T> gather(ArrayRef<T> elems, ArrayRef<U> indices) {
 129:   SmallVector<T> ret;
 130:   ret.reserve(indices.size());
 131:   for (const U &i : indices) {
 132:     ret.push_back(elems[i]);
 133:   }
 134:   return ret;
 135: }
```
**EN:** This block declares or defines callable APIs such as gather, reserve, size, and push_back, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 gather, reserve, size, and push_back 等可调用 API，用来封装这里提供的核心行为。

### Lines 137-140
```cpp
 137: template <typename VecT, typename IdxT>
 138: [[nodiscard]] auto gather(const VecT &elems, const IdxT &indices) {
 139:   return gather(ArrayRef(elems), ArrayRef(indices));
 140: }
```
**EN:** This block declares or defines callable APIs such as gather and ArrayRef, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 gather and ArrayRef 等可调用 API，用来封装这里提供的核心行为。

### Lines 142-151
```cpp
 142: // Is `vec` [0, 1, ..., n]?  Returns true on empty list.
 143: template <typename T> bool isIota(ArrayRef<T> vec) {
 144:   static_assert(std::is_integral_v<T>);
 145:   for (size_t i = 0; i < vec.size(); ++i) {
 146:     if (vec[i] != static_cast<T>(i)) {
 147:       return false;
 148:     }
 149:   }
 150:   return true;
 151: }
```
**EN:** This block declares or defines callable APIs such as isIota, static_assert, and size, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isIota, static_assert, and size 等可调用 API，用来封装这里提供的核心行为。

### Lines 153-155
```cpp
 153: template <typename VecT> bool isIota(const VecT &vec) {
 154:   return isIota(ArrayRef(vec));
 155: }
```
**EN:** This block declares or defines callable APIs such as isIota and ArrayRef, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isIota and ArrayRef 等可调用 API，用来封装这里提供的核心行为。

### Lines 157-162
```cpp
 157: // Is `vals` some permutation of the numbers 0..(vals.size()-1)?
 158: template <typename T> bool isPermutationOfIota(ArrayRef<T> vals) {
 159:   SmallVector<T> sorted(vals);
 160:   llvm::sort(sorted);
 161:   return isIota(sorted);
 162: }
```
**EN:** This block declares or defines callable APIs such as isPermutationOfIota, sorted, sort, and isIota, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isPermutationOfIota, sorted, sort, and isIota 等可调用 API，用来封装这里提供的核心行为。

### Lines 164-166
```cpp
 164: template <typename VecT> bool isPermutationOfIota(const VecT &vec) {
 165:   return isPermutationOfIota(ArrayRef(vec));
 166: }
```
**EN:** This block declares or defines callable APIs such as isPermutationOfIota and ArrayRef, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isPermutationOfIota and ArrayRef 等可调用 API，用来封装这里提供的核心行为。

### Lines 168-177
```cpp
 168: // Is `vec` [i, i+1, ..., i+n]?  Returns true on empty list.
 169: template <typename T> bool isConsecutive(ArrayRef<T> vec) {
 170:   static_assert(std::is_integral_v<T>);
 171:   for (int i = 1; i < vec.size(); i++) {
 172:     if (vec[i] != vec[i - 1] + 1) {
 173:       return false;
 174:     }
 175:   }
 176:   return true;
 177: }
```
**EN:** This block declares or defines callable APIs such as isConsecutive, static_assert, and size, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isConsecutive, static_assert, and size 等可调用 API，用来封装这里提供的核心行为。

### Lines 179-181
```cpp
 179: template <typename VecT> bool isConsecutive(const VecT &vec) {
 180:   return isConsecutive(ArrayRef(vec));
 181: }
```
**EN:** This block declares or defines callable APIs such as isConsecutive and ArrayRef, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isConsecutive and ArrayRef 等可调用 API，用来封装这里提供的核心行为。

### Lines 183-187
```cpp
 183: template <typename T> auto seq(T start, T end, T step) {
 184:   auto len = ceil<T>(end - start, step);
 185:   return llvm::map_range(llvm::seq<T>(0, len),
 186:                          [=](T i) { return start + i * step; });
 187: }
```
**EN:** This block declares or defines callable APIs such as seq and map_range, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 seq and map_range 等可调用 API，用来封装这里提供的核心行为。

### Lines 189-191
```cpp
 189: // Combine the current mask with the given predicate.
 190: Value getPredMask(RewriterBase &rewriter, Type typeLike, Value currentMask,
 191:                   Value pred);
```
**EN:** This block declares or defines callable APIs such as getPredMask, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getPredMask 等可调用 API，用来封装这里提供的核心行为。

### Lines 193-194
```cpp
 193: // Get the value of the induction variable at the end of the loop.
 194: Value getLastInductionValue(OpBuilder &b, scf::ForOp loop);
```
**EN:** This block declares or defines callable APIs such as getLastInductionValue, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getLastInductionValue 等可调用 API，用来封装这里提供的核心行为。

### Lines 196-196
```cpp
 196: bool isHostSideDescriptor(Value v);
```
**EN:** This block declares or defines callable APIs such as isHostSideDescriptor, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isHostSideDescriptor 等可调用 API，用来封装这里提供的核心行为。

### Lines 198-198
```cpp
 198: bool isKernel(FunctionOpInterface funcOp);
```
**EN:** This block declares or defines callable APIs such as isKernel, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isKernel 等可调用 API，用来封装这里提供的核心行为。

### Lines 200-200
```cpp
 200: unsigned getBitwidth(RankedTensorType ty);
```
**EN:** This block declares or defines callable APIs such as getBitwidth, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getBitwidth 等可调用 API，用来封装这里提供的核心行为。

### Lines 202-207
```cpp
 202: // If the value "anchor" is compared against a statically-computed bound, return
 203: // inclusive lower and upper bounds lb <= anchor <= ub. Depending on the
 204: // comparison operator, one of the bounds is a computed one while the other is
 205: // derived from the data type of anchor.
 206: std::optional<ConstantIntRanges> getBoundFromCmpOp(arith::CmpIOp cmpOp,
 207:                                                    Value anchor);
```
**EN:** This block declares or defines callable APIs such as getBoundFromCmpOp, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getBoundFromCmpOp 等可调用 API，用来封装这里提供的核心行为。

### Lines 209-210
```cpp
 209: } // namespace triton
 210: } // namespace mlir
```
**EN:** This block manages the namespace scope for the surrounding declarations.
**CN:** 该代码块负责管理周围声明所属的命名空间作用域。

### Lines 212-212
```cpp
 212: #endif
```
**EN:** This block closes the header guard started at the top of the file.
**CN:** 该代码块结束文件顶部开始的头文件保护宏。

## Key Concepts / 关键概念
- **EN:** LLVM lowering  
  **CN:** LLVM 降级
- **EN:** MLIR dialect definition  
  **CN:** MLIR 方言定义
- **EN:** types  
  **CN:** 类型
- **EN:** MLIR integration  
  **CN:** MLIR 集成

## Dependencies / 依赖关系
- **Project includes / 项目内依赖:**
  - `mlir/Dialect/LLVMIR/LLVMDialect.h`
  - `mlir/IR/BuiltinTypes.h`
  - `triton/Dialect/Triton/IR/Dialect.h`
- **System or external includes / 系统或外部依赖:**
  - `<algorithm>`
  - `<numeric>`
