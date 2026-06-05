# Utility.h — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./include/triton/Analysis/Utility.h`
- **EN:** Provides helper utilities that support transformations or analysis in this area.
- **CN:** 提供支撑该领域分析或变换的辅助工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```cpp
   1: #ifndef TRITON_ANALYSIS_UTILITY_H
   2: #define TRITON_ANALYSIS_UTILITY_H
```
**EN:** This block establishes the header guard so the declarations in this file are only processed once per translation unit.
**CN:** 该代码块建立头文件保护宏，避免同一翻译单元重复包含本文件时出现重复声明。

### Lines 4-10
```cpp
   4: #include "mlir/Analysis/DataFlowFramework.h"
   5: #include "mlir/Analysis/SliceAnalysis.h"
   6: #include "mlir/IR/Builders.h"
   7: #include "mlir/Support/LLVM.h"
   8: #include "triton/Dialect/Triton/IR/Dialect.h"
   9: #include "triton/Dialect/TritonGPU/IR/Dialect.h"
  10: #include "triton/Tools/LinearLayout.h"
```
**EN:** This block imports the direct dependencies needed here, including mlir/Analysis/DataFlowFramework.h, mlir/Analysis/SliceAnalysis.h, mlir/IR/Builders.h, mlir/Support/LLVM.h, triton/Dialect/Triton/IR/Dialect.h, and triton/Dialect/TritonGPU/IR/Dialect.h.
**CN:** 该代码块引入后续声明直接依赖的头文件，例如 mlir/Analysis/DataFlowFramework.h, mlir/Analysis/SliceAnalysis.h, mlir/IR/Builders.h, mlir/Support/LLVM.h, triton/Dialect/Triton/IR/Dialect.h, and triton/Dialect/TritonGPU/IR/Dialect.h。

### Lines 12-12
```cpp
  12: namespace mlir {
```
**EN:** This block opens or closes the namespace scope used to organize APIs under mlir.
**CN:** 该代码块打开或关闭命名空间作用域，用来把相关 API 组织在 mlir 下。

### Lines 14-24
```cpp
  14: inline bool isZeroConst(Value v) {
  15:   auto constantOp = v.getDefiningOp<arith::ConstantOp>();
  16:   if (!constantOp)
  17:     return false;
  18:   if (auto denseAttr = dyn_cast<DenseFPElementsAttr>(constantOp.getValueAttr()))
  19:     return denseAttr.isSplat() && denseAttr.getSplatValue<APFloat>().isZero();
  20:   if (auto denseAttr =
  21:           dyn_cast<DenseIntElementsAttr>(constantOp.getValueAttr()))
  22:     return denseAttr.isSplat() && denseAttr.getSplatValue<APInt>().isZero();
  23:   return false;
  24: }
```
**EN:** This block declares or defines callable APIs such as isZeroConst, getValueAttr, isSplat, and isZero, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isZeroConst, getValueAttr, isSplat, and isZero 等可调用 API，用来封装这里提供的核心行为。

### Lines 26-42
```cpp
  26: class ReduceOpHelper {
  27: public:
  28:   enum class InThreadVectorizeOpKind {
  29:     None,
  30:     AddF,
  31:     MulF,
  32:     MinNumF,
  33:     MaxNumF,
  34:     MinimumF,
  35:     MaximumF,
  36:     AddI,
  37:     MulI,
  38:     MinSI,
  39:     MaxSI,
  40:     MinUI,
  41:     MaxUI,
  42:   };
```
**EN:** This block introduces `ReduceOpHelper`, the main class/struct defined here.
**CN:** 该代码块引入了此文件的核心类/结构体 `ReduceOpHelper`。

### Lines 44-50
```cpp
  44:   explicit ReduceOpHelper(triton::ReduceOp op)
  45:       : op(op.getOperation()), axis(op.getAxis()) {
  46:     auto firstTy = cast<RankedTensorType>(op.getOperands()[0].getType());
  47:     srcTy = firstTy;
  48:     srcShape = firstTy.getShape();
  49:     srcEncoding = firstTy.getEncoding();
  50:     srcElementTypes = op.getElementTypes();
```
**EN:** This block declares or defines callable APIs such as ReduceOpHelper, op, getOperation, axis, getAxis, getOperands, getType, and getShape, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 ReduceOpHelper, op, getOperation, axis, getAxis, getOperands, getType, and getShape 等可调用 API，用来封装这里提供的核心行为。

