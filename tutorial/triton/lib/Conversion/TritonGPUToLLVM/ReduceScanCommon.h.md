# ReduceScanCommon.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `lib/Conversion/TritonGPUToLLVM/ReduceScanCommon.h`
- **Purpose / 作用:** **EN:** Lowers TritonGPU constructs related to Reduce Scan Common into LLVM-compatible IR and rewrite patterns. **CN:** 把与 Reduce Scan Common 相关的 TritonGPU 构造降级为 LLVM 兼容 IR 与重写模式。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2

```cpp
   1: #ifndef TRITON_CONVERSION_TRITONGPU_TO_LLVM_REDUCESCANCOMMON_H
   2: #define TRITON_CONVERSION_TRITONGPU_TO_LLVM_REDUCESCANCOMMON_H
```

- **EN:** Defines or closes a file guard so this header/TableGen fragment is included only once during compilation.
- **CN:** 这里定义或关闭文件保护宏，确保该头文件/TableGen 片段在编译过程中只被包含一次。
### Lines 4-9

```cpp
   4: // TODO: refactor so that it doesn't fail if Allocation.h
   5: // is included after utility.h (due to conflict in `store` macro
   6: // and <atomic>
   7: #include "mlir/Dialect/ControlFlow/IR/ControlFlowOps.h"
   8: #include "mlir/Dialect/LLVMIR/LLVMDialect.h"
   9: #include "mlir/Transforms/DialectConversion.h"
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 11-15

```cpp
  11: //
  12: #include "mlir/IR/TypeUtilities.h"
  13: #include "triton/Conversion/TritonGPUToLLVM/Utility.h"
  14: #include <iterator>
  15: #include <type_traits>
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 17-17

```cpp
  17: #define DEBUG_TYPE "ttgpu_to_llvm"
```

- **EN:** Defines debug logging helpers and compile-time tags used when tracing this pass or utility at runtime.
- **CN:** 这里定义调试日志辅助宏和编译期标签，用于在运行时跟踪该 pass 或工具。
### Lines 19-20

```cpp
  19: using namespace mlir;
  20: using namespace mlir::triton;
```

- **EN:** Introduces namespace aliases/imports (`mlir`, `mlir::triton`) so the rest of the file can use MLIR/Triton symbols without repetitive qualification.
- **CN:** 这里通过命名空间导入（`mlir`, `mlir::triton`）减少后续代码中的重复限定，使 MLIR/Triton 符号使用更简洁。
### Lines 22-24

```cpp
  22: namespace mlir::triton {
  23: class ReduceOp;
  24: class ScanOp;
```

- **EN:** Defines `ReduceOp`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `ReduceOp`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 26-32

```cpp
  26: inline SmallVector<Value>
  27: inlineCombineBlock(ConversionPatternRewriter &rewriter, Block &combineBlock,
  28:                    Block *insertionBlock, Block::iterator insertionPoint,
  29:                    ValueRange combineArgs) {
  30:   auto returnOp = combineBlock.getTerminator();
  31:   rewriter.inlineBlockBefore(&combineBlock, insertionBlock, insertionPoint,
  32:                              combineArgs);
```

- **EN:** Defines `inlineCombineBlock`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `inlineCombineBlock`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 34-34

```cpp
  34:   auto results = SmallVector<Value>(returnOp->getOperands());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 36-39

```cpp
  36:   // Delete the terminator, which is no longer used
  37:   rewriter.eraseOp(returnOp);
  38:   return results;
  39: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. When the lowered form has no direct result, the original op is erased.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 当降级后的形式不再需要直接结果时，原始操作会被删除。
### Lines 41-49

```cpp
  41: inline SmallVector<Value> applyCombineOp(Location loc,
  42:                                          ConversionPatternRewriter &rewriter,
  43:                                          Region &combineOp, ValueRange acc,
  44:                                          ValueRange cur, Value pred = {}) {
  45:   // Allows for passing an uninitialized acc and use cur as the neutral element
  46:   if (acc.size() == 0) {
  47:     return cur;
  48:   }
  49:   assert(cur.size() == acc.size());
```

- **EN:** Defines `applyCombineOp`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `applyCombineOp`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 51-53

