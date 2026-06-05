# PipeliningUtility.h — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./include/triton/Dialect/TritonGPU/Transforms/PipeliningUtility.h`
- **EN:** Provides helper utilities that support transformations or analysis in this area.
- **CN:** 提供支撑该领域分析或变换的辅助工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```cpp
   1: #ifndef TRITON_TRITONGPU_TRANSFORMS_PIPELINER_PIPELINING_UTILITY_H_
   2: #define TRITON_TRITONGPU_TRANSFORMS_PIPELINER_PIPELINING_UTILITY_H_
```
**EN:** This block establishes the header guard so the declarations in this file are only processed once per translation unit.
**CN:** 该代码块建立头文件保护宏，避免同一翻译单元重复包含本文件时出现重复声明。

### Lines 4-9
```cpp
   4: #include "mlir/Dialect/SCF/IR/SCF.h"
   5: #include "triton/Dialect/Triton/IR/Dialect.h"
   6: #include "triton/Dialect/TritonGPU/IR/Dialect.h"
   7: #include <optional>
   8: #include <utility>
   9: #include <vector>
```
**EN:** This block imports the direct dependencies needed here, including mlir/Dialect/SCF/IR/SCF.h, triton/Dialect/Triton/IR/Dialect.h, triton/Dialect/TritonGPU/IR/Dialect.h, <optional>, <utility>, and <vector>.
**CN:** 该代码块引入后续声明直接依赖的头文件，例如 mlir/Dialect/SCF/IR/SCF.h, triton/Dialect/Triton/IR/Dialect.h, triton/Dialect/TritonGPU/IR/Dialect.h, <optional>, <utility>, and <vector>。

### Lines 11-14
```cpp
  11: namespace mlir {
  12: class DominanceInfo;
  13: class ImplicitLocOpBuilder;
  14: namespace triton {
```
**EN:** This block stores supporting state such as DominanceInfo and ImplicitLocOpBuilder, which other APIs in the file consume.
**CN:** 该代码块声明了 DominanceInfo and ImplicitLocOpBuilder 等支撑状态，供本文件中的其他 API 使用。

### Lines 16-27
```cpp
  16: static const char *kNumStagesAttrName = "tt.num_stages";
  17: static const char *kDisallowAccMultiBufferAttrName =
  18:     "tt.disallow_acc_multi_buffer";
  19: static const char *kWarpSpecializeAttrName = "tt.warp_specialize";
  20: static const char *kLoopStageAttrName = "loop.stage";
  21: static const char *kLoopClusterAttrName = "loop.cluster";
  22: static const char *kScheduledMaxStageAttrName = "tt.scheduled_max_stage";
  23: class CoarseSchedule;
  24: class ModuleAxisInfoAnalysis;
  25: //===----------------------------------------------------------------------===//
  26: // Hoisting Utilities
  27: //===----------------------------------------------------------------------===//
```
**EN:** This block stores supporting state such as CoarseSchedule and ModuleAxisInfoAnalysis, which other APIs in the file consume.
**CN:** 该代码块声明了 CoarseSchedule and ModuleAxisInfoAnalysis 等支撑状态，供本文件中的其他 API 使用。

### Lines 29-30
```cpp
  29: // By default, an operation can be hoisted if it is pure scalar operation.
  30: bool isPureScalarOp(Operation *op);
```
**EN:** This block declares or defines callable APIs such as isPureScalarOp, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isPureScalarOp 等可调用 API，用来封装这里提供的核心行为。

### Lines 32-47
```cpp
  32: // Given a set of values and a reference operation, return true if all of the
  33: // values dominate the reference operation OR a set of "trivial" operations can
  34: // be moved before the reference operation such that the value set dominates the
  35: // reference operation.
  36: //
  37: // Returns false if it is not possible to make the values dominate the reference
  38: // operation. The function determines "trivial"-ness with the given callback.
  39: // By default, it determines that memory-effect-free and scalar operations are
  40: // trivial.
  41: bool getDominatingValueSetOpsToHoist(
  42:     DominanceInfo &domInfo, Operation *refOp, ArrayRef<Value> valueSet,
  43:     llvm::SetVector<Operation *> &toHoist,
  44:     function_ref<bool(Operation *)> canHoist = isPureScalarOp,
  45:     function_ref<bool(BlockArgument)> canUseArg = [](BlockArgument) {
  46:       return false;
  47:     });
```
**EN:** This block declares or defines callable APIs such as getDominatingValueSetOpsToHoist and bool, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getDominatingValueSetOpsToHoist and bool 等可调用 API，用来封装这里提供的核心行为。

