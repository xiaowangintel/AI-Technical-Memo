# SparcInstPrinter.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Sparc/MCTargetDesc/SparcInstPrinter.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Provides MC-layer target descriptions such as fixups, asm info, code emission, object writing, and target registration.
  - **CN**: 提供 MC 层的目标描述，例如 fixup、汇编信息、编码发射、目标文件写出以及目标注册。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- SparcInstPrinter.cpp - Convert Sparc MCInst to assembly syntax -----==//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 8-21
```cpp
//
// This class prints an Sparc MCInst to a .s file.
//
//===----------------------------------------------------------------------===//

#include "SparcInstPrinter.h"
#include "Sparc.h"
#include "llvm/ADT/StringExtras.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/MC/MCSymbol.h"
#include "llvm/Support/raw_ostream.h"
```
- **EN**: Pulls in the headers needed for this implementation, including `SparcInstPrinter.h`, `Sparc.h`, `llvm/ADT/StringExtras.h`, `llvm/MC/MCAsmInfo.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `SparcInstPrinter.h`, `Sparc.h`, `llvm/ADT/StringExtras.h`, `llvm/MC/MCAsmInfo.h`。

### Lines 22-33
```cpp
using namespace llvm;

#define DEBUG_TYPE "asm-printer"

// The generated AsmMatcher SparcGenAsmWriter uses "Sparc" as the target
// namespace. But SPARC backend uses "SP" as its namespace.
namespace llvm {
namespace Sparc {
  using namespace SP;
}
}

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 34-41
```cpp
#define GET_INSTRUCTION_NAME
#define PRINT_ALIAS_INSTR
#include "SparcGenAsmWriter.inc"

bool SparcInstPrinter::isV9(const MCSubtargetInfo &STI) const {
  return (STI.hasFeature(Sparc::FeatureV9)) != 0;
}

```
- **EN**: Pulls in the headers needed for this implementation, including `SparcGenAsmWriter.inc`.
- **CN**: 引入该实现所需的头文件，其中包括 `SparcGenAsmWriter.inc`。

### Lines 42-50
```cpp
void SparcInstPrinter::printRegName(raw_ostream &OS, MCRegister Reg) {
  OS << '%' << getRegisterName(Reg);
}

void SparcInstPrinter::printRegName(raw_ostream &OS, MCRegister Reg,
                                    unsigned AltIdx) const {
  OS << '%' << getRegisterName(Reg, AltIdx);
}

```
- **EN**: Implements logic around `printRegName`, `getRegisterName`.
- **CN**: 围绕 `printRegName`, `getRegisterName` 实现具体逻辑。

### Lines 51-59
```cpp
void SparcInstPrinter::printInst(const MCInst *MI, uint64_t Address,
                                 StringRef Annot, const MCSubtargetInfo &STI,
                                 raw_ostream &O) {
  if (!printAliasInstr(MI, Address, STI, O) &&
      !printSparcAliasInstr(MI, STI, O))
    printInstruction(MI, Address, STI, O);
  printAnnotation(O, Annot);
}

```
- **EN**: Implements logic around `printInst`, `printSparcAliasInstr`, `printInstruction`, `printAnnotation`; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `printInst`, `printSparcAliasInstr`, `printInstruction`, `printAnnotation` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 60-73
```cpp
bool SparcInstPrinter::printSparcAliasInstr(const MCInst *MI,
                                            const MCSubtargetInfo &STI,
                                            raw_ostream &O) {
  switch (MI->getOpcode()) {
  default: return false;
  case SP::JMPLrr:
  case SP::JMPLri: {
    if (MI->getNumOperands() != 3)
      return false;
    if (!MI->getOperand(0).isReg())
      return false;
    switch (MI->getOperand(0).getReg().id()) {
    default: return false;
    case SP::G0: // jmp $addr | ret | retl
```
- **EN**: Implements logic around `printSparcAliasInstr`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results; works at the MC layer.
- **CN**: 围绕 `printSparcAliasInstr` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果，工作在 MC 层。

