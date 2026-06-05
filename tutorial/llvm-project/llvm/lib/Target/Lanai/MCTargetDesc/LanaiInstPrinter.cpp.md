# LanaiInstPrinter.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Lanai/MCTargetDesc/LanaiInstPrinter.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Provides MC-layer target descriptions such as fixups, asm info, code emission, object writing, and target registration.
  - **CN**: 提供 MC 层的目标描述，例如 fixup、汇编信息、编码发射、目标文件写出以及目标注册。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===-- LanaiInstPrinter.cpp - Convert Lanai MCInst to asm syntax ---------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This class prints an Lanai MCInst to a .s file.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 12-22
```cpp

#include "LanaiInstPrinter.h"
#include "LanaiAluCode.h"
#include "LanaiCondCode.h"
#include "MCTargetDesc/LanaiMCTargetDesc.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCSymbol.h"
#include "llvm/Support/ErrorHandling.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `LanaiInstPrinter.h`, `LanaiAluCode.h`, `LanaiCondCode.h`, `MCTargetDesc/LanaiMCTargetDesc.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `LanaiInstPrinter.h`, `LanaiAluCode.h`, `LanaiCondCode.h`, `MCTargetDesc/LanaiMCTargetDesc.h`。

### Lines 23-34
```cpp
using namespace llvm;

#define DEBUG_TYPE "asm-printer"

// Include the auto-generated portion of the assembly writer.
#define PRINT_ALIAS_INSTR
#include "LanaiGenAsmWriter.inc"

void LanaiInstPrinter::printRegName(raw_ostream &OS, MCRegister Reg) {
  OS << StringRef(getRegisterName(Reg)).lower();
}

```
- **EN**: Pulls in the headers needed for this implementation, including `LanaiGenAsmWriter.inc`.
- **CN**: 引入该实现所需的头文件，其中包括 `LanaiGenAsmWriter.inc`。

### Lines 35-44
```cpp
bool LanaiInstPrinter::printInst(const MCInst *MI, raw_ostream &OS,
                                 StringRef Alias, unsigned OpNo0,
                                 unsigned OpNo1) {
  OS << "\t" << Alias << " ";
  printOperand(MI, OpNo0, OS);
  OS << ", ";
  printOperand(MI, OpNo1, OS);
  return true;
}

```
- **EN**: Implements logic around `printInst`, `printOperand`; this block returns target-specific results; works at the MC layer.
- **CN**: 围绕 `printInst`, `printOperand` 实现具体逻辑；这一段返回目标相关结果，工作在 MC 层。

### Lines 45-56
```cpp
static bool usesGivenOffset(const MCInst *MI, int AddOffset) {
  unsigned AluCode = MI->getOperand(3).getImm();
  return LPAC::encodeLanaiAluCode(AluCode) == LPAC::ADD &&
         (MI->getOperand(2).getImm() == AddOffset ||
          MI->getOperand(2).getImm() == -AddOffset);
}

static bool isPreIncrementForm(const MCInst *MI, int AddOffset) {
  unsigned AluCode = MI->getOperand(3).getImm();
  return LPAC::isPreOp(AluCode) && usesGivenOffset(MI, AddOffset);
}

```
- **EN**: Implements logic around `usesGivenOffset`, `getOperand`, `encodeLanaiAluCode`, `isPreIncrementForm`, ...; this block returns target-specific results; works at the MC layer.
- **CN**: 围绕 `usesGivenOffset`, `getOperand`, `encodeLanaiAluCode`, `isPreIncrementForm`, ... 实现具体逻辑；这一段返回目标相关结果，工作在 MC 层。

### Lines 57-67
```cpp
static bool isPostIncrementForm(const MCInst *MI, int AddOffset) {
  unsigned AluCode = MI->getOperand(3).getImm();
  return LPAC::isPostOp(AluCode) && usesGivenOffset(MI, AddOffset);
}

static StringRef decIncOperator(const MCInst *MI) {
  if (MI->getOperand(2).getImm() < 0)
    return "--";
  return "++";
}

```
- **EN**: Implements logic around `isPostIncrementForm`, `getOperand`, `isPostOp`, `decIncOperator`; this block applies conditional target rules; returns target-specific results; works at the MC layer.
- **CN**: 围绕 `isPostIncrementForm`, `getOperand`, `isPostOp`, `decIncOperator` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MC 层。

