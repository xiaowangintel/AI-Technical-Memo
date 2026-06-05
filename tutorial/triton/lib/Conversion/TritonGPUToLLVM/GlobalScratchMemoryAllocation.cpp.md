# GlobalScratchMemoryAllocation.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `lib/Conversion/TritonGPUToLLVM/GlobalScratchMemoryAllocation.cpp`
- **Purpose / 作用:** **EN:** Lowers TritonGPU constructs related to Global Scratch Memory Allocation into LLVM-compatible IR and rewrite patterns. **CN:** 把与 Global Scratch Memory Allocation 相关的 TritonGPU 构造降级为 LLVM 兼容 IR 与重写模式。

## Line-by-Line Analysis / 逐行分析

### Lines 1-3

```cpp
   1: #include "mlir/Analysis/Liveness.h"
   2: #include "triton/Conversion/TritonGPUToLLVM/Passes.h"
   3: #include "triton/Dialect/TritonGPU/IR/Dialect.h"
```

- **EN:** Includes the interfaces this file depends on. Triton headers (`Passes.h`, `Dialect.h`) provide domain-specific IR/support, MLIR headers (`Liveness.h`) provide rewriting and analysis infrastructure, LLVM headers (None) supply low-level utilities, and standard/library headers (None) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（`Passes.h`, `Dialect.h`）提供领域特定 IR/支持逻辑，MLIR 头文件（`Liveness.h`）提供重写与分析基础设施，LLVM 头文件（None）提供底层工具，而标准/通用库头文件（None）提供通用能力。
### Lines 5-7

```cpp
   5: using namespace mlir;
   6: using namespace triton;
   7: using namespace triton::gpu;
```

- **EN:** Introduces namespace aliases/imports (`mlir`, `triton`, `triton::gpu`) so the rest of the file can use MLIR/Triton symbols without repetitive qualification.
- **CN:** 这里通过命名空间导入（`mlir`, `triton`, `triton::gpu`）减少后续代码中的重复限定，使 MLIR/Triton 符号使用更简洁。
### Lines 9-12

```cpp
   9: namespace mlir::triton::gpu {
  10: #define GEN_PASS_DEF_TRITONGPUGLOBALSCRATCHALLOCATIONPASS
  11: #include "triton/Conversion/TritonGPUToLLVM/Passes.h.inc"
  12: } // namespace mlir::triton::gpu
```

- **EN:** Pulls in generated definitions from TableGen/MLIR include fragments so the handwritten code can reuse auto-generated declarations.
- **CN:** 这里引入由 TableGen/MLIR 生成的定义片段，使手写代码能够复用自动生成的声明。
### Lines 14-17

```cpp
  14: static int32_t roundUp(int32_t val, int32_t step) {
  15:   auto t = val + step - 1;
  16:   return t - (t % step);
  17: }
```

- **EN:** Defines `roundUp`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `roundUp`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 19-22

```cpp
  19: struct ScratchMemoryInfo {
  20:   int32_t offset = 0;
  21:   uint32_t largestAlignment = 1;
  22: };
```

- **EN:** Defines `ScratchMemoryInfo`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `ScratchMemoryInfo`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 24-33

```cpp
  24: static void assignOffset(Operation *op, OpBuilder &builder,
  25:                          ScratchMemoryInfo &memInfo, uint32_t nbytes,
  26:                          uint32_t align, StringRef offsetAttrName) {
  27:   if (nbytes == 0)
  28:     return;
  29:   memInfo.offset = roundUp(memInfo.offset, align);
  30:   op->setAttr(offsetAttrName, builder.getI32IntegerAttr(memInfo.offset));
  31:   memInfo.offset += nbytes;
  32:   memInfo.largestAlignment = std::max(memInfo.largestAlignment, align);
  33: }
```

- **EN:** Defines `assignOffset`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `assignOffset`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 35-50