### Lines 52-60
```cpp
  52:     for (const auto &t : op.getInputTypes()) {
  53:       if (t.getShape() != srcShape) {
  54:         op.emitError() << "shape mismatch";
  55:       }
  56:       if (t.getEncoding() != srcEncoding) {
  57:         op.emitError() << "encoding mismatch";
  58:       }
  59:     }
  60:   }
```
**EN:** This block declares or defines callable APIs such as getInputTypes, getShape, emitError, and getEncoding, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getInputTypes, getShape, emitError, and getEncoding 等可调用 API，用来封装这里提供的核心行为。

### Lines 62-62
```cpp
  62:   RankedTensorType getSrcTy() { return srcTy; }
```
**EN:** This block declares or defines callable APIs such as getSrcTy, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getSrcTy 等可调用 API，用来封装这里提供的核心行为。

### Lines 64-64
```cpp
  64:   unsigned getInterWarpSizeWithUniqueData();
```
**EN:** This block declares or defines callable APIs such as getInterWarpSizeWithUniqueData, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getInterWarpSizeWithUniqueData 等可调用 API，用来封装这里提供的核心行为。

### Lines 66-66
```cpp
  66:   unsigned getIntraWarpSizeWithUniqueData();
```
**EN:** This block declares or defines callable APIs such as getIntraWarpSizeWithUniqueData, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getIntraWarpSizeWithUniqueData 等可调用 API，用来封装这里提供的核心行为。

### Lines 68-68
```cpp
  68:   bool isReduceWithinCTA();
```
**EN:** This block declares or defines callable APIs such as isReduceWithinCTA, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isReduceWithinCTA 等可调用 API，用来封装这里提供的核心行为。

### Lines 70-70
```cpp
  70:   bool isAssociative();
```
**EN:** This block declares or defines callable APIs such as isAssociative, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isAssociative 等可调用 API，用来封装这里提供的核心行为。

### Lines 72-76
```cpp
  72:   // Callback to allow backends to specify target-specific getter for scratch
  73:   // elements.
  74:   using GetNumScratchElemsFn = std::function<unsigned(
  75:       const triton::LinearLayout &src, const triton::LinearLayout &dst,
  76:       unsigned bitwidth)>;
```
**EN:** This block declares or defines callable APIs such as unsigned, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 unsigned 等可调用 API，用来封装这里提供的核心行为。

### Lines 78-79
```cpp
  78:   unsigned
  79:   getScratchSizeInBytes(GetNumScratchElemsFn numScratchElemsGetter = nullptr);
```
**EN:** This block declares or defines callable APIs such as getScratchSizeInBytes, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getScratchSizeInBytes 等可调用 API，用来封装这里提供的核心行为。

### Lines 81-84
```cpp
  81:   InThreadVectorizeOpKind
  82:   getInThreadVectorizeOpKind(unsigned axisPack,
  83:                              bool supportBitwidth16Elementwise,
  84:                              bool supportBitwidth32Elementwise);
```
**EN:** This block declares or defines callable APIs such as getInThreadVectorizeOpKind, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getInThreadVectorizeOpKind 等可调用 API，用来封装这里提供的核心行为。

### Lines 86-88
```cpp
  86:   static triton::ColumnAction
  87:   moveAxisBasesToFront(const triton::LinearLayout &layout, int axis,
  88:                        bool isVectorized = false);
```
**EN:** This block declares or defines callable APIs such as moveAxisBasesToFront, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 moveAxisBasesToFront 等可调用 API，用来封装这里提供的核心行为。

### Lines 90-92
```cpp
  90:   static triton::LinearLayout
  91:   zeroBasesAlongDimAndReorder(const triton::LinearLayout &layout, unsigned axis,
  92:                               mlir::StringAttr dim);
```
**EN:** This block declares or defines callable APIs such as zeroBasesAlongDimAndReorder, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 zeroBasesAlongDimAndReorder 等可调用 API，用来封装这里提供的核心行为。

### Lines 94-95
```cpp
  94:   static triton::LinearLayout getInterLayout(const triton::LinearLayout &layout,
  95:                                              unsigned axis);
```
**EN:** This block declares or defines callable APIs such as getInterLayout, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getInterLayout 等可调用 API，用来封装这里提供的核心行为。

### Lines 97-98
```cpp
  97:   static triton::LinearLayout reducedRegLaneLayout(RankedTensorType srcTy,
  98:                                                    unsigned axis);
```
**EN:** This block declares or defines callable APIs such as reducedRegLaneLayout, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 reducedRegLaneLayout 等可调用 API，用来封装这里提供的核心行为。

