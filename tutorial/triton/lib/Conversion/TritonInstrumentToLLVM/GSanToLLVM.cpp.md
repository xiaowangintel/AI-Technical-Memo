# GSanToLLVM.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `lib/Conversion/TritonInstrumentToLLVM/GSanToLLVM.cpp`
- **Purpose / 作用:** **EN:** Converts Triton instrumentation logic forglobal sanitizer into LLVM-level form. **CN:** 把与 G San To LLVM 相关的 TritonInstrument 逻辑转换为 LLVM 层表示。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11

```cpp
   1: #include "mlir/Conversion/LLVMCommon/Pattern.h"
   2: #include "mlir/IR/ImplicitLocOpBuilder.h"
   3: #include "mlir/IR/TypeUtilities.h"
   4: #include "third_party/nvidia/include/TritonNVIDIAGPUToLLVM/AtomicPTXBuilder.h"
   5: #include "third_party/nvidia/include/TritonNVIDIAGPUToLLVM/PTXAsmFormat.h"
   6: #include "triton/Conversion/TritonGPUToLLVM/PatternTritonGPUOpToLLVM.h"
   7: #include "triton/Conversion/TritonGPUToLLVM/Utility.h"
   8: #include "triton/Dialect/TritonInstrument/IR/Dialect.h"
   9: #include "llvm/ADT/SmallString.h"
  10: #include "llvm/Support/LogicalResult.h"
  11: #include <limits>
```

- **EN:** Includes the interfaces this file depends on. Triton headers (`PatternTritonGPUOpToLLVM.h`, `Utility.h`, `Dialect.h`) provide domain-specific IR/support, MLIR headers (`Pattern.h`, `ImplicitLocOpBuilder.h`, `TypeUtilities.h`) provide rewriting and analysis infrastructure, LLVM headers (`SmallString.h`, `LogicalResult.h`) supply low-level utilities, and standard/library headers (`third_party/nvidia/include/TritonNVIDIAGPUToLLVM/AtomicPTXBuilder.h`, `third_party/nvidia/include/TritonNVIDIAGPUToLLVM/PTXAsmFormat.h`, `limits`) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（`PatternTritonGPUOpToLLVM.h`, `Utility.h`, `Dialect.h`）提供领域特定 IR/支持逻辑，MLIR 头文件（`Pattern.h`, `ImplicitLocOpBuilder.h`, `TypeUtilities.h`）提供重写与分析基础设施，LLVM 头文件（`SmallString.h`, `LogicalResult.h`）提供底层工具，而标准/通用库头文件（`third_party/nvidia/include/TritonNVIDIAGPUToLLVM/AtomicPTXBuilder.h`, `third_party/nvidia/include/TritonNVIDIAGPUToLLVM/PTXAsmFormat.h`, `limits`）提供通用能力。
### Lines 13-15

```cpp
  13: namespace tt = mlir::triton;
  14: namespace tti = mlir::triton::instrument;
  15: namespace ttg = mlir::triton::gpu;
```

- **EN:** Opens or closes the namespace nesting for tt -> tti -> ttg, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 tt -> tti -> ttg 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 17-17

```cpp
  17: namespace {
```

- **EN:** Opens or closes the namespace nesting for (anonymous), keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 (anonymous) 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 19-23

```cpp
  19: static constexpr unsigned kTensorMapStrideWordBase = 3;
  20: static constexpr unsigned kTensorMapShapeWordBase = 8;
  21: static constexpr unsigned kTensorMapScalarWordBase = 2;
  22: static constexpr unsigned kTensorMapNumQwords = 16;
  23: static constexpr unsigned kGSanShadowGranularityBytes = 4;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 25-28

```cpp
  25: struct GSanSourceLocation {
  26:   Value file;
  27:   Value line;
  28: };
```

- **EN:** Defines `GSanSourceLocation`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `GSanSourceLocation`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 30-42

```cpp
  30: static constexpr StringLiteral kGSanLoadTensorRuntimeFn =
  31:     "__triton_gsan_load_tensor";
  32: static constexpr StringLiteral kGSanStoreTensorRuntimeFn =
  33:     "__triton_gsan_store_tensor";
  34: static constexpr StringLiteral kGSanAtomicTensorRuntimeFn =
  35:     "__triton_gsan_atomic_tensor";
  36: static constexpr StringLiteral kGSanAtomicBeginRuntimeFn =
  37:     "__triton_gsan_atomic_begin_scalar";
  38: static constexpr StringLiteral kGSanAtomicEndRuntimeFn =
  39:     "__triton_gsan_atomic_end_scalar";
  40: static constexpr StringLiteral kGSanInitRuntimeFn = "__triton_gsan_init";
  41: static constexpr StringLiteral kGSanGlobalStateArgAttr =
  42:     "tti.gsan_global_state";
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 44-49

```cpp
  44: LLVM::LLVMFuncOp
  45: getOrCreateGSanRuntimeFunction(ConversionPatternRewriter &rewriter,
  46:                                StringRef funcName) {
  47:   auto moduleOp = rewriter.getBlock()->getParent()->getParentOfType<ModuleOp>();
  48:   if (auto funcOp = moduleOp.lookupSymbol<LLVM::LLVMFuncOp>(funcName))
  49:     return funcOp;
```

