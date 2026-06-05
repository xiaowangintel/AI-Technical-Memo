# AArch64MCPlusBuilder.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Target/AArch64/AArch64MCPlusBuilder.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: bolt/Target/AArch64/AArch64MCPlusBuilder.cpp This file provides AArch64-specific MCPlus builder.. It also sits in code that implements target-specific BOLT behavior. / 该文件实现目标架构相关的 BOLT 行为。 源码头部说明其职责是：bolt/Target/AArch64/AArch64MCPlusBuilder.cpp This file provides AArch64-specific MCPlus builder.。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12

```cpp
//===- bolt/Target/AArch64/AArch64MCPlusBuilder.cpp -----------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file provides AArch64-specific MCPlus builder.
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 13-30

```cpp
#include "AArch64ExpandImm.h"
#include "AArch64InstrInfo.h"
#include "AArch64MCSymbolizer.h"
#include "MCTargetDesc/AArch64AddressingModes.h"
#include "MCTargetDesc/AArch64FixupKinds.h"
#include "MCTargetDesc/AArch64MCAsmInfo.h"
#include "MCTargetDesc/AArch64MCTargetDesc.h"
#include "Utils/AArch64BaseInfo.h"
#include "bolt/Core/BinaryBasicBlock.h"
#include "bolt/Core/BinaryFunction.h"
#include "bolt/Core/MCInstUtils.h"
#include "bolt/Core/MCPlusBuilder.h"
#include "llvm/BinaryFormat/ELF.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCDisassembler/MCDisassembler.h"
#include "llvm/MC/MCInstBuilder.h"
#include "llvm/MC/MCInstrInfo.h"
#include "llvm/MC/MCRegister.h"
```

- EN: Pulls in 18 header(s) from local project, LLVM, system dependencies needed by this range. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 18 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里的注释记录了设计假设、不变量或使用说明。

### Lines 31-38

```cpp
#include "llvm/MC/MCRegisterInfo.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/DataExtractor.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/ErrorHandling.h"

#define DEBUG_TYPE "mcplus"
```

- EN: Pulls in 5 header(s) from LLVM dependencies needed by this range. Establishes header-guard or prologue macros so the file can be compiled safely. Defines macros such as `DEBUG_TYPE` for constants or compile-time switches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 5 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里定义宏 `DEBUG_TYPE`，用于常量或编译期开关。这里的注释记录了设计假设、不变量或使用说明。

### Lines 39-50

```cpp
using namespace llvm;
using namespace bolt;

namespace opts {
extern cl::OptionCategory BoltInstrCategory;
static cl::opt<bool> NoLSEAtomics(
    "no-lse-atomics",
    cl::desc("generate instrumentation code sequence without using LSE atomic "
             "instruction"),
    cl::init(false), cl::Optional, cl::cat(BoltInstrCategory));
} // namespace opts
```

- EN: Works inside namespace scope `llvm`, `bolt`, `opts` to organize symbols. Declares or implements routines including `init`. Notable symbols here include `init`, `llvm`, `bolt`, `opts`.
- CN: 这里位于命名空间 `llvm`, `bolt`, `opts` 中，用于组织符号作用域。这里声明或实现函数，例如 `init`。这里较值得关注的符号包括 `init`, `llvm`, `bolt`, `opts`。

### Lines 51-59

```cpp
namespace {

[[maybe_unused]] static void getSystemFlag(MCInst &Inst, MCPhysReg RegName) {
  Inst.setOpcode(AArch64::MRS);
  Inst.clear();
  Inst.addOperand(MCOperand::createReg(RegName));
  Inst.addOperand(MCOperand::createImm(AArch64SysReg::NZCV));
}
```

- EN: Declares or implements routines including `getSystemFlag`. Notable symbols here include `getSystemFlag`.
- CN: 这里声明或实现函数，例如 `getSystemFlag`。这里较值得关注的符号包括 `getSystemFlag`。

### Lines 60-77

```cpp
[[maybe_unused]] static void setSystemFlag(MCInst &Inst, MCPhysReg RegName) {
  Inst.setOpcode(AArch64::MSR);
  Inst.clear();
  Inst.addOperand(MCOperand::createImm(AArch64SysReg::NZCV));
  Inst.addOperand(MCOperand::createReg(RegName));
}

static void createPushRegisters(MCInst &Inst, MCPhysReg Reg1, MCPhysReg Reg2) {
  Inst.clear();
  unsigned NewOpcode = AArch64::STPXpre;
  Inst.setOpcode(NewOpcode);
  Inst.addOperand(MCOperand::createReg(AArch64::SP));
  Inst.addOperand(MCOperand::createReg(Reg1));
  Inst.addOperand(MCOperand::createReg(Reg2));
  Inst.addOperand(MCOperand::createReg(AArch64::SP));
  Inst.addOperand(MCOperand::createImm(-2));
}
```

- EN: Declares or implements routines including `setSystemFlag`, `createPushRegisters`. Notable symbols here include `setSystemFlag`, `createPushRegisters`.
- CN: 这里声明或实现函数，例如 `setSystemFlag`, `createPushRegisters`。这里较值得关注的符号包括 `setSystemFlag`, `createPushRegisters`。

### Lines 78-88

```cpp
static void createPopRegisters(MCInst &Inst, MCPhysReg Reg1, MCPhysReg Reg2) {
  Inst.clear();
  unsigned NewOpcode = AArch64::LDPXpost;
  Inst.setOpcode(NewOpcode);
  Inst.addOperand(MCOperand::createReg(AArch64::SP));
  Inst.addOperand(MCOperand::createReg(Reg1));
  Inst.addOperand(MCOperand::createReg(Reg2));
  Inst.addOperand(MCOperand::createReg(AArch64::SP));
  Inst.addOperand(MCOperand::createImm(2));
}
```

- EN: Declares or implements routines including `createPopRegisters`. Notable symbols here include `createPopRegisters`.
- CN: 这里声明或实现函数，例如 `createPopRegisters`。这里较值得关注的符号包括 `createPopRegisters`。

### Lines 89-104

```cpp
static void loadReg(MCInst &Inst, MCPhysReg To, MCPhysReg From) {
  Inst.setOpcode(AArch64::LDRXui);
  Inst.clear();
  if (From == AArch64::SP) {
    Inst.setOpcode(AArch64::LDRXpost);
    Inst.addOperand(MCOperand::createReg(From));
    Inst.addOperand(MCOperand::createReg(To));
    Inst.addOperand(MCOperand::createReg(From));
    Inst.addOperand(MCOperand::createImm(16));
  } else {
    Inst.addOperand(MCOperand::createReg(To));
    Inst.addOperand(MCOperand::createReg(From));
    Inst.addOperand(MCOperand::createImm(0));
  }
}
```

- EN: Declares or implements routines including `loadReg`. Notable symbols here include `loadReg`.
- CN: 这里声明或实现函数，例如 `loadReg`。这里较值得关注的符号包括 `loadReg`。

### Lines 105-120

```cpp
static void storeReg(MCInst &Inst, MCPhysReg From, MCPhysReg To) {
  Inst.setOpcode(AArch64::STRXui);
  Inst.clear();
  if (To == AArch64::SP) {
    Inst.setOpcode(AArch64::STRXpre);
    Inst.addOperand(MCOperand::createReg(To));
    Inst.addOperand(MCOperand::createReg(From));
    Inst.addOperand(MCOperand::createReg(To));
    Inst.addOperand(MCOperand::createImm(-16));
  } else {
    Inst.addOperand(MCOperand::createReg(From));
    Inst.addOperand(MCOperand::createReg(To));
    Inst.addOperand(MCOperand::createImm(0));
  }
}
```

- EN: Declares or implements routines including `storeReg`. Notable symbols here include `storeReg`.
- CN: 这里声明或实现函数，例如 `storeReg`。这里较值得关注的符号包括 `storeReg`。

### Lines 121-129

```cpp
static void atomicAdd(MCInst &Inst, MCPhysReg RegTo, MCPhysReg RegCnt) {
  assert(!opts::NoLSEAtomics && "Supports only ARM with LSE extension");
  Inst.setOpcode(AArch64::LDADDX);
  Inst.clear();
  Inst.addOperand(MCOperand::createReg(AArch64::XZR));
  Inst.addOperand(MCOperand::createReg(RegCnt));
  Inst.addOperand(MCOperand::createReg(RegTo));
}
```

- EN: Declares or implements routines including `atomicAdd`, `assert`. Notable symbols here include `atomicAdd`, `assert`.
- CN: 这里声明或实现函数，例如 `atomicAdd`, `assert`。这里较值得关注的符号包括 `atomicAdd`, `assert`。

### Lines 130-138

```cpp
static void createMovz(MCInst &Inst, MCPhysReg Reg, uint64_t Imm) {
  assert(Imm <= UINT16_MAX && "Invalid Imm size");
  Inst.clear();
  Inst.setOpcode(AArch64::MOVZXi);
  Inst.addOperand(MCOperand::createReg(Reg));
  Inst.addOperand(MCOperand::createImm(Imm & 0xFFFF));
  Inst.addOperand(MCOperand::createImm(0));
}
```

- EN: Declares or implements routines including `createMovz`, `assert`. Notable symbols here include `createMovz`, `assert`.
- CN: 这里声明或实现函数，例如 `createMovz`, `assert`。这里较值得关注的符号包括 `createMovz`, `assert`。

### Lines 139-147

```cpp
static InstructionListType createIncMemory(MCPhysReg RegTo, MCPhysReg RegTmp) {
  InstructionListType Insts;
  Insts.emplace_back();
  createMovz(Insts.back(), RegTmp, 1);
  Insts.emplace_back();
  atomicAdd(Insts.back(), RegTo, RegTmp);
  return Insts;
}
```

- EN: Declares or implements routines including `createIncMemory`, `createMovz`, `atomicAdd`. Notable symbols here include `createIncMemory`, `createMovz`, `atomicAdd`.
- CN: 这里声明或实现函数，例如 `createIncMemory`, `createMovz`, `atomicAdd`。这里较值得关注的符号包括 `createIncMemory`, `createMovz`, `atomicAdd`。

### Lines 148-165

```cpp
static InstructionListType createMOVImm(MCPhysReg DstReg, unsigned BitSize,
                                        uint64_t Imm) {
  SmallVector<AArch64_IMM::ImmInsnModel> Insn;
  AArch64_IMM::expandMOVImm(Imm, BitSize, Insn);
  assert(Insn.size() != 0);

  InstructionListType Insts;
  for (auto I = Insn.begin(), E = Insn.end(); I != E; ++I) {
    switch (I->Opcode) {
    case AArch64::ORRWri:
    case AArch64::ORRXri:
    case AArch64::ANDXri:
    case AArch64::EORXri:
      if (I->Op1 == 0)
        Insts.emplace_back(
            MCInstBuilder(I->Opcode)
                .addReg(DstReg)
                .addReg(BitSize == 32 ? AArch64::WZR : AArch64::XZR)
```

- EN: Declares or implements routines including `expandMOVImm`, `assert`, `MCInstBuilder`. Notable symbols here include `expandMOVImm`, `assert`, `MCInstBuilder`.
- CN: 这里声明或实现函数，例如 `expandMOVImm`, `assert`, `MCInstBuilder`。这里较值得关注的符号包括 `expandMOVImm`, `assert`, `MCInstBuilder`。

### Lines 166-183

```cpp
                .addImm(I->Op2));
      else
        Insts.emplace_back(
            MCInstBuilder(I->Opcode).addReg(DstReg).addReg(DstReg).addImm(
                I->Op2));
      break;
    case AArch64::EORXrs:
    case AArch64::EONXrs:
    case AArch64::ORRWrs:
    case AArch64::ORRXrs:
      Insts.emplace_back(MCInstBuilder(I->Opcode)
                             .addReg(DstReg)
                             .addReg(DstReg)
                             .addReg(DstReg)
                             .addImm(I->Op2));
      break;
    case AArch64::MOVNWi:
    case AArch64::MOVNXi:
```

- EN: Declares or implements routines including `MCInstBuilder`. Notable symbols here include `MCInstBuilder`.
- CN: 这里声明或实现函数，例如 `MCInstBuilder`。这里较值得关注的符号包括 `MCInstBuilder`。

### Lines 184-201

```cpp
    case AArch64::MOVZWi:
    case AArch64::MOVZXi:
      Insts.emplace_back(
          MCInstBuilder(I->Opcode).addReg(DstReg).addImm(I->Op1).addImm(
              I->Op2));
      break;
    case AArch64::MOVKWi:
    case AArch64::MOVKXi:
      Insts.emplace_back(MCInstBuilder(I->Opcode)
                             .addReg(DstReg)
                             .addReg(DstReg)
                             .addImm(I->Op1)
                             .addImm(I->Op2));
      break;
    default:
      llvm_unreachable("Unhandled! Please refer to expandMOVImm in llvm");
    }
  }
```

- EN: Declares or implements routines including `MCInstBuilder`, `llvm_unreachable`. Notable symbols here include `MCInstBuilder`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `MCInstBuilder`, `llvm_unreachable`。这里较值得关注的符号包括 `MCInstBuilder`, `llvm_unreachable`。

### Lines 202-210

```cpp
  return Insts;
}

class AArch64MCPlusBuilder : public MCPlusBuilder {
public:
  using MCPlusBuilder::MCPlusBuilder;

  BinaryFunction *InstrCounterIncrFunc{nullptr};
```

- EN: Introduces type definitions such as `AArch64MCPlusBuilder`. Contains local control flow that updates state or selects among execution branches. Notable symbols here include `AArch64MCPlusBuilder`.
- CN: 这里引入类型定义，例如 `AArch64MCPlusBuilder`。这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里较值得关注的符号包括 `AArch64MCPlusBuilder`。

### Lines 211-220

```cpp
  std::unique_ptr<MCSymbolizer>
  createTargetSymbolizer(BinaryFunction &Function,
                         bool CreateNewSymbols) const override {
    return std::make_unique<AArch64MCSymbolizer>(Function, CreateNewSymbols);
  }

  MCPhysReg getStackPointer() const override { return AArch64::SP; }
  MCPhysReg getFramePointer() const override { return AArch64::FP; }
  MCPhysReg getFlagsReg() const override { return AArch64::NZCV; }
```

- EN: Declares or implements routines including `getStackPointer`, `getFramePointer`, `getFlagsReg`. Notable symbols here include `getStackPointer`, `getFramePointer`, `getFlagsReg`.
- CN: 这里声明或实现函数，例如 `getStackPointer`, `getFramePointer`, `getFlagsReg`。这里较值得关注的符号包括 `getStackPointer`, `getFramePointer`, `getFlagsReg`。

### Lines 221-228

```cpp
  bool isBreakpoint(const MCInst &Inst) const override {
    return Inst.getOpcode() == AArch64::BRK;
  }

  bool isPush(const MCInst &Inst) const override {
    return isStoreToStack(Inst);
  }
```

- EN: Declares or implements routines including `isBreakpoint`, `isPush`. Notable symbols here include `isBreakpoint`, `isPush`.
- CN: 这里声明或实现函数，例如 `isBreakpoint`, `isPush`。这里较值得关注的符号包括 `isBreakpoint`, `isPush`。

### Lines 229-246

```cpp
  bool isPop(const MCInst &Inst) const override {
    return isLoadFromStack(Inst);
  }

  // We look for instruction that saves LR to or restores LR from stack.
  //
  // If we ever see an LR save to stack, we assume this block is not an
  // epilogue.
  //
  // If there is no LR save in the block and we see an LR restore from
  // stack, we assume it is an epilogue.
  //
  // If neither is seen, we assume it is not an epilogue.
  //
  // This is not meant to accurately recognize epilogue in all possible
  // cases, but to have BOLT be conservative on treating basic block as
  // epilogue and then turning indirect branch with unknown control flow
  // to tail call.
```

- EN: Declares or implements routines including `isPop`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isPop`.
- CN: 这里声明或实现函数，例如 `isPop`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isPop`。

### Lines 247-259

```cpp
  bool isEpilogue(const BinaryBasicBlock &BB) const override {
    if (BB.succ_size())
      return false;

    bool SeenLRRestoreFromStack = false;
    for (auto It = BB.rbegin(); It != BB.rend(); ++It) {
      const MCInst &Instr = *It;
      // Skip CFI pseudo instruction.
      if (isCFI(Instr))
        continue;
      if (isReturn(Instr))
        return true;
```

- EN: Declares or implements routines including `isEpilogue`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isEpilogue`.
- CN: 这里声明或实现函数，例如 `isEpilogue`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isEpilogue`。

### Lines 260-276

```cpp
      if (isStoreToStack(Instr)) {
        for (const MCOperand &Operand : useOperands(Instr)) {
          if (Operand.isReg() && Operand.getReg() == AArch64::LR) {
            return false;
          }
        }
      } else if (isLoadFromStack(Instr)) {
        for (const MCOperand &Operand : defOperands(Instr)) {
          if (Operand.isReg() && Operand.getReg() == AArch64::LR) {
            SeenLRRestoreFromStack = true;
          }
        }
      }
    }
    return SeenLRRestoreFromStack;
  }
```

- EN: Declares or implements routines including `if`. Notable symbols here include `if`.
- CN: 这里声明或实现函数，例如 `if`。这里较值得关注的符号包括 `if`。

### Lines 277-294

```cpp
  void createCall(MCInst &Inst, const MCSymbol *Target,
                  MCContext *Ctx) override {
    createDirectCall(Inst, Target, Ctx, false);
  }

  bool convertTailCallToCall(MCInst &Inst) override {
    int NewOpcode;
    switch (Inst.getOpcode()) {
    default:
      return false;
    case AArch64::B:
      NewOpcode = AArch64::BL;
      break;
    case AArch64::BR:
      NewOpcode = AArch64::BLR;
      break;
    }
```

- EN: Declares or implements routines including `createDirectCall`, `convertTailCallToCall`. Notable symbols here include `createDirectCall`, `convertTailCallToCall`.
- CN: 这里声明或实现函数，例如 `createDirectCall`, `convertTailCallToCall`。这里较值得关注的符号包括 `createDirectCall`, `convertTailCallToCall`。

### Lines 295-305

```cpp
    Inst.setOpcode(NewOpcode);
    removeAnnotation(Inst, MCPlus::MCAnnotation::kTailCall);
    clearOffset(Inst);
    return true;
  }

  bool equals(const MCSpecifierExpr &A, const MCSpecifierExpr &B,
              CompFuncTy Comp) const override {
    if (A.getSpecifier() != B.getSpecifier())
      return false;
```

- EN: Declares or implements routines including `removeAnnotation`, `clearOffset`. Notable symbols here include `removeAnnotation`, `clearOffset`.
- CN: 这里声明或实现函数，例如 `removeAnnotation`, `clearOffset`。这里较值得关注的符号包括 `removeAnnotation`, `clearOffset`。

### Lines 306-316

```cpp
    return MCPlusBuilder::equals(*A.getSubExpr(), *B.getSubExpr(), Comp);
  }

  bool shortenInstruction(MCInst &, const MCSubtargetInfo &) const override {
    return false;
  }

  SmallVector<MCPhysReg> getTrustedLiveInRegs() const override {
    return {AArch64::LR};
  }
```

- EN: Declares or implements routines including `shortenInstruction`, `getTrustedLiveInRegs`. Notable symbols here include `shortenInstruction`, `getTrustedLiveInRegs`.
- CN: 这里声明或实现函数，例如 `shortenInstruction`, `getTrustedLiveInRegs`。这里较值得关注的符号包括 `shortenInstruction`, `getTrustedLiveInRegs`。

### Lines 317-334

```cpp
  std::optional<MCPhysReg>
  getWrittenAuthenticatedReg(const MCInst &Inst,
                             bool &IsChecked) const override {
    IsChecked = false;
    switch (Inst.getOpcode()) {
    case AArch64::AUTIAZ:
    case AArch64::AUTIBZ:
    case AArch64::AUTIASP:
    case AArch64::AUTIBSP:
    case AArch64::AUTIASPPCi:
    case AArch64::AUTIBSPPCi:
    case AArch64::AUTIASPPCr:
    case AArch64::AUTIBSPPCr:
      return AArch64::LR;
    case AArch64::AUTIA1716:
    case AArch64::AUTIB1716:
    case AArch64::AUTIA171615:
    case AArch64::AUTIB171615:
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 335-352

```cpp
      return AArch64::X17;
    case AArch64::AUTIA:
    case AArch64::AUTIB:
    case AArch64::AUTDA:
    case AArch64::AUTDB:
    case AArch64::AUTIZA:
    case AArch64::AUTIZB:
    case AArch64::AUTDZA:
    case AArch64::AUTDZB:
      return Inst.getOperand(0).getReg();
    case AArch64::LDRAAwriteback:
    case AArch64::LDRABwriteback:
      // Note that LDRA(A|B)indexed are not listed here, as they do not write
      // an authenticated pointer back to the register.
      IsChecked = true;
      return Inst.getOperand(2).getReg();
    default:
      return std::nullopt;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 353-360

```cpp
    }
  }

