# PPCInstPrinter.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/MCTargetDesc/PPCInstPrinter.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file provides MC layer support for the backend for the PowerPC backend.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/MCTargetDesc/PPCInstPrinter.cpp`，主要负责 PowerPC 后端的该后端的 MC 层支持。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- PPCInstPrinter.cpp - Convert PPC MCInst to assembly syntax --------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file. The logic interacts with LLVM's MC layer.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 7-10

```cpp
//===----------------------------------------------------------------------===//
//
// This class prints an PPC MCInst to a .s file.
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This class prints an PPC MCInst to a .s file.". The logic interacts with LLVM's MC layer.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This class prints an PPC MCInst to a .s file.”。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 11-17

```cpp
//===----------------------------------------------------------------------===//

#include "MCTargetDesc/PPCInstPrinter.h"
#include "MCTargetDesc/PPCMCTargetDesc.h"
#include "MCTargetDesc/PPCPredicates.h"
#include "PPCMCAsmInfo.h"
#include "llvm/MC/MCAsmInfo.h"
```
- **EN**: Pulls in direct dependencies required by this MC layer support for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. The logic interacts with LLVM's MC layer.
- **CN**: 这一段引入该该后端的 MC 层支持所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 18-24

```cpp
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCInstrInfo.h"
#include "llvm/MC/MCRegisterInfo.h"
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/MC/MCSymbol.h"
#include "llvm/Support/Casting.h"
```
- **EN**: Pulls in direct dependencies required by this MC layer support for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. Register assignment and register-class constraints matter here.
- **CN**: 这一段引入该该后端的 MC 层支持所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 25-47

```cpp
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/raw_ostream.h"
using namespace llvm;

#define DEBUG_TYPE "asm-printer"

// FIXME: Once the integrated assembler supports full register names, tie this
// to the verbose-asm setting.
static cl::opt<bool>
FullRegNames("ppc-asm-full-reg-names", cl::Hidden, cl::init(false),
             cl::desc("Use full register names when printing assembly"));

// Useful for testing purposes. Prints vs{31-63} as v{0-31} respectively.
static cl::opt<bool>
ShowVSRNumsAsVR("ppc-vsr-nums-as-vr", cl::Hidden, cl::init(false),
             cl::desc("Prints full register names with vs{31-63} as v{0-31}"));

// Prints full register names with percent symbol.
static cl::opt<bool>
FullRegNamesWithPercent("ppc-reg-with-percent-prefix", cl::Hidden,
                        cl::init(false),
                        cl::desc("Prints full register names with percent"));
```
- **EN**: Pulls in direct dependencies required by this MC layer support for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. Notable symbols in this range include `FullRegNames`, `init`, `desc`.
- **CN**: 这一段引入该该后端的 MC 层支持所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 该区间中较显眼的符号包括 `FullRegNames`, `init`, `desc`。

### Lines 48-85

```cpp
#define PRINT_ALIAS_INSTR
#include "PPCGenAsmWriter.inc"

void PPCInstPrinter::printRegName(raw_ostream &OS, MCRegister Reg) {
  const char *RegName = getRegisterName(Reg);
  OS << RegName;
}

