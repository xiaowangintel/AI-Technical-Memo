# TritonGPUAttrDefs.td — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./include/triton/Dialect/TritonGPU/IR/TritonGPUAttrDefs.td`
- **EN:** Declares or defines attribute records and helper APIs used by this subsystem.
- **CN:** 声明或定义该子系统使用的属性记录与辅助 API。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```tablegen
   1: #ifndef TRITONGPU_ATTRDEFS
   2: #define TRITONGPU_ATTRDEFS
```
**EN:** This block establishes the header guard so the declarations in this file are only processed once per translation unit.
**CN:** 该代码块建立头文件保护宏，避免同一翻译单元重复包含本文件时出现重复声明。

### Lines 4-4
```tablegen
   4: include "triton/Dialect/TritonGPU/IR/TritonGPUAttrInterfaces.td"
```
**EN:** This block imports the direct dependencies needed here, including triton/Dialect/TritonGPU/IR/TritonGPUAttrInterfaces.td.
**CN:** 该代码块引入后续声明直接依赖的头文件，例如 triton/Dialect/TritonGPU/IR/TritonGPUAttrInterfaces.td。

### Lines 6-8
```tablegen
   6: //===----------------------------------------------------------------------===//
   7: // Shared Layout Encoding
   8: //===----------------------------------------------------------------------===//
```
**EN:** This comment block records the intent and constraints of the surrounding code: ===----------------------------------------------------------------------===// Shared Layout Encoding ===----------------------------------------------------------------------===//.
**CN:** 该注释块解释了周边代码的设计背景、意图或约束条件，帮助读者理解后续实现。

### Lines 10-14
```tablegen
  10: def SwizzledSharedEncodingAttr
  11:     : TritonGPU_Attr<"SwizzledSharedEncoding", "swizzled_shared_encoding",
  12:                      [SharedEncodingTrait, LayoutEncodingTrait,
  13:                       DeclareLayoutEncodingMethods]> {
  14:   let mnemonic = "swizzled_shared";
```
**EN:** This TableGen def record defines `SwizzledSharedEncodingAttr`.
**CN:** 该 TableGen def 记录定义了 `SwizzledSharedEncodingAttr`。

### Lines 16-19
```tablegen
  16:   let description = [{
  17: An encoding for tensors whose elements may be simultaneously accessed by
  18: different GPU threads in the programs, via shared memory. In other words,
  19: for all indices i \in Z^d, \mathcal{L}(i) = {0, 1, ..., 32*num_warps - 1}.
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 21-22
```tablegen
  21: In order to avoid shared memory bank conflicts, elements may be swizzled.
  22: Here are some examples.  In all cases, the input tensor is [0, 1, ..., n-1].
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 24-24
```tablegen
  24: 1. Basic swizzling
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 26-30
```tablegen
  26:   #ttg.swizzled_shared<{vec=1, perPhase=1, maxPhase=4, order=[1,0]}>
  27:   [ 0,  1,  2,  3],  // xor with 0
  28:   [ 5,  4,  7,  6],  // xor with 1
  29:   [10, 11,  8,  9],  // xor with 2
  30:   [15, 14, 13, 12]   // xor with 3
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 32-33
```tablegen
  32: Here elements of row r are xor'ed with r (or more properly, in[r][c] ->
  33: out[r][c^r]).
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 35-35
```tablegen
  35: 2. Multiple rows per phase
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 37-41
```tablegen
  37:   #ttg.swizzled_shared<{vec=1, perPhase=2, maxPhase=4, order=[1,0]}>
  38:   [ 0,  1,  2,  3],  // phase 0 (xor with 0)
  39:   [ 4,  5,  6,  7],
  40:   [ 9,  8, 11, 10],  // phase 1 (xor with 1)
  41:   [13, 12, 15, 14]
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 43-44
```tablegen
  43: Elements of row r are xor'ed with r/2.  In other words, perPhase=2
  44: means that pairs of 2 rows get the same swizzling.
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 46-46
```tablegen
  46: 3. Max-phase applied
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 48-56
```tablegen
  48:   #ttg.swizzled_shared<{vec=1, perPhase=1, maxPhase=2, order=[1,0]}>
  49:   [ 0,  1,  2,  3],  // phase 0 (xor with 0)
  50:   [ 5,  4,  7,  6],  // phase 1 (xor with 1)
  51:   [ 8,  9, 10, 11],  // phase 0
  52:   [13, 12, 15, 14],  // phase 1
  53:   [16, 17, 18, 19],  // ...
  54:   [21, 20, 23, 22],
  55:   [24, 25, 26, 27],
  56:   [29, 28, 31, 30]
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 58-59
```tablegen
  58: Elements of row r are xor'ed with (r/2) % 2.  In other words, maxPhase=m has the
  59: effect of limiting the maximum value of the xor to m-1.
```
**EN:** This block declares or defines callable APIs such as with, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 with 等可调用 API，用来封装这里提供的核心行为。

### Lines 61-61
```tablegen
  61: 4. Max-phase and per-phase
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 63-71
```tablegen
  63:   #ttg.swizzled_shared<{vec=1, perPhase=2, maxPhase=2, order=[1,0]}>
  64:   [ 0,  1,  2,  3],  // phase 0 (xor with 0)
  65:   [ 4,  5,  6,  7],  // phase 0
  66:   [ 9,  8, 11, 10],  // phase 1 (xor with 1)
  67:   [13, 12, 15, 14],  // phase 1
  68:   [16, 17, 18, 19],  // phase 0
  69:   [20, 21, 22, 23],  // phase 0
  70:   [25, 24, 27, 26],  // phase 1
  71:   [29, 28, 31, 30]]  // phase 1
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 73-75
```tablegen
  73: Here the xor value (the "phase", I guess?) changes every perPhase rows, up to a
  74: maximum value of maxPhase-1.  In other words, elements of row r are xor'ed with
  75: (r/2) % 2.
```
**EN:** This block declares or defines callable APIs such as value and with, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 value and with 等可调用 API，用来封装这里提供的核心行为。

### Lines 77-77
```tablegen
  77: 5. Adding vec
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 79-83
```tablegen
  79:   #ttg.swizzled_shared<{vec=2, perPhase=1, maxPhase=4, order=[1,0]}>
  80:   [ 0,  1,  2,  3,  4,  5,  6,  7],
  81:   [10, 11,  8,  9, 14, 15, 12, 13],
  82:   [20, 21, 22, 23, 16, 17, 18, 19],
  83:   [30, 31, 28, 29, 26, 27, 24, 25]
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 85-86
```tablegen
  85: When vec=2, elements are swizzled in pairs of 2.  In other words, the element at
  86: (r,c) has value
```
**EN:** This block declares or defines callable APIs such as at, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 at 等可调用 API，用来封装这里提供的核心行为。

### Lines 88-89
```tablegen
  88:   ((c / 2) ^ r) * 2 + (c % 2).
  89:   }];
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 91-100
```tablegen
  91:   // swizzle info: vec, perPhase, maxPhase
  92:   // order: the fastest-changing axis first
  93:   let parameters = (
  94:     ins
  95:     "unsigned":$vec,
  96:     "unsigned":$perPhase,
  97:     "unsigned":$maxPhase,
  98:     ArrayRefParameter<"unsigned">:$order,
  99:     "CGAEncodingAttr":$CGALayout
 100:   );
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 102-110
```tablegen
 102:   let builders = [
 103:     AttrBuilder<(ins "DotOperandEncodingAttr":$dotOpEnc,
 104:                      "ArrayRef<int64_t>":$shape,
 105:                      "ArrayRef<unsigned>":$order,
 106:                      "CGAEncodingAttr":$CGALayout,
 107:                      "unsigned":$typeWidthInBit), [{
 108:         bool needTrans = false; // default value
 109:         return get(context, dotOpEnc, shape, order, CGALayout, typeWidthInBit, needTrans);
 110:     }]>,
```
**EN:** This block adds convenience builders so the generated API can construct the entity with common argument patterns.
**CN:** 该代码块添加便捷 builder，使生成 API 能以常见参数组合快速构造该实体。

### Lines 112-120
```tablegen
 112:     // TODO(jlebar): This should not be an overload of
 113:     // SwizzledSharedEncodingAttr::get().  It's misleading, because it does a bunch of
 114:     // nontrivial work based on the given dotOpEnc.
 115:     AttrBuilder<(ins "DotOperandEncodingAttr":$dotOpEnc,
 116:                      "ArrayRef<int64_t>":$shape,
 117:                      "ArrayRef<unsigned>":$order,
 118:                      "CGAEncodingAttr":$CGALayout,
 119:                      "unsigned":$typeWidthInBit,
 120:                      "bool":$needTrans), [{
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 122-127
```tablegen
 122:         // ---- begin MFMA ----
 123:         if (auto mfmaEnc = mlir::dyn_cast<AMDMfmaEncodingAttr>(dotOpEnc.getParent())) {
 124:           return mfmaEnc.composeSharedLayoutForOperand(
 125:               CGALayout, dotOpEnc.getOpIdx(), shape, order, dotOpEnc.getKWidth(),
 126:               typeWidthInBit, needTrans);
 127:         }
```
**EN:** This block declares or defines callable APIs such as getParent, composeSharedLayoutForOperand, getOpIdx, and getKWidth, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getParent, composeSharedLayoutForOperand, getOpIdx, and getKWidth 等可调用 API，用来封装这里提供的核心行为。

### Lines 129-134
```tablegen
 129:         // ---- begin WMMA ----
 130:         if (auto wmmaEnc = mlir::dyn_cast<AMDWmmaEncodingAttr>(dotOpEnc.getParent())) {
 131:           return wmmaEnc.composeSharedLayoutForOperand(
 132:               CGALayout, dotOpEnc.getOpIdx(), shape, order, dotOpEnc.getKWidth(),
 133:               typeWidthInBit, needTrans);
 134:         }
```
**EN:** This block declares or defines callable APIs such as getParent, composeSharedLayoutForOperand, getOpIdx, and getKWidth, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getParent, composeSharedLayoutForOperand, getOpIdx, and getKWidth 等可调用 API，用来封装这里提供的核心行为。

### Lines 137-137
```tablegen
 137:         auto mmaEnc = mlir::dyn_cast<NvidiaMmaEncodingAttr>(dotOpEnc.getParent());
```
**EN:** This block declares or defines callable APIs such as getParent, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getParent 等可调用 API，用来封装这里提供的核心行为。

### Lines 139-140
```tablegen
 139:         if(!mmaEnc)
 140:           return get(context, 1, 1, 1, order, CGALayout);
```
**EN:** This block declares or defines callable APIs such as get, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 get 等可调用 API，用来封装这里提供的核心行为。

### Lines 142-145
```tablegen
 142:         // ---- begin Ampere & Hopper ----
 143:         if (mmaEnc.isAmpere() || mmaEnc.isHopper()) {
 144:           return get(context, dotOpEnc.getOpIdx(), dotOpEnc.getKWidth(), shape, order, CGALayout, typeWidthInBit, needTrans);
 145:         }
```
**EN:** This block declares or defines callable APIs such as isAmpere, isHopper, get, getOpIdx, and getKWidth, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isAmpere, isHopper, get, getOpIdx, and getKWidth 等可调用 API，用来封装这里提供的核心行为。

### Lines 147-149
```tablegen
 147:         // ---- not implemented ----
 148:         llvm_unreachable("unsupported swizzling for provided MMA version");
 149:     }]>,
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 151-163
```tablegen
 151:     // NVIDIA constructor!
 152:     // TODO(lezcano): We should totally get rid of all these constructors...
 153:     AttrBuilder<(ins "int":$opIdx,
 154:                      "unsigned":$kWidth,
 155:                      "ArrayRef<int64_t>":$shape,
 156:                      "ArrayRef<unsigned>":$order,
 157:                      "CGAEncodingAttr":$CGALayout,
 158:                      "unsigned":$bitwidth,
 159:                      "bool":$needTrans), [{
 160:         int K =  getShapePerCTA(CGALayout.getCTASplitNum(), shape)[order[0]];
 161:         // Elems necessary to cover all the banks divided by the inner dimension
 162:         // This packs a few rows together for small K
 163:         int perPhase = std::max<int>(1024 / (bitwidth * K), 1);
```
**EN:** This block declares or defines callable APIs such as getShapePerCTA and getCTASplitNum, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getShapePerCTA and getCTASplitNum 等可调用 API，用来封装这里提供的核心行为。

### Lines 165-180
```tablegen
 165:         int mmaStride = 8;
 166:         int vec = 4 * kWidth;
 167:         // needsTrans is equiv. to flipping the opIdx
 168:         if (needTrans)
 169:           std::swap(vec, mmaStride);
 170:         assert(opIdx == 0 || opIdx == 1);
 171:         int rank = order.size();
 172:         int kDim = opIdx == 0 ? rank-1 : rank-2;
 173:         if (order[0] != kDim)
 174:           std::swap(vec, mmaStride);
 175:         // Count how many vec elements are needed to cover all the banks
 176:         int maxPhase = std::max(std::min<int>(mmaStride, 1024 / (vec * bitwidth)), 1);
 177:         // Account for the row packing from perPhase: mmaStride / perPhase
 178:         maxPhase = std::max(maxPhase / perPhase, 1);
 179:         return get(context, vec, perPhase, maxPhase, order, CGALayout);
 180:     }]>,
```
**EN:** This block declares or defines callable APIs such as swap, size, max, and get, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 swap, size, max, and get 等可调用 API，用来封装这里提供的核心行为。

### Lines 182-189
```tablegen
 182:     AttrBuilder<(ins "DotOperandEncodingAttr":$dotOpEnc,
 183:                      "ArrayRef<int64_t>":$shape,
 184:                      "ArrayRef<unsigned>":$order,
 185:                      "CGAEncodingAttr":$CGALayout,
 186:                      "Type":$eltTy), [{
 187:       unsigned bitwidth = eltTy.getIntOrFloatBitWidth();
 188:       return get(context, dotOpEnc, shape, order, CGALayout, bitwidth);
 189:     }]>,
```
**EN:** This block declares or defines callable APIs such as getIntOrFloatBitWidth and get, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getIntOrFloatBitWidth and get 等可调用 API，用来封装这里提供的核心行为。