  bool isPSignOnLR(const MCInst &Inst) const override {
    std::optional<MCPhysReg> SignReg = getSignedReg(Inst);
    return SignReg && *SignReg == AArch64::LR;
  }
```

- EN: Declares or implements routines including `isPSignOnLR`, `getSignedReg`. Notable symbols here include `isPSignOnLR`, `getSignedReg`.
- CN: 这里声明或实现函数，例如 `isPSignOnLR`, `getSignedReg`。这里较值得关注的符号包括 `isPSignOnLR`, `getSignedReg`。

### Lines 361-368

```cpp
  bool isPAuthOnLR(const MCInst &Inst) const override {
    // LDR(A|B) should not be covered.
    bool IsChecked;
    std::optional<MCPhysReg> AuthReg =
        getWrittenAuthenticatedReg(Inst, IsChecked);
    return !IsChecked && AuthReg && *AuthReg == AArch64::LR;
  }
```

- EN: Declares or implements routines including `isPAuthOnLR`, `getWrittenAuthenticatedReg`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isPAuthOnLR`, `getWrittenAuthenticatedReg`.
- CN: 这里声明或实现函数，例如 `isPAuthOnLR`, `getWrittenAuthenticatedReg`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isPAuthOnLR`, `getWrittenAuthenticatedReg`。

### Lines 369-377

```cpp
  bool isPAuthAndRet(const MCInst &Inst) const override {
    return Inst.getOpcode() == AArch64::RETAA ||
           Inst.getOpcode() == AArch64::RETAB ||
           Inst.getOpcode() == AArch64::RETAASPPCi ||
           Inst.getOpcode() == AArch64::RETABSPPCi ||
           Inst.getOpcode() == AArch64::RETAASPPCr ||
           Inst.getOpcode() == AArch64::RETABSPPCr;
  }
```

- EN: Declares or implements routines including `isPAuthAndRet`. Notable symbols here include `isPAuthAndRet`.
- CN: 这里声明或实现函数，例如 `isPAuthAndRet`。这里较值得关注的符号包括 `isPAuthAndRet`。

### Lines 378-395

```cpp
  void createMatchingAuth(const MCInst &AuthAndRet, MCInst &Auth) override {
    Auth.clear();
    Auth.setOperands(AuthAndRet.getOperands());
    switch (AuthAndRet.getOpcode()) {
    case AArch64::RETAA:
      Auth.setOpcode(AArch64::AUTIASP);
      break;
    case AArch64::RETAB:
      Auth.setOpcode(AArch64::AUTIBSP);
      break;
    case AArch64::RETAASPPCi:
      Auth.setOpcode(AArch64::AUTIASPPCi);
      break;
    case AArch64::RETABSPPCi:
      Auth.setOpcode(AArch64::AUTIBSPPCi);
      break;
    case AArch64::RETAASPPCr:
      Auth.setOpcode(AArch64::AUTIASPPCr);
```

- EN: Declares or implements routines including `createMatchingAuth`. Notable symbols here include `createMatchingAuth`.
- CN: 这里声明或实现函数，例如 `createMatchingAuth`。这里较值得关注的符号包括 `createMatchingAuth`。

### Lines 396-404

```cpp
      break;
    case AArch64::RETABSPPCr:
      Auth.setOpcode(AArch64::AUTIBSPPCr);
      break;
    default:
      llvm_unreachable("Unhandled fused pauth-and-return instruction");
    }
  }
```

- EN: Declares or implements routines including `llvm_unreachable`. Notable symbols here include `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `llvm_unreachable`。这里较值得关注的符号包括 `llvm_unreachable`。

### Lines 405-422

```cpp
  std::optional<MCPhysReg> getSignedReg(const MCInst &Inst) const override {
    switch (Inst.getOpcode()) {
    case AArch64::PACIA:
    case AArch64::PACIB:
    case AArch64::PACDA:
    case AArch64::PACDB:
    case AArch64::PACIZA:
    case AArch64::PACIZB:
    case AArch64::PACDZA:
    case AArch64::PACDZB:
      return Inst.getOperand(0).getReg();
    case AArch64::PACIAZ:
    case AArch64::PACIBZ:
    case AArch64::PACIASP:
    case AArch64::PACIBSP:
    case AArch64::PACIASPPC:
    case AArch64::PACIBSPPC:
    case AArch64::PACNBIASPPC:
```

- EN: Declares or implements routines including `getSignedReg`. Notable symbols here include `getSignedReg`.
- CN: 这里声明或实现函数，例如 `getSignedReg`。这里较值得关注的符号包括 `getSignedReg`。

### Lines 423-434

```cpp
    case AArch64::PACNBIBSPPC:
      return AArch64::LR;
    case AArch64::PACIA1716:
    case AArch64::PACIB1716:
    case AArch64::PACIA171615:
    case AArch64::PACIB171615:
      return AArch64::X17;
    default:
      return std::nullopt;
    }
  }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 435-443

```cpp
  std::optional<MCPhysReg>
  getRegUsedAsRetDest(const MCInst &Inst,
                      bool &IsAuthenticatedInternally) const override {
    assert(isReturn(Inst));
    switch (Inst.getOpcode()) {
    case AArch64::RET:
      IsAuthenticatedInternally = false;
      return Inst.getOperand(0).getReg();
```

- EN: Declares or implements routines including `assert`. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里较值得关注的符号包括 `assert`。

### Lines 444-461

```cpp
    case AArch64::RETAA:
    case AArch64::RETAB:
    case AArch64::RETAASPPCi:
    case AArch64::RETABSPPCi:
    case AArch64::RETAASPPCr:
    case AArch64::RETABSPPCr:
      IsAuthenticatedInternally = true;
      return AArch64::LR;
    case AArch64::ERET:
    case AArch64::ERETAA:
    case AArch64::ERETAB:
      // The ERET* instructions use either register ELR_EL1, ELR_EL2 or
      // ELR_EL3, depending on the current Exception Level at run-time.
      //
      // Furthermore, these registers are not modelled by LLVM as a regular
      // MCPhysReg, so there is no way to indicate that through the current API.
      return std::nullopt;
    default:
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 462-469

```cpp
      llvm_unreachable("Unhandled return instruction");
    }
  }

  MCPhysReg getRegUsedAsIndirectBranchDest(
      const MCInst &Inst, bool &IsAuthenticatedInternally) const override {
    assert(isIndirectCall(Inst) || isIndirectBranch(Inst));
```

- EN: Declares or implements routines including `llvm_unreachable`, `assert`. Notable symbols here include `llvm_unreachable`, `assert`.
- CN: 这里声明或实现函数，例如 `llvm_unreachable`, `assert`。这里较值得关注的符号包括 `llvm_unreachable`, `assert`。

### Lines 470-487

```cpp
    switch (Inst.getOpcode()) {
    case AArch64::BR:
    case AArch64::BLR:
      IsAuthenticatedInternally = false;
      return Inst.getOperand(0).getReg();
    case AArch64::BRAA:
    case AArch64::BRAB:
    case AArch64::BRAAZ:
    case AArch64::BRABZ:
    case AArch64::BLRAA:
    case AArch64::BLRAB:
    case AArch64::BLRAAZ:
    case AArch64::BLRABZ:
      IsAuthenticatedInternally = true;
      return Inst.getOperand(0).getReg();
    default:
      llvm_unreachable("Unhandled indirect branch or call");
    }
```

- EN: Declares or implements routines including `llvm_unreachable`. Notable symbols here include `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `llvm_unreachable`。这里较值得关注的符号包括 `llvm_unreachable`。

### Lines 488-504

```cpp
  }

  std::optional<MCPhysReg>
  getMaterializedAddressRegForPtrAuth(const MCInst &Inst) const override {
    switch (Inst.getOpcode()) {
    case AArch64::ADR:
    case AArch64::ADRP:
      // These instructions produce an address value based on the information
      // encoded into the instruction itself (which should reside in a read-only
      // code memory) and the value of PC register (that is, the location of
      // this instruction), so the produced value is not attacker-controlled.
      return Inst.getOperand(0).getReg();
    default:
      return std::nullopt;
    }
  }
```

- EN: Declares or implements routines including `getMaterializedAddressRegForPtrAuth`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getMaterializedAddressRegForPtrAuth`.
- CN: 这里声明或实现函数，例如 `getMaterializedAddressRegForPtrAuth`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getMaterializedAddressRegForPtrAuth`。

### Lines 505-521

```cpp
  std::optional<std::pair<MCPhysReg, MCPhysReg>>
  analyzeAddressArithmeticsForPtrAuth(const MCInst &Inst) const override {
    switch (Inst.getOpcode()) {
    default:
      return std::nullopt;
    case AArch64::ADDXri:
    case AArch64::SUBXri:
      // The immediate addend is encoded into the instruction itself, so it is
      // not attacker-controlled under Pointer Authentication threat model.
      return std::make_pair(Inst.getOperand(0).getReg(),
                            Inst.getOperand(1).getReg());
    case AArch64::ORRXrs:
      // "mov Xd, Xm" is equivalent to "orr Xd, XZR, Xm, lsl #0"
      if (Inst.getOperand(1).getReg() != AArch64::XZR ||
          Inst.getOperand(3).getImm() != 0)
        return std::nullopt;
```

- EN: Declares or implements routines including `analyzeAddressArithmeticsForPtrAuth`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `analyzeAddressArithmeticsForPtrAuth`.
- CN: 这里声明或实现函数，例如 `analyzeAddressArithmeticsForPtrAuth`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `analyzeAddressArithmeticsForPtrAuth`。

### Lines 522-539

```cpp
      return std::make_pair(Inst.getOperand(0).getReg(),
                            Inst.getOperand(2).getReg());
    }
  }

  std::optional<std::pair<MCPhysReg, MCInst *>>
  getAuthCheckedReg(BinaryBasicBlock &BB) const override {
    // Match several possible hard-coded sequences of instructions which can be
    // emitted by LLVM backend to check that the authenticated pointer is
    // correct (see AArch64AsmPrinter::emitPtrauthCheckAuthenticatedValue).
    //
    // This function only matches sequences involving branch instructions.
    // All these sequences have the form:
    //
    // (0) ... regular code that authenticates a pointer in Xn ...
    // (1) analyze Xn
    // (2) branch to .Lon_success if the pointer is correct
    // (3) BRK #imm (fall-through basic block)
```

- EN: Declares or implements routines including `getAuthCheckedReg`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getAuthCheckedReg`.
- CN: 这里声明或实现函数，例如 `getAuthCheckedReg`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getAuthCheckedReg`。

### Lines 540-554

```cpp
    //
    // In the above pseudocode, (1) + (2) is one of the following sequences:
    //
    // - eor Xtmp, Xn, Xn, lsl #1
    //   tbz Xtmp, #62, .Lon_success
    //
    // - mov Xtmp, Xn
    //   xpac(i|d) Xn (or xpaclri if Xn is LR)
    //   cmp Xtmp, Xn
    //   b.eq .Lon_success
    //
    // Note that any branch destination operand is accepted as .Lon_success -
    // it is the responsibility of the caller of getAuthCheckedReg to inspect
    // the list of successors of this basic block as appropriate.
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 555-563

```cpp
    // Any of the above code sequences assume the fall-through basic block
    // is a dead-end trap instruction.
    const BinaryBasicBlock *BreakBB = BB.getFallthrough();
    if (!BreakBB || BreakBB->empty() || !isTrap(BreakBB->front()))
      return std::nullopt;

    // Iterate over the instructions of BB in reverse order, matching opcodes
    // and operands.
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 564-577

```cpp
    auto It = BB.end();
    auto StepBack = [&]() {
      while (It != BB.begin()) {
        --It;
        // Skip any CFI instructions, but no other pseudos are expected here.
        if (!isCFI(*It))
          return true;
      }
      return false;
    };
    // Step to the last non-CFI instruction.
    if (!StepBack())
      return std::nullopt;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 578-586

```cpp
    using namespace llvm::bolt::LowLevelInstMatcherDSL;
    Reg TestedReg;
    Reg ScratchReg;

    if (matchInst(*It, AArch64::Bcc, Imm(AArch64CC::EQ) /*, .Lon_success*/)) {
      if (!StepBack() || !matchInst(*It, AArch64::SUBSXrs, Reg(AArch64::XZR),
                                    TestedReg, ScratchReg, Imm(0)))
        return std::nullopt;
```

- EN: Works inside namespace scope `llvm` to organize symbols. Declares or implements routines including `Imm`. Notable symbols here include `Imm`, `llvm`.
- CN: 这里位于命名空间 `llvm` 中，用于组织符号作用域。这里声明或实现函数，例如 `Imm`。这里较值得关注的符号包括 `Imm`, `llvm`。

### Lines 587-599

```cpp
      // Either XPAC(I|D) ScratchReg, ScratchReg
      // or     XPACLRI
      if (!StepBack())
        return std::nullopt;
      if (matchInst(*It, AArch64::XPACLRI)) {
        // No operands to check, but using XPACLRI forces TestedReg to be X30.
        if (TestedReg.get() != AArch64::LR)
          return std::nullopt;
      } else if (!matchInst(*It, AArch64::XPACI, ScratchReg, ScratchReg) &&
                 !matchInst(*It, AArch64::XPACD, ScratchReg, ScratchReg)) {
        return std::nullopt;
      }
```

- EN: Declares or implements routines including `if`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `if`.
- CN: 这里声明或实现函数，例如 `if`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `if`。

### Lines 600-611

```cpp
      if (!StepBack() || !matchInst(*It, AArch64::ORRXrs, ScratchReg,
                                    Reg(AArch64::XZR), TestedReg, Imm(0)))
        return std::nullopt;

      return std::make_pair(TestedReg.get(), &*It);
    }

    if (matchInst(*It, AArch64::TBZX, ScratchReg, Imm(62) /*, .Lon_success*/)) {
      if (!StepBack() || !matchInst(*It, AArch64::EORXrs, ScratchReg, TestedReg,
                                    TestedReg, Imm(1)))
        return std::nullopt;
```

- EN: Declares or implements routines including `Reg`, `Imm`. Notable symbols here include `Reg`, `Imm`.
- CN: 这里声明或实现函数，例如 `Reg`, `Imm`。这里较值得关注的符号包括 `Reg`, `Imm`。

### Lines 612-623

```cpp
      return std::make_pair(TestedReg.get(), &*It);
    }

    return std::nullopt;
  }

  std::optional<MCPhysReg> getAuthCheckedReg(const MCInst &Inst,
                                             bool MayOverwrite) const override {
    // Cannot trivially reuse AArch64InstrInfo::getMemOperandWithOffsetWidth()
    // method as it accepts an instance of MachineInstr, not MCInst.
    const MCInstrDesc &Desc = Info->get(Inst.getOpcode());
```

- EN: Declares or implements routines including `get`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `get`.
- CN: 这里声明或实现函数，例如 `get`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `get`。

### Lines 624-641

```cpp
    // If signing oracles are considered, the particular value left in the base
    // register after this instruction is important. This function checks that
    // if the base register was overwritten, it is due to address write-back:
    //
    //     ; good:
    //     autdza  x1           ; x1 is authenticated (may fail)
    //     ldr     x0, [x1, #8] ; x1 is checked and not changed
    //     pacdzb  x1
    //
    //     ; also good:
    //     autdza  x1
    //     ldr     x0, [x1, #8]! ; x1 is checked and incremented by 8
    //     pacdzb  x1
    //
    //     ; bad (the value being signed is not the authenticated one):
    //     autdza  x1
    //     ldr     x1, [x1, #8]  ; x1 is overwritten with an unrelated value
    //     pacdzb  x1
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 642-656

```cpp
    //
    //     ; also bad:
    //     autdza  x1
    //     pacdzb  x1  ; possibly signing the result of failed authentication
    //
    // Note that this function is not needed for authentication oracles, as the
    // particular value left in the register after a successful memory access
    // is not important.
    auto ClobbersBaseRegExceptWriteback = [&](unsigned BaseRegUseIndex) {
      // FIXME: Compute the indices of address operands (base reg and written-
      //        back result) in AArch64InstrInfo instead of this ad-hoc code.
      MCPhysReg BaseReg = Inst.getOperand(BaseRegUseIndex).getReg();
      unsigned WrittenBackDefIndex = Desc.getOperandConstraint(
          BaseRegUseIndex, MCOI::OperandConstraint::TIED_TO);
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 657-666

```cpp
      for (unsigned DefIndex = 0; DefIndex < Desc.getNumDefs(); ++DefIndex) {
        // Address write-back is permitted:
        //
        //    autda x0, x2
        //    ; x0 is authenticated
        //    ldr   x1, [x0, #8]!
        //    ; x0 is trusted (as authenticated and checked)
        if (DefIndex == WrittenBackDefIndex)
          continue;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 667-676

```cpp
        // Any other overwriting is not permitted:
        //
        //    autda x0, x2
        //    ; x0 is authenticated
        //    ldr   w0, [x0]
        //    ; x0 is not authenticated anymore
        if (RegInfo->regsOverlap(Inst.getOperand(DefIndex).getReg(), BaseReg))
          return true;
      }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 677-687

```cpp
      return false;
    };

    // FIXME: Not all load instructions are handled by this->mayLoad(Inst).
    //        On the other hand, MCInstrDesc::mayLoad() is permitted to return
    //        true for non-load instructions (such as AArch64::HINT) which
    //        would result in false negatives.
    if (mayLoad(Inst)) {
      // The first Use operand is the base address register.
      unsigned BaseRegIndex = Desc.getNumDefs();
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 688-696

```cpp
      // Reject non-immediate offsets, as adding a 64-bit register can change
      // the resulting address arbitrarily.
      for (unsigned I = BaseRegIndex + 1, E = Desc.getNumOperands(); I < E; ++I)
        if (Inst.getOperand(I).isReg())
          return std::nullopt;

      if (!MayOverwrite && ClobbersBaseRegExceptWriteback(BaseRegIndex))
        return std::nullopt;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 697-705

```cpp
      return Inst.getOperand(BaseRegIndex).getReg();
    }

    // Store instructions are not handled yet, as they are not important for
    // pauthtest ABI. Though, they could be handled similar to loads, if needed.

    return std::nullopt;
  }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 706-713

```cpp
  bool isADRP(const MCInst &Inst) const override {
    return Inst.getOpcode() == AArch64::ADRP;
  }

  bool isADR(const MCInst &Inst) const override {
    return Inst.getOpcode() == AArch64::ADR;
  }
```

- EN: Declares or implements routines including `isADRP`, `isADR`. Notable symbols here include `isADRP`, `isADR`.
- CN: 这里声明或实现函数，例如 `isADRP`, `isADR`。这里较值得关注的符号包括 `isADRP`, `isADR`。

### Lines 714-723

```cpp
  bool isAddXri(const MCInst &Inst) const override {
    return Inst.getOpcode() == AArch64::ADDXri;
  }

  bool isLoadLiteralGPR(const MCInst &Inst) const override {
    unsigned OpCode = Inst.getOpcode();
    return OpCode == AArch64::LDRWl || OpCode == AArch64::LDRXl ||
           OpCode == AArch64::LDRSWl;
  }
```

- EN: Declares or implements routines including `isAddXri`, `isLoadLiteralGPR`. Notable symbols here include `isAddXri`, `isLoadLiteralGPR`.
- CN: 这里声明或实现函数，例如 `isAddXri`, `isLoadLiteralGPR`。这里较值得关注的符号包括 `isAddXri`, `isLoadLiteralGPR`。

### Lines 724-732

```cpp
  MCPhysReg getADRReg(const MCInst &Inst) const {
    assert((isADR(Inst) || isADRP(Inst)) && "Not an ADR instruction");
    assert(MCPlus::getNumPrimeOperands(Inst) != 0 &&
           "No operands for ADR instruction");
    assert(Inst.getOperand(0).isReg() &&
           "Unexpected operand in ADR instruction");
    return Inst.getOperand(0).getReg();
  }
```

- EN: Declares or implements routines including `getADRReg`, `assert`. Notable symbols here include `getADRReg`, `assert`.
- CN: 这里声明或实现函数，例如 `getADRReg`, `assert`。这里较值得关注的符号包括 `getADRReg`, `assert`。

### Lines 733-742

```cpp
  InstructionListType undoAdrpAddRelaxation(const MCInst &ADRInst,
                                            MCContext *Ctx) const override {
    assert(isADR(ADRInst) && "ADR instruction expected");

    const MCPhysReg Reg = getADRReg(ADRInst);
    const MCSymbol *Target = getTargetSymbol(ADRInst);
    const uint64_t Addend = getTargetAddend(ADRInst);
    return materializeAddress(Target, Ctx, Reg, Addend);
  }
```

- EN: Declares or implements routines including `assert`, `getADRReg`, `getTargetSymbol`, `getTargetAddend`. Notable symbols here include `assert`, `getADRReg`, `getTargetSymbol`, `getTargetAddend`.
- CN: 这里声明或实现函数，例如 `assert`, `getADRReg`, `getTargetSymbol`, `getTargetAddend`。这里较值得关注的符号包括 `assert`, `getADRReg`, `getTargetSymbol`, `getTargetAddend`。

### Lines 743-760

```cpp
  InstructionListType createAdrpLdr(const MCInst &LDRInst,
                                    MCContext *Ctx) const override {
    assert(isLoadLiteralGPR(LDRInst) &&
           "LDR (literal) or LDRSW (literal) expected");
    assert(LDRInst.getOperand(0).isReg() &&
           "unexpected operand in LDR instruction");
    const MCPhysReg DataReg = LDRInst.getOperand(0).getReg();
    MCPhysReg AddrReg;
    unsigned OpCode;
    uint32_t RelType;
    switch (LDRInst.getOpcode()) {
    case AArch64::LDRWl:
      AddrReg = (MCPhysReg)RegInfo->getMatchingSuperReg(
          DataReg, AArch64::sub_32,
          &RegInfo->getRegClass(AArch64::GPR64RegClassID));
      OpCode = AArch64::LDRWui;
      RelType = ELF::R_AARCH64_LDST32_ABS_LO12_NC;
      break;
```

- EN: Declares or implements routines including `assert`, `getRegClass`. Notable symbols here include `assert`, `getRegClass`.
- CN: 这里声明或实现函数，例如 `assert`, `getRegClass`。这里较值得关注的符号包括 `assert`, `getRegClass`。

### Lines 761-774

```cpp
    case AArch64::LDRXl:
      AddrReg = DataReg;
      OpCode = AArch64::LDRXui;
      RelType = ELF::R_AARCH64_LDST64_ABS_LO12_NC;
      break;
    case AArch64::LDRSWl:
      AddrReg = DataReg;
      OpCode = AArch64::LDRSWui;
      RelType = ELF::R_AARCH64_LDST64_ABS_LO12_NC;
      break;
    default:
      llvm_unreachable("LDR (literal) or LDRSW (literal) expected");
    }
```

- EN: Declares or implements routines including `llvm_unreachable`. Notable symbols here include `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `llvm_unreachable`。这里较值得关注的符号包括 `llvm_unreachable`。

### Lines 775-792

```cpp
    const MCSymbol *Target = getTargetSymbol(LDRInst, 1);
    assert(Target && "missing target symbol in LDR instruction");

    InstructionListType Insts(2);
    Insts[0].setOpcode(AArch64::ADRP);
    Insts[0].clear();
    Insts[0].addOperand(MCOperand::createReg(AddrReg));
    Insts[0].addOperand(MCOperand::createImm(0));
    setOperandToSymbolRef(Insts[0], /* OpNum */ 1, Target, 0, Ctx,
                          ELF::R_AARCH64_NONE);
    Insts[1].setOpcode(OpCode);
    Insts[1].clear();
    Insts[1].addOperand(MCOperand::createReg(DataReg));
    Insts[1].addOperand(MCOperand::createReg(AddrReg));
    Insts[1].addOperand(MCOperand::createImm(0));
    Insts[1].addOperand(MCOperand::createImm(0));
    setOperandToSymbolRef(Insts[1], /* OpNum */ 2, Target, 0, Ctx, RelType);
    return Insts;
```

- EN: Declares or implements routines including `getTargetSymbol`, `assert`, `Insts`, `setOperandToSymbolRef`. Notable symbols here include `getTargetSymbol`, `assert`, `Insts`, `setOperandToSymbolRef`.
- CN: 这里声明或实现函数，例如 `getTargetSymbol`, `assert`, `Insts`, `setOperandToSymbolRef`。这里较值得关注的符号包括 `getTargetSymbol`, `assert`, `Insts`, `setOperandToSymbolRef`。

### Lines 793-810

```cpp
  }

  bool isCompAndBranch(const MCInst &Inst) const {
    const unsigned Opcode = Inst.getOpcode();
    switch (Opcode) {
    // Compare register with immediate and branch.
    case AArch64::CBGTWri:
    case AArch64::CBGTXri:
    case AArch64::CBLTWri:
    case AArch64::CBLTXri:
    case AArch64::CBHIWri:
    case AArch64::CBHIXri:
    case AArch64::CBLOWri:
    case AArch64::CBLOXri:
    case AArch64::CBEQWri:
    case AArch64::CBEQXri:
    case AArch64::CBNEWri:
    case AArch64::CBNEXri:
```

- EN: Declares or implements routines including `isCompAndBranch`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isCompAndBranch`.
- CN: 这里声明或实现函数，例如 `isCompAndBranch`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isCompAndBranch`。

### Lines 811-828

```cpp
    // Compare registers and branch.
    case AArch64::CBGTWrr:
    case AArch64::CBGTXrr:
    case AArch64::CBGEWrr:
    case AArch64::CBGEXrr:
    case AArch64::CBHIWrr:
    case AArch64::CBHIXrr:
    case AArch64::CBHSWrr:
    case AArch64::CBHSXrr:
    case AArch64::CBEQWrr:
    case AArch64::CBEQXrr:
    case AArch64::CBNEWrr:
    case AArch64::CBNEXrr:
    // Compare bytes and branch.
    case AArch64::CBBGTWrr:
    case AArch64::CBBGEWrr:
    case AArch64::CBBHIWrr:
    case AArch64::CBBHSWrr:
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 829-843

```cpp
    case AArch64::CBBEQWrr:
    case AArch64::CBBNEWrr:
    // Compare halfwords and branch.
    case AArch64::CBHGTWrr:
    case AArch64::CBHGEWrr:
    case AArch64::CBHHIWrr:
    case AArch64::CBHHSWrr:
    case AArch64::CBHEQWrr:
    case AArch64::CBHNEWrr:
      return true;
    default:
      return false;
    }
  }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 844-857

```cpp
  bool isTB(const MCInst &Inst) const {
    return (Inst.getOpcode() == AArch64::TBNZW ||
            Inst.getOpcode() == AArch64::TBNZX ||
            Inst.getOpcode() == AArch64::TBZW ||
            Inst.getOpcode() == AArch64::TBZX);
  }

  bool isCB(const MCInst &Inst) const {
    return (Inst.getOpcode() == AArch64::CBNZW ||
            Inst.getOpcode() == AArch64::CBNZX ||
            Inst.getOpcode() == AArch64::CBZW ||
            Inst.getOpcode() == AArch64::CBZX);
  }
```

- EN: Declares or implements routines including `isTB`, `isCB`. Notable symbols here include `isTB`, `isCB`.
- CN: 这里声明或实现函数，例如 `isTB`, `isCB`。这里较值得关注的符号包括 `isTB`, `isCB`。

### Lines 858-866

```cpp
  bool isMOVW(const MCInst &Inst) const override {
    return (Inst.getOpcode() == AArch64::MOVKWi ||
            Inst.getOpcode() == AArch64::MOVKXi ||
            Inst.getOpcode() == AArch64::MOVNWi ||
            Inst.getOpcode() == AArch64::MOVNXi ||
            Inst.getOpcode() == AArch64::MOVZXi ||
            Inst.getOpcode() == AArch64::MOVZWi);
  }
```

- EN: Declares or implements routines including `isMOVW`. Notable symbols here include `isMOVW`.
- CN: 这里声明或实现函数，例如 `isMOVW`。这里较值得关注的符号包括 `isMOVW`。

### Lines 867-884

```cpp
  bool isADD(const MCInst &Inst) const {
    return (Inst.getOpcode() == AArch64::ADDSWri ||
            Inst.getOpcode() == AArch64::ADDSWrr ||
            Inst.getOpcode() == AArch64::ADDSWrs ||
            Inst.getOpcode() == AArch64::ADDSWrx ||
            Inst.getOpcode() == AArch64::ADDSXri ||
            Inst.getOpcode() == AArch64::ADDSXrr ||
            Inst.getOpcode() == AArch64::ADDSXrs ||
            Inst.getOpcode() == AArch64::ADDSXrx ||
            Inst.getOpcode() == AArch64::ADDSXrx64 ||
            Inst.getOpcode() == AArch64::ADDWri ||
            Inst.getOpcode() == AArch64::ADDWrr ||
            Inst.getOpcode() == AArch64::ADDWrs ||
            Inst.getOpcode() == AArch64::ADDWrx ||
            Inst.getOpcode() == AArch64::ADDXri ||
            Inst.getOpcode() == AArch64::ADDXrr ||
            Inst.getOpcode() == AArch64::ADDXrs ||
            Inst.getOpcode() == AArch64::ADDXrx ||
```

- EN: Declares or implements routines including `isADD`. Notable symbols here include `isADD`.
- CN: 这里声明或实现函数，例如 `isADD`。这里较值得关注的符号包括 `isADD`。

### Lines 885-902

```cpp
            Inst.getOpcode() == AArch64::ADDXrx64);
  }

  bool isLDRB(const MCInst &Inst) const {
    const unsigned opcode = Inst.getOpcode();
    switch (opcode) {
    case AArch64::LDRBpost:
    case AArch64::LDRBBpost:
    case AArch64::LDRBBpre:
    case AArch64::LDRBBroW:
    case AArch64::LDRBroW:
    case AArch64::LDRBroX:
    case AArch64::LDRBBroX:
    case AArch64::LDRBBui:
    case AArch64::LDRBui:
    case AArch64::LDRBpre:
    case AArch64::LDRSBWpost:
    case AArch64::LDRSBWpre:
```

- EN: Declares or implements routines including `isLDRB`. Notable symbols here include `isLDRB`.
- CN: 这里声明或实现函数，例如 `isLDRB`。这里较值得关注的符号包括 `isLDRB`。

### Lines 903-920

```cpp
    case AArch64::LDRSBWroW:
    case AArch64::LDRSBWroX:
    case AArch64::LDRSBWui:
    case AArch64::LDRSBXpost:
    case AArch64::LDRSBXpre:
    case AArch64::LDRSBXroW:
    case AArch64::LDRSBXroX:
    case AArch64::LDRSBXui:
    case AArch64::LDURBi:
    case AArch64::LDURBBi:
    case AArch64::LDURSBWi:
    case AArch64::LDURSBXi:
    case AArch64::LDTRBi:
    case AArch64::LDTRSBWi:
    case AArch64::LDTRSBXi:
      return true;
    default:
      break;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 921-938

```cpp
    }

