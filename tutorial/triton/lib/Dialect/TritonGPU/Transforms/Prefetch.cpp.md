# Prefetch.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `lib/Dialect/TritonGPU/Transforms/Prefetch.cpp`
- **Purpose / 作用:** **EN:** Implements the Prefetch transformation or optimization pass for the TritonGPU pipeline. **CN:** 为 TritonGPU 编译流程实现与 Prefetch 相关的变换或优化 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```cpp
   1: //===----------------------------------------------------------------------===//
   2: //
   3: // This pass rewrites selected tt.dot loops to pull the prefetched head out of
   4: // the loop and to prefetch the next iteration's operands at the end of the
   5: // loop. The concrete shape is covered by split_pipelined_mmav2_loads in
   6: // test/TritonGPU/prefetch.mlir.
   7: //
   8: // Example:
   9: // %loop = scf.for ... {
  10: //   %wait = ttg.async_wait %tok0, %tok1 {num = 4 : i32}
  11: //   %a_view = ttg.memdesc_index %a[%idx_next]
  12: //   %a_val = ttg.local_load %a_view token %wait
  13: //   %b_view = ttg.memdesc_index %b[%idx_next]
  14: //   %b_val = ttg.local_load %b_view token %wait
  15: //   %acc_next = tt.dot %a_val, %b_val, %acc
  16: //   scf.yield %idx_next, %acc_next
  17: // }
  18: //
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 19-36

```cpp
  19: // becomes:
  20: // %a_view0 = ttg.memdesc_index %a[%idx_next0]
  21: // %b_view0 = ttg.memdesc_index %b[%idx_next0]
  22: // %wait0 = ttg.async_wait %tok0, %tok1 {num = 4 : i32}
  23: // %a0 = ttg.local_load %a_view0 token %wait0
  24: // %b0 = ttg.local_load %b_view0 token %wait0
  25: // %loop = scf.for ... iter_args(..., %wait = %wait0, %a_prefetch = %a0,
  26: //                               %b_prefetch = %b0) {
  27: //   %wait_next = ttg.async_wait %tok0, %tok1 {num = 4 : i32}
  28: //   %a_rem = ttg.local_load %a_tail token %wait
  29: //   %b_rem = ttg.local_load %b_tail token %wait
  30: //   %dot0 = tt.dot %a_prefetch, %b_prefetch, %acc
  31: //   %a_next = ttg.local_load %next_a_head token %wait_next
  32: //   %b_next = ttg.local_load %next_b_head token %wait_next
  33: //   %acc_next = tt.dot %a_rem, %b_rem, %dot0
  34: //   scf.yield ..., %wait_next, %a_next, %b_next
  35: // }
  36: //===----------------------------------------------------------------------===//
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 38-46

```cpp
  38: #include "mlir/Dialect/Arith/IR/Arith.h"
  39: #include "mlir/IR/IRMapping.h"
  40: #include "mlir/Support/LLVM.h"
  41: #include "mlir/Transforms/GreedyPatternRewriteDriver.h"
  42: #include "triton/Dialect/TritonGPU/IR/Dialect.h"
  43: #include "triton/Dialect/TritonGPU/Transforms/Passes.h"
  44: #include "triton/Dialect/TritonGPU/Transforms/Utility.h"
  45: #include "triton/Dialect/TritonNvidiaGPU/IR/Dialect.h"
  46: #include "llvm/Support/Debug.h"
```

- **EN:** Includes the interfaces this file depends on. Triton headers (`Dialect.h`, `Passes.h`, `Utility.h`, `Dialect.h`) provide domain-specific IR/support, MLIR headers (`Arith.h`, `IRMapping.h`, `LLVM.h`, `GreedyPatternRewriteDriver.h`) provide rewriting and analysis infrastructure, LLVM headers (`Debug.h`) supply low-level utilities, and standard/library headers (None) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（`Dialect.h`, `Passes.h`, `Utility.h`, `Dialect.h`）提供领域特定 IR/支持逻辑，MLIR 头文件（`Arith.h`, `IRMapping.h`, `LLVM.h`, `GreedyPatternRewriteDriver.h`）提供重写与分析基础设施，LLVM 头文件（`Debug.h`）提供底层工具，而标准/通用库头文件（None）提供通用能力。
### Lines 48-50

```cpp
  48: #define DEBUG_TYPE "tritongpu-prefetch"
  49: #define DBGS() (llvm::dbgs() << "[" DEBUG_TYPE "]: ")
  50: #define LDBG(X) LLVM_DEBUG(DBGS() << X << "\n")
```

- **EN:** Defines debug logging helpers and compile-time tags used when tracing this pass or utility at runtime.
- **CN:** 这里定义调试日志辅助宏和编译期标签，用于在运行时跟踪该 pass 或工具。
### Lines 52-54

```cpp
  52: namespace mlir {
  53: namespace triton {
  54: namespace gpu {
```

- **EN:** Opens or closes the namespace nesting for mlir -> triton -> gpu, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 mlir -> triton -> gpu 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 56-57

```cpp
  56: #define GEN_PASS_DEF_TRITONGPUPREFETCH
  57: #include "triton/Dialect/TritonGPU/Transforms/Passes.h.inc"
```

- **EN:** Pulls in generated definitions from TableGen/MLIR include fragments so the handwritten code can reuse auto-generated declarations.
- **CN:** 这里引入由 TableGen/MLIR 生成的定义片段，使手写代码能够复用自动生成的声明。
### Lines 59-59

```cpp
  59: namespace {
```

- **EN:** Opens or closes the namespace nesting for (anonymous), keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 (anonymous) 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 61-67

```cpp
  61: class Prefetcher {
  62:   struct CarriedArgs {
  63:     DenseMap<Operation *, unsigned> aSource;
  64:     DenseMap<Operation *, unsigned> bSource;
  65:     DenseMap<Operation *, unsigned> a;
  66:     DenseMap<Operation *, unsigned> b;
  67:   };
```

- **EN:** Defines `Prefetcher`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `Prefetcher`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 69-74

