# LoopGeneratorsKMP.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `polly/lib/CodeGen/LoopGeneratorsKMP.cpp` | `polly/lib/CodeGen/LoopGeneratorsKMP.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | Implements Polly code generation, AST lowering, and runtime emission logic. The leading comment describes it as: IR helper to create loops. | 实现 Polly 的代码生成、AST 降级与运行时生成逻辑。 文件开头注释将其概括为：IR helper to create loops。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````cpp
//===------ LoopGeneratorsKMP.cpp - IR helper to create loops -------------===//
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

#include "polly/CodeGen/LoopGeneratorsKMP.h"
#include "llvm/Analysis/LoopInfo.h"
#include "llvm/IR/Dominators.h"
#include "llvm/IR/Module.h"

using namespace llvm;
using namespace polly;

````
- **EN**: This block records the standard LLVM file banner and license metadata; imports Polly, LLVM-family headers needed by the surrounding code; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 给出 LLVM 标准文件头与许可证元数据; 引入周边逻辑所需的 Polly、LLVM-family 头文件; 保留用于解释意图、用法或算法背景的注释.

### Lines 21-41

````cpp
void ParallelLoopGeneratorKMP::createCallSpawnThreads(Value *SubFn,
                                                      Value *SubFnParam,
                                                      Value *LB, Value *UB,
                                                      Value *Stride) {
  const std::string Name = "__kmpc_fork_call";
  Function *F = M->getFunction(Name);
  Type *KMPCMicroTy = StructType::getTypeByName(M->getContext(), "kmpc_micro");

  if (!KMPCMicroTy) {
    // void (*kmpc_micro)(kmp_int32 *global_tid, kmp_int32 *bound_tid, ...)
    Type *MicroParams[] = {Builder.getPtrTy(0), Builder.getPtrTy(0)};

    KMPCMicroTy = FunctionType::get(Builder.getVoidTy(), MicroParams, true);
  }

  // If F is not available, declare it.
  if (!F) {
    GlobalValue::LinkageTypes Linkage = Function::ExternalLinkage;
    Type *Params[] = {Builder.getPtrTy(0), Builder.getInt32Ty(),
                      Builder.getPtrTy(0)};

````
- **EN**: This block declares or defines routines around `createCallSpawnThreads`, `getFunction`, `getTypeByName`, `getPtrTy` (+1 more); contains control flow with 2 conditional check(s); preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `createCallSpawnThreads`, `getFunction`, `getTypeByName`, `getPtrTy` (+1 more) 相关的例程; 包含控制流结构：2 处条件判断; 保留用于解释意图、用法或算法背景的注释.

### Lines 42-65

````cpp
    FunctionType *Ty = FunctionType::get(Builder.getVoidTy(), Params, true);
    F = Function::Create(Ty, Linkage, Name, M);
  }

  Value *Task =
      Builder.CreatePointerBitCastOrAddrSpaceCast(SubFn, Builder.getPtrTy(0));

  Value *Args[] = {SourceLocationInfo,
                   Builder.getInt32(4) /* Number of arguments (w/o Task) */,
                   Task,
                   LB,
                   UB,
                   Stride,
                   SubFnParam};

  CallInst *Call = Builder.CreateCall(F, Args);
  Call->setDebugLoc(DLGenerated);
}

void ParallelLoopGeneratorKMP::deployParallelExecution(Function *SubFn,
                                                       Value *SubFnParam,
                                                       Value *LB, Value *UB,
                                                       Value *Stride) {
  // Inform OpenMP runtime about the number of threads if greater than zero
````
- **EN**: This block declares or defines routines around `get`, `Create`, `CreatePointerBitCastOrAddrSpaceCast`, `getInt32` (+3 more); preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `get`, `Create`, `CreatePointerBitCastOrAddrSpaceCast`, `getInt32` (+3 more) 相关的例程; 保留用于解释意图、用法或算法背景的注释.

### Lines 66-89

````cpp
  if (PollyNumThreads > 0) {
    Value *GlobalThreadID = createCallGlobalThreadNum();
    createCallPushNumThreads(GlobalThreadID, Builder.getInt32(PollyNumThreads));
  }

  // Tell the runtime we start a parallel loop
  createCallSpawnThreads(SubFn, SubFnParam, LB, UB, Stride);
}