### Lines 68-85
```cpp
bool LanaiInstPrinter::printMemoryLoadIncrement(const MCInst *MI,
                                                raw_ostream &OS,
                                                StringRef Opcode,
                                                int AddOffset) {
  if (isPreIncrementForm(MI, AddOffset)) {
    OS << "\t" << Opcode << "\t[" << decIncOperator(MI) << "%"
       << getRegisterName(MI->getOperand(1).getReg()) << "], %"
       << getRegisterName(MI->getOperand(0).getReg());
    return true;
  }
  if (isPostIncrementForm(MI, AddOffset)) {
    OS << "\t" << Opcode << "\t[%"
       << getRegisterName(MI->getOperand(1).getReg()) << decIncOperator(MI)
       << "], %" << getRegisterName(MI->getOperand(0).getReg());
    return true;
  }
  return false;
}
```
- **EN**: Implements logic around `printMemoryLoadIncrement`, `decIncOperator`, `getRegisterName`; this block applies conditional target rules; returns target-specific results; works at the MC layer.
- **CN**: 围绕 `printMemoryLoadIncrement`, `decIncOperator`, `getRegisterName` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MC 层。

### Lines 86-103
```cpp

bool LanaiInstPrinter::printMemoryStoreIncrement(const MCInst *MI,
                                                 raw_ostream &OS,
                                                 StringRef Opcode,
                                                 int AddOffset) {
  if (isPreIncrementForm(MI, AddOffset)) {
    OS << "\t" << Opcode << "\t%" << getRegisterName(MI->getOperand(0).getReg())
       << ", [" << decIncOperator(MI) << "%"
       << getRegisterName(MI->getOperand(1).getReg()) << "]";
    return true;
  }
  if (isPostIncrementForm(MI, AddOffset)) {
    OS << "\t" << Opcode << "\t%" << getRegisterName(MI->getOperand(0).getReg())
       << ", [%" << getRegisterName(MI->getOperand(1).getReg())
       << decIncOperator(MI) << "]";
    return true;
  }
  return false;
```
- **EN**: Implements logic around `printMemoryStoreIncrement`, `getRegisterName`, `decIncOperator`; this block applies conditional target rules; returns target-specific results; works at the MC layer.
- **CN**: 围绕 `printMemoryStoreIncrement`, `getRegisterName`, `decIncOperator` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MC 层。

### Lines 104-121
```cpp
}

bool LanaiInstPrinter::printAlias(const MCInst *MI, raw_ostream &OS) {
  switch (MI->getOpcode()) {
  case Lanai::LDW_RI:
    // ld 4[*%rN], %rX => ld [++imm], %rX
    // ld -4[*%rN], %rX => ld [--imm], %rX
    // ld 4[%rN*], %rX => ld [imm++], %rX
    // ld -4[%rN*], %rX => ld [imm--], %rX
    return printMemoryLoadIncrement(MI, OS, "ld", 4);
  case Lanai::LDHs_RI:
    return printMemoryLoadIncrement(MI, OS, "ld.h", 2);
  case Lanai::LDHz_RI:
    return printMemoryLoadIncrement(MI, OS, "uld.h", 2);
  case Lanai::LDBs_RI:
    return printMemoryLoadIncrement(MI, OS, "ld.b", 1);
  case Lanai::LDBz_RI:
    return printMemoryLoadIncrement(MI, OS, "uld.b", 1);
```
- **EN**: Implements logic around `printAlias`, `printMemoryLoadIncrement`; this block uses `switch`-based dispatch; returns target-specific results; works at the MC layer.
- **CN**: 围绕 `printAlias`, `printMemoryLoadIncrement` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果，工作在 MC 层。