### Lines 191-200
```tablegen
 191:     AttrBuilder<(ins "DotOperandEncodingAttr":$dotOpEnc,
 192:                      "ArrayRef<int64_t>":$shape,
 193:                      "ArrayRef<unsigned>":$order,
 194:                      "CGAEncodingAttr":$CGALayout,
 195:                      "Type":$eltTy,
 196:                      "bool":$needTrans), [{
 197:       unsigned bitwidth = eltTy.getIntOrFloatBitWidth();
 198:       return get(context, dotOpEnc, shape, order, CGALayout, bitwidth, needTrans);
 199:     }]>,
 200:   ];
```
**EN:** This block declares or defines callable APIs such as getIntOrFloatBitWidth and get, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getIntOrFloatBitWidth and get 等可调用 API，用来封装这里提供的核心行为。

### Lines 202-204
```tablegen
 202:   let hasCustomAssemblyFormat = 1;
 203:   let genVerifyDecl = 1;
 204: }
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 206-209
```tablegen
 206: def PaddedSharedEncodingAttr
 207:     : TritonGPU_Attr<"PaddedSharedEncoding", "padded_shared_encoding",
 208:                      [SharedEncodingTrait, DeclareLayoutEncodingMethods]> {
 209:   let mnemonic = "padded_shared";
```
**EN:** This TableGen def record defines `PaddedSharedEncodingAttr`.
**CN:** 该 TableGen def 记录定义了 `PaddedSharedEncodingAttr`。

### Lines 211-217
```tablegen
 211:   let description = [{
 212: An encoding for tensors whose elements may be simultaneously accessed by
 213: different GPU threads in the programs, via shared memory. In other words,
 214: for all indices i \in Z^d, \mathcal{L}(i) = {0, 1, ..., 32*num_warps - 1}.
 215: Compared to SwizzledSharedEncodingAttr, this encoding combines padding with
 216: element reordering via linear transformation (e.g. row permutation) to avoid
 217: shared memory bank conflicts.
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 219-226
```tablegen
 219: Formally, given a layout:
 220:     padded_shared<[<interval_0>:+<pad_0>, <interval_1>:+<pad_1>, ...]>
 221: We insert a padding of `<pad_i>` elements after every `<interval_i>` elements.
 222: Multi interval-padding pairs are supported for flexibility of multi tiered
 223: padding schemes; they compose in an additive manner. So for a 1-D tensor element
 224: at index i, the corresponding shared memory location index is
 225:     i + \sum_{k} (i / interval_k) * pad_k = 1
 226: `<interval_i>` and `<pad_i>` all need to be power of two.
```
**EN:** This block stores supporting state such as schemes, which other APIs in the file consume.
**CN:** 该代码块声明了 schemes 等支撑状态，供本文件中的其他 API 使用。

### Lines 228-229
```tablegen
 228: Some concrete examples ignoring the linear component, using `eM` to mean tensor
 229: elements and `pN` to mean padding:
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 231-231
```tablegen
 231: 1. Single interval-padding pair:
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 233-236
```tablegen
 233:    #ttg.padded_shared<[2:+2], {...}>
 234:    [e0, e1, p0, p1,
 235:     e2, e3, p2, p3,
 236:     ...]
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 238-238
```tablegen
 238: 2. Double interval-padding pairs:
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 240-245
```tablegen
 240:    #ttg.padded_shared<[2:+1, 4:+2], {...}>
 241:    [e0, e1, p0,
 242:     e2, e3, p1, p2, p3,
 243:     e4, e5, p4,
 244:     e6, e7, p5, p6, p7,
 245:     ...]
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 247-253
```tablegen
 247: Furthermore this encoding allows for a linear remapping from the 1-D shared
 248: memory offset to logical n-D tensor elements. The remapping is given in the form
 249: of linear bases mapping from offset to [dim0, dim1...dimN-1].
 250: See LinearLayout.h for more details how linear layouts are applied to remap
 251: elements.
 252: Some concrete examples using `xN` and `yN` to mean the logical n-D tensor elements
 253: and `pN` to mean padding:
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 255-255
```tablegen
 255: 1. 1D Single interval-padding with strided elements
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 257-260
```tablegen
 257:     #ttg.padded_shared<[2:+2] {offset = [[2], [1]], block = []}>
 258:     [x0, x2, p0 p1,
 259:      x1, x3, p2, p3
 260:      ...]
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 262-262
```tablegen
 262: 2. 2D single interval-padding with rearranged rows.
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 264-276
```tablegen
 264:     #ttg.padded_shared<[16:+1] {offset = [[0, 1], [0, 2], /*gap, stride by 2 rows*/[2, 0], [4, 0], [1, 0]]], block = []}>
 265:     [
 266:       x0y0, x0y1, x0y2, x0y3,
 267:       x2y0, x2y1, x2y2, x2y3,
 268:       x4y0, x4y1, x4y2, x4y3,
 269:       x6y0, x6y1, x6y2, x6y3,
 270:       p0,
 271:       x1y0, x1y1, x1y2, x1y3,
 272:       x3y0, x3y1, x3y2, x3y3,
 273:       x5y0, x5y1, x5y2, x5y3,
 274:       x7y0, x7y1, x7y2, x7y3,
 275:       p1,
 276:     ]
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 278-278
```tablegen
 278: For identity mappings a short form based on order and shape is used to increase readability. The following two encodings are the same:
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 280-281
```tablegen
 280:     #ttg.padded_shared<[2:+2] {order = [1, 0], shape = [16, 32]}>
 281:     #ttg.padded_shared<[2:+2] {offset = [[0, 1], [0, 2], [0, 4], [0, 8], [0, 16], [1, 0], [2, 0], [4, 0], [8, 0]], block = []}>
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 284-284
```tablegen
 284:   }];
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 286-290
```tablegen
 286:   let parameters = (ins
 287:       ArrayRefParameter<"unsigned">:$intervals,
 288:       ArrayRefParameter<"unsigned">:$paddings,
 289:       LinearLayoutParam:$linearComponent
 290:   );
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 292-294
```tablegen
 292:   let builders = [
 293:       AttrBuilder<(ins "ArrayRef<std::pair<unsigned, unsigned>>":$intervalPads,
 294:                        "LinearLayout":$linearComponent)>,
```
**EN:** This block adds convenience builders so the generated API can construct the entity with common argument patterns.
**CN:** 该代码块添加便捷 builder，使生成 API 能以常见参数组合快速构造该实体。

### Lines 296-300
```tablegen
 296:       // Builder to create an identity mapping as the linear component
 297:       AttrBuilder<(ins "ArrayRef<std::pair<unsigned, unsigned>>":$intervalPads,
 298:                        "ArrayRef<unsigned>":$order, "ArrayRef<int64_t>":$shape,
 299:                        "CGAEncodingAttr":$cgaLayout)>,
 300:   ];
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 302-308
```tablegen
 302:   let extraClassDeclaration = extraBaseClassDeclaration # [{
 303:     // Returns the order of the dimensions `dimName` of the layout.
 304:     // If more than one dimension is of size one, it uses defaultOrder to determine
 305:     // the order of the dimensions of size one.
 306:     SmallVector<unsigned> orderPerDim(StringAttr dimName,
 307:                                       ArrayRef<unsigned> defaultOrder) const;
 308:     SmallVector<unsigned> getOrder() const;
```
**EN:** This block injects extra C++ helpers into the generated class, such as orderPerDim and getOrder.
**CN:** 该代码块向生成类中注入额外的 C++ 辅助方法，例如 orderPerDim and getOrder。

### Lines 310-313
```tablegen
 310:     // Returns the bases of the dimensions `dimName` of the linear_component.
 311:     // If skipBroadcast is false, we count a base zero
 312:     SmallVector<unsigned> basesPerDim(StringAttr dimName,
 313:                                       bool skipBroadcast = true) const;
```
**EN:** This block declares or defines callable APIs such as basesPerDim, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 basesPerDim 等可调用 API，用来封装这里提供的核心行为。

### Lines 315-317
```tablegen
 315:     unsigned getMinInterval() const {
 316:       return *llvm::min_element(getIntervals());
 317:     }
```
**EN:** This block declares or defines callable APIs such as getMinInterval, min_element, and getIntervals, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getMinInterval, min_element, and getIntervals 等可调用 API，用来封装这里提供的核心行为。

### Lines 319-325
```tablegen
 319:     // Returns the total number of elements including padding given the input
 320:     // tensor shape.
 321:     int64_t getPaddedSize(ArrayRef<int64_t> shape) const;
 322:   }];
 323:   let hasCustomAssemblyFormat = 1;
 324:   let genVerifyDecl = 1;
 325: }
```
**EN:** This block declares or defines callable APIs such as getPaddedSize, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getPaddedSize 等可调用 API，用来封装这里提供的核心行为。

### Lines 328-331
```tablegen
 328: def PartitionedSharedEncodingAttr
 329:     : TritonGPU_Attr<"PartitionedSharedEncoding", "partitioned_shared_encoding",
 330:                      [SharedEncodingTrait, DeclareLayoutEncodingMethods,  LayoutEncodingTrait]> {
 331:   let mnemonic = "partitioned_shared";
```
**EN:** This TableGen def record defines `PartitionedSharedEncodingAttr`.
**CN:** 该 TableGen def 记录定义了 `PartitionedSharedEncodingAttr`。

### Lines 333-337
```tablegen
 333:   let description = [{
 334:     An encoding for tensors whose elements are partitioned across multiple
 335:     separate shared memory allocations. This reduces shared memory partition
 336:     conflicts by splitting a tensor along a specific dimension into separate
 337:     allocations.
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 339-346
```tablegen
 339:     Parameters:
 340:     - numPartitions: Number of distinct memory partitions (and separate buffers).
 341:       Buffers in different partitions MUST be placed in different physical
 342:       shared memory slots.
 343:     - numGroups: Number of groups. Each group contains numPartitions
 344:       consecutive pieces of the tensor.
 345:     - partitionDim: The dimension along which the tensor is partitioned.
 346:     - partitionLayout: The shared memory layout used within each piece.
```
**EN:** This block declares or defines callable APIs such as partitions, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 partitions 等可调用 API，用来封装这里提供的核心行为。

### Lines 348-352
```tablegen
 348:     The total number of logical pieces is numPartitions * numGroups.
 349:     Pieces are organized as:
 350:       [Group 0: pieces 0..numPartitions-1]
 351:       [Group 1: pieces numPartitions..2*numPartitions-1]
 352:       ...
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 354-354
```tablegen
 354:     ## Memory Allocation
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 356-358
```tablegen
 356:     The allocator creates numPartitions buffers (NOT numLogicalPieces buffers).
 357:     Each buffer contains all pieces from all groups that belong to that partition,
 358:     concatenated together. Buffer size = pieceSize * numGroups.
```
**EN:** This block declares or defines callable APIs such as buffers, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 buffers 等可调用 API，用来封装这里提供的核心行为。

### Lines 360-365
```tablegen
 360:     For example, with numPartitions=2, numGroups=4 and
 361:     partitionDim=0 on a [128, 32] tensor:
 362:     - Total 8 logical pieces, each of size [16, 32] (pieceSize = 16*32 elements)
 363:     - Piece layout: [0, 1, 2, 3, 4, 5, 6, 7]
 364:     - Partitions:    0  1  0  1  0  1  0  1
 365:     - Groups:       |Grp0||Grp1||Grp2||Grp3|
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 367-369
```tablegen
 367:     Physical allocation (2 buffers, each containing 4 pieces):
 368:     - Buffer 0 (Partition 0): [Piece0 | Piece2 | Piece4 | Piece6]
 369:     - Buffer 1 (Partition 1): [Piece1 | Piece3 | Piece5 | Piece7]
```
**EN:** This block declares or defines callable APIs such as allocation, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 allocation 等可调用 API，用来封装这里提供的核心行为。

### Lines 371-373
```tablegen
 371:     Physical allocation guarantee: Buffers in different partitions MUST reside
 372:     in distinct physical shared memory partitions.
 373:   }];
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 375-380
```tablegen
 375:   let parameters = (ins
 376:       "unsigned":$numPartitions,
 377:       "unsigned":$numGroups,
 378:       "unsigned":$partitionDim,
 379:       "SharedEncodingTrait":$partitionLayout
 380:   );
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 382-387
```tablegen
 382:   let extraClassDeclaration = [{
 383:     /// Returns the total number of logical pieces.
 384:     unsigned getNumLogicalPieces() const {
 385:       return getNumPartitions() * getNumGroups();
 386:     }
 387:   }];
```
**EN:** This block injects extra C++ helpers into the generated class, such as getNumLogicalPieces, getNumPartitions, and getNumGroups.
**CN:** 该代码块向生成类中注入额外的 C++ 辅助方法，例如 getNumLogicalPieces, getNumPartitions, and getNumGroups。

### Lines 389-391
```tablegen
 389:   let hasCustomAssemblyFormat = 1;
 390:   let genVerifyDecl = 1;
 391: }
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 393-397
```tablegen
 393: def SharedLinearEncodingAttr
 394:     : TritonGPU_Attr<"SharedLinearEncoding", "shared_linear_encoding",
 395:                      [SharedEncodingTrait, LayoutEncodingTrait,
 396:                       DeclareLayoutEncodingMethods]> {
 397:   let mnemonic = "shared_linear";
```
**EN:** This TableGen def record defines `SharedLinearEncodingAttr`.
**CN:** 该 TableGen def 记录定义了 `SharedLinearEncodingAttr`。

### Lines 399-403
```tablegen
 399:   let description = [{
 400:     Linear shared encodings mirror LinearEncodingAttr but operate on shared
 401:     memory layouts. The LinearLayout parameter captures how shared memory
 402:     offsets (and optionally blocks) map to logical tensor indices.
 403:   }];
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 405-405
```tablegen
 405:   let parameters = (ins LinearLayoutParam:$linearLayout, "unsigned":$layoutAlignment);
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 407-411
```tablegen
 407:   let extraClassDeclaration = [{
 408:     SmallVector<unsigned> basesPerDim(StringAttr dimName,
 409:                                       bool skipBroadcast = true) const;
 410:     SmallVector<unsigned> orderPerDim(StringAttr dimName,
 411:                                       ArrayRef<unsigned> defaultOrder) const;
```
**EN:** This block injects extra C++ helpers into the generated class, such as basesPerDim and orderPerDim.
**CN:** 该代码块向生成类中注入额外的 C++ 辅助方法，例如 basesPerDim and orderPerDim。

### Lines 413-413
```tablegen
 413:     SmallVector<unsigned> getOrder() const;
```
**EN:** This block declares or defines callable APIs such as getOrder, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getOrder 等可调用 API，用来封装这里提供的核心行为。

