# R600InstPrinter.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/MCTargetDesc/R600InstPrinter.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements R600InstPrinter for the LLVM MC target description layer. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM MC 目标描述层中的 R600InstPrinter 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-32: File banner, includes, and setup
```cpp
//===-- R600InstPrinter.cpp - AMDGPU MC Inst -> ASM ---------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
// \file
//===----------------------------------------------------------------------===//

#include "R600InstPrinter.h"
#include "AMDGPUInstPrinter.h"
#include "R600MCTargetDesc.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCInstrInfo.h"
#include "llvm/MC/MCSubtargetInfo.h"

using namespace llvm;

void R600InstPrinter::printInst(const MCInst *MI, uint64_t Address,
                                StringRef Annot, const MCSubtargetInfo &STI,
                                raw_ostream &O) {
  printInstruction(MI, Address, O);
  printAnnotation(O, Annot);
}

void R600InstPrinter::printAbs(const MCInst *MI, unsigned OpNo,
                               raw_ostream &O) {
  AMDGPUInstPrinter::printIfSet(MI, OpNo, O, '|');
}

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation. Main symbols: `R600InstPrinter::printInst`, `R600InstPrinter::printAbs`, `AMDGPUInstPrinter::printIfSet`.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。 主要符号：`R600InstPrinter::printInst`, `R600InstPrinter::printAbs`, `AMDGPUInstPrinter::printIfSet`。

### Lines 33-61: Implements R600InstPrinter::printBankSwizzle
```cpp
void R600InstPrinter::printBankSwizzle(const MCInst *MI, unsigned OpNo,
                                       raw_ostream &O) {
  int BankSwizzle = MI->getOperand(OpNo).getImm();
  switch (BankSwizzle) {
  case 1:
    O << "BS:VEC_021/SCL_122";
    break;
  case 2:
    O << "BS:VEC_120/SCL_212";
    break;
  case 3:
    O << "BS:VEC_102/SCL_221";
    break;
  case 4:
    O << "BS:VEC_201";
    break;
  case 5:
    O << "BS:VEC_210";
    break;
  default:
    break;
  }
}

void R600InstPrinter::printClamp(const MCInst *MI, unsigned OpNo,
                                 raw_ostream &O) {
  AMDGPUInstPrinter::printIfSet(MI, OpNo, O, "_SAT");
}

```
**EN:** This section contains concrete logic for R600InstPrinter::printBankSwizzle. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600InstPrinter::printBankSwizzle`, `R600InstPrinter::printClamp`, `AMDGPUInstPrinter::printIfSet`.
**CN:** 本节包含与 R600InstPrinter::printBankSwizzle 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600InstPrinter::printBankSwizzle`, `R600InstPrinter::printClamp`, `AMDGPUInstPrinter::printIfSet`。

### Lines 62-92: Implements R600InstPrinter::printCT
```cpp
void R600InstPrinter::printCT(const MCInst *MI, unsigned OpNo, raw_ostream &O) {
  unsigned CT = MI->getOperand(OpNo).getImm();
  switch (CT) {
  case 0:
    O << 'U';
    break;
  case 1:
    O << 'N';
    break;
  default:
    break;
  }
}

void R600InstPrinter::printKCache(const MCInst *MI, unsigned OpNo,
                                  raw_ostream &O) {
  int KCacheMode = MI->getOperand(OpNo).getImm();
  if (KCacheMode > 0) {
    int KCacheBank = MI->getOperand(OpNo - 2).getImm();
    O << "CB" << KCacheBank << ':';
    int KCacheAddr = MI->getOperand(OpNo + 2).getImm();
    int LineSize = (KCacheMode == 1) ? 16 : 32;
    O << KCacheAddr * 16 << '-' << KCacheAddr * 16 + LineSize;
  }
}

void R600InstPrinter::printLast(const MCInst *MI, unsigned OpNo,
                                raw_ostream &O) {
  AMDGPUInstPrinter::printIfSet(MI, OpNo, O, "*", " ");
}

```
**EN:** This section contains concrete logic for R600InstPrinter::printCT. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600InstPrinter::printCT`, `R600InstPrinter::printKCache`, `R600InstPrinter::printLast`.
**CN:** 本节包含与 R600InstPrinter::printCT 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600InstPrinter::printCT`, `R600InstPrinter::printKCache`, `R600InstPrinter::printLast`。

### Lines 93-111: Implements R600InstPrinter::printLiteral
```cpp
void R600InstPrinter::printLiteral(const MCInst *MI, unsigned OpNo,
                                   raw_ostream &O) {
  const MCOperand &Op = MI->getOperand(OpNo);
  assert(Op.isImm() || Op.isExpr());
  if (Op.isImm()) {
    int64_t Imm = Op.getImm();
    O << Imm << '(' << llvm::bit_cast<float>(static_cast<uint32_t>(Imm)) << ')';
  }
  if (Op.isExpr()) {
    O << '@';
    MAI.printExpr(O, *Op.getExpr());
  }
}

void R600InstPrinter::printNeg(const MCInst *MI, unsigned OpNo,
                               raw_ostream &O) {
  AMDGPUInstPrinter::printIfSet(MI, OpNo, O, '-');
}

```
**EN:** This section contains concrete logic for R600InstPrinter::printLiteral. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600InstPrinter::printLiteral`, `R600InstPrinter::printNeg`, `AMDGPUInstPrinter::printIfSet`.
**CN:** 本节包含与 R600InstPrinter::printLiteral 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600InstPrinter::printLiteral`, `R600InstPrinter::printNeg`, `AMDGPUInstPrinter::printIfSet`。

### Lines 112-142: Implements R600InstPrinter::printOMOD
```cpp
void R600InstPrinter::printOMOD(const MCInst *MI, unsigned OpNo,
                                raw_ostream &O) {
  switch (MI->getOperand(OpNo).getImm()) {
  default:
    break;
  case 1:
    O << " * 2.0";
    break;
  case 2:
    O << " * 4.0";
    break;
  case 3:
    O << " / 2.0";
    break;
  }
}

