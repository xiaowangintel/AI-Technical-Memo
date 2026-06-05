# LibCallsShrinkWrap.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/LibCallsShrinkWrap.cpp` | `llvm/lib/Transforms/Utils/LibCallsShrinkWrap.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements Lib Calls Shrink Wrap within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 LibCallsShrinkWrap 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-46

```cpp
//===-- LibCallsShrinkWrap.cpp ----------------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This pass shrink-wraps a call to function if the result is not used.
// The call can set errno but is otherwise side effect free. For example:
//    sqrt(val);
//  is transformed to
//    if (val < 0)
//      sqrt(val);
//  Even if the result of library call is not being used, the compiler cannot
//  safely delete the call because the function can set errno on error
//  conditions.
//  Note in many functions, the error condition solely depends on the incoming
//  parameter. In this optimization, we can generate the condition can lead to
//  the errno to shrink-wrap the call. Since the chances of hitting the error
//  condition is low, the runtime call is effectively eliminated.
//
//  These partially dead calls are usually results of C++ abstraction penalty
//  exposed by inlining.
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Utils/LibCallsShrinkWrap.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/Analysis/DomTreeUpdater.h"
#include "llvm/Analysis/GlobalsModRef.h"
#include "llvm/Analysis/TargetLibraryInfo.h"
#include "llvm/IR/Constants.h"
#include "llvm/IR/Dominators.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/IR/InstVisitor.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/MDBuilder.h"
#include "llvm/Transforms/Utils/BasicBlockUtils.h"

#include <cmath>

using namespace llvm;

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Legality, profitability, or canonical-form checks in this region depend on analyses such as DomTreeUpdater, TargetLibraryInfo.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段中的合法性、收益性或规范形态检查依赖 DomTreeUpdater, TargetLibraryInfo 等分析结果。

### Lines 47-91

```cpp
#define DEBUG_TYPE "libcalls-shrinkwrap"

STATISTIC(NumWrappedOneCond, "Number of One-Condition Wrappers Inserted");
STATISTIC(NumWrappedTwoCond, "Number of Two-Condition Wrappers Inserted");

namespace {
class LibCallsShrinkWrap : public InstVisitor<LibCallsShrinkWrap> {
public:
  LibCallsShrinkWrap(const TargetLibraryInfo &TLI, DomTreeUpdater &DTU)
      : TLI(TLI), DTU(DTU){};
  void visitCallInst(CallInst &CI) { checkCandidate(CI); }
  bool perform() {
    bool Changed = false;
    for (auto &CI : WorkList) {
      LLVM_DEBUG(dbgs() << "CDCE calls: " << CI->getCalledFunction()->getName()
                        << "\n");
      if (perform(CI)) {
        Changed = true;
        LLVM_DEBUG(dbgs() << "Transformed\n");
      }
    }
    return Changed;
  }

private:
  bool perform(CallInst *CI);
  void checkCandidate(CallInst &CI);
  void shrinkWrapCI(CallInst *CI, Value *Cond);
  bool performCallDomainErrorOnly(CallInst *CI, const LibFunc &Func);
  bool performCallErrors(CallInst *CI, const LibFunc &Func);
  bool performCallRangeErrorOnly(CallInst *CI, const LibFunc &Func);
  Value *generateOneRangeCond(CallInst *CI, const LibFunc &Func);
  Value *generateTwoRangeCond(CallInst *CI, const LibFunc &Func);
  Value *generateCondForPow(CallInst *CI, const LibFunc &Func);

