# Dialect.h — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./include/triton/Dialect/TritonGPU/IR/Dialect.h`
- **EN:** Declares the MLIR dialect interface and imports generated declarations for this subsystem.
- **CN:** 声明该子系统的 MLIR 方言接口，并引入生成的声明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```cpp
   1: #ifndef TRITON_DIALECT_TRITONGPU_IR_DIALECT_H_
   2: #define TRITON_DIALECT_TRITONGPU_IR_DIALECT_H_
```
**EN:** This block establishes the header guard so the declarations in this file are only processed once per translation unit.
**CN:** 该代码块建立头文件保护宏，避免同一翻译单元重复包含本文件时出现重复声明。

### Lines 4-8
```cpp
   4: #include "mlir/Dialect/GPU/IR/GPUDialect.h"
   5: #include "mlir/Dialect/Tensor/IR/Tensor.h"
   6: #include "mlir/IR/BuiltinOps.h"
   7: #include "mlir/IR/Dialect.h"
   8: #include "mlir/Interfaces/SideEffectInterfaces.h"
```
**EN:** This block imports the direct dependencies needed here, including mlir/Dialect/GPU/IR/GPUDialect.h, mlir/Dialect/Tensor/IR/Tensor.h, mlir/IR/BuiltinOps.h, mlir/IR/Dialect.h, and mlir/Interfaces/SideEffectInterfaces.h.
**CN:** 该代码块引入后续声明直接依赖的头文件，例如 mlir/Dialect/GPU/IR/GPUDialect.h, mlir/Dialect/Tensor/IR/Tensor.h, mlir/IR/BuiltinOps.h, mlir/IR/Dialect.h, and mlir/Interfaces/SideEffectInterfaces.h。

### Lines 10-14
```cpp
  10: // TritonGPU depends on Triton
  11: #include "triton/Dialect/Triton/IR/Dialect.h"
  12: #include "triton/Dialect/TritonGPU/IR/Attributes.h"
  13: #include "triton/Dialect/TritonGPU/IR/Traits.h"
  14: #include "triton/Dialect/TritonGPU/IR/Types.h"
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 16-16
```cpp
  16: #include <unordered_map>
```
**EN:** This block imports the direct dependencies needed here, including <unordered_map>.
**CN:** 该代码块引入后续声明直接依赖的头文件，例如 <unordered_map>。

### Lines 18-21
```cpp
  18: namespace mlir {
  19: class AsmParser;
  20: class AsmPrinter;
  21: } // namespace mlir
```
**EN:** This block stores supporting state such as AsmParser and AsmPrinter, which other APIs in the file consume.
**CN:** 该代码块声明了 AsmParser and AsmPrinter 等支撑状态，供本文件中的其他 API 使用。

### Lines 23-24
```cpp
  23: // LinearLayoutCache Utils
  24: using CacheKey = std::tuple<std::vector<int64_t>, mlir::Attribute>;
```
**EN:** This block introduces type aliases such as using, CacheKey, std::tuple<std::vector<int64_t, and mlir::Attribute to simplify later declarations.
**CN:** 该代码块引入了 using, CacheKey, std::tuple<std::vector<int64_t, and mlir::Attribute 等类型别名，以简化后续声明。

### Lines 26-30
```cpp
  26: namespace llvm {
  27: template <typename T> size_t hash_value(const std::vector<T> &vec) {
  28:   return hash_combine_range(vec.begin(), vec.end());
  29: }
  30: } // namespace llvm
```
**EN:** This block declares or defines callable APIs such as hash_value, hash_combine_range, begin, and end, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 hash_value, hash_combine_range, begin, and end 等可调用 API，用来封装这里提供的核心行为。

### Lines 32-45
```cpp
  32: namespace std {
  33: template <> struct hash<CacheKey> {
  34:   size_t operator()(const CacheKey &key) const noexcept {
  35:     using llvm::hash_value;
  36:     size_t seed = 0;
  37:     std::apply(
  38:         [&seed](const auto &...elems) {
  39:           ((seed = llvm::hash_combine(seed, hash_value(elems))), ...);
  40:         },
  41:         key);
  42:     return seed;
  43:   }
  44: };
  45: } // namespace std
