# FunctionComparator.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/FunctionComparator.cpp` | `llvm/lib/Transforms/Utils/FunctionComparator.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements function Comparator within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 FunctionComparator 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-92

```cpp
//===- FunctionComparator.h - Function Comparator -------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the FunctionComparator and GlobalNumberState classes
// which are used by the MergeFunctions pass for comparing functions.
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Utils/FunctionComparator.h"
#include "llvm/ADT/APFloat.h"
#include "llvm/ADT/APInt.h"
#include "llvm/ADT/ArrayRef.h"
#include "llvm/ADT/SmallPtrSet.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/IR/Attributes.h"
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  if (L.ugt(R))
    return 1;
  if (R.ugt(L))
    return -1;
  return 0;
}

int FunctionComparator::cmpConstantRanges(const ConstantRange &L,
                                          const ConstantRange &R) const {
  if (int Res = cmpAPInts(L.getLower(), R.getLower()))
    return Res;
  return cmpAPInts(L.getUpper(), R.getUpper());
}

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 93-187

```cpp
int FunctionComparator::cmpAPFloats(const APFloat &L, const APFloat &R) const {
  // Floats are ordered first by semantics (i.e. float, double, half, etc.),
  // then by value interpreted as a bitstring (aka APInt).
  const fltSemantics &SL = L.getSemantics(), &SR = R.getSemantics();
  if (int Res = cmpNumbers(APFloat::semanticsPrecision(SL),
                           APFloat::semanticsPrecision(SR)))
    return Res;
  if (int Res = cmpNumbers(APFloat::semanticsMaxExponent(SL),
                           APFloat::semanticsMaxExponent(SR)))
    return Res;
  if (int Res = cmpNumbers(APFloat::semanticsMinExponent(SL),
                           APFloat::semanticsMinExponent(SR)))
    return Res;
  if (int Res = cmpNumbers(APFloat::semanticsSizeInBits(SL),
                           APFloat::semanticsSizeInBits(SR)))
    return Res;
  return cmpAPInts(L.bitcastToAPInt(), R.bitcastToAPInt());
}

int FunctionComparator::cmpMem(StringRef L, StringRef R) const {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      }
      if (LA < RA)
        return -1;
      if (RA < LA)
        return 1;
    }
    if (LI != LE)
      return 1;
    if (RI != RE)
      return -1;
  }
  return 0;
}

```
- EN: Core entities appearing here include cmpAPFloats, cmpMem, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 cmpAPFloats, cmpMem，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 188-277

```cpp
int FunctionComparator::cmpMetadata(const Metadata *L,
                                    const Metadata *R) const {
  // TODO: the following routine coerce the metadata contents into constants
  // or MDStrings before comparison.
  // It ignores any other cases, so that the metadata nodes are considered
  // equal even though this is not correct.
  // We should structurally compare the metadata nodes to be perfect here.

  auto *MDStringL = dyn_cast<MDString>(L);
  auto *MDStringR = dyn_cast<MDString>(R);
  if (MDStringL && MDStringR) {
    if (MDStringL == MDStringR)
      return 0;
    return MDStringL->getString().compare(MDStringR->getString());
  }
  if (MDStringR)
    return -1;
  if (MDStringL)
    return 1;

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
                                                const CallBase &RCS) const {
  assert(LCS.getOpcode() == RCS.getOpcode() && "Can't compare otherwise!");

  if (int Res =
          cmpNumbers(LCS.getNumOperandBundles(), RCS.getNumOperandBundles()))
    return Res;

  for (unsigned I = 0, E = LCS.getNumOperandBundles(); I != E; ++I) {
    auto OBL = LCS.getOperandBundleAt(I);
    auto OBR = RCS.getOperandBundleAt(I);

    if (int Res = OBL.getTagName().compare(OBR.getTagName()))
      return Res;

```
- EN: Core entities appearing here include cmpMDNode, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 cmpMDNode，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 278-375

```cpp
    if (int Res = cmpNumbers(OBL.Inputs.size(), OBR.Inputs.size()))
      return Res;
  }

  return 0;
}