### Lines 415-415
```tablegen
 415:     unsigned getRank() const { return getLinearLayout().getNumOutDims(); }
```
**EN:** This block declares or defines callable APIs such as getRank, getLinearLayout, and getNumOutDims, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getRank, getLinearLayout, and getNumOutDims 等可调用 API，用来封装这里提供的核心行为。

### Lines 417-417
```tablegen
 417:     LinearLayout toLinearLayout(ArrayRef<int64_t> shape) const;
```
**EN:** This block declares or defines callable APIs such as toLinearLayout, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 toLinearLayout 等可调用 API，用来封装这里提供的核心行为。

### Lines 419-420
```tablegen
 419:     int32_t getAlignment() const { return static_cast<int32_t>(getLayoutAlignment()); }
 420:   }];
```
**EN:** This block declares or defines callable APIs such as getAlignment and getLayoutAlignment, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getAlignment and getLayoutAlignment 等可调用 API，用来封装这里提供的核心行为。

### Lines 422-424
```tablegen
 422:   let genVerifyDecl = 1;
 423:   let hasCustomAssemblyFormat = 1;
 424: }
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 426-429
```tablegen
 426: def NVMMASharedEncodingAttr : TritonGPU_Attr<"NVMMASharedEncoding", "nvmma_shared_encoding",
 427:                      [DeclareSharedEncodingMethods, LayoutEncodingTrait,
 428:                       DeclareLayoutEncodingMethods]> {
 429:   let mnemonic = "nvmma_shared";
```
**EN:** This TableGen def record defines `NVMMASharedEncodingAttr`. It is specialized from `TritonGPU_Attr<"NVMMASharedEncoding", "nvmma_shared_encoding",`.
**CN:** 该 TableGen def 记录定义了 `NVMMASharedEncodingAttr`。 它基于 `TritonGPU_Attr<"NVMMASharedEncoding", "nvmma_shared_encoding",` 进一步特化。

### Lines 431-438
```tablegen
 431:   let description = [{
 432:     Represent blocked shared memory matching MMAv3/MMAv5 shared memory input.
 433:     This is meant to represent 2d tiled blocked layout.
 434:     The full layout representation is described here:
 435:     https://docs.nvidia.com/cuda/parallel-thread-execution/#asynchronous-warpgroup-level-matrix-shared-memory-layout
 436:     When the memdesc has more than 2 dimensions the tiling is applied to 8 rows even if the first outer dimension is smaller than 8.
 437:     In this case `transposed` means that the contiguous dimension is the most outer dimension of the memdesc.
 438:   }];
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 441-450
```tablegen
 441:   // fp4Padded: Indicates that this encoding represents a mixed-precision fp4 operand in MMAv5 scaled dot, which needs
 442:   // to be in the special padded layout as described in https://docs.nvidia.com/cuda/parallel-thread-execution/#packing-format-used-for-matrix-a-and-b-by-kind-mxf8f6f4-in-shared-memory
 443:   let parameters = (
 444:     ins
 445:     "unsigned":$swizzlingByteWidth,
 446:     "bool":$transposed,
 447:     "unsigned":$elementBitWidth,
 448:     "bool":$fp4Padded,
 449:     "CGAEncodingAttr":$CGALayout
 450:   );
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 452-461
```tablegen
 452:   let builders = [
 453:     AttrBuilder<(ins "ArrayRef<int64_t>":$shape,
 454:                      "ArrayRef<unsigned>":$order,
 455:                      "CGAEncodingAttr":$CGALayout,
 456:                      "Type":$eltTy,
 457:                      "bool": $fp4Padded), [{
 458:         auto shapePerCTA = getShapePerCTA(CGALayout.getCTASplitNum(), shape);
 459:         int32_t swizzlingByteWidth = 0;
 460:         unsigned eleBitWidth = eltTy.getIntOrFloatBitWidth();
 461:         int packingFactor = fp4Padded ? 2 : 1;
```
**EN:** This block adds convenience builders so the generated API can construct the entity with common argument patterns.
**CN:** 该代码块添加便捷 builder，使生成 API 能以常见参数组合快速构造该实体。

### Lines 463-485
```tablegen
 463:         // get proper shared memory swizzling mode from the contiguous dimension
 464:         // size of the origin blocked layout.
 465:         auto contigDimSizeInByte = shapePerCTA[order[0]] * packingFactor * eleBitWidth / 8;
 466:         if (contigDimSizeInByte >= 128 && contigDimSizeInByte % 128 == 0) {
 467:           swizzlingByteWidth = 128;
 468:         } else if (contigDimSizeInByte >= 64 && contigDimSizeInByte % 64 == 0) {
 469:           swizzlingByteWidth = 64;
 470:         } else if (contigDimSizeInByte >= 32 && contigDimSizeInByte % 32 == 0) {
 471:           swizzlingByteWidth = 32;
 472:         } else {
 473:           swizzlingByteWidth = 0;
 474:         }
 475:         int flattenOutterDim = 1;
 476:         for (int i = 1; i < shapePerCTA.size(); i++) {
 477:           flattenOutterDim *= shapePerCTA[order[i]];
 478:         }
 479:         if (shapePerCTA.size() < 2 || flattenOutterDim < 8) {
 480:           swizzlingByteWidth = 0;
 481:         }
 482:         bool transposed = order.size() > 1 && order[0] == 0;
 483:         return $_get(context, swizzlingByteWidth, transposed, eleBitWidth, fp4Padded, CGALayout);
 484:     }]>
 485:   ];
```
**EN:** This block declares or defines callable APIs such as size and _get, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 size and _get 等可调用 API，用来封装这里提供的核心行为。

### Lines 487-494
```tablegen
 487:   let extraClassDeclaration = extraBaseClassDeclaration # [{
 488:     int getPerPhase() const;
 489:     int getMaxPhase() const;
 490:     int getVec() const;
 491:   }];
 492:   let hasCustomAssemblyFormat = 1;
 493:   let genVerifyDecl = 1;
 494: }
```
**EN:** This block injects extra C++ helpers into the generated class, such as getPerPhase, getMaxPhase, and getVec.
**CN:** 该代码块向生成类中注入额外的 C++ 辅助方法，例如 getPerPhase, getMaxPhase, and getVec。

### Lines 496-500
```tablegen
 496: def AMDRotatingSharedEncodingAttr :
 497:   TritonGPU_Attr<"AMDRotatingSharedEncoding", "amd_rotating_shared_encoding",
 498:                  [SharedEncodingTrait, LayoutEncodingTrait,
 499:                   DeclareLayoutEncodingMethods]> {
 500:   let mnemonic = "amd_rotating_shared";
```
**EN:** This TableGen def record defines `AMDRotatingSharedEncodingAttr`.
**CN:** 该 TableGen def 记录定义了 `AMDRotatingSharedEncodingAttr`。

### Lines 502-508
```tablegen
 502:   let description = [{
 503: This shared encoding is similar to SwizzledSharedEncodingAttr, but instead of
 504: repeating swizzling pattern every `maxPhase*perPhase` rows of the memory object,
 505: called a block, this layout changes swizzling pattern `maxPhase` times, then
 506: repeats the pattern. The name "rotating" comes from the fact that first tensor
 507: element of each block is swizzled with different phase, which is equal to
 508: current block number: 0, 1, 2.. maxPhase-1, 0, 1, 2 ...
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 510-512
```tablegen
 510: This layout is used to reduce bank conflicts in cases where shared memory writes
 511: and reads are performed on layouts with different order. It's meant for hardware
 512: without native shared memory tranpose support.
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 514-517
```tablegen
 514: Swizzling pattern affects only 2 fastest dimensions of a tensor.
 515: In the following text these two dimensions are called row and column:
 516: - row is a fastest dimension
 517: - column is a second fastest dimension
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 519-519
```tablegen
 519: Elements in a row dimension are stored in memory contiguously.
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 521-524
```tablegen
 521: If a matrix of size [128x64] is stored in this shared layout with order [1, 0],
 522: dim 1 (64) will be stored contiguously and called row, dim 0 (128) is will be
 523: called column. If order of shared layout is [0, 1], dim 0 (128) is stored
 524: contiguously becomes a row, dim 1 (64) becomes a column.
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 526-526
```tablegen
 526: Swizzling pattern is following:
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 528-531
```tablegen
 528: Let's consider an element with logical coordinates = (inRowId, inColId).
 529: For simplicity, we do not vectorize memory in examples,
 530: i.e. vec == 1 and layout swizzles inidividual elements.
 531: For vec != 1 example, take a look at SwizzledSharedEncodingAttr documentation.
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 533-533
```tablegen
 533: Swizzled coordinates within memory object are (outRowId, outColId):
```
**EN:** This block declares or defines callable APIs such as are, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 are 等可调用 API，用来封装这里提供的核心行为。

### Lines 535-539
```tablegen
 535:   outRowId = inRowId
 536:   phase   = (inRowId / perPhase) % maxPhase
 537:   blockNo = (inRowId / (perPhase * maxPhase)) % maxPhase
 538:   combinedPhase = phase ^ blockNo
 539:   outColId   = inColId ^ combinedPhase
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 541-541
```tablegen
 541: Actual offset in memory could be computed with following function:
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 543-543
```tablegen
 543: memmory_offset = (outColId + outRowId * num_of_element_in_row) * sizeof(element)
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 546-546
```tablegen
 546: Swizzling examples (matrix is filled with numbers 0, 1, 2, .. columns*rows-1):
```
**EN:** This block declares or defines callable APIs such as examples, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 examples 等可调用 API，用来封装这里提供的核心行为。

### Lines 548-557
```tablegen
 548:   #shared<{vec=1, perPhase=1, maxPhase=2, order=[1,0]}>
 549:   row      elements
 550:     0  [ 0,  1,  2,  3],  // phase = 0 blockNo = 0 (xor with 0)
 551:     1  [ 5,  4,  7,  6],  // phase = 1 blockNo = 0 (xor with 1)
 552:     2  [ 9,  8, 11, 10],  // phase = 0 blockNo = 1 (xor with 1)
 553:     3  [12, 13, 14, 15]   // phase = 1 blockNo = 1 (xor with 0)
 554:     4  [16, 17, 18, 19],  // phase = 0 blockNo = 0 (xor with 0)
 555:     5  [21, 20, 23, 22],  // phase = 1 blockNo = 0 (xor with 1)
 556:     6  [25, 24, 27, 26],  // phase = 0 blockNo = 1 (xor with 1)
 557:     7  [28, 29, 30, 31]   // phase = 1 blockNo = 1 (xor with 0)
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 559-568
```tablegen
 559:   #shared<{vec=1, perPhase=2, maxPhase=2, order=[1,0]}>
 560:   row      elements
 561:     0  [ 0,  1,  2,  3],  // phase = 0 blockNo = 0 (xor with 0)
 562:     1  [ 4,  5,  6,  7],  // phase = 0 blockNo = 0 (xor with 0)
 563:     2  [ 9,  8, 11, 10],  // phase = 1 blockNo = 0 (xor with 1)
 564:     3  [13, 12, 15, 14]   // phase = 1 blockNo = 0 (xor with 1)
 565:     4  [17, 16, 19, 18],  // phase = 0 blockNo = 1 (xor with 1)
 566:     5  [21, 20, 23, 22],  // phase = 0 blockNo = 1 (xor with 1)
 567:     6  [24, 25, 26, 27],  // phase = 1 blockNo = 1 (xor with 0)
 568:     7  [28, 29, 30, 31]   // phase = 1 blockNo = 1 (xor with 0)
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 570-580
```tablegen
 570:   #shared<{vec=1, perPhase=1, maxPhase=4, order=[1,0]}>
 571:   row      elements
 572:     0  [ 0,  1,  2,  3],  // phase = 0 blockNo = 0 (xor with 0)
 573:     1  [ 5,  4,  7,  6],  // phase = 1 blockNo = 0 (xor with 1)
 574:     2  [10, 11,  8,  9],  // phase = 2 blockNo = 0 (xor with 2)
 575:     3  [15, 14, 13, 12]   // phase = 3 blockNo = 0 (xor with 3)
 576:     4  [17, 16, 19, 18],  // phase = 0 blockNo = 1 (xor with 1)
 577:     5  [20, 21, 22, 23],  // phase = 1 blockNo = 1 (xor with 0)
 578:     6  [27, 26, 25, 24],  // phase = 2 blockNo = 1 (xor with 3)
 579:     7  [30, 31, 28, 29]   // phase = 3 blockNo = 1 (xor with 2)
 580:   }];
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 582-589
```tablegen
 582:   let parameters = (
 583:     ins
 584:     "unsigned":$vec,
 585:     "unsigned":$perPhase,
 586:     "unsigned":$maxPhase,
 587:     ArrayRefParameter<"unsigned">:$order,
 588:     "CGAEncodingAttr":$CGALayout
 589:   );
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 591-592
```tablegen
 591:   let hasCustomAssemblyFormat = 1;
 592: }
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 595-599
```tablegen
 595: class DistributedEncoding<string name, string attrMnemonic, list<Trait> traits = []>
 596:   : TritonGPU_Attr<name, attrMnemonic,
 597:                    !listconcat([DistributedEncodingTrait, LayoutEncodingTrait,
 598:                                 DeclareLayoutEncodingMethods],
 599:                                traits)> {
```
**EN:** This block introduces `DistributedEncoding`, the main class/struct defined here. Within the declaration, methods such as listconcat expose its core API.
**CN:** 该代码块引入了此文件的核心类/结构体 `DistributedEncoding`。 其中 listconcat 等方法构成了它的主要接口。

### Lines 601-604
```tablegen
 601:   let description = [{
 602: Distributed encodings have a layout function L that is entirely characterized
 603: by a d-dimensional tensor T. Note that L doesn't need to have the same shape
 604: (or even the same rank) as the tensor it is encoding.
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 606-607
```tablegen
 606: The layout function \mathcal{L} of this layout is then defined, for an
 607: index `i` \in Z^d, as follows:
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 609-609
```tablegen
 609: \mathcal{L}(T)[i_d] = L[(i_d + k_d*T.shape[d]) % L.shape[d]] \forall k_d such as i_d + k_d*T.shape[d] < L.shape[d]
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 611-614
```tablegen
 611: Intuitively, when the tensor dim size T.shape[d] is larger than the layout
 612: dim size L.shape[d], on that particular dim, we distribute values from the
 613: tensor to threads mapped in the layout in a "wrapped around" manner, with
 614: each thread owning multiple values.
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 616-619
```tablegen
 616: OTOH, when the tensor dim size T.shape[d] is smaller than the layout
 617: dim size L.shape[d], on that particular dim, we distribute values from the
 618: tensor to threads mapped in the layout in a "broadcasted" manner, with
 619: each value owned by multiple threads.
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 621-627
```tablegen
 621: For example, for a tensor/layout pair
 622: T = [x  x  x  x  x  x  x  x]
 623:     [x  x  x  x  x  x  x  x]
 624: L = [0  1  2  3 ]
 625:     [4  5  6  7 ]
 626:     [8  9  10 11]
 627:     [12 13 14 15]
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 629-632
```tablegen
 629: Then the data of T would be distributed as follow between the 16 CUDA threads:
 630: L(T) = [ {0,8} , {1,9} , {2,10}, {3,11}, {0,8} , {1, 9} , {2, 10}, {3, 11},
 631:          {4,12}, {5,13}, {6,14}, {7,15}, {4,12}, {5, 13}, {6, 14}, {7, 15} ]
 632:   }];
```
**EN:** This block declares or defines callable APIs such as L, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 L 等可调用 API，用来封装这里提供的核心行为。

