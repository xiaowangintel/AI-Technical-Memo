# InterleaveTMem.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `lib/Dialect/TritonNvidiaGPU/Transforms/InterleaveTMem.cpp`
- **Purpose / 作用:** **EN:** Implements the Interleave T Mem transformation or optimization pass for the TritonNvidiaGPU pipeline. **CN:** 为 TritonNvidiaGPU 编译流程实现与 Interleave T Mem 相关的变换或优化 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-5

```cpp
   1: #include "mlir/Transforms/GreedyPatternRewriteDriver.h"
   2: #include "triton/Dialect/TritonGPU/Transforms/Utility.h"
   3: #include "triton/Dialect/TritonNvidiaGPU/IR/Dialect.h"
   4: #include "triton/Dialect/TritonNvidiaGPU/Transforms/Passes.h"
   5: #include "llvm/ADT/AddressRanges.h"
```

- **EN:** Includes the interfaces this file depends on. Triton headers (`Utility.h`, `Dialect.h`, `Passes.h`) provide domain-specific IR/support, MLIR headers (`GreedyPatternRewriteDriver.h`) provide rewriting and analysis infrastructure, LLVM headers (`AddressRanges.h`) supply low-level utilities, and standard/library headers (None) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（`Utility.h`, `Dialect.h`, `Passes.h`）提供领域特定 IR/支持逻辑，MLIR 头文件（`GreedyPatternRewriteDriver.h`）提供重写与分析基础设施，LLVM 头文件（`AddressRanges.h`）提供底层工具，而标准/通用库头文件（None）提供通用能力。
### Lines 7-7

```cpp
   7: namespace ttg = mlir::triton::gpu;
```

- **EN:** Opens or closes the namespace nesting for ttg, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 ttg 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 9-11

```cpp
   9: namespace mlir {
  10: namespace triton {
  11: namespace nvidia_gpu {
```

- **EN:** Opens or closes the namespace nesting for mlir -> triton -> nvidia_gpu, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 mlir -> triton -> nvidia_gpu 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 13-14

```cpp
  13: #define GEN_PASS_DEF_TRITONNVIDIAGPUINTERLEAVETMEMPASS
  14: #include "triton/Dialect/TritonNvidiaGPU/Transforms/Passes.h.inc"
```

- **EN:** Pulls in generated definitions from TableGen/MLIR include fragments so the handwritten code can reuse auto-generated declarations.
- **CN:** 这里引入由 TableGen/MLIR 生成的定义片段，使手写代码能够复用自动生成的声明。
### Lines 16-16

```cpp
  16: namespace {
```

- **EN:** Opens or closes the namespace nesting for (anonymous), keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 (anonymous) 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 18-25

```cpp
  18: // If we don't know the effects of the op, we add all possible effects.
  19: void addAllValuelessEffects(
  20:     SmallVectorImpl<MemoryEffects::EffectInstance> &effects) {
  21:   effects.emplace_back(MemoryEffects::Effect::get<MemoryEffects::Read>());
  22:   effects.emplace_back(MemoryEffects::Effect::get<MemoryEffects::Write>());
  23:   effects.emplace_back(MemoryEffects::Effect::get<MemoryEffects::Allocate>());
  24:   effects.emplace_back(MemoryEffects::Effect::get<MemoryEffects::Free>());
  25: }
```

- **EN:** Defines `addAllValuelessEffects`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `addAllValuelessEffects`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 27-44

```cpp
  27: bool collectEffects(Operation *op,
  28:                     SmallVectorImpl<MemoryEffects::EffectInstance> &effects) {
  29:   // Collect effect instances the operation. Note that the implementation of
  30:   // getEffects erases all effect instances that have the type other than the
  31:   // template parameter so we collect them first in a local buffer and then
  32:   // copy.
  33:   if (auto iface = dyn_cast<MemoryEffectOpInterface>(op)) {
  34:     SmallVector<MemoryEffects::EffectInstance> localEffects;
  35:     iface.getEffects(localEffects);
  36:     llvm::append_range(effects, localEffects);
  37:     return true;
  38:   }
  39:   if (op->hasTrait<OpTrait::HasRecursiveMemoryEffects>()) {
  40:     for (auto &region : op->getRegions()) {
  41:       for (auto &block : region) {
  42:         for (auto &innerOp : block)
  43:           if (!collectEffects(&innerOp, effects))
  44:             return false;
```

- **EN:** Defines helper `collectEffects` that computes or constructs intermediate data used by the surrounding transformation. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义辅助函数 `collectEffects`，用于计算或构造外围变换所需的中间数据。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 45-48