### Lines 122-136
```cpp
  case Lanai::SW_RI:
    // st %rX, 4[*%rN] => st %rX, [++imm]
    // st %rX, -4[*%rN] => st %rX, [--imm]
    // st %rX, 4[%rN*] => st %rX, [imm++]
    // st %rX, -4[%rN*] => st %rX, [imm--]
    return printMemoryStoreIncrement(MI, OS, "st", 4);
  case Lanai::STH_RI:
    return printMemoryStoreIncrement(MI, OS, "st.h", 2);
  case Lanai::STB_RI:
    return printMemoryStoreIncrement(MI, OS, "st.b", 1);
  default:
    return false;
  }
}

```
- **EN**: Implements logic around `printMemoryStoreIncrement`; this block returns target-specific results.
- **CN**: 围绕 `printMemoryStoreIncrement` 实现具体逻辑；这一段返回目标相关结果。

### Lines 137-145
```cpp
void LanaiInstPrinter::printInst(const MCInst *MI, uint64_t Address,
                                 StringRef Annotation,
                                 const MCSubtargetInfo & /*STI*/,
                                 raw_ostream &OS) {
  if (!printAlias(MI, OS) && !printAliasInstr(MI, Address, OS))
    printInstruction(MI, Address, OS);
  printAnnotation(OS, Annotation);
}

```
- **EN**: Implements logic around `printInst`, `printInstruction`, `printAnnotation`; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `printInst`, `printInstruction`, `printAnnotation` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 146-158
```cpp
void LanaiInstPrinter::printOperand(const MCInst *MI, unsigned OpNo,
                                    raw_ostream &OS) {
  const MCOperand &Op = MI->getOperand(OpNo);
  if (Op.isReg())
    OS << "%" << getRegisterName(Op.getReg());
  else if (Op.isImm())
    OS << formatHex(Op.getImm());
  else {
    assert(Op.isExpr() && "Expected an expression");
    MAI.printExpr(OS, *Op.getExpr());
  }
}

```
- **EN**: Implements logic around `printOperand`, `getOperand`, `getRegisterName`, `formatHex`, ...; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `printOperand`, `getOperand`, `getRegisterName`, `formatHex`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 159-172
```cpp
void LanaiInstPrinter::printMemImmOperand(const MCInst *MI, unsigned OpNo,
                                          raw_ostream &OS) {
  const MCOperand &Op = MI->getOperand(OpNo);
  if (Op.isImm()) {
    OS << '[' << formatHex(Op.getImm()) << ']';
  } else {
    // Symbolic operand will be lowered to immediate value by linker
    assert(Op.isExpr() && "Expected an expression");
    OS << '[';
    MAI.printExpr(OS, *Op.getExpr());
    OS << ']';
  }
}

```
- **EN**: Implements logic around `printMemImmOperand`, `getOperand`, `formatHex`, `assert`, ...; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `printMemImmOperand`, `getOperand`, `formatHex`, `assert`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 173-184
```cpp
void LanaiInstPrinter::printHi16ImmOperand(const MCInst *MI, unsigned OpNo,
                                           raw_ostream &OS) {
  const MCOperand &Op = MI->getOperand(OpNo);
  if (Op.isImm()) {
    OS << formatHex(Op.getImm() << 16);
  } else {
    // Symbolic operand will be lowered to immediate value by linker
    assert(Op.isExpr() && "Expected an expression");
    MAI.printExpr(OS, *Op.getExpr());
  }
}

```
- **EN**: Implements logic around `printHi16ImmOperand`, `getOperand`, `formatHex`, `assert`, ...; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `printHi16ImmOperand`, `getOperand`, `formatHex`, `assert`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 185-196
```cpp
void LanaiInstPrinter::printHi16AndImmOperand(const MCInst *MI, unsigned OpNo,
                                              raw_ostream &OS) {
  const MCOperand &Op = MI->getOperand(OpNo);
  if (Op.isImm()) {
    OS << formatHex((Op.getImm() << 16) | 0xffff);
  } else {
    // Symbolic operand will be lowered to immediate value by linker
    assert(Op.isExpr() && "Expected an expression");
    MAI.printExpr(OS, *Op.getExpr());
  }
}

```
- **EN**: Implements logic around `printHi16AndImmOperand`, `getOperand`, `formatHex`, `assert`, ...; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `printHi16AndImmOperand`, `getOperand`, `formatHex`, `assert`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 197-208
```cpp
void LanaiInstPrinter::printLo16AndImmOperand(const MCInst *MI, unsigned OpNo,
                                              raw_ostream &OS) {
  const MCOperand &Op = MI->getOperand(OpNo);
  if (Op.isImm()) {
    OS << formatHex(0xffff0000 | Op.getImm());
  } else {
    // Symbolic operand will be lowered to immediate value by linker
    assert(Op.isExpr() && "Expected an expression");
    MAI.printExpr(OS, *Op.getExpr());
  }
}

```
- **EN**: Implements logic around `printLo16AndImmOperand`, `getOperand`, `formatHex`, `assert`, ...; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `printLo16AndImmOperand`, `getOperand`, `formatHex`, `assert`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 209-220
```cpp
static void printMemoryBaseRegister(raw_ostream &OS, const unsigned AluCode,
                                    const MCOperand &RegOp) {
  assert(RegOp.isReg() && "Register operand expected");
  OS << "[";
  if (LPAC::isPreOp(AluCode))
    OS << "*";
  OS << "%" << LanaiInstPrinter::getRegisterName(RegOp.getReg());
  if (LPAC::isPostOp(AluCode))
    OS << "*";
  OS << "]";
}

```
- **EN**: Implements logic around `printMemoryBaseRegister`, `assert`, `getRegisterName`; this block applies conditional target rules.
- **CN**: 围绕 `printMemoryBaseRegister`, `assert`, `getRegisterName` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 221-232
```cpp
template <unsigned SizeInBits>
static void printMemoryImmediateOffset(const MCAsmInfo &MAI,
                                       const MCOperand &OffsetOp,
                                       raw_ostream &OS) {
  assert((OffsetOp.isImm() || OffsetOp.isExpr()) && "Immediate expected");
  if (OffsetOp.isImm()) {
    assert(isInt<SizeInBits>(OffsetOp.getImm()) && "Constant value truncated");
    OS << OffsetOp.getImm();
  } else
    MAI.printExpr(OS, *OffsetOp.getExpr());
}

```
- **EN**: Implements logic around `printMemoryImmediateOffset`, `assert`, `getImm`, `printExpr`; this block applies conditional target rules.
- **CN**: 围绕 `printMemoryImmediateOffset`, `assert`, `getImm`, `printExpr` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 233-242
```cpp
void LanaiInstPrinter::printMemRiOperand(const MCInst *MI, int OpNo,
                                         raw_ostream &OS) {
  const MCOperand &RegOp = MI->getOperand(OpNo);
  const MCOperand &OffsetOp = MI->getOperand(OpNo + 1);
  const MCOperand &AluOp = MI->getOperand(OpNo + 2);
  const unsigned AluCode = AluOp.getImm();

  // Offset
  printMemoryImmediateOffset<16>(MAI, OffsetOp, OS);

```
- **EN**: Implements logic around `printMemRiOperand`, `getOperand`, `getImm`, `printMemoryImmediateOffset<16>`; this block works at the MC layer.
- **CN**: 围绕 `printMemRiOperand`, `getOperand`, `getImm`, `printMemoryImmediateOffset<16>` 实现具体逻辑；这一段工作在 MC 层。