```
**EN:** This block declares or defines callable APIs such as operator, apply, hash_combine, and hash_value, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 operator, apply, hash_combine, and hash_value 等可调用 API，用来封装这里提供的核心行为。

### Lines 47-47
```cpp
  47: namespace mlir::triton::gpu {
```
**EN:** This block opens or closes the namespace scope used to organize APIs under mlir::triton::gpu.
**CN:** 该代码块打开或关闭命名空间作用域，用来把相关 API 组织在 mlir::triton::gpu 下。

### Lines 49-53
```cpp
  49: constexpr static char AttrMaxRegistersName[] = "ttg.maxnreg";
  50: constexpr static char AttrNumWarpsName[] = "ttg.num-warps";
  51: constexpr static char AttrNumCTAsName[] = "ttg.num-ctas";
  52: constexpr static char AttrTargetName[] = "ttg.target";
  53: constexpr static char AttrNumThreadsPerWarp[] = "ttg.threads-per-warp";
```
**EN:** This block defines named compile-time constants such as AttrMaxRegistersName, ttg, maxnreg, AttrNumWarpsName, ttg, and num.
**CN:** 该代码块定义了 AttrMaxRegistersName, ttg, maxnreg, AttrNumWarpsName, ttg, and num 等具名编译期常量。

### Lines 55-61
```cpp
  55: // Find the contextual number of warps on which this operation is executed.
  56: int lookupNumWarps(Operation *op);
  57: int lookupNumWarps(Region *region);
  58: // Try to find the contextual number of warps on which this operation is
  59: // executed. Returns nullopt if a warp size cannot be find. This is used for
  60: // verifiers.
  61: std::optional<int> maybeLookupNumWarps(Operation *op);
```
**EN:** This block declares or defines callable APIs such as lookupNumWarps and maybeLookupNumWarps, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 lookupNumWarps and maybeLookupNumWarps 等可调用 API，用来封装这里提供的核心行为。

### Lines 63-67
```cpp
  63: // FIXME: Make this API and that of maybeLookupNumWarps consistent!
  64: // Utility to find the number of threads per warp
  65: int lookupThreadsPerWarp(OpBuilder &rewriter);
  66: int lookupNumCTAs(OpBuilder &rewriter);
  67: int lookupNumCTAs(Operation *op);
```
**EN:** This block declares or defines callable APIs such as lookupThreadsPerWarp and lookupNumCTAs, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 lookupThreadsPerWarp and lookupNumCTAs 等可调用 API，用来封装这里提供的核心行为。

### Lines 69-78
```cpp
  69: template <typename Key, typename Value> class Cache {
  70: public:
  71:   std::optional<Value> get(const Key &key) {
  72:     std::shared_lock lock(mutex);
  73:     auto it = cache.find(key);
  74:     if (it != cache.end()) {
  75:       return it->second;
  76:     }
  77:     return std::nullopt;
  78:   }
```
**EN:** This block introduces `Cache`, the main class/struct defined here. Within the declaration, methods such as get, lock, find, and end expose its core API.
**CN:** 该代码块引入了此文件的核心类/结构体 `Cache`。 其中 get, lock, find, and end 等方法构成了它的主要接口。

### Lines 80-83
```cpp
  80:   void set(Key key, Value result) {
  81:     std::scoped_lock lock(mutex);
  82:     cache.emplace(std::move(key), std::move(result));
  83:   }
```
**EN:** This block declares or defines callable APIs such as set, lock, emplace, and move, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 set, lock, emplace, and move 等可调用 API，用来封装这里提供的核心行为。

### Lines 85-88
```cpp
  85: private:
  86:   std::unordered_map<Key, Value> cache;
  87:   llvm::sys::SmartRWMutex<true> mutex;
  88: };
```
**EN:** This block stores supporting state such as cache and mutex, which other APIs in the file consume.
**CN:** 该代码块声明了 cache and mutex 等支撑状态，供本文件中的其他 API 使用。

### Lines 90-92
```cpp
  90: using LinearLayoutCache = Cache<CacheKey, LinearLayout>;
  91: using LinearEncodingCache = Cache<CacheKey, LinearEncodingAttr>;
  92: } // namespace mlir::triton::gpu
```
**EN:** This block introduces type aliases such as using, LinearLayoutCache, Cache<CacheKey, and LinearLayout to simplify later declarations.
**CN:** 该代码块引入了 using, LinearLayoutCache, Cache<CacheKey, and LinearLayout 等类型别名，以简化后续声明。

### Lines 94-96
```cpp
  94: #define GET_OP_CLASSES
  95: #include "triton/Dialect/TritonGPU/IR/Dialect.h.inc"
  96: #include "triton/Dialect/TritonGPU/IR/Ops.h.inc"