### Lines 634-636
```tablegen
 634:   code extraDistributedDeclaration  = extraBaseClassDeclaration # [{
 635:     // Implemented in subclasses
 636:     SmallVector<unsigned> getRepOrder() const;
```
**EN:** This block declares or defines callable APIs such as getRepOrder, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getRepOrder 等可调用 API，用来封装这里提供的核心行为。

### Lines 638-640
```tablegen
 638:     LinearLayout toLinearLayout(ArrayRef<int64_t> shape) const;
 639:   }];
 640: }
```
**EN:** This block declares or defines callable APIs such as toLinearLayout, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 toLinearLayout 等可调用 API，用来封装这里提供的核心行为。

### Lines 642-644
```tablegen
 642: //===----------------------------------------------------------------------===//
 643: // Linear Layout Encodings (common base class)
 644: //===----------------------------------------------------------------------===//
```
**EN:** This comment block records the intent and constraints of the surrounding code: ===----------------------------------------------------------------------===// Linear Layout Encodings (common base class) ===---------------------------------------------------....
**CN:** 该注释块解释了周边代码的设计背景、意图或约束条件，帮助读者理解后续实现。

### Lines 646-650
```tablegen
 646: class LinearDistributedEncoding<string name, string attrMnemonic,
 647:                                 list<Trait> traits = []>
 648:   : DistributedEncoding<name, attrMnemonic,
 649:       !listconcat([LinearEncodingTrait, DeclareLinearEncodingMethods],
 650:                   traits)> {
```
**EN:** This block introduces `LinearDistributedEncoding`, the main class/struct defined here. Within the declaration, methods such as listconcat expose its core API.
**CN:** 该代码块引入了此文件的核心类/结构体 `LinearDistributedEncoding`。 其中 listconcat 等方法构成了它的主要接口。

### Lines 652-652
```tablegen
 652:   let parameters = (ins LinearLayoutParam:$linearLayout);
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 654-657
```tablegen
 654:   code extraLinearDistributedDeclaration = [{
 655:     unsigned getRank() const { return getLinearLayout().getNumOutDims(); }
 656:   }];
 657: }
```
**EN:** This block declares or defines callable APIs such as getRank, getLinearLayout, and getNumOutDims, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getRank, getLinearLayout, and getNumOutDims 等可调用 API，用来封装这里提供的核心行为。

### Lines 659-661
```tablegen
 659: //===----------------------------------------------------------------------===//
 660: // Linear Layout Encoding
 661: //===----------------------------------------------------------------------===//