Function *ParallelLoopGeneratorKMP::prepareSubFnDefinition(Function *F) const {
  std::vector<Type *> Arguments = {
      Builder.getPtrTy(0), Builder.getPtrTy(0), LongType, LongType, LongType,
      Builder.getPtrTy()};

  FunctionType *FT = FunctionType::get(Builder.getVoidTy(), Arguments, false);
  Function *SubFn = Function::Create(FT, Function::InternalLinkage,
                                     F->getName() + "_polly_subfn", M);
  // Name the function's arguments
  Function::arg_iterator AI = SubFn->arg_begin();
  AI->setName("polly.kmpc.global_tid");
  std::advance(AI, 1);
  AI->setName("polly.kmpc.bound_tid");
  std::advance(AI, 1);
  AI->setName("polly.kmpc.lb");
````
- **EN**: This block declares or defines routines around `createCallGlobalThreadNum`, `createCallPushNumThreads`, `createCallSpawnThreads`, `prepareSubFnDefinition` (+7 more); contains control flow with 1 conditional check(s); preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `createCallGlobalThreadNum`, `createCallPushNumThreads`, `createCallSpawnThreads`, `prepareSubFnDefinition` (+7 more) 相关的例程; 包含控制流结构：1 处条件判断; 保留用于解释意图、用法或算法背景的注释.

### Lines 90-113

````cpp
  std::advance(AI, 1);
  AI->setName("polly.kmpc.ub");
  std::advance(AI, 1);
  AI->setName("polly.kmpc.inc");
  std::advance(AI, 1);
  AI->setName("polly.kmpc.shared");

  return SubFn;
}

// Create a subfunction of the following (preliminary) structure:
//
//        PrevBB
//           |
//           v
//        HeaderBB
//       /   |    _____
//      /    v   v     |
//     / PreHeaderBB   |
//    |      |         |
//    |      v         |
//    |  CheckNextBB   |
//     \     |   \_____/
//      \    |
````
- **EN**: This block declares or defines routines around `advance`, `setName`; emits return paths that hand results or status codes back to callers; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `advance`, `setName` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 保留用于解释意图、用法或算法背景的注释.

### Lines 114-139