void PPCInstPrinter::printInst(const MCInst *MI, uint64_t Address,
                               StringRef Annot, const MCSubtargetInfo &STI,
                               raw_ostream &O) {
  // Customize printing of the addis instruction on AIX. When an operand is a
  // symbol reference, the instruction syntax is changed to look like a load
  // operation, i.e:
  //     Transform:  addis $rD, $rA, $src --> addis $rD, $src($rA).
  if (TT.isOSAIX() &&
      (MI->getOpcode() == PPC::ADDIS8 || MI->getOpcode() == PPC::ADDIS) &&
      MI->getOperand(2).isExpr()) {
    assert((MI->getOperand(0).isReg() && MI->getOperand(1).isReg()) &&
           "The first and the second operand of an addis instruction"
           " should be registers.");

    assert(isa<MCSymbolRefExpr>(MI->getOperand(2).getExpr()) &&
           "The third operand of an addis instruction should be a symbol "
           "reference expression if it is an expression at all.");

    O << "\taddis ";
    printOperand(MI, 0, STI, O);
    O << ", ";
    printOperand(MI, 2, STI, O);
    O << "(";
    printOperand(MI, 1, STI, O);
    O << ")";
    return;
  }

  // Check if the last operand is an expression with the variant kind
  // VK_PCREL_OPT. If this is the case then this is a linker optimization
```
- **EN**: Pulls in direct dependencies required by this MC layer support for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. Subtarget feature gating influences the behavior here.
- **CN**: 这一段引入该该后端的 MC 层支持所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 子目标特性裁剪会影响这里的行为。

### Lines 86-123

```cpp
  // relocation and the .reloc directive needs to be added.
  unsigned LastOp = MI->getNumOperands() - 1;
  if (MI->getNumOperands() > 1) {
    const MCOperand &Operand = MI->getOperand(LastOp);
    if (Operand.isExpr()) {
      const MCExpr *Expr = Operand.getExpr();
      const MCSymbolRefExpr *SymExpr =
          static_cast<const MCSymbolRefExpr *>(Expr);

      if (SymExpr && getSpecifier(SymExpr) == PPC::S_PCREL_OPT) {
        const MCSymbol &Symbol = SymExpr->getSymbol();
        if (MI->getOpcode() == PPC::PLDpc) {
          printInstruction(MI, Address, STI, O);
          O << "\n";
          Symbol.print(O, &MAI);
          O << ":";
          return;
        } else {
          O << "\t.reloc ";
          Symbol.print(O, &MAI);
          O << "-8,R_PPC64_PCREL_OPT,.-(";
          Symbol.print(O, &MAI);
          O << "-8)\n";
        }
      }
    }
  }

  // Check for slwi/srwi mnemonics.
  if (MI->getOpcode() == PPC::RLWINM) {
    unsigned char SH = MI->getOperand(2).getImm();
    unsigned char MB = MI->getOperand(3).getImm();
    unsigned char ME = MI->getOperand(4).getImm();
    bool useSubstituteMnemonic = false;
    if (SH <= 31 && MB == 0 && ME == (31-SH)) {
      O << "\tslwi "; useSubstituteMnemonic = true;
    }
    if (SH <= 31 && MB == (32-SH) && ME == 31) {
```
- **EN**: Implements helper routine(s) `getNumOperands`, `getOperand`, `isExpr` for this portion of the PowerPC backend MC layer support for the backend. The logic interacts with LLVM's MC layer.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 MC 层支持所需的辅助例程 `getNumOperands`, `getOperand`, `isExpr`。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 124-161

```cpp
      O << "\tsrwi "; useSubstituteMnemonic = true;
      SH = 32-SH;
    }
    if (useSubstituteMnemonic) {
      printOperand(MI, 0, STI, O);
      O << ", ";
      printOperand(MI, 1, STI, O);
      O << ", " << (unsigned int)SH;

      printAnnotation(O, Annot);
      return;
    }
  }

  if (MI->getOpcode() == PPC::RLDICR ||
      MI->getOpcode() == PPC::RLDICR_32) {
    unsigned char SH = MI->getOperand(2).getImm();
    unsigned char ME = MI->getOperand(3).getImm();
    // rldicr RA, RS, SH, 63-SH == sldi RA, RS, SH
    if (63-SH == ME) {
      O << "\tsldi ";
      printOperand(MI, 0, STI, O);
      O << ", ";
      printOperand(MI, 1, STI, O);
      O << ", " << (unsigned int)SH;
      printAnnotation(O, Annot);
      return;
    }
  }

  // dcbt[st] is printed manually here because:
  //  1. The assembly syntax is different between embedded and server targets
  //  2. We must print the short mnemonics for TH == 0 because the
  //     embedded/server syntax default will not be stable across assemblers
  //  The syntax for dcbt is:
  //    dcbt ra, rb, th [server]
  //    dcbt th, ra, rb [embedded]
  //  where th can be omitted when it is 0. dcbtst is the same.
```
- **EN**: Implements helper routine(s) `printOperand`, `printAnnotation`, `getOpcode` for this portion of the PowerPC backend MC layer support for the backend.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 MC 层支持所需的辅助例程 `printOperand`, `printAnnotation`, `getOpcode`。

### Lines 162-199

```cpp
  // On AIX, only emit the extended mnemonics for dcbt and dcbtst if
  // the "modern assembler" is available.
  if ((MI->getOpcode() == PPC::DCBT || MI->getOpcode() == PPC::DCBTST) &&
      (!TT.isOSAIX() || STI.hasFeature(PPC::FeatureModernAIXAs))) {
    unsigned char TH = MI->getOperand(0).getImm();
    O << "\tdcbt";
    if (MI->getOpcode() == PPC::DCBTST)
      O << "st";
    if (TH == 16)
      O << "t";
    O << " ";

    bool IsBookE = STI.hasFeature(PPC::FeatureBookE);
    if (IsBookE && TH != 0 && TH != 16)
      O << (unsigned int) TH << ", ";

    printOperand(MI, 1, STI, O);
    O << ", ";
    printOperand(MI, 2, STI, O);

    if (!IsBookE && TH != 0 && TH != 16)
      O << ", " << (unsigned int) TH;

    printAnnotation(O, Annot);
    return;
  }

  if (MI->getOpcode() == PPC::DCBF) {
    unsigned char L = MI->getOperand(0).getImm();
    if (!L || L == 1 || L == 3 || L == 4 || L == 6) {
      O << "\tdcb";
      if (L != 6)
        O << "f";
      if (L == 1)
        O << "l";
      if (L == 3)
        O << "lp";
      if (L == 4)
```
- **EN**: Implements helper routine(s) `getOpcode`, `isOSAIX`, `hasFeature` for this portion of the PowerPC backend MC layer support for the backend. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 MC 层支持所需的辅助例程 `getOpcode`, `isOSAIX`, `hasFeature`。 子目标特性裁剪会影响这里的行为。

### Lines 200-237

```cpp
        O << "ps";
      if (L == 6)
        O << "stps";
      O << " ";

      printOperand(MI, 1, STI, O);
      O << ", ";
      printOperand(MI, 2, STI, O);

      printAnnotation(O, Annot);
      return;
    }
  }

  if (!printAliasInstr(MI, Address, STI, O))
    printInstruction(MI, Address, STI, O);
  printAnnotation(O, Annot);
}