### Lines 100-103
```cpp
 100:   static Value createInThreadVectorizedCombineOp(OpBuilder &builder,
 101:                                                  Location loc,
 102:                                                  InThreadVectorizeOpKind kind,
 103:                                                  Value lhs, Value rhs);
```
**EN:** This block declares or defines callable APIs such as createInThreadVectorizedCombineOp, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 createInThreadVectorizedCombineOp 等可调用 API，用来封装这里提供的核心行为。

### Lines 105-112
```cpp
 105: private:
 106:   triton::ReduceOp op;
 107:   RankedTensorType srcTy;
 108:   ArrayRef<int64_t> srcShape;
 109:   Attribute srcEncoding;
 110:   SmallVector<Type> srcElementTypes;
 111:   int axis;
 112: };
```
**EN:** This block stores supporting state such as op, srcTy, srcShape, srcEncoding, srcElementTypes, and axis, which other APIs in the file consume.
**CN:** 该代码块声明了 op, srcTy, srcShape, srcEncoding, srcElementTypes, and axis 等支撑状态，供本文件中的其他 API 使用。

### Lines 114-139
```cpp
 114: class ScanLoweringHelper {
 115: public:
 116:   explicit ScanLoweringHelper(triton::ScanOp op);
 117:   // Return true if the lowering of the scan op is supported.
 118:   bool isSupported();
 119:   // Return the number of elements per thread along axis dim.
 120:   unsigned getAxisNumElementsPerThread();
 121:   // Return the number of elements per thread along non-axis dims.
 122:   unsigned getNonAxisNumElementsPerThread();
 123:   // Return the number of threads per warp along non-axis dims.
 124:   unsigned getNonAxisNumThreadsPerWarp();
 125:   // Return the flat numbers of threads computing independent scan results.
 126:   unsigned getNonAxisNumThreadsPerCTA();
 127:   // Return the number of warps per CTA along axis dim with unique data.
 128:   unsigned getAxisNumWarpsWithUniqueData();
 129:   // Return the number of threads per warp along axis dim with unique data.
 130:   unsigned getAxisNumThreadsPerWarpWithUniqueData();
 131:   // Return the number of blocks along axis dim.
 132:   unsigned getAxisNumBlocks();
 133:   // Return the number of blocks along non axis dim.
 134:   unsigned getNonAxisNumBlocks();
 135:   // Return the size of the scratch space needed for scan lowering.
 136:   unsigned getScratchSizeInBytes();
 137:   // Return the number of elements of the scratch space needed for scan
 138:   // lowering.
 139:   unsigned getScratchSizeInElems();
```
**EN:** This block introduces `ScanLoweringHelper`, the main class/struct defined here. Within the declaration, methods such as isSupported, getAxisNumElementsPerThread, getNonAxisNumElementsPerThread, getNonAxisNumThreadsPerWarp, getNonAxisNumThreadsPerCTA, and getAxisNumWarpsWithUniqueData expose its core API.
**CN:** 该代码块引入了此文件的核心类/结构体 `ScanLoweringHelper`。 其中 isSupported, getAxisNumElementsPerThread, getNonAxisNumElementsPerThread, getNonAxisNumThreadsPerWarp, getNonAxisNumThreadsPerCTA, and getAxisNumWarpsWithUniqueData 等方法构成了它的主要接口。

### Lines 141-146
```cpp
 141:   // Stride between contiguous element along axis dim.
 142:   unsigned getAxisElementStride();
 143:   // Stride between contiguous threads along axis dim.
 144:   unsigned getAxisThreadStride();
 145:   // Stride between contiguous blocks along axis dim.
 146:   unsigned getAxisBlockStride();
```
**EN:** This block declares or defines callable APIs such as getAxisElementStride, getAxisThreadStride, and getAxisBlockStride, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getAxisElementStride, getAxisThreadStride, and getAxisBlockStride 等可调用 API，用来封装这里提供的核心行为。

