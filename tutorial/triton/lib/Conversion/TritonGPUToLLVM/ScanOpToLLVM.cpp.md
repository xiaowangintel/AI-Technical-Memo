# ScanOpToLLVM.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `lib/Conversion/TritonGPUToLLVM/ScanOpToLLVM.cpp`
- **Purpose / 作用:** **EN:** Lowers TritonGPU constructs related to Scan into LLVM-compatible IR and rewrite patterns. **CN:** 把与 Scan Op To LLVM 相关的 TritonGPU 构造降级为 LLVM 兼容 IR 与重写模式。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8

```cpp
   1: #include "ReduceScanCommon.h"
   2: #include "mlir/Support/LLVM.h"
   3: #include "triton/Analysis/Utility.h"
   4: #include "triton/Conversion/TritonGPUToLLVM/PatternTritonGPUOpToLLVM.h"
   5: #include "triton/Conversion/TritonGPUToLLVM/TargetInfoBase.h"
   6: #include "triton/Conversion/TritonGPUToLLVM/Utility.h"
   7: #include "triton/Tools/LayoutUtils.h"
   8: #include "llvm/ADT/STLExtras.h"
```

- **EN:** Includes the interfaces this file depends on. Triton headers (`Utility.h`, `PatternTritonGPUOpToLLVM.h`, `TargetInfoBase.h`, `Utility.h`, ... (+1 more)) provide domain-specific IR/support, MLIR headers (`LLVM.h`) provide rewriting and analysis infrastructure, LLVM headers (`STLExtras.h`) supply low-level utilities, and standard/library headers (`ReduceScanCommon.h`) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（`Utility.h`, `PatternTritonGPUOpToLLVM.h`, `TargetInfoBase.h`, `Utility.h`, ... (+1 more)）提供领域特定 IR/支持逻辑，MLIR 头文件（`LLVM.h`）提供重写与分析基础设施，LLVM 头文件（`STLExtras.h`）提供底层工具，而标准/通用库头文件（`ReduceScanCommon.h`）提供通用能力。
### Lines 10-11

```cpp
  10: using namespace mlir;
  11: using namespace mlir::triton;
```

- **EN:** Introduces namespace aliases/imports (`mlir`, `mlir::triton`) so the rest of the file can use MLIR/Triton symbols without repetitive qualification.
- **CN:** 这里通过命名空间导入（`mlir`, `mlir::triton`）减少后续代码中的重复限定，使 MLIR/Triton 符号使用更简洁。
### Lines 13-16

```cpp
  13: using ::mlir::LLVM::delinearize;
  14: using ::mlir::LLVM::linearize;
  15: using ::mlir::triton::gpu::getTotalElemsPerThread;
  16: using ::mlir::triton::gpu::toLinearEncoding;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 18-26

```cpp
  18: // apply combine region to acc and cur and accumulate it into acc
  19: static SmallVector<Value> accumulate(ScanLoweringHelper &helper,
  20:                                      ConversionPatternRewriter &rewriter,
  21:                                      ValueRange acc, ValueRange cur,
  22:                                      Value pred = {}) {
  23:   auto loc = helper.getLoc();
  24:   auto &combineOp = helper.getCombineOp();
  25:   return applyCombineOp(loc, rewriter, combineOp, acc, cur, pred);
  26: }
