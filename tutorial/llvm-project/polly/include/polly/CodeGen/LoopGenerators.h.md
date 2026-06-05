# LoopGenerators.h — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `polly/include/polly/CodeGen/LoopGenerators.h` | `polly/include/polly/CodeGen/LoopGenerators.h` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | Declares public Polly code-generation interfaces, builders, and helper types. The leading comment describes it as: IR helper to create loops. | 声明 Polly 代码生成相关的公共接口、构建器与辅助类型。 文件开头注释将其概括为：IR helper to create loops。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-15

````cpp
//===- LoopGenerators.h - IR helper to create loops -------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains functions to create scalar and OpenMP parallel loops
// as LLVM-IR.
//
//===----------------------------------------------------------------------===//
#ifndef POLLY_LOOP_GENERATORS_H
#define POLLY_LOOP_GENERATORS_H

````
- **EN**: This block records the standard LLVM file banner and license metadata; uses preprocessor directives to control compilation, macros, or include guards; defines macros like `POLLY_LOOP_GENERATORS_H`; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 给出 LLVM 标准文件头与许可证元数据; 使用预处理指令控制编译、宏或头文件保护; 定义宏，例如 `POLLY_LOOP_GENERATORS_H`; 保留用于解释意图、用法或算法背景的注释.

### Lines 16-32

````cpp
#include "polly/CodeGen/IRBuilder.h"
#include "polly/Support/ScopHelper.h"
#include "llvm/ADT/SetVector.h"

namespace polly {
using llvm::AllocaInst;
using llvm::BasicBlock;
using llvm::DataLayout;
using llvm::DominatorTree;
using llvm::Function;
using llvm::ICmpInst;
using llvm::LoopInfo;
using llvm::Module;
using llvm::SetVector;
using llvm::Type;
using llvm::Value;

````
- **EN**: This block imports Polly, LLVM-family headers needed by the surrounding code; opens or organizes declarations inside a C++ namespace; touches Polly SCoP abstractions that model analyzable regions.
- **CN**: 该代码块 引入周边逻辑所需的 Polly、LLVM-family 头文件; 在 C++ 命名空间中组织声明或实现; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域.

### Lines 33-47

````cpp
/// General scheduling types of parallel OpenMP for loops.
/// Initialization values taken from OpenMP's enum in kmp.h: sched_type.
/// Currently, only 'static' scheduling may change from chunked to non-chunked.
enum class OMPGeneralSchedulingType {
  StaticChunked = 33,
  StaticNonChunked = 34,
  Dynamic = 35,
  Guided = 36,
  Runtime = 37
};

extern int PollyNumThreads;
extern OMPGeneralSchedulingType PollyScheduling;
extern int PollyChunkSize;

````
- **EN**: This block defines enum values such as `OMPGeneralSchedulingType`; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 定义枚举类型，例如 `OMPGeneralSchedulingType`; 保留用于解释意图、用法或算法背景的注释.

### Lines 48-63

````cpp
/// Create a scalar do/for-style loop.
///
/// @param LowerBound         The starting value of the induction variable.
/// @param UpperBound         The upper bound of the induction variable.
/// @param Stride             The value by which the induction variable
///                           is incremented.
///
/// @param Builder            The builder used to create the loop.
/// @param P                  A pointer to the pass that uses this function.
///                           It is used to update analysis information.
/// @param LI                 The loop info we need to update
/// @param DT                 The dominator tree we need to update
/// @param ExitBlock          The block the loop will exit to.
/// @param Predicate          The predicate used to generate the upper loop
///                           bound.
/// @param Annotator          This function can (optionally) take
````
- **EN**: This block preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 保留用于解释意图、用法或算法背景的注释.

### Lines 64-80

````cpp
///                           a ScopAnnotator which
///                           annotates loops and alias information in the SCoP.
/// @param Parallel           If this loop should be marked parallel in
///                           the Annotator.
/// @param UseGuard           Create a guard in front of the header to check if
///                           the loop is executed at least once, otherwise just
///                           assume it.
/// @param LoopVectDisabled   If the Loop vectorizer should be disabled for this
///                           loop.
///
/// @return Value*    The newly created induction variable for this loop.
Value *createLoop(Value *LowerBound, Value *UpperBound, Value *Stride,
                  PollyIRBuilder &Builder, LoopInfo &LI, DominatorTree &DT,
                  BasicBlock *&ExitBlock, ICmpInst::Predicate Predicate,
                  ScopAnnotator *Annotator = nullptr, bool Parallel = false,
                  bool UseGuard = true, bool LoopVectDisabled = false);

````
- **EN**: This block declares or defines routines around `createLoop`; emits return paths that hand results or status codes back to callers; touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `createLoop` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 81-96

````cpp
/// Create a DebugLoc representing generated instructions.
///
/// The IR verifier requires !dbg metadata to be set in some situations. For
/// instance, if an (inlinable) function has debug info, all its call site must
/// have debug info as well.
llvm::DebugLoc createDebugLocForGeneratedCode(Function *F);

/// The ParallelLoopGenerator allows to create parallelized loops
///
/// To parallelize a loop, we perform the following steps:
///   o  Generate a subfunction which will hold the loop body.
///   o  Create a struct to hold all outer values needed in the loop body.
///   o  Create calls to a runtime library to achieve the actual parallelism.
///      These calls will spawn and join threads, define how the work (here the
///      iterations) are distributed between them and make sure each has access
///      to the struct holding all needed values.
````
- **EN**: This block declares or defines routines around `createDebugLocForGeneratedCode`; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `createDebugLocForGeneratedCode` 相关的例程; 保留用于解释意图、用法或算法背景的注释.

### Lines 97-112

````cpp
///
/// At the moment we support only one parallel runtime, OpenMP.
///
/// If we parallelize the outer loop of the following loop nest,
///
///   S0;
///   for (int i = 0; i < N; i++)
///     for (int j = 0; j < M; j++)
///       S1(i, j);
///   S2;
///
/// we will generate the following code (with different runtime function names):
///
///   S0;
///   auto *values = storeValuesIntoStruct();
///   // Execute subfunction with multiple threads
````
- **EN**: This block contains control flow with 2 loop construct(s); preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 包含控制流结构：2 处循环; 保留用于解释意图、用法或算法背景的注释.

### Lines 113-128

````cpp
///   spawn_threads(subfunction, values);
///   join_threads();
///   S2;
///
///  // This function is executed in parallel by different threads
///   void subfunction(values) {
///     while (auto *WorkItem = getWorkItem()) {
///       int LB = WorkItem.begin();
///       int UB = WorkItem.end();
///       for (int i = LB; i < UB; i++)
///         for (int j = 0; j < M; j++)
///           S1(i, j);
///     }
///     cleanup_thread();
///   }
class ParallelLoopGenerator {
````
- **EN**: This block declares or references types such as `ParallelLoopGenerator`; contains control flow with 3 loop construct(s); preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或引用类型，例如 `ParallelLoopGenerator`; 包含控制流结构：3 处循环; 保留用于解释意图、用法或算法背景的注释.

### Lines 129-144

````cpp
public:
  /// Create a parallel loop generator for the current function.
  ParallelLoopGenerator(PollyIRBuilder &Builder, const DataLayout &DL)
      : Builder(Builder), LongType(Type::getIntNTy(Builder.getContext(),
                                                   DL.getPointerSizeInBits())),
        M(Builder.GetInsertBlock()->getParent()->getParent()),
        DLGenerated(createDebugLocForGeneratedCode(
            Builder.GetInsertBlock()->getParent())) {}

