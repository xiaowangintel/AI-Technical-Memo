# LinearLayoutConversions.h — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./include/triton/Dialect/TritonGPU/IR/LinearLayoutConversions.h`
- **EN:** Declares APIs centered on `LinearLayoutConversions` inside Triton.
- **CN:** 声明 Triton 中围绕 `LinearLayoutConversions` 的 API。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```cpp
   1: // Conversions from TritonGPU layouts (e.g. BlockedEncodingAttr) to
   2: // LinearLayout.
```
**EN:** This comment block records the intent and constraints of the surrounding code: Conversions from TritonGPU layouts (e.g. BlockedEncodingAttr) to LinearLayout.
**CN:** 该注释块解释了周边代码的设计背景、意图或约束条件，帮助读者理解后续实现。

### Lines 4-5
```cpp
   4: #ifndef TRITON_DIALECT_TRITONGPU_IR_LINEARLAYOUTCONVERSIONS_H
   5: #define TRITON_DIALECT_TRITONGPU_IR_LINEARLAYOUTCONVERSIONS_H
```
**EN:** This block establishes the header guard so the declarations in this file are only processed once per translation unit.
**CN:** 该代码块建立头文件保护宏，避免同一翻译单元重复包含本文件时出现重复声明。

### Lines 7-7
```cpp
   7: #include <optional>
```
**EN:** This block imports the direct dependencies needed here, including <optional>.
**CN:** 该代码块引入后续声明直接依赖的头文件，例如 <optional>。

### Lines 9-9
```cpp
   9: #include "triton/Tools/LinearLayout.h"
```
**EN:** This block imports the direct dependencies needed here, including triton/Tools/LinearLayout.h.
**CN:** 该代码块引入后续声明直接依赖的头文件，例如 triton/Tools/LinearLayout.h。

### Lines 11-13
```cpp
  11: namespace mlir::triton {
  12: enum class ScaleDotElemType : uint32_t;
  13: } // namespace mlir::triton
```
**EN:** This block stores supporting state such as uint32_t, which other APIs in the file consume.
**CN:** 该代码块声明了 uint32_t 等支撑状态，供本文件中的其他 API 使用。

### Lines 15-21
```cpp
  15: namespace mlir::triton::gpu {
  16: class SwizzledSharedEncodingAttr;
  17: class NVMMASharedEncodingAttr;
  18: class TensorOrMemDesc;
  19: class MemDescType;
  20: class CGAEncodingAttr;
  21: enum class TMAMode;
```
**EN:** This block stores supporting state such as SwizzledSharedEncodingAttr, NVMMASharedEncodingAttr, TensorOrMemDesc, MemDescType, CGAEncodingAttr, and TMAMode, which other APIs in the file consume.
**CN:** 该代码块声明了 SwizzledSharedEncodingAttr, NVMMASharedEncodingAttr, TensorOrMemDesc, MemDescType, CGAEncodingAttr, and TMAMode 等支撑状态，供本文件中的其他 API 使用。

### Lines 23-56
```cpp
  23: // - BlockedEncodingAttrs have the following input dimensions.
  24: //
  25: //   "register": elements in one thread
  26: //   "lane": threads in a warp
  27: //   "warp": warps in a block/CTA
  28: //   "block": blocks in a cluster
  29: //
  30: // - An n-dimensional SwizzledSharedEncodingAttr has the following input
  31: // dimensions.
  32: //
  33: //   "offset": the n'th element in the allocation, within a particular thread
  34: //      block (i.e. within a CTA).  The offset is measured in elements, not
  35: //      bytes.
  36: //   "block": blocks in a cluster
  37: //
  38: // All layouts have the following output dimensions.
  39: //
  40: //  "dimi" for i in 0..n-1: the location in the n'th logical dimension of the
  41: //  output tensor.  These also are not reordered according to the layout's
  42: //  `order`.
  43: //
  44: // You can flatten the input or output dimensions into a single dimension using
  45: // LinearLayout::flattenIns/Outs().
  46: //
  47: // elemBitWidth is the bit width of one element in the layout.  This is required
  48: // to compute the linear layout for MMAv3 (i.e. Hopper) shared layouts (i.e.
  49: // shared layouts with nvmma_shared layout) but is otherwise unused.
  50: LinearLayout toLinearLayout(RankedTensorType type);
  51: LinearLayout toLinearLayout(MemDescType type);
  52: LinearLayout toLinearLayout(TensorOrMemDesc type);
  53: // UNSAFE OVERLOAD!
  54: // If you call this with a SharedMemoryEncodingAttr, you should call it
  55: // with the allocShape as the shape, otherwise the layout will be incorrect!
  56: LinearLayout toLinearLayout(ArrayRef<int64_t> shape, Attribute layout);
```
**EN:** This block declares or defines callable APIs such as toLinearLayout, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 toLinearLayout 等可调用 API，用来封装这里提供的核心行为。

