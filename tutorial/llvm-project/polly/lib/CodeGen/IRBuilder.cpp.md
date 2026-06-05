# IRBuilder.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `polly/lib/CodeGen/IRBuilder.cpp` | `polly/lib/CodeGen/IRBuilder.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | Implements Polly code generation, AST lowering, and runtime emission logic. | 实现 Polly 的代码生成、AST 降级与运行时生成逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-13

````cpp
//===------ PollyIRBuilder.cpp --------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// The Polly IRBuilder file contains Polly specific extensions for the IRBuilder
// that are used e.g. to emit the llvm.loop.parallel metadata.
//
//===----------------------------------------------------------------------===//

````
- **EN**: This block records the standard LLVM file banner and license metadata; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 给出 LLVM 标准文件头与许可证元数据; 保留用于解释意图、用法或算法背景的注释.

### Lines 14-29

````cpp
#include "polly/CodeGen/IRBuilder.h"
#include "polly/ScopInfo.h"
#include "polly/Support/ScopHelper.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/IR/Metadata.h"

using namespace llvm;
using namespace polly;

static const int MaxArraysInAliasScops = 10;

/// Get a self referencing id metadata node.
///
/// The MDNode looks like this (if arg0/arg1 are not null):
///
///    '!n = distinct !{!n, arg0, arg1}'
````
- **EN**: This block imports Polly, LLVM-family headers needed by the surrounding code; touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 引入周边逻辑所需的 Polly、LLVM-family 头文件; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 30-43

````cpp
///
/// @return The self referencing id metadata node.
static MDNode *getID(LLVMContext &Ctx, Metadata *arg0 = nullptr,
                     Metadata *arg1 = nullptr) {
  MDNode *ID;
  SmallVector<Metadata *, 3> Args;
  // Reserve operand 0 for loop id self reference.
  Args.push_back(nullptr);

  if (arg0)
    Args.push_back(arg0);
  if (arg1)
    Args.push_back(arg1);

````
- **EN**: This block declares or defines routines around `getID`, `push_back`; contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `getID`, `push_back` 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 保留用于解释意图、用法或算法背景的注释.

### Lines 44-58

````cpp
  ID = MDNode::getDistinct(Ctx, Args);
  ID->replaceOperandWith(0, ID);
  return ID;
}

ScopAnnotator::ScopAnnotator() : SE(nullptr), AliasScopeDomain(nullptr) {
  // Push an empty staging BandAttr.
  LoopAttrEnv.emplace_back();
}

ScopAnnotator::~ScopAnnotator() {
  assert(LoopAttrEnv.size() == 1 && "Loop stack imbalance");
  assert(!getStagingAttrEnv() && "Forgot to clear staging attr env");
}

````
- **EN**: This block declares or defines routines around `getDistinct`, `replaceOperandWith`, `ScopAnnotator`, `emplace_back` (+1 more); emits return paths that hand results or status codes back to callers; adds defensive checks for invariants or impossible states; touches Polly SCoP abstractions that model analyzable regions; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `getDistinct`, `replaceOperandWith`, `ScopAnnotator`, `emplace_back` (+1 more) 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 加入用于保护不变量或异常状态的断言/检查; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域；并延续周边实现细节。

### Lines 59-74

````cpp
void ScopAnnotator::buildAliasScopes(Scop &S) {
  SE = S.getSE();

  LLVMContext &Ctx = SE->getContext();
  AliasScopeDomain = getID(Ctx, MDString::get(Ctx, "polly.alias.scope.domain"));

  AliasScopeMap.clear();
  OtherAliasScopeListMap.clear();

  // We are only interested in arrays, but no scalar references. Scalars should
  // be handled easily by basicaa.
  SmallVector<ScopArrayInfo *, 10> Arrays;
  for (ScopArrayInfo *Array : S.arrays())
    if (Array->isArrayKind())
      Arrays.push_back(Array);

````
- **EN**: This block declares or defines routines around `buildAliasScopes`, `getSE`, `getContext`, `getID` (+2 more); contains control flow with 1 loop construct(s), 1 conditional check(s); touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `buildAliasScopes`, `getSE`, `getContext`, `getID` (+2 more) 相关的例程; 包含控制流结构：1 处循环、1 处条件判断; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 75-88

````cpp
  // The construction of alias scopes is quadratic in the number of arrays
  // involved. In case of too many arrays, skip the construction of alias
  // information to avoid quadratic increases in compile time and code size.
  if (Arrays.size() > MaxArraysInAliasScops)
    return;

  std::string AliasScopeStr = "polly.alias.scope.";
  for (const ScopArrayInfo *Array : Arrays) {
    assert(Array->getBasePtr() && "Base pointer must be present");
    AliasScopeMap[Array->getBasePtr()] =
        getID(Ctx, AliasScopeDomain,
              MDString::get(Ctx, (AliasScopeStr + Array->getName()).c_str()));
  }

````
- **EN**: This block declares or defines routines around `getBasePtr`, `getID`, `get`; contains control flow with 1 loop construct(s), 1 conditional check(s); adds defensive checks for invariants or impossible states; touches Polly SCoP abstractions that model analyzable regions; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `getBasePtr`, `getID`, `get` 相关的例程; 包含控制流结构：1 处循环、1 处条件判断; 加入用于保护不变量或异常状态的断言/检查; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域；并延续周边实现细节。

### Lines 89-103

````cpp
  for (const ScopArrayInfo *Array : Arrays) {
    MDNode *AliasScopeList = MDNode::get(Ctx, {});
    for (const auto &AliasScopePair : AliasScopeMap) {
      if (Array->getBasePtr() == AliasScopePair.first)
        continue;

      Metadata *Args = {AliasScopePair.second};
      AliasScopeList =
          MDNode::concatenate(AliasScopeList, MDNode::get(Ctx, Args));
    }

    OtherAliasScopeListMap[Array->getBasePtr()] = AliasScopeList;
  }
}

