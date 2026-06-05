# FunctionBuilder.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `lib/Dialect/TritonInstrument/IR/FunctionBuilder.cpp`
- **Purpose / 作用:** **EN:** Provides IR-level support code for the TritonInstrument dialect around Function Builder. **CN:** 为 TritonInstrument 方言提供与 Function Builder 相关的 IR 层支持代码。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1

```cpp
   1: #include "triton/Dialect/TritonInstrument/IR/FunctionBuilder.h"
```

- **EN:** Includes the interfaces this file depends on. Triton headers (`FunctionBuilder.h`) provide domain-specific IR/support, MLIR headers (None) provide rewriting and analysis infrastructure, LLVM headers (None) supply low-level utilities, and standard/library headers (None) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（`FunctionBuilder.h`）提供领域特定 IR/支持逻辑，MLIR 头文件（None）提供重写与分析基础设施，LLVM 头文件（None）提供底层工具，而标准/通用库头文件（None）提供通用能力。
### Lines 3-3

```cpp
   3: #include <cassert>
```

- **EN:** Includes the interfaces this file depends on. Triton headers (None) provide domain-specific IR/support, MLIR headers (None) provide rewriting and analysis infrastructure, LLVM headers (None) supply low-level utilities, and standard/library headers (`cassert`) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（None）提供领域特定 IR/支持逻辑，MLIR 头文件（None）提供重写与分析基础设施，LLVM 头文件（None）提供底层工具，而标准/通用库头文件（`cassert`）提供通用能力。
### Lines 5-15

```cpp
   5: #include "mlir/Dialect/Arith/IR/Arith.h"
   6: #include "mlir/Dialect/ControlFlow/IR/ControlFlowOps.h"
   7: #include "mlir/IR/Builders.h"
   8: #include "mlir/IR/ImplicitLocOpBuilder.h"
   9: #include "mlir/Support/DebugStringHelper.h"
  10: #include "triton/Conversion/TritonGPUToLLVM/Utility.h"
  11: #include "triton/Dialect/TritonGPU/IR/Dialect.h"
  12: #include "triton/Dialect/TritonGPU/IR/TritonGPUInterfaces.h"
  13: #include "triton/Dialect/TritonInstrument/IR/Dialect.h"
  14: #include "triton/Dialect/TritonInstrument/IR/Utility.h"
  15: #include "triton/Dialect/TritonNvidiaGPU/IR/Dialect.h"
```

- **EN:** Includes the interfaces this file depends on. Triton headers (`Utility.h`, `Dialect.h`, `TritonGPUInterfaces.h`, `Dialect.h`, ... (+2 more)) provide domain-specific IR/support, MLIR headers (`Arith.h`, `ControlFlowOps.h`, `Builders.h`, `ImplicitLocOpBuilder.h`, ... (+1 more)) provide rewriting and analysis infrastructure, LLVM headers (None) supply low-level utilities, and standard/library headers (None) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（`Utility.h`, `Dialect.h`, `TritonGPUInterfaces.h`, `Dialect.h`, ... (+2 more)）提供领域特定 IR/支持逻辑，MLIR 头文件（`Arith.h`, `ControlFlowOps.h`, `Builders.h`, `ImplicitLocOpBuilder.h`, ... (+1 more)）提供重写与分析基础设施，LLVM 头文件（None）提供底层工具，而标准/通用库头文件（None）提供通用能力。
### Lines 17-17

```cpp
  17: namespace mlir::triton::instrument {
```

- **EN:** Opens or closes the namespace nesting for mlir::triton::instrument, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 mlir::triton::instrument 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 19-21

```cpp
  19: namespace ttg = mlir::triton::gpu;
  20: namespace ttng = mlir::triton::nvidia_gpu;
  21: namespace tti = mlir::triton::instrument;
```

- **EN:** Opens or closes the namespace nesting for ttg -> ttng -> tti, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 ttg -> ttng -> tti 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 23-40

```cpp
  23: std::string mangleType(Type t) {
  24:   if (auto intType = dyn_cast<IntegerType>(t)) {
  25:     return ("I" + Twine(intType.getWidth())).str();
  26:   }
  27:   if (auto floatType = dyn_cast<FloatType>(t)) {
  28:     return ("F" + Twine(floatType.getWidth())).str();
  29:   }
  30:   if (auto ptrType = dyn_cast<PointerType>(t)) {
  31:     return "P";
  32:   }
  33:   if (auto tensorType = dyn_cast<RankedTensorType>(t)) {
  34:     std::string result = "T";
  35:     llvm::raw_string_ostream os(result);
  36:     for (int s : tensorType.getShape()) {
  37:       os << s << "x";
  38:     }
  39:     os << mangleType(tensorType.getElementType());
  40:     return result;
```

- **EN:** Defines `mangleType`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `mangleType`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 41-44

```cpp
  41:   }
  42:   // Fallback to hash of the type's string representation.
  43:   return "U" + llvm::utohexstr(llvm::hash_value(mlir::debugString(t)));
  44: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 46-46

```cpp
  46: namespace {
```

- **EN:** Opens or closes the namespace nesting for (anonymous), keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 (anonymous) 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 48-58

```cpp
  48: namespace BarrierBits {
  49: constexpr unsigned initCountLsb = 1;
  50: constexpr unsigned currentCountLsb = 21;
  51: constexpr unsigned txCountLsb = 41;
  52: constexpr unsigned countBitWidth = 20;
  53: constexpr unsigned txCountBitWidth = 21;
  54: constexpr uint64_t countMask = (1ull << countBitWidth) - 1;
  55: constexpr uint64_t txCountMask = (1ull << txCountBitWidth) - 1;
  56: constexpr int64_t txCountMin = -(int64_t)countMask;
  57: constexpr int64_t txCountMax = (int64_t)countMask;
  58: } // namespace BarrierBits
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 60-63

```cpp
  60: namespace WaitingBits {
  61: constexpr unsigned bitsPerThread = 2;
  62: constexpr unsigned flagBit = 0;
  63: constexpr unsigned phaseBit = 1;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 65-71

```cpp
  65: uint32_t makeInterleavedMask(unsigned bit, unsigned numBaseThreads) {
  66:   uint32_t mask = 0;
  67:   for (unsigned i = 0; i < numBaseThreads; ++i)
  68:     mask |= 1u << (bitsPerThread * i + bit);
  69:   return mask;
  70: }
  71: } // namespace WaitingBits
```

- **EN:** Defines `makeInterleavedMask`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `makeInterleavedMask`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 73-77

```cpp
  73: // Information about the optional assert message and tensor type to check.
  74: struct AssertInfo {
  75:   StringRef message;
  76:   Type type;
  77: };
```

- **EN:** Defines `AssertInfo`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `AssertInfo`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 79-87

```cpp
  79: static uint64_t expandActiveMask(uint64_t activeMask, unsigned numBaseThreads) {
  80:   uint64_t expanded = 0;
  81:   for (unsigned i = 0; i < numBaseThreads; ++i) {
  82:     if (activeMask & (1ull << i))
  83:       expanded |=
  84:           1ull << (WaitingBits::bitsPerThread * i + WaitingBits::flagBit);
  85:   }
  86:   return expanded;
  87: }
```

- **EN:** Defines `expandActiveMask`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `expandActiveMask`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 89-95

```cpp
  89: Value createCmpIntTensorScalar(
  90:     ImplicitLocOpBuilder &b, Value tensor, Value scalar,
  91:     arith::CmpIPredicate predicate = arith::CmpIPredicate::eq) {
  92:   auto tensorTy = cast<RankedTensorType>(tensor.getType());
  93:   Value splat = triton::SplatOp::create(b, tensorTy, scalar);
  94:   return arith::CmpIOp::create(b, predicate, tensor, splat);
  95: }
```

- **EN:** Defines helper `createCmpIntTensorScalar` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createCmpIntTensorScalar`，用于计算或构造外围变换所需的中间数据。
### Lines 97-112

```cpp
  97: template <typename OpTy>
  98: Value reduceLastDim(ImplicitLocOpBuilder &b, Value tensor) {
  99:   OpBuilder::InsertionGuard guard(b);
 100:   auto tensorType = cast<RankedTensorType>(tensor.getType());
 101:   assert(tensorType.getRank() > 0 && "cannot reduce a rank-0 tensor");
 102:   int axis = tensorType.getRank() - 1;
 103:   auto reduceOp = triton::ReduceOp::create(b, std::vector<Value>{tensor}, axis);
 104:   auto &region = reduceOp.getRegion();
 105:   auto &block = region.emplaceBlock();
 106:   block.addArguments({tensorType.getElementType(), tensorType.getElementType()},
 107:                      {b.getLoc(), b.getLoc()});
 108:   b.setInsertionPointToStart(&block);
 109:   auto result = OpTy::create(b, block.getArgument(0), block.getArgument(1));
 110:   triton::ReduceReturnOp::create(b, std::vector<Value>{result});
 111:   return reduceOp->getResult(0);
 112: }
```

- **EN:** Defines `reduceLastDim`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `reduceLastDim`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 114-117

```cpp
 114: template <typename OpTy>
 115: Value reduce(ImplicitLocOpBuilder &b, Value tensor, ArrayRef<int> axes) {
 116:   auto tensorType = cast<RankedTensorType>(tensor.getType());
 117:   assert(!axes.empty() && "expected at least one reduction axis");
```

- **EN:** Defines `reduce`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `reduce`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 119-124