  virtual ~ParallelLoopGenerator() {}

  /// Create a parallel loop.
  ///
  /// This function is the main function to automatically generate a parallel
  /// loop with all its components.
  ///
````
- **EN**: This block declares or defines routines around `ParallelLoopGenerator`, `Builder`, `getPointerSizeInBits`, `M` (+3 more); preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `ParallelLoopGenerator`, `Builder`, `getPointerSizeInBits`, `M` (+3 more) 相关的例程; 保留用于解释意图、用法或算法背景的注释.

### Lines 145-160

````cpp
  /// @param LB        The lower bound for the loop we parallelize.
  /// @param UB        The upper bound for the loop we parallelize.
  /// @param Stride    The stride of the loop we parallelize.
  /// @param Values    A set of LLVM-IR Values that should be available in
  ///                  the new loop body.
  /// @param VMap      A map to allow outside access to the new versions of
  ///                  the values in @p Values.
  /// @param LoopBody  A pointer to an iterator that is set to point to the
  ///                  body of the created loop. It should be used to insert
  ///                  instructions that form the actual loop body.
  ///
  /// @return The newly created induction variable for this loop.
  Value *createParallelLoop(Value *LB, Value *UB, Value *Stride,
                            SetVector<Value *> &Values, ValueMapT &VMap,
                            BasicBlock::iterator *LoopBody);

````
- **EN**: This block declares or defines routines around `createParallelLoop`; emits return paths that hand results or status codes back to callers; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `createParallelLoop` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 保留用于解释意图、用法或算法背景的注释.

### Lines 161-173

````cpp
protected:
  /// The IR builder we use to create instructions.
  PollyIRBuilder &Builder;

  /// The loop info for the generated subfunction.
  std::unique_ptr<LoopInfo> SubFnLI;

  /// The dominance tree for the generated subfunction.
  std::unique_ptr<DominatorTree> SubFnDT;

  /// The type of a "long" on this hardware used for backend calls.
  Type *LongType;

````
- **EN**: This block preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 保留用于解释意图、用法或算法背景的注释.

### Lines 174-187

````cpp
  /// The current module
  Module *M;

  /// Debug location for generated code without direct link to any specific
  /// line.
  ///
  /// We only set the DebugLoc where the IR Verifier requires us to. Otherwise,
  /// absent debug location for optimized code should be fine.
  llvm::DebugLoc DLGenerated;

public:
  /// Returns the DominatorTree for the generated subfunction.
  DominatorTree *getCalleeDominatorTree() const { return SubFnDT.get(); }

````
- **EN**: This block declares or defines routines around `getCalleeDominatorTree`; emits return paths that hand results or status codes back to callers; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `getCalleeDominatorTree` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 保留用于解释意图、用法或算法背景的注释.

### Lines 188-206

````cpp
  /// Returns the LoopInfo for the generated subfunction.
  LoopInfo *getCalleeLoopInfo() const { return SubFnLI.get(); }

