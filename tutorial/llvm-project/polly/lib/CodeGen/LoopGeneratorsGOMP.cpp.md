# LoopGeneratorsGOMP.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `polly/lib/CodeGen/LoopGeneratorsGOMP.cpp` | `polly/lib/CodeGen/LoopGeneratorsGOMP.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | Implements Polly code generation, AST lowering, and runtime emission logic. The leading comment describes it as: IR helper to create loops. | 实现 Polly 的代码生成、AST 降级与运行时生成逻辑。 文件开头注释将其概括为：IR helper to create loops。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-12

````cpp
//===------ LoopGeneratorsGOMP.cpp - IR helper to create loops ------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains functions to create parallel loops as LLVM-IR.
//
//===----------------------------------------------------------------------===//

````
- **EN**: This block records the standard LLVM file banner and license metadata; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 给出 LLVM 标准文件头与许可证元数据; 保留用于解释意图、用法或算法背景的注释.

### Lines 13-26

````cpp
#include "polly/CodeGen/LoopGeneratorsGOMP.h"
#include "llvm/Analysis/LoopInfo.h"
#include "llvm/IR/Dominators.h"
#include "llvm/IR/Module.h"

using namespace llvm;
using namespace polly;

void ParallelLoopGeneratorGOMP::createCallSpawnThreads(Value *SubFn,
                                                       Value *SubFnParam,
                                                       Value *LB, Value *UB,
                                                       Value *Stride) {
  const std::string Name = "GOMP_parallel_loop_runtime_start";

````
- **EN**: This block imports Polly, LLVM-family headers needed by the surrounding code; declares or defines routines around `createCallSpawnThreads`.
- **CN**: 该代码块 引入周边逻辑所需的 Polly、LLVM-family 头文件; 声明或定义与 `createCallSpawnThreads` 相关的例程.

### Lines 27-40

````cpp
  Function *F = M->getFunction(Name);

  // If F is not available, declare it.
  if (!F) {
    GlobalValue::LinkageTypes Linkage = Function::ExternalLinkage;

    Type *Params[] = {
        Builder.getPtrTy(), Builder.getPtrTy(), Builder.getInt32Ty(),
        LongType,           LongType,           LongType};

    FunctionType *Ty = FunctionType::get(Builder.getVoidTy(), Params, false);
    F = Function::Create(Ty, Linkage, Name, M);
  }

````
- **EN**: This block declares or defines routines around `getFunction`, `getPtrTy`, `get`, `Create`; contains control flow with 1 conditional check(s); preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `getFunction`, `getPtrTy`, `get`, `Create` 相关的例程; 包含控制流结构：1 处条件判断; 保留用于解释意图、用法或算法背景的注释.

### Lines 41-58

````cpp
  Value *Args[] = {SubFn, SubFnParam, Builder.getInt32(PollyNumThreads),
                   LB,    UB,         Stride};

  CallInst *Call = Builder.CreateCall(F, Args);
  Call->setDebugLoc(DLGenerated);
}

void ParallelLoopGeneratorGOMP::deployParallelExecution(Function *SubFn,
                                                        Value *SubFnParam,
                                                        Value *LB, Value *UB,
                                                        Value *Stride) {
  // Tell the runtime we start a parallel loop
  createCallSpawnThreads(SubFn, SubFnParam, LB, UB, Stride);
  CallInst *Call = Builder.CreateCall(SubFn, SubFnParam);
  Call->setDebugLoc(DLGenerated);
  createCallJoinThreads();
}

````
- **EN**: This block declares or defines routines around `getInt32`, `CreateCall`, `setDebugLoc`, `deployParallelExecution` (+2 more); preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `getInt32`, `CreateCall`, `setDebugLoc`, `deployParallelExecution` (+2 more) 相关的例程; 保留用于解释意图、用法或算法背景的注释.

### Lines 59-74

````cpp
Function *ParallelLoopGeneratorGOMP::prepareSubFnDefinition(Function *F) const {
  FunctionType *FT =
      FunctionType::get(Builder.getVoidTy(), {Builder.getPtrTy()}, false);
  Function *SubFn = Function::Create(FT, Function::InternalLinkage,
                                     F->getName() + "_polly_subfn", M);
  // Name the function's arguments
  SubFn->arg_begin()->setName("polly.par.userContext");
  return SubFn;
}