````cpp
//       v   v
//       ExitBB
//
// HeaderBB will hold allocations, loading of variables and kmp-init calls.
// CheckNextBB will check for more work (dynamic / static chunked) or will be
// empty (static non chunked).
// If there is more work to do: go to PreHeaderBB, otherwise go to ExitBB.
// PreHeaderBB loads the new boundaries (& will lead to the loop body later on).
// Just like CheckNextBB: PreHeaderBB is (preliminary) empty in the static non
// chunked scheduling case. ExitBB marks the end of the parallel execution.
// The possibly empty BasicBlocks will automatically be removed.
std::tuple<Value *, Function *>
ParallelLoopGeneratorKMP::createSubFn(Value *SequentialLoopStride,
                                      AllocaInst *StructData,
                                      SetVector<Value *> Data, ValueMapT &Map) {
  Function *SubFn = createSubFnDefinition();
  LLVMContext &Context = SubFn->getContext();

  // Create basic blocks.
  BasicBlock *HeaderBB = BasicBlock::Create(Context, "polly.par.setup", SubFn);
  // Add terminator so that DT computation doesn't fail.
  auto *UI = new UnreachableInst(Context, HeaderBB);
  SubFnDT = std::make_unique<DominatorTree>(*SubFn);
  SubFnLI = std::make_unique<LoopInfo>(*SubFnDT);
  UI->eraseFromParent();

````
- **EN**: This block declares or defines routines around `createSubFn`, `createSubFnDefinition`, `getContext`, `Create` (+4 more); preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `createSubFn`, `createSubFnDefinition`, `getContext`, `Create` (+4 more) 相关的例程; 保留用于解释意图、用法或算法背景的注释.

### Lines 140-163

````cpp
  BasicBlock *ExitBB = BasicBlock::Create(Context, "polly.par.exit", SubFn);
  BasicBlock *CheckNextBB =
      BasicBlock::Create(Context, "polly.par.checkNext", SubFn);
  BasicBlock *PreHeaderBB =
      BasicBlock::Create(Context, "polly.par.loadIVBounds", SubFn);

  SubFnDT->addNewBlock(ExitBB, HeaderBB);
  SubFnDT->addNewBlock(CheckNextBB, HeaderBB);
  SubFnDT->addNewBlock(PreHeaderBB, HeaderBB);

  // Fill up basic block HeaderBB.
  Builder.SetInsertPoint(HeaderBB);
  Value *LBPtr = Builder.CreateAlloca(LongType, nullptr, "polly.par.LBPtr");
  Value *UBPtr = Builder.CreateAlloca(LongType, nullptr, "polly.par.UBPtr");
  Value *IsLastPtr = Builder.CreateAlloca(Builder.getInt32Ty(), nullptr,
                                          "polly.par.lastIterPtr");
  Value *StridePtr =
      Builder.CreateAlloca(LongType, nullptr, "polly.par.StridePtr");

  // Get iterator for retrieving the previously defined parameters.
  Function::arg_iterator AI = SubFn->arg_begin();
  // First argument holds "global thread ID".
  Value *IDPtr = &*AI;
  // Skip "bound thread ID" since it is not used (but had to be defined).
````
- **EN**: This block declares or defines routines around `Create`, `addNewBlock`, `SetInsertPoint`, `CreateAlloca` (+1 more); preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `Create`, `addNewBlock`, `SetInsertPoint`, `CreateAlloca` (+1 more) 相关的例程; 保留用于解释意图、用法或算法背景的注释.

### Lines 164-184

````cpp
  std::advance(AI, 2);
  // Move iterator to: LB, UB, Stride, Shared variable struct.
  Value *LB = &*AI;
  std::advance(AI, 1);
  Value *UB = &*AI;
  std::advance(AI, 1);
  Value *Stride = &*AI;
  std::advance(AI, 1);
  Value *Shared = &*AI;

  extractValuesFromStruct(Data, StructData->getAllocatedType(), Shared, Map);

  const auto Alignment = llvm::Align(is64BitArch() ? 8 : 4);
  Value *ID = Builder.CreateAlignedLoad(Builder.getInt32Ty(), IDPtr, Alignment,
                                        "polly.par.global_tid");

  Builder.CreateAlignedStore(LB, LBPtr, Alignment);
  Builder.CreateAlignedStore(UB, UBPtr, Alignment);
  Builder.CreateAlignedStore(Builder.getInt32(0), IsLastPtr, Alignment);
  Builder.CreateAlignedStore(Stride, StridePtr, Alignment);

````
- **EN**: This block declares or defines routines around `advance`, `extractValuesFromStruct`, `Align`, `CreateAlignedLoad` (+1 more); preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `advance`, `extractValuesFromStruct`, `Align`, `CreateAlignedLoad` (+1 more) 相关的例程; 保留用于解释意图、用法或算法背景的注释.

### Lines 185-210

````cpp
  // Subtract one as the upper bound provided by openmp is a < comparison
  // whereas the codegenForSequential function creates a <= comparison.
  Value *AdjustedUB = Builder.CreateAdd(UB, ConstantInt::get(LongType, -1),
                                        "polly.indvar.UBAdjusted");

  Value *ChunkSize =
      ConstantInt::get(LongType, std::max<int>(PollyChunkSize, 1));

  OMPGeneralSchedulingType Scheduling =
      getSchedType(PollyChunkSize, PollyScheduling);

  switch (Scheduling) {
  case OMPGeneralSchedulingType::Dynamic:
  case OMPGeneralSchedulingType::Guided:
  case OMPGeneralSchedulingType::Runtime:
    // "DYNAMIC" scheduling types are handled below (including 'runtime')
    {
      UB = AdjustedUB;
      createCallDispatchInit(ID, LB, UB, Stride, ChunkSize);
      Value *HasWork =
          createCallDispatchNext(ID, IsLastPtr, LBPtr, UBPtr, StridePtr);
      Value *HasIteration =
          Builder.CreateICmp(llvm::CmpInst::Predicate::ICMP_EQ, HasWork,
                             Builder.getInt32(1), "polly.hasIteration");
      Builder.CreateCondBr(HasIteration, PreHeaderBB, ExitBB);

````
- **EN**: This block declares or defines routines around `CreateAdd`, `get`, `getSchedType`, `createCallDispatchInit` (+4 more); contains control flow with 1 switch dispatch(es); preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `CreateAdd`, `get`, `getSchedType`, `createCallDispatchInit` (+4 more) 相关的例程; 包含控制流结构：1 处分支派发; 保留用于解释意图、用法或算法背景的注释.

### Lines 211-231

````cpp
      Builder.SetInsertPoint(CheckNextBB);
      HasWork = createCallDispatchNext(ID, IsLastPtr, LBPtr, UBPtr, StridePtr);
      HasIteration =
          Builder.CreateICmp(llvm::CmpInst::Predicate::ICMP_EQ, HasWork,
                             Builder.getInt32(1), "polly.hasWork");
      Builder.CreateCondBr(HasIteration, PreHeaderBB, ExitBB);

      Builder.SetInsertPoint(PreHeaderBB);
      LB = Builder.CreateAlignedLoad(LongType, LBPtr, Alignment,
                                     "polly.indvar.LB");
      UB = Builder.CreateAlignedLoad(LongType, UBPtr, Alignment,
                                     "polly.indvar.UB");
    }
    break;
  case OMPGeneralSchedulingType::StaticChunked:
  case OMPGeneralSchedulingType::StaticNonChunked:
    // "STATIC" scheduling types are handled below
    {
      Builder.CreateAlignedStore(AdjustedUB, UBPtr, Alignment);
      createCallStaticInit(ID, IsLastPtr, LBPtr, UBPtr, StridePtr, ChunkSize);

````
- **EN**: This block declares or defines routines around `SetInsertPoint`, `createCallDispatchNext`, `CreateICmp`, `getInt32` (+4 more); preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `SetInsertPoint`, `createCallDispatchNext`, `CreateICmp`, `getInt32` (+4 more) 相关的例程; 保留用于解释意图、用法或算法背景的注释.

### Lines 232-257

````cpp
      Value *ChunkedStride = Builder.CreateAlignedLoad(
          LongType, StridePtr, Alignment, "polly.kmpc.stride");

      LB = Builder.CreateAlignedLoad(LongType, LBPtr, Alignment,
                                     "polly.indvar.LB");
      UB = Builder.CreateAlignedLoad(LongType, UBPtr, Alignment,
                                     "polly.indvar.UB.temp");

      Value *UBInRange =
          Builder.CreateICmp(llvm::CmpInst::Predicate::ICMP_SLE, UB, AdjustedUB,
                             "polly.indvar.UB.inRange");
      UB = Builder.CreateSelect(UBInRange, UB, AdjustedUB, "polly.indvar.UB");
      Builder.CreateAlignedStore(UB, UBPtr, Alignment);

      Value *HasIteration = Builder.CreateICmp(
          llvm::CmpInst::Predicate::ICMP_SLE, LB, UB, "polly.hasIteration");
      Builder.CreateCondBr(HasIteration, PreHeaderBB, ExitBB);

      if (Scheduling == OMPGeneralSchedulingType::StaticChunked) {
        Builder.SetInsertPoint(PreHeaderBB);
        LB = Builder.CreateAlignedLoad(LongType, LBPtr, Alignment,
                                       "polly.indvar.LB.entry");
        UB = Builder.CreateAlignedLoad(LongType, UBPtr, Alignment,
                                       "polly.indvar.UB.entry");
      }

````
- **EN**: This block declares or defines routines around `CreateAlignedLoad`, `CreateICmp`, `CreateSelect`, `CreateAlignedStore` (+2 more); contains control flow with 1 conditional check(s).
- **CN**: 该代码块 声明或定义与 `CreateAlignedLoad`, `CreateICmp`, `CreateSelect`, `CreateAlignedStore` (+2 more) 相关的例程; 包含控制流结构：1 处条件判断.

### Lines 258-281

````cpp
      Builder.SetInsertPoint(CheckNextBB);

      if (Scheduling == OMPGeneralSchedulingType::StaticChunked) {
        Value *NextLB =
            Builder.CreateAdd(LB, ChunkedStride, "polly.indvar.nextLB");
        Value *NextUB = Builder.CreateAdd(UB, ChunkedStride);

        Value *NextUBOutOfBounds =
            Builder.CreateICmp(llvm::CmpInst::Predicate::ICMP_SGT, NextUB,
                               AdjustedUB, "polly.indvar.nextUB.outOfBounds");
        NextUB = Builder.CreateSelect(NextUBOutOfBounds, AdjustedUB, NextUB,
                                      "polly.indvar.nextUB");

        Builder.CreateAlignedStore(NextLB, LBPtr, Alignment);
        Builder.CreateAlignedStore(NextUB, UBPtr, Alignment);

        Value *HasWork =
            Builder.CreateICmp(llvm::CmpInst::Predicate::ICMP_SLE, NextLB,
                               AdjustedUB, "polly.hasWork");
        Builder.CreateCondBr(HasWork, PreHeaderBB, ExitBB);
      } else {
        Builder.CreateBr(ExitBB);
      }

````
- **EN**: This block declares or defines routines around `SetInsertPoint`, `CreateAdd`, `CreateICmp`, `CreateSelect` (+3 more); contains control flow with 1 conditional check(s).
- **CN**: 该代码块 声明或定义与 `SetInsertPoint`, `CreateAdd`, `CreateICmp`, `CreateSelect` (+3 more) 相关的例程; 包含控制流结构：1 处条件判断.

### Lines 282-305

````cpp
      Builder.SetInsertPoint(PreHeaderBB);
    }
    break;
  }

  Builder.CreateBr(CheckNextBB);
  Builder.SetInsertPoint(std::prev(Builder.GetInsertPoint()));
  BasicBlock *AfterBB;
  Value *IV = createLoop(LB, UB, SequentialLoopStride, Builder, *SubFnLI,
                         *SubFnDT, AfterBB, ICmpInst::ICMP_SLE, nullptr, true,
                         /* UseGuard */ false);

  BasicBlock::iterator LoopBody = Builder.GetInsertPoint();

  // Add code to terminate this subfunction.
  Builder.SetInsertPoint(ExitBB);
  // Static (i.e. non-dynamic) scheduling types, are terminated with a fini-call
  if (Scheduling == OMPGeneralSchedulingType::StaticChunked ||
      Scheduling == OMPGeneralSchedulingType::StaticNonChunked) {
    createCallStaticFini(ID);
  }
  Builder.CreateRetVoid();
  Builder.SetInsertPoint(LoopBody);

````
- **EN**: This block declares or defines routines around `SetInsertPoint`, `CreateBr`, `createLoop`, `GetInsertPoint` (+2 more); contains control flow with 1 conditional check(s); preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `SetInsertPoint`, `CreateBr`, `createLoop`, `GetInsertPoint` (+2 more) 相关的例程; 包含控制流结构：1 处条件判断; 保留用于解释意图、用法或算法背景的注释.

### Lines 306-325

````cpp
  // FIXME: Call SubFnDT->verify() and SubFnLI->verify() to check that the
  // DominatorTree/LoopInfo has been created correctly. Alternatively, recreate
  // from scratch since it is not needed here directly.

  return std::make_tuple(IV, SubFn);
}

