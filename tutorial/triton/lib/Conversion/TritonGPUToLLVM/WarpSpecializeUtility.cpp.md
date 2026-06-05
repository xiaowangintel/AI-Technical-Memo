# WarpSpecializeUtility.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `lib/Conversion/TritonGPUToLLVM/WarpSpecializeUtility.cpp`
- **Purpose / 作用:** **EN:** Lowers TritonGPU constructs related to Warp Specialize Utility into LLVM-compatible IR and rewrite patterns. **CN:** 把与 Warp Specialize Utility 相关的 TritonGPU 构造降级为 LLVM 兼容 IR 与重写模式。

## Line-by-Line Analysis / 逐行分析

### Lines 1-10

```cpp
   1: #include "triton/Conversion/TritonGPUToLLVM/WarpSpecializeUtility.h"
   2: #include "mlir/Analysis/TopologicalSortUtils.h"
   3: #include "mlir/Dialect/LLVMIR/LLVMDialect.h"
   4: #include "mlir/Dialect/LLVMIR/LLVMTypes.h"
   5: #include "mlir/IR/Builders.h"
   6: #include "mlir/IR/BuiltinOps.h"
   7: #include "mlir/IR/ImplicitLocOpBuilder.h"
   8: #include "mlir/IR/OperationSupport.h"
   9: #include "triton/Conversion/TritonGPUToLLVM/Utility.h"
  10: #include "triton/Dialect/TritonGPU/IR/Dialect.h"
```

- **EN:** Includes the interfaces this file depends on. Triton headers (`WarpSpecializeUtility.h`, `Utility.h`, `Dialect.h`) provide domain-specific IR/support, MLIR headers (`TopologicalSortUtils.h`, `LLVMDialect.h`, `LLVMTypes.h`, `Builders.h`, ... (+3 more)) provide rewriting and analysis infrastructure, LLVM headers (None) supply low-level utilities, and standard/library headers (None) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（`WarpSpecializeUtility.h`, `Utility.h`, `Dialect.h`）提供领域特定 IR/支持逻辑，MLIR 头文件（`TopologicalSortUtils.h`, `LLVMDialect.h`, `LLVMTypes.h`, `Builders.h`, ... (+3 more)）提供重写与分析基础设施，LLVM 头文件（None）提供底层工具，而标准/通用库头文件（None）提供通用能力。
### Lines 12-14

```cpp
  12: using namespace mlir;
  13: using namespace mlir::triton;
  14: using namespace mlir::triton::gpu;
```

- **EN:** Introduces namespace aliases/imports (`mlir`, `mlir::triton`, `mlir::triton::gpu`) so the rest of the file can use MLIR/Triton symbols without repetitive qualification.
- **CN:** 这里通过命名空间导入（`mlir`, `mlir::triton`, `mlir::triton::gpu`）减少后续代码中的重复限定，使 MLIR/Triton 符号使用更简洁。
### Lines 16-18

```cpp
  16: //===----------------------------------------------------------------------===//
  17: // lowerWarpSpecializeBarriers
  18: //===----------------------------------------------------------------------===//
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 20-28

```cpp
  20: LogicalResult WarpSpecializeBarrierHelper::createBarrier(
  21:     TritonLLVMIRRewriter &b, unsigned numWarps,
  22:     std::optional<unsigned> partitionIdx) {
  23:   FailureOr<Value> handle = getBarrierHandle(b, partitionIdx);
  24:   if (failed(handle))
  25:     return failure();
  26:   createBarrier(b, numWarps, *handle);
  27:   return success();
  28: }
```

- **EN:** Defines helper `WarpSpecializeBarrierHelper::createBarrier` that computes or constructs intermediate data used by the surrounding transformation. The code uses `LogicalResult` to stay conservative when preconditions are not met.
- **CN:** 这里定义辅助函数 `WarpSpecializeBarrierHelper::createBarrier`，用于计算或构造外围变换所需的中间数据。 代码通过 `LogicalResult` 在前提不满足时保持保守处理。
### Lines 30-32

```cpp
  30: static std::string mangleFunctionName(StringRef name) {
  31:   return (name + "_ws").str();
  32: }
```

- **EN:** Defines `mangleFunctionName`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `mangleFunctionName`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 34-42

```cpp
  34: static LogicalResult lowerBarrier(Operation *op, unsigned numWarps,
  35:                                   std::optional<unsigned> partitionIdx,
  36:                                   WarpSpecializeBarrierHelper &barrierHelper) {
  37:   TritonLLVMIRRewriter b(op->getLoc(), op);
  38:   if (failed(barrierHelper.createBarrier(b, numWarps, partitionIdx)))
  39:     return failure();
  40:   op->erase();
  41:   return success();
  42: }
