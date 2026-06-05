# LoopGenerators.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `polly/lib/CodeGen/LoopGenerators.cpp` | `polly/lib/CodeGen/LoopGenerators.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | Implements Polly code generation, AST lowering, and runtime emission logic. The leading comment describes it as: IR helper to create loops. | 实现 Polly 的代码生成、AST 降级与运行时生成逻辑。 文件开头注释将其概括为：IR helper to create loops。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-13

````cpp
//===------ LoopGenerators.cpp -  IR helper to create loops ---------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains functions to create scalar loops and orchestrate the
// creation of parallel loops as LLVM-IR.
//
//===----------------------------------------------------------------------===//

````
- **EN**: This block records the standard LLVM file banner and license metadata; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 给出 LLVM 标准文件头与许可证元数据; 保留用于解释意图、用法或算法背景的注释.

### Lines 14-27

````cpp
#include "polly/CodeGen/LoopGenerators.h"
#include "polly/Options.h"
#include "polly/ScopDetection.h"
#include "llvm/Analysis/LoopInfo.h"
#include "llvm/IR/DataLayout.h"
#include "llvm/IR/DebugInfoMetadata.h"
#include "llvm/IR/Dominators.h"
#include "llvm/IR/Module.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Transforms/Utils/BasicBlockUtils.h"

using namespace llvm;
using namespace polly;

````
- **EN**: This block imports Polly, LLVM-family headers needed by the surrounding code; touches Polly SCoP abstractions that model analyzable regions.
- **CN**: 该代码块 引入周边逻辑所需的 Polly、LLVM-family 头文件; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域.

### Lines 28-42

````cpp
int polly::PollyNumThreads;
OMPGeneralSchedulingType polly::PollyScheduling;
int polly::PollyChunkSize;

static cl::opt<int, true>
    XPollyNumThreads("polly-num-threads",
                     cl::desc("Number of threads to use (0 = auto)"),
                     cl::Hidden, cl::location(polly::PollyNumThreads),
                     cl::init(0), cl::cat(PollyCategory));

cl::opt<bool> PollyVectorizeMetadata(
    "polly-annotate-metadata-vectorize",
    cl::desc("Append vectorize enable/disable metadata from polly"),
    cl::init(false), cl::ZeroOrMore, cl::cat(PollyCategory));

````
- **EN**: This block registers command-line options such as `PollyVectorizeMetadata`; declares or defines routines around `XPollyNumThreads`, `desc`, `location`, `init` (+1 more).
- **CN**: 该代码块 注册命令行选项，例如 `PollyVectorizeMetadata`; 声明或定义与 `XPollyNumThreads`, `desc`, `location`, `init` (+1 more) 相关的例程.

### Lines 43-57

````cpp
static cl::opt<OMPGeneralSchedulingType, true> XPollyScheduling(
    "polly-scheduling",
    cl::desc("Scheduling type of parallel OpenMP for loops"),
    cl::values(clEnumValN(OMPGeneralSchedulingType::StaticChunked, "static",
                          "Static scheduling"),
               clEnumValN(OMPGeneralSchedulingType::Dynamic, "dynamic",
                          "Dynamic scheduling"),
               clEnumValN(OMPGeneralSchedulingType::Guided, "guided",
                          "Guided scheduling"),
               clEnumValN(OMPGeneralSchedulingType::Runtime, "runtime",
                          "Runtime determined (OMP_SCHEDULE)")),
    cl::Hidden, cl::location(polly::PollyScheduling),
    cl::init(OMPGeneralSchedulingType::Runtime), cl::Optional,
    cl::cat(PollyCategory));

````
- **EN**: This block registers command-line options such as `XPollyScheduling`; declares or defines routines around `XPollyScheduling`, `desc`, `values`, `clEnumValN` (+4 more); mentions schedule-related state that orders statements or iterations.
- **CN**: 该代码块 注册命令行选项，例如 `XPollyScheduling`; 声明或定义与 `XPollyScheduling`, `desc`, `values`, `clEnumValN` (+4 more) 相关的例程; 涉及调度相关状态，用于安排语句或迭代顺序.

### Lines 58-73

````cpp
static cl::opt<int, true>
    XPollyChunkSize("polly-scheduling-chunksize",
                    cl::desc("Chunksize to use by the OpenMP runtime calls"),
                    cl::Hidden, cl::location(polly::PollyChunkSize),
                    cl::init(0), cl::Optional, cl::cat(PollyCategory));

