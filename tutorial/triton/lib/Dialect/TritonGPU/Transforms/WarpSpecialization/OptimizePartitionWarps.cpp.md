# OptimizePartitionWarps.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `lib/Dialect/TritonGPU/Transforms/WarpSpecialization/OptimizePartitionWarps.cpp`
- **Purpose / 作用:** **EN:** Implements the Optimize Partition Warps transformation or optimization pass for the TritonGPU pipeline. **CN:** 为 TritonGPU 编译流程实现与 Optimize Partition Warps 相关的变换或优化 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-10

```cpp
   1: #include "mlir/IR/BuiltinOps.h"
   2: #include "mlir/IR/Verifier.h"
   3: #include "mlir/Pass/Pass.h"
   4: #include "mlir/Pass/PassManager.h"
   5: #include "triton/Analysis/AxisInfo.h"
   6: #include "triton/Conversion/TritonToTritonGPU/Passes.h"
   7: #include "triton/Dialect/Triton/IR/Utility.h"
   8: #include "triton/Dialect/TritonGPU/Transforms/Passes.h"
   9: #include "triton/Dialect/TritonNvidiaGPU/IR/Dialect.h"
  10: #include "llvm/ADT/ScopeExit.h"
```

- **EN:** Includes the interfaces this file depends on. Triton headers (`AxisInfo.h`, `Passes.h`, `Utility.h`, `Passes.h`, ... (+1 more)) provide domain-specific IR/support, MLIR headers (`BuiltinOps.h`, `Verifier.h`, `Pass.h`, `PassManager.h`) provide rewriting and analysis infrastructure, LLVM headers (`ScopeExit.h`) supply low-level utilities, and standard/library headers (None) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（`AxisInfo.h`, `Passes.h`, `Utility.h`, `Passes.h`, ... (+1 more)）提供领域特定 IR/支持逻辑，MLIR 头文件（`BuiltinOps.h`, `Verifier.h`, `Pass.h`, `PassManager.h`）提供重写与分析基础设施，LLVM 头文件（`ScopeExit.h`）提供底层工具，而标准/通用库头文件（None）提供通用能力。
### Lines 12-15

```cpp
  12: using namespace mlir;
  13: using namespace triton;
  14: using namespace triton::gpu;
  15: namespace ttng = triton::nvidia_gpu;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 17-19

```cpp
  17: //===----------------------------------------------------------------------===//
  18: // relayoutWarps
  19: //===----------------------------------------------------------------------===//
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 21-21

