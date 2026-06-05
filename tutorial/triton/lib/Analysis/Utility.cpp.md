# Utility.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `lib/Analysis/Utility.cpp`
- **Purpose / 作用:** **EN:** Implements the utility analysis logic used by Triton and MLIR passes. **CN:** 实现与 Utility 相关的分析逻辑，供 Triton 与 MLIR 的 pass 使用。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1

```cpp
   1: #include "triton/Analysis/Utility.h"
```

- **EN:** Includes the interfaces this file depends on. Triton headers (`Utility.h`) provide domain-specific IR/support, MLIR headers (None) provide rewriting and analysis infrastructure, LLVM headers (None) supply low-level utilities, and standard/library headers (None) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（`Utility.h`）提供领域特定 IR/支持逻辑，MLIR 头文件（None）提供重写与分析基础设施，LLVM 头文件（None）提供底层工具，而标准/通用库头文件（None）提供通用能力。
### Lines 3-3

```cpp
   3: #include <deque>
```

- **EN:** Includes the interfaces this file depends on. Triton headers (None) provide domain-specific IR/support, MLIR headers (None) provide rewriting and analysis infrastructure, LLVM headers (None) supply low-level utilities, and standard/library headers (`deque`) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（None）提供领域特定 IR/支持逻辑，MLIR 头文件（None）提供重写与分析基础设施，LLVM 头文件（None）提供底层工具，而标准/通用库头文件（`deque`）提供通用能力。
### Lines 5-21

```cpp
   5: #include "mlir/Analysis/DataFlow/ConstantPropagationAnalysis.h"
   6: #include "mlir/Analysis/DataFlow/DeadCodeAnalysis.h"
   7: #include "mlir/Dialect/Arith/IR/Arith.h"
   8: #include "mlir/Dialect/LLVMIR/LLVMDialect.h"
   9: #include "mlir/IR/Dialect.h"
  10: #include "mlir/IR/Matchers.h"
  11: #include "mlir/Support/LLVM.h"
  12: #include "triton/Analysis/Allocation.h"
  13: #include "triton/Conversion/MLIRTypes.h"
  14: #include "triton/Dialect/Triton/IR/Dialect.h"
  15: #include "triton/Dialect/Triton/IR/Utility.h"
  16: #include "triton/Dialect/TritonGPU/IR/Dialect.h"
  17: #include "triton/Dialect/TritonGPU/IR/LinearLayoutConversions.h"
  18: #include "triton/Tools/LayoutUtils.h"
  19: #include "triton/Tools/LinearLayout.h"
  20: #include "triton/Tools/Sys/GetEnv.h"
  21: #include "llvm/ADT/SmallSet.h"
```

- **EN:** Includes the interfaces this file depends on. Triton headers (`Allocation.h`, `MLIRTypes.h`, `Dialect.h`, `Utility.h`, ... (+5 more)) provide domain-specific IR/support, MLIR headers (`ConstantPropagationAnalysis.h`, `DeadCodeAnalysis.h`, `Arith.h`, `LLVMDialect.h`, ... (+3 more)) provide rewriting and analysis infrastructure, LLVM headers (`SmallSet.h`) supply low-level utilities, and standard/library headers (None) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（`Allocation.h`, `MLIRTypes.h`, `Dialect.h`, `Utility.h`, ... (+5 more)）提供领域特定 IR/支持逻辑，MLIR 头文件（`ConstantPropagationAnalysis.h`, `DeadCodeAnalysis.h`, `Arith.h`, `LLVMDialect.h`, ... (+3 more)）提供重写与分析基础设施，LLVM 头文件（`SmallSet.h`）提供底层工具，而标准/通用库头文件（None）提供通用能力。
### Lines 23-23

```cpp
  23: namespace mlir {
```

- **EN:** Opens or closes the namespace nesting for mlir, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 mlir 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 25-26

```cpp
  25: using namespace triton;
  26: using namespace triton::gpu;
```

- **EN:** Introduces namespace aliases/imports (`triton`, `triton::gpu`) so the rest of the file can use MLIR/Triton symbols without repetitive qualification.
- **CN:** 这里通过命名空间导入（`triton`, `triton::gpu`）减少后续代码中的重复限定，使 MLIR/Triton 符号使用更简洁。
### Lines 28-37

```cpp
  28: // Cases where distributed shared memory is not required in ConvertLayout:
  29: // (1) numCTAs == 1
  30: // (2) numCTAs > 1 but srcCGALayout == dstCGALayout
  31: // TODO: Case with SliceLayout as srcLayout and numCTAs > 1 is to be implemented
  32: // in the future
  33: bool shouldUseDistSmem(Attribute srcLayout, Attribute dstLayout) {
  34:   unsigned numCTAs = getNumCTAs(srcLayout);
  35:   assert(numCTAs == getNumCTAs(dstLayout) &&
  36:          "Invalid layout conversion: the numbers of CTAs of src and dst "
  37:          "layouts are different");
```

- **EN:** Defines `shouldUseDistSmem`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `shouldUseDistSmem`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 39-41

```cpp
  39:   // Case (1): Never use dsmem when numCTAs == 1
  40:   if (numCTAs == 1)
  41:     return false;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 43-50

```cpp
  43:   // Case where CTAsPerCGA of srcLayout in the sliced dim is not 1 is not
  44:   // implemented yet
  45:   if (auto sliceLayout = mlir::dyn_cast<SliceEncodingAttr>(srcLayout)) {
  46:     auto dim = sliceLayout.getDim();
  47:     auto CTAsPerCGA = getCTAsPerCGA(sliceLayout.getParent());
  48:     if (CTAsPerCGA[dim] != 1)
  49:       llvm::report_fatal_error("Layout conversion to be implemented");
  50:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 52-58

```cpp
  52:   // Case where CTAsPerCGA of dstLayout in the sliced dim is not 1 is supported
  53:   if (auto sliceLayout = mlir::dyn_cast<SliceEncodingAttr>(dstLayout)) {
  54:     auto dim = sliceLayout.getDim();
  55:     auto CTAsPerCGA = getCTAsPerCGA(sliceLayout.getParent());
  56:     if (CTAsPerCGA[dim] != 1)
  57:       return true;
  58:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 60-61

```cpp
  60:   // The above two branches make sure that it is legal to call getCGALayout of
  61:   // srcLayout and dstLayout
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 63-67

```cpp
  63:   // Case (2): Do not use dsmem when srcCGALayout == dstCGALayout
  64:   auto srcCGALayout = getCGALayout(srcLayout);
  65:   auto dstCGALayout = getCGALayout(dstLayout);
  66:   if (srcCGALayout == dstCGALayout)
  67:     return false;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 69-71

```cpp
  69:   // Dsmem access is required when srcCGALayout != dstCGALayout
  70:   return true;
  71: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 73-75

```cpp
  73: unsigned ReduceOpHelper::getInterWarpSizeWithUniqueData() {
  74:   return getWarpsPerCTA(srcEncoding, srcShape)[axis];
  75: }
```

- **EN:** Defines accessor/helper `ReduceOpHelper::getInterWarpSizeWithUniqueData` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `ReduceOpHelper::getInterWarpSizeWithUniqueData`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 77-79

```cpp
  77: unsigned ReduceOpHelper::getIntraWarpSizeWithUniqueData() {
  78:   return getThreadsPerWarp(srcEncoding, srcShape)[axis];
  79: }
```

- **EN:** Defines accessor/helper `ReduceOpHelper::getIntraWarpSizeWithUniqueData` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `ReduceOpHelper::getIntraWarpSizeWithUniqueData`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 81-86

```cpp
  81: bool ReduceOpHelper::isReduceWithinCTA() {
  82:   // TODO: Support reduce across CTAS
  83:   // Layout optimization passes such as PlanCTAPass and
  84:   // RemoveLayoutConversionPass should avoid cross-CTA reduction
  85:   return getCTASplitNum(srcEncoding)[axis] == 1;
  86: }
```

- **EN:** Defines `ReduceOpHelper::isReduceWithinCTA`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `ReduceOpHelper::isReduceWithinCTA`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 88-105

```cpp
  88: bool ReduceOpHelper::isAssociative() {
  89:   auto dtype = srcElementTypes[0];
  90:   if (!type::isFloat(dtype))
  91:     return true;
  92:   size_t reduce_size = srcShape[axis];
  93:   if (reduce_size <= 2)
  94:     return true;
  95:   bool hasNoAssociativeOp = false;
  96:   op.walk([&](Operation *nestedOp) -> WalkResult {
  97:     if (isa<arith::AddFOp, arith::MulFOp>(nestedOp)) {
  98:       // Only when the data type is float point and reduce size greater than 2,
  99:       // and has addf or mulf op, we though it's a non-associative reduce.
 100:       hasNoAssociativeOp = true;
 101:       return WalkResult::interrupt();
 102:     }
 103:     return WalkResult::advance();
 104:   });
 105:   return !hasNoAssociativeOp;
```

- **EN:** Defines `ReduceOpHelper::isAssociative`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. The implementation traverses IR operations to collect facts or apply rewrites globally.
- **CN:** 这里定义 `ReduceOpHelper::isAssociative`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。
### Lines 106-106

```cpp
 106: }
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 108-110

```cpp
 108: unsigned ReduceOpHelper::getScratchSizeInBytes(
 109:     GetNumScratchElemsFn numScratchElemsGetter) {
 110:   auto kLane = StringAttr::get(op.getContext(), "lane");
```

- **EN:** Defines accessor/helper `ReduceOpHelper::getScratchSizeInBytes` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `ReduceOpHelper::getScratchSizeInBytes`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 112-115

```cpp
 112:   auto isReduced = [axis = axis](const LinearLayout &layout) {
 113:     return layout.getOutDimSizes().begin()[axis] == 1;
 114:   };
 115:   auto regLl = reducedRegLaneLayout(srcTy, axis);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 117-134

```cpp
 117:   // All the inputs have the same layout so, since we order them from largest
 118:   // bitsize to smallest, and the first one is aligned, by induction, they are
 119:   // all aligned, so we don't need to align the byte numbers returned here.
 120:   unsigned bytesRegToTmp = 0;
 121:   while (!isReduced(regLl)) {
 122:     auto tmpLl = getInterLayout(regLl, axis);
 123:     // We take the maximum of the elements and multiply by the total bitwidth.
 124:     // We do this as otherwise it's quite tricky to find the correct
 125:     // BaseOffsets in the lowering.
 126:     int bytes = 0;
 127:     for (auto inputTy : op.getInputTypes()) {
 128:       unsigned nelem = 0;
 129:       if (numScratchElemsGetter) {
 130:         nelem = numScratchElemsGetter(regLl, tmpLl, getBitwidth(inputTy));
 131:       } else {
 132:         nelem =
 133:             getNumScratchElemsSwizzledCvt(regLl, tmpLl, getBitwidth(inputTy));
 134:       }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 135-141

```cpp
 135:       bytes += nelem * (getBitwidth(inputTy) / 8);
 136:     }
 137:     bytesRegToTmp = std::max<unsigned>(bytesRegToTmp, bytes);
 138:     regLl = zeroBasesAlongDimAndReorder(tmpLl, axis, kLane);
 139:   }
 140:   return bytesRegToTmp;
 141: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 143-150