// We generate a loop of either of the following structures:
//
//              BeforeBB                      BeforeBB
//                 |                             |
//                 v                             v
//              GuardBB                      PreHeaderBB
//              /      |                         |   _____
//     __  PreHeaderBB  |                        v  \/    |
//    /  \    /         |                     HeaderBB  latch
// latch  HeaderBB      |                        |\       |
````
- **EN**: This block declares or defines routines around `XPollyChunkSize`, `desc`, `location`, `init`; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `XPollyChunkSize`, `desc`, `location`, `init` 相关的例程; 保留用于解释意图、用法或算法背景的注释.

### Lines 74-89

````cpp
//    \  /    \         /                        | \------/
//     <       \       /                         |
//              \     /                          v
//              ExitBB                         ExitBB
//
// depending on whether or not we know that it is executed at least once. If
// not, GuardBB checks if the loop is executed at least once. If this is the
// case we branch to PreHeaderBB and subsequently to the HeaderBB, which
// contains the loop iv 'polly.indvar', the incremented loop iv
// 'polly.indvar_next' as well as the condition to check if we execute another
// iteration of the loop. After the loop has finished, we branch to ExitBB.
// We expect the type of UB, LB, UB+Stride to be large enough for values that
// UB may take throughout the execution of the loop, including the computation
// of indvar + Stride before the final abort.
Value *polly::createLoop(Value *LB, Value *UB, Value *Stride,
                         PollyIRBuilder &Builder, LoopInfo &LI,
````
- **EN**: This block declares or defines routines around `createLoop`; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `createLoop` 相关的例程; 保留用于解释意图、用法或算法背景的注释.

### Lines 90-107

````cpp
                         DominatorTree &DT, BasicBlock *&ExitBB,
                         ICmpInst::Predicate Predicate,
                         ScopAnnotator *Annotator, bool Parallel, bool UseGuard,
                         bool LoopVectDisabled) {
  Function *F = Builder.GetInsertBlock()->getParent();
  LLVMContext &Context = F->getContext();

  assert(LB->getType() == UB->getType() && "Types of loop bounds do not match");
  IntegerType *LoopIVType = dyn_cast<IntegerType>(UB->getType());
  assert(LoopIVType && "UB is not integer?");

  BasicBlock *BeforeBB = Builder.GetInsertBlock();
  BasicBlock *GuardBB =
      UseGuard ? BasicBlock::Create(Context, "polly.loop_if", F) : nullptr;
  BasicBlock *HeaderBB = BasicBlock::Create(Context, "polly.loop_header", F);
  BasicBlock *PreHeaderBB =
      BasicBlock::Create(Context, "polly.loop_preheader", F);

````
- **EN**: This block declares or defines routines around `GetInsertBlock`, `getContext`, `dyn_cast<IntegerType>`, `Create`; adds defensive checks for invariants or impossible states; touches Polly SCoP abstractions that model analyzable regions.
- **CN**: 该代码块 声明或定义与 `GetInsertBlock`, `getContext`, `dyn_cast<IntegerType>`, `Create` 相关的例程; 加入用于保护不变量或异常状态的断言/检查; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域.

### Lines 108-122

````cpp
  // Update LoopInfo
  Loop *OuterLoop = LI.getLoopFor(BeforeBB);
  Loop *NewLoop = LI.AllocateLoop();

  if (OuterLoop)
    OuterLoop->addChildLoop(NewLoop);
  else
    LI.addTopLevelLoop(NewLoop);

  if (OuterLoop) {
    if (GuardBB)
      OuterLoop->addBasicBlockToLoop(GuardBB, LI);
    OuterLoop->addBasicBlockToLoop(PreHeaderBB, LI);
  }

````
- **EN**: This block declares or defines routines around `getLoopFor`, `AllocateLoop`, `addChildLoop`, `addTopLevelLoop` (+1 more); contains control flow with 3 conditional check(s); preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `getLoopFor`, `AllocateLoop`, `addChildLoop`, `addTopLevelLoop` (+1 more) 相关的例程; 包含控制流结构：3 处条件判断; 保留用于解释意图、用法或算法背景的注释.

### Lines 123-138

````cpp
  NewLoop->addBasicBlockToLoop(HeaderBB, LI);

  // Notify the annotator (if present) that we have a new loop, but only
  // after the header block is set.
  if (Annotator)
    Annotator->pushLoop(NewLoop, Parallel);

  // ExitBB
  ExitBB = SplitBlock(BeforeBB, Builder.GetInsertPoint(), &DT, &LI);
  ExitBB->setName("polly.loop_exit");

  // BeforeBB
  if (GuardBB) {
    BeforeBB->getTerminator()->setSuccessor(0, GuardBB);
    DT.addNewBlock(GuardBB, BeforeBB);

````
- **EN**: This block declares or defines routines around `addBasicBlockToLoop`, `pushLoop`, `SplitBlock`, `setName` (+2 more); contains control flow with 2 conditional check(s); preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `addBasicBlockToLoop`, `pushLoop`, `SplitBlock`, `setName` (+2 more) 相关的例程; 包含控制流结构：2 处条件判断; 保留用于解释意图、用法或算法背景的注释.

### Lines 139-150

````cpp
    // GuardBB
    Builder.SetInsertPoint(GuardBB);
    Value *LoopGuard;
    LoopGuard = Builder.CreateICmp(Predicate, LB, UB);
    LoopGuard->setName("polly.loop_guard");
    Builder.CreateCondBr(LoopGuard, PreHeaderBB, ExitBB);
    DT.addNewBlock(PreHeaderBB, GuardBB);
  } else {
    BeforeBB->getTerminator()->setSuccessor(0, PreHeaderBB);
    DT.addNewBlock(PreHeaderBB, BeforeBB);
  }

````
- **EN**: This block declares or defines routines around `SetInsertPoint`, `CreateICmp`, `setName`, `CreateCondBr` (+2 more); preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `SetInsertPoint`, `CreateICmp`, `setName`, `CreateCondBr` (+2 more) 相关的例程; 保留用于解释意图、用法或算法背景的注释.

### Lines 151-164

````cpp
  // PreHeaderBB
  Builder.SetInsertPoint(PreHeaderBB);
  Builder.CreateBr(HeaderBB);

  // HeaderBB
  DT.addNewBlock(HeaderBB, PreHeaderBB);
  Builder.SetInsertPoint(HeaderBB);
  PHINode *IV = Builder.CreatePHI(LoopIVType, 2, "polly.indvar");
  IV->addIncoming(LB, PreHeaderBB);
  Stride = Builder.CreateZExtOrBitCast(Stride, LoopIVType);
  Value *IncrementedIV = Builder.CreateNSWAdd(IV, Stride, "polly.indvar_next");
  Value *LoopCondition =
      Builder.CreateICmp(Predicate, IncrementedIV, UB, "polly.loop_cond");

````
- **EN**: This block declares or defines routines around `SetInsertPoint`, `CreateBr`, `addNewBlock`, `CreatePHI` (+4 more); preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `SetInsertPoint`, `CreateBr`, `addNewBlock`, `CreatePHI` (+4 more) 相关的例程; 保留用于解释意图、用法或算法背景的注释.

### Lines 165-180

````cpp
  // Create the loop latch and annotate it as such.
  CondBrInst *B = Builder.CreateCondBr(LoopCondition, HeaderBB, ExitBB);

  // Don't annotate vectorize metadata when both LoopVectDisabled and
  // PollyVectorizeMetadata are disabled. Annotate vectorize metadata to false
  // when LoopVectDisabled is true. Otherwise we annotate the vectorize metadata
  // to true.
  if (Annotator) {
    std::optional<bool> EnableVectorizeMetadata;
    if (LoopVectDisabled)
      EnableVectorizeMetadata = false;
    else if (PollyVectorizeMetadata)
      EnableVectorizeMetadata = true;
    Annotator->annotateLoopLatch(B, Parallel, EnableVectorizeMetadata);
  }

````
- **EN**: This block declares or defines routines around `CreateCondBr`, `annotateLoopLatch`; contains control flow with 3 conditional check(s); preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `CreateCondBr`, `annotateLoopLatch` 相关的例程; 包含控制流结构：3 处条件判断; 保留用于解释意图、用法或算法背景的注释.

### Lines 181-195

````cpp
  IV->addIncoming(IncrementedIV, HeaderBB);
  if (GuardBB)
    DT.changeImmediateDominator(ExitBB, GuardBB);
  else
    DT.changeImmediateDominator(ExitBB, HeaderBB);

  // The loop body should be added here.
  Builder.SetInsertPoint(HeaderBB->getFirstNonPHIIt());
  return IV;
}