### Lines 49-54
```cpp
  49: // Hoist the given set of operations above the reference operation.
  50: void hoistOpsBefore(Operation *refOp,
  51:                     const llvm::SetVector<Operation *> &toHoist);
  52: // Hoist the given set of operations before the iterator.
  53: void hoistOpsBefore(Block *block, Block::iterator it,
  54:                     const llvm::SetVector<Operation *> &toHoist);
```
**EN:** This block declares or defines callable APIs such as hoistOpsBefore, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 hoistOpsBefore 等可调用 API，用来封装这里提供的核心行为。

### Lines 56-58
```cpp
  56: //===----------------------------------------------------------------------===//
  57: // Sinking Utilities
  58: //===----------------------------------------------------------------------===//
```
**EN:** This comment block records the intent and constraints of the surrounding code: ===----------------------------------------------------------------------===// Sinking Utilities ===----------------------------------------------------------------------===//.
**CN:** 该注释块解释了周边代码的设计背景、意图或约束条件，帮助读者理解后续实现。

### Lines 60-63
```cpp
  60: // Sink a value redefinition into a block, provided that the block is dominated
  61: // by `in` and postdominated by `out`.
  62: Value sinkValueRedefinition(RewriterBase &rewriter, Value in, Value out,
  63:                             Block *block);
```
**EN:** This block declares or defines callable APIs such as sinkValueRedefinition, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 sinkValueRedefinition 等可调用 API，用来封装这里提供的核心行为。

### Lines 65-67
```cpp
  65: //===----------------------------------------------------------------------===//
  66: // Loop Pipelining Utilities
  67: //===----------------------------------------------------------------------===//
```
**EN:** This comment block records the intent and constraints of the surrounding code: ===----------------------------------------------------------------------===// Loop Pipelining Utilities ===---------------------------------------------------------------------....
**CN:** 该注释块解释了周边代码的设计背景、意图或约束条件，帮助读者理解后续实现。

### Lines 69-70
```cpp
  69: bool loopHasDistGreaterThanOne(scf::ForOp forOp);
  70: bool isOuterLoop(scf::ForOp forOp);
```
**EN:** This block declares or defines callable APIs such as loopHasDistGreaterThanOne and isOuterLoop, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 loopHasDistGreaterThanOne and isOuterLoop 等可调用 API，用来封装这里提供的核心行为。

### Lines 72-73
```cpp
  72: /// Function to mask operations during scheduling.
  73: Operation *predicateOp(RewriterBase &rewriter, Operation *op, Value pred);
```
**EN:** This block declares or defines callable APIs such as predicateOp, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 predicateOp 等可调用 API，用来封装这里提供的核心行为。

### Lines 75-77
```cpp
  75: /// Wrap the operation into a MaskOp using the provided predicate, enabling high
  76: /// level predication abstraction during pipelining.
  77: Operation *wrapInMaskOp(RewriterBase &rewriter, Operation *op, Value pred);
```
**EN:** This block declares or defines callable APIs such as wrapInMaskOp, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 wrapInMaskOp 等可调用 API，用来封装这里提供的核心行为。

### Lines 79-81
```cpp
  79: // Utilize high level predication abstraction to perform optimizations before
  80: // lowering to predicated operations
  81: void resolveMaskOp(ModuleOp moduleOp);
```
**EN:** This block declares or defines callable APIs such as resolveMaskOp, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 resolveMaskOp 等可调用 API，用来封装这里提供的核心行为。