void PPCInstPrinter::printPredicateOperand(const MCInst *MI, unsigned OpNo,
                                           const MCSubtargetInfo &STI,
                                           raw_ostream &O, StringRef Modifier) {
  unsigned Code = MI->getOperand(OpNo).getImm();

  if (Modifier == "cc") {
    switch ((PPC::Predicate)Code) {
    case PPC::PRED_LT_MINUS:
    case PPC::PRED_LT_PLUS:
    case PPC::PRED_LT:
      O << "lt";
      return;
    case PPC::PRED_LE_MINUS:
    case PPC::PRED_LE_PLUS:
    case PPC::PRED_LE:
      O << "le";
      return;
    case PPC::PRED_EQ_MINUS:
    case PPC::PRED_EQ_PLUS:
```
- **EN**: Implements helper routine(s) `printOperand`, `printAnnotation`, `printAliasInstr` for this portion of the PowerPC backend MC layer support for the backend. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 MC 层支持所需的辅助例程 `printOperand`, `printAnnotation`, `printAliasInstr`。 子目标特性裁剪会影响这里的行为。

### Lines 238-275

```cpp
    case PPC::PRED_EQ:
      O << "eq";
      return;
    case PPC::PRED_GE_MINUS:
    case PPC::PRED_GE_PLUS:
    case PPC::PRED_GE:
      O << "ge";
      return;
    case PPC::PRED_GT_MINUS:
    case PPC::PRED_GT_PLUS:
    case PPC::PRED_GT:
      O << "gt";
      return;
    case PPC::PRED_NE_MINUS:
    case PPC::PRED_NE_PLUS:
    case PPC::PRED_NE:
      O << "ne";
      return;
    case PPC::PRED_UN_MINUS:
    case PPC::PRED_UN_PLUS:
    case PPC::PRED_UN:
      O << "un";
      return;
    case PPC::PRED_NU_MINUS:
    case PPC::PRED_NU_PLUS:
    case PPC::PRED_NU:
      O << "nu";
      return;
    case PPC::PRED_BIT_SET:
    case PPC::PRED_BIT_UNSET:
      llvm_unreachable("Invalid use of bit predicate code");
    }
    llvm_unreachable("Invalid predicate code");
  }

  if (Modifier == "pm") {
    switch ((PPC::Predicate)Code) {
    case PPC::PRED_LT:
```
- **EN**: Implements helper routine(s) `llvm_unreachable` for this portion of the PowerPC backend MC layer support for the backend. These declarations feed generated pattern-matching logic.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 MC 层支持所需的辅助例程 `llvm_unreachable`。 这些声明会进入生成式模式匹配逻辑。

### Lines 276-313

```cpp
    case PPC::PRED_LE:
    case PPC::PRED_EQ:
    case PPC::PRED_GE:
    case PPC::PRED_GT:
    case PPC::PRED_NE:
    case PPC::PRED_UN:
    case PPC::PRED_NU:
      return;
    case PPC::PRED_LT_MINUS:
    case PPC::PRED_LE_MINUS:
    case PPC::PRED_EQ_MINUS:
    case PPC::PRED_GE_MINUS:
    case PPC::PRED_GT_MINUS:
    case PPC::PRED_NE_MINUS:
    case PPC::PRED_UN_MINUS:
    case PPC::PRED_NU_MINUS:
      O << "-";
      return;
    case PPC::PRED_LT_PLUS:
    case PPC::PRED_LE_PLUS:
    case PPC::PRED_EQ_PLUS:
    case PPC::PRED_GE_PLUS:
    case PPC::PRED_GT_PLUS:
    case PPC::PRED_NE_PLUS:
    case PPC::PRED_UN_PLUS:
    case PPC::PRED_NU_PLUS:
      O << "+";
      return;
    case PPC::PRED_BIT_SET:
    case PPC::PRED_BIT_UNSET:
      llvm_unreachable("Invalid use of bit predicate code");
    }
    llvm_unreachable("Invalid predicate code");
  }

  assert(Modifier == "reg" &&
         "Need to specify 'cc', 'pm' or 'reg' as predicate op modifier!");
  printOperand(MI, OpNo + 1, STI, O);
```
- **EN**: Declares function entry points including `llvm_unreachable`, `printOperand` that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `llvm_unreachable`, `printOperand`。

### Lines 314-351

```cpp
}