Value *ParallelLoopGenerator::createParallelLoop(
    Value *LB, Value *UB, Value *Stride, SetVector<Value *> &UsedValues,
    ValueMapT &Map, BasicBlock::iterator *LoopBody) {

````
- **EN**: This block declares or defines routines around `addIncoming`, `changeImmediateDominator`, `SetInsertPoint`, `createParallelLoop`; contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `addIncoming`, `changeImmediateDominator`, `SetInsertPoint`, `createParallelLoop` 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 保留用于解释意图、用法或算法背景的注释.

### Lines 196-208

````cpp
  AllocaInst *Struct = storeValuesIntoStruct(UsedValues);
  BasicBlock::iterator BeforeLoop = Builder.GetInsertPoint();

  Value *IV;
  Function *SubFn;
  std::tie(IV, SubFn) = createSubFn(Stride, Struct, UsedValues, Map);
  *LoopBody = Builder.GetInsertPoint();
  Builder.SetInsertPoint(BeforeLoop);

  // Add one as the upper bound provided by OpenMP is a < comparison
  // whereas the codegenForSequential function creates a <= comparison.
  UB = Builder.CreateAdd(UB, ConstantInt::get(LongType, 1));

````
- **EN**: This block declares or defines routines around `storeValuesIntoStruct`, `GetInsertPoint`, `tie`, `SetInsertPoint` (+1 more); preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `storeValuesIntoStruct`, `GetInsertPoint`, `tie`, `SetInsertPoint` (+1 more) 相关的例程; 保留用于解释意图、用法或算法背景的注释.

### Lines 209-224

````cpp
  // Execute the prepared subfunction in parallel.
  deployParallelExecution(SubFn, Struct, LB, UB, Stride);

  return IV;
}