### Lines 148-156
```cpp
 148:   Location getLoc() { return scanOp.getLoc(); }
 149:   unsigned getAxis() { return scanOp.getAxis(); }
 150:   bool getReverse() { return scanOp.getReverse(); }
 151:   triton::gpu::LinearEncodingAttr getEncoding() { return srcEncoding; }
 152:   llvm::ArrayRef<int64_t> getShape() { return srcShape; }
 153:   unsigned getNumOperands() { return scanOp.getNumOperands(); }
 154:   SmallVector<Type> getElementTypes() { return srcElementTypes; }
 155:   SmallVector<unsigned> getOrder() { return order; }
 156:   Region &getCombineOp();
```
**EN:** This block declares or defines callable APIs such as getLoc, getAxis, getReverse, getEncoding, getShape, getNumOperands, getElementTypes, and getOrder, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getLoc, getAxis, getReverse, getEncoding, getShape, getNumOperands, getElementTypes, and getOrder 等可调用 API，用来封装这里提供的核心行为。

### Lines 158-165
```cpp
 158: private:
 159:   triton::ScanOp scanOp;
 160:   triton::gpu::LinearEncodingAttr srcEncoding;
 161:   Attribute legacyEncoding;
 162:   llvm::ArrayRef<int64_t> srcShape;
 163:   SmallVector<Type> srcElementTypes;
 164:   SmallVector<unsigned> order;
 165: };
```
**EN:** This block stores supporting state such as scanOp, srcEncoding, legacyEncoding, srcShape, srcElementTypes, and order, which other APIs in the file consume.
**CN:** 该代码块声明了 scanOp, srcEncoding, legacyEncoding, srcShape, srcElementTypes, and order 等支撑状态，供本文件中的其他 API 使用。

### Lines 167-171
```cpp
 167: // Helper class for lowering `tt.gather` operations. This class shares lowering
 168: // logic between shared memory allocation and LLVM codegen.
 169: class GatherLoweringHelper {
 170: public:
 171:   GatherLoweringHelper(triton::GatherOp gatherOp);
```
**EN:** This block introduces `GatherLoweringHelper`, the main class/struct defined here.
**CN:** 该代码块引入了此文件的核心类/结构体 `GatherLoweringHelper`。

### Lines 173-176
```cpp
 173:   // Get the shared memory scratch size required by this op.
 174:   unsigned getScratchSizeInBytes();
 175:   // Determine if the gather can be performed completely within a warp.
 176:   bool isWarpLocal();
```
**EN:** This block declares or defines callable APIs such as getScratchSizeInBytes and isWarpLocal, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getScratchSizeInBytes and isWarpLocal 等可调用 API，用来封装这里提供的核心行为。

### Lines 178-182
```cpp
 178: private:
 179:   triton::GatherOp gatherOp;
 180:   RankedTensorType srcTy;
 181:   RankedTensorType dstTy;
 182: };
```
**EN:** This block stores supporting state such as gatherOp, srcTy, and dstTy, which other APIs in the file consume.
**CN:** 该代码块声明了 gatherOp, srcTy, and dstTy 等支撑状态，供本文件中的其他 API 使用。

### Lines 184-204
```cpp
 184: // This struct represents the factorization of a warp-local layout conversion
 185: // into three components: a register-only permutation, a lane-only permutation,
 186: // and a set of swaps between lane and register basis vectors. Algebraically, it
 187: // represents the factorization P = P_mixed \circ P_lane \circ P_reg. It is used
 188: // to aid in the implementation of the layout conversion using warp-shuffles.
 189: //
 190: // `pReg` and `pLane` are square layouts each with only one input and output
 191: // dimension. `mixedTranspositions` holds pairs of integers (i, j)
 192: // corresponding to the transposition (r_i l_j) of the i-th register basis
 193: // vector with the j-th lane basis vector along with 16-bit selectors for byte
 194: // permute instructions (where each of the four nybbles is in the range [0, 7]).
 195: // `nPack` gives the number of basis vectors that can be used for register
 196: // packing while ensuring packed elements arrive at the same destination lane.
 197: struct DecomposedWarpConversion {
 198:   struct TranspositionInfo {
 199:     std::pair<int, int> transposition;
 200:     uint16_t topPreSel = 0x3210;
 201:     uint16_t botPreSel = 0x7654;
 202:     uint16_t topPostSel = 0x3210;
 203:     uint16_t botPostSel = 0x7654;
 204:   };
```
**EN:** This block introduces `DecomposedWarpConversion`, the main class/struct defined here.
**CN:** 该代码块引入了此文件的核心类/结构体 `DecomposedWarpConversion`。

### Lines 206-209
```cpp
 206:   triton::LinearLayout pReg, pLane;
 207:   SmallVector<TranspositionInfo> mixedTranspositions;
 208:   int nPack;
 209: };
```
**EN:** This block stores supporting state such as pLane, mixedTranspositions, and nPack, which other APIs in the file consume.
**CN:** 该代码块声明了 pLane, mixedTranspositions, and nPack 等支撑状态，供本文件中的其他 API 使用。