### Lines 58-64
```cpp
  58: // Returns the linear component of a padded shared encoding. The encoding must
  59: // satisfy isPaddedEncoding (asserts otherwise).
  60: //
  61: // Unlike toLinearLayout, this makes explicit that the resulting linear layout
  62: // is incomplete — the padding information is not captured in the linear layout.
  63: LinearLayout paddedLinearLayout(MemDescType type);
  64: LinearLayout paddedLinearLayout(ArrayRef<int64_t> shape, Attribute encoding);
```
**EN:** This block declares or defines callable APIs such as paddedLinearLayout, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 paddedLinearLayout 等可调用 API，用来封装这里提供的核心行为。

### Lines 66-78
```cpp
  66: // Convert the shared encoding of a tensor with `nvmma_shared` layout to a
  67: // LinearLayout that maps from a linear shared memory offset to tensor index.
  68: //
  69: // If `disableSwizzle` is set, then the resulting layout does not include
  70: // swizzling.
  71: LinearLayout nvmmaSharedToLinearLayout(ArrayRef<int64_t> shape,
  72:                                        NVMMASharedEncodingAttr shared,
  73:                                        TMAMode mode,
  74:                                        bool disableSwizzle = false);
  75: FailureOr<LinearLayout>
  76: nvmmaSharedToLinearLayout(ArrayRef<int64_t> shape,
  77:                           NVMMASharedEncodingAttr shared, TMAMode mode,
  78:                           bool disableSwizzle, bool emitErrors);
```
**EN:** This block declares or defines callable APIs such as nvmmaSharedToLinearLayout, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 nvmmaSharedToLinearLayout 等可调用 API，用来封装这里提供的核心行为。

### Lines 80-87
```cpp
  80: // Given a linear layout where the input dimensions contain a "block" dimension,
  81: // this method sets the "block" dimension to 0 and removes the corresponding
  82: // output dimensions.
  83: //
  84: // Note that this behavior differs from calling
  85: // `LinearLayout::sublayout(inDimNames, outDimNames)` when "block" is not in
  86: // `inDimNames`. The latter does not modify the output sizes.
  87: LinearLayout getLayoutWithinBlock(const LinearLayout &layout);
```
**EN:** This block declares or defines callable APIs such as getLayoutWithinBlock, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getLayoutWithinBlock 等可调用 API，用来封装这里提供的核心行为。

### Lines 89-97
```cpp
  89: // Combines the layout of a CTA (input dims [register, lane, warp]) with the
  90: // layout of a CGA (i.e. a block), and ensures that the resulting layout has the
  91: // given shape.
  92: //
  93: // See the nomenclature note at the top of LinearLayoutConversions.cpp for why
  94: // the variable with type CGAEncodingAttr is called cgaLayoutAttr.
  95: LinearLayout combineCtaCgaWithShape(LinearLayout ctaLayout,
  96:                                     CGAEncodingAttr cgaLayoutAttr,
  97:                                     ArrayRef<int64_t> shape);
```
**EN:** This block declares or defines callable APIs such as combineCtaCgaWithShape, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 combineCtaCgaWithShape 等可调用 API，用来封装这里提供的核心行为。