    return false;
  }

  bool isLDRH(const MCInst &Inst) const {
    const unsigned opcode = Inst.getOpcode();
    switch (opcode) {
    case AArch64::LDRHpost:
    case AArch64::LDRHHpost:
    case AArch64::LDRHHpre:
    case AArch64::LDRHroW:
    case AArch64::LDRHHroW:
    case AArch64::LDRHroX:
    case AArch64::LDRHHroX:
    case AArch64::LDRHHui:
    case AArch64::LDRHui:
    case AArch64::LDRHpre:
```

- EN: Declares or implements routines including `isLDRH`. Notable symbols here include `isLDRH`.
- CN: 这里声明或实现函数，例如 `isLDRH`。这里较值得关注的符号包括 `isLDRH`。

### Lines 939-956

```cpp
    case AArch64::LDRSHWpost:
    case AArch64::LDRSHWpre:
    case AArch64::LDRSHWroW:
    case AArch64::LDRSHWroX:
    case AArch64::LDRSHWui:
    case AArch64::LDRSHXpost:
    case AArch64::LDRSHXpre:
    case AArch64::LDRSHXroW:
    case AArch64::LDRSHXroX:
    case AArch64::LDRSHXui:
    case AArch64::LDURHi:
    case AArch64::LDURHHi:
    case AArch64::LDURSHWi:
    case AArch64::LDURSHXi:
    case AArch64::LDTRHi:
    case AArch64::LDTRSHWi:
    case AArch64::LDTRSHXi:
      return true;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 957-974

```cpp
    default:
      break;
    }

    return false;
  }

  bool isLDRW(const MCInst &Inst) const {
    const unsigned opcode = Inst.getOpcode();
    switch (opcode) {
    case AArch64::LDRWpost:
    case AArch64::LDRWpre:
    case AArch64::LDRWroW:
    case AArch64::LDRWroX:
    case AArch64::LDRWui:
    case AArch64::LDRWl:
    case AArch64::LDRSWl:
    case AArch64::LDURWi:
```

- EN: Declares or implements routines including `isLDRW`. Notable symbols here include `isLDRW`.
- CN: 这里声明或实现函数，例如 `isLDRW`。这里较值得关注的符号包括 `isLDRW`。

### Lines 975-992

```cpp
    case AArch64::LDRSWpost:
    case AArch64::LDRSWpre:
    case AArch64::LDRSWroW:
    case AArch64::LDRSWroX:
    case AArch64::LDRSWui:
    case AArch64::LDURSWi:
    case AArch64::LDTRWi:
    case AArch64::LDTRSWi:
    case AArch64::LDPWi:
    case AArch64::LDPWpost:
    case AArch64::LDPWpre:
    case AArch64::LDPSWi:
    case AArch64::LDPSWpost:
    case AArch64::LDPSWpre:
    case AArch64::LDNPWi:
      return true;
    default:
      break;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 993-1010

```cpp
    }

    return false;
  }

  bool isLDRX(const MCInst &Inst) const {
    const unsigned opcode = Inst.getOpcode();
    switch (opcode) {
    case AArch64::LDRXpost:
    case AArch64::LDRXpre:
    case AArch64::LDRXroW:
    case AArch64::LDRXroX:
    case AArch64::LDRXui:
    case AArch64::LDRXl:
    case AArch64::LDURXi:
    case AArch64::LDTRXi:
    case AArch64::LDNPXi:
    case AArch64::LDPXi:
```

- EN: Declares or implements routines including `isLDRX`. Notable symbols here include `isLDRX`.
- CN: 这里声明或实现函数，例如 `isLDRX`。这里较值得关注的符号包括 `isLDRX`。

### Lines 1011-1020

```cpp
    case AArch64::LDPXpost:
    case AArch64::LDPXpre:
      return true;
    default:
      break;
    }

    return false;
  }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 1021-1038

```cpp
  bool isLDRS(const MCInst &Inst) const {
    const unsigned opcode = Inst.getOpcode();
    switch (opcode) {
    case AArch64::LDRSl:
    case AArch64::LDRSui:
    case AArch64::LDRSroW:
    case AArch64::LDRSroX:
    case AArch64::LDURSi:
    case AArch64::LDPSi:
    case AArch64::LDNPSi:
    case AArch64::LDRSpre:
    case AArch64::LDRSpost:
    case AArch64::LDPSpost:
    case AArch64::LDPSpre:
      return true;
    default:
      break;
    }
```

- EN: Declares or implements routines including `isLDRS`. Notable symbols here include `isLDRS`.
- CN: 这里声明或实现函数，例如 `isLDRS`。这里较值得关注的符号包括 `isLDRS`。

### Lines 1039-1056

```cpp

    return false;
  }

  bool isLDRD(const MCInst &Inst) const {
    const unsigned opcode = Inst.getOpcode();
    switch (opcode) {
    case AArch64::LDRDl:
    case AArch64::LDRDui:
    case AArch64::LDRDpre:
    case AArch64::LDRDpost:
    case AArch64::LDRDroW:
    case AArch64::LDRDroX:
    case AArch64::LDURDi:
    case AArch64::LDPDi:
    case AArch64::LDNPDi:
    case AArch64::LDPDpost:
    case AArch64::LDPDpre:
```

- EN: Declares or implements routines including `isLDRD`. Notable symbols here include `isLDRD`.
- CN: 这里声明或实现函数，例如 `isLDRD`。这里较值得关注的符号包括 `isLDRD`。

### Lines 1057-1064

```cpp
      return true;
    default:
      break;
    }

    return false;
  }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 1065-1082

```cpp
  bool isLDRQ(const MCInst &Inst) const {
    const unsigned opcode = Inst.getOpcode();
    switch (opcode) {
    case AArch64::LDRQui:
    case AArch64::LDRQl:
    case AArch64::LDRQpre:
    case AArch64::LDRQpost:
    case AArch64::LDRQroW:
    case AArch64::LDRQroX:
    case AArch64::LDURQi:
    case AArch64::LDPQi:
    case AArch64::LDNPQi:
    case AArch64::LDPQpost:
    case AArch64::LDPQpre:
      return true;
    default:
      break;
    }
```

- EN: Declares or implements routines including `isLDRQ`. Notable symbols here include `isLDRQ`.
- CN: 这里声明或实现函数，例如 `isLDRQ`。这里较值得关注的符号包括 `isLDRQ`。

### Lines 1083-1098

```cpp

    return false;
  }

  bool isBRA(const MCInst &Inst) const {
    switch (Inst.getOpcode()) {
    case AArch64::BRAA:
    case AArch64::BRAB:
    case AArch64::BRAAZ:
    case AArch64::BRABZ:
      return true;
    default:
      return false;
    }
  }
```

- EN: Declares or implements routines including `isBRA`. Notable symbols here include `isBRA`.
- CN: 这里声明或实现函数，例如 `isBRA`。这里较值得关注的符号包括 `isBRA`。

### Lines 1099-1116

```cpp
  bool mayLoad(const MCInst &Inst) const override {
    // FIXME: Probably this could be tablegen-erated not to miss any existing
    //        or future opcodes.
    return isLDRB(Inst) || isLDRH(Inst) || isLDRW(Inst) || isLDRX(Inst) ||
           isLDRQ(Inst) || isLDRD(Inst) || isLDRS(Inst);
  }

  bool isAArch64ExclusiveLoad(const MCInst &Inst) const override {
    return (Inst.getOpcode() == AArch64::LDXPX ||
            Inst.getOpcode() == AArch64::LDXPW ||
            Inst.getOpcode() == AArch64::LDXRX ||
            Inst.getOpcode() == AArch64::LDXRW ||
            Inst.getOpcode() == AArch64::LDXRH ||
            Inst.getOpcode() == AArch64::LDXRB ||
            Inst.getOpcode() == AArch64::LDAXPX ||
            Inst.getOpcode() == AArch64::LDAXPW ||
            Inst.getOpcode() == AArch64::LDAXRX ||
            Inst.getOpcode() == AArch64::LDAXRW ||
```

- EN: Declares or implements routines including `mayLoad`, `isLDRQ`, `isAArch64ExclusiveLoad`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `mayLoad`, `isLDRQ`, `isAArch64ExclusiveLoad`.
- CN: 这里声明或实现函数，例如 `mayLoad`, `isLDRQ`, `isAArch64ExclusiveLoad`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `mayLoad`, `isLDRQ`, `isAArch64ExclusiveLoad`。

### Lines 1117-1134

```cpp
            Inst.getOpcode() == AArch64::LDAXRH ||
            Inst.getOpcode() == AArch64::LDAXRB);
  }

  bool isAArch64ExclusiveStore(const MCInst &Inst) const override {
    return (Inst.getOpcode() == AArch64::STXPX ||
            Inst.getOpcode() == AArch64::STXPW ||
            Inst.getOpcode() == AArch64::STXRX ||
            Inst.getOpcode() == AArch64::STXRW ||
            Inst.getOpcode() == AArch64::STXRH ||
            Inst.getOpcode() == AArch64::STXRB ||
            Inst.getOpcode() == AArch64::STLXPX ||
            Inst.getOpcode() == AArch64::STLXPW ||
            Inst.getOpcode() == AArch64::STLXRX ||
            Inst.getOpcode() == AArch64::STLXRW ||
            Inst.getOpcode() == AArch64::STLXRH ||
            Inst.getOpcode() == AArch64::STLXRB);
  }
```

- EN: Declares or implements routines including `isAArch64ExclusiveStore`. Notable symbols here include `isAArch64ExclusiveStore`.
- CN: 这里声明或实现函数，例如 `isAArch64ExclusiveStore`。这里较值得关注的符号包括 `isAArch64ExclusiveStore`。

### Lines 1135-1152

```cpp

  bool isAArch64ExclusiveClear(const MCInst &Inst) const override {
    return (Inst.getOpcode() == AArch64::CLREX);
  }

  bool isLoadFromStack(const MCInst &Inst) const {
    if (!mayLoad(Inst))
      return false;
    for (const MCOperand &Operand : useOperands(Inst)) {
      if (!Operand.isReg())
        continue;
      unsigned Reg = Operand.getReg();
      if (Reg == AArch64::SP || Reg == AArch64::WSP)
        return true;
    }
    return false;
  }
```

- EN: Declares or implements routines including `isAArch64ExclusiveClear`, `isLoadFromStack`. Notable symbols here include `isAArch64ExclusiveClear`, `isLoadFromStack`.
- CN: 这里声明或实现函数，例如 `isAArch64ExclusiveClear`, `isLoadFromStack`。这里较值得关注的符号包括 `isAArch64ExclusiveClear`, `isLoadFromStack`。

### Lines 1153-1160

```cpp
  bool isRegToRegMove(const MCInst &Inst, MCPhysReg &From,
                      MCPhysReg &To) const override {
    if (Inst.getOpcode() == AArch64::FMOVDXr) {
      From = Inst.getOperand(1).getReg();
      To = Inst.getOperand(0).getReg();
      return true;
    }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 1161-1171

```cpp
    if (Inst.getOpcode() != AArch64::ORRXrs)
      return false;
    if (Inst.getOperand(1).getReg() != AArch64::XZR)
      return false;
    if (Inst.getOperand(3).getImm() != 0)
      return false;
    From = Inst.getOperand(2).getReg();
    To = Inst.getOperand(0).getReg();
    return true;
  }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 1172-1186

```cpp
  bool isIndirectCall(const MCInst &Inst) const override {
    return isIndirectCallOpcode(Inst.getOpcode());
  }

  MCPhysReg getSpRegister(int Size) const {
    switch (Size) {
    case 4:
      return AArch64::WSP;
    case 8:
      return AArch64::SP;
    default:
      llvm_unreachable("Unexpected size");
    }
  }
```

- EN: Declares or implements routines including `isIndirectCall`, `getSpRegister`, `llvm_unreachable`. Notable symbols here include `isIndirectCall`, `getSpRegister`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `isIndirectCall`, `getSpRegister`, `llvm_unreachable`。这里较值得关注的符号包括 `isIndirectCall`, `getSpRegister`, `llvm_unreachable`。

### Lines 1187-1204

```cpp
  MCPhysReg getIntArgRegister(unsigned ArgNo) const override {
    switch (ArgNo) {
    case 0:
      return AArch64::X0;
    case 1:
      return AArch64::X1;
    case 2:
      return AArch64::X2;
    case 3:
      return AArch64::X3;
    case 4:
      return AArch64::X4;
    case 5:
      return AArch64::X5;
    case 6:
      return AArch64::X6;
    case 7:
      return AArch64::X7;
```

- EN: Declares or implements routines including `getIntArgRegister`. Notable symbols here include `getIntArgRegister`.
- CN: 这里声明或实现函数，例如 `getIntArgRegister`。这里较值得关注的符号包括 `getIntArgRegister`。

### Lines 1205-1216

```cpp
    default:
      return getNoRegister();
    }
  }

  bool hasPCRelOperand(const MCInst &Inst) const override {
    // ADRP is blacklisted and is an exception. Even though it has a
    // PC-relative operand, this operand is not a complete symbol reference
    // and BOLT shouldn't try to process it in isolation.
    if (isADRP(Inst))
      return false;
```

- EN: Declares or implements routines including `hasPCRelOperand`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `hasPCRelOperand`.
- CN: 这里声明或实现函数，例如 `hasPCRelOperand`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `hasPCRelOperand`。

### Lines 1217-1225

```cpp
    if (isADR(Inst))
      return true;

    // Look for literal addressing mode (see C1-143 ARM DDI 0487B.a)
    const MCInstrDesc &MCII = Info->get(Inst.getOpcode());
    for (unsigned I = 0, E = MCII.getNumOperands(); I != E; ++I)
      if (MCII.operands()[I].OperandType == MCOI::OPERAND_PCREL)
        return true;
```

- EN: Declares or implements routines including `get`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `get`.
- CN: 这里声明或实现函数，例如 `get`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `get`。

### Lines 1226-1240

```cpp
    return false;
  }

  bool evaluateADR(const MCInst &Inst, int64_t &Imm,
                   const MCExpr **DispExpr) const {
    assert((isADR(Inst) || isADRP(Inst)) && "Not an ADR instruction");

    const MCOperand &Label = Inst.getOperand(1);
    if (!Label.isImm()) {
      assert(Label.isExpr() && "Unexpected ADR operand");
      assert(DispExpr && "DispExpr must be set");
      *DispExpr = Label.getExpr();
      return false;
    }
```

- EN: Declares or implements routines including `assert`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `assert`。

### Lines 1241-1248

```cpp
    if (Inst.getOpcode() == AArch64::ADR) {
      Imm = Label.getImm();
      return true;
    }
    Imm = Label.getImm() << 12;
    return true;
  }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 1249-1259

```cpp
  bool evaluateAArch64MemoryOperand(const MCInst &Inst, int64_t &DispImm,
                                    const MCExpr **DispExpr = nullptr) const {
    if (isADR(Inst) || isADRP(Inst))
      return evaluateADR(Inst, DispImm, DispExpr);

    // Literal addressing mode
    const MCInstrDesc &MCII = Info->get(Inst.getOpcode());
    for (unsigned I = 0, E = MCII.getNumOperands(); I != E; ++I) {
      if (MCII.operands()[I].OperandType != MCOI::OPERAND_PCREL)
        continue;
```

- EN: Declares or implements routines including `get`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `get`.
- CN: 这里声明或实现函数，例如 `get`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `get`。

### Lines 1260-1272

```cpp
      if (!Inst.getOperand(I).isImm()) {
        assert(Inst.getOperand(I).isExpr() && "Unexpected PCREL operand");
        assert(DispExpr && "DispExpr must be set");
        *DispExpr = Inst.getOperand(I).getExpr();
        return true;
      }

      DispImm = Inst.getOperand(I).getImm() * 4;
      return true;
    }
    return false;
  }
```

- EN: Declares or implements routines including `assert`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `assert`。

### Lines 1273-1280

```cpp
  bool evaluateMemOperandTarget(const MCInst &Inst, uint64_t &Target,
                                uint64_t Address,
                                uint64_t Size) const override {
    int64_t DispValue;
    const MCExpr *DispExpr = nullptr;
    if (!evaluateAArch64MemoryOperand(Inst, DispValue, &DispExpr))
      return false;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 1281-1292

```cpp
    // Make sure it's a well-formed addressing we can statically evaluate.
    if (DispExpr)
      return false;

    Target = DispValue;
    if (Inst.getOpcode() == AArch64::ADRP)
      Target += Address & ~0xFFFULL;
    else
      Target += Address;
    return true;
  }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 1293-1309

```cpp
  MCInst::iterator getMemOperandDisp(MCInst &Inst) const override {
    MCInst::iterator OI = Inst.begin();
    if (isADR(Inst) || isADRP(Inst)) {
      assert(MCPlus::getNumPrimeOperands(Inst) >= 2 &&
             "Unexpected number of operands");
      return ++OI;
    }
    const MCInstrDesc &MCII = Info->get(Inst.getOpcode());
    for (unsigned I = 0, E = MCII.getNumOperands(); I != E; ++I) {
      if (MCII.operands()[I].OperandType == MCOI::OPERAND_PCREL)
        break;
      ++OI;
    }
    assert(OI != Inst.end() && "Literal operand not found");
    return OI;
  }
```

- EN: Declares or implements routines including `getMemOperandDisp`, `assert`, `get`. Notable symbols here include `getMemOperandDisp`, `assert`, `get`.
- CN: 这里声明或实现函数，例如 `getMemOperandDisp`, `assert`, `get`。这里较值得关注的符号包括 `getMemOperandDisp`, `assert`, `get`。

### Lines 1310-1327

```cpp
  bool replaceMemOperandDisp(MCInst &Inst, MCOperand Operand) const override {
    MCInst::iterator OI = getMemOperandDisp(Inst);
    *OI = Operand;
    return true;
  }

  BitVector getRegsUsedAsParams() const override {
    BitVector Regs = BitVector(RegInfo->getNumRegs(), false);
    Regs |= getAliases(AArch64::X0);
    Regs |= getAliases(AArch64::X1);
    Regs |= getAliases(AArch64::X2);
    Regs |= getAliases(AArch64::X3);
    Regs |= getAliases(AArch64::X4);
    Regs |= getAliases(AArch64::X5);
    Regs |= getAliases(AArch64::X6);
    Regs |= getAliases(AArch64::X7);
    return Regs;
  }
```

- EN: Declares or implements routines including `replaceMemOperandDisp`, `getMemOperandDisp`, `getRegsUsedAsParams`, `BitVector`, `getAliases`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `replaceMemOperandDisp`, `getMemOperandDisp`, `getRegsUsedAsParams`, `BitVector`, `getAliases`.
- CN: 这里声明或实现函数，例如 `replaceMemOperandDisp`, `getMemOperandDisp`, `getRegsUsedAsParams`, `BitVector`, `getAliases`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `replaceMemOperandDisp`, `getMemOperandDisp`, `getRegsUsedAsParams`, `BitVector`, `getAliases`。

### Lines 1328-1344

```cpp

  void getCalleeSavedRegs(BitVector &Regs) const override {
    Regs |= getAliases(AArch64::X18);
    Regs |= getAliases(AArch64::X19);
    Regs |= getAliases(AArch64::X20);
    Regs |= getAliases(AArch64::X21);
    Regs |= getAliases(AArch64::X22);
    Regs |= getAliases(AArch64::X23);
    Regs |= getAliases(AArch64::X24);
    Regs |= getAliases(AArch64::X25);
    Regs |= getAliases(AArch64::X26);
    Regs |= getAliases(AArch64::X27);
    Regs |= getAliases(AArch64::X28);
    Regs |= getAliases(AArch64::LR);
    Regs |= getAliases(AArch64::FP);
  }
```

- EN: Declares or implements routines including `getCalleeSavedRegs`, `getAliases`. Notable symbols here include `getCalleeSavedRegs`, `getAliases`.
- CN: 这里声明或实现函数，例如 `getCalleeSavedRegs`, `getAliases`。这里较值得关注的符号包括 `getCalleeSavedRegs`, `getAliases`。

### Lines 1345-1362

```cpp
  void getDefaultLiveOut(BitVector &Regs) const override {
    // According to the AArch64 ABI the return registers are X0 to X7,
    // which happen to be the same as the parameter registers.
    Regs |= getRegsUsedAsParams();
  }

  void getGPRegs(BitVector &Regs, bool IncludeAlias = true) const override {
    if (IncludeAlias) {
      Regs |= getAliases(AArch64::X0);
      Regs |= getAliases(AArch64::X1);
      Regs |= getAliases(AArch64::X2);
      Regs |= getAliases(AArch64::X3);
      Regs |= getAliases(AArch64::X4);
      Regs |= getAliases(AArch64::X5);
      Regs |= getAliases(AArch64::X6);
      Regs |= getAliases(AArch64::X7);
      Regs |= getAliases(AArch64::X8);
      Regs |= getAliases(AArch64::X9);
```

- EN: Declares or implements routines including `getDefaultLiveOut`, `getRegsUsedAsParams`, `getGPRegs`, `getAliases`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getDefaultLiveOut`, `getRegsUsedAsParams`, `getGPRegs`, `getAliases`.
- CN: 这里声明或实现函数，例如 `getDefaultLiveOut`, `getRegsUsedAsParams`, `getGPRegs`, `getAliases`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getDefaultLiveOut`, `getRegsUsedAsParams`, `getGPRegs`, `getAliases`。

### Lines 1363-1380

```cpp
      Regs |= getAliases(AArch64::X10);
      Regs |= getAliases(AArch64::X11);
      Regs |= getAliases(AArch64::X12);
      Regs |= getAliases(AArch64::X13);
      Regs |= getAliases(AArch64::X14);
      Regs |= getAliases(AArch64::X15);
      Regs |= getAliases(AArch64::X16);
      Regs |= getAliases(AArch64::X17);
      Regs |= getAliases(AArch64::X18);
      Regs |= getAliases(AArch64::X19);
      Regs |= getAliases(AArch64::X20);
      Regs |= getAliases(AArch64::X21);
      Regs |= getAliases(AArch64::X22);
      Regs |= getAliases(AArch64::X23);
      Regs |= getAliases(AArch64::X24);
      Regs |= getAliases(AArch64::X25);
      Regs |= getAliases(AArch64::X26);
      Regs |= getAliases(AArch64::X27);
```

- EN: Declares or implements routines including `getAliases`. Notable symbols here include `getAliases`.
- CN: 这里声明或实现函数，例如 `getAliases`。这里较值得关注的符号包括 `getAliases`。

### Lines 1381-1398

```cpp
      Regs |= getAliases(AArch64::X28);
      Regs |= getAliases(AArch64::LR);
      Regs |= getAliases(AArch64::FP);
      return;
    }
    Regs.set(AArch64::X0);
    Regs.set(AArch64::X1);
    Regs.set(AArch64::X2);
    Regs.set(AArch64::X3);
    Regs.set(AArch64::X4);
    Regs.set(AArch64::X5);
    Regs.set(AArch64::X6);
    Regs.set(AArch64::X7);
    Regs.set(AArch64::X8);
    Regs.set(AArch64::X9);
    Regs.set(AArch64::X10);
    Regs.set(AArch64::X11);
    Regs.set(AArch64::X12);
```

- EN: Declares or implements routines including `getAliases`. Notable symbols here include `getAliases`.
- CN: 这里声明或实现函数，例如 `getAliases`。这里较值得关注的符号包括 `getAliases`。

### Lines 1399-1416

```cpp
    Regs.set(AArch64::X13);
    Regs.set(AArch64::X14);
    Regs.set(AArch64::X15);
    Regs.set(AArch64::X16);
    Regs.set(AArch64::X17);
    Regs.set(AArch64::X18);
    Regs.set(AArch64::X19);
    Regs.set(AArch64::X20);
    Regs.set(AArch64::X21);
    Regs.set(AArch64::X22);
    Regs.set(AArch64::X23);
    Regs.set(AArch64::X24);
    Regs.set(AArch64::X25);
    Regs.set(AArch64::X26);
    Regs.set(AArch64::X27);
    Regs.set(AArch64::X28);
    Regs.set(AArch64::LR);
    Regs.set(AArch64::FP);
```

- EN: Continues the surrounding implementation, wiring declarations, state, and helper logic together.
- CN: 这里继续展开周边实现，衔接声明、状态与辅助逻辑。

### Lines 1417-1426

```cpp
  }

  void removeNonScavengeableRegs(BitVector &Regs) const override {
    BitVector ExclusionMask = getAliases(AArch64::LR);
    ExclusionMask |= getAliases(AArch64::FP);
    ExclusionMask |= getAliases(AArch64::X18); // platform register
    ExclusionMask.flip();
    Regs &= ExclusionMask;
  }
```

- EN: Declares or implements routines including `removeNonScavengeableRegs`, `getAliases`. Notable symbols here include `removeNonScavengeableRegs`, `getAliases`.
- CN: 这里声明或实现函数，例如 `removeNonScavengeableRegs`, `getAliases`。这里较值得关注的符号包括 `removeNonScavengeableRegs`, `getAliases`。

### Lines 1427-1444

```cpp
  const MCExpr *getTargetExprFor(MCInst &Inst, const MCExpr *Expr,
                                 MCContext &Ctx,
                                 uint32_t RelType) const override {

    if (isADR(Inst) || RelType == ELF::R_AARCH64_ADR_PREL_LO21 ||
        RelType == ELF::R_AARCH64_TLSDESC_ADR_PREL21) {
      return MCSpecifierExpr::create(Expr, AArch64::S_ABS, Ctx);
    } else if (isADRP(Inst) || RelType == ELF::R_AARCH64_ADR_PREL_PG_HI21 ||
               RelType == ELF::R_AARCH64_ADR_PREL_PG_HI21_NC ||
               RelType == ELF::R_AARCH64_TLSDESC_ADR_PAGE21 ||
               RelType == ELF::R_AARCH64_TLSIE_ADR_GOTTPREL_PAGE21 ||
               RelType == ELF::R_AARCH64_ADR_GOT_PAGE) {
      // Never emit a GOT reloc, we handled this in
      // RewriteInstance::readRelocations().
      return MCSpecifierExpr::create(Expr, AArch64::S_ABS_PAGE, Ctx);
    } else {
      switch (RelType) {
      case ELF::R_AARCH64_ADD_ABS_LO12_NC:
```

- EN: Declares or implements routines including `if`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `if`.
- CN: 这里声明或实现函数，例如 `if`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `if`。

### Lines 1445-1462

```cpp
      case ELF::R_AARCH64_LD64_GOT_LO12_NC:
      case ELF::R_AARCH64_LDST8_ABS_LO12_NC:
      case ELF::R_AARCH64_LDST16_ABS_LO12_NC:
      case ELF::R_AARCH64_LDST32_ABS_LO12_NC:
      case ELF::R_AARCH64_LDST64_ABS_LO12_NC:
      case ELF::R_AARCH64_LDST128_ABS_LO12_NC:
      case ELF::R_AARCH64_TLSDESC_ADD_LO12:
      case ELF::R_AARCH64_TLSDESC_LD64_LO12:
      case ELF::R_AARCH64_TLSIE_LD64_GOTTPREL_LO12_NC:
      case ELF::R_AARCH64_TLSLE_ADD_TPREL_LO12_NC:
        return MCSpecifierExpr::create(Expr, AArch64::S_LO12, Ctx);
      case ELF::R_AARCH64_MOVW_UABS_G3:
        return MCSpecifierExpr::create(Expr, AArch64::S_ABS_G3, Ctx);
      case ELF::R_AARCH64_MOVW_UABS_G2:
      case ELF::R_AARCH64_MOVW_UABS_G2_NC:
        return MCSpecifierExpr::create(Expr, AArch64::S_ABS_G2_NC, Ctx);
      case ELF::R_AARCH64_MOVW_UABS_G1:
      case ELF::R_AARCH64_MOVW_UABS_G1_NC:
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 1463-1473

```cpp
        return MCSpecifierExpr::create(Expr, AArch64::S_ABS_G1_NC, Ctx);
      case ELF::R_AARCH64_MOVW_UABS_G0:
      case ELF::R_AARCH64_MOVW_UABS_G0_NC:
        return MCSpecifierExpr::create(Expr, AArch64::S_ABS_G0_NC, Ctx);
      default:
        break;
      }
    }
    return Expr;
  }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 1474-1486