```
**EN:** This block enables MLIR/TableGen-generated declarations and then includes the generated `.inc` fragments that materialize them.
**CN:** 该代码块先打开 MLIR/TableGen 生成代码所需的宏开关，再引入真正展开声明的 `.inc` 片段。

### Lines 98-102
```cpp
  98: namespace mlir::triton::gpu {
  99: struct SharedMemory : public SideEffects::Resource::Base<SharedMemory> {
 100:   StringRef getName() const final { return "<SharedMemory>"; }
 101:   SideEffects::Resource *getParent() const override { return nullptr; }
 102: };
```
**EN:** This block declares or defines callable APIs such as getName and getParent, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getName and getParent 等可调用 API，用来封装这里提供的核心行为。

### Lines 104-106
```cpp
 104: // Returns true iff every non-broadcast basis of `ll`, after flattening in and
 105: // out dimensions, maps to a single power-of-2 in the flattened output.
 106: bool hasPowerOfTwoBases(const LinearLayout &ll);
```
**EN:** This block declares or defines callable APIs such as hasPowerOfTwoBases, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 hasPowerOfTwoBases 等可调用 API，用来封装这里提供的核心行为。

### Lines 108-111
```cpp
 108: // Check whether after removing broadcast bases the flattened layout is a
 109: // permutation matrix (each non-broadcast basis maps to a distinct power-of-2
 110: // and the remaining layout is bijective).
 111: bool isPermutationMatrixLayout(const LinearLayout &ll);
```
**EN:** This block declares or defines callable APIs such as isPermutationMatrixLayout, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isPermutationMatrixLayout 等可调用 API，用来封装这里提供的核心行为。

### Lines 113-115
```cpp
 113: // Returns whether the attribute is a GenericLinearEncoding, a WMMA with warp
 114: // swizzling or a slice or DotOp of one of these.
 115: bool isGenericLinearEncoding(Attribute attr);
```
**EN:** This block declares or defines callable APIs such as isGenericLinearEncoding, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isGenericLinearEncoding 等可调用 API，用来封装这里提供的核心行为。

### Lines 117-120
```cpp
 117: // Create a GenericLinearEncoding if the source isGenericLinearEncoding, and a
 118: // LinearEncoding otherwise.
 119: Attribute inferEncodingFromLinearLayout(MLIRContext *ctx, LinearLayout ll,
 120:                                         Attribute srcEnc);
```
**EN:** This block declares or defines callable APIs such as inferEncodingFromLinearLayout, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 inferEncodingFromLinearLayout 等可调用 API，用来封装这里提供的核心行为。

### Lines 122-125
```cpp
 122: // Convert a distributed layout to a linear encoding
 123: LinearEncodingAttr toLinearEncoding(RankedTensorType type);
 124: LinearEncodingAttr toLinearEncoding(DistributedEncodingTrait layout,
 125:                                     ArrayRef<int64_t> shape);
```
**EN:** This block declares or defines callable APIs such as toLinearEncoding, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 toLinearEncoding 等可调用 API，用来封装这里提供的核心行为。

### Lines 127-131
```cpp
 127: // Convert a distributed layout to a generic linear encoding
 128: GenericLinearEncodingAttr toGenericLinearEncoding(RankedTensorType type);
 129: GenericLinearEncodingAttr
 130: toGenericLinearEncoding(DistributedEncodingTrait layout,
 131:                         ArrayRef<int64_t> shape);
```
**EN:** This block declares or defines callable APIs such as toGenericLinearEncoding, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 toGenericLinearEncoding 等可调用 API，用来封装这里提供的核心行为。

### Lines 133-133
```cpp
 133: unsigned getTotalElemsPerThread(Type type);
```
**EN:** This block declares or defines callable APIs such as getTotalElemsPerThread, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getTotalElemsPerThread 等可调用 API，用来封装这里提供的核心行为。

### Lines 135-135
```cpp
 135: unsigned getTotalElemsPerThread(Attribute layout, ArrayRef<int64_t> shape);
```
**EN:** This block declares or defines callable APIs such as getTotalElemsPerThread, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getTotalElemsPerThread 等可调用 API，用来封装这里提供的核心行为。

### Lines 137-139
```cpp
 137: // Get the number of elements in each thread, ignoring register broadcasting.
 138: unsigned getUniqueElemsPerThread(Type type);
 139: unsigned getUniqueElemsPerThread(Attribute layout, ArrayRef<int64_t> shape);
```
**EN:** This block declares or defines callable APIs such as getUniqueElemsPerThread, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getUniqueElemsPerThread 等可调用 API，用来封装这里提供的核心行为。

### Lines 141-141
```cpp
 141: SmallVector<unsigned> getElemsPerThread(Type type);
```
**EN:** This block declares or defines callable APIs such as getElemsPerThread, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getElemsPerThread 等可调用 API，用来封装这里提供的核心行为。

### Lines 143-145
```cpp
 143: FailureOr<RankedTensorType> inferFp4ToFpResultType(RankedTensorType srcType,
 144:                                                    Type elemType, int32_t axis,
 145:                                                    std::optional<Location> loc);
```
**EN:** This block declares or defines callable APIs such as inferFp4ToFpResultType, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 inferFp4ToFpResultType 等可调用 API，用来封装这里提供的核心行为。

### Lines 147-156
```cpp
 147: // Returns the number of warps per CTA that have access to non-replicated
 148: // elements of the tensor. E.g. for a blocked layout with sizePerThread = [1,
 149: // 1], threadsPerWarp = [2, 16], warpsPerCTA = [1, 4] and tensor shape = [2, 2],
 150: // returns [1, 1], since the first warp has access to the full tensor, whereas
 151: // the other warps have access to replicated elements.
 152: SmallVector<unsigned> getWarpsPerCTA(Attribute layout,
 153:                                      ArrayRef<int64_t> tensorShape);
 154: inline SmallVector<unsigned> getWarpsPerCTA(RankedTensorType type) {
 155:   return getWarpsPerCTA(type.getEncoding(), type.getShape());
 156: }
```
**EN:** This block declares or defines callable APIs such as getWarpsPerCTA, getEncoding, and getShape, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getWarpsPerCTA, getEncoding, and getShape 等可调用 API，用来封装这里提供的核心行为。

### Lines 158-164
```cpp
 158: // Returns the number of contiguous elements of the logical tensor that each
 159: // thread has access to, on each dimension of the tensor. For a blocked layout
 160: // with sizePerThread = [1, 4] and tensor shape = [128, 1], the elements
 161: // for thread 0 would be [A_{0, 0}, A_{0, 0}, A_{0, 0}, A_{0, 0}], returns [1,
 162: // 1]. Whereas for a tensor shape [128, 128], the elements for thread 0 would be
 163: // [A_{0, 0}, A_{0, 1}, A_{0, 2}, A_{0, 3}], returns [1, 4].
 164: SmallVector<unsigned> getContigPerThread(RankedTensorType tensorType);
```
**EN:** This block declares or defines callable APIs such as getContigPerThread, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getContigPerThread 等可调用 API，用来封装这里提供的核心行为。

### Lines 166-175
```cpp
 166: // Returns the number of threads per warp that have access to non-replicated
 167: // elements of the tensor. E.g. for a blocked layout with sizePerThread = [1,
 168: // 1], threadsPerWarp = [2, 16] and tensor shape = [2, 2], threads 0, 1, 16, 17
 169: // have access to the full tensor, whereas the other threads have access to
 170: // replicated elements, so this function returns [2, 2].
 171: SmallVector<unsigned> getThreadsPerWarp(Attribute layout,
 172:                                         ArrayRef<int64_t> shape);
 173: inline SmallVector<unsigned> getThreadsPerWarp(RankedTensorType type) {
 174:   return getThreadsPerWarp(type.getEncoding(), type.getShape());
 175: }
```
**EN:** This block declares or defines callable APIs such as getThreadsPerWarp, getEncoding, and getShape, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getThreadsPerWarp, getEncoding, and getShape 等可调用 API，用来封装这里提供的核心行为。

### Lines 177-187
```cpp
 177: // Returns the dimensions of the tensor from minor (fast-varying) to
 178: // major (slow-varying). For distributed layouts, this represents
 179: // the order of the elements within a thread.
 180: // For shared Layout, the order refers to which dimension of the original tensor
 181: // is contiguous in shared memory.
 182: SmallVector<unsigned> getOrder(DistributedEncodingTrait layout,
 183:                                ArrayRef<int64_t> shape);
 184: inline SmallVector<unsigned> getOrder(RankedTensorType type) {
 185:   return getOrder(cast<DistributedEncodingTrait>(type.getEncoding()),
 186:                   type.getShape());
 187: }
```
**EN:** This block declares or defines callable APIs such as getOrder, getEncoding, and getShape, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getOrder, getEncoding, and getShape 等可调用 API，用来封装这里提供的核心行为。

### Lines 189-202
```cpp
 189: SmallVector<unsigned> getOrder(SharedEncodingTrait layout,
 190:                                ArrayRef<int64_t> shape);
 191: inline SmallVector<unsigned> getOrder(MemDescType type) {
 192:   return getOrder(cast<SharedEncodingTrait>(type.getEncoding()),
 193:                   type.getShape());
 194: }
 195: inline SmallVector<unsigned> getOrder(TensorOrMemDesc type) {
 196:   if (auto memDesc = dyn_cast<MemDescType>(type)) {
 197:     return getOrder(memDesc);
 198:   } else {
 199:     auto tensorTy = cast<RankedTensorType>(type);
 200:     return getOrder(tensorTy);
 201:   }
 202: }
```
**EN:** This block declares or defines callable APIs such as getOrder, getEncoding, and getShape, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getOrder, getEncoding, and getShape 等可调用 API，用来封装这里提供的核心行为。

### Lines 204-221
```cpp
 204: // To be removed once we implement arbitrary swizzled layouts
 205: // It chooses heuristically an order for the memory layout in which to save
 206: // a distributed layout taking into account the order of the elements
 207: // and the threads.
 208: SmallVector<unsigned> getOrderForMemory(DistributedEncodingTrait layout,
 209:                                         ArrayRef<int64_t> shape);
 210: inline SmallVector<unsigned> getOrderForMemory(RankedTensorType type) {
 211:   return getOrderForMemory(cast<DistributedEncodingTrait>(type.getEncoding()),
 212:                            type.getShape());
 213: }
 214: inline SmallVector<unsigned> getOrderForMemory(TensorOrMemDesc type) {
 215:   if (auto memDesc = dyn_cast<MemDescType>(type)) {
 216:     return getOrder(memDesc);
 217:   } else {
 218:     auto tensorTy = cast<RankedTensorType>(type);
 219:     return getOrderForMemory(tensorTy);
 220:   }
 221: }
```
**EN:** This block declares or defines callable APIs such as getOrderForMemory, getEncoding, getShape, and getOrder, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getOrderForMemory, getEncoding, getShape, and getOrder 等可调用 API，用来封装这里提供的核心行为。

### Lines 223-235
```cpp
 223: // Returns the dimensions along which warpId's are distributed.
 224: // warpsPerCTA only tells the warp layout in the CTA, e.g. warpsPerCTA = [2, 4]
 225: // tells there are 2 warps along dim0 and 4 warps along dim1.
 226: // warpOrder tells the specific order when distributing warp IDs.
 227: // E.g. warpOrder = [0, 1] means the warp IDs are distributed as follows
 228: // [warp0  warp2  warp4 warp6]
 229: // [warp1  warp3  warp5 warp7]
 230: SmallVector<unsigned> getWarpOrder(DistributedEncodingTrait layout,
 231:                                    ArrayRef<int64_t> shape);
 232: inline SmallVector<unsigned> getWarpOrder(RankedTensorType type) {
 233:   return getWarpOrder(cast<DistributedEncodingTrait>(type.getEncoding()),
 234:                       type.getShape());
 235: }
```
**EN:** This block declares or defines callable APIs such as getWarpOrder, getEncoding, and getShape, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getWarpOrder, getEncoding, and getShape 等可调用 API，用来封装这里提供的核心行为。

### Lines 237-245
```cpp
 237: // Returns the dimensions along which threadId's are distributed.
 238: // Similar to warpOrder, threadOrder is necessary to tell the specific thread
 239: // distribution in the warp.
 240: SmallVector<unsigned> getThreadOrder(DistributedEncodingTrait layout,
 241:                                      ArrayRef<int64_t> shape);
 242: inline SmallVector<unsigned> getThreadOrder(RankedTensorType type) {
 243:   return getThreadOrder(cast<DistributedEncodingTrait>(type.getEncoding()),
 244:                         type.getShape());
 245: }
```
**EN:** This block declares or defines callable APIs such as getThreadOrder, getEncoding, and getShape, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getThreadOrder, getEncoding, and getShape 等可调用 API，用来封装这里提供的核心行为。

### Lines 247-248
```cpp
 247: std::optional<CGAEncodingAttr> parseCGAAttr(AsmParser &parser, Attribute attr,
 248:                                             unsigned rank);
```
**EN:** This block declares or defines callable APIs such as parseCGAAttr, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 parseCGAAttr 等可调用 API，用来封装这里提供的核心行为。

### Lines 250-250
```cpp
 250: void printCGAAttr(AsmPrinter &printer, CGAEncodingAttr layout);
```
**EN:** This block declares or defines callable APIs such as printCGAAttr, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 printCGAAttr 等可调用 API，用来封装这里提供的核心行为。

### Lines 252-252
```cpp
 252: CGAEncodingAttr getCGALayout(Attribute layout);
```
**EN:** This block declares or defines callable APIs such as getCGALayout, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getCGALayout 等可调用 API，用来封装这里提供的核心行为。

### Lines 254-254
```cpp
 254: SmallVector<unsigned> getCTAsPerCGA(Attribute layout);
```
**EN:** This block declares or defines callable APIs such as getCTAsPerCGA, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getCTAsPerCGA 等可调用 API，用来封装这里提供的核心行为。

### Lines 256-256
```cpp
 256: SmallVector<unsigned> getCTASplitNum(Attribute layout);
```
**EN:** This block declares or defines callable APIs such as getCTASplitNum, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getCTASplitNum 等可调用 API，用来封装这里提供的核心行为。

### Lines 258-258
```cpp
 258: SmallVector<unsigned> getCTAOrder(Attribute layout);
```
**EN:** This block declares or defines callable APIs such as getCTAOrder, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getCTAOrder 等可调用 API，用来封装这里提供的核心行为。

### Lines 260-270
```cpp
 260: // Returns the "logical" shape per CTA.
 261: // When shape and CTASplitNum have different number of dimensions, we assume
 262: // only the last N between common dimensions are split.
 263: // Example1: shape = [2, 4, 8], CTASplitNum = [2, 2], ret = [2, 2, 4].
 264: // It can be caused by pipelining.
 265: // Example2: shape = [2, 4], CTASplitNum = [2, 2, 2], ret = [1, 2].
 266: // It can be caused by memory slicing.
 267: SmallVector<int64_t> getShapePerCTA(ArrayRef<unsigned> CTASplitNum,
 268:                                     ArrayRef<int64_t> shape);
 269: SmallVector<int64_t> getShapePerCTA(Attribute layout, ArrayRef<int64_t> shape);
 270: SmallVector<int64_t> getShapePerCTA(Type type);
```
**EN:** This block declares or defines callable APIs such as getShapePerCTA, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getShapePerCTA 等可调用 API，用来封装这里提供的核心行为。

### Lines 272-277
```cpp
 272: // Returns the shape per CTA, which is "physically" allocated.
 273: // Such shapes may be bigger than the logical one due to, for example, padding
 274: // in shared memory.
 275: SmallVector<int64_t> getAllocationShapePerCTA(Attribute layout,
 276:                                               ArrayRef<int64_t> shape);
 277: SmallVector<int64_t> getAllocationShapePerCTA(Type type);
```
**EN:** This block declares or defines callable APIs such as getAllocationShapePerCTA, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getAllocationShapePerCTA 等可调用 API，用来封装这里提供的核心行为。

### Lines 279-279
```cpp
 279: unsigned getNumCTAs(Attribute layout);
```
**EN:** This block declares or defines callable APIs such as getNumCTAs, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getNumCTAs 等可调用 API，用来封装这里提供的核心行为。

### Lines 281-284
```cpp
 281: // Return the order that represents that the batch is in row-major or
 282: // column-major order for a batch of matrices of shape [*, m, n] with
 283: // len(shape) == rank.
 284: SmallVector<unsigned> getMatrixOrder(unsigned rank, bool rowMajor);
```
**EN:** This block declares or defines callable APIs such as getMatrixOrder, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getMatrixOrder 等可调用 API，用来封装这里提供的核心行为。

### Lines 286-290
```cpp
 286: // Return the order that represents that the dot operand is in kContig
 287: // (contiguous in the inner dimension) or it's contiguous on the outer
 288: // dimension.
 289: SmallVector<unsigned> getOrderForDotOperand(unsigned opIdx, unsigned rank,
 290:                                             bool kContig);
```
**EN:** This block declares or defines callable APIs such as getOrderForDotOperand, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getOrderForDotOperand 等可调用 API，用来封装这里提供的核心行为。

### Lines 292-293
```cpp
 292: // Return true if \p cat would be valid with result encoding \p targetEncoding.
 293: bool isLegalCatEncoding(CatOp cat, Attribute targetEncoding);
```
**EN:** This block declares or defines callable APIs such as isLegalCatEncoding, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isLegalCatEncoding 等可调用 API，用来封装这里提供的核心行为。

### Lines 295-303
```cpp
 295: // Return true if a view between the two types cannot be implemented as a no-op.
 296: bool isExpensiveView(ArrayRef<int64_t> srcShape, Attribute srcEncoding,
 297:                      ArrayRef<int64_t> dstShape, Attribute dstEncoding);
 298: inline bool isExpensiveView(Type srcType, Type dstType) {
 299:   auto tensorSrcType = cast<RankedTensorType>(srcType);
 300:   auto tensorDstType = cast<RankedTensorType>(dstType);
 301:   return isExpensiveView(tensorSrcType.getShape(), tensorSrcType.getEncoding(),
 302:                          tensorDstType.getShape(), tensorDstType.getEncoding());
 303: }
```
**EN:** This block declares or defines callable APIs such as isExpensiveView, getShape, and getEncoding, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isExpensiveView, getShape, and getEncoding 等可调用 API，用来封装这里提供的核心行为。

### Lines 305-309
```cpp
 305: // Return a blocked encoding where the shape is distributed contiguously amongst
 306: // the threads, warps, CTAs with 1 element per threads.
 307: triton::gpu::BlockedEncodingAttr
 308: getDefaultBlockedEncoding(MLIRContext *context, ArrayRef<int64_t> shape,
 309:                           int numWarps, int threadsPerWarp, int numCTAs);
```
**EN:** This block declares or defines callable APIs such as getDefaultBlockedEncoding, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getDefaultBlockedEncoding 等可调用 API，用来封装这里提供的核心行为。

### Lines 311-313
```cpp
 311: // Dump information about which threads/registers contain each of the tensor
 312: // elements.
 313: void dumpLayout(RankedTensorType tensorType);
```
**EN:** This block declares or defines callable APIs such as dumpLayout, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 dumpLayout 等可调用 API，用来封装这里提供的核心行为。

### Lines 315-317
```cpp
 315: // Dump the layout from HW point of view and prints what tensor element is held
 316: // by each thread and register.
 317: void dumpHWLayout(RankedTensorType tensorType);
```
**EN:** This block declares or defines callable APIs such as dumpHWLayout, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 dumpHWLayout 等可调用 API，用来封装这里提供的核心行为。

### Lines 319-320
```cpp
 319: // Return a string representation of the layout of the tensor.
 320: std::string getLayoutStr(RankedTensorType tensorType, bool useHWPointOfView);
```
**EN:** This block declares or defines callable APIs such as getLayoutStr, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getLayoutStr 等可调用 API，用来封装这里提供的核心行为。

### Lines 322-323
```cpp
 322: // Return a string representation of the shared layout of the tensor.
 323: std::string getSharedLayoutStr(LinearLayout &ll, bool useHWPointOfView);
```
**EN:** This block declares or defines callable APIs such as getSharedLayoutStr, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getSharedLayoutStr 等可调用 API，用来封装这里提供的核心行为。

### Lines 325-326
```cpp
 325: // Return a string representation of the distributed layout of the tensor.
 326: std::string getDistributedLayoutStr(LinearLayout &ll, bool useHWPointOfView);
```
**EN:** This block declares or defines callable APIs such as getDistributedLayoutStr, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getDistributedLayoutStr 等可调用 API，用来封装这里提供的核心行为。

### Lines 328-329
```cpp
 328: template <typename T>
 329: llvm::SmallVector<T> expandMatrixShapeWithBatch(llvm::ArrayRef<T> s);
```
**EN:** This block declares or defines callable APIs such as expandMatrixShapeWithBatch, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 expandMatrixShapeWithBatch 等可调用 API，用来封装这里提供的核心行为。

### Lines 331-332
```cpp
 331: llvm::SmallVector<unsigned>
 332: expandMatrixOrderWithBatch(llvm::ArrayRef<unsigned> o);
```
**EN:** This block declares or defines callable APIs such as expandMatrixOrderWithBatch, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 expandMatrixOrderWithBatch 等可调用 API，用来封装这里提供的核心行为。

### Lines 334-336
```cpp
 334: // Return true if the two layouts represent the exact same mapping.
 335: bool areLayoutsEquivalent(ArrayRef<int64_t> shape, LayoutEncodingTrait lhs,
 336:                           LayoutEncodingTrait rhs);
```
**EN:** This block declares or defines callable APIs such as areLayoutsEquivalent, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 areLayoutsEquivalent 等可调用 API，用来封装这里提供的核心行为。

### Lines 338-339
```cpp
 338: // Return true if the innermost numElems are contiguous.
 339: bool isInnermostContiguous(MemDescType type, unsigned numElems);
```
**EN:** This block declares or defines callable APIs such as isInnermostContiguous, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isInnermostContiguous 等可调用 API，用来封装这里提供的核心行为。

### Lines 341-342
```cpp
 341: LinearLayout inferReshapeLinearLayout(TensorOrMemDesc srcTy,
 342:                                       ArrayRef<int64_t> dstShape);
```
**EN:** This block declares or defines callable APIs such as inferReshapeLinearLayout, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 inferReshapeLinearLayout 等可调用 API，用来封装这里提供的核心行为。

### Lines 344-348
```cpp
 344: // TMA tensor access modes
 345: enum class TMAMode {
 346:   Tiled, // Regular tiled tensor memory access
 347:   Im2Col // Im2col mode for convolution-friendly access patterns
 348: };
```
**EN:** This block defines an enumeration covering values such as TMA, tensor, access, modes, TMAMode, and Tiled.
**CN:** 该代码块定义了一个枚举类型，覆盖了 TMA, tensor, access, modes, TMAMode, and Tiled 等取值。

### Lines 350-358
```cpp
 350: FailureOr<SmallVector<int64_t>>
 351: getTMABlockShape(ArrayRef<int64_t> shapePerCTA, int elementBitWidth,
 352:                  int swizzleBytes, bool fp4Padded, bool isTransposed,
 353:                  bool packedSize, function_ref<InFlightDiagnostic()> emitError,
 354:                  TMAMode mode);
 355: SmallVector<int64_t> getTMABlockShape(ArrayRef<int64_t> shapePerCTA,
 356:                                       int elementBitWidth, int swizzleBytes,
 357:                                       bool fp4Padded, bool isTransposed,
 358:                                       bool packedSize, TMAMode mode);
```
**EN:** This block declares or defines callable APIs such as getTMABlockShape and InFlightDiagnostic, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getTMABlockShape and InFlightDiagnostic 等可调用 API，用来封装这里提供的核心行为。

### Lines 360-366
```cpp
 360: // Verify the types of operations that operate on memory.
 361: LogicalResult verifyMemoryOpTypes(Operation *op, ShapedType srcTy,
 362:                                   ShapedType dstTy);
 363: // Verify a memory allocation operation.
 364: LogicalResult verifyAllocOp(Operation *op, Value src, MemDescType dstTy);
 365: /// Returns the size in bytes of a scalar type when stored in shared memory.
 366: size_t getSharedMemorySize(Type type);
```
**EN:** This block declares or defines callable APIs such as verifyMemoryOpTypes, verifyAllocOp, and getSharedMemorySize, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 verifyMemoryOpTypes, verifyAllocOp, and getSharedMemorySize 等可调用 API，用来封装这里提供的核心行为。

### Lines 368-371
```cpp
 368: // Extract the PaddedSharedEncodingAttr from an encoding, whether standalone
 369: // or wrapped inside a PartitionedSharedEncodingAttr. Returns nullptr if the
 370: // encoding does not involve padding.
 371: PaddedSharedEncodingAttr getPaddedEncoding(Attribute encoding);
```
**EN:** This block declares or defines callable APIs such as getPaddedEncoding, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getPaddedEncoding 等可调用 API，用来封装这里提供的核心行为。

### Lines 373-375
```cpp
 373: // Returns true if the encoding is a PaddedSharedEncodingAttr, or a
 374: // PartitionedSharedEncodingAttr wrapping a PaddedSharedEncodingAttr.
 375: bool isPaddedEncoding(Attribute encoding);
```
**EN:** This block declares or defines callable APIs such as isPaddedEncoding, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isPaddedEncoding 等可调用 API，用来封装这里提供的核心行为。

### Lines 377-379
```cpp
 377: // Returns the minInterval for a padded encoding (standalone or
 378: // wrapped in partitioned).
 379: unsigned getMinInterval(Attribute encoding);
```
**EN:** This block declares or defines callable APIs such as getMinInterval, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getMinInterval 等可调用 API，用来封装这里提供的核心行为。

### Lines 381-381
```cpp
 381: } // namespace mlir::triton::gpu