void PPCInstPrinter::printATBitsAsHint(const MCInst *MI, unsigned OpNo,
                                       const MCSubtargetInfo &STI,
                                       raw_ostream &O) {
  unsigned Code = MI->getOperand(OpNo).getImm();
  if (Code == 2)
    O << "-";
  else if (Code == 3)
    O << "+";
}

// Template for unsigned immediate operands with validation.
// Validates that the value fits within the specified width and prints it.
template <unsigned Width>
void PPCInstPrinter::printUImmOperand(const MCInst *MI, unsigned OpNo,
                                      const MCSubtargetInfo &STI,
                                      raw_ostream &O) {
  unsigned int Value = MI->getOperand(OpNo).getImm();
  assert(Value <= ((1ULL << Width) - 1) && "Invalid uimm argument!");
  O << (unsigned int)Value;
}

// Template for signed immediate operands with sign extension.
// Sign-extends the value to the specified width and prints it.
template <unsigned Width>
void PPCInstPrinter::printSImmOperand(const MCInst *MI, unsigned OpNo,
                                      const MCSubtargetInfo &STI,
                                      raw_ostream &O) {
  int Value = MI->getOperand(OpNo).getImm();
  Value = SignExtend32<Width>(Value);
  O << (int)Value;
}

void PPCInstPrinter::printImmZeroOperand(const MCInst *MI, unsigned OpNo,
                                         const MCSubtargetInfo &STI,
                                         raw_ostream &O) {
  unsigned int Value = MI->getOperand(OpNo).getImm();
```
- **EN**: Implements helper routine(s) `printATBitsAsHint`, `getOperand`, `getImm` for this portion of the PowerPC backend MC layer support for the backend. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 MC 层支持所需的辅助例程 `printATBitsAsHint`, `getOperand`, `getImm`。 子目标特性裁剪会影响这里的行为。

### Lines 352-389

```cpp
  assert(Value == 0 && "Operand must be zero");
  O << (unsigned int)Value;
}

