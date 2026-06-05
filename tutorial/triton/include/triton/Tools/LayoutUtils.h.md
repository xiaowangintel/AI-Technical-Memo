# LayoutUtils.h — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./include/triton/Tools/LayoutUtils.h`
- **EN:** Declares reusable tool-layer utilities centered on `LayoutUtils`.
- **CN:** 声明围绕 `LayoutUtils` 的可复用工具层设施。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```cpp
   1: #ifndef TRITON_TOOLS_LAYOUTUTILS_H
   2: #define TRITON_TOOLS_LAYOUTUTILS_H
```
**EN:** This block establishes the header guard so the declarations in this file are only processed once per translation unit.
**CN:** 该代码块建立头文件保护宏，避免同一翻译单元重复包含本文件时出现重复声明。

### Lines 4-4
```cpp
   4: #include "triton/Tools/LinearLayout.h"
```
**EN:** This block imports the direct dependencies needed here, including triton/Tools/LinearLayout.h.
**CN:** 该代码块引入后续声明直接依赖的头文件，例如 triton/Tools/LinearLayout.h。

### Lines 6-11
```cpp
   6: namespace mlir::triton {
   7: // Is the sublayout defined from dimNames to dimNames the identity?
   8: // In particular, is the input and  output size in these dimensions
   9: // the same, and are the bases the identity?
  10: bool squareSublayoutIsIdentity(const LinearLayout &ll,
  11:                                ArrayRef<StringAttr> dimNames);
```
**EN:** This block declares or defines callable APIs such as squareSublayoutIsIdentity, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 squareSublayoutIsIdentity 等可调用 API，用来封装这里提供的核心行为。

### Lines 13-65
```cpp
  13: // For each output dimension d, ensure that the layout's output size (i.e., its
  14: // codomain) does not exceed shape[d]. Do this without changing the size of the
  15: // layout's inputs (i.e., leave its domain unchanged).
  16: //
  17: // This function is invariant to the order of the layout's input and output
  18: // dimensions.
  19: //
  20: // We achieve this by setting the largest value in each output dimension d to 0
  21: // because bases that map to a location larger than shape[d]
  22: // effectively duplicate along that dimension.  For example, consider a layout
  23: // with an output dimension size of 32, and we call ensureLayoutNotLargerThan to
  24: // shrink the output dimension size to 8:
  25: //
  26: //   L(register=1) = 8
  27: //   L(register=2) = 4
  28: //   L(register=4) = 1
  29: //   L(lane=1) = 2
  30: //   L(lane=2) = 16
  31: //
  32: // In the first step, we shrink the output dimension size to 16 by setting
  33: // L(lane=2) to 0:
  34: //
  35: //   L(register=1) = 8
  36: //   L(register=2) = 4
  37: //   L(register=4) = 1
  38: //   L(lane=1) = 2
  39: //   L(lane=2) = 0
  40: //
  41: // This means that lane=2 has the same data as lane=0.
  42: //
  43: // Now the output dimension of this layout has a size of 16, which is still
  44: // larger than 8.  We find the current largest value in the output dimension,
  45: // which is L(register=1) = 8, and we set L(register=1) to 0:
  46: //
  47: //   L(register=1) = 0
  48: //   L(register=2) = 4
  49: //   L(register=4) = 1
  50: //   L(lane=1) = 2
  51: //   L(lane=2) = 0
  52: //
  53: // Now the output dimension of this layout has a size of 8, which is the desired
  54: // size.  Note that this method works only because the bases are powers of two,
  55: // which is the case for DistributedLayouts If broadcastRegisters is false, we
  56: // remove any register that's larger than the desired shape. In the example
  57: // above we would have
  58: //   L(register=1) = 4
  59: //   L(register=2) = 1
  60: //   L(lane=1) = 2
  61: //   L(lane=2) = 0
  62: LinearLayout
  63: ensureLayoutNotLargerThan(const LinearLayout &layout,
  64:                           const llvm::SmallDenseMap<StringAttr, int64_t> &shape,
  65:                           bool broadcastRegisters = true);
```
**EN:** This block declares or defines callable APIs such as ensureLayoutNotLargerThan, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 ensureLayoutNotLargerThan 等可调用 API，用来封装这里提供的核心行为。

### Lines 67-76
```cpp
  67: // For each out-dim d, ensure the layout's out-size (i.e. its codomain) is no
  68: // smaller than shape[d].  Do this by increasing the size of the layout's inputs
  69: // along its most-minor dimension ("register" for register layouts, "offset" for
  70: // shared layouts).
  71: //
  72: // This function is invariant to the order of the layout's input dimensions, but
  73: // it cares about the order of the output dims, which should be minor-to-major.
  74: LinearLayout ensureLayoutNotSmallerThan(
  75:     const LinearLayout &layout,
  76:     const llvm::SmallDenseMap<StringAttr, int64_t> &shape);
```
**EN:** This block declares or defines callable APIs such as ensureLayoutNotSmallerThan, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 ensureLayoutNotSmallerThan 等可调用 API，用来封装这里提供的核心行为。