### Lines 99-128
```cpp
  99: LinearLayout chooseWmmaCTALinearLayout(MLIRContext *ctx, unsigned rank,
 100:                                        ArrayRef<unsigned> warpsPerCTA,
 101:                                        ArrayRef<unsigned> tilesPerWarp);
 102: // In this function, we construct a linear layout representing the
 103: // <shared memory offset, iteration, block> -> <tensor element index> mapping
 104: // for entire `src` and `dst` tensors.  We determine the shape of the
 105: // intermediate shared memory buffer needed for a register-to-register
 106: // conversion using the maximum size accessed in each dimension from `src`'s
 107: // layout and `dst`'s layout.  See the getRepShapeForCvt function in
 108: // Allocation.cpp for details. Note that the buffer might be smaller than the
 109: // tensor being converted, so we need multiple "iterations" to move a subregion
 110: // of the `src` tensor to the corresponding subregion of the `dst` tensor.  The
 111: // pesudo code of layout conversion is as follows:
 112: //
 113: // for iter in 0..numIterations:
 114: //   sync threads
 115: //   for vecIdx in [0..numRegisters/storeVec]:
 116: //     registers <- get registers used in iter
 117: //     offsets <- get offsets using the intermediate linear layout
 118: //     store registers[vecIdx * storeVec, (vecIdx + 1) * storeVec)] to shared
 119: //     memory
 120: //   sync threads
 121: //   for vecIdx in [0..numRegisters/loadVec]:
 122: //     registers <- get registers used in iter
 123: //     offsets <- get offsets using the intermediate linear layout
 124: //     load registers[vecIdx * loadVec, (vecIdx + 1) * loadVec)] from shared
 125: //     memory
 126: LinearLayout chooseShemLayoutForRegToRegConversion(
 127:     MLIRContext *ctx, ArrayRef<unsigned> tensorShape,
 128:     ArrayRef<unsigned> repShape, ArrayRef<unsigned> order);
```
**EN:** This block declares or defines callable APIs such as chooseWmmaCTALinearLayout and chooseShemLayoutForRegToRegConversion, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 chooseWmmaCTALinearLayout and chooseShemLayoutForRegToRegConversion 等可调用 API，用来封装这里提供的核心行为。

### Lines 130-135
```cpp
 130: // The primary goal of this function is to efficiently load 2D tiles of a
 131: // tensor from shared memory using the `ds_read_tr` instruction for AMD GPUs.
 132: std::optional<LinearLayout>
 133: chooseDsReadTrLayout(Attribute enc, ArrayRef<int64_t> shape,
 134:                      int32_t elemBitWidth, unsigned instBitWidth,
 135:                      unsigned numLanesInShuffleGroup);
```
**EN:** This block declares or defines callable APIs such as chooseDsReadTrLayout, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 chooseDsReadTrLayout 等可调用 API，用来封装这里提供的核心行为。

### Lines 137-142
```cpp
 137: // Create LinearLayout for scale in scaled mfma.
 138: LinearLayout chooseScaledMfmaScaleLayout(MLIRContext *ctx, int dotOperandIdx,
 139:                                          ArrayRef<int64_t> dotOperandShape,
 140:                                          unsigned mfmaMDim,
 141:                                          ArrayRef<unsigned> tilesPerWarp,
 142:                                          ArrayRef<unsigned> warpsPerCTA);
```
**EN:** This block declares or defines callable APIs such as chooseScaledMfmaScaleLayout, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 chooseScaledMfmaScaleLayout 等可调用 API，用来封装这里提供的核心行为。

### Lines 144-147
```cpp
 144: LinearLayout chooseScaledWmmaScaleLayout(
 145:     MLIRContext *ctx, int dotOperandIdx, ArrayRef<int64_t> dotOperandShape,
 146:     unsigned wmmaMDim, unsigned wmmaNDim, bool isTransposed,
 147:     unsigned scaleFactor, LinearLayout ctaLayout, CGAEncodingAttr cgaLayout);
```
**EN:** This block declares or defines callable APIs such as chooseScaledWmmaScaleLayout, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 chooseScaledWmmaScaleLayout 等可调用 API，用来封装这里提供的核心行为。

