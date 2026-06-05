# Utility.h — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./include/triton/Dialect/TritonInstrument/IR/Utility.h`
- **EN:** Provides helper utilities that support transformations or analysis in this area.
- **CN:** 提供支撑该领域分析或变换的辅助工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```cpp
   1: #ifndef TRITONINSTRUMENT_UTILITY_H
   2: #define TRITONINSTRUMENT_UTILITY_H
```
**EN:** This block establishes the header guard so the declarations in this file are only processed once per translation unit.
**CN:** 该代码块建立头文件保护宏，避免同一翻译单元重复包含本文件时出现重复声明。

### Lines 4-10
```cpp
   4: #include "mlir/Dialect/Arith/IR/Arith.h"
   5: #include "triton/Analysis/BufferRegion.h"
   6: #include "triton/Dialect/Triton/IR/Utility.h"
   7: #include "triton/Dialect/TritonGPU/IR/Attributes.h"
   8: #include "triton/Dialect/TritonInstrument/IR/ConSanConstants.h"
   9: #include "triton/Dialect/TritonInstrument/IR/Dialect.h"
  10: #include "llvm/Support/MathExtras.h"
```
**EN:** This block imports the direct dependencies needed here, including mlir/Dialect/Arith/IR/Arith.h, triton/Analysis/BufferRegion.h, triton/Dialect/Triton/IR/Utility.h, triton/Dialect/TritonGPU/IR/Attributes.h, triton/Dialect/TritonInstrument/IR/ConSanConstants.h, and triton/Dialect/TritonInstrument/IR/Dialect.h.
**CN:** 该代码块引入后续声明直接依赖的头文件，例如 mlir/Dialect/Arith/IR/Arith.h, triton/Analysis/BufferRegion.h, triton/Dialect/Triton/IR/Utility.h, triton/Dialect/TritonGPU/IR/Attributes.h, triton/Dialect/TritonInstrument/IR/ConSanConstants.h, and triton/Dialect/TritonInstrument/IR/Dialect.h。

### Lines 12-12
```cpp
  12: #include <array>
```
**EN:** This block imports the direct dependencies needed here, including <array>.
**CN:** 该代码块引入后续声明直接依赖的头文件，例如 <array>。

### Lines 14-16
```cpp
  14: namespace mlir::triton::gpu {
  15: class GlobalScratchAllocOp;
  16: }
```
**EN:** This block stores supporting state such as GlobalScratchAllocOp, which other APIs in the file consume.
**CN:** 该代码块声明了 GlobalScratchAllocOp 等支撑状态，供本文件中的其他 API 使用。

### Lines 18-20
```cpp
  18: namespace mlir::triton::instrument {
  19: class ConSanTargetHooks;
  20: class FunctionBuilder;
```
**EN:** This block stores supporting state such as ConSanTargetHooks and FunctionBuilder, which other APIs in the file consume.
**CN:** 该代码块声明了 ConSanTargetHooks and FunctionBuilder 等支撑状态，供本文件中的其他 API 使用。

### Lines 22-22
```cpp
  22: constexpr int numMemTypes = getMaxEnumValForMemType() + 1;
```
**EN:** This block declares or defines callable APIs such as getMaxEnumValForMemType, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getMaxEnumValForMemType 等可调用 API，用来封装这里提供的核心行为。

### Lines 24-24
```cpp
  24: constexpr int MAX_NUM_BASE_THREADS = 16;
```
**EN:** This block defines named compile-time constants such as int and MAX_NUM_BASE_THREADS.
**CN:** 该代码块定义了 int and MAX_NUM_BASE_THREADS 等具名编译期常量。

### Lines 26-28
```cpp
  26: namespace CommitKind {
  27: enum Kind { None = -1, AsyncCp = 0, Wgmma, TmaStore, NumCommitKinds };
  28: }
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 30-33
```cpp
  30: // -- ConSan capture-count constants -----------------------------------------
  31: // Each constant corresponds to specific passToWarpSpecialize() calls in
  32: // populateAndPassToWarpSpecialize().  Keep in sync with that function and
  33: // with estimateConSanCaptureCount() below.
