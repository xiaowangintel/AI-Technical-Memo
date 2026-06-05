# ConcurrencySanitizer.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `lib/Dialect/TritonInstrument/Transforms/ConcurrencySanitizer.cpp`
- **Purpose / 作用:** **EN:** Implements the Concurrency Sanitizer transformation or optimization pass for the TritonInstrument pipeline. **CN:** 为 TritonInstrument 编译流程实现与 Concurrency Sanitizer 相关的变换或优化 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11

```cpp
   1: #include "mlir/Analysis/SliceAnalysis.h"
   2: #include "mlir/IR/ImplicitLocOpBuilder.h"
   3: #include "mlir/Transforms/Passes.h"
   4: #include "triton/Dialect/Triton/IR/Utility.h"
   5: #include "triton/Dialect/TritonGPU/IR/Dialect.h"
   6: #include "triton/Dialect/TritonInstrument/IR/Dialect.h"
   7: #include "triton/Dialect/TritonInstrument/IR/FunctionBuilder.h"
   8: #include "triton/Dialect/TritonInstrument/IR/Utility.h"
   9: #include "triton/Dialect/TritonInstrument/Transforms/ConSanTargetHooks.h"
  10: #include "triton/Dialect/TritonNvidiaGPU/IR/Dialect.h"
  11: #include "triton/Tools/Sys/GetEnv.h"
```

- **EN:** Includes the interfaces this file depends on. Triton headers (`Utility.h`, `Dialect.h`, `Dialect.h`, `FunctionBuilder.h`, ... (+4 more)) provide domain-specific IR/support, MLIR headers (`SliceAnalysis.h`, `ImplicitLocOpBuilder.h`, `Passes.h`) provide rewriting and analysis infrastructure, LLVM headers (None) supply low-level utilities, and standard/library headers (None) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（`Utility.h`, `Dialect.h`, `Dialect.h`, `FunctionBuilder.h`, ... (+4 more)）提供领域特定 IR/支持逻辑，MLIR 头文件（`SliceAnalysis.h`, `ImplicitLocOpBuilder.h`, `Passes.h`）提供重写与分析基础设施，LLVM 头文件（None）提供底层工具，而标准/通用库头文件（None）提供通用能力。
### Lines 13-15

```cpp
  13: namespace mlir {
  14: namespace triton {
  15: namespace instrument {
```

- **EN:** Opens or closes the namespace nesting for mlir -> triton -> instrument, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 mlir -> triton -> instrument 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 17-20

```cpp
  17: namespace tt = mlir::triton;
  18: namespace ttg = mlir::triton::gpu;
  19: namespace ttng = mlir::triton::nvidia_gpu;
  20: namespace tti = mlir::triton::instrument;
```

- **EN:** Opens or closes the namespace nesting for tt -> ttg -> ttng -> tti, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 tt -> ttg -> ttng -> tti 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 22-23

```cpp
  22: #define GEN_PASS_DEF_TRITONINSTRUMENTCONCURRENCYSANITIZER
  23: #include "triton/Dialect/TritonInstrument/Transforms/Passes.h.inc"
```

- **EN:** Pulls in generated definitions from TableGen/MLIR include fragments so the handwritten code can reuse auto-generated declarations.
- **CN:** 这里引入由 TableGen/MLIR 生成的定义片段，使手写代码能够复用自动生成的声明。
### Lines 25-28

```cpp
  25: static llvm::StringMap<ConSanHooksFactory> &getHooksRegistry() {
  26:   static llvm::StringMap<ConSanHooksFactory> registry;
  27:   return registry;
  28: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 30-32

```cpp
  30: void registerConSanHooks(llvm::StringRef key, ConSanHooksFactory factory) {
  31:   getHooksRegistry()[key] = std::move(factory);
  32: }
```

- **EN:** Defines `registerConSanHooks`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `registerConSanHooks`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 34-39

```cpp
  34: std::unique_ptr<ConSanTargetHooks> createConSanHooks(llvm::StringRef key) {
  35:   auto it = getHooksRegistry().find(key);
  36:   if (it != getHooksRegistry().end())
  37:     return it->second();
  38:   return nullptr;
  39: }
```

- **EN:** Defines helper `createConSanHooks` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createConSanHooks`，用于计算或构造外围变换所需的中间数据。
### Lines 41-41

```cpp
  41: namespace {
```

- **EN:** Opens or closes the namespace nesting for (anonymous), keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 (anonymous) 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 43-60

```cpp
  43: // OpBuilder listener tracking operations added to the builder to be wrapped
  44: // with a lock acquire/release pair.
  45: class CriticalSectionListener : public ImplicitLocOpBuilder::Listener {
  46: public:
  47:   void notifyOperationInserted(Operation *op,
  48:                                OpBuilder::InsertPoint /*previous*/) override {
  49:     if (firstOp == nullptr) {
  50:       firstOp = op;
  51:     }
  52:     lastOp = op;
  53:   }
  54:   void maybeWrapWithCriticalSection(ImplicitLocOpBuilder &b,
  55:                                     AuxDataMap &auxData, Value pred) {
  56:     Operation *_firstOp = firstOp;
  57:     Operation *_lastOp = lastOp;
  58:     if (firstOp != nullptr && lastOp != nullptr) {
  59:       assert(firstOp->getParentRegion() == lastOp->getParentRegion());
  60:       b.setInsertionPoint(_firstOp);
```

- **EN:** Defines `CriticalSectionListener`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `CriticalSectionListener`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 61-67

```cpp
  61:       tti::ExperimentalLockAcquireOp::create(b, auxData.lock.at(_firstOp).value,
  62:                                              pred);
  63:       b.setInsertionPointAfter(_lastOp);
  64:       tti::ExperimentalLockReleaseOp::create(b, auxData.lock.at(_firstOp).value,
  65:                                              pred);
  66:     }
  67:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 69-72

```cpp
  69: private:
  70:   Operation *firstOp = nullptr;
  71:   Operation *lastOp = nullptr;
  72: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 74-77

```cpp
  74: bool isTensorCoreOp(Operation *op) {
  75:   return isa<ttng::MMAv5OpInterface, ttng::TCGen5CommitOp, ttng::TMEMCopyOp>(
  76:       op);
  77: }
```

- **EN:** Defines `isTensorCoreOp`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `isTensorCoreOp`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 79-87

```cpp
  79: std::optional<int> maybeGetPartitionIdx(Operation *op) {
  80:   if (auto wsOp = op->getParentOfType<ttg::WarpSpecializePartitionsOp>()) {
  81:     return op->getParentRegion()->getRegionNumber();
  82:   }
  83:   if (Operation *parent = op->getParentOp()) {
  84:     return maybeGetPartitionIdx(parent);
  85:   }
  86:   return std::nullopt;
  87: }
```

- **EN:** Defines `maybeGetPartitionIdx`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `maybeGetPartitionIdx`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 89-104