### Lines 83-85
```cpp
  83: // Return true if the given ForOp has the attribute
  84: // `tt.disallow_acc_multi_buffer` set to true.
  85: bool getDisallowAccMultiBuffer(scf::ForOp forOp);
```
**EN:** This block declares or defines callable APIs such as getDisallowAccMultiBuffer, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getDisallowAccMultiBuffer 等可调用 API，用来封装这里提供的核心行为。

### Lines 87-95
```cpp
  87: // Return the definition of the given value. If the value is a loop-carried
  88: // dependency, return the definition and the distance to it.
  89: std::pair<OpResult, int64_t> getDefinitionAndDistance(scf::ForOp forOp,
  90:                                                       Value value);
  91: // Return the defining op of the given value, if the Value is an argument of the
  92: // loop return the associated defining op in the loop and its distance to the
  93: // Value.
  94: std::pair<Operation *, int64_t> getDefiningOpAndDistance(scf::ForOp forOp,
  95:                                                          Value value);
```
**EN:** This block declares or defines callable APIs such as getDefinitionAndDistance and getDefiningOpAndDistance, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getDefinitionAndDistance and getDefiningOpAndDistance 等可调用 API，用来封装这里提供的核心行为。

### Lines 97-100
```cpp
  97: // Return maximum length of the vectorized copy between registers and shared
  98: // memory for the given tensor type and shared encoding.
  99: int getCopyVecBytes(RankedTensorType registerTy,
 100:                     gpu::SharedEncodingTrait sharedEnc);
```
**EN:** This block declares or defines callable APIs such as getCopyVecBytes, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getCopyVecBytes 等可调用 API，用来封装这里提供的核心行为。

### Lines 102-103
```cpp
 102: bool canBeConvertedToAsyncLoad(
 103:     triton::LoadOp loadOp, triton::ModuleAxisInfoAnalysis &axisInfoAnalysis);
```
**EN:** This block declares or defines callable APIs such as canBeConvertedToAsyncLoad, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 canBeConvertedToAsyncLoad 等可调用 API，用来封装这里提供的核心行为。

### Lines 105-107
```cpp
 105: // Serialize the latencies of the operations in the loops into the latency
 106: // attribute.
 107: void serializeLatencies(ModuleOp module, DenseMap<Operation *, int> &opLatency);
```
**EN:** This block declares or defines callable APIs such as serializeLatencies, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 serializeLatencies 等可调用 API，用来封装这里提供的核心行为。

### Lines 109-112
```cpp
 109: // Serialize the self latencies of the operations in the loops into the
 110: // self_latency attribute.
 111: void serializeSelfLatencies(ModuleOp module,
 112:                             DenseMap<Operation *, int> &opSelfLatency);
```
**EN:** This block declares or defines callable APIs such as serializeSelfLatencies, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 serializeSelfLatencies 等可调用 API，用来封装这里提供的核心行为。

### Lines 114-115
```cpp
 114: // Deserialize the latencies of the operations in the loops from the attribute.
 115: DenseMap<Operation *, int> deserializeLatencies(Operation *op);
```
**EN:** This block declares or defines callable APIs such as deserializeLatencies, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 deserializeLatencies 等可调用 API，用来封装这里提供的核心行为。

### Lines 117-124
```cpp
 117: // Create an allocation for multibuffered scalars.
 118: Value createScalarAlloc(ImplicitLocOpBuilder &rewriter, Type type,
 119:                         unsigned numBuffers);
 120: // Create an allocation and init the mbarriers.
 121: Value createBarrierAlloc(Operation *op, int numBarriers, int arriveCount = 1);
 122: // Create an allocation that can hold distance number of tensor shapes.
 123: Value createAlloc(Operation *insertBefore, RankedTensorType ty, Location loc,
 124:                   gpu::SharedEncodingTrait sharedEnc, unsigned distance);
```
**EN:** This block declares or defines callable APIs such as createScalarAlloc, createBarrierAlloc, and createAlloc, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 createScalarAlloc, createBarrierAlloc, and createAlloc 等可调用 API，用来封装这里提供的核心行为。