```
**EN:** This comment block records the intent and constraints of the surrounding code: -- ConSan capture-count constants ----------------------------------------- Each constant corresponds to specific passToWarpSpecialize() calls in populateAndPassToWarpSpecialize....
**CN:** 该注释块解释了周边代码的设计背景、意图或约束条件，帮助读者理解后续实现。

### Lines 35-36
```cpp
  35: // writeVisibility + readVisibility per active memory type.
  36: constexpr int kCapturesPerMemType = 2;
```
**EN:** This block defines named compile-time constants such as writeVisibility, readVisibility, per, active, memory, and type.
**CN:** 该代码块定义了 writeVisibility, readVisibility, per, active, memory, and type 等具名编译期常量。

### Lines 38-39
```cpp
  38: // barrierStates + waiting + activeMasks (only when barriers exist).
  39: constexpr int kBarrierBaseCaptures = 3;
```
**EN:** This block defines named compile-time constants such as barrierStates, waiting, activeMasks, only, when, and barriers.
**CN:** 该代码块定义了 barrierStates, waiting, activeMasks, only, when, and barriers 等具名编译期常量。

### Lines 41-43
```cpp
  41: // writeTracking + readTracking per active memory type (only when barriers
  42: // exist and the memory type has buffers).
  43: constexpr int kBarrierTrackingCapturesPerMemType = 2;
```
**EN:** This block defines named compile-time constants such as writeTracking, readTracking, per, active, memory, and type.
**CN:** 该代码块定义了 writeTracking, readTracking, per, active, memory, and type 等具名编译期常量。

### Lines 45-46
```cpp
  45: // The lock variable (always present).
  46: constexpr int kFixedCaptures = 1;
```
**EN:** This block defines named compile-time constants such as The, lock, variable, always, present, and int.
**CN:** 该代码块定义了 The, lock, variable, always, present, and int 等具名编译期常量。

### Lines 48-49
```cpp
  48: // Size in bytes of each capture (a global-scratch pointer).
  49: constexpr int kCaptureSizeBytes = 8;
```
**EN:** This block defines named compile-time constants such as Size, in, bytes, of, each, and capture.
**CN:** 该代码块定义了 Size, in, bytes, of, each, and capture 等具名编译期常量。

### Lines 51-64
```cpp
  51: /// Estimate the number of WarpSpecialize captures that the
  52: /// ConcurrencySanitizer pass will add via passToWarpSpecialize().
  53: /// \p numActiveMemTypes  Number of memory types with buffers.
  54: /// \p hasBarriers        Whether barriers exist in the module.
  55: /// \p numCommitKinds     Number of distinct commit kinds required.
  56: inline int estimateConSanCaptureCount(int numActiveMemTypes, bool hasBarriers,
  57:                                       int numCommitKinds) {
  58:   int perMemType = kCapturesPerMemType * numActiveMemTypes;
  59:   int barrierCaptures =
  60:       hasBarriers ? kBarrierBaseCaptures +
  61:                         kBarrierTrackingCapturesPerMemType * numActiveMemTypes
  62:                   : 0;
  63:   return perMemType + barrierCaptures + kFixedCaptures + numCommitKinds;
  64: }
```
**EN:** This block declares or defines callable APIs such as estimateConSanCaptureCount, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 estimateConSanCaptureCount 等可调用 API，用来封装这里提供的核心行为。

### Lines 66-87
```cpp
  66: void createAssertInThread(ImplicitLocOpBuilder &b, Value condition,
  67:                           StringRef message);
  68: Operation *createStoreScratchMemory(OpBuilder &b, Location loc, Value alloc,
  69:                                     Value tensor, RankedTensorType tensorType,
  70:                                     bool currentCTAOnly = false);
  71: Value createLoadScratchMemory(OpBuilder &b, Location loc, Value alloc,
  72:                               RankedTensorType tensorType);
  73: gpu::GlobalScratchAllocOp
  74: createThirdPartyScratchAlloc(OpBuilder &b, Location loc, Type ptrType,
  75:                              int64_t sizeInBytes, int64_t alignment,
  76:                              bool sharedClusterState = false);
  77: Value expandOuterSlicedDim(OpBuilder &b, Location loc, Value tensor);
  78: RankedTensorType getIntTensorType(Region *region, ArrayRef<int64_t> shape,
  79:                                   unsigned bitWidth);
  80: TypedValue<RankedTensorType> createConstIntTensor(OpBuilder &builder,
  81:                                                   Location loc, int64_t val,
  82:                                                   RankedTensorType tensorType,
  83:                                                   bool isSigned = false);
  84: uint32_t getMemDescLength(Value buf);
  85: FuncOp getEntryPoint(ModuleOp module);
  86: gpu::DistributedEncodingTrait
  87: getSingleDimSliceEncoding(gpu::DistributedEncodingTrait encoding, int dim);
