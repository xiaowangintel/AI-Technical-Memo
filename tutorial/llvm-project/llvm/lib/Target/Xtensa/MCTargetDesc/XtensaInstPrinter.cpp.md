# XtensaInstPrinter.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Xtensa/MCTargetDesc/XtensaInstPrinter.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Provides MC-layer target descriptions such as fixups, asm info, code emission, object writing, and target registration.
  - **CN**: 提供 MC 层的目标描述，例如 fixup、汇编信息、编码发射、目标文件写出以及目标注册。

## Line-by-Line Analysis / 逐行分析

### Lines 1-9
```cpp
//===- XtensaInstPrinter.cpp - Convert Xtensa MCInst to asm syntax --------===//
//
//                     The LLVM Compiler Infrastructure
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 10-25
```cpp
//
// This class prints an Xtensa MCInst to a .s file.
//
//===----------------------------------------------------------------------===//

#include "XtensaInstPrinter.h"
#include "MCTargetDesc/XtensaMCAsmInfo.h"
#include "llvm/CodeGen/MachineOperand.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCInstrInfo.h"
#include "llvm/MC/MCRegister.h"
#include "llvm/MC/MCSymbol.h"
#include "llvm/Support/Casting.h"
#include "llvm/Support/raw_ostream.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `XtensaInstPrinter.h`, `MCTargetDesc/XtensaMCAsmInfo.h`, `llvm/CodeGen/MachineOperand.h`, `llvm/MC/MCAsmInfo.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `XtensaInstPrinter.h`, `MCTargetDesc/XtensaMCAsmInfo.h`, `llvm/CodeGen/MachineOperand.h`, `llvm/MC/MCAsmInfo.h`。

### Lines 26-35
```cpp
using namespace llvm;

#define DEBUG_TYPE "asm-printer"

#include "XtensaGenAsmWriter.inc"

static void printExpr(const MCExpr *Expr, raw_ostream &OS) {
  int Offset = 0;
  const MCSymbolRefExpr *SRE;

```
- **EN**: Pulls in the headers needed for this implementation, including `XtensaGenAsmWriter.inc`.
- **CN**: 引入该实现所需的头文件，其中包括 `XtensaGenAsmWriter.inc`。

### Lines 36-49
```cpp
  if (!(SRE = cast<MCSymbolRefExpr>(Expr)))
    assert(false && "Unexpected MCExpr type.");

  assert(SRE->getSpecifier() == 0);

  OS << SRE->getSymbol();

  if (Offset) {
    if (Offset > 0)
      OS << '+';
    OS << Offset;
  }
}

```
- **EN**: Implements logic around `assert`, `getSymbol`; this block applies conditional target rules.
- **CN**: 围绕 `assert`, `getSymbol` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 50-60
```cpp
void XtensaInstPrinter::printOperand(const MCOperand &MC, raw_ostream &O) {
  if (MC.isReg())
    O << getRegisterName(MC.getReg());
  else if (MC.isImm())
    O << MC.getImm();
  else if (MC.isExpr())
    printExpr(MC.getExpr(), O);
  else
    report_fatal_error("Invalid operand");
}

```
- **EN**: Implements logic around `printOperand`, `getRegisterName`, `getImm`, `printExpr`, ...; this block applies conditional target rules.
- **CN**: 围绕 `printOperand`, `getRegisterName`, `getImm`, `printExpr`, ... 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 61-71
```cpp
void XtensaInstPrinter::printInst(const MCInst *MI, uint64_t Address,
                                  StringRef Annot, const MCSubtargetInfo &STI,
                                  raw_ostream &O) {
  printInstruction(MI, Address, O);
  printAnnotation(O, Annot);
}

void XtensaInstPrinter::printRegName(raw_ostream &O, MCRegister Reg) {
  O << getRegisterName(Reg);
}

```
- **EN**: Implements logic around `printInst`, `printInstruction`, `printAnnotation`, `printRegName`, ...; this block works at the MC layer.
- **CN**: 围绕 `printInst`, `printInstruction`, `printAnnotation`, `printRegName`, ... 实现具体逻辑；这一段工作在 MC 层。