```cpp
  bool getSymbolRefOperandNum(const MCInst &Inst, unsigned &OpNum) const {
    if (OpNum >= MCPlus::getNumPrimeOperands(Inst))
      return false;

    // Auto-select correct operand number
    if (OpNum == 0) {
      if (isConditionalBranch(Inst) || isADR(Inst) || isADRP(Inst) ||
          isMOVW(Inst))
        OpNum = 1;
      if (isTB(Inst) || isAddXri(Inst) || isCompAndBranch(Inst))
        OpNum = 2;
    }
```

- EN: Declares or implements routines including `getSymbolRefOperandNum`, `isMOVW`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getSymbolRefOperandNum`, `isMOVW`.
- CN: 这里声明或实现函数，例如 `getSymbolRefOperandNum`, `isMOVW`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getSymbolRefOperandNum`, `isMOVW`。

### Lines 1487-1494

```cpp
    return true;
  }

  const MCSymbol *getTargetSymbol(const MCExpr *Expr) const override {
    auto *AArchExpr = dyn_cast<MCSpecifierExpr>(Expr);
    if (AArchExpr && AArchExpr->getSubExpr())
      return getTargetSymbol(AArchExpr->getSubExpr());
```

- EN: Declares or implements routines including `getTargetSymbol`. Notable symbols here include `getTargetSymbol`.
- CN: 这里声明或实现函数，例如 `getTargetSymbol`。这里较值得关注的符号包括 `getTargetSymbol`。

### Lines 1495-1502

```cpp
    return MCPlusBuilder::getTargetSymbol(Expr);
  }

  const MCSymbol *getTargetSymbol(const MCInst &Inst,
                                  unsigned OpNum = 0) const override {
    if (!OpNum && !getSymbolRefOperandNum(Inst, OpNum))
      return nullptr;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 1503-1514

```cpp
    const MCOperand &Op = Inst.getOperand(OpNum);
    if (!Op.isExpr())
      return nullptr;

    return getTargetSymbol(Op.getExpr());
  }

  int64_t getTargetAddend(const MCExpr *Expr) const override {
    auto *AArchExpr = dyn_cast<MCSpecifierExpr>(Expr);
    if (AArchExpr && AArchExpr->getSubExpr())
      return getTargetAddend(AArchExpr->getSubExpr());
```

- EN: Declares or implements routines including `getTargetAddend`. Notable symbols here include `getTargetAddend`.
- CN: 这里声明或实现函数，例如 `getTargetAddend`。这里较值得关注的符号包括 `getTargetAddend`。

### Lines 1515-1522

```cpp
    auto *BinExpr = dyn_cast<MCBinaryExpr>(Expr);
    if (BinExpr && BinExpr->getOpcode() == MCBinaryExpr::Add)
      return getTargetAddend(BinExpr->getRHS());

    auto *ConstExpr = dyn_cast<MCConstantExpr>(Expr);
    if (ConstExpr)
      return ConstExpr->getValue();
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 1523-1530

```cpp
    return 0;
  }

  int64_t getTargetAddend(const MCInst &Inst,
                          unsigned OpNum = 0) const override {
    if (!getSymbolRefOperandNum(Inst, OpNum))
      return 0;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 1531-1545

```cpp
    const MCOperand &Op = Inst.getOperand(OpNum);
    if (!Op.isExpr())
      return 0;

    return getTargetAddend(Op.getExpr());
  }

  void replaceBranchTarget(MCInst &Inst, const MCSymbol *TBB,
                           MCContext *Ctx) const override {
    assert((isCall(Inst) || isBranch(Inst)) && !isIndirectBranch(Inst) &&
           "Invalid instruction");
    assert(MCPlus::getNumPrimeOperands(Inst) >= 1 &&
           "Invalid number of operands");
    MCInst::iterator OI = Inst.begin();
```

- EN: Declares or implements routines including `assert`. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里较值得关注的符号包括 `assert`。

### Lines 1546-1557

```cpp
    if (isConditionalBranch(Inst)) {
      assert(MCPlus::getNumPrimeOperands(Inst) >= 2 &&
             "Invalid number of operands");
      ++OI;
    }

    if (isTB(Inst) || isCompAndBranch(Inst)) {
      assert(MCPlus::getNumPrimeOperands(Inst) >= 3 &&
             "Invalid number of operands");
      OI = Inst.begin() + 2;
    }
```

- EN: Declares or implements routines including `assert`. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里较值得关注的符号包括 `assert`。

### Lines 1558-1575

```cpp
    *OI = MCOperand::createExpr(MCSymbolRefExpr::create(TBB, *Ctx));
  }

  /// Matches indirect branch patterns in AArch64 related to a jump table (JT),
  /// helping us to build the complete CFG. A typical indirect branch to
  /// a jump table entry in AArch64 looks like the following:
  ///
  ///   adrp    x1, #-7585792           # Get JT Page location
  ///   add     x1, x1, #692            # Complement with JT Page offset
  ///   ldrh    w0, [x1, w0, uxtw #1]   # Loads JT entry
  ///   adr     x1, #12                 # Get PC + 12 (end of this BB) used next
  ///   add     x0, x1, w0, sxth #2     # Finish building branch target
  ///                                   # (entries in JT are relative to the end
  ///                                   #  of this BB)
  ///   br      x0                      # Indirect jump instruction
  ///
  /// Return true on successful jump table instruction sequence match, false
  /// otherwise.
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 1576-1585

```cpp
  bool analyzeIndirectBranchFragment(
      const MCInst &Inst,
      DenseMap<const MCInst *, SmallVector<MCInst *, 4>> &UDChain,
      const MCExpr *&JumpTable, int64_t &Offset, int64_t &ScaleValue,
      MCInst *&PCRelBase) const {
    // The only kind of indirect branches we match is jump table, thus ignore
    // authenticating branch instructions early.
    if (isBRA(Inst))
      return false;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 1586-1595

```cpp
    // Expect AArch64 BR
    assert(Inst.getOpcode() == AArch64::BR && "Unexpected opcode");

    JumpTable = nullptr;

    // Match the indirect branch pattern for aarch64
    SmallVector<MCInst *, 4> &UsesRoot = UDChain[&Inst];
    if (UsesRoot.size() == 0 || UsesRoot[0] == nullptr)
      return false;
```

- EN: Declares or implements routines including `assert`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `assert`。

### Lines 1596-1613

```cpp
    const MCInst *DefAdd = UsesRoot[0];

    // Now we match an ADD
    if (!isADD(*DefAdd)) {
      // If the address is not broken up in two parts, this is not branching
      // according to a jump table entry. Fail.
      return false;
    }
    if (DefAdd->getOpcode() == AArch64::ADDXri) {
      // This can happen when there is no offset, but a direct jump that was
      // transformed into an indirect one  (indirect tail call) :
      //   ADRP   x2, Perl_re_compiler
      //   ADD    x2, x2, :lo12:Perl_re_compiler
      //   BR     x2
      return false;
    }
    if (DefAdd->getOpcode() == AArch64::ADDXrs) {
      // Covers the less common pattern where JT entries are relative to
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 1614-1630

```cpp
      // the JT itself (like x86). Seems less efficient since we can't
      // assume the JT is aligned at 4B boundary and thus drop 2 bits from
      // JT values.
      // cde264:
      //    adrp    x12, #21544960  ; 216a000
      //    add     x12, x12, #1696 ; 216a6a0  (JT object in .rodata)
      //    ldrsw   x8, [x12, x8, lsl #2]   --> loads e.g. 0xfeb73bd8
      //  * add     x8, x8, x12   --> = cde278, next block
      //    br      x8
      // cde278:
      //
      // Parsed as ADDXrs reg:x8 reg:x8 reg:x12 imm:0
      return false;
    }
    if (DefAdd->getOpcode() != AArch64::ADDXrx)
      return false;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 1631-1648

```cpp
    // Validate ADD operands
    int64_t OperandExtension = DefAdd->getOperand(3).getImm();
    unsigned ShiftVal = AArch64_AM::getArithShiftValue(OperandExtension);
    AArch64_AM::ShiftExtendType ExtendType =
        AArch64_AM::getArithExtendType(OperandExtension);
    if (ShiftVal != 2) {
      // TODO: Handle the pattern where ShiftVal != 2.
      // The following code sequence below has no shift amount,
      // the range could be 0 to 4.
      // The pattern comes from libc, it occurs when the binary is static.
      //   adr     x6, 0x219fb0 <sigall_set+0x88>
      //   add     x6, x6, x14, lsl #2
      //   ldr     w7, [x6]
      //   add     x6, x6, w7, sxtw => no shift amount
      //   br      x6
      LLVM_DEBUG(dbgs() << "BOLT-DEBUG: "
                           "failed to match indirect branch: ShiftVAL != 2\n");
      return false;
```

- EN: Declares or implements routines including `getOperand`, `getArithShiftValue`, `getArithExtendType`, `LLVM_DEBUG`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getOperand`, `getArithShiftValue`, `getArithExtendType`, `LLVM_DEBUG`.
- CN: 这里声明或实现函数，例如 `getOperand`, `getArithShiftValue`, `getArithExtendType`, `LLVM_DEBUG`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getOperand`, `getArithShiftValue`, `getArithExtendType`, `LLVM_DEBUG`。

### Lines 1649-1659

```cpp
    }

    if (ExtendType == AArch64_AM::SXTB)
      ScaleValue = 1LL;
    else if (ExtendType == AArch64_AM::SXTH)
      ScaleValue = 2LL;
    else if (ExtendType == AArch64_AM::SXTW)
      ScaleValue = 4LL;
    else
      return false;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 1660-1671

```cpp
    // Match an ADR to load base address to be used when addressing JT targets
    SmallVector<MCInst *, 4> &UsesAdd = UDChain[DefAdd];
    if (UsesAdd.size() <= 1 || UsesAdd[1] == nullptr || UsesAdd[2] == nullptr) {
      // This happens when we don't have enough context about this jump table
      // because the jumping code sequence was split in multiple basic blocks.
      // This was observed in the wild in HHVM code (dispatchImpl).
      return false;
    }
    MCInst *DefBaseAddr = UsesAdd[1];
    if (DefBaseAddr->getOpcode() != AArch64::ADR)
      return false;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 1672-1689

```cpp
    PCRelBase = DefBaseAddr;
    // Match LOAD to load the jump table (relative) target
    const MCInst *DefLoad = UsesAdd[2];
    if (!mayLoad(*DefLoad) || (ScaleValue == 1LL && !isLDRB(*DefLoad)) ||
        (ScaleValue == 2LL && !isLDRH(*DefLoad)))
      return false;

    // Match ADD that calculates the JumpTable Base Address (not the offset)
    SmallVector<MCInst *, 4> &UsesLoad = UDChain[DefLoad];
    const MCInst *DefJTBaseAdd = UsesLoad[1];
    MCPhysReg From, To;
    if (DefJTBaseAdd == nullptr || isLoadFromStack(*DefJTBaseAdd) ||
        isRegToRegMove(*DefJTBaseAdd, From, To)) {
      // Sometimes base address may have been defined in another basic block
      // (hoisted). Return with no jump table info.
      return true;
    }
```

- EN: Declares or implements routines including `isRegToRegMove`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isRegToRegMove`.
- CN: 这里声明或实现函数，例如 `isRegToRegMove`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isRegToRegMove`。

### Lines 1690-1702

```cpp
    if (DefJTBaseAdd->getOpcode() == AArch64::ADR) {
      // TODO: Handle the pattern where there is no adrp/add pair.
      // It also occurs when the binary is static.
      //  adr     x13, 0x215a18 <_nl_value_type_LC_COLLATE+0x50>
      //  ldrh    w13, [x13, w12, uxtw #1]
      //  adr     x12, 0x247b30 <__gettextparse+0x5b0>
      //  add     x13, x12, w13, sxth #2
      //  br      x13
      LLVM_DEBUG(dbgs() << "BOLT-DEBUG: failed to match indirect branch: "
                           "nop/adr instead of adrp/add\n");
      return false;
    }
```

- EN: Declares or implements routines including `LLVM_DEBUG`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `LLVM_DEBUG`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `LLVM_DEBUG`。

### Lines 1703-1718

```cpp
    if (DefJTBaseAdd->getOpcode() != AArch64::ADDXri) {
      LLVM_DEBUG(dbgs() << "BOLT-DEBUG: failed to match jump table base "
                           "address pattern! (1)\n");
      return false;
    }

    if (DefJTBaseAdd->getOperand(2).isImm())
      Offset = DefJTBaseAdd->getOperand(2).getImm();
    SmallVector<MCInst *, 4> &UsesJTBaseAdd = UDChain[DefJTBaseAdd];
    const MCInst *DefJTBasePage = UsesJTBaseAdd[1];
    if (DefJTBasePage == nullptr || isLoadFromStack(*DefJTBasePage)) {
      return true;
    }
    if (DefJTBasePage->getOpcode() != AArch64::ADRP)
      return false;
```

- EN: Declares or implements routines including `LLVM_DEBUG`, `getOperand`. Notable symbols here include `LLVM_DEBUG`, `getOperand`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`, `getOperand`。这里较值得关注的符号包括 `LLVM_DEBUG`, `getOperand`。

### Lines 1719-1729

```cpp
    if (DefJTBasePage->getOperand(1).isExpr())
      JumpTable = DefJTBasePage->getOperand(1).getExpr();
    return true;
  }

  DenseMap<const MCInst *, SmallVector<MCInst *, 4>>
  computeLocalUDChain(const MCInst *CurInstr, InstructionIterator Begin,
                      InstructionIterator End) const {
    DenseMap<int, MCInst *> RegAliasTable;
    DenseMap<const MCInst *, SmallVector<MCInst *, 4>> Uses;
```

- EN: Declares or implements routines including `getOperand`. Notable symbols here include `getOperand`.
- CN: 这里声明或实现函数，例如 `getOperand`。这里较值得关注的符号包括 `getOperand`。

### Lines 1730-1747

```cpp
    auto addInstrOperands = [&](const MCInst &Instr) {
      // Update Uses table
      for (const MCOperand &Operand : MCPlus::primeOperands(Instr)) {
        if (!Operand.isReg())
          continue;
        unsigned Reg = Operand.getReg();
        MCInst *AliasInst = RegAliasTable[Reg];
        Uses[&Instr].push_back(AliasInst);
        LLVM_DEBUG({
          dbgs() << "Adding reg operand " << Reg << " refs ";
          if (AliasInst != nullptr)
            AliasInst->dump();
          else
            dbgs() << "\n";
        });
      }
    };
```

- EN: Declares or implements routines including `dbgs`, `dump`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `dbgs`, `dump`.
- CN: 这里声明或实现函数，例如 `dbgs`, `dump`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `dbgs`, `dump`。

### Lines 1748-1759

```cpp
    LLVM_DEBUG(dbgs() << "computeLocalUDChain\n");
    bool TerminatorSeen = false;
    for (auto II = Begin; II != End; ++II) {
      MCInst &Instr = *II;
      // Ignore nops and CFIs
      if (isPseudo(Instr) || isNoop(Instr))
        continue;
      if (TerminatorSeen) {
        RegAliasTable.clear();
        Uses.clear();
      }
```

- EN: Declares or implements routines including `LLVM_DEBUG`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `LLVM_DEBUG`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `LLVM_DEBUG`。

### Lines 1760-1773

```cpp
      LLVM_DEBUG(dbgs() << "Now updating for:\n ");
      LLVM_DEBUG(Instr.dump());
      addInstrOperands(Instr);

      BitVector Regs = BitVector(RegInfo->getNumRegs(), false);
      getWrittenRegs(Instr, Regs);

      // Update register definitions after this point
      for (int Idx : Regs.set_bits()) {
        RegAliasTable[Idx] = &Instr;
        LLVM_DEBUG(dbgs() << "Setting reg " << Idx
                          << " def to current instr.\n");
      }
```

- EN: Declares or implements routines including `LLVM_DEBUG`, `addInstrOperands`, `BitVector`, `getWrittenRegs`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `LLVM_DEBUG`, `addInstrOperands`, `BitVector`, `getWrittenRegs`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`, `addInstrOperands`, `BitVector`, `getWrittenRegs`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `LLVM_DEBUG`, `addInstrOperands`, `BitVector`, `getWrittenRegs`。

### Lines 1774-1781

```cpp
      TerminatorSeen = isTerminator(Instr);
    }

    // Process the last instruction, which is not currently added into the
    // instruction stream
    if (CurInstr)
      addInstrOperands(*CurInstr);
```

- EN: Declares or implements routines including `isTerminator`, `addInstrOperands`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isTerminator`, `addInstrOperands`.
- CN: 这里声明或实现函数，例如 `isTerminator`, `addInstrOperands`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isTerminator`, `addInstrOperands`。

### Lines 1782-1798

```cpp
    return Uses;
  }

  IndirectBranchType
  analyzeIndirectBranch(MCInst &Instruction, InstructionIterator Begin,
                        InstructionIterator End, const unsigned PtrSize,
                        MCInst *&MemLocInstrOut, unsigned &BaseRegNumOut,
                        unsigned &IndexRegNumOut, int64_t &DispValueOut,
                        const MCExpr *&DispExprOut, MCInst *&PCRelBaseOut,
                        MCInst *&FixedEntryLoadInstr) const override {
    MemLocInstrOut = nullptr;
    BaseRegNumOut = AArch64::NoRegister;
    IndexRegNumOut = AArch64::NoRegister;
    DispValueOut = 0;
    DispExprOut = nullptr;
    FixedEntryLoadInstr = nullptr;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 1799-1807

```cpp
    // An instruction referencing memory used by jump instruction (directly or
    // via register). This location could be an array of function pointers
    // in case of indirect tail call, or a jump table.
    MCInst *MemLocInstr = nullptr;

    // Analyze the memory location.
    int64_t ScaleValue, DispValue;
    const MCExpr *DispExpr;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 1808-1821

```cpp
    DenseMap<const MCInst *, SmallVector<llvm::MCInst *, 4>> UDChain =
        computeLocalUDChain(&Instruction, Begin, End);
    MCInst *PCRelBase;
    if (!analyzeIndirectBranchFragment(Instruction, UDChain, DispExpr,
                                       DispValue, ScaleValue, PCRelBase))
      return IndirectBranchType::UNKNOWN;

    MemLocInstrOut = MemLocInstr;
    DispValueOut = DispValue;
    DispExprOut = DispExpr;
    PCRelBaseOut = PCRelBase;
    return IndirectBranchType::POSSIBLE_PIC_JUMP_TABLE;
  }