  // Create an OR of two conditions with given Arg and Arg2.
  Value *createOrCond(CallInst *CI, Value *Arg, CmpInst::Predicate Cmp,
                      float Val, Value *Arg2, CmpInst::Predicate Cmp2,
                      float Val2) {
    IRBuilder<> BBBuilder(CI);
    auto Cond2 = createCond(BBBuilder, Arg2, Cmp2, Val2);
    auto Cond1 = createCond(BBBuilder, Arg, Cmp, Val);
    return BBBuilder.CreateOr(Cond1, Cond2);
  }

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics. Core entities appearing here include LibCallsShrinkWrap, perform, which carry the local state, declarations, or transformation entry points for this region.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。 此处出现的核心实体包括 LibCallsShrinkWrap, perform，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 92-133

```cpp
  // Create an OR of two conditions.
  Value *createOrCond(CallInst *CI, CmpInst::Predicate Cmp, float Val,
                      CmpInst::Predicate Cmp2, float Val2) {
    Value *Arg = CI->getArgOperand(0);
    return createOrCond(CI, Arg, Cmp, Val, Arg, Cmp2, Val2);
  }

  // Create a single condition using IRBuilder.
  Value *createCond(IRBuilder<> &BBBuilder, Value *Arg, CmpInst::Predicate Cmp,
                    float Val) {
    Constant *V = ConstantFP::get(BBBuilder.getContext(), APFloat(Val));
    if (!Arg->getType()->isFloatTy())
      V = ConstantFoldCastInstruction(Instruction::FPExt, V, Arg->getType());
    if (BBBuilder.GetInsertBlock()->getParent()->hasFnAttribute(Attribute::StrictFP))
      BBBuilder.setIsFPConstrained(true);
    return BBBuilder.CreateFCmp(Cmp, Arg, V);
  }

  // Create a single condition with given Arg.
  Value *createCond(CallInst *CI, Value *Arg, CmpInst::Predicate Cmp,
                    float Val) {
    IRBuilder<> BBBuilder(CI);
    return createCond(BBBuilder, Arg, Cmp, Val);
  }

  // Create a single condition.
  Value *createCond(CallInst *CI, CmpInst::Predicate Cmp, float Val) {
    Value *Arg = CI->getArgOperand(0);
    return createCond(CI, Arg, Cmp, Val);
  }

  const TargetLibraryInfo &TLI;
  DomTreeUpdater &DTU;
  SmallVector<CallInst *, 16> WorkList;
};
} // end anonymous namespace

// Perform the transformation to calls with errno set by domain error.
bool LibCallsShrinkWrap::performCallDomainErrorOnly(CallInst *CI,
                                                    const LibFunc &Func) {
  Value *Cond = nullptr;

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DomTreeUpdater, TargetLibraryInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DomTreeUpdater, TargetLibraryInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 134-180

```cpp
  switch (Func) {
  case LibFunc_acos:  // DomainError: (x < -1 || x > 1)
  case LibFunc_acosf: // Same as acos
  case LibFunc_acosl: // Same as acos
  case LibFunc_asin:  // DomainError: (x < -1 || x > 1)
  case LibFunc_asinf: // Same as asin
  case LibFunc_asinl: // Same as asin
  {
    ++NumWrappedTwoCond;
    Cond = createOrCond(CI, CmpInst::FCMP_OLT, -1.0f, CmpInst::FCMP_OGT, 1.0f);
    break;
  }
  case LibFunc_cos:  // DomainError: (x == +inf || x == -inf)
  case LibFunc_cosf: // Same as cos
  case LibFunc_cosl: // Same as cos
  case LibFunc_sin:  // DomainError: (x == +inf || x == -inf)
  case LibFunc_sinf: // Same as sin
  case LibFunc_sinl: // Same as sin
  {
    ++NumWrappedTwoCond;
    Cond = createOrCond(CI, CmpInst::FCMP_OEQ, INFINITY, CmpInst::FCMP_OEQ,
                        -INFINITY);
    break;
  }
  case LibFunc_acosh:  // DomainError: (x < 1)
  case LibFunc_acoshf: // Same as acosh
  case LibFunc_acoshl: // Same as acosh
  {
    ++NumWrappedOneCond;
    Cond = createCond(CI, CmpInst::FCMP_OLT, 1.0f);
    break;
  }
  case LibFunc_sqrt:  // DomainError: (x < 0)
  case LibFunc_sqrtf: // Same as sqrt
  case LibFunc_sqrtl: // Same as sqrt
  {
    ++NumWrappedOneCond;
    Cond = createCond(CI, CmpInst::FCMP_OLT, 0.0f);
    break;
  }
  default:
    return false;
  }
  shrinkWrapCI(CI, Cond);
  return true;
}

```
- EN: This region continues the LibCallsShrinkWrap implementation with local helper logic centered on Func, LibFunc_acos, DomainError, LibFunc_acosf.
- CN: 这一段延续了 LibCallsShrinkWrap 的主体实现，围绕 Func, LibFunc_acos, DomainError, LibFunc_acosf 等局部辅助逻辑展开。

### Lines 181-223

```cpp
// Perform the transformation to calls with errno set by range error.
bool LibCallsShrinkWrap::performCallRangeErrorOnly(CallInst *CI,
                                                   const LibFunc &Func) {
  Value *Cond = nullptr;

  switch (Func) {
  case LibFunc_cosh:
  case LibFunc_coshf:
  case LibFunc_coshl:
  case LibFunc_exp:
  case LibFunc_expf:
  case LibFunc_expl:
  case LibFunc_exp10:
  case LibFunc_exp10f:
  case LibFunc_exp10l:
  case LibFunc_exp2:
  case LibFunc_exp2f:
  case LibFunc_exp2l:
  case LibFunc_sinh:
  case LibFunc_sinhf:
  case LibFunc_sinhl: {
    Cond = generateTwoRangeCond(CI, Func);
    break;
  }
  case LibFunc_expm1:  // RangeError: (709, inf)
  case LibFunc_expm1f: // RangeError: (88, inf)
  case LibFunc_expm1l: // RangeError: (11356, inf)
  {
    Cond = generateOneRangeCond(CI, Func);
    break;
  }
  default:
    return false;
  }
  shrinkWrapCI(CI, Cond);
  return true;
}

// Perform the transformation to calls with errno set by combination of errors.
bool LibCallsShrinkWrap::performCallErrors(CallInst *CI,
                                           const LibFunc &Func) {
  Value *Cond = nullptr;

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 224-267

```cpp
  switch (Func) {
  case LibFunc_atanh:  // DomainError: (x < -1 || x > 1)
                        // PoleError:   (x == -1 || x == 1)
                        // Overall Cond: (x <= -1 || x >= 1)
  case LibFunc_atanhf: // Same as atanh
  case LibFunc_atanhl: // Same as atanh
  {
    ++NumWrappedTwoCond;
    Cond = createOrCond(CI, CmpInst::FCMP_OLE, -1.0f, CmpInst::FCMP_OGE, 1.0f);
    break;
  }
  case LibFunc_log:    // DomainError: (x < 0)
                        // PoleError:   (x == 0)
                        // Overall Cond: (x <= 0)
  case LibFunc_logf:   // Same as log
  case LibFunc_logl:   // Same as log
  case LibFunc_log10:  // Same as log
  case LibFunc_log10f: // Same as log
  case LibFunc_log10l: // Same as log
  case LibFunc_log2:   // Same as log
  case LibFunc_log2f:  // Same as log
  case LibFunc_log2l:  // Same as log
  case LibFunc_logb:   // Same as log
  case LibFunc_logbf:  // Same as log
  case LibFunc_logbl:  // Same as log
  {
    ++NumWrappedOneCond;
    Cond = createCond(CI, CmpInst::FCMP_OLE, 0.0f);
    break;
  }
  case LibFunc_log1p:  // DomainError: (x < -1)
                        // PoleError:   (x == -1)
                        // Overall Cond: (x <= -1)
  case LibFunc_log1pf: // Same as log1p
  case LibFunc_log1pl: // Same as log1p
  {
    ++NumWrappedOneCond;
    Cond = createCond(CI, CmpInst::FCMP_OLE, -1.0f);
    break;
  }
  case LibFunc_pow: // DomainError: x < 0 and y is noninteger
                     // PoleError:   x == 0 and y < 0
                     // RangeError:  overflow or underflow
  case LibFunc_powf:
```
- EN: This region continues the LibCallsShrinkWrap implementation with local helper logic centered on Func, LibFunc_atanh, DomainError, PoleError.
- CN: 这一段延续了 LibCallsShrinkWrap 的主体实现，围绕 Func, LibFunc_atanh, DomainError, PoleError 等局部辅助逻辑展开。

### Lines 268-311

```cpp
  case LibFunc_powl: {
    Cond = generateCondForPow(CI, Func);
    if (Cond == nullptr)
      return false;
    break;
  }
  default:
    return false;
  }
  assert(Cond && "performCallErrors should not see an empty condition");
  shrinkWrapCI(CI, Cond);
  return true;
}

// Checks if CI is a candidate for shrinkwrapping and put it into work list if
// true.
void LibCallsShrinkWrap::checkCandidate(CallInst &CI) {
  if (CI.isNoBuiltin())
    return;
  // A possible improvement is to handle the calls with the return value being
  // used. If there is API for fast libcall implementation without setting
  // errno, we can use the same framework to direct/wrap the call to the fast
  // API in the error free path, and leave the original call in the slow path.
  if (!CI.use_empty())
    return;

  LibFunc Func;
  Function *Callee = CI.getCalledFunction();
  if (!Callee)
    return;
  if (!TLI.getLibFunc(*Callee, Func) || !TLI.has(Func))
    return;

  if (CI.arg_empty())
    return;
  // TODO: Handle long double in other formats.
  Type *ArgType = CI.getArgOperand(0)->getType();
  if (!(ArgType->isFloatTy() || ArgType->isDoubleTy() ||
        ArgType->isX86_FP80Ty()))
    return;

  WorkList.push_back(&CI);
}

```
- EN: Core entities appearing here include checkCandidate, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 checkCandidate，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 312-355

```cpp
// Generate the upper bound condition for RangeError.
Value *LibCallsShrinkWrap::generateOneRangeCond(CallInst *CI,
                                                const LibFunc &Func) {
  float UpperBound;
  switch (Func) {
  case LibFunc_expm1: // RangeError: (709, inf)
    UpperBound = 709.0f;
    break;
  case LibFunc_expm1f: // RangeError: (88, inf)
    UpperBound = 88.0f;
    break;
  case LibFunc_expm1l: // RangeError: (11356, inf)
    UpperBound = 11356.0f;
    break;
  default:
    llvm_unreachable("Unhandled library call!");
  }

  ++NumWrappedOneCond;
  return createCond(CI, CmpInst::FCMP_OGT, UpperBound);
}

// Generate the lower and upper bound condition for RangeError.
Value *LibCallsShrinkWrap::generateTwoRangeCond(CallInst *CI,
                                                const LibFunc &Func) {
  float UpperBound, LowerBound;
  switch (Func) {
  case LibFunc_cosh: // RangeError: (x < -710 || x > 710)
  case LibFunc_sinh: // Same as cosh
    LowerBound = -710.0f;
    UpperBound = 710.0f;
    break;
  case LibFunc_coshf: // RangeError: (x < -89 || x > 89)
  case LibFunc_sinhf: // Same as coshf
    LowerBound = -89.0f;
    UpperBound = 89.0f;
    break;
  case LibFunc_coshl: // RangeError: (x < -11357 || x > 11357)
  case LibFunc_sinhl: // Same as coshl
    LowerBound = -11357.0f;
    UpperBound = 11357.0f;
    break;
  case LibFunc_exp: // RangeError: (x < -745 || x > 709)
    LowerBound = -745.0f;
```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 356-398

```cpp
    UpperBound = 709.0f;
    break;
  case LibFunc_expf: // RangeError: (x < -103 || x > 88)
    LowerBound = -103.0f;
    UpperBound = 88.0f;
    break;
  case LibFunc_expl: // RangeError: (x < -11399 || x > 11356)
    LowerBound = -11399.0f;
    UpperBound = 11356.0f;
    break;
  case LibFunc_exp10: // RangeError: (x < -323 || x > 308)
    LowerBound = -323.0f;
    UpperBound = 308.0f;
    break;
  case LibFunc_exp10f: // RangeError: (x < -45 || x > 38)
    LowerBound = -45.0f;
    UpperBound = 38.0f;
    break;
  case LibFunc_exp10l: // RangeError: (x < -4950 || x > 4932)
    LowerBound = -4950.0f;
    UpperBound = 4932.0f;
    break;
  case LibFunc_exp2: // RangeError: (x < -1074 || x > 1023)
    LowerBound = -1074.0f;
    UpperBound = 1023.0f;
    break;
  case LibFunc_exp2f: // RangeError: (x < -149 || x > 127)
    LowerBound = -149.0f;
    UpperBound = 127.0f;
    break;
  case LibFunc_exp2l: // RangeError: (x < -16445 || x > 11383)
    LowerBound = -16445.0f;
    UpperBound = 11383.0f;
    break;
  default:
    llvm_unreachable("Unhandled library call!");
  }

  ++NumWrappedTwoCond;
  return createOrCond(CI, CmpInst::FCMP_OGT, UpperBound, CmpInst::FCMP_OLT,
                      LowerBound);
}

```
- EN: This region continues the LibCallsShrinkWrap implementation with local helper logic centered on UpperBound, LibFunc_expf, RangeError, LowerBound.
- CN: 这一段延续了 LibCallsShrinkWrap 的主体实现，围绕 UpperBound, LibFunc_expf, RangeError, LowerBound 等局部辅助逻辑展开。

### Lines 399-442

```cpp
// For pow(x,y), We only handle the following cases:
// (1) x is a constant && (x >= 1) && (x < MaxUInt8)
//     Cond is: (y > 127)
// (2) x is a value coming from an integer type.
//   (2.1) if x's bit_size == 8
//         Cond: (x <= 0 || y > 128)
//   (2.2) if x's bit_size is 16
//         Cond: (x <= 0 || y > 64)
//   (2.3) if x's bit_size is 32
//         Cond: (x <= 0 || y > 32)
// Support for powl(x,y) and powf(x,y) are TBD.
//
// Note that condition can be more conservative than the actual condition
// (i.e. we might invoke the calls that will not set the errno.).
//
Value *LibCallsShrinkWrap::generateCondForPow(CallInst *CI,
                                              const LibFunc &Func) {
  // FIXME: LibFunc_powf and powl TBD.
  if (Func != LibFunc_pow) {
    LLVM_DEBUG(dbgs() << "Not handled powf() and powl()\n");
    return nullptr;
  }

  Value *Base = CI->getArgOperand(0);
  Value *Exp = CI->getArgOperand(1);

  // Constant Base case.
  if (ConstantFP *CF = dyn_cast<ConstantFP>(Base)) {
    double D = CF->getValueAPF().convertToDouble();
    if (D < 1.0f || D > APInt::getMaxValue(8).getZExtValue()) {
      LLVM_DEBUG(dbgs() << "Not handled pow(): constant base out of range\n");
      return nullptr;
    }

    ++NumWrappedOneCond;
    return createCond(CI, Exp, CmpInst::FCMP_OGT, 127.0f);
  }

  // If the Base value coming from an integer type.
  Instruction *I = dyn_cast<Instruction>(Base);
  if (!I) {
    LLVM_DEBUG(dbgs() << "Not handled pow(): FP type base\n");
    return nullptr;
  }
```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 443-493

```cpp
  unsigned Opcode = I->getOpcode();
  if (Opcode == Instruction::UIToFP || Opcode == Instruction::SIToFP) {
    unsigned BW = I->getOperand(0)->getType()->getPrimitiveSizeInBits();
    float UpperV = 0.0f;
    if (BW == 8)
      UpperV = 128.0f;
    else if (BW == 16)
      UpperV = 64.0f;
    else if (BW == 32)
      UpperV = 32.0f;
    else {
      LLVM_DEBUG(dbgs() << "Not handled pow(): type too wide\n");
      return nullptr;
    }

    ++NumWrappedTwoCond;
    return createOrCond(CI, Base, CmpInst::FCMP_OLE, 0.0f, Exp,
                        CmpInst::FCMP_OGT, UpperV);
  }
  LLVM_DEBUG(dbgs() << "Not handled pow(): base not from integer convert\n");
  return nullptr;
}

// Wrap conditions that can potentially generate errno to the library call.
void LibCallsShrinkWrap::shrinkWrapCI(CallInst *CI, Value *Cond) {
  assert(Cond != nullptr && "ShrinkWrapCI is not expecting an empty call inst");
  MDNode *BranchWeights =
      MDBuilder(CI->getContext()).createUnlikelyBranchWeights();

  Instruction *NewInst =
      SplitBlockAndInsertIfThen(Cond, CI, false, BranchWeights, &DTU);
  BasicBlock *CallBB = NewInst->getParent();
  CallBB->setName("cdce.call");
  BasicBlock *SuccBB = CallBB->getSingleSuccessor();
  assert(SuccBB && "The split block should have a single successor");
  SuccBB->setName("cdce.end");
  CI->removeFromParent();
  CI->insertInto(CallBB, CallBB->getFirstInsertionPt());
  LLVM_DEBUG(dbgs() << "== Basic Block After ==");
  LLVM_DEBUG(dbgs() << *CallBB->getSinglePredecessor() << *CallBB
                    << *CallBB->getSingleSuccessor() << "\n");
}

// Perform the transformation to a single candidate.
bool LibCallsShrinkWrap::perform(CallInst *CI) {
  LibFunc Func;
  Function *Callee = CI->getCalledFunction();
  assert(Callee && "perform() should apply to a non-empty callee");
  TLI.getLibFunc(*Callee, Func);
  assert(Func && "perform() is not expecting an empty function");

```
- EN: Core entities appearing here include shrinkWrapCI, perform, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 shrinkWrapCI, perform，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 494-523

```cpp
  if (performCallDomainErrorOnly(CI, Func) || performCallRangeErrorOnly(CI, Func))
    return true;
  return performCallErrors(CI, Func);
}

static bool runImpl(Function &F, const TargetLibraryInfo &TLI,
                    DominatorTree *DT) {
  if (F.hasOptSize())
    return false;
  DomTreeUpdater DTU(DT, DomTreeUpdater::UpdateStrategy::Lazy);
  LibCallsShrinkWrap CCDCE(TLI, DTU);
  CCDCE.visit(F);
  bool Changed = CCDCE.perform();

  // Verify the dominator after we've updated it locally.
  assert(!DT ||
         DTU.getDomTree().verify(DominatorTree::VerificationLevel::Fast));
  return Changed;
}

PreservedAnalyses LibCallsShrinkWrapPass::run(Function &F,
                                              FunctionAnalysisManager &FAM) {
  auto &TLI = FAM.getResult<TargetLibraryAnalysis>(F);
  auto *DT = FAM.getCachedResult<DominatorTreeAnalysis>(F);
  if (!runImpl(F, TLI, DT))
    return PreservedAnalyses::all();
  auto PA = PreservedAnalyses();
  PA.preserve<DominatorTreeAnalysis>();
  return PA;
}
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, DomTreeUpdater, TargetLibraryInfo.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, DomTreeUpdater, TargetLibraryInfo 等分析结果。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `LibCallsShrinkWrap, perform, checkCandidate, shrinkWrapCI` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`LibCallsShrinkWrap, perform, checkCandidate, shrinkWrapCI` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `DominatorTree, DomTreeUpdater, TargetLibraryInfo` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DominatorTree, DomTreeUpdater, TargetLibraryInfo` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/GlobalsModRef.h`, `llvm/Analysis/TargetLibraryInfo.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/GlobalsModRef.h`, `llvm/Analysis/TargetLibraryInfo.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/Constants.h`, `llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/InstVisitor.h`, `llvm/IR/Instructions.h`, `llvm/IR/MDBuilder.h`, `llvm/Transforms/Utils/LibCallsShrinkWrap.h`, `llvm/Transforms/Utils/BasicBlockUtils.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Constants.h`, `llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/InstVisitor.h`, `llvm/IR/Instructions.h`, `llvm/IR/MDBuilder.h`, `llvm/Transforms/Utils/LibCallsShrinkWrap.h`, `llvm/Transforms/Utils/BasicBlockUtils.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/ADT/SmallVector.h`, `llvm/ADT/Statistic.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/ADT/SmallVector.h`, `llvm/ADT/Statistic.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `cmath` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`cmath` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `DominatorTree`, `DomTreeUpdater`, `TargetLibraryInfo` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DominatorTree`, `DomTreeUpdater`, `TargetLibraryInfo` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