### Lines 211-220
```cpp
 211: // Produces a decomposition of a permutation describing a warp-local layout
 212: // conversion as described in `DecomposedWarpConversion` above.
 213: //
 214: // This function handles cases where the numbers of register and lane basis
 215: // vectors differ between the two layouts. This is done by padding the smaller
 216: // dimension(s) with zero vectors, ensuring that the layout conversion can be
 217: // represented as a permutation.
 218: DecomposedWarpConversion
 219: getWarpLayoutConvertDecomposition(RankedTensorType srcTy,
 220:                                   RankedTensorType dstTy, int bitwidth);
```
**EN:** This block declares or defines callable APIs such as getWarpLayoutConvertDecomposition, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getWarpLayoutConvertDecomposition 等可调用 API，用来封装这里提供的核心行为。

### Lines 222-245
```cpp
 222: // Decomposes a reshape into simpler pieces.
 223: //
 224: // As an example, suppose we have a reshape from [4,4,4] to [2,2,8,2].
 225: // You might explain what this does as follows.
 226: //
 227: //  - Split the first input dimension into [2,2].
 228: //  - Take the remaining two input dimensions, merge them into a single [16]
 229: //    dim, and then split that into [8,2].
 230: //
 231: // In general, a reshape can be described a sequence of smushing one or more
 232: // input dimensions together and then breaking them apart into one or more
 233: // output dimensions.  So we could represent the example above as follows.
 234: //
 235: //   [
 236: //     ([0], [0, 1]),  # input dim [0] -> output dims [0, 1]
 237: //     ([1, 2], [2, 3]),  # input dims [1, 2] -> output dims [2, 3]
 238: //   ]
 239: //
 240: // Notice that the input dims (first tuple elems) appear in sequential order if
 241: // you read left-to-right-top-to-bottom, and so do the output dims.
 242: //
 243: // This function returns the above decomposition.
 244: SmallVector<std::pair<SmallVector<int64_t>, SmallVector<int64_t>>>
 245: getReshapeDecomposition(ArrayRef<int64_t> srcShape, ArrayRef<int64_t> dstShape);
```
**EN:** This block declares or defines callable APIs such as getReshapeDecomposition, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getReshapeDecomposition 等可调用 API，用来封装这里提供的核心行为。

### Lines 247-249
```cpp
 247: // Returns the number of elements in the scratch space needed.
 248: // If shape is empty, it means no shared memory is needed.
 249: unsigned getNumScratchElements(ArrayRef<unsigned> shape);
```
**EN:** This block declares or defines callable APIs such as getNumScratchElements, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getNumScratchElements 等可调用 API，用来封装这里提供的核心行为。

### Lines 251-251
```cpp
 251: bool supportWMMA(triton::DotOp op);
```
**EN:** This block declares or defines callable APIs such as supportWMMA, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 supportWMMA 等可调用 API，用来封装这里提供的核心行为。

### Lines 253-253
```cpp
 253: bool supportMMA(triton::DotOp op, int version);
```
**EN:** This block declares or defines callable APIs such as supportMMA, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 supportMMA 等可调用 API，用来封装这里提供的核心行为。

### Lines 255-255
```cpp
 255: bool supportMMA(Value value, int version);
```
**EN:** This block declares or defines callable APIs such as supportMMA, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 supportMMA 等可调用 API，用来封装这里提供的核心行为。

### Lines 257-262
```cpp
 257: // Conversion from `srcTy` to `dstTy` involving the minimum amount of data
 258: // transfer provided that both types can be converted to LL (if it can't it'll
 259: // return nullopt). The output will be such that layout.getInDimNames() ==
 260: // layout.getOutDimNames() and the conversion will not include kBlock (resp.
 261: // kWarp or kLane) if it can be avoided
 262: triton::LinearLayout minimalCvtLayout(Type srcTy, Type dstTy);
```
**EN:** This block declares or defines callable APIs such as minimalCvtLayout, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 minimalCvtLayout 等可调用 API，用来封装这里提供的核心行为。

### Lines 264-266
```cpp
 264: // Conversion from `srcTy` to `dstTy` only involves reordering of registers.
 265: // There is no need for data exchange across threads, warps, or blocks.
 266: bool cvtReordersRegisters(RankedTensorType srcTy, RankedTensorType dstTy);
```
**EN:** This block declares or defines callable APIs such as cvtReordersRegisters, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 cvtReordersRegisters 等可调用 API，用来封装这里提供的核心行为。