### Lines 72-83
```cpp
void XtensaInstPrinter::printOperand(const MCInst *MI, int OpNum,
                                     raw_ostream &O) {
  printOperand(MI->getOperand(OpNum), O);
}

void XtensaInstPrinter::printMemOperand(const MCInst *MI, int OpNum,
                                        raw_ostream &OS) {
  OS << getRegisterName(MI->getOperand(OpNum).getReg());
  OS << ", ";
  printOperand(MI, OpNum + 1, OS);
}

```
- **EN**: Implements logic around `printOperand`, `printMemOperand`, `getRegisterName`; this block works at the MC layer.
- **CN**: 围绕 `printOperand`, `printMemOperand`, `getRegisterName` 实现具体逻辑；这一段工作在 MC 层。

### Lines 84-95
```cpp
void XtensaInstPrinter::printBranchTarget(const MCInst *MI, uint64_t Address,
                                          int OpNum, raw_ostream &O) {
  const MCOperand &MC = MI->getOperand(OpNum);
  if (MI->getOperand(OpNum).isImm()) {
    int64_t Val = MC.getImm() + 4;
    printPCRelImm(Address, Val, O);
  } else if (MC.isExpr())
    MAI.printExpr(O, *MC.getExpr());
  else
    llvm_unreachable("Invalid operand");
}

```
- **EN**: Implements logic around `printBranchTarget`, `getOperand`, `getImm`, `printPCRelImm`, ...; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `printBranchTarget`, `getOperand`, `getImm`, `printPCRelImm`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 96-107
```cpp
void XtensaInstPrinter::printLoopTarget(const MCInst *MI, uint64_t Address,
                                        int OpNum, raw_ostream &O) {
  const MCOperand &MC = MI->getOperand(OpNum);
  if (MI->getOperand(OpNum).isImm()) {
    int64_t Val = MC.getImm() + 4;
    printPCRelImm(Address, Val, O);
  } else if (MC.isExpr())
    MAI.printExpr(O, *MC.getExpr());
  else
    llvm_unreachable("Invalid operand");
}

```
- **EN**: Implements logic around `printLoopTarget`, `getOperand`, `getImm`, `printPCRelImm`, ...; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `printLoopTarget`, `getOperand`, `getImm`, `printPCRelImm`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 108-120
```cpp
void XtensaInstPrinter::printJumpTarget(const MCInst *MI, uint64_t Address,
                                        int OpNum, raw_ostream &O) {
  const MCOperand &MC = MI->getOperand(OpNum);
  if (MC.isImm()) {
    int64_t Val = MC.getImm() + 4;
    printPCRelImm(Address, Val, O);
  } else if (MC.isExpr())
    MAI.printExpr(O, *MC.getExpr());
  else
    llvm_unreachable("Invalid operand");
  ;
}

```
- **EN**: Implements logic around `printJumpTarget`, `getOperand`, `getImm`, `printPCRelImm`, ...; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `printJumpTarget`, `getOperand`, `getImm`, `printPCRelImm`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 121-138
```cpp
void XtensaInstPrinter::printCallOperand(const MCInst *MI, uint64_t Address,
                                         int OpNum, raw_ostream &O) {
  const MCOperand &MC = MI->getOperand(OpNum);
  if (MC.isImm()) {
    int64_t Val = MC.getImm() + 4;
    if (PrintBranchImmAsAddress) {
      uint64_t Target = Address;
      Target &= ~0x3;
      Target += Val & (~0x3);
      O << formatHex(Target);
    } else {
      O << ". ";
      if (Val > 0)
        O << '+';
      O << Val;
    }
  } else if (MC.isExpr())
    MAI.printExpr(O, *MC.getExpr());
```
- **EN**: Implements logic around `printCallOperand`, `getOperand`, `getImm`, `formatHex`, ...; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `printCallOperand`, `getOperand`, `getImm`, `formatHex`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 139-156
```cpp
  else
    llvm_unreachable("Invalid operand");
}

void XtensaInstPrinter::printL32RTarget(const MCInst *MI, uint64_t Address,
                                        int OpNum, raw_ostream &O) {
  const MCOperand &MC = MI->getOperand(OpNum);
  if (MC.isImm()) {
    int64_t Value = MI->getOperand(OpNum).getImm();
    if (PrintBranchImmAsAddress) {
      uint64_t Target = (Address + 0x3) & (~0x3);
      Value &= ~0x3;
      Target += Value;
      O << formatHex(Target);
    } else {
      int64_t InstrOff = Value & 0x3;
      Value -= InstrOff;
      assert((Value >= -262144 && Value <= -4) &&
```
- **EN**: Implements logic around `llvm_unreachable`, `printL32RTarget`, `getOperand`, `formatHex`, ...; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `llvm_unreachable`, `printL32RTarget`, `getOperand`, `formatHex`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 157-166
```cpp
             "Invalid argument, value must be in ranges [-262144,-4]");
      Value += ((InstrOff + 0x3) & 0x4) - InstrOff;
      printPCRelImm(Address, Value, O);
    }
  } else if (MC.isExpr())
    MAI.printExpr(O, *MC.getExpr());
  else
    llvm_unreachable("Invalid operand");
}

```
- **EN**: Implements logic around `printPCRelImm`, `printExpr`, `llvm_unreachable`; this block applies conditional target rules.
- **CN**: 围绕 `printPCRelImm`, `printExpr`, `llvm_unreachable` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 167-180
```cpp
void XtensaInstPrinter::printPCRelImm(uint64_t Address, int64_t Offset,
                                      raw_ostream &O) {
  if (PrintBranchImmAsAddress) {
    uint64_t Target = Address + Offset;
    Target &= 0xffffffff;
    O << formatHex(Target);
  } else {
    O << ". ";
    if (Offset > 0)
      O << '+';
    O << Offset;
  }
}

```
- **EN**: Implements logic around `printPCRelImm`, `formatHex`; this block applies conditional target rules.
- **CN**: 围绕 `printPCRelImm`, `formatHex` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 181-192
```cpp
void XtensaInstPrinter::printImm8_AsmOperand(const MCInst *MI, int OpNum,
                                             raw_ostream &O) {
  if (MI->getOperand(OpNum).isImm()) {
    int64_t Value = MI->getOperand(OpNum).getImm();
    assert(isInt<8>(Value) &&
           "Invalid argument, value must be in ranges [-128,127]");
    O << Value;
  } else {
    printOperand(MI, OpNum, O);
  }
}

```
- **EN**: Implements logic around `printImm8_AsmOperand`, `getOperand`, `assert`, `printOperand`; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `printImm8_AsmOperand`, `getOperand`, `assert`, `printOperand` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 193-204
```cpp
void XtensaInstPrinter::printImm8_sh8_AsmOperand(const MCInst *MI, int OpNum,
                                                 raw_ostream &O) {
  if (MI->getOperand(OpNum).isImm()) {
    int64_t Value = MI->getOperand(OpNum).getImm();
    assert((isInt<16>(Value) && ((Value & 0xFF) == 0)) &&
           "Invalid argument, value must be multiples of 256 in range "
           "[-32768,32512]");
    O << Value;
  } else
    printOperand(MI, OpNum, O);
}

```
- **EN**: Implements logic around `printImm8_sh8_AsmOperand`, `getOperand`, `assert`, `printOperand`; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `printImm8_sh8_AsmOperand`, `getOperand`, `assert`, `printOperand` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 205-215
```cpp
void XtensaInstPrinter::printImm12_AsmOperand(const MCInst *MI, int OpNum,
                                              raw_ostream &O) {
  if (MI->getOperand(OpNum).isImm()) {
    int64_t Value = MI->getOperand(OpNum).getImm();
    assert((Value >= -2048 && Value <= 2047) &&
           "Invalid argument, value must be in ranges [-2048,2047]");
    O << Value;
  } else
    printOperand(MI, OpNum, O);
}

```
- **EN**: Implements logic around `printImm12_AsmOperand`, `getOperand`, `assert`, `printOperand`; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `printImm12_AsmOperand`, `getOperand`, `assert`, `printOperand` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 216-226
```cpp
void XtensaInstPrinter::printImm12m_AsmOperand(const MCInst *MI, int OpNum,
                                               raw_ostream &O) {
  if (MI->getOperand(OpNum).isImm()) {
    int64_t Value = MI->getOperand(OpNum).getImm();
    assert((Value >= -2048 && Value <= 2047) &&
           "Invalid argument, value must be in ranges [-2048,2047]");
    O << Value;
  } else
    printOperand(MI, OpNum, O);
}

```
- **EN**: Implements logic around `printImm12m_AsmOperand`, `getOperand`, `assert`, `printOperand`; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `printImm12m_AsmOperand`, `getOperand`, `assert`, `printOperand` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 227-236
```cpp
void XtensaInstPrinter::printUimm4_AsmOperand(const MCInst *MI, int OpNum,
                                              raw_ostream &O) {
  if (MI->getOperand(OpNum).isImm()) {
    int64_t Value = MI->getOperand(OpNum).getImm();
    assert((Value >= 0 && Value <= 15) && "Invalid argument");
    O << Value;
  } else
    printOperand(MI, OpNum, O);
}

```
- **EN**: Implements logic around `printUimm4_AsmOperand`, `getOperand`, `assert`, `printOperand`; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `printUimm4_AsmOperand`, `getOperand`, `assert`, `printOperand` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 237-246
```cpp
void XtensaInstPrinter::printUimm5_AsmOperand(const MCInst *MI, int OpNum,
                                              raw_ostream &O) {
  if (MI->getOperand(OpNum).isImm()) {
    int64_t Value = MI->getOperand(OpNum).getImm();
    assert((Value >= 0 && Value <= 31) && "Invalid argument");
    O << Value;
  } else
    printOperand(MI, OpNum, O);
}

```
- **EN**: Implements logic around `printUimm5_AsmOperand`, `getOperand`, `assert`, `printOperand`; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `printUimm5_AsmOperand`, `getOperand`, `assert`, `printOperand` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 247-257
```cpp
void XtensaInstPrinter::printShimm1_31_AsmOperand(const MCInst *MI, int OpNum,
                                                  raw_ostream &O) {
  if (MI->getOperand(OpNum).isImm()) {
    int64_t Value = MI->getOperand(OpNum).getImm();
    assert((Value >= 1 && Value <= 31) &&
           "Invalid argument, value must be in range [1,31]");
    O << Value;
  } else
    printOperand(MI, OpNum, O);
}

```
- **EN**: Implements logic around `printShimm1_31_AsmOperand`, `getOperand`, `assert`, `printOperand`; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `printShimm1_31_AsmOperand`, `getOperand`, `assert`, `printOperand` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 258-268
```cpp
void XtensaInstPrinter::printImm1_16_AsmOperand(const MCInst *MI, int OpNum,
                                                raw_ostream &O) {
  if (MI->getOperand(OpNum).isImm()) {
    int64_t Value = MI->getOperand(OpNum).getImm();
    assert((Value >= 1 && Value <= 16) &&
           "Invalid argument, value must be in range [1,16]");
    O << Value;
  } else
    printOperand(MI, OpNum, O);
}

```
- **EN**: Implements logic around `printImm1_16_AsmOperand`, `getOperand`, `assert`, `printOperand`; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `printImm1_16_AsmOperand`, `getOperand`, `assert`, `printOperand` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 269-279
```cpp
void XtensaInstPrinter::printImm1n_15_AsmOperand(const MCInst *MI, int OpNum,
                                                 raw_ostream &O) {
  if (MI->getOperand(OpNum).isImm()) {
    int64_t Value = MI->getOperand(OpNum).getImm();
    assert((Value >= -1 && (Value != 0) && Value <= 15) &&
           "Invalid argument, value must be in ranges <-1,-1> or <1,15>");
    O << Value;
  } else
    printOperand(MI, OpNum, O);
}

```
- **EN**: Implements logic around `printImm1n_15_AsmOperand`, `getOperand`, `assert`, `printOperand`; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `printImm1n_15_AsmOperand`, `getOperand`, `assert`, `printOperand` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 280-290
```cpp
void XtensaInstPrinter::printImm32n_95_AsmOperand(const MCInst *MI, int OpNum,
                                                  raw_ostream &O) {
  if (MI->getOperand(OpNum).isImm()) {
    int64_t Value = MI->getOperand(OpNum).getImm();
    assert((Value >= -32 && Value <= 95) &&
           "Invalid argument, value must be in ranges <-32,95>");
    O << Value;
  } else
    printOperand(MI, OpNum, O);
}

```
- **EN**: Implements logic around `printImm32n_95_AsmOperand`, `getOperand`, `assert`, `printOperand`; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `printImm32n_95_AsmOperand`, `getOperand`, `assert`, `printOperand` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 291-301
```cpp
void XtensaInstPrinter::printImm8n_7_AsmOperand(const MCInst *MI, int OpNum,
                                                raw_ostream &O) {
  if (MI->getOperand(OpNum).isImm()) {
    int64_t Value = MI->getOperand(OpNum).getImm();
    assert((Value >= -8 && Value <= 7) &&
           "Invalid argument, value must be in ranges <-8,7>");
    O << Value;
  } else
    printOperand(MI, OpNum, O);
}

```
- **EN**: Implements logic around `printImm8n_7_AsmOperand`, `getOperand`, `assert`, `printOperand`; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `printImm8n_7_AsmOperand`, `getOperand`, `assert`, `printOperand` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 302-312
```cpp
void XtensaInstPrinter::printImm64n_4n_AsmOperand(const MCInst *MI, int OpNum,
                                                  raw_ostream &O) {
  if (MI->getOperand(OpNum).isImm()) {
    int64_t Value = MI->getOperand(OpNum).getImm();
    assert((Value >= -64 && Value <= -4) & ((Value & 0x3) == 0) &&
           "Invalid argument, value must be in ranges <-64,-4>");
    O << Value;
  } else
    printOperand(MI, OpNum, O);
}

```
- **EN**: Implements logic around `printImm64n_4n_AsmOperand`, `getOperand`, `assert`, `printOperand`; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `printImm64n_4n_AsmOperand`, `getOperand`, `assert`, `printOperand` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 313-323
```cpp
void XtensaInstPrinter::printOffset8m8_AsmOperand(const MCInst *MI, int OpNum,
                                                  raw_ostream &O) {
  if (MI->getOperand(OpNum).isImm()) {
    int64_t Value = MI->getOperand(OpNum).getImm();
    assert((Value >= 0 && Value <= 255) &&
           "Invalid argument, value must be in range [0,255]");
    O << Value;
  } else
    printOperand(MI, OpNum, O);
}

```
- **EN**: Implements logic around `printOffset8m8_AsmOperand`, `getOperand`, `assert`, `printOperand`; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `printOffset8m8_AsmOperand`, `getOperand`, `assert`, `printOperand` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 324-334
```cpp
void XtensaInstPrinter::printOffset8m16_AsmOperand(const MCInst *MI, int OpNum,
                                                   raw_ostream &O) {
  if (MI->getOperand(OpNum).isImm()) {
    int64_t Value = MI->getOperand(OpNum).getImm();
    assert((Value >= 0 && Value <= 510 && ((Value & 0x1) == 0)) &&
           "Invalid argument, value must be multiples of two in range [0,510]");
    O << Value;
  } else
    printOperand(MI, OpNum, O);
}

```
- **EN**: Implements logic around `printOffset8m16_AsmOperand`, `getOperand`, `assert`, `printOperand`; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `printOffset8m16_AsmOperand`, `getOperand`, `assert`, `printOperand` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 335-346
```cpp
void XtensaInstPrinter::printOffset8m32_AsmOperand(const MCInst *MI, int OpNum,
                                                   raw_ostream &O) {
  if (MI->getOperand(OpNum).isImm()) {
    int64_t Value = MI->getOperand(OpNum).getImm();
    assert(
        (Value >= 0 && Value <= 1020 && ((Value & 0x3) == 0)) &&
        "Invalid argument, value must be multiples of four in range [0,1020]");
    O << Value;
  } else
    printOperand(MI, OpNum, O);
}

```
- **EN**: Implements logic around `printOffset8m32_AsmOperand`, `getOperand`, `assert`, `printOperand`; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `printOffset8m32_AsmOperand`, `getOperand`, `assert`, `printOperand` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 347-357
```cpp
void XtensaInstPrinter::printOffset4m32_AsmOperand(const MCInst *MI, int OpNum,
                                                   raw_ostream &O) {
  if (MI->getOperand(OpNum).isImm()) {
    int64_t Value = MI->getOperand(OpNum).getImm();
    assert((Value >= 0 && Value <= 60 && ((Value & 0x3) == 0)) &&
           "Invalid argument, value must be multiples of four in range [0,60]");
    O << Value;
  } else
    printOperand(MI, OpNum, O);
}

```
- **EN**: Implements logic around `printOffset4m32_AsmOperand`, `getOperand`, `assert`, `printOperand`; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `printOffset4m32_AsmOperand`, `getOperand`, `assert`, `printOperand` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 358-369
```cpp
void XtensaInstPrinter::printEntry_Imm12_AsmOperand(const MCInst *MI, int OpNum,
                                                    raw_ostream &O) {
  if (MI->getOperand(OpNum).isImm()) {
    int64_t Value = MI->getOperand(OpNum).getImm();
    assert((Value >= 0 && Value <= 32760) &&
           "Invalid argument, value must be multiples of eight in range "
           "<0,32760>");
    O << Value;
  } else
    printOperand(MI, OpNum, O);
}

```
- **EN**: Implements logic around `printEntry_Imm12_AsmOperand`, `getOperand`, `assert`, `printOperand`; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `printEntry_Imm12_AsmOperand`, `getOperand`, `assert`, `printOperand` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 370-387
```cpp
void XtensaInstPrinter::printB4const_AsmOperand(const MCInst *MI, int OpNum,
                                                raw_ostream &O) {
  if (MI->getOperand(OpNum).isImm()) {
    int64_t Value = MI->getOperand(OpNum).getImm();

    switch (Value) {
    case -1:
    case 1:
    case 2:
    case 3:
    case 4:
    case 5:
    case 6:
    case 7:
    case 8:
    case 10:
    case 12:
    case 16:
```
- **EN**: Implements logic around `printB4const_AsmOperand`, `getOperand`; this block uses `switch`-based dispatch; applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `printB4const_AsmOperand`, `getOperand` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，工作在 MC 层。

