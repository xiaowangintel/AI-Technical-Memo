# VEInstPrinter.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/VE/MCTargetDesc/VEInstPrinter.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Provides MC-layer target descriptions such as fixups, asm info, code emission, object writing, and target registration.
  - **CN**: 提供 MC 层的目标描述，例如 fixup、汇编信息、编码发射、目标文件写出以及目标注册。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- VEInstPrinter.cpp - Convert VE MCInst to assembly syntax -----------==//
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
// This class prints an VE MCInst to a .s file.
//
//===----------------------------------------------------------------------===//

#include "VEInstPrinter.h"
#include "VE.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCRegisterInfo.h"
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/MC/MCSymbol.h"
#include "llvm/Support/raw_ostream.h"
```
- **EN**: Pulls in the headers needed for this implementation, including `VEInstPrinter.h`, `VE.h`, `llvm/MC/MCAsmInfo.h`, `llvm/MC/MCExpr.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `VEInstPrinter.h`, `VE.h`, `llvm/MC/MCAsmInfo.h`, `llvm/MC/MCExpr.h`。

### Lines 22-30
```cpp

using namespace llvm;

#define DEBUG_TYPE "ve-asmprinter"

#define GET_INSTRUCTION_NAME
#define PRINT_ALIAS_INSTR
#include "VEGenAsmWriter.inc"

```
- **EN**: Pulls in the headers needed for this implementation, including `VEGenAsmWriter.inc`.
- **CN**: 引入该实现所需的头文件，其中包括 `VEGenAsmWriter.inc`。

### Lines 31-39
```cpp
void VEInstPrinter::printRegName(raw_ostream &OS, MCRegister Reg) {
  // Generic registers have identical register name among register classes.
  unsigned AltIdx = VE::AsmName;
  // Misc registers have each own name, so no use alt-names.
  if (MRI.getRegClass(VE::MISCRegClassID).contains(Reg))
    AltIdx = VE::NoRegAltName;
  OS << '%' << getRegisterName(Reg, AltIdx);
}

```
- **EN**: Implements logic around `printRegName`, `getRegisterName`; this block applies conditional target rules.
- **CN**: 围绕 `printRegName`, `getRegisterName` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 40-47
```cpp
void VEInstPrinter::printInst(const MCInst *MI, uint64_t Address,
                              StringRef Annot, const MCSubtargetInfo &STI,
                              raw_ostream &OS) {
  if (!printAliasInstr(MI, Address, STI, OS))
    printInstruction(MI, Address, STI, OS);
  printAnnotation(OS, Annot);
}

```
- **EN**: Implements logic around `printInst`, `printInstruction`, `printAnnotation`; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `printInst`, `printInstruction`, `printAnnotation` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 48-56
```cpp
void VEInstPrinter::printOperand(const MCInst *MI, int OpNum,
                                 const MCSubtargetInfo &STI, raw_ostream &O) {
  const MCOperand &MO = MI->getOperand(OpNum);

  if (MO.isReg()) {
    printRegName(O, MO.getReg());
    return;
  }

```
- **EN**: Implements logic around `printOperand`, `getOperand`, `printRegName`; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `printOperand`, `getOperand`, `printRegName` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 57-63
```cpp
  if (MO.isImm()) {
    // Expects signed 32bit literals.
    int32_t TruncatedImm = static_cast<int32_t>(MO.getImm());
    O << TruncatedImm;
    return;
  }

```
- **EN**: Implements logic around `static_cast<int32_t>`; this block applies conditional target rules.
- **CN**: 围绕 `static_cast<int32_t>` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 64-77
```cpp
  assert(MO.isExpr() && "Unknown operand kind in printOperand");
  MAI.printExpr(O, *MO.getExpr());
}

void VEInstPrinter::printMemASXOperand(const MCInst *MI, int OpNum,
                                       const MCSubtargetInfo &STI,
                                       raw_ostream &O) {
  if (MI->getOperand(OpNum + 2).isImm() &&
      MI->getOperand(OpNum + 2).getImm() == 0) {
    // don't print "+0"
  } else {
    printOperand(MI, OpNum + 2, STI, O);
  }
  if (MI->getOperand(OpNum + 1).isImm() &&
```
- **EN**: Implements logic around `assert`, `printExpr`, `printMemASXOperand`, `getOperand`, ...; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `assert`, `printExpr`, `printMemASXOperand`, `getOperand`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 78-91
```cpp
      MI->getOperand(OpNum + 1).getImm() == 0 &&
      MI->getOperand(OpNum).isImm() && MI->getOperand(OpNum).getImm() == 0) {
    if (MI->getOperand(OpNum + 2).isImm() &&
        MI->getOperand(OpNum + 2).getImm() == 0) {
      O << "0";
    } else {
      // don't print "+0,+0"
    }
  } else {
    O << "(";
    if (MI->getOperand(OpNum + 1).isImm() &&
        MI->getOperand(OpNum + 1).getImm() == 0) {
      // don't print "+0"
    } else {
```
- **EN**: Implements logic around `getOperand`; this block applies conditional target rules.
- **CN**: 围绕 `getOperand` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 92-103
```cpp
      printOperand(MI, OpNum + 1, STI, O);
    }
    if (MI->getOperand(OpNum).isImm() && MI->getOperand(OpNum).getImm() == 0) {
      // don't print "+0"
    } else {
      O << ", ";
      printOperand(MI, OpNum, STI, O);
    }
    O << ")";
  }
}

```
- **EN**: Implements logic around `printOperand`; this block applies conditional target rules.
- **CN**: 围绕 `printOperand` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 104-117
```cpp
void VEInstPrinter::printMemASOperandASX(const MCInst *MI, int OpNum,
                                         const MCSubtargetInfo &STI,
                                         raw_ostream &O) {
  if (MI->getOperand(OpNum + 1).isImm() &&
      MI->getOperand(OpNum + 1).getImm() == 0) {
    // don't print "+0"
  } else {
    printOperand(MI, OpNum + 1, STI, O);
  }
  if (MI->getOperand(OpNum).isImm() && MI->getOperand(OpNum).getImm() == 0) {
    if (MI->getOperand(OpNum + 1).isImm() &&
        MI->getOperand(OpNum + 1).getImm() == 0) {
      O << "0";
    } else {
```
- **EN**: Implements logic around `printMemASOperandASX`, `getOperand`, `printOperand`; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `printMemASOperandASX`, `getOperand`, `printOperand` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 118-126
```cpp
      // don't print "(0)"
    }
  } else {
    O << "(, ";
    printOperand(MI, OpNum, STI, O);
    O << ")";
  }
}

```
- **EN**: Implements logic around `printOperand`.
- **CN**: 围绕 `printOperand` 实现具体逻辑。

