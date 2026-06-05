# RISCVMCPlusBuilder.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Target/RISCV/RISCVMCPlusBuilder.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: bolt/Target/RISCV/RISCVMCPlusBuilder.cpp This file provides RISCV-specific MCPlus builder.. It also sits in code that implements target-specific BOLT behavior. / 该文件实现目标架构相关的 BOLT 行为。 源码头部说明其职责是：bolt/Target/RISCV/RISCVMCPlusBuilder.cpp This file provides RISCV-specific MCPlus builder.。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12

```cpp
//===- bolt/Target/RISCV/RISCVMCPlusBuilder.cpp -----------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file provides RISCV-specific MCPlus builder.
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 13-22

```cpp
#include "MCTargetDesc/RISCVMCAsmInfo.h"
#include "MCTargetDesc/RISCVMCTargetDesc.h"
#include "bolt/Core/MCPlusBuilder.h"
#include "llvm/BinaryFormat/ELF.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCInstBuilder.h"
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/Support/ErrorHandling.h"
```

- EN: Pulls in 9 header(s) from local project, LLVM, system dependencies needed by this range. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 9 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里的注释记录了设计假设、不变量或使用说明。

### Lines 23-33

```cpp
#define DEBUG_TYPE "mcplus"

using namespace llvm;
using namespace bolt;