```cpp
 143: ReduceOpHelper::InThreadVectorizeOpKind
 144: ReduceOpHelper::getInThreadVectorizeOpKind(unsigned axisPack,
 145:                                            bool supportBitwidth16Elementwise,
 146:                                            bool supportBitwidth32Elementwise) {
 147:   Operation *reduceOperation = op.getOperation();
 148:   if (axisPack < 4 || reduceOperation->getNumOperands() != 1 ||
 149:       reduceOperation->getNumResults() != 1)
 150:     return InThreadVectorizeOpKind::None;
```

- **EN:** Defines accessor/helper `ReduceOpHelper::getInThreadVectorizeOpKind` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `ReduceOpHelper::getInThreadVectorizeOpKind`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 152-158

```cpp
 152:   assert(reduceOperation->getNumRegions() == 1 &&
 153:          "expected a single combine region");
 154:   Region &combineRegion = reduceOperation->getRegion(0);
 155:   Block &block = combineRegion.front();
 156:   if (block.getOperations().size() != 2)
 157:     return InThreadVectorizeOpKind::None;
 158:   Operation &combiner = block.front();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 160-165

```cpp
 160:   Type elemTy = srcElementTypes.front();
 161:   unsigned bitwidth = elemTy.getIntOrFloatBitWidth();
 162:   if (bitwidth == 16 && !supportBitwidth16Elementwise)
 163:     return InThreadVectorizeOpKind::None;
 164:   if (bitwidth == 32 && !supportBitwidth32Elementwise)
 165:     return InThreadVectorizeOpKind::None;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 167-170

```cpp
 167:   bool is16Bit = bitwidth == 16;
 168:   bool isF32 = elemTy.isF32();
 169:   if (!is16Bit && !isF32)
 170:     return InThreadVectorizeOpKind::None;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 172-187

```cpp
 172:   if (isa<arith::AddFOp>(combiner)) {
 173:     return (is16Bit || isF32) ? InThreadVectorizeOpKind::AddF
 174:                               : InThreadVectorizeOpKind::None;
 175:   }
 176:   if (isa<arith::MulFOp>(combiner)) {
 177:     return (is16Bit || isF32) ? InThreadVectorizeOpKind::MulF
 178:                               : InThreadVectorizeOpKind::None;
 179:   }
 180:   if (isa<arith::MinNumFOp>(combiner)) {
 181:     return is16Bit ? InThreadVectorizeOpKind::MinNumF
 182:                    : InThreadVectorizeOpKind::None;
 183:   }
 184:   if (isa<arith::MaxNumFOp>(combiner)) {
 185:     return is16Bit ? InThreadVectorizeOpKind::MaxNumF
 186:                    : InThreadVectorizeOpKind::None;
 187:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 188-195

```cpp
 188:   if (isa<arith::MinimumFOp>(combiner)) {
 189:     return is16Bit ? InThreadVectorizeOpKind::MinimumF
 190:                    : InThreadVectorizeOpKind::None;
 191:   }
 192:   if (isa<arith::MaximumFOp>(combiner)) {
 193:     return is16Bit ? InThreadVectorizeOpKind::MaximumF
 194:                    : InThreadVectorizeOpKind::None;
 195:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 197-198

```cpp
 197:   if (!elemTy.isInteger(16))
 198:     return InThreadVectorizeOpKind::None;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 200-217

```cpp
 200:   if (isa<arith::AddIOp>(combiner)) {
 201:     return InThreadVectorizeOpKind::AddI;
 202:   }
 203:   if (isa<arith::MulIOp>(combiner)) {
 204:     return InThreadVectorizeOpKind::MulI;
 205:   }
 206:   if (isa<arith::MinSIOp>(combiner)) {
 207:     return InThreadVectorizeOpKind::MinSI;
 208:   }
 209:   if (isa<arith::MaxSIOp>(combiner)) {
 210:     return InThreadVectorizeOpKind::MaxSI;
 211:   }
 212:   if (isa<arith::MinUIOp>(combiner)) {
 213:     return InThreadVectorizeOpKind::MinUI;
 214:   }
 215:   if (isa<arith::MaxUIOp>(combiner)) {
 216:     return InThreadVectorizeOpKind::MaxUI;
 217:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 219-220

```cpp
 219:   return InThreadVectorizeOpKind::None;
 220: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 222-239

```cpp
 222: Value ReduceOpHelper::createInThreadVectorizedCombineOp(
 223:     OpBuilder &builder, Location loc, InThreadVectorizeOpKind kind, Value lhs,
 224:     Value rhs) {
 225:   auto vecTy = lhs.getType();
 226:   Value result;
 227:   switch (kind) {
 228:   case InThreadVectorizeOpKind::AddF:
 229:     result = LLVM::FAddOp::create(builder, loc, lhs, rhs);
 230:     break;
 231:   case InThreadVectorizeOpKind::MulF:
 232:     result = LLVM::FMulOp::create(builder, loc, lhs, rhs);
 233:     break;
 234:   case InThreadVectorizeOpKind::MinNumF:
 235:     result = LLVM::MinNumOp::create(builder, loc, vecTy, lhs, rhs);
 236:     break;
 237:   case InThreadVectorizeOpKind::MaxNumF:
 238:     result = LLVM::MaxNumOp::create(builder, loc, vecTy, lhs, rhs);
 239:     break;
```

- **EN:** Defines helper `ReduceOpHelper::createInThreadVectorizedCombineOp` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `ReduceOpHelper::createInThreadVectorizedCombineOp`，用于计算或构造外围变换所需的中间数据。
### Lines 240-257

```cpp
 240:   case InThreadVectorizeOpKind::MinimumF:
 241:     result = LLVM::MinimumOp::create(builder, loc, vecTy, lhs, rhs);
 242:     break;
 243:   case InThreadVectorizeOpKind::MaximumF:
 244:     result = LLVM::MaximumOp::create(builder, loc, vecTy, lhs, rhs);
 245:     break;
 246:   case InThreadVectorizeOpKind::AddI:
 247:     result = LLVM::AddOp::create(builder, loc, lhs, rhs);
 248:     break;
 249:   case InThreadVectorizeOpKind::MulI:
 250:     result = LLVM::MulOp::create(builder, loc, lhs, rhs);
 251:     break;
 252:   case InThreadVectorizeOpKind::MinSI:
 253:     result = LLVM::SMinOp::create(builder, loc, vecTy, lhs, rhs);
 254:     break;
 255:   case InThreadVectorizeOpKind::MaxSI:
 256:     result = LLVM::SMaxOp::create(builder, loc, vecTy, lhs, rhs);
 257:     break;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 258-269

```cpp
 258:   case InThreadVectorizeOpKind::MinUI:
 259:     result = LLVM::UMinOp::create(builder, loc, vecTy, lhs, rhs);
 260:     break;
 261:   case InThreadVectorizeOpKind::MaxUI:
 262:     result = LLVM::UMaxOp::create(builder, loc, vecTy, lhs, rhs);
 263:     break;
 264:   case InThreadVectorizeOpKind::None:
 265:   default:
 266:     llvm::report_fatal_error("Unsupported in-thread vectorize op kind");
 267:   }
 268:   return result;
 269: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 271-277

```cpp
 271: ColumnAction ReduceOpHelper::moveAxisBasesToFront(const LinearLayout &layout,
 272:                                                   int axis, bool isVectorized) {
 273:   auto *ctx = layout.getOutDimNames().begin()->getContext();
 274:   auto kReg = StringAttr::get(ctx, "register");
 275:   const auto &bases = layout.getBases().lookup(kReg);
 276:   if (bases.empty())
 277:     return ColumnAction::identity(kReg, bases.size());
```

- **EN:** Defines `ReduceOpHelper::moveAxisBasesToFront`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `ReduceOpHelper::moveAxisBasesToFront`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 279-294