  /// Create a struct for all @p Values and store them in there.
  ///
  /// @param Values The values which should be stored in the struct.
  ///
  /// @return The created struct.
  AllocaInst *storeValuesIntoStruct(SetVector<Value *> &Values);

  /// Extract all values from the @p Struct and construct the mapping.
  ///
  /// @param Values The values which were stored in the struct.
  /// @param Struct The struct holding all the values in @p Values.
  /// @param VMap   A map to associate every element of @p Values with the
  ///               new llvm value loaded from the @p Struct.
  void extractValuesFromStruct(SetVector<Value *> Values, Type *Ty,
                               Value *Struct, ValueMapT &VMap);

````
- **EN**: This block declares or defines routines around `getCalleeLoopInfo`, `storeValuesIntoStruct`, `extractValuesFromStruct`; emits return paths that hand results or status codes back to callers; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `getCalleeLoopInfo`, `storeValuesIntoStruct`, `extractValuesFromStruct` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 保留用于解释意图、用法或算法背景的注释.

### Lines 207-223

````cpp
  /// Create the definition of the parallel subfunction.
  ///
  /// @return A pointer to the subfunction.
  Function *createSubFnDefinition();

  /// Create the runtime library calls for spawn and join of the worker threads.
  /// Additionally, places a call to the specified subfunction.
  ///
  /// @param SubFn      The subfunction which holds the loop body.
  /// @param SubFnParam The parameter for the subfunction (basically the struct
  ///                   filled with the outside values).
  /// @param LB         The lower bound for the loop we parallelize.
  /// @param UB         The upper bound for the loop we parallelize.
  /// @param Stride     The stride of the loop we parallelize.
  virtual void deployParallelExecution(Function *SubFn, Value *SubFnParam,
                                       Value *LB, Value *UB, Value *Stride) = 0;

````
- **EN**: This block declares or defines routines around `createSubFnDefinition`, `deployParallelExecution`; emits return paths that hand results or status codes back to callers; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `createSubFnDefinition`, `deployParallelExecution` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 保留用于解释意图、用法或算法背景的注释.

### Lines 224-239

````cpp
  /// Prepare the definition of the parallel subfunction.
  /// Creates the argument list and names them (as well as the subfunction).
  ///
  /// @param F A pointer to the (parallel) subfunction's parent function.
  ///
  /// @return The pointer to the (parallel) subfunction.
  virtual Function *prepareSubFnDefinition(Function *F) const = 0;

  /// Create the parallel subfunction.
  ///
  /// @param Stride The induction variable increment.
  /// @param Struct A struct holding all values in @p Values.
  /// @param Values A set of LLVM-IR Values that should be available in
  ///               the new loop body.
  /// @param VMap   A map to allow outside access to the new versions of
  ///               the values in @p Values.
````
- **EN**: This block declares or defines routines around `prepareSubFnDefinition`; emits return paths that hand results or status codes back to callers; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `prepareSubFnDefinition` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 保留用于解释意图、用法或算法背景的注释.

### Lines 240-248

````cpp
  /// @param SubFn  The newly created subfunction is returned here.
  ///
  /// @return The newly created induction variable.
  virtual std::tuple<Value *, Function *>
  createSubFn(Value *Stride, AllocaInst *Struct, SetVector<Value *> UsedValues,
              ValueMapT &VMap) = 0;
};
} // end namespace polly
#endif
````
- **EN**: This block uses preprocessor directives to control compilation, macros, or include guards; declares or defines routines around `createSubFn`; emits return paths that hand results or status codes back to callers; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 使用预处理指令控制编译、宏或头文件保护; 声明或定义与 `createSubFn` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 保留用于解释意图、用法或算法背景的注释.

## Key Concepts / 关键概念

- **Loop transformation**
  - **CN**: 循环变换
- **Code generation**
  - **CN**: 代码生成
- **Debug and diagnostics**
  - **CN**: 调试与诊断
- **OpenMP/runtime support**
  - **CN**: OpenMP/运行时支持
- **Template/type wrappers**
  - **CN**: 模板与类型包装
- **SCoP modeling**
  - **CN**: SCoP 建模

## Dependencies / 依赖关系

- **Polly headers**: `polly/CodeGen/IRBuilder.h`, `polly/Support/ScopHelper.h` — Direct Polly APIs, data structures, or pass interfaces used here.
  **Polly headers（CN）**：`polly/CodeGen/IRBuilder.h`, `polly/Support/ScopHelper.h` —— 此处直接使用的 Polly API、数据结构或 Pass 接口。
- **LLVM-family headers**: `llvm/ADT/SetVector.h` — LLVM/Clang/MLIR infrastructure for IR, passes, utilities, or diagnostics.
  **LLVM-family headers（CN）**：`llvm/ADT/SetVector.h` —— 用于 IR、Pass、工具或诊断的 LLVM/Clang/MLIR 基础设施。