### Lines 268-270
```cpp
 268: // Conversion from `srcTy` to `dstTy` involves data exchange across threads
 269: // within a warp.  No data exchange across warps or blocks is needed.
 270: bool cvtNeedsWarpShuffle(RankedTensorType srcTy, RankedTensorType dstTy);
```
**EN:** This block declares or defines callable APIs such as cvtNeedsWarpShuffle, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 cvtNeedsWarpShuffle 等可调用 API，用来封装这里提供的核心行为。

### Lines 272-274
```cpp
 272: // Conversion from `srcTy` to `dstTy` involves data exchange across threads,
 273: // warps, and possibly blocks.
 274: bool cvtNeedsSharedMemory(RankedTensorType srcTy, RankedTensorType dstTy);
```
**EN:** This block declares or defines callable APIs such as cvtNeedsSharedMemory, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 cvtNeedsSharedMemory 等可调用 API，用来封装这里提供的核心行为。

### Lines 276-278
```cpp
 276: // TODO: Move utility functions that belong to ConvertLayoutOp to class
 277: // ConvertLayoutOpHelper in the future
 278: bool shouldUseDistSmem(Attribute srcLayout, Attribute dstLayout);
```
**EN:** This block declares or defines callable APIs such as shouldUseDistSmem, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 shouldUseDistSmem 等可调用 API，用来封装这里提供的核心行为。

### Lines 280-281
```cpp
 280: /// Create a basic DataFlowSolver with constant and dead code analysis included.
 281: std::unique_ptr<DataFlowSolver> createDataFlowSolver();
```
**EN:** This block declares or defines callable APIs such as createDataFlowSolver, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 createDataFlowSolver 等可调用 API，用来封装这里提供的核心行为。

### Lines 283-283
```cpp
 283: namespace triton {
```
**EN:** This block opens or closes the namespace scope used to organize APIs under triton.
**CN:** 该代码块打开或关闭命名空间作用域，用来把相关 API 组织在 triton 下。

### Lines 285-289
```cpp
 285: /// This class represents a call graph for a given ModuleOp and holds
 286: /// data of type T associated with each FunctionOpInterface.
 287: template <typename T> class CallGraph {
 288: public:
 289:   using FuncDataMapT = DenseMap<FunctionOpInterface, T>;
```
**EN:** This block introduces `CallGraph`, the main class/struct defined here.
**CN:** 该代码块引入了此文件的核心类/结构体 `CallGraph`。

### Lines 291-292
```cpp
 291:   /// Constructor that builds the call graph for the given moduleOp.
 292:   explicit CallGraph(ModuleOp moduleOp) : moduleOp(moduleOp) { build(); }
```
**EN:** This block declares or defines callable APIs such as CallGraph, moduleOp, and build, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 CallGraph, moduleOp, and build 等可调用 API，用来封装这里提供的核心行为。

### Lines 294-305
```cpp
 294:   /// Walks the call graph and applies the provided update functions
 295:   /// to the edges and nodes.
 296:   template <WalkOrder UpdateEdgeOrder = WalkOrder::PreOrder,
 297:             WalkOrder UpdateNodeOrder = WalkOrder::PreOrder,
 298:             typename UpdateEdgeFn, typename UpdateNodeFn>
 299:   void walk(UpdateEdgeFn updateEdgeFn, UpdateNodeFn updateNodeFn) {
 300:     DenseSet<FunctionOpInterface> visited;
 301:     for (auto root : roots) {
 302:       doWalk<UpdateEdgeOrder, UpdateNodeOrder>(root, visited, updateEdgeFn,
 303:                                                updateNodeFn);
 304:     }
 305:   }
```
**EN:** This block declares or defines callable APIs such as walk, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 walk 等可调用 API，用来封装这里提供的核心行为。

### Lines 307-313
```cpp
 307:   /// Retrieves the data associated with a function
 308:   T *getFuncData(FunctionOpInterface funcOp) {
 309:     if (funcMap.count(funcOp)) {
 310:       return &funcMap[funcOp];
 311:     }
 312:     return nullptr;
 313:   }
```
**EN:** This block declares or defines callable APIs such as getFuncData and count, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getFuncData and count 等可调用 API，用来封装这里提供的核心行为。