```
**EN:** This comment block records the intent and constraints of the surrounding code: ===----------------------------------------------------------------------===// Linear Layout Encoding ===----------------------------------------------------------------------===//.
**CN:** 该注释块解释了周边代码的设计背景、意图或约束条件，帮助读者理解后续实现。

### Lines 663-665
```tablegen
 663: def LinearEncodingAttr
 664:     : LinearDistributedEncoding<"LinearEncoding", "linear_encoding"> {
 665:   let mnemonic = "linear";
```
**EN:** This TableGen def record defines `LinearEncodingAttr`.
**CN:** 该 TableGen def 记录定义了 `LinearEncodingAttr`。

### Lines 667-671
```tablegen
 667:   let description = [{
 668:     An encoding backed by a LinearLayout with input dims
 669:     {register, lane, warp, block} and output dims {dim0, dim1, ...}. See the
 670:     docs in LinearLayout.h for the general definition of linear layouts;
 671:     this attribute is a strictly more restricted form of that abstraction.
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 673-679
```tablegen
 673:     Compared to GenericLinearEncodingAttr, this is more constrained
 674:     LinearLayout-based encoding:
 675:     - Every basis vector (register, lane, warp, and block) must be
 676:       non-swizzled, i.e. have at most one non-zero output component (the
 677:       basis either moves along a single output dim, or is a broadcast/zero
 678:       basis).
 679:     - After removing broadcast bases the layout must be bijective.
```
**EN:** This block declares or defines callable APIs such as vector and component, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 vector and component 等可调用 API，用来封装这里提供的核心行为。

### Lines 681-686
```tablegen
 681:     GenericLinearEncodingAttr relaxes the above in two ways:
 682:     - Warp bases MAY be swizzled (non-zero in multiple output dims).
 683:       Register, lane, and block bases must still be non-swizzled.
 684:     - The layout is only required to be surjective (not bijective) after
 685:       removing broadcast bases.
 686:   }];
```
**EN:** This block declares or defines callable APIs such as swizzled and surjective, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 swizzled and surjective 等可调用 API，用来封装这里提供的核心行为。

### Lines 688-691
```tablegen
 688:   let extraClassDeclaration = extraLinearDistributedDeclaration # [{
 689:     SmallVector<unsigned> getWarpsPerCTA() const;
 690:     SmallVector<unsigned> getWarpOrder() const;
 691:   }];
```
**EN:** This block injects extra C++ helpers into the generated class, such as getWarpsPerCTA and getWarpOrder.
**CN:** 该代码块向生成类中注入额外的 C++ 辅助方法，例如 getWarpsPerCTA and getWarpOrder。

### Lines 693-700
```tablegen
 693:   let genVerifyDecl = 1;
 694:   // Example of assembly format:
 695:   // <{register = [[0, 1], [8, 0], [0, 8], [64, 0]],
 696:   //   lane = [[0, 2], [0, 4], [1, 0], [2, 0], [4, 0]],
 697:   //   warp = [[16, 0], [32, 0]],
 698:   //   block = []}>
 699:   let hasCustomAssemblyFormat = 1;
 700: }
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 702-704
```tablegen
 702: //===----------------------------------------------------------------------===//
 703: // Generic Linear Layout Encoding
 704: //===----------------------------------------------------------------------===//
```
**EN:** This comment block records the intent and constraints of the surrounding code: ===----------------------------------------------------------------------===// Generic Linear Layout Encoding ===----------------------------------------------------------------....
**CN:** 该注释块解释了周边代码的设计背景、意图或约束条件，帮助读者理解后续实现。

### Lines 706-709
```tablegen
 706: def GenericLinearEncodingAttr
 707:     : LinearDistributedEncoding<"GenericLinearEncoding",
 708:                                 "generic_linear_encoding"> {
 709:   let mnemonic = "generic_linear";
```
**EN:** This TableGen def record defines `GenericLinearEncodingAttr`.
**CN:** 该 TableGen def 记录定义了 `GenericLinearEncodingAttr`。

### Lines 711-714
```tablegen
 711:   let description = [{
 712:     A linear layout encoding with relaxed constraints compared to LinearEncoding.
 713:     Like LinearEncoding, it wraps a LinearLayout with input dims
 714:     {register, lane, warp, block} and output dims {dim0, dim1, ...}.
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 716-719
```tablegen
 716:     Unlike LinearEncoding:
 717:     - Warp bases MAY be swizzled (non-zero in multiple output dims).
 718:     - The layout is NOT required to be bijective after removing broadcast bases,
 719:       but it's still required to be surjective.
```
**EN:** This block declares or defines callable APIs such as swizzled, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 swizzled 等可调用 API，用来封装这里提供的核心行为。

### Lines 721-724
```tablegen
 721:     Register, lane, and block bases must still be non-swizzled (at most one
 722:     non-zero output component per basis vector), which preserves existing
 723:     within-thread, within-warp, and CGA lowering paths.
 724:   }];
```
**EN:** This block declares or defines callable APIs such as swizzled, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 swizzled 等可调用 API，用来封装这里提供的核心行为。

### Lines 726-726
```tablegen
 726:   let extraClassDeclaration = extraLinearDistributedDeclaration;
```
**EN:** This block injects extra C++ helpers into the generated class.
**CN:** 该代码块向生成类中注入额外的 C++ 辅助方法。

### Lines 728-730
```tablegen
 728:   let genVerifyDecl = 1;
 729:   let hasCustomAssemblyFormat = 1;
 730: }
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 732-734
```tablegen
 732: //===----------------------------------------------------------------------===//
 733: // Blocked Layout Encoding
 734: //===----------------------------------------------------------------------===//
```
**EN:** This comment block records the intent and constraints of the surrounding code: ===----------------------------------------------------------------------===// Blocked Layout Encoding ===----------------------------------------------------------------------=....
**CN:** 该注释块解释了周边代码的设计背景、意图或约束条件，帮助读者理解后续实现。

### Lines 736-737
```tablegen
 736: def BlockedEncodingAttr : DistributedEncoding<"BlockedEncoding", "blocked_encoding"> {
 737:   let mnemonic = "blocked";
```
**EN:** This TableGen def record defines `BlockedEncodingAttr`. It is specialized from `DistributedEncoding<"BlockedEncoding", "blocked_encoding">`.
**CN:** 该 TableGen def 记录定义了 `BlockedEncodingAttr`。 它基于 `DistributedEncoding<"BlockedEncoding", "blocked_encoding">` 进一步特化。

### Lines 739-743
```tablegen
 739:   let description = [{
 740: An encoding where each warp owns a contiguous portion of the target tensor. This is typically the kind of data layout
 741: used to promote memory coalescing in LoadInst and StoreInst.
 742: It is characterized by three tuples -- thread tile size, warp tile size, and block tile size -- which
 743: specify the amount of elements owned by each CUDA thread, warp and CTA respectively.
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 745-745
```tablegen
 745: Example 1, a row-major coalesced layout may partition a 16x16 tensor over 2 warps (i.e. 64 threads) as follows:
```
**EN:** This block declares or defines callable APIs such as warps, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 warps 等可调用 API，用来封装这里提供的核心行为。

### Lines 747-753
```tablegen
 747: [ 0  0  1  1  2  2  3  3  ; 32 32 33 33 34 34 35 35 ]
 748: [ 0  0  1  1  2  2  3  3  ; 32 32 33 33 34 34 35 35 ]
 749: [ 4  4  5  5  6  6  7  7  ; 36 36 37 37 38 38 39 39 ]
 750: [ 4  4  5  5  6  6  7  7  ; 36 36 37 37 38 38 39 39 ]
 751: ...
 752: [ 28 28 29 29 30 30 31 31 ; 60 60 61 61 62 62 63 63 ]
 753: [ 28 28 29 29 30 30 31 31 ; 60 60 61 61 62 62 63 63 ]
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 755-755
```tablegen
 755: for
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 757-761
```tablegen
 757: #ttg.blocked_layout<{
 758:   sizePerThread = {2, 2}
 759:   threadsPerWarp = {8, 4}
 760:   blocked = {{0, 1}}
 761: }>
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 763-763
```tablegen
 763: Example 2, a row-major coalesced layout may partition a 32x32 tensor over 2 warps (i.e. 64 threads) as follows:
```
**EN:** This block declares or defines callable APIs such as warps, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 warps 等可调用 API，用来封装这里提供的核心行为。

### Lines 765-779
```tablegen
 765: [ 0  0  1  1  2  2  3  3  ; 32 32 33 33 34 34 35 35  0  0  1  1  2  2  3  3  ; 32 32 33 33 34 34 35 35 ]
 766: [ 0  0  1  1  2  2  3  3  ; 32 32 33 33 34 34 35 35  0  0  1  1  2  2  3  3  ; 32 32 33 33 34 34 35 35 ]
 767: [ 4  4  5  5  6  6  7  7  ; 36 36 37 37 38 38 39 39  4  4  5  5  6  6  7  7  ; 36 36 37 37 38 38 39 39 ]
 768: [ 4  4  5  5  6  6  7  7  ; 36 36 37 37 38 38 39 39  4  4  5  5  6  6  7  7  ; 36 36 37 37 38 38 39 39 ]
 769: ...                                                 ...
 770: [ 28 28 29 29 30 30 31 31 ; 60 60 61 61 62 62 63 63  28 28 29 29 30 30 31 31 ; 60 60 61 61 62 62 63 63 ]
 771: [ 28 28 29 29 30 30 31 31 ; 60 60 61 61 62 62 63 63  28 28 29 29 30 30 31 31 ; 60 60 61 61 62 62 63 63 ]
 772: [ 0  0  1  1  2  2  3  3  ; 32 32 33 33 34 34 35 35  0  0  1  1  2  2  3  3  ; 32 32 33 33 34 34 35 35 ]
 773: [ 0  0  1  1  2  2  3  3  ; 32 32 33 33 34 34 35 35  0  0  1  1  2  2  3  3  ; 32 32 33 33 34 34 35 35 ]
 774: [ 4  4  5  5  6  6  7  7  ; 36 36 37 37 38 38 39 39  4  4  5  5  6  6  7  7  ; 36 36 37 37 38 38 39 39 ]
 775: [ 4  4  5  5  6  6  7  7  ; 36 36 37 37 38 38 39 39  4  4  5  5  6  6  7  7  ; 36 36 37 37 38 38 39 39 ]
 776: ...                                                 ...
 777: [ 28 28 29 29 30 30 31 31 ; 60 60 61 61 62 62 63 63  28 28 29 29 30 30 31 31 ; 60 60 61 61 62 62 63 63 ]
 778: [ 28 28 29 29 30 30 31 31 ; 60 60 61 61 62 62 63 63  28 28 29 29 30 30 31 31 ; 60 60 61 61 62 62 63 63 ]
 779: for
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 781-785
```tablegen
 781: #ttg.blocked_layout<{
 782:   sizePerThread = {2, 2}
 783:   threadsPerWarp = {8, 4}
 784:   blocked = {{0, 1}}
 785: }>
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 787-788
```tablegen
 787: Example 3, A row-major coalesced layout may partition a 32x32 tensor over 2 warps (i.e. 64 threads) and
 788: 4 CTAs (taking 2x2 for example) as follows:
```
**EN:** This block declares or defines callable APIs such as warps and CTAs, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 warps and CTAs 等可调用 API，用来封装这里提供的核心行为。

### Lines 790-797
```tablegen
 790: CTA [0,0]                                              CTA [0,1]
 791: [ 0  0  1  1  2  2  3  3  ; 32 32 33 33 34 34 35 35 ]  [ 0  0  1  1  2  2  3  3  ; 32 32 33 33 34 34 35 35 ]
 792: [ 0  0  1  1  2  2  3  3  ; 32 32 33 33 34 34 35 35 ]  [ 0  0  1  1  2  2  3  3  ; 32 32 33 33 34 34 35 35 ]
 793: [ 4  4  5  5  6  6  7  7  ; 36 36 37 37 38 38 39 39 ]  [ 4  4  5  5  6  6  7  7  ; 36 36 37 37 38 38 39 39 ]
 794: [ 4  4  5  5  6  6  7  7  ; 36 36 37 37 38 38 39 39 ]  [ 4  4  5  5  6  6  7  7  ; 36 36 37 37 38 38 39 39 ]
 795: ...                                                    ...
 796: [ 28 28 29 29 30 30 31 31 ; 60 60 61 61 62 62 63 63 ]  [ 28 28 29 29 30 30 31 31 ; 60 60 61 61 62 62 63 63 ]
 797: [ 28 28 29 29 30 30 31 31 ; 60 60 61 61 62 62 63 63 ]  [ 28 28 29 29 30 30 31 31 ; 60 60 61 61 62 62 63 63 ]
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 799-807
```tablegen
 799: CTA [1,0]                                              CTA [1,1]
 800: [ 0  0  1  1  2  2  3  3  ; 32 32 33 33 34 34 35 35 ]  [ 0  0  1  1  2  2  3  3  ; 32 32 33 33 34 34 35 35 ]
 801: [ 0  0  1  1  2  2  3  3  ; 32 32 33 33 34 34 35 35 ]  [ 0  0  1  1  2  2  3  3  ; 32 32 33 33 34 34 35 35 ]
 802: [ 4  4  5  5  6  6  7  7  ; 36 36 37 37 38 38 39 39 ]  [ 4  4  5  5  6  6  7  7  ; 36 36 37 37 38 38 39 39 ]
 803: [ 4  4  5  5  6  6  7  7  ; 36 36 37 37 38 38 39 39 ]  [ 4  4  5  5  6  6  7  7  ; 36 36 37 37 38 38 39 39 ]
 804: ...                                                    ...
 805: [ 28 28 29 29 30 30 31 31 ; 60 60 61 61 62 62 63 63 ]  [ 28 28 29 29 30 30 31 31 ; 60 60 61 61 62 62 63 63 ]
 806: [ 28 28 29 29 30 30 31 31 ; 60 60 61 61 62 62 63 63 ]  [ 28 28 29 29 30 30 31 31 ; 60 60 61 61 62 62 63 63 ]
 807: for
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 809-814
```tablegen
 809: #ttg.blocked_layout<{
 810:   sizePerThread = {2, 2}
 811:   threadsPerWarp = {8, 4}
 812:   blocked = {{0, 1}, {1, 0}}
 813: }>
 814: }];
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 816-821
```tablegen
 816:   let parameters = (
 817:     ins
 818:     ArrayRefParameter<"unsigned">:$sizePerThread,
 819:     ArrayRefParameter<"unsigned">:$threadsPerWarp,
 820:     ArrayRefParameter<"unsigned">:$warpsPerCTA,
 821:     ArrayRefParameter<"unsigned">:$order, // the fastest-changing axis first
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 823-827
```tablegen
 823:     // CGALayout is optional in the textual IR.  If omitted, we infer it to be a
 824:     // CGA with a single CTA (i.e. the trivial map onto dim0..dimn-1)
 825:     "CGAEncodingAttr":$CGALayout
 826:   );
 827:   let genVerifyDecl = 1;
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 829-839
```tablegen
 829:   let builders = [
 830:     AttrBuilder<(ins "ArrayRef<int64_t>":$shape,
 831:                      "ArrayRef<unsigned>":$sizePerThread,
 832:                      "ArrayRef<unsigned>":$order,
 833:                      "unsigned":$numWarps,
 834:                      "unsigned":$numThreadsPerWarp,
 835:                      "CGAEncodingAttr":$CGALayout), [{
 836:       unsigned rank = sizePerThread.size();
 837:       SmallVector<unsigned, 4> threadsPerWarp(rank);
 838:       SmallVector<unsigned, 4> warpsPerCTA(rank);
 839:       SmallVector<int64_t> shapePerCTA = getShapePerCTA(CGALayout.getCTASplitNum(), shape);
```
**EN:** This block adds convenience builders so the generated API can construct the entity with common argument patterns.
**CN:** 该代码块添加便捷 builder，使生成 API 能以常见参数组合快速构造该实体。

### Lines 841-845
```tablegen
 841:       unsigned remainingLanes = numThreadsPerWarp;
 842:       unsigned remainingThreads = numWarps * numThreadsPerWarp;
 843:       unsigned remainingWarps = numWarps;
 844:       unsigned prevLanes = 1;
 845:       unsigned prevWarps = 1;
```
**EN:** This block stores supporting state such as numThreadsPerWarp and numWarps, which other APIs in the file consume.
**CN:** 该代码块声明了 numThreadsPerWarp and numWarps 等支撑状态，供本文件中的其他 API 使用。

### Lines 847-858
```tablegen
 847:       // starting from the contiguous dimension
 848:       for (unsigned d = 0; d < rank - 1; ++d) {
 849:         unsigned i = order[d];
 850:         unsigned threadsPerCTA = std::clamp<unsigned>(remainingThreads, 1, std::max<unsigned>(1, shapePerCTA[i] / sizePerThread[i]));
 851:         threadsPerWarp[i] = std::clamp<unsigned>(threadsPerCTA, 1, remainingLanes);
 852:         warpsPerCTA[i] = std::clamp<unsigned>(threadsPerCTA / threadsPerWarp[i], 1, remainingWarps);
 853:         remainingWarps /= warpsPerCTA[i];
 854:         remainingLanes /= threadsPerWarp[i];
 855:         remainingThreads /= threadsPerCTA;
 856:         prevLanes *= threadsPerWarp[i];
 857:         prevWarps *= warpsPerCTA[i];
 858:       }
```
**EN:** This block stores supporting state such as order, warpsPerCTA, threadsPerWarp, and threadsPerCTA, which other APIs in the file consume.
**CN:** 该代码块声明了 order, warpsPerCTA, threadsPerWarp, and threadsPerCTA 等支撑状态，供本文件中的其他 API 使用。

### Lines 860-862
```tablegen
 860:       // Expand the last dimension to fill the remaining lanes and warps
 861:       threadsPerWarp[order[rank - 1]] = numThreadsPerWarp / prevLanes;
 862:       warpsPerCTA[order[rank - 1]] = numWarps / prevWarps;
```
**EN:** This block stores supporting state such as prevLanes and prevWarps, which other APIs in the file consume.
**CN:** 该代码块声明了 prevLanes and prevWarps 等支撑状态，供本文件中的其他 API 使用。

### Lines 864-865
```tablegen
 864:       return $_get(context, sizePerThread, threadsPerWarp, warpsPerCTA, order, CGALayout);
 865:     }]>,
```
**EN:** This block declares or defines callable APIs such as _get, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 _get 等可调用 API，用来封装这里提供的核心行为。

### Lines 867-876
```tablegen
 867:     AttrBuilder<(ins "ArrayRef<int64_t>":$shape,
 868:                      "ArrayRef<unsigned>":$sizePerThread,
 869:                      "ArrayRef<unsigned>":$order,
 870:                      "unsigned":$numWarps,
 871:                      "unsigned":$numThreadsPerWarp,
 872:                      "unsigned":$numCTAs), [{
 873:       unsigned rank = sizePerThread.size();
 874:       SmallVector<unsigned, 4> CTAsPerCGA(rank);
 875:       SmallVector<unsigned, 4> CTASplitNum(rank);
 876:       ArrayRef<unsigned> CTAOrder = order;
```
**EN:** This block declares or defines callable APIs such as size, CTAsPerCGA, and CTASplitNum, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 size, CTAsPerCGA, and CTASplitNum 等可调用 API，用来封装这里提供的核心行为。

### Lines 878-878
```tablegen
 878:       unsigned remainingCTAs = numCTAs;
```
**EN:** This block stores supporting state such as numCTAs, which other APIs in the file consume.
**CN:** 该代码块声明了 numCTAs 等支撑状态，供本文件中的其他 API 使用。

### Lines 880-886
```tablegen
 880:       // starting from the most strided dimension
 881:       for (int d = rank - 1; d >= 0; --d) {
 882:         unsigned i = order[d];
 883:         CTAsPerCGA[i] = std::clamp<unsigned>(remainingCTAs, 1, std::max<unsigned>(1, shape[i] / sizePerThread[i]));
 884:         CTASplitNum[i] = CTAsPerCGA[i];
 885:         remainingCTAs /= CTAsPerCGA[i];
 886:       }
```
**EN:** This block stores supporting state such as order and CTAsPerCGA, which other APIs in the file consume.
**CN:** 该代码块声明了 order and CTAsPerCGA 等支撑状态，供本文件中的其他 API 使用。

### Lines 888-888
```tablegen
 888:       CTAsPerCGA[rank - 1] *= remainingCTAs; // wrap at CTA level
```
**EN:** This block stores supporting state such as remainingCTAs, which other APIs in the file consume.
**CN:** 该代码块声明了 remainingCTAs 等支撑状态，供本文件中的其他 API 使用。

### Lines 890-893
```tablegen
 890:       CGAEncodingAttr CGALayout = CGAEncodingAttr::fromSplitParams(context, CTAsPerCGA, CTASplitNum, CTAOrder);
 891:       return get(context, shape, sizePerThread, order, numWarps, numThreadsPerWarp, CGALayout);
 892:     }]>
 893:   ];
```
**EN:** This block declares or defines callable APIs such as fromSplitParams and get, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 fromSplitParams and get 等可调用 API，用来封装这里提供的核心行为。

### Lines 895-895
```tablegen
 895:   let extraClassDeclaration = extraDistributedDeclaration;
```
**EN:** This block injects extra C++ helpers into the generated class.
**CN:** 该代码块向生成类中注入额外的 C++ 辅助方法。

### Lines 897-898
```tablegen
 897:   let hasCustomAssemblyFormat = 1;
 898: }
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 900-901
```tablegen
 900: def AMDMfmaEncodingAttr : DistributedEncoding<"AMDMfmaEncoding", "amd_mfma_encoding", [MmaEncodingTrait]> {
 901:   let mnemonic = "amd_mfma";
```
**EN:** This TableGen def record defines `AMDMfmaEncodingAttr`. It is specialized from `DistributedEncoding<"AMDMfmaEncoding", "amd_mfma_encoding", [MmaEncodingTrait]>`.
**CN:** 该 TableGen def 记录定义了 `AMDMfmaEncodingAttr`。 它基于 `DistributedEncoding<"AMDMfmaEncoding", "amd_mfma_encoding", [MmaEncodingTrait]>` 进一步特化。

### Lines 903-905
```tablegen
 903:   let description = [{
 904: An encoding for tensors that have been produced by MFMA matrix core instructions,
 905: available on AMD Instinct GPUs of CDNA architectures.
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 907-918
```tablegen
 907: It is characterized by the following parameters:
 908: - `version`: The GPU architecture:
 909:   - 1: gfx908: CDNA1
 910:   - 2: gfx90a: CDNA2
 911:   - 3: gfx942: CDNA3
 912:   - 4: gfx950: CDNA4
 913: - `warpsPerCTA`: The warp layout in the block.
 914: - `instrShape`: The shape in the form of (M, N, K) of the matrix.
 915: - `isTransposed`: Indicates the result tensor is transposed so that it can be converted to dotOperand layout
 916: without going to shared memory. This is used in the case of chained dot (E.g. Flash-Attention kernel).
 917: - `tilesPerWarp`: The tile layout within a warp. Defaults to unit tile layout, i.e., single tile on all dimensions.
 918: - `elementBitWidth`: Bit width of the output element type. Supported values are 32 and 64. Defaults to 32.
```
**EN:** This block declares or defines callable APIs such as of and dot, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 of and dot 等可调用 API，用来封装这里提供的核心行为。

### Lines 920-922
```tablegen
 920: Example 1:
 921: Suppose we have a tensor with a shape of [32, 64], warpsPerCTA set to [1, 2] and MDim=NDim=32.
 922: The data will be distributed between threads as follows:
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 924-957
```tablegen
 924:                 warp 0                                 warp 1
 925: -----------------/\--------------      -----------------/\--------------
 926: [ 0   1   2   3  ...... 30  31 ]      [ 64  65  66  67 ...... 94   95  ]
 927: [ 0   1   2   3  ...... 30  31 ]      [ 64  65  66  67 ...... 94   95  ]
 928: [ 0   1   2   3  ...... 30  31 ]      [ 64  65  66  67 ...... 94   95  ]
 929: [ 0   1   2   3  ...... 30  31 ]      [ 64  65  66  67 ...... 94   95  ]
 930: [ 32  33  34  35 ...... 62  63 ]      [ 96  97  98  99 ...... 126  127 ]
 931: [ 32  33  34  35 ...... 62  63 ]      [ 96  97  98  99 ...... 126  127 ]
 932: [ 32  33  34  35 ...... 62  63 ]      [ 96  97  98  99 ...... 126  127 ]
 933: [ 32  33  34  35 ...... 62  63 ]      [ 96  97  98  99 ...... 126  127 ]
 934: [ 0   1   2   3  ...... 30  31 ]      [ 64  65  66  67 ...... 94   95  ]
 935: [ 0   1   2   3  ...... 30  31 ]      [ 64  65  66  67 ...... 94   95  ]
 936: [ 0   1   2   3  ...... 30  31 ]      [ 64  65  66  67 ...... 94   95  ]
 937: [ 0   1   2   3  ...... 30  31 ]      [ 64  65  66  67 ...... 94   95  ]
 938: [ 32  33  34  35 ...... 62  63 ]      [ 96  97  98  99 ...... 126  127 ]
 939: [ 32  33  34  35 ...... 62  63 ]      [ 96  97  98  99 ...... 126  127 ]
 940: [ 32  33  34  35 ...... 62  63 ]      [ 96  97  98  99 ...... 126  127 ]
 941: [ 32  33  34  35 ...... 62  63 ]      [ 96  97  98  99 ...... 126  127 ]
 942: [ 0   1   2   3  ...... 30  31 ]      [ 64  65  66  67 ...... 94   95  ]
 943: [ 0   1   2   3  ...... 30  31 ]      [ 64  65  66  67 ...... 94   95  ]
 944: [ 0   1   2   3  ...... 30  31 ]      [ 64  65  66  67 ...... 94   95  ]
 945: [ 0   1   2   3  ...... 30  31 ]      [ 64  65  66  67 ...... 94   95  ]
 946: [ 32  33  34  35 ...... 62  63 ]      [ 96  97  98  99 ...... 126  127 ]
 947: [ 32  33  34  35 ...... 62  63 ]      [ 96  97  98  99 ...... 126  127 ]
 948: [ 32  33  34  35 ...... 62  63 ]      [ 96  97  98  99 ...... 126  127 ]
 949: [ 32  33  34  35 ...... 62  63 ]      [ 96  97  98  99 ...... 126  127 ]
 950: [ 0   1   2   3  ...... 30  31 ]      [ 64  65  66  67 ...... 94   95  ]
 951: [ 0   1   2   3  ...... 30  31 ]      [ 64  65  66  67 ...... 94   95  ]
 952: [ 0   1   2   3  ...... 30  31 ]      [ 64  65  66  67 ...... 94   95  ]
 953: [ 0   1   2   3  ...... 30  31 ]      [ 64  65  66  67 ...... 94   95  ]
 954: [ 32  33  34  35 ...... 62  63 ]      [ 96  97  98  99 ...... 126  127 ]
 955: [ 32  33  34  35 ...... 62  63 ]      [ 96  97  98  99 ...... 126  127 ]
 956: [ 32  33  34  35 ...... 62  63 ]      [ 96  97  98  99 ...... 126  127 ]
 957: [ 32  33  34  35 ...... 62  63 ]      [ 96  97  98  99 ...... 126  127 ]
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 959-961
```tablegen
 959: Example 2:
 960: Suppose we have a tensor with a shape of [16, 32], warpsPerCTA set to [1, 2] and MDim=NDim=16.
 961: The data will be distributed between threads as follows:
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 963-980
```tablegen
 963:                 warp 0                                 warp 1
 964: -----------------/\-------------      ------------------/\---------------
 965: [ 0   1   2   3  ...... 14  15 ]      [ 64  65  66  67  ...... 78   79  ]
 966: [ 0   1   2   3  ...... 14  15 ]      [ 64  65  66  67  ...... 78   79  ]
 967: [ 0   1   2   3  ...... 14  15 ]      [ 64  65  66  67  ...... 78   79  ]
 968: [ 0   1   2   3  ...... 14  15 ]      [ 64  65  66  67  ...... 78   79  ]
 969: [ 16  17  18  19 ...... 30  31 ]      [ 80  81  82  83  ...... 94   95  ]
 970: [ 16  17  18  19 ...... 30  31 ]      [ 80  81  82  83  ...... 94   95  ]
 971: [ 16  17  18  19 ...... 30  31 ]      [ 80  81  82  83  ...... 94   95  ]
 972: [ 16  17  18  19 ...... 30  31 ]      [ 80  81  82  83  ...... 94   95  ]
 973: [ 32  33  34  35 ...... 46  47 ]      [ 96  97  98  99  ...... 110  111 ]
 974: [ 32  33  34  35 ...... 46  47 ]      [ 96  97  98  99  ...... 110  111 ]
 975: [ 32  33  34  35 ...... 46  47 ]      [ 96  97  98  99  ...... 110  111 ]
 976: [ 32  33  34  35 ...... 46  47 ]      [ 96  97  98  99  ...... 110  111 ]
 977: [ 48  49  50  51 ...... 62  63 ]      [ 112 113 114 115 ...... 126  127 ]
 978: [ 48  49  50  51 ...... 62  63 ]      [ 112 113 114 115 ...... 126  127 ]
 979: [ 48  49  50  51 ...... 62  63 ]      [ 112 113 114 115 ...... 126  127 ]
 980: [ 48  49  50  51 ...... 62  63 ]      [ 112 113 114 115 ...... 126  127 ]
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 982-986
```tablegen
 982: Example 3:
 983: Suppose we have a tensor with a shape of [8, 8], warpsPerCTA set to [2, 2] and nonKDim set to 4.
 984: The data will be distributed between threads as follows(note that each element is duplicated in 16 threads):
 985: Suppose we have a tensor with a shape of [8, 8], warpsPerCTA set to [2, 2] and MDim=NDim=4.
 986: The data will be distributed between threads as follows(note that each element is duplicated in 16 threads):
```
**EN:** This block declares or defines callable APIs such as follows, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 follows 等可调用 API，用来封装这里提供的核心行为。

### Lines 988-999
```tablegen
 988: M  N ->                    warp 0                                                       warp 2
 989: | --------------------------/\--------------------------   ------------------------------/\------------------------------
 990: V [ 0,4,8...60   1,5...61     2,6...62     3,7...63    ]   [ 128,132...188  129,133...189  130,134...190  131,135...191 ]
 991:   [ 0,4,8...60   1,5...61     2,6...62     3,7...63    ]   [ 128,132...188  129,133...189  130,134...190  131,135...191 ]
 992:   [ 0,4,8...60   1,5...61     2,6...62     3,7...63    ]   [ 128,132...188  129,133...189  130,134...190  131,135...191 ]
 993:   [ 0,4,8...60   1,5...61     2,6...62     3,7...63    ]   [ 128,132...188  129,133...189  130,134...190  131,135...191 ]
 994:                            warp 1                                                       warp 3
 995:   --------------------------/\--------------------------   ------------------------------/\------------------------------
 996:   [ 64,68...124  65,69...125  66,70...126  67,71...127 ]   [ 192,196...252  193,197...253  194,198...254  195,199...255 ]
 997:   [ 64,68...124  65,69...125  66,70...126  67,71...127 ]   [ 192,196...252  193,197...253  194,198...254  195,199...255 ]
 998:   [ 64,68...124  65,69...125  66,70...126  67,71...127 ]   [ 192,196...252  193,197...253  194,198...254  195,199...255 ]
 999:   [ 64,68...124  65,69...125  66,70...126  67,71...127 ]   [ 192,196...252  193,197...253  194,198...254  195,199...255 ]
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1001-1005
```tablegen
1001: Example 4:
1002: This example demonstrates semantics of tilesPerWarp parameter. The MFMA layout (with tilesPerWarp=[1,1])
1003: assumes that each warp within a CTA tile computes a single MFMA tile. When the tensor is larger than
1004: a single CTA tile, these tiles are repeated across the tensor. In this setup, the output tiles computed
1005: by each warp were strided by the number of warps per CTA tile in both row and column dimensions.
```
**EN:** This block declares or defines callable APIs such as layout, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 layout 等可调用 API，用来封装这里提供的核心行为。

### Lines 1007-1008
```tablegen
1007: For instance, with 16 MFMA tiles and warpsPerCTA = [2, 2], the distribution of warps across the MFMA
1008: tiles looked like:
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1010-1013
```tablegen
1010: w0 w1 w0 w1
1011: w2 w3 w2 w3
1012: w0 w1 w0 w1
1013: w2 w3 w2 w3
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1015-1016
```tablegen
1015: tilesPerWarp parameter allows each warp to compute contiguous MFMA tiles in the row and/or column dimensions.
1016: Using the same example with tilesPerWarp = [2, 2], the layout becomes:
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1018-1022
```tablegen
1018: w0 w0 w1 w1
1019: w0 w0 w1 w1
1020: w2 w2 w3 w3
1021: w2 w2 w3 w3
1022: }];
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1024-1033
```tablegen
1024:   let parameters = (
1025:     ins
1026:     "unsigned": $version,
1027:     ArrayRefParameter<"unsigned">:$warpsPerCTA,
1028:     ArrayRefParameter<"unsigned">:$instrShape,
1029:     "bool":$isTransposed,
1030:     "CGAEncodingAttr":$CGALayout,
1031:     ArrayRefParameter<"unsigned">:$tilesPerWarp,
1032:     "unsigned":$elementBitWidth
1033:   );
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1035-1050
```tablegen
1035:   let builders = [
1036:     AttrBuilder<(ins "unsigned":$version,
1037:                      "ArrayRef<unsigned>":$warpsPerCTA,
1038:                      "ArrayRef<unsigned>":$instrShape,
1039:                      "bool":$isTransposed,
1040:                      "CGAEncodingAttr":$CGALayout,
1041:                      CArg<"ArrayRef<unsigned>", "{}">:$tpw,
1042:                      CArg<"unsigned", "0">:$elementBitWidth), [{
1043:       SmallVector<unsigned> tilesPerWarp(tpw);
1044:       if (tilesPerWarp.empty())
1045:         tilesPerWarp = SmallVector<unsigned>(warpsPerCTA.size(), 1);
1046:       if (elementBitWidth == 0)
1047:         elementBitWidth = 32;
1048:       return $_get($_ctxt, version, warpsPerCTA, instrShape, isTransposed, CGALayout, tilesPerWarp, elementBitWidth);
1049:     }]>
1050:   ];
```
**EN:** This block adds convenience builders so the generated API can construct the entity with common argument patterns.
**CN:** 该代码块添加便捷 builder，使生成 API 能以常见参数组合快速构造该实体。

### Lines 1052-1055
```tablegen
1052:   let extraClassDeclaration = extraDistributedDeclaration # [{
1053:     SmallVector<int64_t> getInstrShapeForOperand(int kWidth, int opIdx) const;
1054:     SmallVector<int64_t> getRepForOperand(ArrayRef<int64_t> operandShape, int kWidth, int opIdx) const;
1055:     SmallVector<unsigned> getRepOrderForOperand(int opIdx) const;
```
**EN:** This block injects extra C++ helpers into the generated class, such as getInstrShapeForOperand, getRepForOperand, and getRepOrderForOperand.
**CN:** 该代码块向生成类中注入额外的 C++ 辅助方法，例如 getInstrShapeForOperand, getRepForOperand, and getRepOrderForOperand。

### Lines 1057-1058
```tablegen
1057:     // Check if tilesPerWarp is 1 in every dimension.
1058:     bool hasUnitTilesPerWarp() const;
```
**EN:** This block declares or defines callable APIs such as hasUnitTilesPerWarp, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 hasUnitTilesPerWarp 等可调用 API，用来封装这里提供的核心行为。

### Lines 1060-1066
```tablegen
1060:     // Returns a swizzled shared layout matching this MFMA layout for the
1061:     // dot operand at the given |operandIdx| with |operandShape|.
1062:     SwizzledSharedEncodingAttr composeSharedLayoutForOperand(
1063:         CGAEncodingAttr cgaLayout, int operandIdx, ArrayRef<int64_t> operandShape,
1064:         ArrayRef<unsigned> sharedOrder, unsigned vectorSize,
1065:         unsigned elemBitWidth, bool needTrans) const;
1066:   }];
```
**EN:** This block declares or defines callable APIs such as composeSharedLayoutForOperand, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 composeSharedLayoutForOperand 等可调用 API，用来封装这里提供的核心行为。

### Lines 1068-1071
```tablegen
1068:   let genVerifyDecl = 1;
1069:   let hasCustomAssemblyFormat = 1;
1070:   let skipDefaultBuilders = 1;
1071: }
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1073-1074
```tablegen
1073: def AMDWmmaEncodingAttr : DistributedEncoding<"AMDWmmaEncoding", "amd_wmma_encoding", [MmaEncodingTrait]> {
1074:   let mnemonic = "amd_wmma";
```
**EN:** This TableGen def record defines `AMDWmmaEncodingAttr`. It is specialized from `DistributedEncoding<"AMDWmmaEncoding", "amd_wmma_encoding", [MmaEncodingTrait]>`.
**CN:** 该 TableGen def 记录定义了 `AMDWmmaEncodingAttr`。 它基于 `DistributedEncoding<"AMDWmmaEncoding", "amd_wmma_encoding", [MmaEncodingTrait]>` 进一步特化。

### Lines 1076-1078
```tablegen
1076:   let description = [{
1077: An encoding for tensors that have been produced by WMMA matrix core instructions,
1078: available on AMD Radeon GPUs of RDNA architectures.
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 1080-1090
```tablegen
1080: It is characterized by the following parameters:
1081: - `version` indicates the GPU architecture:
1082:   - 1: RDNA3; e.g., gfx1100, gfx1101
1083:   - 2: RDNA4; e.g., gfx1200, gfx1201
1084:   - 3: gfx1250
1085: - `ctaLayout` indicates the warp layout in the block. This is a generalization
1086:    compared to previous warp layout representation using warpsPerCTA and tilesPerWarp
1087:    parameters.
1088: - `instrShape` indicates the shape in the form of (M, N, K) of the matrix
1089:    operation performed by a single WMMA instruction. Defaults to (16, 16, 16).
1090: - `isTransposed` indicates the layout of the result tensor is transposed.
```
**EN:** This block declares or defines callable APIs such as of and to, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 of and to 等可调用 API，用来封装这里提供的核心行为。

### Lines 1092-1095
```tablegen
1092: Example 1:
1093: Suppose we have a tensor with shape [32, 64], `warpsPerCTA` set to [2, 2].
1094: Matrix elements represent which lane owns the element. Currently only wave32 mode
1095: is supported.
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1097-1097
```tablegen
1097: // ----------------------------------- version = 1 ----------------------------------- //
```
**EN:** This comment block records the intent and constraints of the surrounding code: ----------------------------------- version = 1 ----------------------------------- //.
**CN:** 该注释块解释了周边代码的设计背景、意图或约束条件，帮助读者理解后续实现。

### Lines 1099-1107
```tablegen
1099: Row |                  warp 0                                    warp 1
1100:     |/-------------------^-------------------\ /-------------------^-------------------\
1101: 0   |[0  1  2  ... 14 15] [0  1  2  ... 14 15] [0  1  2  ... 14 15] [0  1  2  ... 14 15]
1102: 1   |[16 17 18 ... 30 31] [16 17 18 ... 30 31] [16 17 18 ... 30 31] [16 17 18 ... 30 31]
1103: 2   |[0  1  2  ... 14 15] [0  1  2  ... 14 15] [0  1  2  ... 14 15] [0  1  2  ... 14 15]
1104: 3   |[16 17 18 ... 30 31] [16 17 18 ... 30 31] [16 17 18 ... 30 31] [16 17 18 ... 30 31]
1105:     | ...                  ...                  ...                  ...
1106: 14  |[0  1  2  ... 14 15] [0  1  2  ... 14 15] [0  1  2  ... 14 15] [0  1  2  ... 14 15]
1107: 15  |[16 17 18 ... 30 31] [16 17 18 ... 30 31] [16 17 18 ... 30 31] [16 17 18 ... 30 31]
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1109-1117
```tablegen
1109:     |                  warp 2                                    warp 3
1110: 16  |/-------------------^-------------------\ /-------------------^-------------------\
1111: 17  |[0  1  2  ... 14 15] [0  1  2  ... 14 15] [0  1  2  ... 14 15] [0  1  2  ... 14 15]
1112: 18  |[16 17 18 ... 30 31] [16 17 18 ... 30 31] [16 17 18 ... 30 31] [16 17 18 ... 30 31]
1113: 19  |[0  1  2  ... 14 15] [0  1  2  ... 14 15] [0  1  2  ... 14 15] [0  1  2  ... 14 15]
1114: 20  |[16 17 18 ... 30 31] [16 17 18 ... 30 31] [16 17 18 ... 30 31] [16 17 18 ... 30 31]
1115:     | ...                  ...                  ...                  ...
1116: 30  |[0  1  2  ... 14 15] [0  1  2  ... 14 15] [0  1  2  ... 14 15] [0  1  2  ... 14 15]
1117: 31  |[16 17 18 ... 30 31] [16 17 18 ... 30 31] [16 17 18 ... 30 31] [16 17 18 ... 30 31]
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1119-1119
```tablegen
1119: // ------------------------ version = 2/3, isTransposed = false ------------------------ //
```
**EN:** This comment block records the intent and constraints of the surrounding code: ------------------------ version = 2/3, isTransposed = false ------------------------ //.
**CN:** 该注释块解释了周边代码的设计背景、意图或约束条件，帮助读者理解后续实现。

### Lines 1121-1145
```tablegen
1121: Row |       warp 0                warp 1
1122:     |/--------^---------\ /---------^--------\
1123: 0   |[0  1  2  ... 14 15] [0  1  2  ... 14 15]
1124: 1   |[0  1  2  ... 14 15] [0  1  2  ... 14 15]
1125: ..  | ...                    ...
1126: 6   |[0  1  2  ... 14 15] [0  1  2  ... 14 15]
1127: 7   |[0  1  2  ... 14 15] [0  1  2  ... 14 15]
1128: 8   |[16 17 18 ... 30 31] [16 17 18 ... 30 31]
1129: 9   |[16 17 18 ... 30 31] [16 17 18 ... 30 31]
1130: ..  | ...                  ...
1131: 14  |[16 17 18 ... 30 31] [16 17 18 ... 30 31]
1132: 15  |[16 17 18 ... 30 31] [16 17 18 ... 30 31]
1133:     |
1134:     |       warp 2                warp 3
1135:     |/--------^---------\ /---------^--------\
1136: 16  |[0  1  2  ... 14 15] [0  1  2  ... 14 15]
1137: 17  |[0  1  2  ... 14 15] [0  1  2  ... 14 15]
1138: ..  | ...                    ...
1139: 22  |[0  1  2  ... 14 15] [0  1  2  ... 14 15]
1140: 23  |[0  1  2  ... 14 15] [0  1  2  ... 14 15]
1141: 24  |[16 17 18 ... 30 31] [16 17 18 ... 30 31]
1142: 25  |[16 17 18 ... 30 31] [16 17 18 ... 30 31]
1143: ..  | ...                  ...
1144: 30  |[16 17 18 ... 30 31] [16 17 18 ... 30 31]
1145: 31  |[16 17 18 ... 30 31] [16 17 18 ... 30 31]
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1147-1147
```tablegen
1147: // ------------------------ version = 2/3, isTransposed = true ------------------------ //
```
**EN:** This comment block records the intent and constraints of the surrounding code: ------------------------ version = 2/3, isTransposed = true ------------------------ //.
**CN:** 该注释块解释了周边代码的设计背景、意图或约束条件，帮助读者理解后续实现。

### Lines 1149-1165
```tablegen
1149:     |               warp 0                     warp 1
1150:     |/----------------^----------------\ /-------^-------\
1151: Col>| 0  1  2  3  4  5  6  7  8  ... 15  16 17 18  ... 32
1152: Row |
1153: 0   |[0  0  0  0  0  0  0  0  16 ... 16] [0  0  0  ... 16]
1154: 1   |[1  1  1  1  1  1  1  1  17 ... 17] [1  1  1  ... 17]
1155: ..  | ...                  ...
1156: 14  |[14 14 14 14 14 14 14 14 30 ... 30] [14 14 14 ... 30]
1157: 15  |[15 15 15 15 15 15 15 15 31 ... 31] [15 15 15 ... 31]
1158:     |
1159:     |               warp 2                     warp 3
1160:     |/----------------^----------------\ /-------^-------\
1161: 16  |[0  0  0  0  0  0  0  0  16 ... 16] [0  0  0  ... 16]
1162: 17  |[1  1  1  1  1  1  1  1  17 ... 17] [1  1  1  ... 17]
1163: ..  | ...                  ...
1164: 30  |[14 14 14 14 14 14 14 14 30 ... 30] [14 14 14 ... 30]
1165: 31  |[15 15 15 15 15 15 15 15 31 ... 31] [15 15 15 ... 31]
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1167-1171
```tablegen
1167: Example 2:
1168: This example illustrates the purpose of the ctaLayout parameter.
1169: ctaLayout is a linear layout describing how warps are arranged across WMMA tiles.
1170: Previously, this information was encoded using warpsPerCTA and tilesPerWarp parametes.
1171: For instance, a configuration with 4 warps, represented as:
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1173-1173
```tablegen
1173: warpsPerCTA = [2, 2], tilesPerWarp = [1, 1]
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1175-1175
```tablegen
1175: would translate to:
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1177-1177
```tablegen
1177: ctaLayout = {reg = [], warp = [[0, 1], [1, 0]]}
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1179-1180
```tablegen
1179: By default, WMMA assumes that each warp in a CTA computes exactly one WMMA tile.
1180: In the grid below, each w* label indicates which warp computes that tile:
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1182-1185
```tablegen
1182: w0 w1 w0 w1
1183: w2 w3 w2 w3
1184: w0 w1 w0 w1
1185: w2 w3 w2 w3
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1187-1188
```tablegen
1187: To express more complex layouts, we must also account for repetitions within the mapping.
1188: For example, the configuration formerly described as:
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1190-1190
```tablegen
1190: warpsPerCTA = [2, 2], tilesPerWarp  = [2, 2]
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1192-1192
```tablegen
1192: would translate to:
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1194-1194
```tablegen
1194: ctaLayout = {reg = [[0, 1], [1, 0]], warps = [[0, 2], [2, 0]] }
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1196-1199
```tablegen
1196: w0 w0 w1 w1
1197: w0 w0 w1 w1
1198: w2 w2 w3 w3
1199: w2 w2 w3 w3
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1201-1203
```tablegen
1201: This parameter provides a more general way to define warp mappings than what
1202: warpsPerCTA and tilesPerWarp alone could express.
1203: For instance:
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1205-1205
```tablegen
1205: ctaLayout = {reg = [[1, 0], [0, 1]], warps = [[0, 2], [2, 0]]}
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1207-1207
```tablegen
1207: still represents a layout similar to:
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1209-1209
```tablegen
1209: warpsPerCTA  = [2, 2], tilesPerWarp = [2, 2]
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1211-1211
```tablegen
1211: but with a different ordering of repetitions.
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1213-1215
```tablegen
1213: The motivation for this broader formulation comes from the need to describe swizzled warp
1214: layouts, which help avoid LDS partition conflicts on architectures such as gfx1250.
1215: A valid example of such swizzled configuration is:
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1217-1217
```tablegen
1217: ctaLayout = {reg = [[2, 0]], warps = [[2, 1], [1, 0]]}
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1219-1219
```tablegen
1219: With corresponding mapping:
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1221-1224
```tablegen
1221: w0 w1 <- second tile computed by w1
1222: w2 w3
1223: w0 w1 <- first tile computed by w1
1224: w2 w3
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1226-1227
```tablegen
1226: Note that ctaLayout naturally composes with layout definied on a single WMMA tile
1227: to form final WMMA layout.
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1229-1229
```tablegen
1229: wmmaLayout = tileLayout * ctaLayout
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1231-1232
```tablegen
1231: This simplifies both WMMA and dotOperand layouts lowering to linear layout.
1232:   }];
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1234-1241
```tablegen
1234:   let parameters = (
1235:     ins
1236:     "unsigned": $version,
1237:     LinearLayoutParam:$ctaLayout,
1238:     "bool":$isTransposed,
1239:     "CGAEncodingAttr":$CGALayout,
1240:     ArrayRefParameter<"unsigned">:$instrShape
1241:   );
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1243-1244
```tablegen
1243:   let genVerifyDecl = 1;
1244:   let hasCustomAssemblyFormat = 1;
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1246-1251
```tablegen
1246:   let extraClassDeclaration = extraDistributedDeclaration # [{
1247:     SmallVector<unsigned> getRepOrderForOperand(int opIdx) const;
1248:     LinearLayout getTileLayout(unsigned rank) const;
1249:     static SmallVector<unsigned, 3> getDefaultInstrShape() {
1250:       return {16, 16, 16};
1251:     }
```
**EN:** This block injects extra C++ helpers into the generated class, such as getRepOrderForOperand, getTileLayout, and getDefaultInstrShape.
**CN:** 该代码块向生成类中注入额外的 C++ 辅助方法，例如 getRepOrderForOperand, getTileLayout, and getDefaultInstrShape。

### Lines 1253-1256
```tablegen
1253:     // Returns nonK dim of operand opIdx (swapped for asymmetric transposed WMMA).
1254:     unsigned getOperandNonKDim(unsigned opIdx) const;
1255:     static unsigned getOperandNonKDim(unsigned mDim, unsigned nDim,
1256:                                       bool isTransposed, unsigned opIdx);
```
**EN:** This block declares or defines callable APIs such as getOperandNonKDim, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getOperandNonKDim 等可调用 API，用来封装这里提供的核心行为。

### Lines 1258-1263
```tablegen
1258:     // Returns a swizzled shared layout matching this WMMA layout for the
1259:     // dot operand at the given |operandIdx| with |operandShape|.
1260:     SwizzledSharedEncodingAttr composeSharedLayoutForOperand(
1261:         CGAEncodingAttr cgaLayout, int operandIdx, ArrayRef<int64_t> operandShape,
1262:         ArrayRef<unsigned> sharedOrder, unsigned kWidth,
1263:         unsigned elemBitWidth, bool needTrans) const;
```
**EN:** This block declares or defines callable APIs such as composeSharedLayoutForOperand, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 composeSharedLayoutForOperand 等可调用 API，用来封装这里提供的核心行为。

### Lines 1265-1268
```tablegen
1265:     // Returns true if the two WMMA layouts are equal, ignoring the CGA layout.
1266:     bool isEqualIgnoringCGALayout(AMDWmmaEncodingAttr other) const;
1267:   }];
1268: }
```
**EN:** This block declares or defines callable APIs such as isEqualIgnoringCGALayout, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isEqualIgnoringCGALayout 等可调用 API，用来封装这里提供的核心行为。

### Lines 1270-1271
```tablegen
1270: def NvidiaMmaEncodingAttr : DistributedEncoding<"NvidiaMmaEncoding", "nvidia_mma_encoding", [MmaEncodingTrait]> {
1271:   let mnemonic = "nvidia_mma";
```
**EN:** This TableGen def record defines `NvidiaMmaEncodingAttr`. It is specialized from `DistributedEncoding<"NvidiaMmaEncoding", "nvidia_mma_encoding", [MmaEncodingTrait]>`.
**CN:** 该 TableGen def 记录定义了 `NvidiaMmaEncodingAttr`。 它基于 `DistributedEncoding<"NvidiaMmaEncoding", "nvidia_mma_encoding", [MmaEncodingTrait]>` 进一步特化。

### Lines 1273-1274
```tablegen
1273:   let description = [{
1274: An encoding for tensors that have been produced by tensor cores.
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 1276-1284
```tablegen
1276: It is characterized by two parameters:
1277: - A 'versionMajor' which specifies the generation the tensor cores
1278:   whose output is being partitioned:
1279:   - 1 for first-gen tensor cores (Volta), and
1280:   - 2 for second-gen tensor cores (Turing/Ampere).
1281: - A 'versionMinor' which indicates the specific layout of a tensor core
1282:   generation, e.g. for Volta, there might be multiple kinds of layouts
1283:   annotated by 0,1,2 and so on.
1284: - A `blockTileSize` to indicate how data should be partitioned between warps.
```
**EN:** This block declares or defines callable APIs such as cores, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 cores 等可调用 API，用来封装这里提供的核心行为。

### Lines 1286-1286
```tablegen
1286: // -------------------------------- version = 1 --------------------------- //
```
**EN:** This comment block records the intent and constraints of the surrounding code: -------------------------------- version = 1 --------------------------- //.
**CN:** 该注释块解释了周边代码的设计背景、意图或约束条件，帮助读者理解后续实现。

### Lines 1288-1291
```tablegen
1288: For first-gen tensor cores, the implicit warpTileSize is [16, 16].
1289: Note: the layout is different from the recommended in PTX ISA
1290: https://docs.nvidia.com/cuda/parallel-thread-execution/index.html
1291: (mma.884 section, FP32 accumulator).
```
**EN:** This block declares or defines callable APIs such as html, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 html 等可调用 API，用来封装这里提供的核心行为。

### Lines 1293-1294
```tablegen
1293: For example, when versionMinor=1, the matrix L corresponding to
1294: blockTileSize=[32,16] is:
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1296-1313
```tablegen
1296:                                warp 0
1297: --------------------------------/\-------------------------------
1298: [ 0   0   2   2   8   8   10  10   0   0   2   2   8   8   10  10 ]
1299: [ 1   1   3   3   9   9   11  11   1   1   3   3   9   9   11  11 ]
1300: [ 0   0   2   2   8   8   10  10   0   0   2   2   8   8   10  10 ]
1301: [ 1   1   3   3   9   9   11  11   1   1   3   3   9   9   11  11 ]
1302: [ 4   4   6   6   12  12  14  14   4   4   6   6   12  12  14  14 ]
1303: [ 5   5   7   7   13  13  15  15   5   5   7   7   13  13  15  15 ]
1304: [ 4   4   6   6   12  12  14  14   4   4   6   6   12  12  14  14 ]
1305: [ 5   5   7   7   13  13  15  15   5   5   7   7   13  13  15  15 ]
1306: [ 16  16  18  18  20  20  22  22   16  16  18  18  20  20  22  22 ]
1307: [ 17  17  19  19  21  21  23  23   17  17  19  19  21  21  23  23 ]
1308: [ 16  16  18  18  20  20  22  22   16  16  18  18  20  20  22  22 ]
1309: [ 17  17  19  19  21  21  23  23   17  17  19  19  21  21  23  23 ]
1310: [ 24  24  26  26  28  28  30  30   24  24  26  26  28  28  30  30 ]
1311: [ 25  25  27  27  29  29  31  31   25  25  27  27  29  29  31  31 ]
1312: [ 24  24  26  26  28  28  30  30   24  24  26  26  28  28  30  30 ]
1313: [ 25  25  27  27  29  29  31  31   25  25  27  27  29  29  31  31 ]
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1315-1319
```tablegen
1315:                           warp 1 = warp0 + 32
1316: --------------------------------/\-------------------------------
1317: [ 32  32  34  34  40  40  42  42   32  32  34  34  40  40  42  42 ]
1318: [ 33  33  35  35  41  41  43  43   33  33  35  35  41  41  43  43 ]
1319: [ ............................................................... ]
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1322-1322
```tablegen
1322: // -------------------------------- version = 2 --------------------------- //
```
**EN:** This comment block records the intent and constraints of the surrounding code: -------------------------------- version = 2 --------------------------- //.
**CN:** 该注释块解释了周边代码的设计背景、意图或约束条件，帮助读者理解后续实现。

### Lines 1324-1327
```tablegen
1324: For second-gen tensor cores, the implicit warpTileSize is [16, 8].
1325: Information about this layout can be found in the official PTX documentation
1326: https://docs.nvidia.com/cuda/parallel-thread-execution/index.html
1327: (mma.16816 section, FP32 accumulator).
```
**EN:** This block declares or defines callable APIs such as html, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 html 等可调用 API，用来封装这里提供的核心行为。

### Lines 1329-1339
```tablegen
1329: For example, the matrix L corresponding to blockTileSize=[32,16] is:
1330:                 warp 0                          warp 2
1331: -----------------/\-------------  ----------------/\-------------
1332: [ 0   0   1   1   2   2   3   3   32  32  33  33  34  34  35  35
1333: [ 4   4   5   5   6   6   7   7   36  36  37  37  38  38  39  39
1334: [ ..............................  ..............................
1335: [ 28  28  29  29  30  30  31  31  60  60  61  61  62  62  63  63
1336: [ 0   0   1   1   2   2   3   3   32  32  33  33  34  34  35  35
1337: [ 4   4   5   5   6   6   7   7   36  36  37  37  38  38  39  39
1338: [ ..............................  ..............................
1339: [ 28  28  29  29  30  30  31  31  60  60  61  61  62  62  63  63
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1341-1350
```tablegen
1341:               warp 1                           warp 3
1342: ----------------/\-------------   ----------------/\-------------
1343: [ 64  64  65  65  66  66  67  67  96  96  97  97  98  98  99  99
1344: [ 68  68  69  69  70  70  71  71  100 100 101 101 102 102 103 103
1345: [ ..............................  ...............................
1346: [ 92  92  93  93  94  94  95  95  124 124 125 125 126 126 127 127
1347: [ 64  64  65  65  66  66  67  67  96  96  97  97  98  98  99  99
1348: [ 68  68  69  69  70  70  71  71  100 100 101 101 102 102 103 103
1349: [ ..............................  ...............................
1350: [ 92  92  93  93  94  94  95  95  124 124 125 125 126 126 127 127
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1352-1352
```tablegen
1352: }];
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1354-1361
```tablegen
1354:   let parameters = (
1355:     ins
1356:     "unsigned":$versionMajor,
1357:     "unsigned":$versionMinor,
1358:     ArrayRefParameter<"unsigned">:$warpsPerCTA,
1359:     "CGAEncodingAttr":$CGALayout,
1360:     ArrayRefParameter<"unsigned">:$instrShape
1361:   );
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1364-1368
```tablegen
1364:   let extraClassDeclaration = extraDistributedDeclaration # [{
1365:     bool isVolta() const;
1366:     bool isTuring() const;
1367:     bool isAmpere() const;
1368:     bool isHopper() const;
```
**EN:** This block injects extra C++ helpers into the generated class, such as isVolta, isTuring, isAmpere, and isHopper.
**CN:** 该代码块向生成类中注入额外的 C++ 辅助方法，例如 isVolta, isTuring, isAmpere, and isHopper。

### Lines 1370-1374
```tablegen
1370:     SmallVector<int64_t> getRepForOperand(ArrayRef<int64_t> shape,
1371:                                           int bitwidth, int kWidth,
1372:                                           int opIdx) const;
1373:     SmallVector<unsigned> getRepOrderForOperand(int opIdx) const;
1374:   }];
```
**EN:** This block declares or defines callable APIs such as getRepForOperand and getRepOrderForOperand, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getRepForOperand and getRepOrderForOperand 等可调用 API，用来封装这里提供的核心行为。

### Lines 1376-1377
```tablegen
1376:   let hasCustomAssemblyFormat = 1;
1377: }
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1379-1380
```tablegen
1379: def SliceEncodingAttr : DistributedEncoding<"SliceEncoding", "slice_encoding"> {
1380:   let mnemonic = "slice";
```
**EN:** This TableGen def record defines `SliceEncodingAttr`. It is specialized from `DistributedEncoding<"SliceEncoding", "slice_encoding">`.
**CN:** 该 TableGen def 记录定义了 `SliceEncodingAttr`。 它基于 `DistributedEncoding<"SliceEncoding", "slice_encoding">` 进一步特化。

### Lines 1382-1384
```tablegen
1382:   let description = [{
1383:     Given a `parent` layout and a `dim`, squeezes the given `dim` in the `parent`
1384:     layout and distributes values in a tensor T according to the new layout.
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 1386-1386
```tablegen
1386:     For example, given
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1388-1392
```tablegen
1388:     T = [x  x  x  x  x  x  x  x]
1389:     L_parent = [0  1  2  3 ]
1390:                [4  5  6  7 ]
1391:                [8  9  10 11]
1392:                [12 13 14 15] (with 16 CUDA threads)
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1394-1395
```tablegen
1394:     With dim = 0, squeezing out dim 0, we have
1395:     L = [{0,4,8,12},  {1,5,9,13}, {2,6,10,14},  {3,7,11,15} ]
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1397-1398
```tablegen
1397:     Then the data of T would be distributed as follow between the 16 CUDA threads:
1398:     L(T) = [ {0,4,8,12} , {1,5,9,13} , ... {3,7,11,15}, {0,4,8,12} , ..., {3,7,11,15} ]
```
**EN:** This block declares or defines callable APIs such as L, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 L 等可调用 API，用来封装这里提供的核心行为。

### Lines 1400-1401
```tablegen
1400:     With dim = 1, squeezing out dim 1, we have
1401:     L = [ {0,1,2,3}, {4,5,6,7}, {8,9,10,11}, {12,13,14,15} ]
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1403-1404
```tablegen
1403:     Then the data of T would be distributed as follow between the 16 CUDA threads:
1404:     L = [ {0,1,2,3}, {4,5,6,7}, ..., {12,13,14,15}, {0,1,2,3}, ..., {12,13,14,15} ]
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1406-1408
```tablegen
1406:     This is useful for constructing the inverse layout of an expand_dims operation
1407:     during some optimization passes.
1408:   }];
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1410-1414
```tablegen
1410:   let parameters = (
1411:     ins
1412:     "unsigned":$dim,
1413:     "DistributedEncodingTrait":$parent
1414:   );
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1416-1418
```tablegen
1416:   let extraClassDeclaration = extraDistributedDeclaration # [{
1417:     template<class T>
1418:     SmallVector<T> paddedShape(ArrayRef<T> shape) const;
```
**EN:** This block injects extra C++ helpers into the generated class, such as paddedShape.
**CN:** 该代码块向生成类中注入额外的 C++ 辅助方法，例如 paddedShape。

### Lines 1420-1423
```tablegen
1420:     unsigned getRank() const {
1421:       return mlir::cast<LayoutEncodingTrait>(getParent()).getRank() - 1;
1422:     }
1423:   }];
```
**EN:** This block declares or defines callable APIs such as getRank and getParent, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getRank and getParent 等可调用 API，用来封装这里提供的核心行为。

### Lines 1425-1427
```tablegen
1425:   let hasCustomAssemblyFormat = 1;
1426:   let genVerifyDecl = 1;
1427: }
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1429-1430
```tablegen
1429: def DotOperandEncodingAttr : DistributedEncoding<"DotOperandEncoding", "dot_operand_encoding"> {
1430:   let mnemonic = "dot_op";
```
**EN:** This TableGen def record defines `DotOperandEncodingAttr`. It is specialized from `DistributedEncoding<"DotOperandEncoding", "dot_operand_encoding">`.
**CN:** 该 TableGen def 记录定义了 `DotOperandEncodingAttr`。 它基于 `DistributedEncoding<"DotOperandEncoding", "dot_operand_encoding">` 进一步特化。

### Lines 1432-1436
```tablegen
1432:   let description = [{
1433: In the TritonGPU dialect, given `d = tt.dot a, b, c` tt.dot's operands a and b
1434: must be of DotOperandEncodingAttr layout, if the dot is MMA v1 or v2 (i.e.
1435: pre-Hopper).  For MMA v3, the operands are *almost always* in a regular shared
1436: encoding, but sometimes the LHS is also a dot-operand encoding.
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 1438-1438
```tablegen
1438: a's opIdx is 0, b's opIdx is 1.
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1440-1440
```tablegen
1440: The parent field is the layout of d.
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1442-1444
```tablegen
1442: kWidth defines number of consecutive elements stored by one thread along k dimension.
1443: Some layouts do not use this parameter, either because they have a fixed number of
1444: elements along the K dim, or they use all elements of the tensor along the K dim.
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1446-1449
```tablegen
1446: # WGMMA Notes
1447: We require kWidth to be provided for Hopper because the dtype at loading might be
1448: different from the dtype at WGMMA, due to casting. The kWidth is determined by the
1449: dtype at WGMMA.
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1451-1456
```tablegen
1451: The encoded tensor consists of operand A for possibly multiple wgmma instructions.
1452: For each wgmma, each warp in a warp group feeds a single "warp matrix"
1453: Each warp matrix consists of 2x2 "quads".
1454: Each thread holds several elements in each quad. Right before a wgmma,
1455: the sum of bitwidth of
1456: the elements in each quad should add up to 32.
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1458-1466
```tablegen
1458: These values are stored unrolled in `elements`.
1459: The ordering of dimensions is as follows by convention:
1460: batch (only 1 batch for Hopper currently)
1461: matM (m-index of the "warp matrix")
1462: matK (k-index of the "warp matrix")
1463: quadK (k-index of the "quad" in the core matrix)
1464: quadM (m-index of the "quad" in the core matrix)
1465: vecIdx (index of the element in the quad; this is always along the k-dim)
1466:   }];
```
**EN:** This block declares or defines callable APIs such as batch, matM, matK, quadK, quadM, and vecIdx, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 batch, matM, matK, quadK, quadM, and vecIdx 等可调用 API，用来封装这里提供的核心行为。

### Lines 1468-1473
```tablegen
1468:   let parameters = (
1469:     ins
1470:     "unsigned":$opIdx,
1471:     "Attribute":$parent,
1472:     DefaultValuedParameter<"unsigned", "0">:$kWidth
1473:   );
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1475-1487
```tablegen
1475:   let builders = [
1476:     AttrBuilder<(ins "unsigned":$opIdx,
1477:                      "Attribute":$parent,
1478:                      "Type":$eltTy), [{
1479:       NvidiaMmaEncodingAttr parentAttr = mlir::dyn_cast<NvidiaMmaEncodingAttr>(parent);
1480:       if (!parentAttr || (!parentAttr.isAmpere() && !parentAttr.isHopper()))
1481:         return $_get(context, opIdx, parent, 0);
1482:       // For MMAV2 and V3
1483:       unsigned bitwidth = eltTy.getIntOrFloatBitWidth();
1484:       unsigned kWidth = std::max(32 / bitwidth, 1u);
1485:       return $_get(context, opIdx, parent, kWidth);
1486:     }]>
1487:   ];
```
**EN:** This block adds convenience builders so the generated API can construct the entity with common argument patterns.
**CN:** 该代码块添加便捷 builder，使生成 API 能以常见参数组合快速构造该实体。

### Lines 1489-1492
```tablegen
1489:   let assemblyFormat = "`<` `{` struct(params) `}` `>`";
1490:   let genVerifyDecl = 1;
1491:   let extraClassDeclaration = extraDistributedDeclaration;
1492: }
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 1494-1499
```tablegen
1494: def TTG_SharedMemorySpace : AttrDef<TritonGPU_Dialect, "SharedMemorySpace"> {
1495:   let mnemonic = "shared_memory";
1496:   let description = [{
1497:     Attribute to indicate that the memory descriptor points to shared memory.
1498:   }];
1499: }
```
**EN:** This TableGen def record defines `TTG_SharedMemorySpace`. It is specialized from `AttrDef<TritonGPU_Dialect, "SharedMemorySpace">`.
**CN:** 该 TableGen def 记录定义了 `TTG_SharedMemorySpace`。 它基于 `AttrDef<TritonGPU_Dialect, "SharedMemorySpace">` 进一步特化。

### Lines 1501-1501
```tablegen
1501: #endif
```
**EN:** This block closes the header guard started at the top of the file.
**CN:** 该代码块结束文件顶部开始的头文件保护宏。

## Key Concepts / 关键概念
- **EN:** axis reasoning  
  **CN:** 轴信息推理
- **EN:** layout conversion and reasoning  
  **CN:** 布局转换与推理
- **EN:** layout encodings  
  **CN:** 布局编码
- **EN:** matrix-multiply acceleration  
  **CN:** 矩阵乘加加速
- **EN:** dot-product lowering  
  **CN:** 点积降级
- **EN:** MLIR dialect definition  
  **CN:** MLIR 方言定义
- **EN:** interfaces  
  **CN:** 接口
- **EN:** attributes  
  **CN:** 属性

## Dependencies / 依赖关系
- **Project includes / 项目内依赖:**
  - `triton/Dialect/TritonGPU/IR/TritonGPUAttrInterfaces.td`
