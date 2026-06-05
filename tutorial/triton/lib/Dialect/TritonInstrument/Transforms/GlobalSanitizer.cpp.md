# GlobalSanitizer.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `lib/Dialect/TritonInstrument/Transforms/GlobalSanitizer.cpp`
- **Purpose / 作用:** **EN:** Implements the Global Sanitizer transformation or optimization pass for the TritonInstrument pipeline. **CN:** 为 TritonInstrument 编译流程实现与 Global Sanitizer 相关的变换或优化 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1

```cpp
   1: #include "triton/Dialect/TritonInstrument/Transforms/Passes.h"
```

- **EN:** Includes the interfaces this file depends on. Triton headers (`Passes.h`) provide domain-specific IR/support, MLIR headers (None) provide rewriting and analysis infrastructure, LLVM headers (None) supply low-level utilities, and standard/library headers (None) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（`Passes.h`）提供领域特定 IR/支持逻辑，MLIR 头文件（None）提供重写与分析基础设施，LLVM 头文件（None）提供底层工具，而标准/通用库头文件（None）提供通用能力。
### Lines 3-16

```cpp
   3: #include "mlir/Dialect/Arith/IR/Arith.h"
   4: #include "mlir/IR/Builders.h"
   5: #include "mlir/IR/BuiltinAttributes.h"
   6: #include "mlir/IR/BuiltinOps.h"
   7: #include "triton/Dialect/Triton/IR/Dialect.h"
   8: #include "triton/Dialect/Triton/IR/Types.h"
   9: #include "triton/Dialect/Triton/IR/Utility.h"
  10: #include "triton/Dialect/TritonGPU/IR/Dialect.h"
  11: #include "triton/Dialect/TritonInstrument/IR/Dialect.h"
  12: #include "triton/Dialect/TritonInstrument/IR/Utility.h"
  13: #include "triton/Dialect/TritonNvidiaGPU/IR/Dialect.h"
  14: #include "llvm/ADT/DenseSet.h"
  15: #include "llvm/ADT/STLExtras.h"
  16: #include "llvm/ADT/TypeSwitch.h"
```

- **EN:** Includes the interfaces this file depends on. Triton headers (`Dialect.h`, `Types.h`, `Utility.h`, `Dialect.h`, ... (+3 more)) provide domain-specific IR/support, MLIR headers (`Arith.h`, `Builders.h`, `BuiltinAttributes.h`, `BuiltinOps.h`) provide rewriting and analysis infrastructure, LLVM headers (`DenseSet.h`, `STLExtras.h`, `TypeSwitch.h`) supply low-level utilities, and standard/library headers (None) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（`Dialect.h`, `Types.h`, `Utility.h`, `Dialect.h`, ... (+3 more)）提供领域特定 IR/支持逻辑，MLIR 头文件（`Arith.h`, `Builders.h`, `BuiltinAttributes.h`, `BuiltinOps.h`）提供重写与分析基础设施，LLVM 头文件（`DenseSet.h`, `STLExtras.h`, `TypeSwitch.h`）提供底层工具，而标准/通用库头文件（None）提供通用能力。
### Lines 18-19

```cpp
  18: #include <algorithm>
  19: #include <optional>
```

- **EN:** Includes the interfaces this file depends on. Triton headers (None) provide domain-specific IR/support, MLIR headers (None) provide rewriting and analysis infrastructure, LLVM headers (None) supply low-level utilities, and standard/library headers (`algorithm`, `optional`) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（None）提供领域特定 IR/支持逻辑，MLIR 头文件（None）提供重写与分析基础设施，LLVM 头文件（None）提供底层工具，而标准/通用库头文件（`algorithm`, `optional`）提供通用能力。
### Lines 21-21

```cpp
  21: namespace mlir::triton::instrument {
```

- **EN:** Opens or closes the namespace nesting for mlir::triton::instrument, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 mlir::triton::instrument 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 23-25

```cpp
  23: namespace tt = mlir::triton;
  24: namespace ttg = mlir::triton::gpu;
  25: namespace ttng = mlir::triton::nvidia_gpu;
```

- **EN:** Opens or closes the namespace nesting for tt -> ttg -> ttng, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 tt -> ttg -> ttng 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 27-28

```cpp
  27: #define GEN_PASS_DEF_TRITONINSTRUMENTGLOBALSANITIZER
  28: #include "triton/Dialect/TritonInstrument/Transforms/Passes.h.inc"
```