// Create a subfunction of the following (preliminary) structure:
//
//    PrevBB
//       |
//       v
//    HeaderBB
````
- **EN**: This block declares or defines routines around `prepareSubFnDefinition`, `get`, `Create`, `getName` (+1 more); emits return paths that hand results or status codes back to callers; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `prepareSubFnDefinition`, `get`, `Create`, `getName` (+1 more) 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 保留用于解释意图、用法或算法背景的注释.

### Lines 75-90

````cpp
//       |   _____
//       v  v    |
//   CheckNextBB  PreHeaderBB
//       |\       |
//       | \______/
//       |
//       v
//     ExitBB
//
// HeaderBB will hold allocations and loading of variables.
// CheckNextBB will check for more work.
// If there is more work to do: go to PreHeaderBB, otherwise go to ExitBB.
// PreHeaderBB loads the new boundaries (& will lead to the loop body later on).
// ExitBB marks the end of the parallel execution.
std::tuple<Value *, Function *>
ParallelLoopGeneratorGOMP::createSubFn(Value *Stride, AllocaInst *StructData,
````
- **EN**: This block declares or defines routines around `createSubFn`; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `createSubFn` 相关的例程; 保留用于解释意图、用法或算法背景的注释.

### Lines 91-104

````cpp
                                       SetVector<Value *> Data,
                                       ValueMapT &Map) {
  if (PollyScheduling != OMPGeneralSchedulingType::Runtime) {
    // User tried to influence the scheduling type (currently not supported)
    errs() << "warning: Polly's GNU OpenMP backend solely "
              "supports the scheduling type 'runtime'.\n";
  }

  if (PollyChunkSize != 0) {
    // User tried to influence the chunk size (currently not supported)
    errs() << "warning: Polly's GNU OpenMP backend solely "
              "supports the default chunk size.\n";
  }

````
- **EN**: This block declares or defines routines around `errs`; contains control flow with 2 conditional check(s); preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `errs` 相关的例程; 包含控制流结构：2 处条件判断; 保留用于解释意图、用法或算法背景的注释.

### Lines 105-121

````cpp
  Function *SubFn = createSubFnDefinition();
  LLVMContext &Context = SubFn->getContext();

  // Create basic blocks.
  BasicBlock *HeaderBB = BasicBlock::Create(Context, "polly.par.setup", SubFn);
  // Add terminator so that DT computation doesn't fail.
  auto *UI = new UnreachableInst(Context, HeaderBB);
  SubFnDT = std::make_unique<DominatorTree>(*SubFn);
  SubFnLI = std::make_unique<LoopInfo>(*SubFnDT);
  UI->eraseFromParent();

  BasicBlock *ExitBB = BasicBlock::Create(Context, "polly.par.exit", SubFn);
  BasicBlock *CheckNextBB =
      BasicBlock::Create(Context, "polly.par.checkNext", SubFn);
  BasicBlock *PreHeaderBB =
      BasicBlock::Create(Context, "polly.par.loadIVBounds", SubFn);

````
- **EN**: This block declares or defines routines around `createSubFnDefinition`, `getContext`, `Create`, `UnreachableInst` (+3 more); preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `createSubFnDefinition`, `getContext`, `Create`, `UnreachableInst` (+3 more) 相关的例程; 保留用于解释意图、用法或算法背景的注释.

### Lines 122-135

````cpp
  SubFnDT->addNewBlock(ExitBB, HeaderBB);
  SubFnDT->addNewBlock(CheckNextBB, HeaderBB);
  SubFnDT->addNewBlock(PreHeaderBB, HeaderBB);

  // Fill up basic block HeaderBB.
  Builder.SetInsertPoint(HeaderBB);
  Value *LBPtr = Builder.CreateAlloca(LongType, nullptr, "polly.par.LBPtr");
  Value *UBPtr = Builder.CreateAlloca(LongType, nullptr, "polly.par.UBPtr");
  Value *UserContext = &*SubFn->arg_begin();

  extractValuesFromStruct(Data, StructData->getAllocatedType(), UserContext,
                          Map);
  Builder.CreateBr(CheckNextBB);

````
- **EN**: This block declares or defines routines around `addNewBlock`, `SetInsertPoint`, `CreateAlloca`, `arg_begin` (+2 more); preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `addNewBlock`, `SetInsertPoint`, `CreateAlloca`, `arg_begin` (+2 more) 相关的例程; 保留用于解释意图、用法或算法背景的注释.

### Lines 136-147

````cpp
  // Add code to check if another set of iterations will be executed.
  Builder.SetInsertPoint(CheckNextBB);
  Value *Next = createCallGetWorkItem(LBPtr, UBPtr);
  Value *HasNextSchedule = Builder.CreateTrunc(
      Next, Builder.getInt1Ty(), "polly.par.hasNextScheduleBlock");
  Builder.CreateCondBr(HasNextSchedule, PreHeaderBB, ExitBB);

  // Add code to load the iv bounds for this set of iterations.
  Builder.SetInsertPoint(PreHeaderBB);
  Value *LB = Builder.CreateLoad(LongType, LBPtr, "polly.par.LB");
  Value *UB = Builder.CreateLoad(LongType, UBPtr, "polly.par.UB");

````
- **EN**: This block declares or defines routines around `SetInsertPoint`, `createCallGetWorkItem`, `CreateTrunc`, `getInt1Ty` (+2 more); mentions schedule-related state that orders statements or iterations; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `SetInsertPoint`, `createCallGetWorkItem`, `CreateTrunc`, `getInt1Ty` (+2 more) 相关的例程; 涉及调度相关状态，用于安排语句或迭代顺序; 保留用于解释意图、用法或算法背景的注释.

### Lines 148-159

````cpp
  // Subtract one as the upper bound provided by OpenMP is a < comparison
  // whereas the codegenForSequential function creates a <= comparison.
  UB = Builder.CreateSub(UB, ConstantInt::get(LongType, 1),
                         "polly.par.UBAdjusted");

  Builder.CreateBr(CheckNextBB);
  Builder.SetInsertPoint(std::prev(Builder.GetInsertPoint()));
  BasicBlock *AfterBB;
  Value *IV =
      createLoop(LB, UB, Stride, Builder, *SubFnLI, *SubFnDT, AfterBB,
                 ICmpInst::ICMP_SLE, nullptr, true, /* UseGuard */ false);

````
- **EN**: This block declares or defines routines around `CreateSub`, `CreateBr`, `SetInsertPoint`, `createLoop`; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `CreateSub`, `CreateBr`, `SetInsertPoint`, `createLoop` 相关的例程; 保留用于解释意图、用法或算法背景的注释.

### Lines 160-172

````cpp
  BasicBlock::iterator LoopBody = Builder.GetInsertPoint();

  // Add code to terminate this subfunction.
  Builder.SetInsertPoint(ExitBB);
  createCallCleanupThread();
  Builder.CreateRetVoid();

  Builder.SetInsertPoint(LoopBody);

  // FIXME: Call SubFnDT->verify() and SubFnLI->verify() to check that the
  // DominatorTree/LoopInfo has been created correctly. Alternatively, recreate
  // from scratch since it is not needed here directly.

````
- **EN**: This block declares or defines routines around `GetInsertPoint`, `SetInsertPoint`, `createCallCleanupThread`, `CreateRetVoid`; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `GetInsertPoint`, `SetInsertPoint`, `createCallCleanupThread`, `CreateRetVoid` 相关的例程; 保留用于解释意图、用法或算法背景的注释.

### Lines 173-189

````cpp
  return std::make_tuple(IV, SubFn);
}