````
- **EN**: This block declares or defines routines around `get`, `concatenate`, `getBasePtr`; contains control flow with 2 loop construct(s), 1 conditional check(s); touches Polly SCoP abstractions that model analyzable regions.
- **CN**: 该代码块 声明或定义与 `get`, `concatenate`, `getBasePtr` 相关的例程; 包含控制流结构：2 处循环、1 处条件判断; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域.

### Lines 104-116

````cpp
void ScopAnnotator::pushLoop(Loop *L, bool IsParallel) {
  ActiveLoops.push_back(L);

  if (IsParallel) {
    LLVMContext &Ctx = SE->getContext();
    MDNode *AccessGroup = MDNode::getDistinct(Ctx, {});
    ParallelLoops.push_back(AccessGroup);
  }

  // Open an empty BandAttr context for loops nested in this one.
  LoopAttrEnv.emplace_back();
}

````
- **EN**: This block declares or defines routines around `pushLoop`, `push_back`, `getContext`, `getDistinct` (+1 more); contains control flow with 1 conditional check(s); touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `pushLoop`, `push_back`, `getContext`, `getDistinct` (+1 more) 相关的例程; 包含控制流结构：1 处条件判断; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 117-130

````cpp
void ScopAnnotator::popLoop(bool IsParallel) {
  ActiveLoops.pop_back();

  if (IsParallel) {
    assert(!ParallelLoops.empty() && "Expected a parallel loop to pop");
    ParallelLoops.pop_back();
  }

  // Exit the subloop context.
  assert(!getStagingAttrEnv() && "Forgot to clear staging attr env");
  assert(LoopAttrEnv.size() >= 2 && "Popped too many");
  LoopAttrEnv.pop_back();
}

````
- **EN**: This block declares or defines routines around `popLoop`, `pop_back`; contains control flow with 1 conditional check(s); adds defensive checks for invariants or impossible states; touches Polly SCoP abstractions that model analyzable regions; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `popLoop`, `pop_back` 相关的例程; 包含控制流结构：1 处条件判断; 加入用于保护不变量或异常状态的断言/检查; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域；并延续周边实现细节。

### Lines 131-147

````cpp
static void addVectorizeMetadata(LLVMContext &Ctx,
                                 SmallVector<Metadata *, 3> *Args,
                                 bool EnableLoopVectorizer) {
  MDString *PropName = MDString::get(Ctx, "llvm.loop.vectorize.enable");
  ConstantInt *Value =
      ConstantInt::get(Type::getInt1Ty(Ctx), EnableLoopVectorizer);
  ValueAsMetadata *PropValue = ValueAsMetadata::get(Value);
  Args->push_back(MDNode::get(Ctx, {PropName, PropValue}));
}

void addParallelMetadata(LLVMContext &Ctx, SmallVector<Metadata *, 3> *Args,
                         llvm::SmallVector<llvm::MDNode *, 8> ParallelLoops) {
  MDString *PropName = MDString::get(Ctx, "llvm.loop.parallel_accesses");
  MDNode *AccGroup = ParallelLoops.back();
  Args->push_back(MDNode::get(Ctx, {PropName, AccGroup}));
}

````
- **EN**: This block declares or defines routines around `addVectorizeMetadata`, `get`, `push_back`, `addParallelMetadata` (+1 more).
- **CN**: 该代码块 声明或定义与 `addVectorizeMetadata`, `get`, `push_back`, `addParallelMetadata` (+1 more) 相关的例程.