### Lines 388-400
```cpp
    case 32:
    case 64:
    case 128:
    case 256:
      break;
    default:
      assert((0) && "Invalid B4const argument");
    }
    O << Value;
  } else
    printOperand(MI, OpNum, O);
}

```
- **EN**: Implements logic around `assert`, `printOperand`.
- **CN**: 围绕 `assert`, `printOperand` 实现具体逻辑。

### Lines 401-418
```cpp
void XtensaInstPrinter::printB4constu_AsmOperand(const MCInst *MI, int OpNum,
                                                 raw_ostream &O) {
  if (MI->getOperand(OpNum).isImm()) {
    int64_t Value = MI->getOperand(OpNum).getImm();

    switch (Value) {
    case 32768:
    case 65536:
    case 2:
    case 3:
    case 4:
    case 5:
    case 6:
    case 7:
    case 8:
    case 10:
    case 12:
    case 16:
```
- **EN**: Implements logic around `printB4constu_AsmOperand`, `getOperand`; this block uses `switch`-based dispatch; applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `printB4constu_AsmOperand`, `getOperand` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，工作在 MC 层。

### Lines 419-431
```cpp
    case 32:
    case 64:
    case 128:
    case 256:
      break;
    default:
      assert((0) && "Invalid B4constu argument");
    }
    O << Value;
  } else
    printOperand(MI, OpNum, O);
}

```
- **EN**: Implements logic around `assert`, `printOperand`.
- **CN**: 围绕 `assert`, `printOperand` 实现具体逻辑。