Function *ParallelLoopGenerator::createSubFnDefinition() {
  Function *F = Builder.GetInsertBlock()->getParent();
  Function *SubFn = prepareSubFnDefinition(F);

  // Certain backends (e.g., NVPTX) do not support '.'s in function names.
  // Hence, we ensure that all '.'s are replaced by '_'s.
  std::string FunctionName = SubFn->getName().str();
  std::replace(FunctionName.begin(), FunctionName.end(), '.', '_');
  SubFn->setName(FunctionName);

````
- **EN**: This block declares or defines routines around `deployParallelExecution`, `createSubFnDefinition`, `GetInsertBlock`, `prepareSubFnDefinition` (+3 more); emits return paths that hand results or status codes back to callers; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `deployParallelExecution`, `createSubFnDefinition`, `GetInsertBlock`, `prepareSubFnDefinition` (+3 more) 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 保留用于解释意图、用法或算法背景的注释.

### Lines 225-237

````cpp
  // Do not run any polly pass on the new function.
  SubFn->addFnAttr(PollySkipFnAttr);

  return SubFn;
}

AllocaInst *
ParallelLoopGenerator::storeValuesIntoStruct(SetVector<Value *> &Values) {
  SmallVector<Type *, 8> Members;

  for (Value *V : Values)
    Members.push_back(V->getType());

````
- **EN**: This block declares or defines routines around `addFnAttr`, `storeValuesIntoStruct`, `push_back`; contains control flow with 1 loop construct(s); emits return paths that hand results or status codes back to callers; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `addFnAttr`, `storeValuesIntoStruct`, `push_back` 相关的例程; 包含控制流结构：1 处循环; 包含返回路径，用于向调用者交回结果或状态码; 保留用于解释意图、用法或算法背景的注释.

### Lines 238-254

````cpp
  const DataLayout &DL = Builder.GetInsertBlock()->getModule()->getDataLayout();

  // We do not want to allocate the alloca inside any loop, thus we allocate it
  // in the entry block of the function and use annotations to denote the actual
  // live span (similar to clang).
  BasicBlock &EntryBB = Builder.GetInsertBlock()->getParent()->getEntryBlock();
  BasicBlock::iterator IP = EntryBB.getFirstInsertionPt();
  StructType *Ty = StructType::get(Builder.getContext(), Members);
  AllocaInst *Struct = new AllocaInst(Ty, DL.getAllocaAddrSpace(), nullptr,
                                      "polly.par.userContext", IP);

  for (unsigned i = 0; i < Values.size(); i++) {
    Value *Address = Builder.CreateStructGEP(Ty, Struct, i);
    Address->setName("polly.subfn.storeaddr." + Values[i]->getName());
    Builder.CreateStore(Values[i], Address);
  }

````
- **EN**: This block declares or defines routines around `GetInsertBlock`, `getFirstInsertionPt`, `get`, `AllocaInst` (+3 more); contains control flow with 1 loop construct(s); preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `GetInsertBlock`, `getFirstInsertionPt`, `get`, `AllocaInst` (+3 more) 相关的例程; 包含控制流结构：1 处循环; 保留用于解释意图、用法或算法背景的注释.

### Lines 255-268

````cpp
  return Struct;
}

