# AVRShiftExpand.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/AVR/AVRShiftExpand.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Provides target-specific support code, declarations, or helper routines used by the backend.
- 目的（中文）: 提供目标后端使用的专用支持代码、声明或辅助例程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===- AVRShift.cpp - Shift Expansion Pass --------------------------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: /// \file
  10: /// Expand non-8-bit and non-16-bit shift instructions (shl, lshr, ashr) to
  11: /// inline loops, just like avr-gcc. This must be done in IR because otherwise
  12: /// the type legalizer will turn 32-bit shifts into (non-existing) library calls
  13: /// such as __ashlsi3.
  14: //
  15: //===----------------------------------------------------------------------===//
  16: 
  17: #include "AVR.h"
  18: #include "llvm/IR/IRBuilder.h"
  19: #include "llvm/IR/InstIterator.h"
  20: 
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。

### Lines 21-40

```cpp
  21: using namespace llvm;
  22: 
  23: namespace {
  24: 
  25: class AVRShiftExpand : public FunctionPass {
  26: public:
  27:   static char ID;
  28: 
  29:   AVRShiftExpand() : FunctionPass(ID) {}
  30: 
  31:   bool runOnFunction(Function &F) override;
  32: 
  33:   StringRef getPassName() const override { return "AVR Shift Expansion"; }
  34: 
  35: private:
  36:   void expand(BinaryOperator *BI);
  37: };
  38: 
  39: } // end of anonymous namespace
  40: 
```

- EN: The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. This chunk introduces interfaces or data structures such as AVRShiftExpand, which organize the target-specific behavior exposed by the file.
- 中文: 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 这一段引入了 AVRShiftExpand 等接口或数据结构，用于组织该文件暴露的目标专用行为。

### Lines 41-60

```cpp
  41: char AVRShiftExpand::ID = 0;
  42: 
  43: INITIALIZE_PASS(AVRShiftExpand, "avr-shift-expand", "AVR Shift Expansion",
  44:                 false, false)
  45: 
  46: Pass *llvm::createAVRShiftExpandPass() { return new AVRShiftExpand(); }
  47: 
  48: bool AVRShiftExpand::runOnFunction(Function &F) {
  49:   SmallVector<BinaryOperator *, 1> ShiftInsts;
  50:   auto &Ctx = F.getContext();
  51:   for (Instruction &I : instructions(F)) {
  52:     if (!I.isShift())
  53:       // Only expand shift instructions (shl, lshr, ashr).
  54:       continue;
  55:     if (I.getType() == Type::getInt8Ty(Ctx) ||
  56:         I.getType() == Type::getInt16Ty(Ctx))
  57:       // Only expand non-8-bit and non-16-bit shifts, since those are expanded
  58:       // directly during isel.
  59:       continue;
  60:     if (isa<ConstantInt>(I.getOperand(1)))
```

- EN: Function bodies or method definitions such as runOnFunction contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: runOnFunction 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 61-80

```cpp
  61:       // Only expand when the shift amount is not known.
  62:       // Known shift amounts are (currently) better expanded inline.
  63:       continue;
  64:     ShiftInsts.push_back(cast<BinaryOperator>(&I));
  65:   }
  66: 
  67:   // The expanding itself needs to be done separately as expand() will remove
  68:   // these instructions. Removing instructions while iterating over a basic
  69:   // block is not a great idea.
  70:   for (auto *I : ShiftInsts) {
  71:     expand(I);
  72:   }
  73: 
  74:   // Return whether this function expanded any shift instructions.
  75:   return ShiftInsts.size() > 0;
  76: }
  77: 
  78: void AVRShiftExpand::expand(BinaryOperator *BI) {
  79:   auto &Ctx = BI->getContext();
  80:   IRBuilder<> Builder(BI);
```

- EN: Function bodies or method definitions such as expand contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis.
- 中文: expand 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。

### Lines 81-100

