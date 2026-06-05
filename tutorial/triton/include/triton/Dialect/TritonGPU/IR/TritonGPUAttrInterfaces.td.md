# TritonGPUAttrInterfaces.td — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./include/triton/Dialect/TritonGPU/IR/TritonGPUAttrInterfaces.td`
- **EN:** Defines reusable interfaces that generated operations or types implement.
- **CN:** 定义生成操作或类型可实现的可复用接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-3
```tablegen
   1: //===----------------------------------------------------------------------===//
   2: // Shared attr interface definitions for TritonGPU and TritonNvidiaGPU.
   3: //===----------------------------------------------------------------------===//
```
**EN:** This comment block records the intent and constraints of the surrounding code: ===----------------------------------------------------------------------===// Shared attr interface definitions for TritonGPU and TritonNvidiaGPU. ===--------------------------....
**CN:** 该注释块解释了周边代码的设计背景、意图或约束条件，帮助读者理解后续实现。

### Lines 5-6
```tablegen
   5: #ifndef TRITONGPU_ATTRINTERFACES_TD
   6: #define TRITONGPU_ATTRINTERFACES_TD
```
**EN:** This block establishes the header guard so the declarations in this file are only processed once per translation unit.
**CN:** 该代码块建立头文件保护宏，避免同一翻译单元重复包含本文件时出现重复声明。

### Lines 8-8
```tablegen
   8: include "triton/Dialect/TritonGPU/IR/TritonGPUAttrBase.td"
```
**EN:** This block imports the direct dependencies needed here, including triton/Dialect/TritonGPU/IR/TritonGPUAttrBase.td.
**CN:** 该代码块引入后续声明直接依赖的头文件，例如 triton/Dialect/TritonGPU/IR/TritonGPUAttrBase.td。

### Lines 10-25
```tablegen
  10: def LayoutEncodingTrait : AttrInterface<"LayoutEncodingTrait"> {
  11:   let cppNamespace = "::mlir::triton::gpu";
  12:   let description = [{
  13:     Common trait for all TTGIR layouts.
  14:   }];
  15:   let methods = [
  16:     InterfaceMethod<"Get the CGA layout backing this encoding.",
  17:                     "CGAEncodingAttr", "getCGALayout">,
  18:     InterfaceMethod<"Get the rank of the layout.", "unsigned", "getRank",
  19:                     (ins), [{}], [{
  20:       return $_attr.getCGALayout().getRank();
  21:     }]>
  22:   ];
  23: }
  24: def DeclareLayoutEncodingMethods : DeclareAttrInterfaceMethods<
  25:   LayoutEncodingTrait, ["getCGALayout"]>;
```
**EN:** This TableGen def record defines `LayoutEncodingTrait`. It is specialized from `AttrInterface<"LayoutEncodingTrait">`.
**CN:** 该 TableGen def 记录定义了 `LayoutEncodingTrait`。 它基于 `AttrInterface<"LayoutEncodingTrait">` 进一步特化。

### Lines 27-28
```tablegen
  27: def SharedEncodingTrait : AttrInterface<"SharedEncodingTrait"> {
  28:   let cppNamespace = "::mlir::triton::gpu";
```
**EN:** This TableGen def record defines `SharedEncodingTrait`. It is specialized from `AttrInterface<"SharedEncodingTrait">`.
**CN:** 该 TableGen def 记录定义了 `SharedEncodingTrait`。 它基于 `AttrInterface<"SharedEncodingTrait">` 进一步特化。