### Lines 78-87
```cpp
  78: inline LinearLayout
  79: ensureLayoutNotSmallerThan(const LinearLayout &layout,
  80:                            const llvm::ArrayRef<StringAttr> dimNames,
  81:                            const llvm::ArrayRef<int64_t> shape) {
  82:   llvm::SmallDenseMap<StringAttr, int64_t> namedDims;
  83:   for (auto [dimName, length] : llvm::zip_equal(dimNames, shape))
  84:     namedDims[dimName] = length;
  85:   assert(namedDims.size() == shape.size() && "duplicate dimension names given");
  86:   return ensureLayoutNotSmallerThan(layout, namedDims);
  87: }
```
**EN:** This block declares or defines callable APIs such as ensureLayoutNotSmallerThan, zip_equal, and size, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 ensureLayoutNotSmallerThan, zip_equal, and size 等可调用 API，用来封装这里提供的核心行为。

### Lines 89-91
```cpp
  89: // Return a vector of the standard out dimension names for tensor layouts. These
  90: // are "dim0", "dim1", etc.
  91: SmallVector<StringAttr> standardOutDimNames(MLIRContext *ctx, int rank);
```
**EN:** This block declares or defines callable APIs such as standardOutDimNames, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 standardOutDimNames 等可调用 API，用来封装这里提供的核心行为。

### Lines 93-96
```cpp
  93: // Return a vector of the standard out dimension name/value pairs, i.e.
  94: // ("dim0", dstShape[0]), ("dim1", dstShape[1]), etc.
  95: SmallVector<std::pair<StringAttr, int32_t>>
  96: standardOutDimPairs(MLIRContext *ctx, ArrayRef<int64_t> dstShape);
```
**EN:** This block declares or defines callable APIs such as standardOutDimPairs, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 standardOutDimPairs 等可调用 API，用来封装这里提供的核心行为。

### Lines 98-102
```cpp
  98: // Return an identity mapping from `inDimName` to the standard out dimensions,
  99: // with the dimensions sized according to the shape. The bases are sorted
 100: // according to `order`, with the most minor dimension first.
 101: LinearLayout identityStandardND(StringAttr inDimName, ArrayRef<unsigned> shape,
 102:                                 ArrayRef<unsigned> order);
```
**EN:** This block declares or defines callable APIs such as identityStandardND, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 identityStandardND 等可调用 API，用来封装这里提供的核心行为。

### Lines 104-106
```cpp
 104: // Return a layout with the same in/out dimensions as `layout` but with all
 105: // bases set to 0.
 106: LinearLayout zerosLike(const LinearLayout &layout);
```
**EN:** This block declares or defines callable APIs such as zerosLike, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 zerosLike 等可调用 API，用来封装这里提供的核心行为。

### Lines 108-114
```cpp
 108: // For a layout A with A.hasInDim(kReg), find a permutation of registers action
 109: // such that action.apply(A) may be divisible by B
 110: // It's not always true that the action returned by this function will
 111: // allow us to divideLeft (resp. divideRight), but it is true that if it if
 112: // there exists one, it is the one returned by this function.
 113: std::optional<ColumnAction> regPermForDivide(const LinearLayout &A,
 114:                                              const LinearLayout &B, bool left);
```
**EN:** This block declares or defines callable APIs such as regPermForDivide, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 regPermForDivide 等可调用 API，用来封装这里提供的核心行为。

### Lines 116-118
```cpp
 116: // For a layout A with A.hasInDim(kReg), find a permutation of registers action
 117: // such that action.apply(A) has the broadcasted registers removed
 118: ColumnAction actionRemoveBroadcastedRegs(const LinearLayout &layout);
```
**EN:** This block declares or defines callable APIs such as actionRemoveBroadcastedRegs, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 actionRemoveBroadcastedRegs 等可调用 API，用来封装这里提供的核心行为。

### Lines 120-122
```cpp
 120: std::pair<int64_t, ColumnAction>
 121: actionAdditiveStrides(const LinearLayout &layout, const LinearLayout addrLayout,
 122:                       uint64_t maskSpanOffsets, int64_t regsPerInst);
```
**EN:** This block declares or defines callable APIs such as actionAdditiveStrides, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 actionAdditiveStrides 等可调用 API，用来封装这里提供的核心行为。

### Lines 124-127
```cpp
 124: // For a layout A with A.hasInDim(kReg), repeat the values so that they have
 125: // the same broadcasting as layout
 126: SmallVector<Value> broadcastAs(const SmallVector<Value> &values,
 127:                                const LinearLayout &layout);
```
**EN:** This block declares or defines callable APIs such as broadcastAs, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 broadcastAs 等可调用 API，用来封装这里提供的核心行为。