### Lines 149-152
```cpp
 149: LinearLayout getSM120DotScaledScaleLayout(MLIRContext *ctx,
 150:                                           ArrayRef<int64_t> shape, int opIdx,
 151:                                           ArrayRef<unsigned> warpsPerCTA,
 152:                                           CGAEncodingAttr cgaLayout);
```
**EN:** This block declares or defines callable APIs such as getSM120DotScaledScaleLayout, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getSM120DotScaledScaleLayout 等可调用 API，用来封装这里提供的核心行为。

### Lines 154-157
```cpp
 154: // Create LinearLayout for nvidia mma tile.
 155: LinearLayout nvidiaMmaTile(MLIRContext *ctx, ArrayRef<unsigned> tileShape,
 156:                            unsigned kWidth, ArrayRef<unsigned> order,
 157:                            ArrayRef<unsigned> repOrder);
```
**EN:** This block declares or defines callable APIs such as nvidiaMmaTile, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 nvidiaMmaTile 等可调用 API，用来封装这里提供的核心行为。

### Lines 159-163
```cpp
 159: // Create a LinearLayout similar to mfmaLayout, but changing each thread to hold
 160: // 8 elements. This layout is useful for emitting the widest 128-bit global
 161: // store instructions. Since it closely resembles mfmaLayout, conversion between
 162: // the two can be done using transferWithinWarp, without involving LDS
 163: std::optional<LinearLayout> chooseMfmaLikeStoreLayout(RankedTensorType valType);
```
**EN:** This block declares or defines callable APIs such as chooseMfmaLikeStoreLayout, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 chooseMfmaLikeStoreLayout 等可调用 API，用来封装这里提供的核心行为。

### Lines 165-167
```cpp
 165: // Create the core layout (atom in the PTX manual) a given nvmma shared encoding
 166: LinearLayout getCoreMatrixLinearLayout(NVMMASharedEncodingAttr shared,
 167:                                        bool disableSwizzle);
```
**EN:** This block declares or defines callable APIs such as getCoreMatrixLinearLayout, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getCoreMatrixLinearLayout 等可调用 API，用来封装这里提供的核心行为。

### Lines 169-185
```cpp
 169: // Create a TDM (Tensor DMA) LinearLayout: (message, warp, block) ->
 170: // (dim0, dim1, ...).  TDM is warp-granular.  The "warp" sublayout is an
 171: // identity over `warpsPerCTA`, zero-padded up to log2(numWarps) so the
 172: // full module warpId is covered; padded rows expose the redundant bits
 173: // as free variables (via getFreeVariableMasks("warp")) so partial copies
 174: // (K < numWarps) can pred-off inactive warps.  "message" covers the
 175: // per-warp tile (surjectivity); "block" comes from `cgaLayout`.
 176: //
 177: // `warpUsedHint`: power-of-two-popcount bitmask whose K = popcount(hint)
 178: // set bits select active warps.  The varying warpId bit positions in the
 179: // active set are the warp bits that contribute to per-warp offsets; all
 180: // other warpId bits become free variables for predicating inactive warps.
 181: // Empty = no-hint default (lowest log2(K) bits, K = prod(warpsPerCTA)).
 182: LinearLayout getTDMLinearLayout(ArrayRef<int64_t> blockShape,
 183:                                 ArrayRef<unsigned> warpsPerCTA,
 184:                                 const LinearLayout &cgaLayout, int totalWarps,
 185:                                 std::optional<uint32_t> warpUsedHint = {});
```
**EN:** This block declares or defines callable APIs such as getTDMLinearLayout, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getTDMLinearLayout 等可调用 API，用来封装这里提供的核心行为。

### Lines 187-188
```cpp
 187: } // namespace mlir::triton::gpu
 188: #endif // TRITON_DIALECT_TRITONGPU_IR_LINEARLAYOUTCONVERSIONS_H
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

## Key Concepts / 关键概念
- **EN:** shared-memory allocation  
  **CN:** 共享内存分配
- **EN:** buffer modeling  
  **CN:** 缓冲区建模
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

## Dependencies / 依赖关系
- **Project includes / 项目内依赖:**
  - `triton/Tools/LinearLayout.h`
- **System or external includes / 系统或外部依赖:**
  - `<optional>`
