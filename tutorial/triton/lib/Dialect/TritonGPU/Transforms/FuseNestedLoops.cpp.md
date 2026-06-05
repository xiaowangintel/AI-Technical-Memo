# FuseNestedLoops.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `lib/Dialect/TritonGPU/Transforms/FuseNestedLoops.cpp`
- **Purpose / 作用:** **EN:** Implements the Fuse Nested Loops transformation or optimization pass for the TritonGPU pipeline. **CN:** 为 TritonGPU 编译流程实现与 Fuse Nested Loops 相关的变换或优化 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14

```cpp
   1: #include "mlir/Analysis/TopologicalSortUtils.h"
   2: #include "mlir/Dialect/LLVMIR/LLVMDialect.h"
   3: #include "mlir/Dialect/UB/IR/UBOps.h"
   4: #include "mlir/IR/Dominance.h"
   5: #include "mlir/IR/ImplicitLocOpBuilder.h"
   6: #include "mlir/Interfaces/SideEffectInterfaces.h"
   7: #include "mlir/Transforms/LoopInvariantCodeMotionUtils.h"
   8: #include "mlir/Transforms/RegionUtils.h"
   9: #include "triton/Dialect/Triton/IR/Dialect.h"
  10: #include "triton/Dialect/TritonGPU/Transforms/Passes.h"
  11: #include "triton/Dialect/TritonGPU/Transforms/PipeliningUtility.h"
  12: #include "llvm/ADT/STLExtras.h"
  13: #include "llvm/Support/Debug.h"
  14: #include <queue>
```

- **EN:** Includes the interfaces this file depends on. Triton headers (`Dialect.h`, `Passes.h`, `PipeliningUtility.h`) provide domain-specific IR/support, MLIR headers (`TopologicalSortUtils.h`, `LLVMDialect.h`, `UBOps.h`, `Dominance.h`, ... (+4 more)) provide rewriting and analysis infrastructure, LLVM headers (`STLExtras.h`, `Debug.h`) supply low-level utilities, and standard/library headers (`queue`) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（`Dialect.h`, `Passes.h`, `PipeliningUtility.h`）提供领域特定 IR/支持逻辑，MLIR 头文件（`TopologicalSortUtils.h`, `LLVMDialect.h`, `UBOps.h`, `Dominance.h`, ... (+4 more)）提供重写与分析基础设施，LLVM 头文件（`STLExtras.h`, `Debug.h`）提供底层工具，而标准/通用库头文件（`queue`）提供通用能力。
### Lines 16-18

```cpp
  16: namespace mlir {
  17: namespace triton {
  18: namespace gpu {
```

- **EN:** Opens or closes the namespace nesting for mlir -> triton -> gpu, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 mlir -> triton -> gpu 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 20-22

```cpp
  20: //===----------------------------------------------------------------------===//
  21: // Pass Definition
  22: //===----------------------------------------------------------------------===//
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 24-25

```cpp
  24: #define GEN_PASS_DEF_TRITONGPUFUSENESTEDLOOPS
  25: #include "triton/Dialect/TritonGPU/Transforms/Passes.h.inc"
```

- **EN:** Pulls in generated definitions from TableGen/MLIR include fragments so the handwritten code can reuse auto-generated declarations.
- **CN:** 这里引入由 TableGen/MLIR 生成的定义片段，使手写代码能够复用自动生成的声明。
### Lines 27-32

```cpp
  27: // This attribute is set by the front-end to control whether fusion is on.
  28: static constexpr llvm::StringLiteral kFlattenAttr = "tt.flatten";
  29: // This attribute indicates the inner loop length has been speculated.
  30: static constexpr llvm::StringLiteral kMustExecuteAttrName = "ttg.must-execute";
  31: // This attribute is just used for testing the pass.
  32: static constexpr llvm::StringLiteral kAlwaysFuseAttrName = "ttg.always-fuse";
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 34-37

```cpp
  34: namespace {
  35: struct FuseNestedLoopsPass
  36:     : public impl::TritonGPUFuseNestedLoopsBase<FuseNestedLoopsPass> {
  37:   using TritonGPUFuseNestedLoopsBase::TritonGPUFuseNestedLoopsBase;
```

- **EN:** Defines `FuseNestedLoopsPass`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `FuseNestedLoopsPass`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 39-40

```cpp
  39:   void runOnOperation() override;
  40: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 42-44

```cpp
  42: //===----------------------------------------------------------------------===//
  43: // LoopNest
  44: //===----------------------------------------------------------------------===//
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 46-49

```cpp
  46: // A node in the loop nest represents a single for loop with a list of
  47: // immediately nested loops.
  48: struct LoopNestNode {
  49:   LoopNestNode(scf::ForOp loop) : loop(loop) {}
```

- **EN:** Defines `LoopNestNode`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `LoopNestNode`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 51-55

```cpp
  51:   // The for loop.
  52:   scf::ForOp loop;
  53:   // Loops nested immediately below this loop.
  54:   SmallVector<LoopNestNode *, 1> children;
  55: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 57-59

```cpp
  57: // A loop nest is a tree of loops.
  58: struct LoopNest {
  59:   LoopNest(scf::ForOp outermost);
```

- **EN:** Defines `LoopNest`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `LoopNest`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 61-64

```cpp
  61:   // Print the loop nest.
  62:   void print(raw_ostream &os) const;
  63:   // Dump the loop nest for debugging.
  64:   LLVM_DUMP_METHOD void dump() const;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 66-67

```cpp
  66:   // Owner of the memory of the nodes.
  67:   SmallVector<std::unique_ptr<LoopNestNode>> nodes;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 69-74

```cpp
  69:   // The outermost loop in the nest, which has no preconditions. Even if the
  70:   // outermost loop is contained within an if, its preconditions relative to the
  71:   // loop nest are empty.
  72:   LoopNestNode *root;
  73: };
  74: } // namespace
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 76-79

```cpp
  76: LoopNest::LoopNest(scf::ForOp outermost)
  77:     : root(
  78:           nodes.emplace_back(std::make_unique<LoopNestNode>(outermost)).get()) {
  79: }
```

- **EN:** Defines `LoopNest::LoopNest`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `LoopNest::LoopNest`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 81-89

```cpp
  81: void LoopNest::print(raw_ostream &os) const {
  82:   // Print just the first line of the loop's textual IR.
  83:   std::string buffer;
  84:   auto printLoopFirstLine = [&](scf::ForOp loop) {
  85:     buffer.clear();
  86:     llvm::raw_string_ostream str(buffer);
  87:     loop.print(str);
  88:     os << buffer.substr(0, buffer.find('\n'));
  89:   };
```

- **EN:** Defines `LoopNest::print`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `LoopNest::print`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 91-95

```cpp
  91:   os << "LoopNest:\n";
  92:   SmallVector<std::pair<LoopNestNode *, unsigned>> stack;
  93:   stack.emplace_back(root, 0);
  94:   while (!stack.empty()) {
  95:     auto [node, indent] = stack.pop_back_val();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 97-100

```cpp
  97:     // Print the current loop.
  98:     os << std::string(indent * 2, ' ');
  99:     printLoopFirstLine(node->loop);
 100:     os << "\n";
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 102-107

```cpp
 102:     // Push the children of the current loop.
 103:     for (LoopNestNode *child : node->children)
 104:       stack.emplace_back(child, indent + 1);
 105:   }
 106:   os << "\n";
 107: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 109-109

```cpp
 109: void LoopNest::dump() const { print(llvm::dbgs()); }
```

- **EN:** Defines debug logging helpers and compile-time tags used when tracing this pass or utility at runtime.
- **CN:** 这里定义调试日志辅助宏和编译期标签，用于在运行时跟踪该 pass 或工具。
### Lines 111-113

```cpp
 111: //===----------------------------------------------------------------------===//
 112: // findLoopNests
 113: //===----------------------------------------------------------------------===//
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 115-117