### Lines 30-39
```tablegen
  30:   let description = [{
  31:     Common trait describing shared memory.
  32:   }];
  33:   let methods = [
  34:     InterfaceMethod<"Return the default alignment for the layout.",
  35:                     "int32_t", "getAlignment", (ins), [{}], [{ return 16; }]>,
  36:   ];
  37: }
  38: def DeclareSharedEncodingMethods : DeclareAttrInterfaceMethods<
  39:   SharedEncodingTrait, ["getAlignment"]>;
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 41-42
```tablegen
  41: def DistributedEncodingTrait : AttrInterface<"DistributedEncodingTrait"> {
  42:   let cppNamespace = "::mlir::triton::gpu";
```
**EN:** This TableGen def record defines `DistributedEncodingTrait`. It is specialized from `AttrInterface<"DistributedEncodingTrait">`.
**CN:** 该 TableGen def 记录定义了 `DistributedEncodingTrait`。 它基于 `AttrInterface<"DistributedEncodingTrait">` 进一步特化。

### Lines 44-46
```tablegen
  44:   let description = [{
  45: The Distributed encoding describes the layout L with the 4-level compute hierarchy on GPU.
  46: It is abstracted from the top to the bottom as CTAs Per CGA->Warps Per CTA->Threads Per Warp->Values Per Thread.
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 48-56
```tablegen
  48: For CTAs Per CGA and Warps Per CTA level, the linear id is distributed contiguously with the shape and order.
  49: For example, for a shape/order pair defines a distribution layout
  50: shape = [4, 4]
  51: order = [0, 1] // The fastest-changing axis first
  52: ->
  53: layout = [0  4  8  12]
  54:          [1  5  9  13]
  55:          [2  6  10 14]
  56:          [3  7  11 15]
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 58-58
```tablegen
  58: For the Threads Per Warp and Values Per Thread level, the linear id distribution is variant for each sub-class encoding.
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 60-62
```tablegen
  60: If the layout does not completely cover the tensor, we tile it until we cover the entire tensor.
  61: We call each individual tile "rep".
  62:   }];
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 64-87
```tablegen
  64:   let methods = [
  65:     InterfaceMethod<"Get the order of reps (tiles of this layout that tile the whole tensor). The fastest-changing axis first",
  66:                     "SmallVector<unsigned>",
  67:                     "getRepOrder">,
  68:     InterfaceMethod<"Return total element size per thread.",
  69:                     "unsigned",
  70:                     "getTotalElemsPerThread",
  71:                      (ins "ArrayRef<int64_t>":$shape),
  72:                      /*defaultImplementation=*/[{
  73:                          return toLinearEncoding($_self, shape).getTotalElemsPerThread(shape);
  74:                      }]>,
  75:     InterfaceMethod<"Return element size per thread in each dimension.",
  76:                     "SmallVector<unsigned>",
  77:                     "getElemsPerThread",
  78:                      (ins "ArrayRef<int64_t>":$shape),
  79:                      /*defaultImplementation=*/[{
  80:                          return toLinearEncoding($_self, shape).getElemsPerThread(shape);
  81:                      }]>,
  82:     InterfaceMethod<"Convert to LinearLayout.",
  83:                     "LinearLayout",
  84:                     "toLinearLayout",
  85:                     (ins "ArrayRef<int64_t>":$shape)>,
  86:   ];
  87: }
```
**EN:** This block declares or defines callable APIs such as reps, toLinearEncoding, getTotalElemsPerThread, and getElemsPerThread, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 reps, toLinearEncoding, getTotalElemsPerThread, and getElemsPerThread 等可调用 API，用来封装这里提供的核心行为。

### Lines 89-97
```tablegen
  89: def LinearEncodingTrait : AttrInterface<"LinearEncodingTrait"> {
  90:   let cppNamespace = "::mlir::triton::gpu";
  91:   let description = [{
  92:     Common trait for distributed encodings backed by a LinearLayout
  93:     (LinearEncodingAttr and GenericLinearEncodingAttr).
  94:   }];
  95:   let methods = [
  96:     InterfaceMethod<"Get the underlying linear layout.",
  97:                     "const LinearLayout &", "getLinearLayout">,
```
**EN:** This TableGen def record defines `LinearEncodingTrait`. It is specialized from `AttrInterface<"LinearEncodingTrait">`.
**CN:** 该 TableGen def 记录定义了 `LinearEncodingTrait`。 它基于 `AttrInterface<"LinearEncodingTrait">` 进一步特化。

### Lines 99-165
```tablegen
  99:     InterfaceMethod<[{
 100:       Generalizes getThreadsPerWarp, getWarpsPerCTA, getCTAsPerCGA to linear layouts.
 101:       Returns the bases of the dimensions `dimName` of the layout.
 102:       If skipBroadcast is false, we count a base zero.
 103:     }],
 104:                     "SmallVector<unsigned>", "basesPerDim",
 105:                     (ins "StringAttr":$dimName,
 106:                          "bool":$skipBroadcast),
 107:                     [{}], [{
 108:       return LinearEncodingTrait::basesPerDim(
 109:           $_attr.getLinearLayout(), dimName, skipBroadcast);
 110:     }]>,
 111:     InterfaceMethod<[{
 112:       Generalizes get{Warp,Thread,CTA}Order to linear layouts.
 113:       Returns the order of the dimensions `dimName` of the layout.
 114:       If more than one dimension is of size one, it uses defaultOrder to determine the order
 115:       of the dimensions of size one.
 116:     }],
 117:                     "SmallVector<unsigned>", "orderPerDim",
 118:                     (ins "StringAttr":$dimName,
 119:                          "ArrayRef<unsigned>":$defaultOrder),
 120:                     [{}], [{
 121:       return LinearEncodingTrait::orderPerDim(
 122:           $_attr.getLinearLayout(), dimName, defaultOrder);
 123:     }]>,
 124:     InterfaceMethod<"Get the layout order.",
 125:                     "SmallVector<unsigned>", "getOrder",
 126:                     (ins), [{}], [{
 127:       const auto &ll = $_attr.getLinearLayout();
 128:       auto rank = ll.getNumOutDims();
 129:       SmallVector<unsigned> order(rank);
 130:       // Choose [rank-1, rank-2, ... 0] as the default order in case
 131:       // there are dims that do not move in the register.
 132:       // This order is as good as any.
 133:       std::iota(order.rbegin(), order.rend(), 0);
 134:       return $_attr.orderPerDim(
 135:           StringAttr::get($_attr.getContext(), "register"), order);
 136:     }]>,
 137:     InterfaceMethod<"Get contiguity for a given input dimension.",
 138:                     "SmallVector<unsigned>", "getContig",
 139:                     (ins "const char *":$inDim,
 140:                          "SmallVector<unsigned>":$lowerContig),
 141:                     [{}], [{
 142:       return LinearEncodingTrait::getContig(
 143:           $_attr.getLinearLayout(), inDim, lowerContig, $_attr.getOrder());
 144:     }]>,
 145:     // [FIXME LL] Supports legacy behaviour. We should remove these functions.
 146:     InterfaceMethod<"Get size per thread.",
 147:                     "SmallVector<unsigned>", "getSizePerThread",
 148:                     (ins), [{}], [{
 149:       auto cgaSplitNum = $_attr.getCGALayout().getCTASplitNum();
 150:       return LinearEncodingTrait::getSizePerThread(
 151:           $_attr.getLinearLayout(), cgaSplitNum);
 152:     }]>,
 153:     InterfaceMethod<"Get the CGA layout.",
 154:                     "CGAEncodingAttr", "getCGALayout",
 155:                     (ins), [{}], [{
 156:       return LinearEncodingTrait::getCGALayout($_attr.getLinearLayout());
 157:     }]>,
 158:     InterfaceMethod<"Convert to LinearLayout.",
 159:                     "LinearLayout", "toLinearLayout",
 160:                     (ins "ArrayRef<int64_t>":$shape),
 161:                     [{}], [{
 162:       auto repOrder = $_attr.getRepOrder();
 163:       return LinearEncodingTrait::toLinearLayout(
 164:           $_attr.getLinearLayout(), repOrder, shape);
 165:     }]>,
```
**EN:** This block declares or defines callable APIs such as basesPerDim, getLinearLayout, orderPerDim, getNumOutDims, order, iota, rbegin, and rend, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 basesPerDim, getLinearLayout, orderPerDim, getNumOutDims, order, iota, rbegin, and rend 等可调用 API，用来封装这里提供的核心行为。

### Lines 167-240
```tablegen
 167:     InterfaceMethod<"Get threads per warp in each dimension.",
 168:                     "SmallVector<unsigned>", "getThreadsPerWarp",
 169:                     (ins), [{}], [{
 170:       return $_attr.basesPerDim(
 171:           StringAttr::get($_attr.getContext(), "lane"), /*skipBroadcast=*/true);
 172:     }]>,
 173:     InterfaceMethod<"Get the thread order.",
 174:                     "SmallVector<unsigned>", "getThreadOrder",
 175:                     (ins), [{}], [{
 176:       return $_attr.orderPerDim(
 177:           StringAttr::get($_attr.getContext(), "lane"), $_attr.getOrder());
 178:     }]>,
 179:     InterfaceMethod<"Get contiguity per thread.",
 180:                     "SmallVector<unsigned>", "getContigPerThread",
 181:                     (ins), [{}], [{
 182:       SmallVector<unsigned> c($_attr.getOrder().size(), 1);
 183:       return $_attr.getContig("register", c);
 184:     }]>,
 185:     InterfaceMethod<"Get contiguity per warp.",
 186:                     "SmallVector<unsigned>", "getContigPerWarp",
 187:                     (ins), [{}], [{
 188:       return $_attr.getContig("lane", $_attr.getContigPerThread());
 189:     }]>,
 190:     InterfaceMethod<"Get the rep order (also satisfies DistributedEncodingTrait).",
 191:                     "SmallVector<unsigned>", "getRepOrder",
 192:                     (ins), [{}], [{
 193:       // This is not correct, but:
 194:       // - It happens to agree in most places with the legacy layout
 195:       // - getRepOrder does not make sense for LinearEncodingAttr as it already has
 196:       //   the same shape as the tensor that uses it
 197:       return $_attr.getOrder();
 198:     }]>,
 199:     InterfaceMethod<"Get elements per thread (overrides DistributedEncodingTrait default).",
 200:                     "SmallVector<unsigned>", "getElemsPerThread",
 201:                     (ins "ArrayRef<int64_t>":$shape),
 202:                     [{}], [{
 203:       return LinearEncodingTrait::getElemsPerThread(
 204:           $_attr.getLinearLayout(), $_attr.getOrder(), shape);
 205:     }]>,
 206:     InterfaceMethod<"Get total elements per thread (overrides DistributedEncodingTrait default).",
 207:                     "unsigned", "getTotalElemsPerThread",
 208:                     (ins "ArrayRef<int64_t>":$shape),
 209:                     [{}], [{
 210:       return LinearEncodingTrait::getTotalElemsPerThread(
 211:           $_attr.getLinearLayout(), $_attr.getOrder(), shape);
 212:     }]>,
 213:   ];
 214:   let extraClassDeclaration = [{
 215:     static SmallVector<unsigned> basesPerDim(const LinearLayout &ll,
 216:                                              StringAttr dimName,
 217:                                              bool skipBroadcast);
 218:     static SmallVector<unsigned> orderPerDim(const LinearLayout &ll,
 219:                                              StringAttr dimName,
 220:                                              ArrayRef<unsigned> defaultOrder);
 221:     static SmallVector<unsigned> getContig(const LinearLayout &ll,
 222:                                            const char *inDim,
 223:                                            SmallVector<unsigned> lowerContig,
 224:                                            ArrayRef<unsigned> order);
 225:     static SmallVector<unsigned> getSizePerThread(
 226:         const LinearLayout &ll, ArrayRef<unsigned> cgaSplitNum);
 227:     static CGAEncodingAttr getCGALayout(const LinearLayout &ll);
 228:     static LinearLayout toLinearLayout(const LinearLayout &ll,
 229:                                        ArrayRef<unsigned> repOrder,
 230:                                        ArrayRef<int64_t> shape);
 231:     static SmallVector<unsigned> getElemsPerThread(
 232:         const LinearLayout &ll, ArrayRef<unsigned> repOrder,
 233:         ArrayRef<int64_t> shape);
 234:     static unsigned getTotalElemsPerThread(
 235:         const LinearLayout &ll, ArrayRef<unsigned> repOrder,
 236:         ArrayRef<int64_t> shape);
 237:   }];
 238: }
 239: def DeclareLinearEncodingMethods : DeclareAttrInterfaceMethods<
 240:   LinearEncodingTrait, ["getLinearLayout"]>;
```
**EN:** This block injects extra C++ helpers into the generated class, such as basesPerDim, get, getContext, orderPerDim, getOrder, and size.
**CN:** 该代码块向生成类中注入额外的 C++ 辅助方法，例如 basesPerDim, get, getContext, orderPerDim, getOrder, and size。

### Lines 242-250
```tablegen
 242: def MmaEncodingTrait : AttrInterface<"MmaEncodingTrait"> {
 243:   let cppNamespace = "::mlir::triton::gpu";
 244:   let methods = [
 245:     InterfaceMethod<"Get the order of reps (tiles of this layout that tile the whole tensor). The fastest-changing axis first",
 246:                     "SmallVector<unsigned>",
 247:                     "getRepOrderForOperand",
 248:                     (ins "int":$opIdx)>,
 249:   ];
 250: }
```
**EN:** This TableGen def record defines `MmaEncodingTrait`. It is specialized from `AttrInterface<"MmaEncodingTrait">`.
**CN:** 该 TableGen def 记录定义了 `MmaEncodingTrait`。 它基于 `AttrInterface<"MmaEncodingTrait">` 进一步特化。

### Lines 252-252
```tablegen
 252: #endif // TRITONGPU_ATTRINTERFACES_TD
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
- **EN:** MLIR dialect definition  
  **CN:** MLIR 方言定义
- **EN:** interfaces  
  **CN:** 接口
- **EN:** attributes  
  **CN:** 属性

## Dependencies / 依赖关系
- **Project includes / 项目内依赖:**
  - `triton/Dialect/TritonGPU/IR/TritonGPUAttrBase.td`