Value *ParallelLoopGeneratorGOMP::createCallGetWorkItem(Value *LBPtr,
                                                        Value *UBPtr) {
  const std::string Name = "GOMP_loop_runtime_next";

  Function *F = M->getFunction(Name);

  // If F is not available, declare it.
  if (!F) {
    GlobalValue::LinkageTypes Linkage = Function::ExternalLinkage;
    Type *Params[] = {Builder.getPtrTy(0), Builder.getPtrTy(0)};
    FunctionType *Ty = FunctionType::get(Builder.getInt8Ty(), Params, false);
    F = Function::Create(Ty, Linkage, Name, M);
  }

````
- **EN**: This block declares or defines routines around `createCallGetWorkItem`, `getFunction`, `getPtrTy`, `get` (+1 more); contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `createCallGetWorkItem`, `getFunction`, `getPtrTy`, `get` (+1 more) 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 保留用于解释意图、用法或算法背景的注释.

### Lines 190-202

````cpp
  Value *Args[] = {LBPtr, UBPtr};
  CallInst *Call = Builder.CreateCall(F, Args);
  Call->setDebugLoc(DLGenerated);
  Value *Return = Builder.CreateICmpNE(
      Call, Builder.CreateZExt(Builder.getFalse(), Call->getType()));
  return Return;
}