```cpp
 119:   llvm::SmallDenseSet<int> reducedAxes;
 120:   for (int axis : axes) {
 121:     assert(axis >= 0 && axis < tensorType.getRank() &&
 122:            "invalid reduction axis");
 123:     assert(reducedAxes.insert(axis).second && "duplicate reduction axis");
 124:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 126-133

```cpp
 126:   SmallVector<int32_t> transposeOrder;
 127:   SmallVector<int64_t> flattenedShape;
 128:   for (int dim = 0; dim < tensorType.getRank(); ++dim) {
 129:     if (reducedAxes.contains(dim))
 130:       continue;
 131:     transposeOrder.push_back(dim);
 132:     flattenedShape.push_back(tensorType.getShape()[dim]);
 133:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 135-140

```cpp
 135:   int64_t reducedSize = 1;
 136:   for (int axis : axes) {
 137:     transposeOrder.push_back(axis);
 138:     reducedSize *= tensorType.getShape()[axis];
 139:   }
 140:   flattenedShape.push_back(reducedSize);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 142-145

```cpp
 142:   tensor = triton::TransOp::create(b, tensor, transposeOrder);
 143:   tensor = triton::ReshapeOp::create(b, flattenedShape, tensor);
 144:   return reduceLastDim<OpTy>(b, tensor);
 145: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 147-157

```cpp
 147: template <typename OpTy>
 148: Value reduceAll(ImplicitLocOpBuilder &b, Value tensor) {
 149:   auto tensorType = dyn_cast<RankedTensorType>(tensor.getType());
 150:   if (!tensorType || tensorType.getRank() == 0)
 151:     return tensor;
 152:   if (tensorType.getRank() != 1) {
 153:     tensor = triton::ReshapeOp::create(b, {tensorType.getNumElements()}, tensor,
 154:                                        /*allowReorder=*/true);
 155:   }
 156:   return reduceLastDim<OpTy>(b, tensor);
 157: }
```

- **EN:** Defines `reduceAll`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `reduceAll`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 159-172

```cpp
 159: FuncOp getOrCreateFunction(
 160:     ModuleOp module, const std::string &name, llvm::ArrayRef<Type> argTypes,
 161:     ManglingArgs specializationArgs, int numWarps, Type assertType,
 162:     std::function<void(ImplicitLocOpBuilder &b, Block *entryBlock)> buildBody) {
 163:   ManglingArgs manglingArgs;
 164:   manglingArgs.append(argTypes);
 165:   manglingArgs.append(specializationArgs);
 166:   if (assertType) {
 167:     manglingArgs.append(assertType);
 168:   }
 169:   std::string funcName = manglingArgs.mangle(name, numWarps);
 170:   if (auto existing = module.lookupSymbol<FuncOp>(funcName)) {
 171:     return existing;
 172:   }
```

- **EN:** Defines accessor/helper `getOrCreateFunction` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getOrCreateFunction`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 174-191

```cpp
 174:   OpBuilder moduleBuilder(module.getContext());
 175:   moduleBuilder.setInsertionPointToStart(module.getBody());
 176:   Location loc = module.getLoc();
 177:   SmallVector<Type> resultTypes = {};
 178:   if (assertType) {
 179:     resultTypes.push_back(assertType);
 180:   }
 181:   auto funcType = moduleBuilder.getFunctionType(argTypes, resultTypes);
 182:   FuncOp func = FuncOp::create(moduleBuilder, loc, funcName, funcType);
 183:   func.setVisibility(SymbolTable::Visibility::Private);
 184:   func->setAttr(ttg::AttrNumWarpsName,
 185:                 moduleBuilder.getI32IntegerAttr(numWarps));
 186:   func->setAttr("always_use_warp_shuffle", moduleBuilder.getUnitAttr());
 187:   for (auto [i, argType] : llvm::enumerate(argTypes)) {
 188:     if (isa<PointerType>(argType)) {
 189:       func.setArgAttr(i, "tt.divisibility",
 190:                       moduleBuilder.getI32IntegerAttr(16));
 191:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 192-198

```cpp
 192:   }
 193:   Block *entryBlock = func.addEntryBlock();
 194:   OpBuilder bodyBuilder = OpBuilder::atBlockBegin(entryBlock);
 195:   ImplicitLocOpBuilder fb(loc, bodyBuilder);
 196:   buildBody(fb, entryBlock);
 197:   return func;
 198: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 200-217

```cpp
 200: // Create a call to a function with body given by `buildBody`.
 201: // If the function does not exist, it will be created, otherwise the
 202: // existing function will be used.
 203: // If `assertInfo` is provided, the function should return a tensor of
 204: // the given type and the result of the function will be asserted.
 205: void createCallToCachedFunction(
 206:     ImplicitLocOpBuilder &b, const std::string &name, ArrayRef<Value> args,
 207:     std::optional<AssertInfo> assertInfo, ManglingArgs specializationArgs,
 208:     std::function<void(ImplicitLocOpBuilder &b, Block *entryBlock)> buildBody) {
 209:   ModuleOp module = b.getInsertionPoint()->getParentOfType<ModuleOp>();
 210:   int numWarps = ttg::lookupNumWarps(b.getInsertionPoint()->getParentRegion());
 211:   SmallVector<Type> argTypes = llvm::to_vector(
 212:       llvm::map_range(args, [](Value v) { return v.getType(); }));
 213:   Type assertType = assertInfo ? assertInfo->type : nullptr;
 214:   triton::FuncOp func =
 215:       getOrCreateFunction(module, name, argTypes, specializationArgs, numWarps,
 216:                           assertType, buildBody);
 217:   SmallVector<Type> resultTypes = {};
```

- **EN:** Defines helper `createCallToCachedFunction` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createCallToCachedFunction`，用于计算或构造外围变换所需的中间数据。
### Lines 218-227

```cpp
 218:   if (assertInfo) {
 219:     resultTypes.push_back(assertInfo->type);
 220:   }
 221:   auto callOp = triton::CallOp::create(b, func.getName(), resultTypes, args);
 222:   if (assertInfo) {
 223:     Value result = callOp->getResult(0);
 224:     StringRef message = b.getStringAttr(assertInfo->message);
 225:     createAssertInThread(b, result, message);
 226:   }
 227: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 229-237

```cpp
 229: Value createBufferDescriptor(ImplicitLocOpBuilder &b, Value offsetI32,
 230:                              Value lengthI32) {
 231:   auto i64Type = b.getI64Type();
 232:   Value offsetI64 = arith::ExtUIOp::create(b, i64Type, offsetI32);
 233:   Value lengthI64 = arith::ExtUIOp::create(b, i64Type, lengthI32);
 234:   Value shiftAmount = arith::ConstantIntOp::create(b, 32, 64);
 235:   Value lengthShifted = arith::ShLIOp::create(b, lengthI64, shiftAmount);
 236:   return arith::OrIOp::create(b, lengthShifted, offsetI64);
 237: }
```

- **EN:** Defines helper `createBufferDescriptor` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createBufferDescriptor`，用于计算或构造外围变换所需的中间数据。
### Lines 239-248

```cpp
 239: std::tuple<Block *, Block *, Block *> createIfBlock(ImplicitLocOpBuilder &b,
 240:                                                     Value cnd) {
 241:   // #prevBlock
 242:   // if (condition) {
 243:   //   #ifBlock
 244:   // }
 245:   // #thenBlock
 246:   Block *prevBlock = b.getInsertionBlock();
 247:   Block::iterator insertPoint = b.getInsertionPoint();
 248:   Block *ifBlock = prevBlock->splitBlock(insertPoint);
```

- **EN:** Defines helper `createIfBlock` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createIfBlock`，用于计算或构造外围变换所需的中间数据。
### Lines 250-257

```cpp
 250:   // Split a block after the call.
 251:   Block *thenBlock = ifBlock->splitBlock(ifBlock->begin());
 252:   b.setInsertionPointToEnd(ifBlock);
 253:   cf::BranchOp::create(b, thenBlock);
 254:   b.setInsertionPointToEnd(prevBlock);
 255:   cf::CondBranchOp::create(b, cnd, ifBlock, ValueRange{}, thenBlock,
 256:                            ValueRange{});
 257:   b.setInsertionPointToStart(thenBlock);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 259-260

```cpp
 259:   return {prevBlock, ifBlock, thenBlock};
 260: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 262-267

```cpp
 262: Value createConvertLayout(ImplicitLocOpBuilder &b, Value tensor,
 263:                           Attribute encoding) {
 264:   auto tensorType = cast<RankedTensorType>(tensor.getType());
 265:   auto dstType = tensorType.cloneWithEncoding(encoding);
 266:   return ttg::ConvertLayoutOp::create(b, dstType, tensor);
 267: }
```

- **EN:** Defines helper `createConvertLayout` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createConvertLayout`，用于计算或构造外围变换所需的中间数据。
### Lines 269-286

```cpp
 269: Value convertAndBroadcast(ImplicitLocOpBuilder &b, Value tensor,
 270:                           ArrayRef<int> keptDims, RankedTensorType dstType) {
 271:   auto loc = b.getLoc();
 272:   auto tensorType = cast<RankedTensorType>(tensor.getType());
 273:   auto encoding = cast<ttg::DistributedEncodingTrait>(dstType.getEncoding());
 274:   assert(static_cast<size_t>(tensorType.getRank()) == keptDims.size() &&
 275:          "expected one kept dimension per source tensor rank");
 276:   llvm::SmallDenseSet<int> keptDimsSet(keptDims.begin(), keptDims.end());
 277:   Attribute sliceEncoding = encoding;
 278:   for (int dim = encoding.getRepOrder().size() - 1; dim >= 0; --dim) {
 279:     if (!keptDimsSet.contains(dim))
 280:       sliceEncoding = ttg::SliceEncodingAttr::get(
 281:           b.getContext(), dim,
 282:           cast<ttg::DistributedEncodingTrait>(sliceEncoding));
 283:   }
 284:   tensor = createConvertLayout(b, tensor, sliceEncoding);
 285:   while (cast<RankedTensorType>(tensor.getType()).getRank() < dstType.getRank())
 286:     tensor = tti::expandOuterSlicedDim(b, loc, tensor);
```

- **EN:** Defines helper `convertAndBroadcast` that computes or constructs intermediate data used by the surrounding transformation. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义辅助函数 `convertAndBroadcast`，用于计算或构造外围变换所需的中间数据。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 287-292

```cpp
 287:   auto resultType = RankedTensorType::get(
 288:       dstType.getShape(), tensorType.getElementType(), encoding);
 289:   if (cast<RankedTensorType>(tensor.getType()) == resultType)
 290:     return tensor;
 291:   return triton::BroadcastOp::create(b, resultType, tensor);
 292: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 294-304

```cpp
 294: Value expandAliases(ImplicitLocOpBuilder &b, Value bufferMask,
 295:                     Value aliasMatrix, RankedTensorType aliasMatrixType) {
 296:   assert(aliasMatrixType.getRank() == 2 &&
 297:          "Alias matrix expected to be rank-2");
 298:   auto bufferMaskType = cast<RankedTensorType>(bufferMask.getType());
 299:   Value bufMaskMatrix =
 300:       convertAndBroadcast(b, bufferMask, {0}, aliasMatrixType);
 301:   Value aliasingMask = arith::AndIOp::create(b, aliasMatrix, bufMaskMatrix);
 302:   Value aliasVector = reduce<arith::OrIOp>(b, aliasingMask, {0});
 303:   return createConvertLayout(b, aliasVector, bufferMaskType.getEncoding());
 304: }
```

- **EN:** Defines `expandAliases`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `expandAliases`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 306-314

```cpp
 306: Value adjustIntegerWidth(ImplicitLocOpBuilder &b, Value value,
 307:                          IntegerType targetType) {
 308:   auto srcType = cast<IntegerType>(value.getType());
 309:   if (srcType.getWidth() == targetType.getWidth())
 310:     return value;
 311:   if (srcType.getWidth() < targetType.getWidth())
 312:     return arith::ExtUIOp::create(b, targetType, value);
 313:   return arith::TruncIOp::create(b, targetType, value);
 314: }
```

- **EN:** Defines `adjustIntegerWidth`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `adjustIntegerWidth`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 316-321

```cpp
 316: Value createThreadColumnMask(ImplicitLocOpBuilder &b, Value threadMask,
 317:                              RankedTensorType tensorType, int columnDim) {
 318:   auto loc = b.getLoc();
 319:   auto encoding = cast<ttg::DistributedEncodingTrait>(tensorType.getEncoding());
 320:   auto sliceEncoding = tti::getSingleDimSliceEncoding(encoding, columnDim);
 321:   int columns = tensorType.getShape()[columnDim];
```

- **EN:** Defines helper `createThreadColumnMask` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createThreadColumnMask`，用于计算或构造外围变换所需的中间数据。
### Lines 323-325

```cpp
 323:   RankedTensorType rangeType =
 324:       RankedTensorType::get({columns}, b.getI32Type(), sliceEncoding);
 325:   Value range = triton::MakeRangeOp::create(b, rangeType, 0, columns);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 327-332

```cpp
 327:   auto elemType = cast<IntegerType>(tensorType.getElementType());
 328:   RankedTensorType rangeElemType =
 329:       RankedTensorType::get({columns}, elemType, sliceEncoding);
 330:   Value rangeElem = range;
 331:   if (elemType.getWidth() != 32)
 332:     rangeElem = arith::ExtUIOp::create(b, rangeElemType, range);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 334-334

```cpp
 334:   Value indices = convertAndBroadcast(b, rangeElem, {columnDim}, tensorType);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 336-337

```cpp
 336:   Value threadMaskElem = adjustIntegerWidth(b, threadMask, elemType);
 337:   Value maskTensor = triton::SplatOp::create(b, tensorType, threadMaskElem);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 339-344

```cpp
 339:   Value shifted = arith::ShRUIOp::create(b, maskTensor, indices);
 340:   Value one = tti::createConstIntTensor(b, loc, 1, tensorType);
 341:   Value bits = arith::AndIOp::create(b, shifted, one);
 342:   Value zero = tti::createConstIntTensor(b, loc, 0, tensorType);
 343:   return arith::CmpIOp::create(b, arith::CmpIPredicate::ne, bits, zero);
 344: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 346-361

```cpp
 346: Value createDimMask(ImplicitLocOpBuilder &b, Value index,
 347:                     RankedTensorType tensorType, int dim) {
 348:   assert(dim >= 0 && dim < tensorType.getRank() && "invalid tensor dimension");
 349:   auto encoding = cast<ttg::DistributedEncodingTrait>(tensorType.getEncoding());
 350:   auto sliceEncoding = tti::getSingleDimSliceEncoding(encoding, dim);
 351:   auto indexType = RankedTensorType::get({tensorType.getShape()[dim]},
 352:                                          b.getI32Type(), sliceEncoding);
 353:   Value range = triton::MakeRangeOp::create(b, indexType, /*start=*/0,
 354:                                             /*end=*/tensorType.getShape()[dim]);
 355:   Value indexTensor = triton::SplatOp::create(b, indexType, index);
 356:   Value mask1D =
 357:       arith::CmpIOp::create(b, arith::CmpIPredicate::eq, range, indexTensor);
 358:   auto maskType =
 359:       cast<RankedTensorType>(tensorType.cloneWith(std::nullopt, b.getI1Type()));
 360:   return convertAndBroadcast(b, mask1D, {dim}, maskType);
 361: }
```

- **EN:** Defines helper `createDimMask` that computes or constructs intermediate data used by the surrounding transformation. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义辅助函数 `createDimMask`，用于计算或构造外围变换所需的中间数据。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 363-375

```cpp
 363: Value createDimIndices(ImplicitLocOpBuilder &b, RankedTensorType tensorType,
 364:                        int dim) {
 365:   assert(dim >= 0 && dim < tensorType.getRank() && "invalid tensor dimension");
 366:   auto encoding = cast<ttg::DistributedEncodingTrait>(tensorType.getEncoding());
 367:   auto sliceEncoding = tti::getSingleDimSliceEncoding(encoding, dim);
 368:   auto indexType = RankedTensorType::get({tensorType.getShape()[dim]},
 369:                                          b.getI32Type(), sliceEncoding);
 370:   Value range = triton::MakeRangeOp::create(b, indexType, /*start=*/0,
 371:                                             /*end=*/tensorType.getShape()[dim]);
 372:   auto fullIndexType = cast<RankedTensorType>(
 373:       tensorType.cloneWith(std::nullopt, b.getI32Type()));
 374:   return convertAndBroadcast(b, range, {dim}, fullIndexType);
 375: }
```

- **EN:** Defines helper `createDimIndices` that computes or constructs intermediate data used by the surrounding transformation. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义辅助函数 `createDimIndices`，用于计算或构造外围变换所需的中间数据。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 377-381

```cpp
 377: Value createCurrentCTAMask(ImplicitLocOpBuilder &b) {
 378:   Value ctaId = tti::ExperimentalClusterCTAIdOp::create(b, b.getLoc());
 379:   return arith::ShLIOp::create(b, arith::ConstantIntOp::create(b, 1, 32),
 380:                                ctaId);
 381: }
```

- **EN:** Defines helper `createCurrentCTAMask` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createCurrentCTAMask`，用于计算或构造外围变换所需的中间数据。
### Lines 383-385

```cpp
 383: Value createCTASetMask(ImplicitLocOpBuilder &b, RankedTensorType tensorType,
 384:                        int dim, Value ctas) {
 385:   int numCTAs = ttg::lookupNumCTAs(b);
```

- **EN:** Defines helper `createCTASetMask` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createCTASetMask`，用于计算或构造外围变换所需的中间数据。
### Lines 387-399

```cpp
 387:   // Turn the scalar recipient bitset into a tensor mask over logical CTA rows:
 388:   // build a [0, numCTAs) row-index vector on `dim`, broadcast it to the state
 389:   // tensor shape, and test one bit of `recipientCTAs` per row.
 390:   auto loc = b.getLoc();
 391:   auto encoding = cast<ttg::DistributedEncodingTrait>(tensorType.getEncoding());
 392:   auto rowSliceEncoding = tti::getSingleDimSliceEncoding(encoding, dim);
 393:   auto rowType =
 394:       RankedTensorType::get({numCTAs}, b.getI32Type(), rowSliceEncoding);
 395:   Value rowIdx = triton::MakeRangeOp::create(b, rowType, /*start=*/0,
 396:                                              /*end=*/numCTAs);
 397:   auto indexType = cast<RankedTensorType>(
 398:       tensorType.cloneWith(std::nullopt, b.getI32Type()));
 399:   rowIdx = convertAndBroadcast(b, rowIdx, {dim}, indexType);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 401-407

```cpp
 401:   Value recipientBitsTensor = triton::SplatOp::create(b, indexType, ctas);
 402:   Value shifted = arith::ShRUIOp::create(b, recipientBitsTensor, rowIdx);
 403:   Value one = tti::createConstIntTensor(b, loc, 1, indexType);
 404:   Value selectedBit = arith::AndIOp::create(b, shifted, one);
 405:   Value zero = tti::createConstIntTensor(b, loc, 0, indexType);
 406:   return arith::CmpIOp::create(b, arith::CmpIPredicate::ne, selectedBit, zero);
 407: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 409-415

```cpp
 409: Value createLeadCTAEffectMask(ImplicitLocOpBuilder &b,
 410:                               RankedTensorType tensorType, Value effectCTAs) {
 411:   Value lhsMask = createCTASetMask(b, tensorType, /*dim=*/0, effectCTAs);
 412:   Value leadCTAMask =
 413:       createCTASetMask(b, tensorType, /*dim=*/2, createCurrentCTAMask(b));
 414:   return arith::AndIOp::create(b, lhsMask, leadCTAMask);
 415: }
```

- **EN:** Defines helper `createLeadCTAEffectMask` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createLeadCTAEffectMask`，用于计算或构造外围变换所需的中间数据。
### Lines 417-431

```cpp
 417: Operation *createMaskedStoreScratchMemory(ImplicitLocOpBuilder &b, Location loc,
 418:                                           Value alloc, Value tensor,
 419:                                           RankedTensorType tensorType,
 420:                                           Value mask) {
 421:   int64_t numCTAs = ttg::lookupNumCTAs(b);
 422:   if (numCTAs > 1) {
 423:     // This should hopefully be folded with the previous load in the caller
 424:     // function
 425:     Value oldTensor = tti::createLoadScratchMemory(b, loc, alloc, tensorType);
 426:     // and this with the previous selectOp, if there is any
 427:     tensor = arith::SelectOp::create(b, loc, mask, tensor, oldTensor);
 428:   }
 429:   return tti::createStoreScratchMemory(b, loc, alloc, tensor, tensorType,
 430:                                        /*currentCTAOnly=*/false);
 431: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 433-441

```cpp
 433: Operation *createCTAScopedStoreScratchMemory(ImplicitLocOpBuilder &b,
 434:                                              Location loc, Value alloc,
 435:                                              Value tensor,
 436:                                              RankedTensorType tensorType,
 437:                                              Value recipientCTAs) {
 438:   return createMaskedStoreScratchMemory(
 439:       b, loc, alloc, tensor, tensorType,
 440:       createCTASetMask(b, tensorType, /*dim=*/0, recipientCTAs));
 441: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 443-443

```cpp
 443: } // namespace
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 445-459

```cpp
 445: void FunctionBuilder::createFillGlobalTensorCall(ImplicitLocOpBuilder &b,
 446:                                                  Value ptr,
 447:                                                  RankedTensorType type,
 448:                                                  Value scalar) {
 449:   createCallToCachedFunction(
 450:       b, "fill_global_tensor", {ptr, scalar}, /*assertInfo=*/std::nullopt,
 451:       {type}, [type](ImplicitLocOpBuilder &fb, Block *entryBlock) {
 452:         Value ptr = entryBlock->getArgument(0);
 453:         Value scalar = entryBlock->getArgument(1);
 454:         Value tensor = triton::SplatOp::create(fb, type, scalar);
 455:         createStoreScratchMemory(fb, fb.getLoc(), ptr, tensor, type,
 456:                                  /*currentCTAOnly=*/false);
 457:         triton::ReturnOp::create(fb);
 458:       });
 459: }
```

- **EN:** Defines helper `FunctionBuilder::createFillGlobalTensorCall` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `FunctionBuilder::createFillGlobalTensorCall`，用于计算或构造外围变换所需的中间数据。
### Lines 461-463

```cpp
 461: void FunctionBuilder::createSetWaitingCall(ImplicitLocOpBuilder &b, Value mbar,
 462:                                            int thread, Value phase, Value pred,
 463:                                            Operation *insertPoint) {
```

- **EN:** Defines helper `FunctionBuilder::createSetWaitingCall` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `FunctionBuilder::createSetWaitingCall`，用于计算或构造外围变换所需的中间数据。
### Lines 465-482

```cpp
 465:   if (auxData.barriers.empty() || auxData.waiting.empty()) {
 466:     return;
 467:   }
 468:   if (!pred) {
 469:     pred = arith::ConstantIntOp::create(b, 1, 1);
 470:   }
 471:   Value threadVal = arith::ConstantIntOp::create(b, thread, 32);
 472:   Value barriersVal = auxData.barriers.at(insertPoint).value;
 473:   auto barriersType =
 474:       cast<RankedTensorType>(auxData.barriers.at(insertPoint).type);
 475:   Value waitingVal = auxData.waiting.at(insertPoint).value;
 476:   auto waitingType =
 477:       cast<RankedTensorType>(auxData.waiting.at(insertPoint).type);
 478:   uint32_t length = getMemDescLength(mbar);
 479:   Value mbarOffset = tti::ExperimentalMemDescToI32Op::create(b, mbar);
 480:   Value lengthVal = arith::ConstantIntOp::create(b, length, 32);
 481:   SmallVector<Value> args = {mbarOffset, lengthVal,   threadVal, phase,
 482:                              pred,       barriersVal, waitingVal};
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 483-491

```cpp
 483:   createCallToCachedFunction(
 484:       b, "set_waiting", args,
 485:       /*assertInfo=*/std::nullopt, {barriersType, waitingType},
 486:       [waitingType](ImplicitLocOpBuilder &fb, Block *entryBlock) {
 487:         Value mbarOffset = entryBlock->getArgument(0);
 488:         Value lengthVal = entryBlock->getArgument(1);
 489:         Value baseThread = entryBlock->getArgument(2);
 490:         Value phase = entryBlock->getArgument(3);
 491:         Value pred = entryBlock->getArgument(4);
```

- **EN:** Defines helper `createCallToCachedFunction` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createCallToCachedFunction`，用于计算或构造外围变换所需的中间数据。
### Lines 493-494

```cpp
 493:         Value barriers = entryBlock->getArgument(5);
 494:         Value waitingPtr = entryBlock->getArgument(6);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 496-497

```cpp
 496:         auto [prevBlock, ifBlock, thenBlock] = createIfBlock(fb, pred);
 497:         fb.setInsertionPointToStart(ifBlock);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 499-508

```cpp
 499:         Value waiting = tti::createLoadScratchMemory(fb, fb.getLoc(),
 500:                                                      waitingPtr, waitingType);
 501:         Value descriptor = createBufferDescriptor(fb, mbarOffset, lengthVal);
 502:         Value barriersEqBar =
 503:             createCmpIntTensorScalar(fb, barriers, descriptor);
 504:         barriersEqBar =
 505:             convertAndBroadcast(fb, barriersEqBar, {1}, waitingType);
 506:         Value ctaMask =
 507:             createLeadCTAEffectMask(fb, waitingType, createCurrentCTAMask(fb));
 508:         barriersEqBar = arith::AndIOp::create(fb, barriersEqBar, ctaMask);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 510-517

```cpp
 510:         Value bitsPerThread =
 511:             arith::ConstantIntOp::create(fb, WaitingBits::bitsPerThread, 32);
 512:         Value flagBit =
 513:             arith::ConstantIntOp::create(fb, WaitingBits::flagBit, 32);
 514:         Value phaseBit =
 515:             arith::ConstantIntOp::create(fb, WaitingBits::phaseBit, 32);
 516:         Value one = arith::ConstantIntOp::create(fb, 1, 32);
 517:         Value minusOne = arith::ConstantIntOp::create(fb, -1, 32);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 519-522

```cpp
 519:         Value baseTimesBits =
 520:             arith::MulIOp::create(fb, baseThread, bitsPerThread);
 521:         Value flagShift = arith::AddIOp::create(fb, baseTimesBits, flagBit);
 522:         Value phaseShift = arith::AddIOp::create(fb, baseTimesBits, phaseBit);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 524-529

```cpp
 524:         Value flagMaskScalar = arith::ShLIOp::create(fb, one, flagShift);
 525:         Value phaseMaskScalar = arith::ShLIOp::create(fb, one, phaseShift);
 526:         Value combinedMask =
 527:             arith::OrIOp::create(fb, flagMaskScalar, phaseMaskScalar);
 528:         Value clearMaskScalar =
 529:             arith::XOrIOp::create(fb, combinedMask, minusOne);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 531-536

```cpp
 531:         Value flagMaskTensor =
 532:             triton::SplatOp::create(fb, waitingType, flagMaskScalar);
 533:         Value clearMaskTensor =
 534:             triton::SplatOp::create(fb, waitingType, clearMaskScalar);
 535:         Value phaseShiftTensor =
 536:             triton::SplatOp::create(fb, waitingType, phaseShift);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 538-541

```cpp
 538:         Value clearedWaiting =
 539:             arith::AndIOp::create(fb, waiting, clearMaskTensor);
 540:         Value withFlag =
 541:             arith::OrIOp::create(fb, clearedWaiting, flagMaskTensor);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 543-548

```cpp
 543:         Value phaseScalar = arith::AndIOp::create(fb, phase, one);
 544:         Value phaseTensor =
 545:             triton::SplatOp::create(fb, waitingType, phaseScalar);
 546:         Value phaseBits =
 547:             arith::ShLIOp::create(fb, phaseTensor, phaseShiftTensor);
 548:         Value pendingWaiting = arith::OrIOp::create(fb, withFlag, phaseBits);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 550-552

```cpp
 550:         auto condType = cast<RankedTensorType>(barriersEqBar.getType());
 551:         Value predTensor = triton::SplatOp::create(fb, condType, pred);
 552:         Value cond = arith::AndIOp::create(fb, barriersEqBar, predTensor);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 554-557

```cpp
 554:         Value newWaiting =
 555:             arith::SelectOp::create(fb, cond, pendingWaiting, waiting);
 556:         createMaskedStoreScratchMemory(fb, fb.getLoc(), waitingPtr, newWaiting,
 557:                                        waitingType, ctaMask);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 559-562

```cpp
 559:         fb.setInsertionPointToEnd(thenBlock);
 560:         triton::ReturnOp::create(fb);
 561:       });
 562: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 564-573

```cpp
 564: void FunctionBuilder::createClearWaitingCall(ImplicitLocOpBuilder &b,
 565:                                              Value mbar, int thread, Value pred,
 566:                                              Operation *insertPoint) {
 567:   if (auxData.barriers.empty() || auxData.waiting.empty()) {
 568:     return;
 569:   }
 570:   if (!pred) {
 571:     pred = arith::ConstantIntOp::create(b, 1, 1);
 572:   }
 573:   Value threadVal = arith::ConstantIntOp::create(b, thread, 32);
```

- **EN:** Defines helper `FunctionBuilder::createClearWaitingCall` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `FunctionBuilder::createClearWaitingCall`，用于计算或构造外围变换所需的中间数据。
### Lines 575-580

```cpp
 575:   Value barriersVal = auxData.barriers.at(insertPoint).value;
 576:   auto barriersType =
 577:       cast<RankedTensorType>(auxData.barriers.at(insertPoint).type);
 578:   Value waitingVal = auxData.waiting.at(insertPoint).value;
 579:   auto waitingType =
 580:       cast<RankedTensorType>(auxData.waiting.at(insertPoint).type);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 582-594

```cpp
 582:   uint32_t length = getMemDescLength(mbar);
 583:   Value mbarOffset = tti::ExperimentalMemDescToI32Op::create(b, mbar);
 584:   Value lengthVal = arith::ConstantIntOp::create(b, length, 32);
 585:   SmallVector<Value> args = {mbarOffset, lengthVal,   threadVal,
 586:                              pred,       barriersVal, waitingVal};
 587:   createCallToCachedFunction(
 588:       b, "clear_waiting", args,
 589:       /*assertInfo=*/std::nullopt, {barriersType, waitingType},
 590:       [waitingType](ImplicitLocOpBuilder &fb, Block *entryBlock) {
 591:         Value mbarOffset = entryBlock->getArgument(0);
 592:         Value lengthVal = entryBlock->getArgument(1);
 593:         Value baseThread = entryBlock->getArgument(2);
 594:         Value pred = entryBlock->getArgument(3);
```

- **EN:** Defines helper `createCallToCachedFunction` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createCallToCachedFunction`，用于计算或构造外围变换所需的中间数据。
### Lines 596-597

```cpp
 596:         Value barriers = entryBlock->getArgument(4);
 597:         Value waitingPtr = entryBlock->getArgument(5);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 599-600

```cpp
 599:         auto [prevBlock, ifBlock, thenBlock] = createIfBlock(fb, pred);
 600:         fb.setInsertionPointToStart(ifBlock);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 602-611

```cpp
 602:         Value waiting = tti::createLoadScratchMemory(fb, fb.getLoc(),
 603:                                                      waitingPtr, waitingType);
 604:         Value descriptor = createBufferDescriptor(fb, mbarOffset, lengthVal);
 605:         Value barriersEqBar =
 606:             createCmpIntTensorScalar(fb, barriers, descriptor);
 607:         barriersEqBar =
 608:             convertAndBroadcast(fb, barriersEqBar, {1}, waitingType);
 609:         Value ctaMask =
 610:             createLeadCTAEffectMask(fb, waitingType, createCurrentCTAMask(fb));
 611:         barriersEqBar = arith::AndIOp::create(fb, barriersEqBar, ctaMask);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 613-620

```cpp
 613:         Value bitsPerThread =
 614:             arith::ConstantIntOp::create(fb, WaitingBits::bitsPerThread, 32);
 615:         Value flagBit =
 616:             arith::ConstantIntOp::create(fb, WaitingBits::flagBit, 32);
 617:         Value phaseBit =
 618:             arith::ConstantIntOp::create(fb, WaitingBits::phaseBit, 32);
 619:         Value one = arith::ConstantIntOp::create(fb, 1, 32);
 620:         Value minusOne = arith::ConstantIntOp::create(fb, -1, 32);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 622-625

```cpp
 622:         Value baseTimesBits =
 623:             arith::MulIOp::create(fb, baseThread, bitsPerThread);
 624:         Value flagShift = arith::AddIOp::create(fb, baseTimesBits, flagBit);
 625:         Value phaseShift = arith::AddIOp::create(fb, baseTimesBits, phaseBit);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 627-632

```cpp
 627:         Value flagMaskScalar = arith::ShLIOp::create(fb, one, flagShift);
 628:         Value phaseMaskScalar = arith::ShLIOp::create(fb, one, phaseShift);
 629:         Value combinedMask =
 630:             arith::OrIOp::create(fb, flagMaskScalar, phaseMaskScalar);
 631:         Value clearMaskScalar =
 632:             arith::XOrIOp::create(fb, combinedMask, minusOne);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 634-637

```cpp
 634:         Value clearMaskTensor =
 635:             triton::SplatOp::create(fb, waitingType, clearMaskScalar);
 636:         Value clearedWaiting =
 637:             arith::AndIOp::create(fb, waiting, clearMaskTensor);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 639-640

```cpp
 639:         Value newWaiting =
 640:             arith::SelectOp::create(fb, barriersEqBar, clearedWaiting, waiting);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 642-647

```cpp
 642:         createMaskedStoreScratchMemory(fb, fb.getLoc(), waitingPtr, newWaiting,
 643:                                        waitingType, ctaMask);
 644:         fb.setInsertionPointToEnd(thenBlock);
 645:         triton::ReturnOp::create(fb);
 646:       });
 647: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 649-666

```cpp
 649: void FunctionBuilder::createSetActiveMaskCall(ImplicitLocOpBuilder &b,
 650:                                               int activeMask,
 651:                                               Operation *insertPoint) {
 652:   if (auxData.activeMasks.empty())
 653:     return;
 654:   int64_t expandedActiveMask =
 655:       expandActiveMask(activeMask, auxData.threadLayout.numBaseThreads);
 656:   Value expandedActiveMaskVal =
 657:       arith::ConstantIntOp::create(b, expandedActiveMask, 32);
 658:   Value activeMasksVal = auxData.activeMasks.at(insertPoint).value;
 659:   auto activeMasksType =
 660:       cast<RankedTensorType>(auxData.activeMasks.at(insertPoint).type);
 661:   SmallVector<Value> args = {expandedActiveMaskVal, activeMasksVal};
 662:   createCallToCachedFunction(
 663:       b, "set_active_mask", args,
 664:       /*assertInfo=*/std::nullopt, {activeMasksType},
 665:       [activeMasksType](ImplicitLocOpBuilder &fb, Block *entryBlock) {
 666:         Value expandedActiveMaskVal = entryBlock->getArgument(0);
```

- **EN:** Defines helper `FunctionBuilder::createSetActiveMaskCall` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `FunctionBuilder::createSetActiveMaskCall`，用于计算或构造外围变换所需的中间数据。
### Lines 667-667

```cpp
 667:         Value activeMasksPtr = entryBlock->getArgument(1);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 669-676

```cpp
 669:         Value newActiveMasks =
 670:             triton::SplatOp::create(fb, activeMasksType, expandedActiveMaskVal);
 671:         tti::createStoreScratchMemory(fb, fb.getLoc(), activeMasksPtr,
 672:                                       newActiveMasks, activeMasksType,
 673:                                       /*currentCTAOnly=*/true);
 674:         triton::ReturnOp::create(fb);
 675:       });
 676: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 678-695

```cpp
 678: void FunctionBuilder::createRetireActiveThreadCall(ImplicitLocOpBuilder &b,
 679:                                                    int thread,
 680:                                                    Operation *insertPoint) {
 681:   if (auxData.activeMasks.empty())
 682:     return;
 683:   int64_t threadMask =
 684:       expandActiveMask(1u << thread, auxData.threadLayout.numBaseThreads);
 685:   Value clearMaskVal = arith::ConstantIntOp::create(b, ~threadMask, 32);
 686:   Value activeMasksVal = auxData.activeMasks.at(insertPoint).value;
 687:   auto activeMasksType =
 688:       cast<RankedTensorType>(auxData.activeMasks.at(insertPoint).type);
 689:   SmallVector<Value> args = {clearMaskVal, activeMasksVal};
 690:   createCallToCachedFunction(
 691:       b, "retire_active_thread", args,
 692:       /*assertInfo=*/std::nullopt, {activeMasksType},
 693:       [activeMasksType](ImplicitLocOpBuilder &fb, Block *entryBlock) {
 694:         Value clearMaskVal = entryBlock->getArgument(0);
 695:         Value activeMasksPtr = entryBlock->getArgument(1);
```

- **EN:** Defines helper `FunctionBuilder::createRetireActiveThreadCall` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `FunctionBuilder::createRetireActiveThreadCall`，用于计算或构造外围变换所需的中间数据。
### Lines 697-711

```cpp
 697:         Value activeMasks = tti::createLoadScratchMemory(
 698:             fb, fb.getLoc(), activeMasksPtr, activeMasksType);
 699:         Value clearMask =
 700:             triton::SplatOp::create(fb, activeMasksType, clearMaskVal);
 701:         Value retiredMasks = arith::AndIOp::create(fb, activeMasks, clearMask);
 702:         Value oneMask =
 703:             tti::createConstIntTensor(fb, fb.getLoc(), 1, activeMasksType);
 704:         Value newActiveMasks =
 705:             arith::MaxUIOp::create(fb, retiredMasks, oneMask);
 706:         tti::createStoreScratchMemory(fb, fb.getLoc(), activeMasksPtr,
 707:                                       newActiveMasks, activeMasksType,
 708:                                       /*currentCTAOnly=*/true);
 709:         triton::ReturnOp::create(fb);
 710:       });
 711: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 713-729

```cpp
 713: void FunctionBuilder::createCheckAllActiveWaitingCall(ImplicitLocOpBuilder &b,
 714:                                                       Value pred,
 715:                                                       Operation *insertPoint) {
 716:   if (auxData.waiting.empty() || auxData.barrierStates.empty()) {
 717:     return;
 718:   }
 719:   if (!pred) {
 720:     pred = arith::ConstantIntOp::create(b, 1, 1);
 721:   }
 722:   uint32_t flagMask = WaitingBits::makeInterleavedMask(
 723:       WaitingBits::flagBit, auxData.threadLayout.numBaseThreads);
 724:   uint32_t phaseMask = WaitingBits::makeInterleavedMask(
 725:       WaitingBits::phaseBit, auxData.threadLayout.numBaseThreads);
 726:   Value waitingVal = auxData.waiting.at(insertPoint).value;
 727:   auto waitingType =
 728:       cast<RankedTensorType>(auxData.waiting.at(insertPoint).type);
 729:   Value barrierStatesVal = auxData.barrierStates.at(insertPoint).value;
```

- **EN:** Defines helper `FunctionBuilder::createCheckAllActiveWaitingCall` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `FunctionBuilder::createCheckAllActiveWaitingCall`，用于计算或构造外围变换所需的中间数据。
### Lines 730-746

```cpp
 730:   auto barrierStatesType =
 731:       cast<RankedTensorType>(auxData.barrierStates.at(insertPoint).type);
 732:   Region *region = b.getInsertionPoint()->getParentRegion();
 733:   auto waitingGlobalType = tti::getIntTensorType(
 734:       region, waitingType.getShape(),
 735:       waitingType.getElementType().getIntOrFloatBitWidth());
 736:   auto barrierStatesGlobalType = tti::getIntTensorType(
 737:       region, barrierStatesType.getShape(),
 738:       barrierStatesType.getElementType().getIntOrFloatBitWidth());
 739:   Value activeMasksVal = auxData.activeMasks.at(insertPoint).value;
 740:   auto activeMasksType =
 741:       cast<RankedTensorType>(auxData.activeMasks.at(insertPoint).type);
 742:   auto activeMasksGlobalType = tti::getIntTensorType(
 743:       region, activeMasksType.getShape(),
 744:       activeMasksType.getElementType().getIntOrFloatBitWidth());
 745:   SmallVector<Value> args = {pred, waitingVal, barrierStatesVal,
 746:                              activeMasksVal};
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 747-755

```cpp
 747:   AssertInfo assertInfo{
 748:       "Deadlock detected: all unfinished threads are waiting on mbarriers",
 749:       b.getI1Type()};
 750:   createCallToCachedFunction(
 751:       b, "check_all_active_waiting", args, assertInfo,
 752:       {waitingGlobalType, barrierStatesGlobalType, activeMasksGlobalType},
 753:       [waitingGlobalType, barrierStatesGlobalType, activeMasksGlobalType,
 754:        flagMask, phaseMask](ImplicitLocOpBuilder &fb, Block *entryBlock) {
 755:         Value pred = entryBlock->getArgument(0);
```

- **EN:** Defines helper `createCallToCachedFunction` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createCallToCachedFunction`，用于计算或构造外围变换所需的中间数据。
### Lines 757-759

```cpp
 757:         Value waitingPtr = entryBlock->getArgument(1);
 758:         Value barrierStatesPtr = entryBlock->getArgument(2);
 759:         Value activeMasksPtr = entryBlock->getArgument(3);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 761-764

```cpp
 761:         Value waiting = tti::createLoadScratchMemory(
 762:             fb, fb.getLoc(), waitingPtr, waitingGlobalType);
 763:         Value barrierStates = tti::createLoadScratchMemory(
 764:             fb, fb.getLoc(), barrierStatesPtr, barrierStatesGlobalType);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 766-769

```cpp
 766:         Value flagMaskTensor = tti::createConstIntTensor(
 767:             fb, fb.getLoc(), flagMask, waitingGlobalType);
 768:         Value phaseMaskTensor = tti::createConstIntTensor(
 769:             fb, fb.getLoc(), phaseMask, waitingGlobalType);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 771-776

```cpp
 771:         Value flags = arith::AndIOp::create(fb, waiting, flagMaskTensor);
 772:         Value phases = arith::AndIOp::create(fb, waiting, phaseMaskTensor);
 773:         Value shiftOneTensor =
 774:             tti::createConstIntTensor(fb, fb.getLoc(), 1, waitingGlobalType);
 775:         Value phasesAligned =
 776:             arith::ShRUIOp::create(fb, phases, shiftOneTensor);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 778-782

```cpp
 778:         Value phasesComplement =
 779:             arith::XOrIOp::create(fb, phasesAligned, flagMaskTensor);
 780:         Value waitingPhase0 =
 781:             arith::AndIOp::create(fb, flags, phasesComplement);
 782:         Value waitingPhase1 = arith::AndIOp::create(fb, flags, phasesAligned);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 784-788

```cpp
 784:         Value oneState = tti::createConstIntTensor(fb, fb.getLoc(), 1,
 785:                                                    barrierStatesGlobalType);
 786:         Value barrierPhase = arith::AndIOp::create(fb, barrierStates, oneState);
 787:         Value phaseIsOne = arith::CmpIOp::create(fb, arith::CmpIPredicate::eq,
 788:                                                  barrierPhase, oneState);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 790-806

```cpp
 790:         phaseIsOne =
 791:             convertAndBroadcast(fb, phaseIsOne, {0, 1}, waitingGlobalType);
 792:         Value effectiveWaiting = arith::SelectOp::create(
 793:             fb, phaseIsOne, waitingPhase1, waitingPhase0);
 794:         Value waitingOr = reduce<arith::OrIOp>(fb, effectiveWaiting, {0, 1});
 795:         auto waitingOrType = cast<RankedTensorType>(waitingOr.getType());
 796:         Value activeMasks = tti::createLoadScratchMemory(
 797:             fb, fb.getLoc(), activeMasksPtr, activeMasksGlobalType);
 798:         Value activeMaskTensor =
 799:             createConvertLayout(fb, activeMasks, waitingOrType.getEncoding());
 800:         Value waitingMasked =
 801:             arith::AndIOp::create(fb, waitingOr, activeMaskTensor);
 802:         Value eqPerCTA = arith::CmpIOp::create(fb, arith::CmpIPredicate::eq,
 803:                                                waitingMasked, activeMaskTensor);
 804:         Value allFinishedOrWaiting = reduceAll<arith::AndIOp>(fb, eqPerCTA);
 805:         Value zeroMask = tti::createConstIntTensor(fb, fb.getLoc(), 0,
 806:                                                    activeMasksGlobalType);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 807-811

```cpp
 807:         Value activePerCTA = arith::CmpIOp::create(fb, arith::CmpIPredicate::ne,
 808:                                                    activeMasks, zeroMask);
 809:         Value anyUnfinished = reduceAll<arith::OrIOp>(fb, activePerCTA);
 810:         Value deadlocked =
 811:             arith::AndIOp::create(fb, allFinishedOrWaiting, anyUnfinished);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 813-819

```cpp
 813:         Value vTrue = arith::ConstantOp::create(
 814:             fb, deadlocked.getType(), fb.getIntegerAttr(fb.getI1Type(), 1));
 815:         Value ok = arith::XOrIOp::create(fb, deadlocked, vTrue);
 816:         Value predicatedOk = arith::SelectOp::create(fb, pred, ok, vTrue);
 817:         triton::ReturnOp::create(fb, predicatedOk);
 818:       });
 819: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 821-838

```cpp
 821: void FunctionBuilder::createVerifyBarrierCanInitCall(ImplicitLocOpBuilder &b,
 822:                                                      Value mbar, Value pred,
 823:                                                      Operation *insertPoint,
 824:                                                      Value recipientCTAs) {
 825:   assert(!auxData.barriers.empty() &&
 826:          "barrier descriptors must exist when verifying barrier init");
 827:   assert(!auxData.barrierStates.empty() &&
 828:          "barrier states must exist when verifying barrier init");
 829:   if (!pred) {
 830:     pred = arith::ConstantIntOp::create(b, 1, 1);
 831:   }
 832:   Value barriersVal = auxData.barriers.at(insertPoint).value;
 833:   auto barriersType =
 834:       cast<RankedTensorType>(auxData.barriers.at(insertPoint).type);
 835:   Value barrierStatesVal = auxData.barrierStates.at(insertPoint).value;
 836:   auto barrierStatesType =
 837:       cast<RankedTensorType>(auxData.barrierStates.at(insertPoint).type);
 838:   uint32_t length = getMemDescLength(mbar);
```

- **EN:** Defines helper `FunctionBuilder::createVerifyBarrierCanInitCall` that computes or constructs intermediate data used by the surrounding transformation. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义辅助函数 `FunctionBuilder::createVerifyBarrierCanInitCall`，用于计算或构造外围变换所需的中间数据。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 839-854

```cpp
 839:   Value mbarOffset = tti::ExperimentalMemDescToI32Op::create(b, mbar);
 840:   Value lengthVal = arith::ConstantIntOp::create(b, length, 32);
 841:   SmallVector<Value> args = {mbarOffset,  lengthVal,        pred,
 842:                              barriersVal, barrierStatesVal, recipientCTAs};
 843:   AssertInfo assertInfo{"Barrier re-initialized without prior invalidation",
 844:                         b.getI1Type()};
 845:   createCallToCachedFunction(
 846:       b, "verify_barrier_can_init", args, assertInfo,
 847:       {barriersType, barrierStatesType},
 848:       [barrierStatesType](ImplicitLocOpBuilder &fb, Block *entryBlock) {
 849:         Value mbarOffset = entryBlock->getArgument(0);
 850:         Value lengthVal = entryBlock->getArgument(1);
 851:         Value pred = entryBlock->getArgument(2);
 852:         Value barriers = entryBlock->getArgument(3);
 853:         Value statesPtr = entryBlock->getArgument(4);
 854:         Value recipientCTAs = entryBlock->getArgument(5);
```

- **EN:** Defines helper `createCallToCachedFunction` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createCallToCachedFunction`，用于计算或构造外围变换所需的中间数据。
### Lines 856-873

```cpp
 856:         Value states = tti::createLoadScratchMemory(fb, fb.getLoc(), statesPtr,
 857:                                                     barrierStatesType);
 858:         Value descriptor = createBufferDescriptor(fb, mbarOffset, lengthVal);
 859:         Value mask = createCmpIntTensorScalar(fb, barriers, descriptor);
 860:         mask = convertAndBroadcast(fb, mask, {1}, barrierStatesType);
 861:         Value zero =
 862:             tti::createConstIntTensor(fb, fb.getLoc(), 0, barrierStatesType);
 863:         Value canInit =
 864:             arith::CmpIOp::create(fb, arith::CmpIPredicate::eq, states, zero);
 865:         auto condType = cast<RankedTensorType>(canInit.getType());
 866:         Value vTrue = tti::createConstIntTensor(fb, fb.getLoc(), 1, condType);
 867:         canInit = arith::SelectOp::create(fb, mask, canInit, vTrue);
 868:         Value ctaMask =
 869:             createCTASetMask(fb, condType, /*dim=*/0, recipientCTAs);
 870:         canInit = arith::SelectOp::create(fb, ctaMask, canInit, vTrue);
 871:         Value predTensor = triton::SplatOp::create(fb, condType, pred);
 872:         canInit = arith::SelectOp::create(fb, predTensor, canInit, vTrue);
 873:         triton::ReturnOp::create(fb, reduceAll<arith::AndIOp>(fb, canInit));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 874-875

```cpp
 874:       });
 875: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 877-894

```cpp
 877: void FunctionBuilder::createVerifyBarrierInitializedCall(
 878:     ImplicitLocOpBuilder &b, Value mbar, Value pred, Operation *insertPoint,
 879:     Value recipientCTAs) {
 880:   assert(!auxData.barriers.empty() &&
 881:          "barrier descriptors must exist when verifying barrier use");
 882:   assert(!auxData.barrierStates.empty() &&
 883:          "barrier states must exist when verifying barrier use");
 884:   if (!pred) {
 885:     pred = arith::ConstantIntOp::create(b, 1, 1);
 886:   }
 887:   Value barriersVal = auxData.barriers.at(insertPoint).value;
 888:   auto barriersType =
 889:       cast<RankedTensorType>(auxData.barriers.at(insertPoint).type);
 890:   Value barrierStatesVal = auxData.barrierStates.at(insertPoint).value;
 891:   auto barrierStatesType =
 892:       cast<RankedTensorType>(auxData.barrierStates.at(insertPoint).type);
 893:   uint32_t length = getMemDescLength(mbar);
 894:   Value mbarOffset = tti::ExperimentalMemDescToI32Op::create(b, mbar);
```

- **EN:** Defines helper `FunctionBuilder::createVerifyBarrierInitializedCall` that computes or constructs intermediate data used by the surrounding transformation. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义辅助函数 `FunctionBuilder::createVerifyBarrierInitializedCall`，用于计算或构造外围变换所需的中间数据。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 895-910

```cpp
 895:   Value lengthVal = arith::ConstantIntOp::create(b, length, 32);
 896:   SmallVector<Value> args = {mbarOffset,  lengthVal,        pred,
 897:                              barriersVal, barrierStatesVal, recipientCTAs};
 898:   AssertInfo assertInfo{
 899:       "Barrier used before initialization or after invalidation",
 900:       b.getI1Type()};
 901:   createCallToCachedFunction(
 902:       b, "verify_barrier_initialized", args, assertInfo,
 903:       {barriersType, barrierStatesType},
 904:       [barrierStatesType](ImplicitLocOpBuilder &fb, Block *entryBlock) {
 905:         Value mbarOffset = entryBlock->getArgument(0);
 906:         Value lengthVal = entryBlock->getArgument(1);
 907:         Value pred = entryBlock->getArgument(2);
 908:         Value barriers = entryBlock->getArgument(3);
 909:         Value statesPtr = entryBlock->getArgument(4);
 910:         Value recipientCTAs = entryBlock->getArgument(5);
```

- **EN:** Defines helper `createCallToCachedFunction` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createCallToCachedFunction`，用于计算或构造外围变换所需的中间数据。
### Lines 912-929

```cpp
 912:         Value states = tti::createLoadScratchMemory(fb, fb.getLoc(), statesPtr,
 913:                                                     barrierStatesType);
 914:         Value descriptor = createBufferDescriptor(fb, mbarOffset, lengthVal);
 915:         Value mask = createCmpIntTensorScalar(fb, barriers, descriptor);
 916:         mask = convertAndBroadcast(fb, mask, {1}, barrierStatesType);
 917:         Value zero =
 918:             tti::createConstIntTensor(fb, fb.getLoc(), 0, barrierStatesType);
 919:         Value initialized =
 920:             arith::CmpIOp::create(fb, arith::CmpIPredicate::ne, states, zero);
 921:         auto condType = cast<RankedTensorType>(initialized.getType());
 922:         Value vTrue = tti::createConstIntTensor(fb, fb.getLoc(), 1, condType);
 923:         initialized = arith::SelectOp::create(fb, mask, initialized, vTrue);
 924:         Value ctaMask =
 925:             createCTASetMask(fb, condType, /*dim=*/0, recipientCTAs);
 926:         initialized = arith::SelectOp::create(fb, ctaMask, initialized, vTrue);
 927:         Value predTensor = triton::SplatOp::create(fb, condType, pred);
 928:         Value predicatedInitialized =
 929:             arith::SelectOp::create(fb, predTensor, initialized, vTrue);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 930-933

```cpp
 930:         triton::ReturnOp::create(
 931:             fb, reduceAll<arith::AndIOp>(fb, predicatedInitialized));
 932:       });
 933: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 935-940

```cpp
 935: void FunctionBuilder::createInitBarrierStateCall(ImplicitLocOpBuilder &b,
 936:                                                  Value mbar, int count,
 937:                                                  Value pred,
 938:                                                  Operation *insertPoint) {
 939:   assert(count >= 0 && (uint64_t)count <= BarrierBits::countMask &&
 940:          "barrier init count exceeds barrier state capacity");
```

- **EN:** Defines helper `FunctionBuilder::createInitBarrierStateCall` that computes or constructs intermediate data used by the surrounding transformation. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义辅助函数 `FunctionBuilder::createInitBarrierStateCall`，用于计算或构造外围变换所需的中间数据。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 942-959

```cpp
 942:   if (auxData.barriers.empty() || auxData.barrierStates.empty()) {
 943:     return;
 944:   }
 945:   if (!pred) {
 946:     pred = arith::ConstantIntOp::create(b, 1, 1);
 947:   }
 948:   Value countVal = arith::ConstantIntOp::create(b, count, 32);
 949:   Value barriersVal = auxData.barriers.at(insertPoint).value;
 950:   auto barriersType =
 951:       cast<RankedTensorType>(auxData.barriers.at(insertPoint).type);
 952:   Value barrierStatesVal = auxData.barrierStates.at(insertPoint).value;
 953:   auto barrierStatesType =
 954:       cast<RankedTensorType>(auxData.barrierStates.at(insertPoint).type);
 955:   uint32_t length = getMemDescLength(mbar);
 956:   Value mbarOffset = tti::ExperimentalMemDescToI32Op::create(b, mbar);
 957:   Value lengthVal = arith::ConstantIntOp::create(b, length, 32);
 958:   SmallVector<Value> args = {mbarOffset, lengthVal,   countVal,
 959:                              pred,       barriersVal, barrierStatesVal};
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 960-967

```cpp
 960:   createCallToCachedFunction(
 961:       b, "init_barrier_state", args,
 962:       /*assertInfo=*/std::nullopt, {barriersType, barrierStatesType},
 963:       [barrierStatesType](ImplicitLocOpBuilder &fb, Block *entryBlock) {
 964:         Value mbarOffset = entryBlock->getArgument(0);
 965:         Value lengthVal = entryBlock->getArgument(1);
 966:         Value count = entryBlock->getArgument(2);
 967:         Value pred = entryBlock->getArgument(3);
```

- **EN:** Defines helper `createCallToCachedFunction` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createCallToCachedFunction`，用于计算或构造外围变换所需的中间数据。
### Lines 969-970

```cpp
 969:         Value barriers = entryBlock->getArgument(4);
 970:         Value statesPtr = entryBlock->getArgument(5);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 972-976

```cpp
 972:         Value states = tti::createLoadScratchMemory(fb, fb.getLoc(), statesPtr,
 973:                                                     barrierStatesType);
 974:         Value descriptor = createBufferDescriptor(fb, mbarOffset, lengthVal);
 975:         Value mask = createCmpIntTensorScalar(fb, barriers, descriptor);
 976:         mask = convertAndBroadcast(fb, mask, {1}, barrierStatesType);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 978-984

```cpp
 978:         Value countWide = adjustIntegerWidth(
 979:             fb, count, cast<IntegerType>(barrierStatesType.getElementType()));
 980:         Value countMask =
 981:             arith::ConstantIntOp::create(fb, BarrierBits::countMask, 64);
 982:         Value maskedCount = arith::AndIOp::create(fb, countWide, countMask);
 983:         Value countTensor =
 984:             triton::SplatOp::create(fb, barrierStatesType, maskedCount);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 986-989

```cpp
 986:         Value shiftInitTensor = tti::createConstIntTensor(
 987:             fb, fb.getLoc(), BarrierBits::initCountLsb, barrierStatesType);
 988:         Value shiftCurrentTensor = tti::createConstIntTensor(
 989:             fb, fb.getLoc(), BarrierBits::currentCountLsb, barrierStatesType);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 991-995

```cpp
 991:         Value initField =
 992:             arith::ShLIOp::create(fb, countTensor, shiftInitTensor);
 993:         Value currentField =
 994:             arith::ShLIOp::create(fb, countTensor, shiftCurrentTensor);
 995:         Value newState = arith::OrIOp::create(fb, initField, currentField);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 997-1006

```cpp
 997:         Value updated = arith::SelectOp::create(fb, mask, newState, states);
 998:         auto condType = cast<RankedTensorType>(mask.getType());
 999:         Value predTensor = triton::SplatOp::create(fb, condType, pred);
1000:         updated = arith::SelectOp::create(fb, predTensor, updated, states);
1001:         createCTAScopedStoreScratchMemory(fb, fb.getLoc(), statesPtr, updated,
1002:                                           barrierStatesType,
1003:                                           createCurrentCTAMask(fb));
1004:         triton::ReturnOp::create(fb);
1005:       });
1006: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1008-1025

```cpp
1008: void FunctionBuilder::createInvalidateBarrierStateCall(ImplicitLocOpBuilder &b,
1009:                                                        Value mbar, Value pred,
1010:                                                        Operation *insertPoint) {
1011:   assert(!auxData.barriers.empty() &&
1012:          "barrier descriptors must exist when invalidating a barrier");
1013:   assert(!auxData.barrierStates.empty() &&
1014:          "barrier states must exist when invalidating a barrier");
1015:   assert(!auxData.waiting.empty() &&
1016:          "waiting state must exist when invalidating a barrier");
1017:   if (!pred) {
1018:     pred = arith::ConstantIntOp::create(b, 1, 1);
1019:   }
1020:   Value barriersVal = auxData.barriers.at(insertPoint).value;
1021:   auto barriersType =
1022:       cast<RankedTensorType>(auxData.barriers.at(insertPoint).type);
1023:   Value barrierStatesVal = auxData.barrierStates.at(insertPoint).value;
1024:   auto barrierStatesType =
1025:       cast<RankedTensorType>(auxData.barrierStates.at(insertPoint).type);
```

- **EN:** Defines helper `FunctionBuilder::createInvalidateBarrierStateCall` that computes or constructs intermediate data used by the surrounding transformation. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义辅助函数 `FunctionBuilder::createInvalidateBarrierStateCall`，用于计算或构造外围变换所需的中间数据。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 1026-1043

```cpp
1026:   Value waitingVal = auxData.waiting.at(insertPoint).value;
1027:   auto waitingType =
1028:       cast<RankedTensorType>(auxData.waiting.at(insertPoint).type);
1029:   uint32_t length = getMemDescLength(mbar);
1030:   Value mbarOffset = tti::ExperimentalMemDescToI32Op::create(b, mbar);
1031:   Value lengthVal = arith::ConstantIntOp::create(b, length, 32);
1032:   SmallVector<Value> args = {mbarOffset,  lengthVal,        pred,
1033:                              barriersVal, barrierStatesVal, waitingVal};
1034:   createCallToCachedFunction(
1035:       b, "invalidate_barrier_state", args,
1036:       /*assertInfo=*/std::nullopt,
1037:       {barriersType, barrierStatesType, waitingType},
1038:       [barrierStatesType, waitingType](ImplicitLocOpBuilder &fb,
1039:                                        Block *entryBlock) {
1040:         Value mbarOffset = entryBlock->getArgument(0);
1041:         Value lengthVal = entryBlock->getArgument(1);
1042:         Value pred = entryBlock->getArgument(2);
1043:         Value barriers = entryBlock->getArgument(3);
```

- **EN:** Defines helper `createCallToCachedFunction` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createCallToCachedFunction`，用于计算或构造外围变换所需的中间数据。
### Lines 1044-1045

```cpp
1044:         Value statesPtr = entryBlock->getArgument(4);
1045:         Value waitingPtr = entryBlock->getArgument(5);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1047-1053

```cpp
1047:         Value states = tti::createLoadScratchMemory(fb, fb.getLoc(), statesPtr,
1048:                                                     barrierStatesType);
1049:         Value waiting = tti::createLoadScratchMemory(fb, fb.getLoc(),
1050:                                                      waitingPtr, waitingType);
1051:         Value descriptor = createBufferDescriptor(fb, mbarOffset, lengthVal);
1052:         Value mask = createCmpIntTensorScalar(fb, barriers, descriptor);
1053:         mask = convertAndBroadcast(fb, mask, {1}, barrierStatesType);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1055-1072

```cpp
1055:         Value zeroState =
1056:             tti::createConstIntTensor(fb, fb.getLoc(), 0, barrierStatesType);
1057:         Value zeroWaiting =
1058:             tti::createConstIntTensor(fb, fb.getLoc(), 0, waitingType);
1059:         Value updatedStates =
1060:             arith::SelectOp::create(fb, mask, zeroState, states);
1061:         auto stateCondType = cast<RankedTensorType>(mask.getType());
1062:         Value statePredTensor =
1063:             triton::SplatOp::create(fb, stateCondType, pred);
1064:         updatedStates =
1065:             arith::SelectOp::create(fb, statePredTensor, updatedStates, states);
1066:         Value waitingMask = convertAndBroadcast(fb, mask, {0, 1}, waitingType);
1067:         Value waitingCTAMask =
1068:             createLeadCTAEffectMask(fb, waitingType, createCurrentCTAMask(fb));
1069:         waitingMask = arith::AndIOp::create(fb, waitingMask, waitingCTAMask);
1070:         Value updatedWaiting =
1071:             arith::SelectOp::create(fb, waitingMask, zeroWaiting, waiting);
1072:         auto waitingCondType = cast<RankedTensorType>(waitingMask.getType());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1073-1085

```cpp
1073:         Value waitingPredTensor =
1074:             triton::SplatOp::create(fb, waitingCondType, pred);
1075:         updatedWaiting = arith::SelectOp::create(fb, waitingPredTensor,
1076:                                                  updatedWaiting, waiting);
1077:         createCTAScopedStoreScratchMemory(fb, fb.getLoc(), statesPtr,
1078:                                           updatedStates, barrierStatesType,
1079:                                           createCurrentCTAMask(fb));
1080:         createMaskedStoreScratchMemory(fb, fb.getLoc(), waitingPtr,
1081:                                        updatedWaiting, waitingType,
1082:                                        waitingCTAMask);
1083:         triton::ReturnOp::create(fb);
1084:       });
1085: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1087-1094

```cpp
1087: void FunctionBuilder::createVerifyBarrierArriveCall(
1088:     ImplicitLocOpBuilder &b, Value mbar, int count, Value pred,
1089:     Operation *insertPoint, Value recipientCTAs, int txCount) {
1090:   assert(count >= 0 && (uint64_t)count <= BarrierBits::countMask &&
1091:          "barrier arrive count exceeds barrier state capacity");
1092:   assert(txCount >= BarrierBits::txCountMin &&
1093:          txCount <= BarrierBits::txCountMax &&
1094:          "barrier tx-count delta exceeds barrier state capacity");
```

- **EN:** Defines helper `FunctionBuilder::createVerifyBarrierArriveCall` that computes or constructs intermediate data used by the surrounding transformation. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义辅助函数 `FunctionBuilder::createVerifyBarrierArriveCall`，用于计算或构造外围变换所需的中间数据。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 1096-1112

```cpp
1096:   if (auxData.barriers.empty() || auxData.barrierStates.empty()) {
1097:     return;
1098:   }
1099:   if (!pred) {
1100:     pred = arith::ConstantIntOp::create(b, 1, 1);
1101:   }
1102:   Value countVal = arith::ConstantIntOp::create(b, count, 32);
1103:   Value txCountVal = arith::ConstantIntOp::create(b, txCount, 64);
1104:   Value barriersVal = auxData.barriers.at(insertPoint).value;
1105:   auto barriersType =
1106:       cast<RankedTensorType>(auxData.barriers.at(insertPoint).type);
1107:   Value barrierStatesVal = auxData.barrierStates.at(insertPoint).value;
1108:   auto barrierStatesType =
1109:       cast<RankedTensorType>(auxData.barrierStates.at(insertPoint).type);
1110:   uint32_t length = getMemDescLength(mbar);
1111:   Value mbarOffset = tti::ExperimentalMemDescToI32Op::create(b, mbar);
1112:   Value lengthVal = arith::ConstantIntOp::create(b, length, 32);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1113-1128

```cpp
1113:   SmallVector<Value> args = {mbarOffset,       lengthVal,    countVal,
1114:                              txCountVal,       pred,         barriersVal,
1115:                              barrierStatesVal, recipientCTAs};
1116:   AssertInfo assertInfo{
1117:       "Barrier arrive underflow: current count or tx-count would become "
1118:       "invalid",
1119:       b.getI1Type()};
1120:   createCallToCachedFunction(
1121:       b, "verify_barrier_arrive", args, assertInfo,
1122:       {barriersType, barrierStatesType},
1123:       [barrierStatesType](ImplicitLocOpBuilder &fb, Block *entryBlock) {
1124:         Value mbarOffset = entryBlock->getArgument(0);
1125:         Value lengthVal = entryBlock->getArgument(1);
1126:         Value count = entryBlock->getArgument(2);
1127:         Value txCount = entryBlock->getArgument(3);
1128:         Value pred = entryBlock->getArgument(4);
```

- **EN:** Defines helper `createCallToCachedFunction` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createCallToCachedFunction`，用于计算或构造外围变换所需的中间数据。
### Lines 1130-1132

```cpp
1130:         Value barriers = entryBlock->getArgument(5);
1131:         Value statesPtr = entryBlock->getArgument(6);
1132:         Value recipientCTAs = entryBlock->getArgument(7);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1134-1138

```cpp
1134:         Value states = tti::createLoadScratchMemory(fb, fb.getLoc(), statesPtr,
1135:                                                     barrierStatesType);
1136:         Value descriptor = createBufferDescriptor(fb, mbarOffset, lengthVal);
1137:         Value mask = createCmpIntTensorScalar(fb, barriers, descriptor);
1138:         mask = convertAndBroadcast(fb, mask, {1}, barrierStatesType);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1140-1150

```cpp
1140:         Value zero32 =
1141:             tti::createConstIntTensor(fb, fb.getLoc(), 0, barrierStatesType);
1142:         Value maskFF = tti::createConstIntTensor(
1143:             fb, fb.getLoc(), BarrierBits::countMask, barrierStatesType);
1144:         Value shiftCurrentTensor = tti::createConstIntTensor(
1145:             fb, fb.getLoc(), BarrierBits::currentCountLsb, barrierStatesType);
1146:         Value shiftTxTensor = tti::createConstIntTensor(
1147:             fb, fb.getLoc(), BarrierBits::txCountLsb, barrierStatesType);
1148:         Value shiftTxSignTensor = tti::createConstIntTensor(
1149:             fb, fb.getLoc(), 64 - BarrierBits::txCountBitWidth,
1150:             barrierStatesType);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1152-1160

```cpp
1152:         Value currentCount =
1153:             arith::ShRUIOp::create(fb, states, shiftCurrentTensor);
1154:         currentCount = arith::AndIOp::create(fb, currentCount, maskFF);
1155:         Value currentTxCount =
1156:             arith::ShRUIOp::create(fb, states, shiftTxTensor);
1157:         currentTxCount =
1158:             arith::ShLIOp::create(fb, currentTxCount, shiftTxSignTensor);
1159:         currentTxCount =
1160:             arith::ShRSIOp::create(fb, currentTxCount, shiftTxSignTensor);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1162-1170

```cpp
1162:         Value countMask =
1163:             arith::ConstantIntOp::create(fb, BarrierBits::countMask, 64);
1164:         Value countWide = adjustIntegerWidth(
1165:             fb, count, cast<IntegerType>(barrierStatesType.getElementType()));
1166:         Value maskedCount = arith::AndIOp::create(fb, countWide, countMask);
1167:         Value arriveCount =
1168:             triton::SplatOp::create(fb, barrierStatesType, maskedCount);
1169:         Value txCountTensor =
1170:             triton::SplatOp::create(fb, barrierStatesType, txCount);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1172-1185

```cpp
1172:         Value newCurrent = arith::SubIOp::create(fb, currentCount, arriveCount);
1173:         Value newCurrentMasked =
1174:             arith::SelectOp::create(fb, mask, newCurrent, zero32);
1175:         Value newTxCount =
1176:             arith::AddIOp::create(fb, currentTxCount, txCountTensor);
1177:         Value newTxCountMasked =
1178:             arith::SelectOp::create(fb, mask, newTxCount, zero32);
1179:         Value arrivalsNonNegative = arith::CmpIOp::create(
1180:             fb, arith::CmpIPredicate::sge, newCurrentMasked, zero32);
1181:         Value minTxCount = tti::createConstIntTensor(
1182:             fb, fb.getLoc(), BarrierBits::txCountMin, barrierStatesType,
1183:             /*isSigned=*/true);
1184:         Value maxTxCount = tti::createConstIntTensor(
1185:             fb, fb.getLoc(), BarrierBits::txCountMax, barrierStatesType);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1186-1203

```cpp
1186:         Value txCountInRange = arith::AndIOp::create(
1187:             fb,
1188:             arith::CmpIOp::create(fb, arith::CmpIPredicate::sge,
1189:                                   newTxCountMasked, minTxCount),
1190:             arith::CmpIOp::create(fb, arith::CmpIPredicate::sle,
1191:                                   newTxCountMasked, maxTxCount));
1192:         Value valid =
1193:             arith::AndIOp::create(fb, arrivalsNonNegative, txCountInRange);
1194:         Value vTrue = tti::createConstIntTensor(
1195:             fb, fb.getLoc(), 1, cast<RankedTensorType>(valid.getType()));
1196:         auto condType = cast<RankedTensorType>(valid.getType());
1197:         Value ctaMask =
1198:             createCTASetMask(fb, condType, /*dim=*/0, recipientCTAs);
1199:         valid = arith::SelectOp::create(fb, ctaMask, valid, vTrue);
1200:         Value predTensor = triton::SplatOp::create(
1201:             fb, cast<RankedTensorType>(valid.getType()), pred);
1202:         Value predicatedValid =
1203:             arith::SelectOp::create(fb, predTensor, valid, vTrue);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1205-1208

```cpp
1205:         triton::ReturnOp::create(fb,
1206:                                  reduceAll<arith::AndIOp>(fb, predicatedValid));
1207:       });
1208: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1210-1217

```cpp
1210: void FunctionBuilder::createUpdateBarrierStateCall(
1211:     ImplicitLocOpBuilder &b, Value mbar, int count, Value pred,
1212:     Operation *insertPoint, Value recipientCTAs, int txCount) {
1213:   assert(count >= 0 && (uint64_t)count <= BarrierBits::countMask &&
1214:          "barrier update count exceeds barrier state capacity");
1215:   assert(txCount >= BarrierBits::txCountMin &&
1216:          txCount <= BarrierBits::txCountMax &&
1217:          "barrier tx-count delta exceeds barrier state capacity");
```

- **EN:** Defines helper `FunctionBuilder::createUpdateBarrierStateCall` that computes or constructs intermediate data used by the surrounding transformation. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义辅助函数 `FunctionBuilder::createUpdateBarrierStateCall`，用于计算或构造外围变换所需的中间数据。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 1219-1235

```cpp
1219:   if (auxData.barriers.empty() || auxData.barrierStates.empty()) {
1220:     return;
1221:   }
1222:   if (!pred) {
1223:     pred = arith::ConstantIntOp::create(b, 1, 1);
1224:   }
1225:   Value countVal = arith::ConstantIntOp::create(b, count, 32);
1226:   Value txCountVal = arith::ConstantIntOp::create(b, txCount, 64);
1227:   Value barriersVal = auxData.barriers.at(insertPoint).value;
1228:   auto barriersType =
1229:       cast<RankedTensorType>(auxData.barriers.at(insertPoint).type);
1230:   Value barrierStatesVal = auxData.barrierStates.at(insertPoint).value;
1231:   auto barrierStatesType =
1232:       cast<RankedTensorType>(auxData.barrierStates.at(insertPoint).type);
1233:   uint32_t length = getMemDescLength(mbar);
1234:   Value mbarOffset = tti::ExperimentalMemDescToI32Op::create(b, mbar);
1235:   Value lengthVal = arith::ConstantIntOp::create(b, length, 32);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1236-1247

```cpp
1236:   SmallVector<Value> args = {mbarOffset,       lengthVal,    countVal,
1237:                              txCountVal,       pred,         barriersVal,
1238:                              barrierStatesVal, recipientCTAs};
1239:   createCallToCachedFunction(
1240:       b, "update_barrier_state", args,
1241:       /*assertInfo=*/std::nullopt, {barriersType, barrierStatesType},
1242:       [barrierStatesType](ImplicitLocOpBuilder &fb, Block *entryBlock) {
1243:         Value mbarOffset = entryBlock->getArgument(0);
1244:         Value lengthVal = entryBlock->getArgument(1);
1245:         Value count = entryBlock->getArgument(2);
1246:         Value txCount = entryBlock->getArgument(3);
1247:         Value pred = entryBlock->getArgument(4);
```

- **EN:** Defines helper `createCallToCachedFunction` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createCallToCachedFunction`，用于计算或构造外围变换所需的中间数据。
### Lines 1249-1251

```cpp
1249:         Value barriers = entryBlock->getArgument(5);
1250:         Value statesPtr = entryBlock->getArgument(6);
1251:         Value recipientCTAs = entryBlock->getArgument(7);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1253-1254

```cpp
1253:         auto [prevBlock, ifBlock, thenBlock] = createIfBlock(fb, pred);
1254:         fb.setInsertionPointToStart(ifBlock);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1256-1260

```cpp
1256:         Value states = tti::createLoadScratchMemory(fb, fb.getLoc(), statesPtr,
1257:                                                     barrierStatesType);
1258:         Value descriptor = createBufferDescriptor(fb, mbarOffset, lengthVal);
1259:         Value mask = createCmpIntTensorScalar(fb, barriers, descriptor);
1260:         mask = convertAndBroadcast(fb, mask, {1}, barrierStatesType);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1262-1276

```cpp
1262:         Value zero32 =
1263:             tti::createConstIntTensor(fb, fb.getLoc(), 0, barrierStatesType);
1264:         Value one32 =
1265:             tti::createConstIntTensor(fb, fb.getLoc(), 1, barrierStatesType);
1266:         Value maskFF = tti::createConstIntTensor(
1267:             fb, fb.getLoc(), BarrierBits::countMask, barrierStatesType);
1268:         Value shiftInitTensor = tti::createConstIntTensor(
1269:             fb, fb.getLoc(), BarrierBits::initCountLsb, barrierStatesType);
1270:         Value shiftCurrentTensor = tti::createConstIntTensor(
1271:             fb, fb.getLoc(), BarrierBits::currentCountLsb, barrierStatesType);
1272:         Value shiftTxTensor = tti::createConstIntTensor(
1273:             fb, fb.getLoc(), BarrierBits::txCountLsb, barrierStatesType);
1274:         Value shiftTxSignTensor = tti::createConstIntTensor(
1275:             fb, fb.getLoc(), 64 - BarrierBits::txCountBitWidth,
1276:             barrierStatesType);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1278-1289

```cpp
1278:         Value phase = arith::AndIOp::create(fb, states, one32);
1279:         Value initCount = arith::ShRUIOp::create(fb, states, shiftInitTensor);
1280:         initCount = arith::AndIOp::create(fb, initCount, maskFF);
1281:         Value currentCount =
1282:             arith::ShRUIOp::create(fb, states, shiftCurrentTensor);
1283:         currentCount = arith::AndIOp::create(fb, currentCount, maskFF);
1284:         Value currentTxCount =
1285:             arith::ShRUIOp::create(fb, states, shiftTxTensor);
1286:         currentTxCount =
1287:             arith::ShLIOp::create(fb, currentTxCount, shiftTxSignTensor);
1288:         currentTxCount =
1289:             arith::ShRSIOp::create(fb, currentTxCount, shiftTxSignTensor);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1291-1299

```cpp
1291:         Value countMask =
1292:             arith::ConstantIntOp::create(fb, BarrierBits::countMask, 64);
1293:         Value countWide = adjustIntegerWidth(
1294:             fb, count, cast<IntegerType>(barrierStatesType.getElementType()));
1295:         Value maskedCount = arith::AndIOp::create(fb, countWide, countMask);
1296:         Value arriveCount =
1297:             triton::SplatOp::create(fb, barrierStatesType, maskedCount);
1298:         Value txCountTensor =
1299:             triton::SplatOp::create(fb, barrierStatesType, txCount);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1301-1307

```cpp
1301:         Value newCurrent = arith::SubIOp::create(fb, currentCount, arriveCount);
1302:         Value newCurrentMasked =
1303:             arith::SelectOp::create(fb, mask, newCurrent, currentCount);
1304:         Value newTxCount =
1305:             arith::AddIOp::create(fb, currentTxCount, txCountTensor);
1306:         Value newTxCountMasked =
1307:             arith::SelectOp::create(fb, mask, newTxCount, currentTxCount);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1309-1322

```cpp
1309:         Value zeroCond = arith::AndIOp::create(
1310:             fb,
1311:             arith::CmpIOp::create(fb, arith::CmpIPredicate::eq,
1312:                                   newCurrentMasked, zero32),
1313:             arith::CmpIOp::create(fb, arith::CmpIPredicate::eq,
1314:                                   newTxCountMasked, zero32));
1315:         zeroCond = arith::AndIOp::create(fb, zeroCond, mask);
1316:         Value zeroCondI32 =
1317:             arith::ExtUIOp::create(fb, barrierStatesType, zeroCond);
1318:         Value newPhase = arith::XOrIOp::create(fb, phase, zeroCondI32);
1319:         Value newCurrentValue =
1320:             arith::SelectOp::create(fb, zeroCond, initCount, newCurrentMasked);
1321:         Value newTxCountValue =
1322:             arith::SelectOp::create(fb, zeroCond, zero32, newTxCountMasked);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1324-1334

```cpp
1324:         Value initField = arith::ShLIOp::create(fb, initCount, shiftInitTensor);
1325:         Value currentField =
1326:             arith::ShLIOp::create(fb, newCurrentValue, shiftCurrentTensor);
1327:         Value txCountMask = tti::createConstIntTensor(
1328:             fb, fb.getLoc(), BarrierBits::txCountMask, barrierStatesType);
1329:         Value txCountField =
1330:             arith::AndIOp::create(fb, newTxCountValue, txCountMask);
1331:         txCountField = arith::ShLIOp::create(fb, txCountField, shiftTxTensor);
1332:         Value newState = arith::OrIOp::create(fb, newPhase, initField);
1333:         newState = arith::OrIOp::create(fb, newState, currentField);
1334:         newState = arith::OrIOp::create(fb, newState, txCountField);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1336-1338

```cpp
1336:         Value updated = arith::SelectOp::create(fb, mask, newState, states);
1337:         createCTAScopedStoreScratchMemory(fb, fb.getLoc(), statesPtr, updated,
1338:                                           barrierStatesType, recipientCTAs);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1340-1343

```cpp
1340:         fb.setInsertionPointToEnd(thenBlock);
1341:         triton::ReturnOp::create(fb);
1342:       });
1343: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1345-1347

```cpp
1345: void FunctionBuilder::createSetWriteVisibilityCall(
1346:     ImplicitLocOpBuilder &b, Value buf, uint32_t length, uint64_t threadMask,
1347:     Value pred, MemType memType, Operation *insertPoint, Value effectCTAs) {
```

- **EN:** Defines helper `FunctionBuilder::createSetWriteVisibilityCall` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `FunctionBuilder::createSetWriteVisibilityCall`，用于计算或构造外围变换所需的中间数据。
### Lines 1349-1364

```cpp
1349:   if (auxData.buffers[(int)memType].empty() ||
1350:       auxData.writeVisibility[(int)memType].empty()) {
1351:     return;
1352:   }
1353:   if (!pred)
1354:     pred = arith::ConstantIntOp::create(b, 1, 1);
1355:   Value threadMaskVal = arith::ConstantIntOp::create(b, threadMask, 64);
1356:   Value buffersVal = auxData.buffers[(int)memType].at(insertPoint).value;
1357:   auto buffersType = cast<RankedTensorType>(
1358:       auxData.buffers[(int)memType].at(insertPoint).type);
1359:   Value writeVisibilityVal =
1360:       auxData.writeVisibility[(int)memType].at(insertPoint).value;
1361:   auto writeVisibilityType = cast<RankedTensorType>(
1362:       auxData.writeVisibility[(int)memType].at(insertPoint).type);
1363:   Value bufOffset = tti::ExperimentalMemDescToI32Op::create(b, buf);
1364:   Value lengthVal = arith::ConstantIntOp::create(b, length, 32);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1365-1379

```cpp
1365:   SmallVector<Value> args = {bufOffset,     lengthVal,  pred,
1366:                              threadMaskVal, buffersVal, writeVisibilityVal,
1367:                              effectCTAs};
1368:   createCallToCachedFunction(
1369:       b, "set_write_visibility", args,
1370:       /*assertInfo=*/std::nullopt,
1371:       {buffersType, writeVisibilityType, (uint64_t)memType},
1372:       [writeVisibilityType](ImplicitLocOpBuilder &fb, Block *entryBlock) {
1373:         Value bufOffset = entryBlock->getArgument(0);
1374:         Value lengthVal = entryBlock->getArgument(1);
1375:         Value pred = entryBlock->getArgument(2);
1376:         Value threadMaskVal = entryBlock->getArgument(3);
1377:         Value buffers = entryBlock->getArgument(4);
1378:         Value writeVisibilityPtr = entryBlock->getArgument(5);
1379:         Value effectCTAs = entryBlock->getArgument(6);
```

- **EN:** Defines helper `createCallToCachedFunction` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createCallToCachedFunction`，用于计算或构造外围变换所需的中间数据。
### Lines 1381-1382

```cpp
1381:         auto [prevBlock, ifBlock, thenBlock] = createIfBlock(fb, pred);
1382:         fb.setInsertionPointToStart(ifBlock);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1384-1401

```cpp
1384:         Value writeVisibility = tti::createLoadScratchMemory(
1385:             fb, fb.getLoc(), writeVisibilityPtr, writeVisibilityType);
1386:         Value descriptor = createBufferDescriptor(fb, bufOffset, lengthVal);
1387:         Value buffersEqBuf = createCmpIntTensorScalar(fb, buffers, descriptor);
1388:         buffersEqBuf =
1389:             convertAndBroadcast(fb, buffersEqBuf, {1}, writeVisibilityType);
1390:         Value relationMask =
1391:             createLeadCTAEffectMask(fb, writeVisibilityType, effectCTAs);
1392:         buffersEqBuf = arith::AndIOp::create(fb, buffersEqBuf, relationMask);
1393:         auto elemType = cast<IntegerType>(writeVisibilityType.getElementType());
1394:         Value threadMaskElem = adjustIntegerWidth(fb, threadMaskVal, elemType);
1395:         Value threadMaskTensor =
1396:             triton::SplatOp::create(fb, writeVisibilityType, threadMaskElem);
1397:         Value newVisibility = arith::SelectOp::create(
1398:             fb, buffersEqBuf, threadMaskTensor, writeVisibility);
1399:         createMaskedStoreScratchMemory(fb, fb.getLoc(), writeVisibilityPtr,
1400:                                        newVisibility, writeVisibilityType,
1401:                                        relationMask);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1403-1406

```cpp
1403:         fb.setInsertionPointToEnd(thenBlock);
1404:         triton::ReturnOp::create(fb);
1405:       });
1406: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1408-1410

```cpp
1408: void FunctionBuilder::createSetReadVisibilityCall(
1409:     ImplicitLocOpBuilder &b, Value buf, uint32_t length, uint64_t threadMask,
1410:     Value pred, MemType memType, Operation *insertPoint, Value effectCTAs) {
```

- **EN:** Defines helper `FunctionBuilder::createSetReadVisibilityCall` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `FunctionBuilder::createSetReadVisibilityCall`，用于计算或构造外围变换所需的中间数据。
### Lines 1412-1427

```cpp
1412:   if (auxData.buffers[(int)memType].empty() ||
1413:       auxData.readVisibility[(int)memType].empty()) {
1414:     return;
1415:   }
1416:   if (!pred)
1417:     pred = arith::ConstantIntOp::create(b, 1, 1);
1418:   Value threadMaskVal = arith::ConstantIntOp::create(b, threadMask, 64);
1419:   Value buffersVal = auxData.buffers[(int)memType].at(insertPoint).value;
1420:   auto buffersType = cast<RankedTensorType>(
1421:       auxData.buffers[(int)memType].at(insertPoint).type);
1422:   Value readVisibilityVal =
1423:       auxData.readVisibility[(int)memType].at(insertPoint).value;
1424:   auto readVisibilityType = cast<RankedTensorType>(
1425:       auxData.readVisibility[(int)memType].at(insertPoint).type);
1426:   Value bufOffset = tti::ExperimentalMemDescToI32Op::create(b, buf);
1427:   Value lengthVal = arith::ConstantIntOp::create(b, length, 32);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1428-1442

```cpp
1428:   SmallVector<Value> args = {bufOffset,     lengthVal,  pred,
1429:                              threadMaskVal, buffersVal, readVisibilityVal,
1430:                              effectCTAs};
1431:   createCallToCachedFunction(
1432:       b, "set_read_visibility", args,
1433:       /*assertInfo=*/std::nullopt,
1434:       {buffersType, readVisibilityType, (uint64_t)memType},
1435:       [readVisibilityType](ImplicitLocOpBuilder &fb, Block *entryBlock) {
1436:         Value bufOffset = entryBlock->getArgument(0);
1437:         Value lengthVal = entryBlock->getArgument(1);
1438:         Value pred = entryBlock->getArgument(2);
1439:         Value threadMaskVal = entryBlock->getArgument(3);
1440:         Value buffers = entryBlock->getArgument(4);
1441:         Value readVisibilityPtr = entryBlock->getArgument(5);
1442:         Value effectCTAs = entryBlock->getArgument(6);
```

- **EN:** Defines helper `createCallToCachedFunction` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createCallToCachedFunction`，用于计算或构造外围变换所需的中间数据。
### Lines 1444-1445

```cpp
1444:         auto [prevBlock, ifBlock, thenBlock] = createIfBlock(fb, pred);
1445:         fb.setInsertionPointToStart(ifBlock);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1447-1464

```cpp
1447:         Value readVisibility = tti::createLoadScratchMemory(
1448:             fb, fb.getLoc(), readVisibilityPtr, readVisibilityType);
1449:         Value descriptor = createBufferDescriptor(fb, bufOffset, lengthVal);
1450:         Value buffersEqBuf = createCmpIntTensorScalar(fb, buffers, descriptor);
1451:         buffersEqBuf =
1452:             convertAndBroadcast(fb, buffersEqBuf, {1}, readVisibilityType);
1453:         Value relationMask =
1454:             createLeadCTAEffectMask(fb, readVisibilityType, effectCTAs);
1455:         Value threadCTAMask =
1456:             createCTASetMask(fb, readVisibilityType, /*dim=*/2, effectCTAs);
1457:         threadCTAMask = arith::AndIOp::create(
1458:             fb, threadCTAMask,
1459:             createCTASetMask(fb, readVisibilityType, /*dim=*/4, effectCTAs));
1460:         Value sameCTA = arith::CmpIOp::create(
1461:             fb, arith::CmpIPredicate::eq,
1462:             createDimIndices(fb, readVisibilityType, /*dim=*/2),
1463:             createDimIndices(fb, readVisibilityType, /*dim=*/4));
1464:         threadCTAMask = arith::AndIOp::create(fb, threadCTAMask, sameCTA);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1465-1482

```cpp
1465:         relationMask = arith::AndIOp::create(fb, relationMask, threadCTAMask);
1466:         buffersEqBuf = arith::AndIOp::create(fb, buffersEqBuf, relationMask);
1467:         auto elemType = cast<IntegerType>(readVisibilityType.getElementType());
1468:         Value threadMaskElem = adjustIntegerWidth(fb, threadMaskVal, elemType);
1469:         Value threadBit =
1470:             triton::SplatOp::create(fb, readVisibilityType, threadMaskElem);
1471:         Value threadColumnMask =
1472:             createThreadColumnMask(fb, threadMaskVal, readVisibilityType,
1473:                                    /*columnDim=*/3);
1474:         Value readVisibilityOrThreadBit =
1475:             arith::OrIOp::create(fb, readVisibility, threadBit);
1476:         Value bufAndThread =
1477:             arith::AndIOp::create(fb, buffersEqBuf, threadColumnMask);
1478:         Value newVisibility = arith::SelectOp::create(
1479:             fb, bufAndThread, readVisibilityOrThreadBit, readVisibility);
1480:         createMaskedStoreScratchMemory(fb, fb.getLoc(), readVisibilityPtr,
1481:                                        newVisibility, readVisibilityType,
1482:                                        relationMask);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1484-1487

```cpp
1484:         fb.setInsertionPointToEnd(thenBlock);
1485:         triton::ReturnOp::create(fb);
1486:       });
1487: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1489-1506

```cpp
1489: void FunctionBuilder::createClearWriteTrackingCall(ImplicitLocOpBuilder &b,
1490:                                                    Value buf, uint32_t length,
1491:                                                    Value pred, MemType memType,
1492:                                                    Operation *insertPoint,
1493:                                                    Value effectCTAs) {
1494:   if (auxData.buffers[(int)memType].empty() ||
1495:       auxData.writeTracking[(int)memType].empty()) {
1496:     return;
1497:   }
1498:   if (!pred)
1499:     pred = arith::ConstantIntOp::create(b, 1, 1);
1500:   Value buffersVal = auxData.buffers[(int)memType].at(insertPoint).value;
1501:   auto buffersType = cast<RankedTensorType>(
1502:       auxData.buffers[(int)memType].at(insertPoint).type);
1503:   Value writeTrackingVal =
1504:       auxData.writeTracking[(int)memType].at(insertPoint).value;
1505:   auto writeTrackingType = cast<RankedTensorType>(
1506:       auxData.writeTracking[(int)memType].at(insertPoint).type);
```

- **EN:** Defines helper `FunctionBuilder::createClearWriteTrackingCall` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `FunctionBuilder::createClearWriteTrackingCall`，用于计算或构造外围变换所需的中间数据。
### Lines 1507-1521

```cpp
1507:   Value bufOffset = tti::ExperimentalMemDescToI32Op::create(b, buf);
1508:   Value lengthVal = arith::ConstantIntOp::create(b, length, 32);
1509:   SmallVector<Value> args = {bufOffset,  lengthVal,        pred,
1510:                              buffersVal, writeTrackingVal, effectCTAs};
1511:   createCallToCachedFunction(
1512:       b, "clear_write_tracking", args,
1513:       /*assertInfo=*/std::nullopt,
1514:       {buffersType, writeTrackingType, (uint64_t)memType},
1515:       [writeTrackingType](ImplicitLocOpBuilder &fb, Block *entryBlock) {
1516:         Value bufOffset = entryBlock->getArgument(0);
1517:         Value lengthVal = entryBlock->getArgument(1);
1518:         Value pred = entryBlock->getArgument(2);
1519:         Value buffers = entryBlock->getArgument(3);
1520:         Value writeTrackingPtr = entryBlock->getArgument(4);
1521:         Value effectCTAs = entryBlock->getArgument(5);
```

- **EN:** Defines helper `createCallToCachedFunction` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createCallToCachedFunction`，用于计算或构造外围变换所需的中间数据。
### Lines 1523-1524

```cpp
1523:         auto [prevBlock, ifBlock, thenBlock] = createIfBlock(fb, pred);
1524:         fb.setInsertionPointToStart(ifBlock);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1526-1540

```cpp
1526:         Value writeTracking = tti::createLoadScratchMemory(
1527:             fb, fb.getLoc(), writeTrackingPtr, writeTrackingType);
1528:         Value descriptor = createBufferDescriptor(fb, bufOffset, lengthVal);
1529:         Value buffersEqBuf = createCmpIntTensorScalar(fb, buffers, descriptor);
1530:         buffersEqBuf =
1531:             convertAndBroadcast(fb, buffersEqBuf, {1}, writeTrackingType);
1532:         Value ctaMask =
1533:             createCTASetMask(fb, writeTrackingType, /*dim=*/0, effectCTAs);
1534:         buffersEqBuf = arith::AndIOp::create(fb, buffersEqBuf, ctaMask);
1535:         Value zero =
1536:             tti::createConstIntTensor(fb, fb.getLoc(), 0, writeTrackingType);
1537:         Value newTracking =
1538:             arith::SelectOp::create(fb, buffersEqBuf, zero, writeTracking);
1539:         createMaskedStoreScratchMemory(fb, fb.getLoc(), writeTrackingPtr,
1540:                                        newTracking, writeTrackingType, ctaMask);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1542-1545

```cpp
1542:         fb.setInsertionPointToEnd(thenBlock);
1543:         triton::ReturnOp::create(fb);
1544:       });
1545: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1547-1564

```cpp
1547: void FunctionBuilder::createClearReadVisibilityCall(ImplicitLocOpBuilder &b,
1548:                                                     Value buf, uint32_t length,
1549:                                                     Value pred, MemType memType,
1550:                                                     Operation *insertPoint,
1551:                                                     Value effectCTAs) {
1552:   if (auxData.buffers[(int)memType].empty() ||
1553:       auxData.readVisibility[(int)memType].empty()) {
1554:     return;
1555:   }
1556:   if (!pred)
1557:     pred = arith::ConstantIntOp::create(b, 1, 1);
1558:   Value buffersVal = auxData.buffers[(int)memType].at(insertPoint).value;
1559:   auto buffersType = cast<RankedTensorType>(
1560:       auxData.buffers[(int)memType].at(insertPoint).type);
1561:   Value readVisibilityVal =
1562:       auxData.readVisibility[(int)memType].at(insertPoint).value;
1563:   auto readVisibilityType = cast<RankedTensorType>(
1564:       auxData.readVisibility[(int)memType].at(insertPoint).type);
```

- **EN:** Defines helper `FunctionBuilder::createClearReadVisibilityCall` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `FunctionBuilder::createClearReadVisibilityCall`，用于计算或构造外围变换所需的中间数据。
### Lines 1565-1579

```cpp
1565:   Value bufOffset = tti::ExperimentalMemDescToI32Op::create(b, buf);
1566:   Value lengthVal = arith::ConstantIntOp::create(b, length, 32);
1567:   SmallVector<Value> args = {bufOffset,  lengthVal,         pred,
1568:                              buffersVal, readVisibilityVal, effectCTAs};
1569:   createCallToCachedFunction(
1570:       b, "clear_read_visibility", args,
1571:       /*assertInfo=*/std::nullopt,
1572:       {buffersType, readVisibilityType, (uint64_t)memType},
1573:       [readVisibilityType](ImplicitLocOpBuilder &fb, Block *entryBlock) {
1574:         Value bufOffset = entryBlock->getArgument(0);
1575:         Value lengthVal = entryBlock->getArgument(1);
1576:         Value pred = entryBlock->getArgument(2);
1577:         Value buffers = entryBlock->getArgument(3);
1578:         Value readVisibilityPtr = entryBlock->getArgument(4);
1579:         Value effectCTAs = entryBlock->getArgument(5);
```

- **EN:** Defines helper `createCallToCachedFunction` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createCallToCachedFunction`，用于计算或构造外围变换所需的中间数据。
### Lines 1581-1582

```cpp
1581:         auto [prevBlock, ifBlock, thenBlock] = createIfBlock(fb, pred);
1582:         fb.setInsertionPointToStart(ifBlock);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1584-1599

```cpp
1584:         Value readVisibility = tti::createLoadScratchMemory(
1585:             fb, fb.getLoc(), readVisibilityPtr, readVisibilityType);
1586:         Value descriptor = createBufferDescriptor(fb, bufOffset, lengthVal);
1587:         Value buffersEqBuf = createCmpIntTensorScalar(fb, buffers, descriptor);
1588:         buffersEqBuf =
1589:             convertAndBroadcast(fb, buffersEqBuf, {1}, readVisibilityType);
1590:         Value ctaMask =
1591:             createCTASetMask(fb, readVisibilityType, /*dim=*/0, effectCTAs);
1592:         buffersEqBuf = arith::AndIOp::create(fb, buffersEqBuf, ctaMask);
1593:         Value zero =
1594:             tti::createConstIntTensor(fb, fb.getLoc(), 0, readVisibilityType);
1595:         Value newVisibility =
1596:             arith::SelectOp::create(fb, buffersEqBuf, zero, readVisibility);
1597:         createMaskedStoreScratchMemory(fb, fb.getLoc(), readVisibilityPtr,
1598:                                        newVisibility, readVisibilityType,
1599:                                        ctaMask);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1601-1604

```cpp
1601:         fb.setInsertionPointToEnd(thenBlock);
1602:         triton::ReturnOp::create(fb);
1603:       });
1604: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1606-1610

```cpp
1606: void FunctionBuilder::createClearReadTrackingCall(ImplicitLocOpBuilder &b,
1607:                                                   Value buf, uint32_t length,
1608:                                                   Value pred, MemType memType,
1609:                                                   Operation *insertPoint,
1610:                                                   Value effectCTAs) {
```

- **EN:** Defines helper `FunctionBuilder::createClearReadTrackingCall` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `FunctionBuilder::createClearReadTrackingCall`，用于计算或构造外围变换所需的中间数据。
### Lines 1612-1628

```cpp
1612:   if (auxData.buffers[(int)memType].empty() ||
1613:       auxData.readTracking[(int)memType].empty()) {
1614:     return;
1615:   }
1616:   if (!pred)
1617:     pred = arith::ConstantIntOp::create(b, 1, 1);
1618:   Value buffersVal = auxData.buffers[(int)memType].at(insertPoint).value;
1619:   auto buffersType = cast<RankedTensorType>(
1620:       auxData.buffers[(int)memType].at(insertPoint).type);
1621:   Value readTrackingVal =
1622:       auxData.readTracking[(int)memType].at(insertPoint).value;
1623:   auto readTrackingType = cast<RankedTensorType>(
1624:       auxData.readTracking[(int)memType].at(insertPoint).type);
1625:   Value bufOffset = tti::ExperimentalMemDescToI32Op::create(b, buf);
1626:   Value lengthVal = arith::ConstantIntOp::create(b, length, 32);
1627:   SmallVector<Value> args = {bufOffset,  lengthVal,       pred,
1628:                              buffersVal, readTrackingVal, effectCTAs};
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1629-1639

```cpp
1629:   createCallToCachedFunction(
1630:       b, "clear_read_tracking", args,
1631:       /*assertInfo=*/std::nullopt,
1632:       {buffersType, readTrackingType, (uint64_t)memType},
1633:       [readTrackingType](ImplicitLocOpBuilder &fb, Block *entryBlock) {
1634:         Value bufOffset = entryBlock->getArgument(0);
1635:         Value lengthVal = entryBlock->getArgument(1);
1636:         Value pred = entryBlock->getArgument(2);
1637:         Value buffers = entryBlock->getArgument(3);
1638:         Value readTrackingPtr = entryBlock->getArgument(4);
1639:         Value effectCTAs = entryBlock->getArgument(5);
```

- **EN:** Defines helper `createCallToCachedFunction` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createCallToCachedFunction`，用于计算或构造外围变换所需的中间数据。
### Lines 1641-1642

```cpp
1641:         auto [prevBlock, ifBlock, thenBlock] = createIfBlock(fb, pred);
1642:         fb.setInsertionPointToStart(ifBlock);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1644-1658

```cpp
1644:         Value readTracking = tti::createLoadScratchMemory(
1645:             fb, fb.getLoc(), readTrackingPtr, readTrackingType);
1646:         Value descriptor = createBufferDescriptor(fb, bufOffset, lengthVal);
1647:         Value buffersEqBuf = createCmpIntTensorScalar(fb, buffers, descriptor);
1648:         buffersEqBuf =
1649:             convertAndBroadcast(fb, buffersEqBuf, {1}, readTrackingType);
1650:         Value ctaMask =
1651:             createCTASetMask(fb, readTrackingType, /*dim=*/0, effectCTAs);
1652:         buffersEqBuf = arith::AndIOp::create(fb, buffersEqBuf, ctaMask);
1653:         Value zero =
1654:             tti::createConstIntTensor(fb, fb.getLoc(), 0, readTrackingType);
1655:         Value newTracking =
1656:             arith::SelectOp::create(fb, buffersEqBuf, zero, readTracking);
1657:         createMaskedStoreScratchMemory(fb, fb.getLoc(), readTrackingPtr,
1658:                                        newTracking, readTrackingType, ctaMask);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1660-1663

```cpp
1660:         fb.setInsertionPointToEnd(thenBlock);
1661:         triton::ReturnOp::create(fb);
1662:       });
1663: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1665-1682

```cpp
1665: void FunctionBuilder::createTrackVisibleWritesCall(ImplicitLocOpBuilder &b,
1666:                                                    Value mbar, int thread,
1667:                                                    Value pred, MemType memType,
1668:                                                    Operation *insertPoint,
1669:                                                    Value barrierCTAs) {
1670:   if (auxData.barriers.empty() ||
1671:       auxData.writeVisibility[(int)memType].empty() ||
1672:       auxData.writeTracking[(int)memType].empty()) {
1673:     return;
1674:   }
1675:   if (!pred)
1676:     pred = arith::ConstantIntOp::create(b, 1, 1);
1677:   Value threadVal = arith::ConstantIntOp::create(b, thread, 32);
1678:   Value barriersVal = auxData.barriers.at(insertPoint).value;
1679:   auto barriersType =
1680:       cast<RankedTensorType>(auxData.barriers.at(insertPoint).type);
1681:   Value writeVisibilityVal =
1682:       auxData.writeVisibility[(int)memType].at(insertPoint).value;
```

- **EN:** Defines helper `FunctionBuilder::createTrackVisibleWritesCall` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `FunctionBuilder::createTrackVisibleWritesCall`，用于计算或构造外围变换所需的中间数据。
### Lines 1683-1694

```cpp
1683:   auto writeVisibilityType = cast<RankedTensorType>(
1684:       auxData.writeVisibility[(int)memType].at(insertPoint).type);
1685:   Value writeTrackingVal =
1686:       auxData.writeTracking[(int)memType].at(insertPoint).value;
1687:   auto writeTrackingType = cast<RankedTensorType>(
1688:       auxData.writeTracking[(int)memType].at(insertPoint).type);
1689:   uint32_t length = getMemDescLength(mbar);
1690:   Value mbarOffset = tti::ExperimentalMemDescToI32Op::create(b, mbar);
1691:   Value lengthVal = arith::ConstantIntOp::create(b, length, 32);
1692:   SmallVector<Value> args = {mbarOffset,       lengthVal,   pred,
1693:                              threadVal,        barriersVal, writeVisibilityVal,
1694:                              writeTrackingVal, barrierCTAs};
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1695-1708

```cpp
1695:   createCallToCachedFunction(
1696:       b, "track_visible_writes", args,
1697:       /*assertInfo=*/std::nullopt,
1698:       {barriersType, writeVisibilityType, writeTrackingType, (uint64_t)memType},
1699:       [writeVisibilityType, writeTrackingType](ImplicitLocOpBuilder &fb,
1700:                                                Block *entryBlock) {
1701:         Value mbarOffset = entryBlock->getArgument(0);
1702:         Value lengthVal = entryBlock->getArgument(1);
1703:         Value pred = entryBlock->getArgument(2);
1704:         Value threadVal = entryBlock->getArgument(3);
1705:         Value barriers = entryBlock->getArgument(4);
1706:         Value writeVisibilityPtr = entryBlock->getArgument(5);
1707:         Value writeTrackingPtr = entryBlock->getArgument(6);
1708:         Value barrierCTAs = entryBlock->getArgument(7);
```

- **EN:** Defines helper `createCallToCachedFunction` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createCallToCachedFunction`，用于计算或构造外围变换所需的中间数据。
### Lines 1710-1711

```cpp
1710:         auto [prevBlock, ifBlock, thenBlock] = createIfBlock(fb, pred);
1711:         fb.setInsertionPointToStart(ifBlock);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1713-1729

```cpp
1713:         Value writeVisibility = tti::createLoadScratchMemory(
1714:             fb, fb.getLoc(), writeVisibilityPtr, writeVisibilityType);
1715:         Value writeTracking = tti::createLoadScratchMemory(
1716:             fb, fb.getLoc(), writeTrackingPtr, writeTrackingType);
1717:         Value descriptor = createBufferDescriptor(fb, mbarOffset, lengthVal);
1718:         Value barriersEqBar =
1719:             createCmpIntTensorScalar(fb, barriers, descriptor);
1720:         barriersEqBar =
1721:             convertAndBroadcast(fb, barriersEqBar, {3}, writeTrackingType);
1722:         Value barrierCTAMask =
1723:             createCTASetMask(fb, writeTrackingType, /*dim=*/2, barrierCTAs);
1724:         barriersEqBar =
1725:             arith::AndIOp::create(fb, barriersEqBar, barrierCTAMask);
1726:         Value threadI64 =
1727:             arith::ExtUIOp::create(fb, fb.getI64Type(), threadVal);
1728:         Value one64 = arith::ConstantIntOp::create(fb, 1, 64);
1729:         Value threadBitScalar = arith::ShLIOp::create(fb, one64, threadI64);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1730-1747

```cpp
1730:         Value threadBit =
1731:             triton::SplatOp::create(fb, writeVisibilityType, threadBitScalar);
1732:         Value visibleWrites =
1733:             arith::AndIOp::create(fb, writeVisibility, threadBit);
1734:         visibleWrites = arith::CmpIOp::create(fb, arith::CmpIPredicate::eq,
1735:                                               visibleWrites, threadBit);
1736:         Value sourceCTAMask = createCTASetMask(
1737:             fb, writeVisibilityType, /*dim=*/2, createCurrentCTAMask(fb));
1738:         visibleWrites = arith::AndIOp::create(fb, visibleWrites, sourceCTAMask);
1739:         visibleWrites = reduceLastDim<arith::OrIOp>(fb, visibleWrites);
1740:         visibleWrites =
1741:             convertAndBroadcast(fb, visibleWrites, {0, 1}, writeTrackingType);
1742:         Value barAndVisible =
1743:             arith::AndIOp::create(fb, barriersEqBar, visibleWrites);
1744:         Value writeTrackingOne =
1745:             tti::createConstIntTensor(fb, fb.getLoc(), 1, writeTrackingType);
1746:         Value newTracking = arith::SelectOp::create(
1747:             fb, barAndVisible, writeTrackingOne, writeTracking);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1748-1750

```cpp
1748:         createMaskedStoreScratchMemory(fb, fb.getLoc(), writeTrackingPtr,
1749:                                        newTracking, writeTrackingType,
1750:                                        barrierCTAMask);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1752-1755

```cpp
1752:         fb.setInsertionPointToEnd(thenBlock);
1753:         triton::ReturnOp::create(fb);
1754:       });
1755: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1757-1761

```cpp
1757: void FunctionBuilder::createTrackVisibleReadsCall(ImplicitLocOpBuilder &b,
1758:                                                   Value mbar, int thread,
1759:                                                   Value pred, MemType memType,
1760:                                                   Operation *insertPoint,
1761:                                                   Value barrierCTAs) {
```

- **EN:** Defines helper `FunctionBuilder::createTrackVisibleReadsCall` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `FunctionBuilder::createTrackVisibleReadsCall`，用于计算或构造外围变换所需的中间数据。
### Lines 1763-1779

```cpp
1763:   if (auxData.barriers.empty() ||
1764:       auxData.readVisibility[(int)memType].empty() ||
1765:       auxData.readTracking[(int)memType].empty()) {
1766:     return;
1767:   }
1768:   if (!pred)
1769:     pred = arith::ConstantIntOp::create(b, 1, 1);
1770:   Value threadVal = arith::ConstantIntOp::create(b, thread, 32);
1771:   Value barriersVal = auxData.barriers.at(insertPoint).value;
1772:   auto barriersType =
1773:       cast<RankedTensorType>(auxData.barriers.at(insertPoint).type);
1774:   Value readVisibilityVal =
1775:       auxData.readVisibility[(int)memType].at(insertPoint).value;
1776:   auto readVisibilityType = cast<RankedTensorType>(
1777:       auxData.readVisibility[(int)memType].at(insertPoint).type);
1778:   Value readTrackingVal =
1779:       auxData.readTracking[(int)memType].at(insertPoint).value;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1780-1797

```cpp
1780:   auto readTrackingType = cast<RankedTensorType>(
1781:       auxData.readTracking[(int)memType].at(insertPoint).type);
1782:   uint32_t length = getMemDescLength(mbar);
1783:   Value mbarOffset = tti::ExperimentalMemDescToI32Op::create(b, mbar);
1784:   Value lengthVal = arith::ConstantIntOp::create(b, length, 32);
1785:   SmallVector<Value> args = {mbarOffset,      lengthVal,   pred,
1786:                              threadVal,       barriersVal, readVisibilityVal,
1787:                              readTrackingVal, barrierCTAs};
1788:   createCallToCachedFunction(
1789:       b, "track_visible_reads", args,
1790:       /*assertInfo=*/std::nullopt,
1791:       {barriersType, readVisibilityType, readTrackingType, (uint64_t)memType},
1792:       [readVisibilityType, readTrackingType](ImplicitLocOpBuilder &fb,
1793:                                              Block *entryBlock) {
1794:         Value mbarOffset = entryBlock->getArgument(0);
1795:         Value lengthVal = entryBlock->getArgument(1);
1796:         Value pred = entryBlock->getArgument(2);
1797:         Value threadVal = entryBlock->getArgument(3);
```

- **EN:** Defines helper `createCallToCachedFunction` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createCallToCachedFunction`，用于计算或构造外围变换所需的中间数据。
### Lines 1798-1801

```cpp
1798:         Value barriers = entryBlock->getArgument(4);
1799:         Value readVisibilityPtr = entryBlock->getArgument(5);
1800:         Value readTrackingPtr = entryBlock->getArgument(6);
1801:         Value barrierCTAs = entryBlock->getArgument(7);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1803-1804

```cpp
1803:         auto [prevBlock, ifBlock, thenBlock] = createIfBlock(fb, pred);
1804:         fb.setInsertionPointToStart(ifBlock);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1806-1822

```cpp
1806:         Value readVisibility = tti::createLoadScratchMemory(
1807:             fb, fb.getLoc(), readVisibilityPtr, readVisibilityType);
1808:         Value readTracking = tti::createLoadScratchMemory(
1809:             fb, fb.getLoc(), readTrackingPtr, readTrackingType);
1810:         Value descriptor = createBufferDescriptor(fb, mbarOffset, lengthVal);
1811:         Value barriersEqBar =
1812:             createCmpIntTensorScalar(fb, barriers, descriptor);
1813:         barriersEqBar =
1814:             convertAndBroadcast(fb, barriersEqBar, {3}, readTrackingType);
1815:         Value barrierCTAMask =
1816:             createCTASetMask(fb, readTrackingType, /*dim=*/2, barrierCTAs);
1817:         barriersEqBar =
1818:             arith::AndIOp::create(fb, barriersEqBar, barrierCTAMask);
1819:         Value threadColumnMask =
1820:             createDimMask(fb, threadVal, readVisibilityType, /*dim=*/3);
1821:         Value readVisibilityZero =
1822:             tti::createConstIntTensor(fb, fb.getLoc(), 0, readVisibilityType);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1823-1838

```cpp
1823:         Value visibleReads = arith::SelectOp::create(
1824:             fb, threadColumnMask, readVisibility, readVisibilityZero);
1825:         Value sourceCTAMask = createCTASetMask(
1826:             fb, readVisibilityType, /*dim=*/2, createCurrentCTAMask(fb));
1827:         visibleReads = arith::SelectOp::create(fb, sourceCTAMask, visibleReads,
1828:                                                readVisibilityZero);
1829:         visibleReads = reduce<arith::OrIOp>(fb, visibleReads, {2, 3});
1830:         visibleReads =
1831:             convertAndBroadcast(fb, visibleReads, {0, 1, 4}, readTrackingType);
1832:         Value readTrackingOrVisible =
1833:             arith::OrIOp::create(fb, readTracking, visibleReads);
1834:         Value newTracking = arith::SelectOp::create(
1835:             fb, barriersEqBar, readTrackingOrVisible, readTracking);
1836:         createMaskedStoreScratchMemory(fb, fb.getLoc(), readTrackingPtr,
1837:                                        newTracking, readTrackingType,
1838:                                        barrierCTAMask);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1840-1843

```cpp
1840:         fb.setInsertionPointToEnd(thenBlock);
1841:         triton::ReturnOp::create(fb);
1842:       });
1843: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1845-1862

```cpp
1845: void FunctionBuilder::createTrackBarrierWriteForBufferCall(
1846:     ImplicitLocOpBuilder &b, Value mbar, Value buf, uint32_t length, Value pred,
1847:     MemType memType, Operation *insertPoint, Value barrierCTAs,
1848:     Value effectCTAs) {
1849:   if (auxData.barriers.empty() || auxData.buffers[(int)memType].empty() ||
1850:       auxData.writeTracking[(int)memType].empty()) {
1851:     return;
1852:   }
1853:   if (!pred)
1854:     pred = arith::ConstantIntOp::create(b, 1, 1);
1855:   Value barriersVal = auxData.barriers.at(insertPoint).value;
1856:   auto barriersType =
1857:       cast<RankedTensorType>(auxData.barriers.at(insertPoint).type);
1858:   Value buffersVal = auxData.buffers[(int)memType].at(insertPoint).value;
1859:   auto buffersType = cast<RankedTensorType>(
1860:       auxData.buffers[(int)memType].at(insertPoint).type);
1861:   Value writeTrackingVal =
1862:       auxData.writeTracking[(int)memType].at(insertPoint).value;
```

- **EN:** Defines helper `FunctionBuilder::createTrackBarrierWriteForBufferCall` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `FunctionBuilder::createTrackBarrierWriteForBufferCall`，用于计算或构造外围变换所需的中间数据。
### Lines 1863-1880

```cpp
1863:   auto writeTrackingType = cast<RankedTensorType>(
1864:       auxData.writeTracking[(int)memType].at(insertPoint).type);
1865:   uint32_t mbarLength = getMemDescLength(mbar);
1866:   Value mbarOffset = tti::ExperimentalMemDescToI32Op::create(b, mbar);
1867:   Value mbarLengthVal = arith::ConstantIntOp::create(b, mbarLength, 32);
1868:   Value bufOffset = tti::ExperimentalMemDescToI32Op::create(b, buf);
1869:   Value bufLengthVal = arith::ConstantIntOp::create(b, length, 32);
1870:   SmallVector<Value> args = {mbarOffset, mbarLengthVal,    pred,
1871:                              bufOffset,  bufLengthVal,     barriersVal,
1872:                              buffersVal, writeTrackingVal, barrierCTAs,
1873:                              effectCTAs};
1874:   createCallToCachedFunction(
1875:       b, "track_barrier_write_for_buffer", args,
1876:       /*assertInfo=*/std::nullopt,
1877:       {barriersType, buffersType, writeTrackingType, (uint64_t)memType},
1878:       [writeTrackingType](ImplicitLocOpBuilder &fb, Block *entryBlock) {
1879:         Value mbarOffset = entryBlock->getArgument(0);
1880:         Value mbarLengthVal = entryBlock->getArgument(1);
```

- **EN:** Defines helper `createCallToCachedFunction` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createCallToCachedFunction`，用于计算或构造外围变换所需的中间数据。
### Lines 1881-1888

```cpp
1881:         Value pred = entryBlock->getArgument(2);
1882:         Value bufOffset = entryBlock->getArgument(3);
1883:         Value bufLengthVal = entryBlock->getArgument(4);
1884:         Value barriers = entryBlock->getArgument(5);
1885:         Value buffers = entryBlock->getArgument(6);
1886:         Value writeTrackingPtr = entryBlock->getArgument(7);
1887:         Value barrierCTAs = entryBlock->getArgument(8);
1888:         Value effectCTAs = entryBlock->getArgument(9);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1890-1891

```cpp
1890:         auto [prevBlock, ifBlock, thenBlock] = createIfBlock(fb, pred);
1891:         fb.setInsertionPointToStart(ifBlock);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1893-1910

```cpp
1893:         Value writeTracking = tti::createLoadScratchMemory(
1894:             fb, fb.getLoc(), writeTrackingPtr, writeTrackingType);
1895:         Value barrierDescriptor =
1896:             createBufferDescriptor(fb, mbarOffset, mbarLengthVal);
1897:         Value barriersEqBar =
1898:             createCmpIntTensorScalar(fb, barriers, barrierDescriptor);
1899:         barriersEqBar =
1900:             convertAndBroadcast(fb, barriersEqBar, {3}, writeTrackingType);
1901:         Value bufferDescriptor =
1902:             createBufferDescriptor(fb, bufOffset, bufLengthVal);
1903:         Value buffersEqBuf =
1904:             createCmpIntTensorScalar(fb, buffers, bufferDescriptor);
1905:         buffersEqBuf =
1906:             convertAndBroadcast(fb, buffersEqBuf, {1}, writeTrackingType);
1907:         Value bufferCTAMask =
1908:             createCTASetMask(fb, writeTrackingType, /*dim=*/0, effectCTAs);
1909:         Value barrierCTAMask =
1910:             createCTASetMask(fb, writeTrackingType, /*dim=*/2, barrierCTAs);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1911-1921

```cpp
1911:         Value trackMask =
1912:             arith::AndIOp::create(fb, barriersEqBar, buffersEqBuf);
1913:         trackMask = arith::AndIOp::create(fb, trackMask, bufferCTAMask);
1914:         trackMask = arith::AndIOp::create(fb, trackMask, barrierCTAMask);
1915:         Value writeTrackingOne =
1916:             tti::createConstIntTensor(fb, fb.getLoc(), 1, writeTrackingType);
1917:         Value newTracking = arith::SelectOp::create(
1918:             fb, trackMask, writeTrackingOne, writeTracking);
1919:         createMaskedStoreScratchMemory(fb, fb.getLoc(), writeTrackingPtr,
1920:                                        newTracking, writeTrackingType,
1921:                                        trackMask);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1923-1926

```cpp
1923:         fb.setInsertionPointToEnd(thenBlock);
1924:         triton::ReturnOp::create(fb);
1925:       });
1926: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1928-1945

```cpp
1928: void FunctionBuilder::createClearBarrierWriteTrackingCall(
1929:     ImplicitLocOpBuilder &b, Value mbar, Value pred, MemType memType,
1930:     Operation *insertPoint) {
1931:   if (auxData.writeTracking[(int)memType].empty()) {
1932:     return;
1933:   }
1934:   assert(!auxData.barriers.empty() &&
1935:          "barrier descriptors must exist when clearing barrier write tracking");
1936:   if (!pred) {
1937:     pred = arith::ConstantIntOp::create(b, 1, 1);
1938:   }
1939:   Value barriersVal = auxData.barriers.at(insertPoint).value;
1940:   auto barriersType =
1941:       cast<RankedTensorType>(auxData.barriers.at(insertPoint).type);
1942:   Value writeTrackingVal =
1943:       auxData.writeTracking[(int)memType].at(insertPoint).value;
1944:   auto writeTrackingType = cast<RankedTensorType>(
1945:       auxData.writeTracking[(int)memType].at(insertPoint).type);
```

- **EN:** Defines helper `FunctionBuilder::createClearBarrierWriteTrackingCall` that computes or constructs intermediate data used by the surrounding transformation. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义辅助函数 `FunctionBuilder::createClearBarrierWriteTrackingCall`，用于计算或构造外围变换所需的中间数据。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 1946-1960

```cpp
1946:   uint32_t length = getMemDescLength(mbar);
1947:   Value mbarOffset = tti::ExperimentalMemDescToI32Op::create(b, mbar);
1948:   Value lengthVal = arith::ConstantIntOp::create(b, length, 32);
1949:   SmallVector<Value> args = {mbarOffset, lengthVal, pred, barriersVal,
1950:                              writeTrackingVal};
1951:   createCallToCachedFunction(
1952:       b, "clear_barrier_write_tracking", args,
1953:       /*assertInfo=*/std::nullopt,
1954:       {barriersType, writeTrackingType, (uint64_t)memType},
1955:       [writeTrackingType](ImplicitLocOpBuilder &fb, Block *entryBlock) {
1956:         Value mbarOffset = entryBlock->getArgument(0);
1957:         Value lengthVal = entryBlock->getArgument(1);
1958:         Value pred = entryBlock->getArgument(2);
1959:         Value barriers = entryBlock->getArgument(3);
1960:         Value writeTrackingPtr = entryBlock->getArgument(4);
```

- **EN:** Defines helper `createCallToCachedFunction` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createCallToCachedFunction`，用于计算或构造外围变换所需的中间数据。
### Lines 1962-1963

```cpp
1962:         auto [prevBlock, ifBlock, thenBlock] = createIfBlock(fb, pred);
1963:         fb.setInsertionPointToStart(ifBlock);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1965-1982

```cpp
1965:         Value writeTracking = tti::createLoadScratchMemory(
1966:             fb, fb.getLoc(), writeTrackingPtr, writeTrackingType);
1967:         Value descriptor = createBufferDescriptor(fb, mbarOffset, lengthVal);
1968:         Value barriersEqBar =
1969:             createCmpIntTensorScalar(fb, barriers, descriptor);
1970:         barriersEqBar =
1971:             convertAndBroadcast(fb, barriersEqBar, {3}, writeTrackingType);
1972:         Value barrierCTAMask = createCTASetMask(
1973:             fb, writeTrackingType, /*dim=*/2, createCurrentCTAMask(fb));
1974:         barriersEqBar =
1975:             arith::AndIOp::create(fb, barriersEqBar, barrierCTAMask);
1976:         Value zero =
1977:             tti::createConstIntTensor(fb, fb.getLoc(), 0, writeTrackingType);
1978:         Value updated =
1979:             arith::SelectOp::create(fb, barriersEqBar, zero, writeTracking);
1980:         createMaskedStoreScratchMemory(fb, fb.getLoc(), writeTrackingPtr,
1981:                                        updated, writeTrackingType,
1982:                                        barrierCTAMask);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1984-1987

```cpp
1984:         fb.setInsertionPointToEnd(thenBlock);
1985:         triton::ReturnOp::create(fb);
1986:       });
1987: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1989-2006

```cpp
1989: void FunctionBuilder::createClearBarrierReadTrackingCall(
1990:     ImplicitLocOpBuilder &b, Value mbar, Value pred, MemType memType,
1991:     Operation *insertPoint) {
1992:   if (auxData.readTracking[(int)memType].empty()) {
1993:     return;
1994:   }
1995:   assert(!auxData.barriers.empty() &&
1996:          "barrier descriptors must exist when clearing barrier read tracking");
1997:   if (!pred) {
1998:     pred = arith::ConstantIntOp::create(b, 1, 1);
1999:   }
2000:   Value barriersVal = auxData.barriers.at(insertPoint).value;
2001:   auto barriersType =
2002:       cast<RankedTensorType>(auxData.barriers.at(insertPoint).type);
2003:   Value readTrackingVal =
2004:       auxData.readTracking[(int)memType].at(insertPoint).value;
2005:   auto readTrackingType = cast<RankedTensorType>(
2006:       auxData.readTracking[(int)memType].at(insertPoint).type);
```

- **EN:** Defines helper `FunctionBuilder::createClearBarrierReadTrackingCall` that computes or constructs intermediate data used by the surrounding transformation. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义辅助函数 `FunctionBuilder::createClearBarrierReadTrackingCall`，用于计算或构造外围变换所需的中间数据。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 2007-2021

```cpp
2007:   uint32_t length = getMemDescLength(mbar);
2008:   Value mbarOffset = tti::ExperimentalMemDescToI32Op::create(b, mbar);
2009:   Value lengthVal = arith::ConstantIntOp::create(b, length, 32);
2010:   SmallVector<Value> args = {mbarOffset, lengthVal, pred, barriersVal,
2011:                              readTrackingVal};
2012:   createCallToCachedFunction(
2013:       b, "clear_barrier_read_tracking", args,
2014:       /*assertInfo=*/std::nullopt,
2015:       {barriersType, readTrackingType, (uint64_t)memType},
2016:       [readTrackingType](ImplicitLocOpBuilder &fb, Block *entryBlock) {
2017:         Value mbarOffset = entryBlock->getArgument(0);
2018:         Value lengthVal = entryBlock->getArgument(1);
2019:         Value pred = entryBlock->getArgument(2);
2020:         Value barriers = entryBlock->getArgument(3);
2021:         Value readTrackingPtr = entryBlock->getArgument(4);
```

- **EN:** Defines helper `createCallToCachedFunction` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createCallToCachedFunction`，用于计算或构造外围变换所需的中间数据。
### Lines 2023-2024

```cpp
2023:         auto [prevBlock, ifBlock, thenBlock] = createIfBlock(fb, pred);
2024:         fb.setInsertionPointToStart(ifBlock);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2026-2043

```cpp
2026:         Value readTracking = tti::createLoadScratchMemory(
2027:             fb, fb.getLoc(), readTrackingPtr, readTrackingType);
2028:         Value descriptor = createBufferDescriptor(fb, mbarOffset, lengthVal);
2029:         Value barriersEqBar =
2030:             createCmpIntTensorScalar(fb, barriers, descriptor);
2031:         barriersEqBar =
2032:             convertAndBroadcast(fb, barriersEqBar, {3}, readTrackingType);
2033:         Value barrierCTAMask = createCTASetMask(fb, readTrackingType, /*dim=*/2,
2034:                                                 createCurrentCTAMask(fb));
2035:         barriersEqBar =
2036:             arith::AndIOp::create(fb, barriersEqBar, barrierCTAMask);
2037:         Value zero =
2038:             tti::createConstIntTensor(fb, fb.getLoc(), 0, readTrackingType);
2039:         Value updated =
2040:             arith::SelectOp::create(fb, barriersEqBar, zero, readTracking);
2041:         createMaskedStoreScratchMemory(fb, fb.getLoc(), readTrackingPtr,
2042:                                        updated, readTrackingType,
2043:                                        barrierCTAMask);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2045-2048

```cpp
2045:         fb.setInsertionPointToEnd(thenBlock);
2046:         triton::ReturnOp::create(fb);
2047:       });
2048: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2050-2052

```cpp
2050: void FunctionBuilder::createTransferVisibleWritesCall(
2051:     ImplicitLocOpBuilder &b, Value mbar, uint64_t threadMask, Value pred,
2052:     MemType memType, Operation *insertPoint) {
```

- **EN:** Defines helper `FunctionBuilder::createTransferVisibleWritesCall` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `FunctionBuilder::createTransferVisibleWritesCall`，用于计算或构造外围变换所需的中间数据。
### Lines 2054-2070

```cpp
2054:   if (auxData.barriers.empty() ||
2055:       auxData.writeVisibility[(int)memType].empty() ||
2056:       auxData.writeTracking[(int)memType].empty()) {
2057:     return;
2058:   }
2059:   if (!pred)
2060:     pred = arith::ConstantIntOp::create(b, 1, 1);
2061:   Value threadMaskVal = arith::ConstantIntOp::create(b, threadMask, 64);
2062:   Value barriersVal = auxData.barriers.at(insertPoint).value;
2063:   auto barriersType =
2064:       cast<RankedTensorType>(auxData.barriers.at(insertPoint).type);
2065:   Value writeVisibilityVal =
2066:       auxData.writeVisibility[(int)memType].at(insertPoint).value;
2067:   auto writeVisibilityType = cast<RankedTensorType>(
2068:       auxData.writeVisibility[(int)memType].at(insertPoint).type);
2069:   Value writeTrackingVal =
2070:       auxData.writeTracking[(int)memType].at(insertPoint).value;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2071-2088

```cpp
2071:   auto writeTrackingType = cast<RankedTensorType>(
2072:       auxData.writeTracking[(int)memType].at(insertPoint).type);
2073:   uint32_t length = getMemDescLength(mbar);
2074:   Value mbarOffset = tti::ExperimentalMemDescToI32Op::create(b, mbar);
2075:   Value lengthVal = arith::ConstantIntOp::create(b, length, 32);
2076:   SmallVector<Value> args = {mbarOffset,      lengthVal,   pred,
2077:                              threadMaskVal,   barriersVal, writeVisibilityVal,
2078:                              writeTrackingVal};
2079:   createCallToCachedFunction(
2080:       b, "transfer_visible_writes", args,
2081:       /*assertInfo=*/std::nullopt,
2082:       {barriersType, writeVisibilityType, writeTrackingType, (uint64_t)memType},
2083:       [writeVisibilityType, writeTrackingType](ImplicitLocOpBuilder &fb,
2084:                                                Block *entryBlock) {
2085:         Value mbarOffset = entryBlock->getArgument(0);
2086:         Value lengthVal = entryBlock->getArgument(1);
2087:         Value pred = entryBlock->getArgument(2);
2088:         Value threadMaskVal = entryBlock->getArgument(3);
```

- **EN:** Defines helper `createCallToCachedFunction` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createCallToCachedFunction`，用于计算或构造外围变换所需的中间数据。
### Lines 2089-2091

```cpp
2089:         Value barriers = entryBlock->getArgument(4);
2090:         Value writeVisibilityPtr = entryBlock->getArgument(5);
2091:         Value writeTrackingPtr = entryBlock->getArgument(6);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2093-2094

```cpp
2093:         auto [prevBlock, ifBlock, thenBlock] = createIfBlock(fb, pred);
2094:         fb.setInsertionPointToStart(ifBlock);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2096-2113

```cpp
2096:         Value writeVisibility = tti::createLoadScratchMemory(
2097:             fb, fb.getLoc(), writeVisibilityPtr, writeVisibilityType);
2098:         Value writeTracking = tti::createLoadScratchMemory(
2099:             fb, fb.getLoc(), writeTrackingPtr, writeTrackingType);
2100:         Value descriptor = createBufferDescriptor(fb, mbarOffset, lengthVal);
2101:         Value barriersEqBar =
2102:             createCmpIntTensorScalar(fb, barriers, descriptor);
2103:         barriersEqBar =
2104:             convertAndBroadcast(fb, barriersEqBar, {3}, writeTrackingType);
2105:         Value currentCTA = createCurrentCTAMask(fb);
2106:         Value barrierCTAMask =
2107:             createCTASetMask(fb, writeTrackingType, /*dim=*/2, currentCTA);
2108:         barriersEqBar =
2109:             arith::AndIOp::create(fb, barriersEqBar, barrierCTAMask);
2110:         Value zeroTracking =
2111:             tti::createConstIntTensor(fb, fb.getLoc(), 0, writeTrackingType);
2112:         Value trackingBuffers = arith::SelectOp::create(
2113:             fb, barriersEqBar, writeTracking, zeroTracking);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2114-2130

```cpp
2114:         trackingBuffers = reduce<arith::OrIOp>(fb, trackingBuffers, {2, 3});
2115:         trackingBuffers = convertAndBroadcast(fb, trackingBuffers, {0, 1},
2116:                                               writeVisibilityType);
2117:         auto trackingBuffersType =
2118:             cast<RankedTensorType>(trackingBuffers.getType());
2119:         Value trackingBuffersOne =
2120:             tti::createConstIntTensor(fb, fb.getLoc(), 1, trackingBuffersType);
2121:         trackingBuffers = arith::CmpIOp::create(
2122:             fb, arith::CmpIPredicate::eq, trackingBuffers, trackingBuffersOne);
2123:         auto elemType = cast<IntegerType>(writeVisibilityType.getElementType());
2124:         Value threadMaskElem = adjustIntegerWidth(fb, threadMaskVal, elemType);
2125:         Value threadMaskTensor =
2126:             triton::SplatOp::create(fb, writeVisibilityType, threadMaskElem);
2127:         Value zeroVisibility =
2128:             tti::createConstIntTensor(fb, fb.getLoc(), 0, writeVisibilityType);
2129:         Value trackingThreadBit = arith::SelectOp::create(
2130:             fb, trackingBuffers, threadMaskTensor, zeroVisibility);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2131-2137

```cpp
2131:         Value newVisibility =
2132:             arith::OrIOp::create(fb, writeVisibility, trackingThreadBit);
2133:         Value waitingCTAMask =
2134:             createCTASetMask(fb, writeVisibilityType, /*dim=*/2, currentCTA);
2135:         createMaskedStoreScratchMemory(fb, fb.getLoc(), writeVisibilityPtr,
2136:                                        newVisibility, writeVisibilityType,
2137:                                        waitingCTAMask);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2139-2142

```cpp
2139:         fb.setInsertionPointToEnd(thenBlock);
2140:         triton::ReturnOp::create(fb);
2141:       });
2142: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2144-2146

```cpp
2144: void FunctionBuilder::createTransferVisibleReadsCall(
2145:     ImplicitLocOpBuilder &b, Value mbar, uint64_t threadMask, Value pred,
2146:     MemType memType, Operation *insertPoint) {
```

- **EN:** Defines helper `FunctionBuilder::createTransferVisibleReadsCall` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `FunctionBuilder::createTransferVisibleReadsCall`，用于计算或构造外围变换所需的中间数据。
### Lines 2148-2164

```cpp
2148:   if (auxData.barriers.empty() ||
2149:       auxData.readVisibility[(int)memType].empty() ||
2150:       auxData.readTracking[(int)memType].empty()) {
2151:     return;
2152:   }
2153:   if (!pred)
2154:     pred = arith::ConstantIntOp::create(b, 1, 1);
2155:   Value threadMaskVal = arith::ConstantIntOp::create(b, threadMask, 64);
2156:   Value barriersVal = auxData.barriers.at(insertPoint).value;
2157:   auto barriersType =
2158:       cast<RankedTensorType>(auxData.barriers.at(insertPoint).type);
2159:   Value readVisibilityVal =
2160:       auxData.readVisibility[(int)memType].at(insertPoint).value;
2161:   auto readVisibilityType = cast<RankedTensorType>(
2162:       auxData.readVisibility[(int)memType].at(insertPoint).type);
2163:   Value readTrackingVal =
2164:       auxData.readTracking[(int)memType].at(insertPoint).value;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2165-2182

```cpp
2165:   auto readTrackingType = cast<RankedTensorType>(
2166:       auxData.readTracking[(int)memType].at(insertPoint).type);
2167:   uint32_t length = getMemDescLength(mbar);
2168:   Value mbarOffset = tti::ExperimentalMemDescToI32Op::create(b, mbar);
2169:   Value lengthVal = arith::ConstantIntOp::create(b, length, 32);
2170:   SmallVector<Value> args = {mbarOffset,     lengthVal,   pred,
2171:                              threadMaskVal,  barriersVal, readVisibilityVal,
2172:                              readTrackingVal};
2173:   createCallToCachedFunction(
2174:       b, "transfer_visible_reads", args,
2175:       /*assertInfo=*/std::nullopt,
2176:       {barriersType, readVisibilityType, readTrackingType, (uint64_t)memType},
2177:       [readVisibilityType, readTrackingType](ImplicitLocOpBuilder &fb,
2178:                                              Block *entryBlock) {
2179:         Value mbarOffset = entryBlock->getArgument(0);
2180:         Value lengthVal = entryBlock->getArgument(1);
2181:         Value pred = entryBlock->getArgument(2);
2182:         Value threadMaskVal = entryBlock->getArgument(3);
```

- **EN:** Defines helper `createCallToCachedFunction` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createCallToCachedFunction`，用于计算或构造外围变换所需的中间数据。
### Lines 2183-2185

```cpp
2183:         Value barriers = entryBlock->getArgument(4);
2184:         Value readVisibilityPtr = entryBlock->getArgument(5);
2185:         Value readTrackingPtr = entryBlock->getArgument(6);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2187-2188

```cpp
2187:         auto [prevBlock, ifBlock, thenBlock] = createIfBlock(fb, pred);
2188:         fb.setInsertionPointToStart(ifBlock);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2190-2207

```cpp
2190:         Value readVisibility = tti::createLoadScratchMemory(
2191:             fb, fb.getLoc(), readVisibilityPtr, readVisibilityType);
2192:         Value readTracking = tti::createLoadScratchMemory(
2193:             fb, fb.getLoc(), readTrackingPtr, readTrackingType);
2194:         Value descriptor = createBufferDescriptor(fb, mbarOffset, lengthVal);
2195:         Value barriersEqBar =
2196:             createCmpIntTensorScalar(fb, barriers, descriptor);
2197:         barriersEqBar =
2198:             convertAndBroadcast(fb, barriersEqBar, {3}, readTrackingType);
2199:         Value currentCTA = createCurrentCTAMask(fb);
2200:         Value barrierCTAMask =
2201:             createCTASetMask(fb, readTrackingType, /*dim=*/2, currentCTA);
2202:         barriersEqBar =
2203:             arith::AndIOp::create(fb, barriersEqBar, barrierCTAMask);
2204:         Value readTrackingZero =
2205:             tti::createConstIntTensor(fb, fb.getLoc(), 0, readTrackingType);
2206:         Value trackingBar = arith::SelectOp::create(
2207:             fb, barriersEqBar, readTracking, readTrackingZero);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2208-2224

```cpp
2208:         trackingBar = reduce<arith::OrIOp>(fb, trackingBar, {2, 3});
2209:         trackingBar =
2210:             convertAndBroadcast(fb, trackingBar, {0, 1, 4}, readVisibilityType);
2211:         Value readVisibilityOrTracking =
2212:             arith::OrIOp::create(fb, readVisibility, trackingBar);
2213:         Value threadColumnMask =
2214:             createThreadColumnMask(fb, threadMaskVal, readVisibilityType,
2215:                                    /*columnDim=*/3);
2216:         Value waitingCTAMask =
2217:             createCTASetMask(fb, readVisibilityType, /*dim=*/2, currentCTA);
2218:         threadColumnMask =
2219:             arith::AndIOp::create(fb, threadColumnMask, waitingCTAMask);
2220:         Value newVisibility = arith::SelectOp::create(
2221:             fb, threadColumnMask, readVisibilityOrTracking, readVisibility);
2222:         createMaskedStoreScratchMemory(fb, fb.getLoc(), readVisibilityPtr,
2223:                                        newVisibility, readVisibilityType,
2224:                                        waitingCTAMask);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2226-2229

```cpp
2226:         fb.setInsertionPointToEnd(thenBlock);
2227:         triton::ReturnOp::create(fb);
2228:       });
2229: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2231-2248

```cpp
2231: void FunctionBuilder::createVerifyWriteVisibilityCall(
2232:     ImplicitLocOpBuilder &b, Value buf, uint32_t length, int thread,
2233:     StringRef operandName, Value pred, MemType memType, Operation *insertPoint,
2234:     Value effectCTAs) {
2235:   if (auxData.buffers[(int)memType].empty() ||
2236:       auxData.writeVisibility[(int)memType].empty() ||
2237:       (auxData.hasNonTrivialAliasing[(int)memType] &&
2238:        auxData.aliasMatrices[(int)memType].empty())) {
2239:     return;
2240:   }
2241:   if (!pred)
2242:     pred = arith::ConstantIntOp::create(b, 1, 1);
2243:   Value threadVal = arith::ConstantIntOp::create(b, thread, 32);
2244:   Value buffersVal = auxData.buffers[(int)memType].at(insertPoint).value;
2245:   auto buffersType = cast<RankedTensorType>(
2246:       auxData.buffers[(int)memType].at(insertPoint).type);
2247:   Value writeVisibilityVal =
2248:       auxData.writeVisibility[(int)memType].at(insertPoint).value;
```

- **EN:** Defines helper `FunctionBuilder::createVerifyWriteVisibilityCall` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `FunctionBuilder::createVerifyWriteVisibilityCall`，用于计算或构造外围变换所需的中间数据。
### Lines 2249-2266

```cpp
2249:   auto writeVisibilityType = cast<RankedTensorType>(
2250:       auxData.writeVisibility[(int)memType].at(insertPoint).type);
2251:   Value bufOffset = tti::ExperimentalMemDescToI32Op::create(b, buf);
2252:   Value lengthVal = arith::ConstantIntOp::create(b, length, 32);
2253:   std::string message = "Buffer being accessed has outstanding writes.";
2254:   if (!operandName.empty())
2255:     message += " Operand: " + operandName.str();
2256:   AssertInfo assertInfo{message, b.getI1Type()};
2257:   Type aliasMatrixTypeBase;
2258:   auto buildVerifyWriteBody = [&writeVisibilityType,
2259:                                &aliasMatrixTypeBase](bool useAlias) {
2260:     return [=](ImplicitLocOpBuilder &fb, Block *entryBlock) {
2261:       Value bufOffset = entryBlock->getArgument(0);
2262:       Value lengthVal = entryBlock->getArgument(1);
2263:       Value pred = entryBlock->getArgument(2);
2264:       Value threadVal = entryBlock->getArgument(3);
2265:       Value buffers = entryBlock->getArgument(4);
2266:       Value writeVisibilityPtr = entryBlock->getArgument(5);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2267-2268

```cpp
2267:       Value effectCTAs = entryBlock->getArgument(6);
2268:       Value aliasMatrix = useAlias ? entryBlock->getArgument(7) : Value();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2270-2287

```cpp
2270:       Value writeVisibility = tti::createLoadScratchMemory(
2271:           fb, fb.getLoc(), writeVisibilityPtr, writeVisibilityType);
2272:       Value descriptor = createBufferDescriptor(fb, bufOffset, lengthVal);
2273:       Value buffersEqBuf = createCmpIntTensorScalar(fb, buffers, descriptor);
2274:       if (useAlias) {
2275:         buffersEqBuf =
2276:             expandAliases(fb, buffersEqBuf, aliasMatrix,
2277:                           cast<RankedTensorType>(aliasMatrixTypeBase));
2278:       }
2279:       buffersEqBuf =
2280:           convertAndBroadcast(fb, buffersEqBuf, {1}, writeVisibilityType);
2281:       Value relationMask =
2282:           createLeadCTAEffectMask(fb, writeVisibilityType, effectCTAs);
2283:       buffersEqBuf = arith::AndIOp::create(fb, buffersEqBuf, relationMask);
2284:       Value writeVisibilityZero =
2285:           tti::createConstIntTensor(fb, fb.getLoc(), 0, writeVisibilityType);
2286:       Value bufVisibility = arith::SelectOp::create(
2287:           fb, buffersEqBuf, writeVisibility, writeVisibilityZero);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2288-2303

```cpp
2288:       Value noOneIsWriting = arith::CmpIOp::create(
2289:           fb, arith::CmpIPredicate::eq, bufVisibility, writeVisibilityZero);
2290:       Value threadI64 = arith::ExtUIOp::create(fb, fb.getI64Type(), threadVal);
2291:       Value threadMask =
2292:           triton::SplatOp::create(fb, writeVisibilityType, threadI64);
2293:       Value buffersEqBufExt =
2294:           arith::ExtUIOp::create(fb, writeVisibilityType, buffersEqBuf);
2295:       Value bufferThreadBit =
2296:           arith::ShLIOp::create(fb, buffersEqBufExt, threadMask);
2297:       Value bufferHasVisibility =
2298:           arith::AndIOp::create(fb, bufVisibility, bufferThreadBit);
2299:       bufferHasVisibility = arith::CmpIOp::create(
2300:           fb, arith::CmpIPredicate::eq, bufferHasVisibility, bufferThreadBit);
2301:       Value writeVisible =
2302:           arith::OrIOp::create(fb, noOneIsWriting, bufferHasVisibility);
2303:       Value allWritesVisible = reduceAll<arith::AndIOp>(fb, writeVisible);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2305-2320

```cpp
2305:       Value vTrue = arith::ConstantOp::create(
2306:           fb, allWritesVisible.getType(), fb.getIntegerAttr(fb.getI1Type(), 1));
2307:       Value predicatedWriteVisible =
2308:           arith::SelectOp::create(fb, pred, allWritesVisible, vTrue);
2309:       triton::ReturnOp::create(fb, predicatedWriteVisible);
2310:     };
2311:   };
2312:   if (auxData.hasNonTrivialAliasing[(int)memType]) {
2313:     Value aliasMatrixVal =
2314:         auxData.aliasMatrices[(int)memType].at(insertPoint).value;
2315:     aliasMatrixTypeBase =
2316:         auxData.aliasMatrices[(int)memType].at(insertPoint).type;
2317:     auto aliasMatrixType = cast<RankedTensorType>(aliasMatrixTypeBase);
2318:     SmallVector<Value> args = {bufOffset,  lengthVal,     pred,
2319:                                threadVal,  buffersVal,    writeVisibilityVal,
2320:                                effectCTAs, aliasMatrixVal};
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2321-2334

```cpp
2321:     createCallToCachedFunction(
2322:         b, "verify_write_visibility", args, assertInfo,
2323:         {buffersType, writeVisibilityType, aliasMatrixType, (uint64_t)memType},
2324:         buildVerifyWriteBody(/*useAlias=*/true));
2325:   } else {
2326:     SmallVector<Value> args = {bufOffset, lengthVal,  pred,
2327:                                threadVal, buffersVal, writeVisibilityVal,
2328:                                effectCTAs};
2329:     createCallToCachedFunction(
2330:         b, "verify_write_visibility_noalias", args, assertInfo,
2331:         {buffersType, writeVisibilityType, (uint64_t)memType},
2332:         buildVerifyWriteBody(/*useAlias=*/false));
2333:   }
2334: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2336-2353

```cpp
2336: void FunctionBuilder::createVerifyReadVisibilityCall(
2337:     ImplicitLocOpBuilder &b, Value buf, uint32_t length, int thread,
2338:     StringRef operandName, Value pred, MemType memType, Operation *insertPoint,
2339:     Value effectCTAs) {
2340:   if (auxData.buffers[(int)memType].empty() ||
2341:       auxData.readVisibility[(int)memType].empty() ||
2342:       (auxData.hasNonTrivialAliasing[(int)memType] &&
2343:        auxData.aliasMatrices[(int)memType].empty())) {
2344:     return;
2345:   }
2346:   if (!pred)
2347:     pred = arith::ConstantIntOp::create(b, 1, 1);
2348:   Value threadVal = arith::ConstantIntOp::create(b, thread, 32);
2349:   Value buffersVal = auxData.buffers[(int)memType].at(insertPoint).value;
2350:   auto buffersType = cast<RankedTensorType>(
2351:       auxData.buffers[(int)memType].at(insertPoint).type);
2352:   Value readVisibilityVal =
2353:       auxData.readVisibility[(int)memType].at(insertPoint).value;
```

- **EN:** Defines helper `FunctionBuilder::createVerifyReadVisibilityCall` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `FunctionBuilder::createVerifyReadVisibilityCall`，用于计算或构造外围变换所需的中间数据。
### Lines 2354-2371

```cpp
2354:   auto readVisibilityType = cast<RankedTensorType>(
2355:       auxData.readVisibility[(int)memType].at(insertPoint).type);
2356:   Value bufOffset = tti::ExperimentalMemDescToI32Op::create(b, buf);
2357:   Value lengthVal = arith::ConstantIntOp::create(b, length, 32);
2358:   std::string message = "Buffer being accessed has outstanding reads";
2359:   if (!operandName.empty())
2360:     message += ". Operand: " + operandName.str();
2361:   AssertInfo assertInfo{message, b.getI1Type()};
2362:   Type aliasMatrixTypeBase;
2363:   auto buildVerifyReadBody = [&readVisibilityType,
2364:                               &aliasMatrixTypeBase](bool useAlias) {
2365:     return [=](ImplicitLocOpBuilder &fb, Block *entryBlock) {
2366:       Value bufOffset = entryBlock->getArgument(0);
2367:       Value lengthVal = entryBlock->getArgument(1);
2368:       Value pred = entryBlock->getArgument(2);
2369:       Value threadVal = entryBlock->getArgument(3);
2370:       Value buffers = entryBlock->getArgument(4);
2371:       Value readVisibilityPtr = entryBlock->getArgument(5);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2372-2373

```cpp
2372:       Value effectCTAs = entryBlock->getArgument(6);
2373:       Value aliasMatrix = useAlias ? entryBlock->getArgument(7) : Value();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2375-2392

```cpp
2375:       Value readVisibility = tti::createLoadScratchMemory(
2376:           fb, fb.getLoc(), readVisibilityPtr, readVisibilityType);
2377:       Value descriptor = createBufferDescriptor(fb, bufOffset, lengthVal);
2378:       Value buffersEqBuf = createCmpIntTensorScalar(fb, buffers, descriptor);
2379:       if (useAlias) {
2380:         buffersEqBuf =
2381:             expandAliases(fb, buffersEqBuf, aliasMatrix,
2382:                           cast<RankedTensorType>(aliasMatrixTypeBase));
2383:       }
2384:       buffersEqBuf =
2385:           convertAndBroadcast(fb, buffersEqBuf, {1}, readVisibilityType);
2386:       Value bufferCTAMask =
2387:           createCTASetMask(fb, readVisibilityType, /*dim=*/0, effectCTAs);
2388:       Value relationMask =
2389:           createLeadCTAEffectMask(fb, readVisibilityType, effectCTAs);
2390:       buffersEqBuf = arith::AndIOp::create(fb, buffersEqBuf, bufferCTAMask);
2391:       Value readVisibilityZero =
2392:           tti::createConstIntTensor(fb, fb.getLoc(), 0, readVisibilityType);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2393-2408

```cpp
2393:       Value bufVisibility = arith::SelectOp::create(
2394:           fb, buffersEqBuf, readVisibility, readVisibilityZero);
2395:       Value totalVisibility = reduce<arith::OrIOp>(fb, bufVisibility, {2, 3});
2396:       Value threadColumnMask =
2397:           createDimMask(fb, threadVal, readVisibilityType, /*dim=*/3);
2398:       Value accessorVisibility = arith::SelectOp::create(
2399:           fb, relationMask, bufVisibility, readVisibilityZero);
2400:       accessorVisibility = arith::SelectOp::create(
2401:           fb, threadColumnMask, accessorVisibility, readVisibilityZero);
2402:       accessorVisibility = reduce<arith::OrIOp>(fb, accessorVisibility, {3});
2403:       auto accessorVisibilityType =
2404:           cast<RankedTensorType>(accessorVisibility.getType());
2405:       totalVisibility = convertAndBroadcast(fb, totalVisibility, {0, 1, 3},
2406:                                             accessorVisibilityType);
2407:       Value threadAndTotalVisibility =
2408:           arith::AndIOp::create(fb, accessorVisibility, totalVisibility);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2409-2426

```cpp
2409:       Value hasVisibility =
2410:           arith::CmpIOp::create(fb, arith::CmpIPredicate::eq,
2411:                                 threadAndTotalVisibility, totalVisibility);
2412:       Value selectedAccessors =
2413:           arith::AndIOp::create(fb, buffersEqBuf, relationMask);
2414:       selectedAccessors = reduce<arith::OrIOp>(fb, selectedAccessors, {3, 4});
2415:       selectedAccessors = convertAndBroadcast(fb, selectedAccessors, {0, 1, 2},
2416:                                               accessorVisibilityType);
2417:       Value one = tti::createConstIntTensor(
2418:           fb, fb.getLoc(), 1,
2419:           cast<RankedTensorType>(selectedAccessors.getType()));
2420:       Value unmatchedAccessors =
2421:           arith::XOrIOp::create(fb, selectedAccessors, one);
2422:       hasVisibility =
2423:           arith::OrIOp::create(fb, hasVisibility, unmatchedAccessors);
2424:       hasVisibility = reduceAll<arith::AndIOp>(fb, hasVisibility);
2425:       Value vTrue = arith::ConstantOp::create(
2426:           fb, hasVisibility.getType(), fb.getIntegerAttr(fb.getI1Type(), 1));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2427-2444

```cpp
2427:       Value predicatedHasVisibility =
2428:           arith::SelectOp::create(fb, pred, hasVisibility, vTrue);
2429:       triton::ReturnOp::create(fb, predicatedHasVisibility);
2430:     };
2431:   };
2432:   if (auxData.hasNonTrivialAliasing[(int)memType]) {
2433:     Value aliasMatrixVal =
2434:         auxData.aliasMatrices[(int)memType].at(insertPoint).value;
2435:     aliasMatrixTypeBase =
2436:         auxData.aliasMatrices[(int)memType].at(insertPoint).type;
2437:     auto aliasMatrixType = cast<RankedTensorType>(aliasMatrixTypeBase);
2438:     SmallVector<Value> args = {bufOffset,  lengthVal,     pred,
2439:                                threadVal,  buffersVal,    readVisibilityVal,
2440:                                effectCTAs, aliasMatrixVal};
2441:     createCallToCachedFunction(
2442:         b, "verify_read_visibility", args, assertInfo,
2443:         {buffersType, readVisibilityType, aliasMatrixType, (uint64_t)memType},
2444:         buildVerifyReadBody(/*useAlias=*/true));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2445-2454

```cpp
2445:   } else {
2446:     SmallVector<Value> args = {bufOffset, lengthVal,  pred,
2447:                                threadVal, buffersVal, readVisibilityVal,
2448:                                effectCTAs};
2449:     createCallToCachedFunction(
2450:         b, "verify_read_visibility_noalias", args, assertInfo,
2451:         {buffersType, readVisibilityType, (uint64_t)memType},
2452:         buildVerifyReadBody(/*useAlias=*/false));
2453:   }
2454: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2456-2460

```cpp
2456: void FunctionBuilder::createCopyWriteVisibilityCall(ImplicitLocOpBuilder &b,
2457:                                                     int sourceThread,
2458:                                                     uint64_t destMask,
2459:                                                     Value pred, MemType memType,
2460:                                                     Operation *insertPoint) {
```

- **EN:** Defines helper `FunctionBuilder::createCopyWriteVisibilityCall` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `FunctionBuilder::createCopyWriteVisibilityCall`，用于计算或构造外围变换所需的中间数据。
### Lines 2462-2479

```cpp
2462:   if (auxData.writeVisibility[(int)memType].empty()) {
2463:     return;
2464:   }
2465:   if (!pred)
2466:     pred = arith::ConstantIntOp::create(b, 1, 1);
2467:   auto writeVis = auxData.writeVisibility[(int)memType].at(insertPoint);
2468:   auto writeVisibilityType = cast<RankedTensorType>(writeVis.type);
2469:   Value sourceThreadVal = arith::ConstantIntOp::create(b, sourceThread, 32);
2470:   Value destMaskVal = arith::ConstantIntOp::create(b, destMask, 64);
2471:   SmallVector<Value> args = {sourceThreadVal, destMaskVal, pred,
2472:                              writeVis.value};
2473:   createCallToCachedFunction(
2474:       b, "copy_write_visibility", args,
2475:       /*assertInfo=*/std::nullopt, {writeVisibilityType, (uint64_t)memType},
2476:       [writeVisibilityType,
2477:        totalNumThreads = auxData.threadLayout.totalNumThreads](
2478:           ImplicitLocOpBuilder &fb, Block *entryBlock) {
2479:         Value sourceThread = entryBlock->getArgument(0);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2480-2482

```cpp
2480:         Value destMaskVal = entryBlock->getArgument(1);
2481:         Value pred = entryBlock->getArgument(2);
2482:         Value writeVisibilityPtr = entryBlock->getArgument(3);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2484-2485

```cpp
2484:         auto [prevBlock, ifBlock, thenBlock] = createIfBlock(fb, pred);
2485:         fb.setInsertionPointToStart(ifBlock);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2487-2491

```cpp
2487:         Value writeVisibility = tti::createLoadScratchMemory(
2488:             fb, fb.getLoc(), writeVisibilityPtr, writeVisibilityType);
2489:         auto elemType = cast<IntegerType>(writeVisibilityType.getElementType());
2490:         Value zeroTensor =
2491:             tti::createConstIntTensor(fb, fb.getLoc(), 0, writeVisibilityType);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2493-2507

```cpp
2493:         uint64_t fullMask = totalNumThreads == 64
2494:                                 ? std::numeric_limits<uint64_t>::max()
2495:                                 : (std::numeric_limits<uint64_t>::max() >>
2496:                                    (64 - totalNumThreads));
2497:         Value fullMaskVal = arith::ConstantIntOp::create(fb, fullMask, 64);
2498:         Value destMaskElem = adjustIntegerWidth(fb, destMaskVal, elemType);
2499:         Value fullMaskElem = adjustIntegerWidth(fb, fullMaskVal, elemType);
2500:         Value clearMaskElem =
2501:             arith::XOrIOp::create(fb, destMaskElem, fullMaskElem);
2502:         Value destMaskTensor =
2503:             triton::SplatOp::create(fb, writeVisibilityType, destMaskElem);
2504:         Value clearMaskTensor =
2505:             triton::SplatOp::create(fb, writeVisibilityType, clearMaskElem);
2506:         Value cleared =
2507:             arith::AndIOp::create(fb, writeVisibility, clearMaskTensor);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2509-2521

```cpp
2509:         Value sourceThreadElem = adjustIntegerWidth(fb, sourceThread, elemType);
2510:         Value oneScalar = arith::ConstantOp::create(
2511:             fb, elemType, fb.getIntegerAttr(elemType, 1));
2512:         Value sourceMaskElem =
2513:             arith::ShLIOp::create(fb, oneScalar, sourceThreadElem);
2514:         Value sourceMaskTensor =
2515:             triton::SplatOp::create(fb, writeVisibilityType, sourceMaskElem);
2516:         Value sourceBits =
2517:             arith::AndIOp::create(fb, writeVisibility, sourceMaskTensor);
2518:         Value sourceIsSet = arith::CmpIOp::create(fb, arith::CmpIPredicate::ne,
2519:                                                   sourceBits, zeroTensor);
2520:         Value replicated = arith::SelectOp::create(fb, sourceIsSet,
2521:                                                    destMaskTensor, zeroTensor);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2523-2528

```cpp
2523:         Value updatedCurrent = arith::OrIOp::create(fb, cleared, replicated);
2524:         Value currentCTAMask = createCTASetMask(
2525:             fb, writeVisibilityType, /*dim=*/2, createCurrentCTAMask(fb));
2526:         createMaskedStoreScratchMemory(fb, fb.getLoc(), writeVisibilityPtr,
2527:                                        updatedCurrent, writeVisibilityType,
2528:                                        currentCTAMask);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2530-2533

```cpp
2530:         fb.setInsertionPointToEnd(thenBlock);
2531:         triton::ReturnOp::create(fb);
2532:       });
2533: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2535-2539

```cpp
2535: void FunctionBuilder::createCopyReadVisibilityCall(ImplicitLocOpBuilder &b,
2536:                                                    int sourceThread,
2537:                                                    uint64_t destMask,
2538:                                                    Value pred, MemType memType,
2539:                                                    Operation *insertPoint) {
```

- **EN:** Defines helper `FunctionBuilder::createCopyReadVisibilityCall` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `FunctionBuilder::createCopyReadVisibilityCall`，用于计算或构造外围变换所需的中间数据。
### Lines 2541-2558

```cpp
2541:   if (auxData.readVisibility[(int)memType].empty()) {
2542:     return;
2543:   }
2544:   if (!pred)
2545:     pred = arith::ConstantIntOp::create(b, 1, 1);
2546:   auto readVis = auxData.readVisibility[(int)memType].at(insertPoint);
2547:   auto readVisibilityType = cast<RankedTensorType>(readVis.type);
2548:   Value sourceThreadVal = arith::ConstantIntOp::create(b, sourceThread, 32);
2549:   SmallVector<Value> args = {sourceThreadVal, pred, readVis.value};
2550:   createCallToCachedFunction(
2551:       b, "copy_read_visibility", args,
2552:       /*assertInfo=*/std::nullopt,
2553:       {readVisibilityType, destMask, (uint64_t)memType},
2554:       [readVisibilityType, destMask](ImplicitLocOpBuilder &fb,
2555:                                      Block *entryBlock) {
2556:         Value sourceThread = entryBlock->getArgument(0);
2557:         Value pred = entryBlock->getArgument(1);
2558:         Value readVisibilityPtr = entryBlock->getArgument(2);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2560-2561

```cpp
2560:         auto [prevBlock, ifBlock, thenBlock] = createIfBlock(fb, pred);
2561:         fb.setInsertionPointToStart(ifBlock);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2563-2571

```cpp
2563:         Value readVisibility = tti::createLoadScratchMemory(
2564:             fb, fb.getLoc(), readVisibilityPtr, readVisibilityType);
2565:         Value zeroTensor =
2566:             tti::createConstIntTensor(fb, fb.getLoc(), 0, readVisibilityType);
2567:         Value destMaskTensor = createThreadColumnMask(
2568:             fb, arith::ConstantIntOp::create(fb, destMask, 64),
2569:             readVisibilityType, /*columnDim=*/3);
2570:         Value cleared = arith::SelectOp::create(fb, destMaskTensor, zeroTensor,
2571:                                                 readVisibility);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2573-2581

```cpp
2573:         Value sourceColumnMask =
2574:             createDimMask(fb, sourceThread, readVisibilityType, /*dim=*/3);
2575:         Value sourceColumn = arith::SelectOp::create(
2576:             fb, sourceColumnMask, readVisibility, zeroTensor);
2577:         Value sourceVector = reduce<arith::OrIOp>(fb, sourceColumn, {3});
2578:         Value broadcastRow = convertAndBroadcast(fb, sourceVector, {0, 1, 2, 4},
2579:                                                  readVisibilityType);
2580:         Value replicated = arith::SelectOp::create(fb, destMaskTensor,
2581:                                                    broadcastRow, zeroTensor);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2583-2588

```cpp
2583:         Value updated = arith::OrIOp::create(fb, cleared, replicated);
2584:         Value currentCTAMask = createCTASetMask(
2585:             fb, readVisibilityType, /*dim=*/2, createCurrentCTAMask(fb));
2586:         createMaskedStoreScratchMemory(fb, fb.getLoc(), readVisibilityPtr,
2587:                                        updated, readVisibilityType,
2588:                                        currentCTAMask);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2590-2593

```cpp
2590:         fb.setInsertionPointToEnd(thenBlock);
2591:         triton::ReturnOp::create(fb);
2592:       });
2593: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2595-2608

```cpp
2595: void FunctionBuilder::createPublishClusterVisibilityCall(
2596:     ImplicitLocOpBuilder &b, Value pred, MemType memType,
2597:     Operation *insertPoint) {
2598:   if (auxData.writeVisibility[(int)memType].empty() ||
2599:       auxData.readVisibility[(int)memType].empty()) {
2600:     return;
2601:   }
2602:   if (!pred)
2603:     pred = arith::ConstantIntOp::create(b, 1, 1);
2604:   auto writeVis = auxData.writeVisibility[(int)memType].at(insertPoint);
2605:   auto readVis = auxData.readVisibility[(int)memType].at(insertPoint);
2606:   auto writeVisibilityType = cast<RankedTensorType>(writeVis.type);
2607:   auto readVisibilityType = cast<RankedTensorType>(readVis.type);
2608:   SmallVector<Value> args = {pred, writeVis.value, readVis.value};
```

- **EN:** Defines helper `FunctionBuilder::createPublishClusterVisibilityCall` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `FunctionBuilder::createPublishClusterVisibilityCall`，用于计算或构造外围变换所需的中间数据。
### Lines 2609-2618

```cpp
2609:   createCallToCachedFunction(
2610:       b, "publish_cluster_visibility", args,
2611:       /*assertInfo=*/std::nullopt,
2612:       {writeVisibilityType, readVisibilityType, (uint64_t)memType},
2613:       [writeVisibilityType, readVisibilityType,
2614:        numBaseThreads = auxData.threadLayout.numBaseThreads](
2615:           ImplicitLocOpBuilder &fb, Block *entryBlock) {
2616:         Value pred = entryBlock->getArgument(0);
2617:         Value writeVisibilityPtr = entryBlock->getArgument(1);
2618:         Value readVisibilityPtr = entryBlock->getArgument(2);
```

- **EN:** Defines helper `createCallToCachedFunction` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createCallToCachedFunction`，用于计算或构造外围变换所需的中间数据。
### Lines 2620-2621

```cpp
2620:         auto [prevBlock, ifBlock, thenBlock] = createIfBlock(fb, pred);
2621:         fb.setInsertionPointToStart(ifBlock);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2623-2626

```cpp
2623:         Value writeVisibility = tti::createLoadScratchMemory(
2624:             fb, fb.getLoc(), writeVisibilityPtr, writeVisibilityType);
2625:         Value readVisibility = tti::createLoadScratchMemory(
2626:             fb, fb.getLoc(), readVisibilityPtr, readVisibilityType);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2628-2645

```cpp
2628:         // Cluster barriers publish generic-proxy synchronous work. Base-thread
2629:         // visibility distinguishes those facts from async-only TMA/TC/CLC
2630:         // effects, which are published by their own completion path.
2631:         uint64_t baseThreadMask = (1ULL << numBaseThreads) - 1;
2632:         Value baseMask = tti::createConstIntTensor(
2633:             fb, fb.getLoc(), baseThreadMask, writeVisibilityType);
2634:         Value zeroWrites =
2635:             tti::createConstIntTensor(fb, fb.getLoc(), 0, writeVisibilityType);
2636:         Value hasBaseWrite = arith::CmpIOp::create(
2637:             fb, arith::CmpIPredicate::ne,
2638:             arith::AndIOp::create(fb, writeVisibility, baseMask), zeroWrites);
2639:         Value syncWrites = arith::SelectOp::create(fb, hasBaseWrite,
2640:                                                    writeVisibility, zeroWrites);
2641:         Value writesForCluster = reduce<arith::OrIOp>(fb, syncWrites, {2});
2642:         writesForCluster = convertAndBroadcast(fb, writesForCluster, {0, 1},
2643:                                                writeVisibilityType);
2644:         Value newWriteVisibility =
2645:             arith::OrIOp::create(fb, writeVisibility, writesForCluster);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2646-2648

```cpp
2646:         tti::createStoreScratchMemory(fb, fb.getLoc(), writeVisibilityPtr,
2647:                                       newWriteVisibility, writeVisibilityType,
2648:                                       /*currentCTAOnly=*/false);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2650-2666

```cpp
2650:         Value readBaseMask = tti::createConstIntTensor(
2651:             fb, fb.getLoc(), baseThreadMask, readVisibilityType);
2652:         Value zeroReads =
2653:             tti::createConstIntTensor(fb, fb.getLoc(), 0, readVisibilityType);
2654:         Value hasBaseRead = arith::CmpIOp::create(
2655:             fb, arith::CmpIPredicate::ne,
2656:             arith::AndIOp::create(fb, readVisibility, readBaseMask), zeroReads);
2657:         Value syncReads =
2658:             arith::SelectOp::create(fb, hasBaseRead, readVisibility, zeroReads);
2659:         Value readsForCluster = reduce<arith::OrIOp>(fb, syncReads, {2, 3, 4});
2660:         readsForCluster = convertAndBroadcast(fb, readsForCluster, {0, 1},
2661:                                               readVisibilityType);
2662:         Value newReadVisibility =
2663:             arith::OrIOp::create(fb, readVisibility, readsForCluster);
2664:         tti::createStoreScratchMemory(fb, fb.getLoc(), readVisibilityPtr,
2665:                                       newReadVisibility, readVisibilityType,
2666:                                       /*currentCTAOnly=*/false);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2668-2671

```cpp
2668:         fb.setInsertionPointToEnd(thenBlock);
2669:         triton::ReturnOp::create(fb);
2670:       });
2671: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2673-2688

```cpp
2673: void FunctionBuilder::createStageAccessForCommitCall(
2674:     ImplicitLocOpBuilder &b, Value buf, uint32_t length, int thread, Value pred,
2675:     MemType memType, CommitKind::Kind commitKind, Operation *insertPoint) {
2676:   if (auxData.buffers[(int)memType].empty() ||
2677:       auxData.commits[commitKind].empty()) {
2678:     return;
2679:   }
2680:   if (!pred)
2681:     pred = arith::ConstantIntOp::create(b, 1, 1);
2682:   ValueType buffers = auxData.buffers[(int)memType].at(insertPoint);
2683:   ValueType outstandingCommits = auxData.commits[commitKind].at(insertPoint);
2684:   auto buffersType = cast<RankedTensorType>(buffers.type);
2685:   auto commitsType = cast<RankedTensorType>(outstandingCommits.type);
2686:   Value threadVal = arith::ConstantIntOp::create(b, thread, 32);
2687:   Value bufOffset = tti::ExperimentalMemDescToI32Op::create(b, buf);
2688:   Value lengthVal = arith::ConstantIntOp::create(b, length, 32);
```

- **EN:** Defines helper `FunctionBuilder::createStageAccessForCommitCall` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `FunctionBuilder::createStageAccessForCommitCall`，用于计算或构造外围变换所需的中间数据。
### Lines 2689-2701

```cpp
2689:   SmallVector<Value> args = {bufOffset,     lengthVal,
2690:                              pred,          threadVal,
2691:                              buffers.value, outstandingCommits.value};
2692:   createCallToCachedFunction(
2693:       b, "stage_access_for_commit", args,
2694:       /*assertInfo=*/std::nullopt, {buffersType, commitsType},
2695:       [commitsType](ImplicitLocOpBuilder &fb, Block *entryBlock) {
2696:         Value bufOffset = entryBlock->getArgument(0);
2697:         Value lengthVal = entryBlock->getArgument(1);
2698:         Value pred = entryBlock->getArgument(2);
2699:         Value threadVal = entryBlock->getArgument(3);
2700:         Value buffers = entryBlock->getArgument(4);
2701:         Value outstandingCommitsPtr = entryBlock->getArgument(5);
```

- **EN:** Defines helper `createCallToCachedFunction` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createCallToCachedFunction`，用于计算或构造外围变换所需的中间数据。
### Lines 2703-2704

```cpp
2703:         auto [prevBlock, ifBlock, thenBlock] = createIfBlock(fb, pred);
2704:         fb.setInsertionPointToStart(ifBlock);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2706-2723

```cpp
2706:         Value commits = tti::createLoadScratchMemory(
2707:             fb, fb.getLoc(), outstandingCommitsPtr, commitsType);
2708:         Value descriptor = createBufferDescriptor(fb, bufOffset, lengthVal);
2709:         Value buffersEqBuf = createCmpIntTensorScalar(fb, buffers, descriptor);
2710:         buffersEqBuf = convertAndBroadcast(fb, buffersEqBuf, {1}, commitsType);
2711:         Value ctaMask = createCTASetMask(fb, commitsType, /*dim=*/0,
2712:                                          createCurrentCTAMask(fb));
2713:         buffersEqBuf = arith::AndIOp::create(fb, buffersEqBuf, ctaMask);
2714:         Value threadColumnMask =
2715:             createDimMask(fb, threadVal, commitsType, /*dim=*/2);
2716:         Value bufAndThread =
2717:             arith::AndIOp::create(fb, buffersEqBuf, threadColumnMask);
2718:         Value minusOne =
2719:             tti::createConstIntTensor(fb, fb.getLoc(), -1, commitsType, true);
2720:         Value updated =
2721:             arith::SelectOp::create(fb, bufAndThread, minusOne, commits);
2722:         createMaskedStoreScratchMemory(fb, fb.getLoc(), outstandingCommitsPtr,
2723:                                        updated, commitsType, ctaMask);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2725-2728

```cpp
2725:         fb.setInsertionPointToEnd(thenBlock);
2726:         triton::ReturnOp::create(fb);
2727:       });
2728: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2730-2747

```cpp
2730: void FunctionBuilder::createCommitAccessesCall(ImplicitLocOpBuilder &b,
2731:                                                int thread, Value pred,
2732:                                                CommitKind::Kind commitKind,
2733:                                                Operation *insertPoint) {
2734:   if (auxData.commits[commitKind].empty()) {
2735:     return;
2736:   }
2737:   if (!pred)
2738:     pred = arith::ConstantIntOp::create(b, 1, 1);
2739:   ValueType outstandingCommits = auxData.commits[commitKind].at(insertPoint);
2740:   auto commitsType = cast<RankedTensorType>(outstandingCommits.type);
2741:   Value threadVal = arith::ConstantIntOp::create(b, thread, 32);
2742:   SmallVector<Value> args = {threadVal, pred, outstandingCommits.value};
2743:   createCallToCachedFunction(
2744:       b, "commit_accesses", args,
2745:       /*assertInfo=*/std::nullopt, {commitsType},
2746:       [commitsType](ImplicitLocOpBuilder &fb, Block *entryBlock) {
2747:         Value threadVal = entryBlock->getArgument(0);
```

- **EN:** Defines helper `FunctionBuilder::createCommitAccessesCall` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `FunctionBuilder::createCommitAccessesCall`，用于计算或构造外围变换所需的中间数据。
### Lines 2748-2749

```cpp
2748:         Value pred = entryBlock->getArgument(1);
2749:         Value outstandingCommitsPtr = entryBlock->getArgument(2);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2751-2752

```cpp
2751:         auto [prevBlock, ifBlock, thenBlock] = createIfBlock(fb, pred);
2752:         fb.setInsertionPointToStart(ifBlock);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2754-2761

```cpp
2754:         Value commits = tti::createLoadScratchMemory(
2755:             fb, fb.getLoc(), outstandingCommitsPtr, commitsType);
2756:         Type elementType = commitsType.getElementType();
2757:         Value zero = arith::ConstantOp::create(
2758:             fb, elementType, fb.getIntegerAttr(elementType, 0));
2759:         Value minusOne = arith::ConstantOp::create(
2760:             fb, elementType, fb.getIntegerAttr(elementType, -1));
2761:         Value ones = tti::createConstIntTensor(fb, fb.getLoc(), 1, commitsType);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2763-2772

```cpp
2763:         Value threadMask = createDimMask(fb, threadVal, commitsType, /*dim=*/2);
2764:         Value ctaMask = createCTASetMask(fb, commitsType, /*dim=*/0,
2765:                                          createCurrentCTAMask(fb));
2766:         threadMask = arith::AndIOp::create(fb, threadMask, ctaMask);
2767:         auto commitsGtZero = createCmpIntTensorScalar(
2768:             fb, commits, zero, arith::CmpIPredicate::sgt);
2769:         commitsGtZero = arith::AndIOp::create(fb, commitsGtZero, threadMask);
2770:         Value commitsPlusOne = arith::AddIOp::create(fb, commits, ones);
2771:         commits =
2772:             arith::SelectOp::create(fb, commitsGtZero, commitsPlusOne, commits);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2774-2778

```cpp
2774:         auto commitsEqMinusOne = createCmpIntTensorScalar(
2775:             fb, commits, minusOne, arith::CmpIPredicate::eq);
2776:         commitsEqMinusOne =
2777:             arith::AndIOp::create(fb, commitsEqMinusOne, threadMask);
2778:         commits = arith::SelectOp::create(fb, commitsEqMinusOne, ones, commits);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2780-2781

```cpp
2780:         createMaskedStoreScratchMemory(fb, fb.getLoc(), outstandingCommitsPtr,
2781:                                        commits, commitsType, ctaMask);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2783-2786

```cpp
2783:         fb.setInsertionPointToEnd(thenBlock);
2784:         triton::ReturnOp::create(fb);
2785:       });
2786: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2788-2805

```cpp
2788: void FunctionBuilder::createClearOutstandingCommitsTransferWritesCall(
2789:     ImplicitLocOpBuilder &b, int thread, uint64_t transferThreadMask,
2790:     int outstandingNum, Value pred, CommitKind::Kind commitKind,
2791:     MemType memType, Operation *insertPoint) {
2792:   if (auxData.commits[commitKind].empty() ||
2793:       auxData.writeVisibility[(int)memType].empty()) {
2794:     return;
2795:   }
2796:   if (!pred)
2797:     pred = arith::ConstantIntOp::create(b, 1, 1);
2798:   ValueType outstandingCommits = auxData.commits[commitKind].at(insertPoint);
2799:   ValueType writeVisibility =
2800:       auxData.writeVisibility[(int)memType].at(insertPoint);
2801:   auto commitsType = cast<RankedTensorType>(outstandingCommits.type);
2802:   auto writeVisibilityType = cast<RankedTensorType>(writeVisibility.type);
2803:   Value threadVal = arith::ConstantIntOp::create(b, thread, 32);
2804:   Value transferMaskVal =
2805:       arith::ConstantIntOp::create(b, transferThreadMask, 64);
```

- **EN:** Defines helper `FunctionBuilder::createClearOutstandingCommitsTransferWritesCall` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `FunctionBuilder::createClearOutstandingCommitsTransferWritesCall`，用于计算或构造外围变换所需的中间数据。
### Lines 2806-2820

```cpp
2806:   Value outstandingNumVal = arith::ConstantIntOp::create(b, outstandingNum, 32);
2807:   SmallVector<Value> args = {
2808:       threadVal, transferMaskVal,          outstandingNumVal,
2809:       pred,      outstandingCommits.value, writeVisibility.value};
2810:   createCallToCachedFunction(
2811:       b, "clear_outstanding_commits_transfer_writes", args,
2812:       /*assertInfo=*/std::nullopt, {commitsType, writeVisibilityType},
2813:       [commitsType, writeVisibilityType](ImplicitLocOpBuilder &fb,
2814:                                          Block *entryBlock) {
2815:         Value threadVal = entryBlock->getArgument(0);
2816:         Value transferMaskVal = entryBlock->getArgument(1);
2817:         Value outstandingNumVal = entryBlock->getArgument(2);
2818:         Value pred = entryBlock->getArgument(3);
2819:         Value outstandingCommitsPtr = entryBlock->getArgument(4);
2820:         Value writeVisibilityPtr = entryBlock->getArgument(5);
```

- **EN:** Defines helper `createCallToCachedFunction` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createCallToCachedFunction`，用于计算或构造外围变换所需的中间数据。
### Lines 2822-2823

```cpp
2822:         auto [prevBlock, ifBlock, thenBlock] = createIfBlock(fb, pred);
2823:         fb.setInsertionPointToStart(ifBlock);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2825-2828

```cpp
2825:         Value outstandingCommits = tti::createLoadScratchMemory(
2826:             fb, fb.getLoc(), outstandingCommitsPtr, commitsType);
2827:         Value writeVisibility = tti::createLoadScratchMemory(
2828:             fb, fb.getLoc(), writeVisibilityPtr, writeVisibilityType);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2830-2843

```cpp
2830:         auto elemIntType = cast<IntegerType>(commitsType.getElementType());
2831:         Value outstandingNumElem =
2832:             adjustIntegerWidth(fb, outstandingNumVal, elemIntType);
2833:         Value threadColumnMask =
2834:             createDimMask(fb, threadVal, commitsType, /*dim=*/2);
2835:         Value commitCTAMask = createCTASetMask(fb, commitsType, /*dim=*/0,
2836:                                                createCurrentCTAMask(fb));
2837:         threadColumnMask =
2838:             arith::AndIOp::create(fb, threadColumnMask, commitCTAMask);
2839:         auto outstandingCommitsGtOutstandingNum =
2840:             createCmpIntTensorScalar(fb, outstandingCommits, outstandingNumElem,
2841:                                      arith::CmpIPredicate::sgt);
2842:         outstandingCommitsGtOutstandingNum = arith::AndIOp::create(
2843:             fb, outstandingCommitsGtOutstandingNum, threadColumnMask);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2845-2861

```cpp
2845:         Value rowMask =
2846:             reduceLastDim<arith::OrIOp>(fb, outstandingCommitsGtOutstandingNum);
2847:         rowMask = convertAndBroadcast(fb, rowMask, {0, 1}, writeVisibilityType);
2848:         Value transferMaskElem = adjustIntegerWidth(
2849:             fb, transferMaskVal,
2850:             cast<IntegerType>(writeVisibilityType.getElementType()));
2851:         Value transferMaskTensor =
2852:             triton::SplatOp::create(fb, writeVisibilityType, transferMaskElem);
2853:         Value writeVisibilityOrThreadBit =
2854:             arith::OrIOp::create(fb, writeVisibility, transferMaskTensor);
2855:         Value writeVisibilityUpdated = arith::SelectOp::create(
2856:             fb, rowMask, writeVisibilityOrThreadBit, writeVisibility);
2857:         Value writeMask = createCTASetMask(fb, writeVisibilityType, /*dim=*/2,
2858:                                            createCurrentCTAMask(fb));
2859:         createMaskedStoreScratchMemory(fb, fb.getLoc(), writeVisibilityPtr,
2860:                                        writeVisibilityUpdated,
2861:                                        writeVisibilityType, writeMask);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2863-2870

```cpp
2863:         Value outstandingCommitsZero =
2864:             tti::createConstIntTensor(fb, fb.getLoc(), 0, commitsType);
2865:         outstandingCommits =
2866:             arith::SelectOp::create(fb, outstandingCommitsGtOutstandingNum,
2867:                                     outstandingCommitsZero, outstandingCommits);
2868:         createMaskedStoreScratchMemory(fb, fb.getLoc(), outstandingCommitsPtr,
2869:                                        outstandingCommits, commitsType,
2870:                                        commitCTAMask);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2872-2875

```cpp
2872:         fb.setInsertionPointToEnd(thenBlock);
2873:         triton::ReturnOp::create(fb);
2874:       });
2875: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2877-2894

```cpp
2877: void FunctionBuilder::createClearOutstandingCommitsTransferReadsCall(
2878:     ImplicitLocOpBuilder &b, int thread, uint64_t transferThreadMask,
2879:     int outstandingNum, Value pred, CommitKind::Kind commitKind,
2880:     MemType memType, Operation *insertPoint) {
2881:   if (auxData.commits[commitKind].empty() ||
2882:       auxData.readVisibility[(int)memType].empty()) {
2883:     return;
2884:   }
2885:   ValueType outstandingCommits = auxData.commits[commitKind].at(insertPoint);
2886:   ValueType readVisibility =
2887:       auxData.readVisibility[(int)memType].at(insertPoint);
2888:   if (!pred)
2889:     pred = arith::ConstantIntOp::create(b, 1, 1);
2890:   auto commitsType = cast<RankedTensorType>(outstandingCommits.type);
2891:   auto readVisibilityType = cast<RankedTensorType>(readVisibility.type);
2892:   Value threadVal = arith::ConstantIntOp::create(b, thread, 32);
2893:   Value transferMaskVal =
2894:       arith::ConstantIntOp::create(b, transferThreadMask, 64);
```

- **EN:** Defines helper `FunctionBuilder::createClearOutstandingCommitsTransferReadsCall` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `FunctionBuilder::createClearOutstandingCommitsTransferReadsCall`，用于计算或构造外围变换所需的中间数据。
### Lines 2895-2909

```cpp
2895:   Value outstandingNumVal = arith::ConstantIntOp::create(b, outstandingNum, 32);
2896:   SmallVector<Value> args = {
2897:       threadVal, transferMaskVal,          outstandingNumVal,
2898:       pred,      outstandingCommits.value, readVisibility.value};
2899:   createCallToCachedFunction(
2900:       b, "clear_outstanding_commits_transfer_reads", args,
2901:       /*assertInfo=*/std::nullopt, {commitsType, readVisibilityType},
2902:       [commitsType, readVisibilityType](ImplicitLocOpBuilder &fb,
2903:                                         Block *entryBlock) {
2904:         Value threadVal = entryBlock->getArgument(0);
2905:         Value transferMaskVal = entryBlock->getArgument(1);
2906:         Value outstandingNumVal = entryBlock->getArgument(2);
2907:         Value pred = entryBlock->getArgument(3);
2908:         Value outstandingCommitsPtr = entryBlock->getArgument(4);
2909:         Value readVisibilityPtr = entryBlock->getArgument(5);
```

- **EN:** Defines helper `createCallToCachedFunction` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createCallToCachedFunction`，用于计算或构造外围变换所需的中间数据。
### Lines 2911-2912

```cpp
2911:         auto [prevBlock, ifBlock, thenBlock] = createIfBlock(fb, pred);
2912:         fb.setInsertionPointToStart(ifBlock);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2914-2917

```cpp
2914:         Value outstandingCommits = tti::createLoadScratchMemory(
2915:             fb, fb.getLoc(), outstandingCommitsPtr, commitsType);
2916:         Value readVisibility = tti::createLoadScratchMemory(
2917:             fb, fb.getLoc(), readVisibilityPtr, readVisibilityType);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2919-2932

```cpp
2919:         auto elemIntType = cast<IntegerType>(commitsType.getElementType());
2920:         Value outstandingNumElem =
2921:             adjustIntegerWidth(fb, outstandingNumVal, elemIntType);
2922:         Value threadColumnMask =
2923:             createDimMask(fb, threadVal, commitsType, /*dim=*/2);
2924:         Value commitCTAMask = createCTASetMask(fb, commitsType, /*dim=*/0,
2925:                                                createCurrentCTAMask(fb));
2926:         threadColumnMask =
2927:             arith::AndIOp::create(fb, threadColumnMask, commitCTAMask);
2928:         auto outstandingCommitsGtOutstandingNum =
2929:             createCmpIntTensorScalar(fb, outstandingCommits, outstandingNumElem,
2930:                                      arith::CmpIPredicate::sgt);
2931:         outstandingCommitsGtOutstandingNum = arith::AndIOp::create(
2932:             fb, outstandingCommitsGtOutstandingNum, threadColumnMask);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2934-2950

```cpp
2934:         Value rowMask =
2935:             reduceLastDim<arith::OrIOp>(fb, outstandingCommitsGtOutstandingNum);
2936:         rowMask = convertAndBroadcast(fb, rowMask, {0, 1}, readVisibilityType);
2937:         Value cMask = createCTASetMask(fb, readVisibilityType, /*dim=*/4,
2938:                                        createCurrentCTAMask(fb));
2939:         rowMask = arith::AndIOp::create(fb, rowMask, cMask);
2940:         Value transferMaskElem = adjustIntegerWidth(
2941:             fb, transferMaskVal,
2942:             cast<IntegerType>(readVisibilityType.getElementType()));
2943:         Value transferMaskTensor =
2944:             triton::SplatOp::create(fb, readVisibilityType, transferMaskElem);
2945:         Value readVisibilityOrThreadBit =
2946:             arith::OrIOp::create(fb, readVisibility, transferMaskTensor);
2947:         Value readVisibilityUpdated = arith::SelectOp::create(
2948:             fb, rowMask, readVisibilityOrThreadBit, readVisibility);
2949:         Value readMask = createCTASetMask(fb, readVisibilityType, /*dim=*/2,
2950:                                           createCurrentCTAMask(fb));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2951-2953

```cpp
2951:         createMaskedStoreScratchMemory(fb, fb.getLoc(), readVisibilityPtr,
2952:                                        readVisibilityUpdated,
2953:                                        readVisibilityType, readMask);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2955-2962

```cpp
2955:         Value outstandingCommitsZero =
2956:             tti::createConstIntTensor(fb, fb.getLoc(), 0, commitsType);
2957:         outstandingCommits =
2958:             arith::SelectOp::create(fb, outstandingCommitsGtOutstandingNum,
2959:                                     outstandingCommitsZero, outstandingCommits);
2960:         createMaskedStoreScratchMemory(fb, fb.getLoc(), outstandingCommitsPtr,
2961:                                        outstandingCommits, commitsType,
2962:                                        commitCTAMask);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2964-2967

```cpp
2964:         fb.setInsertionPointToEnd(thenBlock);
2965:         triton::ReturnOp::create(fb);
2966:       });
2967: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 2969-2984

```cpp
2969: void FunctionBuilder::createClearOutstandingCommitsTransferBothCall(
2970:     ImplicitLocOpBuilder &b, int thread, uint64_t transferThreadMask,
2971:     int outstandingNum, Value pred, CommitKind::Kind commitKind,
2972:     MemType memType, Operation *insertPoint) {
2973:   if (auxData.commits[commitKind].empty())
2974:     return;
2975:   bool hasWriteVis = !auxData.writeVisibility[(int)memType].empty();
2976:   bool hasReadVis = !auxData.readVisibility[(int)memType].empty();
2977:   if (!hasWriteVis && !hasReadVis)
2978:     return;
2979:   if (!hasWriteVis) {
2980:     createClearOutstandingCommitsTransferReadsCall(
2981:         b, thread, transferThreadMask, outstandingNum, pred, commitKind,
2982:         memType, insertPoint);
2983:     return;
2984:   }
```

- **EN:** Defines helper `FunctionBuilder::createClearOutstandingCommitsTransferBothCall` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `FunctionBuilder::createClearOutstandingCommitsTransferBothCall`，用于计算或构造外围变换所需的中间数据。
### Lines 2985-3001

```cpp
2985:   if (!hasReadVis) {
2986:     createClearOutstandingCommitsTransferWritesCall(
2987:         b, thread, transferThreadMask, outstandingNum, pred, commitKind,
2988:         memType, insertPoint);
2989:     return;
2990:   }
2991:   if (!pred)
2992:     pred = arith::ConstantIntOp::create(b, 1, 1);
2993:   ValueType outstandingCommits = auxData.commits[commitKind].at(insertPoint);
2994:   ValueType writeVisibility =
2995:       auxData.writeVisibility[(int)memType].at(insertPoint);
2996:   ValueType readVisibility =
2997:       auxData.readVisibility[(int)memType].at(insertPoint);
2998:   auto commitsType = cast<RankedTensorType>(outstandingCommits.type);
2999:   auto writeVisibilityType = cast<RankedTensorType>(writeVisibility.type);
3000:   auto readVisibilityType = cast<RankedTensorType>(readVisibility.type);
3001:   Value threadVal = arith::ConstantIntOp::create(b, thread, 32);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 3002-3019

```cpp
3002:   Value transferMaskVal =
3003:       arith::ConstantIntOp::create(b, transferThreadMask, 64);
3004:   Value outstandingNumVal = arith::ConstantIntOp::create(b, outstandingNum, 32);
3005:   SmallVector<Value> args = {threadVal,
3006:                              transferMaskVal,
3007:                              outstandingNumVal,
3008:                              pred,
3009:                              outstandingCommits.value,
3010:                              writeVisibility.value,
3011:                              readVisibility.value};
3012:   createCallToCachedFunction(
3013:       b, "clear_outstanding_commits_transfer_both", args,
3014:       /*assertInfo=*/std::nullopt,
3015:       {commitsType, writeVisibilityType, readVisibilityType},
3016:       [commitsType, writeVisibilityType,
3017:        readVisibilityType](ImplicitLocOpBuilder &fb, Block *entryBlock) {
3018:         Value threadVal = entryBlock->getArgument(0);
3019:         Value transferMaskVal = entryBlock->getArgument(1);
```

- **EN:** Defines helper `createCallToCachedFunction` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createCallToCachedFunction`，用于计算或构造外围变换所需的中间数据。
### Lines 3020-3024

```cpp
3020:         Value outstandingNumVal = entryBlock->getArgument(2);
3021:         Value pred = entryBlock->getArgument(3);
3022:         Value outstandingCommitsPtr = entryBlock->getArgument(4);
3023:         Value writeVisibilityPtr = entryBlock->getArgument(5);
3024:         Value readVisibilityPtr = entryBlock->getArgument(6);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 3026-3027

```cpp
3026:         auto [prevBlock, ifBlock, thenBlock] = createIfBlock(fb, pred);
3027:         fb.setInsertionPointToStart(ifBlock);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 3029-3034

```cpp
3029:         Value outstandingCommits = tti::createLoadScratchMemory(
3030:             fb, fb.getLoc(), outstandingCommitsPtr, commitsType);
3031:         Value writeVisibility = tti::createLoadScratchMemory(
3032:             fb, fb.getLoc(), writeVisibilityPtr, writeVisibilityType);
3033:         Value readVisibility = tti::createLoadScratchMemory(
3034:             fb, fb.getLoc(), readVisibilityPtr, readVisibilityType);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 3036-3049

```cpp
3036:         auto elemIntType = cast<IntegerType>(commitsType.getElementType());
3037:         Value outstandingNumElem =
3038:             adjustIntegerWidth(fb, outstandingNumVal, elemIntType);
3039:         Value threadColumnMask =
3040:             createDimMask(fb, threadVal, commitsType, /*dim=*/2);
3041:         Value commitCTAMask = createCTASetMask(fb, commitsType, /*dim=*/0,
3042:                                                createCurrentCTAMask(fb));
3043:         threadColumnMask =
3044:             arith::AndIOp::create(fb, threadColumnMask, commitCTAMask);
3045:         auto outstandingCommitsGtOutstandingNum =
3046:             createCmpIntTensorScalar(fb, outstandingCommits, outstandingNumElem,
3047:                                      arith::CmpIPredicate::sgt);
3048:         outstandingCommitsGtOutstandingNum = arith::AndIOp::create(
3049:             fb, outstandingCommitsGtOutstandingNum, threadColumnMask);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 3051-3066

```cpp
3051:         // Update write visibility
3052:         Value writeRowMask =
3053:             reduceLastDim<arith::OrIOp>(fb, outstandingCommitsGtOutstandingNum);
3054:         writeRowMask =
3055:             convertAndBroadcast(fb, writeRowMask, {0, 1}, writeVisibilityType);
3056:         Value writeTransferMaskElem = adjustIntegerWidth(
3057:             fb, transferMaskVal,
3058:             cast<IntegerType>(writeVisibilityType.getElementType()));
3059:         Value writeTransferMaskTensor = triton::SplatOp::create(
3060:             fb, writeVisibilityType, writeTransferMaskElem);
3061:         Value writeVisibilityOrThreadBit =
3062:             arith::OrIOp::create(fb, writeVisibility, writeTransferMaskTensor);
3063:         Value writeVisibilityUpdated = arith::SelectOp::create(
3064:             fb, writeRowMask, writeVisibilityOrThreadBit, writeVisibility);
3065:         Value writeMask = createCTASetMask(fb, writeVisibilityType, /*dim=*/2,
3066:                                            createCurrentCTAMask(fb));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 3067-3069

```cpp
3067:         createMaskedStoreScratchMemory(fb, fb.getLoc(), writeVisibilityPtr,
3068:                                        writeVisibilityUpdated,
3069:                                        writeVisibilityType, writeMask);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 3071-3087

```cpp
3071:         // Update read visibility
3072:         Value readRowMask =
3073:             reduceLastDim<arith::OrIOp>(fb, outstandingCommitsGtOutstandingNum);
3074:         readRowMask =
3075:             convertAndBroadcast(fb, readRowMask, {0, 1}, readVisibilityType);
3076:         Value cMask = createCTASetMask(fb, readVisibilityType, /*dim=*/4,
3077:                                        createCurrentCTAMask(fb));
3078:         readRowMask = arith::AndIOp::create(fb, readRowMask, cMask);
3079:         Value readTransferMaskElem = adjustIntegerWidth(
3080:             fb, transferMaskVal,
3081:             cast<IntegerType>(readVisibilityType.getElementType()));
3082:         Value readTransferMaskTensor = triton::SplatOp::create(
3083:             fb, readVisibilityType, readTransferMaskElem);
3084:         Value readVisibilityOrThreadBit =
3085:             arith::OrIOp::create(fb, readVisibility, readTransferMaskTensor);
3086:         Value readVisibilityUpdated = arith::SelectOp::create(
3087:             fb, readRowMask, readVisibilityOrThreadBit, readVisibility);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 3088-3092

```cpp
3088:         Value readMask = createCTASetMask(fb, readVisibilityType, /*dim=*/2,
3089:                                           createCurrentCTAMask(fb));
3090:         createMaskedStoreScratchMemory(fb, fb.getLoc(), readVisibilityPtr,
3091:                                        readVisibilityUpdated,
3092:                                        readVisibilityType, readMask);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 3094-3102

```cpp
3094:         // Clear outstanding commits once
3095:         Value outstandingCommitsZero =
3096:             tti::createConstIntTensor(fb, fb.getLoc(), 0, commitsType);
3097:         outstandingCommits =
3098:             arith::SelectOp::create(fb, outstandingCommitsGtOutstandingNum,
3099:                                     outstandingCommitsZero, outstandingCommits);
3100:         createMaskedStoreScratchMemory(fb, fb.getLoc(), outstandingCommitsPtr,
3101:                                        outstandingCommits, commitsType,
3102:                                        commitCTAMask);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 3104-3107

```cpp
3104:         fb.setInsertionPointToEnd(thenBlock);
3105:         triton::ReturnOp::create(fb);
3106:       });
3107: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 3109-3126

```cpp
3109: void FunctionBuilder::createCheckOutstandingCommitsCall(
3110:     ImplicitLocOpBuilder &b, Value buf, uint32_t length, int thread,
3111:     StringRef pendingAccessType, Value pred, MemType memType,
3112:     CommitKind::Kind commitKind, Operation *insertPoint, Value effectCTAs,
3113:     bool excludeSelf) {
3114:   if (auxData.buffers[(int)memType].empty() ||
3115:       auxData.commits[commitKind].empty() ||
3116:       (auxData.hasNonTrivialAliasing[(int)memType] &&
3117:        auxData.aliasMatrices[(int)memType].empty())) {
3118:     return;
3119:   }
3120:   ValueType buffers = auxData.buffers[(int)memType].at(insertPoint);
3121:   ValueType outstandingCommits = auxData.commits[commitKind].at(insertPoint);
3122:   assert(thread < auxData.threadLayout.numBaseThreads &&
3123:          "Commit-count tracking must operate on base threads");
3124:   Value bufOffset = tti::ExperimentalMemDescToI32Op::create(b, buf);
3125:   if (!pred)
3126:     pred = arith::ConstantIntOp::create(b, 1, 1);
```

- **EN:** Defines helper `FunctionBuilder::createCheckOutstandingCommitsCall` that computes or constructs intermediate data used by the surrounding transformation. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义辅助函数 `FunctionBuilder::createCheckOutstandingCommitsCall`，用于计算或构造外围变换所需的中间数据。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 3127-3135

```cpp
3127:   auto buffersType = cast<RankedTensorType>(buffers.type);
3128:   auto commitsType = cast<RankedTensorType>(outstandingCommits.type);
3129:   Value threadVal = arith::ConstantIntOp::create(b, thread, 32);
3130:   Value lengthVal = arith::ConstantIntOp::create(b, length, 32);
3131:   std::string message =
3132:       "Accessing buffer with pending access. Pending access type: " +
3133:       pendingAccessType.str();
3134:   AssertInfo assertInfo{message, b.getI1Type()};
3135:   Type aliasMatrixTypeBase;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 3137-3147

```cpp
3137:   auto buildCheckOutstandingCommitsBody = [&commitsType, &aliasMatrixTypeBase](
3138:                                               bool useAlias, bool exclSelf) {
3139:     return [=](ImplicitLocOpBuilder &fb, Block *entryBlock) {
3140:       Value bufOffset = entryBlock->getArgument(0);
3141:       Value lengthVal = entryBlock->getArgument(1);
3142:       Value pred = entryBlock->getArgument(2);
3143:       Value threadVal = entryBlock->getArgument(3);
3144:       Value buffers = entryBlock->getArgument(4);
3145:       Value outstandingCommitsPtr = entryBlock->getArgument(5);
3146:       Value effectCTAs = entryBlock->getArgument(6);
3147:       Value aliasMatrix = useAlias ? entryBlock->getArgument(7) : Value();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 3149-3164

```cpp
3149:       Value outstandingCommits = tti::createLoadScratchMemory(
3150:           fb, fb.getLoc(), outstandingCommitsPtr, commitsType);
3151:       Value descriptor = createBufferDescriptor(fb, bufOffset, lengthVal);
3152:       Value buffersEqBuf = createCmpIntTensorScalar(fb, buffers, descriptor);
3153:       if (useAlias) {
3154:         buffersEqBuf =
3155:             expandAliases(fb, buffersEqBuf, aliasMatrix,
3156:                           cast<RankedTensorType>(aliasMatrixTypeBase));
3157:       }
3158:       buffersEqBuf = convertAndBroadcast(fb, buffersEqBuf, {1}, commitsType);
3159:       Value ctaMask = createCTASetMask(fb, commitsType, /*dim=*/0, effectCTAs);
3160:       buffersEqBuf = arith::AndIOp::create(fb, buffersEqBuf, ctaMask);
3161:       Value zeroTensor =
3162:           tti::createConstIntTensor(fb, fb.getLoc(), 0, commitsType);
3163:       Value selectedRows = arith::SelectOp::create(
3164:           fb, buffersEqBuf, outstandingCommits, zeroTensor);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 3165-3178

```cpp
3165:       if (exclSelf) {
3166:         Value threadColumnMask =
3167:             createDimMask(fb, threadVal, commitsType, /*dim=*/2);
3168:         selectedRows = arith::SelectOp::create(fb, threadColumnMask, zeroTensor,
3169:                                                selectedRows);
3170:       }
3171:       Value selectedEqZero = arith::CmpIOp::create(fb, arith::CmpIPredicate::eq,
3172:                                                    selectedRows, zeroTensor);
3173:       Value allSelectedEqZero = reduceAll<arith::AndIOp>(fb, selectedEqZero);
3174:       Value vTrue =
3175:           arith::ConstantOp::create(fb, allSelectedEqZero.getType(),
3176:                                     fb.getIntegerAttr(fb.getI1Type(), 1));
3177:       Value predicatedSelectedEqZero =
3178:           arith::SelectOp::create(fb, pred, allSelectedEqZero, vTrue);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 3180-3196

```cpp
3180:       triton::ReturnOp::create(fb, predicatedSelectedEqZero);
3181:     };
3182:   };
3183:   if (auxData.hasNonTrivialAliasing[(int)memType]) {
3184:     ValueType aliasMatrix = auxData.aliasMatrices[(int)memType].at(insertPoint);
3185:     aliasMatrixTypeBase = aliasMatrix.type;
3186:     auto aliasMatrixType = cast<RankedTensorType>(aliasMatrixTypeBase);
3187:     SmallVector<Value> args = {bufOffset,     lengthVal,
3188:                                pred,          threadVal,
3189:                                buffers.value, outstandingCommits.value,
3190:                                effectCTAs,    aliasMatrix.value};
3191:     std::string funcName = excludeSelf ? "check_outstanding_commits_excl_self"
3192:                                        : "check_outstanding_commits";
3193:     createCallToCachedFunction(
3194:         b, funcName, args, assertInfo,
3195:         {buffersType, commitsType, aliasMatrixType, (uint64_t)thread},
3196:         buildCheckOutstandingCommitsBody(/*useAlias=*/true, excludeSelf));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 3197-3210

```cpp
3197:   } else {
3198:     SmallVector<Value> args = {bufOffset,     lengthVal,
3199:                                pred,          threadVal,
3200:                                buffers.value, outstandingCommits.value,
3201:                                effectCTAs};
3202:     std::string funcName = excludeSelf
3203:                                ? "check_outstanding_commits_excl_self_noalias"
3204:                                : "check_outstanding_commits_noalias";
3205:     createCallToCachedFunction(
3206:         b, funcName, args, assertInfo,
3207:         {buffersType, commitsType, (uint64_t)thread},
3208:         buildCheckOutstandingCommitsBody(/*useAlias=*/false, excludeSelf));
3209:   }
3210: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 3212-3212

```cpp
3212: } // namespace mlir::triton::instrument
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。

## Key Concepts / 关键概念
- **EN:** The file defines IR semantics for function builder in its dialect layer.
  **CN:** 本文件在方言层为 Function Builder 定义 IR 语义。
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
- **Internal Triton headers / Triton 内部头文件:** `triton/Dialect/TritonInstrument/IR/FunctionBuilder.h`, `triton/Conversion/TritonGPUToLLVM/Utility.h`, `triton/Dialect/TritonGPU/IR/Dialect.h`, `triton/Dialect/TritonGPU/IR/TritonGPUInterfaces.h`, `triton/Dialect/TritonInstrument/IR/Dialect.h`, `triton/Dialect/TritonInstrument/IR/Utility.h`, ... (+1 more)
- **MLIR headers / MLIR 头文件:** `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/ControlFlow/IR/ControlFlowOps.h`, `mlir/IR/Builders.h`, `mlir/IR/ImplicitLocOpBuilder.h`, `mlir/Support/DebugStringHelper.h`
- **LLVM headers / LLVM 头文件:** None
- **Standard/library headers / 标准或通用库头文件:** `cassert`
- **Primary APIs used / 主要 API:** `ModuleOp`, `RankedTensorType`, `OpBuilder`
- **IR role / IR 角色:** The code is consumed by parsers, printers, verifiers, folders, and downstream passes that need stable dialect semantics. / 这些代码会被解析器、打印器、验证器、折叠逻辑以及需要稳定方言语义的下游 pass 使用。