```

- EN: Declares or implements routines including `computeLocalUDChain`. Notable symbols here include `computeLocalUDChain`.
- CN: 这里声明或实现函数，例如 `computeLocalUDChain`。这里较值得关注的符号包括 `computeLocalUDChain`。

### Lines 1822-1839

```cpp
  ///  Matches PLT entry pattern and returns the associated GOT entry address.
  ///  Typical PLT entry looks like the following:
  ///
  ///    adrp    x16, 230000
  ///    ldr     x17, [x16, #3040]
  ///    add     x16, x16, #0xbe0
  ///    br      x17
  ///
  ///  The other type of trampolines are located in .plt.got, that are used for
  ///  non-lazy bindings so doesn't use x16 arg to transfer .got entry address:
  ///
  ///    adrp    x16, 230000
  ///    ldr     x17, [x16, #3040]
  ///    br      x17
  ///    nop
  ///
  uint64_t analyzePLTEntry(MCInst &Instruction, InstructionIterator Begin,
                           InstructionIterator End,
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 1840-1847

```cpp
                           uint64_t BeginPC) const override {
    // Check branch instruction
    MCInst *Branch = &Instruction;
    assert(Branch->getOpcode() == AArch64::BR && "Unexpected opcode");

    DenseMap<const MCInst *, SmallVector<llvm::MCInst *, 4>> UDChain =
        computeLocalUDChain(Branch, Begin, End);
```

- EN: Declares or implements routines including `assert`, `computeLocalUDChain`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `assert`, `computeLocalUDChain`.
- CN: 这里声明或实现函数，例如 `assert`, `computeLocalUDChain`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `assert`, `computeLocalUDChain`。

### Lines 1848-1857

```cpp
    // Match ldr instruction
    SmallVector<MCInst *, 4> &BranchUses = UDChain[Branch];
    if (BranchUses.size() < 1 || BranchUses[0] == nullptr)
      return 0;

    // Check ldr instruction
    const MCInst *Ldr = BranchUses[0];
    if (Ldr->getOpcode() != AArch64::LDRXui)
      return 0;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 1858-1867

```cpp
    // Get ldr value
    const unsigned ScaleLdr = 8; // LDRX operates on 8 bytes segments
    assert(Ldr->getOperand(2).isImm() && "Unexpected ldr operand");
    const uint64_t Offset = Ldr->getOperand(2).getImm() * ScaleLdr;

    // Match adrp instruction
    SmallVector<MCInst *, 4> &LdrUses = UDChain[Ldr];
    if (LdrUses.size() < 2 || LdrUses[1] == nullptr)
      return 0;
```

- EN: Declares or implements routines including `assert`, `getOperand`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `assert`, `getOperand`.
- CN: 这里声明或实现函数，例如 `assert`, `getOperand`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `assert`, `getOperand`。

### Lines 1868-1881

```cpp
    // Check adrp instruction
    MCInst *Adrp = LdrUses[1];
    if (Adrp->getOpcode() != AArch64::ADRP)
      return 0;

    // Get adrp instruction PC
    const unsigned InstSize = 4;
    uint64_t AdrpPC = BeginPC;
    for (InstructionIterator It = Begin; It != End; ++It) {
      if (&(*It) == Adrp)
        break;
      AdrpPC += InstSize;
    }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 1882-1891

```cpp
    // Get adrp value
    uint64_t Base;
    assert(Adrp->getOperand(1).isImm() && "Unexpected adrp operand");
    bool Ret = evaluateMemOperandTarget(*Adrp, Base, AdrpPC, InstSize);
    assert(Ret && "Failed to evaluate adrp");
    (void)Ret;

    return Base + Offset;
  }
```

- EN: Declares or implements routines including `assert`, `evaluateMemOperandTarget`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `assert`, `evaluateMemOperandTarget`.
- CN: 这里声明或实现函数，例如 `assert`, `evaluateMemOperandTarget`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `assert`, `evaluateMemOperandTarget`。

### Lines 1892-1909

```cpp
  /// This function is used to patch PLT entries to include a BTI instruction.
  /// This currently only works for binaries linked using LLD.
  ///
  /// PLT entry before patching:
  ///
  ///    adrp x16, Page(&(.got.plt[n]))
  ///    ldr  x17, [x16, Offset(&(.got.plt[n]))]
  ///    add  x16, x16, Offset(&(.got.plt[n]))
  ///    br   x17
  ///    nop
  ///    nop
  ///
  /// PLT entry after patching:
  ///
  ///    bti c
  ///    adrp x16, Page(&(.got.plt[n]))
  ///    ldr  x17, [x16, Offset(&(.got.plt[n]))]
  ///    add  x16, x16, Offset(&(.got.plt[n]))
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 1910-1927

```cpp
  ///    br   x17
  ///    nop
  ///
  /// Safety considerations:
  ///
  /// The PLT entry will become incorrect if shifting the ADRP by one
  /// instruction (4 bytes) moves it across a page boundary.
  ///
  /// The PLT entry is 24 bytes, and page size is 4096 (or 16384) bytes.
  /// Their GCD is 8 bytes, meaning that shifting the ADRP is safe, as long as
  /// it is shifted by less than 8 bytes.
  ///
  /// If the PLT entry does not contain extra nops, this function will create an
  /// error. This can happen in binaries linked using BFD.
  void patchPLTEntryForBTI(BinaryFunction &PLTFunction, MCInst &Call) override {
    BinaryContext &BC = PLTFunction.getBinaryContext();
    assert(PLTFunction.isPLTFunction() &&
           "patchPLTEntryForBTI called on a non-PLT function");
```

- EN: Declares or implements routines including `patchPLTEntryForBTI`, `assert`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `patchPLTEntryForBTI`, `assert`.
- CN: 这里声明或实现函数，例如 `patchPLTEntryForBTI`, `assert`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `patchPLTEntryForBTI`, `assert`。

### Lines 1928-1945

```cpp
    // Checking if the PLT entry already starts with the BTI needed for Call.
    auto FirstBBI = PLTFunction.begin();
    auto FirstII = FirstBBI->begin();
    assert(FirstII != FirstBBI->end() && "Cannot patch empty PLT entry");
    if (isCallCoveredByBTI(Call, *FirstII))
      return;
    // Checking if there are extra nops at the end. If not, BOLT cannot patch
    // the PLT entry.
    auto LastBBI = std::prev(PLTFunction.end());
    auto LastII = std::prev(LastBBI->end());
    if (!isNoop(*LastII)) {
      errs() << "BOLT-ERROR: Cannot patch PLT entry "
             << PLTFunction.getPrintName()
             << " to have a BTI landing pad. Relink the binary using LLD.\n";
      exit(1);
    }
    // If the PLT does not have a BTI, and it has nops, create a new instruction
    // sequence to patch the entry with.
```

- EN: Declares or implements routines including `begin`, `assert`, `prev`, `errs`, `exit`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `begin`, `assert`, `prev`, `errs`, `exit`.
- CN: 这里声明或实现函数，例如 `begin`, `assert`, `prev`, `errs`, `exit`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `begin`, `assert`, `prev`, `errs`, `exit`。

### Lines 1946-1955

```cpp
    InstructionListType NewPLTSeq;
    MCInst BTIInst;
    createBTI(BTIInst, BTIKind::C);
    NewPLTSeq.push_back(BTIInst);
    // Only adding the instructions from the first BB (adrp, ldr, add, br) to
    // NewPLTSeq.
    NewPLTSeq.insert(NewPLTSeq.end(), FirstBBI->begin(), FirstBBI->end());
    BC.createInstructionPatch(PLTFunction.getAddress(), NewPLTSeq);
  }
```

- EN: Declares or implements routines including `createBTI`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `createBTI`.
- CN: 这里声明或实现函数，例如 `createBTI`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `createBTI`。

### Lines 1956-1973

```cpp
  /// Decode entry instruction of \p Function without CFG. If it's a BTI
  /// matching \p Call, do nothing. If it's a nop, patch it to a BTI. If it's
  /// neither, emit an error.
  void patchFunctionEntryForBTI(BinaryFunction &Function,
                                MCInst &Call) override {
    BinaryContext &BC = Function.getBinaryContext();
    const uint64_t InstrAddr = Function.getAddress();
    ErrorOr<ArrayRef<uint8_t>> FunctionData = Function.getData();
    if (!FunctionData) {
      errs() << "BOLT-ERROR: corresponding section is non-executable or "
             << "empty for function " << Function.getPrintName();
      exit(1);
    }
    // getInstruction writes this, its value doesn't matter here.
    uint64_t InstrSize = 0;
    MCInst FirstInst;
    if (FunctionData->empty() ||
        !BC.DisAsm->getInstruction(FirstInst, InstrSize, *FunctionData,
```

- EN: Declares or implements routines including `errs`, `exit`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `errs`, `exit`.
- CN: 这里声明或实现函数，例如 `errs`, `exit`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `errs`, `exit`。

### Lines 1974-1991

```cpp
                                   InstrAddr, nulls())) {
      errs() << "BOLT-ERROR: unable to disassemble first instruction of "
             << Function.getPrintName()
             << formatv(" at address {0:x}\n", InstrAddr);
      exit(1);
    }
    if (isCallCoveredByBTI(Call, FirstInst))
      return;
    if (!isNoop(FirstInst)) {
      errs() << "BOLT-ERROR: Cannot add BTI to function without CFG "
             << Function.getPrintName()
             << ". Recompile the binary using -fpatchable-function-entry 1 to "
                "include a nop at the entry";
      exit(1);
    }
    InstructionListType NewEntry;
    MCInst BTIInst;
    createBTI(BTIInst, BTIKind::C);
```

- EN: Declares or implements routines including `nulls`, `errs`, `formatv`, `exit`, `createBTI`. Notable symbols here include `nulls`, `errs`, `formatv`, `exit`, `createBTI`.
- CN: 这里声明或实现函数，例如 `nulls`, `errs`, `formatv`, `exit`, `createBTI`。这里较值得关注的符号包括 `nulls`, `errs`, `formatv`, `exit`, `createBTI`。

### Lines 1992-2001

```cpp
    NewEntry.push_back(BTIInst);
    BC.createInstructionPatch(Function.getAddress(), NewEntry);
  }

  void applyBTIFixupToSymbol(BinaryContext &BC, const MCSymbol *TargetSymbol,
                             MCInst &Call) override {
    BinaryFunction *TargetFunction = BC.getFunctionForSymbol(TargetSymbol);
    applyBTIFixupCommon(TargetSymbol, TargetFunction, nullptr, Call);
  }
```

- EN: Declares or implements routines including `applyBTIFixupCommon`. Notable symbols here include `applyBTIFixupCommon`.
- CN: 这里声明或实现函数，例如 `applyBTIFixupCommon`。这里较值得关注的符号包括 `applyBTIFixupCommon`。

### Lines 2002-2011

```cpp
  void applyBTIFixupToTarget(BinaryBasicBlock &StubBB) override {
    BinaryFunction &Func = *StubBB.getFunction();
    BinaryContext &BC = Func.getBinaryContext();
    const MCSymbol *RealTargetSym = BC.MIB->getTargetSymbol(*StubBB.begin());
    BinaryFunction *TargetFunction = BC.getFunctionForSymbol(RealTargetSym);
    BinaryBasicBlock *TgtBB = Func.getBasicBlockForLabel(RealTargetSym);
    applyBTIFixupCommon(RealTargetSym, TargetFunction, TgtBB,
                        *StubBB.getLastNonPseudoInstr());
  }
```

- EN: Declares or implements routines including `applyBTIFixupToTarget`, `getTargetSymbol`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `applyBTIFixupToTarget`, `getTargetSymbol`.
- CN: 这里声明或实现函数，例如 `applyBTIFixupToTarget`, `getTargetSymbol`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `applyBTIFixupToTarget`, `getTargetSymbol`。

### Lines 2012-2029

```cpp
  void applyBTIFixupCommon(const MCSymbol *RealTargetSym,
                           BinaryFunction *TargetFunction,
                           BinaryBasicBlock *TargetBB, MCInst &Call) override {
    // TODO: add support for editing each type, and remove errors.
    if (!TargetFunction && !TargetBB) {
      errs() << "BOLT-ERROR: Cannot add BTI to function with symbol "
             << RealTargetSym->getName() << "\n";
      exit(1);
    }
    if (TargetFunction && TargetFunction->isPLTFunction()) {
      patchPLTEntryForBTI(*TargetFunction, Call);
      return;
    }
    if (TargetFunction &&
        (TargetFunction->isIgnored() || !TargetFunction->hasCFG())) {
      patchFunctionEntryForBTI(*TargetFunction, Call);
      return;
    }
```

- EN: Declares or implements routines including `errs`, `getName`, `exit`, `patchPLTEntryForBTI`, `patchFunctionEntryForBTI`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `errs`, `getName`, `exit`, `patchPLTEntryForBTI`, `patchFunctionEntryForBTI`.
- CN: 这里声明或实现函数，例如 `errs`, `getName`, `exit`, `patchPLTEntryForBTI`, `patchFunctionEntryForBTI`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `errs`, `getName`, `exit`, `patchPLTEntryForBTI`, `patchFunctionEntryForBTI`。

### Lines 2030-2047

```cpp
    if (!TargetBB)
      // No need to check TargetFunction for nullptr, because
      // !TargetBB &&!TargetFunction has already been checked.
      TargetBB = &*TargetFunction->begin();
    if (TargetBB) {
      if (!TargetBB->hasParent()) {
        errs() << "BOLT-ERROR: Cannot add BTI to block with no parent "
                  "function. Targeted symbol: "
               << RealTargetSym->getName() << "\n";
        exit(1);
      }
      insertBTI(*TargetBB, Call);
      return;
    }
    errs() << "BOLT-ERROR: unhandled case when applying BTI fixup\n";
    exit(1);
  }
```

- EN: Declares or implements routines including `begin`, `errs`, `getName`, `exit`, `insertBTI`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `begin`, `errs`, `getName`, `exit`, `insertBTI`.
- CN: 这里声明或实现函数，例如 `begin`, `errs`, `getName`, `exit`, `insertBTI`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `begin`, `errs`, `getName`, `exit`, `insertBTI`。

### Lines 2048-2065

```cpp
  unsigned getInvertedBranchOpcode(unsigned Opcode) const {
    // clang-format off
    switch (Opcode) {
    default:
      llvm_unreachable("Failed to invert branch opcode");
      return Opcode;
    case AArch64::TBZW:     return AArch64::TBNZW;
    case AArch64::TBZX:     return AArch64::TBNZX;
    case AArch64::TBNZW:    return AArch64::TBZW;
    case AArch64::TBNZX:    return AArch64::TBZX;
    case AArch64::CBZW:     return AArch64::CBNZW;
    case AArch64::CBZX:     return AArch64::CBNZX;
    case AArch64::CBNZW:    return AArch64::CBZW;
    case AArch64::CBNZX:    return AArch64::CBZX;
    // Compare register with immediate and branch.
    case AArch64::CBGTWri:  return AArch64::CBLTWri; // +1
    case AArch64::CBGTXri:  return AArch64::CBLTXri; // +1
    case AArch64::CBLTWri:  return AArch64::CBGTWri; // -1
```

- EN: Declares or implements routines including `getInvertedBranchOpcode`, `llvm_unreachable`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getInvertedBranchOpcode`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `getInvertedBranchOpcode`, `llvm_unreachable`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getInvertedBranchOpcode`, `llvm_unreachable`。

### Lines 2066-2083

```cpp
    case AArch64::CBLTXri:  return AArch64::CBGTXri; // -1
    case AArch64::CBHIWri:  return AArch64::CBLOWri; // +1
    case AArch64::CBHIXri:  return AArch64::CBLOXri; // +1
    case AArch64::CBLOWri:  return AArch64::CBHIWri; // -1
    case AArch64::CBLOXri:  return AArch64::CBHIXri; // -1
    case AArch64::CBEQWri:  return AArch64::CBNEWri;
    case AArch64::CBEQXri:  return AArch64::CBNEXri;
    case AArch64::CBNEWri:  return AArch64::CBEQWri;
    case AArch64::CBNEXri:  return AArch64::CBEQXri;
    // Compare registers and branch.
    case AArch64::CBGTWrr:  return AArch64::CBGEWrr; // swap
    case AArch64::CBGTXrr:  return AArch64::CBGEXrr; // swap
    case AArch64::CBGEWrr:  return AArch64::CBGTWrr; // swap
    case AArch64::CBGEXrr:  return AArch64::CBGTXrr; // swap
    case AArch64::CBHIWrr:  return AArch64::CBHSWrr; // swap
    case AArch64::CBHIXrr:  return AArch64::CBHSXrr; // swap
    case AArch64::CBHSWrr:  return AArch64::CBHIWrr; // swap
    case AArch64::CBHSXrr:  return AArch64::CBHIXrr; // swap
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 2084-2101

```cpp
    case AArch64::CBEQWrr:  return AArch64::CBNEWrr;
    case AArch64::CBEQXrr:  return AArch64::CBNEXrr;
    case AArch64::CBNEWrr:  return AArch64::CBEQWrr;
    case AArch64::CBNEXrr:  return AArch64::CBEQXrr;
    // Compare bytes and branch.
    case AArch64::CBBGTWrr: return AArch64::CBBGEWrr; // swap
    case AArch64::CBBGEWrr: return AArch64::CBBGTWrr; // swap
    case AArch64::CBBHIWrr: return AArch64::CBBHSWrr; // swap
    case AArch64::CBBHSWrr: return AArch64::CBBHIWrr; // swap
    case AArch64::CBBEQWrr: return AArch64::CBBNEWrr;
    case AArch64::CBBNEWrr: return AArch64::CBBEQWrr;
    // Compare halfwords and branch.
    case AArch64::CBHGTWrr: return AArch64::CBHGEWrr; // swap
    case AArch64::CBHGEWrr: return AArch64::CBHGTWrr; // swap
    case AArch64::CBHHIWrr: return AArch64::CBHHSWrr; // swap
    case AArch64::CBHHSWrr: return AArch64::CBHHIWrr; // swap
    case AArch64::CBHEQWrr: return AArch64::CBHNEWrr;
    case AArch64::CBHNEWrr: return AArch64::CBHEQWrr;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 2102-2111

```cpp
    }
    // clang-format on
  }

  unsigned getCondCode(const MCInst &Inst) const override {
    // AArch64 does not use conditional codes, so we just return the opcode
    // of the conditional branch here.
    return Inst.getOpcode();
  }
```

- EN: Declares or implements routines including `getCondCode`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getCondCode`.
- CN: 这里声明或实现函数，例如 `getCondCode`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getCondCode`。

### Lines 2112-2122

```cpp
  unsigned getCanonicalBranchCondCode(unsigned Opcode) const override {
    switch (Opcode) {
    default:
      return Opcode;
    case AArch64::TBNZW:    return AArch64::TBZW;
    case AArch64::TBNZX:    return AArch64::TBZX;
    case AArch64::CBNZW:    return AArch64::CBZW;
    case AArch64::CBNZX:    return AArch64::CBZX;
    }
  }
```

- EN: Declares or implements routines including `getCanonicalBranchCondCode`. Notable symbols here include `getCanonicalBranchCondCode`.
- CN: 这里声明或实现函数，例如 `getCanonicalBranchCondCode`。这里较值得关注的符号包括 `getCanonicalBranchCondCode`。

### Lines 2123-2140

```cpp
  bool needsRegSwap(unsigned Opcode) const {
    switch (Opcode) {
    default:
      return false;
    // Compare registers and branch.
    case AArch64::CBGTWrr:
    case AArch64::CBGTXrr:
    case AArch64::CBGEWrr:
    case AArch64::CBGEXrr:
    case AArch64::CBHIWrr:
    case AArch64::CBHIXrr:
    case AArch64::CBHSWrr:
    case AArch64::CBHSXrr:
    // Compare bytes and branch.
    case AArch64::CBBGTWrr:
    case AArch64::CBBGEWrr:
    case AArch64::CBBHIWrr:
    case AArch64::CBBHSWrr:
```

- EN: Declares or implements routines including `needsRegSwap`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `needsRegSwap`.
- CN: 这里声明或实现函数，例如 `needsRegSwap`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `needsRegSwap`。

### Lines 2141-2149

```cpp
    // Compare halfwords and branch.
    case AArch64::CBHGTWrr:
    case AArch64::CBHGEWrr:
    case AArch64::CBHHIWrr:
    case AArch64::CBHHSWrr:
      return true;
    }
  }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 2150-2161

```cpp
  bool needsImmDec(unsigned Opcode) const {
    switch (Opcode) {
    default:
      return false;
    case AArch64::CBGTWri:
    case AArch64::CBGTXri:
    case AArch64::CBHIWri:
    case AArch64::CBHIXri:
      return true;
    }
  }
```

- EN: Declares or implements routines including `needsImmDec`. Notable symbols here include `needsImmDec`.
- CN: 这里声明或实现函数，例如 `needsImmDec`。这里较值得关注的符号包括 `needsImmDec`。

### Lines 2162-2173

```cpp
  bool needsImmInc(unsigned Opcode) const {
    switch (Opcode) {
    default:
      return false;
    case AArch64::CBLTWri:
    case AArch64::CBLTXri:
    case AArch64::CBLOWri:
    case AArch64::CBLOXri:
      return true;
    }
  }
```

- EN: Declares or implements routines including `needsImmInc`. Notable symbols here include `needsImmInc`.
- CN: 这里声明或实现函数，例如 `needsImmInc`。这里较值得关注的符号包括 `needsImmInc`。

### Lines 2174-2184

```cpp
  bool isReversibleBranch(const MCInst &Inst) const override {
    if (isCompAndBranch(Inst)) {
      unsigned InvertedOpcode = getInvertedBranchOpcode(Inst.getOpcode());
      if (needsImmDec(InvertedOpcode) && Inst.getOperand(1).getImm() == 0)
        return false;
      if (needsImmInc(InvertedOpcode) && Inst.getOperand(1).getImm() == 63)
        return false;
    }
    return MCPlusBuilder::isReversibleBranch(Inst);
  }
```

- EN: Declares or implements routines including `isReversibleBranch`, `getInvertedBranchOpcode`. Notable symbols here include `isReversibleBranch`, `getInvertedBranchOpcode`.
- CN: 这里声明或实现函数，例如 `isReversibleBranch`, `getInvertedBranchOpcode`。这里较值得关注的符号包括 `isReversibleBranch`, `getInvertedBranchOpcode`。

### Lines 2185-2202

```cpp
  void reverseBranchCondition(MCInst &Inst, const MCSymbol *TBB,
                              MCContext *Ctx) const override {
    if (!isReversibleBranch(Inst)) {
      errs() << "BOLT-ERROR: Cannot reverse branch " << Inst << "\n";
      exit(1);
    }

    if (isTB(Inst) || isCB(Inst) || isCompAndBranch(Inst)) {
      unsigned InvertedOpcode = getInvertedBranchOpcode(Inst.getOpcode());
      Inst.setOpcode(InvertedOpcode);
      assert(Inst.getOpcode() != 0 && "Invalid branch instruction");
      // The FEAT_CMPBR compare-and-branch instructions cannot encode all
      // the possible condition codes, therefore we either have to adjust
      // the immediate value by +-1, or to swap the register operands
      // when reversing the branch condition.
      if (needsRegSwap(InvertedOpcode))
        std::swap(Inst.getOperand(0), Inst.getOperand(1));
      else if (needsImmDec(InvertedOpcode))
```

- EN: Declares or implements routines including `errs`, `exit`, `getInvertedBranchOpcode`, `assert`, `swap`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `errs`, `exit`, `getInvertedBranchOpcode`, `assert`, `swap`.
- CN: 这里声明或实现函数，例如 `errs`, `exit`, `getInvertedBranchOpcode`, `assert`, `swap`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `errs`, `exit`, `getInvertedBranchOpcode`, `assert`, `swap`。

### Lines 2203-2218

```cpp
        Inst.getOperand(1).setImm(Inst.getOperand(1).getImm() - 1);
      else if (needsImmInc(InvertedOpcode))
        Inst.getOperand(1).setImm(Inst.getOperand(1).getImm() + 1);
    } else if (Inst.getOpcode() == AArch64::Bcc) {
      Inst.getOperand(0).setImm(AArch64CC::getInvertedCondCode(
          static_cast<AArch64CC::CondCode>(Inst.getOperand(0).getImm())));
      assert(Inst.getOperand(0).getImm() != AArch64CC::AL &&
             Inst.getOperand(0).getImm() != AArch64CC::NV &&
             "Can't reverse ALWAYS cond code");
    } else {
      LLVM_DEBUG(Inst.dump());
      llvm_unreachable("Unrecognized branch instruction");
    }
    replaceBranchTarget(Inst, TBB, Ctx);
  }
```

- EN: Declares or implements routines including `if`, `assert`, `LLVM_DEBUG`, `llvm_unreachable`, `replaceBranchTarget`. Notable symbols here include `if`, `assert`, `LLVM_DEBUG`, `llvm_unreachable`, `replaceBranchTarget`.
- CN: 这里声明或实现函数，例如 `if`, `assert`, `LLVM_DEBUG`, `llvm_unreachable`, `replaceBranchTarget`。这里较值得关注的符号包括 `if`, `assert`, `LLVM_DEBUG`, `llvm_unreachable`, `replaceBranchTarget`。

### Lines 2219-2235

```cpp
  int getPCRelEncodingSize(const MCInst &Inst) const override {
    if (isCompAndBranch(Inst))
      return 11;
    if (isTB(Inst))
      return 16;
    if (isCB(Inst))
      return 21;
    switch (Inst.getOpcode()) {
    default:
      llvm_unreachable("Failed to get pcrel encoding size");
      return 0;
    case AArch64::B:        return 28;
    case AArch64::BL:       return 28;
    case AArch64::Bcc:      return 21;
    }
  }
```

- EN: Declares or implements routines including `getPCRelEncodingSize`, `llvm_unreachable`. Notable symbols here include `getPCRelEncodingSize`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `getPCRelEncodingSize`, `llvm_unreachable`。这里较值得关注的符号包括 `getPCRelEncodingSize`, `llvm_unreachable`。

### Lines 2236-2253

```cpp
  int getShortJmpEncodingSize() const override { return 33; }

  int getUncondBranchEncodingSize() const override { return 28; }

  // This helper function creates the snippet of code that compares a register
  // RegNo with an immediate Imm, and jumps to Target if they are equal.
  // cmp RegNo, #Imm
  // b.eq Target
  // where cmp is an alias for subs, which results in the code below:
  // subs xzr, RegNo, #Imm
  // b.eq Target.
  InstructionListType createCmpJE(MCPhysReg RegNo, int64_t Imm,
                                  const MCSymbol *Target,
                                  MCContext *Ctx) const override {
    InstructionListType Code;
    Code.emplace_back(MCInstBuilder(AArch64::SUBSXri)
                          .addReg(AArch64::XZR)
                          .addReg(RegNo)
```

- EN: Declares or implements routines including `getShortJmpEncodingSize`, `getUncondBranchEncodingSize`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getShortJmpEncodingSize`, `getUncondBranchEncodingSize`.
- CN: 这里声明或实现函数，例如 `getShortJmpEncodingSize`, `getUncondBranchEncodingSize`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getShortJmpEncodingSize`, `getUncondBranchEncodingSize`。

### Lines 2254-2261

```cpp
                          .addImm(Imm)
                          .addImm(0));
    Code.emplace_back(MCInstBuilder(AArch64::Bcc)
                          .addImm(AArch64CC::EQ)
                          .addExpr(MCSymbolRefExpr::create(Target, *Ctx)));
    return Code;
  }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 2262-2279

```cpp
  // This helper function creates the snippet of code that compares a register
  // RegNo with an immediate Imm, and jumps to Target if they are not equal.
  // cmp RegNo, #Imm
  // b.ne Target
  // where cmp is an alias for subs, which results in the code below:
  // subs xzr, RegNo, #Imm
  // b.ne Target.
  InstructionListType createCmpJNE(MCPhysReg RegNo, int64_t Imm,
                                   const MCSymbol *Target,
                                   MCContext *Ctx) const override {
    InstructionListType Code;
    Code.emplace_back(MCInstBuilder(AArch64::SUBSXri)
                          .addReg(AArch64::XZR)
                          .addReg(RegNo)
                          .addImm(Imm)
                          .addImm(0));
    Code.emplace_back(MCInstBuilder(AArch64::Bcc)
                          .addImm(AArch64CC::NE)
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 2280-2288

```cpp
                          .addExpr(MCSymbolRefExpr::create(Target, *Ctx)));
    return Code;
  }

  void createTailCall(MCInst &Inst, const MCSymbol *Target,
                      MCContext *Ctx) override {
    return createDirectCall(Inst, Target, Ctx, /*IsTailCall*/ true);
  }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 2289-2299

```cpp
  void createLongTailCall(InstructionListType &Seq, const MCSymbol *Target,
                          MCContext *Ctx) override {
    createShortJmp(Seq, Target, Ctx, /*IsTailCall*/ true);
  }

  void createTrap(MCInst &Inst) const override {
    Inst.clear();
    Inst.setOpcode(AArch64::BRK);
    Inst.addOperand(MCOperand::createImm(1));
  }
```

- EN: Declares or implements routines including `createShortJmp`, `createTrap`. Notable symbols here include `createShortJmp`, `createTrap`.
- CN: 这里声明或实现函数，例如 `createShortJmp`, `createTrap`。这里较值得关注的符号包括 `createShortJmp`, `createTrap`。

### Lines 2300-2312

```cpp
  bool convertJmpToTailCall(MCInst &Inst) override {
    setTailCall(Inst);
    return true;
  }

  bool convertTailCallToJmp(MCInst &Inst) override {
    removeAnnotation(Inst, MCPlus::MCAnnotation::kTailCall);
    clearOffset(Inst);
    if (getConditionalTailCall(Inst))
      unsetConditionalTailCall(Inst);
    return true;
  }
```

- EN: Declares or implements routines including `convertJmpToTailCall`, `setTailCall`, `convertTailCallToJmp`, `removeAnnotation`, `clearOffset`, and 1 more. Notable symbols here include `convertJmpToTailCall`, `setTailCall`, `convertTailCallToJmp`, `removeAnnotation`, `clearOffset`, `unsetConditionalTailCall`.
- CN: 这里声明或实现函数，例如 `convertJmpToTailCall`, `setTailCall`, `convertTailCallToJmp`, `removeAnnotation`, `clearOffset`, and 1 more。这里较值得关注的符号包括 `convertJmpToTailCall`, `setTailCall`, `convertTailCallToJmp`, `removeAnnotation`, `clearOffset`, `unsetConditionalTailCall`。

### Lines 2313-2320

```cpp
  InstructionListType createIndirectPLTCall(MCInst &&DirectCall,
                                            const MCSymbol *TargetLocation,
                                            MCContext *Ctx) override {
    const bool IsTailCall = isTailCall(DirectCall);
    assert((DirectCall.getOpcode() == AArch64::BL ||
            (DirectCall.getOpcode() == AArch64::B && IsTailCall)) &&
           "64-bit direct (tail) call instruction expected");
```

- EN: Declares or implements routines including `isTailCall`, `assert`, `direct`. Notable symbols here include `isTailCall`, `assert`, `direct`.
- CN: 这里声明或实现函数，例如 `isTailCall`, `assert`, `direct`。这里较值得关注的符号包括 `isTailCall`, `assert`, `direct`。

### Lines 2321-2334

```cpp
    InstructionListType Code;
    // Code sequence for indirect plt call:
    // adrp	x16 <symbol>
    // ldr	x17, [x16, #<offset>]
    // blr	x17  ; or 'br' for tail calls

    MCInst InstAdrp;
    InstAdrp.setOpcode(AArch64::ADRP);
    InstAdrp.addOperand(MCOperand::createReg(AArch64::X16));
    InstAdrp.addOperand(MCOperand::createImm(0));
    setOperandToSymbolRef(InstAdrp, /* OpNum */ 1, TargetLocation,
                          /* Addend */ 0, Ctx, ELF::R_AARCH64_ADR_GOT_PAGE);
    Code.emplace_back(InstAdrp);
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 2335-2343

```cpp
    MCInst InstLoad;
    InstLoad.setOpcode(AArch64::LDRXui);
    InstLoad.addOperand(MCOperand::createReg(AArch64::X17));
    InstLoad.addOperand(MCOperand::createReg(AArch64::X16));
    InstLoad.addOperand(MCOperand::createImm(0));
    setOperandToSymbolRef(InstLoad, /* OpNum */ 2, TargetLocation,
                          /* Addend */ 0, Ctx, ELF::R_AARCH64_LD64_GOT_LO12_NC);
    Code.emplace_back(InstLoad);
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 2344-2352

```cpp
    MCInst InstCall;
    InstCall.setOpcode(IsTailCall ? AArch64::BR : AArch64::BLR);
    InstCall.addOperand(MCOperand::createReg(AArch64::X17));
    moveAnnotations(std::move(DirectCall), InstCall);
    Code.emplace_back(InstCall);

    return Code;
  }
```

- EN: Declares or implements routines including `moveAnnotations`. Notable symbols here include `moveAnnotations`.
- CN: 这里声明或实现函数，例如 `moveAnnotations`。这里较值得关注的符号包括 `moveAnnotations`。

### Lines 2353-2363

```cpp
  bool lowerTailCall(MCInst &Inst) override {
    removeAnnotation(Inst, MCPlus::MCAnnotation::kTailCall);
    if (getConditionalTailCall(Inst))
      unsetConditionalTailCall(Inst);
    return true;
  }

  bool isNoop(const MCInst &Inst) const override {
    return Inst.getOpcode() == AArch64::NOP;
  }
```

- EN: Declares or implements routines including `lowerTailCall`, `removeAnnotation`, `unsetConditionalTailCall`, `isNoop`. Notable symbols here include `lowerTailCall`, `removeAnnotation`, `unsetConditionalTailCall`, `isNoop`.
- CN: 这里声明或实现函数，例如 `lowerTailCall`, `removeAnnotation`, `unsetConditionalTailCall`, `isNoop`。这里较值得关注的符号包括 `lowerTailCall`, `removeAnnotation`, `unsetConditionalTailCall`, `isNoop`。

### Lines 2364-2381

```cpp
  void createNoop(MCInst &Inst) const override {
    Inst.setOpcode(AArch64::NOP);
    Inst.clear();
  }

  bool isTrap(const MCInst &Inst) const override {
    if (Inst.getOpcode() != AArch64::BRK)
      return false;
    // Only match the immediate values that are likely to indicate this BRK
    // instruction is emitted to terminate the program immediately and not to
    // be handled by a SIGTRAP handler, for example.
    switch (Inst.getOperand(0).getImm()) {
    case 0xc470:
    case 0xc471:
    case 0xc472:
    case 0xc473:
      // Explicit Pointer Authentication check failed, see
      // AArch64AsmPrinter::emitPtrauthCheckAuthenticatedValue().
```

- EN: Declares or implements routines including `createNoop`, `isTrap`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `createNoop`, `isTrap`.
- CN: 这里声明或实现函数，例如 `createNoop`, `isTrap`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `createNoop`, `isTrap`。

### Lines 2382-2396

```cpp
      return true;
    case 0x1:
      // __builtin_trap(), as emitted by Clang.
      return true;
    case 0x3e8: // decimal 1000
      // __builtin_trap(), as emitted by GCC.
      return true;
    default:
      // Some constants may indicate intentionally recoverable break-points.
      // This is the case at least for 0xf000, which is used by
      // __builtin_debugtrap() supported by Clang.
      return false;
    }
  }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 2397-2414

```cpp
  bool isStorePair(const MCInst &Inst) const {
    const unsigned opcode = Inst.getOpcode();

    auto isStorePairImmOffset = [&]() {
      switch (opcode) {
      case AArch64::STPWi:
      case AArch64::STPXi:
      case AArch64::STPSi:
      case AArch64::STPDi:
      case AArch64::STPQi:
      case AArch64::STNPWi:
      case AArch64::STNPXi:
      case AArch64::STNPSi:
      case AArch64::STNPDi:
      case AArch64::STNPQi:
        return true;
      default:
        break;
```

- EN: Declares or implements routines including `isStorePair`. Notable symbols here include `isStorePair`.
- CN: 这里声明或实现函数，例如 `isStorePair`。这里较值得关注的符号包括 `isStorePair`。

### Lines 2415-2431

```cpp
      }

      return false;
    };

    auto isStorePairPostIndex = [&]() {
      switch (opcode) {
      case AArch64::STPWpost:
      case AArch64::STPXpost:
      case AArch64::STPSpost:
      case AArch64::STPDpost:
      case AArch64::STPQpost:
        return true;
      default:
        break;
      }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 2432-2446

```cpp
      return false;
    };

    auto isStorePairPreIndex = [&]() {
      switch (opcode) {
      case AArch64::STPWpre:
      case AArch64::STPXpre:
      case AArch64::STPSpre:
      case AArch64::STPDpre:
      case AArch64::STPQpre:
        return true;
      default:
        break;
      }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 2447-2456

```cpp
      return false;
    };

    return isStorePairImmOffset() || isStorePairPostIndex() ||
           isStorePairPreIndex();
  }

  bool isStoreReg(const MCInst &Inst) const {
    const unsigned opcode = Inst.getOpcode();
```

- EN: Declares or implements routines including `isStorePairPreIndex`, `isStoreReg`. Notable symbols here include `isStorePairPreIndex`, `isStoreReg`.
- CN: 这里声明或实现函数，例如 `isStorePairPreIndex`, `isStoreReg`。这里较值得关注的符号包括 `isStorePairPreIndex`, `isStoreReg`。

### Lines 2457-2472

```cpp
    auto isStoreRegUnscaleImm = [&]() {
      switch (opcode) {
      case AArch64::STURBi:
      case AArch64::STURBBi:
      case AArch64::STURHi:
      case AArch64::STURHHi:
      case AArch64::STURWi:
      case AArch64::STURXi:
      case AArch64::STURSi:
      case AArch64::STURDi:
      case AArch64::STURQi:
        return true;
      default:
        break;
      }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 2473-2490

```cpp
      return false;
    };

    auto isStoreRegScaledImm = [&]() {
      switch (opcode) {
      case AArch64::STRBui:
      case AArch64::STRBBui:
      case AArch64::STRHui:
      case AArch64::STRHHui:
      case AArch64::STRWui:
      case AArch64::STRXui:
      case AArch64::STRSui:
      case AArch64::STRDui:
      case AArch64::STRQui:
        return true;
      default:
        break;
      }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 2491-2508

```cpp

      return false;
    };

    auto isStoreRegImmPostIndexed = [&]() {
      switch (opcode) {
      case AArch64::STRBpost:
      case AArch64::STRBBpost:
      case AArch64::STRHpost:
      case AArch64::STRHHpost:
      case AArch64::STRWpost:
      case AArch64::STRXpost:
      case AArch64::STRSpost:
      case AArch64::STRDpost:
      case AArch64::STRQpost:
        return true;
      default:
        break;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 2509-2526

```cpp
      }

      return false;
    };

    auto isStoreRegImmPreIndexed = [&]() {
      switch (opcode) {
      case AArch64::STRBpre:
      case AArch64::STRBBpre:
      case AArch64::STRHpre:
      case AArch64::STRHHpre:
      case AArch64::STRWpre:
      case AArch64::STRXpre:
      case AArch64::STRSpre:
      case AArch64::STRDpre:
      case AArch64::STRQpre:
        return true;
      default:
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 2527-2543

```cpp
        break;
      }

      return false;
    };

    auto isStoreRegUnscaleUnpriv = [&]() {
      switch (opcode) {
      case AArch64::STTRBi:
      case AArch64::STTRHi:
      case AArch64::STTRWi:
      case AArch64::STTRXi:
        return true;
      default:
        break;
      }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 2544-2561

```cpp
      return false;
    };

    auto isStoreRegTrunc = [&]() {
      switch (opcode) {
      case AArch64::STRBBroW:
      case AArch64::STRBBroX:
      case AArch64::STRBroW:
      case AArch64::STRBroX:
      case AArch64::STRDroW:
      case AArch64::STRDroX:
      case AArch64::STRHHroW:
      case AArch64::STRHHroX:
      case AArch64::STRHroW:
      case AArch64::STRHroX:
      case AArch64::STRQroW:
      case AArch64::STRQroX:
      case AArch64::STRSroW:
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 2562-2571

```cpp
      case AArch64::STRSroX:
      case AArch64::STRWroW:
      case AArch64::STRWroX:
      case AArch64::STRXroW:
      case AArch64::STRXroX:
        return true;
      default:
        break;
      }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 2572-2579

```cpp
      return false;
    };

    return isStoreRegUnscaleImm() || isStoreRegScaledImm() ||
           isStoreRegImmPreIndexed() || isStoreRegImmPostIndexed() ||
           isStoreRegUnscaleUnpriv() || isStoreRegTrunc();
  }
```

- EN: Declares or implements routines including `isStoreRegImmPreIndexed`, `isStoreRegUnscaleUnpriv`. Notable symbols here include `isStoreRegImmPreIndexed`, `isStoreRegUnscaleUnpriv`.
- CN: 这里声明或实现函数，例如 `isStoreRegImmPreIndexed`, `isStoreRegUnscaleUnpriv`。这里较值得关注的符号包括 `isStoreRegImmPreIndexed`, `isStoreRegUnscaleUnpriv`。

### Lines 2580-2597

```cpp
  bool mayStore(const MCInst &Inst) const override {
    return isStorePair(Inst) || isStoreReg(Inst) ||
           isAArch64ExclusiveStore(Inst);
  }

  bool isCleanRegXOR(const MCInst &Inst) const override {
    switch (Inst.getOpcode()) {
    case AArch64::EORXrs:
    case AArch64::EORWrs:
      return Inst.getOperand(1).getReg() == Inst.getOperand(2).getReg() &&
             Inst.getOperand(3).getImm() == 0;
    case AArch64::ORRXrs:
      return Inst.getOperand(1).getReg() == AArch64::XZR &&
             Inst.getOperand(2).getReg() == AArch64::XZR &&
             Inst.getOperand(3).getImm() == 0;
    case AArch64::ORRWrs:
      return Inst.getOperand(1).getReg() == AArch64::WZR &&
             Inst.getOperand(2).getReg() == AArch64::WZR &&
```

- EN: Declares or implements routines including `mayStore`, `isAArch64ExclusiveStore`, `isCleanRegXOR`. Notable symbols here include `mayStore`, `isAArch64ExclusiveStore`, `isCleanRegXOR`.
- CN: 这里声明或实现函数，例如 `mayStore`, `isAArch64ExclusiveStore`, `isCleanRegXOR`。这里较值得关注的符号包括 `mayStore`, `isAArch64ExclusiveStore`, `isCleanRegXOR`。

### Lines 2598-2607

```cpp
             Inst.getOperand(3).getImm() == 0;
    case AArch64::MOVZXi:
    case AArch64::MOVZWi:
      return Inst.getOperand(1).isImm() && Inst.getOperand(1).getImm() == 0 &&
             Inst.getOperand(2).getImm() == 0;
    default:
      return false;
    }
  }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 2608-2615

```cpp
  bool isStoreToStack(const MCInst &Inst) const {
    if (!mayStore(Inst))
      return false;

    for (const MCOperand &Operand : useOperands(Inst)) {
      if (!Operand.isReg())
        continue;
```

- EN: Declares or implements routines including `isStoreToStack`. Notable symbols here include `isStoreToStack`.
- CN: 这里声明或实现函数，例如 `isStoreToStack`。这里较值得关注的符号包括 `isStoreToStack`。

### Lines 2616-2623

```cpp
      unsigned Reg = Operand.getReg();
      if (Reg == AArch64::SP || Reg == AArch64::WSP)
        return true;
    }

    return false;
  }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 2624-2633

```cpp
  void createDirectCall(MCInst &Inst, const MCSymbol *Target, MCContext *Ctx,
                        bool IsTailCall) override {
    Inst.setOpcode(IsTailCall ? AArch64::B : AArch64::BL);
    Inst.clear();
    Inst.addOperand(MCOperand::createExpr(getTargetExprFor(
        Inst, MCSymbolRefExpr::create(Target, *Ctx), *Ctx, 0)));
    if (IsTailCall)
      convertJmpToTailCall(Inst);
  }
```

- EN: Declares or implements routines including `create`, `convertJmpToTailCall`. Notable symbols here include `create`, `convertJmpToTailCall`.
- CN: 这里声明或实现函数，例如 `create`, `convertJmpToTailCall`。这里较值得关注的符号包括 `create`, `convertJmpToTailCall`。

### Lines 2634-2645

```cpp
  bool isShortRangeBranch(const MCInst &Inst) const override {
    return isCompAndBranch(Inst);
  }

  void createDirectBranch(MCInst &Inst, const MCSymbol *Target,
                          MCContext *Ctx) override {
    Inst.setOpcode(AArch64::B);
    Inst.clear();
    Inst.addOperand(MCOperand::createExpr(getTargetExprFor(
        Inst, MCSymbolRefExpr::create(Target, *Ctx), *Ctx, 0)));
  }
```

- EN: Declares or implements routines including `isShortRangeBranch`, `create`. Notable symbols here include `isShortRangeBranch`, `create`.
- CN: 这里声明或实现函数，例如 `isShortRangeBranch`, `create`。这里较值得关注的符号包括 `isShortRangeBranch`, `create`。

### Lines 2646-2654

```cpp
  bool analyzeBranch(InstructionIterator Begin, InstructionIterator End,
                     const MCSymbol *&TBB, const MCSymbol *&FBB,
                     MCInst *&CondBranch,
                     MCInst *&UncondBranch) const override {
    auto I = End;

    while (I != Begin) {
      --I;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 2655-2662

```cpp
      // Ignore nops and CFIs
      if (isPseudo(*I) || isNoop(*I))
        continue;

      // Stop when we find the first non-terminator
      if (!isTerminator(*I) || isTailCall(*I) || !isBranch(*I))
        break;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 2663-2675

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

### Lines 2676-2691

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

### Lines 2692-2709

```cpp
      llvm_unreachable("multiple conditional branches in one BB");
    }
    return true;
  }

  void createLongJmp(InstructionListType &Seq, const MCSymbol *Target,
                     MCContext *Ctx, bool IsTailCall) override {
    // ip0 (r16) is reserved to the linker (refer to 5.3.1.1 of "Procedure Call
    //   Standard for the ARM 64-bit Architecture (AArch64)".
    // The sequence of instructions we create here is the following:
    //  movz ip0, #:abs_g3:<addr>
    //  movk ip0, #:abs_g2_nc:<addr>
    //  movk ip0, #:abs_g1_nc:<addr>
    //  movk ip0, #:abs_g0_nc:<addr>
    //  br ip0
    MCInst Inst;
    Inst.setOpcode(AArch64::MOVZXi);
    Inst.addOperand(MCOperand::createReg(AArch64::X16));
```

- EN: Declares or implements routines including `llvm_unreachable`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `llvm_unreachable`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `llvm_unreachable`。

### Lines 2710-2723

```cpp
    Inst.addOperand(MCOperand::createExpr(
        MCSpecifierExpr::create(Target, AArch64::S_ABS_G3, *Ctx)));
    Inst.addOperand(MCOperand::createImm(0x30));
    Seq.emplace_back(Inst);

    Inst.clear();
    Inst.setOpcode(AArch64::MOVKXi);
    Inst.addOperand(MCOperand::createReg(AArch64::X16));
    Inst.addOperand(MCOperand::createReg(AArch64::X16));
    Inst.addOperand(MCOperand::createExpr(
        MCSpecifierExpr::create(Target, AArch64::S_ABS_G2_NC, *Ctx)));
    Inst.addOperand(MCOperand::createImm(0x20));
    Seq.emplace_back(Inst);
```

- EN: Declares or implements routines including `create`. Notable symbols here include `create`.
- CN: 这里声明或实现函数，例如 `create`。这里较值得关注的符号包括 `create`。

### Lines 2724-2732

```cpp
    Inst.clear();
    Inst.setOpcode(AArch64::MOVKXi);
    Inst.addOperand(MCOperand::createReg(AArch64::X16));
    Inst.addOperand(MCOperand::createReg(AArch64::X16));
    Inst.addOperand(MCOperand::createExpr(
        MCSpecifierExpr::create(Target, AArch64::S_ABS_G1_NC, *Ctx)));
    Inst.addOperand(MCOperand::createImm(0x10));
    Seq.emplace_back(Inst);
```

- EN: Declares or implements routines including `create`. Notable symbols here include `create`.
- CN: 这里声明或实现函数，例如 `create`。这里较值得关注的符号包括 `create`。

### Lines 2733-2741

```cpp
    Inst.clear();
    Inst.setOpcode(AArch64::MOVKXi);
    Inst.addOperand(MCOperand::createReg(AArch64::X16));
    Inst.addOperand(MCOperand::createReg(AArch64::X16));
    Inst.addOperand(MCOperand::createExpr(
        MCSpecifierExpr::create(Target, AArch64::S_ABS_G0_NC, *Ctx)));
    Inst.addOperand(MCOperand::createImm(0));
    Seq.emplace_back(Inst);
```

- EN: Declares or implements routines including `create`. Notable symbols here include `create`.
- CN: 这里声明或实现函数，例如 `create`。这里较值得关注的符号包括 `create`。

### Lines 2742-2749

```cpp
    Inst.clear();
    Inst.setOpcode(AArch64::BR);
    Inst.addOperand(MCOperand::createReg(AArch64::X16));
    if (IsTailCall)
      setTailCall(Inst);
    Seq.emplace_back(Inst);
  }
```

- EN: Declares or implements routines including `setTailCall`. Notable symbols here include `setTailCall`.
- CN: 这里声明或实现函数，例如 `setTailCall`。这里较值得关注的符号包括 `setTailCall`。

### Lines 2750-2767

```cpp
  void createShortJmp(InstructionListType &Seq, const MCSymbol *Target,
                      MCContext *Ctx, bool IsTailCall) override {
    // ip0 (r16) is reserved to the linker (refer to 5.3.1.1 of "Procedure Call
    //   Standard for the ARM 64-bit Architecture (AArch64)".
    // The sequence of instructions we create here is the following:
    //  adrp ip0, imm
    //  add ip0, ip0, imm
    //  br ip0
    MCPhysReg Reg = AArch64::X16;
    InstructionListType Insts = materializeAddress(Target, Ctx, Reg);
    Insts.emplace_back();
    MCInst &Inst = Insts.back();
    Inst.clear();
    Inst.setOpcode(AArch64::BR);
    Inst.addOperand(MCOperand::createReg(Reg));
    if (IsTailCall)
      setTailCall(Inst);
    Seq.swap(Insts);
```

- EN: Declares or implements routines including `materializeAddress`, `setTailCall`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `materializeAddress`, `setTailCall`.
- CN: 这里声明或实现函数，例如 `materializeAddress`, `setTailCall`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `materializeAddress`, `setTailCall`。

### Lines 2768-2783

```cpp
  }

  /// Matching pattern here is
  ///
  ///    ADRP  x16, imm
  ///    ADD   x16, x16, imm
  ///    BR    x16
  ///
  uint64_t matchLinkerVeneer(InstructionIterator Begin, InstructionIterator End,
                             uint64_t Address, const MCInst &CurInst,
                             MCInst *&TargetHiBits, MCInst *&TargetLowBits,
                             uint64_t &Target) const override {
    if (CurInst.getOpcode() != AArch64::BR || !CurInst.getOperand(0).isReg() ||
        CurInst.getOperand(0).getReg() != AArch64::X16)
      return 0;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 2784-2798

```cpp
    auto I = End;
    if (I == Begin)
      return 0;

    --I;
    Address -= 4;
    if (I == Begin || I->getOpcode() != AArch64::ADDXri ||
        MCPlus::getNumPrimeOperands(*I) < 3 || !I->getOperand(0).isReg() ||
        !I->getOperand(1).isReg() ||
        I->getOperand(0).getReg() != AArch64::X16 ||
        I->getOperand(1).getReg() != AArch64::X16 || !I->getOperand(2).isImm())
      return 0;
    TargetLowBits = &*I;
    uint64_t Addr = I->getOperand(2).getImm() & 0xFFF;
```

- EN: Declares or implements routines including `getNumPrimeOperands`, `getOperand`. Notable symbols here include `getNumPrimeOperands`, `getOperand`.
- CN: 这里声明或实现函数，例如 `getNumPrimeOperands`, `getOperand`。这里较值得关注的符号包括 `getNumPrimeOperands`, `getOperand`。

### Lines 2799-2811

```cpp
    --I;
    Address -= 4;
    if (I != Begin || I->getOpcode() != AArch64::ADRP ||
        MCPlus::getNumPrimeOperands(*I) < 2 || !I->getOperand(0).isReg() ||
        !I->getOperand(1).isImm() || I->getOperand(0).getReg() != AArch64::X16)
      return 0;
    TargetHiBits = &*I;
    Addr |= (Address + ((uint64_t)I->getOperand(1).getImm() << 12)) &
            0xFFFFFFFFFFFFF000ULL;
    Target = Addr;
    return 3;
  }
```

- EN: Declares or implements routines including `getNumPrimeOperands`, `getOperand`. Notable symbols here include `getNumPrimeOperands`, `getOperand`.
- CN: 这里声明或实现函数，例如 `getNumPrimeOperands`, `getOperand`。这里较值得关注的符号包括 `getNumPrimeOperands`, `getOperand`。

### Lines 2812-2824

```cpp
  /// Match the following pattern:
  ///
  ///   LDR x16, .L1
  ///   BR  x16
  /// L1:
  ///   .quad Target
  ///
  /// Populate \p TargetAddress with the Target value on successful match.
  bool matchAbsLongVeneer(const BinaryFunction &BF,
                          uint64_t &TargetAddress) const override {
    if (BF.size() != 1 || BF.getMaxSize() < 16)
      return false;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 2825-2835

```cpp
    if (!BF.hasConstantIsland())
      return false;

    const BinaryBasicBlock &BB = BF.front();
    if (BB.size() != 2)
      return false;

    const MCInst &LDRInst = BB.getInstructionAtIndex(0);
    if (LDRInst.getOpcode() != AArch64::LDRXl)
      return false;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 2836-2843

```cpp
    if (!LDRInst.getOperand(0).isReg() ||
        LDRInst.getOperand(0).getReg() != AArch64::X16)
      return false;

    const MCSymbol *TargetSym = getTargetSymbol(LDRInst, 1);
    if (!TargetSym)
      return false;
```

- EN: Declares or implements routines including `getTargetSymbol`. Notable symbols here include `getTargetSymbol`.
- CN: 这里声明或实现函数，例如 `getTargetSymbol`。这里较值得关注的符号包括 `getTargetSymbol`。

### Lines 2844-2854

```cpp
    const MCInst &BRInst = BB.getInstructionAtIndex(1);
    if (BRInst.getOpcode() != AArch64::BR)
      return false;
    if (!BRInst.getOperand(0).isReg() ||
        BRInst.getOperand(0).getReg() != AArch64::X16)
      return false;

    const BinaryFunction::IslandInfo &IInfo = BF.getIslandInfo();
    if (IInfo.HasDynamicRelocations)
      return false;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 2855-2863

```cpp
    auto Iter = IInfo.Offsets.find(8);
    if (Iter == IInfo.Offsets.end() || Iter->second != TargetSym)
      return false;

    // Extract the absolute value stored inside the island.
    StringRef SectionContents = BF.getOriginSection()->getContents();
    StringRef FunctionContents = SectionContents.substr(
        BF.getAddress() - BF.getOriginSection()->getAddress(), BF.getMaxSize());
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 2864-2872

```cpp
    const BinaryContext &BC = BF.getBinaryContext();
    unsigned CodePointerSize = BC.AsmInfo->getCodePointerSize();
    DataExtractor DE(FunctionContents, BC.AsmInfo->isLittleEndian());
    uint64_t Offset = 8;
    TargetAddress = DE.getUnsigned(&Offset, CodePointerSize);

    return true;
  }
```

- EN: Declares or implements routines including `getCodePointerSize`, `DE`. Notable symbols here include `getCodePointerSize`, `DE`.
- CN: 这里声明或实现函数，例如 `getCodePointerSize`, `DE`。这里较值得关注的符号包括 `getCodePointerSize`, `DE`。

### Lines 2873-2885

```cpp
  bool matchAdrpAddPair(const MCInst &Adrp, const MCInst &Add) const override {
    if (!isADRP(Adrp) || !isAddXri(Add))
      return false;

    assert(Adrp.getOperand(0).isReg() &&
           "Unexpected operand in ADRP instruction");
    MCPhysReg AdrpReg = Adrp.getOperand(0).getReg();
    assert(Add.getOperand(1).isReg() &&
           "Unexpected operand in ADDXri instruction");
    MCPhysReg AddReg = Add.getOperand(1).getReg();
    return AdrpReg == AddReg;
  }
```

- EN: Declares or implements routines including `matchAdrpAddPair`, `assert`. Notable symbols here include `matchAdrpAddPair`, `assert`.
- CN: 这里声明或实现函数，例如 `matchAdrpAddPair`, `assert`。这里较值得关注的符号包括 `matchAdrpAddPair`, `assert`。

### Lines 2886-2899

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
    if (ImmOpNo == -1U)
      return false;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 2900-2914

```cpp
    Value = Inst.getOperand(ImmOpNo).getImm();

    setOperandToSymbolRef(Inst, ImmOpNo, Symbol, Addend, Ctx, RelType);

    return true;
  }

  void createUncondBranch(MCInst &Inst, const MCSymbol *TBB,
                          MCContext *Ctx) const override {
    Inst.setOpcode(AArch64::B);
    Inst.clear();
    Inst.addOperand(MCOperand::createExpr(
        getTargetExprFor(Inst, MCSymbolRefExpr::create(TBB, *Ctx), *Ctx, 0)));
  }
```

- EN: Declares or implements routines including `setOperandToSymbolRef`, `getTargetExprFor`. Notable symbols here include `setOperandToSymbolRef`, `getTargetExprFor`.
- CN: 这里声明或实现函数，例如 `setOperandToSymbolRef`, `getTargetExprFor`。这里较值得关注的符号包括 `setOperandToSymbolRef`, `getTargetExprFor`。

### Lines 2915-2932

```cpp
  bool shouldRecordCodeRelocation(uint32_t RelType) const override {
    switch (RelType) {
    case ELF::R_AARCH64_ABS64:
    case ELF::R_AARCH64_ABS32:
    case ELF::R_AARCH64_ABS16:
    case ELF::R_AARCH64_ADD_ABS_LO12_NC:
    case ELF::R_AARCH64_ADR_GOT_PAGE:
    case ELF::R_AARCH64_ADR_PREL_LO21:
    case ELF::R_AARCH64_ADR_PREL_PG_HI21:
    case ELF::R_AARCH64_ADR_PREL_PG_HI21_NC:
    case ELF::R_AARCH64_LD64_GOT_LO12_NC:
    case ELF::R_AARCH64_LDST8_ABS_LO12_NC:
    case ELF::R_AARCH64_LDST16_ABS_LO12_NC:
    case ELF::R_AARCH64_LDST32_ABS_LO12_NC:
    case ELF::R_AARCH64_LDST64_ABS_LO12_NC:
    case ELF::R_AARCH64_LDST128_ABS_LO12_NC:
    case ELF::R_AARCH64_TLSDESC_ADD_LO12:
    case ELF::R_AARCH64_TLSDESC_ADR_PAGE21:
```

- EN: Declares or implements routines including `shouldRecordCodeRelocation`. Notable symbols here include `shouldRecordCodeRelocation`.
- CN: 这里声明或实现函数，例如 `shouldRecordCodeRelocation`。这里较值得关注的符号包括 `shouldRecordCodeRelocation`。

### Lines 2933-2950

```cpp
    case ELF::R_AARCH64_TLSDESC_ADR_PREL21:
    case ELF::R_AARCH64_TLSDESC_LD64_LO12:
    case ELF::R_AARCH64_TLSIE_ADR_GOTTPREL_PAGE21:
    case ELF::R_AARCH64_TLSIE_LD64_GOTTPREL_LO12_NC:
    case ELF::R_AARCH64_TLSLE_MOVW_TPREL_G0:
    case ELF::R_AARCH64_TLSLE_MOVW_TPREL_G0_NC:
    case ELF::R_AARCH64_MOVW_UABS_G0:
    case ELF::R_AARCH64_MOVW_UABS_G0_NC:
    case ELF::R_AARCH64_MOVW_UABS_G1:
    case ELF::R_AARCH64_MOVW_UABS_G1_NC:
    case ELF::R_AARCH64_MOVW_UABS_G2:
    case ELF::R_AARCH64_MOVW_UABS_G2_NC:
    case ELF::R_AARCH64_MOVW_UABS_G3:
    case ELF::R_AARCH64_PREL16:
    case ELF::R_AARCH64_PREL32:
    case ELF::R_AARCH64_PREL64:
      return true;
    case ELF::R_AARCH64_CALL26:
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 2951-2962

```cpp
    case ELF::R_AARCH64_JUMP26:
    case ELF::R_AARCH64_TSTBR14:
    case ELF::R_AARCH64_CONDBR19:
    case ELF::R_AARCH64_TLSDESC_CALL:
    case ELF::R_AARCH64_TLSLE_ADD_TPREL_HI12:
    case ELF::R_AARCH64_TLSLE_ADD_TPREL_LO12_NC:
      return false;
    default:
      llvm_unreachable("Unexpected AArch64 relocation type in code");
    }
  }
```

- EN: Declares or implements routines including `llvm_unreachable`. Notable symbols here include `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `llvm_unreachable`。这里较值得关注的符号包括 `llvm_unreachable`。

### Lines 2963-2972

```cpp
  StringRef getTrapFillValue() const override {
    return StringRef("\0\0\0\0", 4);
  }

  void createReturn(MCInst &Inst) const override {
    Inst.setOpcode(AArch64::RET);
    Inst.clear();
    Inst.addOperand(MCOperand::createReg(AArch64::LR));
  }
```

- EN: Declares or implements routines including `getTrapFillValue`, `createReturn`. Notable symbols here include `getTrapFillValue`, `createReturn`.
- CN: 这里声明或实现函数，例如 `getTrapFillValue`, `createReturn`。这里较值得关注的符号包括 `getTrapFillValue`, `createReturn`。

### Lines 2973-2983

```cpp
  void createStackPointerIncrement(
      MCInst &Inst, int Size,
      bool NoFlagsClobber = false /*unused for AArch64*/) const override {
    Inst.setOpcode(AArch64::SUBXri);
    Inst.clear();
    Inst.addOperand(MCOperand::createReg(AArch64::SP));
    Inst.addOperand(MCOperand::createReg(AArch64::SP));
    Inst.addOperand(MCOperand::createImm(Size));
    Inst.addOperand(MCOperand::createImm(0));
  }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 2984-2994

```cpp
  void createStackPointerDecrement(
      MCInst &Inst, int Size,
      bool NoFlagsClobber = false /*unused for AArch64*/) const override {
    Inst.setOpcode(AArch64::ADDXri);
    Inst.clear();
    Inst.addOperand(MCOperand::createReg(AArch64::SP));
    Inst.addOperand(MCOperand::createReg(AArch64::SP));
    Inst.addOperand(MCOperand::createImm(Size));
    Inst.addOperand(MCOperand::createImm(0));
  }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 2995-3005

```cpp
  void createIndirectBranch(MCInst &Inst, MCPhysReg MemBaseReg,
                            int64_t Disp) const {
    Inst.setOpcode(AArch64::BR);
    Inst.clear();
    Inst.addOperand(MCOperand::createReg(MemBaseReg));
  }

  InstructionListType createInstrumentedIndCallHandlerExitBB() const override {
    // Code sequence for instrumented indirect call handler:
    //   ret
```

- EN: Declares or implements routines including `createInstrumentedIndCallHandlerExitBB`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `createInstrumentedIndCallHandlerExitBB`.
- CN: 这里声明或实现函数，例如 `createInstrumentedIndCallHandlerExitBB`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `createInstrumentedIndCallHandlerExitBB`。

### Lines 3006-3013

```cpp
    InstructionListType Insts;

    Insts.emplace_back();
    createReturn(Insts.back());

    return Insts;
  }
```

- EN: Declares or implements routines including `createReturn`. Notable symbols here include `createReturn`.
- CN: 这里声明或实现函数，例如 `createReturn`。这里较值得关注的符号包括 `createReturn`。

### Lines 3014-3029

```cpp
  InstructionListType
  createInstrumentedIndTailCallHandlerExitBB() const override {
    return createInstrumentedIndCallHandlerExitBB();
  }

  InstructionListType createGetter(MCContext *Ctx, const char *name) const {
    InstructionListType Insts(4);
    MCSymbol *Locs = Ctx->getOrCreateSymbol(name);
    InstructionListType Addr = materializeAddress(Locs, Ctx, AArch64::X0);
    std::copy(Addr.begin(), Addr.end(), Insts.begin());
    assert(Addr.size() == 2 && "Invalid Addr size");
    loadReg(Insts[2], AArch64::X0, AArch64::X0);
    createReturn(Insts[3]);
    return Insts;
  }
```

- EN: Declares or implements routines including `createInstrumentedIndTailCallHandlerExitBB`, `createGetter`, `Insts`, `getOrCreateSymbol`, `materializeAddress`, and 4 more. Notable symbols here include `createInstrumentedIndTailCallHandlerExitBB`, `createGetter`, `Insts`, `getOrCreateSymbol`, `materializeAddress`, `copy`.
- CN: 这里声明或实现函数，例如 `createInstrumentedIndTailCallHandlerExitBB`, `createGetter`, `Insts`, `getOrCreateSymbol`, `materializeAddress`, and 4 more。这里较值得关注的符号包括 `createInstrumentedIndTailCallHandlerExitBB`, `createGetter`, `Insts`, `getOrCreateSymbol`, `materializeAddress`, `copy`。

### Lines 3030-3038

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

### Lines 3039-3046

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

### Lines 3047-3060

```cpp
  void convertIndirectCallToLoad(MCInst &Inst, MCPhysReg Reg) override {
    bool IsTailCall = isTailCall(Inst);
    if (IsTailCall)
      removeAnnotation(Inst, MCPlus::MCAnnotation::kTailCall);
    if (Inst.getOpcode() == AArch64::BR || Inst.getOpcode() == AArch64::BLR) {
      Inst.setOpcode(AArch64::ORRXrs);
      Inst.insert(Inst.begin(), MCOperand::createReg(Reg));
      Inst.insert(Inst.begin() + 1, MCOperand::createReg(AArch64::XZR));
      Inst.insert(Inst.begin() + 3, MCOperand::createImm(0));
      return;
    }
    llvm_unreachable("not implemented");
  }
```

- EN: Declares or implements routines including `convertIndirectCallToLoad`, `isTailCall`, `removeAnnotation`, `llvm_unreachable`. Notable symbols here include `convertIndirectCallToLoad`, `isTailCall`, `removeAnnotation`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `convertIndirectCallToLoad`, `isTailCall`, `removeAnnotation`, `llvm_unreachable`。这里较值得关注的符号包括 `convertIndirectCallToLoad`, `isTailCall`, `removeAnnotation`, `llvm_unreachable`。

### Lines 3061-3069

```cpp
  InstructionListType createLoadImmediate(const MCPhysReg Dest,
                                          uint64_t Imm) const override {
    if (RegInfo->getRegClass(AArch64::GPR64RegClassID).contains(Dest))
      return createMOVImm(Dest, 64, Imm);
    if (RegInfo->getRegClass(AArch64::GPR32RegClassID).contains(Dest))
      return createMOVImm(Dest, 32, Imm);
    llvm_unreachable("Unexpected RegClass");
  }
```

- EN: Declares or implements routines including `llvm_unreachable`. Notable symbols here include `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `llvm_unreachable`。这里较值得关注的符号包括 `llvm_unreachable`。

### Lines 3070-3087

```cpp
  void createIndirectCallInst(MCInst &Inst, bool IsTailCall,
                              MCPhysReg Reg) const {
    Inst.clear();
    Inst.setOpcode(IsTailCall ? AArch64::BR : AArch64::BLR);
    Inst.addOperand(MCOperand::createReg(Reg));
  }

  InstructionListType createInstrumentedIndirectCall(MCInst &&CallInst,
                                                     MCSymbol *HandlerFuncAddr,
                                                     size_t CallSiteID,
                                                     MCContext *Ctx) override {
    // Code sequence used to enter indirect call instrumentation helper:
    // snippet requires 2 registers: target address and call site id
    //   stp CallIDReg, x30, [sp, #-16]!
    //   movz/k CallIDReg, CallSiteID
    //   stp TAReg, CallIDReg, [sp, #-16]! ; push address and id for lib
    //   adr + add TAReg, *HandlerFuncAddr ; __bolt_instr_ind_call_handler_func
    //   blr TAReg
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 3088-3095

```cpp
    //   ldr TAReg, [sp], #16 ; restore target address
    //   ldp CallIDReg, x30, [sp], #16
    //   blr TAReg

    const MCRegister TAReg = CallInst.getOperand(0).getReg();
    const MCRegister CallIDReg =
        TAReg != AArch64::X0 ? AArch64::X0 : AArch64::X1;
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 3096-3105

```cpp
    InstructionListType Insts;
    Insts.emplace_back();
    createPushRegisters(Insts.back(), CallIDReg, AArch64::LR);

    InstructionListType LoadImm = createLoadImmediate(CallIDReg, CallSiteID);
    Insts.insert(Insts.end(), LoadImm.begin(), LoadImm.end());

    Insts.emplace_back();
    createPushRegisters(Insts.back(), TAReg, CallIDReg);
```

- EN: Declares or implements routines including `createPushRegisters`, `createLoadImmediate`. Notable symbols here include `createPushRegisters`, `createLoadImmediate`.
- CN: 这里声明或实现函数，例如 `createPushRegisters`, `createLoadImmediate`。这里较值得关注的符号包括 `createPushRegisters`, `createLoadImmediate`。

### Lines 3106-3113

```cpp
    Insts.resize(Insts.size() + 2);
    InstructionListType Addr = materializeAddress(HandlerFuncAddr, Ctx, TAReg);
    assert(Addr.size() == 2 && "Invalid Addr size");
    std::copy(Addr.begin(), Addr.end(), Insts.end() - Addr.size());

    Insts.emplace_back();
    createIndirectCallInst(Insts.back(), false, TAReg);
```

- EN: Declares or implements routines including `materializeAddress`, `assert`, `copy`, `createIndirectCallInst`. Notable symbols here include `materializeAddress`, `assert`, `copy`, `createIndirectCallInst`.
- CN: 这里声明或实现函数，例如 `materializeAddress`, `assert`, `copy`, `createIndirectCallInst`。这里较值得关注的符号包括 `materializeAddress`, `assert`, `copy`, `createIndirectCallInst`。

### Lines 3114-3121

```cpp
    Insts.emplace_back();
    loadReg(Insts.back(), TAReg, getStackPointer());

    Insts.emplace_back();
    createPopRegisters(Insts.back(), CallIDReg, AArch64::LR);

    Insts.emplace_back(CallInst);
```

- EN: Declares or implements routines including `loadReg`, `createPopRegisters`. Notable symbols here include `loadReg`, `createPopRegisters`.
- CN: 这里声明或实现函数，例如 `loadReg`, `createPopRegisters`。这里较值得关注的符号包括 `loadReg`, `createPopRegisters`。

### Lines 3122-3139

```cpp
    return Insts;
  }

  InstructionListType
  createInstrumentedIndCallHandlerEntryBB(const MCSymbol *InstrTrampoline,
                                          const MCSymbol *IndCallHandler,
                                          MCContext *Ctx) override {
    // Code sequence used to check whether InstrTrampoline was initialized
    // and call it if so, returns via IndCallHandler
    //   adrp    x0, InstrTrampoline
    //   ldr     x0, [x0, #lo12:InstrTrampoline]
    //   subs    x0, x0, #0x0
    //   b.eq    IndCallHandler
    //   str     x30, [sp, #-16]!
    //   blr     x0
    //   ldr     x30, [sp], #16
    //   b       IndCallHandler
    InstructionListType Insts;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 3140-3149

```cpp

    // load handler address
    MCInst InstAdrp;
    InstAdrp.setOpcode(AArch64::ADRP);
    InstAdrp.addOperand(MCOperand::createReg(getIntArgRegister(0)));
    InstAdrp.addOperand(MCOperand::createImm(0));
    setOperandToSymbolRef(InstAdrp, /* OpNum */ 1, InstrTrampoline,
                          /* Addend */ 0, Ctx, ELF::R_AARCH64_ADR_GOT_PAGE);
    Insts.emplace_back(InstAdrp);
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 3150-3158

```cpp
    MCInst InstLoad;
    InstLoad.setOpcode(AArch64::LDRXui);
    InstLoad.addOperand(MCOperand::createReg(getIntArgRegister(0)));
    InstLoad.addOperand(MCOperand::createReg(getIntArgRegister(0)));
    InstLoad.addOperand(MCOperand::createImm(0));
    setOperandToSymbolRef(InstLoad, /* OpNum */ 2, InstrTrampoline,
                          /* Addend */ 0, Ctx, ELF::R_AARCH64_LD64_GOT_LO12_NC);
    Insts.emplace_back(InstLoad);
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 3159-3169

```cpp
    InstructionListType CmpJmp =
        createCmpJE(getIntArgRegister(0), 0, IndCallHandler, Ctx);
    Insts.insert(Insts.end(), CmpJmp.begin(), CmpJmp.end());

    Insts.emplace_back();
    storeReg(Insts.back(), AArch64::LR, getStackPointer());

    Insts.emplace_back();
    Insts.back().setOpcode(AArch64::BLR);
    Insts.back().addOperand(MCOperand::createReg(getIntArgRegister(0)));
```

- EN: Declares or implements routines including `createCmpJE`, `storeReg`. Notable symbols here include `createCmpJE`, `storeReg`.
- CN: 这里声明或实现函数，例如 `createCmpJE`, `storeReg`。这里较值得关注的符号包括 `createCmpJE`, `storeReg`。

### Lines 3170-3178

```cpp
    Insts.emplace_back();
    loadReg(Insts.back(), AArch64::LR, getStackPointer());

    Insts.emplace_back();
    createDirectBranch(Insts.back(), IndCallHandler, Ctx);

    return Insts;
  }
```

- EN: Declares or implements routines including `loadReg`, `createDirectBranch`. Notable symbols here include `loadReg`, `createDirectBranch`.
- CN: 这里声明或实现函数，例如 `loadReg`, `createDirectBranch`。这里较值得关注的符号包括 `loadReg`, `createDirectBranch`。

### Lines 3179-3196

```cpp
  // Instrumentation code sequence using LSE atomic instruction has a total of
  // 6 instructions:
  //
  //     stp    x0, x1, [sp, #-0x10]!
  //     adrp   x0, page_address(counter)
  //     add    x0, x0, page_offset(counter)
  //     mov    x1, #0x1
  //     stadd  x1, [x0]
  //     ldp    x0, x1, [sp], #0x10
  //
  // Instrumentation code sequence without using LSE atomic instruction has
  // 8 instructions at instrumentation place, with 6 instructions in the helper:
  //
  //     stp    x0, x30, [sp, #-0x10]!
  //     stp    x1, x2, [sp, #-0x10]!
  //     adrp   x0, page_address(counter)
  //     add    x0, x0, page_offset(counter)
  //     adrp   x1, page_address(helper)
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 3197-3208

```cpp
  //     add    x1, x1, page_offset(helper)
  //     blr    x1
  //     ldp    x0, x30, [sp], #0x10
  //
  //   <helper>:
  //     ldaxr  x1, [x0]
  //     add    x1, x1, #0x1
  //     stlxr  w2, x1, [x0]
  //     cbnz   w2, <helper>
  //     ldp    x1, x2, [sp], #0x10
  //     ret
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 3209-3216

```cpp
  void createInstrCounterIncrFunc(BinaryContext &BC) override {
    assert(InstrCounterIncrFunc == nullptr &&
           "helper function of counter increment for instrumentation "
           "has already been created");

    if (!opts::NoLSEAtomics)
      return;
```

- EN: Declares or implements routines including `createInstrCounterIncrFunc`. Notable symbols here include `createInstrCounterIncrFunc`.
- CN: 这里声明或实现函数，例如 `createInstrCounterIncrFunc`。这里较值得关注的符号包括 `createInstrCounterIncrFunc`。

### Lines 3217-3234

```cpp
    MCContext *Ctx = BC.Ctx.get();
    InstrCounterIncrFunc = BC.createInjectedBinaryFunction(
        "__bolt_instr_counter_incr", /*IsSimple*/ false);
    std::vector<std::unique_ptr<BinaryBasicBlock>> BBs;

    BBs.emplace_back(InstrCounterIncrFunc->createBasicBlock());
    InstructionListType Instrs(4);
    Instrs[0].setOpcode(AArch64::LDAXRX);
    Instrs[0].clear();
    Instrs[0].addOperand(MCOperand::createReg(AArch64::X1));
    Instrs[0].addOperand(MCOperand::createReg(AArch64::X0));
    Instrs[1].setOpcode(AArch64::ADDXri);
    Instrs[1].clear();
    Instrs[1].addOperand(MCOperand::createReg(AArch64::X1));
    Instrs[1].addOperand(MCOperand::createReg(AArch64::X1));
    Instrs[1].addOperand(MCOperand::createImm(1));
    Instrs[1].addOperand(MCOperand::createImm(0));
    Instrs[2].setOpcode(AArch64::STLXRX);
```

- EN: Declares or implements routines including `Instrs`. Notable symbols here include `Instrs`.
- CN: 这里声明或实现函数，例如 `Instrs`。这里较值得关注的符号包括 `Instrs`。

### Lines 3235-3246

```cpp
    Instrs[2].clear();
    Instrs[2].addOperand(MCOperand::createReg(AArch64::W2));
    Instrs[2].addOperand(MCOperand::createReg(AArch64::X1));
    Instrs[2].addOperand(MCOperand::createReg(AArch64::X0));
    Instrs[3].setOpcode(AArch64::CBNZW);
    Instrs[3].clear();
    Instrs[3].addOperand(MCOperand::createReg(AArch64::W2));
    Instrs[3].addOperand(MCOperand::createExpr(
        MCSymbolRefExpr::create(BBs.back()->getLabel(), *Ctx)));
    BBs.back()->addInstructions(Instrs.begin(), Instrs.end());
    BBs.back()->setCFIState(0);
```

- EN: Declares or implements routines including `create`. Notable symbols here include `create`.
- CN: 这里声明或实现函数，例如 `create`。这里较值得关注的符号包括 `create`。

### Lines 3247-3256

```cpp
    BBs.emplace_back(InstrCounterIncrFunc->createBasicBlock());
    InstructionListType InstrsEpilog(2);
    createPopRegisters(InstrsEpilog[0], AArch64::X1, AArch64::X2);
    createReturn(InstrsEpilog[1]);
    BBs.back()->addInstructions(InstrsEpilog.begin(), InstrsEpilog.end());
    BBs.back()->setCFIState(0);

    BBs[0]->addSuccessor(BBs[0].get());
    BBs[0]->addSuccessor(BBs[1].get());
```

- EN: Declares or implements routines including `InstrsEpilog`, `createPopRegisters`, `createReturn`, `addSuccessor`. Notable symbols here include `InstrsEpilog`, `createPopRegisters`, `createReturn`, `addSuccessor`.
- CN: 这里声明或实现函数，例如 `InstrsEpilog`, `createPopRegisters`, `createReturn`, `addSuccessor`。这里较值得关注的符号包括 `InstrsEpilog`, `createPopRegisters`, `createReturn`, `addSuccessor`。

### Lines 3257-3267

```cpp
    InstrCounterIncrFunc->insertBasicBlocks(nullptr, std::move(BBs),
                                            /*UpdateLayout*/ true,
                                            /*UpdateCFIState*/ false);
    InstrCounterIncrFunc->updateState(BinaryFunction::State::CFG_Finalized);

    LLVM_DEBUG({
      dbgs() << "BOLT-DEBUG: instrumentation counter increment helper:\n";
      InstrCounterIncrFunc->dump();
    });
  }
```

- EN: Declares or implements routines including `insertBasicBlocks`, `updateState`, `dbgs`, `dump`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `insertBasicBlocks`, `updateState`, `dbgs`, `dump`.
- CN: 这里声明或实现函数，例如 `insertBasicBlocks`, `updateState`, `dbgs`, `dump`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `insertBasicBlocks`, `updateState`, `dbgs`, `dump`。

### Lines 3268-3280

```cpp
  InstructionListType createInstrIncMemory(const MCSymbol *Target,
                                           MCContext *Ctx, bool IsLeaf,
                                           unsigned CodePointerSize) override {
    unsigned int I = 0;
    InstructionListType Instrs(opts::NoLSEAtomics ? 8 : 6);

    if (opts::NoLSEAtomics) {
      createPushRegisters(Instrs[I++], AArch64::X0, AArch64::LR);
      createPushRegisters(Instrs[I++], AArch64::X1, AArch64::X2);
    } else {
      createPushRegisters(Instrs[I++], AArch64::X0, AArch64::X1);
    }
```

- EN: Declares or implements routines including `Instrs`, `createPushRegisters`. Notable symbols here include `Instrs`, `createPushRegisters`.
- CN: 这里声明或实现函数，例如 `Instrs`, `createPushRegisters`。这里较值得关注的符号包括 `Instrs`, `createPushRegisters`。

### Lines 3281-3298

```cpp
    InstructionListType Addr = materializeAddress(Target, Ctx, AArch64::X0);
    assert(Addr.size() == 2 && "Invalid Addr size");
    std::copy(Addr.begin(), Addr.end(), Instrs.begin() + I);
    I += Addr.size();

    if (opts::NoLSEAtomics) {
      const MCSymbol *Helper = InstrCounterIncrFunc->getSymbol();
      InstructionListType HelperAddr =
          materializeAddress(Helper, Ctx, AArch64::X1);
      assert(HelperAddr.size() == 2 && "Invalid HelperAddr size");
      std::copy(HelperAddr.begin(), HelperAddr.end(), Instrs.begin() + I);
      I += HelperAddr.size();
      createIndirectCallInst(Instrs[I++], /*IsTailCall*/ false, AArch64::X1);
    } else {
      InstructionListType Insts = createIncMemory(AArch64::X0, AArch64::X1);
      assert(Insts.size() == 2 && "Invalid Insts size");
      std::copy(Insts.begin(), Insts.end(), Instrs.begin() + I);
      I += Insts.size();
```

- EN: Declares or implements routines including `materializeAddress`, `assert`, `copy`, `getSymbol`, `createIndirectCallInst`, and 1 more. Notable symbols here include `materializeAddress`, `assert`, `copy`, `getSymbol`, `createIndirectCallInst`, `createIncMemory`.
- CN: 这里声明或实现函数，例如 `materializeAddress`, `assert`, `copy`, `getSymbol`, `createIndirectCallInst`, and 1 more。这里较值得关注的符号包括 `materializeAddress`, `assert`, `copy`, `getSymbol`, `createIndirectCallInst`, `createIncMemory`。

### Lines 3299-3311

```cpp
    }
    createPopRegisters(Instrs[I++], AArch64::X0,
                       opts::NoLSEAtomics ? AArch64::LR : AArch64::X1);
    return Instrs;
  }

  std::vector<MCInst> createSymbolTrampoline(const MCSymbol *TgtSym,
                                             MCContext *Ctx) override {
    std::vector<MCInst> Insts;
    createShortJmp(Insts, TgtSym, Ctx, /*IsTailCall*/ true);
    return Insts;
  }
```

- EN: Declares or implements routines including `createShortJmp`. Notable symbols here include `createShortJmp`.
- CN: 这里声明或实现函数，例如 `createShortJmp`。这里较值得关注的符号包括 `createShortJmp`。

### Lines 3312-3320

```cpp
  void createBTI(MCInst &Inst, BTIKind BTI) const override {
    Inst.setOpcode(AArch64::HINT);
    Inst.clear();
    bool CallTarget = BTI == BTIKind::C || BTI == BTIKind::JC;
    bool JumpTarget = BTI == BTIKind::J || BTI == BTIKind::JC;
    unsigned HintNum = getBTIHintNum(CallTarget, JumpTarget);
    Inst.addOperand(MCOperand::createImm(HintNum));
  }
```

- EN: Declares or implements routines including `createBTI`, `getBTIHintNum`. Notable symbols here include `createBTI`, `getBTIHintNum`.
- CN: 这里声明或实现函数，例如 `createBTI`, `getBTIHintNum`。这里较值得关注的符号包括 `createBTI`, `getBTIHintNum`。

### Lines 3321-3329

```cpp
  bool isBTILandingPad(MCInst &Inst, BTIKind BTI) const override {
    bool CallTarget = BTI == BTIKind::C || BTI == BTIKind::JC;
    bool JumpTarget = BTI == BTIKind::J || BTI == BTIKind::JC;
    unsigned HintNum = getBTIHintNum(CallTarget, JumpTarget);
    bool IsExplicitBTI = Inst.getOpcode() == AArch64::HINT &&
                         MCPlus::getNumPrimeOperands(Inst) == 1 &&
                         Inst.getOperand(0).isImm() &&
                         Inst.getOperand(0).getImm() == HintNum;
```

- EN: Declares or implements routines including `isBTILandingPad`, `getBTIHintNum`, `getNumPrimeOperands`. Notable symbols here include `isBTILandingPad`, `getBTIHintNum`, `getNumPrimeOperands`.
- CN: 这里声明或实现函数，例如 `isBTILandingPad`, `getBTIHintNum`, `getNumPrimeOperands`。这里较值得关注的符号包括 `isBTILandingPad`, `getBTIHintNum`, `getNumPrimeOperands`。

### Lines 3330-3342

```cpp
    // Only "BTI C" can be implicit.
    bool IsImplicitBTI =
        HintNum == getBTIHintNum(true, false) && isImplicitBTIC(Inst);
    return IsExplicitBTI || IsImplicitBTI;
  }

  bool isImplicitBTIC(MCInst &Inst) const override {
    // PACI[AB]SP are always implicitly BTI C, independently of
    // SCTLR_EL1.BT[01].
    return Inst.getOpcode() == AArch64::PACIASP ||
           Inst.getOpcode() == AArch64::PACIBSP;
  }
```

- EN: Declares or implements routines including `getBTIHintNum`, `isImplicitBTIC`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getBTIHintNum`, `isImplicitBTIC`.
- CN: 这里声明或实现函数，例如 `getBTIHintNum`, `isImplicitBTIC`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getBTIHintNum`, `isImplicitBTIC`。

### Lines 3343-3351

```cpp
  bool isCallCoveredByBTI(MCInst &Call, MCInst &Pad) const override {
    assert((isIndirectCall(Call) || isIndirectBranch(Call)) &&
           "Not an indirect call or branch.");

    // A BLR can be accepted by a BTI c.
    if (isIndirectCall(Call))
      return isBTILandingPad(Pad, BTIKind::C) ||
             isBTILandingPad(Pad, BTIKind::JC);
```

- EN: Declares or implements routines including `isCallCoveredByBTI`, `assert`, `isBTILandingPad`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isCallCoveredByBTI`, `assert`, `isBTILandingPad`.
- CN: 这里声明或实现函数，例如 `isCallCoveredByBTI`, `assert`, `isBTILandingPad`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isCallCoveredByBTI`, `assert`, `isBTILandingPad`。

### Lines 3352-3369

```cpp
    // A BR can be accepted by a BTI j or BTI c (and BTI jc) IF the operand is
    // x16 or x17. If the operand is not x16 or x17, it can be accepted by a BTI
    // j or BTI jc (and not BTI c).
    if (isIndirectBranch(Call)) {
      assert(MCPlus::getNumPrimeOperands(Call) == 1 &&
             "Indirect branch needs to have 1 operand.");
      assert(Call.getOperand(0).isReg() &&
             "Indirect branch does not have a register operand.");
      MCPhysReg Reg = Call.getOperand(0).getReg();
      if (Reg == AArch64::X16 || Reg == AArch64::X17)
        return isBTILandingPad(Pad, BTIKind::C) ||
               isBTILandingPad(Pad, BTIKind::J) ||
               isBTILandingPad(Pad, BTIKind::JC);
      return isBTILandingPad(Pad, BTIKind::J) ||
             isBTILandingPad(Pad, BTIKind::JC);
    }
    return false;
  }
```

- EN: Declares or implements routines including `assert`, `isBTILandingPad`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `assert`, `isBTILandingPad`.
- CN: 这里声明或实现函数，例如 `assert`, `isBTILandingPad`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `assert`, `isBTILandingPad`。

### Lines 3370-3387

```cpp

  void insertBTI(BinaryBasicBlock &BB, MCInst &Call) const override {
    auto II = BB.getFirstNonPseudo();
    // Only check the first instruction for non-empty BasicBlocks
    bool Empty = (II == BB.end());
    if (!Empty && isCallCoveredByBTI(Call, *II))
      return;
    // A BLR can be accepted by a BTI c.
    if (isIndirectCall(Call)) {
      // if we have a BTI j at the start, extend it to a BTI jc,
      // otherwise insert a new BTI c.
      if (!Empty && isBTILandingPad(*II, BTIKind::J)) {
        createBTI(*II, BTIKind::JC);
      } else {
        MCInst BTIInst;
        createBTI(BTIInst, BTIKind::C);
        BB.insertInstruction(II, BTIInst);
      }
```

- EN: Declares or implements routines including `insertBTI`, `createBTI`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `insertBTI`, `createBTI`.
- CN: 这里声明或实现函数，例如 `insertBTI`, `createBTI`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `insertBTI`, `createBTI`。

### Lines 3388-3405

```cpp
    }

    // A BR can be accepted by a BTI j or BTI c (and BTI jc) IF the operand is
    // x16 or x17. If the operand is not x16 or x17, it can be accepted by a
    // BTI j or BTI jc (and not BTI c).
    if (isIndirectBranch(Call)) {
      assert(MCPlus::getNumPrimeOperands(Call) == 1 &&
             "Indirect branch needs to have 1 operand.");
      assert(Call.getOperand(0).isReg() &&
             "Indirect branch does not have a register operand.");
      MCPhysReg Reg = Call.getOperand(0).getReg();
      if (Reg == AArch64::X16 || Reg == AArch64::X17) {
        // Add a new BTI c
        MCInst BTIInst;
        createBTI(BTIInst, BTIKind::C);
        BB.insertInstruction(II, BTIInst);
      } else {
        // If BB starts with a BTI c, extend it to BTI jc,
```

- EN: Declares or implements routines including `assert`, `createBTI`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `assert`, `createBTI`.
- CN: 这里声明或实现函数，例如 `assert`, `createBTI`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `assert`, `createBTI`。

### Lines 3406-3417

```cpp
        // otherwise insert a new BTI j.
        if (!Empty && isBTILandingPad(*II, BTIKind::C)) {
          createBTI(*II, BTIKind::JC);
        } else {
          MCInst BTIInst;
          createBTI(BTIInst, BTIKind::J);
          BB.insertInstruction(II, BTIInst);
        }
      }
    }
  }
```

- EN: Declares or implements routines including `createBTI`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `createBTI`.
- CN: 这里声明或实现函数，例如 `createBTI`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `createBTI`。

### Lines 3418-3435

```cpp
  InstructionListType materializeAddress(const MCSymbol *Target, MCContext *Ctx,
                                         MCPhysReg RegName,
                                         int64_t Addend = 0) const override {
    // Get page-aligned address and add page offset
    InstructionListType Insts(2);
    Insts[0].setOpcode(AArch64::ADRP);
    Insts[0].clear();
    Insts[0].addOperand(MCOperand::createReg(RegName));
    Insts[0].addOperand(MCOperand::createImm(0));
    setOperandToSymbolRef(Insts[0], /* OpNum */ 1, Target, Addend, Ctx,
                          ELF::R_AARCH64_NONE);
    Insts[1].setOpcode(AArch64::ADDXri);
    Insts[1].clear();
    Insts[1].addOperand(MCOperand::createReg(RegName));
    Insts[1].addOperand(MCOperand::createReg(RegName));
    Insts[1].addOperand(MCOperand::createImm(0));
    Insts[1].addOperand(MCOperand::createImm(0));
    setOperandToSymbolRef(Insts[1], /* OpNum */ 2, Target, Addend, Ctx,
```

- EN: Declares or implements routines including `Insts`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `Insts`.
- CN: 这里声明或实现函数，例如 `Insts`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `Insts`。

### Lines 3436-3444

```cpp
                          ELF::R_AARCH64_ADD_ABS_LO12_NC);
    return Insts;
  }

  std::optional<Relocation>
  createRelocation(const MCFixup &Fixup,
                   const MCAsmBackend &MAB) const override {
    MCFixupKindInfo FKI = MAB.getFixupKindInfo(Fixup.getKind());
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 3445-3462

```cpp
    assert(FKI.TargetOffset == 0 && "0-bit relocation offset expected");
    const uint64_t RelOffset = Fixup.getOffset();

    uint32_t RelType;
    if (Fixup.getKind() == MCFixupKind(AArch64::fixup_aarch64_pcrel_call26))
      RelType = ELF::R_AARCH64_CALL26;
    else if (Fixup.getKind() ==
             MCFixupKind(AArch64::fixup_aarch64_pcrel_branch26))
      RelType = ELF::R_AARCH64_JUMP26;
    else if (Fixup.isPCRel()) {
      switch (FKI.TargetSize) {
      default:
        return std::nullopt;
      case 16:
        RelType = ELF::R_AARCH64_PREL16;
        break;
      case 32:
        RelType = ELF::R_AARCH64_PREL32;
```

- EN: Declares or implements routines including `assert`, `MCFixupKind`. Notable symbols here include `assert`, `MCFixupKind`.
- CN: 这里声明或实现函数，例如 `assert`, `MCFixupKind`。这里较值得关注的符号包括 `assert`, `MCFixupKind`。

### Lines 3463-3480

```cpp
        break;
      case 64:
        RelType = ELF::R_AARCH64_PREL64;
        break;
      }
    } else {
      switch (FKI.TargetSize) {
      default:
        return std::nullopt;
      case 16:
        RelType = ELF::R_AARCH64_ABS16;
        break;
      case 32:
        RelType = ELF::R_AARCH64_ABS32;
        break;
      case 64:
        RelType = ELF::R_AARCH64_ABS64;
        break;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 3481-3488

```cpp
      }
    }

    auto [RelSymbol, RelAddend] = extractFixupExpr(Fixup);

    return Relocation({RelOffset, RelSymbol, RelType, RelAddend, 0});
  }
```

- EN: Declares or implements routines including `extractFixupExpr`. Notable symbols here include `extractFixupExpr`.
- CN: 这里声明或实现函数，例如 `extractFixupExpr`。这里较值得关注的符号包括 `extractFixupExpr`。

### Lines 3489-3506

```cpp
  uint16_t getMinFunctionAlignment() const override { return 4; }

  std::optional<uint32_t>
  getInstructionSize(const MCInst &Inst) const override {
    return 4;
  }

  std::optional<uint64_t>
  extractMoveImmediate(const MCInst &Inst, MCPhysReg TargetReg) const override {
    // Match MOVZ instructions (both X and W register variants) with no shift.
    if ((Inst.getOpcode() == AArch64::MOVZXi ||
         Inst.getOpcode() == AArch64::MOVZWi) &&
        Inst.getOperand(2).getImm() == 0 &&
        getAliases(TargetReg)[Inst.getOperand(0).getReg()])
      return Inst.getOperand(1).getImm();
    return std::nullopt;
  }
```

- EN: Declares or implements routines including `getMinFunctionAlignment`, `getInstructionSize`, `extractMoveImmediate`, `getAliases`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getMinFunctionAlignment`, `getInstructionSize`, `extractMoveImmediate`, `getAliases`.
- CN: 这里声明或实现函数，例如 `getMinFunctionAlignment`, `getInstructionSize`, `extractMoveImmediate`, `getAliases`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getMinFunctionAlignment`, `getInstructionSize`, `extractMoveImmediate`, `getAliases`。

### Lines 3507-3516

```cpp
  std::optional<uint64_t>
  findMemcpySizeInBytes(const BinaryBasicBlock &BB,
                        InstructionListType::iterator CallInst) const override {
    MCPhysReg SizeReg = getIntArgRegister(2);
    if (SizeReg == getNoRegister())
      return std::nullopt;

    BitVector WrittenRegs(RegInfo->getNumRegs());
    const BitVector &SizeRegAliases = getAliases(SizeReg);
```

- EN: Declares or implements routines including `getIntArgRegister`, `WrittenRegs`, `getAliases`. Notable symbols here include `getIntArgRegister`, `WrittenRegs`, `getAliases`.
- CN: 这里声明或实现函数，例如 `getIntArgRegister`, `WrittenRegs`, `getAliases`。这里较值得关注的符号包括 `getIntArgRegister`, `WrittenRegs`, `getAliases`。

### Lines 3517-3527

```cpp
    for (auto InstIt = CallInst; InstIt != BB.begin(); --InstIt) {
      const MCInst &Inst = *InstIt;
      WrittenRegs.reset();
      getWrittenRegs(Inst, WrittenRegs);

      if (WrittenRegs.anyCommon(SizeRegAliases))
        return extractMoveImmediate(Inst, SizeReg);
    }
    return std::nullopt;
  }
```

- EN: Declares or implements routines including `getWrittenRegs`. Notable symbols here include `getWrittenRegs`.
- CN: 这里声明或实现函数，例如 `getWrittenRegs`。这里较值得关注的符号包括 `getWrittenRegs`。

### Lines 3528-3535

```cpp
  InstructionListType
  createInlineMemcpy(bool ReturnEnd,
                     std::optional<uint64_t> KnownSize) const override {
    assert(KnownSize.has_value() &&
           "AArch64 memcpy inlining requires known size");
    InstructionListType Code;
    uint64_t Size = *KnownSize;
```

- EN: Declares or implements routines including `assert`. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里较值得关注的符号包括 `assert`。

### Lines 3536-3547

```cpp
    generateSizeSpecificMemcpy(Code, Size);

    // If _memcpy8, adjust X0 to return dest+size instead of dest.
    if (ReturnEnd)
      Code.emplace_back(MCInstBuilder(AArch64::ADDXri)
                            .addReg(AArch64::X0)
                            .addReg(AArch64::X0)
                            .addImm(Size)
                            .addImm(0));
    return Code;
  }
```

- EN: Declares or implements routines including `generateSizeSpecificMemcpy`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `generateSizeSpecificMemcpy`.
- CN: 这里声明或实现函数，例如 `generateSizeSpecificMemcpy`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `generateSizeSpecificMemcpy`。

### Lines 3548-3561

```cpp
  InstructionListType generateSizeSpecificMemcpy(InstructionListType &Code,
                                                 uint64_t Size) const {
    auto AddLoadStorePair = [&](unsigned LoadOpc, unsigned StoreOpc,
                                unsigned Reg, unsigned Offset = 0) {
      Code.emplace_back(MCInstBuilder(LoadOpc)
                            .addReg(Reg)
                            .addReg(AArch64::X1)
                            .addImm(Offset));
      Code.emplace_back(MCInstBuilder(StoreOpc)
                            .addReg(Reg)
                            .addReg(AArch64::X0)
                            .addImm(Offset));
    };
```

- EN: Continues the surrounding implementation, wiring declarations, state, and helper logic together.
- CN: 这里继续展开周边实现，衔接声明、状态与辅助逻辑。

### Lines 3562-3579

```cpp
    // Generate optimal instruction sequences based on exact size.
    switch (Size) {
    case 1:
      AddLoadStorePair(AArch64::LDRBBui, AArch64::STRBBui, AArch64::W9);
      break;
    case 2:
      AddLoadStorePair(AArch64::LDRHHui, AArch64::STRHHui, AArch64::W9);
      break;
    case 4:
      AddLoadStorePair(AArch64::LDRWui, AArch64::STRWui, AArch64::W9);
      break;
    case 8:
      AddLoadStorePair(AArch64::LDRXui, AArch64::STRXui, AArch64::X9);
      break;
    case 16:
      AddLoadStorePair(AArch64::LDRQui, AArch64::STRQui, AArch64::Q16);
      break;
    case 32:
```

- EN: Declares or implements routines including `AddLoadStorePair`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `AddLoadStorePair`.
- CN: 这里声明或实现函数，例如 `AddLoadStorePair`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `AddLoadStorePair`。

### Lines 3580-3589

```cpp
      AddLoadStorePair(AArch64::LDRQui, AArch64::STRQui, AArch64::Q16, 0);
      AddLoadStorePair(AArch64::LDRQui, AArch64::STRQui, AArch64::Q17, 1);
      break;

    default:
      // For sizes up to 64 bytes, greedily use the largest possible loads.
      // Caller should have already filtered out sizes > 64 bytes.
      assert(Size <= 64 &&
             "Size should be <= 64 bytes for AArch64 memcpy inlining");
```

- EN: Declares or implements routines including `AddLoadStorePair`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `AddLoadStorePair`.
- CN: 这里声明或实现函数，例如 `AddLoadStorePair`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `AddLoadStorePair`。

### Lines 3590-3600

```cpp
      uint64_t Remaining = Size;
      uint64_t Offset = 0;

      const std::array<std::tuple<uint64_t, unsigned, unsigned, unsigned>, 5>
          LoadStoreOps = {
              {{16, AArch64::LDRQui, AArch64::STRQui, AArch64::Q16},
               {8, AArch64::LDRXui, AArch64::STRXui, AArch64::X9},
               {4, AArch64::LDRWui, AArch64::STRWui, AArch64::W9},
               {2, AArch64::LDRHHui, AArch64::STRHHui, AArch64::W9},
               {1, AArch64::LDRBBui, AArch64::STRBBui, AArch64::W9}}};
```

- EN: Continues the surrounding implementation, wiring declarations, state, and helper logic together.
- CN: 这里继续展开周边实现，衔接声明、状态与辅助逻辑。

### Lines 3601-3612

```cpp
      for (const auto &[OpSize, LoadOp, StoreOp, TempReg] : LoadStoreOps)
        while (Remaining >= OpSize) {
          AddLoadStorePair(LoadOp, StoreOp, TempReg, Offset / OpSize);
          Remaining -= OpSize;
          Offset += OpSize;
        }
      break;
    }
    return Code;
  }
};
```

- EN: Declares or implements routines including `AddLoadStorePair`. Notable symbols here include `AddLoadStorePair`.
- CN: 这里声明或实现函数，例如 `AddLoadStorePair`。这里较值得关注的符号包括 `AddLoadStorePair`。

### Lines 3613-3624

```cpp
} // end anonymous namespace

namespace llvm {
namespace bolt {

MCPlusBuilder *createAArch64MCPlusBuilder(const MCInstrAnalysis *Analysis,
                                          const MCInstrInfo *Info,
                                          const MCRegisterInfo *RegInfo,
                                          const MCSubtargetInfo *STI) {
  return new AArch64MCPlusBuilder(Analysis, Info, RegInfo, STI);
}
```

- EN: Works inside namespace scope `namespace`, `bolt` to organize symbols. Contains local control flow that updates state or selects among execution branches. Notable symbols here include `namespace`, `bolt`.
- CN: 这里位于命名空间 `namespace`, `bolt` 中，用于组织符号作用域。这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里较值得关注的符号包括 `namespace`, `bolt`。

### Lines 3625-3626

```cpp
} // namespace bolt
} // namespace llvm
```

- EN: Works inside namespace scope `bolt`, `llvm` to organize symbols. Notable symbols here include `bolt`, `llvm`.
- CN: 这里位于命名空间 `bolt`, `llvm` 中，用于组织符号作用域。这里较值得关注的符号包括 `bolt`, `llvm`。

## Key Concepts / 关键概念

- `AArch64MCPlusBuilder`: class or struct interface / 类或结构体接口
- `init`: function or method entry point / 函数或方法入口
- `getSystemFlag`: function or method entry point / 函数或方法入口
- `setSystemFlag`: function or method entry point / 函数或方法入口
- `createPushRegisters`: function or method entry point / 函数或方法入口
- `createPopRegisters`: function or method entry point / 函数或方法入口
- `DEBUG_TYPE`: macro or compile-time switch / 宏或编译期开关
- `llvm`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Core/BinaryBasicBlock.h`, `bolt/Core/BinaryFunction.h`, `bolt/Core/MCInstUtils.h`, `bolt/Core/MCPlusBuilder.h`
- LLVM headers / LLVM 头文件: `llvm/BinaryFormat/ELF.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCDisassembler/MCDisassembler.h`, `llvm/MC/MCInstBuilder.h`, `llvm/MC/MCInstrInfo.h`, `llvm/MC/MCRegister.h`, `llvm/MC/MCRegisterInfo.h`, `llvm/Support/CommandLine.h`, `llvm/Support/DataExtractor.h`, `llvm/Support/Debug.h`, `llvm/Support/ErrorHandling.h`
- System headers / 系统头文件: `AArch64ExpandImm.h`, `AArch64InstrInfo.h`, `AArch64MCSymbolizer.h`, `MCTargetDesc/AArch64AddressingModes.h`, `MCTargetDesc/AArch64FixupKinds.h`, `MCTargetDesc/AArch64MCAsmInfo.h`, `MCTargetDesc/AArch64MCTargetDesc.h`, `Utils/AArch64BaseInfo.h`
- Directory context / 目录上下文: `bolt/lib/Target/AArch64` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Target/AArch64` 下的相邻文件通常与本文件协作组成对应子系统