```cpp
  69:   /// Loop being rewritten.
  70:   scf::ForOp forOp;
  71:   /// Original loop terminator, used to recover yielded values.
  72:   scf::YieldOp yieldOp;
  73:   unsigned prefetchWidth = 32;
  74:   int computeCapability;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 76-86

```cpp
  76:   /// Dots that will be rewritten to use prologue/next-iteration prefetches.
  77:   SetVector<triton::DotOp> dots;
  78:   DenseMap<Value, Value> dot2aSource;
  79:   DenseMap<Value, Value> dot2bSource;
  80:   DenseMap<Value, Value> dot2aToken;
  81:   DenseMap<Value, Value> dot2bToken;
  82:   DenseMap<Value, SmallVector<Value>> dot2aVals;
  83:   DenseMap<Value, SmallVector<Value>> dot2bVals;
  84:   /// Original dot operand -> prologue-prefetched value.
  85:   DenseMap<Value, Value> operand2headPrefetch;
  86:   DenseMap<Value, Value> initMaterializations;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 88-96

```cpp
  88:   Value generatePrefetch(Value v, unsigned opIdx, bool isPrologue,
  89:                          Attribute dotEncoding, OpBuilder &builder,
  90:                          Value token = Value(),
  91:                          std::optional<int64_t> offsetK = std::nullopt,
  92:                          std::optional<int64_t> shapeK = std::nullopt);
  93:   unsigned getKWidthScale(Attribute dotEncoding, Type elementType) const;
  94:   unsigned getDotOperandKWidth(Attribute dotEncoding, Type elementType) const;
  95:   unsigned getPrefetchWidth(Attribute dotEncoding, Type elementType,
  96:                             unsigned kWidth) const;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 98-115

```cpp
  98:   bool isLoopCarriedValue(Value v);
  99:   Value getIncomingValue(Value v);
 100:   Value getYieldValue(Value v);
 101:   bool isPromotableValue(Value v);
 102:   Value cloneLoopValue(Value v, OpBuilder &builder,
 103:                        llvm::function_ref<Value(BlockArgument)> mapBlockArg,
 104:                        DenseMap<Value, Value> &cache);
 105:   Value materializeInitValue(Value v, OpBuilder &builder,
 106:                              DenseMap<Value, Value> &cache);
 107:   void appendMaterializedLoopArgIfNeeded(
 108:       triton::DotOp dot, Value value, DenseMap<Operation *, unsigned> &argMap,
 109:       SmallVector<Value> &loopArgs, OpBuilder &builder);
 110:   Value getTrackedValue(triton::DotOp dot, bool isA, bool isToken);
 111:   const DenseMap<Operation *, unsigned> &
 112:   getCarriedArgMap(const CarriedArgs &carriedArgs, bool isA, bool isToken);
 113:   Value getCurrentTrackedValue(triton::DotOp dot, bool isA, bool isToken,
 114:                                scf::ForOp newForOp, IRMapping &mapping,
 115:                                const CarriedArgs &carriedArgs);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 116-123

```cpp
 116:   Value getNextTrackedValue(triton::DotOp dot, bool isA, bool isToken,
 117:                             OpBuilder &builder, IRMapping &mapping);
 118:   SmallVector<Value> createLoopArgs(OpBuilder &builder,
 119:                                     CarriedArgs &carriedArgs);
 120:   void cloneLoopBody(scf::ForOp newForOp, OpBuilder &builder,
 121:                      IRMapping &mapping, const CarriedArgs &carriedArgs);
 122:   SmallVector<Value> createYieldValues(OpBuilder &builder, IRMapping &mapping,
 123:                                        const CarriedArgs &carriedArgs);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 125-126

```cpp
 125:   void cloneElementwiseOps(Value &bRem, const SmallVector<Value> &vals,
 126:                            OpBuilder &builder);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 128-129

```cpp
 128: public:
 129:   Prefetcher() = delete;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 131-134

```cpp
 131:   Prefetcher(scf::ForOp forOp, int computeCapability)
 132:       : forOp(forOp), computeCapability(computeCapability) {
 133:     yieldOp = cast<scf::YieldOp>(forOp.getBody()->getTerminator());
 134:   }
```

- **EN:** Defines `Prefetcher`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `Prefetcher`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 136-136

```cpp
 136:   LogicalResult initialize();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 138-138

```cpp
 138:   void emitPrologue();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 140-141

```cpp
 140:   scf::ForOp createNewForOp();
 141: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 143-159

```cpp
 143: void Prefetcher::cloneElementwiseOps(Value &ret, const SmallVector<Value> &vals,
 144:                                      OpBuilder &builder) {
 145:   IRMapping mapping;
 146:   mapping.map(vals[1], ret);
 147:   for (int i = 2; i < vals.size(); i++) {
 148:     Value v = vals[i];
 149:     Value curr = builder.clone(*v.getDefiningOp(), mapping)->getResult(0);
 150:     if (isa<RankedTensorType>(curr.getType())) {
 151:       auto retType = RankedTensorType::get(
 152:           cast<RankedTensorType>(ret.getType()).getShape(),
 153:           cast<RankedTensorType>(curr.getType()).getElementType(),
 154:           cast<RankedTensorType>(curr.getDefiningOp()->getOperand(0).getType())
 155:               .getEncoding());
 156:       curr.setType(retType);
 157:     }
 158:     mapping.map(v, curr);
 159:   }
```

- **EN:** Defines `Prefetcher::cloneElementwiseOps`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `Prefetcher::cloneElementwiseOps`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 160-162