- **EN:** Pulls in generated definitions from TableGen/MLIR include fragments so the handwritten code can reuse auto-generated declarations.
- **CN:** 这里引入由 TableGen/MLIR 生成的定义片段，使手写代码能够复用自动生成的声明。
### Lines 30-30

```cpp
  30: namespace {
```

- **EN:** Opens or closes the namespace nesting for (anonymous), keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 (anonymous) 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 32-34

```cpp
  32: static constexpr const char kGSanGlobalStateArgAttr[] = "tti.gsan_global_state";
  33: static constexpr const char kDisableSetMaxRegisterAttr[] =
  34:     "tti.disable_setmaxregister";
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 36-40

```cpp
  36: struct DescriptorInfo {
  37:   Value base;
  38:   SmallVector<Value> shape;
  39:   SmallVector<Value> strides;
  40: };
```

- **EN:** Defines `DescriptorInfo`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `DescriptorInfo`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 42-43

```cpp
  42: static void setTMAPtrAxisHints(OpBuilder &builder, Value ptr) {
  43:   auto ptrTy = cast<RankedTensorType>(ptr.getType());
```

- **EN:** Defines accessor/helper `setTMAPtrAxisHints` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `setTMAPtrAxisHints`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 45-47

```cpp
  45:   Operation *def = ptr.getDefiningOp();
  46:   if (!def)
  47:     return;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 49-59

```cpp
  49:   auto rank = ptrTy.getRank();
  50:   SmallVector<int32_t> contiguity(rank, 1);
  51:   contiguity.back() = ptrTy.getShape().back();
  52:   SmallVector<int32_t> divisibility(rank, 1);
  53:   divisibility.back() = 16;
  54:   auto attrTy = RankedTensorType::get({rank}, builder.getI32Type());
  55:   def->setDiscardableAttr("tt.contiguity",
  56:                           DenseIntElementsAttr::get(attrTy, contiguity));
  57:   def->setDiscardableAttr("tt.divisibility",
  58:                           DenseIntElementsAttr::get(attrTy, divisibility));
  59: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 61-65

```cpp
  61: static Value castToI64(OpBuilder &builder, Location loc, Value value) {
  62:   if (value.getType().isInteger(64))
  63:     return value;
  64:   return builder.createOrFold<arith::ExtSIOp>(loc, builder.getI64Type(), value);
  65: }
```

- **EN:** Defines `castToI64`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `castToI64`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 67-74

```cpp
  67: static SmallVector<Value> castToI64(OpBuilder &builder, Location loc,
  68:                                     ValueRange values) {
  69:   SmallVector<Value> result;
  70:   result.reserve(values.size());
  71:   for (Value value : values)
  72:     result.push_back(castToI64(builder, loc, value));
  73:   return result;
  74: }
```

- **EN:** Defines `castToI64`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `castToI64`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 76-93

```cpp
  76: static ttg::BlockedEncodingAttr
  77: getInstrumentationEncoding(OpBuilder &builder, ArrayRef<int64_t> shape,
  78:                            Type elemType) {
  79:   int numWarps = ttg::lookupNumWarps(builder.getInsertionBlock()->getParent());
  80:   int threadsPerWarp = ttg::lookupThreadsPerWarp(builder);
  81:   int numCTAs = ttg::lookupNumCTAs(builder.getInsertionBlock()->getParentOp());
  82:   auto base = ttg::getDefaultBlockedEncoding(builder.getContext(), shape,
  83:                                              numWarps, threadsPerWarp, numCTAs);
  84:   SmallVector<unsigned> order = llvm::to_vector(base.getOrder());
  85:   SmallVector<unsigned> warpsPerCTA = llvm::to_vector(base.getWarpsPerCTA());
  86:   SmallVector<unsigned> sizePerThread(shape.size(), 1);
  87:   unsigned elemBits = elemType.getIntOrFloatBitWidth();
  88:   unsigned maxElems = std::max(128u / elemBits, 1u);
  89:   if (!order.empty()) {
  90:     unsigned dim = order.front();
  91:     // Distribute last dim to maximize contiguity within a thread
  92:     if (order.size() > 1 && warpsPerCTA[dim] > 1) {
  93:       warpsPerCTA[order[1]] *= warpsPerCTA[dim];
```

- **EN:** Defines accessor/helper `getInstrumentationEncoding` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getInstrumentationEncoding`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 94-95

```cpp
  94:       warpsPerCTA[dim] = 1;
  95:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 97-104

```cpp
  97:     auto threadsOnDim = base.getThreadsPerWarp()[dim] * warpsPerCTA[dim];
  98:     auto numUniqueElems = ceil(static_cast<unsigned>(shape[dim]), threadsOnDim);
  99:     sizePerThread[dim] = std::min(maxElems, numUniqueElems);
 100:   }
 101:   return ttg::BlockedEncodingAttr::get(builder.getContext(), sizePerThread,
 102:                                        base.getThreadsPerWarp(), warpsPerCTA,
 103:                                        order, base.getCGALayout());
 104: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 106-116

```cpp
 106: static Value expandAllSlicedDims(OpBuilder &builder, Location loc,
 107:                                  Value tensor) {
 108:   auto type = cast<RankedTensorType>(tensor.getType());
 109:   auto sliceEncoding = dyn_cast<ttg::SliceEncodingAttr>(type.getEncoding());
 110:   while (sliceEncoding) {
 111:     tensor = expandOuterSlicedDim(builder, loc, tensor);
 112:     type = cast<RankedTensorType>(tensor.getType());
 113:     sliceEncoding = dyn_cast<ttg::SliceEncodingAttr>(type.getEncoding());
 114:   }
 115:   return tensor;
 116: }
```

- **EN:** Defines `expandAllSlicedDims`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `expandAllSlicedDims`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 118-125

```cpp
 118: static DescriptorInfo getDescriptorInfo(Value desc, OpBuilder &builder) {
 119:   if (!isa<tt::TensorDescType>(desc.getType())) {
 120:     std::string msg;
 121:     llvm::raw_string_ostream stream(msg);
 122:     stream << "GSan: Unsupported descriptor type" << desc.getType();
 123:     llvm::report_fatal_error(msg.c_str());
 124:   }
 125:   auto descTy = cast<tt::TensorDescType>(desc.getType());
```

- **EN:** Defines accessor/helper `getDescriptorInfo` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getDescriptorInfo`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 127-134

```cpp
 127:   auto elemTy = descTy.getSignlessBlockType().getElementType();
 128:   auto basePtrTy = tt::getPointerType(elemTy);
 129:   unsigned rank = descTy.getShape().size();
 130:   SmallVector<Type> resultTypes;
 131:   resultTypes.reserve(1 + 2 * rank);
 132:   resultTypes.push_back(basePtrTy);
 133:   resultTypes.append(rank, builder.getI64Type());
 134:   resultTypes.append(rank, builder.getI64Type());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 136-138

```cpp
 136:   auto info = ExperimentalGSanTensorDescInfoOp::create(builder, desc.getLoc(),
 137:                                                        resultTypes, desc);
 138:   auto results = info->getResults();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 140-145

```cpp
 140:   DescriptorInfo descriptorInfo;
 141:   descriptorInfo.base = results.front();
 142:   descriptorInfo.shape.assign(results.begin() + 1, results.begin() + 1 + rank);
 143:   descriptorInfo.strides.assign(results.begin() + 1 + rank, results.end());
 144:   return descriptorInfo;
 145: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 147-153

```cpp
 147: static Value createExpandedOffsetRange(OpBuilder &builder, Location loc,
 148:                                        RankedTensorType fullI64Type,
 149:                                        Value offset, unsigned dim) {
 150:   auto fullEncoding =
 151:       cast<ttg::DistributedEncodingTrait>(fullI64Type.getEncoding());
 152:   auto sliceEncoding = getSingleDimSliceEncoding(fullEncoding, dim);
 153:   int64_t dimSize = fullI64Type.getShape()[dim];
```

- **EN:** Defines helper `createExpandedOffsetRange` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createExpandedOffsetRange`，用于计算或构造外围变换所需的中间数据。
### Lines 155-158

```cpp
 155:   auto sliceI32Type =
 156:       RankedTensorType::get({dimSize}, builder.getI32Type(), sliceEncoding);
 157:   auto sliceI64Type =
 158:       RankedTensorType::get({dimSize}, builder.getI64Type(), sliceEncoding);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 160-173

```cpp
 160:   Value range = tt::MakeRangeOp::create(builder, loc, sliceI32Type, 0, dimSize);
 161:   Value rangeI64 = arith::ExtSIOp::create(builder, loc, sliceI64Type, range);
 162:   Value offsetI64 = castToI64(builder, loc, offset);
 163:   Value offsetSplat =
 164:       tt::SplatOp::create(builder, loc, sliceI64Type, offsetI64);
 165:   Value result =
 166:       arith::AddIOp::create(builder, loc, sliceI64Type, offsetSplat, rangeI64);
 167:   result = expandAllSlicedDims(builder, loc, result);
 168:   if (cast<RankedTensorType>(result.getType()).getShape() !=
 169:       fullI64Type.getShape()) {
 170:     result = tt::BroadcastOp::create(builder, loc, fullI64Type, result);
 171:   }
 172:   return result;
 173: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 175-187

```cpp
 175: static Value convertAndBroadcast(OpBuilder &builder, Location loc, Value tensor,
 176:                                  int dim, RankedTensorType dstType) {
 177:   auto tensorType = cast<RankedTensorType>(tensor.getType());
 178:   auto encoding = cast<ttg::DistributedEncodingTrait>(dstType.getEncoding());
 179:   auto sliceEncoding = getSingleDimSliceEncoding(encoding, dim);
 180:   auto sliceType = RankedTensorType::get(
 181:       tensorType.getShape(), tensorType.getElementType(), sliceEncoding);
 182:   tensor = ttg::ConvertLayoutOp::create(builder, loc, sliceType, tensor);
 183:   tensor = expandAllSlicedDims(builder, loc, tensor);
 184:   if (cast<RankedTensorType>(tensor.getType()).getShape() != dstType.getShape())
 185:     tensor = tt::BroadcastOp::create(builder, loc, dstType, tensor);
 186:   return tensor;
 187: }
```

- **EN:** Defines helper `convertAndBroadcast` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `convertAndBroadcast`，用于计算或构造外围变换所需的中间数据。
### Lines 189-196

```cpp
 189: static Value createMaskFromRanges(OpBuilder &builder, Location loc,
 190:                                   const DescriptorInfo &desc,
 191:                                   ArrayRef<Value> offsetRanges,
 192:                                   RankedTensorType fullI64Type) {
 193:   auto maskType = RankedTensorType::get(
 194:       fullI64Type.getShape(), builder.getI1Type(), fullI64Type.getEncoding());
 195:   Value zero = createConstIntTensor(builder, loc, 0, fullI64Type,
 196:                                     /*isSigned=*/true);
```

- **EN:** Defines helper `createMaskFromRanges` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createMaskFromRanges`，用于计算或构造外围变换所需的中间数据。
### Lines 198-213

```cpp
 198:   Value mask;
 199:   for (auto [dim, offsets] : llvm::enumerate(offsetRanges)) {
 200:     Value upperBound =
 201:         tt::SplatOp::create(builder, loc, fullI64Type, desc.shape[dim]);
 202:     Value lower = arith::CmpIOp::create(builder, loc, arith::CmpIPredicate::sge,
 203:                                         offsets, zero);
 204:     Value upper = arith::CmpIOp::create(builder, loc, arith::CmpIPredicate::slt,
 205:                                         offsets, upperBound);
 206:     Value dimMask = arith::AndIOp::create(builder, loc, lower, upper);
 207:     dimMask = cast<RankedTensorType>(dimMask.getType()) == maskType
 208:                   ? dimMask
 209:                   : tt::BroadcastOp::create(builder, loc, maskType, dimMask);
 210:     mask = mask ? arith::AndIOp::create(builder, loc, mask, dimMask) : dimMask;
 211:   }
 212:   return mask;
 213: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 215-231

```cpp
 215: static Value createPtrFromRanges(OpBuilder &builder, Location loc,
 216:                                  const DescriptorInfo &desc,
 217:                                  ArrayRef<Value> offsetRanges,
 218:                                  RankedTensorType fullI64Type) {
 219:   auto ptrTensorType = RankedTensorType::get(
 220:       fullI64Type.getShape(), desc.base.getType(), fullI64Type.getEncoding());
 221:   Value ptr = tt::SplatOp::create(builder, loc, ptrTensorType, desc.base);
 222:   for (auto [dim, offsets] : llvm::enumerate(offsetRanges)) {
 223:     Value stride =
 224:         tt::SplatOp::create(builder, loc, fullI64Type, desc.strides[dim]);
 225:     Value offsetWithStride =
 226:         arith::MulIOp::create(builder, loc, fullI64Type, offsets, stride);
 227:     ptr = tt::AddPtrOp::create(builder, loc, ptrTensorType, ptr,
 228:                                offsetWithStride);
 229:   }
 230:   return ptr;
 231: }
```

- **EN:** Defines helper `createPtrFromRanges` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createPtrFromRanges`，用于计算或构造外围变换所需的中间数据。
### Lines 233-241

```cpp
 233: static std::pair<Value, Value>
 234: createTiledAccess(OpBuilder &builder, Location loc, const DescriptorInfo &desc,
 235:                   ArrayRef<int64_t> blockShape, ValueRange offsets,
 236:                   std::optional<Value> pred) {
 237:   auto encoding = getInstrumentationEncoding(
 238:       builder, blockShape,
 239:       cast<tt::PointerType>(desc.base.getType()).getPointeeType());
 240:   auto fullI64Type =
 241:       RankedTensorType::get(blockShape, builder.getI64Type(), encoding);
```

- **EN:** Defines helper `createTiledAccess` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createTiledAccess`，用于计算或构造外围变换所需的中间数据。
### Lines 243-248

```cpp
 243:   SmallVector<Value> offsetRanges;
 244:   offsetRanges.reserve(offsets.size());
 245:   for (auto [dim, offset] : llvm::enumerate(offsets)) {
 246:     offsetRanges.push_back(
 247:         createExpandedOffsetRange(builder, loc, fullI64Type, offset, dim));
 248:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 250-261

```cpp
 250:   Value ptr =
 251:       createPtrFromRanges(builder, loc, desc, offsetRanges, fullI64Type);
 252:   Value mask =
 253:       createMaskFromRanges(builder, loc, desc, offsetRanges, fullI64Type);
 254:   if (pred) {
 255:     auto maskType = cast<RankedTensorType>(mask.getType());
 256:     Value predTensor = tt::SplatOp::create(builder, loc, maskType, *pred);
 257:     mask = arith::AndIOp::create(builder, loc, mask, predTensor);
 258:   }
 259:   setTMAPtrAxisHints(builder, ptr);
 260:   return std::make_pair(ptr, mask);
 261: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 263-270

```cpp
 263: static std::pair<Value, Value> createGatherScatterAccess(
 264:     OpBuilder &builder, Location loc, const DescriptorInfo &desc,
 265:     ArrayRef<int64_t> blockShape, Value xOffsets, Value yOffset) {
 266:   auto encoding = getInstrumentationEncoding(
 267:       builder, blockShape,
 268:       cast<tt::PointerType>(desc.base.getType()).getPointeeType());
 269:   auto fullI64Type =
 270:       RankedTensorType::get(blockShape, builder.getI64Type(), encoding);
```

- **EN:** Defines helper `createGatherScatterAccess` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createGatherScatterAccess`，用于计算或构造外围变换所需的中间数据。
### Lines 272-288

```cpp
 272:   auto xOffsetsTy = cast<RankedTensorType>(xOffsets.getType());
 273:   auto xOffsetsI64Ty = RankedTensorType::get(
 274:       xOffsetsTy.getShape(), builder.getI64Type(), xOffsetsTy.getEncoding());
 275:   Value xOffsetsI64 =
 276:       arith::ExtSIOp::create(builder, loc, xOffsetsI64Ty, xOffsets);
 277:   Value xRange =
 278:       convertAndBroadcast(builder, loc, xOffsetsI64, /*dim=*/0, fullI64Type);
 279:   Value yRange =
 280:       createExpandedOffsetRange(builder, loc, fullI64Type, yOffset, /*dim=*/1);
 281:   SmallVector<Value> offsetRanges = {xRange, yRange};
 282:   auto ptrs =
 283:       createPtrFromRanges(builder, loc, desc, offsetRanges, fullI64Type);
 284:   auto mask =
 285:       createMaskFromRanges(builder, loc, desc, offsetRanges, fullI64Type);
 286:   setTMAPtrAxisHints(builder, ptrs);
 287:   return std::make_pair(ptrs, mask);
 288: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 290-292

```cpp
 290: static void instrumentAsyncTMALoad(ttng::AsyncTMACopyGlobalToLocalOp op) {
 291:   if (isa<ttng::TensorDescIm2ColType>(op.getDesc().getType()))
 292:     return;
```

- **EN:** Defines `instrumentAsyncTMALoad`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `instrumentAsyncTMALoad`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 294-295

```cpp
 294:   OpBuilder builder(op);
 295:   auto desc = getDescriptorInfo(op.getDesc(), builder);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 297-303

```cpp
 297:   auto offsets = castToI64(builder, op.getLoc(), op.getCoord());
 298:   auto access = createTiledAccess(builder, op.getLoc(), desc,
 299:                                   op.getResult().getType().getShape(), offsets,
 300:                                   op.getPred());
 301:   ExperimentalGSanTensorAccessOp::create(builder, op.getLoc(), access.first,
 302:                                          access.second, /*isStore=*/false);
 303: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 305-309

```cpp
 305: static void instrumentAsyncTMAStore(Operation *op, Value descValue,
 306:                                     ArrayRef<int64_t> blockShape,
 307:                                     ValueRange coords) {
 308:   OpBuilder builder(op);
 309:   auto desc = getDescriptorInfo(descValue, builder);
```

- **EN:** Defines `instrumentAsyncTMAStore`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `instrumentAsyncTMAStore`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 311-316

```cpp
 311:   auto offsets = castToI64(builder, op->getLoc(), coords);
 312:   auto access = createTiledAccess(builder, op->getLoc(), desc, blockShape,
 313:                                   offsets, std::nullopt);
 314:   ExperimentalGSanTensorAccessOp::create(builder, op->getLoc(), access.first,
 315:                                          access.second, /*isStore=*/true);
 316: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 318-320

```cpp
 318: static void instrumentAsyncTMAReduce(ttng::AsyncTMAReduceOp op) {
 319:   OpBuilder builder(op);
 320:   auto desc = getDescriptorInfo(op.getDesc(), builder);
```

- **EN:** Defines `instrumentAsyncTMAReduce`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `instrumentAsyncTMAReduce`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 322-329

```cpp
 322:   auto offsets = castToI64(builder, op.getLoc(), op.getCoord());
 323:   auto access = createTiledAccess(builder, op.getLoc(), desc,
 324:                                   op.getSrc().getType().getShape(), offsets,
 325:                                   std::nullopt);
 326:   ExperimentalGSanAtomicTensorAccessOp::create(
 327:       builder, op.getLoc(), access.first, access.second, MemSemantic::RELAXED,
 328:       MemSyncScope::GPU);
 329: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 331-333

```cpp
 331: static void instrumentAsyncTMAGather(ttng::AsyncTMAGatherOp op) {
 332:   OpBuilder builder(op);
 333:   auto desc = getDescriptorInfo(op.getDesc(), builder);
```

- **EN:** Defines `instrumentAsyncTMAGather`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `instrumentAsyncTMAGather`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 335-345

```cpp
 335:   auto access = createGatherScatterAccess(builder, op.getLoc(), desc,
 336:                                           op.getResult().getType().getShape(),
 337:                                           op.getXOffsets(), op.getYOffset());
 338:   auto maskType = cast<RankedTensorType>(access.second.getType());
 339:   Value predTensor =
 340:       tt::SplatOp::create(builder, op.getLoc(), maskType, op.getPred());
 341:   Value mask =
 342:       arith::AndIOp::create(builder, op.getLoc(), access.second, predTensor);
 343:   ExperimentalGSanTensorAccessOp::create(builder, op.getLoc(), access.first,
 344:                                          mask, /*isStore=*/false);
 345: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 347-349

```cpp
 347: static void instrumentAsyncTMAScatter(ttng::AsyncTMAScatterOp op) {
 348:   OpBuilder builder(op);
 349:   auto desc = getDescriptorInfo(op.getDesc(), builder);
```

- **EN:** Defines `instrumentAsyncTMAScatter`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `instrumentAsyncTMAScatter`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 351-356

```cpp
 351:   auto access = createGatherScatterAccess(builder, op.getLoc(), desc,
 352:                                           op.getSrc().getType().getShape(),
 353:                                           op.getXOffsets(), op.getYOffset());
 354:   ExperimentalGSanTensorAccessOp::create(builder, op.getLoc(), access.first,
 355:                                          access.second, /*isStore=*/true);
 356: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 358-367

```cpp
 358: class GlobalSanitizerPass
 359:     : public impl::TritonInstrumentGlobalSanitizerBase<GlobalSanitizerPass> {
 360: public:
 361:   void runOnOperation() override {
 362:     ModuleOp module = getOperation();
 363:     OpBuilder builder(module);
 364:     Type gsanStatePtrTy = tt::PointerType::get(builder.getI8Type(), 1);
 365:     DenseSet<StringRef> calledFuncs;
 366:     module.walk(
 367:         [&](tt::CallOp callOp) { calledFuncs.insert(callOp.getCallee()); });
```

- **EN:** Defines `GlobalSanitizerPass`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `GlobalSanitizerPass`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 369-377

```cpp
 369:     SmallVector<tt::FuncOp> funcs;
 370:     module.walk([&](tt::FuncOp func) { funcs.push_back(func); });
 371:     for (tt::FuncOp func : funcs) {
 372:       auto funcTy = func.getFunctionType();
 373:       SmallVector<Type> inputTys(funcTy.getInputs().begin(),
 374:                                  funcTy.getInputs().end());
 375:       inputTys.push_back(gsanStatePtrTy);
 376:       func.setType(FunctionType::get(module.getContext(), inputTys,
 377:                                      funcTy.getResults()));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. The implementation traverses IR operations to collect facts or apply rewrites globally. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 379-389

```cpp
 379:       func.getBody().addArgument(gsanStatePtrTy, func.getLoc());
 380:       SmallVector<Attribute> newArgAttrs;
 381:       if (auto argAttrs = func.getAllArgAttrs())
 382:         newArgAttrs.append(argAttrs.begin(), argAttrs.end());
 383:       while (newArgAttrs.size() < func.getNumArguments()) {
 384:         newArgAttrs.push_back(DictionaryAttr::get(module.getContext()));
 385:       }
 386:       if (!newArgAttrs.empty())
 387:         func.setAllArgAttrs(newArgAttrs);
 388:       func.setArgAttr(func.getNumArguments() - 1, kGSanGlobalStateArgAttr,
 389:                       builder.getUnitAttr());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 391-396

```cpp
 391:       bool isEntry = !calledFuncs.contains(func.getSymName());
 392:       if (isEntry) {
 393:         OpBuilder b(&func.front(), func.front().begin());
 394:         ExperimentalGSanInitOp::create(b, func.getLoc());
 395:       }
 396:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 398-403

```cpp
 398:     SmallVector<tt::CallOp> callOps;
 399:     module.walk([&](tt::CallOp op) { callOps.push_back(op); });
 400:     for (tt::CallOp callOp : callOps) {
 401:       auto caller = callOp->getParentOfType<tt::FuncOp>();
 402:       assert(caller && caller.getNumArguments() > 0 &&
 403:              "expected triton.call to be nested under a Triton function");
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. The implementation traverses IR operations to collect facts or apply rewrites globally. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 405-407

```cpp
 405:       SmallVector<Value> operands(callOp.getOperands().begin(),
 406:                                   callOp.getOperands().end());
 407:       operands.push_back(caller.getArgument(caller.getNumArguments() - 1));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 409-416

```cpp
 409:       OpBuilder b(callOp);
 410:       auto newCallOp =
 411:           tt::CallOp::create(b, callOp.getLoc(), callOp.getCallee(),
 412:                              callOp.getResultTypes(), operands);
 413:       newCallOp->setAttrs(callOp->getAttrs());
 414:       callOp->replaceAllUsesWith(newCallOp->getResults());
 415:       callOp.erase();
 416:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 418-434

```cpp
 418:     module.walk([&](Operation *op) {
 419:       IRRewriter b(op);
 420:       mlir::TypeSwitch<Operation *>(op)
 421:           .Case([&](tt::LoadOp op) {
 422:             ExperimentalGSanTensorAccessOp::create(
 423:                 b, op.getLoc(), op.getPtr(), op.getMask(), /*isStore=*/false);
 424:           })
 425:           .Case([&](tt::StoreOp op) {
 426:             ExperimentalGSanTensorAccessOp::create(
 427:                 b, op.getLoc(), op.getPtr(), op.getMask(), /*isStore=*/true);
 428:           })
 429:           .Case([&](ttg::AsyncCopyGlobalToLocalOp op) {
 430:             ExperimentalGSanTensorAccessOp::create(
 431:                 b, op.getLoc(), op.getSrc(), op.getMask(), /*isStore=*/false);
 432:           })
 433:           .Case([&](ttng::AsyncTMACopyGlobalToLocalOp op) {
 434:             instrumentAsyncTMALoad(op);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. The implementation traverses IR operations to collect facts or apply rewrites globally.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。
### Lines 435-452

```cpp
 435:           })
 436:           .Case(
 437:               [&](ttng::AsyncTMAGatherOp op) { instrumentAsyncTMAGather(op); })
 438:           .Case([&](ttng::AsyncTMACopyLocalToGlobalOp op) {
 439:             instrumentAsyncTMAStore(op, op.getDesc(),
 440:                                     op.getSrc().getType().getShape(),
 441:                                     op.getCoord());
 442:           })
 443:           .Case(
 444:               [&](ttng::AsyncTMAReduceOp op) { instrumentAsyncTMAReduce(op); })
 445:           .Case([&](ttng::AsyncTMAScatterOp op) {
 446:             instrumentAsyncTMAScatter(op);
 447:           })
 448:           .Case([&](tt::AtomicRMWOp op) {
 449:             auto newOp = ExperimentalGSanAtomicRMWOp::create(
 450:                 b, op.getLoc(), op.getType(), op.getAtomicRmwOp(), op.getPtr(),
 451:                 op.getVal(), op.getMask(), op.getSem(), op.getScope());
 452:             newOp->setAttrs(op->getAttrs());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 453-467

```cpp
 453:             b.replaceOp(op, newOp);
 454:           })
 455:           .Case([&](tt::AtomicCASOp op) {
 456:             auto newOp = ExperimentalGSanAtomicCASOp::create(
 457:                 b, op.getLoc(), op.getType(), op.getPtr(), op.getCmp(),
 458:                 op.getVal(), op.getSem(), op.getScope());
 459:             newOp->setAttrs(op->getAttrs());
 460:             b.replaceOp(op, newOp);
 461:           })
 462:           .Case([&](ttg::WarpSpecializeOp op) {
 463:             op->setAttr(kDisableSetMaxRegisterAttr, builder.getUnitAttr());
 464:           });
 465:     });
 466:   }
 467: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. It finishes by replacing the original operation with the lowered form.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 它会在转换成功后用降级后的形式替换原始操作。
### Lines 469-469

```cpp
 469: } // namespace
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 471-471

```cpp
 471: } // namespace mlir::triton::instrument
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。

## Key Concepts / 关键概念
- **EN:** The main concern is pass-driven transformation around global sanitizer.
  **CN:** 核心关注点是围绕 Global Sanitizer 的 pass 驱动变换。
- **EN:** Tensor shape and element-type reasoning is central to the implementation.
  **CN:** 张量形状与元素类型推导是实现中的核心内容。
- **EN:** Encoding attributes describe how distributed GPU data is laid out across threads and warps.
  **CN:** 编码属性描述 GPU 分布式数据在线程与 warp 之间的布局方式。
- **EN:** The code reasons at module scope rather than only on isolated operations.
  **CN:** 代码在模块范围内进行分析，而不只是处理单个操作。
- **EN:** Tensor Memory Accelerator related logic appears in this implementation.
  **CN:** 此实现涉及 Tensor Memory Accelerator（TMA）相关逻辑。
- **EN:** Hardware execution parameters such as warps and threads-per-warp affect legality and performance decisions.
  **CN:** warp 数与每个 warp 的线程数等硬件执行参数会影响合法性和性能决策。

## Dependencies / 依赖关系
- **Internal Triton headers / Triton 内部头文件:** `triton/Dialect/TritonInstrument/Transforms/Passes.h`, `triton/Dialect/Triton/IR/Dialect.h`, `triton/Dialect/Triton/IR/Types.h`, `triton/Dialect/Triton/IR/Utility.h`, `triton/Dialect/TritonGPU/IR/Dialect.h`, `triton/Dialect/TritonInstrument/IR/Dialect.h`, ... (+3 more)
- **MLIR headers / MLIR 头文件:** `mlir/Dialect/Arith/IR/Arith.h`, `mlir/IR/Builders.h`, `mlir/IR/BuiltinAttributes.h`, `mlir/IR/BuiltinOps.h`
- **LLVM headers / LLVM 头文件:** `llvm/ADT/DenseSet.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/TypeSwitch.h`
- **Standard/library headers / 标准或通用库头文件:** `algorithm`, `optional`
- **Generated fragments / 生成片段:** `triton/Dialect/TritonInstrument/Transforms/Passes.h.inc`
- **Primary APIs used / 主要 API:** `ModuleOp`, `RankedTensorType`, `OpBuilder`
- **Pipeline role / 流程角色:** This file participates in Triton pass pipelines and usually expects upstream analyses or dialect invariants to have prepared the IR. / 本文件参与 Triton 的 pass 流程，通常依赖上游分析或方言不变量先把 IR 准备好。