```cpp
  35: static void setModuleScratchAttrs(Operation *op, OpBuilder &builder,
  36:                                   const ScratchMemoryInfo &globalMemInfo,
  37:                                   const ScratchMemoryInfo &profileMemInfo) {
  38:   int32_t totalGlobalMemorySize =
  39:       roundUp(globalMemInfo.offset, globalMemInfo.largestAlignment);
  40:   int32_t totalProfileMemorySize =
  41:       roundUp(profileMemInfo.offset, profileMemInfo.largestAlignment);
  42:   op->setAttr("ttg.global_scratch_memory_size",
  43:               builder.getI32IntegerAttr(totalGlobalMemorySize));
  44:   op->setAttr("ttg.global_scratch_memory_alignment",
  45:               builder.getI32IntegerAttr(globalMemInfo.largestAlignment));
  46:   op->setAttr("ttg.profile_scratch_memory_size",
  47:               builder.getI32IntegerAttr(totalProfileMemorySize));
  48:   op->setAttr("ttg.profile_scratch_memory_alignment",
  49:               builder.getI32IntegerAttr(profileMemInfo.largestAlignment));
  50: }
```

- **EN:** Defines accessor/helper `setModuleScratchAttrs` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `setModuleScratchAttrs`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 52-64

```cpp
  52: static void allocateGMem(Operation *parentOp,
  53:                          llvm::SetVector<Operation *> &callStack) {
  54:   // Recursively visit any dependency functions
  55:   parentOp->walk([&](triton::CallOp call) {
  56:     auto callable = call.resolveCallable();
  57:     if (!callable->hasAttr("ttg.global_scratch_memory_size") ||
  58:         !callable->hasAttr("ttg.profile_scratch_memory_size")) {
  59:       auto inserted = callStack.insert(parentOp);
  60:       assert(inserted && "call cycle detected");
  61:       allocateGMem(callable, callStack);
  62:       callStack.remove(parentOp);
  63:     }
  64:   });
```

- **EN:** Defines `allocateGMem`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. The implementation traverses IR operations to collect facts or apply rewrites globally. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `allocateGMem`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 66-69

```cpp
  66:   MLIRContext *ctx = parentOp->getContext();
  67:   OpBuilder builder(ctx);
  68:   ScratchMemoryInfo globalMemInfo;
  69:   ScratchMemoryInfo profileMemInfo;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 71-88

```cpp
  71:   // Dumb allocation that ignores liveness and makes no attempt to minimize
  72:   // padding
  73:   // TODO: Use a real algorithm
  74:   parentOp->walk<WalkOrder::PostOrder>([&](Operation *op) {
  75:     if (auto alloc = dyn_cast<triton::gpu::GlobalScratchAllocOp>(op)) {
  76:       bool isThirdPartyAlloc = alloc->hasAttr("third_party_allocation");
  77:       ScratchMemoryInfo &memInfo =
  78:           isThirdPartyAlloc ? profileMemInfo : globalMemInfo;
  79:       assignOffset(op, builder, memInfo, alloc.getNbytes(),
  80:                    alloc.getAlignment(), "ttg.global_scratch_memory_offset");
  81:     } else if (auto callOp = dyn_cast<triton::CallOp>(op)) {
  82:       auto callable = callOp.resolveCallable();
  83:       auto globalNbytesAttr = callable->getAttrOfType<IntegerAttr>(
  84:           "ttg.global_scratch_memory_size");
  85:       auto globalAlignAttr = callable->getAttrOfType<IntegerAttr>(
  86:           "ttg.global_scratch_memory_alignment");
  87:       auto profileNbytesAttr = callable->getAttrOfType<IntegerAttr>(
  88:           "ttg.profile_scratch_memory_size");
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 89-92

```cpp
  89:       auto profileAlignAttr = callable->getAttrOfType<IntegerAttr>(
  90:           "ttg.profile_scratch_memory_alignment");
  91:       assert(globalNbytesAttr && globalAlignAttr && profileNbytesAttr &&
  92:              profileAlignAttr);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 94-105

```cpp
  94:       assignOffset(op, builder, globalMemInfo,
  95:                    globalNbytesAttr.getValue().getZExtValue(),
  96:                    globalAlignAttr.getValue().getZExtValue(),
  97:                    "ttg.global_scratch_memory_offset");
  98:       assignOffset(op, builder, profileMemInfo,
  99:                    profileNbytesAttr.getValue().getZExtValue(),
 100:                    profileAlignAttr.getValue().getZExtValue(),
 101:                    "ttg.profile_scratch_memory_offset");
 102:     }
 103:   });
 104:   setModuleScratchAttrs(parentOp, builder, globalMemInfo, profileMemInfo);
 105: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 107-116

