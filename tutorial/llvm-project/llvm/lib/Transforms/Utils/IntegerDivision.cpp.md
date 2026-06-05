# IntegerDivision.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/IntegerDivision.cpp` | `llvm/lib/Transforms/Utils/IntegerDivision.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements expand integer division within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 IntegerDivision 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-66

```cpp
//===-- IntegerDivision.cpp - Expand integer division ---------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains an implementation of 32bit and 64bit scalar integer
// division for targets that don't have native support. It's largely derived
// from compiler-rt's implementations of __udivsi3 and __udivmoddi4,
// but hand-tuned for targets that prefer less control flow.
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Utils/IntegerDivision.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/IR/Instruction.h"
#include "llvm/IR/Instructions.h"
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // ;   %xored        = xor i32 %urem, %dividend_sgn
  // ;   %srem         = sub i32 %xored, %dividend_sgn
  Dividend            = Builder.CreateFreeze(Dividend);
  Divisor             = Builder.CreateFreeze(Divisor);
  Value *DividendSign = Builder.CreateAShr(Dividend, Shift);
  Value *DivisorSign  = Builder.CreateAShr(Divisor, Shift);
  Value *DvdXor       = Builder.CreateXor(Dividend, DividendSign);
  Value *DvsXor       = Builder.CreateXor(Divisor, DivisorSign);
  Value *UDividend    = Builder.CreateSub(DvdXor, DividendSign);
  Value *UDivisor     = Builder.CreateSub(DvsXor, DivisorSign);
  Value *URem         = Builder.CreateURem(UDividend, UDivisor);
  Value *Xored        = Builder.CreateXor(URem, DividendSign);
  Value *SRem         = Builder.CreateSub(Xored, DividendSign);

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 67-124

```cpp
  if (Instruction *URemInst = dyn_cast<Instruction>(URem))
    Builder.SetInsertPoint(URemInst);

  return SRem;
}


/// Generate code to compute the remainder of two unsigned integers. Returns the
/// remainder. Builder's insert point should be pointing where the caller wants
/// code generated, e.g. at the urem instruction. This will generate a udiv in
/// the process, and Builder's insert point will be pointing at the udiv (if
/// present, i.e. not folded), ready to be expanded if the user wishes
static Value *generateUnsignedRemainderCode(Value *Dividend, Value *Divisor,
                                            IRBuilder<> &Builder) {
  // Remainder = Dividend - Quotient*Divisor

  // Following instructions are generated for both i32 and i64

  // ;   %quotient  = udiv i32 %dividend, %divisor
  // ;   %product   = mul i32 %divisor, %quotient
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

  // Following instructions are generated for both i32 (shift 31) and
  // i64 (shift 63).

  // ;   %tmp    = ashr i32 %dividend, 31
  // ;   %tmp1   = ashr i32 %divisor, 31
  // ;   %tmp2   = xor i32 %tmp, %dividend
  // ;   %u_dvnd = sub nsw i32 %tmp2, %tmp
  // ;   %tmp3   = xor i32 %tmp1, %divisor
  // ;   %u_dvsr = sub nsw i32 %tmp3, %tmp1
  // ;   %q_sgn  = xor i32 %tmp1, %tmp
  // ;   %q_mag  = udiv i32 %u_dvnd, %u_dvsr
  // ;   %tmp4   = xor i32 %q_mag, %q_sgn
  // ;   %q      = sub i32 %tmp4, %q_sgn
```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 125-182

```cpp
  Dividend      = Builder.CreateFreeze(Dividend);
  Divisor       = Builder.CreateFreeze(Divisor);
  Value *Tmp    = Builder.CreateAShr(Dividend, Shift);
  Value *Tmp1   = Builder.CreateAShr(Divisor, Shift);
  Value *Tmp2   = Builder.CreateXor(Tmp, Dividend);
  Value *U_Dvnd = Builder.CreateSub(Tmp2, Tmp);
  Value *Tmp3   = Builder.CreateXor(Tmp1, Divisor);
  Value *U_Dvsr = Builder.CreateSub(Tmp3, Tmp1);
  Value *Q_Sgn  = Builder.CreateXor(Tmp1, Tmp);
  Value *Q_Mag  = Builder.CreateUDiv(U_Dvnd, U_Dvsr);
  Value *Tmp4   = Builder.CreateXor(Q_Mag, Q_Sgn);
  Value *Q      = Builder.CreateSub(Tmp4, Q_Sgn);

  if (Instruction *UDiv = dyn_cast<Instruction>(Q_Mag))
    Builder.SetInsertPoint(UDiv);

  return Q;
}

