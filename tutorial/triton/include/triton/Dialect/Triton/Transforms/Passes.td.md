# Passes.td — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./include/triton/Dialect/Triton/Transforms/Passes.td`
- **EN:** Defines TableGen pass records, options, and registration metadata for this subsystem.
- **CN:** 定义该子系统的 TableGen pass 记录、选项与注册元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```tablegen
   1: #ifndef TRITON_PASSES
   2: #define TRITON_PASSES
```
**EN:** This block establishes the header guard so the declarations in this file are only processed once per translation unit.
**CN:** 该代码块建立头文件保护宏，避免同一翻译单元重复包含本文件时出现重复声明。

### Lines 4-4
```tablegen
   4: include "mlir/Pass/PassBase.td"
```
**EN:** This block imports the direct dependencies needed here, including mlir/Pass/PassBase.td.
**CN:** 该代码块引入后续声明直接依赖的头文件，例如 mlir/Pass/PassBase.td。

### Lines 6-10
```tablegen
   6: def TritonCombineOps : Pass</*cli-arg*/"triton-combine", /*Op*/"mlir::ModuleOp"> {
   7:   let summary = "combine ops";
   8:   let description = [{
   9:     This pass aims to optimize the five following patterns:
  10:     - `dot(a, b, 0) + c => dot(a, b, c)`
```
**EN:** This TableGen def record defines `TritonCombineOps` with the summary “combine ops”. It is specialized from `Pass</*cli-arg*/"triton-combine", /*Op*/"mlir::ModuleOp">`.
**CN:** 该 TableGen def 记录定义了 `TritonCombineOps`，其摘要为“combine ops”。 它基于 `Pass</*cli-arg*/"triton-combine", /*Op*/"mlir::ModuleOp">` 进一步特化。

### Lines 12-12
```tablegen
  12:     - `addptr(addptr(ptr, idx0), idx1) => addptr(ptr, AddI(idx0, idx1))`
```
**EN:** This block declares or defines callable APIs such as addptr and AddI, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 addptr and AddI 等可调用 API，用来封装这里提供的核心行为。

### Lines 14-15
```tablegen
  14:     - `select(cond, load(ptrs, broadcast(cond), ???), other) =>
  15:          load(ptrs, broadcast(cond), other)`
```
**EN:** This block declares or defines callable APIs such as select, load, and broadcast, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 select, load, and broadcast 等可调用 API，用来封装这里提供的核心行为。

### Lines 17-20
```tablegen
  17:     - `broadcast(constant) => reshaped_constant`
  18:     - `torch.sum(x[:,:,None].expand(-1,-1,n) * y[None,:,:].expand(m,-1,-1),1)
  19:        => dot(x,y,splat(0))`
  20:   }];
```
**EN:** This block declares or defines callable APIs such as broadcast, sum, expand, dot, and splat, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 broadcast, sum, expand, dot, and splat 等可调用 API，用来封装这里提供的核心行为。

### Lines 22-23
```tablegen
  22:   let dependentDialects = ["mlir::arith::ArithDialect"];
  23: }
```
**EN:** This block lists dialect dependencies that must be loaded before the generated pass or dialect can operate correctly.
**CN:** 该代码块列出了生成 pass 或方言运行前必须加载的依赖方言。

### Lines 25-33
```tablegen
  25: def TritonReorderBroadcast : Pass</*cli-arg*/"triton-reorder-broadcast", /*Op*/"mlir::ModuleOp"> {
  26:   let summary = "Moves broadcast and splat after elementwise operations";
  27:   let description = [{
  28:     The purpose of this pass is to transform:
  29:       - `elementwise(broadcast(a)) => broadcast(elementwise(a))`
  30:       - `elementwise(splat(a), splat(b), ...) => splat(elementwise(a, b, ...))`
  31:     In the event of a match, the broadcast (or splat) operation is delayed
  32:     and performed after the ElementWise operation.
  33:   }];
```
**EN:** This TableGen def record defines `TritonReorderBroadcast` with the summary “Moves broadcast and splat after elementwise operations”. It is specialized from `Pass</*cli-arg*/"triton-reorder-broadcast", /*Op*/"mlir::ModuleOp">`.
**CN:** 该 TableGen def 记录定义了 `TritonReorderBroadcast`，其摘要为“Moves broadcast and splat after elementwise operations”。 它基于 `Pass</*cli-arg*/"triton-reorder-broadcast", /*Op*/"mlir::ModuleOp">` 进一步特化。

### Lines 35-36
```tablegen
  35:   let dependentDialects = ["mlir::triton::TritonDialect"];
  36: }
```
**EN:** This block lists dialect dependencies that must be loaded before the generated pass or dialect can operate correctly.
**CN:** 该代码块列出了生成 pass 或方言运行前必须加载的依赖方言。

### Lines 38-44
```tablegen
  38: def TritonRewriteTensorDescriptorToPointer : Pass</*cli-arg*/"triton-rewrite-tensor-descriptor-to-pointer", /*Op*/"mlir::ModuleOp"> {
  39:   let summary = "Rewrite load/stores of tensor descriptors into pointer load/stores";
  40:   let description = [{
  41:     This pass rewrites all load/store semantics initiated by a `tt.make_tensor_descriptor` into pointer semantics. After
  42:     this pass, `tt.make_tensor_descriptor`  will disappear, and it generates logics to compute the pointer/mask/other
  43:     for each load/store.
  44:   }];
```
**EN:** This TableGen def record defines `TritonRewriteTensorDescriptorToPointer` with the summary “Rewrite load/stores of tensor descriptors into pointer load/stores”. It is specialized from `Pass</*cli-arg*/"triton-rewrite-tensor-descriptor-to-pointer", /*Op*/"mlir::ModuleOp">`.
**CN:** 该 TableGen def 记录定义了 `TritonRewriteTensorDescriptorToPointer`，其摘要为“Rewrite load/stores of tensor descriptors into pointer load/stores”。 它基于 `Pass</*cli-arg*/"triton-rewrite-tensor-descriptor-to-pointer", /*Op*/"mlir::ModuleOp">` 进一步特化。