// Truncating version specifically for BUILD_VECTOR operands that may be
// sign-extended (e.g., -1 becomes 0xFFFFFFFF). Truncates to 8 bits without
// validation, unlike the standard printUImmOperand<8> which validates.
void PPCInstPrinter::printU8ImmOperandTrunc(const MCInst *MI, unsigned OpNo,
                                            const MCSubtargetInfo &STI,
                                            raw_ostream &O) {
  unsigned char Value = MI->getOperand(OpNo).getImm();
  O << (unsigned int)Value;
}

void PPCInstPrinter::printS16ImmOperand(const MCInst *MI, unsigned OpNo,
                                        const MCSubtargetInfo &STI,
                                        raw_ostream &O) {
  if (MI->getOperand(OpNo).isImm())
    O << (short)MI->getOperand(OpNo).getImm();
  else
    printOperand(MI, OpNo, STI, O);
}

void PPCInstPrinter::printS32ImmOperand(const MCInst *MI, unsigned OpNo,
                                        const MCSubtargetInfo &STI,
                                        raw_ostream &O) {
  if (MI->getOperand(OpNo).isImm()) {
    long long Value = MI->getOperand(OpNo).getImm();
    assert(isInt<32>(Value) && "Invalid s32imm argument!");
    O << (long long)Value;
  } else
    printOperand(MI, OpNo, STI, O);
}

void PPCInstPrinter::printNegS32ImmOperand(const MCInst *MI, unsigned OpNo,
                                           const MCSubtargetInfo &STI,
                                           raw_ostream &O) {
  if (MI->getOperand(OpNo).isImm()) {
```
- **EN**: Implements helper routine(s) `extended`, `printU8ImmOperandTrunc`, `getOperand` for this portion of the PowerPC backend MC layer support for the backend. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 MC 层支持所需的辅助例程 `extended`, `printU8ImmOperandTrunc`, `getOperand`。 子目标特性裁剪会影响这里的行为。

### Lines 390-427

```cpp
    long long Value = MI->getOperand(OpNo).getImm();
    assert(isInt<32>(Value) && "Invalid s32imm argument!");
    O << (long long)Value;
  } else
    printOperand(MI, OpNo, STI, O);
}

void PPCInstPrinter::printS34ImmOperand(const MCInst *MI, unsigned OpNo,
                                        const MCSubtargetInfo &STI,
                                        raw_ostream &O) {
  if (MI->getOperand(OpNo).isImm()) {
    long long Value = MI->getOperand(OpNo).getImm();
    assert(isInt<34>(Value) && "Invalid s34imm argument!");
    O << (long long)Value;
  }
  else
    printOperand(MI, OpNo, STI, O);
}

void PPCInstPrinter::printU16ImmOperand(const MCInst *MI, unsigned OpNo,
                                        const MCSubtargetInfo &STI,
                                        raw_ostream &O) {
  if (MI->getOperand(OpNo).isImm())
    O << (unsigned short)MI->getOperand(OpNo).getImm();
  else
    printOperand(MI, OpNo, STI, O);
}

void PPCInstPrinter::printBranchOperand(const MCInst *MI, uint64_t Address,
                                        unsigned OpNo,
                                        const MCSubtargetInfo &STI,
                                        raw_ostream &O) {
  if (!MI->getOperand(OpNo).isImm())
    return printOperand(MI, OpNo, STI, O);
  int32_t Imm = SignExtend32<32>((unsigned)MI->getOperand(OpNo).getImm() << 2);
  if (PrintBranchImmAsAddress) {
    uint64_t Target = Address + Imm;
    if (!TT.isPPC64())
```
- **EN**: Implements helper routine(s) `getOperand`, `getImm`, `printOperand` for this portion of the PowerPC backend MC layer support for the backend. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 MC 层支持所需的辅助例程 `getOperand`, `getImm`, `printOperand`。 子目标特性裁剪会影响这里的行为。

### Lines 428-465

```cpp
      Target &= 0xffffffff;
    O << formatHex(Target);
  } else {
    // Branches can take an immediate operand. This is used by the branch
    // selection pass to print, for example `.+8` (for ELF) or `$+8` (for AIX)
    // to express an eight byte displacement from the program counter.
    if (!TT.isOSAIX())
      O << ".";
    else
      O << "$";

    if (Imm >= 0)
      O << "+";
    O << Imm;
  }
}