Value *ParallelLoopGeneratorKMP::createCallGlobalThreadNum() {
  const std::string Name = "__kmpc_global_thread_num";
  Function *F = M->getFunction(Name);

  // If F is not available, declare it.
  if (!F) {
    GlobalValue::LinkageTypes Linkage = Function::ExternalLinkage;
    Type *Params[] = {Builder.getPtrTy(0)};

    FunctionType *Ty = FunctionType::get(Builder.getInt32Ty(), Params, false);
    F = Function::Create(Ty, Linkage, Name, M);
  }

````
- **EN**: This block declares or defines routines around `createCallGlobalThreadNum`, `getFunction`, `getPtrTy`, `get` (+1 more); contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `createCallGlobalThreadNum`, `getFunction`, `getPtrTy`, `get` (+1 more) 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 保留用于解释意图、用法或算法背景的注释.

### Lines 326-345

````cpp
  CallInst *Call = Builder.CreateCall(F, {SourceLocationInfo});
  Call->setDebugLoc(DLGenerated);
  return Call;
}

void ParallelLoopGeneratorKMP::createCallPushNumThreads(Value *GlobalThreadID,
                                                        Value *NumThreads) {
  const std::string Name = "__kmpc_push_num_threads";
  Function *F = M->getFunction(Name);

  // If F is not available, declare it.
  if (!F) {
    GlobalValue::LinkageTypes Linkage = Function::ExternalLinkage;
    Type *Params[] = {Builder.getPtrTy(0), Builder.getInt32Ty(),
                      Builder.getInt32Ty()};

    FunctionType *Ty = FunctionType::get(Builder.getVoidTy(), Params, false);
    F = Function::Create(Ty, Linkage, Name, M);
  }

````
- **EN**: This block declares or defines routines around `CreateCall`, `setDebugLoc`, `createCallPushNumThreads`, `getFunction` (+4 more); contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `CreateCall`, `setDebugLoc`, `createCallPushNumThreads`, `getFunction` (+4 more) 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 保留用于解释意图、用法或算法背景的注释.

### Lines 346-369

````cpp
  Value *Args[] = {SourceLocationInfo, GlobalThreadID, NumThreads};

  CallInst *Call = Builder.CreateCall(F, Args);
  Call->setDebugLoc(DLGenerated);
}