/// Generates code to divide two unsigned scalar 32-bit or 64-bit integers.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // Our CFG is going to look like:
  // +---------------------+
  // | special-cases       |
  // |   ...               |
  // +---------------------+
  //  |       |
  //  |   +----------+
  //  |   |  bb1     |
  //  |   |  ...     |
  //  |   +----------+
  //  |    |      |
  //  |    |  +------------+
  //  |    |  |  preheader |
  //  |    |  |  ...       |
```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 183-244

```cpp
  //  |    |  +------------+
  //  |    |      |
  //  |    |      |      +---+
  //  |    |      |      |   |
  //  |    |  +------------+ |
  //  |    |  |  do-while  | |
  //  |    |  |  ...       | |
  //  |    |  +------------+ |
  //  |    |      |      |   |
  //  |   +-----------+  +---+
  //  |   | loop-exit |
  //  |   |  ...      |
  //  |   +-----------+
  //  |     |
  // +-------+
  // | ...   |
  // | end   |
  // +-------+
  BasicBlock *SpecialCases = Builder.GetInsertBlock();
  SpecialCases->setName(Twine(SpecialCases->getName(), "_udiv-special-cases"));
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // ;   %retVal      = select i1 %ret0, i32 0, i32 %dividend
  // ;   %earlyRet    = select i1 %ret0, i1 true, %retDividend
  // ;   br i1 %earlyRet, label %end, label %bb1
  Builder.SetInsertPoint(SpecialCases);
  Divisor            = Builder.CreateFreeze(Divisor);
  Dividend           = Builder.CreateFreeze(Dividend);
  Value *Ret0_1      = Builder.CreateICmpEQ(Divisor, Zero);
  Value *Ret0_2      = Builder.CreateICmpEQ(Dividend, Zero);
  Value *Ret0_3      = Builder.CreateOr(Ret0_1, Ret0_2);
  Value *Tmp0 = Builder.CreateCall(CTLZ, {Divisor, True});
  Value *Tmp1 = Builder.CreateCall(CTLZ, {Dividend, True});
  Value *SR          = Builder.CreateSub(Tmp0, Tmp1);
  Value *Ret0_4      = Builder.CreateICmpUGT(SR, MSB);

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 245-306

```cpp
  // Add 'unlikely' branch weights. We mark the case where either the divisor
  // or the dividend is equal to zero as unlikely.
  Value *Ret0        = Builder.CreateLogicalOr(Ret0_3, Ret0_4);
  applyProfMetadataIfEnabled(Ret0, [&](Instruction *Inst) {
    Inst->setMetadata(
        LLVMContext::MD_prof,
        MDBuilder(Inst->getContext()).createUnlikelyBranchWeights());
  });
  Value *RetDividend = Builder.CreateICmpEQ(SR, MSB);

  // Conservatively, we treat the case |divisor| > |dividend| as unknown
  Value *RetVal      = Builder.CreateSelect(Ret0, Zero, Dividend);
  applyProfMetadataIfEnabled(RetVal, [&](Instruction *Inst) {
    setExplicitlyUnknownBranchWeightsIfProfiled(*Inst, DEBUG_TYPE, F);
  });
  Value *EarlyRet    = Builder.CreateLogicalOr(Ret0, RetDividend);
  applyProfMetadataIfEnabled(EarlyRet, [&](Instruction *Inst) {
    setExplicitlyUnknownBranchWeightsIfProfiled(*Inst, DEBUG_TYPE, F);
  });

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    Inst->setMetadata(
        LLVMContext::MD_prof,
        MDBuilder(Inst->getContext()).createUnlikelyBranchWeights());
  });

  // ; preheader:                                           ; preds = %bb1
  // ;   %tmp3 = lshr i32 %dividend, %sr_1
  // ;   %tmp4 = add i32 %divisor, -1
  // ;   br label %do-while
  Builder.SetInsertPoint(Preheader);
  Value *Tmp3 = Builder.CreateLShr(Dividend, SR_1);
  Value *Tmp4 = Builder.CreateAdd(Divisor, NegOne);
  Builder.CreateBr(DoWhile);

```
- EN: Core entities appearing here include applyProfMetadataIfEnabled, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 applyProfMetadataIfEnabled，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 307-370