### Lines 315-318
```cpp
 315:   /// Getters
 316:   ModuleOp getModuleOp() const { return moduleOp; }
 317:   SmallVector<FunctionOpInterface> getRoots() const { return roots; }
 318:   size_t getNumFunctions() const { return funcMap.size(); }
```
**EN:** This block declares or defines callable APIs such as getModuleOp, getRoots, getNumFunctions, and size, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getModuleOp, getRoots, getNumFunctions, and size 等可调用 API，用来封装这里提供的核心行为。

### Lines 320-323
```cpp
 320:   /// Returns true if the given function is a root.
 321:   bool isRoot(FunctionOpInterface funcOp) const {
 322:     return llvm::is_contained(roots, funcOp);
 323:   }
```
**EN:** This block declares or defines callable APIs such as isRoot and is_contained, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isRoot and is_contained 等可调用 API，用来封装这里提供的核心行为。

### Lines 325-347
```cpp
 325:   /// Maps the data and the graph nodes associated with a funcOp to a
 326:   /// targetFuncOp.
 327:   template <typename FROM, typename TO>
 328:   void mapFuncOp(FROM funcOp, TO targetFuncOp) {
 329:     // Iterate over graph and replace
 330:     for (auto &kv : graph) {
 331:       for (auto &edge : kv.second) {
 332:         if (edge.second == funcOp) {
 333:           edge.second = targetFuncOp;
 334:         }
 335:       }
 336:     }
 337:     graph[targetFuncOp] = graph[funcOp];
 338:     // Replace in roots
 339:     for (auto it = roots.begin(); it != roots.end(); ++it) {
 340:       if (*it == funcOp) {
 341:         *it = targetFuncOp;
 342:         break;
 343:       }
 344:     }
 345:     // Replace in funcMap
 346:     funcMap[targetFuncOp] = funcMap[funcOp];
 347:   }
```
**EN:** This block declares or defines callable APIs such as mapFuncOp, begin, and end, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 mapFuncOp, begin, and end 等可调用 API，用来封装这里提供的核心行为。

### Lines 349-360
```cpp
 349:   /// Maps the graph edges associated with a callOp to a targetCallOp.
 350:   template <typename FROM, typename TO>
 351:   void mapCallOp(FROM callOp, TO targetCallOp) {
 352:     // Iterate over graph and replace
 353:     for (auto &kv : graph) {
 354:       for (auto &edge : kv.second) {
 355:         if (edge.first == callOp) {
 356:           edge.first = targetCallOp;
 357:         }
 358:       }
 359:     }
 360:   }
```
**EN:** This block declares or defines callable APIs such as mapCallOp, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 mapCallOp 等可调用 API，用来封装这里提供的核心行为。

### Lines 362-385
```cpp
 362: private:
 363:   void build() {
 364:     SymbolTableCollection symbolTable;
 365:     DenseSet<FunctionOpInterface> visited;
 366:     // Build graph
 367:     moduleOp.walk([&](Operation *op) {
 368:       auto caller = op->getParentOfType<FunctionOpInterface>();
 369:       if (auto callOp = dyn_cast<CallOpInterface>(op)) {
 370:         auto *callee = callOp.resolveCallableInTable(&symbolTable);
 371:         auto funcOp = dyn_cast_or_null<FunctionOpInterface>(callee);
 372:         if (funcOp) {
 373:           graph[caller].emplace_back(
 374:               std::pair<CallOpInterface, FunctionOpInterface>(callOp, funcOp));
 375:           visited.insert(funcOp);
 376:         }
 377:       }
 378:     });
 379:     // Find roots
 380:     moduleOp.walk([&](FunctionOpInterface funcOp) {
 381:       if (!visited.count(funcOp)) {
 382:         roots.push_back(funcOp);
 383:       }
 384:     });
 385:   }
```
**EN:** This block declares or defines callable APIs such as build, walk, resolveCallableInTable, emplace_back, insert, count, and push_back, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 build, walk, resolveCallableInTable, emplace_back, insert, count, and push_back 等可调用 API，用来封装这里提供的核心行为。