```cpp
 115: // Forward declaration.
 116: static void findLoopNests(Operation *container,
 117:                           SmallVectorImpl<LoopNest> &nests);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 119-124

```cpp
 119: // Recursively construct a loop nest.
 120: static void constructLoopNest(LoopNestNode *parent, LoopNest &nest,
 121:                               SmallVectorImpl<LoopNest> &nests) {
 122:   parent->loop->walk<mlir::WalkOrder::PreOrder>([&](Operation *op) {
 123:     if (op == parent->loop)
 124:       return WalkResult::advance();
```

- **EN:** Defines `constructLoopNest`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `constructLoopNest`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 126-133

```cpp
 126:     if (auto forOp = dyn_cast<scf::ForOp>(op)) {
 127:       auto &child =
 128:           nest.nodes.emplace_back(std::make_unique<LoopNestNode>(forOp));
 129:       parent->children.push_back(child.get());
 130:       // Recurse with the current loop nest.
 131:       constructLoopNest(child.get(), nest, nests);
 132:       return WalkResult::skip();
 133:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 135-144

```cpp
 135:     // If the traversal encounters any other operation with regions, restart the
 136:     // traversal and construct new loop nests. This means ops like `scf.while`
 137:     // divide the analysis domain, but it also means loop fusion won't "see"
 138:     // across `scf.if`, for example.
 139:     // TODO: Handle loop nests with preconditions. The traversal can keep a
 140:     // stack of `scf.if` preconditions while constructing the loop nest.
 141:     if (op->getNumRegions()) {
 142:       findLoopNests(op, nests);
 143:       return WalkResult::skip();
 144:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 146-148

```cpp
 146:     return WalkResult::advance();
 147:   });
 148: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 150-163

```cpp
 150: // Find all the loop nests in the operation. The only region operation that
 151: // allows CFG regions is `tt.func`. That means we can just walk starting from
 152: // the function body and can build loop nests directly off the region trees
 153: // contained in the function -- we don't have to worry about CFGs inside the
 154: // nested region trees.
 155: static void findLoopNests(Operation *container,
 156:                           SmallVectorImpl<LoopNest> &nests) {
 157:   container->walk<mlir::WalkOrder::PreOrder>([&](scf::ForOp loop) {
 158:     LoopNest nest(loop);
 159:     constructLoopNest(nest.root, nest, nests);
 160:     nests.push_back(std::move(nest));
 161:     return WalkResult::skip();
 162:   });
 163: }
```

- **EN:** Defines `findLoopNests`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `findLoopNests`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 165-167

```cpp
 165: //===----------------------------------------------------------------------===//
 166: // Logue
 167: //===----------------------------------------------------------------------===//
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 169-176

```cpp
 169: namespace {
 170: // A prologue or epilogue.
 171: struct Logue {
 172:   // Move the ops in the logue before the iterator.
 173:   void moveBefore(Block *block, Block::iterator it) {
 174:     for (Operation *op : ops)
 175:       op->moveBefore(block, it);
 176:   }
```

- **EN:** Defines `Logue`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `Logue`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 178-188

```cpp
 178:   // Replace all uses of the logue results with the given values, where `logue`
 179:   // comprises all the ops in `containingRegion`.
 180:   void replaceAllUsesWith(ValueRange values, Region &containingRegion) {
 181:     for (auto [newOut, output] : llvm::zip(values, outputs)) {
 182:       // Replace uses of the prologue outputs that are not in the prologue, i.e.
 183:       // inside the `then` region where it got spliced.
 184:       output.replaceUsesWithIf(newOut, [&](OpOperand &use) {
 185:         return !containingRegion.isAncestor(use.getOwner()->getParentRegion());
 186:       });
 187:     }
 188:   }
```

- **EN:** Defines `replaceAllUsesWith`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `replaceAllUsesWith`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 190-195

```cpp
 190:   // Get the number of outputs.
 191:   unsigned getNumOutputs() const { return outputs.size(); }
 192:   // Get the outputs as a `ValueRange`.
 193:   ValueRange getOutputs() const { return outputs; }
 194:   // Get the types of the outputs.
 195:   TypeRange getOutputTypes() const { return getOutputs().getTypes(); }
```

- **EN:** Defines accessor/helper `getNumOutputs` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getNumOutputs`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 197-203

```cpp
 197:   // A contiguous range of ops representing the prologue or epilogue.
 198:   SmallVector<Operation *> ops;
 199:   // The outputs of the logue. These are the SSA value results of `ops` that are
 200:   // used by ops outside of `ops`.
 201:   SmallVector<Value> outputs;
 202: };
 203: } // namespace
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 205-210

```cpp
 205: // Given a range of ops, form it into a logue by finding the outputs.
 206: static Logue createLogueFrom(llvm::iterator_range<Block::iterator> ops,
 207:                              mlir::DominanceInfo &domInfo) {
 208:   Logue logue;
 209:   for (Operation &op : ops)
 210:     logue.ops.push_back(&op);
```

- **EN:** Defines helper `createLogueFrom` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createLogueFrom`，用于计算或构造外围变换所需的中间数据。
### Lines 212-213

```cpp
 212:   if (ops.empty())
 213:     return logue;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 215-224

```cpp
 215:   // An op result is an output of the logue if the last operation in the logue
 216:   // dominates any of its users.
 217:   Operation &lastOp = *std::prev(ops.end());
 218:   auto isOutput = [&](OpResult result) {
 219:     for (Operation *user : result.getUsers()) {
 220:       if (domInfo.properlyDominates(&lastOp, user))
 221:         return true;
 222:     }
 223:     return false;
 224:   };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 226-232

```cpp
 226:   // Find the outputs.
 227:   for (Operation &op : ops) {
 228:     for (OpResult result : op.getOpResults()) {
 229:       if (isOutput(result))
 230:         logue.outputs.push_back(result);
 231:     }
 232:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 234-235

```cpp
 234:   return logue;
 235: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 237-239

```cpp
 237: //===----------------------------------------------------------------------===//
 238: // fuseOneLevel
 239: //===----------------------------------------------------------------------===//
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 241-253

```cpp
 241: // Only hoist operations that are side-effect free and "cheap" (i.e. only scalar
 242: // operands). Importantly, we need to be able to hoist code generated by fusing
 243: // children loops into their parents so the algorithm can be applied
 244: // recursively. This includes integer division, which are not speculatable, but
 245: // we know they will never divide by zero.
 246: static bool canHoistLoopBoundComputation(Operation *op) {
 247:   auto isScalar = [](Type type) {
 248:     return type.isIntOrIndexOrFloat() || isa<PointerType>(type);
 249:   };
 250:   return (isMemoryEffectFree(op) || hasSingleEffect<MemoryEffects::Read>(op)) &&
 251:          llvm::all_of(op->getOperandTypes(), isScalar) &&
 252:          llvm::all_of(op->getResultTypes(), isScalar);
 253: }
```

- **EN:** Defines `canHoistLoopBoundComputation`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `canHoistLoopBoundComputation`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 255-265

```cpp
 255: // Determine if all of `values` are or can be made invariant to the outer loop
 256: // by hoisting operations. `toHoist` is shared across all child loop bounds.
 257: static bool isOuterLoopInvariant(mlir::DominanceInfo &domInfo, scf::ForOp outer,
 258:                                  ArrayRef<Value> values,
 259:                                  llvm::SetVector<Operation *> &toHoist) {
 260:   return getDominatingValueSetOpsToHoist(
 261:       domInfo, outer, values, toHoist, canHoistLoopBoundComputation,
 262:       [&](BlockArgument arg) {
 263:         return isa<FuncOp>(arg.getOwner()->getParentOp());
 264:       });
 265: }
```

- **EN:** Defines `isOuterLoopInvariant`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `isOuterLoopInvariant`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 267-276

```cpp
 267: static bool canSliceBounds(mlir::DominanceInfo &domInfo, scf::ForOp outer,
 268:                            ArrayRef<Value> values,
 269:                            llvm::SetVector<Operation *> &ops) {
 270:   return getDominatingValueSetOpsToHoist(
 271:       domInfo, outer, values, ops, canHoistLoopBoundComputation,
 272:       [&](BlockArgument arg) {
 273:         return arg == outer.getInductionVar() ||
 274:                isa<FuncOp>(arg.getOwner()->getParentOp());
 275:       });
 276: }
```

- **EN:** Defines `canSliceBounds`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `canSliceBounds`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 278-283

```cpp
 278: // Pessimistically assume the internal storage bitwidth for index types.
 279: static unsigned getIntTypeWidth(Type type) {
 280:   if (isa<IndexType>(type))
 281:     return IndexType::kInternalStorageBitWidth;
 282:   return cast<IntegerType>(type).getWidth();
 283: }
```

- **EN:** Defines accessor/helper `getIntTypeWidth` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getIntTypeWidth`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 285-293

```cpp
 285: // Generate IR to compute the number of iterations of a loop.
 286: static Value computeNumIters(ImplicitLocOpBuilder &b, Value lowerBound,
 287:                              Value upperBound, Value step) {
 288:   // len(range(lb, ub, step)) = ceildiv(ub - lb, step)
 289:   // This works even if step is negative.
 290:   Value diff = arith::SubIOp::create(b, upperBound, lowerBound);
 291:   // Let someone else prove it can be unsigned.
 292:   return arith::CeilDivSIOp::create(b, diff, step);
 293: }
```

- **EN:** Defines helper `computeNumIters` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `computeNumIters`，用于计算或构造外围变换所需的中间数据。
### Lines 295-299

```cpp
 295: // Generate IR to compute the number of iterations of a loop.
 296: static Value computeNumIters(ImplicitLocOpBuilder &b, scf::ForOp loop) {
 297:   return computeNumIters(b, loop.getLowerBound(), loop.getUpperBound(),
 298:                          loop.getStep());
 299: }
```

- **EN:** Defines helper `computeNumIters` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `computeNumIters`，用于计算或构造外围变换所需的中间数据。
### Lines 301-312

```cpp
 301: // Cast an integer or index value to an integer or index `type`, if necessary.
 302: static Value castIntIfNecessary(ImplicitLocOpBuilder &b, Value value,
 303:                                 Type type) {
 304:   if (value.getType() == type)
 305:     return value;
 306:   if (isa<IndexType>(value.getType()) || isa<IndexType>(type))
 307:     return arith::IndexCastOp::create(b, type, value);
 308:   if (cast<IntegerType>(value.getType()).getWidth() >
 309:       cast<IntegerType>(type).getWidth())
 310:     return arith::TruncIOp::create(b, type, value);
 311:   return arith::ExtSIOp::create(b, type, value);
 312: }
```

- **EN:** Defines `castIntIfNecessary`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `castIntIfNecessary`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 314-322

```cpp
 314: // To model an "undef" value, i.e. a value that is known to never be read on
 315: // live code paths, create a zero-valued constant where possible, otherwise use
 316: // a poison value. PTXAS appears to generate better code with zeros compared to
 317: // poison values.
 318: static Value createPoisonOrZero(ImplicitLocOpBuilder &b, Type type) {
 319:   Type elTy = getElementTypeOrSelf(type);
 320:   if (!elTy.isIntOrIndexOrFloat() ||
 321:       (!isa<RankedTensorType>(type) && type != elTy))
 322:     return ub::PoisonOp::create(b, type);
```

- **EN:** Defines helper `createPoisonOrZero` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createPoisonOrZero`，用于计算或构造外围变换所需的中间数据。
### Lines 324-329

```cpp
 324:   TypedAttr attr = isa<FloatType>(elTy) ? TypedAttr(b.getFloatAttr(elTy, 0))
 325:                                         : b.getIntegerAttr(elTy, 0);
 326:   if (auto tensor = dyn_cast<RankedTensorType>(type))
 327:     attr = SplatElementsAttr::get(tensor, attr);
 328:   return arith::ConstantOp::create(b, attr);
 329: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 331-333

```cpp
 331: static scf::YieldOp getYield(Region &body) {
 332:   return cast<scf::YieldOp>(body.front().back());
 333: }
```

- **EN:** Defines accessor/helper `getYield` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getYield`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 335-341

```cpp
 335: static scf::IfOp eraseIfResults(ImplicitLocOpBuilder &b, scf::IfOp ifOp,
 336:                                 llvm::BitVector indices,
 337:                                 SmallVector<Value> replaceWith) {
 338:   OpBuilder::InsertionGuard guard(b);
 339:   b.setInsertionPoint(ifOp);
 340:   while (indices.size() < ifOp.getNumResults())
 341:     indices.push_back(false);
```

- **EN:** Defines `eraseIfResults`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `eraseIfResults`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 343-344

```cpp
 343:   getYield(ifOp.getThenRegion())->eraseOperands(indices);
 344:   getYield(ifOp.getElseRegion())->eraseOperands(indices);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. When the lowered form has no direct result, the original op is erased.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 当降级后的形式不再需要直接结果时，原始操作会被删除。
### Lines 346-349

```cpp
 346:   TypeRange newTypes = getYield(ifOp.getThenRegion()).getOperandTypes();
 347:   auto newIf = scf::IfOp::create(b, newTypes, ifOp.getCondition());
 348:   newIf.getThenRegion().takeBody(ifOp.getThenRegion());
 349:   newIf.getElseRegion().takeBody(ifOp.getElseRegion());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 351-360

```cpp
 351:   SmallVector<Value> replacements;
 352:   auto replIt = replaceWith.begin();
 353:   auto resIt = newIf->result_begin();
 354:   for (unsigned i : llvm::seq(ifOp.getNumResults()))
 355:     replacements.push_back(indices[i] ? *replIt++ : *resIt++);
 356:   assert(ValueRange(replacements).getTypes() == ifOp.getResultTypes());
 357:   ifOp.replaceAllUsesWith(replacements);
 358:   ifOp.erase();
 359:   return newIf;
 360: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 362-365

```cpp
 362: namespace {
 363: struct InnerLoop {
 364:   InnerLoop(scf::ForOp op, llvm::SetVector<Operation *> slicedOps)
 365:       : op(op), slicedOps(std::move(slicedOps)) {}
```

- **EN:** Defines `InnerLoop`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `InnerLoop`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 367-368

```cpp
 367:   // Return true if the loop bounds are outer loop invariant.
 368:   bool isOuterLoopInvariant() const { return slicedOps.empty(); }
```

- **EN:** Defines `isOuterLoopInvariant`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `isOuterLoopInvariant`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 370-375

```cpp
 370:   // The actual loop op.
 371:   scf::ForOp op;
 372:   // Ops that must be sliced to compute the loop bounds
 373:   llvm::SetVector<Operation *> slicedOps;
 374: };
 375: } // namespace
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 377-394

```cpp
 377: // Given a one level loop nest in the form
 378: //
 379: //   for i in range(lbi, ubi, stepi):
 380: //     prologue0(i)
 381: //     for j0 in range(lbj0, ubj0, stepj0):
 382: //       body0(i, j0)
 383: //     epilogue1(i)
 384: //     for j1 in range(lbj1, ubj1, stepj1):
 385: //       body1(i, j1)
 386: //     epilogue2(i)
 387: //     ...
 388: //     for jN in range(lbjN, ubjN, stepjN):
 389: //       bodyN(i, jN)
 390: //     epilogue(i)
 391: //
 392: // Rewrite this into a single loop in the form:
 393: //
 394: //   len_i = len(range(lbi, ubi, stepi))
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 395-412

```cpp
 395: //   len_j0 = len(range(lbj0, ubj0, stepj0))
 396: //   len_j1 = len(range(lbj1, ubj1, stepj1))
 397: //   ...
 398: //   len_jN = len(range(lbjN, ubjN, stepjN))
 399: //   inner_len = max(1, len_j0) + max(1, len_j1) + ... + max(1, len_jN) - N
 400: //   total_iters = len_i * inner_len
 401: //
 402: //   T = 0
 403: //   i = lbi - stepi
 404: //   for _ in range(total_iters):
 405: //     if T == 0:
 406: //       i += stepi
 407: //       prologue0(i)
 408: //       j0 = lbj0
 409: //     if T >= 0 and T < len_j0:
 410: //       body0(i, j0)
 411: //       j0 += stepj0
 412: //
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 413-430

```cpp
 413: //     if T == max(1, len_j0) - 1:
 414: //       prologue1(i)
 415: //       j1 = lbj1
 416: //     if T >= max(1, len_j0) - 1
 417: //    and T <  max(1, len_j0) - 1 + len_j1:
 418: //       body1(i, j1)
 419: //       j1 += stepj1
 420: //
 421: //     if T == max(1, len_j0) + max(1, len_j1) - 2:
 422: //       prologue2(i)
 423: //       j2 = lbj2
 424: //     if T >= max(1, len_j0) + max(1, len_j1) - 2
 425: //    and T <  max(1, len_j0) + max(1, len_j1) - 2 + len_j2:
 426: //       body2(i, j2)
 427: //       j2 += stepj2
 428: //
 429: //     ...
 430: //
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 431-448

```cpp
 431: //     if T == max(1, len_j0) + max(1, len_j1) + ... + max(1, len_jN-1) - N:
 432: //       prologueN(i)
 433: //       jN = lbjN
 434: //     if T >= max(1, len_j0) + max(1, len_j1) + ... + max(1, len_jN-1) - N
 435: //    and T <  max(1, len_j0) + max(1, len_j1) + ... + max(1, len_jN-1) - N +
 436: //             len_jN:
 437: //       bodyN(i, jN)
 438: //       jN += stepjN
 439: //
 440: //     if T == max(1, len_j0) + max(1, len_j1) + ... + max(1, len_jN) - (N + 1):
 441: //       epilogue(i)
 442: //     T = 0 if T == (inner_len - 1) else T + 1
 443: //
 444: // This routine can be applied recursively on a loop nest tree, leaf-to-root, to
 445: // flatten the loop nest into a single loop. However, this routine only fuses
 446: // child loops whose loop bounds are invariant to the parent loop. For child
 447: // loops where this is not the case, the function will ignore them.
 448: //
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 449-466

```cpp
 449: // We could fuse loops with parent-loop-variant or even data-dependent bounds,
 450: // but this will require generating `scf.while` in a form that is not friendly
 451: // to the pipeliner. In order to effectively fuse and pipeline these kinds of
 452: // loop nests, loop nest fusion and the pipeliner need to share a higher-level
 453: // representation (or perhaps be the same pass).
 454: //
 455: // Note that there are many potential forms of the fused loop. This routine will
 456: // attempt to minimize the number of fused loop iterations by overlapping the
 457: // iteration spaces of the child loops and the epilogues. E.g. the last
 458: // iteration of bodyjK will execute on the same fused loop iteration as
 459: // epilogueK and the first iteration of bodyj(K+1). Hence the `- N` term in the
 460: // total number of iterations.
 461: //
 462: // What the above Python-pseudo-code glosses over is SSA dependency management.
 463: // To interpret the pseudocode as SSA IR, just imagine everything is put back
 464: // into allocas and SSA formation re-runs after fusion, which one should note
 465: // will introduce undefs.
 466: //
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 467-484

```cpp
 467: // Handling dependencies will require turning implicit captures into
 468: // loop-carried dependencies. Consider:
 469: //
 470: //   scf.for %i = %lbi to %ubi step %stepi {
 471: //     %a = tt.call @func(%i)
 472: //     scf.for %j = %lbj to %ubj step %stepj {
 473: //       %b = tt.call @use(%a, %j)
 474: //     }
 475: //   }
 476: //
 477: // This needs to be rewritten into:
 478: //
 479: //   %poison = ub.poison
 480: //   %Tlast, %ilast, %jlast, %alast = scf.for %unused = ...
 481: //       iter_args(%Tprev = %c-1_i32,
 482: //                 %iprev = %lbi - %stepi,
 483: //                 %jprev = %poison,
 484: //                 %aprev = %poison) -> (i32, i32, i32, i32) {
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 485-502

```cpp
 485: //     %T = (%Tprev + 1) mod (...)
 486: //     %a, %i, %j = scf.if %T == 0 {
 487: //       %inext = %iprev + 1
 488: //       %jnext = %lbj - %stepj
 489: //
 490: //       %anext = tt.call @func(%i)
 491: //       yield %inext, %jnext, %anext
 492: //     } else {
 493: //       yield %iprev, %jprev, %aprev
 494: //     }
 495: //
 496: //     scf.if %T >= 0 and %T < ... {
 497: //       tt.call @use(%a, %j)
 498: //     }
 499: //
 500: // Note: the induction variables will be initialized to their lower bound to
 501: // avoid underflow in lbjk - stepjk, with the exception of the outer loop
 502: // induction variable, which needs to be incremented inside the prologue to
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 503-512

```cpp
 503: // avoid a dependency on the epilogue. This helps the scheduler behave.
 504: //
 505: // Any inputs and outputs of the loop bodies would also need to be handled
 506: // similarly: initialized as undef if appropriate and carried through the fused
 507: // loop. This is why fusion will increase liveranges. To minimize the number of
 508: // additional loop-carried values, the routine will analyze the subblock of IR
 509: // inside each `prologueK` and determine its "outputs" as intermediate SSA
 510: // values that are used later in the loop nest.
 511: static void fuseOneLevel(LoopNestNode *parent, mlir::DominanceInfo &domInfo) {
 512:   scf::ForOp outer = parent->loop;
```

- **EN:** Defines `fuseOneLevel`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `fuseOneLevel`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 514-518

```cpp
 514:   SmallVector<InnerLoop> innerLoops;
 515:   llvm::SetVector<Operation *> toHoist;
 516:   for (LoopNestNode *child : parent->children) {
 517:     scf::ForOp inner = child->loop;
 518:     assert(child->children.empty() && "fuseOneLevel runs leaf-to-root");
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 520-530

```cpp
 520:     // Check if the inner loop bounds are or can be made invariant to the outer
 521:     // loop. Check them all at once to avoid adding ops to `toHoist` if not
 522:     // necessary.
 523:     if (isOuterLoopInvariant(
 524:             domInfo, outer,
 525:             {inner.getLowerBound(), inner.getUpperBound(), inner.getStep()},
 526:             toHoist)) {
 527:       // Add this child to the list of loops to fuse.
 528:       innerLoops.push_back({child->loop, {}});
 529:       continue;
 530:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 532-541

```cpp
 532:     // Check if the loop bounds can be sliced.
 533:     llvm::SetVector<Operation *> slicedOps;
 534:     if (canSliceBounds(
 535:             domInfo, outer,
 536:             {inner.getLowerBound(), inner.getUpperBound(), inner.getStep()},
 537:             slicedOps)) {
 538:       innerLoops.push_back({child->loop, std::move(slicedOps)});
 539:       continue;
 540:     }
 541:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 543-547

```cpp
 543:   // From the perspective of the overall analysis, we can delete all the
 544:   // children of the current loop node. Child loops that cannot be fused are now
 545:   // treated opaquely by the rest of the analysis. This allows partial fusing of
 546:   // the constructed loop nest.
 547:   parent->children.clear();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 549-551

```cpp
 549:   // If there are no child loops to fuse, then there is nothing to do.
 550:   if (innerLoops.empty())
 551:     return;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 553-556

```cpp
 553:   // The transformation will definitely succeed on `childrenToFuse`. `toHoist`
 554:   // only contains the operations that must be hoisted for `childrenToFuse` to
 555:   // be fusible.
 556:   hoistOpsBefore(outer, toHoist);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 558-561

```cpp
 558:   // Determine the integer type to use for the length computations. Use an
 559:   // integer bitwidth twice the size of the largest integer, up to 64 bits, to
 560:   // avoid overflow.
 561:   unsigned intTyWidth = getIntTypeWidth(outer.getInductionVar().getType());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 563-572

```cpp
 563:   // Generate the computations of the fused loop bounds.
 564:   Location loc = outer.getLoc();
 565:   ImplicitLocOpBuilder b(loc, outer);
 566:   for (InnerLoop &loop : innerLoops) {
 567:     intTyWidth = std::max(intTyWidth,
 568:                           getIntTypeWidth(loop.op.getInductionVar().getType()));
 569:   }
 570:   auto intTy = b.getIntegerType(intTyWidth);
 571:   bool allInvariant = llvm::all_of(
 572:       innerLoops, [](InnerLoop &loop) { return loop.isOuterLoopInvariant(); });
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 574-584

```cpp
 574:   Value lenOuter = computeNumIters(b, outer);
 575:   SmallVector<Value> lenInners;
 576:   for (InnerLoop &loop : innerLoops) {
 577:     // len_jk = len(range(lbjk, ubjk, stepjk))
 578:     Value lenInner;
 579:     if (loop.isOuterLoopInvariant())
 580:       lenInner = castIntIfNecessary(b, computeNumIters(b, loop.op), intTy);
 581:     else
 582:       lenInner = createPoisonOrZero(b, intTy);
 583:     lenInners.push_back(lenInner);
 584:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 586-588

```cpp
 586:   auto intTyCst = [&](int64_t v) {
 587:     return arith::ConstantOp::create(b, IntegerAttr::get(intTy, v));
 588:   };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 590-599

```cpp
 590:   // inner_len = max(1, len_j0) + max(1, len_j1) + ... + max(1, len_jN) - N
 591:   unsigned N = innerLoops.size() - 1;
 592:   Value innerLen = intTyCst(0);
 593:   for (auto [loop, lenInner] : llvm::zip(innerLoops, lenInners)) {
 594:     if (!loop.isOuterLoopInvariant())
 595:       continue;
 596:     innerLen = arith::AddIOp::create(
 597:         b, innerLen, arith::MaxSIOp::create(b, intTyCst(1), lenInner));
 598:   }
 599:   innerLen = arith::SubIOp::create(b, innerLen, intTyCst(N));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 601-603

```cpp
 601:   // total_iters = len_i * inner_len
 602:   Value totalIters = arith::MulIOp::create(
 603:       b, castIntIfNecessary(b, lenOuter, intTy), innerLen);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 605-621

```cpp
 605:   // Generate a loop to compute the total number of iterations for inner loops
 606:   // whose bounds are not outer loop invariant.
 607:   IRMapping mapping;
 608:   auto peeledLen =
 609:       scf::ForOp::create(b, outer.getLowerBound(), outer.getUpperBound(),
 610:                          outer.getStep(), {totalIters});
 611:   totalIters = peeledLen.getRegionIterArg(0);
 612:   mapping.map(outer.getInductionVar(), peeledLen.getInductionVar());
 613:   b.setInsertionPointToStart(peeledLen.getBody());
 614:   for (InnerLoop &loop : innerLoops) {
 615:     if (loop.isOuterLoopInvariant())
 616:       continue;
 617:     // Cloned the sliced ops into the peeled loop.
 618:     for (Operation *op : topologicalSort(loop.slicedOps)) {
 619:       if (!mapping.contains(op))
 620:         b.clone(*op, mapping);
 621:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 622-633

```cpp
 622:     Value numIters =
 623:         computeNumIters(b, mapping.lookupOrDefault(loop.op.getLowerBound()),
 624:                         mapping.lookupOrDefault(loop.op.getUpperBound()),
 625:                         mapping.lookupOrDefault(loop.op.getStep()));
 626:     numIters = castIntIfNecessary(b, numIters, intTy);
 627:     // Accumulate into the total number of iterations.
 628:     numIters = arith::MaxSIOp::create(b, intTyCst(1), numIters);
 629:     totalIters = arith::AddIOp::create(b, totalIters, numIters);
 630:   }
 631:   scf::YieldOp::create(b, totalIters);
 632:   totalIters = peeledLen.getResults().front();
 633:   b.setInsertionPointAfter(peeledLen);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 635-651

```cpp
 635:   // The outputs of the prologue, each epilogue, and all inner loop bodies need
 636:   // to carried through the fused loop.
 637:   SmallVector<Logue> logues;
 638:   auto addLogue = [&](Block::iterator begin, Block::iterator end) {
 639:     logues.push_back(createLogueFrom({begin, end}, domInfo));
 640:   };
 641:   // prologue0
 642:   addLogue(outer.getBody()->begin(), innerLoops.front().op->getIterator());
 643:   // prologuek where 0 < k <= N
 644:   for (auto i : llvm::seq<unsigned>(0, innerLoops.size() - 1)) {
 645:     addLogue(std::next(innerLoops[i].op->getIterator()),
 646:              innerLoops[i + 1].op->getIterator());
 647:   }
 648:   // epilogue
 649:   addLogue(std::next(innerLoops.back().op->getIterator()),
 650:            // Don't include the outer loop yield.
 651:            std::prev(outer.getBody()->end()));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 653-660

```cpp
 653:   // We need iter args for:
 654:   // - The fused loop induction var
 655:   // - The outer loop induction var
 656:   // - The outer loop iter args
 657:   // - The induction vars for each inner loop
 658:   // - The outputs of each child loop
 659:   // - The outputs of each logue
 660:   SmallVector<Value> fusedInits;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 662-666

```cpp
 662:   // T = 0
 663:   fusedInits.push_back(intTyCst(0));
 664:   // i = lbi - stepi
 665:   fusedInits.push_back(
 666:       arith::SubIOp::create(b, outer.getLowerBound(), outer.getStep()));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 668-673

```cpp
 668:   unsigned outerArgsStartIdx = fusedInits.size();
 669:   llvm::append_range(fusedInits, outer.getInits());
 670:   unsigned lenInnersStartIdx = fusedInits.size();
 671:   llvm::append_range(fusedInits, lenInners);
 672:   unsigned innerLenStartIdx = fusedInits.size();
 673:   fusedInits.push_back(innerLen);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 675-690

```cpp
 675:   // Everything else is initialized to undef.
 676:   unsigned ivarStartIdx = fusedInits.size();
 677:   for (InnerLoop &loop : innerLoops) {
 678:     fusedInits.push_back(
 679:         createPoisonOrZero(b, loop.op.getInductionVar().getType()));
 680:   }
 681:   unsigned innerOutsStartIdx = fusedInits.size();
 682:   for (InnerLoop &loop : innerLoops) {
 683:     for (Type resultType : loop.op.getResultTypes())
 684:       fusedInits.push_back(createPoisonOrZero(b, resultType));
 685:   }
 686:   unsigned logueOutsStartIdx = fusedInits.size();
 687:   for (Logue &logue : llvm::drop_end(logues)) {
 688:     for (Type outputType : logue.getOutputTypes())
 689:       fusedInits.push_back(createPoisonOrZero(b, outputType));
 690:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 692-705

```cpp
 692:   // for _ in range(total_iters):
 693:   auto fused =
 694:       scf::ForOp::create(b, intTyCst(0), totalIters, intTyCst(1), fusedInits);
 695:   // Replace the outer loop args with the args in the fused loop args.
 696:   for (auto [arg, fusedArg] :
 697:        llvm::zip(outer.getRegionIterArgs(),
 698:                  fused.getRegionIterArgs().slice(outerArgsStartIdx))) {
 699:     arg.replaceAllUsesWith(fusedArg);
 700:   }
 701:   ValueRange lenInnersRange =
 702:       fused.getRegionIterArgs().slice(lenInnersStartIdx, lenInners.size());
 703:   for (auto [lenInner, lenInnerArg] : llvm::zip(lenInners, lenInnersRange))
 704:     lenInner = lenInnerArg;
 705:   b.setInsertionPointToStart(fused.getBody());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 707-710

```cpp
 707:   Value T = fused.getRegionIterArg(0);
 708:   // `i` is computed inside the first prologue.
 709:   Value curI = fused.getRegionIterArg(1);
 710:   Value i;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 712-728

```cpp
 712:   ArrayRef<BlockArgument> ivars = fused.getRegionIterArgs().slice(ivarStartIdx);
 713:   auto bodyOutsIt =
 714:       ValueRange(fused.getRegionIterArgs()).begin() + innerOutsStartIdx;
 715:   auto logueOutsIt =
 716:       ValueRange(fused.getRegionIterArgs()).begin() + logueOutsStartIdx;
 717:   SmallVector<scf::IfOp> prologueIfs, bodyIfs;
 718:   for (unsigned k = 0; k <= N; ++k) {
 719:     // if T == max(1, len_j0) + ... max(1, len_jk-1) - k
 720:     //   [[if k == 0]] i += stepi
 721:     //   prologuek(i)
 722:     //   jk = lbjk
 723:     Value innerStartT = intTyCst(0);
 724:     for (unsigned i = 0; i < k; ++i) {
 725:       innerStartT = arith::AddIOp::create(
 726:           b, innerStartT, arith::MaxSIOp::create(b, intTyCst(1), lenInners[i]));
 727:     }
 728:     innerStartT = arith::SubIOp::create(b, innerStartT, intTyCst(k));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 729-730

```cpp
 729:     Value prologueCond =
 730:         arith::CmpIOp::create(b, arith::CmpIPredicate::eq, T, innerStartT);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 732-735

```cpp
 732:     // The `scf.if` outputs will be `jk` and the outputs of prologuek. We also
 733:     // have to initialize the inner loop iter args.
 734:     scf::ForOp inner = innerLoops[k].op;
 735:     Logue &prologue = logues[k];
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 737-746

```cpp
 737:     SmallVector<Type> prologueOutTypes{inner.getInductionVar().getType()};
 738:     llvm::append_range(prologueOutTypes, prologue.getOutputTypes());
 739:     llvm::append_range(prologueOutTypes, inner.getInits().getTypes());
 740:     if (k == 0) {
 741:       prologueOutTypes.push_back(curI.getType());
 742:       prologueOutTypes.append(innerLoops.size(), intTy);
 743:       prologueOutTypes.push_back(innerLen.getType());
 744:     }
 745:     auto prologueIf = scf::IfOp::create(b, prologueOutTypes, prologueCond);
 746:     prologueIfs.push_back(prologueIf);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 748-750

```cpp
 748:     // Splice prologuek into the `then` region.
 749:     Block *thenBlock = b.createBlock(&prologueIf.getThenRegion());
 750:     prologue.moveBefore(thenBlock, thenBlock->end());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 752-757

```cpp
 752:     if (k == 0) {
 753:       // Increment `i` and replace its uses inside the prologue.
 754:       b.setInsertionPointToStart(thenBlock);
 755:       i = arith::AddIOp::create(b, curI, outer.getStep());
 756:       mlir::replaceAllUsesInRegionWith(outer.getInductionVar(), i,
 757:                                        prologueIf.getThenRegion());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 759-776

```cpp
 759:       // Compute the variant inner loop lengths.
 760:       IRMapping mapping;
 761:       for (auto [loop, lenInner] : llvm::zip(innerLoops, lenInners)) {
 762:         if (loop.isOuterLoopInvariant())
 763:           continue;
 764:         for (Operation *op : topologicalSort(loop.slicedOps)) {
 765:           if (!mapping.contains(op))
 766:             b.clone(*op, mapping);
 767:         }
 768:         lenInner =
 769:             computeNumIters(b, mapping.lookupOrDefault(loop.op.getLowerBound()),
 770:                             mapping.lookupOrDefault(loop.op.getUpperBound()),
 771:                             mapping.lookupOrDefault(loop.op.getStep()));
 772:         lenInner = castIntIfNecessary(b, lenInner, intTy);
 773:         innerLen = arith::AddIOp::create(
 774:             b, innerLen, arith::MaxSIOp::create(b, intTyCst(1), lenInner));
 775:       }
 776:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 778-789

```cpp
 778:     // Yield the initialized jk, the prologue outputs, and the initial values of
 779:     // the inner loop.
 780:     b.setInsertionPointToEnd(thenBlock);
 781:     SmallVector<Value> thenOuts{inner.getLowerBound()};
 782:     llvm::append_range(thenOuts, prologue.getOutputs());
 783:     llvm::append_range(thenOuts, inner.getInits());
 784:     if (k == 0) {
 785:       thenOuts.push_back(i);
 786:       llvm::append_range(thenOuts, lenInners);
 787:       thenOuts.push_back(innerLen);
 788:     }
 789:     scf::YieldOp::create(b, thenOuts);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 791-808

```cpp
 791:     // In the `else` region, just yield the last values of jk, the outputs, and
 792:     // the iter args.
 793:     b.createBlock(&prologueIf.getElseRegion());
 794:     Value lastJk = ivars[k];
 795:     unsigned numOuts = prologue.getNumOutputs();
 796:     SmallVector<Value> elseOuts{lastJk};
 797:     elseOuts.append(logueOutsIt, logueOutsIt + numOuts);
 798:     elseOuts.append(bodyOutsIt, bodyOutsIt + inner.getNumResults());
 799:     if (k == 0) {
 800:       elseOuts.push_back(curI);
 801:       llvm::append_range(elseOuts, lenInnersRange);
 802:       // Peephole the passthrough of `innerLen` since MLIR will not optimize it
 803:       // away for us.
 804:       elseOuts.push_back(
 805:           allInvariant ? innerLen : fused.getRegionIterArg(innerLenStartIdx));
 806:     }
 807:     logueOutsIt += numOuts;
 808:     scf::YieldOp::create(b, elseOuts);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 810-827

```cpp
 810:     // The results of the `scf.if` become the values of jk and the prologue
 811:     // outputs for the rest of the fused loop.
 812:     Value jk = prologueIf.getResult(0);
 813:     ValueRange prologueOuts = prologueIf.getResults().slice(1, numOuts);
 814:     ValueRange prologueInits =
 815:         prologueIf.getResults().slice(1 + numOuts, inner.getNumResults());
 816:     inner.getInductionVar().replaceAllUsesWith(jk);
 817:     prologue.replaceAllUsesWith(prologueOuts, prologueIf.getThenRegion());
 818:     for (auto [init, iterArg] :
 819:          llvm::zip(prologueInits, inner.getRegionIterArgs()))
 820:       iterArg.replaceAllUsesWith(init);
 821:     // Replace uses of `i` elsewhere with the prologue result.
 822:     if (k == 0) {
 823:       ValueRange results = prologueIf.getResults();
 824:       i = results.drop_back(1 + lenInners.size()).back();
 825:       lenInners = results.drop_back().take_back(lenInners.size());
 826:       innerLen = results.back();
 827:       outer.getInductionVar().replaceAllUsesWith(i);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 828-828

```cpp
 828:     }
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 830-842

```cpp
 830:     // if  T >= max(1, len_j0) + max(1, len_j1) + ... + max(1, len_jk-1) - k
 831:     // and T <  max(1, len_j0) + max(1, len_j1) + ... + max(1, len_jk-1) - k +
 832:     //          len_jk
 833:     //   bodyk(i, jk)
 834:     //   jk += stepjk
 835:     b.setInsertionPointAfter(prologueIf);
 836:     Value innerEndT = arith::AddIOp::create(
 837:         b, innerStartT, castIntIfNecessary(b, lenInners[k], intTy));
 838:     Value ge =
 839:         arith::CmpIOp::create(b, arith::CmpIPredicate::sge, T, innerStartT);
 840:     Value lt =
 841:         arith::CmpIOp::create(b, arith::CmpIPredicate::slt, T, innerEndT);
 842:     Value bodyCond = arith::AndIOp::create(b, ge, lt);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 844-848

```cpp
 844:     // The outputs will be the outputs of the inner loop body and the next jk.
 845:     SmallVector<Type> bodyOutTypes{jk.getType()};
 846:     llvm::append_range(bodyOutTypes, inner->getResultTypes());
 847:     auto bodyIf = scf::IfOp::create(b, bodyOutTypes, bodyCond);
 848:     bodyIfs.push_back(bodyIf);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 850-856

```cpp
 850:     // Splice bodyk into the `then` region.
 851:     inner.getBody()->eraseArguments([](Value arg) { return true; });
 852:     bodyIf.getThenRegion().takeBody(inner.getBodyRegion());
 853:     auto yield = getYield(bodyIf.getThenRegion());
 854:     b.setInsertionPoint(yield);
 855:     Value nextJk = arith::AddIOp::create(b, jk, inner.getStep());
 856:     yield->insertOperands(0, nextJk);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 858-863

```cpp
 858:     // The `else` region just forwards the values.
 859:     b.createBlock(&bodyIf.getElseRegion());
 860:     SmallVector<Value> bodyForwardedOuts{jk};
 861:     bodyForwardedOuts.append(bodyOutsIt, bodyOutsIt + inner.getNumResults());
 862:     bodyOutsIt += inner->getNumResults();
 863:     scf::YieldOp::create(b, bodyForwardedOuts);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 865-868

```cpp
 865:     // Now we can replace the results of the inner loop with the outputs of the
 866:     // body if.
 867:     inner.replaceAllUsesWith(
 868:         bodyIf.getResults().slice(1, inner.getNumResults()));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 870-876

```cpp
 870:     // If the inner loop must execute, then its body does not have to be wrapped
 871:     // in a conditional.
 872:     if (inner->hasAttr(kMustExecuteAttrName)) {
 873:       b.setInsertionPoint(bodyIf);
 874:       bodyIf.getConditionMutable().assign(
 875:           arith::ConstantOp::create(b, b.getBoolAttr(true)));
 876:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 878-880

```cpp
 878:     // Move the insertion point for the next iteration.
 879:     b.setInsertionPointAfter(bodyIf);
 880:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 882-884

```cpp
 882:   // if T == len_j0 + len_j1 + ... + len_jN - N - 1:
 883:   //   epilogue(i)
 884:   Logue &epilogue = logues.back();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 886-896

```cpp
 886:   // The only possible use of an epilogue output is the yield.
 887:   auto outerYield = cast<scf::YieldOp>(outer.getBody()->getTerminator());
 888:   SmallVector<Value> usedIterArgs;
 889:   for (Value output : epilogue.getOutputs()) {
 890:     for (OpOperand &use : output.getUses()) {
 891:       if (use.getOwner() == outerYield) {
 892:         usedIterArgs.push_back(fused.getRegionIterArgs().drop_front(
 893:             outerArgsStartIdx)[use.getOperandNumber()]);
 894:       }
 895:     }
 896:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 898-902

```cpp
 898:   auto epilogueCond =
 899:       arith::CmpIOp::create(b, arith::CmpIPredicate::eq, T,
 900:                             arith::SubIOp::create(b, innerLen, intTyCst(1)));
 901:   auto epilogueIf =
 902:       scf::IfOp::create(b, epilogue.getOutputTypes(), epilogueCond);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 904-905

```cpp
 904:   Block *thenBlock = b.createBlock(&epilogueIf.getThenRegion());
 905:   epilogue.moveBefore(thenBlock, thenBlock->end());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 907-912

```cpp
 907:   b.setInsertionPointToEnd(thenBlock);
 908:   scf::YieldOp::create(b, epilogue.getOutputs());
 909:   b.createBlock(&epilogueIf.getElseRegion());
 910:   scf::YieldOp::create(b, usedIterArgs);
 911:   epilogue.replaceAllUsesWith(epilogueIf.getResults(),
 912:                               epilogueIf.getThenRegion());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 914-920

```cpp
 914:   // T = 0 if T == (inner_len - 1) else T + 1
 915:   b.setInsertionPointToEnd(fused.getBody());
 916:   Value nextT = arith::AddIOp::create(b, T, intTyCst(1));
 917:   Value rollover =
 918:       arith::CmpIOp::create(b, arith::CmpIPredicate::eq, T,
 919:                             arith::SubIOp::create(b, innerLen, intTyCst(1)));
 920:   T = arith::SelectOp::create(b, rollover, intTyCst(0), nextT);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 922-936

```cpp
 922:   // Finally, create the yield of the fused loop.
 923:   SmallVector<Value> outerOuts{T, i};
 924:   llvm::append_range(outerOuts, outerYield.getOperands());
 925:   llvm::append_range(outerOuts, lenInners);
 926:   outerOuts.push_back(innerLen);
 927:   for (scf::IfOp bodyIf : bodyIfs)
 928:     outerOuts.push_back(/*jk=*/bodyIf.getResult(0));
 929:   for (auto [bodyIf, loop] : llvm::zip(bodyIfs, innerLoops)) {
 930:     llvm::append_range(outerOuts,
 931:                        bodyIf.getResults().slice(1, loop.op.getNumResults()));
 932:   }
 933:   for (auto [logueIf, logue] : llvm::zip(prologueIfs, llvm::drop_end(logues))) {
 934:     llvm::append_range(outerOuts,
 935:                        logueIf.getResults().slice(1, logue.getNumOutputs()));
 936:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 938-940

```cpp
 938:   scf::YieldOp::create(b, outerOuts);
 939:   outer.replaceAllUsesWith(
 940:       fused.getResults().slice(outerArgsStartIdx, outer.getNumResults()));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 942-954

```cpp
 942:   // Reduce dependencies across inner loops by hoisting the initialization of
 943:   // inner loop iter args to the outer loop when possible, and then placing the
 944:   // reset of these values in the epilogue.
 945:   auto fusedInitsIt = fused.getInitsMutable().begin() + innerOutsStartIdx;
 946:   auto fusedArgsIt = fused.getRegionIterArgs().begin() + innerOutsStartIdx;
 947:   auto fusedYieldIt = getYield(fused.getBodyRegion())->getOpOperands().begin() +
 948:                       innerOutsStartIdx;
 949:   SmallVector<OpOperand *> yieldsToUpdate;
 950:   SmallVector<Value> reset, forwarded;
 951:   for (auto [loop, ifOp, bodyIf, prologue] :
 952:        llvm::zip(innerLoops, prologueIfs, bodyIfs, logues)) {
 953:     unsigned numResults = loop.op.getNumResults();
 954:     unsigned prologueSkip = 1 + prologue.getNumOutputs();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 956-970

```cpp
 956:     llvm::BitVector removeIndices(prologueSkip + numResults);
 957:     SmallVector<Value> replaceWith;
 958:     for (auto [i, init] : llvm::enumerate(loop.op.getInits())) {
 959:       if (init.getParentRegion() == &fused.getBodyRegion())
 960:         continue;
 961:       // Initialize this in the outer loop.
 962:       fusedInitsIt[i].assign(init);
 963:       replaceWith.push_back(fusedArgsIt[i]);
 964:       removeIndices.set(prologueSkip + i);
 965:       yieldsToUpdate.push_back(&fusedYieldIt[i]);
 966:       forwarded.push_back(bodyIf.getResult(1 + i));
 967:       reset.push_back(init);
 968:     }
 969:     // Remove the initializers in the corresponding prologue.
 970:     eraseIfResults(b, ifOp, removeIndices, replaceWith);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 972-989

```cpp
 972:     fusedInitsIt += numResults;
 973:     fusedArgsIt += numResults;
 974:     fusedYieldIt += numResults;
 975:   }
 976:   if (!yieldsToUpdate.empty()) {
 977:     MutableOperandRange(getYield(epilogueIf.getThenRegion())).append(reset);
 978:     MutableOperandRange(getYield(epilogueIf.getElseRegion())).append(forwarded);
 979:     b.setInsertionPoint(epilogueIf);
 980:     TypeRange newTypes = getYield(epilogueIf.getThenRegion()).getOperandTypes();
 981:     auto newIf = scf::IfOp::create(b, newTypes, epilogueIf.getCondition());
 982:     newIf.getThenRegion().takeBody(epilogueIf.getThenRegion());
 983:     newIf.getElseRegion().takeBody(epilogueIf.getElseRegion());
 984:     epilogueIf.replaceAllUsesWith(
 985:         newIf.getResults().take_front(epilogueIf.getNumResults()));
 986:     ResultRange newResults =
 987:         newIf.getResults().drop_front(epilogueIf.getNumResults());
 988:     for (auto [i, yieldOperand] : llvm::enumerate(yieldsToUpdate))
 989:       yieldOperand->set(newResults[i]);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 990-991

```cpp
 990:     epilogueIf.erase();
 991:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 993-998

```cpp
 993:   // Propagate warp specialization flags.
 994:   if (outer->hasAttr(kWarpSpecializeAttrName) ||
 995:       llvm::any_of(innerLoops, [](InnerLoop &loop) {
 996:         return loop.op->hasAttr(kWarpSpecializeAttrName);
 997:       }))
 998:     fused->setAttr(kWarpSpecializeAttrName, b.getUnitAttr());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1000-1006

```cpp
1000:   // Propagate the `tt.disallow_acc_multi_buffer` attribute to the parent loop.
1001:   bool disallowAccMultiBuffer = getDisallowAccMultiBuffer(outer);
1002:   for (InnerLoop &loop : innerLoops) {
1003:     disallowAccMultiBuffer |= getDisallowAccMultiBuffer(loop.op);
1004:   }
1005:   if (disallowAccMultiBuffer)
1006:     fused->setAttr(kDisallowAccMultiBufferAttrName, b.getUnitAttr());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1008-1023

```cpp
1008:   // Update the parent's loop to the fused loop. Set the new stage count to the
1009:   // max stage count of the inner loops.
1010:   int numStages = 1;
1011:   if (auto stageAttr = outer->getAttrOfType<IntegerAttr>(kNumStagesAttrName))
1012:     numStages = stageAttr.getInt();
1013:   for (InnerLoop &loop : innerLoops) {
1014:     if (auto stageAttr =
1015:             loop.op->getAttrOfType<IntegerAttr>(kNumStagesAttrName))
1016:       numStages = std::max<int>(numStages, stageAttr.getInt());
1017:     loop.op.erase();
1018:   }
1019:   outer.erase();
1020:   parent->loop = fused;
1021:   if (numStages > 1)
1022:     fused->setAttr(kNumStagesAttrName, b.getI32IntegerAttr(numStages));
1023: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1025-1027

```cpp
1025: //===----------------------------------------------------------------------===//
1026: // flattenLoopNest
1027: //===----------------------------------------------------------------------===//
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 1029-1035

```cpp
1029: // Completely flatten a loop nest by recursively fusing loops in a post-order
1030: // traversal with `fuseOneLevel`.
1031: static void flattenLoopNest(LoopNestNode *node, mlir::DominanceInfo &domInfo) {
1032:   for (LoopNestNode *child : node->children)
1033:     flattenLoopNest(child, domInfo);
1034:   fuseOneLevel(node, domInfo);
1035: }
```

- **EN:** Defines `flattenLoopNest`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `flattenLoopNest`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1037-1039

```cpp
1037: //===----------------------------------------------------------------------===//
1038: // Pass Implementation
1039: //===----------------------------------------------------------------------===//
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 1041-1045

```cpp
1041: // Fuse simple loop nests with a single outer and inner loop, and where the
1042: // inner loop has a `tt.dot` operation.
1043: static bool shouldFuse(const LoopNest &nest) {
1044:   if (nest.root->loop->hasAttr(kAlwaysFuseAttrName))
1045:     return true;
```

- **EN:** Defines `shouldFuse`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `shouldFuse`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1047-1050

```cpp
1047:   // Only fuse simple loop nests.
1048:   return nest.nodes.size() == 2 && nest.root->children.size() == 1 &&
1049:          nest.root->loop->hasAttr(kFlattenAttr);
1050: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1052-1069

```cpp
1052: // This function identifies a subgraph of cheap ops that can be sunk between two
1053: // regions in the loop nest and moves them, reducing their liveranges.
1054: static void sinkOps(Region &limit, Block *sinkBlock, Block::iterator sinkBefore,
1055:                     llvm::iterator_range<Block::iterator> prologue,
1056:                     function_ref<bool(Operation *)> inSinkRegion) {
1057:   llvm::SetVector<Operation *> sunkOps;
1058:   auto canBeSunk = [&](Operation &op) -> std::pair<bool, bool> {
1059:     if (!isPure(&op) || isa<DotOpInterface>(op))
1060:       return {false, false};
1061:     // An op can be sunk if all its users are inside the inner loop or are
1062:     // marked for sinking.
1063:     bool isRoot = true;
1064:     for (Operation *user : op.getUsers()) {
1065:       if (inSinkRegion(user))
1066:         continue;
1067:       isRoot = false;
1068:       if (sunkOps.contains(user))
1069:         continue;
```

- **EN:** Defines `sinkOps`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `sinkOps`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1070-1073

```cpp
1070:       return {false, false};
1071:     }
1072:     return {true, isRoot};
1073:   };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1075-1085

```cpp
1075:   // Find the subgraph of operations that can be sunk.
1076:   SmallVector<Operation *> roots;
1077:   for (Operation &op : llvm::reverse(prologue)) {
1078:     auto [canSink, isRoot] = canBeSunk(op);
1079:     if (canSink)
1080:       sunkOps.insert(&op);
1081:     if (isRoot)
1082:       roots.push_back(&op);
1083:   }
1084:   if (sunkOps.empty())
1085:     return;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 1087-1088

```cpp
1087:   hoistOpsBefore(sinkBlock, sinkBefore, sunkOps);
1088: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1090-1100

```cpp
1090: // Sink ops from the prologue into the epilogue when possible.
1091: static void optimizeEpilogueDependencies(scf::ForOp outerLoop,
1092:                                          scf::ForOp innerLoop,
1093:                                          mlir::DominanceInfo &domInfo) {
1094:   auto inEpilogue = [&](Operation *op) {
1095:     return domInfo.properlyDominates(innerLoop, op, /*enclosingOpOk=*/false);
1096:   };
1097:   Region &limit = outerLoop.getBodyRegion();
1098:   sinkOps(limit, outerLoop.getBody(), std::next(innerLoop->getIterator()),
1099:           {outerLoop.getBody()->begin(), innerLoop->getIterator()}, inEpilogue);
1100: }
```

- **EN:** Defines `optimizeEpilogueDependencies`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `optimizeEpilogueDependencies`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1102-1114

```cpp
1102: // Crudely match llvm.assume(ub > lb) or llvm.assume(lb < ub).
1103: static LogicalResult matchPositiveTripCount(scf::ForOp loop) {
1104:   for (Operation *user : loop.getUpperBound().getUsers()) {
1105:     if (auto cmp = dyn_cast<arith::CmpIOp>(user)) {
1106:       if (llvm::none_of(cmp->getUsers(),
1107:                         [](Operation *op) { return isa<LLVM::AssumeOp>(op); }))
1108:         continue;
1109:       if (cmp.getPredicate() == (loop.getUnsignedCmp()
1110:                                      ? arith::CmpIPredicate::ugt
1111:                                      : arith::CmpIPredicate::sgt) &&
1112:           cmp.getLhs() == loop.getUpperBound() &&
1113:           cmp.getRhs() == loop.getLowerBound())
1114:         return success();
```

- **EN:** Defines `matchPositiveTripCount`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `matchPositiveTripCount`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1115-1124

```cpp
1115:       if (cmp.getPredicate() == (loop.getUnsignedCmp()
1116:                                      ? arith::CmpIPredicate::ult
1117:                                      : arith::CmpIPredicate::slt) &&
1118:           cmp.getLhs() == loop.getLowerBound() &&
1119:           cmp.getRhs() == loop.getUpperBound())
1120:         return success();
1121:     }
1122:   }
1123:   return failure();
1124: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. The code uses `LogicalResult` to stay conservative when preconditions are not met.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 代码通过 `LogicalResult` 在前提不满足时保持保守处理。
### Lines 1126-1134

```cpp
1126: // Speculate the length of the inner loop such that the loop is known to execute
1127: // at least once. This way, the inner loop body does not have to be placed
1128: // inside a conditional in the fused loop, which interacts better with the
1129: // pipeliner.
1130: static LogicalResult speculateInnerLoopLength(scf::ForOp outerLoop,
1131:                                               scf::ForOp innerLoop,
1132:                                               mlir::DominanceInfo &domInfo) {
1133:   Location loc = innerLoop.getLoc();
1134:   ImplicitLocOpBuilder b(loc, outerLoop);
```

- **EN:** Defines `speculateInnerLoopLength`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `speculateInnerLoopLength`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1136-1140

```cpp
1136:   // Check if the inner loop is known to execute at least once.
1137:   if (succeeded(matchPositiveTripCount(innerLoop))) {
1138:     innerLoop->setAttr(kMustExecuteAttrName, b.getUnitAttr());
1139:     return success();
1140:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1142-1149

```cpp
1142:   // The inner loop bounds must be outer-loop invariant to speculate from
1143:   // outside the loop nest.
1144:   llvm::SetVector<Operation *> toHoist;
1145:   if (!isOuterLoopInvariant(domInfo, outerLoop,
1146:                             {innerLoop.getLowerBound(),
1147:                              innerLoop.getUpperBound(), innerLoop.getStep()},
1148:                             toHoist))
1149:     return failure();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1151-1152

```cpp
1151:   // Hoist the inner loop bounds computations if necessary.
1152:   hoistOpsBefore(outerLoop, toHoist);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1154-1155

```cpp
1154:   // Mark the inner loop.
1155:   innerLoop->setAttr(kMustExecuteAttrName, b.getUnitAttr());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1157-1163

```cpp
1157:   // Speculate on whether the length of the inner loop is zero.
1158:   Value lenInner = computeNumIters(b, innerLoop);
1159:   auto zeroAttr = IntegerAttr::get(lenInner.getType(), 0);
1160:   Value innerLoopEmpty =
1161:       arith::CmpIOp::create(b, arith::CmpIPredicate::eq, lenInner,
1162:                             arith::ConstantOp::create(b, zeroAttr));
1163:   auto ifOp = scf::IfOp::create(b, outerLoop.getResultTypes(), innerLoopEmpty);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1165-1173

```cpp
1165:   // In the `then` branch, the inner loop does not execute. Clone the loop nest
1166:   // into it and remove the inner loop.
1167:   mlir::IRMapping map;
1168:   b.createBlock(&ifOp.getThenRegion());
1169:   auto newLoop = cast<scf::ForOp>(b.clone(*outerLoop, map));
1170:   scf::YieldOp::create(b, newLoop.getResults());
1171:   auto newInnerLoop = cast<scf::ForOp>(map.lookup(innerLoop));
1172:   newInnerLoop.replaceAllUsesWith(newInnerLoop.getInits());
1173:   newInnerLoop.erase();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1175-1176

```cpp
1175:   // Clear up the warp specialization attributes for the specialized loop.
1176:   newLoop->removeAttr(kWarpSpecializeAttrName);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1178-1183

```cpp
1178:   // Move the loop nest into the `else` branch.
1179:   outerLoop.replaceAllUsesWith(ifOp.getResults());
1180:   b.createBlock(&ifOp.getElseRegion());
1181:   outerLoop->remove();
1182:   b.insert(outerLoop);
1183:   scf::YieldOp::create(b, outerLoop.getResults());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1185-1186

```cpp
1185:   return success();
1186: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1188-1190

```cpp
1188: static LogicalResult preprocessLoopNest(const LoopNest &nest,
1189:                                         mlir::DominanceInfo &domInfo) {
1190:   assert(nest.nodes.size() == 2 && nest.root->children.size() == 1);
```

- **EN:** Defines `preprocessLoopNest`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `preprocessLoopNest`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 1192-1193

```cpp
1192:   scf::ForOp &outerLoop = nest.root->loop;
1193:   scf::ForOp &innerLoop = nest.root->children.front()->loop;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1195-1198

```cpp
1195:   moveLoopInvariantCode(outerLoop);
1196:   optimizeEpilogueDependencies(outerLoop, innerLoop, domInfo);
1197:   return speculateInnerLoopLength(outerLoop, innerLoop, domInfo);
1198: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1200-1201

```cpp
1200: void FuseNestedLoopsPass::runOnOperation() {
1201:   auto &domInfo = getAnalysis<DominanceInfo>();
```

- **EN:** Defines `FuseNestedLoopsPass::runOnOperation`, the pass entry point. It gathers analysis information and applies the file's transformation logic to the current operation.
- **CN:** 这里定义 `FuseNestedLoopsPass::runOnOperation`，即 pass 的入口函数。它会收集分析信息，并把本文件的变换逻辑应用到当前操作上。
### Lines 1203-1215

```cpp
1203:   for (auto func : getOperation().getOps<FuncOp>()) {
1204:     SmallVector<LoopNest> nests;
1205:     findLoopNests(func, nests);
1206:     for (LoopNest &nest : nests) {
1207:       if (!shouldFuse(nest))
1208:         continue;
1209:       if (!nest.root->loop->hasAttr(kAlwaysFuseAttrName) &&
1210:           failed(preprocessLoopNest(nest, domInfo)))
1211:         continue;
1212:       flattenLoopNest(nest.root, domInfo);
1213:     }
1214:   }
1215: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 1217-1219

```cpp
1217: } // namespace gpu
1218: } // namespace triton
1219: } // namespace mlir
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。

## Key Concepts / 关键概念
- **EN:** The main concern is pass-driven transformation around fuse nested loops.
  **CN:** 核心关注点是围绕 Fuse Nested Loops 的 pass 驱动变换。
- **EN:** Tensor shape and element-type reasoning is central to the implementation.
  **CN:** 张量形状与元素类型推导是实现中的核心内容。

## Dependencies / 依赖关系
- **Internal Triton headers / Triton 内部头文件:** `triton/Dialect/Triton/IR/Dialect.h`, `triton/Dialect/TritonGPU/Transforms/Passes.h`, `triton/Dialect/TritonGPU/Transforms/PipeliningUtility.h`, `triton/Dialect/TritonGPU/Transforms/Passes.h.inc`
- **MLIR headers / MLIR 头文件:** `mlir/Analysis/TopologicalSortUtils.h`, `mlir/Dialect/LLVMIR/LLVMDialect.h`, `mlir/Dialect/UB/IR/UBOps.h`, `mlir/IR/Dominance.h`, `mlir/IR/ImplicitLocOpBuilder.h`, `mlir/Interfaces/SideEffectInterfaces.h`, ... (+2 more)
- **LLVM headers / LLVM 头文件:** `llvm/ADT/STLExtras.h`, `llvm/Support/Debug.h`
- **Standard/library headers / 标准或通用库头文件:** `queue`
- **Generated fragments / 生成片段:** `triton/Dialect/TritonGPU/Transforms/Passes.h.inc`
- **Primary APIs used / 主要 API:** `RankedTensorType`, `OpBuilder`
- **Pipeline role / 流程角色:** This file participates in Triton pass pipelines and usually expects upstream analyses or dialect invariants to have prepared the IR. / 本文件参与 Triton 的 pass 流程，通常依赖上游分析或方言不变量先把 IR 准备好。