```cpp
  21: using RunPipelineFn = function_ref<LogicalResult(OpPassManager &, ModuleOp)>;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 23-31

```cpp
  23: // Take the body of a partition into a new `tt.func`. We can use this to run a
  24: // full compiler pipeline on the partition.
  25: static OwningOpRef<ModuleOp> takeIntoFunction(ModuleAxisInfoAnalysis &axisInfo,
  26:                                               Region *partition, int numWarps) {
  27:   // Forward the module attributes (target, number of threads per warp, etc.)
  28:   // onto the container module.
  29:   ModuleOp mod = axisInfo.getModuleOp();
  30:   OwningOpRef<ModuleOp> container = ModuleOp::create(mod.getLoc());
  31:   Block *containerBlock = container->getBody();
```

- **EN:** Defines `takeIntoFunction`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `takeIntoFunction`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 33-38

```cpp
  33:   auto b = OpBuilder::atBlockBegin(containerBlock);
  34:   FunctionType funcType = b.getFunctionType(partition->getArgumentTypes(), {});
  35:   auto containerFunc = FuncOp::create(b, mod.getLoc(), "container", funcType);
  36:   containerFunc.getBody().takeBody(*partition);
  37:   container.get()->setAttrs(mod->getAttrs());
  38:   container.get()->setAttr(AttrNumWarpsName, b.getI32IntegerAttr(numWarps));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 40-45

```cpp
  40:   // Replace `ttg.warp_return` with `tt.return` to make the IR valid.
  41:   containerFunc.walk([&](WarpReturnOp op) {
  42:     b.setInsertionPoint(op);
  43:     ReturnOp::create(b, op.getLoc());
  44:     op.erase();
  45:   });
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. The implementation traverses IR operations to collect facts or apply rewrites globally.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。
### Lines 47-49

```cpp
  47:   // This should make valid IR.
  48:   if (failed(mlir::verify(*container)))
  49:     llvm::report_fatal_error("expected partition region to make valid IR");
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 51-65

```cpp
  51:   // Attach axis info properties.
  52:   auto wsOp = partition->getParentOfType<WarpSpecializeOp>();
  53:   auto *funcInfo =
  54:       axisInfo.getFuncData(wsOp->getParentOfType<FunctionOpInterface>());
  55:   assert(funcInfo && "expected to find function axis info");
  56:   for (auto [i, capture] :
  57:        llvm::enumerate(wsOp.getPartitionOp().getExplicitCaptures())) {
  58:     AxisInfo info = funcInfo->lookup(capture);
  59:     containerFunc.setArgAttr(i, "tt.contiguity",
  60:                              b.getI64IntegerAttr(info.getContiguity(0)));
  61:     containerFunc.setArgAttr(i, "tt.divisibility",
  62:                              b.getI64IntegerAttr(info.getDivisibility(0)));
  63:     containerFunc.setArgAttr(i, "tt.constancy",
  64:                              b.getI64IntegerAttr(info.getConstancy(0)));
  65:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 67-68

```cpp
  67:   return container;
  68: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 70-73

```cpp
  70: // Take the partition body out of the container module and function.
  71: static void extractPartitionBody(OwningOpRef<ModuleOp> container,
  72:                                  Region *partition) {
  73:   auto containerFunc = cast<FuncOp>(container->lookupSymbol("container"));
```

- **EN:** Defines `extractPartitionBody`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `extractPartitionBody`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 75-80

```cpp
  75:   // Rewrite the returns.
  76:   containerFunc.walk([](ReturnOp op) {
  77:     OpBuilder b(op);
  78:     WarpReturnOp::create(b, op.getLoc());
  79:     op.erase();
  80:   });
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 82-83

```cpp
  82:   partition->takeBody(containerFunc.getBody());
  83: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 85-90

```cpp
  85: // Reset the layouts of operations in a region and re-run layout assignment.
  86: static LogicalResult relayoutWarps(ModuleAxisInfoAnalysis &axisInfo,
  87:                                    Region *partition, int prevNumWarps,
  88:                                    int newNumWarps, RunPipelineFn runPipeline) {
  89:   OwningOpRef<ModuleOp> container =
  90:       takeIntoFunction(axisInfo, partition, prevNumWarps);
```

- **EN:** Defines `relayoutWarps`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `relayoutWarps`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 92-102

```cpp
  92:   // Start by removing all tensor encodings.
  93:   mlir::AttrTypeReplacer replacer;
  94:   replacer.addReplacement(
  95:       [](RankedTensorType ty) { return ty.cloneWithEncoding({}); });
  96:   // But don't remove them from the tensors inside descriptors.
  97:   replacer.addReplacement([](TensorDescType ty) -> std::pair<Type, WalkResult> {
  98:     return {ty, WalkResult::skip()};
  99:   });
 100:   replacer.recursivelyReplaceElementsIn(*container, /*replaceAttrs=*/false,
 101:                                         /*replaceLocs=*/false,
 102:                                         /*replaceTypes=*/true);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 104-109

```cpp
 104:   ModuleOp mod = axisInfo.getModuleOp();
 105:   auto target = mod->getAttrOfType<StringAttr>(AttrTargetName);
 106:   if (!target)
 107:     return mlir::emitError(mod.getLoc(), "module missing target specification");
 108:   int threadsPerWarp = TritonGPUDialect::getThreadsPerWarp(mod);
 109:   int numCTAs = TritonGPUDialect::getNumCTAs(mod);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 111-124

```cpp
 111:   // Enable `convert-triton-to-tritongpu` to rematerialize source layouts for
 112:   // TTG dialect operations. They will get cleared later.
 113:   OpPassManager pm;
 114:   pm.addPass(
 115:       createConvertTritonToTritonGPU({target.str(), newNumWarps, threadsPerWarp,
 116:                                       numCTAs, /*enableSourceRemat=*/true}));
 117:   pm.addPass(createRelayoutTritonGPU());
 118:   if (failed(runPipeline(pm, *container)))
 119:     return failure();
 120:   // Clear source rematerializations by propagating the source layout.
 121:   container->walk([](UnrealizedConversionCastOp op) {
 122:     op.getResult(0).replaceAllUsesWith(op.getOperand(0));
 123:     op.erase();
 124:   });
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 126-133

```cpp
 126:   pm.clear();
 127:   pm.addPass(createTritonGPUCoalesce());
 128:   pm.addPass(createTritonGPURemoveLayoutConversions());
 129:   pm.addPass(createTritonGPUOptimizeThreadLocality());
 130:   pm.addPass(createTritonGPUAccelerateMatmul());
 131:   pm.addPass(createTritonGPURemoveLayoutConversions());
 132:   if (failed(runPipeline(pm, *container)))
 133:     return failure();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 135-137

```cpp
 135:   extractPartitionBody(std::move(container), partition);
 136:   return success();
 137: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 139-141

```cpp
 139: //===----------------------------------------------------------------------===//
 140: // optimizePartitionWarps
 141: //===----------------------------------------------------------------------===//
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 143-151

```cpp
 143: // Get the number of i32 registers required to store a tensor.
 144: static unsigned getTensorNumI32Regs(RankedTensorType ty) {
 145:   unsigned numElems = getTotalElemsPerThread(ty) *
 146:                       product(getThreadsPerWarp(ty)) *
 147:                       product(getWarpsPerCTA(ty));
 148:   unsigned elSize =
 149:       isa<PointerType>(ty.getElementType()) ? 64 : ty.getElementTypeBitWidth();
 150:   return numElems * elSize / 32;
 151: }
```

- **EN:** Defines accessor/helper `getTensorNumI32Regs` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getTensorNumI32Regs`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 153-170

```cpp
 153: static LogicalResult optimizePartitionNumWarps(ModuleAxisInfoAnalysis &axisInfo,
 154:                                                WarpSpecializeOp wsOp,
 155:                                                RunPipelineFn runPipeline) {
 156:   // Extremely rough estimate of the number of registers needed per partition.
 157:   // For each partition, get the number of i32 registers used by the largest
 158:   // tensor value.
 159:   //
 160:   // Because the partition region is isolated from above, we could in theory
 161:   // compile it to PTX and read the number of registers that got allocated.
 162:   SmallVector<unsigned> maxTensorRegs;
 163:   for (Region *partition : wsOp.getPartitionRegions()) {
 164:     unsigned &tensorRegs = maxTensorRegs.emplace_back(0);
 165:     partition->walk([&](Operation *op) {
 166:       for (Type type :
 167:            llvm::concat<Type>(op->getOperandTypes(), op->getResultTypes())) {
 168:         if (auto tensor = dyn_cast<RankedTensorType>(type))
 169:           tensorRegs = std::max(tensorRegs, getTensorNumI32Regs(tensor));
 170:       }
```

- **EN:** Defines `optimizePartitionNumWarps`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. The implementation traverses IR operations to collect facts or apply rewrites globally. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `optimizePartitionNumWarps`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 171-175

```cpp
 171:     });
 172:     // Assume that the largest tensor accounts for half of the registers used
 173:     // by a warpgroup.
 174:     tensorRegs *= 2;
 175:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 177-191

```cpp
 177:   // Reduce the number of warps used by partitions. For partitions with no
 178:   // tensor computations, always reduce them to 1 warp.
 179:   //
 180:   // We can't use `nvvm.setmaxnreg` because this requires a known value for
 181:   // `maxnreg` on the kernel, which is currently controlled by the frontend.
 182:   // Thus, assume PTXAS will evenly distribute the total pool of registers
 183:   // across all warps.
 184:   //
 185:   // If the compiler could control that, then we could allow non-uniform
 186:   // register distributions, mostly beneficial for single-warp warpgroups that
 187:   // just do some artihmetic.
 188:   constexpr unsigned nTotalRegs = 1 << 16; // for Blackwell SMs
 189:   const unsigned threadsPerWarp =
 190:       TritonGPUDialect::getThreadsPerWarp(axisInfo.getModuleOp());
 191:   const unsigned defaultNumWarps = lookupNumWarps(wsOp);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 193-194

```cpp
 193:   SmallVector<int32_t> partitionNumWarps =
 194:       llvm::to_vector(wsOp.getPartitionNumWarps());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 196-211

```cpp
 196:   // Determine if a partition has a lower limit on the number of warps.
 197:   SmallVector<int32_t> minWarpsForPartition(partitionNumWarps.size(), 1);
 198:   for (auto [minWarps, region] :
 199:        llvm::zip(minWarpsForPartition, wsOp.getPartitionRegions())) {
 200:     region->walk([minWarps = &minWarps](Operation *op) {
 201:       // Some instructions have critical throughput if have low register usage.
 202:       // Make sure there are enough warps for these ops to execute quickly.
 203:       // TMAStoreLikeOps stay in the main partition, so they should not appear
 204:       // in partition regions here.
 205:       if (isa<ttng::TMALoadLikeOpInterface>(op))
 206:         *minWarps = 2;
 207:       // TMEM ops require at least 4 warps to be able to read all lanes.
 208:       else if (isa<ttng::TMEMLoadOp, ttng::TMEMStoreOp, ttng::TMEMAllocOp>(op))
 209:         *minWarps = 4;
 210:     });
 211:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 213-215

```cpp
 213:   bool changed;
 214:   do {
 215:     changed = false;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 217-234

```cpp
 217:     // Assuming even distribution of registers, given the total number of warps
 218:     // currently allocated, we can guess the number of registers PTXAS will
 219:     // distribute to each warp.
 220:     //
 221:     // For example, given 18 warps and a tensor<128x256xf32> contained in an
 222:     // 8-warp partition, we have (nTotalRegs/32/18) = ~113 regs per thread, and
 223:     // the tensor requires 128 regs per thread in its partition. In this case,
 224:     // nothing can be done.
 225:     //
 226:     // However, given a tensor<128x128xf32>, this requires only 64 regs per
 227:     // thread in 8 warps. If we reduce the size of the warp to 4, the overall
 228:     // regs per thread increases to (nTotalRegs/32/14) = ~146 regs per thread,
 229:     // while the tensor now requires 128 regs per thread. This works.
 230:     //
 231:     // The next iteration sees ~170 regs per thread, but the tensor will require
 232:     // 256, which is too many. So the algorithm stops at 4 warps. Evidently, if
 233:     // there are other partitions that can be reduced, we have to iterate this
 234:     // algorithm.
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 235-236

```cpp
 235:     int32_t curTotalNumWarps = std::accumulate(
 236:         partitionNumWarps.begin(), partitionNumWarps.end(), defaultNumWarps);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 238-254

```cpp
 238:     for (auto [minWarps, numWarps, tensorRegs] :
 239:          llvm::zip(minWarpsForPartition, partitionNumWarps, maxTensorRegs)) {
 240:       if (numWarps <= minWarps)
 241:         continue;
 242:       // Check if reducing the number of warps will still fit the tensor. If it
 243:       // didn't fit to begin with, it won't fit after shrinking.
 244:       unsigned reqRegsPerThread = tensorRegs / threadsPerWarp / (numWarps / 2);
 245:       unsigned nextTotalNumWarps = curTotalNumWarps - (numWarps / 2);
 246:       unsigned nextRegsPerThread =
 247:           nTotalRegs / threadsPerWarp / nextTotalNumWarps;
 248:       if (reqRegsPerThread <= nextRegsPerThread) {
 249:         numWarps /= 2;
 250:         changed = true;
 251:         break;
 252:       }
 253:     }
 254:   } while (changed);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 256-261

```cpp
 256:   SmallVector<int32_t> estRegUsage(partitionNumWarps.size());
 257:   for (auto [partition, newNumWarps, prevNumWarps, tensorRegs, estRegs] :
 258:        llvm::zip(wsOp.getPartitionRegions(), partitionNumWarps,
 259:                  wsOp.getPartitionNumWarps(), maxTensorRegs, estRegUsage)) {
 260:     // "Guess" the register usage for each partition.
 261:     estRegs = tensorRegs ? 88 : 24;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 263-275

```cpp
 263:     // Layouts need to be reassigned if the number of warps changed and there
 264:     // are tensor computations.
 265:     if (newNumWarps == prevNumWarps || !tensorRegs)
 266:       continue;
 267:     // We need to reassign layouts.
 268:     if (failed(relayoutWarps(axisInfo, partition, prevNumWarps, newNumWarps,
 269:                              runPipeline)))
 270:       return failure();
 271:   }
 272:   wsOp.setRequestedRegisters(estRegUsage);
 273:   wsOp.setPartitionNumWarps(partitionNumWarps);
 274:   return success();
 275: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. The code uses `LogicalResult` to stay conservative when preconditions are not met.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 代码通过 `LogicalResult` 在前提不满足时保持保守处理。
### Lines 277-279

```cpp
 277: //===----------------------------------------------------------------------===//
 278: // Pass Definition
 279: //===----------------------------------------------------------------------===//
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 281-284

```cpp
 281: namespace mlir::triton::gpu {
 282: #define GEN_PASS_DEF_TRITONGPUOPTIMIZEPARTITIONWARPS
 283: #include "triton/Dialect/TritonGPU/Transforms/Passes.h.inc"
 284: } // namespace mlir::triton::gpu
```

- **EN:** Pulls in generated definitions from TableGen/MLIR include fragments so the handwritten code can reuse auto-generated declarations.
- **CN:** 这里引入由 TableGen/MLIR 生成的定义片段，使手写代码能够复用自动生成的声明。
### Lines 286-291

```cpp
 286: namespace {
 287: struct OptimizePartitionWarps
 288:     : triton::gpu::impl::TritonGPUOptimizePartitionWarpsBase<
 289:           OptimizePartitionWarps> {
 290:   using TritonGPUOptimizePartitionWarpsBase::
 291:       TritonGPUOptimizePartitionWarpsBase;
```

- **EN:** Defines `OptimizePartitionWarps`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `OptimizePartitionWarps`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 293-295

```cpp
 293:   void runOnOperation() override;
 294: };
 295: } // namespace
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 297-299

```cpp
 297: void OptimizePartitionWarps::runOnOperation() {
 298:   SmallVector<WarpSpecializeOp> wsOps;
 299:   getOperation().walk([&](WarpSpecializeOp wsOp) { wsOps.push_back(wsOp); });
```

- **EN:** Defines `OptimizePartitionWarps::runOnOperation`, the pass entry point. It gathers analysis information and applies the file's transformation logic to the current operation. The implementation traverses IR operations to collect facts or apply rewrites globally.
- **CN:** 这里定义 `OptimizePartitionWarps::runOnOperation`，即 pass 的入口函数。它会收集分析信息，并把本文件的变换逻辑应用到当前操作上。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。
### Lines 301-303

```cpp
 301:   if (wsOps.empty()) {
 302:     return;
 303:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 305-312

```cpp
 305:   ModuleAxisInfoAnalysis axisInfo(getOperation());
 306:   auto runPipelineFn = [&](OpPassManager &pm, ModuleOp container) {
 307:     // The module must be directly nested under the current op for `runPipeline`
 308:     // to work.
 309:     getOperation().push_back(container);
 310:     llvm::scope_exit remove([&] { container->remove(); });
 311:     return runPipeline(pm, container);
 312:   };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 314-319

```cpp
 314:   for (auto wsOp : wsOps) {
 315:     if (failed(optimizePartitionNumWarps(axisInfo, wsOp, runPipelineFn))) {
 316:       return signalPassFailure();
 317:     }
 318:   }
 319: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。

## Key Concepts / 关键概念
- **EN:** The main concern is pass-driven transformation around optimize partition warps.
  **CN:** 核心关注点是围绕 Optimize Partition Warps 的 pass 驱动变换。
- **EN:** Tensor shape and element-type reasoning is central to the implementation.
  **CN:** 张量形状与元素类型推导是实现中的核心内容。
- **EN:** The code reasons at module scope rather than only on isolated operations.
  **CN:** 代码在模块范围内进行分析，而不只是处理单个操作。
- **EN:** Verification logic enforces structural invariants early in the pipeline.
  **CN:** 验证逻辑在编译流程早期强制检查结构不变量。
- **EN:** Axis information captures per-dimension contiguity, divisibility, or constancy facts.
  **CN:** AxisInfo 记录逐维的连续性、可整除性或常量性等信息。
- **EN:** Tensor Memory Accelerator related logic appears in this implementation.
  **CN:** 此实现涉及 Tensor Memory Accelerator（TMA）相关逻辑。

## Dependencies / 依赖关系
- **Internal Triton headers / Triton 内部头文件:** `triton/Analysis/AxisInfo.h`, `triton/Conversion/TritonToTritonGPU/Passes.h`, `triton/Dialect/Triton/IR/Utility.h`, `triton/Dialect/TritonGPU/Transforms/Passes.h`, `triton/Dialect/TritonNvidiaGPU/IR/Dialect.h`, `triton/Dialect/TritonGPU/Transforms/Passes.h.inc`
- **MLIR headers / MLIR 头文件:** `mlir/IR/BuiltinOps.h`, `mlir/IR/Verifier.h`, `mlir/Pass/Pass.h`, `mlir/Pass/PassManager.h`
- **LLVM headers / LLVM 头文件:** `llvm/ADT/ScopeExit.h`
- **Standard/library headers / 标准或通用库头文件:** None
- **Generated fragments / 生成片段:** `triton/Dialect/TritonGPU/Transforms/Passes.h.inc`
- **Primary APIs used / 主要 API:** `ModuleOp`, `RankedTensorType`, `OpBuilder`
- **Pipeline role / 流程角色:** This file participates in Triton pass pipelines and usually expects upstream analyses or dialect invariants to have prepared the IR. / 本文件参与 Triton 的 pass 流程，通常依赖上游分析或方言不变量先把 IR 准备好。