void ParallelLoopGeneratorKMP::createCallStaticInit(Value *GlobalThreadID,
                                                    Value *IsLastPtr,
                                                    Value *LBPtr, Value *UBPtr,
                                                    Value *StridePtr,
                                                    Value *ChunkSize) {
  const std::string Name =
      is64BitArch() ? "__kmpc_for_static_init_8" : "__kmpc_for_static_init_4";
  Function *F = M->getFunction(Name);

  // If F is not available, declare it.
  if (!F) {
    GlobalValue::LinkageTypes Linkage = Function::ExternalLinkage;

    Type *Params[] = {Builder.getPtrTy(0),
                      Builder.getInt32Ty(),
                      Builder.getInt32Ty(),
                      Builder.getPtrTy(0),
                      Builder.getPtrTy(0),
````
- **EN**: This block declares or defines routines around `CreateCall`, `setDebugLoc`, `createCallStaticInit`, `is64BitArch` (+3 more); contains control flow with 1 conditional check(s); preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `CreateCall`, `setDebugLoc`, `createCallStaticInit`, `is64BitArch` (+3 more) 相关的例程; 包含控制流结构：1 处条件判断; 保留用于解释意图、用法或算法背景的注释.

### Lines 370-391

````cpp
                      Builder.getPtrTy(0),
                      Builder.getPtrTy(0),
                      LongType,
                      LongType};

    FunctionType *Ty = FunctionType::get(Builder.getVoidTy(), Params, false);
    F = Function::Create(Ty, Linkage, Name, M);
  }

  // The parameter 'ChunkSize' will hold strictly positive integer values,
  // regardless of PollyChunkSize's value
  Value *Args[] = {
      SourceLocationInfo,
      GlobalThreadID,
      Builder.getInt32(int(getSchedType(PollyChunkSize, PollyScheduling))),
      IsLastPtr,
      LBPtr,
      UBPtr,
      StridePtr,
      ConstantInt::get(LongType, 1),
      ChunkSize};

````
- **EN**: This block declares or defines routines around `getPtrTy`, `get`, `Create`, `getInt32`; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `getPtrTy`, `get`, `Create`, `getInt32` 相关的例程; 保留用于解释意图、用法或算法背景的注释.

### Lines 392-413

````cpp
  CallInst *Call = Builder.CreateCall(F, Args);
  Call->setDebugLoc(DLGenerated);
}