### Lines 243-254
```cpp
  // Register
  printMemoryBaseRegister(OS, AluCode, RegOp);
}

void LanaiInstPrinter::printMemRrOperand(const MCInst *MI, int OpNo,
                                         raw_ostream &OS) {
  const MCOperand &RegOp = MI->getOperand(OpNo);
  const MCOperand &OffsetOp = MI->getOperand(OpNo + 1);
  const MCOperand &AluOp = MI->getOperand(OpNo + 2);
  const unsigned AluCode = AluOp.getImm();
  assert(OffsetOp.isReg() && RegOp.isReg() && "Registers expected.");

```
- **EN**: Implements logic around `printMemoryBaseRegister`, `printMemRrOperand`, `getOperand`, `getImm`, ...; this block works at the MC layer.
- **CN**: 围绕 `printMemoryBaseRegister`, `printMemRrOperand`, `getOperand`, `getImm`, ... 实现具体逻辑；这一段工作在 MC 层。

### Lines 255-266
```cpp
  // [ Base OP Offset ]
  OS << "[";
  if (LPAC::isPreOp(AluCode))
    OS << "*";
  OS << "%" << getRegisterName(RegOp.getReg());
  if (LPAC::isPostOp(AluCode))
    OS << "*";
  OS << " " << LPAC::lanaiAluCodeToString(AluCode) << " ";
  OS << "%" << getRegisterName(OffsetOp.getReg());
  OS << "]";
}

```
- **EN**: Implements logic around `getRegisterName`, `lanaiAluCodeToString`; this block applies conditional target rules.
- **CN**: 围绕 `getRegisterName`, `lanaiAluCodeToString` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 267-276
```cpp
void LanaiInstPrinter::printMemSplsOperand(const MCInst *MI, int OpNo,
                                           raw_ostream &OS) {
  const MCOperand &RegOp = MI->getOperand(OpNo);
  const MCOperand &OffsetOp = MI->getOperand(OpNo + 1);
  const MCOperand &AluOp = MI->getOperand(OpNo + 2);
  const unsigned AluCode = AluOp.getImm();

  // Offset
  printMemoryImmediateOffset<10>(MAI, OffsetOp, OS);

```
- **EN**: Implements logic around `printMemSplsOperand`, `getOperand`, `getImm`, `printMemoryImmediateOffset<10>`; this block works at the MC layer.
- **CN**: 围绕 `printMemSplsOperand`, `getOperand`, `getImm`, `printMemoryImmediateOffset<10>` 实现具体逻辑；这一段工作在 MC 层。