void ParallelLoopGeneratorGOMP::createCallJoinThreads() {
  const std::string Name = "GOMP_parallel_end";

  Function *F = M->getFunction(Name);

````
- **EN**: This block declares or defines routines around `CreateCall`, `setDebugLoc`, `CreateICmpNE`, `CreateZExt` (+2 more); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `CreateCall`, `setDebugLoc`, `CreateICmpNE`, `CreateZExt` (+2 more) 相关的例程; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 203-214

````cpp
  // If F is not available, declare it.
  if (!F) {
    GlobalValue::LinkageTypes Linkage = Function::ExternalLinkage;

    FunctionType *Ty = FunctionType::get(Builder.getVoidTy(), false);
    F = Function::Create(Ty, Linkage, Name, M);
  }

  CallInst *Call = Builder.CreateCall(F, {});
  Call->setDebugLoc(DLGenerated);
}

````
- **EN**: This block declares or defines routines around `get`, `Create`, `CreateCall`, `setDebugLoc`; contains control flow with 1 conditional check(s); preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `get`, `Create`, `CreateCall`, `setDebugLoc` 相关的例程; 包含控制流结构：1 处条件判断; 保留用于解释意图、用法或算法背景的注释.

### Lines 215-230

````cpp
void ParallelLoopGeneratorGOMP::createCallCleanupThread() {
  const std::string Name = "GOMP_loop_end_nowait";

  Function *F = M->getFunction(Name);

  // If F is not available, declare it.
  if (!F) {
    GlobalValue::LinkageTypes Linkage = Function::ExternalLinkage;

    FunctionType *Ty = FunctionType::get(Builder.getVoidTy(), false);
    F = Function::Create(Ty, Linkage, Name, M);
  }

  CallInst *Call = Builder.CreateCall(F, {});
  Call->setDebugLoc(DLGenerated);
}
````
- **EN**: This block declares or defines routines around `createCallCleanupThread`, `getFunction`, `get`, `Create` (+2 more); contains control flow with 1 conditional check(s); preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `createCallCleanupThread`, `getFunction`, `get`, `Create` (+2 more) 相关的例程; 包含控制流结构：1 处条件判断; 保留用于解释意图、用法或算法背景的注释.

## Key Concepts / 关键概念

- **Code generation**
  - **CN**: 代码生成
- **OpenMP/runtime support**
  - **CN**: OpenMP/运行时支持
- **Loop transformation**
  - **CN**: 循环变换
- **Debug and diagnostics**
  - **CN**: 调试与诊断
- **Schedule construction**
  - **CN**: 调度构建

## Dependencies / 依赖关系

- **Polly headers**: `polly/CodeGen/LoopGeneratorsGOMP.h` — Direct Polly APIs, data structures, or pass interfaces used here.
  **Polly headers（CN）**：`polly/CodeGen/LoopGeneratorsGOMP.h` —— 此处直接使用的 Polly API、数据结构或 Pass 接口。
- **LLVM-family headers**: `llvm/Analysis/LoopInfo.h`, `llvm/IR/Dominators.h`, `llvm/IR/Module.h` — LLVM/Clang/MLIR infrastructure for IR, passes, utilities, or diagnostics.
  **LLVM-family headers（CN）**：`llvm/Analysis/LoopInfo.h`, `llvm/IR/Dominators.h`, `llvm/IR/Module.h` —— 用于 IR、Pass、工具或诊断的 LLVM/Clang/MLIR 基础设施。