```

- **EN:** Defines `accumulate`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `accumulate`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 28-43

```cpp
  28: // Scan a contiguous elements within a thread and update `srcValues` in place.
  29: static void
  30: scanThreadContiguousElements(SmallVector<SmallVector<Value>> &srcValues,
  31:                              ConversionPatternRewriter &rewriter,
  32:                              ScanLoweringHelper &helper) {
  33:   // Depending on layout contiguous elements along axis dim may not be
  34:   // contiguous in srcValues. Keep track of what elements belong to the same
  35:   // chunk of contiguous elements.
  36:   unsigned scanElementsPerThreads = helper.getAxisNumElementsPerThread();
  37:   unsigned numChunks = srcValues.size() / scanElementsPerThreads;
  38:   unsigned stride = helper.getAxisElementStride();
  39:   SmallVector<SmallVector<Value>> accs(numChunks);
  40:   for (unsigned srcIndex = 0; srcIndex < srcValues.size(); srcIndex++) {
  41:     // Change this into emitOffsetForLayout?
  42:     unsigned accIndex = (srcIndex % stride) +
  43:                         ((srcIndex / stride) / scanElementsPerThreads) * stride;
```

- **EN:** Defines `scanThreadContiguousElements`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `scanThreadContiguousElements`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 45-49

```cpp
  45:     accs[accIndex] =
  46:         accumulate(helper, rewriter, accs[accIndex], srcValues[srcIndex]);
  47:     srcValues[srcIndex] = accs[accIndex];
  48:   }
  49: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 51-68

```cpp
  51: // Apply a scan across threads of the warp for the last element of each
  52: // contiguous group of elements.
  53: static void warpScan(SmallVector<SmallVector<Value>> &srcValues,
  54:                      ConversionPatternRewriter &rewriter,
  55:                      const TargetInfoBase &targetInfo,
  56:                      ScanLoweringHelper &helper, Value laneIdAxis) {
  57:   Location loc = helper.getLoc();
  58:   auto b = TritonLLVMOpBuilder(loc, rewriter);
  59:   unsigned scanElementsPerThreads = helper.getAxisNumElementsPerThread();
  60:   unsigned elementStride = helper.getAxisElementStride();
  61:   unsigned threadStride = helper.getAxisThreadStride();
  62:   unsigned scanDim = helper.getAxisNumThreadsPerWarpWithUniqueData();
  63:   for (unsigned srcIndex = 0; srcIndex < srcValues.size(); srcIndex++) {
  64:     unsigned elementIdx = (srcIndex / elementStride) % scanElementsPerThreads;
  65:     // Only consider the last element of each contiguous chunk of elements.
  66:     if (elementIdx != scanElementsPerThreads - 1)
  67:       continue;
  68:     // Reduce within warps.
```

- **EN:** Defines `warpScan`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `warpScan`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 69-84

```cpp
  69:     SmallVector<Value> acc = srcValues[srcIndex];
  70:     for (unsigned i = 1; i <= scanDim / 2; i <<= 1) {
  71:       SmallVector<Value> shfl(acc.size());
  72:       for (unsigned j = 0; j < acc.size(); ++j) {
  73:         shfl[j] = targetInfo.shuffleUp(rewriter, loc, acc[j], i * threadStride);
  74:       }
  75:       Value mask = b.icmp_sge(laneIdAxis, b.i32_val(i));
  76:       SmallVector<Value> tempAcc =
  77:           accumulate(helper, rewriter, shfl, acc, mask);
  78:       for (unsigned j = 0; j < acc.size(); ++j) {
  79:         acc[j] = b.select(mask, tempAcc[j], acc[j]);
  80:       }
  81:     }
  82:     srcValues[srcIndex] = std::move(acc);
  83:   }
  84: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 86-103

```cpp
  86: // For each set of contiguous elements within a thread we store the partial
  87: // reduction into shared memory. Each parallel scan and each warp will store its
  88: // own partial reductions. The shared memory is organized as follow:
  89: //          -----------------------------------------------------------------
  90: // chunk 0: | acc[0] warp 0 | acc[1] warp 0 | acc[0] warp 1 | acc[1] warp 1 |
  91: // chunk 1: | acc[0] warp 0 | acc[1] warp 0 | acc[0] warp 1 | acc[1] warp 1 |
  92: static void storeWarpAccumulator(SmallVector<SmallVector<Value>> &srcValues,
  93:                                  ConversionPatternRewriter &rewriter,
  94:                                  ScanLoweringHelper &helper, Value laneId,
  95:                                  Value warpId, SmallVector<Value> smemBases,
  96:                                  SmallVector<Type> smemTypes,
  97:                                  Value parallelLaneId, Value isRepresentative,
  98:                                  const TargetInfoBase &targetInfo) {
  99:   Location loc = helper.getLoc();
 100:   auto b = TritonLLVMOpBuilder(loc, rewriter);
 101:   unsigned scanElementsPerThreads = helper.getAxisNumElementsPerThread();
 102:   unsigned scanDim = helper.getAxisNumThreadsPerWarpWithUniqueData();
 103:   unsigned numParallelLane = helper.getNonAxisNumThreadsPerCTA();
```

- **EN:** Defines `storeWarpAccumulator`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `storeWarpAccumulator`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 104-106

```cpp
 104:   unsigned axisNumWarps = helper.getAxisNumWarpsWithUniqueData();
 105:   unsigned chunkId = 0;
 106:   unsigned elementStride = helper.getAxisElementStride();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 108-125

```cpp
 108:   for (unsigned srcIndex = 0; srcIndex < srcValues.size(); srcIndex++) {
 109:     unsigned elementIdx = (srcIndex / elementStride) % scanElementsPerThreads;
 110:     // Only consider the last element of each contiguous chunk of elements.
 111:     if (elementIdx != scanElementsPerThreads - 1)
 112:       continue;
 113:     auto lastElement = srcValues[srcIndex];
 114:     Value mask = b.icmp_eq(laneId, b.i32_val(scanDim - 1));
 115:     mask = b.and_(mask, isRepresentative);
 116:     Value index =
 117:         b.add(parallelLaneId, b.mul(warpId, b.i32_val(numParallelLane)));
 118:     index = b.add(index, b.i32_val(chunkId * numParallelLane * axisNumWarps));
 119:     for (unsigned i = 0; i < lastElement.size(); ++i) {
 120:       Value writePtr =
 121:           b.gep(smemBases[i].getType(), smemTypes[i], smemBases[i], index);
 122:       targetInfo.storeShared(rewriter, loc, writePtr, lastElement[i], mask);
 123:     }
 124:     chunkId++;
 125:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 126-126

```cpp
 126: }
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 128-145

```cpp
 128: // Read the partial reductions from shared memory from each chunk of contiguous
 129: // elements for each warp and parallel scan. Then combine the partial reduction
 130: // with the right elements. Within a given contiguous element chunk we update
 131: // all the elements by accumulating the value from the last element of the
 132: // reduced value from the previous lane.
 133: static void AddPartialReduce(SmallVector<SmallVector<Value>> &srcValues,
 134:                              ConversionPatternRewriter &rewriter,
 135:                              const TargetInfoBase &targetInfo,
 136:                              ScanLoweringHelper &helper,
 137:                              ArrayRef<Value> smemBases,
 138:                              ArrayRef<Type> smemTypes, Value warpId,
 139:                              Value laneIdAxis, Value parallelLaneId) {
 140:   Location loc = helper.getLoc();
 141:   auto b = TritonLLVMOpBuilder(loc, rewriter);
 142:   unsigned numParallelLane = helper.getNonAxisNumThreadsPerCTA();
 143:   unsigned scanElementsPerThreads = helper.getAxisNumElementsPerThread();
 144:   unsigned parallelElementsPerThread = helper.getNonAxisNumElementsPerThread();
 145:   unsigned elementStride = helper.getAxisElementStride();
```

- **EN:** Defines `AddPartialReduce`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `AddPartialReduce`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 146-163

```cpp
 146:   unsigned threadStride = helper.getAxisThreadStride();
 147:   unsigned axisNumWarps = helper.getAxisNumWarpsWithUniqueData();
 148:   Value maskNotFirstWarp = b.icmp_ne(warpId, b.i32_val(0));
 149:   Value maskNotFirstLane = b.icmp_ne(laneIdAxis, b.i32_val(0));
 150:   Value maskNotFirstThread = b.or_(maskNotFirstWarp, maskNotFirstLane);
 151:   struct Accumulator {
 152:     SmallVector<Value> acc;
 153:     SmallVector<Value> maskedAcc;
 154:   };
 155:   unsigned numScanBlocks = helper.getAxisNumBlocks();
 156:   unsigned numParallelBlocks = helper.getNonAxisNumBlocks();
 157:   assert(numScanBlocks * numParallelBlocks * parallelElementsPerThread *
 158:              scanElementsPerThreads ==
 159:          srcValues.size());
 160:   SmallVector<Accumulator> accumulators(numParallelBlocks *
 161:                                         parallelElementsPerThread);
 162:   unsigned chunkId = 0;
 163:   unsigned blockStride = helper.getAxisBlockStride();
```

- **EN:** Defines `Accumulator`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `Accumulator`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 164-179

```cpp
 164:   for (unsigned srcIndex = 0; srcIndex < srcValues.size(); srcIndex++) {
 165:     unsigned elementIdx = (srcIndex / elementStride) % scanElementsPerThreads;
 166:     // Only consider the last element of each contiguous chunk of elements.
 167:     if (elementIdx != scanElementsPerThreads - 1)
 168:       continue;
 169:     // Accumulate the partial reduction from shared memory. Decide which
 170:     // accumulator to combine based on whether the elements belong to the same
 171:     // dimension along axis.
 172:     unsigned blockId = chunkId / parallelElementsPerThread;
 173:     unsigned parallelBlockId =
 174:         blockId % blockStride +
 175:         ((blockId / blockStride) / numScanBlocks) * blockStride;
 176:     unsigned accumulatorIndex = chunkId % parallelElementsPerThread +
 177:                                 parallelBlockId * parallelElementsPerThread;
 178:     Accumulator &accumulator = accumulators[accumulatorIndex];
 179:     unsigned axisBlockId = (blockId / blockStride) % numScanBlocks;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 180-190

```cpp
 180:     for (unsigned i = 0; i < axisNumWarps; ++i) {
 181:       Value index =
 182:           b.add(parallelLaneId,
 183:                 b.i32_val(numParallelLane * (i + chunkId * axisNumWarps)));
 184:       SmallVector<Value> partialReduce(helper.getNumOperands());
 185:       for (unsigned j = 0; j < helper.getNumOperands(); ++j) {
 186:         auto elemTy = smemTypes[j];
 187:         Value ptr = b.gep(smemBases[j].getType(), elemTy, smemBases[j], index);
 188:         partialReduce[j] =
 189:             targetInfo.loadShared(rewriter, loc, ptr, elemTy, b.true_val());
 190:       }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 192-204

```cpp
 192:       if (accumulator.acc.size() == 0) {
 193:         accumulator.acc = partialReduce;
 194:         accumulator.maskedAcc = partialReduce;
 195:         continue;
 196:       }
 197:       Value mask = b.icmp_sge(warpId, b.i32_val(i + 1));
 198:       accumulator.acc =
 199:           accumulate(helper, rewriter, accumulator.acc, partialReduce);
 200:       for (unsigned j = 0; j < helper.getNumOperands(); ++j) {
 201:         accumulator.maskedAcc[j] =
 202:             b.select(mask, accumulator.acc[j], accumulator.maskedAcc[j]);
 203:       }
 204:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 206-223

```cpp
 206:     Value pred = axisBlockId == 0 ? maskNotFirstWarp : Value{};
 207:     auto temp = accumulate(helper, rewriter, accumulator.maskedAcc,
 208:                            srcValues[srcIndex], pred);
 209:     if (axisBlockId == 0) {
 210:       // For the first warp and first chunk we don't have anything to
 211:       // accumulate.
 212:       auto val = srcValues[srcIndex];
 213:       for (unsigned i = 0; i < helper.getNumOperands(); ++i) {
 214:         temp[i] = b.select(maskNotFirstWarp, temp[i], val[i]);
 215:       }
 216:     }
 217:     srcValues[srcIndex] = temp;
 218:     // Update the rest of the contiguous elements.
 219:     SmallVector<Value> lastElement(helper.getNumOperands());
 220:     for (unsigned i = 0; i < helper.getNumOperands(); ++i) {
 221:       auto elem = targetInfo.shuffleUp(rewriter, loc, temp[i], threadStride);
 222:       lastElement[i] =
 223:           b.select(maskNotFirstLane, elem, accumulator.maskedAcc[i]);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 224-241

```cpp
 224:     }
 225:     for (unsigned i = 1; i < scanElementsPerThreads; ++i) {
 226:       pred = axisBlockId == 0 ? maskNotFirstThread : Value{};
 227:       auto laneValue = srcValues[srcIndex - i * elementStride];
 228:       laneValue = accumulate(helper, rewriter, lastElement, laneValue, pred);
 229:       if (axisBlockId == 0) {
 230:         // For the first warp and first chunk we don't have anything to
 231:         // accumulate.
 232:         for (unsigned j = 0; j < helper.getNumOperands(); ++j) {
 233:           laneValue[j] = b.select(maskNotFirstThread, laneValue[j],
 234:                                   srcValues[srcIndex - i * elementStride][j]);
 235:         }
 236:       }
 237:       srcValues[srcIndex - i * elementStride] = std::move(laneValue);
 238:     }
 239:     // For the next chunk start back from the value containing the
 240:     // accumulated value of all the warps.
 241:     accumulator.maskedAcc = accumulator.acc;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 242-244

```cpp
 242:     chunkId++;
 243:   }
 244: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 246-262

```cpp
 246: static void AddPartialReduceOneWarp(SmallVector<SmallVector<Value>> &srcValues,
 247:                                     ConversionPatternRewriter &rewriter,
 248:                                     const TargetInfoBase &targetInfo,
 249:                                     ScanLoweringHelper &helper, Value warpId,
 250:                                     Value laneIdAxis, Value laneIdLast) {
 251:   Location loc = helper.getLoc();
 252:   auto b = TritonLLVMOpBuilder(loc, rewriter);
 253:   unsigned scanElementsPerThreads = helper.getAxisNumElementsPerThread();
 254:   unsigned parallelElementsPerThread = helper.getNonAxisNumElementsPerThread();
 255:   unsigned elementStride = helper.getAxisElementStride();
 256:   unsigned threadStride = helper.getAxisThreadStride();
 257:   unsigned scanDim = helper.getAxisNumThreadsPerWarpWithUniqueData();
 258:   Value maskFirstWarp = b.icmp_eq(warpId, b.i32_val(0));
 259:   Value maskFirstLane = b.icmp_eq(laneIdAxis, b.i32_val(0));
 260:   Value maskFirstThread = b.and_(maskFirstWarp, maskFirstLane);
 261:   unsigned numScanBlocks = helper.getAxisNumBlocks();
 262:   unsigned numParallelBlocks = helper.getNonAxisNumBlocks();
```

- **EN:** Defines `AddPartialReduceOneWarp`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `AddPartialReduceOneWarp`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 263-280

```cpp
 263:   assert(numScanBlocks * numParallelBlocks * parallelElementsPerThread *
 264:              scanElementsPerThreads ==
 265:          srcValues.size());
 266:   SmallVector<SmallVector<Value>> accumulators(numParallelBlocks *
 267:                                                parallelElementsPerThread);
 268:   unsigned chunkId = 0;
 269:   unsigned blockStride = helper.getAxisBlockStride();
 270:   for (unsigned srcIndex = 0; srcIndex < srcValues.size(); srcIndex++) {
 271:     unsigned elementIdx = (srcIndex / elementStride) % scanElementsPerThreads;
 272:     // Only consider the last element of each contiguous chunk of elements.
 273:     if (elementIdx != scanElementsPerThreads - 1)
 274:       continue;
 275:     unsigned blockId = chunkId / parallelElementsPerThread;
 276:     unsigned parallelBlockId =
 277:         blockId % blockStride +
 278:         ((blockId / blockStride) / numScanBlocks) * blockStride;
 279:     unsigned accumulatorIndex = chunkId % parallelElementsPerThread +
 280:                                 parallelBlockId * parallelElementsPerThread;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 281-297

```cpp
 281:     auto &accumulator = accumulators[accumulatorIndex];
 282:     unsigned axisBlockId = (blockId / blockStride) % numScanBlocks;
 283:     if (axisBlockId == 0) // First chunk and first block
 284:       accumulator = srcValues[srcIndex];
 285:     else
 286:       srcValues[srcIndex] =
 287:           accumulate(helper, rewriter, accumulator, srcValues[srcIndex]);
 288:     // Update the rest of the contiguous elements.
 289:     auto lastElement = srcValues[srcIndex];
 290:     if (scanDim > 1) {
 291:       for (unsigned i = 0; i < helper.getNumOperands(); ++i) {
 292:         lastElement[i] = targetInfo.shuffleUp(
 293:             rewriter, loc, srcValues[srcIndex][i], threadStride);
 294:         lastElement[i] =
 295:             b.select(maskFirstLane, accumulator[i], lastElement[i]);
 296:         if (numScanBlocks > 1)
 297:           // Update accumulator with the value from the last lane.
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 298-315

```cpp
 298:           accumulator[i] = targetInfo.shuffleIdx(
 299:               rewriter, loc, srcValues[srcIndex][i], laneIdLast);
 300:       }
 301:     } else if (numScanBlocks > 1) {
 302:       accumulator = srcValues[srcIndex];
 303:     }
 304:     for (unsigned i = 1; i < scanElementsPerThreads; ++i) {
 305:       auto laneValue = srcValues[srcIndex - i * elementStride];
 306:       laneValue = accumulate(helper, rewriter, lastElement, laneValue);
 307:       if (axisBlockId == 0) {
 308:         for (unsigned j = 0; j < helper.getNumOperands(); ++j) {
 309:           // For the first warp and first chunk we don't have anything to
 310:           // accumulate.
 311:           laneValue[j] = b.select(maskFirstThread,
 312:                                   srcValues[srcIndex - i * elementStride][j],
 313:                                   laneValue[j]);
 314:         }
 315:       }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 316-322

```cpp
 316:       srcValues[srcIndex - i * elementStride] = std::move(laneValue);
 317:     }
 318:     // For the next chunk start back from the value containing the
 319:     // accumulated value of all the warps.
 320:     chunkId++;
 321:   }
 322: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 324-335

```cpp
 324: namespace {
 325: struct ScanOpConversion
 326:     : public ConvertTritonGPUReduceScanToLLVMPattern<triton::ScanOp> {
 327: public:
 328:   using ConvertTritonGPUReduceScanToLLVMPattern<
 329:       triton::ScanOp>::ConvertTritonGPUReduceScanToLLVMPattern;
 330:   explicit ScanOpConversion(LLVMTypeConverter &typeConverter,
 331:                             const TargetInfoBase &targetInfo,
 332:                             PatternBenefit benefit = 1)
 333:       : ConvertTritonGPUReduceScanToLLVMPattern<triton::ScanOp>(typeConverter,
 334:                                                                 benefit),
 335:         targetInfo(targetInfo) {}
```

- **EN:** Defines `ScanOpConversion`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `ScanOpConversion`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 337-343

```cpp
 337:   LogicalResult
 338:   matchAndRewrite(triton::ScanOp op, OpAdaptor adaptor,
 339:                   ConversionPatternRewriter &rewriter) const override {
 340:     if (succeeded(emitFastScan(op, adaptor, rewriter, targetInfo)))
 341:       return success();
 342:     return failure();
 343:   }
```

- **EN:** Defines `matchAndRewrite`, the core MLIR rewrite callback. It checks whether the source operation matches the pattern and then materializes the lowered replacement. The code uses `LogicalResult` to stay conservative when preconditions are not met.
- **CN:** 这里定义 `matchAndRewrite`，即 MLIR 模式重写的核心回调。它先检查源操作是否匹配模式，再构造降级后的替代结果。 代码通过 `LogicalResult` 在前提不满足时保持保守处理。
### Lines 345-360

```cpp
 345: private:
 346:   const TargetInfoBase &targetInfo;
 347:   std::tuple<SmallVector<Value>, Value>
 348:   getMultiDimLaneId(ConversionPatternRewriter &rewriter,
 349:                     ScanLoweringHelper &helper, Value laneId) const;
 350:   std::tuple<SmallVector<Value>, Value>
 351:   getMultiDimWarpId(ConversionPatternRewriter &rewriter,
 352:                     ScanLoweringHelper &helper, Value warpId) const;
 353:   std::tuple<Value, Value, Value, Value>
 354:   getDelinearizedIds(ConversionPatternRewriter &rewriter,
 355:                      ScanLoweringHelper &helper, Value laneId,
 356:                      Value warpId) const;
 357:   LogicalResult emitFastScan(triton::ScanOp op, triton::ScanOpAdaptor adaptor,
 358:                              ConversionPatternRewriter &rewriter,
 359:                              const TargetInfoBase &targetInfo) const;
 360: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 362-371

```cpp
 362: std::tuple<SmallVector<Value>, Value>
 363: ScanOpConversion::getMultiDimLaneId(ConversionPatternRewriter &rewriter,
 364:                                     ScanLoweringHelper &helper,
 365:                                     Value laneId) const {
 366:   auto loc = helper.getLoc();
 367:   auto srcEncoding = helper.getEncoding();
 368:   auto kWarp = rewriter.getStringAttr("lane");
 369:   return delinearize(rewriter, loc, srcEncoding, helper.getShape(), kWarp,
 370:                      laneId);
 371: }
```

- **EN:** Defines accessor/helper `ScanOpConversion::getMultiDimLaneId` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `ScanOpConversion::getMultiDimLaneId`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 373-382

```cpp
 373: std::tuple<SmallVector<Value>, Value>
 374: ScanOpConversion::getMultiDimWarpId(ConversionPatternRewriter &rewriter,
 375:                                     ScanLoweringHelper &helper,
 376:                                     Value warpId) const {
 377:   auto loc = helper.getLoc();
 378:   auto srcEncoding = helper.getEncoding();
 379:   auto kWarp = rewriter.getStringAttr("warp");
 380:   return delinearize(rewriter, loc, srcEncoding, helper.getShape(), kWarp,
 381:                      warpId);
 382: }
```

- **EN:** Defines accessor/helper `ScanOpConversion::getMultiDimWarpId` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `ScanOpConversion::getMultiDimWarpId`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 384-393

```cpp
 384: // Break up the threadId into lane and warp id along the scan dimension and
 385: // compute a flat id for the parallel dimensions.
 386: std::tuple<Value, Value, Value, Value>
 387: ScanOpConversion::getDelinearizedIds(ConversionPatternRewriter &rewriter,
 388:                                      ScanLoweringHelper &helper, Value laneId,
 389:                                      Value warpId) const {
 390:   auto loc = helper.getLoc();
 391:   auto b = TritonLLVMOpBuilder(loc, rewriter);
 392:   unsigned axis = helper.getAxis();
 393:   auto srcEncoding = helper.getEncoding();
```

- **EN:** Defines accessor/helper `ScanOpConversion::getDelinearizedIds` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `ScanOpConversion::getDelinearizedIds`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 395-400

```cpp
 395:   auto threadsPerWarp = srcEncoding.getThreadsPerWarp();
 396:   auto warpsPerCTA = srcEncoding.getWarpsPerCTA();
 397:   auto [multiDimLaneId, isRepresentativeLane] =
 398:       getMultiDimLaneId(rewriter, helper, laneId);
 399:   auto [multiDimWarpId, isRepresentativeWarp] =
 400:       getMultiDimWarpId(rewriter, helper, warpId);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 402-403

```cpp
 402:   Value laneIdAxis = multiDimLaneId[axis];
 403:   Value warpIdAxis = multiDimWarpId[axis];
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 405-419

```cpp
 405:   multiDimLaneId[axis] = b.i32_val(0);
 406:   threadsPerWarp[axis] = 1;
 407:   Value laneIdParallel = linearize(rewriter, loc, multiDimLaneId,
 408:                                    threadsPerWarp, helper.getOrder());
 409:   multiDimWarpId[axis] = b.i32_val(0);
 410:   warpsPerCTA[axis] = 1;
 411:   Value warpIdParallel =
 412:       linearize(rewriter, loc, multiDimWarpId, warpsPerCTA, helper.getOrder());
 413:   Value flatIdParallel = b.add(
 414:       laneIdParallel,
 415:       b.mul(warpIdParallel, b.i32_val(helper.getNonAxisNumThreadsPerWarp())));
 416:   auto isRepresentative = b.and_(isRepresentativeLane, isRepresentativeWarp);
 417:   return std::make_tuple(laneIdAxis, warpIdAxis, flatIdParallel,
 418:                          isRepresentative);
 419: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 421-431

```cpp
 421: SmallVector<SmallVector<Value>>
 422: unpackInputs(Location loc, triton::ScanOp op, triton::ScanOpAdaptor adaptor,
 423:              ConversionPatternRewriter &rewriter, unsigned nElems,
 424:              const ColumnAction &removeBroadcastRegs) {
 425:   auto operands = adaptor.getOperands();
 426:   SmallVector<SmallVector<Value>> srcValues(nElems);
 427:   for (unsigned i = 0; i < op.getNumOperands(); ++i) {
 428:     auto values = unpackLLElements(loc, operands[i], rewriter);
 429:     if (!removeBroadcastRegs.isIdentity()) {
 430:       values = removeBroadcastRegs.apply(values);
 431:     }
```

- **EN:** Defines `unpackInputs`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `unpackInputs`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 433-439

```cpp
 433:     assert(values.size() == srcValues.size());
 434:     for (unsigned j = 0; j < srcValues.size(); ++j) {
 435:       srcValues[j].push_back(values[j]);
 436:     }
 437:   }
 438:   return srcValues;
 439: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 441-458

```cpp
 441: // Flip the srcValues. Both reverses the chunks and reverses the lanes.
 442: // Lane reversal is done with a butterfly shuffle flip (divide and flip).
 443: SmallVector<SmallVector<Value>>
 444: flipSrcValues(Location loc, triton::ScanOp op,
 445:               ConversionPatternRewriter &rewriter,
 446:               const TargetInfoBase &targetInfo,
 447:               SmallVector<SmallVector<Value>> srcValues, int iWarpSize) {
 448:   SmallVector<SmallVector<Value>> values(srcValues.size());
 449:   for (int i = 0; i < srcValues.size(); ++i) {
 450:     int revIndex = srcValues.size() - i - 1;
 451:     for (unsigned j = 0; j < op.getNumOperands(); ++j) {
 452:       for (unsigned k = iWarpSize / 2; k >= 1; k = k / 2) {
 453:         srcValues[revIndex][j] =
 454:             targetInfo.shuffleXor(rewriter, loc, srcValues[revIndex][j], k);
 455:       }
 456:       values[i].push_back(srcValues[revIndex][j]);
 457:     }
 458:   }
```

- **EN:** Defines `flipSrcValues`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `flipSrcValues`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 459-460

```cpp
 459:   return values;
 460: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 462-474

```cpp
 462: // Lowering using warp shuffle operations to do warp level scan.
 463: LogicalResult
 464: ScanOpConversion::emitFastScan(triton::ScanOp op, triton::ScanOpAdaptor adaptor,
 465:                                ConversionPatternRewriter &rewriter,
 466:                                const TargetInfoBase &targetInfo) const {
 467:   ScanLoweringHelper helper(op);
 468:   auto origLayout = triton::gpu::toLinearLayout(
 469:       cast<RankedTensorType>(op.getOperands()[0].getType()));
 470:   auto removeBroadcastRegs = actionRemoveBroadcastedRegs(origLayout);
 471:   auto loc = helper.getLoc();
 472:   auto b = TritonLLVMOpBuilder(loc, rewriter);
 473:   if (!helper.isSupported())
 474:     return op.emitError("TODO: unsupported scan layout");
```

- **EN:** Defines `ScanOpConversion::emitFastScan`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `ScanOpConversion::emitFastScan`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 476-481

```cpp
 476:   Value threadId = getThreadId(rewriter, loc);
 477:   auto mod = op->getParentOfType<ModuleOp>();
 478:   unsigned iWarpSize = triton::gpu::TritonGPUDialect::getThreadsPerWarp(mod);
 479:   Value warpSize = b.i32_val(iWarpSize);
 480:   Value warpId = b.udiv(threadId, warpSize);
 481:   Value laneId = b.urem(threadId, warpSize);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 483-489

```cpp
 483:   auto [laneIdAxis, warpIdAxis, flatIdParallel, isRepresentative] =
 484:       getDelinearizedIds(rewriter, helper, laneId, warpId);
 485:   auto axisNumWarps = helper.getAxisNumWarpsWithUniqueData();
 486:   unsigned nElems =
 487:       helper.getEncoding().getTotalElemsPerThread(helper.getShape());
 488:   auto srcValues =
 489:       unpackInputs(loc, op, adaptor, rewriter, nElems, removeBroadcastRegs);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 491-502

```cpp
 491:   // For the reverse option we apply flip(scan(flip()) in
 492:   // order to avoid having a separate code path in the reverse direction.
 493:   // We do this by 1) reversing chunks, 2) reversing lanes, 3) reversing
 494:   // warp ids and then undoing this below.
 495:   // (Note: Tried pretty hard to get shflDownSync to work but I ended up
 496:   // having to add a lot of the complex cross warp code (if rev switch
 497:   // first/last etc). Reverse first seems more maintainable.)
 498:   if (op.getReverse()) {
 499:     warpIdAxis = b.sub(b.i32_val(axisNumWarps - 1), warpIdAxis);
 500:     srcValues =
 501:         flipSrcValues(loc, op, rewriter, targetInfo, srcValues, iWarpSize);
 502:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 504-508

```cpp
 504:   // Scan contiguous elements in a thread and update `srcValues`.
 505:   scanThreadContiguousElements(srcValues, rewriter, helper);
 506:   // Apply warp level scan to the last element of each chunk of contiguous
 507:   // elements.
 508:   warpScan(srcValues, rewriter, targetInfo, helper, laneIdAxis);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 510-519

```cpp
 510:   if (axisNumWarps > 1) {
 511:     // Slow path for the case where there are multiple warps with unique data on
 512:     // the axis.
 513:     auto elems = helper.getScratchSizeInElems();
 514:     SmallVector<Value> smemBases =
 515:         getSmemBases(op, elems, rewriter, targetInfo);
 516:     SmallVector<Type> smemTypes(op.getNumOperands());
 517:     for (unsigned i = 0; i < op.getNumOperands(); ++i) {
 518:       smemTypes[i] = getElementType(op, i);
 519:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 521-538

```cpp
 521:     // Store the partial reducing for each warp into shared memory.
 522:     storeWarpAccumulator(srcValues, rewriter, helper, laneIdAxis, warpIdAxis,
 523:                          smemBases, smemTypes, flatIdParallel, isRepresentative,
 524:                          targetInfo);
 525:     b.barrier(triton::gpu::AddrSpace::Local);
 526:     // Read back the partial reduction of each warp and accumulate them based on
 527:     // warpId. Then update each chunk of contiguous elements by adding the
 528:     // accumulated value from the previous lane.
 529:     AddPartialReduce(srcValues, rewriter, targetInfo, helper, smemBases,
 530:                      smemTypes, warpIdAxis, laneIdAxis, flatIdParallel);
 531:   } else if (srcValues.size() > 1) {
 532:     // Fast path for the case where there is only one warp with unique data on
 533:     // the axis.
 534:     unsigned scanDim = helper.getAxisNumThreadsPerWarpWithUniqueData();
 535:     auto multiDimLaneId =
 536:         std::get<0>(getMultiDimLaneId(rewriter, helper, laneId));
 537:     multiDimLaneId[helper.getAxis()] = b.i32_val(scanDim - 1);
 538:     auto linearEncoding = helper.getEncoding();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 539-544

```cpp
 539:     auto kLane = StringAttr::get(rewriter.getContext(), "lane");
 540:     Value laneIdLast =
 541:         linearize(rewriter, loc, multiDimLaneId, linearEncoding, kLane);
 542:     AddPartialReduceOneWarp(srcValues, rewriter, targetInfo, helper, warpIdAxis,
 543:                             laneIdAxis, laneIdLast);
 544:   } // else axisNumWarps == 1 and srcValues.size() == 1, nothing to do.
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 546-556

```cpp
 546:   auto transpose = [](const SmallVector<SmallVector<Value>> &v) {
 547:     assert(v.size() > 0 && v[0].size() > 0);
 548:     auto ret = SmallVector<SmallVector<Value>>(v[0].size(),
 549:                                                SmallVector<Value>(v.size()));
 550:     for (int i = 0; i < v.size(); ++i) {
 551:       for (int j = 0; j < v[0].size(); ++j) {
 552:         ret[j][i] = v[i][j];
 553:       }
 554:     }
 555:     return ret;
 556:   };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 558-562

```cpp
 558:   SmallVector<Value> results(op.getNumOperands());
 559:   if (op.getReverse()) {
 560:     srcValues =
 561:         flipSrcValues(loc, op, rewriter, targetInfo, srcValues, iWarpSize);
 562:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 564-578

```cpp
 564:   auto valuesTransposed = transpose(srcValues);
 565:   if (!removeBroadcastRegs.isIdentity()) {
 566:     for (auto &values : valuesTransposed) {
 567:       values = broadcastAs(values, origLayout);
 568:     }
 569:   }
 570:   for (unsigned i = 0; i < op.getNumOperands(); ++i) {
 571:     auto resultTy = dyn_cast<RankedTensorType>(op.getResult()[i].getType());
 572:     results[i] = packLLElements(loc, getTypeConverter(), valuesTransposed[i],
 573:                                 rewriter, resultTy);
 574:   }
 575:   rewriter.replaceOp(op, results);
 576:   return success();
 577: }
 578: } // namespace
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. It finishes by replacing the original operation with the lowered form. Type mapping is delegated to the LLVM type converter or related conversion helpers.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 它会在转换成功后用降级后的形式替换原始操作。 类型映射委托给 LLVM 类型转换器或相关转换辅助函数。
### Lines 580-584

```cpp
 580: void mlir::triton::populateScanOpToLLVMPatterns(
 581:     LLVMTypeConverter &typeConverter, RewritePatternSet &patterns,
 582:     const TargetInfoBase &targetInfo, PatternBenefit benefit) {
 583:   patterns.add<ScanOpConversion>(typeConverter, targetInfo, benefit);
 584: }
```

- **EN:** Defines `mlir::triton::populateScanOpToLLVMPatterns`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `mlir::triton::populateScanOpToLLVMPatterns`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。

## Key Concepts / 关键概念
- **EN:** The file focuses on lowering scan op to llvm related IR into a more target-oriented representation.
  **CN:** 本文件重点是把与 Scan Op To LLVM 相关的 IR 降级为更面向目标的表示。
- **EN:** Type conversion bridges Triton/MLIR types to LLVM-level data layouts.
  **CN:** 类型转换负责把 Triton/MLIR 类型映射到 LLVM 层的数据布局。
- **EN:** Pattern rewriting is the main mechanism for canonicalization and lowering in this file.
  **CN:** 模式重写是本文件进行规范化与降级转换的核心机制。
- **EN:** Conversion rewrites replace source dialect ops with target dialect values while preserving semantics.
  **CN:** 转换式重写通过替换源方言操作来保持语义并生成目标方言值。
- **EN:** Tensor shape and element-type reasoning is central to the implementation.
  **CN:** 张量形状与元素类型推导是实现中的核心内容。
- **EN:** Linear layout utilities translate between logical tensor coordinates and physical placement.
  **CN:** 线性布局工具负责在逻辑张量坐标与物理放置之间进行转换。

## Dependencies / 依赖关系
- **Internal Triton headers / Triton 内部头文件:** `triton/Analysis/Utility.h`, `triton/Conversion/TritonGPUToLLVM/PatternTritonGPUOpToLLVM.h`, `triton/Conversion/TritonGPUToLLVM/TargetInfoBase.h`, `triton/Conversion/TritonGPUToLLVM/Utility.h`, `triton/Tools/LayoutUtils.h`
- **MLIR headers / MLIR 头文件:** `mlir/Support/LLVM.h`
- **LLVM headers / LLVM 头文件:** `llvm/ADT/STLExtras.h`
- **Standard/library headers / 标准或通用库头文件:** `ReduceScanCommon.h`
- **Primary APIs used / 主要 API:** `PatternRewriter`, `ConversionPatternRewriter`, `LLVMTypeConverter`, `ModuleOp`, `RankedTensorType`, `LinearLayout`, ... (+2 more)
- **Lowering role / 降级角色:** The implementation depends on source-dialect semantics being valid and emits forms expected by later LLVM or GPU stages. / 该实现依赖源方言语义已经正确，并输出供后续 LLVM 或 GPU 阶段使用的形式。