### Lines 126-129
```cpp
 126: // Determine if the operation is a TMA load.
 127: inline bool isTMALoad(Operation *op) {
 128:   return isa<DescriptorLoadLikeOpInterface>(op);
 129: }
```
**EN:** This block declares or defines callable APIs such as isTMALoad, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isTMALoad 等可调用 API，用来封装这里提供的核心行为。

### Lines 131-132
```cpp
 131: // Determine if the operation can be lowered to an async load.
 132: bool canBeAsyncLoad(Operation *op);
```
**EN:** This block declares or defines callable APIs such as canBeAsyncLoad, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 canBeAsyncLoad 等可调用 API，用来封装这里提供的核心行为。

### Lines 134-144
```cpp
 134: // Fold consecutive wait ops of the same kind into a single wait.
 135: // `isCounterBarrier` returns true on ops that act as a hard boundary while
 136: // scanning forward (typically the producers whose tokens a later wait
 137: // consumes). `createWait` builds the merged wait from the union of operand
 138: // tokens and the minimum `num`.
 139: void combineRedundantWaitOps(
 140:     llvm::SmallSetVector<Operation *, 8> &waitOps,
 141:     llvm::function_ref<bool(Operation * /*candidate*/)> isCounterBarrier,
 142:     llvm::function_ref<Operation *(OpBuilder &, Location,
 143:                                    ValueRange /*operands*/, unsigned /*num*/)>
 144:         createWait);
```
**EN:** This block declares or defines callable APIs such as combineRedundantWaitOps and bool, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 combineRedundantWaitOps and bool 等可调用 API，用来封装这里提供的核心行为。

### Lines 146-148
```cpp
 146: // Get the type of the view of a multi-buffered tensor value.
 147: gpu::MemDescType getBufferViewType(gpu::MemDescType allocTy,
 148:                                    bool mutableMemory = true);
```
**EN:** This block declares or defines callable APIs such as getBufferViewType, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getBufferViewType 等可调用 API，用来封装这里提供的核心行为。

### Lines 150-153
```cpp
 150: // Get a mutable, multi-buffered version of the given memdesc type, with
 151: // multiplicity "depth".
 152: gpu::MemDescType getMultiBufferedType(gpu::MemDescType memDescType,
 153:                                       int32_t depth);
```
**EN:** This block declares or defines callable APIs such as getMultiBufferedType, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getMultiBufferedType 等可调用 API，用来封装这里提供的核心行为。

### Lines 155-158
```cpp
 155: // Get a generic shared encoding for a tensor.
 156: gpu::SharedEncodingTrait getSharedEncoding(RankedTensorType ty);
 157: // Get a shared encoding for a tensor based on its uses.
 158: gpu::SharedEncodingTrait getSharedEncoding(Operation *loadOp);
```
**EN:** This block declares or defines callable APIs such as getSharedEncoding, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getSharedEncoding 等可调用 API，用来封装这里提供的核心行为。

### Lines 160-162
```cpp
 160: // Get the number of stages to pipeline the loop with, if it is explicitly
 161: // specified.
 162: int getNumStagesOrDefault(scf::ForOp forOp, int defaultNumStages);
```
**EN:** This block declares or defines callable APIs such as getNumStagesOrDefault, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getNumStagesOrDefault 等可调用 API，用来封装这里提供的核心行为。

### Lines 164-171
```cpp
 164: // Given a result of MemDescIndex, or Alloca, create a MemDescIndex with a
 165: // single buffer slice (leading dimension equal to 1), at the given index.
 166: TypedValue<triton::gpu::MemDescType>
 167: createSingleBufferView(OpBuilder &builder, Value alloc, Value idx);
 168: // Given a result of MemDescIndex, or Alloca, create a MemDescIndex with a
 169: // single buffer slice (leading dimension equal to 1), at the given index.
 170: TypedValue<triton::gpu::MemDescType>
 171: createSingleBufferView(OpBuilder &builder, Value alloc, int idx);
```
**EN:** This block declares or defines callable APIs such as createSingleBufferView, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 createSingleBufferView 等可调用 API，用来封装这里提供的核心行为。