```cpp
  81:   Type *InputTy = cast<Instruction>(BI)->getType();
  82:   Type *Int8Ty = Type::getInt8Ty(Ctx);
  83:   Value *Int8Zero = ConstantInt::get(Int8Ty, 0);
  84: 
  85:   // Split the current basic block at the point of the existing shift
  86:   // instruction and insert a new basic block for the loop.
  87:   BasicBlock *BB = BI->getParent();
  88:   Function *F = BB->getParent();
  89:   BasicBlock *EndBB = BB->splitBasicBlock(BI, "shift.done");
  90:   BasicBlock *LoopBB = BasicBlock::Create(Ctx, "shift.loop", F, EndBB);
  91: 
  92:   // Truncate the shift amount to i8, which is trivially lowered to a single
  93:   // AVR register.
  94:   Builder.SetInsertPoint(&BB->back());
  95:   Value *ShiftAmount = Builder.CreateTrunc(BI->getOperand(1), Int8Ty);
  96: 
  97:   // Replace the unconditional branch that splitBasicBlock created with a
  98:   // conditional branch.
  99:   Value *Cmp1 = Builder.CreateICmpEQ(ShiftAmount, Int8Zero);
 100:   Builder.CreateCondBr(Cmp1, EndBB, LoopBB);
```

- EN: This range continues the implementation of the backend component described by AVRShiftExpand.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 101-120

```cpp
 101:   BB->back().eraseFromParent();
 102: 
 103:   // Create the loop body starting with PHI nodes.
 104:   Builder.SetInsertPoint(LoopBB);
 105:   PHINode *ShiftAmountPHI = Builder.CreatePHI(Int8Ty, 2);
 106:   ShiftAmountPHI->addIncoming(ShiftAmount, BB);
 107:   PHINode *ValuePHI = Builder.CreatePHI(InputTy, 2);
 108:   ValuePHI->addIncoming(BI->getOperand(0), BB);
 109: 
 110:   // Subtract the shift amount by one, as we're shifting one this loop
 111:   // iteration.
 112:   Value *ShiftAmountSub =
 113:       Builder.CreateSub(ShiftAmountPHI, ConstantInt::get(Int8Ty, 1));
 114:   ShiftAmountPHI->addIncoming(ShiftAmountSub, LoopBB);
 115: 
 116:   // Emit the actual shift instruction. The difference is that this shift
 117:   // instruction has a constant shift amount, which can be emitted inline
 118:   // without a library call.
 119:   Value *ValueShifted;
 120:   switch (BI->getOpcode()) {
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。

### Lines 121-140

```cpp
 121:   case Instruction::Shl:
 122:     ValueShifted = Builder.CreateShl(ValuePHI, ConstantInt::get(InputTy, 1));
 123:     break;
 124:   case Instruction::LShr:
 125:     ValueShifted = Builder.CreateLShr(ValuePHI, ConstantInt::get(InputTy, 1));
 126:     break;
 127:   case Instruction::AShr:
 128:     ValueShifted = Builder.CreateAShr(ValuePHI, ConstantInt::get(InputTy, 1));
 129:     break;
 130:   default:
 131:     llvm_unreachable("asked to expand an instruction that is not a shift");
 132:   }
 133:   ValuePHI->addIncoming(ValueShifted, LoopBB);
 134: 
 135:   // Branch to either the loop again (if there is more to shift) or to the
 136:   // basic block after the loop (if all bits are shifted).
 137:   Value *Cmp2 = Builder.CreateICmpEQ(ShiftAmountSub, Int8Zero);
 138:   Builder.CreateCondBr(Cmp2, EndBB, LoopBB);
 139: 
 140:   // Collect the resulting value. This is necessary in the IR but won't produce
```

- EN: This range continues the implementation of the backend component described by AVRShiftExpand.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 141-150

```cpp
 141:   // any actual instructions.
 142:   Builder.SetInsertPoint(BI);
 143:   PHINode *Result = Builder.CreatePHI(InputTy, 2);
 144:   Result->addIncoming(BI->getOperand(0), BB);
 145:   Result->addIncoming(ValueShifted, LoopBB);
 146: 
 147:   // Replace the original shift instruction.
 148:   BI->replaceAllUsesWith(Result);
 149:   BI->eraseFromParent();
 150: }
```

- EN: This range continues the implementation of the backend component described by AVRShiftExpand.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

## Key Concepts / 关键概念

- Target backend support code / 目标后端支持代码
- LLVM CodeGen integration / LLVM CodeGen 集成

## Dependencies / 依赖关系

- Direct includes / 直接包含: `AVR.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/InstIterator.h`