```cpp
 279:   // We keep the first basis where it is if it's vectorized to pack it without
 280:   // PRMT/MOV, and then we move the rest of the bases that don't move the axis
 281:   // to the front after it
 282:   SmallVector<size_t> perm;
 283:   if (isVectorized)
 284:     perm.push_back(0);
 285:   SmallVector<size_t> back;
 286:   for (size_t i = isVectorized ? 1 : 0; i < bases.size(); ++i) {
 287:     if (bases[i][axis] != 0)
 288:       perm.push_back(i);
 289:     else
 290:       back.push_back(i);
 291:   }
 292:   perm.append(back.begin(), back.end());
 293:   return ColumnAction(perm, kReg, bases.size());
 294: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 296-312

```cpp
 296: LinearLayout
 297: ReduceOpHelper::zeroBasesAlongDimAndReorder(const LinearLayout &layout,
 298:                                             unsigned axis, StringAttr dim) {
 299:   // Zeros out the basis along the specified axis in the given hardware
 300:   // dimension, and reindexes the remaining bases along axis so that each
 301:   // element is in linearly increasing order from the hardware's perspective.
 302:   // Note that for this reordering we need the operator to be commutative, but
 303:   // it's the only way to have a performant lowering.
 304:   LinearLayout::BasesT newBases;
 305:   for (auto [inDim, bases] : layout.getBases()) {
 306:     std::vector<std::vector<int32_t>> newInBases = bases;
 307:     if (inDim == dim) {
 308:       for (auto &basis : newInBases)
 309:         basis[axis] = 0;
 310:     }
 311:     newBases[inDim] = std::move(newInBases);
 312:   }
```

- **EN:** Defines `ReduceOpHelper::zeroBasesAlongDimAndReorder`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `ReduceOpHelper::zeroBasesAlongDimAndReorder`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 314-322

```cpp
 314:   int32_t nextAxisBase = 1;
 315:   for (auto &[inDim, inDimBases] : newBases) {
 316:     for (auto &basis : inDimBases) {
 317:       if (basis[axis] == 0)
 318:         continue;
 319:       basis[axis] = nextAxisBase;
 320:       nextAxisBase *= 2;
 321:     }
 322:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 324-325

```cpp
 324:   return LinearLayout(std::move(newBases), to_vector(layout.getOutDimNames()));
 325: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 327-336

```cpp
 327: LinearLayout ReduceOpHelper::getInterLayout(const LinearLayout &layout,
 328:                                             unsigned axis) {
 329:   auto *ctx = layout.getOutDimNames().begin()->getContext();
 330:   auto kLane = mlir::StringAttr::get(ctx, "lane");
 331:   auto kWarp = mlir::StringAttr::get(ctx, "warp");
 332:   auto kBlock = mlir::StringAttr::get(ctx, "block");
 333:   auto bases = layout.getBases();
 334:   auto &laneBases = bases[kLane];
 335:   auto &warpBases = bases[kWarp];
 336:   auto &blockBases = bases[kBlock];
```

- **EN:** Defines accessor/helper `ReduceOpHelper::getInterLayout` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `ReduceOpHelper::getInterLayout`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 338-345

```cpp
 338:   auto collectAxisBases = [&](ArrayRef<std::vector<int32_t>> bases) {
 339:     SmallVector<unsigned> out;
 340:     for (unsigned i = 0; i < bases.size(); ++i) {
 341:       if (bases[i][axis] != 0)
 342:         out.push_back(i);
 343:     }
 344:     return out;
 345:   };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 347-348

```cpp
 347:   SmallVector<unsigned> warpAxisBases = collectAxisBases(warpBases);
 348:   SmallVector<unsigned> blockAxisBases = collectAxisBases(blockBases);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 350-354

```cpp
 350:   SmallVector<unsigned> zeroLaneBases;
 351:   for (unsigned i = 0; i < laneBases.size(); ++i) {
 352:     if (llvm::all_of(laneBases[i], [](int32_t v) { return v == 0; }))
 353:       zeroLaneBases.push_back(i);
 354:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 356-356

```cpp
 356:   auto totalAxisBases = warpAxisBases.size() + blockAxisBases.size();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 358-372

```cpp
 358:   // First try to place all warp/block axis bases into lane bases that are
 359:   // currently zero. If we can do this we will be able to perform the full
 360:   // reduction with just one convert_layout
 361:   if (zeroLaneBases.size() >= totalAxisBases) {
 362:     unsigned laneIdx = 0;
 363:     for (unsigned idx : warpAxisBases) {
 364:       std::swap(laneBases[zeroLaneBases[laneIdx]], warpBases[idx]);
 365:       ++laneIdx;
 366:     }
 367:     for (unsigned idx : blockAxisBases) {
 368:       std::swap(laneBases[zeroLaneBases[laneIdx]], blockBases[idx]);
 369:       ++laneIdx;
 370:     }
 371:     return LinearLayout(std::move(bases), to_vector(layout.getOutDimNames()));
 372:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 374-391

```cpp
 374:   // If we can fit all the bases inside the lane dimension, we can perform the
 375:   // reduction with two convert_layouts
 376:   // The first cvt to move the relevant bases to the lane dimension
 377:   // The second to move all the bases we moved out of the lane dimension back to
 378:   // their original positions
 379:   if (warpAxisBases.size() + blockAxisBases.size() <= laneBases.size()) {
 380:     assert(totalAxisBases <= laneBases.size() &&
 381:            "unexpected lane base count for axis layout");
 382:     unsigned laneIdx = 0;
 383:     for (unsigned idx : warpAxisBases) {
 384:       std::swap(laneBases[laneIdx], warpBases[idx]);
 385:       ++laneIdx;
 386:     }
 387:     for (unsigned idx : blockAxisBases) {
 388:       std::swap(laneBases[laneIdx], blockBases[idx]);
 389:       ++laneIdx;
 390:     }
 391:     return LinearLayout(std::move(bases), to_vector(layout.getOutDimNames()));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 392-392

```cpp
 392:   }
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 394-403

```cpp
 394:   // Assumptions (easily relaxed if AMD needs it)
 395:   // We assume that
 396:   // max number of warps * max number of blocks <= (max number of lanes)^2
 397:   // We check this in logarithmic space (number of bases)
 398:   // This is true in nvidia as the max numbers are warps=64 ctas=16 so that
 399:   // 64 * 16 = 1024 = 32 * 32 = laneBases.size() * laneBases.size()
 400:   // This implies that, even if we have to perform 3 cvt_layouts, we can perform
 401:   // first one that does not cross CTAs, and then two that may cross CTAs
 402:   assert(blockBases.size() <= laneBases.size());
 403:   assert(warpBases.size() + blockBases.size() <= 2 * laneBases.size());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 405-412

```cpp
 405:   // Otherwise, fit as many warp bases as possible into the lane dimension
 406:   unsigned laneIdx = 0;
 407:   for (unsigned idx : warpAxisBases) {
 408:     std::swap(laneBases[laneIdx], warpBases[idx]);
 409:     ++laneIdx;
 410:     if (laneIdx >= laneBases.size())
 411:       break;
 412:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 414-415

```cpp
 414:   return LinearLayout(std::move(bases), to_vector(layout.getOutDimNames()));
 415: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 417-421

```cpp
 417: LinearLayout ReduceOpHelper::reducedRegLaneLayout(RankedTensorType srcTy,
 418:                                                   unsigned axis) {
 419:   auto *ctx = srcTy.getContext();
 420:   auto kReg = StringAttr::get(ctx, "register");
 421:   auto kLane = StringAttr::get(ctx, "lane");
```

- **EN:** Defines `ReduceOpHelper::reducedRegLaneLayout`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `ReduceOpHelper::reducedRegLaneLayout`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 423-424

```cpp
 423:   auto reduced = toLinearLayout(srcTy);
 424:   reduced = actionRemoveBroadcastedRegs(reduced).apply(reduced);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 426-431

```cpp
 426:   reduced = moveAxisBasesToFront(reduced, axis).apply(reduced);
 427:   reduced = zeroBasesAlongDimAndReorder(reduced, axis, kReg);
 428:   reduced = actionRemoveBroadcastedRegs(reduced).apply(reduced);
 429:   reduced = zeroBasesAlongDimAndReorder(reduced, axis, kLane);
 430:   return reduced;
 431: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 433-449

```cpp
 433: ScanLoweringHelper::ScanLoweringHelper(triton::ScanOp op) : scanOp(op) {
 434:   auto firstTy = cast<RankedTensorType>(op.getOperands()[0].getType());
 435:   srcShape = firstTy.getShape();
 436:   legacyEncoding = firstTy.getEncoding();
 437:   // Remove broadcasting in the registers
 438:   // We also remove it in the lowering and re-add it when we pack the results
 439:   auto origLayout = triton::gpu::toLinearLayout(firstTy);
 440:   auto removeBroadcastRegs = actionRemoveBroadcastedRegs(origLayout);
 441:   origLayout = removeBroadcastRegs.apply(origLayout);
 442:   srcEncoding = triton::gpu::LinearEncodingAttr::get(op.getContext(),
 443:                                                      std::move(origLayout));
 444:   srcElementTypes = op.getElementTypes();
 445:   // The codegen does not support different element/thread/warp order so
 446:   // we choose one a priori. We choose that of the blocked encoding.
 447:   // When we generalise this code to other layouts we'll probably need to
 448:   // get rid of all this logic and the *Stride auxiliary methods
 449:   // and replace them by transposes and reshapes on the LinearLayout
```

- **EN:** Defines helper `ScanLoweringHelper::ScanLoweringHelper` that performs a lowering step from Triton/MLIR semantics to a lower-level representation.
- **CN:** 这里定义辅助函数 `ScanLoweringHelper::ScanLoweringHelper`，负责把 Triton/MLIR 语义中的一个步骤降级到更底层表示。
### Lines 450-455

```cpp
 450:   if (auto blockedEncoding =
 451:           dyn_cast<triton::gpu::BlockedEncodingAttr>(legacyEncoding)) {
 452:     order = llvm::to_vector(blockedEncoding.getOrder());
 453:   } else {
 454:     order = srcEncoding.getOrder();
 455:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 457-465

```cpp
 457:   for (const auto &t : op.getInputTypes()) {
 458:     if (t.getShape() != srcShape) {
 459:       op.emitError() << "shape mismatch";
 460:     }
 461:     if (t.getEncoding() != legacyEncoding) {
 462:       op.emitError() << "encoding mismatch";
 463:     }
 464:   }
 465: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 467-469

```cpp
 467: unsigned ScanLoweringHelper::getAxisNumElementsPerThread() {
 468:   return getEncoding().getContigPerThread()[getAxis()];
 469: }
```

- **EN:** Defines accessor/helper `ScanLoweringHelper::getAxisNumElementsPerThread` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `ScanLoweringHelper::getAxisNumElementsPerThread`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 471-475

```cpp
 471: unsigned ScanLoweringHelper::getNonAxisNumElementsPerThread() {
 472:   auto contigPerThread = getEncoding().getContigPerThread();
 473:   contigPerThread[getAxis()] = 1;
 474:   return product<unsigned>(contigPerThread);
 475: }
```

- **EN:** Defines accessor/helper `ScanLoweringHelper::getNonAxisNumElementsPerThread` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `ScanLoweringHelper::getNonAxisNumElementsPerThread`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 477-477

```cpp
 477: Region &ScanLoweringHelper::getCombineOp() { return scanOp.getCombineOp(); }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 479-481

```cpp
 479: unsigned ScanLoweringHelper::getAxisNumThreadsPerWarpWithUniqueData() {
 480:   return getEncoding().getThreadsPerWarp()[getAxis()];
 481: }
```

- **EN:** Defines accessor/helper `ScanLoweringHelper::getAxisNumThreadsPerWarpWithUniqueData` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `ScanLoweringHelper::getAxisNumThreadsPerWarpWithUniqueData`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 483-486

```cpp
 483: unsigned ScanLoweringHelper::getNonAxisNumThreadsPerWarp() {
 484:   auto nThreads = product(getEncoding().getThreadsPerWarp());
 485:   return nThreads / getAxisNumThreadsPerWarpWithUniqueData();
 486: }
```

- **EN:** Defines accessor/helper `ScanLoweringHelper::getNonAxisNumThreadsPerWarp` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `ScanLoweringHelper::getNonAxisNumThreadsPerWarp`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 488-493

```cpp
 488: // Return the flat numbers of threads computing independent scan results.
 489: unsigned ScanLoweringHelper::getNonAxisNumThreadsPerCTA() {
 490:   auto nWarps = product(getEncoding().getWarpsPerCTA());
 491:   return (nWarps / getAxisNumWarpsWithUniqueData()) *
 492:          getNonAxisNumThreadsPerWarp();
 493: }
```

- **EN:** Defines accessor/helper `ScanLoweringHelper::getNonAxisNumThreadsPerCTA` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `ScanLoweringHelper::getNonAxisNumThreadsPerCTA`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 495-497

```cpp
 495: unsigned ScanLoweringHelper::getAxisNumWarpsWithUniqueData() {
 496:   return getEncoding().getWarpsPerCTA()[getAxis()];
 497: }
```

- **EN:** Defines accessor/helper `ScanLoweringHelper::getAxisNumWarpsWithUniqueData` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `ScanLoweringHelper::getAxisNumWarpsWithUniqueData`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 499-507

```cpp
 499: unsigned ScanLoweringHelper::getAxisNumBlocks() {
 500:   auto contigPerThread = getEncoding().getContigPerThread();
 501:   auto threadsPerWarp = getEncoding().getThreadsPerWarp();
 502:   auto warpsPerCTA = getEncoding().getWarpsPerCTA();
 503:   unsigned axis = getAxis();
 504:   return ceil<unsigned>(
 505:       getShape()[axis],
 506:       (contigPerThread[axis] * threadsPerWarp[axis] * warpsPerCTA[axis]));
 507: }
```

- **EN:** Defines accessor/helper `ScanLoweringHelper::getAxisNumBlocks` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `ScanLoweringHelper::getAxisNumBlocks`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 509-524

```cpp
 509: unsigned ScanLoweringHelper::getNonAxisNumBlocks() {
 510:   auto contigPerThread = getEncoding().getContigPerThread();
 511:   auto threadsPerWarp = getEncoding().getThreadsPerWarp();
 512:   auto warpsPerCTA = getEncoding().getWarpsPerCTA();
 513:   auto rank = contigPerThread.size();
 514:   unsigned axis = getAxis();
 515:   unsigned numBlocks = 1;
 516:   for (unsigned i = 0; i < rank; i++) {
 517:     if (i == axis)
 518:       continue;
 519:     numBlocks *=
 520:         ceil<unsigned>(getShape()[i], (contigPerThread[i] * threadsPerWarp[i] *
 521:                                        warpsPerCTA[i]));
 522:   }
 523:   return numBlocks;
 524: }
```

- **EN:** Defines accessor/helper `ScanLoweringHelper::getNonAxisNumBlocks` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `ScanLoweringHelper::getNonAxisNumBlocks`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 526-532

```cpp
 526: bool ScanLoweringHelper::isSupported() {
 527:   // TODO: Support the following cases:
 528:   // 1. Scan on non-blocking encodings
 529:   if (!isa<BlockedEncodingAttr>(legacyEncoding))
 530:     return false;
 531:   return true;
 532: }
```

- **EN:** Defines `ScanLoweringHelper::isSupported`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `ScanLoweringHelper::isSupported`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 534-541

```cpp
 534: unsigned ScanLoweringHelper::getScratchSizeInElems() {
 535:   unsigned numWarps = product(getEncoding().getWarpsPerCTA());
 536:   unsigned numNonAxisElementsPerWarp =
 537:       getNonAxisNumThreadsPerWarp() * getNonAxisNumElementsPerThread();
 538:   unsigned numElements = numWarps * numNonAxisElementsPerWarp *
 539:                          getAxisNumBlocks() * getNonAxisNumBlocks();
 540:   return numElements;
 541: }
```

- **EN:** Defines accessor/helper `ScanLoweringHelper::getScratchSizeInElems` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `ScanLoweringHelper::getScratchSizeInElems`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 543-546

```cpp
 543: unsigned ScanLoweringHelper::getScratchSizeInBytes() {
 544:   // Lowering will fail later if the layout is not supported.
 545:   if (!isSupported())
 546:     return 0;
```

- **EN:** Defines accessor/helper `ScanLoweringHelper::getScratchSizeInBytes` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `ScanLoweringHelper::getScratchSizeInBytes`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 548-556

```cpp
 548:   unsigned axisNumWarps = getAxisNumWarpsWithUniqueData();
 549:   if (axisNumWarps == 1)
 550:     return 0;
 551:   unsigned elementSizeInBytes = 0;
 552:   for (const auto &ty : srcElementTypes) {
 553:     elementSizeInBytes += ceil<unsigned>(ty.getIntOrFloatBitWidth(), 8);
 554:   }
 555:   return elementSizeInBytes * getScratchSizeInElems();
 556: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 558-561

```cpp
 558: static SmallVector<DecomposedWarpConversion::TranspositionInfo>
 559: getTranspositionSelectors(SmallVector<std::pair<int, int>> &mixedTranspositions,
 560:                           std::vector<std::vector<int32_t>> &regBases,
 561:                           int bitwidth);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 563-580

```cpp
 563: DecomposedWarpConversion
 564: getWarpLayoutConvertDecomposition(RankedTensorType srcTy,
 565:                                   RankedTensorType dstTy, int bitwidth) {
 566:   // Two layouts, ll_src and ll_dst, representing the same tensor can be
 567:   // viewed as surjections of GF(2) vector spaces:
 568:   //
 569:   //            ll_src: H_src -> M   and   ll_dst: H_dst -> M,
 570:   //
 571:   // where each is represented by a 'subpermutation' matrix, i.e., a permutation
 572:   // matrix with zero columns possibly inserted. A layout conversion can be
 573:   // viewed as a map P': H_src -> H_dst which factors ll_src = ll_dst \circ P'.
 574:   //
 575:   // For a conversion not needing data movement between different warps, we
 576:   // choose the following representation, where P is a permutation matrix and
 577:   // K_1 and K_2 are (possibly trivial) spaces meant to ensure equally sized
 578:   // lane and register dimensions between layouts:
 579:   //                                  P
 580:   //     H_src -> H_src \oplus K_1 -------> H_dst \oplus K_2 -> H_dst.
```

- **EN:** Defines accessor/helper `getWarpLayoutConvertDecomposition` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getWarpLayoutConvertDecomposition`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 581-598

```cpp
 581:   //
 582:   // As a permutation, P can be viewed as a product of cycles permuting lane and
 583:   // register index bits. Any such permutation can be expressed as a composition
 584:   //
 585:   //                    P = P_mixed \circ P_lane \circ P_reg,
 586:   //
 587:   // where P_mixed is a product of disjoint transpositions (r_i l_j) between
 588:   // lane and register bits and where P_lane and P_reg are permutations purely
 589:   // involving lane bits and register bits, respectively. Such a representation
 590:   // is not unique, and we choose the factorization method which slices out
 591:   // subsequences of consecutive lane bits from cycles involving both bit types.
 592:   // Further explanation of this method is below.
 593:   //
 594:   // The decomposition is performed in three stages. First, we compute the
 595:   // permutation matrix `P` by using `invertAndCompose` to generate a skeleton
 596:   // and then fill in any zero columns. Second, we walk the cycles of `P` to
 597:   // factor out mixed transpositions to build `mixedTranspositions`, `pReg`, and
 598:   // `pLane`. Finally, we determine any selectors needed for byte permute
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 599-599

```cpp
 599:   // instructions in place of `selp` instructions when packing registers.
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 601-610

```cpp
 601:   // We remove any broadcasting in the register dimensions of the layouts before
 602:   // forming the permutation `P` as the components of the decomposition directly
 603:   // inform the number of emitted instructions, and leaving broadcasting in
 604:   // would unnecessarily inflate the count.
 605:   auto srcLayout = toLinearLayout(srcTy);
 606:   auto dstLayout = toLinearLayout(dstTy);
 607:   auto removeBroadcastSrc = actionRemoveBroadcastedRegs(srcLayout);
 608:   auto removeBroadcastDst = actionRemoveBroadcastedRegs(dstLayout);
 609:   srcLayout = removeBroadcastSrc.apply(srcLayout);
 610:   dstLayout = removeBroadcastDst.apply(dstLayout);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 612-618

```cpp
 612:   // We want to describe the conversion from `srcLayout` to `dstLayout` as a
 613:   // permutation. Since this requires that each input dimension have the same
 614:   // size in each of the layouts, we first pad the lane and register dimensions
 615:   // with zero vectors if needed.
 616:   auto *ctx = srcTy.getContext();
 617:   StringAttr kReg = StringAttr::get(ctx, "register");
 618:   StringAttr kLane = StringAttr::get(ctx, "lane");
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 620-637

```cpp
 620:   // Determine the target sizes of the register and lane dimensions for padding.
 621:   int nSrcRegBases = srcLayout.getInDimSizeLog2(kReg);
 622:   int nDstRegBases = dstLayout.getInDimSizeLog2(kReg);
 623:   int nSrcLaneBases = srcLayout.getInDimSizeLog2(kLane);
 624:   int nDstLaneBases = dstLayout.getInDimSizeLog2(kLane);
 625:   int nRegBases = std::max(nSrcRegBases, nDstRegBases);
 626:   int nLaneBases = std::max(nSrcLaneBases, nDstLaneBases);
 627:   // Restrict attention to the input dimensions which matter.
 628:   SmallVector<StringAttr> inDimNames{kReg, kLane};
 629:   auto outDimNames = llvm::to_vector(srcLayout.getOutDimNames());
 630:   auto S = srcLayout.sublayout(inDimNames, outDimNames);
 631:   auto T = dstLayout.sublayout(inDimNames, outDimNames);
 632:   // Conditionally pad.
 633:   if (nSrcRegBases != nDstRegBases || nSrcLaneBases != nDstLaneBases) {
 634:     auto padWithZeros = [&](const LinearLayout &ll) {
 635:       auto newBases = ll.getBases();
 636:       auto padDim = [&](StringAttr dim, int dimSize) {
 637:         auto &dimBases = newBases[dim];
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 638-651

```cpp
 638:         dimBases.reserve(dimSize);
 639:         for (int i = ll.getInDimSizeLog2(dim); i < dimSize; ++i)
 640:           dimBases.emplace_back(outDimNames.size(), 0);
 641:       };
 642:       padDim(kReg, nRegBases);
 643:       padDim(kLane, nLaneBases);
 644:       // Surjectivity is not expected in general since we do not consider
 645:       // the 'warp' and 'block' dimensions of the original layouts.
 646:       return LinearLayout(std::move(newBases), ll.getOutDims(),
 647:                           /*requireSurjective=*/false);
 648:     };
 649:     S = padWithZeros(S);
 650:     T = padWithZeros(T);
 651:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 653-656

```cpp
 653:   // We compute T^transpose \circ S, which serves as a skeleton for `P`, then
 654:   // fill in zero columns, prioritizing producing fixed points. As we only need
 655:   // the basis vectors of `P`, we never actually produce the LinearLayout.
 656:   auto pBases = S.invertAndCompose(T).getBases();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 658-675

```cpp
 658:   // Find the common and uncommon zeros of S and T
 659:   S = S.flattenOuts();
 660:   T = T.flattenOuts();
 661:   SmallVector<std::pair<int32_t, int32_t>> srcFreeZeros;
 662:   SmallVector<std::pair<int32_t, int32_t>> dstFreeZeros;
 663:   for (auto [dimIdx, dim] : llvm::enumerate(inDimNames)) {
 664:     for (int inIdx = 0; inIdx < S.getInDimSizeLog2(dim); ++inIdx) {
 665:       int sVal = S.getBasis(dim, inIdx)[0];
 666:       int tVal = T.getBasis(dim, inIdx)[0];
 667:       if (sVal == 0 && tVal == 0) {
 668:         pBases[dim][inIdx][dimIdx] = 1 << inIdx;
 669:       } else if (sVal == 0) {
 670:         srcFreeZeros.emplace_back(dimIdx, inIdx);
 671:       } else if (tVal == 0) {
 672:         dstFreeZeros.emplace_back(dimIdx, inIdx);
 673:       }
 674:     }
 675:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 676-682

```cpp
 676:   // Fill in non-fixed-point zero vectors
 677:   for (auto [srcZeroLoc, dstZeroLoc] : llvm::zip(srcFreeZeros, dstFreeZeros)) {
 678:     auto [srcDimIdx, srcIdx] = srcZeroLoc;
 679:     auto [dstDimIdx, dstIdx] = dstZeroLoc;
 680:     auto inDim = inDimNames[srcDimIdx];
 681:     pBases[inDim][srcIdx][dstDimIdx] = 1 << dstIdx;
 682:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 684-693

```cpp
 684:   // We walk the cycles of `P` to build the bases for `pReg` and `pLane` while
 685:   // factoring out mixed transpositions from cycles that include both register
 686:   // and lane basis vectors. `pReg` and `pLane` themselves only have one input
 687:   // and output dimension each.
 688:   LinearLayout::BasesT pRegBases, pLaneBases;
 689:   auto &regBases = pRegBases[kReg];
 690:   auto &laneBases = pLaneBases[kLane];
 691:   regBases.resize(nRegBases, {0});
 692:   laneBases.resize(nLaneBases, {0});
 693:   SmallVector<std::pair<int, int>> mixedTranspositions;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 695-698

```cpp
 695:   llvm::BitVector visited(nRegBases + nLaneBases, false);
 696:   auto flatIdx = [&](StringAttr dim, int32_t index) {
 697:     return (dim == kReg) ? index : nRegBases + index;
 698:   };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 700-704

```cpp
 700:   for (auto dim : inDimNames) {
 701:     int inDimSize = S.getInDimSizeLog2(dim);
 702:     for (int i = 0; i < inDimSize; ++i) {
 703:       if (visited.test(flatIdx(dim, i)))
 704:         continue;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 706-711

```cpp
 706:       // Start a new cycle, tracking the entry basis vector and the 'current'
 707:       // one as we walk the cycle.
 708:       StringAttr entryDim = dim;
 709:       int32_t entryIdx = i;
 710:       StringAttr currDim = entryDim;
 711:       int32_t currIdx = entryIdx;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 713-727

```cpp
 713:       // We slice out subsequences of consecutive lane basis vectors appearing
 714:       // in mixed cycles by factoring out transpositions (r_i l_j) as in
 715:       //
 716:       // (.. r_m l_j .. l_k r_i ..) = (r_i l_j) * (.. r_m r_i ..)(l_j .. l_k).
 717:       //
 718:       // The permutations are applied right-to-left, and the block `l_j .. l_k`
 719:       // indicates a contiguous subsequence of lane basis vectors. Note that the
 720:       // transposition does not commute with the other two cycles.
 721:       //
 722:       // The following variables are used to track the start and end points of
 723:       // such subsequences.
 724:       int32_t /*r_m*/ regStartIdx = -1;
 725:       int32_t /*l_j*/ laneStartIdx = -1;
 726:       int32_t /*l_k*/ laneEndIdx = -1;
 727:       int32_t /*r_i*/ regEndIdx = -1;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 729-745

```cpp
 729:       do {
 730:         // Determine the next basis vector in the current cycle.
 731:         visited.set(flatIdx(currDim, currIdx));
 732:         auto nextVec = pBases.lookup(currDim)[currIdx];
 733:         StringAttr nextDim;
 734:         int32_t nextIdx;
 735:         for (auto [nextDimIdx, nextVal] : llvm::enumerate(nextVec)) {
 736:           if (nextVal != 0) {
 737:             nextDim = inDimNames[nextDimIdx];
 738:             nextIdx = llvm::Log2_32(nextVal);
 739:           }
 740:         }
 741:         // Set a `pReg` or `pLane` vector, or mark an r->l or l->r transition.
 742:         if (currDim == kReg && nextDim == kReg) {
 743:           regBases[currIdx][0] = 1 << nextIdx;
 744:         } else if (currDim == kLane && nextDim == kLane) {
 745:           laneBases[currIdx][0] = 1 << nextIdx;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 746-763

```cpp
 746:         } else if (currDim == kReg && nextDim == kLane) {
 747:           regStartIdx = currIdx;
 748:           laneStartIdx = nextIdx;
 749:         } else {
 750:           regEndIdx = nextIdx;
 751:           laneEndIdx = currIdx;
 752:         }
 753:         // If a subsequence of the form (.. r_m l_j .. l_k r_i ..) has been
 754:         // found, perform the prescribed factorization.
 755:         if (regEndIdx >= 0) {
 756:           // Assign r_m to map to r_i as in (.. r_m r_i ..).
 757:           regBases[regStartIdx][0] = 1 << regEndIdx;
 758:           // Assign l_k to map to l_j as in (l_j .. l_k).
 759:           laneBases[laneEndIdx][0] = 1 << laneStartIdx;
 760:           // Record (r_i l_j) as a factor.
 761:           mixedTranspositions.emplace_back(regEndIdx, laneStartIdx);
 762:           // Reset the auxiliary variables.
 763:           regStartIdx = laneStartIdx = laneEndIdx = regEndIdx = -1;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 764-764

```cpp
 764:         }
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 766-771

```cpp
 766:         currDim = nextDim;
 767:         currIdx = nextIdx;
 768:       } while (flatIdx(currDim, currIdx) != flatIdx(entryDim, entryIdx));
 769:     }
 770:   }
 771:   assert(visited.all() && "Cycle walk incomplete");
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 773-778

```cpp
 773:   // Determine degree of packing and selectors.
 774:   int m = mixedTranspositions.size();
 775:   int nPackPrelim = llvm::Log2_32(std::clamp(32 / bitwidth, 1, 4));
 776:   int nPack = std::min(nPackPrelim, nRegBases - m);
 777:   auto processedTranspos =
 778:       getTranspositionSelectors(mixedTranspositions, regBases, nPack);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 780-786

```cpp
 780:   auto pReg = LinearLayout(std::move(pRegBases), {{kReg, 1 << nRegBases}},
 781:                            /*requireSurjective=*/true);
 782:   auto pLane = LinearLayout(std::move(pLaneBases), {{kLane, 1 << nLaneBases}},
 783:                             /*requireSurjective=*/true);
 784:   return {std::move(pReg), std::move(pLane), std::move(processedTranspos),
 785:           nPack};
 786: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 788-795

```cpp
 788: static SmallVector<DecomposedWarpConversion::TranspositionInfo>
 789: getTranspositionSelectors(SmallVector<std::pair<int, int>> &mixedTranspositions,
 790:                           std::vector<std::vector<int32_t>> &regBases,
 791:                           int nPack) {
 792:   // When possible, we fuse permutations of 'low' register bits together
 793:   // with a mixed transposition, resulting in byte permute instructions instead
 794:   // of `select` instructions. After processing, no low register bits appear in
 795:   // the returned list of mixed transpositions.
```

- **EN:** Defines accessor/helper `getTranspositionSelectors` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getTranspositionSelectors`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 797-814

```cpp
 797:   SmallVector<DecomposedWarpConversion::TranspositionInfo> ret;
 798:   ret.reserve(mixedTranspositions.size());
 799:   if (nPack == 0) {
 800:     for (auto &t : mixedTranspositions)
 801:       ret.push_back(DecomposedWarpConversion::TranspositionInfo{t});
 802:     return ret;
 803:   }
 804:   // This algorithm performs further algebraic processing.
 805:   //
 806:   // Suppose nPack > 0 and for simplicity that P is a cycle. We are given an
 807:   // initial decomposition P = pMixed * pLane * pReg. A (mixed) transposition
 808:   // involving an intra-register bit, say (r0 l0), can equivalently be
 809:   // expressed as (rN r0)(rN l0)(rN r0) for N >= nPack. The lowering of (rN l0)
 810:   // involves decomposing the permutation into three linear transformations,
 811:   // with the first and third transformations expressible as tiles of prmt
 812:   // instructions. The effect of (rN r0) is to permute the values of the
 813:   // default selectors, 0x7654 and 0x3210, viewing N as bit 2 of the values.
 814:   //
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 815-832

```cpp
 815:   // This rewrite does not address the presence of intra-register bits in pReg,
 816:   // which often causes extra instructions to be generated. The goal of this
 817:   // algorithm is to peel off intra-register bits occuring in pReg whenever
 818:   // possible and to attach them to some mixed transposition while being mindful
 819:   // of the ordering of these extracted and conjugating factors for correctness.
 820:   //
 821:   // For a concrete example, consider P := (l0 r0 r1 l1 r2) with nPack = 2.
 822:   // Initially, we have P = (r2 l1)(r0 l0) * (r0 r1 r2) (= pMixed * pReg).
 823:   // We can reorder the two transpositions and decompose pReg to get
 824:   //
 825:   //                P = (r0 l0)(r2 l1) * (r2 r1)(r2 r0).
 826:   //
 827:   // We see (r2 r1)(r2 r0) can act as prmt selector modifiers for (r2 l1), and
 828:   // for (r0 l0), we can pick an unused register bit, such as r3, and write
 829:   // (r0 l0) = (r3 r0)(r3 l0)(r3 r0). This gives us our two TranspositionInfo
 830:   // objects:
 831:   //
 832:   //            P = (r3 r0)(r3 l0)(r3 r0) * (r2 l1)(r2 r1)(r2 r0).
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 833-847

```cpp
 833:   //
 834:   // Since r0 is used in both TranspositionInfo objects, we must maintain this
 835:   // relative order (right term applied first) for equality to hold. However,
 836:   // since the conjugating term of the left term, (r3 r0), is disjoint from
 837:   // (r2 l1), the lowering algorithm is still valid. Roughly, it performs:
 838:   //
 839:   //           prmt      selp         shfl         prmt          prmt
 840:   //   P  =  (r3 r0) * (r2 r2) * (r3 l0)(r2 l1) * (r3 r0) * (r2 r1)(r2 r0).
 841:   //
 842:   // The selector algorithm processes transpositions sequentially, excising
 843:   // intra-register bits from pReg or conjugating mixed transpositions by higher
 844:   // "partner" register bits as in the example. In noncommutative cases, we
 845:   // reorder the transposition list to produce the factorization which places
 846:   // low-bit modifiers responsible for the ordering constraint on the
 847:   // pre-shuffle side.
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 849-866

```cpp
 849:   auto permuteSelector = [nPack](uint16_t sel, int bitIdx) {
 850:     // Swap bit 2 and bit `lo` of the nibbles in `sel`.
 851:     int lo = bitIdx + (2 - nPack);
 852:     uint16_t maskHi = 0x4444;
 853:     uint16_t maskLo = 0x1111 << lo;
 854:     uint16_t fixed = sel & ~maskHi & ~maskLo;
 855:     int shift = 2 - lo;
 856:     return fixed | ((maskHi & sel) >> shift) | ((maskLo & sel) << shift);
 857:   };
 858:   auto generateSelectors = [&](int head, int tail, auto &&lowBits) {
 859:     uint16_t topSel = 0x3210;
 860:     uint16_t botSel = 0x7654;
 861:     for (auto lowBit : lowBits) {
 862:       topSel = permuteSelector(topSel, lowBit);
 863:       botSel = permuteSelector(botSel, lowBit);
 864:       // Fix the low bits between `tail` and `head` in pReg.
 865:       if (lowBit != head && lowBit != tail)
 866:         regBases[lowBit][0] = 1 << lowBit;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 867-869

```cpp
 867:     }
 868:     return std::pair{topSel, botSel};
 869:   };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 871-873

```cpp
 871:   llvm::SmallSet<int32_t, 6> pairedRegBits;
 872:   for (auto [rBit, lBit] : mixedTranspositions)
 873:     pairedRegBits.insert(rBit);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 875-892

```cpp
 875:   // A low bit in a mixed transposition must be replaced by a high bit. The
 876:   // choice of high bit can affect instruction count. If the first high bit
 877:   // found when walking along `pReg` is unpaired, then that bit is the best
 878:   // choice. We reorder the transpositions to guarantee this during processing.
 879:   // This also guarantees the correct ordering for the lowering algorithm.
 880:   auto next = [&](int b) { return llvm::Log2_32(regBases[b][0]); };
 881:   auto nextHighFree = [&](auto p) {
 882:     int curr = p.first;
 883:     do {
 884:       if (curr >= nPack)
 885:         return true;
 886:       curr = next(curr);
 887:     } while (!pairedRegBits.contains(curr));
 888:     return false;
 889:   };
 890:   std::stable_partition(mixedTranspositions.begin(), mixedTranspositions.end(),
 891:                         nextHighFree);
 892:   // If `P` has an isolated low-bit mixed transposition, and `pReg` maps a low
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 893-910

```cpp
 893:   // bit to an open high bit, then the high bit should be used as the partner.
 894:   // This folds cases like P = (r0 l0)(r2 r1).
 895:   auto prev = [&](int b) {
 896:     int tail = b;
 897:     int curr = next(b);
 898:     while (curr != b) {
 899:       tail = curr;
 900:       curr = next(curr);
 901:     }
 902:     return tail;
 903:   };
 904:   auto findPartner = [&](int lowBit, auto &preShufLoBits) {
 905:     if (nPack == 2) {
 906:       int otherLow = 1 - lowBit;
 907:       int b = next(otherLow);
 908:       if (next(lowBit) == lowBit && b >= nPack && !pairedRegBits.contains(b) &&
 909:           !pairedRegBits.contains(otherLow)) {
 910:         preShufLoBits.push_back(otherLow);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 911-921

```cpp
 911:         regBases[prev(otherLow)][0] = 1 << b;
 912:         pairedRegBits.insert(b);
 913:         return b;
 914:       }
 915:     }
 916:     int potentialPartner = nPack;
 917:     while (pairedRegBits.contains(potentialPartner))
 918:       ++potentialPartner;
 919:     pairedRegBits.insert(potentialPartner);
 920:     return potentialPartner;
 921:   };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 923-931

```cpp
 923:   for (auto p : mixedTranspositions) {
 924:     int rBit = p.first;
 925:     int lBit = p.second;
 926:     SmallVector<int> cycle;
 927:     int currBit = rBit;
 928:     do {
 929:       cycle.push_back(currBit);
 930:       currBit = next(currBit);
 931:     } while (currBit != rBit);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 933-948

```cpp
 933:     // Walk forward and backward along the current `cycle` from `rBit` until we
 934:     // reach a boundary (either a high bit or a different paired low bit). The
 935:     // low-bit segment on the forward side becomes the post-shuffle selector
 936:     // modifiers, while those on the backward side become pre-shuffle modifiers.
 937:     auto isBoundary = [&](int bit) {
 938:       return bit >= nPack || (pairedRegBits.contains(bit) && bit != rBit);
 939:     };
 940:     auto forwardEnd = llvm::find_if(cycle, isBoundary);
 941:     auto backwardEnd = std::find_if(cycle.rbegin(), cycle.rend(), isBoundary);
 942:     SmallVector<int> postShufLoBits(cycle.begin(), forwardEnd);
 943:     SmallVector<int> preShufLoBits(cycle.rbegin(), backwardEnd);
 944:     // We slice out a segment of low bits (.. `tail` .. `head` ..) from pReg by
 945:     // setting `tail` -> `head` and fixing the low bits in between.
 946:     int head;
 947:     int tail;
 948:     int partnerBit = -1;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 950-967

```cpp
 950:     // Determine selector modifiers and low bit excision from pReg.
 951:     if (forwardEnd == cycle.end()) {
 952:       // Isolated low bits with single mixed transposition. E.g. (l0 r0 r1)
 953:       if (cycle.size() == 2)
 954:         postShufLoBits.pop_back();
 955:       head = tail = cycle.back();
 956:     } else if (*forwardEnd < nPack) {
 957:       // End at a different paired low bit. E.g. (l0 r0 l1 r1)
 958:       head = rBit;
 959:       tail = next(head);
 960:       preShufLoBits.push_back(head);
 961:     } else {
 962:       // End at original or unpaired high bit. E.g. (l0 r0 r2) or (l0 r2)
 963:       head = partnerBit = *forwardEnd;
 964:       pairedRegBits.insert(partnerBit);
 965:       tail = *backwardEnd;
 966:       while (tail < nPack && pairedRegBits.contains(tail)) {
 967:         // Leading factor in a noncommutative case.
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 968-971

```cpp
 968:         preShufLoBits.push_back(tail);
 969:         tail = prev(tail);
 970:       }
 971:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 973-978

```cpp
 973:     if (partnerBit < 0)
 974:       partnerBit = findPartner(head, preShufLoBits);
 975:     auto [topPostSel, botPostSel] =
 976:         generateSelectors(head, tail, llvm::reverse(postShufLoBits));
 977:     auto [topPreSel, botPreSel] = generateSelectors(head, tail, preShufLoBits);
 978:     regBases[tail][0] = 1 << head;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 980-985

```cpp
 980:     DecomposedWarpConversion::TranspositionInfo info;
 981:     info.transposition = {partnerBit, lBit};
 982:     info.topPreSel = topPreSel;
 983:     info.botPreSel = botPreSel;
 984:     info.topPostSel = topPostSel;
 985:     info.botPostSel = botPostSel;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 987-1000

```cpp
 987:     ret.push_back(info);
 988:   }
 989:   if (nPack == 2 && regBases[0][0] == 2 && regBases[1][0] == 1 && ret.size()) {
 990:     // If (r0 r1) remains in pReg, fold it into a mixed transposition.
 991:     auto &t = ret.front();
 992:     for (int lowBit : {0, 1, 0}) {
 993:       t.topPreSel = permuteSelector(t.topPreSel, lowBit);
 994:       t.botPreSel = permuteSelector(t.botPreSel, lowBit);
 995:     }
 996:     regBases[0][0] = 1;
 997:     regBases[1][0] = 2;
 998:   }
 999:   return ret;
1000: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1002-1005

```cpp
1002: SmallVector<std::pair<SmallVector<int64_t>, SmallVector<int64_t>>>
1003: getReshapeDecomposition(ArrayRef<int64_t> srcShape,
1004:                         ArrayRef<int64_t> dstShape) {
1005:   SmallVector<std::pair<SmallVector<int64_t>, SmallVector<int64_t>>> ret;
```

- **EN:** Defines accessor/helper `getReshapeDecomposition` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getReshapeDecomposition`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 1007-1011

```cpp
1007:   if (srcShape.empty()) {
1008:     assert(dstShape.empty());
1009:     return ret;
1010:   }
1011:   ret.push_back({});
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 1013-1030

```cpp
1013:   int srcIdx = 0;
1014:   int dstIdx = 0;
1015:   int srcNElems = 1;
1016:   int dstNElems = 1;
1017:   while (srcIdx < srcShape.size() || dstIdx < dstShape.size()) {
1018:     if (srcNElems < dstNElems || //
1019:         (srcIdx < srcShape.size() && srcNElems == 1) ||
1020:         (srcIdx < srcShape.size() && srcShape[srcIdx] == 1)) {
1021:       assert(srcIdx < srcShape.size());
1022:       srcNElems *= srcShape[srcIdx];
1023:       ret.back().first.push_back(srcIdx);
1024:       srcIdx++;
1025:     } else if (dstNElems < srcNElems ||
1026:                (dstIdx < dstShape.size() && dstShape[dstIdx] == 1)) {
1027:       assert(dstIdx < dstShape.size());
1028:       dstNElems *= dstShape[dstIdx];
1029:       ret.back().second.push_back(dstIdx);
1030:       dstIdx++;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 1031-1038

```cpp
1031:     } else {
1032:       ret.push_back({});
1033:       srcNElems = 1;
1034:       dstNElems = 1;
1035:     }
1036:   }
1037:   return ret;
1038: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1040-1049

```cpp
1040: unsigned ScanLoweringHelper::getAxisElementStride() {
1041:   auto order = getOrder();
1042:   unsigned stride = 1;
1043:   for (unsigned dim : order) {
1044:     if (dim == getAxis())
1045:       return stride;
1046:     stride *= getEncoding().getContigPerThread()[dim];
1047:   }
1048:   llvm_unreachable("Axis not found in order");
1049: }
```

- **EN:** Defines accessor/helper `ScanLoweringHelper::getAxisElementStride` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `ScanLoweringHelper::getAxisElementStride`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 1051-1062

```cpp
1051: unsigned ScanLoweringHelper::getAxisThreadStride() {
1052:   auto encoding = getEncoding();
1053:   auto ll = encoding.getLinearLayout();
1054:   auto kThread = StringAttr::get(encoding.getContext(), "lane");
1055:   const auto &bases = ll.getBases().lookup(kThread);
1056:   unsigned axis = getAxis();
1057:   for (unsigned i = 0; i < bases.size(); ++i) {
1058:     if (bases[i][axis] != 0)
1059:       return 1 << i;
1060:   }
1061:   return 1;
1062: }
```

- **EN:** Defines accessor/helper `ScanLoweringHelper::getAxisThreadStride` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `ScanLoweringHelper::getAxisThreadStride`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 1064-1078

```cpp
1064: unsigned ScanLoweringHelper::getAxisBlockStride() {
1065:   auto order = getOrder();
1066:   unsigned stride = 1;
1067:   auto contigPerThread = getEncoding().getContigPerThread();
1068:   auto threadsPerWarp = getEncoding().getThreadsPerWarp();
1069:   auto warpsPerCTA = getEncoding().getWarpsPerCTA();
1070:   for (unsigned dim : order) {
1071:     if (dim == getAxis())
1072:       return stride;
1073:     stride *= ceil<unsigned int>(getShape()[dim], contigPerThread[dim] *
1074:                                                       threadsPerWarp[dim] *
1075:                                                       warpsPerCTA[dim]);
1076:   }
1077:   llvm_unreachable("Axis not found in order");
1078: }
```

- **EN:** Defines accessor/helper `ScanLoweringHelper::getAxisBlockStride` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `ScanLoweringHelper::getAxisBlockStride`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 1080-1081

```cpp
1080: GatherLoweringHelper::GatherLoweringHelper(triton::GatherOp gatherOp)
1081:     : gatherOp(gatherOp) {}
```

- **EN:** Defines helper `GatherLoweringHelper::GatherLoweringHelper` that performs a lowering step from Triton/MLIR semantics to a lower-level representation.
- **CN:** 这里定义辅助函数 `GatherLoweringHelper::GatherLoweringHelper`，负责把 Triton/MLIR 语义中的一个步骤降级到更底层表示。
### Lines 1083-1086

```cpp
1083: unsigned GatherLoweringHelper::getScratchSizeInBytes() {
1084:   // If the gather is warp-local, no scratch space is needed.
1085:   if (isWarpLocal())
1086:     return 0;
```

- **EN:** Defines accessor/helper `GatherLoweringHelper::getScratchSizeInBytes` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `GatherLoweringHelper::getScratchSizeInBytes`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 1088-1094

```cpp
1088:   // Otherwise, performing the gather will require scratch space to communicate
1089:   // the source tensor across threads. For now, assume the whole source tensor
1090:   // is written back to shared memory.
1091:   RankedTensorType srcType = gatherOp.getSrc().getType();
1092:   return product(srcType.getShape()) *
1093:          ceil<unsigned>(srcType.getElementTypeBitWidth(), 8);
1094: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1096-1102

```cpp
1096: bool GatherLoweringHelper::isWarpLocal() {
1097:   // The gather is warp-local if for each column along the gather axis in the
1098:   // source and index tensors, all the elements are owned by the same warp.
1099:   RankedTensorType srcType = gatherOp.getSrc().getType();
1100:   RankedTensorType idxType = gatherOp.getIndices().getType();
1101:   LinearLayout srcLayout = toLinearLayout(srcType);
1102:   LinearLayout idxLayout = toLinearLayout(idxType);
```

- **EN:** Defines `GatherLoweringHelper::isWarpLocal`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `GatherLoweringHelper::isWarpLocal`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1104-1109

```cpp
1104:   Builder b(gatherOp.getContext());
1105:   StringAttr kBlock = b.getStringAttr("block");
1106:   StringAttr kWarp = b.getStringAttr("warp");
1107:   StringAttr kLane = b.getStringAttr("lane");
1108:   StringAttr kGatherDim =
1109:       b.getStringAttr("dim" + std::to_string(gatherOp.getAxis()));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1111-1127

```cpp
1111:   // The tensor layouts must be distributed layouts, where the basis matrix is a
1112:   // subpermutation matrix (permutation matrix plus zeros for broadcasting).
1113:   // FIXME(jeff): Check this invariant somehow.
1114:   //
1115:   // We want to know if all elements of a column along the gather axis are
1116:   // mapped to the same set of warps, which means the gather can be performed
1117:   // entirely within the warp. We need to query
1118:   //
1119:   //   srcLayout.invert().sublayoutIsZero({kGatherDim}, {kBlock, kWarp})
1120:   //
1121:   // But due to broadcasting, the matrix might not be invertible. But since the
1122:   // matrix is a permutation matrix (checked below), we can instead query
1123:   //
1124:   //   srcLayout.sublayoutIsZero({kBlock, kWarp}, {kGatherDim})
1125:   //
1126:   // Which implies that changing the warp will not change the gather dimension.
1127:   // And since there is no swizzling, this applies to all warps.
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 1128-1130

```cpp
1128:   if (!srcLayout.sublayoutIsZero({kBlock, kWarp}, kGatherDim) ||
1129:       !idxLayout.sublayoutIsZero({kBlock, kWarp}, kGatherDim))
1130:     return false;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1132-1137

```cpp
1132:   SmallVector<StringAttr> otherDims;
1133:   for (unsigned dim = 0, rank = srcType.getRank(); dim < rank; ++dim) {
1134:     if (dim != gatherOp.getAxis()) {
1135:       otherDims.push_back(b.getStringAttr("dim" + Twine(dim)));
1136:     }
1137:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 1139-1145

```cpp
1139:   // If the gather axis `dimN` is invariant to the warp, but the `(block, warp)`
1140:   // mapping to all other dimensions must be the same for both layouts. If so,
1141:   // then the warp that owns a particular index element also owns all the source
1142:   // elements it could index into.
1143:   if (srcLayout.sublayout({kBlock, kWarp}, otherDims) !=
1144:       idxLayout.sublayout({kBlock, kWarp}, otherDims))
1145:     return false;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1147-1149

```cpp
1147:   // The two constraints above ensure that data-movement to perform the gather
1148:   // operation are contained within a warp. The subsequent constraints simplify
1149:   // codegen.
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 1151-1157

```cpp
1151:   // Require that for any given gather column, the threads mapped to the column
1152:   // in the index and source tensors are the same. This means we don't need to
1153:   // xor shuffle across threads before emitting index shuffles; we push warp
1154:   // shuffling to layout conversions.
1155:   return srcLayout.sublayout(kLane, otherDims) ==
1156:          idxLayout.sublayout(kLane, otherDims);
1157: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1159-1163

```cpp
1159: unsigned getNumScratchElements(ArrayRef<unsigned> shape) {
1160:   if (shape.empty())
1161:     return 0;
1162:   return product<unsigned>(shape);
1163: }
```

- **EN:** Defines accessor/helper `getNumScratchElements` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getNumScratchElements`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 1165-1182

```cpp
1165: bool supportMMA(triton::DotOp op, int version) {
1166:   // Refer to mma section for the data type supported by Volta and Hopper
1167:   // Tensor Core in
1168:   // https://docs.nvidia.com/cuda/parallel-thread-execution/index.html#warp-level-matrix-fragment-mma-884-f16
1169:   auto aElemTy = op.getA().getType().getElementType();
1170:   auto bElemTy = op.getB().getType().getElementType();
1171:   if (aElemTy.isF32() && bElemTy.isF32() &&
1172:       op.getInputPrecision() != InputPrecision::TF32)
1173:     return false;
1174:   if (version == 5) {
1175:     if (triton::tools::getBoolEnv("DISABLE_MMA_V5"))
1176:       return false;
1177:     RankedTensorType typeA = op.getA().getType();
1178:     int k = typeA.getShape().back();
1179:     auto retType = op.getType();
1180:     auto retShapePerCTA = getShapePerCTA(retType);
1181:     auto rank = retShapePerCTA.size();
1182:     int numWarps = lookupNumWarps(op);
```

- **EN:** Defines `supportMMA`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `supportMMA`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1183-1198

```cpp
1183:     // Allow int8 * int8 -> int32 for MMAv5, reject other integer combinations
1184:     if (aElemTy.isInteger() || bElemTy.isInteger() ||
1185:         retType.getElementType().isInteger()) {
1186:       if (!aElemTy.isInteger(8) || !bElemTy.isInteger(8) ||
1187:           !retType.getElementType().isInteger(32))
1188:         return false;
1189:     }
1190:     if (op.getType().getRank() != 2)
1191:       return false;
1192:     if (numWarps != 4 && numWarps != 8) {
1193:       // Currently only support numWarps 4 or 8 for TMEM load and store.
1194:       return false;
1195:     }
1196:     // If k size is smaller than the native mma size, we cannot use MMA.
1197:     if (k < 256 / aElemTy.getIntOrFloatBitWidth())
1198:       return false;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1199-1215

```cpp
1199:     if (!(retShapePerCTA[rank - 2] % 64 == 0 &&
1200:           retShapePerCTA[rank - 1] % 8 == 0))
1201:       return false;
1202:     if (aElemTy.isF64() || bElemTy.isF64() ||
1203:         retType.getElementType().isF64()) {
1204:       // tcgen05.mma doesn't support F64.
1205:       return false;
1206:     }
1207:     return true;
1208:   }
1209:   if (version == 3) {
1210:     if (triton::tools::getBoolEnv("DISABLE_MMA_V3"))
1211:       return false;
1212:     auto retType = op.getType();
1213:     RankedTensorType typeA = op.getA().getType();
1214:     int k = typeA.getShape().back();
1215:     // If k size is smaller than the native mma size, we cannot use MMA.
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1216-1231

```cpp
1216:     if (k < 256 / aElemTy.getIntOrFloatBitWidth())
1217:       return false;
1218:     auto retShapePerCTA = getShapePerCTA(retType);
1219:     auto rank = retShapePerCTA.size();
1220:     int numWarps = lookupNumWarps(op);
1221:     // TODO(Keren): for now, fallback to MMAv2 if handling batch matmul.
1222:     if (rank == 3)
1223:       return false;
1224:     if (!(numWarps % 4 == 0 && retShapePerCTA[rank - 2] % 64 == 0 &&
1225:           retShapePerCTA[rank - 1] % 8 == 0 &&
1226:           (llvm::isa<Float8E5M2Type, Float8E4M3FNType>(aElemTy) ||
1227:            aElemTy.isInteger(8) || aElemTy.isF16() || aElemTy.isBF16() ||
1228:            aElemTy.isF32()))) {
1229:       return false;
1230:     }
1231:     // We cannot use MMA_V3 if we need to accumulate in F32 within the MMA op.
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1232-1243

```cpp
1232:     if (op.getMaxNumImpreciseAcc() < 32 &&
1233:         (llvm::isa<Float8E5M2Type, Float8E4M3FNType>(aElemTy)) &&
1234:         cast<RankedTensorType>(op.getType()).getElementType().isF32()) {
1235:       return false;
1236:     }
1237:   }
1238:   if (aElemTy.isF32() && bElemTy.isF32()) {
1239:     assert(op.getInputPrecision() == InputPrecision::TF32);
1240:     return version >= 2;
1241:   }
1242:   return supportMMA(op.getA(), version) && supportMMA(op.getB(), version);
1243: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 1245-1260

```cpp
1245: bool supportMMA(Value value, int version) {
1246:   // Tell whether a DotOp support MMA by the operand type(either $a or $b).
1247:   // We cannot get both the operand types(in TypeConverter), here we assume the
1248:   // types of both the operands are identical here.
1249:   assert((version == 1 || version == 2 || version == 3) &&
1250:          "Unexpected MMA layout version found");
1251:   auto elemTy =
1252:       cast<triton::gpu::TensorOrMemDesc>(value.getType()).getElementType();
1253:   // FP8 is not natively supported on all mma versions but it can always be
1254:   // promoted to fp16 therefore we can always support it.
1255:   bool isFP8 = llvm::isa<Float8E5M2Type, Float8E4M3FNType, Float8E5M2FNUZType,
1256:                          Float8E4M3FNUZType>(elemTy);
1257:   return isFP8 || elemTy.isF16() || elemTy.isBF16() ||
1258:          ((elemTy.isF32() || elemTy.isF64()) && version >= 2) ||
1259:          (elemTy.isInteger(8) && version >= 2);
1260: }
```

- **EN:** Defines `supportMMA`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `supportMMA`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 1262-1278

```cpp
1262: // We get the smallest submap of srcTy^{-1} * dstTy that is not the identity
1263: // under the common dimensions. The idea here is that if we have a
1264: // transformation that's the identity on kBlock, we don't need to use
1265: // distributed shared memory. If it's also the identity on kWarp, we can
1266: // transfer via warp-shuffles, and if it's the identity on kLane just have to
1267: // reorder the registers.
1268: LinearLayout minimalCvtLayout(Type srcTy_, Type dstTy_) {
1269:   auto srcTy = cast<triton::gpu::TensorOrMemDesc>(srcTy_);
1270:   auto dstTy = cast<triton::gpu::TensorOrMemDesc>(dstTy_);
1271:   LinearLayout srcLayout = toLinearLayout(srcTy);
1272:   LinearLayout dstLayout = toLinearLayout(dstTy);
1273:   auto sDims = to_vector(srcLayout.getInDimNames());
1274:   auto dDims = to_vector(dstLayout.getInDimNames());
1275:   SmallVector<StringAttr> dims;
1276:   for (int i = 0; i < std::min(sDims.size(), dDims.size()); ++i) {
1277:     auto srcDim = sDims[sDims.size() - i - 1];
1278:     auto dstDim = dDims[dDims.size() - i - 1];
```

- **EN:** Defines `minimalCvtLayout`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `minimalCvtLayout`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 1279-1283

```cpp
1279:     if (srcDim != dstDim) {
1280:       break;
1281:     }
1282:     dims.push_back(srcDim);
1283:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1285-1295

```cpp
1285:   auto comp = dstLayout.invertAndCompose(srcLayout);
1286:   // We try to quotient by the slowers moving subspace first
1287:   for (auto dim : dims) {
1288:     auto quotient = comp.quotient(dim);
1289:     if (!quotient.has_value()) {
1290:       break;
1291:     }
1292:     comp = *quotient;
1293:   }
1294:   return comp;
1295: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1297-1303

```cpp
1297: bool cvtReordersRegisters(RankedTensorType srcTy, RankedTensorType dstTy) {
1298:   auto layout = minimalCvtLayout(srcTy, dstTy);
1299:   MLIRContext *ctx = srcTy.getContext();
1300:   auto kRegister = StringAttr::get(ctx, "register");
1301:   auto outDims = to_vector(layout.getOutDimNames());
1302:   return outDims.empty() || ArrayRef(outDims) == ArrayRef({kRegister});
1303: }
```

- **EN:** Defines `cvtReordersRegisters`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `cvtReordersRegisters`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1305-1316

```cpp
1305: bool cvtNeedsWarpShuffle(RankedTensorType srcTy, RankedTensorType dstTy) {
1306:   auto layout = minimalCvtLayout(srcTy, dstTy);
1307:   MLIRContext *ctx = srcTy.getContext();
1308:   auto kRegister = StringAttr::get(ctx, "register");
1309:   auto kLane = StringAttr::get(ctx, "lane");
1310:   if (to_vector(layout.getOutDimNames()) ==
1311:       SmallVector<StringAttr, 2>{kRegister, kLane}) {
1312:     auto factors = getWarpLayoutConvertDecomposition(srcTy, dstTy, 32);
1313:     return (factors.mixedTranspositions.size() < 2);
1314:   }
1315:   return false;
1316: }
```

- **EN:** Defines `cvtNeedsWarpShuffle`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `cvtNeedsWarpShuffle`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1318-1321

```cpp
1318: bool cvtNeedsSharedMemory(RankedTensorType srcTy, RankedTensorType dstTy) {
1319:   return !cvtReordersRegisters(srcTy, dstTy) &&
1320:          !cvtNeedsWarpShuffle(srcTy, dstTy);
1321: }
```

- **EN:** Defines `cvtNeedsSharedMemory`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `cvtNeedsSharedMemory`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1323-1328

```cpp
1323: std::unique_ptr<DataFlowSolver> createDataFlowSolver() {
1324:   auto solver = std::make_unique<DataFlowSolver>();
1325:   solver->load<dataflow::DeadCodeAnalysis>();
1326:   solver->load<dataflow::SparseConstantPropagation>();
1327:   return solver;
1328: }
```

- **EN:** Defines helper `createDataFlowSolver` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createDataFlowSolver`，用于计算或构造外围变换所需的中间数据。
### Lines 1330-1345

```cpp
1330: bool isCvtDimSync(const triton::LinearLayout &srcLayout,
1331:                   const triton::LinearLayout &dstLayout, StringAttr dim) {
1332:   // We can use a dimension-level sync when the conversion is trivial over that
1333:   // dimension and there is no broadcasting over it.
1334:   auto *ctx = srcLayout.getInDimNames().begin()->getContext();
1335:   auto kWarp = StringAttr::get(ctx, "warp");
1336:   auto kBlock = StringAttr::get(ctx, "block");
1337:   assert(srcLayout.hasInDim(dim) && dstLayout.hasInDim(dim) &&
1338:          "expected dim to be present in both layouts");
1339:   auto comp = dstLayout.invertAndCompose(srcLayout);
1340:   if (dim == kWarp) {
1341:     // We check that it's trivial over block and warps and that
1342:     // there is no broadcasting over warp, as if there is, we'll
1343:     // deduplicate the writes and the reads will read from data
1344:     // that other warp has written.
1345:     auto isBlockSync = isCvtDimSync(srcLayout, dstLayout, kBlock);
```

- **EN:** Defines `isCvtDimSync`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `isCvtDimSync`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 1346-1354

```cpp
1346:     return isBlockSync && comp.isTrivialOver(dim) &&
1347:            srcLayout.getFreeVariableMasks()[dim] == 0 &&
1348:            dstLayout.getFreeVariableMasks()[dim] == 0;
1349:   } else {
1350:     assert(dim == kBlock);
1351:     return comp.isTrivialOver(dim);
1352:   }
1353: }
1354: } // namespace mlir
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。

## Key Concepts / 关键概念
- **EN:** The main theme is static analysis around utility, so correctness depends on conservative fact propagation.
  **CN:** 主线是围绕 Utility 的静态分析，因此正确性依赖保守的信息传播。
- **EN:** Tensor shape and element-type reasoning is central to the implementation.
  **CN:** 张量形状与元素类型推导是实现中的核心内容。
- **EN:** Linear layout utilities translate between logical tensor coordinates and physical placement.
  **CN:** 线性布局工具负责在逻辑张量坐标与物理放置之间进行转换。
- **EN:** Encoding attributes describe how distributed GPU data is laid out across threads and warps.
  **CN:** 编码属性描述 GPU 分布式数据在线程与 warp 之间的布局方式。
- **EN:** Dataflow analysis tracks facts across operations to make conservative optimization decisions.
  **CN:** 数据流分析会跨操作跟踪事实，以做出保守但有效的优化决策。
- **EN:** Tensor memory specific allocation, layout, or synchronization rules are important here.
  **CN:** 这里重点处理张量内存（TMEM）的分配、布局或同步规则。

## Dependencies / 依赖关系
- **Internal Triton headers / Triton 内部头文件:** `triton/Analysis/Utility.h`, `triton/Analysis/Allocation.h`, `triton/Conversion/MLIRTypes.h`, `triton/Dialect/Triton/IR/Dialect.h`, `triton/Dialect/Triton/IR/Utility.h`, `triton/Dialect/TritonGPU/IR/Dialect.h`, ... (+4 more)
- **MLIR headers / MLIR 头文件:** `mlir/Analysis/DataFlow/ConstantPropagationAnalysis.h`, `mlir/Analysis/DataFlow/DeadCodeAnalysis.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/LLVMIR/LLVMDialect.h`, `mlir/IR/Dialect.h`, `mlir/IR/Matchers.h`, ... (+1 more)
- **LLVM headers / LLVM 头文件:** `llvm/ADT/SmallSet.h`
- **Standard/library headers / 标准或通用库头文件:** `deque`
- **Primary APIs used / 主要 API:** `RankedTensorType`, `LinearLayout`, `OpBuilder`