### Lines 148-163

````cpp
void ScopAnnotator::annotateLoopLatch(
    CondBrInst *B, bool IsParallel,
    std::optional<bool> EnableVectorizeMetadata) const {
  LLVMContext &Ctx = SE->getContext();
  SmallVector<Metadata *, 3> Args;

  // For the LoopID self-reference.
  Args.push_back(nullptr);

  // Add the user-defined loop properties to the annotation, if any. Any
  // additional properties are appended.
  // FIXME: What to do if these conflict?
  MDNode *MData = nullptr;
  if (BandAttr *AttrEnv = getActiveAttrEnv()) {
    MData = AttrEnv->Metadata;
    if (MData)
````
- **EN**: This block declares or defines routines around `annotateLoopLatch`, `getContext`, `push_back`; contains control flow with 2 conditional check(s); touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `annotateLoopLatch`, `getContext`, `push_back` 相关的例程; 包含控制流结构：2 处条件判断; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 164-179

````cpp
      llvm::append_range(Args, drop_begin(MData->operands(), 1));
  }
  if (IsParallel)
    addParallelMetadata(Ctx, &Args, ParallelLoops);
  if (EnableVectorizeMetadata.has_value())
    addVectorizeMetadata(Ctx, &Args, *EnableVectorizeMetadata);

  // No metadata to annotate.
  if (!MData && Args.size() <= 1)
    return;

  // Reuse the MData node if possible, this will avoid having to create another
  // one that cannot be merged because LoopIDs are 'distinct'. However, we have
  // to create a new one if we add properties.
  if (!MData || Args.size() > MData->getNumOperands()) {
    MData = MDNode::getDistinct(Ctx, Args);
````
- **EN**: This block declares or defines routines around `append_range`, `addParallelMetadata`, `addVectorizeMetadata`, `getDistinct`; contains control flow with 4 conditional check(s); preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `append_range`, `addParallelMetadata`, `addVectorizeMetadata`, `getDistinct` 相关的例程; 包含控制流结构：4 处条件判断; 保留用于解释意图、用法或算法背景的注释.

### Lines 180-194

````cpp
    MData->replaceOperandWith(0, MData);
  }
  B->setMetadata(LLVMContext::MD_loop, MData);
}

/// Get the pointer operand
///
/// @param Inst The instruction to be analyzed.
/// @return the pointer operand in case @p Inst is a memory access
///         instruction and nullptr otherwise.
static llvm::Value *getMemAccInstPointerOperand(Instruction *Inst) {
  auto MemInst = MemAccInst::dyn_cast(Inst);
  if (!MemInst)
    return nullptr;

````
- **EN**: This block declares or defines routines around `replaceOperandWith`, `setMetadata`, `getMemAccInstPointerOperand`, `dyn_cast`; contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `replaceOperandWith`, `setMetadata`, `getMemAccInstPointerOperand`, `dyn_cast` 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 保留用于解释意图、用法或算法背景的注释.

### Lines 195-213

````cpp
  return MemInst.getPointerOperand();
}

/// Find the base pointer of an array access.
///
/// This should be equivalent to ScalarEvolution::getPointerBase, which we
/// cannot use here the IR is still under construction which ScalarEvolution
/// assumes to not be modified.
static Value *findBasePtr(Value *Val) {
  while (true) {
    if (auto *Gep = dyn_cast<GEPOperator>(Val)) {
      Val = Gep->getPointerOperand();
      continue;
    }
    if (auto *Cast = dyn_cast<BitCastOperator>(Val)) {
      Val = Cast->getOperand(0);
      continue;
    }

````
- **EN**: This block declares or defines routines around `findBasePtr`, `getPointerOperand`, `getOperand`; contains control flow with 1 loop construct(s), 2 conditional check(s); emits return paths that hand results or status codes back to callers; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `findBasePtr`, `getPointerOperand`, `getOperand` 相关的例程; 包含控制流结构：1 处循环、2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 保留用于解释意图、用法或算法背景的注释.

### Lines 214-229

````cpp
    break;
  }

  return Val;
}