- **EN:** Defines accessor/helper `getOrCreateGSanRuntimeFunction` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getOrCreateGSanRuntimeFunction`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 51-68

```cpp
  51:   auto *ctx = rewriter.getContext();
  52:   SmallVector<Type> argTys;
  53:   if (funcName == kGSanInitRuntimeFn) {
  54:     argTys = {ptr_ty(ctx), ptr_ty(ctx), i32_ty};
  55:   } else if (funcName == kGSanLoadTensorRuntimeFn ||
  56:              funcName == kGSanStoreTensorRuntimeFn) {
  57:     argTys = {ptr_ty(ctx), ptr_ty(ctx), i32_ty, i32_ty, ptr_ty(ctx), i32_ty};
  58:   } else if (funcName == kGSanAtomicTensorRuntimeFn) {
  59:     argTys = {ptr_ty(ctx), ptr_ty(ctx), i32_ty,      i32_ty,
  60:               i32_ty,      i32_ty,      ptr_ty(ctx), i32_ty};
  61:   } else if (funcName == kGSanAtomicBeginRuntimeFn) {
  62:     argTys = {ptr_ty(ctx), ptr_ty(ctx), i32_ty,      i64_ty, i32_ty,
  63:               i32_ty,      i32_ty,      ptr_ty(ctx), i32_ty};
  64:   } else if (funcName == kGSanAtomicEndRuntimeFn) {
  65:     argTys = {ptr_ty(ctx), i32_ty, i32_ty, i32_ty, i32_ty, ptr_ty(ctx), i32_ty};
  66:   } else {
  67:     llvm_unreachable("unexpected GSan runtime symbol");
  68:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 69-74

```cpp
  69:   auto funcTy = LLVM::LLVMFunctionType::get(void_ty(ctx), argTys);
  70:   RewriterBase::InsertionGuard guard(rewriter);
  71:   rewriter.setInsertionPointToStart(moduleOp.getBody());
  72:   return LLVM::LLVMFuncOp::create(rewriter, UnknownLoc::get(ctx), funcName,
  73:                                   funcTy);
  74: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 76-81

```cpp
  76: LLVM::LLVMStructType
  77: getGSanAtomicEventStateType(ConversionPatternRewriter &rewriter) {
  78:   auto *ctx = rewriter.getContext();
  79:   return LLVM::LLVMStructType::getLiteral(
  80:       ctx, {ptr_ty(ctx), array_ty(ptr_ty(ctx), 3), i8_ty});
  81: }
```

- **EN:** Defines accessor/helper `getGSanAtomicEventStateType` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getGSanAtomicEventStateType`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 83-93

```cpp
  83: FileLineColLoc extractSourceLocation(Location loc) {
  84:   if (auto fileLoc = dyn_cast<FileLineColLoc>(loc))
  85:     return fileLoc;
  86:   if (auto nameLoc = dyn_cast<NameLoc>(loc))
  87:     return extractSourceLocation(nameLoc.getChildLoc());
  88:   if (auto opaqueLoc = dyn_cast<OpaqueLoc>(loc))
  89:     return extractSourceLocation(opaqueLoc.getFallbackLocation());
  90:   if (auto fusedLoc = dyn_cast<FusedLoc>(loc))
  91:     return extractSourceLocation(fusedLoc.getLocations().front());
  92:   if (auto callSiteLoc = dyn_cast<CallSiteLoc>(loc))
  93:     return extractSourceLocation(callSiteLoc.getCallee());
```

- **EN:** Defines `extractSourceLocation`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `extractSourceLocation`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 95-97

```cpp
  95:   StringAttr unknownFile = StringAttr::get(loc.getContext(), "<unknown>");
  96:   return FileLineColLoc::get(unknownFile, 0, 0);
  97: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 99-103

```cpp
  99: GSanSourceLocation
 100: materializeSourceLocation(ConversionPatternRewriter &rewriter, Location loc) {
 101:   auto fileLoc = extractSourceLocation(loc);
 102:   auto *ctx = rewriter.getContext();
 103:   TritonLLVMOpBuilder b(loc, rewriter);
```

- **EN:** Defines `materializeSourceLocation`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `materializeSourceLocation`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 105-110

```cpp
 105:   llvm::SmallString<64> fileName(fileLoc.getFilename().getValue());
 106:   fileName.push_back('\0');
 107:   Value file = LLVM::addStringToModule(UnknownLoc::get(ctx), rewriter,
 108:                                        "gsanLocation_", fileName);
 109:   return {file, b.i32_val(fileLoc.getLine())};
 110: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 112-114

```cpp
 112: ////////////////////////////////////////////
 113: // Utility functions
 114: ////////////////////////////////////////////
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 116-125

```cpp
 116: Value prepareTensorStackArg(ConversionPatternRewriter &rewriter, Location loc,
 117:                             ArrayRef<Value> ptrElems, ArrayRef<Value> maskElems,
 118:                             uint32_t regMask, Value threadPred,
 119:                             unsigned elemIndexStride) {
 120:   auto *ctx = rewriter.getContext();
 121:   TritonLLVMOpBuilder b(loc, rewriter);
 122:   Value one = b.i32_val(1);
 123:   Value zero = b.i32_val(0);
 124:   Type i8Ty = rewriter.getI8Type();
 125:   Type i64Ty = rewriter.getI64Type();
```

- **EN:** Defines `prepareTensorStackArg`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `prepareTensorStackArg`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 127-133

```cpp
 127:   unsigned numElems = ptrElems.size();
 128:   auto ptrArrayTy = array_ty(i64Ty, numElems);
 129:   auto maskArrayTy = array_ty(i8Ty, numElems);
 130:   SmallVector<Type> argsFieldTys = {ptrArrayTy, maskArrayTy};
 131:   auto argsTy = LLVM::LLVMStructType::getLiteral(ctx, argsFieldTys);
 132:   auto argsBuffer = LLVM::AllocaOp::create(rewriter, loc, ptr_ty(ctx), argsTy,
 133:                                            one, /*alignment=*/0);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 135-140

```cpp
 135:   for (unsigned i = 0; i < numElems; ++i) {
 136:     Value idx = b.i32_val(i);
 137:     Value ptrValue = b.ptrtoint(i64_ty, ptrElems[i]);
 138:     Value ptrSlot =
 139:         b.gep(ptr_ty(ctx), argsTy, argsBuffer, ValueRange{zero, zero, idx});
 140:     b.store(ptrValue, ptrSlot);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 142-150

```cpp
 142:     Value maskValue = maskElems.empty() ? b.true_val() : maskElems[i];
 143:     if (!isCanonicalIndex(i * elemIndexStride, regMask))
 144:       maskValue = b.false_val();
 145:     maskValue = ttg::maybeAnd(rewriter, loc, maskValue, threadPred);
 146:     Value maskByte = b.zext(i8Ty, maskValue);
 147:     Value maskSlot =
 148:         b.gep(ptr_ty(ctx), argsTy, argsBuffer, ValueRange{zero, one, idx});
 149:     b.store(maskByte, maskSlot);
 150:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 152-153

```cpp
 152:   return argsBuffer;
 153: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 155-162

```cpp
 155: void emitTensorAccessRuntimeCall(ConversionPatternRewriter &rewriter,
 156:                                  Location loc, Value gsanGlobalStatePtr,
 157:                                  ArrayRef<Value> ptrElems,
 158:                                  ArrayRef<Value> maskElems, uint32_t regMask,
 159:                                  Value threadPred, int32_t bytesPerElem,
 160:                                  bool isStore, unsigned elemIndexStride = 1) {
 161:   if (ptrElems.empty())
 162:     return;
```

- **EN:** Defines `emitTensorAccessRuntimeCall`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `emitTensorAccessRuntimeCall`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 164-170

```cpp
 164:   TritonLLVMOpBuilder b(loc, rewriter);
 165:   auto stackPtr = prepareTensorStackArg(rewriter, loc, ptrElems, maskElems,
 166:                                         regMask, threadPred, elemIndexStride);
 167:   StringRef funcName =
 168:       isStore ? kGSanStoreTensorRuntimeFn : kGSanLoadTensorRuntimeFn;
 169:   auto runtimeFunc = getOrCreateGSanRuntimeFunction(rewriter, funcName);
 170:   auto sourceLoc = materializeSourceLocation(rewriter, loc);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 172-175

```cpp
 172:   b.call(runtimeFunc,
 173:          ValueRange{gsanGlobalStatePtr, stackPtr, b.i32_val(ptrElems.size()),
 174:                     b.i32_val(bytesPerElem), sourceLoc.file, sourceLoc.line});
 175: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 177-185

```cpp
 177: void emitAtomicTensorAccessRuntimeCall(ConversionPatternRewriter &rewriter,
 178:                                        Location loc, Value gsanGlobalStatePtr,
 179:                                        ArrayRef<Value> ptrElems,
 180:                                        ArrayRef<Value> maskElems,
 181:                                        uint32_t regMask, Value threadPred,
 182:                                        int32_t bytesPerElem, MemSemantic sem,
 183:                                        MemSyncScope scope) {
 184:   if (ptrElems.empty())
 185:     return;
```

- **EN:** Defines `emitAtomicTensorAccessRuntimeCall`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `emitAtomicTensorAccessRuntimeCall`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 187-193

```cpp
 187:   TritonLLVMOpBuilder b(loc, rewriter);
 188:   auto stackPtr =
 189:       prepareTensorStackArg(rewriter, loc, ptrElems, maskElems, regMask,
 190:                             threadPred, /*elemIndexStride=*/1);
 191:   auto runtimeFunc =
 192:       getOrCreateGSanRuntimeFunction(rewriter, kGSanAtomicTensorRuntimeFn);
 193:   auto sourceLoc = materializeSourceLocation(rewriter, loc);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 195-201

```cpp
 195:   b.call(runtimeFunc,
 196:          ValueRange{gsanGlobalStatePtr, stackPtr, b.i32_val(ptrElems.size()),
 197:                     b.i32_val(bytesPerElem),
 198:                     b.i32_val(static_cast<int32_t>(sem)),
 199:                     b.i32_val(static_cast<int32_t>(scope)), sourceLoc.file,
 200:                     sourceLoc.line});
 201: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 203-205

```cpp
 203: unsigned getCanonicalIndex(unsigned index, unsigned freeVarMask) {
 204:   return index & ~freeVarMask;
 205: }
```

- **EN:** Defines accessor/helper `getCanonicalIndex` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getCanonicalIndex`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 207-213

```cpp
 207: Value broadcastScalarAtomicResult(Operation *op, Type valueElemTy,
 208:                                   Value resultVal,
 209:                                   ConversionPatternRewriter &rewriter,
 210:                                   TritonLLVMOpBuilder &b, Value threadPred,
 211:                                   const TargetInfoBase &targetInfo) {
 212:   if (!op->hasAttr("allocation.offset"))
 213:     return resultVal;
```

- **EN:** Defines `broadcastScalarAtomicResult`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `broadcastScalarAtomicResult`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 215-221

```cpp
 215:   auto loc = op->getLoc();
 216:   Value smemBase = LLVM::getSharedMemoryBase(loc, rewriter, targetInfo, op);
 217:   targetInfo.storeShared(rewriter, loc, smemBase, resultVal, threadPred);
 218:   b.barrier(ttg::AddrSpace::Local);
 219:   return targetInfo.loadShared(rewriter, loc, smemBase, valueElemTy,
 220:                                b.true_val());
 221: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 223-228

```cpp
 223: Value materializeI32Bool(ConversionPatternRewriter &rewriter,
 224:                          TritonLLVMOpBuilder &b, Value pred) {
 225:   if (!pred)
 226:     return b.i32_val(1);
 227:   return b.zext(i32_ty, pred);
 228: }
```

- **EN:** Defines `materializeI32Bool`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `materializeI32Bool`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 230-245

```cpp
 230: void emitGSanAtomicBeginCall(ConversionPatternRewriter &rewriter, Location loc,
 231:                              Value gsanGlobalStatePtr, Value eventStatePtr,
 232:                              Value pred, Value ptr, int32_t bytesPerElem,
 233:                              int32_t sem, int32_t scope,
 234:                              GSanSourceLocation sourceLoc) {
 235:   TritonLLVMOpBuilder b(loc, rewriter);
 236:   Value statePtr = b.bitcast(eventStatePtr, ptr_ty(rewriter.getContext()));
 237:   auto runtimeFunc =
 238:       getOrCreateGSanRuntimeFunction(rewriter, kGSanAtomicBeginRuntimeFn);
 239:   b.call(runtimeFunc,
 240:          ValueRange{gsanGlobalStatePtr, statePtr,
 241:                     materializeI32Bool(rewriter, b, pred),
 242:                     b.ptrtoint(i64_ty, ptr), b.i32_val(bytesPerElem),
 243:                     b.i32_val(sem), b.i32_val(scope), sourceLoc.file,
 244:                     sourceLoc.line});
 245: }
```

- **EN:** Defines `emitGSanAtomicBeginCall`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `emitGSanAtomicBeginCall`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 247-259

```cpp
 247: void emitGSanAtomicEndCall(ConversionPatternRewriter &rewriter, Location loc,
 248:                            Value eventStatePtr, Value pred, Value didWrite,
 249:                            int32_t sem, int32_t scope,
 250:                            GSanSourceLocation sourceLoc) {
 251:   TritonLLVMOpBuilder b(loc, rewriter);
 252:   auto runtimeFunc =
 253:       getOrCreateGSanRuntimeFunction(rewriter, kGSanAtomicEndRuntimeFn);
 254:   Value statePtr = b.bitcast(eventStatePtr, ptr_ty(rewriter.getContext()));
 255:   b.call(runtimeFunc,
 256:          ValueRange{statePtr, materializeI32Bool(rewriter, b, pred),
 257:                     materializeI32Bool(rewriter, b, didWrite), b.i32_val(sem),
 258:                     b.i32_val(scope), sourceLoc.file, sourceLoc.line});
 259: }
```

- **EN:** Defines `emitGSanAtomicEndCall`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `emitGSanAtomicEndCall`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 261-267

```cpp
 261: template <typename OpT>
 262: unsigned getTensorAccessVecSize(OpT op,
 263:                                 ModuleAxisInfoAnalysis &axisInfoAnalysis,
 264:                                 bool keepWithinSingleShadowCell) {
 265:   auto ptrTy = op.getPtr().getType();
 266:   auto bytesPerElem = std::max(8u, tt::getPointeeBitWidth(ptrTy)) / 8;
 267:   auto contiguity = axisInfoAnalysis.getContiguity(op.getPtr());
```

- **EN:** Defines accessor/helper `getTensorAccessVecSize` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getTensorAccessVecSize`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 269-274

```cpp
 269:   if (keepWithinSingleShadowCell) {
 270:     if (bytesPerElem >= kGSanShadowGranularityBytes)
 271:       return 1;
 272:     contiguity =
 273:         std::min(contiguity, kGSanShadowGranularityBytes / bytesPerElem);
 274:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 276-277

```cpp
 276:   if (!op.getMask())
 277:     return contiguity;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 279-284

```cpp
 279:   auto maskAlign = axisInfoAnalysis.getMaskAlignment(op.getMask());
 280:   if (bytesPerElem < kGSanShadowGranularityBytes) {
 281:     maskAlign = std::max(maskAlign, kGSanShadowGranularityBytes / bytesPerElem);
 282:   }
 283:   return std::min(contiguity, maskAlign);
 284: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 286-291

```cpp
 286: void mergeTensorAccessElements(ConversionPatternRewriter &rewriter,
 287:                                Location loc, SmallVector<Value> &ptrElems,
 288:                                SmallVector<Value> &maskElems, unsigned mergeVec,
 289:                                unsigned maskAlign, int32_t &bytesPerElem) {
 290:   if (mergeVec <= 1)
 291:     return;
```

- **EN:** Defines `mergeTensorAccessElements`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `mergeTensorAccessElements`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 293-297

```cpp
 293:   SmallVector<Value> mergedPtrElems;
 294:   SmallVector<Value> mergedMaskElems;
 295:   mergedPtrElems.reserve(ptrElems.size() / mergeVec);
 296:   if (!maskElems.empty())
 297:     mergedMaskElems.reserve(ptrElems.size() / mergeVec);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 299-309

```cpp
 299:   for (unsigned i = 0; i < ptrElems.size(); i += mergeVec) {
 300:     mergedPtrElems.push_back(ptrElems[i]);
 301:     if (maskElems.empty())
 302:       continue;
 303:     Value mergedMask = maskElems[i];
 304:     for (unsigned j = maskAlign; j < mergeVec; j += maskAlign) {
 305:       mergedMask =
 306:           arith::OrIOp::create(rewriter, loc, mergedMask, maskElems[i + j]);
 307:     }
 308:     mergedMaskElems.push_back(mergedMask);
 309:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 311-314

```cpp
 311:   ptrElems = std::move(mergedPtrElems);
 312:   maskElems = std::move(mergedMaskElems);
 313:   bytesPerElem *= mergeVec;
 314: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 316-325

```cpp
 316: Value bitcastToScalarInt(ConversionPatternRewriter &rewriter, Location loc,
 317:                          Value value) {
 318:   Type ty = value.getType();
 319:   if (ty.isInteger())
 320:     return value;
 321:   auto intTy =
 322:       IntegerType::get(rewriter.getContext(), ty.getIntOrFloatBitWidth());
 323:   TritonLLVMOpBuilder b(loc, rewriter);
 324:   return b.bitcast(value, intTy);
 325: }
```

- **EN:** Defines `bitcastToScalarInt`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `bitcastToScalarInt`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 327-342

```cpp
 327: FailureOr<Value> getGSanGlobalStateArg(Operation *op,
 328:                                        ConversionPatternRewriter &rewriter,
 329:                                        Location loc) {
 330:   auto funcOp = op->getParentOfType<FunctionOpInterface>();
 331:   for (unsigned i = 0; i < funcOp.getNumArguments(); ++i) {
 332:     if (!funcOp.getArgAttr(i, kGSanGlobalStateArgAttr))
 333:       continue;
 334:     Value arg = funcOp.getArgument(i);
 335:     if (arg.getType() == ptr_ty(rewriter.getContext()))
 336:       return arg;
 337:     TritonLLVMOpBuilder b(loc, rewriter);
 338:     arg = b.addrspacecast(ptr_ty(rewriter.getContext()), arg);
 339:     return arg;
 340:   }
 341:   return emitError(loc, "Unable to find gsan global state");
 342: }
```

- **EN:** Defines accessor/helper `getGSanGlobalStateArg` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getGSanGlobalStateArg`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 344-351

```cpp
 344: static LLVM::LLVMStructType
 345: getTensorDescStructType(ConversionPatternRewriter &rewriter, Type basePtrTy) {
 346:   SmallVector<Type> fieldTypes;
 347:   fieldTypes.reserve(1 + 2 * (kTensorMapNumQwords - 1));
 348:   fieldTypes.push_back(basePtrTy);
 349:   fieldTypes.append(2 * (kTensorMapNumQwords - 1), i32_ty);
 350:   return LLVM::LLVMStructType::getLiteral(rewriter.getContext(), fieldTypes);
 351: }
```

- **EN:** Defines accessor/helper `getTensorDescStructType` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getTensorDescStructType`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 353-362

```cpp
 353: static Value extractTensorDescWord(ConversionPatternRewriter &rewriter,
 354:                                    Location loc, Value descStruct,
 355:                                    unsigned word) {
 356:   assert(word >= kTensorMapScalarWordBase && word < 2 * kTensorMapNumQwords &&
 357:          "tensor descriptor word index out of range");
 358:   TritonLLVMOpBuilder b(loc, rewriter);
 359:   Value wordValue =
 360:       b.extract_val(i32_ty, descStruct, word - kTensorMapScalarWordBase + 1);
 361:   return b.zext(i64_ty, wordValue);
 362: }
```

- **EN:** Defines `extractTensorDescWord`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `extractTensorDescWord`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 364-377

```cpp
 364: static SmallVector<Value>
 365: decodeTensorDescShape(ConversionPatternRewriter &rewriter, Location loc,
 366:                       Value descStruct, unsigned rank) {
 367:   TritonLLVMOpBuilder b(loc, rewriter);
 368:   SmallVector<Value> shape;
 369:   shape.reserve(rank);
 370:   for (unsigned dim = 0; dim < rank; ++dim) {
 371:     unsigned packedIdx = rank - 1 - dim;
 372:     Value dimMinusOne = extractTensorDescWord(
 373:         rewriter, loc, descStruct, kTensorMapShapeWordBase + packedIdx);
 374:     shape.push_back(b.add(dimMinusOne, b.i64_val(1)));
 375:   }
 376:   return shape;
 377: }
```

- **EN:** Defines `decodeTensorDescShape`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `decodeTensorDescShape`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 379-396

```cpp
 379: static SmallVector<Value>
 380: decodeTensorDescStrides(ConversionPatternRewriter &rewriter, Location loc,
 381:                         Value descStruct, unsigned rank, unsigned elemBytes) {
 382:   TritonLLVMOpBuilder b(loc, rewriter);
 383:   SmallVector<Value> strides;
 384:   strides.reserve(rank);
 385:   for (unsigned dim = 0; dim < rank; ++dim) {
 386:     if (dim + 1 == rank) {
 387:       strides.push_back(b.i64_val(1));
 388:       continue;
 389:     }
 390:     unsigned packedIdx = rank - 2 - dim;
 391:     Value strideUnits = extractTensorDescWord(
 392:         rewriter, loc, descStruct, kTensorMapStrideWordBase + packedIdx);
 393:     Value strideBytes = b.mul(strideUnits, b.i64_val(16));
 394:     strides.push_back(b.udiv(strideBytes, b.i64_val(elemBytes)));
 395:   }
 396:   return strides;
```

- **EN:** Defines `decodeTensorDescStrides`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `decodeTensorDescStrides`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 397-397

```cpp
 397: }
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 399-401

```cpp
 399: ////////////////////////////////////////////
 400: // Patterns
 401: ////////////////////////////////////////////
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 403-409

```cpp
 403: struct GSanTensorAccessOpConversion
 404:     : public ConvertOpToLLVMPattern<tti::ExperimentalGSanTensorAccessOp> {
 405: public:
 406:   using ConvertOpToLLVMPattern<
 407:       tti::ExperimentalGSanTensorAccessOp>::ConvertOpToLLVMPattern;
 408:   const TargetInfoBase *targetInfo;
 409:   ModuleAxisInfoAnalysis *axisInfoAnalysis;
```

- **EN:** Defines `GSanTensorAccessOpConversion`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `GSanTensorAccessOpConversion`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 411-416

```cpp
 411:   GSanTensorAccessOpConversion(LLVMTypeConverter &typeConverter,
 412:                                ModuleAxisInfoAnalysis &axisInfoAnalysis,
 413:                                const TargetInfoBase &targetInfo,
 414:                                PatternBenefit benefit = 1)
 415:       : ConvertOpToLLVMPattern(typeConverter, benefit), targetInfo(&targetInfo),
 416:         axisInfoAnalysis(&axisInfoAnalysis) {}
```

- **EN:** Defines `GSanTensorAccessOpConversion`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `GSanTensorAccessOpConversion`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 418-421

```cpp
 418:   unsigned getVecSize(tti::ExperimentalGSanTensorAccessOp op) const {
 419:     return getTensorAccessVecSize(op, *axisInfoAnalysis,
 420:                                   /*keepWithinSingleShadowCell=*/false);
 421:   }
```

- **EN:** Defines accessor/helper `getVecSize` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getVecSize`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 423-433

```cpp
 423:   LogicalResult
 424:   matchAndRewrite(tti::ExperimentalGSanTensorAccessOp op, OpAdaptor adaptor,
 425:                   ConversionPatternRewriter &rewriter) const override {
 426:     Location loc = op.getLoc();
 427:     auto ptrTy = op.getPtr().getType();
 428:     int32_t bytesPerElem = tt::getPointeeBitWidth(ptrTy) / 8;
 429:     auto ptrElems = unpackLLElements(loc, adaptor.getPtr(), rewriter);
 430:     SmallVector<Value> maskElems;
 431:     if (Value llMask = adaptor.getMask()) {
 432:       maskElems = unpackLLElements(loc, llMask, rewriter);
 433:     }
```

- **EN:** Defines `matchAndRewrite`, the core MLIR rewrite callback. It checks whether the source operation matches the pattern and then materializes the lowered replacement.
- **CN:** 这里定义 `matchAndRewrite`，即 MLIR 模式重写的核心回调。它先检查源操作是否匹配模式，再构造降级后的替代结果。
### Lines 435-439

```cpp
 435:     unsigned mergeVec = getVecSize(op);
 436:     auto maskAlign =
 437:         op.getMask() ? axisInfoAnalysis->getMaskAlignment(op.getMask()) : 1;
 438:     mergeTensorAccessElements(rewriter, loc, ptrElems, maskElems, mergeVec,
 439:                               maskAlign, bytesPerElem);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 441-452

```cpp
 441:     auto ctx = op.getContext();
 442:     auto kReg = str_attr("reg");
 443:     auto freeVarMasks = getFreeVariableMasks(ptrTy);
 444:     auto threadPred = ttg::emitRedundantThreadPredicate(freeVarMasks, rewriter,
 445:                                                         loc, *targetInfo);
 446:     auto gsanGlobalStatePtr = getGSanGlobalStateArg(op, rewriter, loc);
 447:     if (failed(gsanGlobalStatePtr))
 448:       return failure();
 449:     emitTensorAccessRuntimeCall(rewriter, loc, *gsanGlobalStatePtr, ptrElems,
 450:                                 maskElems, freeVarMasks.lookup(kReg),
 451:                                 threadPred, bytesPerElem, op.getIsStore(),
 452:                                 mergeVec);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 454-457

```cpp
 454:     rewriter.eraseOp(op);
 455:     return success();
 456:   }
 457: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. When the lowered form has no direct result, the original op is erased.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 当降级后的形式不再需要直接结果时，原始操作会被删除。
### Lines 459-465

```cpp
 459: struct GSanAtomicTensorAccessOpConversion
 460:     : public ConvertOpToLLVMPattern<tti::ExperimentalGSanAtomicTensorAccessOp> {
 461: public:
 462:   using ConvertOpToLLVMPattern<
 463:       tti::ExperimentalGSanAtomicTensorAccessOp>::ConvertOpToLLVMPattern;
 464:   const TargetInfoBase *targetInfo;
 465:   ModuleAxisInfoAnalysis *axisInfoAnalysis;
```

- **EN:** Defines `GSanAtomicTensorAccessOpConversion`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `GSanAtomicTensorAccessOpConversion`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 467-472

```cpp
 467:   GSanAtomicTensorAccessOpConversion(LLVMTypeConverter &typeConverter,
 468:                                      ModuleAxisInfoAnalysis &axisInfoAnalysis,
 469:                                      const TargetInfoBase &targetInfo,
 470:                                      PatternBenefit benefit = 1)
 471:       : ConvertOpToLLVMPattern(typeConverter, benefit), targetInfo(&targetInfo),
 472:         axisInfoAnalysis(&axisInfoAnalysis) {}
```

- **EN:** Defines `GSanAtomicTensorAccessOpConversion`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `GSanAtomicTensorAccessOpConversion`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 474-479

```cpp
 474:   unsigned getVecSize(tti::ExperimentalGSanAtomicTensorAccessOp op) const {
 475:     // GSan tracks conflicts at shadow-cell granularity, so atomics may only be
 476:     // coalesced while they still fit inside a single shadow cell.
 477:     return getTensorAccessVecSize(op, *axisInfoAnalysis,
 478:                                   /*keepWithinSingleShadowCell=*/true);
 479:   }
```

- **EN:** Defines accessor/helper `getVecSize` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getVecSize`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 481-491

```cpp
 481:   LogicalResult
 482:   matchAndRewrite(tti::ExperimentalGSanAtomicTensorAccessOp op,
 483:                   OpAdaptor adaptor,
 484:                   ConversionPatternRewriter &rewriter) const override {
 485:     Location loc = op.getLoc();
 486:     auto ptrTy = op.getPtr().getType();
 487:     auto ptrElems = unpackLLElements(loc, adaptor.getPtr(), rewriter);
 488:     SmallVector<Value> maskElems;
 489:     if (Value llMask = adaptor.getMask()) {
 490:       maskElems = unpackLLElements(loc, llMask, rewriter);
 491:     }
```

- **EN:** Defines `matchAndRewrite`, the core MLIR rewrite callback. It checks whether the source operation matches the pattern and then materializes the lowered replacement.
- **CN:** 这里定义 `matchAndRewrite`，即 MLIR 模式重写的核心回调。它先检查源操作是否匹配模式，再构造降级后的替代结果。
### Lines 493-498

```cpp
 493:     int32_t bytesPerElem = std::max(1u, tt::getPointeeBitWidth(ptrTy) / 8);
 494:     unsigned mergeVec = getVecSize(op);
 495:     auto maskAlign =
 496:         op.getMask() ? axisInfoAnalysis->getMaskAlignment(op.getMask()) : 1;
 497:     mergeTensorAccessElements(rewriter, loc, ptrElems, maskElems, mergeVec,
 498:                               maskAlign, bytesPerElem);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 500-511

```cpp
 500:     auto ctx = op.getContext();
 501:     auto kReg = str_attr("reg");
 502:     auto freeVarMasks = getFreeVariableMasks(ptrTy);
 503:     auto regMask = freeVarMasks.lookup(kReg);
 504:     auto threadPred = ttg::emitRedundantThreadPredicate(freeVarMasks, rewriter,
 505:                                                         loc, *targetInfo);
 506:     auto gsanGlobalStatePtr = getGSanGlobalStateArg(op, rewriter, loc);
 507:     if (failed(gsanGlobalStatePtr))
 508:       return failure();
 509:     emitAtomicTensorAccessRuntimeCall(rewriter, loc, *gsanGlobalStatePtr,
 510:                                       ptrElems, maskElems, regMask, threadPred,
 511:                                       bytesPerElem, op.getSem(), op.getScope());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 513-516

```cpp
 513:     rewriter.eraseOp(op);
 514:     return success();
 515:   }
 516: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. When the lowered form has no direct result, the original op is erased.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 当降级后的形式不再需要直接结果时，原始操作会被删除。
### Lines 518-523

```cpp
 518: struct GSanAtomicRMWOpConversion
 519:     : public ConvertOpToLLVMPattern<tti::ExperimentalGSanAtomicRMWOp> {
 520: public:
 521:   using ConvertOpToLLVMPattern<
 522:       tti::ExperimentalGSanAtomicRMWOp>::ConvertOpToLLVMPattern;
 523:   const TargetInfoBase *targetInfo;
```

- **EN:** Defines `GSanAtomicRMWOpConversion`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `GSanAtomicRMWOpConversion`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 525-529

```cpp
 525:   GSanAtomicRMWOpConversion(LLVMTypeConverter &typeConverter,
 526:                             const TargetInfoBase &targetInfo,
 527:                             PatternBenefit benefit = 1)
 528:       : ConvertOpToLLVMPattern(typeConverter, benefit),
 529:         targetInfo(&targetInfo) {}
```

- **EN:** Defines `GSanAtomicRMWOpConversion`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `GSanAtomicRMWOpConversion`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 531-538

```cpp
 531:   LogicalResult
 532:   matchAndRewrite(tti::ExperimentalGSanAtomicRMWOp op, OpAdaptor adaptor,
 533:                   ConversionPatternRewriter &rewriter) const override {
 534:     auto *ctx = rewriter.getContext();
 535:     Location loc = op.getLoc();
 536:     auto gsanGlobalStatePtr = getGSanGlobalStateArg(op, rewriter, loc);
 537:     if (failed(gsanGlobalStatePtr))
 538:       return failure();
```

- **EN:** Defines `matchAndRewrite`, the core MLIR rewrite callback. It checks whether the source operation matches the pattern and then materializes the lowered replacement.
- **CN:** 这里定义 `matchAndRewrite`，即 MLIR 模式重写的核心回调。它先检查源操作是否匹配模式，再构造降级后的替代结果。
### Lines 540-544

```cpp
 540:     auto moduleOp = op->getParentOfType<ModuleOp>();
 541:     assert(moduleOp && "Parent ModuleOp not found for atomic op");
 542:     auto rmwOp = op.getAtomicRmwOp();
 543:     auto sem = op.getSem();
 544:     auto scope = op.getScope();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 546-549

```cpp
 546:     TritonLLVMOpBuilder b(loc, rewriter);
 547:     Value llPtr = adaptor.getPtr();
 548:     Value llVal = adaptor.getVal();
 549:     Value llMask = adaptor.getMask();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 551-555

```cpp
 551:     auto ptrElements = unpackLLElements(loc, llPtr, rewriter);
 552:     auto valElements = unpackLLElements(loc, llVal, rewriter);
 553:     SmallVector<Value> maskElements;
 554:     if (llMask)
 555:       maskElements = unpackLLElements(loc, llMask, rewriter);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 557-571

```cpp
 557:     auto valueTy = op.getType();
 558:     auto tensorTy = dyn_cast<RankedTensorType>(valueTy);
 559:     Type valueElemTy = valElements[0].getType();
 560:     unsigned valueElemNBits = valueElemTy.getIntOrFloatBitWidth();
 561:     int32_t bytesPerElem = std::max<int32_t>(1, valueElemNBits / 8);
 562:     auto elemsPerThread = ttg::getTotalElemsPerThread(op.getVal().getType());
 563:     auto freeVarMasks = getFreeVariableMasks(op.getPtr().getType());
 564:     Value threadPred = ttg::emitRedundantThreadPredicate(freeVarMasks, rewriter,
 565:                                                          loc, *targetInfo);
 566:     uint32_t regMask = freeVarMasks.lookup(str_attr("reg"));
 567:     auto sourceLoc = materializeSourceLocation(rewriter, loc);
 568:     auto eventStateTy = getGSanAtomicEventStateType(rewriter);
 569:     Value eventState = LLVM::AllocaOp::create(rewriter, loc, ptr_ty(ctx),
 570:                                               eventStateTy, b.i32_val(1),
 571:                                               /*alignment=*/0);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 573-573

```cpp
 573:     SmallVector<Value> resultVals(elemsPerThread);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 575-580

```cpp
 575:     for (size_t i = 0; i < elemsPerThread; ++i) {
 576:       if (auto canonicalIdx = getCanonicalIndex(i, regMask);
 577:           i != canonicalIdx) {
 578:         resultVals[i] = resultVals[canonicalIdx];
 579:         continue;
 580:       }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 582-586

```cpp
 582:       Value pred =
 583:           llMask ? ttg::maybeAnd(rewriter, loc, threadPred, maskElements[i])
 584:                  : threadPred;
 585:       Value rmwPtr = ptrElements[i];
 586:       Value rmwVal = valElements[i];
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 588-591

```cpp
 588:       emitGSanAtomicBeginCall(rewriter, loc, *gsanGlobalStatePtr, eventState,
 589:                               pred, rmwPtr, bytesPerElem,
 590:                               static_cast<int32_t>(sem),
 591:                               static_cast<int32_t>(scope), sourceLoc);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 593-597

```cpp
 593:       SmallVector<Value> rmwVals{rmwVal};
 594:       auto old = NVIDIA::emitPtxAtomicRMW(rewriter, loc, valueElemTy, rmwPtr,
 595:                                           rmwVals, rmwOp, sem, scope, pred);
 596:       if (failed(old))
 597:         return failure();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 599-603

```cpp
 599:       emitGSanAtomicEndCall(rewriter, loc, eventState, pred, pred,
 600:                             static_cast<int32_t>(sem),
 601:                             static_cast<int32_t>(scope), sourceLoc);
 602:       resultVals[i] = *old;
 603:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 605-608

```cpp
 605:     if (op.getResult().use_empty()) {
 606:       rewriter.eraseOp(op);
 607:       return success();
 608:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. When the lowered form has no direct result, the original op is erased.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 当降级后的形式不再需要直接结果时，原始操作会被删除。
### Lines 610-615

```cpp
 610:     if (!tensorTy) {
 611:       Value scalarResult = broadcastScalarAtomicResult(
 612:           op, valueElemTy, resultVals[0], rewriter, b, threadPred, *targetInfo);
 613:       rewriter.replaceOp(op, {scalarResult});
 614:       return success();
 615:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. It finishes by replacing the original operation with the lowered form.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 它会在转换成功后用降级后的形式替换原始操作。
### Lines 617-621

```cpp
 617:     finalizeTensorAtomicResults(op, tensorTy, rewriter, resultVals, valueElemTy,
 618:                                 b, threadPred, *targetInfo, getTypeConverter());
 619:     return success();
 620:   }
 621: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Type mapping is delegated to the LLVM type converter or related conversion helpers.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 类型映射委托给 LLVM 类型转换器或相关转换辅助函数。
### Lines 623-628

```cpp
 623: struct GSanAtomicCASOpConversion
 624:     : public ConvertOpToLLVMPattern<tti::ExperimentalGSanAtomicCASOp> {
 625: public:
 626:   using ConvertOpToLLVMPattern<
 627:       tti::ExperimentalGSanAtomicCASOp>::ConvertOpToLLVMPattern;
 628:   const TargetInfoBase *targetInfo;
```

- **EN:** Defines `GSanAtomicCASOpConversion`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `GSanAtomicCASOpConversion`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 630-634

```cpp
 630:   GSanAtomicCASOpConversion(LLVMTypeConverter &typeConverter,
 631:                             const TargetInfoBase &targetInfo,
 632:                             PatternBenefit benefit = 1)
 633:       : ConvertOpToLLVMPattern(typeConverter, benefit),
 634:         targetInfo(&targetInfo) {}
```

- **EN:** Defines `GSanAtomicCASOpConversion`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `GSanAtomicCASOpConversion`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 636-643

```cpp
 636:   LogicalResult
 637:   matchAndRewrite(tti::ExperimentalGSanAtomicCASOp op, OpAdaptor adaptor,
 638:                   ConversionPatternRewriter &rewriter) const override {
 639:     auto *ctx = rewriter.getContext();
 640:     Location loc = op.getLoc();
 641:     auto gsanGlobalStatePtr = getGSanGlobalStateArg(op, rewriter, loc);
 642:     if (failed(gsanGlobalStatePtr))
 643:       return failure();
```

- **EN:** Defines `matchAndRewrite`, the core MLIR rewrite callback. It checks whether the source operation matches the pattern and then materializes the lowered replacement.
- **CN:** 这里定义 `matchAndRewrite`，即 MLIR 模式重写的核心回调。它先检查源操作是否匹配模式，再构造降级后的替代结果。
### Lines 645-648

```cpp
 645:     auto moduleOp = op->getParentOfType<ModuleOp>();
 646:     assert(moduleOp && "Parent ModuleOp not found for atomic op");
 647:     auto sem = op.getSem();
 648:     auto scope = op.getScope();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 650-653

```cpp
 650:     TritonLLVMOpBuilder b(loc, rewriter);
 651:     Value llPtr = adaptor.getPtr();
 652:     Value llCmp = adaptor.getCmp();
 653:     Value llVal = adaptor.getVal();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 655-657

```cpp
 655:     auto ptrElements = unpackLLElements(loc, llPtr, rewriter);
 656:     auto cmpElements = unpackLLElements(loc, llCmp, rewriter);
 657:     auto valElements = unpackLLElements(loc, llVal, rewriter);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 659-673

```cpp
 659:     auto valueTy = op.getType();
 660:     auto tensorTy = dyn_cast<RankedTensorType>(valueTy);
 661:     Type valueElemTy = valElements[0].getType();
 662:     unsigned valueElemNBits = valueElemTy.getIntOrFloatBitWidth();
 663:     int32_t bytesPerElem = valueElemNBits / 8;
 664:     auto elemsPerThread = ttg::getTotalElemsPerThread(op.getVal().getType());
 665:     auto freeVarMasks = getFreeVariableMasks(op.getPtr().getType());
 666:     Value threadPred = ttg::emitRedundantThreadPredicate(freeVarMasks, rewriter,
 667:                                                          loc, *targetInfo);
 668:     uint32_t regMask = freeVarMasks.lookup(str_attr("reg"));
 669:     auto sourceLoc = materializeSourceLocation(rewriter, loc);
 670:     auto eventStateTy = getGSanAtomicEventStateType(rewriter);
 671:     Value eventState = LLVM::AllocaOp::create(rewriter, loc, ptr_ty(ctx),
 672:                                               eventStateTy, b.i32_val(1),
 673:                                               /*alignment=*/0);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 675-675

```cpp
 675:     SmallVector<Value> resultVals(elemsPerThread);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 677-682

```cpp
 677:     for (size_t i = 0; i < elemsPerThread; ++i) {
 678:       if (auto canonicalIdx = getCanonicalIndex(i, regMask);
 679:           canonicalIdx != i) {
 680:         resultVals[i] = resultVals[canonicalIdx];
 681:         continue;
 682:       }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 684-687

```cpp
 684:       Value pred = threadPred;
 685:       Value casPtr = ptrElements[i];
 686:       Value casCmp = cmpElements[i];
 687:       Value casVal = valElements[i];
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 689-692

```cpp
 689:       emitGSanAtomicBeginCall(rewriter, loc, *gsanGlobalStatePtr, eventState,
 690:                               pred, casPtr, bytesPerElem,
 691:                               static_cast<int32_t>(sem),
 692:                               static_cast<int32_t>(scope), sourceLoc);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 694-695

```cpp
 694:       Value old = NVIDIA::emitPtxAtomicCAS(rewriter, loc, valueElemTy, casPtr,
 695:                                            casCmp, casVal, sem, scope, pred);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 697-706

```cpp
 697:       auto oldInt = bitcastToScalarInt(rewriter, loc, old);
 698:       auto cmpInt = bitcastToScalarInt(rewriter, loc, casCmp);
 699:       Value didWrite = LLVM::ICmpOp::create(
 700:           rewriter, loc, i1_ty, LLVM::ICmpPredicate::eq, oldInt, cmpInt);
 701:       didWrite = ttg::maybeAnd(rewriter, loc, pred, didWrite);
 702:       emitGSanAtomicEndCall(rewriter, loc, eventState, pred, didWrite,
 703:                             static_cast<int32_t>(sem),
 704:                             static_cast<int32_t>(scope), sourceLoc);
 705:       resultVals[i] = old;
 706:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 708-711

```cpp
 708:     if (op.getResult().use_empty()) {
 709:       rewriter.eraseOp(op);
 710:       return success();
 711:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. When the lowered form has no direct result, the original op is erased.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 当降级后的形式不再需要直接结果时，原始操作会被删除。
### Lines 713-718

```cpp
 713:     if (!tensorTy) {
 714:       Value scalarResult = broadcastScalarAtomicResult(
 715:           op, valueElemTy, resultVals[0], rewriter, b, threadPred, *targetInfo);
 716:       rewriter.replaceOp(op, {scalarResult});
 717:       return success();
 718:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. It finishes by replacing the original operation with the lowered form.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 它会在转换成功后用降级后的形式替换原始操作。
### Lines 720-724

```cpp
 720:     finalizeTensorAtomicResults(op, tensorTy, rewriter, resultVals, valueElemTy,
 721:                                 b, threadPred, *targetInfo, getTypeConverter());
 722:     return success();
 723:   }
 724: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Type mapping is delegated to the LLVM type converter or related conversion helpers.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 类型映射委托给 LLVM 类型转换器或相关转换辅助函数。
### Lines 726-730

```cpp
 726: struct GSanTensorDescInfoOpConversion
 727:     : public ConvertOpToLLVMPattern<tti::ExperimentalGSanTensorDescInfoOp> {
 728: public:
 729:   using ConvertOpToLLVMPattern<
 730:       tti::ExperimentalGSanTensorDescInfoOp>::ConvertOpToLLVMPattern;
```

- **EN:** Defines `GSanTensorDescInfoOpConversion`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `GSanTensorDescInfoOpConversion`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 732-738

```cpp
 732:   LogicalResult
 733:   matchAndRewrite(tti::ExperimentalGSanTensorDescInfoOp op, OpAdaptor adaptor,
 734:                   ConversionPatternRewriter &rewriter) const override {
 735:     Location loc = op.getLoc();
 736:     auto descTy = dyn_cast<tt::TensorDescInterface>(op.getDesc().getType());
 737:     if (!descTy)
 738:       return rewriter.notifyMatchFailure(op, "expected tensor descriptor type");
```

- **EN:** Defines `matchAndRewrite`, the core MLIR rewrite callback. It checks whether the source operation matches the pattern and then materializes the lowered replacement.
- **CN:** 这里定义 `matchAndRewrite`，即 MLIR 模式重写的核心回调。它先检查源操作是否匹配模式，再构造降级后的替代结果。
### Lines 740-744

```cpp
 740:     auto elemTy = descTy.getSignlessBlockType().getElementType();
 741:     if (!elemTy.isIntOrFloat() || (elemTy.getIntOrFloatBitWidth() % 8) != 0) {
 742:       return rewriter.notifyMatchFailure(op,
 743:                                          "expected byte-addressable element");
 744:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 746-751

```cpp
 746:     unsigned rank = descTy.getShape().size();
 747:     unsigned elemBytes = elemTy.getIntOrFloatBitWidth() / 8;
 748:     if (op->getNumResults() != 1 + 2 * rank) {
 749:       return rewriter.notifyMatchFailure(
 750:           op, "descriptor info result count does not match descriptor rank");
 751:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 753-768

```cpp
 753:     TritonLLVMOpBuilder b(loc, rewriter);
 754:     Type ptrTy = getTypeConverter()->convertType(op->getResult(0).getType());
 755:     auto structTy = getTensorDescStructType(rewriter, ptrTy);
 756:     Value descStruct = b.load(structTy, adaptor.getDesc());
 757:     SmallVector<Value> decoded;
 758:     decoded.reserve(op->getNumResults());
 759:     decoded.push_back(b.extract_val(ptrTy, descStruct, 0));
 760:     auto shape = decodeTensorDescShape(rewriter, loc, descStruct, rank);
 761:     decoded.append(shape.begin(), shape.end());
 762:     auto strides =
 763:         decodeTensorDescStrides(rewriter, loc, descStruct, rank, elemBytes);
 764:     decoded.append(strides.begin(), strides.end());
 765:     rewriter.replaceOp(op, decoded);
 766:     return success();
 767:   }
 768: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. It finishes by replacing the original operation with the lowered form. Type mapping is delegated to the LLVM type converter or related conversion helpers.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 它会在转换成功后用降级后的形式替换原始操作。 类型映射委托给 LLVM 类型转换器或相关转换辅助函数。
### Lines 770-774

```cpp
 770: struct GSanInitOpConversion
 771:     : public ConvertOpToLLVMPattern<tti::ExperimentalGSanInitOp> {
 772: public:
 773:   using ConvertOpToLLVMPattern<
 774:       tti::ExperimentalGSanInitOp>::ConvertOpToLLVMPattern;
```

- **EN:** Defines `GSanInitOpConversion`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `GSanInitOpConversion`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 776-783

```cpp
 776:   LogicalResult
 777:   matchAndRewrite(tti::ExperimentalGSanInitOp op,
 778:                   [[maybe_unused]] OpAdaptor adaptor,
 779:                   ConversionPatternRewriter &rewriter) const override {
 780:     auto loc = op.getLoc();
 781:     auto gsanGlobalStatePtr = getGSanGlobalStateArg(op, rewriter, loc);
 782:     if (failed(gsanGlobalStatePtr))
 783:       return failure();
```

- **EN:** Defines `matchAndRewrite`, the core MLIR rewrite callback. It checks whether the source operation matches the pattern and then materializes the lowered replacement.
- **CN:** 这里定义 `matchAndRewrite`，即 MLIR 模式重写的核心回调。它先检查源操作是否匹配模式，再构造降级后的替代结果。
### Lines 785-786

```cpp
 785:     auto runtimeFunc =
 786:         getOrCreateGSanRuntimeFunction(rewriter, kGSanInitRuntimeFn);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 788-796

```cpp
 788:     TritonLLVMOpBuilder b(loc, rewriter);
 789:     auto sourceLoc = materializeSourceLocation(rewriter, loc);
 790:     b.call(runtimeFunc,
 791:            ValueRange{*gsanGlobalStatePtr, sourceLoc.file, sourceLoc.line});
 792:     b.barrier(ttg::AddrSpace::Local);
 793:     rewriter.eraseOp(op);
 794:     return success();
 795:   }
 796: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. When the lowered form has no direct result, the original op is erased.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 当降级后的形式不再需要直接结果时，原始操作会被删除。
### Lines 798-798

```cpp
 798: } // namespace
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 800-812

```cpp
 800: void mlir::triton::populateGSanToLLVMPatterns(
 801:     LLVMTypeConverter &typeConverter, RewritePatternSet &patterns,
 802:     ModuleAxisInfoAnalysis &axisInfoAnalysis,
 803:     const TargetInfoBase &targetInfo) {
 804:   patterns.add<GSanInitOpConversion>(typeConverter);
 805:   patterns.add<GSanTensorDescInfoOpConversion>(typeConverter);
 806:   patterns.add<GSanAtomicCASOpConversion>(typeConverter, targetInfo);
 807:   patterns.add<GSanAtomicRMWOpConversion>(typeConverter, targetInfo);
 808:   patterns.add<GSanAtomicTensorAccessOpConversion>(
 809:       typeConverter, axisInfoAnalysis, targetInfo);
 810:   patterns.add<GSanTensorAccessOpConversion>(typeConverter, axisInfoAnalysis,
 811:                                              targetInfo);
 812: }
```

- **EN:** Defines `mlir::triton::populateGSanToLLVMPatterns`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `mlir::triton::populateGSanToLLVMPatterns`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。

## Key Concepts / 关键概念
- **EN:** The file focuses on lowering g san to llvm related IR into a more target-oriented representation.
  **CN:** 本文件重点是把与 G San To LLVM 相关的 IR 降级为更面向目标的表示。
- **EN:** MLIR dialect conversion patterns drive per-op lowering to LLVM-compatible IR.
  **CN:** MLIR 方言转换模式负责按操作粒度把 IR 降级为 LLVM 兼容表示。
- **EN:** Type conversion bridges Triton/MLIR types to LLVM-level data layouts.
  **CN:** 类型转换负责把 Triton/MLIR 类型映射到 LLVM 层的数据布局。
- **EN:** Pattern rewriting is the main mechanism for canonicalization and lowering in this file.
  **CN:** 模式重写是本文件进行规范化与降级转换的核心机制。
- **EN:** Conversion rewrites replace source dialect ops with target dialect values while preserving semantics.
  **CN:** 转换式重写通过替换源方言操作来保持语义并生成目标方言值。
- **EN:** Tensor shape and element-type reasoning is central to the implementation.
  **CN:** 张量形状与元素类型推导是实现中的核心内容。

## Dependencies / 依赖关系
- **Internal Triton headers / Triton 内部头文件:** `triton/Conversion/TritonGPUToLLVM/PatternTritonGPUOpToLLVM.h`, `triton/Conversion/TritonGPUToLLVM/Utility.h`, `triton/Dialect/TritonInstrument/IR/Dialect.h`
- **MLIR headers / MLIR 头文件:** `mlir/Conversion/LLVMCommon/Pattern.h`, `mlir/IR/ImplicitLocOpBuilder.h`, `mlir/IR/TypeUtilities.h`
- **LLVM headers / LLVM 头文件:** `llvm/ADT/SmallString.h`, `llvm/Support/LogicalResult.h`
- **Standard/library headers / 标准或通用库头文件:** `third_party/nvidia/include/TritonNVIDIAGPUToLLVM/AtomicPTXBuilder.h`, `third_party/nvidia/include/TritonNVIDIAGPUToLLVM/PTXAsmFormat.h`, `limits`
- **Primary APIs used / 主要 API:** `PatternRewriter`, `ConversionPatternRewriter`, `LLVMTypeConverter`, `ModuleOp`, `RankedTensorType`, `PatternBenefit`, ... (+1 more)
- **Lowering role / 降级角色:** The implementation depends on source-dialect semantics being valid and emits forms expected by later LLVM or GPU stages. / 该实现依赖源方言语义已经正确，并输出供后续 LLVM 或 GPU 阶段使用的形式。