void PPCInstPrinter::printAbsBranchOperand(const MCInst *MI, unsigned OpNo,
                                           const MCSubtargetInfo &STI,
                                           raw_ostream &O) {
  if (!MI->getOperand(OpNo).isImm())
    return printOperand(MI, OpNo, STI, O);

  uint64_t Imm = static_cast<uint64_t>(MI->getOperand(OpNo).getImm()) << 2;
  if (!TT.isPPC64())
    Imm = static_cast<uint32_t>(Imm);
  O << formatHex(Imm);
}

void PPCInstPrinter::printcrbitm(const MCInst *MI, unsigned OpNo,
                                 const MCSubtargetInfo &STI, raw_ostream &O) {
  MCRegister CCReg = MI->getOperand(OpNo).getReg();
  unsigned RegNo;
  switch (CCReg.id()) {
  default: llvm_unreachable("Unknown CR register");
  case PPC::CR0: RegNo = 0; break;
  case PPC::CR1: RegNo = 1; break;
  case PPC::CR2: RegNo = 2; break;
```
- **EN**: Implements helper routine(s) `formatHex`, `isOSAIX`, `printAbsBranchOperand` for this portion of the PowerPC backend MC layer support for the backend. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 MC 层支持所需的辅助例程 `formatHex`, `isOSAIX`, `printAbsBranchOperand`。 子目标特性裁剪会影响这里的行为。

### Lines 466-503

```cpp
  case PPC::CR3: RegNo = 3; break;
  case PPC::CR4: RegNo = 4; break;
  case PPC::CR5: RegNo = 5; break;
  case PPC::CR6: RegNo = 6; break;
  case PPC::CR7: RegNo = 7; break;
  }
  O << (0x80 >> RegNo);
}

void PPCInstPrinter::printMemRegImm(const MCInst *MI, unsigned OpNo,
                                    const MCSubtargetInfo &STI,
                                    raw_ostream &O) {
  printS16ImmOperand(MI, OpNo, STI, O);
  O << '(';
  if (MI->getOperand(OpNo+1).getReg() == PPC::R0)
    O << "0";
  else
    printOperand(MI, OpNo + 1, STI, O);
  O << ')';
}

void PPCInstPrinter::printMemRegImmHash(const MCInst *MI, unsigned OpNo,
                                        const MCSubtargetInfo &STI,
                                        raw_ostream &O) {
  O << MI->getOperand(OpNo).getImm();
  O << '(';
  printOperand(MI, OpNo + 1, STI, O);
  O << ')';
}

void PPCInstPrinter::printMemRegImm34PCRel(const MCInst *MI, unsigned OpNo,
                                           const MCSubtargetInfo &STI,
                                           raw_ostream &O) {
  printS34ImmOperand(MI, OpNo, STI, O);
  O << '(';
  printImmZeroOperand(MI, OpNo + 1, STI, O);
  O << ')';
}
```
- **EN**: Implements helper routine(s) `printMemRegImm`, `printS16ImmOperand`, `getOperand` for this portion of the PowerPC backend MC layer support for the backend. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 MC 层支持所需的辅助例程 `printMemRegImm`, `printS16ImmOperand`, `getOperand`。 子目标特性裁剪会影响这里的行为。

### Lines 504-541

```cpp

void PPCInstPrinter::printMemRegImm34(const MCInst *MI, unsigned OpNo,
                                      const MCSubtargetInfo &STI,
                                      raw_ostream &O) {
  printS34ImmOperand(MI, OpNo, STI, O);
  O << '(';
  printOperand(MI, OpNo + 1, STI, O);
  O << ')';
}