### Lines 277-291
```cpp
  // Register
  printMemoryBaseRegister(OS, AluCode, RegOp);
}

void LanaiInstPrinter::printCCOperand(const MCInst *MI, int OpNo,
                                      raw_ostream &OS) {
  LPCC::CondCode CC =
      static_cast<LPCC::CondCode>(MI->getOperand(OpNo).getImm());
  // Handle the undefined value here for printing so we don't abort().
  if (CC >= LPCC::UNKNOWN)
    OS << "<und>";
  else
    OS << lanaiCondCodeToString(CC);
}

```
- **EN**: Implements logic around `printMemoryBaseRegister`, `printCCOperand`, `CondCode>`, `lanaiCondCodeToString`; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `printMemoryBaseRegister`, `printCCOperand`, `CondCode>`, `lanaiCondCodeToString` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 292-301
```cpp
void LanaiInstPrinter::printPredicateOperand(const MCInst *MI, unsigned OpNo,
                                             raw_ostream &OS) {
  LPCC::CondCode CC =
      static_cast<LPCC::CondCode>(MI->getOperand(OpNo).getImm());
  // Handle the undefined value here for printing so we don't abort().
  if (CC >= LPCC::UNKNOWN)
    OS << "<und>";
  else if (CC != LPCC::ICC_T)
    OS << "." << lanaiCondCodeToString(CC);
}
```
- **EN**: Implements logic around `printPredicateOperand`, `CondCode>`, `lanaiCondCodeToString`; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `printPredicateOperand`, `CondCode>`, `lanaiCondCodeToString` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

## Key Concepts / 关键概念

- **MC layer / MC 层**:
  - **EN**: Connects the backend to LLVM's MC infrastructure
  - **CN**: 把后端接入 LLVM 的 MC 基础设施

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `LanaiInstPrinter.h`, `LanaiAluCode.h`, `LanaiCondCode.h`, `MCTargetDesc/LanaiMCTargetDesc.h`, `llvm/MC/MCAsmInfo.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCInst.h`, `llvm/MC/MCSymbol.h`, `llvm/Support/ErrorHandling.h`, `LanaiGenAsmWriter.inc`
- **LLVM subsystems / LLVM 子系统**: MC, Support