### Lines 129-134
```cpp
 129: // Compute the supremum of two lists.
 130: // Error out if the supremum does not exist (e.g. [a, b] and [b, a]).
 131: // If the supremum is not unique, we return the first list first
 132: // (e.g. [a, b], [a, c] -> [a, b, c]).
 133: SmallVector<StringAttr> supremum(const SmallVector<StringAttr> &x,
 134:                                  const SmallVector<StringAttr> &y);
```
**EN:** This block declares or defines callable APIs such as supremum, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 supremum 等可调用 API，用来封装这里提供的核心行为。

### Lines 136-138
```cpp
 136: // Return a new layout reshaped to the given shape.
 137: LinearLayout reshapeLayout(MLIRContext *ctx, LinearLayout layout,
 138:                            ArrayRef<int64_t> shape);
```
**EN:** This block declares or defines callable APIs such as reshapeLayout, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 reshapeLayout 等可调用 API，用来封装这里提供的核心行为。

### Lines 140-142
```cpp
 140: // Return a new layout with the dimensions transposed according to the given
 141: // order.
 142: LinearLayout transposeLinearLayout(LinearLayout layout, ArrayRef<int> order);
```
**EN:** This block declares or defines callable APIs such as transposeLinearLayout, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 transposeLinearLayout 等可调用 API，用来封装这里提供的核心行为。

### Lines 144-148
```cpp
 144: // Given a distributed into shmem layout, return the largest vectorisation
 145: // that can be used to lower the layout via ld/st.
 146: std::pair<int, ColumnAction>
 147: largestVectorisation(MLIRContext *ctx, const LinearLayout &cvt, int bitwidth,
 148:                      std::optional<int> maybeMaxVecElems = std::nullopt);
```
**EN:** This block declares or defines callable APIs such as largestVectorisation, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 largestVectorisation 等可调用 API，用来封装这里提供的核心行为。

### Lines 150-183
```cpp
 150: // Close cousin of doing zerosLike(tile) * divideLeft(cvt, tile)
 151: // This one is a tad more general in the sense that it allows to divide
 152: //  cvt:
 153: // - register=1 -> (0, 1)
 154: //   register=2 -> (8, 0)
 155: //   register=4 -> (0, 8)
 156: //   register=8 -> (0, 16)
 157: //   register=16 -> (0, 32)
 158: //   register=32 -> (0, 64)
 159: //   register=64 -> (16, 0)
 160: // - lane=1 -> (0, 2)
 161: //   lane=2 -> (0, 4)
 162: //   lane=4 -> (1, 0)
 163: //   lane=8 -> (2, 0)
 164: //   lane=16 -> (4, 0)
 165: // - warp=1 -> (32, 0)
 166: //   warp=2 -> (64, 0)
 167: // - block is a size 1 dimension
 168: // where out dims are: [row (size 128), col (size 128)]
 169: // tile:
 170: //  - register=1 -> (0, 1)
 171: //    register=2 -> (8, 0)
 172: //  - lane=1 -> (0, 2)
 173: //    lane=2 -> (0, 4)
 174: //    lane=4 -> (1, 0)
 175: //    lane=8 -> (2, 0)
 176: //    lane=16 -> (4, 0)
 177: //  - warp=1 -> (32, 0)
 178: //    warp=2 -> (64, 0)
 179: // where out dims are: [row (size 128), col (size 8)]
 180: // which would not be possible to lower via the divideLeft approach as we
 181: // cannot divide by the tile given the `register=64 -> (16, 0)` basis.
 182: std::optional<LinearLayout> getReps(const LinearLayout &cvt,
 183:                                     const LinearLayout &tile);
```
**EN:** This block declares or defines callable APIs such as getReps, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getReps 等可调用 API，用来封装这里提供的核心行为。

### Lines 185-188
```cpp
 185: // Given a layout mapping onto dim0..dimn, remove a dimension `dim`
 186: // and rename the rest as dim0..dimn-1
 187: LinearLayout removeStandardDim(const LinearLayout &layout, int dim);
 188: } // namespace mlir::triton
```
**EN:** This block declares or defines callable APIs such as removeStandardDim, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 removeStandardDim 等可调用 API，用来封装这里提供的核心行为。

### Lines 190-190
```cpp
 190: #endif // TRITON_TOOLS_LAYOUTUTILS_H
```
**EN:** This block closes the header guard started at the top of the file.
**CN:** 该代码块结束文件顶部开始的头文件保护宏。

## Key Concepts / 关键概念
- **EN:** layout conversion and reasoning  
  **CN:** 布局转换与推理
- **EN:** linear layout algebra  
  **CN:** 线性布局代数
- **EN:** LLVM lowering  
  **CN:** LLVM 降级
- **EN:** attributes  
  **CN:** 属性
- **EN:** MLIR integration  
  **CN:** MLIR 集成

## Dependencies / 依赖关系
- **Project includes / 项目内依赖:**
  - `triton/Tools/LinearLayout.h`