void PPCInstPrinter::printMemRegReg(const MCInst *MI, unsigned OpNo,
                                    const MCSubtargetInfo &STI,
                                    raw_ostream &O) {
  // When used as the base register, r0 reads constant zero rather than
  // the value contained in the register.  For this reason, the darwin
  // assembler requires that we print r0 as 0 (no r) when used as the base.
  if (MI->getOperand(OpNo).getReg() == PPC::R0)
    O << "0";
  else
    printOperand(MI, OpNo, STI, O);
  O << ", ";
  printOperand(MI, OpNo + 1, STI, O);
}

void PPCInstPrinter::printTLSCall(const MCInst *MI, unsigned OpNo,
                                  const MCSubtargetInfo &STI, raw_ostream &O) {
  // On PPC64, VariantKind is VK_None, but on PPC32, it's VK_PLT, and it must
  // come at the _end_ of the expression.
  const MCOperand &Op = MI->getOperand(OpNo);
  const MCSymbolRefExpr *RefExp = nullptr;
  const MCExpr *Rhs = nullptr;
  if (const MCBinaryExpr *BinExpr = dyn_cast<MCBinaryExpr>(Op.getExpr())) {
    RefExp = cast<MCSymbolRefExpr>(BinExpr->getLHS());
    Rhs = BinExpr->getRHS();
  } else
    RefExp = cast<MCSymbolRefExpr>(Op.getExpr());

  O << RefExp->getSymbol().getName();
```
- **EN**: Implements helper routine(s) `printMemRegImm34`, `printS34ImmOperand`, `printOperand` for this portion of the PowerPC backend MC layer support for the backend. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 MC 层支持所需的辅助例程 `printMemRegImm34`, `printS34ImmOperand`, `printOperand`。 子目标特性裁剪会影响这里的行为。

### Lines 542-579

```cpp
  // The variant kind VK_NOTOC needs to be handled as a special case
  // because we do not want the assembly to print out the @notoc at the
  // end like __tls_get_addr(x@tlsgd)@notoc. Instead we want it to look
  // like __tls_get_addr@notoc(x@tlsgd).
  if (getSpecifier(RefExp) == PPC::S_NOTOC)
    O << '@' << MAI.getSpecifierName(RefExp->getKind());
  O << '(';
  printOperand(MI, OpNo + 1, STI, O);
  O << ')';
  if (getSpecifier(RefExp) != PPC::S_None &&
      getSpecifier(RefExp) != PPC::S_NOTOC)
    O << '@' << MAI.getSpecifierName(RefExp->getKind());
  if (Rhs) {
    SmallString<0> Buf;
    raw_svector_ostream Tmp(Buf);
    MAI.printExpr(Tmp, *Rhs);
    if (isdigit(Buf[0]))
      O << '+';
    O << Buf;
  }
}

/// showRegistersWithPercentPrefix - Check if this register name should be
/// printed with a percentage symbol as prefix.
bool PPCInstPrinter::showRegistersWithPercentPrefix(const char *RegName) const {
  if ((!FullRegNamesWithPercent && !MAI.useFullRegisterNames()) ||
      TT.getOS() == Triple::AIX)
    return false;

  switch (RegName[0]) {
  default:
    return false;
  case 'r':
  case 'f':
  case 'q':
  case 'v':
  case 'c':
    return true;
```
- **EN**: Implements helper routine(s) `__tls_get_addr`, `notoc`, `getSpecifier` for this portion of the PowerPC backend MC layer support for the backend.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 MC 层支持所需的辅助例程 `__tls_get_addr`, `notoc`, `getSpecifier`。

### Lines 580-617

```cpp
  }
}

