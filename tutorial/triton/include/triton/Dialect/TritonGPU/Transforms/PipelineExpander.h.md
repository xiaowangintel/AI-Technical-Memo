# PipelineExpander.h — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./include/triton/Dialect/TritonGPU/Transforms/PipelineExpander.h`
- **EN:** Declares transformation support utilities centered on `PipelineExpander`.
- **CN:** 声明围绕 `PipelineExpander` 的变换支持工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-5
```cpp
   1: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   2: // See https://llvm.org/LICENSE.txt for license information.
   3: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   4: #ifndef TRITON_DIALECT_TRITONGPU_TRANSFORMS_PIPELINE_H_
   5: #define TRITON_DIALECT_TRITONGPU_TRANSFORMS_PIPELINE_H_
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 7-8
```cpp
   7: // This is a fork of upstream pipeline transformation. This will be merged back
   8: // upstream once we have a stable solution.
```
**EN:** This comment block records the intent and constraints of the surrounding code: This is a fork of upstream pipeline transformation. This will be merged back upstream once we have a stable solution.
**CN:** 该注释块解释了周边代码的设计背景、意图或约束条件，帮助读者理解后续实现。

### Lines 10-12
```cpp
  10: #include "mlir/Support/LLVM.h"
  11: #include "mlir/Support/LogicalResult.h"
  12: #include "llvm/ADT/ArrayRef.h"