void ParallelLoopGeneratorKMP::createCallStaticFini(Value *GlobalThreadID) {
  const std::string Name = "__kmpc_for_static_fini";
  Function *F = M->getFunction(Name);

  // If F is not available, declare it.
  if (!F) {
    GlobalValue::LinkageTypes Linkage = Function::ExternalLinkage;
    Type *Params[] = {Builder.getPtrTy(0), Builder.getInt32Ty()};
    FunctionType *Ty = FunctionType::get(Builder.getVoidTy(), Params, false);
    F = Function::Create(Ty, Linkage, Name, M);
  }

  Value *Args[] = {SourceLocationInfo, GlobalThreadID};

  CallInst *Call = Builder.CreateCall(F, Args);
  Call->setDebugLoc(DLGenerated);
}

````
- **EN**: This block declares or defines routines around `CreateCall`, `setDebugLoc`, `createCallStaticFini`, `getFunction` (+3 more); contains control flow with 1 conditional check(s); preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `CreateCall`, `setDebugLoc`, `createCallStaticFini`, `getFunction` (+3 more) 相关的例程; 包含控制流结构：1 处条件判断; 保留用于解释意图、用法或算法背景的注释.

### Lines 414-433

````cpp
void ParallelLoopGeneratorKMP::createCallDispatchInit(Value *GlobalThreadID,
                                                      Value *LB, Value *UB,
                                                      Value *Inc,
                                                      Value *ChunkSize) {
  const std::string Name =
      is64BitArch() ? "__kmpc_dispatch_init_8" : "__kmpc_dispatch_init_4";
  Function *F = M->getFunction(Name);

  // If F is not available, declare it.
  if (!F) {
    GlobalValue::LinkageTypes Linkage = Function::ExternalLinkage;

    Type *Params[] = {Builder.getPtrTy(0),
                      Builder.getInt32Ty(),
                      Builder.getInt32Ty(),
                      LongType,
                      LongType,
                      LongType,
                      LongType};

````
- **EN**: This block declares or defines routines around `createCallDispatchInit`, `is64BitArch`, `getFunction`, `getPtrTy` (+1 more); contains control flow with 1 conditional check(s); preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `createCallDispatchInit`, `is64BitArch`, `getFunction`, `getPtrTy` (+1 more) 相关的例程; 包含控制流结构：1 处条件判断; 保留用于解释意图、用法或算法背景的注释.

### Lines 434-461

````cpp
    FunctionType *Ty = FunctionType::get(Builder.getVoidTy(), Params, false);
    F = Function::Create(Ty, Linkage, Name, M);
  }

  // The parameter 'ChunkSize' will hold strictly positive integer values,
  // regardless of PollyChunkSize's value
  Value *Args[] = {
      SourceLocationInfo,
      GlobalThreadID,
      Builder.getInt32(int(getSchedType(PollyChunkSize, PollyScheduling))),
      LB,
      UB,
      Inc,
      ChunkSize};

  CallInst *Call = Builder.CreateCall(F, Args);
  Call->setDebugLoc(DLGenerated);
}