void R600InstPrinter::printMemOperand(const MCInst *MI, unsigned OpNo,
                                      raw_ostream &O) {
  printOperand(MI, OpNo, O);
  O << ", ";
  printOperand(MI, OpNo + 1, O);
}

void R600InstPrinter::printOperand(const MCInst *MI, unsigned OpNo,
                                   raw_ostream &O) {
  if (OpNo >= MI->getNumOperands()) {
    O << "/*Missing OP" << OpNo << "*/";
    return;
  }

```
**EN:** This section contains concrete logic for R600InstPrinter::printOMOD. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600InstPrinter::printOMOD`, `R600InstPrinter::printMemOperand`, `R600InstPrinter::printOperand`.
**CN:** 本节包含与 R600InstPrinter::printOMOD 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600InstPrinter::printOMOD`, `R600InstPrinter::printMemOperand`, `R600InstPrinter::printOperand`。

### Lines 143-175: Declares getOperand
```cpp
  const MCOperand &Op = MI->getOperand(OpNo);
  if (Op.isReg()) {
    switch (Op.getReg().id()) {
    // This is the default predicate state, so we don't need to print it.
    case R600::PRED_SEL_OFF:
      break;

    default:
      O << getRegisterName(Op.getReg());
      break;
    }
  } else if (Op.isImm()) {
    O << Op.getImm();
  } else if (Op.isDFPImm()) {
    // We special case 0.0 because otherwise it will be printed as an integer.
    if (Op.getDFPImm() == 0.0)
      O << "0.0";
    else {
      O << bit_cast<double>(Op.getDFPImm());
    }
  } else if (Op.isExpr()) {
    const MCExpr *Exp = Op.getExpr();
    MAI.printExpr(O, *Exp);
  } else {
    O << "/*INV_OP*/";
  }
}

void R600InstPrinter::printRel(const MCInst *MI, unsigned OpNo,
                               raw_ostream &O) {
  AMDGPUInstPrinter::printIfSet(MI, OpNo, O, '+');
}

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `R600InstPrinter::printRel`, `AMDGPUInstPrinter::printIfSet`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`R600InstPrinter::printRel`, `AMDGPUInstPrinter::printIfSet`。

### Lines 176-205: Implements R600InstPrinter::printRSel
```cpp
void R600InstPrinter::printRSel(const MCInst *MI, unsigned OpNo,
                                raw_ostream &O) {
  unsigned Sel = MI->getOperand(OpNo).getImm();
  switch (Sel) {
  case 0:
    O << 'X';
    break;
  case 1:
    O << 'Y';
    break;
  case 2:
    O << 'Z';
    break;
  case 3:
    O << 'W';
    break;
  case 4:
    O << '0';
    break;
  case 5:
    O << '1';
    break;
  case 7:
    O << '_';
    break;
  default:
    break;
  }
}

```
**EN:** This section contains concrete logic for R600InstPrinter::printRSel. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600InstPrinter::printRSel`.
**CN:** 本节包含与 R600InstPrinter::printRSel 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600InstPrinter::printRSel`。

### Lines 206-224: Header dependencies and setup
```cpp
void R600InstPrinter::printUpdateExecMask(const MCInst *MI, unsigned OpNo,
                                          raw_ostream &O) {
  AMDGPUInstPrinter::printIfSet(MI, OpNo, O, "ExecMask,");
}

void R600InstPrinter::printUpdatePred(const MCInst *MI, unsigned OpNo,
                                      raw_ostream &O) {
  AMDGPUInstPrinter::printIfSet(MI, OpNo, O, "Pred,");
}

void R600InstPrinter::printWrite(const MCInst *MI, unsigned OpNo,
                                 raw_ostream &O) {
  const MCOperand &Op = MI->getOperand(OpNo);
  if (Op.getImm() == 0) {
    O << " (MASKED)";
  }
}

#include "R600GenAsmWriter.inc"
```
**EN:** These includes pull in the LLVM and AMDGPU declarations required by the rest of the file, making later symbols and helper APIs available. Main symbols: `R600InstPrinter::printUpdateExecMask`, `AMDGPUInstPrinter::printIfSet`, `R600InstPrinter::printUpdatePred`.
**CN:** 这些头文件引入本文件后续实现所需的 LLVM 与 AMDGPU 声明，使后面的符号和辅助 API 可以直接使用。 主要符号：`R600InstPrinter::printUpdateExecMask`, `AMDGPUInstPrinter::printIfSet`, `R600InstPrinter::printUpdatePred`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `R600InstPrinter::printInst`, `R600InstPrinter::printAbs`, `AMDGPUInstPrinter::printIfSet`, `R600InstPrinter::printBankSwizzle`, `R600InstPrinter::printClamp`, `R600InstPrinter::printCT`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; assembly handling / 汇编处理
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"R600InstPrinter.h"`
- `"AMDGPUInstPrinter.h"`
- `"R600MCTargetDesc.h"`
- `"llvm/MC/MCAsmInfo.h"`
- `"llvm/MC/MCExpr.h"`
- `"llvm/MC/MCInst.h"`
- `"llvm/MC/MCInstrInfo.h"`
- `"llvm/MC/MCSubtargetInfo.h"`
- `"R600GenAsmWriter.inc"`