```cpp
  89: int getCurrentThread(Operation *op, const ConSanTargetHooks *hooks,
  90:                      const AuxDataMap::ThreadLayout &threadLayout) {
  91:   // Default partition is 0, other partitions are idx + 1
  92:   int thread = maybeGetPartitionIdx(op).value_or(-1) + 1;
  93:   if (hooks->isTMAOp(op)) {
  94:     assert(threadLayout.hasTMAThreads() &&
  95:            "TMA thread class must exist when instrumenting a TMA op");
  96:     thread += threadLayout.tmaThreadOffset;
  97:     return thread;
  98:   }
  99:   if (isTensorCoreOp(op)) {
 100:     assert(threadLayout.hasTCThreads() &&
 101:            "TC thread class must exist when instrumenting a tensor-core op");
 102:     thread += threadLayout.tcThreadOffset;
 103:     return thread;
 104:   }
```

- **EN:** Defines accessor/helper `getCurrentThread` that exposes or updates operation state in a compact, reusable way. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义访问器/辅助函数 `getCurrentThread`，以紧凑且可复用的方式读取或更新操作状态。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 105-112

```cpp
 105:   if (hooks->isCLCOp(op)) {
 106:     assert(threadLayout.hasCLCThreads() &&
 107:            "CLC thread class must exist when instrumenting a CLC op");
 108:     thread += threadLayout.clcThreadOffset;
 109:     return thread;
 110:   }
 111:   return thread;
 112: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 114-116

```cpp
 114: int getBaseThread(int thread, const AuxDataMap::ThreadLayout &threadLayout) {
 115:   return thread % threadLayout.numBaseThreads;
 116: }
```

- **EN:** Defines accessor/helper `getBaseThread` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getBaseThread`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 118-134

```cpp
 118: // Peer threads are the equivalent threads in the TMA, TC, CLC and normal
 119: // thread classes.
 120: // If a thread is a base thread, return the mask with the peers, otherwise
 121: // return the mask with the thread itself.
 122: uint64_t getThreadPeersMask(int thread,
 123:                             const AuxDataMap::ThreadLayout &threadLayout) {
 124:   uint64_t mask = 1ULL << thread;
 125:   if (thread < threadLayout.numBaseThreads) {
 126:     if (threadLayout.hasTMAThreads())
 127:       mask |= 1ULL << (thread + threadLayout.tmaThreadOffset);
 128:     if (threadLayout.hasTCThreads())
 129:       mask |= 1ULL << (thread + threadLayout.tcThreadOffset);
 130:     if (threadLayout.hasCLCThreads())
 131:       mask |= 1ULL << (thread + threadLayout.clcThreadOffset);
 132:   }
 133:   return mask;
 134: }
```

- **EN:** Defines accessor/helper `getThreadPeersMask` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getThreadPeersMask`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 136-141

```cpp
 136: int getActiveMask(ttg::WarpSpecializeOp wsOp) {
 137:   int activeMask = 1;
 138:   for (Region *region : wsOp.getNonEmptyPartitionRegions())
 139:     activeMask |= 1 << (region->getRegionNumber() + 1);
 140:   return activeMask;
 141: }
```

- **EN:** Defines accessor/helper `getActiveMask` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getActiveMask`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 143-147

```cpp
 143: Value currentCTAMask(ImplicitLocOpBuilder &b) {
 144:   Value ctaId = tti::ExperimentalClusterCTAIdOp::create(b, b.getLoc());
 145:   return arith::ShLIOp::create(b, arith::ConstantIntOp::create(b, 1, 32),
 146:                                ctaId);
 147: }
```

- **EN:** Defines `currentCTAMask`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `currentCTAMask`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 149-153

```cpp
 149: Value allCTAsMask(ImplicitLocOpBuilder &b) {
 150:   int numCTAs = ttg::lookupNumCTAs(b);
 151:   assert(numCTAs <= 16 && "ConSan CTA bitsets assume at most 16 CTAs");
 152:   return arith::ConstantIntOp::create(b, (1u << numCTAs) - 1, 32);
 153: }
```

- **EN:** Defines `allCTAsMask`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `allCTAsMask`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 155-162

```cpp
 155: bool shouldInitializeAllocations() {
 156:   std::string envValue = tt::tools::getStrEnv("TRITON_CONSAN_INIT_ALLOCATIONS");
 157:   if (envValue.empty())
 158:     return true;
 159:   if (auto enabled = tt::tools::isEnvValueBool(envValue))
 160:     return *enabled;
 161:   llvm::report_fatal_error("TRITON_CONSAN_INIT_ALLOCATIONS must be a boolean");
 162: }
```

- **EN:** Defines `shouldInitializeAllocations`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `shouldInitializeAllocations`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 164-176

```cpp
 164: llvm::APInt getIntegerNaNPattern(unsigned bitWidth) {
 165:   switch (bitWidth) {
 166:   case 16:
 167:     // 0x7FC0 is a NaN in both bfloat16 and float16 interpretations.
 168:     return llvm::APInt(16, 0x7FC0);
 169:   case 32:
 170:     return llvm::APFloat::getNaN(llvm::APFloat::IEEEsingle()).bitcastToAPInt();
 171:   case 64:
 172:     return llvm::APFloat::getNaN(llvm::APFloat::IEEEdouble()).bitcastToAPInt();
 173:   default:
 174:     return llvm::APInt::getAllOnes(bitWidth);
 175:   }
 176: }
```