```cpp
  // ; do-while:                                 ; preds = %do-while, %preheader
  // ;   %carry_1 = phi i32 [ 0, %preheader ], [ %carry, %do-while ]
  // ;   %sr_3    = phi i32 [ %sr_1, %preheader ], [ %sr_2, %do-while ]
  // ;   %r_1     = phi i32 [ %tmp3, %preheader ], [ %r, %do-while ]
  // ;   %q_2     = phi i32 [ %q, %preheader ], [ %q_1, %do-while ]
  // ;   %tmp5  = shl i32 %r_1, 1
  // ;   %tmp6  = lshr i32 %q_2, 31
  // ;   %tmp7  = or i32 %tmp5, %tmp6
  // ;   %tmp8  = shl i32 %q_2, 1
  // ;   %q_1   = or i32 %carry_1, %tmp8
  // ;   %tmp9  = sub i32 %tmp4, %tmp7
  // ;   %tmp10 = ashr i32 %tmp9, 31
  // ;   %carry = and i32 %tmp10, 1
  // ;   %tmp11 = and i32 %tmp10, %divisor
  // ;   %r     = sub i32 %tmp7, %tmp11
  // ;   %sr_2  = add i32 %sr_3, -1
  // ;   %tmp12 = icmp eq i32 %sr_2, 0
  // ;   br i1 %tmp12, label %loop-exit, label %do-while
  Builder.SetInsertPoint(DoWhile);
  PHINode *Carry_1 = Builder.CreatePHI(DivTy, 2);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // ;   br label %end
  Builder.SetInsertPoint(LoopExit);
  PHINode *Carry_2 = Builder.CreatePHI(DivTy, 2);
  PHINode *Q_3     = Builder.CreatePHI(DivTy, 2);
  Value *Tmp13 = Builder.CreateShl(Q_3, One);
  Value *Q_4   = Builder.CreateOr(Carry_2, Tmp13);
  Builder.CreateBr(End);

  // ; end:                                 ; preds = %loop-exit, %special-cases
  // ;   %q_5 = phi i32 [ %q_4, %loop-exit ], [ %retVal, %special-cases ]
  // ;   ret i32 %q_5
  Builder.SetInsertPoint(End, End->begin());
  PHINode *Q_5 = Builder.CreatePHI(DivTy, 2);

```
- EN: Core entities appearing here include applyProfMetadataIfEnabled, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 applyProfMetadataIfEnabled，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 371-432

```cpp
  // Populate the Phis, since all values have now been created. Our Phis were:
  // ;   %carry_1 = phi i32 [ 0, %preheader ], [ %carry, %do-while ]
  Carry_1->addIncoming(Zero, Preheader);
  Carry_1->addIncoming(Carry, DoWhile);
  // ;   %sr_3 = phi i32 [ %sr_1, %preheader ], [ %sr_2, %do-while ]
  SR_3->addIncoming(SR_1, Preheader);
  SR_3->addIncoming(SR_2, DoWhile);
  // ;   %r_1 = phi i32 [ %tmp3, %preheader ], [ %r, %do-while ]
  R_1->addIncoming(Tmp3, Preheader);
  R_1->addIncoming(R, DoWhile);
  // ;   %q_2 = phi i32 [ %q, %preheader ], [ %q_1, %do-while ]
  Q_2->addIncoming(Q, Preheader);
  Q_2->addIncoming(Q_1, DoWhile);
  // ;   %carry_2 = phi i32 [ 0, %bb1 ], [ %carry, %do-while ]
  Carry_2->addIncoming(Zero, BB1);
  Carry_2->addIncoming(Carry, DoWhile);
  // ;   %q_3 = phi i32 [ %q, %bb1 ], [ %q_1, %do-while ]
  Q_3->addIncoming(Q, BB1);
  Q_3->addIncoming(Q_1, DoWhile);
  // ;   %q_5 = phi i32 [ %q_4, %loop-exit ], [ %retVal, %special-cases ]
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    Rem->replaceAllUsesWith(Remainder);
    Rem->dropAllReferences();
    Rem->eraseFromParent();

    // If we didn't actually generate an urem instruction, we're done
    // This happens for example if the input were constant. In this case the
    // Builder insertion point was unchanged
    if (IsInsertPoint)
      return true;

    BinaryOperator *BO = dyn_cast<BinaryOperator>(Builder.GetInsertPoint());
    Rem = BO;
  }

```
- EN: Core entities appearing here include expandRemainder, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 expandRemainder，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 433-493