### Lines 432-441
```cpp
void XtensaInstPrinter::printImm7_22_AsmOperand(const MCInst *MI, int OpNum,
                                                raw_ostream &O) {
  if (MI->getOperand(OpNum).isImm()) {
    int64_t Value = MI->getOperand(OpNum).getImm();
    assert((Value >= 7 && Value <= 22) &&
           "Invalid argument, value must be in range <7,22>");
    O << Value;
  } else
    printOperand(MI, OpNum, O);
}
```
- **EN**: Implements logic around `printImm7_22_AsmOperand`, `getOperand`, `assert`, `printOperand`; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `printImm7_22_AsmOperand`, `getOperand`, `assert`, `printOperand` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

## Key Concepts / 关键概念

- **MC layer / MC 层**:
  - **EN**: Connects the backend to LLVM's MC infrastructure
  - **CN**: 把后端接入 LLVM 的 MC 基础设施

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `XtensaInstPrinter.h`, `MCTargetDesc/XtensaMCAsmInfo.h`, `llvm/CodeGen/MachineOperand.h`, `llvm/MC/MCAsmInfo.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCInstrInfo.h`, `llvm/MC/MCRegister.h`, `llvm/MC/MCSymbol.h`, `llvm/Support/Casting.h`, `llvm/Support/raw_ostream.h`, `XtensaGenAsmWriter.inc`
- **LLVM subsystems / LLVM 子系统**: MC, CodeGen, Support