```cpp
 160:   if (vals.size() > 1)
 161:     ret = mapping.lookup(vals.back());
 162: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 164-173

```cpp
 164: Value Prefetcher::generatePrefetch(Value v, unsigned opIdx, bool isPrologue,
 165:                                    Attribute dotEncoding, OpBuilder &builder,
 166:                                    Value token, std::optional<int64_t> offsetK,
 167:                                    std::optional<int64_t> shapeK) {
 168:   // opIdx: 0 => a, 1 => b
 169:   auto type = cast<triton::gpu::MemDescType>(v.getType());
 170:   SmallVector<int64_t> shape{type.getShape().begin(), type.getShape().end()};
 171:   auto rank = shape.size();
 172:   SmallVector<int32_t> offset(rank, 0);
 173:   Type elementType = type.getElementType();
```

- **EN:** Defines `Prefetcher::generatePrefetch`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic.
- **CN:** 这里定义 `Prefetcher::generatePrefetch`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。
### Lines 175-176

```cpp
 175:   // k => (prefetchWidth, k - prefetchWidth)
 176:   int64_t kIdx = opIdx == 0 ? rank - 1 : rank - 2;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 178-179

```cpp
 178:   offset[kIdx] = isPrologue ? 0 : prefetchWidth;
 179:   shape[kIdx] = isPrologue ? prefetchWidth : (shape[kIdx] - prefetchWidth);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 181-184

```cpp
 181:   if (shapeK)
 182:     shape[kIdx] = *shapeK;
 183:   if (offsetK)
 184:     offset[kIdx] = *offsetK;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 186-191

```cpp
 186:   Value newSmem = triton::gpu::MemDescSubsliceOp::create(
 187:       builder, v.getLoc(),
 188:       triton::gpu::MemDescType::get(
 189:           shape, elementType, type.getEncoding(), type.getMemorySpace(),
 190:           type.getMutableMemory(), type.getAllocShape()),
 191:       v, offset);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Shared-memory descriptors and layout-aware addressing are explicit parts of the logic.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 共享内存描述符与布局相关寻址是这段逻辑的显式组成部分。
### Lines 193-197

```cpp
 193:   auto dotOperandEnc = triton::gpu::DotOperandEncodingAttr::get(
 194:       builder.getContext(), opIdx, dotEncoding, prefetchWidth / 8);
 195:   Value prefetchSlice = triton::gpu::LocalLoadOp::create(
 196:       builder, v.getLoc(),
 197:       RankedTensorType::get(shape, elementType, dotOperandEnc), newSmem, token);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 199-200

```cpp
 199:   return prefetchSlice;
 200: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 202-207

```cpp
 202: unsigned Prefetcher::getPrefetchWidth(Attribute dotEncoding, Type elementType,
 203:                                       unsigned kWidth) const {
 204:   if (kWidth == 0)
 205:     return 256 / elementType.getIntOrFloatBitWidth();
 206:   return 8 * kWidth;
 207: }
```

- **EN:** Defines accessor/helper `Prefetcher::getPrefetchWidth` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `Prefetcher::getPrefetchWidth`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 209-213

```cpp
 209: bool Prefetcher::isLoopCarriedValue(Value v) {
 210:   auto arg = dyn_cast_if_present<BlockArgument>(v);
 211:   return arg && arg.getOwner() == forOp.getBody() &&
 212:          arg.getArgNumber() >= forOp.getNumInductionVars();
 213: }
```

- **EN:** Defines `Prefetcher::isLoopCarriedValue`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `Prefetcher::isLoopCarriedValue`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 215-220

```cpp
 215: Value Prefetcher::getIncomingValue(Value v) {
 216:   if (!isLoopCarriedValue(v))
 217:     return Value();
 218:   auto arg = cast<BlockArgument>(v);
 219:   return forOp.getTiedLoopInit(arg)->get();
 220: }
```

- **EN:** Defines accessor/helper `Prefetcher::getIncomingValue` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `Prefetcher::getIncomingValue`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 222-228

```cpp
 222: Value Prefetcher::getYieldValue(Value v) {
 223:   if (!isLoopCarriedValue(v))
 224:     return Value();
 225:   auto arg = cast<BlockArgument>(v);
 226:   unsigned yieldIdx = arg.getArgNumber() - forOp.getNumInductionVars();
 227:   return yieldOp.getOperand(yieldIdx);
 228: }
```

- **EN:** Defines accessor/helper `Prefetcher::getYieldValue` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `Prefetcher::getYieldValue`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 230-247

```cpp
 230: bool Prefetcher::isPromotableValue(Value v) {
 231:   // Null operands are treated as trivially promotable.
 232:   // e.g., local_load no tokens
 233:   if (!v)
 234:     return true;
 235:   if (auto arg = dyn_cast<BlockArgument>(v))
 236:     return arg.getOwner() != forOp.getBody() || isLoopCarriedValue(arg) ||
 237:            arg == forOp.getInductionVar();
 238:   // Loop-carried block arguments can be remapped to either the init value or
 239:   // the yielded next-iteration value during rewrite.
 240:   if (isLoopCarriedValue(v))
 241:     return true;
 242:   Operation *op = v.getDefiningOp();
 243:   // Other block arguments / values without a defining op are assumed safe.
 244:   if (!op)
 245:     return true;
 246:   // Values defined outside this loop body are already available where we
 247:   // materialize the prologue/yield expressions, so they do not need cloning.
```

- **EN:** Defines `Prefetcher::isPromotableValue`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `Prefetcher::isPromotableValue`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 248-264

```cpp
 248:   if (op->getBlock() != forOp.getBody())
 249:     return true;
 250:   // Nested control flow is not handled by the cloning logic below.
 251:   if (op->getNumRegions() != 0)
 252:     return false;
 253:   // Only clone simple elementwise/constant ops plus the specific loop-local
 254:   // ops needed to rebuild the async-wait + memdesc-index chain.
 255:   if (!op->hasTrait<OpTrait::Elementwise>() &&
 256:       !op->hasTrait<OpTrait::ConstantLike>() &&
 257:       !isa<triton::gpu::AsyncWaitOp, triton::gpu::MemDescIndexOp>(op))
 258:     return false;
 259:   // Every operand must also be promotable, otherwise the whole expression is
 260:   // rejected.
 261:   return llvm::all_of(op->getOperands(), [this](Value operand) {
 262:     return isPromotableValue(operand);
 263:   });
 264: }
```

- **EN:** Defines `llvm::all_of`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `llvm::all_of`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 266-283

```cpp
 266: Value Prefetcher::cloneLoopValue(
 267:     Value v, OpBuilder &builder,
 268:     llvm::function_ref<Value(BlockArgument)> mapBlockArg,
 269:     DenseMap<Value, Value> &cache) {
 270:   // Null values are allowed for optional operands such as local_load tokens.
 271:   if (!v)
 272:     return Value();
 273:   // Reuse previously cloned values when reconstructing a shared expression DAG.
 274:   if (auto it = cache.find(v); it != cache.end())
 275:     return it->second;
 276:   // Block arguments are remapped by the caller depending on whether we are
 277:   // materializing the loop init or the yielded next-iteration value.
 278:   if (auto arg = dyn_cast<BlockArgument>(v))
 279:     return cache[v] = mapBlockArg(arg);
 280:   Operation *op = v.getDefiningOp();
 281:   // Values defined outside this loop body can be reused directly.
 282:   if (op->getBlock() != forOp.getBody())
 283:     return cache[v] = v;
```

- **EN:** Defines `Prefetcher::cloneLoopValue`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `Prefetcher::cloneLoopValue`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 285-295

```cpp
 285:   // Recursively rebuild the loop-local expression with remapped operands.
 286:   IRMapping operandMapping;
 287:   for (Value operand : op->getOperands())
 288:     operandMapping.map(operand,
 289:                        cloneLoopValue(operand, builder, mapBlockArg, cache));
 290:   Operation *clonedOp = builder.clone(*op, operandMapping);
 291:   for (auto [result, clonedResult] :
 292:        llvm::zip(op->getResults(), clonedOp->getResults()))
 293:     cache[result] = clonedResult;
 294:   return cache[v];
 295: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 297-309

```cpp
 297: Value Prefetcher::materializeInitValue(Value v, OpBuilder &builder,
 298:                                        DenseMap<Value, Value> &cache) {
 299:   return cloneLoopValue(
 300:       v, builder,
 301:       [this](BlockArgument arg) -> Value {
 302:         if (arg.getOwner() != forOp.getBody())
 303:           return arg;
 304:         if (arg == forOp.getInductionVar())
 305:           return forOp.getLowerBound();
 306:         return forOp.getTiedLoopInit(arg)->get();
 307:       },
 308:       cache);
 309: }
```

- **EN:** Defines `Prefetcher::materializeInitValue`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `Prefetcher::materializeInitValue`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 311-319

```cpp
 311: void Prefetcher::appendMaterializedLoopArgIfNeeded(
 312:     triton::DotOp dot, Value value, DenseMap<Operation *, unsigned> &argMap,
 313:     SmallVector<Value> &loopArgs, OpBuilder &builder) {
 314:   if (!value || isLoopCarriedValue(value))
 315:     return;
 316:   argMap[dot] = loopArgs.size();
 317:   loopArgs.push_back(
 318:       materializeInitValue(value, builder, initMaterializations));
 319: }
```

- **EN:** Defines `Prefetcher::appendMaterializedLoopArgIfNeeded`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `Prefetcher::appendMaterializedLoopArgIfNeeded`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 321-325

```cpp
 321: Value Prefetcher::getTrackedValue(triton::DotOp dot, bool isA, bool isToken) {
 322:   if (isToken)
 323:     return isA ? dot2aToken.lookup(dot) : dot2bToken.lookup(dot);
 324:   return isA ? dot2aSource.lookup(dot) : dot2bSource.lookup(dot);
 325: }
```

- **EN:** Defines accessor/helper `Prefetcher::getTrackedValue` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `Prefetcher::getTrackedValue`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 327-333

```cpp
 327: const DenseMap<Operation *, unsigned> &
 328: Prefetcher::getCarriedArgMap(const CarriedArgs &carriedArgs, bool isA,
 329:                              bool isToken) {
 330:   if (isToken)
 331:     return isA ? carriedArgs.a : carriedArgs.b;
 332:   return isA ? carriedArgs.aSource : carriedArgs.bSource;
 333: }
```

- **EN:** Defines accessor/helper `Prefetcher::getCarriedArgMap` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `Prefetcher::getCarriedArgMap`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 335-352

```cpp
 335: Value Prefetcher::getCurrentTrackedValue(triton::DotOp dot, bool isA,
 336:                                          bool isToken, scf::ForOp newForOp,
 337:                                          IRMapping &mapping,
 338:                                          const CarriedArgs &carriedArgs) {
 339:   Value value = getTrackedValue(dot, isA, isToken);
 340:   if (!value)
 341:     return Value();
 342:   // If token or source value is initially loop carried. It means local_load is
 343:   // done outside of the loop and we can directly use the tracked value
 344:   if (isLoopCarriedValue(value))
 345:     return mapping.lookupOrDefault(value);
 346:   const auto &argMap = getCarriedArgMap(carriedArgs, isA, isToken);
 347:   auto it = argMap.find(dot);
 348:   if (it == argMap.end())
 349:     // The arg is invalid for prefetching
 350:     return isToken ? Value() : mapping.lookupOrDefault(value);
 351:   // The arg is initalized outside of the loop and passed into the new loop as
 352:   // an argument
```

- **EN:** Defines accessor/helper `Prefetcher::getCurrentTrackedValue` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `Prefetcher::getCurrentTrackedValue`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 353-354

```cpp
 353:   return newForOp.getRegionIterArgs()[it->second];
 354: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 356-362

```cpp
 356: Value Prefetcher::getNextTrackedValue(triton::DotOp dot, bool isA, bool isToken,
 357:                                       OpBuilder &builder, IRMapping &mapping) {
 358:   Value value = getTrackedValue(dot, isA, isToken);
 359:   if (!value)
 360:     return Value();
 361:   if (isLoopCarriedValue(value))
 362:     return mapping.lookupOrDefault(getYieldValue(value));
```

- **EN:** Defines accessor/helper `Prefetcher::getNextTrackedValue` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `Prefetcher::getNextTrackedValue`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 364-377

```cpp
 364:   DenseMap<Value, Value> yieldCache;
 365:   return cloneLoopValue(
 366:       value, builder,
 367:       [this, &builder, &mapping](BlockArgument arg) -> Value {
 368:         if (arg.getOwner() != forOp.getBody())
 369:           return arg;
 370:         if (arg == forOp.getInductionVar())
 371:           return arith::AddIOp::create(builder, forOp.getLoc(),
 372:                                        mapping.lookupOrDefault(arg),
 373:                                        forOp.getStep());
 374:         return mapping.lookupOrDefault(getYieldValue(arg));
 375:       },
 376:       yieldCache);
 377: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 379-381

```cpp
 379: LogicalResult Prefetcher::initialize() {
 380:   Block *loop = forOp.getBody();
 381:   auto kBlock = StringAttr::get(forOp.getContext(), "block");
```

- **EN:** Defines `Prefetcher::initialize`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `Prefetcher::initialize`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 383-391

```cpp
 383:   auto getEncoding = [](Value v) {
 384:     return cast<TensorOrMemDesc>(v.getType()).getEncoding();
 385:   };
 386:   auto isBroadcasted = [kBlock, &getEncoding](Value v) {
 387:     auto cgaLayout = getCGALayout(getEncoding(v)).getLinearLayout();
 388:     if (!cgaLayout.hasInDim(kBlock))
 389:       return false;
 390:     return cgaLayout.getFreeVariableMasks()[kBlock] != 0;
 391:   };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 393-410

```cpp
 393:   SmallVector<triton::DotOp> dotsInFor;
 394:   for (Operation &op : *loop) {
 395:     if (auto dotOp = dyn_cast<triton::DotOp>(op)) {
 396:       // Only accepts dotOps encoded as Nvidia MMA v2 or AMD MFMA
 397:       auto dstMmaEnc =
 398:           dyn_cast<NvidiaMmaEncodingAttr>(getEncoding(dotOp.getResult()));
 399:       auto dstMfmaEnc =
 400:           dyn_cast<AMDMfmaEncodingAttr>(getEncoding(dotOp.getResult()));
 401:       if (!dstMfmaEnc && (!dstMmaEnc || dstMmaEnc.getVersionMajor() != 2))
 402:         // Don't rewrite if any other type is found.
 403:         return failure();
 404:       dotsInFor.push_back(dotOp);
 405:     }
 406:     if (isa<triton::nvidia_gpu::TMAOpInterface>(op)) {
 407:       // Don't rewrite if syncTMACopy or asyncTMACopy is found since they may
 408:       // have dependencies with the dot op that are not handled by the current
 409:       // implementation.
 410:       return failure();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 411-412

```cpp
 411:     }
 412:   }
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 414-415

```cpp
 414:   if (dotsInFor.empty())
 415:     return failure();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 417-420

```cpp
 417:   // TODO: segfault (original for still has uses)
 418:   // when used in flash attention that has 2 dots in the loop
 419:   if (dotsInFor.size() > 1)
 420:     return failure();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 422-438

```cpp
 422:   // Walk back from the dot operand to the shared-memory value consumed by the
 423:   // local_load chain.
 424:   auto getPrefetchSrc = [](Value v) -> SmallVector<Value> {
 425:     // Walk backwards through the single-use chain until we find local_load.
 426:     Operation *op = v.getDefiningOp();
 427:     if (!op)
 428:       return {};
 429:     bool foundConvertFromShared = false;
 430:     SmallVector<Value> rets;
 431:     rets.push_back(op->getResult(0));
 432:     LDBG("Prefetch src: " << *op);
 433:     while (op) {
 434:       if (!op->getResult(0).hasOneUse())
 435:         break;
 436:       if (auto load = dyn_cast<triton::gpu::LocalLoadOp>(op)) {
 437:         rets.push_back(load.getSrc());
 438:         // Only handle the direct dot-operand load chain for now.
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 439-450

```cpp
 439:         if (isa<DotOperandEncodingAttr>(load.getType().getEncoding()))
 440:           foundConvertFromShared = true;
 441:         break;
 442:       }
 443:       if (op->getNumOperands() != 1)
 444:         break;
 445:       rets.push_back(op->getOperand(0));
 446:       op = op->getOperand(0).getDefiningOp();
 447:       if (op)
 448:         LDBG("op: " << *op);
 449:     }
 450:     std::reverse(rets.begin(), rets.end());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 452-455

```cpp
 452:     if (foundConvertFromShared)
 453:       return rets;
 454:     return {};
 455:   };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 457-467

```cpp
 457:   auto getLoadToken = [](Value v) -> Value {
 458:     Operation *op = v.getDefiningOp();
 459:     while (op) {
 460:       if (auto load = dyn_cast<triton::gpu::LocalLoadOp>(op))
 461:         return load.getToken();
 462:       if (op->getNumOperands() != 1)
 463:         break;
 464:       op = op->getOperand(0).getDefiningOp();
 465:     }
 466:     return Value();
 467:   };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 469-479

```cpp
 469:   for (triton::DotOp dot : dotsInFor) {
 470:     auto aType = dot.getA().getType();
 471:     auto bType = dot.getB().getType();
 472:     auto dotEncoding = dot.getType().getEncoding();
 473:     auto aEnc =
 474:         mlir::cast<triton::gpu::DotOperandEncodingAttr>(aType.getEncoding());
 475:     auto bEnc =
 476:         mlir::cast<triton::gpu::DotOperandEncodingAttr>(bType.getEncoding());
 477:     int aKWidth = aEnc.getKWidth();
 478:     int bKWidth = bEnc.getKWidth();
 479:     assert(aKWidth == bKWidth);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 481-481

```cpp
 481:     auto kSize = aType.getShape().back();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 483-485

```cpp
 483:     // Match the chunk width expected by the dot operand encoding.
 484:     prefetchWidth =
 485:         getPrefetchWidth(dotEncoding, aType.getElementType(), aKWidth);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 487-491

```cpp
 487:     // Skip prefetching if kSize is less than prefetchWidth
 488:     if (kSize < prefetchWidth)
 489:       continue;
 490:     auto aVals = getPrefetchSrc(dot.getA());
 491:     auto bVals = getPrefetchSrc(dot.getB());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 493-510

```cpp
 493:     if (aVals.size() && bVals.size()) {
 494:       Value aSmem = aVals.front();
 495:       Value bSmem = bVals.front();
 496:       if (isBroadcasted(aSmem) || isBroadcasted(bSmem))
 497:         continue;
 498:       dot2aVals[dot] = aVals;
 499:       dot2bVals[dot] = bVals;
 500:       dot2aSource[dot] = aSmem;
 501:       dot2bSource[dot] = bSmem;
 502:       dot2aToken[dot] = getLoadToken(dot.getA());
 503:       dot2bToken[dot] = getLoadToken(dot.getB());
 504:       Value aHeaderDef = getIncomingValue(aSmem);
 505:       Value bHeaderDef = getIncomingValue(bSmem);
 506:       bool hasLoopCarriedSrc = aHeaderDef && bHeaderDef;
 507:       bool canPromoteSplitDot =
 508:           (dot2aToken[dot] || dot2bToken[dot]) && isPromotableValue(aSmem) &&
 509:           isPromotableValue(bSmem) && isPromotableValue(dot2aToken[dot]) &&
 510:           isPromotableValue(dot2bToken[dot]);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 511-515

```cpp
 511:       if (hasLoopCarriedSrc || canPromoteSplitDot) {
 512:         dots.insert(dot);
 513:       }
 514:     }
 515:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 517-520

```cpp
 517:   if (dots.empty())
 518:     return failure();
 519:   return success();
 520: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. The code uses `LogicalResult` to stay conservative when preconditions are not met.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 代码通过 `LogicalResult` 在前提不满足时保持保守处理。
### Lines 522-523

```cpp
 522: void Prefetcher::emitPrologue() {
 523:   OpBuilder builder(forOp);
```

- **EN:** Defines `Prefetcher::emitPrologue`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `Prefetcher::emitPrologue`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 525-538

```cpp
 525:   for (triton::DotOp dot : dots) {
 526:     Attribute dotEncoding = dot.getType().getEncoding();
 527:     Value aPrefetched = generatePrefetch(
 528:         materializeInitValue(dot2aSource[dot], builder, initMaterializations),
 529:         0, true, dotEncoding, builder,
 530:         materializeInitValue(dot2aToken.lookup(dot), builder,
 531:                              initMaterializations));
 532:     cloneElementwiseOps(aPrefetched, dot2aVals[dot], builder);
 533:     Value bPrefetched = generatePrefetch(
 534:         materializeInitValue(dot2bSource[dot], builder, initMaterializations),
 535:         1, true, dotEncoding, builder,
 536:         materializeInitValue(dot2bToken.lookup(dot), builder,
 537:                              initMaterializations));
 538:     cloneElementwiseOps(bPrefetched, dot2bVals[dot], builder);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 540-543

```cpp
 540:     operand2headPrefetch[dot.getA()] = aPrefetched;
 541:     operand2headPrefetch[dot.getB()] = bPrefetched;
 542:   }
 543: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 545-562

```cpp
 545: SmallVector<Value> Prefetcher::createLoopArgs(OpBuilder &builder,
 546:                                               CarriedArgs &carriedArgs) {
 547:   SmallVector<Value> loopArgs;
 548:   for (auto v : forOp.getInitArgs())
 549:     loopArgs.push_back(v);
 550:   for (triton::DotOp dot : dots) {
 551:     appendMaterializedLoopArgIfNeeded(dot, dot2aSource.lookup(dot),
 552:                                       carriedArgs.aSource, loopArgs, builder);
 553:     appendMaterializedLoopArgIfNeeded(dot, dot2bSource.lookup(dot),
 554:                                       carriedArgs.bSource, loopArgs, builder);
 555:     appendMaterializedLoopArgIfNeeded(dot, dot2aToken.lookup(dot),
 556:                                       carriedArgs.a, loopArgs, builder);
 557:     appendMaterializedLoopArgIfNeeded(dot, dot2bToken.lookup(dot),
 558:                                       carriedArgs.b, loopArgs, builder);
 559:     loopArgs.push_back(operand2headPrefetch[dot.getA()]);
 560:     loopArgs.push_back(operand2headPrefetch[dot.getB()]);
 561:   }
 562:   return loopArgs;
```

- **EN:** Defines helper `Prefetcher::createLoopArgs` that computes or constructs intermediate data used by the surrounding transformation. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义辅助函数 `Prefetcher::createLoopArgs`，用于计算或构造外围变换所需的中间数据。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 563-563

```cpp
 563: }
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 565-576

```cpp
 565: void Prefetcher::cloneLoopBody(scf::ForOp newForOp, OpBuilder &builder,
 566:                                IRMapping &mapping,
 567:                                const CarriedArgs &carriedArgs) {
 568:   // Keep late-sunk ops before the loop terminator.
 569:   auto setInsertionPointBeforeYield = [](OpBuilder &builder,
 570:                                          scf::ForOp newForOp) {
 571:     if (newForOp.getBody()->mightHaveTerminator()) {
 572:       builder.setInsertionPoint(newForOp.getBody()->getTerminator());
 573:     } else {
 574:       builder.setInsertionPointToEnd(newForOp.getBody());
 575:     }
 576:   };
```

- **EN:** Defines `Prefetcher::cloneLoopBody`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `Prefetcher::cloneLoopBody`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 578-594

```cpp
 578:   for (Operation &op : forOp.getBody()->without_terminator()) {
 579:     // If we're currently trying to sink a prefetched dot, we need to stop
 580:     // sinking it (by resetting the insertion point to the end) if we find
 581:     // control flow, or anything that depends on the dot op.
 582:     if (op.getNumRegions() > 0) {
 583:       setInsertionPointBeforeYield(builder, newForOp);
 584:     }
 585:     for (auto operand : op.getOperands()) {
 586:       if (auto def = operand.getDefiningOp()) {
 587:         auto dot = dyn_cast<triton::DotOp>(def);
 588:         if (dot && dots.contains(dot)) {
 589:           setInsertionPointBeforeYield(builder, newForOp);
 590:         }
 591:       }
 592:     }
 593:     Operation *newOp = builder.clone(op, mapping);
 594:     auto dot = dyn_cast<triton::DotOp>(&op);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 595-604

```cpp
 595:     if (dot && dots.contains(dot)) {
 596:       Attribute dotEncoding = dot.getType().getEncoding();
 597:       // First dot uses the values prefetched before entering the loop.
 598:       Operation *firstDot = builder.clone(*dot, mapping);
 599:       if (Value a = operand2headPrefetch.lookup(dot.getA()))
 600:         firstDot->setOperand(
 601:             0, newForOp.getTiedLoopRegionIterArg(&*a.use_begin()));
 602:       if (Value b = operand2headPrefetch.lookup(dot.getB()))
 603:         firstDot->setOperand(
 604:             1, newForOp.getTiedLoopRegionIterArg(&*b.use_begin()));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 606-617

```cpp
 606:       // Emit additional dots for the remainder of K after the prefetched head.
 607:       const int64_t kChunk = prefetchWidth;
 608:       int64_t kOff = kChunk;
 609:       int64_t kRem = dot.getA().getType().getShape().back() - kChunk;
 610:       Operation *prevDot = firstDot;
 611:       if (kRem == 0) {
 612:         // There is only one dot while prefetchWidth == kSize so delay issuing
 613:         // it. Meanwhile, newOp should be set to firstDot to make sure the dot
 614:         // result is updated to yield.
 615:         builder.setInsertionPoint(prevDot);
 616:         newOp = firstDot;
 617:       }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 619-630

```cpp
 619:       while (kRem != 0) {
 620:         int64_t kShape = kChunk;
 621:         auto insertionPoint = builder.saveInsertionPoint();
 622:         builder.setInsertionPoint(prevDot);
 623:         Value aRem = generatePrefetch(
 624:             getCurrentTrackedValue(dot, /*isA=*/true, /*isToken=*/false,
 625:                                    newForOp, mapping, carriedArgs),
 626:             0, false, dotEncoding, builder,
 627:             getCurrentTrackedValue(dot, /*isA=*/true, /*isToken=*/true,
 628:                                    newForOp, mapping, carriedArgs),
 629:             kOff, kShape);
 630:         cloneElementwiseOps(aRem, dot2aVals[dot], builder);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 631-648

```cpp
 631:         Value bRem = generatePrefetch(
 632:             getCurrentTrackedValue(dot, /*isA=*/false, /*isToken=*/false,
 633:                                    newForOp, mapping, carriedArgs),
 634:             1, false, dotEncoding, builder,
 635:             getCurrentTrackedValue(dot, /*isA=*/false, /*isToken=*/true,
 636:                                    newForOp, mapping, carriedArgs),
 637:             kOff, kShape);
 638:         cloneElementwiseOps(bRem, dot2bVals[dot], builder);
 639:         builder.restoreInsertionPoint(insertionPoint);
 640:         newOp = builder.clone(*dot, mapping);
 641:         newOp->setOperand(0, aRem);
 642:         newOp->setOperand(1, bRem);
 643:         newOp->setOperand(2, prevDot->getResult(0));
 644:         prevDot = newOp;
 645:         kOff += kShape;
 646:         kRem -= kShape;
 647:         if (kRem == 0) {
 648:           // We want to delay issuing the last dot as long as possible, ideally
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 649-660

```cpp
 649:           // until after the prefetch.  To accomplish this, set the insertion
 650:           // point above the dot.  If we find anything dependent on the dot (at
 651:           // the top of this loop), we resume inserting after it.
 652:           builder.setInsertionPoint(prevDot);
 653:         }
 654:       }
 655:     }
 656:     // Forward all uses in the cloned body to the rewritten operations.
 657:     for (unsigned dstIdx : llvm::seq(unsigned(0), op.getNumResults()))
 658:       mapping.map(op.getResult(dstIdx), newOp->getResult(dstIdx));
 659:   }
 660: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 662-676

```cpp
 662: SmallVector<Value>
 663: Prefetcher::createYieldValues(OpBuilder &builder, IRMapping &mapping,
 664:                               const CarriedArgs &carriedArgs) {
 665:   SmallVector<Value> yieldValues;
 666:   for (Value v : forOp.getBody()->getTerminator()->getOperands())
 667:     yieldValues.push_back(mapping.lookupOrDefault(v));
 668:   for (triton::DotOp dot : dots) {
 669:     Value nextASource = getNextTrackedValue(
 670:         dot, /*isA=*/true, /*isToken=*/false, builder, mapping);
 671:     Value nextBSource = getNextTrackedValue(
 672:         dot, /*isA=*/false, /*isToken=*/false, builder, mapping);
 673:     Value nextAToken = getNextTrackedValue(dot, /*isA=*/true, /*isToken=*/true,
 674:                                            builder, mapping);
 675:     Value nextBToken = getNextTrackedValue(dot, /*isA=*/false, /*isToken=*/true,
 676:                                            builder, mapping);
```

- **EN:** Defines helper `Prefetcher::createYieldValues` that computes or constructs intermediate data used by the surrounding transformation. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义辅助函数 `Prefetcher::createYieldValues`，用于计算或构造外围变换所需的中间数据。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 678-695

```cpp
 678:     if (carriedArgs.aSource.contains(dot))
 679:       yieldValues.push_back(nextASource);
 680:     if (carriedArgs.bSource.contains(dot))
 681:       yieldValues.push_back(nextBSource);
 682:     if (carriedArgs.a.contains(dot))
 683:       yieldValues.push_back(nextAToken);
 684:     if (carriedArgs.b.contains(dot))
 685:       yieldValues.push_back(nextBToken);
 686:     Attribute dotEncoding = dot.getType().getEncoding();
 687:     Value aToYield = generatePrefetch(nextASource, 0, true, dotEncoding,
 688:                                       builder, nextAToken);
 689:     cloneElementwiseOps(aToYield, dot2aVals[dot], builder);
 690:     yieldValues.push_back(aToYield);
 691:     Value bToYield = generatePrefetch(nextBSource, 1, true, dotEncoding,
 692:                                       builder, nextBToken);
 693:     cloneElementwiseOps(bToYield, dot2bVals[dot], builder);
 694:     yieldValues.push_back(bToYield);
 695:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 696-697

```cpp
 696:   return yieldValues;
 697: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 699-702

```cpp
 699: scf::ForOp Prefetcher::createNewForOp() {
 700:   OpBuilder builder(forOp);
 701:   CarriedArgs carriedArgs;
 702:   SmallVector<Value> loopArgs = createLoopArgs(builder, carriedArgs);
```

- **EN:** Defines helper `Prefetcher::createNewForOp` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `Prefetcher::createNewForOp`，用于计算或构造外围变换所需的中间数据。
### Lines 704-706

```cpp
 704:   auto newForOp =
 705:       scf::ForOp::create(builder, forOp.getLoc(), forOp.getLowerBound(),
 706:                          forOp.getUpperBound(), forOp.getStep(), loopArgs);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 708-712

```cpp
 708:   builder.setInsertionPointToStart(newForOp.getBody());
 709:   IRMapping mapping;
 710:   for (const auto &arg : llvm::enumerate(forOp.getRegionIterArgs()))
 711:     mapping.map(arg.value(), newForOp.getRegionIterArgs()[arg.index()]);
 712:   mapping.map(forOp.getInductionVar(), newForOp.getInductionVar());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 714-714

```cpp
 714:   cloneLoopBody(newForOp, builder, mapping, carriedArgs);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 716-723

```cpp
 716:   SmallVector<Value> yieldValues =
 717:       createYieldValues(builder, mapping, carriedArgs);
 718:   // Replace the loop terminator with the rebuilt yield.
 719:   builder.setInsertionPointToEnd(newForOp.getBody());
 720:   if (!yieldValues.empty())
 721:     scf::YieldOp::create(builder, yieldOp.getLoc(), yieldValues);
 722:   return newForOp;
 723: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 725-725

```cpp
 725: } // anonymous namespace
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 727-728

```cpp
 727: struct PrefetchPass : public impl::TritonGPUPrefetchBase<PrefetchPass> {
 728:   void runOnOperation() override {
```

- **EN:** Defines `PrefetchPass`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `PrefetchPass`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 730-746

```cpp
 730:     // Canonicalize convert ops to make the pattern matching easier.
 731:     RewritePatternSet cleanUpPatterns(&getContext());
 732:     ModuleOp m = getOperation();
 733:     int computeCapability = 0;
 734:     if (auto targetAttr =
 735:             m->getAttrOfType<StringAttr>(triton::gpu::AttrTargetName);
 736:         targetAttr && targetAttr.getValue().starts_with("cuda:")) {
 737:       computeCapability = getNVIDIAComputeCapability(m);
 738:     }
 739:     triton::gpu::ConvertLayoutOp::getCanonicalizationPatterns(cleanUpPatterns,
 740:                                                               &getContext());
 741:     if (mlir::applyPatternsGreedily(getOperation(), std::move(cleanUpPatterns))
 742:             .failed()) {
 743:       signalPassFailure();
 744:     }
 745:     m->walk([&](scf::ForOp forOp) {
 746:       Prefetcher prefetcher(forOp, computeCapability);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. The implementation traverses IR operations to collect facts or apply rewrites globally.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。
### Lines 748-749

```cpp
 748:       if (prefetcher.initialize().failed())
 749:         return;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 751-751

```cpp
 751:       prefetcher.emitPrologue();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 753-753

```cpp
 753:       scf::ForOp newForOp = prefetcher.createNewForOp();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 755-761

```cpp
 755:       // replace the original loop
 756:       for (unsigned i = 0; i < forOp->getNumResults(); ++i)
 757:         forOp->getResult(i).replaceAllUsesWith(newForOp->getResult(i));
 758:       forOp->erase();
 759:     });
 760:   }
 761: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 763-765

```cpp
 763: } // namespace gpu
 764: } // namespace triton
 765: } // namespace mlir
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。

## Key Concepts / 关键概念
- **EN:** The main concern is pass-driven transformation around prefetch.
  **CN:** 核心关注点是围绕 Prefetch 的 pass 驱动变换。
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
- **Internal Triton headers / Triton 内部头文件:** `triton/Dialect/TritonGPU/IR/Dialect.h`, `triton/Dialect/TritonGPU/Transforms/Passes.h`, `triton/Dialect/TritonGPU/Transforms/Utility.h`, `triton/Dialect/TritonNvidiaGPU/IR/Dialect.h`, `triton/Dialect/TritonGPU/Transforms/Passes.h.inc`
- **MLIR headers / MLIR 头文件:** `mlir/Dialect/Arith/IR/Arith.h`, `mlir/IR/IRMapping.h`, `mlir/Support/LLVM.h`, `mlir/Transforms/GreedyPatternRewriteDriver.h`
- **LLVM headers / LLVM 头文件:** `llvm/Support/Debug.h`
- **Standard/library headers / 标准或通用库头文件:** None
- **Generated fragments / 生成片段:** `triton/Dialect/TritonGPU/Transforms/Passes.h.inc`
- **Primary APIs used / 主要 API:** `ModuleOp`, `RankedTensorType`, `MemDescType`, `LinearLayout`, `OpBuilder`
- **Pipeline role / 流程角色:** This file participates in Triton pass pipelines and usually expects upstream analyses or dialect invariants to have prepared the IR. / 本文件参与 Triton 的 pass 流程，通常依赖上游分析或方言不变量先把 IR 准备好。