### Lines 387-413
```cpp
 387:   template <WalkOrder UpdateEdgeOrder = WalkOrder::PreOrder,
 388:             WalkOrder UpdateNodeOrder = WalkOrder::PreOrder,
 389:             typename UpdateEdgeFn, typename UpdateNodeFn>
 390:   void doWalk(FunctionOpInterface funcOp,
 391:               DenseSet<FunctionOpInterface> &visited, UpdateEdgeFn updateEdgeFn,
 392:               UpdateNodeFn updateNodeFn) {
 393:     if (visited.count(funcOp)) {
 394:       llvm::report_fatal_error("Cycle detected in call graph");
 395:     }
 396:     if constexpr (UpdateNodeOrder == WalkOrder::PreOrder) {
 397:       updateNodeFn(funcOp);
 398:     }
 399:     for (auto [callOp, callee] : graph[funcOp]) {
 400:       if constexpr (UpdateEdgeOrder == WalkOrder::PreOrder) {
 401:         updateEdgeFn(callOp, callee);
 402:       }
 403:       doWalk<UpdateEdgeOrder, UpdateNodeOrder>(callee, visited, updateEdgeFn,
 404:                                                updateNodeFn);
 405:       if constexpr (UpdateEdgeOrder == WalkOrder::PostOrder) {
 406:         updateEdgeFn(callOp, callee);
 407:       }
 408:     }
 409:     if constexpr (UpdateNodeOrder == WalkOrder::PostOrder) {
 410:       updateNodeFn(funcOp);
 411:     }
 412:     visited.erase(funcOp);
 413:   }
```
**EN:** This block declares or defines callable APIs such as doWalk, count, report_fatal_error, constexpr, updateNodeFn, updateEdgeFn, and erase, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 doWalk, count, report_fatal_error, constexpr, updateNodeFn, updateEdgeFn, and erase 等可调用 API，用来封装这里提供的核心行为。

### Lines 415-422
```cpp
 415: protected:
 416:   ModuleOp moduleOp;
 417:   DenseMap<FunctionOpInterface,
 418:            SmallVector<std::pair<CallOpInterface, FunctionOpInterface>>>
 419:       graph;
 420:   FuncDataMapT funcMap;
 421:   SmallVector<FunctionOpInterface> roots;
 422: };
```
**EN:** This block stores supporting state such as moduleOp, graph, funcMap, and roots, which other APIs in the file consume.
**CN:** 该代码块声明了 moduleOp, graph, funcMap, and roots 等支撑状态，供本文件中的其他 API 使用。

### Lines 424-424
```cpp
 424: } // namespace triton
```
**EN:** This block manages the namespace scope for the surrounding declarations.
**CN:** 该代码块负责管理周围声明所属的命名空间作用域。

### Lines 426-427
```cpp
 426: // Create a basic DataFlowSolver with constant and dead code analysis included.
 427: std::unique_ptr<DataFlowSolver> createDataFlowSolver();
```
**EN:** This block declares or defines callable APIs such as createDataFlowSolver, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 createDataFlowSolver 等可调用 API，用来封装这里提供的核心行为。

### Lines 429-430
```cpp
 429: bool isCvtDimSync(const triton::LinearLayout &srcLayout,
 430:                   const triton::LinearLayout &dstLayout, StringAttr dim);
```
**EN:** This block declares or defines callable APIs such as isCvtDimSync, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isCvtDimSync 等可调用 API，用来封装这里提供的核心行为。

### Lines 432-432
```cpp
 432: } // namespace mlir
```
**EN:** This block manages the namespace scope for the surrounding declarations.
**CN:** 该代码块负责管理周围声明所属的命名空间作用域。

### Lines 434-434
```cpp
 434: #endif // TRITON_ANALYSIS_UTILITY_H
```
**EN:** This block closes the header guard started at the top of the file.
**CN:** 该代码块结束文件顶部开始的头文件保护宏。

## Key Concepts / 关键概念
- **EN:** axis reasoning  
  **CN:** 轴信息推理
- **EN:** layout conversion and reasoning  
  **CN:** 布局转换与推理
- **EN:** linear layout algebra  
  **CN:** 线性布局代数
- **EN:** layout encodings  
  **CN:** 布局编码
- **EN:** warp-level execution  
  **CN:** warp 级执行
- **EN:** LLVM lowering  
  **CN:** LLVM 降级
- **EN:** MLIR dialect definition  
  **CN:** MLIR 方言定义
- **EN:** operation definitions  
  **CN:** 操作定义

## Dependencies / 依赖关系
- **Project includes / 项目内依赖:**
  - `mlir/Analysis/DataFlowFramework.h`
  - `mlir/Analysis/SliceAnalysis.h`
  - `mlir/IR/Builders.h`
  - `mlir/Support/LLVM.h`
  - `triton/Dialect/Triton/IR/Dialect.h`
  - `triton/Dialect/TritonGPU/IR/Dialect.h`
  - `triton/Tools/LinearLayout.h`