```cpp
  Value *Remainder = generateUnsignedRemainderCode(Rem->getOperand(0),
                                                   Rem->getOperand(1), Builder);

  Rem->replaceAllUsesWith(Remainder);
  Rem->dropAllReferences();
  Rem->eraseFromParent();

  // Expand the udiv
  if (BinaryOperator *UDiv = dyn_cast<BinaryOperator>(Builder.GetInsertPoint())) {
    assert(UDiv->getOpcode() == Instruction::UDiv && "Non-udiv in expansion?");
    expandDivision(UDiv);
  }

  return true;
}

/// Generate code to divide two integers, replacing Div with the generated
/// code. This currently generates code similarly to compiler-rt's
/// implementations, but future work includes generating more specialized code
/// when more information about the operands are known.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      return true;

    BinaryOperator *BO = dyn_cast<BinaryOperator>(Builder.GetInsertPoint());
    Div = BO;
  }

  // Insert the unsigned division code
  Value *Quotient = generateUnsignedDivisionCode(Div->getOperand(0),
                                                 Div->getOperand(1),
                                                 Builder);
  Div->replaceAllUsesWith(Quotient);
  Div->dropAllReferences();
  Div->eraseFromParent();

```
- EN: Core entities appearing here include expandDivision, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 expandDivision，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 494-557

```cpp
  return true;
}

/// Generate code to compute the remainder of two integers of bitwidth up to
/// 32 bits. Uses the above routines and extends the inputs/truncates the
/// outputs to operate in 32 bits; that is, these routines are good for targets
/// that have no or very little suppport for smaller than 32 bit integer
/// arithmetic.
///
/// Replace Rem with emulation code.
bool llvm::expandRemainderUpTo32Bits(BinaryOperator *Rem) {
  assert((Rem->getOpcode() == Instruction::SRem ||
          Rem->getOpcode() == Instruction::URem) &&
          "Trying to expand remainder from a non-remainder function");

  Type *RemTy = Rem->getType();
  assert(!RemTy->isVectorTy() && "Div over vectors not supported");

  unsigned RemTyBitWidth = RemTy->getIntegerBitWidth();

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

  return expandRemainder(cast<BinaryOperator>(ExtRem));
}

/// Generate code to compute the remainder of two integers of bitwidth up to
/// 64 bits. Uses the above routines and extends the inputs/truncates the
/// outputs to operate in 64 bits.
///
/// Replace Rem with emulation code.
bool llvm::expandRemainderUpTo64Bits(BinaryOperator *Rem) {
  assert((Rem->getOpcode() == Instruction::SRem ||
          Rem->getOpcode() == Instruction::URem) &&
          "Trying to expand remainder from a non-remainder function");

```
- EN: Core entities appearing here include expandRemainderUpTo32Bits, expandRemainderUpTo64Bits, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 expandRemainderUpTo32Bits, expandRemainderUpTo64Bits，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 558-618