/// Constants comparison:
/// 1. Check whether type of L constant could be losslessly bitcasted to R
/// type.
/// 2. Compare constant contents.
/// For more details see declaration comments.
int FunctionComparator::cmpConstants(const Constant *L,
                                     const Constant *R) const {
  Type *TyL = L->getType();
  Type *TyR = R->getType();

  // Check whether types are bitcastable. This part is just re-factored
  // Type::canLosslesslyBitCastTo method, but instead of returning true/false,
  // we also pack into result which type is "less" for us.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

  if (int Res = cmpNumbers(L->getValueID(), R->getValueID()))
    return Res;

  if (const auto *SeqL = dyn_cast<ConstantDataSequential>(L)) {
    const auto *SeqR = cast<ConstantDataSequential>(R);
    // This handles ConstantDataArray and ConstantDataVector. Note that we
    // compare the two raw data arrays, which might differ depending on the host
    // endianness. This isn't a problem though, because the endiness of a module
    // will affect the order of the constants, but this order is the same
    // for a given input module and host platform.
    return cmpMem(SeqL->getRawDataValues(), SeqR->getRawDataValues());
  }

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 376-455

```cpp
  switch (L->getValueID()) {
  case Value::UndefValueVal:
  case Value::PoisonValueVal:
  case Value::ConstantTokenNoneVal:
    return TypesRes;
  case Value::ConstantIntVal: {
    const APInt &LInt = cast<ConstantInt>(L)->getValue();
    const APInt &RInt = cast<ConstantInt>(R)->getValue();
    return cmpAPInts(LInt, RInt);
  }
  case Value::ConstantFPVal: {
    const APFloat &LAPF = cast<ConstantFP>(L)->getValueAPF();
    const APFloat &RAPF = cast<ConstantFP>(R)->getValueAPF();
    return cmpAPFloats(LAPF, RAPF);
  }
  case Value::ConstantArrayVal: {
    const ConstantArray *LA = cast<ConstantArray>(L);
    const ConstantArray *RA = cast<ConstantArray>(R);
    uint64_t NumElementsL = cast<ArrayType>(TyL)->getNumElements();
    uint64_t NumElementsR = cast<ArrayType>(TyR)->getNumElements();
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    for (unsigned i = 0; i < NumOperandsL; ++i) {
      if (int Res = cmpConstants(cast<Constant>(LE->getOperand(i)),
                                 cast<Constant>(RE->getOperand(i))))
        return Res;
    }
    if (auto *GEPL = dyn_cast<GEPOperator>(LE)) {
      auto *GEPR = cast<GEPOperator>(RE);
      if (int Res = cmpTypes(GEPL->getSourceElementType(),
                             GEPR->getSourceElementType()))
        return Res;
      if (int Res = cmpNumbers(GEPL->getNoWrapFlags().getRaw(),
                               GEPR->getNoWrapFlags().getRaw()))
        return Res;

```
- EN: This region continues the FunctionComparator implementation with local helper logic centered on Value, UndefValueVal, PoisonValueVal, ConstantTokenNoneVal.
- CN: 这一段延续了 FunctionComparator 的主体实现，围绕 Value, UndefValueVal, PoisonValueVal, ConstantTokenNoneVal 等局部辅助逻辑展开。

### Lines 456-547

```cpp
      std::optional<ConstantRange> InRangeL = GEPL->getInRange();
      std::optional<ConstantRange> InRangeR = GEPR->getInRange();
      if (InRangeL) {
        if (!InRangeR)
          return 1;
        if (int Res = cmpConstantRanges(*InRangeL, *InRangeR))
          return Res;
      } else if (InRangeR) {
        return -1;
      }
    }
    if (auto *OBOL = dyn_cast<OverflowingBinaryOperator>(LE)) {
      auto *OBOR = cast<OverflowingBinaryOperator>(RE);
      if (int Res =
              cmpNumbers(OBOL->hasNoUnsignedWrap(), OBOR->hasNoUnsignedWrap()))
        return Res;
      if (int Res =
              cmpNumbers(OBOL->hasNoSignedWrap(), OBOR->hasNoSignedWrap()))
        return Res;
    }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  }
  default: // Unknown constant, abort.
    LLVM_DEBUG(dbgs() << "Looking at valueID " << L->getValueID() << "\n");
    llvm_unreachable("Constant ValueID not recognized.");
    return -1;
  }
}

int FunctionComparator::cmpGlobalValues(GlobalValue *L, GlobalValue *R) const {
  uint64_t LNumber = GlobalNumbers->getNumber(L);
  uint64_t RNumber = GlobalNumbers->getNumber(R);
  return cmpNumbers(LNumber, RNumber);
}

```
- EN: Core entities appearing here include cmpGlobalValues, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 cmpGlobalValues，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 548-646

```cpp
/// cmpType - compares two types,
/// defines total ordering among the types set.
/// See method declaration comments for more details.
int FunctionComparator::cmpTypes(Type *TyL, Type *TyR) const {
  PointerType *PTyL = dyn_cast<PointerType>(TyL);
  PointerType *PTyR = dyn_cast<PointerType>(TyR);

  const DataLayout &DL = FnL->getDataLayout();
  if (PTyL && PTyL->getAddressSpace() == 0)
    TyL = DL.getIntPtrType(TyL);
  if (PTyR && PTyR->getAddressSpace() == 0)
    TyR = DL.getIntPtrType(TyR);

  if (TyL == TyR)
    return 0;

  if (int Res = cmpNumbers(TyL->getTypeID(), TyR->getTypeID()))
    return Res;

  switch (TyL->getTypeID()) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    auto *STyL = cast<VectorType>(TyL);
    auto *STyR = cast<VectorType>(TyR);
    if (STyL->getElementCount().isScalable() !=
        STyR->getElementCount().isScalable())
      return cmpNumbers(STyL->getElementCount().isScalable(),
                        STyR->getElementCount().isScalable());
    if (STyL->getElementCount() != STyR->getElementCount())
      return cmpNumbers(STyL->getElementCount().getKnownMinValue(),
                        STyR->getElementCount().getKnownMinValue());
    return cmpTypes(STyL->getElementType(), STyR->getElementType());
  }
  }
}

```
- EN: Core entities appearing here include cmpTypes, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout.
- CN: 此处出现的核心实体包括 cmpTypes，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。

### Lines 647-735

```cpp
// Determine whether the two operations are the same except that pointer-to-A
// and pointer-to-B are equivalent. This should be kept in sync with
// Instruction::isSameOperationAs.
// Read method declaration comments for more details.
int FunctionComparator::cmpOperations(const Instruction *L,
                                      const Instruction *R,
                                      bool &needToCmpOperands) const {
  needToCmpOperands = true;
  if (int Res = cmpValues(L, R))
    return Res;

  // Differences from Instruction::isSameOperationAs:
  //  * replace type comparison with calls to cmpTypes.
  //  * we test for I->getRawSubclassOptionalData (nuw/nsw/tail) at the top.
  //  * because of the above, we don't test for the tail bit on calls later on.
  if (int Res = cmpNumbers(L->getOpcode(), R->getOpcode()))
    return Res;

  if (const GetElementPtrInst *GEPL = dyn_cast<GetElementPtrInst>(L)) {
    needToCmpOperands = false;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
                      cast<StoreInst>(R)->getSyncScopeID());
  }
  if (const CmpInst *CI = dyn_cast<CmpInst>(L))
    return cmpNumbers(CI->getPredicate(), cast<CmpInst>(R)->getPredicate());
  if (auto *CBL = dyn_cast<CallBase>(L)) {
    auto *CBR = cast<CallBase>(R);
    if (int Res = cmpNumbers(CBL->getCallingConv(), CBR->getCallingConv()))
      return Res;
    if (int Res = cmpAttrs(CBL->getAttributes(), CBR->getAttributes()))
      return Res;
    if (int Res = cmpOperandBundlesSchema(*CBL, *CBR))
      return Res;
    if (const CallInst *CI = dyn_cast<CallInst>(L))
      if (int Res = cmpNumbers(CI->getTailCallKind(),
```
- EN: This region continues the FunctionComparator implementation with local helper logic centered on Determine, This, Instruction, Read.
- CN: 这一段延续了 FunctionComparator 的主体实现，围绕 Determine, This, Instruction, Read 等局部辅助逻辑展开。

### Lines 736-829

```cpp
                               cast<CallInst>(R)->getTailCallKind()))
        return Res;
    return cmpMDNode(L->getMetadata(LLVMContext::MD_range),
                     R->getMetadata(LLVMContext::MD_range));
  }
  if (const SwitchInst *SI = dyn_cast<SwitchInst>(L)) {
    for (auto [LCase, RCase] : zip(SI->cases(), cast<SwitchInst>(R)->cases()))
      if (int Res = cmpConstants(LCase.getCaseValue(), RCase.getCaseValue()))
        return Res;
    return 0;
  }
  if (const InsertValueInst *IVI = dyn_cast<InsertValueInst>(L)) {
    ArrayRef<unsigned> LIndices = IVI->getIndices();
    ArrayRef<unsigned> RIndices = cast<InsertValueInst>(R)->getIndices();
    if (int Res = cmpNumbers(LIndices.size(), RIndices.size()))
      return Res;
    for (size_t i = 0, e = LIndices.size(); i != e; ++i) {
      if (int Res = cmpNumbers(LIndices[i], RIndices[i]))
        return Res;
    }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  if (const PHINode *PNL = dyn_cast<PHINode>(L)) {
    const PHINode *PNR = cast<PHINode>(R);
    // Ensure that in addition to the incoming values being identical
    // (checked by the caller of this function), the incoming blocks
    // are also identical.
    for (unsigned i = 0, e = PNL->getNumIncomingValues(); i != e; ++i) {
      if (int Res =
              cmpValues(PNL->getIncomingBlock(i), PNR->getIncomingBlock(i)))
        return Res;
    }
  }
  return 0;
}

```
- EN: This region continues the FunctionComparator implementation with local helper logic centered on CallInst, Res, LLVMContext, MD_range.
- CN: 这一段延续了 FunctionComparator 的主体实现，围绕 CallInst, Res, LLVMContext, MD_range 等局部辅助逻辑展开。

### Lines 830-920

```cpp
// Determine whether two GEP operations perform the same underlying arithmetic.
// Read method declaration comments for more details.
int FunctionComparator::cmpGEPs(const GEPOperator *GEPL,
                                const GEPOperator *GEPR) const {
  unsigned int ASL = GEPL->getPointerAddressSpace();
  unsigned int ASR = GEPR->getPointerAddressSpace();

  if (int Res = cmpNumbers(ASL, ASR))
    return Res;

  // When we have target data, we can reduce the GEP down to the value in bytes
  // added to the address.
  const DataLayout &DL = FnL->getDataLayout();
  unsigned OffsetBitWidth = DL.getIndexSizeInBits(ASL);
  APInt OffsetL(OffsetBitWidth, 0), OffsetR(OffsetBitWidth, 0);
  if (GEPL->accumulateConstantOffset(DL, OffsetL) &&
      GEPR->accumulateConstantOffset(DL, OffsetR))
    return cmpAPInts(OffsetL, OffsetR);
  if (int Res =
          cmpTypes(GEPL->getSourceElementType(), GEPR->getSourceElementType()))
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    return cmpConstants(ConstL, ConstR);
  }

  if (ConstL)
    return 1;
  if (ConstR)
    return -1;

  const MetadataAsValue *MetadataValueL = dyn_cast<MetadataAsValue>(L);
  const MetadataAsValue *MetadataValueR = dyn_cast<MetadataAsValue>(R);
  if (MetadataValueL && MetadataValueR) {
    if (MetadataValueL == MetadataValueR)
      return 0;

```
- EN: Core entities appearing here include cmpValues, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout.
- CN: 此处出现的核心实体包括 cmpValues，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。

### Lines 921-1010

```cpp
    return cmpMetadata(MetadataValueL->getMetadata(),
                       MetadataValueR->getMetadata());
  }

  if (MetadataValueL)
    return 1;
  if (MetadataValueR)
    return -1;

  const InlineAsm *InlineAsmL = dyn_cast<InlineAsm>(L);
  const InlineAsm *InlineAsmR = dyn_cast<InlineAsm>(R);

  if (InlineAsmL && InlineAsmR)
    return cmpInlineAsm(InlineAsmL, InlineAsmR);
  if (InlineAsmL)
    return 1;
  if (InlineAsmR)
    return -1;

  auto LeftSN = sn_mapL.insert(std::make_pair(L, sn_mapL.size())),
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      return Res;
  }

  if (int Res = cmpNumbers(FnL->isVarArg(), FnR->isVarArg()))
    return Res;

  // TODO: if it's internal and only used in direct calls, we could handle this
  // case too.
  if (int Res = cmpNumbers(FnL->getCallingConv(), FnR->getCallingConv()))
    return Res;

  if (int Res = cmpTypes(FnL->getFunctionType(), FnR->getFunctionType()))
    return Res;

```
- EN: Core entities appearing here include compareSignature, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 compareSignature，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 1011-1067

```cpp
  assert(FnL->arg_size() == FnR->arg_size() &&
         "Identically typed functions have different numbers of args!");

  // Visit the arguments so that they get enumerated in the order they're
  // passed in.
  for (Function::const_arg_iterator ArgLI = FnL->arg_begin(),
                                    ArgRI = FnR->arg_begin(),
                                    ArgLE = FnL->arg_end();
       ArgLI != ArgLE; ++ArgLI, ++ArgRI) {
    if (cmpValues(&*ArgLI, &*ArgRI) != 0)
      llvm_unreachable("Arguments repeat!");
  }
  return 0;
}

// Test whether the two functions have equivalent behaviour.
int FunctionComparator::compare() {
  beginCompare();

  if (int Res = compareSignature())
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    const Instruction *TermL = BBL->getTerminator();
    const Instruction *TermR = BBR->getTerminator();

    assert(TermL->getNumSuccessors() == TermR->getNumSuccessors());
    for (unsigned i = 0, e = TermL->getNumSuccessors(); i != e; ++i) {
      if (!VisitedBBs.insert(TermL->getSuccessor(i)).second)
        continue;

      FnLBBs.push_back(TermL->getSuccessor(i));
      FnRBBs.push_back(TermR->getSuccessor(i));
    }
  }
  return 0;
}
```
- EN: Core entities appearing here include compare, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 compare，它们承载了该区间涉及的局部状态、声明或变换入口。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `cmpNumbers, cmpAligns, cmpOrderings, cmpAPInts, cmpAPFloats, cmpMem, cmpMDNode, cmpGlobalValues` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`cmpNumbers, cmpAligns, cmpOrderings, cmpAPInts, cmpAPFloats, cmpMem, cmpMDNode, cmpGlobalValues` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `DataLayout` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DataLayout` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: IR/transform headers: `llvm/IR/Attributes.h`, `llvm/IR/BasicBlock.h`, `llvm/IR/Constant.h`, `llvm/IR/Constants.h`, `llvm/IR/DataLayout.h`, `llvm/IR/DerivedTypes.h`, `llvm/IR/Function.h`, `llvm/IR/GlobalValue.h`, `llvm/IR/InlineAsm.h`, `llvm/IR/InstrTypes.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Attributes.h`, `llvm/IR/BasicBlock.h`, `llvm/IR/Constant.h`, `llvm/IR/Constants.h`, `llvm/IR/DataLayout.h`, `llvm/IR/DerivedTypes.h`, `llvm/IR/Function.h`, `llvm/IR/GlobalValue.h`, `llvm/IR/InlineAsm.h`, `llvm/IR/InstrTypes.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/Casting.h`, `llvm/Support/Compiler.h`, `llvm/Support/Debug.h`, `llvm/Support/ErrorHandling.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/APFloat.h`, `llvm/ADT/APInt.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/SmallPtrSet.h`, `llvm/ADT/SmallVector.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Casting.h`, `llvm/Support/Compiler.h`, `llvm/Support/Debug.h`, `llvm/Support/ErrorHandling.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/APFloat.h`, `llvm/ADT/APInt.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/SmallPtrSet.h`, `llvm/ADT/SmallVector.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `cassert`, `cstddef`, `cstdint`, `utility` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`cassert`, `cstddef`, `cstdint`, `utility` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `DataLayout` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DataLayout` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