```
**EN:** This block declares or defines callable APIs such as createAssertInThread, createStoreScratchMemory, createLoadScratchMemory, createThirdPartyScratchAlloc, expandOuterSlicedDim, getIntTensorType, createConstIntTensor, and getMemDescLength, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 createAssertInThread, createStoreScratchMemory, createLoadScratchMemory, createThirdPartyScratchAlloc, expandOuterSlicedDim, getIntTensorType, createConstIntTensor, and getMemDescLength 等可调用 API，用来封装这里提供的核心行为。

### Lines 89-95
```cpp
  89: inline Value maybeAnd(ImplicitLocOpBuilder &b, Value lhs, Value rhs) {
  90:   if (!lhs)
  91:     return rhs;
  92:   if (!rhs)
  93:     return lhs;
  94:   return arith::AndIOp::create(b, lhs, rhs);
  95: }
```
**EN:** This block declares or defines callable APIs such as maybeAnd and create, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 maybeAnd and create 等可调用 API，用来封装这里提供的核心行为。

### Lines 97-99
```cpp
  97: struct ValueType {
  98:   Value value;
  99:   Type type;
```
**EN:** This block introduces `ValueType`, the main class/struct defined here.
**CN:** 该代码块引入了此文件的核心类/结构体 `ValueType`。

### Lines 101-105
```cpp
 101:   ValueType() = default;
 102:   ValueType(Value value, Type type) : value(value), type(type) {}
 103:   ValueType(std::pair<Value, Type> value)
 104:       : value(value.first), type(value.second) {}
 105: };