void ParallelLoopGenerator::extractValuesFromStruct(
    SetVector<Value *> OldValues, Type *Ty, Value *Struct, ValueMapT &Map) {
  for (unsigned i = 0; i < OldValues.size(); i++) {
    Value *Address = Builder.CreateStructGEP(Ty, Struct, i);
    Type *ElemTy = cast<GetElementPtrInst>(Address)->getResultElementType();
    Value *NewValue = Builder.CreateLoad(ElemTy, Address);
    NewValue->setName("polly.subfunc.arg." + OldValues[i]->getName());
    Map[OldValues[i]] = NewValue;
  }
}

````
- **EN**: This block declares or defines routines around `extractValuesFromStruct`, `CreateStructGEP`, `cast<GetElementPtrInst>`, `CreateLoad` (+1 more); contains control flow with 1 loop construct(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `extractValuesFromStruct`, `CreateStructGEP`, `cast<GetElementPtrInst>`, `CreateLoad` (+1 more) 相关的例程; 包含控制流结构：1 处循环; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 269-279

````cpp
DebugLoc polly::createDebugLocForGeneratedCode(Function *F) {
  if (!F)
    return DebugLoc();

  LLVMContext &Ctx = F->getContext();
  DISubprogram *DILScope =
      dyn_cast_or_null<DISubprogram>(F->getMetadata(LLVMContext::MD_dbg));
  if (!DILScope)
    return DebugLoc();
  return DILocation::get(Ctx, 0, 0, DILScope);
}
````
- **EN**: This block declares or defines routines around `createDebugLocForGeneratedCode`, `getContext`, `dyn_cast_or_null<DISubprogram>`; contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; touches Polly SCoP abstractions that model analyzable regions.
- **CN**: 该代码块 声明或定义与 `createDebugLocForGeneratedCode`, `getContext`, `dyn_cast_or_null<DISubprogram>` 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域.

## Key Concepts / 关键概念

- **Loop transformation**
  - **CN**: 循环变换
- **Code generation**
  - **CN**: 代码生成
- **OpenMP/runtime support**
  - **CN**: OpenMP/运行时支持
- **Debug and diagnostics**
  - **CN**: 调试与诊断
- **SCoP modeling**
  - **CN**: SCoP 建模

## Dependencies / 依赖关系

- **Polly headers**: `polly/CodeGen/LoopGenerators.h`, `polly/Options.h`, `polly/ScopDetection.h` — Direct Polly APIs, data structures, or pass interfaces used here.
  **Polly headers（CN）**：`polly/CodeGen/LoopGenerators.h`, `polly/Options.h`, `polly/ScopDetection.h` —— 此处直接使用的 Polly API、数据结构或 Pass 接口。
- **LLVM-family headers**: `llvm/Analysis/LoopInfo.h`, `llvm/IR/DataLayout.h`, `llvm/IR/DebugInfoMetadata.h`, `llvm/IR/Dominators.h`, `llvm/IR/Module.h`, `llvm/Support/CommandLine.h`, `llvm/Transforms/Utils/BasicBlockUtils.h` — LLVM/Clang/MLIR infrastructure for IR, passes, utilities, or diagnostics.
  **LLVM-family headers（CN）**：`llvm/Analysis/LoopInfo.h`, `llvm/IR/DataLayout.h`, `llvm/IR/DebugInfoMetadata.h`, `llvm/IR/Dominators.h`, `llvm/IR/Module.h`, `llvm/Support/CommandLine.h`, `llvm/Transforms/Utils/BasicBlockUtils.h` —— 用于 IR、Pass、工具或诊断的 LLVM/Clang/MLIR 基础设施。