### Lines 173-175
```cpp
 173: Value createIncrementModulo(OpBuilder &builder, Location loc, Value counter,
 174:                             Value modulus, Value zero, Value one,
 175:                             Value *outWrapCond = nullptr);
```
**EN:** This block declares or defines callable APIs such as createIncrementModulo, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 createIncrementModulo 等可调用 API，用来封装这里提供的核心行为。

### Lines 177-177
```cpp
 177: scf::ForOp lowerTMADescriptors(scf::ForOp forOp, CoarseSchedule &schedule);
```
**EN:** This block declares or defines callable APIs such as lowerTMADescriptors, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 lowerTMADescriptors 等可调用 API，用来封装这里提供的核心行为。

### Lines 179-181
```cpp
 179: DenseSet<Operation *>
 180: getTopLevelUsersInLoop(Operation *op, scf::ForOp forOp,
 181:                        std::function<bool(Operation *)> filter = nullptr);
```
**EN:** This block declares or defines callable APIs such as getTopLevelUsersInLoop and bool, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getTopLevelUsersInLoop and bool 等可调用 API，用来封装这里提供的核心行为。

### Lines 183-187
```cpp
 183: // Return the "first" op in terms of the stage and cluser ordering
 184: Operation *
 185: getFirstUseOfPipelinedOp(ArrayRef<Operation *> ops, scf::ForOp forOp,
 186:                          CoarseSchedule &schedule,
 187:                          std::function<bool(Operation *)> filterUse = nullptr);
```
**EN:** This block declares or defines callable APIs such as getFirstUseOfPipelinedOp and bool, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getFirstUseOfPipelinedOp and bool 等可调用 API，用来封装这里提供的核心行为。

### Lines 189-193
```cpp
 189: // Return the "last" op in terms of the stage and cluser ordering
 190: Operation *
 191: getLastUseOfPipelinedOp(ArrayRef<Operation *> ops, scf::ForOp forOp,
 192:                         CoarseSchedule &schedule,
 193:                         std::function<bool(Operation *)> filterUse = nullptr);
```
**EN:** This block declares or defines callable APIs such as getLastUseOfPipelinedOp and bool, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getLastUseOfPipelinedOp and bool 等可调用 API，用来封装这里提供的核心行为。

### Lines 195-198
```cpp
 195: // Clean up attributes passing over schedules across stages in pipelining
 196: void removePipeliningAttributes(ModuleOp moduleOp);
 197: } // namespace triton
 198: } // namespace mlir
```
**EN:** This block declares or defines callable APIs such as removePipeliningAttributes, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 removePipeliningAttributes 等可调用 API，用来封装这里提供的核心行为。

### Lines 200-200
```cpp
 200: #endif // TRITON_TRITONGPU_TRANSFORMS_PIPELINER_PIPELINING_UTILITY_H_
```
**EN:** This block closes the header guard started at the top of the file.
**CN:** 该代码块结束文件顶部开始的头文件保护宏。

## Key Concepts / 关键概念
- **EN:** axis reasoning  
  **CN:** 轴信息推理
- **EN:** buffer modeling  
  **CN:** 缓冲区建模
- **EN:** layout encodings  
  **CN:** 布局编码
- **EN:** warp-level execution  
  **CN:** warp 级执行
- **EN:** LLVM lowering  
  **CN:** LLVM 降级
- **EN:** MLIR dialect definition  
  **CN:** MLIR 方言定义
- **EN:** operation definitions  
  **CN:** 操作定义
- **EN:** attributes  
  **CN:** 属性

## Dependencies / 依赖关系
- **Project includes / 项目内依赖:**
  - `mlir/Dialect/SCF/IR/SCF.h`
  - `triton/Dialect/Triton/IR/Dialect.h`
  - `triton/Dialect/TritonGPU/IR/Dialect.h`
- **System or external includes / 系统或外部依赖:**
  - `<optional>`
  - `<utility>`
  - `<vector>`