Value *ParallelLoopGeneratorKMP::createCallDispatchNext(Value *GlobalThreadID,
                                                        Value *IsLastPtr,
                                                        Value *LBPtr,
                                                        Value *UBPtr,
                                                        Value *StridePtr) {
  const std::string Name =
      is64BitArch() ? "__kmpc_dispatch_next_8" : "__kmpc_dispatch_next_4";
  Function *F = M->getFunction(Name);

````
- **EN**: This block declares or defines routines around `get`, `Create`, `getInt32`, `CreateCall` (+4 more); preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `get`, `Create`, `getInt32`, `CreateCall` (+4 more) 相关的例程; 保留用于解释意图、用法或算法背景的注释.

### Lines 462-481

````cpp
  // If F is not available, declare it.
  if (!F) {
    GlobalValue::LinkageTypes Linkage = Function::ExternalLinkage;

    Type *Params[] = {Builder.getPtrTy(0), Builder.getInt32Ty(),
                      Builder.getPtrTy(0), Builder.getPtrTy(0),
                      Builder.getPtrTy(0), Builder.getPtrTy(0)};

    FunctionType *Ty = FunctionType::get(Builder.getInt32Ty(), Params, false);
    F = Function::Create(Ty, Linkage, Name, M);
  }

  Value *Args[] = {SourceLocationInfo, GlobalThreadID, IsLastPtr, LBPtr, UBPtr,
                   StridePtr};

  CallInst *Call = Builder.CreateCall(F, Args);
  Call->setDebugLoc(DLGenerated);
  return Call;
}

````
- **EN**: This block declares or defines routines around `getPtrTy`, `get`, `Create`, `CreateCall` (+1 more); contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `getPtrTy`, `get`, `Create`, `CreateCall` (+1 more) 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 保留用于解释意图、用法或算法背景的注释.

### Lines 482-503