### Lines 127-140
```cpp
void VEInstPrinter::printMemASOperandRRM(const MCInst *MI, int OpNum,
                                         const MCSubtargetInfo &STI,
                                         raw_ostream &O) {
  if (MI->getOperand(OpNum + 1).isImm() &&
      MI->getOperand(OpNum + 1).getImm() == 0) {
    // don't print "+0"
  } else {
    printOperand(MI, OpNum + 1, STI, O);
  }
  if (MI->getOperand(OpNum).isImm() && MI->getOperand(OpNum).getImm() == 0) {
    if (MI->getOperand(OpNum + 1).isImm() &&
        MI->getOperand(OpNum + 1).getImm() == 0) {
      O << "0";
    } else {
```
- **EN**: Implements logic around `printMemASOperandRRM`, `getOperand`, `printOperand`; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `printMemASOperandRRM`, `getOperand`, `printOperand` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 141-149
```cpp
      // don't print "(0)"
    }
  } else {
    O << "(";
    printOperand(MI, OpNum, STI, O);
    O << ")";
  }
}

```
- **EN**: Implements logic around `printOperand`.
- **CN**: 围绕 `printOperand` 实现具体逻辑。

### Lines 150-163
```cpp
void VEInstPrinter::printMemASOperandHM(const MCInst *MI, int OpNum,
                                        const MCSubtargetInfo &STI,
                                        raw_ostream &O) {
  if (MI->getOperand(OpNum + 1).isImm() &&
      MI->getOperand(OpNum + 1).getImm() == 0) {
    // don't print "+0"
  } else {
    printOperand(MI, OpNum + 1, STI, O);
  }
  O << "(";
  if (MI->getOperand(OpNum).isReg())
    printOperand(MI, OpNum, STI, O);
  O << ")";
}
```
- **EN**: Implements logic around `printMemASOperandHM`, `getOperand`, `printOperand`; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `printMemASOperandHM`, `getOperand`, `printOperand` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 164-174
```cpp

void VEInstPrinter::printMImmOperand(const MCInst *MI, int OpNum,
                                     const MCSubtargetInfo &STI,
                                     raw_ostream &O) {
  int MImm = (int)MI->getOperand(OpNum).getImm() & 0x7f;
  if (MImm > 63)
    O << "(" << MImm - 64 << ")0";
  else
    O << "(" << MImm << ")1";
}

```
- **EN**: Implements logic around `printMImmOperand`, `getOperand`; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `printMImmOperand`, `getOperand` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 175-185
```cpp
void VEInstPrinter::printCCOperand(const MCInst *MI, int OpNum,
                                   const MCSubtargetInfo &STI, raw_ostream &O) {
  int CC = (int)MI->getOperand(OpNum).getImm();
  O << VECondCodeToString((VECC::CondCode)CC);
}

void VEInstPrinter::printRDOperand(const MCInst *MI, int OpNum,
                                   const MCSubtargetInfo &STI, raw_ostream &O) {
  int RD = (int)MI->getOperand(OpNum).getImm();
  O << VERDToString((VERD::RoundingMode)RD);
}
```
- **EN**: Implements logic around `printCCOperand`, `getOperand`, `VECondCodeToString`, `printRDOperand`, ...; this block works at the MC layer.
- **CN**: 围绕 `printCCOperand`, `getOperand`, `VECondCodeToString`, `printRDOperand`, ... 实现具体逻辑；这一段工作在 MC 层。

## Key Concepts / 关键概念

- **MC layer / MC 层**:
  - **EN**: Connects the backend to LLVM's MC infrastructure
  - **CN**: 把后端接入 LLVM 的 MC 基础设施

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `VEInstPrinter.h`, `VE.h`, `llvm/MC/MCAsmInfo.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCInst.h`, `llvm/MC/MCRegisterInfo.h`, `llvm/MC/MCSubtargetInfo.h`, `llvm/MC/MCSymbol.h`, `llvm/Support/raw_ostream.h`, `VEGenAsmWriter.inc`
- **LLVM subsystems / LLVM 子系统**: MC, Support
- **Generated macros / 生成宏**: `GET_INSTRUCTION_NAME`