/// getVerboseConditionalRegName - This method expands the condition register
/// when requested explicitly or targetting Darwin.
const char *
PPCInstPrinter::getVerboseConditionRegName(MCRegister Reg,
                                           unsigned RegEncoding) const {
  if (!FullRegNames && !MAI.useFullRegisterNames())
    return nullptr;
  if (Reg < PPC::CR0EQ || Reg > PPC::CR7UN)
    return nullptr;
  const char *CRBits[] = {
    "lt", "gt", "eq", "un",
    "4*cr1+lt", "4*cr1+gt", "4*cr1+eq", "4*cr1+un",
    "4*cr2+lt", "4*cr2+gt", "4*cr2+eq", "4*cr2+un",
    "4*cr3+lt", "4*cr3+gt", "4*cr3+eq", "4*cr3+un",
    "4*cr4+lt", "4*cr4+gt", "4*cr4+eq", "4*cr4+un",
    "4*cr5+lt", "4*cr5+gt", "4*cr5+eq", "4*cr5+un",
    "4*cr6+lt", "4*cr6+gt", "4*cr6+eq", "4*cr6+un",
    "4*cr7+lt", "4*cr7+gt", "4*cr7+eq", "4*cr7+un"
  };
  return CRBits[RegEncoding];
}

// showRegistersWithPrefix - This method determines whether registers
// should be number-only or include the prefix.
bool PPCInstPrinter::showRegistersWithPrefix() const {
  return FullRegNamesWithPercent || FullRegNames || MAI.useFullRegisterNames();
}

void PPCInstPrinter::printOperand(const MCInst *MI, unsigned OpNo,
                                  const MCSubtargetInfo &STI, raw_ostream &O) {
  const MCOperand &Op = MI->getOperand(OpNo);
  if (Op.isReg()) {
    MCRegister Reg = Op.getReg();
    if (!ShowVSRNumsAsVR)
      Reg = PPC::getRegNumForOperand(MII.get(MI->getOpcode()), Reg, OpNo);
```
- **EN**: Implements helper routine(s) `getVerboseConditionRegName`, `useFullRegisterNames`, `showRegistersWithPrefix` for this portion of the PowerPC backend MC layer support for the backend. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 MC 层支持所需的辅助例程 `getVerboseConditionRegName`, `useFullRegisterNames`, `showRegistersWithPrefix`。 子目标特性裁剪会影响这里的行为。

### Lines 618-639

```cpp

    const char *RegName;
    RegName = getVerboseConditionRegName(Reg, MRI.getEncodingValue(Reg));
    if (RegName == nullptr)
     RegName = getRegisterName(Reg);
    if (showRegistersWithPercentPrefix(RegName))
      O << "%";
    if (!showRegistersWithPrefix())
      RegName = PPC::stripRegisterPrefix(RegName);

    O << RegName;
    return;
  }

  if (Op.isImm()) {
    O << Op.getImm();
    return;
  }

  assert(Op.isExpr() && "unknown operand kind in printOperand");
  MAI.printExpr(O, *Op.getExpr());
}
```
- **EN**: Implements helper routine(s) `getVerboseConditionRegName`, `getEncodingValue`, `getRegisterName` for this portion of the PowerPC backend MC layer support for the backend. Register assignment and register-class constraints matter here.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 MC 层支持所需的辅助例程 `getVerboseConditionRegName`, `getEncodingValue`, `getRegisterName`。 这里重点涉及寄存器分配与寄存器类约束。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- MC layer support for the backend / 该后端的 MC 层支持
- Register modeling / 寄存器建模
- Subtarget features / 子目标特性
- Instruction selection or opcode handling / 指令选择或操作码处理
- Object format integration / 目标文件格式集成
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `MCTargetDesc/PPCInstPrinter.h`
- `MCTargetDesc/PPCMCTargetDesc.h`
- `MCTargetDesc/PPCPredicates.h`
- `PPCMCAsmInfo.h`
- `llvm/MC/MCAsmInfo.h`
- `llvm/MC/MCExpr.h`
- `llvm/MC/MCInst.h`
- `llvm/MC/MCInstrInfo.h`
- `llvm/MC/MCRegisterInfo.h`
- `llvm/MC/MCSubtargetInfo.h`
- `llvm/MC/MCSymbol.h`
- `llvm/Support/Casting.h`
- `llvm/Support/CommandLine.h`
- `llvm/Support/raw_ostream.h`
- `PPCGenAsmWriter.inc`

### Important Collaborators / 重要协作组件

- LLVM MC layer / LLVM MC 层
- LLVM Support utilities / LLVM Support 工具层
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