````cpp
// TODO: This function currently creates a source location dummy. It might be
// necessary to (actually) provide information, in the future.
GlobalVariable *ParallelLoopGeneratorKMP::createSourceLocation() {
  const std::string LocName = ".loc.dummy";
  GlobalVariable *SourceLocDummy = M->getGlobalVariable(LocName);

  if (SourceLocDummy == nullptr) {
    const std::string StructName = "struct.ident_t";
    StructType *IdentTy =
        StructType::getTypeByName(M->getContext(), StructName);

    // If the ident_t StructType is not available, declare it.
    // in LLVM-IR: ident_t = type { i32, i32, i32, i32, i8* }
    if (!IdentTy) {
      Type *LocMembers[] = {Builder.getInt32Ty(), Builder.getInt32Ty(),
                            Builder.getInt32Ty(), Builder.getInt32Ty(),
                            Builder.getPtrTy()};

      IdentTy =
          StructType::create(M->getContext(), LocMembers, StructName, false);
    }

````
- **EN**: This block declares or defines routines around `createSourceLocation`, `getGlobalVariable`, `getTypeByName`, `getInt32Ty` (+2 more); contains control flow with 2 conditional check(s); preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `createSourceLocation`, `getGlobalVariable`, `getTypeByName`, `getInt32Ty` (+2 more) 相关的例程; 包含控制流结构：2 处条件判断; 保留用于解释意图、用法或算法背景的注释.

### Lines 504-523

````cpp
    const auto ArrayType =
        llvm::ArrayType::get(Builder.getInt8Ty(), /* Length */ 23);

    // Global Variable Definitions
    GlobalVariable *StrVar =
        new GlobalVariable(*M, ArrayType, true, GlobalValue::PrivateLinkage,
                           nullptr, ".str.ident");
    StrVar->setAlignment(llvm::Align(1));

    SourceLocDummy = new GlobalVariable(
        *M, IdentTy, true, GlobalValue::PrivateLinkage, nullptr, LocName);
    SourceLocDummy->setAlignment(llvm::Align(8));

    // Constant Definitions
    Constant *InitStr = ConstantDataArray::getString(
        M->getContext(), "Source location dummy.", true);

    Constant *StrPtr = static_cast<Constant *>(Builder.CreateInBoundsGEP(
        ArrayType, StrVar, {Builder.getInt32(0), Builder.getInt32(0)}));

````
- **EN**: This block declares or defines routines around `get`, `GlobalVariable`, `setAlignment`, `getString` (+3 more); preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `get`, `GlobalVariable`, `setAlignment`, `getString` (+3 more) 相关的例程; 保留用于解释意图、用法或算法背景的注释.

### Lines 524-546

````cpp
    Constant *LocInitStruct = ConstantStruct::get(
        IdentTy, {Builder.getInt32(0), Builder.getInt32(0), Builder.getInt32(0),
                  Builder.getInt32(0), StrPtr});

    // Initialize variables
    StrVar->setInitializer(InitStr);
    SourceLocDummy->setInitializer(LocInitStruct);
  }

  return SourceLocDummy;
}

bool ParallelLoopGeneratorKMP::is64BitArch() {
  return (LongType->getIntegerBitWidth() == 64);
}

OMPGeneralSchedulingType ParallelLoopGeneratorKMP::getSchedType(
    int ChunkSize, OMPGeneralSchedulingType Scheduling) const {
  if (ChunkSize == 0 && Scheduling == OMPGeneralSchedulingType::StaticChunked)
    return OMPGeneralSchedulingType::StaticNonChunked;

  return Scheduling;
}
````
- **EN**: This block declares or defines routines around `get`, `getInt32`, `setInitializer`, `is64BitArch` (+1 more); contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `get`, `getInt32`, `setInitializer`, `is64BitArch` (+1 more) 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 保留用于解释意图、用法或算法背景的注释.

## Key Concepts / 关键概念

- **Code generation**
  - **CN**: 代码生成
- **Loop transformation**
  - **CN**: 循环变换
- **OpenMP/runtime support**
  - **CN**: OpenMP/运行时支持
- **Debug and diagnostics**
  - **CN**: 调试与诊断

## Dependencies / 依赖关系

- **Polly headers**: `polly/CodeGen/LoopGeneratorsKMP.h` — Direct Polly APIs, data structures, or pass interfaces used here.
  **Polly headers（CN）**：`polly/CodeGen/LoopGeneratorsKMP.h` —— 此处直接使用的 Polly API、数据结构或 Pass 接口。
- **LLVM-family headers**: `llvm/Analysis/LoopInfo.h`, `llvm/IR/Dominators.h`, `llvm/IR/Module.h` — LLVM/Clang/MLIR infrastructure for IR, passes, utilities, or diagnostics.
  **LLVM-family headers（CN）**：`llvm/Analysis/LoopInfo.h`, `llvm/IR/Dominators.h`, `llvm/IR/Module.h` —— 用于 IR、Pass、工具或诊断的 LLVM/Clang/MLIR 基础设施。