- **EN:** Defines accessor/helper `getIntegerNaNPattern` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getIntegerNaNPattern`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 178-195

```cpp
 178: Value createPoisonTensor(ImplicitLocOpBuilder &b,
 179:                          ttg::MemDescType memDescType) {
 180:   auto region = b.getInsertionBlock()->getParent();
 181:   Type elementType = memDescType.getElementType();
 182:   RankedTensorType poisonType;
 183:   if (isa<ttng::TensorMemorySpaceAttr>(memDescType.getMemorySpace())) {
 184:     auto encoding = ttng::getDefaultLayoutForTmemLdSt(
 185:         memDescType, ttg::lookupNumWarps(region));
 186:     poisonType =
 187:         RankedTensorType::get(memDescType.getShape(), elementType, encoding);
 188:   } else {
 189:     auto encoding = ttg::getDefaultBlockedEncoding(
 190:         b.getContext(), memDescType.getShape(), ttg::lookupNumWarps(region),
 191:         ttg::lookupThreadsPerWarp(b), ttg::lookupNumCTAs(b));
 192:     encoding = ttg::BlockedEncodingAttr::get(
 193:         b.getContext(), encoding.getSizePerThread(),
 194:         encoding.getThreadsPerWarp(), encoding.getWarpsPerCTA(),
 195:         encoding.getOrder(), ttg::getCGALayout(memDescType.getEncoding()));
```

- **EN:** Defines helper `createPoisonTensor` that computes or constructs intermediate data used by the surrounding transformation. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic.
- **CN:** 这里定义辅助函数 `createPoisonTensor`，用于计算或构造外围变换所需的中间数据。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。
### Lines 196-198

```cpp
 196:     poisonType =
 197:         RankedTensorType::get(memDescType.getShape(), elementType, encoding);
 198:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 200-212

```cpp
 200:   DenseElementsAttr poison;
 201:   if (auto floatType = dyn_cast<FloatType>(elementType)) {
 202:     poison = DenseElementsAttr::get(
 203:         poisonType, llvm::APFloat::getNaN(floatType.getFloatSemantics()));
 204:   } else if (auto integerType = dyn_cast<IntegerType>(elementType)) {
 205:     poison = DenseElementsAttr::get(
 206:         poisonType, getIntegerNaNPattern(integerType.getWidth()));
 207:   } else {
 208:     llvm::report_fatal_error(
 209:         "ConSan allocation initialization expects integer or float elements");
 210:   }
 211:   return arith::ConstantOp::create(b, b.getLoc(), poisonType, poison);
 212: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 214-224

```cpp
 214: Value createSingleBufferView(ImplicitLocOpBuilder &b, Value alloc,
 215:                              int64_t buffer) {
 216:   auto allocType = cast<ttg::MemDescType>(alloc.getType());
 217:   SmallVector<int64_t> shape(allocType.getShape().begin() + 1,
 218:                              allocType.getShape().end());
 219:   auto viewType = ttg::MemDescType::get(
 220:       shape, allocType.getElementType(), allocType.getEncoding(),
 221:       allocType.getMemorySpace(), allocType.getMutableMemory());
 222:   Value index = arith::ConstantIntOp::create(b, buffer, 32);
 223:   return ttg::MemDescIndexOp::create(b, b.getLoc(), viewType, alloc, index);
 224: }
```

- **EN:** Defines helper `createSingleBufferView` that computes or constructs intermediate data used by the surrounding transformation. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic.
- **CN:** 这里定义辅助函数 `createSingleBufferView`，用于计算或构造外围变换所需的中间数据。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。
### Lines 226-236

```cpp
 226: void initializeAllocation(ImplicitLocOpBuilder &b, Value alloc) {
 227:   auto allocType = cast<ttg::MemDescType>(alloc.getType());
 228:   SmallVector<Value> leaves;
 229:   unsigned storeRank = allocType.getRank();
 230:   if (isa<ttng::TensorMemorySpaceAttr>(allocType.getMemorySpace())) {
 231:     storeRank = 2;
 232:   } else {
 233:     auto encoding = dyn_cast<ttg::LayoutEncodingTrait>(allocType.getEncoding());
 234:     assert(encoding && "shared allocation must have a layout encoding");
 235:     storeRank = encoding.getRank();
 236:   }
```

- **EN:** Defines `initializeAllocation`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `initializeAllocation`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 238-245

```cpp
 238:   if (allocType.getRank() == storeRank) {
 239:     leaves.push_back(alloc);
 240:   } else {
 241:     assert(allocType.getRank() == storeRank + 1 &&
 242:            "only single-dimension multibuffer allocations are supported");
 243:     for (int64_t buffer = 0; buffer < allocType.getDimSize(0); ++buffer)
 244:       leaves.push_back(createSingleBufferView(b, alloc, buffer));
 245:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 247-258

```cpp
 247:   for (Value leaf : leaves) {
 248:     Value poison =
 249:         createPoisonTensor(b, cast<ttg::MemDescType>(leaf.getType()));
 250:     if (isa<ttng::TensorMemorySpaceAttr>(
 251:             cast<ttg::MemDescType>(leaf.getType()).getMemorySpace())) {
 252:       Value pred = arith::ConstantIntOp::create(b, 1, 1);
 253:       ttng::TMEMStoreOp::create(b, leaf, poison, pred);
 254:     } else {
 255:       ttg::LocalStoreOp::create(b, poison, leaf);
 256:     }
 257:   }
 258: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。
### Lines 260-266

```cpp
 260: bool canInitializeAllocation(Value alloc) {
 261:   auto allocType = cast<ttg::MemDescType>(alloc.getType());
 262:   if (!isa<ttng::TensorMemorySpaceAttr>(allocType.getMemorySpace()))
 263:     return true;
 264:   unsigned numWarps = ttg::lookupNumWarps(alloc.getDefiningOp());
 265:   return numWarps % 4 == 0;
 266: }
```

- **EN:** Defines `canInitializeAllocation`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic.
- **CN:** 这里定义 `canInitializeAllocation`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。
### Lines 268-272

```cpp
 268: uint16_t getBlockBroadcastMask(Value alloc) {
 269:   auto allocTy = cast<ttg::MemDescType>(alloc.getType());
 270:   auto kBlock = StringAttr::get(alloc.getContext(), "block");
 271:   return toLinearLayout(allocTy).getFreeVariableMasks().lookup(kBlock);
 272: }
```

- **EN:** Defines accessor/helper `getBlockBroadcastMask` that exposes or updates operation state in a compact, reusable way. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic.
- **CN:** 这里定义访问器/辅助函数 `getBlockBroadcastMask`，以紧凑且可复用的方式读取或更新操作状态。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。
### Lines 274-283

```cpp
 274: Value createCTABitset(ImplicitLocOpBuilder &b, uint32_t pattern,
 275:                       uint32_t baseMask) {
 276:   // Create a CTA bitset by shifting `pattern` by the non-broadcast CTA bits of
 277:   // the current CTA.
 278:   Value ctaId = tti::ExperimentalClusterCTAIdOp::create(b, b.getLoc());
 279:   Value base = arith::AndIOp::create(
 280:       b, ctaId, arith::ConstantIntOp::create(b, baseMask, 32));
 281:   return arith::ShLIOp::create(b, arith::ConstantIntOp::create(b, pattern, 32),
 282:                                base);
 283: }
```

- **EN:** Defines helper `createCTABitset` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createCTABitset`，用于计算或构造外围变换所需的中间数据。
### Lines 285-294

```cpp
 285: Value getMulticastRecipientCTAs(ImplicitLocOpBuilder &b, Value alloc) {
 286:   // Return the CTA rows touched by an alloc: current CTA for
 287:   // non-broadcast allocs, or all CTAs in the current multicast group.
 288:   uint16_t broadcastMask = getBlockBroadcastMask(alloc);
 289:   if (!broadcastMask)
 290:     return currentCTAMask(b);
 291:   int numCTAs = ttg::lookupNumCTAs(b);
 292:   auto encoding = ttng::getTMAMulticastMaskEncoding(numCTAs, broadcastMask);
 293:   return createCTABitset(b, encoding.pattern, encoding.fixedBits);
 294: }
```

- **EN:** Defines accessor/helper `getMulticastRecipientCTAs` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getMulticastRecipientCTAs`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 296-303

```cpp
 296: Value getLeaderCTA(ImplicitLocOpBuilder &b, Value barrier) {
 297:   uint16_t broadcastMask = getBlockBroadcastMask(barrier);
 298:   if (!broadcastMask)
 299:     return currentCTAMask(b);
 300:   int numCTAs = ttg::lookupNumCTAs(b);
 301:   auto encoding = ttng::getTMAMulticastMaskEncoding(numCTAs, broadcastMask);
 302:   return createCTABitset(b, /*pattern=*/1, encoding.fixedBits);
 303: }
```

- **EN:** Defines accessor/helper `getLeaderCTA` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getLeaderCTA`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 305-317

```cpp
 305: Value getMulticastBarrierRecipientCTAs(ImplicitLocOpBuilder &b, Value result,
 306:                                        Value barrier) {
 307:   uint32_t resultBroadcastMask = getBlockBroadcastMask(result);
 308:   uint32_t barrierBroadcastMask = getBlockBroadcastMask(barrier);
 309:   int numCTAs = ttg::lookupNumCTAs(b);
 310:   uint32_t recipientBroadcastMask =
 311:       resultBroadcastMask & ~barrierBroadcastMask & (numCTAs - 1);
 312:   auto encoding =
 313:       ttng::getTMAMulticastMaskEncoding(numCTAs, recipientBroadcastMask);
 314:   uint32_t baseMask =
 315:       ~(resultBroadcastMask | barrierBroadcastMask) & (numCTAs - 1);
 316:   return createCTABitset(b, encoding.pattern, baseMask);
 317: }
```

- **EN:** Defines accessor/helper `getMulticastBarrierRecipientCTAs` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getMulticastBarrierRecipientCTAs`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 319-322

```cpp
 319: Value getRecipientCTAsForBroadcastMasks(ImplicitLocOpBuilder &b,
 320:                                         ArrayRef<uint16_t> broadcastMasks) {
 321:   if (broadcastMasks.empty())
 322:     return currentCTAMask(b);
```

- **EN:** Defines accessor/helper `getRecipientCTAsForBroadcastMasks` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getRecipientCTAsForBroadcastMasks`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 324-341

```cpp
 324:   int numCTAs = ttg::lookupNumCTAs(b);
 325:   Value ctaId = tti::ExperimentalClusterCTAIdOp::create(b, b.getLoc());
 326:   Value recipientCTAs = arith::ConstantIntOp::create(b, 0, 32);
 327:   // Match eager tcgen05_commit lowering in
 328:   // DotOpToLLVM/MMAv5.cpp:createMMACommit: build one concrete recipient bitset
 329:   // per descriptor, then OR those bitsets.
 330:   for (uint16_t broadcastBits : broadcastMasks) {
 331:     // Compute the map that goes from cta_id to lead_cta_id (fixedBits)
 332:     // and the pattern that goes from cta_0 to its multicast group (pattern).
 333:     auto encoding = ttng::getTMAMulticastMaskEncoding(numCTAs, broadcastBits);
 334:     Value fixedBitsVal =
 335:         arith::ConstantIntOp::create(b, encoding.fixedBits, 32);
 336:     Value base = arith::AndIOp::create(b, ctaId, fixedBitsVal);
 337:     Value patternVal = arith::ConstantIntOp::create(b, encoding.pattern, 32);
 338:     Value descRecipientCTAs = arith::ShLIOp::create(b, patternVal, base);
 339:     recipientCTAs = arith::OrIOp::create(b, recipientCTAs, descRecipientCTAs);
 340:   }
 341:   return recipientCTAs;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 342-342

```cpp
 342: }
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 344-358

```cpp
 344: SmallVector<uint16_t> getTensorCoreBarrierBroadcastMasks(Operation *op) {
 345:   assert(isTensorCoreOp(op) && "expected a tensor-core op");
 346:   bool twoCTAs = ttng::getModuleTwoCTAs(op);
 347:   SmallVector<Value> commitDescs;
 348:   if (auto commitOp = dyn_cast<ttng::TCGen5CommitOp>(op)) {
 349:     llvm::append_range(commitDescs, commitOp.getDescs());
 350:   } else if (auto mmaOp = dyn_cast<ttng::MMAv5OpInterface>(op)) {
 351:     commitDescs = mmaOp.getCompletionDescs();
 352:   } else if (isa<ttng::TMEMCopyOp>(op)) {
 353:     // TMEMCopy does not have descs (empty)
 354:   } else {
 355:     llvm_unreachable("unknown tensor-core op");
 356:   }
 357:   return ttng::getCTABroadcastMasks(twoCTAs, commitDescs);
 358: }
```

- **EN:** Defines accessor/helper `getTensorCoreBarrierBroadcastMasks` that exposes or updates operation state in a compact, reusable way. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义访问器/辅助函数 `getTensorCoreBarrierBroadcastMasks`，以紧凑且可复用的方式读取或更新操作状态。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 360-372

```cpp
 360: Value getMemEffectCTAs(ImplicitLocOpBuilder &b, Operation *op) {
 361:   if (auto tmaLoad = dyn_cast<ttng::TMALoadLikeOpInterface>(op)) {
 362:     if (tmaLoad.getMulticast())
 363:       return getMulticastRecipientCTAs(b, tmaLoad.getResult());
 364:     return currentCTAMask(b);
 365:   }
 366:   if (isa<ttng::CLCTryCancelOp>(op))
 367:     return allCTAsMask(b);
 368:   if (isa<ttng::MMAv5OpInterface, ttng::TMEMCopyOp>(op))
 369:     return getRecipientCTAsForBroadcastMasks(
 370:         b, ttng::getCTABroadcastMasks(ttng::getModuleTwoCTAs(op), {}));
 371:   return currentCTAMask(b);
 372: }
```

- **EN:** Defines accessor/helper `getMemEffectCTAs` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getMemEffectCTAs`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 374-388

```cpp
 374: Value getBarrierRecipientCTAs(ImplicitLocOpBuilder &b, Operation *op) {
 375:   if (auto expectOp = dyn_cast<ttng::BarrierExpectOp>(op))
 376:     return getLeaderCTA(b, expectOp.getAlloc());
 377:   if (auto arriveOp = dyn_cast<ttng::ArriveBarrierOp>(op))
 378:     return getLeaderCTA(b, arriveOp.getAlloc());
 379:   if (auto arriveOp = dyn_cast<ttng::AsyncCopyMbarrierArriveOp>(op))
 380:     return getLeaderCTA(b, arriveOp.getBarrier());
 381:   if (auto tmaLoad = dyn_cast<ttng::TMALoadLikeOpInterface>(op)) {
 382:     if (tmaLoad.getMulticast())
 383:       return getMulticastBarrierRecipientCTAs(b, tmaLoad.getResult(),
 384:                                               tmaLoad.getBarrier());
 385:     return getLeaderCTA(b, tmaLoad.getBarrier());
 386:   }
 387:   if (isa<ttng::CLCTryCancelOp>(op))
 388:     return allCTAsMask(b);
```

- **EN:** Defines accessor/helper `getBarrierRecipientCTAs` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getBarrierRecipientCTAs`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 390-394

```cpp
 390:   if (isTensorCoreOp(op))
 391:     return getRecipientCTAsForBroadcastMasks(
 392:         b, getTensorCoreBarrierBroadcastMasks(op));
 393:   return currentCTAMask(b);
 394: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 396-399

```cpp
 396: class ConcurrencySanitizerImpl {
 397: public:
 398:   ConcurrencySanitizerImpl(ModuleOp module, const ConSanTargetHooks *hooks)
 399:       : module(module), hooks(hooks) {}
```

- **EN:** Defines `ConcurrencySanitizerImpl`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `ConcurrencySanitizerImpl`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 401-405

```cpp
 401:   LogicalResult run() {
 402:     tti::FunctionBuilder funcBuilder(module, auxData);
 403:     if (failed(auxData.populateAndPassToWarpSpecialize(module, funcBuilder,
 404:                                                        hooks)))
 405:       return failure();
```

- **EN:** Defines `run`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `run`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 407-407

```cpp
 407:     tt::FuncOp entryPoint = tti::getEntryPoint(module);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 409-414

```cpp
 409:     ImplicitLocOpBuilder b(entryPoint.getLoc(), entryPoint);
 410:     b.setInsertionPointToStart(&entryPoint.getBody().front());
 411:     instrumentMemoryOperations(b, funcBuilder);
 412:     initializeAllocations();
 413:     return success();
 414:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 416-419

```cpp
 416: private:
 417:   void initializeAllocations() {
 418:     if (!shouldInitializeAllocations())
 419:       return;
```

- **EN:** Defines `initializeAllocations`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `initializeAllocations`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 421-431

```cpp
 421:     SmallVector<Operation *> allocationsToInitialize;
 422:     module.walk([&](Operation *op) {
 423:       if (auto alloc = dyn_cast<ttg::LocalAllocOp>(op)) {
 424:         if (!alloc.getSrc())
 425:           allocationsToInitialize.push_back(op);
 426:       }
 427:       if (auto alloc = dyn_cast<ttng::TMEMAllocOp>(op)) {
 428:         if (!alloc.getSrc())
 429:           allocationsToInitialize.push_back(op);
 430:       }
 431:     });
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. The implementation traverses IR operations to collect facts or apply rewrites globally.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。
### Lines 433-440

```cpp
 433:     for (Operation *op : allocationsToInitialize) {
 434:       ImplicitLocOpBuilder b(op->getLoc(), op);
 435:       b.setInsertionPointAfter(op);
 436:       Value alloc = op->getResult(0);
 437:       if (canInitializeAllocation(alloc))
 438:         initializeAllocation(b, alloc);
 439:     }
 440:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 442-446

```cpp
 442:   void instrumentMemoryOperations(ImplicitLocOpBuilder &b,
 443:                                   tti::FunctionBuilder &funcBuilder) {
 444:     module.walk([&](Operation *op) {
 445:       CriticalSectionListener listener;
 446:       b.setListener(&listener);
```

- **EN:** Defines `instrumentMemoryOperations`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. The implementation traverses IR operations to collect facts or apply rewrites globally.
- **CN:** 这里定义 `instrumentMemoryOperations`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。
### Lines 448-458

```cpp
 448:       int thread = getCurrentThread(op, hooks, auxData.threadLayout);
 449:       int baseThread = getBaseThread(thread, auxData.threadLayout);
 450:       b.setLoc(op->getLoc());
 451:       b.setInsertionPoint(op);
 452:       if (isa<ttg::LocalAllocOp, ttng::TMEMAllocOp>(op)) {
 453:         // Place insert point after specific ops:
 454:         // allocs - we want to
 455:         //   check if it is not overwriting any earlier allocation, but the
 456:         //   memref value can be referenced only after it is created.
 457:         b.setInsertionPointAfter(op);
 458:       }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 460-468

```cpp
 460:       if (auto info = hooks->getBarrierWaitInfo(op)) {
 461:         // For waits we want to instrument it before and after, so we do it
 462:         // manually inside instrumentBarrierWait (disable the critical section
 463:         // listener and return early)
 464:         b.setListener(nullptr);
 465:         instrumentBarrierWait(op, info->alloc, info->phase, info->pred, thread,
 466:                               baseThread, funcBuilder);
 467:         return;
 468:       }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 470-487

```cpp
 470:       instrumentMemEffects(b, op, thread, funcBuilder);
 471:       b.setLoc(op->getLoc());
 472:       if (auto wsOp = dyn_cast<ttg::WarpSpecializeOp>(op)) {
 473:         funcBuilder.createSetActiveMaskCall(b, getActiveMask(wsOp), op);
 474:         auto partitionRegions = wsOp.getNonEmptyPartitionRegions();
 475:         if (!partitionRegions.empty()) {
 476:           uint64_t destMask = 0;
 477:           for (Region *region : partitionRegions)
 478:             destMask |= getThreadPeersMask(region->getRegionNumber() + 1,
 479:                                            auxData.threadLayout);
 480:           if (destMask) {
 481:             for (MemType memType : {MemType::SHARED_MEM, MemType::TENSOR_MEM}) {
 482:               funcBuilder.createCopyWriteVisibilityCall(b, thread, destMask,
 483:                                                         nullptr, memType, op);
 484:               funcBuilder.createCopyReadVisibilityCall(b, thread, destMask,
 485:                                                        nullptr, memType, op);
 486:             }
 487:           }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 488-505

```cpp
 488:         }
 489:       }
 490:       if (auto info = hooks->getBarrierInitInfo(op)) {
 491:         Value pred = hooks->getIssuerCTAPred(b, op);
 492:         funcBuilder.createVerifyBarrierCanInitCall(b, info->alloc, pred, op,
 493:                                                    currentCTAMask(b));
 494:         funcBuilder.createInitBarrierStateCall(b, info->alloc, info->count,
 495:                                                pred, op);
 496:       }
 497:       if (auto info = hooks->getBarrierInvalidateInfo(op)) {
 498:         Value barrier = info->alloc;
 499:         Value pred = hooks->getIssuerCTAPred(b, op);
 500:         funcBuilder.createVerifyBarrierInitializedCall(b, barrier, pred, op,
 501:                                                        currentCTAMask(b));
 502:         funcBuilder.createInvalidateBarrierStateCall(b, barrier, pred, op);
 503:         for (MemType memType : {MemType::SHARED_MEM, MemType::TENSOR_MEM}) {
 504:           funcBuilder.createClearBarrierWriteTrackingCall(b, barrier, pred,
 505:                                                           memType, op);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 506-520

```cpp
 506:           funcBuilder.createClearBarrierReadTrackingCall(b, barrier, pred,
 507:                                                          memType, op);
 508:         }
 509:       }
 510:       if (auto asyncCommitGroupOp = dyn_cast<ttg::AsyncCommitGroupOp>(op)) {
 511:         if (!auxData.commits[CommitKind::AsyncCp].empty())
 512:           funcBuilder.createCommitAccessesCall(b, thread, nullptr,
 513:                                                CommitKind::AsyncCp, op);
 514:       }
 515:       if (auto asyncWaitOp = dyn_cast<ttg::AsyncWaitOp>(op)) {
 516:         funcBuilder.createClearOutstandingCommitsTransferWritesCall(
 517:             b, baseThread, getThreadPeersMask(thread, auxData.threadLayout),
 518:             asyncWaitOp.getNum(), nullptr, CommitKind::AsyncCp,
 519:             MemType::SHARED_MEM, op);
 520:       }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 521-537

```cpp
 521:       if (auto wgmmaWaitOp = dyn_cast<ttng::WarpGroupDotWaitOp>(op)) {
 522:         funcBuilder.createClearOutstandingCommitsTransferReadsCall(
 523:             b, baseThread, getThreadPeersMask(thread, auxData.threadLayout),
 524:             wgmmaWaitOp.getPendings(), nullptr, CommitKind::Wgmma,
 525:             MemType::SHARED_MEM, op);
 526:       }
 527:       if (auto info = hooks->getWaitOpInfo(op)) {
 528:         if (info->transferWrites && info->transferReads) {
 529:           funcBuilder.createClearOutstandingCommitsTransferBothCall(
 530:               b, baseThread, getThreadPeersMask(thread, auxData.threadLayout),
 531:               info->pendingCount, nullptr, info->commitKind,
 532:               MemType::SHARED_MEM, op);
 533:         } else if (info->transferWrites) {
 534:           funcBuilder.createClearOutstandingCommitsTransferWritesCall(
 535:               b, baseThread, getThreadPeersMask(thread, auxData.threadLayout),
 536:               info->pendingCount, nullptr, info->commitKind,
 537:               MemType::SHARED_MEM, op);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 538-555

```cpp
 538:         } else if (info->transferReads) {
 539:           funcBuilder.createClearOutstandingCommitsTransferReadsCall(
 540:               b, baseThread, getThreadPeersMask(thread, auxData.threadLayout),
 541:               info->pendingCount, nullptr, info->commitKind,
 542:               MemType::SHARED_MEM, op);
 543:         }
 544:       }
 545:       if (auto clusterBarrier = dyn_cast<ttng::ClusterBarrierOp>(op)) {
 546:         if (!clusterBarrier.getRelaxed() &&
 547:             !llvm::is_contained(auxData.nonPublishingClusterBarriers, op)) {
 548:           b.setInsertionPointAfter(op);
 549:           // Publish the cluster-wide frontier once, then keep every CTA at
 550:           // this synchronization point until the publication completes.
 551:           b.setListener(nullptr);
 552:           Value ctaId = tti::ExperimentalClusterCTAIdOp::create(b, b.getLoc());
 553:           Value zero = arith::ConstantIntOp::create(b, 0, 32);
 554:           Value isCTA0 =
 555:               arith::CmpIOp::create(b, arith::CmpIPredicate::eq, ctaId, zero);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 556-567

```cpp
 556:           Value lock = auxData.lock.at(op).value;
 557:           tti::ExperimentalLockAcquireOp::create(b, lock, isCTA0);
 558:           for (MemType memType : {MemType::SHARED_MEM, MemType::TENSOR_MEM})
 559:             funcBuilder.createPublishClusterVisibilityCall(b, isCTA0, memType,
 560:                                                            op);
 561:           tti::ExperimentalLockReleaseOp::create(b, lock, isCTA0);
 562:           auto publishBarrier = ttng::ClusterBarrierOp::create(b, b.getLoc());
 563:           auxData.nonPublishingClusterBarriers.push_back(
 564:               publishBarrier.getOperation());
 565:           b.setListener(&listener);
 566:         }
 567:       }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 569-586

```cpp
 569:       if (isa<ttg::WarpYieldOp, ttg::WarpReturnOp>(op) &&
 570:           !auxData.activeMasks.empty()) {
 571:         auto wsOp = op->getParentOfType<ttg::WarpSpecializeOp>();
 572:         bool shouldRetire =
 573:             isa<ttg::WarpYieldOp>(op) ||
 574:             llvm::is_contained(wsOp.getNonEmptyPartitionRegions(),
 575:                                op->getParentRegion());
 576:         if (shouldRetire) {
 577:           b.setLoc(wsOp.getLoc());
 578:           funcBuilder.createRetireActiveThreadCall(b, baseThread, op);
 579:           funcBuilder.createCheckAllActiveWaitingCall(b, nullptr, op);
 580:         }
 581:       }
 582:       if (isa<tt::ReturnOp>(op) && !auxData.activeMasks.empty() &&
 583:           op->getParentOfType<tt::FuncOp>() == tti::getEntryPoint(module)) {
 584:         funcBuilder.createSetActiveMaskCall(b, 0, op);
 585:         funcBuilder.createCheckAllActiveWaitingCall(b, nullptr, op);
 586:       }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 588-591

```cpp
 588:       listener.maybeWrapWithCriticalSection(b, auxData, nullptr);
 589:       b.setListener(nullptr);
 590:     });
 591:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 593-610

```cpp
 593:   void instrumentBarrierWait(Operation *op, Value alloc, Value phase,
 594:                              Value pred, int thread, int baseThread,
 595:                              tti::FunctionBuilder &funcBuilder) {
 596:     ImplicitLocOpBuilder wb(op->getLoc(), op);
 597:     pred = tti::maybeAnd(wb, pred, hooks->getIssuerCTAPred(wb, op));
 598:     Value lock = auxData.lock.at(op).value;
 599:     // Pre-wait: mark waiting threads and check for deadlock.
 600:     tti::ExperimentalLockAcquireOp::create(wb, lock, pred);
 601:     funcBuilder.createVerifyBarrierInitializedCall(wb, alloc, pred, op,
 602:                                                    currentCTAMask(wb));
 603:     funcBuilder.createSetWaitingCall(wb, alloc, baseThread, phase, pred, op);
 604:     funcBuilder.createCheckAllActiveWaitingCall(wb, pred, op);
 605:     tti::ExperimentalLockReleaseOp::create(wb, lock, pred);
 606:     // Post-wait: transfer visible writes and reads to all peer threads,
 607:     // and clear waiting for this barrier.
 608:     assert(!auxData.barriers.empty() &&
 609:            "barrier descriptors must exist when instrumenting wait");
 610:     wb.setInsertionPointAfter(op);
```

- **EN:** Defines `instrumentBarrierWait`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `instrumentBarrierWait`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 611-622

```cpp
 611:     tti::ExperimentalLockAcquireOp::create(wb, lock, pred);
 612:     for (MemType memType : {MemType::SHARED_MEM, MemType::TENSOR_MEM}) {
 613:       funcBuilder.createTransferVisibleWritesCall(
 614:           wb, alloc, getThreadPeersMask(thread, auxData.threadLayout), pred,
 615:           memType, op);
 616:       funcBuilder.createTransferVisibleReadsCall(
 617:           wb, alloc, getThreadPeersMask(thread, auxData.threadLayout), pred,
 618:           memType, op);
 619:     }
 620:     funcBuilder.createClearWaitingCall(wb, alloc, baseThread, pred, op);
 621:     tti::ExperimentalLockReleaseOp::create(wb, lock, pred);
 622:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 624-641

```cpp
 624:   void instrumentMemEffects(ImplicitLocOpBuilder &b, Operation *op, int thread,
 625:                             tti::FunctionBuilder &funcBuilder) {
 626:     int baseThread = getBaseThread(thread, auxData.threadLayout);
 627:     std::optional<MemEffectsOpInfo> opInfo = hooks->getMemEffectsOpInfo(op);
 628:     if (!opInfo) {
 629:       return;
 630:     }
 631:     Value pred = opInfo->pred;
 632:     Value issuerCTAPred = hooks->getIssuerCTAPred(b, op);
 633:     pred = tti::maybeAnd(b, pred, issuerCTAPred);
 634:     Value effectCTAs = getMemEffectCTAs(b, op);
 635:     for (auto effect : opInfo->operandEffects) {
 636:       Value buf = effect.buf;
 637:       auto bufType = cast<ttg::MemDescType>(buf.getType());
 638:       MemType memType = MemType::TENSOR_MEM;
 639:       if (isa<ttg::SharedEncodingTrait>(bufType.getEncoding())) {
 640:         memType = MemType::SHARED_MEM;
 641:       }
```

- **EN:** Defines `instrumentMemEffects`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic.
- **CN:** 这里定义 `instrumentMemEffects`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。
### Lines 642-659

```cpp
 642:       if (effect.rw == MemEffectsOpInfo::Effects::Read) {
 643:         // For op that is reading, we only need to check if anything else
 644:         // is writing to the same buffer.
 645:         addWriteChecks(b, funcBuilder, op, buf, effect.length, pred, memType,
 646:                        thread, effect.operandName, effectCTAs,
 647:                        opInfo->commitKind);
 648:         if (opInfo->trackingKind == MemEffectsOpInfo::TrackingKind::Barrier) {
 649:           funcBuilder.createSetReadVisibilityCall(
 650:               b, buf, effect.length,
 651:               getThreadPeersMask(thread, auxData.threadLayout), pred, memType,
 652:               op, effectCTAs);
 653:         }
 654:         if (opInfo->trackingKind ==
 655:             MemEffectsOpInfo::TrackingKind::CommitCount) {
 656:           assert(memType == MemType::SHARED_MEM);
 657:           funcBuilder.createStageAccessForCommitCall(b, buf, effect.length,
 658:                                                      baseThread, pred, memType,
 659:                                                      opInfo->commitKind, op);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 660-677

```cpp
 660:         }
 661:       }
 662:       if (effect.rw == MemEffectsOpInfo::Effects::Write) {
 663:         // Op is writing to the buffer, we need to check if anything else
 664:         // is reading or writing to the same buffer.
 665:         addWriteChecks(b, funcBuilder, op, buf, effect.length, pred, memType,
 666:                        thread, effect.operandName, effectCTAs,
 667:                        opInfo->commitKind);
 668:         addReadChecks(b, funcBuilder, op, buf, effect.length, pred, memType,
 669:                       thread, effect.operandName, effectCTAs,
 670:                       opInfo->commitKind);
 671:         if (opInfo->trackingKind == MemEffectsOpInfo::TrackingKind::Barrier) {
 672:           funcBuilder.createSetWriteVisibilityCall(
 673:               b, buf, effect.length,
 674:               getThreadPeersMask(thread, auxData.threadLayout), pred, memType,
 675:               op, effectCTAs);
 676:           funcBuilder.createClearWriteTrackingCall(b, buf, effect.length, pred,
 677:                                                    memType, op, effectCTAs);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 678-695

```cpp
 678:           funcBuilder.createClearReadVisibilityCall(b, buf, effect.length, pred,
 679:                                                     memType, op, effectCTAs);
 680:           funcBuilder.createClearReadTrackingCall(b, buf, effect.length, pred,
 681:                                                   memType, op, effectCTAs);
 682:         }
 683:         if (opInfo->trackingKind ==
 684:             MemEffectsOpInfo::TrackingKind::CommitCount) {
 685:           assert(memType == MemType::SHARED_MEM);
 686:           funcBuilder.createStageAccessForCommitCall(b, buf, effect.length,
 687:                                                      baseThread, pred, memType,
 688:                                                      opInfo->commitKind, op);
 689:         }
 690:       }
 691:     }
 692:     for (const auto &barrierInfo : opInfo->barriers) {
 693:       Value barrier = barrierInfo.barrier;
 694:       Value combinedPred = tti::maybeAnd(b, barrierInfo.pred, pred);
 695:       Value recipientCTAs = getBarrierRecipientCTAs(b, op);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 696-713

```cpp
 696:       funcBuilder.createVerifyBarrierInitializedCall(b, barrier, combinedPred,
 697:                                                      op, recipientCTAs);
 698:       if (barrierInfo.trackingMode ==
 699:           MemEffectsOpInfo::BarrierTrackingMode::Frontier) {
 700:         // If the op has barriers, we treat it as a commit emitted for each
 701:         // barrier.
 702:         for (MemType memType : {MemType::SHARED_MEM, MemType::TENSOR_MEM}) {
 703:           funcBuilder.createTrackVisibleWritesCall(
 704:               b, barrier, thread, combinedPred, memType, op, recipientCTAs);
 705:           funcBuilder.createTrackVisibleReadsCall(
 706:               b, barrier, thread, combinedPred, memType, op, recipientCTAs);
 707:         }
 708:       } else if (barrierInfo.trackingMode ==
 709:                  MemEffectsOpInfo::BarrierTrackingMode::EffectWrites) {
 710:         for (const auto &effect : opInfo->operandEffects) {
 711:           if (effect.rw != MemEffectsOpInfo::Effects::Write)
 712:             continue;
 713:           auto bufType = cast<ttg::MemDescType>(effect.buf.getType());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。
### Lines 714-730

```cpp
 714:           MemType memType = MemType::TENSOR_MEM;
 715:           if (isa<ttg::SharedEncodingTrait>(bufType.getEncoding()))
 716:             memType = MemType::SHARED_MEM;
 717:           funcBuilder.createTrackBarrierWriteForBufferCall(
 718:               b, barrier, effect.buf, effect.length, combinedPred, memType, op,
 719:               recipientCTAs, effectCTAs);
 720:         }
 721:       }
 722:       if (barrierInfo.count > 0 || barrierInfo.txCount != 0) {
 723:         funcBuilder.createVerifyBarrierArriveCall(
 724:             b, barrier, barrierInfo.count, combinedPred, op, recipientCTAs,
 725:             barrierInfo.txCount);
 726:         funcBuilder.createUpdateBarrierStateCall(
 727:             b, barrier, barrierInfo.count, combinedPred, op, recipientCTAs,
 728:             barrierInfo.txCount);
 729:       }
 730:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 731-737

```cpp
 731:     if (opInfo->implicitCommit) {
 732:       assert(opInfo->trackingKind ==
 733:              MemEffectsOpInfo::TrackingKind::CommitCount);
 734:       funcBuilder.createCommitAccessesCall(b, baseThread, pred,
 735:                                            opInfo->commitKind, op);
 736:     }
 737:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 739-756

```cpp
 739:   void addWriteChecks(ImplicitLocOpBuilder &b,
 740:                       tti::FunctionBuilder &funcBuilder, Operation *op,
 741:                       Value buf, uint32_t length, Value pred, MemType memType,
 742:                       int thread, const std::string &operandName,
 743:                       Value effectCTAs,
 744:                       CommitKind::Kind opCommitKind = CommitKind::None) {
 745:     funcBuilder.createVerifyWriteVisibilityCall(
 746:         b, buf, length, thread, operandName, pred, memType, op, effectCTAs);
 747:     // commit-num-based synchronization is only supported for shared memory
 748:     if (memType == MemType::SHARED_MEM) {
 749:       for (const auto &commitKindDesc :
 750:            hooks->getOutstandingWriteCommitKinds()) {
 751:         bool excludeSelf = (opCommitKind == commitKindDesc.kind &&
 752:                             hooks->isOrderedCommitKind(opCommitKind));
 753:         funcBuilder.createCheckOutstandingCommitsCall(
 754:             b, buf, length, getBaseThread(thread, auxData.threadLayout),
 755:             commitKindDesc.operationDesc, pred, memType, commitKindDesc.kind,
 756:             op, effectCTAs, excludeSelf);
```

- **EN:** Defines `addWriteChecks`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `addWriteChecks`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 757-759

```cpp
 757:       }
 758:     }
 759:   }
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 761-778

```cpp
 761:   void addReadChecks(ImplicitLocOpBuilder &b, tti::FunctionBuilder &funcBuilder,
 762:                      Operation *op, Value buf, uint32_t length, Value pred,
 763:                      MemType memType, int thread,
 764:                      const std::string &operandName, Value effectCTAs,
 765:                      CommitKind::Kind opCommitKind = CommitKind::None) {
 766:     funcBuilder.createVerifyReadVisibilityCall(
 767:         b, buf, length, thread, operandName, pred, memType, op, effectCTAs);
 768:     // commit-num-based synchronization is only supported for shared memory
 769:     if (memType == MemType::SHARED_MEM) {
 770:       for (const auto &commitKindDesc :
 771:            hooks->getOutstandingReadCommitKinds()) {
 772:         bool excludeSelf = (opCommitKind == commitKindDesc.kind &&
 773:                             hooks->isOrderedCommitKind(opCommitKind));
 774:         funcBuilder.createCheckOutstandingCommitsCall(
 775:             b, buf, length, getBaseThread(thread, auxData.threadLayout),
 776:             commitKindDesc.operationDesc, pred, memType, commitKindDesc.kind,
 777:             op, effectCTAs, excludeSelf);
 778:       }
```

- **EN:** Defines `addReadChecks`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `addReadChecks`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 779-780

```cpp
 779:     }
 780:   }
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 782-785

```cpp
 782:   ModuleOp module;
 783:   AuxDataMap auxData;
 784:   const ConSanTargetHooks *hooks;
 785: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 787-787

```cpp
 787: } // namespace
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 789-794

```cpp
 789: LogicalResult runConcurrencySanitizer(ModuleOp module,
 790:                                       const ConSanTargetHooks *hooks) {
 791:   assert(hooks && "hooks must not be null");
 792:   ConcurrencySanitizerImpl impl(module, hooks);
 793:   return impl.run();
 794: }
```

- **EN:** Defines `runConcurrencySanitizer`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `runConcurrencySanitizer`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 796-813

```cpp
 796: class ConcurrencySanitizerPass
 797:     : public impl::TritonInstrumentConcurrencySanitizerBase<
 798:           ConcurrencySanitizerPass> {
 799: public:
 800:   void runOnOperation() override {
 801:     ModuleOp module = getOperation();
 802:     auto targetAttr = module->getAttrOfType<StringAttr>(ttg::AttrTargetName);
 803:     assert(targetAttr && "module missing ttg.target attribute");
 804:     StringRef target = targetAttr.strref();
 805:     StringRef key = target.starts_with("cuda:")  ? "nvidia"
 806:                     : target.starts_with("hip:") ? "amd"
 807:                                                  : "";
 808:     auto hooks = createConSanHooks(key);
 809:     assert(hooks && "no ConSan hooks registered for target");
 810:     if (failed(runConcurrencySanitizer(module, hooks.get())))
 811:       return signalPassFailure();
 812:   }
 813: };
```

- **EN:** Defines `ConcurrencySanitizerPass`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `ConcurrencySanitizerPass`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 815-817

```cpp
 815: } // namespace instrument
 816: } // namespace triton
 817: } // namespace mlir
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。

## Key Concepts / 关键概念
- **EN:** The main concern is pass-driven transformation around concurrency sanitizer.
  **CN:** 核心关注点是围绕 Concurrency Sanitizer 的 pass 驱动变换。
- **EN:** Memory descriptor types carry layout, rank, and address-space information.
  **CN:** 内存描述符类型携带布局、维度和地址空间信息。
- **EN:** Tensor shape and element-type reasoning is central to the implementation.
  **CN:** 张量形状与元素类型推导是实现中的核心内容。
- **EN:** Linear layout utilities translate between logical tensor coordinates and physical placement.
  **CN:** 线性布局工具负责在逻辑张量坐标与物理放置之间进行转换。
- **EN:** Encoding attributes describe how distributed GPU data is laid out across threads and warps.
  **CN:** 编码属性描述 GPU 分布式数据在线程与 warp 之间的布局方式。
- **EN:** The code reasons at module scope rather than only on isolated operations.
  **CN:** 代码在模块范围内进行分析，而不只是处理单个操作。

## Dependencies / 依赖关系
- **Internal Triton headers / Triton 内部头文件:** `triton/Dialect/Triton/IR/Utility.h`, `triton/Dialect/TritonGPU/IR/Dialect.h`, `triton/Dialect/TritonInstrument/IR/Dialect.h`, `triton/Dialect/TritonInstrument/IR/FunctionBuilder.h`, `triton/Dialect/TritonInstrument/IR/Utility.h`, `triton/Dialect/TritonInstrument/Transforms/ConSanTargetHooks.h`, ... (+3 more)
- **MLIR headers / MLIR 头文件:** `mlir/Analysis/SliceAnalysis.h`, `mlir/IR/ImplicitLocOpBuilder.h`, `mlir/Transforms/Passes.h`
- **LLVM headers / LLVM 头文件:** None
- **Standard/library headers / 标准或通用库头文件:** None
- **Generated fragments / 生成片段:** `triton/Dialect/TritonInstrument/Transforms/Passes.h.inc`
- **Primary APIs used / 主要 API:** `ModuleOp`, `RankedTensorType`, `MemDescType`, `LinearLayout`, `OpBuilder`
- **Pipeline role / 流程角色:** This file participates in Triton pass pipelines and usually expects upstream analyses or dialect invariants to have prepared the IR. / 本文件参与 Triton 的 pass 流程，通常依赖上游分析或方言不变量先把 IR 准备好。