```
**EN:** This block declares or defines callable APIs such as ValueType, value, and type, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 ValueType, value, and type 等可调用 API，用来封装这里提供的核心行为。

### Lines 107-124
```cpp
 107: // Map from IR region to ConSan auxiliary data.
 108: //
 109: // Aux data is created in the entry function and then either rematerialized or
 110: // captured into warp-specialize partition regions. Each map member below is
 111: // keyed by the IR region that owns the value visible at an instrumentation
 112: // insertion point. For scratch-backed state, ValueType::value is the scratch
 113: // pointer and ValueType::type is the logical tensor type loaded from/stored to
 114: // that pointer. For tensor descriptors and constants, ValueType::value is the
 115: // tensor itself and ValueType::type is its type.
 116: struct AuxDataMap {
 117:   struct ThreadLayout {
 118:     int numBaseThreads = 1;
 119:     int numBaseThreadSlots = 1;
 120:     int tmaThreadOffset = -1;
 121:     int tcThreadOffset = -1;
 122:     int clcThreadOffset = -1;
 123:     int totalNumThreads = 1;
 124:     int numThreadSlots = 1;
```
**EN:** This block introduces `AuxDataMap`, the main class/struct defined here.
**CN:** 该代码块引入了此文件的核心类/结构体 `AuxDataMap`。

### Lines 126-129
```cpp
 126:     bool hasTMAThreads() const { return tmaThreadOffset >= 0; }
 127:     bool hasTCThreads() const { return tcThreadOffset >= 0; }
 128:     bool hasCLCThreads() const { return clcThreadOffset >= 0; }
 129:   };
```
**EN:** This block declares or defines callable APIs such as hasTMAThreads, hasTCThreads, and hasCLCThreads, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 hasTMAThreads, hasTCThreads, and hasCLCThreads 等可调用 API，用来封装这里提供的核心行为。

### Lines 131-143
```cpp
 131:   struct RegionToValueMap {
 132:     DenseMap<Region *, ValueType> values;
 133:     ValueType at(Region *region) {
 134:       if (values.find(region) == values.end()) {
 135:         assert(false && "Region not found in AuxDataMap");
 136:       }
 137:       return values[region];
 138:     }
 139:     ValueType at(Operation *op) {
 140:       return at(getEnclosingParitionOrFunctionRegion(op));
 141:     }
 142:     void insert(Region *region, ValueType value) { values[region] = value; }
 143:     bool empty() const { return values.empty(); }
```
**EN:** This block introduces `RegionToValueMap`, the main class/struct defined here. Within the declaration, methods such as at, find, end, getEnclosingParitionOrFunctionRegion, insert, and empty expose its core API.
**CN:** 该代码块引入了此文件的核心类/结构体 `RegionToValueMap`。 其中 at, find, end, getEnclosingParitionOrFunctionRegion, insert, and empty 等方法构成了它的主要接口。

### Lines 145-147
```cpp
 145:   private:
 146:     Region *getEnclosingParitionOrFunctionRegion(Operation *op);
 147:   };
```
**EN:** This block declares or defines callable APIs such as getEnclosingParitionOrFunctionRegion, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getEnclosingParitionOrFunctionRegion 等可调用 API，用来封装这里提供的核心行为。

### Lines 149-161
```cpp
 149:   // Shape notation:
 150:   //   C = CTAs in the cluster.
 151:   //   Cbar, Cbuf, Cthr, Cmask = CTA dimensions qualifying barriers, buffers,
 152:   //       threads, and thread masks respectively. Each has extent C.
 153:   //   B = tracked buffers for one memory type, power-of-two padded.
 154:   //   K = tracked mbarriers, power-of-two padded.
 155:   //   T = logical ConSan thread bit slots used by this module, power-of-two
 156:   //       padded for the distributed layout.
 157:   //   P = base-thread commit columns used by this module, power-of-two padded.
 158:   //
 159:   // Storage notation:
 160:   //   tensor  = distributed tensor value.
 161:   //   scratch = pointer to shared-cluster global scratch memory.
```
**EN:** This comment block records the intent and constraints of the surrounding code: Shape notation: C = CTAs in the cluster. Cbar, Cbuf, Cthr, Cmask = CTA dimensions qualifying barriers, buffers, threads, and thread masks respectively. Each has extent C. B = tr....
**CN:** 该注释块解释了周边代码的设计背景、意图或约束条件，帮助读者理解后续实现。

### Lines 163-166
```cpp
 163:   // tensor, <B x i64>
 164:   // Per-memory-type packed buffer descriptors. Each i64 stores the 32-bit base
 165:   // offset and 32-bit length of one shared-memory or tensor-memory region.
 166:   RegionToValueMap buffers[numMemTypes];
```
**EN:** This block stores supporting state such as buffers, which other APIs in the file consume.
**CN:** 该代码块声明了 buffers 等支撑状态，供本文件中的其他 API 使用。

### Lines 168-171
```cpp
 168:   // tensor, <K x i64>
 169:   // Packed descriptors for tracked mbarrier allocations. Barriers are shared
 170:   // memory descriptors.
 171:   RegionToValueMap barriers;
```
**EN:** This block stores supporting state such as barriers, which other APIs in the file consume.
**CN:** 该代码块声明了 barriers 等支撑状态，供本文件中的其他 API 使用。

### Lines 173-177
```cpp
 173:   // scratch, <Cbar x K x i64>
 174:   // Packed barrier lifecycle state. Zero means invalid/uninitialized. Bit 0 is
 175:   // phase, bits [1..20] are the initial arrival count, bits [21..40] are the
 176:   // current arrival count, and bits [41..61] hold a signed tx-count.
 177:   RegionToValueMap barrierStates;
```
**EN:** This block stores supporting state such as barrierStates, which other APIs in the file consume.
**CN:** 该代码块声明了 barrierStates 等支撑状态，供本文件中的其他 API 使用。

### Lines 179-182
```cpp
 179:   // scratch, <Cbuf x B x Cmask x i64>
 180:   // Per-memory-type write frontier. Bit i means logical ConSan thread i can see
 181:   // the latest write to the buffer row.
 182:   RegionToValueMap writeVisibility[numMemTypes];
```
**EN:** This block stores supporting state such as writeVisibility, which other APIs in the file consume.
**CN:** 该代码块声明了 writeVisibility 等支撑状态，供本文件中的其他 API 使用。

### Lines 184-186
```cpp
 184:   // scratch, <Cbuf x B x Cbar x K x i8>
 185:   // Per-memory-type buffer/barrier map for writes that a barrier tracks.
 186:   RegionToValueMap writeTracking[numMemTypes];
```
**EN:** This block stores supporting state such as writeTracking, which other APIs in the file consume.
**CN:** 该代码块声明了 writeTracking 等支撑状态，供本文件中的其他 API 使用。

### Lines 188-191
```cpp
 188:   // scratch, <Cbuf x B x Cthr x T x Cmask x i64>
 189:   // Per-memory-type read frontier. For each buffer and logical thread lane, the
 190:   // i64 value is a bitmask of reads visible to that lane's thread.
 191:   RegionToValueMap readVisibility[numMemTypes];
```
**EN:** This block stores supporting state such as readVisibility, which other APIs in the file consume.
**CN:** 该代码块声明了 readVisibility 等支撑状态，供本文件中的其他 API 使用。

### Lines 193-196
```cpp
 193:   // scratch, <Cbuf x B x Cbar x K x Cmask x i64>
 194:   // Per-memory-type buffer/barrier map for read visibility masks that a barrier
 195:   // tracks.
 196:   RegionToValueMap readTracking[numMemTypes];
```
**EN:** This block stores supporting state such as readTracking, which other APIs in the file consume.
**CN:** 该代码块声明了 readTracking 等支撑状态，供本文件中的其他 API 使用。

### Lines 198-204
```cpp
 198:   // scratch, <C x B x P x i8>
 199:   // Per-commit-kind outstanding commit counters for shared-memory buffers.
 200:   // Entries are 0 for none, -1 for staged but uncommitted, and positive for a
 201:   // committed access with an outstanding-group distance.
 202:   // Just one C dimension as ampere async_copy, WGMMA and TMA store are
 203:   // intra-CTA.
 204:   RegionToValueMap commits[CommitKind::NumCommitKinds];
```
**EN:** This block stores supporting state such as commits, which other APIs in the file consume.
**CN:** 该代码块声明了 commits 等支撑状态，供本文件中的其他 API 使用。

### Lines 206-210
```cpp
 206:   // tensor, <B x B x i1>
 207:   // Optional per-memory-type alias matrix. Created only when BufferRegion
 208:   // analysis finds cross-buffer aliasing; checks expand selected buffer rows
 209:   // through this matrix.
 210:   RegionToValueMap aliasMatrices[numMemTypes];
```
**EN:** This block stores supporting state such as aliasMatrices, which other APIs in the file consume.
**CN:** 该代码块声明了 aliasMatrices 等支撑状态，供本文件中的其他 API 使用。

### Lines 212-214
```cpp
 212:   // scratch pointer, i32
 213:   // Shared-cluster lock used to serialize ConSan instrumentation updates.
 214:   RegionToValueMap lock;
```
**EN:** This block stores supporting state such as lock, which other APIs in the file consume.
**CN:** 该代码块声明了 lock 等支撑状态，供本文件中的其他 API 使用。

### Lines 216-219
```cpp
 216:   // Consan inserts internal cluster barriers for its own protocols. They must
 217:   // keep their synchronization semantics, but they are not user-visible
 218:   // publication points.
 219:   SmallVector<Operation *> nonPublishingClusterBarriers;
```
**EN:** This block stores supporting state such as nonPublishingClusterBarriers, which other APIs in the file consume.
**CN:** 该代码块声明了 nonPublishingClusterBarriers 等支撑状态，供本文件中的其他 API 使用。

### Lines 221-224
```cpp
 221:   // scratch, <Cbar x K x Cthr x i32>
 222:   // Deadlock-detection bitfield. Each base thread uses two bits: waiting flag
 223:   // and stored phase.
 224:   RegionToValueMap waiting;
```
**EN:** This block stores supporting state such as waiting, which other APIs in the file consume.
**CN:** 该代码块声明了 waiting 等支撑状态，供本文件中的其他 API 使用。

### Lines 226-230
```cpp
 226:   // scratch, <C x i32>
 227:   // Deadlock-detection bitfield. Outside warp specialization this is 1; inside
 228:   // it, set bits denote base threads that have not yet reached their
 229:   // terminator.
 230:   RegionToValueMap activeMasks;
```
**EN:** This block stores supporting state such as activeMasks, which other APIs in the file consume.
**CN:** 该代码块声明了 activeMasks 等支撑状态，供本文件中的其他 API 使用。

### Lines 232-234
```cpp
 232:   // True when a memory type has cross-buffer aliasing and therefore requires
 233:   // aliasMatrices to make visibility and commit checks conservative.
 234:   std::array<bool, numMemTypes> hasNonTrivialAliasing{};
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 236-238
```cpp
 236:   // Dense logical-thread numbering for this module. Base threads are always
 237:   // present; TMA/TC/CLC peer ranges are added only when the module uses them.
 238:   ThreadLayout threadLayout;
```
**EN:** This block stores supporting state such as threadLayout, which other APIs in the file consume.
**CN:** 该代码块声明了 threadLayout 等支撑状态，供本文件中的其他 API 使用。

### Lines 240-242
```cpp
 240:   LogicalResult populateAndPassToWarpSpecialize(ModuleOp module,
 241:                                                 FunctionBuilder &funcBuilder,
 242:                                                 const ConSanTargetHooks *hooks);
```
**EN:** This block declares or defines callable APIs such as populateAndPassToWarpSpecialize, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 populateAndPassToWarpSpecialize 等可调用 API，用来封装这里提供的核心行为。

### Lines 244-255
```cpp
 244: private:
 245:   void getBuffersAndBarriers(
 246:       ModuleOp module,
 247:       SmallVector<SmallVector<triton::BufferRegion>, 2> &bufRegions,
 248:       SmallVector<triton::BufferRegion> &barrierRegions);
 249:   void passToWarpSpecialize(triton::FuncOp func, ValueType value,
 250:                             RegionToValueMap &map, int &captureCounter,
 251:                             int64_t &captureBytes);
 252:   void createInWarpSpecialize(
 253:       triton::FuncOp func, RegionToValueMap &map,
 254:       std::function<ValueType(ImplicitLocOpBuilder &)> createFn);
 255: };
```
**EN:** This block declares or defines callable APIs such as getBuffersAndBarriers, passToWarpSpecialize, createInWarpSpecialize, and ValueType, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getBuffersAndBarriers, passToWarpSpecialize, createInWarpSpecialize, and ValueType 等可调用 API，用来封装这里提供的核心行为。

### Lines 257-257
```cpp
 257: } // namespace mlir::triton::instrument
```
**EN:** This block manages the namespace scope for the surrounding declarations.
**CN:** 该代码块负责管理周围声明所属的命名空间作用域。

### Lines 259-259
```cpp
 259: #endif // TRITONINSTRUMENT_UTILITY_H
```
**EN:** This block closes the header guard started at the top of the file.
**CN:** 该代码块结束文件顶部开始的头文件保护宏。

## Key Concepts / 关键概念
- **EN:** buffer modeling  
  **CN:** 缓冲区建模
- **EN:** layout conversion and reasoning  
  **CN:** 布局转换与推理
- **EN:** layout encodings  
  **CN:** 布局编码
- **EN:** warp-level execution  
  **CN:** warp 级执行
- **EN:** LLVM lowering  
  **CN:** LLVM 降级
- **EN:** compiler passes  
  **CN:** 编译 pass
- **EN:** MLIR dialect definition  
  **CN:** MLIR 方言定义
- **EN:** operation definitions  
  **CN:** 操作定义

## Dependencies / 依赖关系
- **Project includes / 项目内依赖:**
  - `mlir/Dialect/Arith/IR/Arith.h`
  - `triton/Analysis/BufferRegion.h`
  - `triton/Dialect/Triton/IR/Utility.h`
  - `triton/Dialect/TritonGPU/IR/Attributes.h`
  - `triton/Dialect/TritonInstrument/IR/ConSanConstants.h`
  - `triton/Dialect/TritonInstrument/IR/Dialect.h`
  - `llvm/Support/MathExtras.h`
- **System or external includes / 系统或外部依赖:**
  - `<array>`