```
**EN:** This block imports the direct dependencies needed here, including mlir/Support/LLVM.h, mlir/Support/LogicalResult.h, and llvm/ADT/ArrayRef.h.
**CN:** 该代码块引入后续声明直接依赖的头文件，例如 mlir/Support/LLVM.h, mlir/Support/LogicalResult.h, and llvm/ADT/ArrayRef.h。

### Lines 14-14
```cpp
  14: namespace mlir {
```
**EN:** This block opens or closes the namespace scope used to organize APIs under mlir.
**CN:** 该代码块打开或关闭命名空间作用域，用来把相关 API 组织在 mlir 下。

### Lines 16-18
```cpp
  16: class RewriterBase;
  17: class Operation;
  18: class Value;
```
**EN:** This block introduces `RewriterBase`, the main class/struct defined here.
**CN:** 该代码块引入了此文件的核心类/结构体 `RewriterBase`。

### Lines 20-22
```cpp
  20: namespace scf {
  21: class ForOp;
  22: }
```
**EN:** This block stores supporting state such as ForOp, which other APIs in the file consume.
**CN:** 该代码块声明了 ForOp 等支撑状态，供本文件中的其他 API 使用。

### Lines 24-24
```cpp
  24: namespace triton {
```
**EN:** This block opens or closes the namespace scope used to organize APIs under triton.
**CN:** 该代码块打开或关闭命名空间作用域，用来把相关 API 组织在 triton 下。

### Lines 26-46
```cpp
  26: /// Options to dictate how loops should be pipelined.
  27: struct PipeliningOption {
  28:   /// Lambda returning all the operations in the forOp, with their stage, in the
  29:   /// order picked for the pipelined loop.
  30:   using GetScheduleFnType = std::function<void(
  31:       scf::ForOp, std::vector<std::pair<Operation *, unsigned>> &)>;
  32:   GetScheduleFnType getScheduleFn = nullptr;
  33:   enum class PipelinerPart {
  34:     Prologue,
  35:     Kernel,
  36:     Epilogue,
  37:   };
  38:   /// Lambda called by the pipeliner to allow the user to annotate the IR while
  39:   /// it is generated.
  40:   /// The callback passes the operation created along with the part of the
  41:   /// pipeline and the iteration index. The iteration index is always 0 for the
  42:   /// kernel. For the prologue and epilogue, it corresponds to the iteration
  43:   /// peeled out of the loop in the range [0, maxStage[.
  44:   using AnnotationlFnType =
  45:       std::function<void(Operation *, PipelinerPart, unsigned)>;
  46:   AnnotationlFnType annotateFn = nullptr;
```
**EN:** This block introduces `PipeliningOption`, the main class/struct defined here. Within the declaration, methods such as void expose its core API.
**CN:** 该代码块引入了此文件的核心类/结构体 `PipeliningOption`。 其中 void 等方法构成了它的主要接口。

### Lines 48-52
```cpp
  48:   /// Control whether the epilogue should be peeled out of the loop or
  49:   /// operations should be predicated to skip the early stages in the last loop
  50:   /// iterations. If the epilogue is predicated; the user needs to provide a
  51:   /// lambda to generate the predicated version of operations.
  52:   bool peelEpilogue = true;
```
**EN:** This block stores supporting state such as true, which other APIs in the file consume.
**CN:** 该代码块声明了 true 等支撑状态，供本文件中的其他 API 使用。

### Lines 54-58
```cpp
  54:   /// Control whether the transformation checks that the number of iterations is
  55:   /// greater or equal to the number of stages and skip the transformation if
  56:   /// this is not the case. If the loop is dynamic and this is set to true the
  57:   /// pipeliner will have to predicate operations in the prologue/epilogue.
  58:   bool supportDynamicLoops = false;
```
**EN:** This block stores supporting state such as false, which other APIs in the file consume.
**CN:** 该代码块声明了 false 等支撑状态，供本文件中的其他 API 使用。

### Lines 60-64
```cpp
  60:   /// If set, use this function to emit the predicate stage ops instead of the
  61:   /// default one.
  62:   using EmitPredicateStageFnType = std::function<Value(
  63:       RewriterBase &, Value, Value, Value, uint64_t, uint64_t)>;
  64:   EmitPredicateStageFnType emitPredicateStageFn = nullptr;
```
**EN:** This block declares or defines callable APIs such as Value, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 Value 等可调用 API，用来封装这里提供的核心行为。

### Lines 66-74
```cpp
  66:   // Callback to predicate operations when the prologue or epilogue are not
  67:   // peeled. This takes the original operation, an i1 predicate value and the
  68:   // pattern rewriter. It is expected to replace the given operation with
  69:   // the predicated equivalent and return it, or return nullptr if the
  70:   // predication is impossible. In the latter case, pipelining will fail and
  71:   // may leave IR in a partially transformed state.
  72:   using PredicateOpFnType =
  73:       std::function<Operation *(RewriterBase &, Operation *, Value)>;
  74:   PredicateOpFnType predicateFn = nullptr;
```
**EN:** This block stores supporting state such as nullptr, which other APIs in the file consume.
**CN:** 该代码块声明了 nullptr 等支撑状态，供本文件中的其他 API 使用。

### Lines 76-77
```cpp
  76:   // TODO: add option to decide if the prologue should be peeled.
  77: };
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 79-102
```cpp
  79: /// Generate a pipelined version of the scf.for loop based on the schedule given
  80: /// as option. This applies the mechanical transformation of changing the loop
  81: /// and generating the prologue/epilogue for the pipelining and doesn't make any
  82: /// decision regarding the schedule.
  83: /// Based on the options the loop is split into several stages.
  84: /// The transformation assumes that the scheduling given by user is valid.
  85: /// For example if we break a loop into 3 stages named S0, S1, S2 we would
  86: /// generate the following code with the number in parenthesis as the iteration
  87: /// index:
  88: ///
  89: ///   S0(0)                        // Prologue
  90: ///   S0(1) S1(0)                  // Prologue
  91: ///   scf.for %I = %C0 to %N - 2 {
  92: ///     S0(I+2) S1(I+1) S2(I)       // Pipelined kernel
  93: ///   }
  94: ///   S1(N) S2(N-1)                // Epilogue
  95: ///   S2(N)                        // Epilogue
  96: ///
  97: /// If `modifiedIR` is provided, it will be set to a value that indicates
  98: /// whether pipelining modified the IR before failing, signaling to the caller
  99: /// whether they can proceed with different transformations.
 100: FailureOr<scf::ForOp> pipelineForLoop(RewriterBase &rewriter, scf::ForOp forOp,
 101:                                       const PipeliningOption &options,
 102:                                       bool *modifiedIR = nullptr);
```
**EN:** This block declares or defines callable APIs such as pipelineForLoop, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 pipelineForLoop 等可调用 API，用来封装这里提供的核心行为。

### Lines 104-106
```cpp
 104: Value emitPredicateForStage(RewriterBase &rewriter, Value inductionVar,
 105:                             Value upperBound, Value step, uint64_t maxStage,
 106:                             uint64_t stage);
```
**EN:** This block declares or defines callable APIs such as emitPredicateForStage, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 emitPredicateForStage 等可调用 API，用来封装这里提供的核心行为。

### Lines 108-109
```cpp
 108: } // namespace triton
 109: } // namespace mlir
```
**EN:** This block manages the namespace scope for the surrounding declarations.
**CN:** 该代码块负责管理周围声明所属的命名空间作用域。

### Lines 111-111
```cpp
 111: #endif // TRITON_DIALECT_TRITONGPU_TRANSFORMS_PIPELINE_H_
```
**EN:** This block closes the header guard started at the top of the file.
**CN:** 该代码块结束文件顶部开始的头文件保护宏。

## Key Concepts / 关键概念
- **EN:** LLVM lowering  
  **CN:** LLVM 降级
- **EN:** compiler passes  
  **CN:** 编译 pass
- **EN:** MLIR dialect definition  
  **CN:** MLIR 方言定义
- **EN:** operation definitions  
  **CN:** 操作定义
- **EN:** types  
  **CN:** 类型
- **EN:** software pipelining  
  **CN:** 软件流水化
- **EN:** scheduling  
  **CN:** 调度
- **EN:** MLIR integration  
  **CN:** MLIR 集成

## Dependencies / 依赖关系
- **Project includes / 项目内依赖:**
  - `mlir/Support/LLVM.h`
  - `mlir/Support/LogicalResult.h`
  - `llvm/ADT/ArrayRef.h`