```
**EN:** This block manages the namespace scope for the surrounding declarations.
**CN:** 该代码块负责管理周围声明所属的命名空间作用域。

### Lines 383-383
```cpp
 383: #endif // TRITON_DIALECT_TRITONGPU_IR_DIALECT_H_
```
**EN:** This block closes the header guard started at the top of the file.
**CN:** 该代码块结束文件顶部开始的头文件保护宏。

## Key Concepts / 关键概念
- **EN:** layout conversion and reasoning  
  **CN:** 布局转换与推理
- **EN:** linear layout algebra  
  **CN:** 线性布局代数
- **EN:** layout encodings  
  **CN:** 布局编码
- **EN:** warp-level execution  
  **CN:** warp 级执行
- **EN:** matrix-multiply acceleration  
  **CN:** 矩阵乘加加速
- **EN:** dot-product lowering  
  **CN:** 点积降级
- **EN:** LLVM lowering  
  **CN:** LLVM 降级
- **EN:** MLIR dialect definition  
  **CN:** MLIR 方言定义

## Dependencies / 依赖关系
- **Project includes / 项目内依赖:**
  - `mlir/Dialect/GPU/IR/GPUDialect.h`
  - `mlir/Dialect/Tensor/IR/Tensor.h`
  - `mlir/IR/BuiltinOps.h`
  - `mlir/IR/Dialect.h`
  - `mlir/Interfaces/SideEffectInterfaces.h`
  - `triton/Dialect/Triton/IR/Dialect.h`
  - `triton/Dialect/TritonGPU/IR/Attributes.h`
  - `triton/Dialect/TritonGPU/IR/Traits.h`
  - `triton/Dialect/TritonGPU/IR/Types.h`
- **System or external includes / 系统或外部依赖:**
  - `<unordered_map>`
- **Generated includes / 生成代码依赖:**
  - `triton/Dialect/TritonGPU/IR/Dialect.h.inc`
  - `triton/Dialect/TritonGPU/IR/Ops.h.inc`