```cpp
  Type *RemTy = Rem->getType();
  assert(!RemTy->isVectorTy() && "Div over vectors not supported");

  unsigned RemTyBitWidth = RemTy->getIntegerBitWidth();

  if (RemTyBitWidth >= 64)
    return expandRemainder(Rem);

  // If bitwidth smaller than 64 extend inputs, extend output and proceed
  // with 64 bit division.
  IRBuilder<> Builder(Rem);

  Value *ExtDividend;
  Value *ExtDivisor;
  Value *ExtRem;
  Value *Trunc;
  Type *Int64Ty = Builder.getInt64Ty();

  if (Rem->getOpcode() == Instruction::SRem) {
    ExtDividend = Builder.CreateSExt(Rem->getOperand(0), Int64Ty);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  Type *DivTy = Div->getType();
  assert(!DivTy->isVectorTy() && "Div over vectors not supported");

  unsigned DivTyBitWidth = DivTy->getIntegerBitWidth();

  assert(DivTyBitWidth <= 32 && "Div of bitwidth greater than 32 not supported");

  if (DivTyBitWidth == 32)
    return expandDivision(Div);

  // If bitwidth smaller than 32 extend inputs, extend output and proceed
  // with 32 bit division.
  IRBuilder<> Builder(Div);

```
- EN: Core entities appearing here include expandDivisionUpTo32Bits, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 expandDivisionUpTo32Bits，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 619-681

```cpp
  Value *ExtDividend;
  Value *ExtDivisor;
  Value *ExtDiv;
  Value *Trunc;
  Type *Int32Ty = Builder.getInt32Ty();

  if (Div->getOpcode() == Instruction::SDiv) {
    ExtDividend = Builder.CreateSExt(Div->getOperand(0), Int32Ty);
    ExtDivisor = Builder.CreateSExt(Div->getOperand(1), Int32Ty);
    ExtDiv = Builder.CreateSDiv(ExtDividend, ExtDivisor);
  } else {
    ExtDividend = Builder.CreateZExt(Div->getOperand(0), Int32Ty);
    ExtDivisor = Builder.CreateZExt(Div->getOperand(1), Int32Ty);
    ExtDiv = Builder.CreateUDiv(ExtDividend, ExtDivisor);
  }
  Trunc = Builder.CreateTrunc(ExtDiv, DivTy);

  Div->replaceAllUsesWith(Trunc);
  Div->dropAllReferences();
  Div->eraseFromParent();
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  Value *Trunc;
  Type *Int64Ty = Builder.getInt64Ty();

  if (Div->getOpcode() == Instruction::SDiv) {
    ExtDividend = Builder.CreateSExt(Div->getOperand(0), Int64Ty);
    ExtDivisor = Builder.CreateSExt(Div->getOperand(1), Int64Ty);
    ExtDiv = Builder.CreateSDiv(ExtDividend, ExtDivisor);
  } else {
    ExtDividend = Builder.CreateZExt(Div->getOperand(0), Int64Ty);
    ExtDivisor = Builder.CreateZExt(Div->getOperand(1), Int64Ty);
    ExtDiv = Builder.CreateUDiv(ExtDividend, ExtDivisor);
  }
  Trunc = Builder.CreateTrunc(ExtDiv, DivTy);

```
- EN: Core entities appearing here include expandDivisionUpTo64Bits, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 expandDivisionUpTo64Bits，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 682-687

```cpp
  Div->replaceAllUsesWith(Trunc);
  Div->dropAllReferences();
  Div->eraseFromParent();

  return expandDivision(cast<BinaryOperator>(ExtDiv));
}
```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `applyProfMetadataIfEnabled, expandRemainder, expandDivision, expandRemainderUpTo32Bits, expandRemainderUpTo64Bits, expandDivisionUpTo32Bits, expandDivisionUpTo64Bits` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`applyProfMetadataIfEnabled, expandRemainder, expandDivision, expandRemainderUpTo32Bits, expandRemainderUpTo64Bits, expandDivisionUpTo32Bits, expandDivisionUpTo64Bits` 构成该文件对外 API 与主要实现挂钩。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: IR/transform headers: `llvm/IR/Function.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h`, `llvm/IR/Intrinsics.h`, `llvm/IR/LLVMContext.h`, `llvm/IR/MDBuilder.h`, `llvm/IR/ProfDataUtils.h`, `llvm/IR/Value.h`, `llvm/Transforms/Utils/IntegerDivision.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Function.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h`, `llvm/IR/Intrinsics.h`, `llvm/IR/LLVMContext.h`, `llvm/IR/MDBuilder.h`, `llvm/IR/ProfDataUtils.h`, `llvm/IR/Value.h`, `llvm/Transforms/Utils/IntegerDivision.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/Casting.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Casting.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