void ScopAnnotator::annotate(Instruction *Inst) {
  if (!Inst->mayReadOrWriteMemory())
    return;

  switch (ParallelLoops.size()) {
  case 0:
    // Not parallel to anything: no access group needed.
    break;
  case 1:
    // Single parallel loop: use directly.
````
- **EN**: This block declares or defines routines around `annotate`; contains control flow with 1 conditional check(s), 1 switch dispatch(es); emits return paths that hand results or status codes back to callers; touches Polly SCoP abstractions that model analyzable regions; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `annotate` 相关的例程; 包含控制流结构：1 处条件判断、1 处分支派发; 包含返回路径，用于向调用者交回结果或状态码; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域；并延续周边实现细节。

### Lines 230-242

````cpp
    Inst->setMetadata(LLVMContext::MD_access_group,
                      cast<MDNode>(ParallelLoops.front()));
    break;
  default:
    // Parallel to multiple loops: refer to list of access groups.
    Inst->setMetadata(LLVMContext::MD_access_group,
                      MDNode::get(SE->getContext(),
                                  ArrayRef<Metadata *>(
                                      (Metadata *const *)ParallelLoops.data(),
                                      ParallelLoops.size())));
    break;
  }

````
- **EN**: This block declares or defines routines around `setMetadata`, `cast<MDNode>`, `get`, `data` (+1 more); preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `setMetadata`, `cast<MDNode>`, `get`, `data` (+1 more) 相关的例程; 保留用于解释意图、用法或算法背景的注释.

### Lines 243-256

````cpp
  // TODO: Use the ScopArrayInfo once available here.
  if (!AliasScopeDomain)
    return;

  // Do not apply annotations on memory operations that take more than one
  // pointer. It would be ambiguous to which pointer the annotation applies.
  // FIXME: How can we specify annotations for all pointer arguments?
  if (isa<CallInst>(Inst) && !isa<MemSetInst>(Inst))
    return;

  auto *Ptr = getMemAccInstPointerOperand(Inst);
  if (!Ptr)
    return;

````
- **EN**: This block declares or defines routines around `getMemAccInstPointerOperand`; contains control flow with 3 conditional check(s); touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `getMemAccInstPointerOperand` 相关的例程; 包含控制流结构：3 处条件判断; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 257-272

````cpp
  Value *BasePtr = findBasePtr(Ptr);
  if (!BasePtr)
    return;

  auto AliasScope = AliasScopeMap.lookup(BasePtr);

  if (!AliasScope) {
    BasePtr = AlternativeAliasBases.lookup(BasePtr);
    if (!BasePtr)
      return;

    AliasScope = AliasScopeMap.lookup(BasePtr);
    if (!AliasScope)
      return;
  }

````
- **EN**: This block declares or defines routines around `findBasePtr`, `lookup`; contains control flow with 4 conditional check(s); touches Polly SCoP abstractions that model analyzable regions.
- **CN**: 该代码块 声明或定义与 `findBasePtr`, `lookup` 相关的例程; 包含控制流结构：4 处条件判断; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域.

### Lines 273-279

````cpp
  assert(OtherAliasScopeListMap.count(BasePtr) &&
         "BasePtr either expected in AliasScopeMap and OtherAlias...Map");
  auto *OtherAliasScopeList = OtherAliasScopeListMap[BasePtr];

  Inst->setMetadata("alias.scope", MDNode::get(SE->getContext(), AliasScope));
  Inst->setMetadata("noalias", OtherAliasScopeList);
}
````
- **EN**: This block declares or defines routines around `setMetadata`; adds defensive checks for invariants or impossible states; touches Polly SCoP abstractions that model analyzable regions.
- **CN**: 该代码块 声明或定义与 `setMetadata` 相关的例程; 加入用于保护不变量或异常状态的断言/检查; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域.

## Key Concepts / 关键概念

- **SCoP modeling**
  - **CN**: SCoP 建模
- **Loop transformation**
  - **CN**: 循环变换
- **Memory access tracking**
  - **CN**: 内存访问跟踪
- **Code generation**
  - **CN**: 代码生成
- **Debug and diagnostics**
  - **CN**: 调试与诊断

## Dependencies / 依赖关系

- **Polly headers**: `polly/CodeGen/IRBuilder.h`, `polly/ScopInfo.h`, `polly/Support/ScopHelper.h` — Direct Polly APIs, data structures, or pass interfaces used here.
  **Polly headers（CN）**：`polly/CodeGen/IRBuilder.h`, `polly/ScopInfo.h`, `polly/Support/ScopHelper.h` —— 此处直接使用的 Polly API、数据结构或 Pass 接口。
- **LLVM-family headers**: `llvm/ADT/SmallVector.h`, `llvm/IR/Metadata.h` — LLVM/Clang/MLIR infrastructure for IR, passes, utilities, or diagnostics.
  **LLVM-family headers（CN）**：`llvm/ADT/SmallVector.h`, `llvm/IR/Metadata.h` —— 用于 IR、Pass、工具或诊断的 LLVM/Clang/MLIR 基础设施。