### Lines 46-47
```tablegen
  46:   let dependentDialects = ["mlir::triton::TritonDialect"];
  47: }
```
**EN:** This block lists dialect dependencies that must be loaded before the generated pass or dialect can operate correctly.
**CN:** 该代码块列出了生成 pass 或方言运行前必须加载的依赖方言。

### Lines 49-54
```tablegen
  49: def TritonLoopUnroll : Pass</*cli-arg*/"triton-loop-unroll", /*Op*/"mlir::ModuleOp"> {
  50:   let summary = "Loop unroller";
  51:   let description = [{
  52:     The pass unrolls a scf loop with tt.loop_unroll_factor attribute. The attribute specialises how many iterations
  53:     the loop should be unrolled.
  54:   }];
```
**EN:** This TableGen def record defines `TritonLoopUnroll` with the summary “Loop unroller”. It is specialized from `Pass</*cli-arg*/"triton-loop-unroll", /*Op*/"mlir::ModuleOp">`.
**CN:** 该 TableGen def 记录定义了 `TritonLoopUnroll`，其摘要为“Loop unroller”。 它基于 `Pass</*cli-arg*/"triton-loop-unroll", /*Op*/"mlir::ModuleOp">` 进一步特化。

### Lines 56-57
```tablegen
  56:   let dependentDialects = ["mlir::triton::TritonDialect"];
  57: }
```
**EN:** This block lists dialect dependencies that must be loaded before the generated pass or dialect can operate correctly.
**CN:** 该代码块列出了生成 pass 或方言运行前必须加载的依赖方言。

### Lines 59-66
```tablegen
  59: def TritonLoopInvariantCodeMotion : Pass</*cli-arg*/"triton-licm", /*Op*/"mlir::ModuleOp"> {
  60:   let summary = "MLIR's LICM plus hoist load ops out of loops with masks.";
  61:   let description = [{
  62:     This pass uses MLIR's LICM pass as base. Additionally, it hoists load ops
  63:     out of loops that consists of pure/read-only ops. For scf.for loops, it
  64:     generates a trip-count check. For scf.while loops, it clones the condition
  65:     from the before body.
  66:   }];
```
**EN:** This TableGen def record defines `TritonLoopInvariantCodeMotion` with the summary “MLIR's LICM plus hoist load ops out of loops with masks.”. It is specialized from `Pass</*cli-arg*/"triton-licm", /*Op*/"mlir::ModuleOp">`.
**CN:** 该 TableGen def 记录定义了 `TritonLoopInvariantCodeMotion`，其摘要为“MLIR's LICM plus hoist load ops out of loops with masks.”。 它基于 `Pass</*cli-arg*/"triton-licm", /*Op*/"mlir::ModuleOp">` 进一步特化。

### Lines 68-69
```tablegen
  68:   let dependentDialects = ["mlir::triton::TritonDialect"];
  69: }
```
**EN:** This block lists dialect dependencies that must be loaded before the generated pass or dialect can operate correctly.
**CN:** 该代码块列出了生成 pass 或方言运行前必须加载的依赖方言。

### Lines 71-72
```tablegen
  71: def TritonLoopAwareCSE : Pass<"triton-loop-aware-cse", "mlir::ModuleOp"> {
  72:   let summary = "CSE within loop bodies";
```
**EN:** This TableGen def record defines `TritonLoopAwareCSE` with the summary “CSE within loop bodies”. It is specialized from `Pass<"triton-loop-aware-cse", "mlir::ModuleOp">`.
**CN:** 该 TableGen def 记录定义了 `TritonLoopAwareCSE`，其摘要为“CSE within loop bodies”。 它基于 `Pass<"triton-loop-aware-cse", "mlir::ModuleOp">` 进一步特化。

### Lines 74-80
```tablegen
  74:   let description = [{
  75:     The `triton-loop-aware-cse` pass performs recursive common subexpression
  76:     elimination within loop bodies. Unlike regular CSE, which is a single-pass
  77:     greedy algorithm, this pass can recursively eliminate loop iteration
  78:     arguments and subcomputations that always have the same value.
  79:   }];
  80: }
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 82-82
```tablegen
  82: #endif
```
**EN:** This block closes the header guard started at the top of the file.
**CN:** 该代码块结束文件顶部开始的头文件保护宏。

## Key Concepts / 关键概念
- **EN:** dot-product lowering  
  **CN:** 点积降级
- **EN:** compiler passes  
  **CN:** 编译 pass
- **EN:** MLIR dialect definition  
  **CN:** MLIR 方言定义
- **EN:** operation definitions  
  **CN:** 操作定义
- **EN:** attributes  
  **CN:** 属性
- **EN:** MLIR integration  
  **CN:** MLIR 集成

## Dependencies / 依赖关系
- **Project includes / 项目内依赖:**
  - `mlir/Pass/PassBase.td`