```

- **EN:** Defines helper `lowerBarrier` that performs a lowering step from Triton/MLIR semantics to a lower-level representation. The code uses `LogicalResult` to stay conservative when preconditions are not met.
- **CN:** 这里定义辅助函数 `lowerBarrier`，负责把 Triton/MLIR 语义中的一个步骤降级到更底层表示。 代码通过 `LogicalResult` 在前提不满足时保持保守处理。
### Lines 44-55

```cpp
  44: static LogicalResult lowerCallOp(LLVM::CallOp callOp, unsigned numWarps,
  45:                                  std::optional<unsigned> partitionIdx,
  46:                                  WarpSpecializeBarrierHelper &barrierHelper) {
  47:   TritonLLVMIRRewriter b(callOp->getLoc(), callOp);
  48:   FailureOr<Value> handle = barrierHelper.getBarrierHandle(b, partitionIdx);
  49:   if (failed(handle))
  50:     return failure();
  51:   // Forward the barrier handle.
  52:   callOp.setCallee(mangleFunctionName(*callOp.getCallee()));
  53:   callOp.getCalleeOperandsMutable().append(*handle);
  54:   return success();
  55: }
```

- **EN:** Defines helper `lowerCallOp` that performs a lowering step from Triton/MLIR semantics to a lower-level representation. The code uses `LogicalResult` to stay conservative when preconditions are not met.
- **CN:** 这里定义辅助函数 `lowerCallOp`，负责把 Triton/MLIR 语义中的一个步骤降级到更底层表示。 代码通过 `LogicalResult` 在前提不满足时保持保守处理。
### Lines 57-61

```cpp
  57: static LogicalResult
  58: lowerKernelBarriers(LLVM::LLVMFuncOp func,
  59:                     const DenseSet<StringAttr> &innerFunctions,
  60:                     WarpSpecializeBarrierHelper &barrierHelper) {
  61:   unsigned defaultNumWarps = lookupNumWarps(func);
```

- **EN:** Defines helper `lowerKernelBarriers` that performs a lowering step from Triton/MLIR semantics to a lower-level representation.
- **CN:** 这里定义辅助函数 `lowerKernelBarriers`，负责把 Triton/MLIR 语义中的一个步骤降级到更底层表示。
### Lines 63-80

```cpp
  63:   // Turn all barrier ops into warp group barriers.
  64:   // HACK: Right now, higher-level passes generate all barriers that we
  65:   // interpret as warp group barriers, but they generate explicit warp group
  66:   // barriers.
  67:   SmallVector<WarpSpecializeOp> wsOps;
  68:   WalkResult result = func.walk<mlir::WalkOrder::PreOrder>([&](Operation *op) {
  69:     // Walk into default regions but not partition regions.
  70:     if (auto wsOp = dyn_cast<WarpSpecializePartitionsOp>(op)) {
  71:       wsOps.push_back(wsOp.getParentOp());
  72:       return WalkResult::skip();
  73:     }
  74:     if (barrierHelper.isBarrierOp(op)) {
  75:       auto barRes =
  76:           lowerBarrier(op, defaultNumWarps, /*partitionIdx=*/{}, barrierHelper);
  77:       // Since this is a PreOrder walk, we have to return skip() on success so
  78:       // we do not attempt to visit the regions in the now deleted op.
  79:       return barRes.succeeded() ? WalkResult::skip() : WalkResult::interrupt();
  80:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 81-90

```cpp
  81:     if (auto callOp = dyn_cast<LLVM::CallOp>(op)) {
  82:       if (!innerFunctions.contains(callOp.getCalleeAttr().getAttr()))
  83:         return WalkResult::advance();
  84:       return WalkResult(lowerCallOp(callOp, defaultNumWarps,
  85:                                     /*partitionIdx=*/{}, barrierHelper));
  86:     }
  87:     return WalkResult::advance();
  88:   });
  89:   if (result.wasInterrupted())
  90:     return failure();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 92-109

```cpp
  92:   // Each partition executes simultaneously, so each will get a different
  93:   // barrier ID, but note this means there is a maximum of 16 barriers.
  94:   for (WarpSpecializeOp op : wsOps) {
  95:     for (auto [idx, partition] : llvm::enumerate(op.getPartitionRegions())) {
  96:       unsigned numWarps = op.getPartitionNumWarps()[idx];
  97:       WalkResult result = partition->walk([&, idx = idx](Operation *op) {
  98:         if (barrierHelper.isBarrierOp(op)) {
  99:           return WalkResult(lowerBarrier(op, numWarps, idx, barrierHelper));
 100:         }
 101:         if (auto callOp = dyn_cast<LLVM::CallOp>(op)) {
 102:           if (!innerFunctions.contains(callOp.getCalleeAttr().getAttr()))
 103:             return WalkResult::advance();
 104:           return WalkResult(lowerCallOp(callOp, numWarps, idx, barrierHelper));
 105:         }
 106:         return WalkResult::advance();
 107:       });
 108:       if (result.wasInterrupted())
 109:         return failure();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 110-111

```cpp
 110:     }
 111:   }
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 113-114

```cpp
 113:   return success();
 114: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 116-126

```cpp
 116: static LogicalResult
 117: lowerInnerFunctionBarriers(LLVM::LLVMFuncOp func,
 118:                            const DenseSet<StringAttr> &innerFunctions,
 119:                            WarpSpecializeBarrierHelper &barrierHelper) {
 120:   // Append a barrier handle argument.
 121:   LLVM::LLVMFunctionType type = func.getFunctionType();
 122:   SmallVector<Type> newArgTypes = to_vector(type.getParams());
 123:   newArgTypes.push_back(barrierHelper.getBarrierHandleType(type.getContext()));
 124:   func.setFunctionType(LLVM::LLVMFunctionType::get(
 125:       type.getReturnType(), newArgTypes, type.isVarArg()));
 126:   Value handle = func.getBody().addArgument(newArgTypes.back(), func.getLoc());
```

- **EN:** Defines helper `lowerInnerFunctionBarriers` that performs a lowering step from Triton/MLIR semantics to a lower-level representation.
- **CN:** 这里定义辅助函数 `lowerInnerFunctionBarriers`，负责把 Triton/MLIR 语义中的一个步骤降级到更底层表示。
### Lines 128-134

```cpp
 128:   // Mangle the function to distinguish it from non-warp-specialized versions.
 129:   func.setSymName(mangleFunctionName(func.getSymName()));
 130:   if (ArrayAttr argAttrs = func.getArgAttrsAttr()) {
 131:     SmallVector<Attribute> newArgAttrs = to_vector(argAttrs.getValue());
 132:     newArgAttrs.push_back(DictionaryAttr::get(func.getContext(), {}));
 133:     func.setArgAttrsAttr(ArrayAttr::get(func.getContext(), newArgAttrs));
 134:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 136-142

```cpp
 136:   // Lower barrier ops.
 137:   auto numWarpsAttr = func->getAttrOfType<IntegerAttr>("ws_num_warps");
 138:   if (!numWarpsAttr) {
 139:     return func.emitError("function missing '") << "ws_num_warps"
 140:                                                 << "' attribute";
 141:   }
 142:   unsigned numWarps = numWarpsAttr.getInt();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 144-155

```cpp
 144:   func.walk([&](Operation *op) {
 145:     if (barrierHelper.isBarrierOp(op)) {
 146:       TritonLLVMIRRewriter b(op->getLoc(), op);
 147:       barrierHelper.createBarrier(b, numWarps, handle);
 148:       op->erase();
 149:     } else if (auto callOp = dyn_cast<LLVM::CallOp>(op)) {
 150:       if (!innerFunctions.contains(callOp.getCalleeAttr().getAttr()))
 151:         return;
 152:       callOp.setCallee(mangleFunctionName(*callOp.getCallee()));
 153:       callOp.getCalleeOperandsMutable().append(handle);
 154:     }
 155:   });
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. The implementation traverses IR operations to collect facts or apply rewrites globally.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。
### Lines 157-158

```cpp
 157:   return success();
 158: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 160-176

```cpp
 160: LogicalResult mlir::triton::lowerWarpSpecializeBarriers(
 161:     ModuleOp module, WarpSpecializeBarrierHelper &barrierHelper) {
 162:   SmallVector<LLVM::LLVMFuncOp> wsKernels;
 163:   // Find all kernels and the warp specialize ops in them.
 164:   for (LLVM::LLVMFuncOp func : module.getOps<LLVM::LLVMFuncOp>()) {
 165:     WalkResult result =
 166:         func.walk([&](WarpSpecializeOp op) { return WalkResult::interrupt(); });
 167:     // Nothing to do. This kernel is not warp specialized.
 168:     if (!result.wasInterrupted())
 169:       continue;
 170:     if (func.getLinkage() != LLVM::Linkage::External) {
 171:       return func.emitError(
 172:           "only top-level kernel functions can be warp-specialized");
 173:     }
 174:     wsKernels.push_back(func);
 175:   }
 176:   // No warp specialization found.
```

- **EN:** Defines helper `mlir::triton::lowerWarpSpecializeBarriers` that performs a lowering step from Triton/MLIR semantics to a lower-level representation. The implementation traverses IR operations to collect facts or apply rewrites globally. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义辅助函数 `mlir::triton::lowerWarpSpecializeBarriers`，负责把 Triton/MLIR 语义中的一个步骤降级到更底层表示。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 177-178

```cpp
 177:   if (wsKernels.empty())
 178:     return success();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 180-184

```cpp
 180:   DenseSet<StringAttr> innerFunctions;
 181:   for (LLVM::LLVMFuncOp func : module.getOps<LLVM::LLVMFuncOp>()) {
 182:     if (func.getLinkage() != LLVM::Linkage::External)
 183:       innerFunctions.insert(func.getSymNameAttr());
 184:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 186-189

```cpp
 186:   for (LLVM::LLVMFuncOp func : wsKernels) {
 187:     if (failed(lowerKernelBarriers(func, innerFunctions, barrierHelper)))
 188:       return failure();
 189:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 191-196

```cpp
 191:   for (LLVM::LLVMFuncOp func : module.getOps<LLVM::LLVMFuncOp>()) {
 192:     if (func.getLinkage() == LLVM::Linkage::External)
 193:       continue;
 194:     if (failed(lowerInnerFunctionBarriers(func, innerFunctions, barrierHelper)))
 195:       return failure();
 196:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 198-199

```cpp
 198:   return success();
 199: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 201-203

```cpp
 201: //===----------------------------------------------------------------------===//
 202: // convertOpTypes
 203: //===----------------------------------------------------------------------===//
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 205-221

```cpp
 205: void mlir::triton::convertOpTypes(Operation *op,
 206:                                   const TypeConverter &typeConverter) {
 207:   ImplicitLocOpBuilder b(op->getLoc(), op);
 208:   // WarpSpecializePartitionsOp exists in a region that must only contain a
 209:   // single op. This also means that we know that its operands always dominate
 210:   // the enclosing WarpSpecializeOp, so we can insert the casts there instead.
 211:   if (isa<WarpSpecializePartitionsOp>(op))
 212:     b.setInsertionPoint(op->getParentOp());
 213:   SmallVector<Value> operands = llvm::to_vector(op->getOperands());
 214:   for (Value &operand : operands) {
 215:     Type type = typeConverter.convertType(operand.getType());
 216:     if (type != operand.getType()) {
 217:       operand =
 218:           UnrealizedConversionCastOp::create(b, type, operand).getResult(0);
 219:     }
 220:   }
 221:   op->setOperands(operands);
```

- **EN:** Defines helper `mlir::triton::convertOpTypes` that computes or constructs intermediate data used by the surrounding transformation. Type mapping is delegated to the LLVM type converter or related conversion helpers. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义辅助函数 `mlir::triton::convertOpTypes`，用于计算或构造外围变换所需的中间数据。 类型映射委托给 LLVM 类型转换器或相关转换辅助函数。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 223-232

```cpp
 223:   for (Region &region : op->getRegions()) {
 224:     b.setInsertionPointToStart(&region.front());
 225:     for (BlockArgument arg : llvm::to_vector(region.getArguments())) {
 226:       Type type = typeConverter.convertType(arg.getType());
 227:       BlockArgument newArg = region.addArgument(type, arg.getLoc());
 228:       auto cast = UnrealizedConversionCastOp::create(b, arg.getType(), newArg);
 229:       arg.replaceAllUsesWith(cast.getResult(0));
 230:       region.eraseArgument(0);
 231:     }
 232:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Type mapping is delegated to the LLVM type converter or related conversion helpers.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 类型映射委托给 LLVM 类型转换器或相关转换辅助函数。
### Lines 234-243

```cpp
 234:   SmallVector<Type> resultTypes;
 235:   (void)typeConverter.convertTypes(op->getResultTypes(), resultTypes);
 236:   if (TypeRange(resultTypes) == op->getResultTypes())
 237:     return;
 238:   OperationState state(op->getLoc(), op->getName(), op->getOperands(),
 239:                        resultTypes, op->getAttrs());
 240:   for (Region &region : op->getRegions())
 241:     state.addRegion()->takeBody(region);
 242:   b.setInsertionPoint(op);
 243:   Operation *newOp = b.create(state);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Type mapping is delegated to the LLVM type converter or related conversion helpers. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 类型映射委托给 LLVM 类型转换器或相关转换辅助函数。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 245-252

```cpp
 245:   SmallVector<Value> results;
 246:   for (auto [i, result, type] :
 247:        llvm::enumerate(newOp->getResults(), op->getResultTypes())) {
 248:     auto cast = UnrealizedConversionCastOp::create(b, type, result);
 249:     op->getResult(i).replaceAllUsesWith(cast.getResult(0));
 250:   }
 251:   op->erase();
 252: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 254-256

```cpp
 254: //===----------------------------------------------------------------------===//
 255: // elideTrivialCaptures
 256: //===----------------------------------------------------------------------===//
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 258-271

```cpp
 258: static LogicalResult findTrivialSubcomputation(LLVM::LLVMFuncOp func,
 259:                                                Value capture,
 260:                                                SetVector<Operation *> &ops) {
 261:   SetVector<Value> worklist;
 262:   worklist.insert(capture);
 263:   for (unsigned i = 0; i != worklist.size(); ++i) {
 264:     Value capture = worklist[i];
 265:     // Check for a kernel argument.
 266:     if (auto arg = dyn_cast<BlockArgument>(capture)) {
 267:       if (arg.getOwner() == &func.getBody().front())
 268:         continue;
 269:       // Otherwise, this is some other block argument that cannot be elided.
 270:       return failure();
 271:     }
```

- **EN:** Defines `findTrivialSubcomputation`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `findTrivialSubcomputation`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 273-283

```cpp
 273:     Operation *op = capture.getDefiningOp();
 274:     // Check if the defining op can be rematerialized. At the LLVM level,
 275:     // checking for pure is probably a good enough heuristic.
 276:     if (isPure(op)) {
 277:       ops.insert(op);
 278:       worklist.insert(op->operand_begin(), op->operand_end());
 279:       continue;
 280:     }
 281:     // The op cannot be rematerialized.
 282:     return failure();
 283:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 285-288

```cpp
 285:   // Cap the number of ops that can be rematerialized.
 286:   // FIXME: This is arbitrary.
 287:   return success(ops.size() <= 16);
 288: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 290-306

```cpp
 290: void mlir::triton::elideTrivialCaptures(LLVM::LLVMFuncOp func,
 291:                                         ArrayRef<WarpSpecializeOp> wsOps) {
 292:   // The goal is to completely eliminate captures by hoisting or rematerializing
 293:   // computations. We could minimize captures by rematerializing
 294:   // subcomputations, but that is much more complicated. Prefer rematerializing
 295:   // because that reduces liveranges. If subgraphs are duplicated more than
 296:   // once, we will rely on CSE to clean them up.
 297:   SetVector<Operation *> subgraph;
 298:   for (WarpSpecializeOp wsOp : wsOps) {
 299:     auto partOp = wsOp.getPartitionOp();
 300:     llvm::BitVector toErase(partOp.getNumOperands());
 301:     for (auto [i, capture] : llvm::enumerate(partOp.getExplicitCaptures())) {
 302:       subgraph.clear();
 303:       if (failed(findTrivialSubcomputation(func, capture, subgraph)))
 304:         continue;
 305:       toErase.set(i);
 306:       subgraph = topologicalSort(subgraph);
```

- **EN:** Defines `mlir::triton::elideTrivialCaptures`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `mlir::triton::elideTrivialCaptures`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 308-321

```cpp
 308:       for (Region *region : wsOp.getPartitionRegions()) {
 309:         OpBuilder b(region);
 310:         IRMapping mapping;
 311:         for (Operation *op : subgraph) {
 312:           b.clone(*op, mapping);
 313:         }
 314:         Value remat = capture;
 315:         if (!subgraph.empty()) {
 316:           unsigned resultIdx = cast<OpResult>(capture).getResultNumber();
 317:           remat = mapping.lookup(subgraph.back())->getResult(resultIdx);
 318:         }
 319:         region->getArgument(i).replaceAllUsesWith(remat);
 320:       }
 321:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 323-328

```cpp
 323:     partOp->eraseOperands(toErase);
 324:     for (Region *region : wsOp.getPartitionRegions()) {
 325:       region->front().eraseArguments(toErase);
 326:     }
 327:   }
 328: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. When the lowered form has no direct result, the original op is erased.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 当降级后的形式不再需要直接结果时，原始操作会被删除。
### Lines 330-342

```cpp
 330: /// Disable LICM (Loop Invariant Code Motion) for a loop. This prevents LLVM
 331: /// from hoisting code out of the switch loop generated by the
 332: /// `ttg.warp_specialize` lowering, which could result in long liveranges and
 333: /// cause register spilling in partition regions.
 334: static void disableLICM(LLVM::BrOp latchBr) {
 335:   Builder b(latchBr.getContext());
 336:   MLIRContext *ctx = b.getContext();
 337:   auto licmMD = LLVM::LoopLICMAttr::get(ctx, b.getBoolAttr(true), {});
 338:   auto loopMD =
 339:       LLVM::LoopAnnotationAttr::get(b.getContext(), {}, {}, {}, {}, {}, licmMD,
 340:                                     {}, {}, {}, {}, {}, {}, {}, {}, {});
 341:   latchBr.setLoopAnnotationAttr(loopMD);
 342: }
```

- **EN:** Defines `disableLICM`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `disableLICM`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 344-346

```cpp
 344: //===----------------------------------------------------------------------===//
 345: // lowerWarpSpecializeCommon
 346: //===----------------------------------------------------------------------===//
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 348-356

```cpp
 348: static void rewritePartitionRegions(WarpSpecializeOp ws, Block *switchLoop,
 349:                                     const TargetInfoBase &targetInfo,
 350:                                     const WarpSpecializeCallbacks &callbacks,
 351:                                     unsigned switchLoopBarrierIdx) {
 352:   TritonLLVMIRRewriter b(ws.getLoc(), ws.getContext());
 353:   for (Region *partition : ws.getPartitionRegions()) {
 354:     // Load the explicit captures from shared memory and replace the block args
 355:     // if there are any.
 356:     b.setInsertionPointToStart(&partition->front());
```

- **EN:** Defines `rewritePartitionRegions`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `rewritePartitionRegions`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 358-360

```cpp
 358:     callbacks.reallocRegisters(b, ws,
 359:                                RegisterReallocPhase::WorkerPartitionStart,
 360:                                partition->getRegionNumber());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 362-379

```cpp
 362:     if (partition->getNumArguments()) {
 363:       auto captureType = LLVM::LLVMStructType::getLiteral(
 364:           b.getContext(), llvm::to_vector(partition->getArgumentTypes()),
 365:           /*isPacked=*/true);
 366:       Value capturePtr =
 367:           LLVM::getSharedMemoryBase(b.getLoc(), b, targetInfo, ws);
 368:       LLVM::LLVMPointerType ptrTy =
 369:           ptr_ty(b.getContext(), targetInfo.getSharedAddressSpace());
 370:       for (auto [i, arg] :
 371:            llvm::zip(llvm::seq<int32_t>(partition->getNumArguments()),
 372:                      partition->getArguments())) {
 373:         Value ptr =
 374:             b.gep(ptrTy, captureType, capturePtr, ArrayRef<LLVM::GEPArg>{0, i});
 375:         // Each thread in the warp group needs a copy of the value.
 376:         Value value = b.load(arg.getType(), ptr, /*align=*/1);
 377:         arg.replaceAllUsesWith(value);
 378:       }
 379:       partition->front().eraseArguments([](auto) { return true; });
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 380-380

```cpp
 380:     }
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 382-384

```cpp
 382:     // The shared memory is only live for the entry into the region, so put
 383:     // another barrier here.
 384:     callbacks.createAllBarrier(b, switchLoopBarrierIdx);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 386-396

```cpp
 386:     // Rewrite all warp returns.
 387:     partition->walk([&](WarpReturnOp op) {
 388:       TritonLLVMIRRewriter b(op.getLoc(), op);
 389:       callbacks.createAllBarrier(b, switchLoopBarrierIdx);
 390:       callbacks.reallocRegisters(b, ws,
 391:                                  RegisterReallocPhase::WorkerPartitionEnd,
 392:                                  partition->getRegionNumber());
 393:       b.replaceOpWithNewOp<LLVM::BrOp>(op, switchLoop);
 394:     });
 395:   }
 396: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. It finishes by replacing the original operation with the lowered form. The implementation traverses IR operations to collect facts or apply rewrites globally.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 它会在转换成功后用降级后的形式替换原始操作。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。
### Lines 398-403

```cpp
 398: LogicalResult mlir::triton::lowerWarpSpecializeCommon(
 399:     LLVM::LLVMFuncOp func, ArrayRef<WarpSpecializeOp> wsOps, Block *entry,
 400:     Block *header, Block *switchLoop, Value wid, MLIRContext *ctx,
 401:     unsigned defaultNumWarps, unsigned totalNumWarps,
 402:     const TargetInfoBase &targetInfo, const WarpSpecializeCallbacks &callbacks,
 403:     unsigned switchLoopBarrierIdx) {
```

- **EN:** Defines helper `mlir::triton::lowerWarpSpecializeCommon` that performs a lowering step from Triton/MLIR semantics to a lower-level representation.
- **CN:** 这里定义辅助函数 `mlir::triton::lowerWarpSpecializeCommon`，负责把 Triton/MLIR 语义中的一个步骤降级到更底层表示。
### Lines 405-407

```cpp
 405:   TritonLLVMIRRewriter b(func.getLoc(), ctx);
 406:   Type int8Type = b.getIntegerType(8);
 407:   LLVM::LLVMPointerType ptrTy = ptr_ty(ctx, targetInfo.getSharedAddressSpace());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 409-414

```cpp
 409:   b.setInsertionPointToStart(switchLoop);
 410:   callbacks.reallocRegisters(b, wsOps[0], RegisterReallocPhase::SwitchLoopStart,
 411:                              0);
 412:   callbacks.createAllBarrier(b, switchLoopBarrierIdx);
 413:   Value statePtr = LLVM::getSharedMemoryBase(b.getLoc(), b, targetInfo, func);
 414:   Value relWid = b.sub(wid, b.i32_val(defaultNumWarps));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 416-423

```cpp
 416:   // The default warp group will populate the state pointer with the state ID
 417:   // for all warps.
 418:   // %warp_state_ptr = getelementptr ptr %state_tr[%rel_wid]
 419:   // %warp_state = load i8 %warp_state_ptr
 420:   Value warpStatePtr = b.gep(ptrTy, int8Type, statePtr, relWid);
 421:   // All threads in a warp reading from the same smem address will not create
 422:   // bank conflicts and is better than predicated load.
 423:   Value warpState = b.load(int8Type, warpStatePtr);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 425-435

```cpp
 425:   // Pull the partition regions out. Switch based on the state ID to the right
 426:   // partition.
 427:   SmallVector<Block *> partitionBlocks;
 428:   SmallVector<int32_t> partitionStates;
 429:   int32_t partitionStateCounter = 0;
 430:   // This represents the data that the default warp group will fill into the
 431:   // state pointer before entering each `warp_specialize` region, which maps
 432:   // a warp ID to a state ID in the switch.
 433:   int32_t maxNumWarps = totalNumWarps - defaultNumWarps;
 434:   SmallVector<SmallVector<int32_t>> warpToState(
 435:       wsOps.size(), SmallVector<int32_t>(maxNumWarps, -1));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 437-451

```cpp
 437:   for (size_t i = 0; i < wsOps.size(); ++i) {
 438:     WarpSpecializeOp op = wsOps[i];
 439:     auto &stateMap = warpToState[i];
 440:     rewritePartitionRegions(op, switchLoop, targetInfo, callbacks,
 441:                             switchLoopBarrierIdx);
 442:     for (auto [partition, partitionNumWarps, startId] :
 443:          llvm::zip(op.getPartitionRegions(), op.getPartitionNumWarps(),
 444:                    *op.getWarpGroupStartIds())) {
 445:       partitionStates.push_back(partitionStateCounter++);
 446:       partitionBlocks.push_back(&partition->front());
 447:       for (int32_t &stateId : MutableArrayRef(stateMap).slice(
 448:                startId - defaultNumWarps, partitionNumWarps))
 449:         stateId = partitionStates.back();
 450:     }
 451:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 453-456

```cpp
 453:   if (partitionStateCounter > std::numeric_limits<uint8_t>::max()) {
 454:     return mlir::emitError(func.getLoc(),
 455:                            "FIXME: too many warp group partitions");
 456:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 458-464

```cpp
 458:   // Splice them in reverse order so the IR is easier to read.
 459:   Region::BlockListType &funcBlocks = func.getBody().getBlocks();
 460:   for (Block *block : llvm::reverse(partitionBlocks)) {
 461:     Region *region = block->getParent();
 462:     funcBlocks.splice(std::next(switchLoop->getIterator()),
 463:                       region->getBlocks());
 464:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 466-473

```cpp
 466:   // Default destination.
 467:   Block *defaultBlock = new Block;
 468:   funcBlocks.insert(std::next(switchLoop->getIterator()), defaultBlock);
 469:   b.setInsertionPointToStart(defaultBlock);
 470:   callbacks.createAllBarrier(b, switchLoopBarrierIdx);
 471:   callbacks.createAllBarrier(b, switchLoopBarrierIdx);
 472:   auto latchBr = LLVM::BrOp::create(b, b.getLoc(), switchLoop);
 473:   disableLICM(latchBr);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 475-479

```cpp
 475:   // Exit state.
 476:   Block *switchExit = new Block;
 477:   funcBlocks.insert(std::next(defaultBlock->getIterator()), switchExit);
 478:   partitionBlocks.push_back(switchExit);
 479:   partitionStates.push_back(partitionStateCounter);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 481-488

```cpp
 481:   // Create the switch.
 482:   b.setInsertionPointToEnd(switchLoop);
 483:   SmallVector<APInt> caseValues;
 484:   for (int32_t state : partitionStates)
 485:     caseValues.push_back(APInt(8, state));
 486:   LLVM::SwitchOp::create(b, b.getLoc(), warpState, defaultBlock, ValueRange(),
 487:                          caseValues, partitionBlocks,
 488:                          SmallVector<ValueRange>(partitionBlocks.size()));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 490-503

```cpp
 490:   // Now add synchronization around the default regions.
 491:   for (size_t i = 0; i < wsOps.size(); ++i) {
 492:     WarpSpecializeOp ws = wsOps[i];
 493:     auto &stateMap = warpToState[i];
 494:     Block *before = ws->getBlock();
 495:     Block *after = b.splitBlock(before, ws->getIterator());
 496:     TritonLLVMIRRewriter b(ws.getLoc(), OpBuilder::atBlockEnd(before));
 497:     Type int8Type = b.getIntegerType(8);
 498:     Value statePtrWs =
 499:         LLVM::getSharedMemoryBase(b.getLoc(), b, targetInfo, func);
 500:     for (auto [j, state] : llvm::enumerate(stateMap)) {
 501:       Value stateVal = b.i8_val(state);
 502:       b.store(stateVal, b.gep(ptrTy, int8Type, statePtrWs, LLVM::GEPArg(j)));
 503:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 505-520

```cpp
 505:     // Store the captures if there are any.
 506:     auto partOp = ws.getPartitionOp();
 507:     if (partOp.getNumOperands()) {
 508:       auto captureType = LLVM::LLVMStructType::getLiteral(
 509:           b.getContext(), llvm::to_vector(partOp.getOperandTypes()),
 510:           /*isPacked=*/true);
 511:       Value capturePtr =
 512:           LLVM::getSharedMemoryBase(b.getLoc(), b, targetInfo, ws);
 513:       for (auto [j, arg] :
 514:            llvm::zip(llvm::seq<int32_t>(partOp.getNumOperands()),
 515:                      partOp.getOperands())) {
 516:         Value ptr =
 517:             b.gep(ptrTy, captureType, capturePtr, ArrayRef<LLVM::GEPArg>{0, j});
 518:         b.store(arg, ptr, /*align=*/1);
 519:       }
 520:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 522-528

```cpp
 522:     // First barrier releases the waiting warpgroups. The second barrier ensures
 523:     // they have read the captures before the memory is released upon entry.
 524:     callbacks.createAllBarrier(b, switchLoopBarrierIdx);
 525:     callbacks.reallocRegisters(b, ws,
 526:                                RegisterReallocPhase::DefaultPartitionStart, 0);
 527:     callbacks.createAllBarrier(b, switchLoopBarrierIdx);
 528:     LLVM::BrOp::create(b, b.getLoc(), &ws.getDefaultRegion().front());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 530-538

```cpp
 530:     ws.getDefaultRegion().walk([&, ws = ws](WarpYieldOp op) mutable {
 531:       TritonLLVMIRRewriter b(op.getLoc(), op);
 532:       callbacks.createAllBarrier(b, switchLoopBarrierIdx);
 533:       callbacks.reallocRegisters(b, ws,
 534:                                  RegisterReallocPhase::DefaultPartitionEnd, 0);
 535:       b.replaceOpWithNewOp<LLVM::BrOp>(op, op.getOperands(), after);
 536:     });
 537:     after->getParent()->getBlocks().splice(after->getIterator(),
 538:                                            ws.getDefaultRegion().getBlocks());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. It finishes by replacing the original operation with the lowered form.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 它会在转换成功后用降级后的形式替换原始操作。
### Lines 540-546

```cpp
 540:     // Replace the results.
 541:     auto outputs = after->addArguments(
 542:         ws.getResultTypes(),
 543:         SmallVector<Location>(ws.getNumResults(), ws.getLoc()));
 544:     ws.replaceAllUsesWith(outputs);
 545:     ws.erase();
 546:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 548-560

```cpp
 548:   // Signal all warp groups to exit.
 549:   func.walk([&](LLVM::ReturnOp op) {
 550:     TritonLLVMIRRewriter b(op.getLoc(), op);
 551:     Type int8Type = b.getIntegerType(8);
 552:     Value statePtrExit =
 553:         LLVM::getSharedMemoryBase(b.getLoc(), b, targetInfo, func);
 554:     Value cst = b.i8_val(partitionStateCounter);
 555:     for (int32_t i : llvm::seq(maxNumWarps))
 556:       b.store(cst, b.gep(ptrTy, int8Type, statePtrExit, LLVM::GEPArg(i)));
 557:     callbacks.createAllBarrier(b, switchLoopBarrierIdx);
 558:   });
 559:   b.setInsertionPointToStart(switchExit);
 560:   LLVM::ReturnOp::create(b, b.getLoc(), ValueRange());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. The implementation traverses IR operations to collect facts or apply rewrites globally.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。
### Lines 562-563

```cpp
 562:   return success();
 563: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。

## Key Concepts / 关键概念
- **EN:** The file focuses on lowering warp specialize utility related IR into a more target-oriented representation.
  **CN:** 本文件重点是把与 Warp Specialize Utility 相关的 IR 降级为更面向目标的表示。
- **EN:** The code reasons at module scope rather than only on isolated operations.
  **CN:** 代码在模块范围内进行分析，而不只是处理单个操作。
- **EN:** Hardware execution parameters such as warps and threads-per-warp affect legality and performance decisions.
  **CN:** warp 数与每个 warp 的线程数等硬件执行参数会影响合法性和性能决策。
- **EN:** Synchronization and ordering constraints matter to preserve correctness across threads, warps, or memory spaces.
  **CN:** 同步与顺序约束很重要，它们保证跨线程、warp 或内存空间的正确性。

## Dependencies / 依赖关系
- **Internal Triton headers / Triton 内部头文件:** `triton/Conversion/TritonGPUToLLVM/WarpSpecializeUtility.h`, `triton/Conversion/TritonGPUToLLVM/Utility.h`, `triton/Dialect/TritonGPU/IR/Dialect.h`
- **MLIR headers / MLIR 头文件:** `mlir/Analysis/TopologicalSortUtils.h`, `mlir/Dialect/LLVMIR/LLVMDialect.h`, `mlir/Dialect/LLVMIR/LLVMTypes.h`, `mlir/IR/Builders.h`, `mlir/IR/BuiltinOps.h`, `mlir/IR/ImplicitLocOpBuilder.h`, ... (+1 more)
- **LLVM headers / LLVM 头文件:** None
- **Standard/library headers / 标准或通用库头文件:** None
- **Primary APIs used / 主要 API:** `ModuleOp`, `OpBuilder`, `OperationState`
- **Lowering role / 降级角色:** The implementation depends on source-dialect semantics being valid and emits forms expected by later LLVM or GPU stages. / 该实现依赖源方言语义已经正确，并输出供后续 LLVM 或 GPU 阶段使用的形式。