```cpp
 107: namespace {
 108: class TritonGPUGlobalScratchAllocationPass
 109:     : public mlir::triton::gpu::impl::TritonGPUGlobalScratchAllocationPassBase<
 110:           TritonGPUGlobalScratchAllocationPass> {
 111: public:
 112:   void runOnOperation() override {
 113:     runGlobalScratchMemoryAllocation(getOperation());
 114:   }
 115: };
 116: } // namespace
```

- **EN:** Defines `TritonGPUGlobalScratchAllocationPass`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `TritonGPUGlobalScratchAllocationPass`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 118-119

```cpp
 118: void mlir::triton::gpu::runGlobalScratchMemoryAllocation(ModuleOp mod) {
 119:   bool seenKernel = false;
```

- **EN:** Defines `mlir::triton::gpu::runGlobalScratchMemoryAllocation`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `mlir::triton::gpu::runGlobalScratchMemoryAllocation`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 121-123

```cpp
 121:   SetVector<Operation *> callStack;
 122:   mod->walk([&](triton::FuncOp func) {
 123:     allocateGMem(func, callStack);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. The implementation traverses IR operations to collect facts or apply rewrites globally.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。
### Lines 125-142

```cpp
 125:     if (func.getVisibility() == SymbolTable::Visibility::Public) {
 126:       assert(!seenKernel);
 127:       seenKernel = true;
 128:       auto size =
 129:           func->getAttrOfType<IntegerAttr>("ttg.global_scratch_memory_size");
 130:       auto align = func->getAttrOfType<IntegerAttr>(
 131:           "ttg.global_scratch_memory_alignment");
 132:       auto profileSize =
 133:           func->getAttrOfType<IntegerAttr>("ttg.profile_scratch_memory_size");
 134:       auto profileAlign = func->getAttrOfType<IntegerAttr>(
 135:           "ttg.profile_scratch_memory_alignment");
 136:       assert(size);
 137:       assert(align);
 138:       assert(profileSize);
 139:       assert(profileAlign);
 140:       mod->setAttr("ttg.global_scratch_memory_size", size);
 141:       mod->setAttr("ttg.global_scratch_memory_alignment", align);
 142:       mod->setAttr("ttg.profile_scratch_memory_size", profileSize);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 143-147

```cpp
 143:       mod->setAttr("ttg.profile_scratch_memory_alignment", profileAlign);
 144:     }
 145:   });
 146:   assert(seenKernel);
 147: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。

## Key Concepts / 关键概念
- **EN:** The file focuses on lowering global scratch memory allocation related IR into a more target-oriented representation.
  **CN:** 本文件重点是把与 Global Scratch Memory Allocation 相关的 IR 降级为更面向目标的表示。
- **EN:** The code reasons at module scope rather than only on isolated operations.
  **CN:** 代码在模块范围内进行分析，而不只是处理单个操作。

## Dependencies / 依赖关系
- **Internal Triton headers / Triton 内部头文件:** `triton/Conversion/TritonGPUToLLVM/Passes.h`, `triton/Dialect/TritonGPU/IR/Dialect.h`, `triton/Conversion/TritonGPUToLLVM/Passes.h.inc`
- **MLIR headers / MLIR 头文件:** `mlir/Analysis/Liveness.h`
- **LLVM headers / LLVM 头文件:** None
- **Standard/library headers / 标准或通用库头文件:** None
- **Generated fragments / 生成片段:** `triton/Conversion/TritonGPUToLLVM/Passes.h.inc`
- **Primary APIs used / 主要 API:** `ModuleOp`, `OpBuilder`
- **Lowering role / 降级角色:** The implementation depends on source-dialect semantics being valid and emits forms expected by later LLVM or GPU stages. / 该实现依赖源方言语义已经正确，并输出供后续 LLVM 或 GPU 阶段使用的形式。