namespace {

class RISCVMCPlusBuilder : public MCPlusBuilder {
public:
  using MCPlusBuilder::MCPlusBuilder;
```

- EN: Establishes header-guard or prologue macros so the file can be compiled safely. Works inside namespace scope `llvm`, `bolt` to organize symbols. Introduces type definitions such as `RISCVMCPlusBuilder`. Defines macros such as `DEBUG_TYPE` for constants or compile-time switches.
- CN: 这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里位于命名空间 `llvm`, `bolt` 中，用于组织符号作用域。这里引入类型定义，例如 `RISCVMCPlusBuilder`。这里定义宏 `DEBUG_TYPE`，用于常量或编译期开关。

### Lines 34-44

```cpp
  bool equals(const MCSpecifierExpr &A, const MCSpecifierExpr &B,
              CompFuncTy Comp) const override {
    const auto &RISCVExprA = cast<MCSpecifierExpr>(A);
    const auto &RISCVExprB = cast<MCSpecifierExpr>(B);
    if (RISCVExprA.getSpecifier() != RISCVExprB.getSpecifier())
      return false;

    return MCPlusBuilder::equals(*RISCVExprA.getSubExpr(),
                                 *RISCVExprB.getSubExpr(), Comp);
  }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 45-60

```cpp
  void getCalleeSavedRegs(BitVector &Regs) const override {
    Regs |= getAliases(RISCV::X2);
    Regs |= getAliases(RISCV::X8);
    Regs |= getAliases(RISCV::X9);
    Regs |= getAliases(RISCV::X18);
    Regs |= getAliases(RISCV::X19);
    Regs |= getAliases(RISCV::X20);
    Regs |= getAliases(RISCV::X21);
    Regs |= getAliases(RISCV::X22);
    Regs |= getAliases(RISCV::X23);
    Regs |= getAliases(RISCV::X24);
    Regs |= getAliases(RISCV::X25);
    Regs |= getAliases(RISCV::X26);
    Regs |= getAliases(RISCV::X27);
  }
```

- EN: Declares or implements routines including `getCalleeSavedRegs`, `getAliases`. Notable symbols here include `getCalleeSavedRegs`, `getAliases`.
- CN: 这里声明或实现函数，例如 `getCalleeSavedRegs`, `getAliases`。这里较值得关注的符号包括 `getCalleeSavedRegs`, `getAliases`。

### Lines 61-78

```cpp
  bool shouldRecordCodeRelocation(uint32_t RelType) const override {
    switch (RelType) {
    case ELF::R_RISCV_JAL:
    case ELF::R_RISCV_CALL:
    case ELF::R_RISCV_CALL_PLT:
    case ELF::R_RISCV_BRANCH:
    case ELF::R_RISCV_RVC_BRANCH:
    case ELF::R_RISCV_RVC_JUMP:
    case ELF::R_RISCV_GOT_HI20:
    case ELF::R_RISCV_PCREL_HI20:
    case ELF::R_RISCV_PCREL_LO12_I:
    case ELF::R_RISCV_PCREL_LO12_S:
    case ELF::R_RISCV_HI20:
    case ELF::R_RISCV_LO12_I:
    case ELF::R_RISCV_LO12_S:
    case ELF::R_RISCV_TLS_GOT_HI20:
    case ELF::R_RISCV_TLS_GD_HI20:
      return true;
```

- EN: Declares or implements routines including `shouldRecordCodeRelocation`. Notable symbols here include `shouldRecordCodeRelocation`.
- CN: 这里声明或实现函数，例如 `shouldRecordCodeRelocation`。这里较值得关注的符号包括 `shouldRecordCodeRelocation`。

### Lines 79-90

```cpp
    default:
      llvm_unreachable("Unexpected RISCV relocation type in code");
    }
  }

  bool isNop(const MCInst &Inst) const {
    return Inst.getOpcode() == RISCV::ADDI &&
           Inst.getOperand(0).getReg() == RISCV::X0 &&
           Inst.getOperand(1).getReg() == RISCV::X0 &&
           Inst.getOperand(2).getImm() == 0;
  }
```

- EN: Declares or implements routines including `llvm_unreachable`, `isNop`. Notable symbols here include `llvm_unreachable`, `isNop`.
- CN: 这里声明或实现函数，例如 `llvm_unreachable`, `isNop`。这里较值得关注的符号包括 `llvm_unreachable`, `isNop`。

### Lines 91-98

```cpp
  bool isCNop(const MCInst &Inst) const {
    return Inst.getOpcode() == RISCV::C_NOP;
  }

  bool isNoop(const MCInst &Inst) const override {
    return isNop(Inst) || isCNop(Inst);
  }
```

- EN: Declares or implements routines including `isCNop`, `isNoop`. Notable symbols here include `isCNop`, `isNoop`.
- CN: 这里声明或实现函数，例如 `isCNop`, `isNoop`。这里较值得关注的符号包括 `isCNop`, `isNoop`。

### Lines 99-108

```cpp
  bool isPseudo(const MCInst &Inst) const override {
    switch (Inst.getOpcode()) {
    default:
      return MCPlusBuilder::isPseudo(Inst);
    case RISCV::PseudoCALL:
    case RISCV::PseudoTAIL:
      return false;
    }
  }
```

- EN: Declares or implements routines including `isPseudo`. Notable symbols here include `isPseudo`.
- CN: 这里声明或实现函数，例如 `isPseudo`。这里较值得关注的符号包括 `isPseudo`。

### Lines 109-122

```cpp
  bool isIndirectCall(const MCInst &Inst) const override {
    if (!isCall(Inst))
      return false;

    switch (Inst.getOpcode()) {
    default:
      return false;
    case RISCV::JALR:
    case RISCV::C_JALR:
    case RISCV::C_JR:
      return true;
    }
  }
```

- EN: Declares or implements routines including `isIndirectCall`. Notable symbols here include `isIndirectCall`.
- CN: 这里声明或实现函数，例如 `isIndirectCall`。这里较值得关注的符号包括 `isIndirectCall`。

### Lines 123-132

```cpp
  bool hasPCRelOperand(const MCInst &Inst) const override {
    switch (Inst.getOpcode()) {
    default:
      return false;
    case RISCV::JAL:
    case RISCV::AUIPC:
      return true;
    }
  }
```

- EN: Declares or implements routines including `hasPCRelOperand`. Notable symbols here include `hasPCRelOperand`.
- CN: 这里声明或实现函数，例如 `hasPCRelOperand`。这里较值得关注的符号包括 `hasPCRelOperand`。

### Lines 133-150

```cpp
  unsigned getInvertedBranchOpcode(unsigned Opcode) const {
    switch (Opcode) {
    default:
      llvm_unreachable("Failed to invert branch opcode");
      return Opcode;
    case RISCV::BEQ:
      return RISCV::BNE;
    case RISCV::BNE:
      return RISCV::BEQ;
    case RISCV::BLT:
      return RISCV::BGE;
    case RISCV::BGE:
      return RISCV::BLT;
    case RISCV::BLTU:
      return RISCV::BGEU;
    case RISCV::BGEU:
      return RISCV::BLTU;
    case RISCV::C_BEQZ:
```

- EN: Declares or implements routines including `getInvertedBranchOpcode`, `llvm_unreachable`. Notable symbols here include `getInvertedBranchOpcode`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `getInvertedBranchOpcode`, `llvm_unreachable`。这里较值得关注的符号包括 `getInvertedBranchOpcode`, `llvm_unreachable`。

### Lines 151-163

```cpp
      return RISCV::C_BNEZ;
    case RISCV::C_BNEZ:
      return RISCV::C_BEQZ;
    }
  }

  void reverseBranchCondition(MCInst &Inst, const MCSymbol *TBB,
                              MCContext *Ctx) const override {
    auto Opcode = getInvertedBranchOpcode(Inst.getOpcode());
    Inst.setOpcode(Opcode);
    replaceBranchTarget(Inst, TBB, Ctx);
  }
```

- EN: Declares or implements routines including `getInvertedBranchOpcode`, `replaceBranchTarget`. Notable symbols here include `getInvertedBranchOpcode`, `replaceBranchTarget`.
- CN: 这里声明或实现函数，例如 `getInvertedBranchOpcode`, `replaceBranchTarget`。这里较值得关注的符号包括 `getInvertedBranchOpcode`, `replaceBranchTarget`。

### Lines 164-173

```cpp
  void replaceBranchTarget(MCInst &Inst, const MCSymbol *TBB,
                           MCContext *Ctx) const override {
    assert((isCall(Inst) || isBranch(Inst)) && !isIndirectBranch(Inst) &&
           "Invalid instruction");

    unsigned SymOpIndex;
    auto Result = getSymbolRefOperandNum(Inst, SymOpIndex);
    (void)Result;
    assert(Result && "unimplemented branch");
```

- EN: Declares or implements routines including `assert`, `getSymbolRefOperandNum`. Notable symbols here include `assert`, `getSymbolRefOperandNum`.
- CN: 这里声明或实现函数，例如 `assert`, `getSymbolRefOperandNum`。这里较值得关注的符号包括 `assert`, `getSymbolRefOperandNum`。

### Lines 174-190

```cpp
    Inst.getOperand(SymOpIndex) =
        MCOperand::createExpr(MCSymbolRefExpr::create(TBB, *Ctx));
  }

  IndirectBranchType analyzeIndirectBranch(
      MCInst &Instruction, InstructionIterator Begin, InstructionIterator End,
      const unsigned PtrSize, MCInst *&MemLocInstr, unsigned &BaseRegNum,
      unsigned &IndexRegNum, int64_t &DispValue, const MCExpr *&DispExpr,
      MCInst *&PCRelBaseOut, MCInst *&FixedEntryLoadInst) const override {
    MemLocInstr = nullptr;
    BaseRegNum = 0;
    IndexRegNum = 0;
    DispValue = 0;
    DispExpr = nullptr;
    PCRelBaseOut = nullptr;
    FixedEntryLoadInst = nullptr;
```

- EN: Declares or implements routines including `createExpr`. Notable symbols here include `createExpr`.
- CN: 这里声明或实现函数，例如 `createExpr`。这里较值得关注的符号包括 `createExpr`。

### Lines 191-200

```cpp
    // Check for the following long tail call sequence:
    // 1: auipc xi, %pcrel_hi(sym)
    // jalr zero, %pcrel_lo(1b)(xi)
    if (Instruction.getOpcode() == RISCV::JALR && Begin != End) {
      MCInst &PrevInst = *std::prev(End);
      if (isRISCVCall(PrevInst, Instruction) &&
          Instruction.getOperand(0).getReg() == RISCV::X0)
        return IndirectBranchType::POSSIBLE_TAIL_CALL;
    }
```

- EN: Declares or implements routines including `prev`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `prev`.
- CN: 这里声明或实现函数，例如 `prev`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `prev`。

### Lines 201-217

```cpp
    return IndirectBranchType::UNKNOWN;
  }

  bool convertJmpToTailCall(MCInst &Inst) override {
    if (isTailCall(Inst))
      return false;

    switch (Inst.getOpcode()) {
    default:
      llvm_unreachable("unsupported tail call opcode");
    case RISCV::JAL:
    case RISCV::JALR:
    case RISCV::C_J:
    case RISCV::C_JR:
      break;
    }
```

- EN: Declares or implements routines including `convertJmpToTailCall`, `llvm_unreachable`. Notable symbols here include `convertJmpToTailCall`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `convertJmpToTailCall`, `llvm_unreachable`。这里较值得关注的符号包括 `convertJmpToTailCall`, `llvm_unreachable`。

### Lines 218-230

```cpp
    setTailCall(Inst);
    return true;
  }

  void createReturn(MCInst &Inst) const override {
    // TODO "c.jr ra" when RVC is enabled
    Inst.setOpcode(RISCV::JALR);
    Inst.clear();
    Inst.addOperand(MCOperand::createReg(RISCV::X0));
    Inst.addOperand(MCOperand::createReg(RISCV::X1));
    Inst.addOperand(MCOperand::createImm(0));
  }
```

- EN: Declares or implements routines including `setTailCall`, `createReturn`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `setTailCall`, `createReturn`.
- CN: 这里声明或实现函数，例如 `setTailCall`, `createReturn`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `setTailCall`, `createReturn`。

### Lines 231-238

```cpp
  void createUncondBranch(MCInst &Inst, const MCSymbol *TBB,
                          MCContext *Ctx) const override {
    Inst.setOpcode(RISCV::JAL);
    Inst.clear();
    Inst.addOperand(MCOperand::createReg(RISCV::X0));
    Inst.addOperand(MCOperand::createExpr(MCSymbolRefExpr::create(TBB, *Ctx)));
  }
```

- EN: Continues the surrounding implementation, wiring declarations, state, and helper logic together.
- CN: 这里继续展开周边实现，衔接声明、状态与辅助逻辑。

### Lines 239-250

```cpp
  StringRef getTrapFillValue() const override {
    return StringRef("\0\0\0\0", 4);
  }

  void createCall(unsigned Opcode, MCInst &Inst, const MCSymbol *Target,
                  MCContext *Ctx) {
    Inst.setOpcode(Opcode);
    Inst.clear();
    Inst.addOperand(MCOperand::createExpr(MCSpecifierExpr::create(
        MCSymbolRefExpr::create(Target, *Ctx), RISCV::S_CALL_PLT, *Ctx)));
  }
```

- EN: Declares or implements routines including `getTrapFillValue`, `create`. Notable symbols here include `getTrapFillValue`, `create`.
- CN: 这里声明或实现函数，例如 `getTrapFillValue`, `create`。这里较值得关注的符号包括 `getTrapFillValue`, `create`。

### Lines 251-260

```cpp
  void createCall(MCInst &Inst, const MCSymbol *Target,
                  MCContext *Ctx) override {
    return createCall(RISCV::PseudoCALL, Inst, Target, Ctx);
  }

  void createLongTailCall(InstructionListType &Seq, const MCSymbol *Target,
                          MCContext *Ctx) override {
    createShortJmp(Seq, Target, Ctx, /*IsTailCall*/ true);
  }
```

- EN: Declares or implements routines including `createShortJmp`. Notable symbols here include `createShortJmp`.
- CN: 这里声明或实现函数，例如 `createShortJmp`。这里较值得关注的符号包括 `createShortJmp`。

### Lines 261-271

```cpp
  void createTailCall(MCInst &Inst, const MCSymbol *Target,
                      MCContext *Ctx) override {
    return createCall(RISCV::PseudoTAIL, Inst, Target, Ctx);
  }

  bool analyzeBranch(InstructionIterator Begin, InstructionIterator End,
                     const MCSymbol *&TBB, const MCSymbol *&FBB,
                     MCInst *&CondBranch,
                     MCInst *&UncondBranch) const override {
    auto I = End;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 272-282

```cpp
    while (I != Begin) {
      --I;

      // Ignore nops and CFIs
      if (isPseudo(*I) || isNoop(*I))
        continue;

      // Stop when we find the first non-terminator
      if (!isTerminator(*I) || isTailCall(*I) || !isBranch(*I))
        break;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 283-295

```cpp
      // Handle unconditional branches.
      if (isUnconditionalBranch(*I)) {
        // If any code was seen after this unconditional branch, we've seen
        // unreachable code. Ignore them.
        CondBranch = nullptr;
        UncondBranch = &*I;
        const MCSymbol *Sym = getTargetSymbol(*I);
        assert(Sym != nullptr &&
               "Couldn't extract BB symbol from jump operand");
        TBB = Sym;
        continue;
      }
```

- EN: Declares or implements routines including `getTargetSymbol`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getTargetSymbol`.
- CN: 这里声明或实现函数，例如 `getTargetSymbol`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getTargetSymbol`。

### Lines 296-311

```cpp
      // Handle conditional branches and ignore indirect branches
      if (isIndirectBranch(*I))
        return false;

      if (CondBranch == nullptr) {
        const MCSymbol *TargetBB = getTargetSymbol(*I);
        if (TargetBB == nullptr) {
          // Unrecognized branch target
          return false;
        }
        FBB = TBB;
        TBB = TargetBB;
        CondBranch = &*I;
        continue;
      }
```

- EN: Declares or implements routines including `getTargetSymbol`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getTargetSymbol`.
- CN: 这里声明或实现函数，例如 `getTargetSymbol`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getTargetSymbol`。

### Lines 312-329

```cpp
      llvm_unreachable("multiple conditional branches in one BB");
    }

    return true;
  }

  bool getSymbolRefOperandNum(const MCInst &Inst, unsigned &OpNum) const {
    switch (Inst.getOpcode()) {
    default:
      return false;
    case RISCV::C_J:
      OpNum = 0;
      return true;
    case RISCV::AUIPC:
    case RISCV::JAL:
    case RISCV::C_BEQZ:
    case RISCV::C_BNEZ:
      OpNum = 1;
```

- EN: Declares or implements routines including `llvm_unreachable`, `getSymbolRefOperandNum`. Notable symbols here include `llvm_unreachable`, `getSymbolRefOperandNum`.
- CN: 这里声明或实现函数，例如 `llvm_unreachable`, `getSymbolRefOperandNum`。这里较值得关注的符号包括 `llvm_unreachable`, `getSymbolRefOperandNum`。

### Lines 330-341

```cpp
      return true;
    case RISCV::BEQ:
    case RISCV::BGE:
    case RISCV::BGEU:
    case RISCV::BNE:
    case RISCV::BLT:
    case RISCV::BLTU:
      OpNum = 2;
      return true;
    }
  }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 342-349

```cpp
  const MCSymbol *getTargetSymbol(const MCExpr *Expr) const override {
    auto *RISCVExpr = dyn_cast<MCSpecifierExpr>(Expr);
    if (RISCVExpr && RISCVExpr->getSubExpr())
      return getTargetSymbol(RISCVExpr->getSubExpr());

    return MCPlusBuilder::getTargetSymbol(Expr);
  }
```

- EN: Declares or implements routines including `getTargetSymbol`. Notable symbols here include `getTargetSymbol`.
- CN: 这里声明或实现函数，例如 `getTargetSymbol`。这里较值得关注的符号包括 `getTargetSymbol`。

### Lines 350-358

```cpp
  const MCSymbol *getTargetSymbol(const MCInst &Inst,
                                  unsigned OpNum = 0) const override {
    if (!OpNum && !getSymbolRefOperandNum(Inst, OpNum))
      return nullptr;

    const MCOperand &Op = Inst.getOperand(OpNum);
    if (!Op.isExpr())
      return nullptr;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 359-368

```cpp
    return getTargetSymbol(Op.getExpr());
  }

  bool lowerTailCall(MCInst &Inst) override {
    removeAnnotation(Inst, MCPlus::MCAnnotation::kTailCall);
    if (getConditionalTailCall(Inst))
      unsetConditionalTailCall(Inst);
    return true;
  }
```

- EN: Declares or implements routines including `lowerTailCall`, `removeAnnotation`, `unsetConditionalTailCall`. Notable symbols here include `lowerTailCall`, `removeAnnotation`, `unsetConditionalTailCall`.
- CN: 这里声明或实现函数，例如 `lowerTailCall`, `removeAnnotation`, `unsetConditionalTailCall`。这里较值得关注的符号包括 `lowerTailCall`, `removeAnnotation`, `unsetConditionalTailCall`。

### Lines 369-378

```cpp
  uint64_t analyzePLTEntry(MCInst &Instruction, InstructionIterator Begin,
                           InstructionIterator End,
                           uint64_t BeginPC) const override {
    auto I = Begin;

    assert(I != End);
    auto &AUIPC = *I++;
    assert(AUIPC.getOpcode() == RISCV::AUIPC);
    assert(AUIPC.getOperand(0).getReg() == RISCV::X28);
```

- EN: Declares or implements routines including `assert`. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里较值得关注的符号包括 `assert`。

### Lines 379-391

```cpp
    assert(I != End);
    auto &LD = *I++;
    assert(LD.getOpcode() == RISCV::LD);
    assert(LD.getOperand(0).getReg() == RISCV::X28);
    assert(LD.getOperand(1).getReg() == RISCV::X28);

    assert(I != End);
    auto &JALR = *I++;
    (void)JALR;
    assert(JALR.getOpcode() == RISCV::JALR);
    assert(JALR.getOperand(0).getReg() == RISCV::X6);
    assert(JALR.getOperand(1).getReg() == RISCV::X28);
```

- EN: Declares or implements routines including `assert`. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里较值得关注的符号包括 `assert`。

### Lines 392-403

```cpp
    assert(I != End);
    auto &NOP = *I++;
    (void)NOP;
    assert(isNoop(NOP));

    assert(I == End);

    auto AUIPCOffset = AUIPC.getOperand(1).getImm() << 12;
    auto LDOffset = LD.getOperand(2).getImm();
    return BeginPC + AUIPCOffset + LDOffset;
  }
```

- EN: Declares or implements routines including `assert`. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里较值得关注的符号包括 `assert`。

### Lines 404-416

```cpp
  bool replaceImmWithSymbolRef(MCInst &Inst, const MCSymbol *Symbol,
                               int64_t Addend, MCContext *Ctx, int64_t &Value,
                               uint32_t RelType) const override {
    unsigned ImmOpNo = -1U;

    for (unsigned Index = 0; Index < MCPlus::getNumPrimeOperands(Inst);
         ++Index) {
      if (Inst.getOperand(Index).isImm()) {
        ImmOpNo = Index;
        break;
      }
    }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 417-424

```cpp
    if (ImmOpNo == -1U)
      return false;

    Value = Inst.getOperand(ImmOpNo).getImm();
    setOperandToSymbolRef(Inst, ImmOpNo, Symbol, Addend, Ctx, RelType);
    return true;
  }
```

- EN: Declares or implements routines including `setOperandToSymbolRef`. Notable symbols here include `setOperandToSymbolRef`.
- CN: 这里声明或实现函数，例如 `setOperandToSymbolRef`。这里较值得关注的符号包括 `setOperandToSymbolRef`。

### Lines 425-442

```cpp
  const MCExpr *getTargetExprFor(MCInst &Inst, const MCExpr *Expr,
                                 MCContext &Ctx,
                                 uint32_t RelType) const override {
    switch (RelType) {
    default:
      return Expr;
    case ELF::R_RISCV_GOT_HI20:
    case ELF::R_RISCV_TLS_GOT_HI20:
    case ELF::R_RISCV_TLS_GD_HI20:
      // The GOT is reused so no need to create GOT relocations
    case ELF::R_RISCV_PCREL_HI20:
      return MCSpecifierExpr::create(Expr, RISCV::S_PCREL_HI, Ctx);
    case ELF::R_RISCV_PCREL_LO12_I:
    case ELF::R_RISCV_PCREL_LO12_S:
      return MCSpecifierExpr::create(Expr, RISCV::S_PCREL_LO, Ctx);
    case ELF::R_RISCV_HI20:
      return MCSpecifierExpr::create(Expr, ELF::R_RISCV_HI20, Ctx);
    case ELF::R_RISCV_LO12_I:
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 443-451

```cpp
    case ELF::R_RISCV_LO12_S:
      return MCSpecifierExpr::create(Expr, RISCV::S_LO, Ctx);
    case ELF::R_RISCV_CALL:
      return MCSpecifierExpr::create(Expr, RISCV::S_CALL_PLT, Ctx);
    case ELF::R_RISCV_CALL_PLT:
      return MCSpecifierExpr::create(Expr, RISCV::S_CALL_PLT, Ctx);
    }
  }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 452-461

```cpp
  bool evaluateMemOperandTarget(const MCInst &Inst, uint64_t &Target,
                                uint64_t Address,
                                uint64_t Size) const override {
    return false;
  }

  bool isCallAuipc(const MCInst &Inst) const {
    if (Inst.getOpcode() != RISCV::AUIPC)
      return false;
```

- EN: Declares or implements routines including `isCallAuipc`. Notable symbols here include `isCallAuipc`.
- CN: 这里声明或实现函数，例如 `isCallAuipc`。这里较值得关注的符号包括 `isCallAuipc`。

### Lines 462-469

```cpp
    const auto &ImmOp = Inst.getOperand(1);
    if (!ImmOp.isExpr())
      return false;

    const auto *ImmExpr = ImmOp.getExpr();
    if (!isa<MCSpecifierExpr>(ImmExpr))
      return false;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 470-478

```cpp
    switch (cast<MCSpecifierExpr>(ImmExpr)->getSpecifier()) {
    default:
      return false;
    case RISCV::S_CALL_PLT:
    case ELF::R_RISCV_CALL_PLT:
      return true;
    }
  }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 479-486

```cpp
  bool isRISCVCall(const MCInst &First, const MCInst &Second) const override {
    if (!isCallAuipc(First))
      return false;

    assert(Second.getOpcode() == RISCV::JALR);
    return true;
  }
```

- EN: Declares or implements routines including `isRISCVCall`, `assert`. Notable symbols here include `isRISCVCall`, `assert`.
- CN: 这里声明或实现函数，例如 `isRISCVCall`, `assert`。这里较值得关注的符号包括 `isRISCVCall`, `assert`。

### Lines 487-502

```cpp
  uint16_t getMinFunctionAlignment() const override {
    if (STI->hasFeature(RISCV::FeatureStdExtC) ||
        STI->hasFeature(RISCV::FeatureStdExtZca))
      return 2;
    return 4;
  }

  void createStackPointerIncrement(
      MCInst &Inst, int imm,
      bool NoFlagsClobber = false /*unused for RISCV*/) const override {
    Inst = MCInstBuilder(RISCV::ADDI)
               .addReg(RISCV::X2)
               .addReg(RISCV::X2)
               .addImm(-imm);
  }
```

- EN: Declares or implements routines including `getMinFunctionAlignment`, `hasFeature`, `MCInstBuilder`. Notable symbols here include `getMinFunctionAlignment`, `hasFeature`, `MCInstBuilder`.
- CN: 这里声明或实现函数，例如 `getMinFunctionAlignment`, `hasFeature`, `MCInstBuilder`。这里较值得关注的符号包括 `getMinFunctionAlignment`, `hasFeature`, `MCInstBuilder`。

### Lines 503-511

```cpp
  void createStackPointerDecrement(
      MCInst &Inst, int imm,
      bool NoFlagsClobber = false /*unused for RISCV*/) const override {
    Inst = MCInstBuilder(RISCV::ADDI)
               .addReg(RISCV::X2)
               .addReg(RISCV::X2)
               .addImm(imm);
  }
```

- EN: Declares or implements routines including `MCInstBuilder`. Notable symbols here include `MCInstBuilder`.
- CN: 这里声明或实现函数，例如 `MCInstBuilder`。这里较值得关注的符号包括 `MCInstBuilder`。

### Lines 512-521

```cpp
  void loadReg(MCInst &Inst, MCPhysReg To, MCPhysReg From,
               int64_t offset) const {
    Inst = MCInstBuilder(RISCV::LD).addReg(To).addReg(From).addImm(offset);
  }

  void storeReg(MCInst &Inst, MCPhysReg From, MCPhysReg To,
                int64_t offset) const {
    Inst = MCInstBuilder(RISCV::SD).addReg(From).addReg(To).addImm(offset);
  }
```

- EN: Declares or implements routines including `MCInstBuilder`. Notable symbols here include `MCInstBuilder`.
- CN: 这里声明或实现函数，例如 `MCInstBuilder`。这里较值得关注的符号包括 `MCInstBuilder`。

### Lines 522-534

```cpp
  void spillRegs(InstructionListType &Insts,
                 const SmallVector<unsigned> &Regs) const {
    Insts.emplace_back();
    createStackPointerIncrement(Insts.back(), Regs.size() * 8);

    int64_t Offset = 0;
    for (auto Reg : Regs) {
      Insts.emplace_back();
      storeReg(Insts.back(), Reg, RISCV::X2, Offset);
      Offset += 8;
    }
  }
```

- EN: Declares or implements routines including `createStackPointerIncrement`, `storeReg`. Notable symbols here include `createStackPointerIncrement`, `storeReg`.
- CN: 这里声明或实现函数，例如 `createStackPointerIncrement`, `storeReg`。这里较值得关注的符号包括 `createStackPointerIncrement`, `storeReg`。

### Lines 535-543

```cpp
  void reloadRegs(InstructionListType &Insts,
                  const SmallVector<unsigned> &Regs) const {
    int64_t Offset = 0;
    for (auto Reg : Regs) {
      Insts.emplace_back();
      loadReg(Insts.back(), Reg, RISCV::X2, Offset);
      Offset += 8;
    }
```

- EN: Declares or implements routines including `loadReg`. Notable symbols here include `loadReg`.
- CN: 这里声明或实现函数，例如 `loadReg`。这里较值得关注的符号包括 `loadReg`。

### Lines 544-555

```cpp
    Insts.emplace_back();
    createStackPointerDecrement(Insts.back(), Regs.size() * 8);
  }

  void atomicAdd(MCInst &Inst, MCPhysReg RegAtomic, MCPhysReg RegTo,
                 MCPhysReg RegCnt) const {
    Inst = MCInstBuilder(RISCV::AMOADD_D)
               .addReg(RegAtomic)
               .addReg(RegTo)
               .addReg(RegCnt);
  }
```

- EN: Declares or implements routines including `createStackPointerDecrement`, `MCInstBuilder`. Notable symbols here include `createStackPointerDecrement`, `MCInstBuilder`.
- CN: 这里声明或实现函数，例如 `createStackPointerDecrement`, `MCInstBuilder`。这里较值得关注的符号包括 `createStackPointerDecrement`, `MCInstBuilder`。

### Lines 556-565

```cpp
  InstructionListType createRegCmpJE(MCPhysReg RegNo, const MCSymbol *Target,
                                     MCContext *Ctx) const {
    InstructionListType Insts;
    Insts.emplace_back(MCInstBuilder(RISCV::BEQ)
                           .addReg(RegNo)
                           .addReg(RISCV::X0)
                           .addExpr(MCSymbolRefExpr::create(Target, *Ctx)));
    return Insts;
  }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 566-578

```cpp
  void createTrap(MCInst &Inst) const override {
    Inst.clear();
    Inst.setOpcode(RISCV::EBREAK);
  }

  void createNoop(MCInst &Inst) const override {
    Inst.clear();
    Inst = MCInstBuilder(RISCV::ADDI)
               .addReg(RISCV::X0)
               .addReg(RISCV::X0)
               .addImm(0);
  }
```

- EN: Declares or implements routines including `createTrap`, `createNoop`, `MCInstBuilder`. Notable symbols here include `createTrap`, `createNoop`, `MCInstBuilder`.
- CN: 这里声明或实现函数，例如 `createTrap`, `createNoop`, `MCInstBuilder`。这里较值得关注的符号包括 `createTrap`, `createNoop`, `MCInstBuilder`。

### Lines 579-595

```cpp
  void createShortJmp(InstructionListType &Seq, const MCSymbol *Target,
                      MCContext *Ctx, bool IsTailCall) override {
    // The sequence of instructions we create here is the following:
    //  auipc   a5, hi20(Target)
    //  addi    a5, a5, low12(Target)
    //  jr x5 => jalr x0, x5, 0
    MCPhysReg Reg = RISCV::X5;
    InstructionListType Insts = materializeAddress(Target, Ctx, Reg);
    Insts.emplace_back();
    MCInst &Inst = Insts.back();
    Inst.clear();
    Inst = MCInstBuilder(RISCV::JALR).addReg(RISCV::X0).addReg(Reg).addImm(0);
    if (IsTailCall)
      setTailCall(Inst);
    Seq.swap(Insts);
  }
```

- EN: Declares or implements routines including `materializeAddress`, `MCInstBuilder`, `setTailCall`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `materializeAddress`, `MCInstBuilder`, `setTailCall`.
- CN: 这里声明或实现函数，例如 `materializeAddress`, `MCInstBuilder`, `setTailCall`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `materializeAddress`, `MCInstBuilder`, `setTailCall`。

### Lines 596-605

```cpp
  InstructionListType createGetter(MCContext *Ctx, const char *name) const {
    InstructionListType Insts(4);
    MCSymbol *Locs = Ctx->getOrCreateSymbol(name);
    InstructionListType Addr = materializeAddress(Locs, Ctx, RISCV::X10);
    std::copy(Addr.begin(), Addr.end(), Insts.begin());
    loadReg(Insts[2], RISCV::X10, RISCV::X10, 0);
    createReturn(Insts[3]);
    return Insts;
  }
```

- EN: Declares or implements routines including `createGetter`, `Insts`, `getOrCreateSymbol`, `materializeAddress`, `copy`, and 2 more. Notable symbols here include `createGetter`, `Insts`, `getOrCreateSymbol`, `materializeAddress`, `copy`, `loadReg`.
- CN: 这里声明或实现函数，例如 `createGetter`, `Insts`, `getOrCreateSymbol`, `materializeAddress`, `copy`, and 2 more。这里较值得关注的符号包括 `createGetter`, `Insts`, `getOrCreateSymbol`, `materializeAddress`, `copy`, `loadReg`。

### Lines 606-616

```cpp
  InstructionListType createIncMemory(MCPhysReg RegTo, MCPhysReg RegCnt,
                                      MCPhysReg RegAtomic) const {
    InstructionListType Insts;
    Insts.emplace_back();
    Insts.back() =
        MCInstBuilder(RISCV::ADDI).addReg(RegCnt).addReg(RegAtomic).addImm(1);
    Insts.emplace_back();
    atomicAdd(Insts.back(), RegAtomic, RegTo, RegCnt);
    return Insts;
  }
```

- EN: Declares or implements routines including `MCInstBuilder`, `atomicAdd`. Notable symbols here include `MCInstBuilder`, `atomicAdd`.
- CN: 这里声明或实现函数，例如 `MCInstBuilder`, `atomicAdd`。这里较值得关注的符号包括 `MCInstBuilder`, `atomicAdd`。

### Lines 617-627

```cpp
  InstructionListType materializeAddress(const MCSymbol *Target, MCContext *Ctx,
                                         MCPhysReg RegName,
                                         int64_t Addend = 0) const override {
    // Get the symbol address by auipc + addi
    InstructionListType Insts(2);
    MCSymbol *AuipcLabel = Ctx->createNamedTempSymbol("pcrel_hi");
    Insts[0] = MCInstBuilder(RISCV::AUIPC).addReg(RegName).addImm(0);
    setOperandToSymbolRef(Insts[0], /* OpNum */ 1, Target, Addend, Ctx,
                          ELF::R_RISCV_PCREL_HI20);
    setInstLabel(Insts[0], AuipcLabel);
```

- EN: Declares or implements routines including `Insts`, `createNamedTempSymbol`, `MCInstBuilder`, `setInstLabel`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `Insts`, `createNamedTempSymbol`, `MCInstBuilder`, `setInstLabel`.
- CN: 这里声明或实现函数，例如 `Insts`, `createNamedTempSymbol`, `MCInstBuilder`, `setInstLabel`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `Insts`, `createNamedTempSymbol`, `MCInstBuilder`, `setInstLabel`。

### Lines 628-645

```cpp
    Insts[1] =
        MCInstBuilder(RISCV::ADDI).addReg(RegName).addReg(RegName).addImm(0);
    setOperandToSymbolRef(Insts[1], /* OpNum */ 2, AuipcLabel, Addend, Ctx,
                          ELF::R_RISCV_PCREL_LO12_I);
    return Insts;
  }

  InstructionListType createInstrIncMemory(const MCSymbol *Target,
                                           MCContext *Ctx, bool IsLeaf,
                                           unsigned CodePointerSize) override {
    // We need 2 scratch registers: one for the target address (x10), and one
    // for the increment value (x11).
    // addi sp, sp, -16
    // sd x10, 0(sp)
    // sd x11, 8(sp)
    // la x10, target         # 1: auipc x10, %pcrel_hi(target)
    //                        # addi x10, x10, %pcrel_lo(1b)
    // li x11, 1              # addi x11, zero, 1
```

- EN: Declares or implements routines including `MCInstBuilder`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `MCInstBuilder`.
- CN: 这里声明或实现函数，例如 `MCInstBuilder`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `MCInstBuilder`。

### Lines 646-661

```cpp
    // amoadd.d zero, x10, x11
    // ld x10, 0(sp)
    // ld x11, 8(sp)
    // addi sp, sp, 16

    InstructionListType Insts;
    spillRegs(Insts, {RISCV::X10, RISCV::X11});
    InstructionListType Addr = materializeAddress(Target, Ctx, RISCV::X10);
    Insts.insert(Insts.end(), Addr.begin(), Addr.end());
    InstructionListType IncInsts =
        createIncMemory(RISCV::X10, RISCV::X11, RISCV::X0);
    Insts.insert(Insts.end(), IncInsts.begin(), IncInsts.end());
    reloadRegs(Insts, {RISCV::X10, RISCV::X11});
    return Insts;
  }
```

- EN: Declares or implements routines including `spillRegs`, `materializeAddress`, `createIncMemory`, `reloadRegs`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `spillRegs`, `materializeAddress`, `createIncMemory`, `reloadRegs`.
- CN: 这里声明或实现函数，例如 `spillRegs`, `materializeAddress`, `createIncMemory`, `reloadRegs`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `spillRegs`, `materializeAddress`, `createIncMemory`, `reloadRegs`。

### Lines 662-677

```cpp
  void createDirectCall(MCInst &Inst, const MCSymbol *Target, MCContext *Ctx,
                        bool IsTailCall) override {
    Inst.setOpcode(RISCV::JAL);
    Inst.clear();
    if (IsTailCall) {
      Inst.addOperand(MCOperand::createReg(RISCV::X0));
      Inst.addOperand(MCOperand::createExpr(getTargetExprFor(
          Inst, MCSymbolRefExpr::create(Target, *Ctx), *Ctx, 0)));
      convertJmpToTailCall(Inst);
    } else {
      Inst.addOperand(MCOperand::createReg(RISCV::X1));
      Inst.addOperand(MCOperand::createExpr(getTargetExprFor(
          Inst, MCSymbolRefExpr::create(Target, *Ctx), *Ctx, 0)));
    }
  }
```

- EN: Declares or implements routines including `create`, `convertJmpToTailCall`. Notable symbols here include `create`, `convertJmpToTailCall`.
- CN: 这里声明或实现函数，例如 `create`, `convertJmpToTailCall`。这里较值得关注的符号包括 `create`, `convertJmpToTailCall`。

### Lines 678-693

```cpp
  void createIndirectCallInst(MCInst &Inst, bool IsTailCall, MCPhysReg Reg,
                              int64_t Disp) const {
    Inst.clear();
    Inst.setOpcode(RISCV::JALR);
    Inst.clear();
    if (IsTailCall) {
      Inst.addOperand(MCOperand::createReg(RISCV::X0));
      Inst.addOperand(MCOperand::createReg(Reg));
      Inst.addOperand(MCOperand::createImm(Disp));
    } else {
      Inst.addOperand(MCOperand::createReg(RISCV::X1));
      Inst.addOperand(MCOperand::createReg(Reg));
      Inst.addOperand(MCOperand::createImm(Disp));
    }
  }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 694-711

```cpp
  InstructionListType
  createInstrumentedIndCallHandlerEntryBB(const MCSymbol *InstrTrampoline,
                                          const MCSymbol *IndCallHandler,
                                          MCContext *Ctx) override {
    // Code sequence used to check whether InstrTampoline was initialized
    // and call it if so, returns via IndCallHandler
    //   sp      -16(sp)
    //   sd      x10, 0(sp)
    //   sd      x11, 0(sp)
    //   la      x10, InstrTrampoline -> auipc + addi
    //   ld      x10, [x10]
    //   beq     x10, x11, IndCallHandler
    //   sp      -16(sp)
    //   sd      x1, 0(sp)
    //   jalr    x1,x10,0
    //   ld      x1, [sp], #16
    //   sp      16(sp)
    //   jal     x0, IndCallHandler
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 712-729

```cpp

    InstructionListType Insts;
    spillRegs(Insts, {RISCV::X10, RISCV::X11});
    InstructionListType Addr =
        materializeAddress(InstrTrampoline, Ctx, RISCV::X10);
    Insts.insert(Insts.end(), Addr.begin(), Addr.end());
    Insts.emplace_back();
    loadReg(Insts.back(), RISCV::X10, RISCV::X10, 0);
    InstructionListType cmpJmp =
        createRegCmpJE(RISCV::X10, IndCallHandler, Ctx);
    Insts.insert(Insts.end(), cmpJmp.begin(), cmpJmp.end());
    Insts.emplace_back();
    createStackPointerIncrement(Insts.back(), 16);
    Insts.emplace_back();
    storeReg(Insts.back(), RISCV::X1, RISCV::X2, 0);
    Insts.emplace_back();
    createIndirectCallInst(Insts.back(), /*IsTailCall*/ false, RISCV::X10, 0);
    Insts.emplace_back();
```

- EN: Declares or implements routines including `spillRegs`, `materializeAddress`, `loadReg`, `createRegCmpJE`, `createStackPointerIncrement`, and 2 more. Notable symbols here include `spillRegs`, `materializeAddress`, `loadReg`, `createRegCmpJE`, `createStackPointerIncrement`, `storeReg`.
- CN: 这里声明或实现函数，例如 `spillRegs`, `materializeAddress`, `loadReg`, `createRegCmpJE`, `createStackPointerIncrement`, and 2 more。这里较值得关注的符号包括 `spillRegs`, `materializeAddress`, `loadReg`, `createRegCmpJE`, `createStackPointerIncrement`, `storeReg`。

### Lines 730-737

```cpp
    loadReg(Insts.back(), RISCV::X1, RISCV::X2, 0);
    Insts.emplace_back();
    createStackPointerDecrement(Insts.back(), 16);
    Insts.emplace_back();
    createDirectCall(Insts.back(), IndCallHandler, Ctx, /*IsTailCall*/ true);
    return Insts;
  }
```

- EN: Declares or implements routines including `loadReg`, `createStackPointerDecrement`, `createDirectCall`. Notable symbols here include `loadReg`, `createStackPointerDecrement`, `createDirectCall`.
- CN: 这里声明或实现函数，例如 `loadReg`, `createStackPointerDecrement`, `createDirectCall`。这里较值得关注的符号包括 `loadReg`, `createStackPointerDecrement`, `createDirectCall`。

### Lines 738-750

```cpp
  InstructionListType createInstrumentedIndCallHandlerExitBB() const override {
    InstructionListType Insts;
    reloadRegs(Insts, {RISCV::X10, RISCV::X11});
    Insts.emplace_back();
    loadReg(Insts.back(), RISCV::X5, RISCV::X2, 0);
    Insts.emplace_back();
    createStackPointerDecrement(Insts.back(), 16);
    reloadRegs(Insts, {RISCV::X10, RISCV::X11});
    Insts.emplace_back();
    createIndirectCallInst(Insts.back(), /*IsTailCall*/ true, RISCV::X5, 0);
    return Insts;
  }
```

- EN: Declares or implements routines including `createInstrumentedIndCallHandlerExitBB`, `reloadRegs`, `loadReg`, `createStackPointerDecrement`, `createIndirectCallInst`. Notable symbols here include `createInstrumentedIndCallHandlerExitBB`, `reloadRegs`, `loadReg`, `createStackPointerDecrement`, `createIndirectCallInst`.
- CN: 这里声明或实现函数，例如 `createInstrumentedIndCallHandlerExitBB`, `reloadRegs`, `loadReg`, `createStackPointerDecrement`, `createIndirectCallInst`。这里较值得关注的符号包括 `createInstrumentedIndCallHandlerExitBB`, `reloadRegs`, `loadReg`, `createStackPointerDecrement`, `createIndirectCallInst`。

### Lines 751-762

```cpp
  InstructionListType
  createInstrumentedIndTailCallHandlerExitBB() const override {
    return createInstrumentedIndCallHandlerExitBB();
  }

  std::vector<MCInst> createSymbolTrampoline(const MCSymbol *TgtSym,
                                             MCContext *Ctx) override {
    std::vector<MCInst> Insts;
    createShortJmp(Insts, TgtSym, Ctx, /*IsTailCall*/ true);
    return Insts;
  }
```

- EN: Declares or implements routines including `createInstrumentedIndTailCallHandlerExitBB`, `createShortJmp`. Notable symbols here include `createInstrumentedIndTailCallHandlerExitBB`, `createShortJmp`.
- CN: 这里声明或实现函数，例如 `createInstrumentedIndTailCallHandlerExitBB`, `createShortJmp`。这里较值得关注的符号包括 `createInstrumentedIndTailCallHandlerExitBB`, `createShortJmp`。

### Lines 763-771

```cpp
  InstructionListType createNumCountersGetter(MCContext *Ctx) const override {
    return createGetter(Ctx, "__bolt_num_counters");
  }

  InstructionListType
  createInstrLocationsGetter(MCContext *Ctx) const override {
    return createGetter(Ctx, "__bolt_instr_locations");
  }
```

- EN: Declares or implements routines including `createNumCountersGetter`, `createInstrLocationsGetter`. Notable symbols here include `createNumCountersGetter`, `createInstrLocationsGetter`.
- CN: 这里声明或实现函数，例如 `createNumCountersGetter`, `createInstrLocationsGetter`。这里较值得关注的符号包括 `createNumCountersGetter`, `createInstrLocationsGetter`。

### Lines 772-779

```cpp
  InstructionListType createInstrTablesGetter(MCContext *Ctx) const override {
    return createGetter(Ctx, "__bolt_instr_tables");
  }

  InstructionListType createInstrNumFuncsGetter(MCContext *Ctx) const override {
    return createGetter(Ctx, "__bolt_instr_num_funcs");
  }
```

- EN: Declares or implements routines including `createInstrTablesGetter`, `createInstrNumFuncsGetter`. Notable symbols here include `createInstrTablesGetter`, `createInstrNumFuncsGetter`.
- CN: 这里声明或实现函数，例如 `createInstrTablesGetter`, `createInstrNumFuncsGetter`。这里较值得关注的符号包括 `createInstrTablesGetter`, `createInstrNumFuncsGetter`。

### Lines 780-788

```cpp
  void convertIndirectCallToLoad(MCInst &Inst, MCPhysReg Reg) override {
    bool IsTailCall = isTailCall(Inst);
    if (IsTailCall)
      removeAnnotation(Inst, MCPlus::MCAnnotation::kTailCall);
    Inst.setOpcode(RISCV::ADD);
    Inst.insert(Inst.begin(), MCOperand::createReg(Reg));
    Inst.insert(Inst.begin() + 1, MCOperand::createReg(RISCV::X0));
  }
```

- EN: Declares or implements routines including `convertIndirectCallToLoad`, `isTailCall`, `removeAnnotation`. Notable symbols here include `convertIndirectCallToLoad`, `isTailCall`, `removeAnnotation`.
- CN: 这里声明或实现函数，例如 `convertIndirectCallToLoad`, `isTailCall`, `removeAnnotation`。这里较值得关注的符号包括 `convertIndirectCallToLoad`, `isTailCall`, `removeAnnotation`。

### Lines 789-806

```cpp
  InstructionListType createLoadImmediate(const MCPhysReg Dest,
                                          uint64_t Imm) const override {
    InstructionListType Insts;
    // get IMM higher 32bit
    Insts.emplace_back(
        MCInstBuilder(RISCV::LUI).addReg(Dest).addImm((Imm >> 44) & 0xFFFFF));
    Insts.emplace_back(MCInstBuilder(RISCV::LUI)
                           .addReg(RISCV::X5)
                           .addImm((Imm >> 32) & 0xFFF));
    Insts.emplace_back(MCInstBuilder(RISCV::SRLI)
                           .addReg(RISCV::X5)
                           .addReg(RISCV::X5)
                           .addImm(12));
    Insts.emplace_back(
        MCInstBuilder(RISCV::OR).addReg(Dest).addReg(Dest).addReg(RISCV::X5));
    Insts.emplace_back(
        MCInstBuilder(RISCV::SLLI).addReg(Dest).addReg(Dest).addImm(32));
```

- EN: Declares or implements routines including `MCInstBuilder`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `MCInstBuilder`.
- CN: 这里声明或实现函数，例如 `MCInstBuilder`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `MCInstBuilder`。

### Lines 807-820

```cpp
    // get IMM lower 32bit
    Insts.emplace_back(MCInstBuilder(RISCV::LUI)
                           .addReg(RISCV::X5)
                           .addImm((Imm >> 12) & 0xFFFFF));
    Insts.emplace_back(
        MCInstBuilder(RISCV::LUI).addReg(RISCV::X6).addImm((Imm)&0xFFF));
    Insts.emplace_back(MCInstBuilder(RISCV::SRLI)
                           .addReg(RISCV::X6)
                           .addReg(RISCV::X6)
                           .addImm(12));
    Insts.emplace_back(
        MCInstBuilder(RISCV::OR).addReg(RISCV::X5).addReg(RISCV::X5).addReg(
            RISCV::X6));
```

- EN: Declares or implements routines including `MCInstBuilder`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `MCInstBuilder`.
- CN: 这里声明或实现函数，例如 `MCInstBuilder`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `MCInstBuilder`。

### Lines 821-838

```cpp
    // get 64bit IMM
    Insts.emplace_back(
        MCInstBuilder(RISCV::OR).addReg(Dest).addReg(Dest).addReg(RISCV::X5));
    return Insts;
  }

  InstructionListType createInstrumentedIndirectCall(MCInst &&CallInst,
                                                     MCSymbol *HandlerFuncAddr,
                                                     size_t CallSiteID,
                                                     MCContext *Ctx) override {
    // Code sequence used to enter indirect call instrumentation helper:
    //   addi  sp, sp, -0x10
    //   sd  a0, 0x0(sp)
    //   sd  a1, 0x8(sp)
    //   mov target x0  convertIndirectCallToLoad -> add a0, zero, target
    //   mov x1 CallSiteID createLoadImmediate
    //   addi  sp, sp, -0x10
    //   sd  a0, 0x0(sp)
```

- EN: Declares or implements routines including `MCInstBuilder`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `MCInstBuilder`.
- CN: 这里声明或实现函数，例如 `MCInstBuilder`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `MCInstBuilder`。

### Lines 839-855

```cpp
    //   sd  a1, 0x8(sp)
    //   la x0 *HandlerFuncAddr -> auipc + addi
    //   jalr x0

    InstructionListType Insts;
    spillRegs(Insts, {RISCV::X10, RISCV::X11});
    Insts.emplace_back(CallInst);
    convertIndirectCallToLoad(Insts.back(), RISCV::X10);
    InstructionListType LoadImm = createLoadImmediate(RISCV::X11, CallSiteID);
    Insts.insert(Insts.end(), LoadImm.begin(), LoadImm.end());
    spillRegs(Insts, {RISCV::X10, RISCV::X11});
    InstructionListType Addr =
        materializeAddress(HandlerFuncAddr, Ctx, RISCV::X5);
    Insts.insert(Insts.end(), Addr.begin(), Addr.end());
    Insts.emplace_back();
    createIndirectCallInst(Insts.back(), isTailCall(CallInst), RISCV::X5, 0);
```

- EN: Declares or implements routines including `spillRegs`, `convertIndirectCallToLoad`, `createLoadImmediate`, `materializeAddress`, `createIndirectCallInst`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `spillRegs`, `convertIndirectCallToLoad`, `createLoadImmediate`, `materializeAddress`, `createIndirectCallInst`.
- CN: 这里声明或实现函数，例如 `spillRegs`, `convertIndirectCallToLoad`, `createLoadImmediate`, `materializeAddress`, `createIndirectCallInst`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `spillRegs`, `convertIndirectCallToLoad`, `createLoadImmediate`, `materializeAddress`, `createIndirectCallInst`。

### Lines 856-863

```cpp
    // // Carry over metadata including tail call marker if present.
    stripAnnotations(Insts.back());
    moveAnnotations(std::move(CallInst), Insts.back());

    return Insts;
  }
};
```

- EN: Declares or implements routines including `stripAnnotations`, `moveAnnotations`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `stripAnnotations`, `moveAnnotations`.
- CN: 这里声明或实现函数，例如 `stripAnnotations`, `moveAnnotations`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `stripAnnotations`, `moveAnnotations`。

### Lines 864-875

```cpp
} // end anonymous namespace

namespace llvm {
namespace bolt {

MCPlusBuilder *createRISCVMCPlusBuilder(const MCInstrAnalysis *Analysis,
                                        const MCInstrInfo *Info,
                                        const MCRegisterInfo *RegInfo,
                                        const MCSubtargetInfo *STI) {
  return new RISCVMCPlusBuilder(Analysis, Info, RegInfo, STI);
}
```

- EN: Works inside namespace scope `namespace`, `bolt` to organize symbols. Contains local control flow that updates state or selects among execution branches. Notable symbols here include `namespace`, `bolt`.
- CN: 这里位于命名空间 `namespace`, `bolt` 中，用于组织符号作用域。这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里较值得关注的符号包括 `namespace`, `bolt`。

### Lines 876-877

```cpp
} // namespace bolt
} // namespace llvm
```

- EN: Works inside namespace scope `bolt`, `llvm` to organize symbols. Notable symbols here include `bolt`, `llvm`.
- CN: 这里位于命名空间 `bolt`, `llvm` 中，用于组织符号作用域。这里较值得关注的符号包括 `bolt`, `llvm`。

## Key Concepts / 关键概念

- `RISCVMCPlusBuilder`: class or struct interface / 类或结构体接口
- `getCalleeSavedRegs`: function or method entry point / 函数或方法入口
- `getAliases`: function or method entry point / 函数或方法入口
- `shouldRecordCodeRelocation`: function or method entry point / 函数或方法入口
- `llvm_unreachable`: function or method entry point / 函数或方法入口
- `isNop`: function or method entry point / 函数或方法入口
- `DEBUG_TYPE`: macro or compile-time switch / 宏或编译期开关
- `llvm`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Core/MCPlusBuilder.h`
- LLVM headers / LLVM 头文件: `llvm/BinaryFormat/ELF.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCInst.h`, `llvm/MC/MCInstBuilder.h`, `llvm/MC/MCSubtargetInfo.h`, `llvm/Support/ErrorHandling.h`
- System headers / 系统头文件: `MCTargetDesc/RISCVMCAsmInfo.h`, `MCTargetDesc/RISCVMCTargetDesc.h`
- Directory context / 目录上下文: `bolt/lib/Target/RISCV` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Target/RISCV` 下的相邻文件通常与本文件协作组成对应子系统