### Lines 74-87
```cpp
      if (MI->getOperand(2).isImm() &&
          MI->getOperand(2).getImm() == 8) {
        switch (MI->getOperand(1).getReg().id()) {
        default: break;
        case SP::I7: O << "\tret"; return true;
        case SP::O7: O << "\tretl"; return true;
        }
      }
      O << "\tjmp "; printMemOperand(MI, 1, STI, O);
      return true;
    case SP::O7: // call $addr
      O << "\tcall "; printMemOperand(MI, 1, STI, O);
      return true;
    }
```
- **EN**: Implements logic around `getOperand`, `printMemOperand`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getOperand`, `printMemOperand` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果。

### Lines 88-101
```cpp
  }
  case SP::V9FCMPS:  case SP::V9FCMPD:  case SP::V9FCMPQ:
  case SP::V9FCMPES: case SP::V9FCMPED: case SP::V9FCMPEQ: {
    if (isV9(STI)
        || (MI->getNumOperands() != 3)
        || (!MI->getOperand(0).isReg())
        || (MI->getOperand(0).getReg() != SP::FCC0))
      return false;
    // if V8, skip printing %fcc0.
    switch(MI->getOpcode()) {
    default:
    case SP::V9FCMPS:  O << "\tfcmps "; break;
    case SP::V9FCMPD:  O << "\tfcmpd "; break;
    case SP::V9FCMPQ:  O << "\tfcmpq "; break;
```
- **EN**: Implements logic around `getNumOperands`, `getOperand`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getNumOperands`, `getOperand` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果。

### Lines 102-113
```cpp
    case SP::V9FCMPES: O << "\tfcmpes "; break;
    case SP::V9FCMPED: O << "\tfcmped "; break;
    case SP::V9FCMPEQ: O << "\tfcmpeq "; break;
    }
    printOperand(MI, 1, STI, O);
    O << ", ";
    printOperand(MI, 2, STI, O);
    return true;
  }
  }
}

```
- **EN**: Implements logic around `printOperand`; this block returns target-specific results.
- **CN**: 围绕 `printOperand` 实现具体逻辑；这一段返回目标相关结果。

### Lines 114-127
```cpp
void SparcInstPrinter::printOperand(const MCInst *MI, int opNum,
                                    const MCSubtargetInfo &STI,
                                    raw_ostream &O) {
  const MCOperand &MO = MI->getOperand (opNum);

  if (MO.isReg()) {
    MCRegister Reg = MO.getReg();
    if (isV9(STI))
      printRegName(O, Reg, SP::RegNamesStateReg);
    else
      printRegName(O, Reg);
    return ;
  }

```
- **EN**: Implements logic around `printOperand`, `getOperand`, `getReg`, `printRegName`; this block applies conditional target rules; returns target-specific results; works at the MC layer.
- **CN**: 围绕 `printOperand`, `getOperand`, `getReg`, `printRegName` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MC 层。

### Lines 128-141
```cpp
  if (MO.isImm()) {
    switch (MI->getOpcode()) {
      default:
        markup(O, Markup::Immediate) << formatImm(int32_t(MO.getImm()));
        return;

      case SP::TICCri: // Fall through
      case SP::TICCrr: // Fall through
      case SP::TRAPri: // Fall through
      case SP::TRAPrr: // Fall through
      case SP::TXCCri: // Fall through
      case SP::TXCCrr: // Fall through
        // Only seven-bit values up to 127.
        O << ((int) MO.getImm() & 0x7f);
```
- **EN**: Implements logic around `markup`, `getImm`; this block uses `switch`-based dispatch; applies conditional target rules.
- **CN**: 围绕 `markup`, `getImm` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则。

### Lines 142-149
```cpp
        return;
    }
  }

  assert(MO.isExpr() && "Unknown operand kind in printOperand");
  MAI.printExpr(O, *MO.getExpr());
}

```
- **EN**: Implements logic around `assert`, `printExpr`.
- **CN**: 围绕 `assert`, `printExpr` 实现具体逻辑。

### Lines 150-161
```cpp
void SparcInstPrinter::printMemOperand(const MCInst *MI, int opNum,
                                       const MCSubtargetInfo &STI,
                                       raw_ostream &O) {
  const MCOperand &Op1 = MI->getOperand(opNum);
  const MCOperand &Op2 = MI->getOperand(opNum + 1);

  bool PrintedFirstOperand = false;
  if (Op1.isReg() && Op1.getReg() != SP::G0) {
    printOperand(MI, opNum, STI, O);
    PrintedFirstOperand = true;
  }

```
- **EN**: Implements logic around `printMemOperand`, `getOperand`, `printOperand`; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `printMemOperand`, `getOperand`, `printOperand` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 162-174
```cpp
  // Skip the second operand iff it adds nothing (literal 0 or %g0) and we've
  // already printed the first one
  const bool SkipSecondOperand =
      PrintedFirstOperand && ((Op2.isReg() && Op2.getReg() == SP::G0) ||
                              (Op2.isImm() && Op2.getImm() == 0));

  if (!SkipSecondOperand) {
    if (PrintedFirstOperand)
      O << '+';
    printOperand(MI, opNum + 1, STI, O);
  }
}

```
- **EN**: Implements logic around `isReg`, `isImm`, `printOperand`; this block applies conditional target rules.
- **CN**: 围绕 `isReg`, `isImm`, `printOperand` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 175-188
```cpp
void SparcInstPrinter::printCCOperand(const MCInst *MI, int opNum,
                                      const MCSubtargetInfo &STI,
                                      raw_ostream &O) {
  int CC = (int)MI->getOperand(opNum).getImm();
  switch (MI->getOpcode()) {
  default: break;
  case SP::FBCOND:
  case SP::FBCONDA:
  case SP::FBCOND_V9:
  case SP::FBCONDA_V9:
  case SP::BPFCC:
  case SP::BPFCCA:
  case SP::BPFCCNT:
  case SP::BPFCCANT:
```
- **EN**: Implements logic around `printCCOperand`, `getOperand`; this block uses `switch`-based dispatch; works at the MC layer.
- **CN**: 围绕 `printCCOperand`, `getOperand` 实现具体逻辑；这一段使用 `switch` 分派，工作在 MC 层。

### Lines 189-202
```cpp
  case SP::MOVFCCrr:  case SP::V9MOVFCCrr:
  case SP::MOVFCCri:  case SP::V9MOVFCCri:
  case SP::FMOVS_FCC: case SP::V9FMOVS_FCC:
  case SP::FMOVD_FCC: case SP::V9FMOVD_FCC:
  case SP::FMOVQ_FCC: case SP::V9FMOVQ_FCC:
    // Make sure CC is a fp conditional flag.
    CC = (CC < SPCC::FCC_BEGIN) ? (CC + SPCC::FCC_BEGIN) : CC;
    break;
  case SP::CPBCOND:
  case SP::CPBCONDA:
    // Make sure CC is a cp conditional flag.
    CC = (CC < SPCC::CPCC_BEGIN) ? (CC + SPCC::CPCC_BEGIN) : CC;
    break;
  case SP::BPR:
```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 203-216
```cpp
  case SP::BPRA:
  case SP::BPRNT:
  case SP::BPRANT:
  case SP::MOVRri:
  case SP::MOVRrr:
  case SP::FMOVRS:
  case SP::FMOVRD:
  case SP::FMOVRQ:
    // Make sure CC is a register conditional flag.
    CC = (CC < SPCC::REG_BEGIN) ? (CC + SPCC::REG_BEGIN) : CC;
    break;
  }
  O << SPARCCondCodeToString((SPCC::CondCodes)CC);
}
```
- **EN**: Implements logic around `SPARCCondCodeToString`.
- **CN**: 围绕 `SPARCCondCodeToString` 实现具体逻辑。

### Lines 217-224
```cpp

bool SparcInstPrinter::printGetPCX(const MCInst *MI, unsigned opNum,
                                   const MCSubtargetInfo &STI,
                                   raw_ostream &O) {
  llvm_unreachable("FIXME: Implement SparcInstPrinter::printGetPCX.");
  return true;
}

```
- **EN**: Implements logic around `printGetPCX`, `llvm_unreachable`; this block returns target-specific results; works at the MC layer.
- **CN**: 围绕 `printGetPCX`, `llvm_unreachable` 实现具体逻辑；这一段返回目标相关结果，工作在 MC 层。

### Lines 225-231
```cpp
void SparcInstPrinter::printMembarTag(const MCInst *MI, int opNum,
                                      const MCSubtargetInfo &STI,
                                      raw_ostream &O) {
  static const char *const TagNames[] = {
      "#LoadLoad",  "#StoreLoad", "#LoadStore", "#StoreStore",
      "#Lookaside", "#MemIssue",  "#Sync"};

```
- **EN**: Implements logic around `printMembarTag`; this block works at the MC layer.
- **CN**: 围绕 `printMembarTag` 实现具体逻辑；这一段工作在 MC 层。

### Lines 232-238
```cpp
  unsigned Imm = MI->getOperand(opNum).getImm();

  if (Imm > 127) {
    O << Imm;
    return;
  }

```
- **EN**: Implements logic around `getOperand`; this block applies conditional target rules.
- **CN**: 围绕 `getOperand` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 239-245
```cpp
  ListSeparator LS(" | ");
  for (unsigned i = 0; i < std::size(TagNames); i++) {
    if (Imm & (1 << i))
      O << LS << TagNames[i];
  }
}

```
- **EN**: Implements logic around `LS`; this block applies conditional target rules.
- **CN**: 围绕 `LS` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 246-255
```cpp
void SparcInstPrinter::printASITag(const MCInst *MI, int opNum,
                                   const MCSubtargetInfo &STI, raw_ostream &O) {
  unsigned Imm = MI->getOperand(opNum).getImm();
  auto ASITag = SparcASITag::lookupASITagByEncoding(Imm);
  if (isV9(STI) && ASITag)
    O << '#' << ASITag->Name;
  else
    O << Imm;
}

```
- **EN**: Implements logic around `printASITag`, `getOperand`, `lookupASITagByEncoding`; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `printASITag`, `getOperand`, `lookupASITagByEncoding` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 256-266
```cpp
void SparcInstPrinter::printPrefetchTag(const MCInst *MI, int opNum,
                                        const MCSubtargetInfo &STI,
                                        raw_ostream &O) {
  unsigned Imm = MI->getOperand(opNum).getImm();
  auto PrefetchTag = SparcPrefetchTag::lookupPrefetchTagByEncoding(Imm);
  if (PrefetchTag)
    O << '#' << PrefetchTag->Name;
  else
    O << Imm;
}

```
- **EN**: Implements logic around `printPrefetchTag`, `getOperand`, `lookupPrefetchTagByEncoding`; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `printPrefetchTag`, `getOperand`, `lookupPrefetchTagByEncoding` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 267-280
```cpp
void SparcInstPrinter::printCTILabel(const MCInst *MI, uint64_t Address,
                                     unsigned OpNum, const MCSubtargetInfo &STI,
                                     raw_ostream &O) {
  const MCOperand &Op = MI->getOperand(OpNum);

  // If the label has already been resolved to an immediate offset (say, when
  // we're running the disassembler), just print the immediate.
  if (Op.isImm()) {
    int64_t Offset = Op.getImm();
    if (PrintBranchImmAsAddress) {
      uint64_t Target = Address + Offset;
      if (STI.getTargetTriple().isSPARC32())
        Target &= 0xffffffff;
      O << formatHex(Target);
```
- **EN**: Implements logic around `printCTILabel`, `getOperand`, `getImm`, `formatHex`; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `printCTILabel`, `getOperand`, `getImm`, `formatHex` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 281-289
```cpp
    } else {
      O << ".";
      if (Offset >= 0)
        O << "+";
      O << Offset;
    }
    return;
  }

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 290-292
```cpp
  // Otherwise, just print the expression.
  MAI.printExpr(O, *Op.getExpr());
}
```
- **EN**: Implements logic around `printExpr`.
- **CN**: 围绕 `printExpr` 实现具体逻辑。

## Key Concepts / 关键概念

- **MC layer / MC 层**:
  - **EN**: Connects the backend to LLVM's MC infrastructure
  - **CN**: 把后端接入 LLVM 的 MC 基础设施

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `SparcInstPrinter.h`, `Sparc.h`, `llvm/ADT/StringExtras.h`, `llvm/MC/MCAsmInfo.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCInst.h`, `llvm/MC/MCSubtargetInfo.h`, `llvm/MC/MCSymbol.h`, `llvm/Support/raw_ostream.h`, `SparcGenAsmWriter.inc`
- **LLVM subsystems / LLVM 子系统**: MC, Support
- **Generated macros / 生成宏**: `GET_INSTRUCTION_NAME`