```cpp
  45:       }
  46:     }
  47:     return true;
  48:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 50-54

```cpp
  50:   // We need to be conservative here in case the op doesn't have the interface
  51:   // and assume it can have any possible effect.
  52:   addAllValuelessEffects(effects);
  53:   return false;
  54: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 56-59

```cpp
  56: struct AccessRange {
  57:   SmallVector<std::optional<llvm::AddressRange>> ranges;
  58:   unsigned rankOffset = 0;
  59: };
```

- **EN:** Defines `AccessRange`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `AccessRange`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 61-61

```cpp
  61: std::pair<Value, AccessRange> findBufferAccess(Value a);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 63-79

```cpp
  63: std::pair<Value, AccessRange>
  64: findBufferAccessMemdescSubview(Operation *subview) {
  65:   OpBuilder builder(subview);
  66:   Location loc = subview->getLoc();
  67:   TypedValue<ttg::MemDescType> src;
  68:   SmallVector<int64_t> shape;
  69:   SmallVector<Value> offsets;
  70:   if (auto indexOp = dyn_cast<ttg::MemDescIndexOp>(subview)) {
  71:     src = indexOp.getSrc();
  72:     shape = to_vector(indexOp.getType().getShape());
  73:     offsets = {indexOp.getIndex()};
  74:     for (int i = 0, e = std::max<int>(0, shape.size() - 1); i < e; ++i)
  75:       offsets.push_back(arith::ConstantIntOp::create(builder, loc, 0, 32));
  76:   } else {
  77:     auto subsliceOp = cast<ttg::MemDescSubsliceOp>(subview);
  78:     src = subsliceOp.getSrc();
  79:     shape = to_vector(subsliceOp.getType().getShape());
```

- **EN:** Defines `findBufferAccessMemdescSubview`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `findBufferAccessMemdescSubview`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 80-90

```cpp
  80:     for (auto offset : subsliceOp.getOffsets())
  81:       offsets.push_back(arith::ConstantIntOp::create(builder, loc, offset, 32));
  82:   }
  83:   auto [alloc, parentAccess] = findBufferAccess(src);
  84:   if (!alloc)
  85:     return {};
  86:   // Handle subview of a subview. The first `rankOffset` access sizes are
  87:   // the same as in the parent access.
  88:   AccessRange childAccess;
  89:   for (auto i : llvm::seq(parentAccess.rankOffset))
  90:     childAccess.ranges.push_back(parentAccess.ranges[i]);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 92-100

```cpp
  92:   // The subview may have a smaller rank, in which case its access size is
  93:   // just 1 for the higher dims.
  94:   childAccess.rankOffset = src.getType().getRank() - shape.size();
  95:   for (auto [i, offset] : llvm::enumerate(offsets)) {
  96:     auto parentRange = parentAccess.ranges[i + parentAccess.rankOffset];
  97:     if (!parentRange) {
  98:       childAccess.ranges.push_back({});
  99:       continue;
 100:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 102-107

```cpp
 102:     // If the offset is not known, then the entire dim may be accessed.
 103:     APInt value;
 104:     if (!matchPattern(offset, m_ConstantInt(&value))) {
 105:       childAccess.ranges.push_back({});
 106:       continue;
 107:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 109-116

```cpp
 109:     uint64_t accessStart = parentRange->start() + value.getSExtValue();
 110:     uint64_t accessSize = 1;
 111:     if (i >= childAccess.rankOffset)
 112:       accessSize = shape[i - childAccess.rankOffset];
 113:     childAccess.ranges.push_back({{accessStart, accessStart + accessSize}});
 114:   }
 115:   return {alloc, std::move(childAccess)};
 116: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 118-123

```cpp
 118: // Simple local alias analysis that looks for a single underlying allocation and
 119: // an access subrange.
 120: std::pair<Value, AccessRange> findBufferAccess(Value a) {
 121:   // Handle block arguments.
 122:   if (auto arg = dyn_cast<BlockArgument>(a)) {
 123:     Operation *parentOp = arg.getOwner()->getParentOp();
```

- **EN:** Defines `findBufferAccess`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `findBufferAccess`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 125-128

```cpp
 125:     // Look through `ttg.warp_specialize` explicit captures.
 126:     if (auto wsOp = dyn_cast<ttg::WarpSpecializePartitionsOp>(parentOp)) {
 127:       return findBufferAccess(wsOp.getExplicitCaptures()[arg.getArgNumber()]);
 128:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 130-132

```cpp
 130:     // Unknown block argument.
 131:     return {};
 132:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 134-141

```cpp
 134:   Operation *defOp = a.getDefiningOp();
 135:   // Accessing the alloc accesses the whole buffer.
 136:   if (auto alloc = dyn_cast<TMEMAllocOp>(defOp)) {
 137:     AccessRange access;
 138:     for (uint64_t dim : alloc.getType().getShape())
 139:       access.ranges.push_back({{0, dim}});
 140:     return {a, std::move(access)};
 141:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 143-146

```cpp
 143:   // Trans and Reshape views don't change the access size.
 144:   if (isa<ttg::MemDescTransOp, ttg::MemDescReshapeOp>(defOp)) {
 145:     return findBufferAccess(defOp->getOperand(0));
 146:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 148-151

```cpp
 148:   // Subviews can reduce the access sizes.
 149:   if (isa<ttg::MemDescIndexOp, ttg::MemDescSubsliceOp>(defOp)) {
 150:     return findBufferAccessMemdescSubview(defOp);
 151:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 153-165

```cpp
 153:   // Subslice is a subview only on the N dimension.
 154:   if (auto subslice = dyn_cast<TMEMSubSliceOp>(defOp)) {
 155:     auto [alloc, parentAccess] = findBufferAccess(subslice.getSrc());
 156:     if (!alloc)
 157:       return {};
 158:     if (!parentAccess.ranges[1])
 159:       return {alloc, parentAccess};
 160:     uint64_t mStart = parentAccess.ranges[1]->start() + subslice.getN();
 161:     uint64_t mSize = subslice.getType().getShape()[1];
 162:     AccessRange childAccess = parentAccess;
 163:     childAccess.ranges[1] = {{mStart, mStart + mSize}};
 164:     return {alloc, std::move(childAccess)};
 165:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 167-169

```cpp
 167:   // Unknown defining op.
 168:   return {};
 169: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 171-187

```cpp
 171: bool tmemMayAlias(Value a, Value b) {
 172:   auto [aAlloc, aRanges] = findBufferAccess(a);
 173:   auto [bAlloc, bRanges] = findBufferAccess(b);
 174:   // If the underlying buffer was not identified, assume mayalias.
 175:   if (!aAlloc || !bAlloc)
 176:     return true;
 177:   // If the buffers are different, they don't alias.
 178:   if (aAlloc != bAlloc)
 179:     return false;
 180:   // If the access ranges along any dimension are known to not overlap, then the
 181:   // accesses don't alias.
 182:   for (auto [aRange, bRange] : llvm::zip(aRanges.ranges, bRanges.ranges)) {
 183:     // If either access range at this dim is unknown, we can't determine if they
 184:     // don't overlap.
 185:     if (!aRange || !bRange)
 186:       continue;
 187:     // The access ranges are known and don't overlap.
```

- **EN:** Defines `tmemMayAlias`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `tmemMayAlias`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 188-192

```cpp
 188:     if (!aRange->intersects(*bRange))
 189:       return false;
 190:   }
 191:   return true;
 192: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 194-211

```cpp
 194: // Sink tmem_loads as close to their use as possible to reduce register
 195: // pressure.
 196: bool sinkOps(Value buffer, ArrayRef<Operation *> useChain) {
 197:   Operation *insertBefore = nullptr;
 198:   Operation *next = useChain.back()->getNextNode();
 199:   while (next && !next->hasTrait<OpTrait::IsTerminator>()) {
 200:     insertBefore = next;
 201:     bool dep = false;
 202:     for (auto operand : getNestedOperands(next)) {
 203:       if (llvm::any_of(useChain, [&](Operation *op) {
 204:             return llvm::is_contained(op->getResults(), operand);
 205:           })) {
 206:         dep = true;
 207:         break;
 208:       }
 209:     }
 210:     // Don't sink past barrier signals, since they may guard the liverange
 211:     // of the buffer.
```

- **EN:** Defines `sinkOps`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `sinkOps`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 212-229

```cpp
 212:     if (isa<ArriveBarrierOp>(next))
 213:       break;
 214:     if (!isMemoryEffectFree(next)) {
 215:       SmallVector<MemoryEffects::EffectInstance> effects;
 216:       collectEffects(next, effects);
 217:       for (auto effect : effects) {
 218:         // Look for potentially aliasing write or free effects.
 219:         if (!isa<MemoryEffects::Write, MemoryEffects::Free>(effect.getEffect()))
 220:           continue;
 221:         if (isa<SideEffects::DefaultResource>(effect.getResource())) {
 222:           dep = true;
 223:           break;
 224:         }
 225:         if (isa<TensorMemory>(effect.getResource()) &&
 226:             (!effect.getValue() || tmemMayAlias(effect.getValue(), buffer))) {
 227:           dep = true;
 228:           break;
 229:         }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 230-242

```cpp
 230:       }
 231:     }
 232:     if (dep)
 233:       break;
 234:     next = next->getNextNode();
 235:   }
 236:   if (insertBefore && insertBefore != useChain.back()->getNextNode()) {
 237:     for (Operation *op : useChain)
 238:       op->moveBefore(insertBefore);
 239:     return true;
 240:   }
 241:   return false;
 242: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 244-253

```cpp
 244: // Try to sink a load and a collection of its users.
 245: bool trySinkOp(Operation *op, Value buffer) {
 246:   SmallVector<Operation *> useChain{op};
 247:   while (useChain.back()->hasOneUse() &&
 248:          isPure(*useChain.back()->user_begin()) &&
 249:          useChain.back()->getNextNode() == *useChain.back()->user_begin()) {
 250:     useChain.push_back(*useChain.back()->user_begin());
 251:   }
 252:   return sinkOps(buffer, useChain);
 253: }
```

- **EN:** Defines `trySinkOp`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `trySinkOp`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 255-255

```cpp
 255: } // anonymous namespace
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 257-261

```cpp
 257: struct TritonNvidiaGPUInterleaveTMemPass
 258:     : public impl::TritonNvidiaGPUInterleaveTMemPassBase<
 259:           TritonNvidiaGPUInterleaveTMemPass> {
 260:   using impl::TritonNvidiaGPUInterleaveTMemPassBase<
 261:       TritonNvidiaGPUInterleaveTMemPass>::TritonNvidiaGPUInterleaveTMemPassBase;
```

- **EN:** Defines `TritonNvidiaGPUInterleaveTMemPass`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `TritonNvidiaGPUInterleaveTMemPass`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 263-278

```cpp
 263:   void runOnOperation() override {
 264:     ModuleOp m = getOperation();
 265:     SmallVector<std::pair<Operation *, Value>> opsToSink;
 266:     m.walk([&](Operation *op) {
 267:       if (auto load = dyn_cast<TMEMLoadOp>(op))
 268:         opsToSink.emplace_back(load, load.getSrc());
 269:       else if (auto alloc = dyn_cast<TMEMAllocOp>(op))
 270:         opsToSink.emplace_back(alloc, alloc.getResult());
 271:     });
 272:     for (auto [op, buffer] : opsToSink) {
 273:       while (trySinkOp(op, buffer)) {
 274:         // Keep trying to sink loads and their users.
 275:       }
 276:     }
 277:   }
 278: };
```

- **EN:** Defines `runOnOperation`, the pass entry point. It gathers analysis information and applies the file's transformation logic to the current operation. The implementation traverses IR operations to collect facts or apply rewrites globally. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `runOnOperation`，即 pass 的入口函数。它会收集分析信息，并把本文件的变换逻辑应用到当前操作上。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 280-282

```cpp
 280: } // namespace nvidia_gpu
 281: } // namespace triton
 282: } // namespace mlir
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。

## Key Concepts / 关键概念
- **EN:** The main concern is pass-driven transformation around interleave t mem.
  **CN:** 核心关注点是围绕 Interleave T Mem 的 pass 驱动变换。
- **EN:** Memory descriptor types carry layout, rank, and address-space information.
  **CN:** 内存描述符类型携带布局、维度和地址空间信息。
- **EN:** The code reasons at module scope rather than only on isolated operations.
  **CN:** 代码在模块范围内进行分析，而不只是处理单个操作。
- **EN:** Tensor memory specific allocation, layout, or synchronization rules are important here.
  **CN:** 这里重点处理张量内存（TMEM）的分配、布局或同步规则。
- **EN:** Synchronization and ordering constraints matter to preserve correctness across threads, warps, or memory spaces.
  **CN:** 同步与顺序约束很重要，它们保证跨线程、warp 或内存空间的正确性。

## Dependencies / 依赖关系
- **Internal Triton headers / Triton 内部头文件:** `triton/Dialect/TritonGPU/Transforms/Utility.h`, `triton/Dialect/TritonNvidiaGPU/IR/Dialect.h`, `triton/Dialect/TritonNvidiaGPU/Transforms/Passes.h`, `triton/Dialect/TritonNvidiaGPU/Transforms/Passes.h.inc`
- **MLIR headers / MLIR 头文件:** `mlir/Transforms/GreedyPatternRewriteDriver.h`
- **LLVM headers / LLVM 头文件:** `llvm/ADT/AddressRanges.h`
- **Standard/library headers / 标准或通用库头文件:** None
- **Generated fragments / 生成片段:** `triton/Dialect/TritonNvidiaGPU/Transforms/Passes.h.inc`
- **Primary APIs used / 主要 API:** `ModuleOp`, `MemDescType`, `OpBuilder`
- **Pipeline role / 流程角色:** This file participates in Triton pass pipelines and usually expects upstream analyses or dialect invariants to have prepared the IR. / 本文件参与 Triton 的 pass 流程，通常依赖上游分析或方言不变量先把 IR 准备好。