```cpp
  51:   // Create a new copy of the combine block, and try to speculatively inline it
  52:   Block *currentBlock = rewriter.getBlock();
  53:   Region &parent = *currentBlock->getParent();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 55-57

```cpp
  55:   rewriter.cloneRegionBefore(combineOp, parent,
  56:                              std::next(currentBlock->getIterator()));
  57:   Block &newCombine = *currentBlock->getNextNode();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 59-63

```cpp
  59:   llvm::SmallVector<Value> combineArgs(2 * acc.size());
  60:   for (unsigned i = 0; i < acc.size(); ++i) {
  61:     combineArgs[i] = acc[i];
  62:     combineArgs[acc.size() + i] = cur[i];
  63:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 65-67

```cpp
  65:   auto isRegionSpeculatable =
  66:       std::all_of(newCombine.begin(), newCombine.end(),
  67:                   [](auto &op) { return isSpeculatable(&op); });
```

- **EN:** Defines `std::all_of`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `std::all_of`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 69-73

```cpp
  69:   if (!pred || isRegionSpeculatable) {
  70:     // Fast path, region has no side effects so we can unconditionally execute
  71:     return inlineCombineBlock(rewriter, newCombine, currentBlock,
  72:                               rewriter.getInsertionPoint(), combineArgs);
  73:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 75-86

```cpp
  75:   // Slow case, create an if to only execute region when pred is true
  76:   // #currentBlock
  77:   // if (pred) {
  78:   //   #newCombine
  79:   //   results = combineOp(cur, acc)
  80:   //   yield results
  81:   // } else {
  82:   //    yield undef
  83:   // }
  84:   // #thenBlock
  85:   Block *thenBlock =
  86:       rewriter.splitBlock(currentBlock, rewriter.getInsertionPoint());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 88-89

```cpp
  88:   auto returnOp = newCombine.getTerminator();
  89:   auto results = SmallVector<Value>(returnOp->getOperands());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 91-101

```cpp
  91:   rewriter.setInsertionPointToEnd(currentBlock);
  92:   SmallVector<Value> thenBlockArgs;
  93:   thenBlockArgs.reserve(results.size());
  94:   for (auto result : results) {
  95:     auto ty = result.getType();
  96:     auto undef = LLVM::UndefOp::create(rewriter, loc, ty);
  97:     thenBlockArgs.push_back(undef);
  98:     thenBlock->addArgument(ty, loc);
  99:   }
 100:   LLVM::CondBrOp::create(rewriter, loc, pred, &newCombine, combineArgs,
 101:                          thenBlock, thenBlockArgs);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 103-108

```cpp
 103:   // Split a block after the call.
 104:   rewriter.setInsertionPointToEnd(&newCombine);
 105:   rewriter.replaceOpWithNewOp<LLVM::BrOp>(returnOp, results, thenBlock);
 106:   rewriter.setInsertionPointToStart(thenBlock);
 107:   return SmallVector<Value>(thenBlock->getArguments());
 108: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. It finishes by replacing the original operation with the lowered form.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 它会在转换成功后用降级后的形式替换原始操作。
### Lines 110-110

```cpp
 110: } // namespace mlir::triton
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 112-118

```cpp
 112: template <typename SourceOp>
 113: class ConvertTritonGPUReduceScanToLLVMPattern
 114:     : public ConvertOpToLLVMPattern<SourceOp> {
 115: public:
 116:   // Make sure the class is only instantiated with Reduce and Scan
 117:   static_assert(std::is_same_v<SourceOp, ReduceOp> ||
 118:                 std::is_same_v<SourceOp, ScanOp>);
```

- **EN:** Defines `ConvertTritonGPUReduceScanToLLVMPattern`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `ConvertTritonGPUReduceScanToLLVMPattern`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 120-121

```cpp
 120:   using ConvertOpToLLVMPattern<SourceOp>::getTypeConverter;
 121:   using ConvertOpToLLVMPattern<SourceOp>::ConvertOpToLLVMPattern;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Type mapping is delegated to the LLVM type converter or related conversion helpers.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 类型映射委托给 LLVM 类型转换器或相关转换辅助函数。
### Lines 123-127

```cpp
 123:   // Return the pointee type of the shared memory pointer for operand i.
 124:   Type getElementType(SourceOp op, int i) const {
 125:     auto ty = op.getInputTypes()[i].getElementType();
 126:     return getTypeConverter()->convertType(ty);
 127:   }
```

- **EN:** Defines accessor/helper `getElementType` that exposes or updates operation state in a compact, reusable way. Type mapping is delegated to the LLVM type converter or related conversion helpers.
- **CN:** 这里定义访问器/辅助函数 `getElementType`，以紧凑且可复用的方式读取或更新操作状态。 类型映射委托给 LLVM 类型转换器或相关转换辅助函数。
### Lines 129-138

```cpp
 129:   // Helper to compute the smem bases in both reductions and scans
 130:   SmallVector<Value> getSmemBases(SourceOp op, unsigned elems,
 131:                                   ConversionPatternRewriter &rewriter,
 132:                                   const TargetInfoBase &targetInfo) const {
 133:     auto loc = op.getLoc();
 134:     auto b = TritonLLVMOpBuilder(loc, rewriter);
 135:     // indices will store the index of the op operands in descending order
 136:     // of their bitwidths
 137:     std::vector<unsigned> indices(op.getNumOperands());
 138:     std::iota(indices.begin(), indices.end(), 0);
```

- **EN:** Defines accessor/helper `getSmemBases` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getSmemBases`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 140-157

```cpp
 140:     std::sort(indices.begin(), indices.end(), [&](unsigned i, unsigned j) {
 141:       return op.getElementTypes()[i].getIntOrFloatBitWidth() >
 142:              op.getElementTypes()[j].getIntOrFloatBitWidth();
 143:     });
 144:     // Assign base index to each operand in their order in indices
 145:     std::map<unsigned, Value> indexToBase;
 146:     auto basePtr =
 147:         LLVM::getSharedMemoryBase(loc, rewriter, targetInfo, op.getOperation());
 148:     indexToBase[indices[0]] = basePtr;
 149:     for (unsigned i = 1; i < op.getNumOperands(); ++i) {
 150:       indexToBase[indices[i]] =
 151:           b.gep(basePtr.getType(), getElementType(op, indices[i - 1]),
 152:                 indexToBase[indices[i - 1]], b.i32_val(elems));
 153:     }
 154:     // smemBases[k] is the base pointer for the k-th operand
 155:     SmallVector<Value> smemBases(op.getNumOperands());
 156:     for (unsigned i = 0; i < op.getNumOperands(); ++i) {
 157:       smemBases[i] = indexToBase[i];
```

- **EN:** Defines `std::sort`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `std::sort`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 158-161

```cpp
 158:     }
 159:     return smemBases;
 160:   }
 161: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 163-163

```cpp
 163: #endif
```

- **EN:** Defines or closes a file guard so this header/TableGen fragment is included only once during compilation.
- **CN:** 这里定义或关闭文件保护宏，确保该头文件/TableGen 片段在编译过程中只被包含一次。

## Key Concepts / 关键概念
- **EN:** The file focuses on lowering reduce scan common related IR into a more target-oriented representation.
  **CN:** 本文件重点是把与 Reduce Scan Common 相关的 IR 降级为更面向目标的表示。
- **EN:** MLIR dialect conversion patterns drive per-op lowering to LLVM-compatible IR.
  **CN:** MLIR 方言转换模式负责按操作粒度把 IR 降级为 LLVM 兼容表示。
- **EN:** Pattern rewriting is the main mechanism for canonicalization and lowering in this file.
  **CN:** 模式重写是本文件进行规范化与降级转换的核心机制。
- **EN:** Conversion rewrites replace source dialect ops with target dialect values while preserving semantics.
  **CN:** 转换式重写通过替换源方言操作来保持语义并生成目标方言值。

## Dependencies / 依赖关系
- **Internal Triton headers / Triton 内部头文件:** `triton/Conversion/TritonGPUToLLVM/Utility.h`
- **MLIR headers / MLIR 头文件:** `mlir/Dialect/ControlFlow/IR/ControlFlowOps.h`, `mlir/Dialect/LLVMIR/LLVMDialect.h`, `mlir/Transforms/DialectConversion.h`, `mlir/IR/TypeUtilities.h`
- **LLVM headers / LLVM 头文件:** None
- **Standard/library headers / 标准或通用库头文件:** `iterator`, `type_traits`
- **Primary APIs used / 主要 API:** `PatternRewriter`, `ConversionPatternRewriter`, `OpBuilder`
- **Lowering role / 降级角色:** The implementation depends on source-dialect semantics being valid and emits forms expected by later LLVM or GPU stages. / 该实现依赖源方言语义已经正确，并输出供后续 LLVM 或 GPU 阶段使用的形式。
