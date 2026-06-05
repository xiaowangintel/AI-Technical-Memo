# MSP430AsmPrinter.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/MSP430/MSP430AsmPrinter.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Lowers MachineInstr to the MC layer and emits target-specific assembly/object output.
  - **CN**: 把 MachineInstr 降到 MC 层，并输出目标相关的汇编/目标文件内容。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- MSP430AsmPrinter.cpp - MSP430 LLVM assembly writer ----------------===//
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
// This file contains a printer that converts from our internal representation
// of machine-dependent LLVM code to the MSP430 assembly language.
//
//===----------------------------------------------------------------------===//

#include "MCTargetDesc/MSP430InstPrinter.h"
#include "MSP430MCInstLower.h"
#include "MSP430TargetMachine.h"
#include "TargetInfo/MSP430TargetInfo.h"
#include "llvm/BinaryFormat/ELF.h"
#include "llvm/CodeGen/AsmPrinter.h"
#include "llvm/CodeGen/MachineConstantPool.h"
#include "llvm/CodeGen/MachineInstr.h"
```
- **EN**: Pulls in the headers needed for this implementation, including `MCTargetDesc/MSP430InstPrinter.h`, `MSP430MCInstLower.h`, `MSP430TargetMachine.h`, `TargetInfo/MSP430TargetInfo.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `MCTargetDesc/MSP430InstPrinter.h`, `MSP430MCInstLower.h`, `MSP430TargetMachine.h`, `TargetInfo/MSP430TargetInfo.h`。

### Lines 22-33
```cpp
#include "llvm/CodeGen/MachineModuleInfo.h"
#include "llvm/IR/Mangler.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCSectionELF.h"
#include "llvm/MC/MCStreamer.h"
#include "llvm/MC/MCSymbol.h"
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Support/Compiler.h"
#include "llvm/Support/raw_ostream.h"
using namespace llvm;

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/CodeGen/MachineModuleInfo.h`, `llvm/IR/Mangler.h`, `llvm/MC/MCAsmInfo.h`, `llvm/MC/MCInst.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/CodeGen/MachineModuleInfo.h`, `llvm/IR/Mangler.h`, `llvm/MC/MCAsmInfo.h`, `llvm/MC/MCInst.h`。

### Lines 34-41
```cpp
#define DEBUG_TYPE "asm-printer"

namespace {
  class MSP430AsmPrinter : public AsmPrinter {
  public:
    MSP430AsmPrinter(TargetMachine &TM, std::unique_ptr<MCStreamer> Streamer)
        : AsmPrinter(TM, std::move(Streamer), ID) {}

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 42-55
```cpp
    StringRef getPassName() const override { return "MSP430 Assembly Printer"; }

    bool runOnMachineFunction(MachineFunction &MF) override;

    void PrintSymbolOperand(const MachineOperand &MO, raw_ostream &O) override;
    void printOperand(const MachineInstr *MI, int OpNum, raw_ostream &O,
                      bool PrefixHash = true);
    void printSrcMemOperand(const MachineInstr *MI, int OpNum,
                            raw_ostream &O);
    bool PrintAsmOperand(const MachineInstr *MI, unsigned OpNo,
                         const char *ExtraCode, raw_ostream &O) override;
    bool PrintAsmMemoryOperand(const MachineInstr *MI, unsigned OpNo,
                               const char *ExtraCode, raw_ostream &O) override;
    void emitInstruction(const MachineInstr *MI) override;
```
- **EN**: Implements logic around `getPassName`, `runOnMachineFunction`, `PrintSymbolOperand`, `printOperand`, ...; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getPassName`, `runOnMachineFunction`, `PrintSymbolOperand`, `printOperand`, ... 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 56-62
```cpp

    void EmitInterruptVectorSection(MachineFunction &ISR);

    static char ID;
  };
} // end of anonymous namespace

```
- **EN**: Implements logic around `EmitInterruptVectorSection`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `EmitInterruptVectorSection` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 63-70
```cpp
void MSP430AsmPrinter::PrintSymbolOperand(const MachineOperand &MO,
                                          raw_ostream &O) {
  uint64_t Offset = MO.getOffset();
  if (Offset)
    O << '(' << Offset << '+';

  getSymbol(MO.getGlobal())->print(O, MAI);

```
- **EN**: Implements logic around `PrintSymbolOperand`, `getOffset`, `getSymbol`; this block applies conditional target rules.
- **CN**: 围绕 `PrintSymbolOperand`, `getOffset`, `getSymbol` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 71-84
```cpp
  if (Offset)
    O << ')';
}

void MSP430AsmPrinter::printOperand(const MachineInstr *MI, int OpNum,
                                    raw_ostream &O, bool PrefixHash) {
  const MachineOperand &MO = MI->getOperand(OpNum);
  switch (MO.getType()) {
  default: llvm_unreachable("Not implemented yet!");
  case MachineOperand::MO_Register:
    O << MSP430InstPrinter::getRegisterName(MO.getReg());
    return;
  case MachineOperand::MO_Immediate:
    if (PrefixHash)
```
- **EN**: Implements logic around `printOperand`, `getOperand`, `llvm_unreachable`, `getRegisterName`; this block uses `switch`-based dispatch; applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `printOperand`, `getOperand`, `llvm_unreachable`, `getRegisterName` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 85-98
```cpp
      O << '#';
    O << MO.getImm();
    return;
  case MachineOperand::MO_MachineBasicBlock:
    MO.getMBB()->getSymbol()->print(O, MAI);
    return;
  case MachineOperand::MO_GlobalAddress: {
    // If the global address expression is a part of displacement field with a
    // register base, we should not emit any prefix symbol here, e.g.
    //   mov.w glb(r1), r2
    // Otherwise (!) msp430-as will silently miscompile the output :(
    if (PrefixHash)
      O << '#';
    PrintSymbolOperand(MO, O);
```
- **EN**: Implements logic around `getImm`, `getMBB`, `PrintSymbolOperand`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getImm`, `getMBB`, `PrintSymbolOperand` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 99-108
```cpp
    return;
  }
  }
}

void MSP430AsmPrinter::printSrcMemOperand(const MachineInstr *MI, int OpNum,
                                          raw_ostream &O) {
  const MachineOperand &Base = MI->getOperand(OpNum);
  const MachineOperand &Disp = MI->getOperand(OpNum+1);

```
- **EN**: Implements logic around `printSrcMemOperand`, `getOperand`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `printSrcMemOperand`, `getOperand` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 109-115
```cpp
  // Print displacement first

  // Imm here is in fact global address - print extra modifier.
  if (Disp.isImm() && Base.getReg() == MSP430::SR)
    O << '&';
  printOperand(MI, OpNum + 1, O, /*PrefixHash=*/false);

```
- **EN**: Implements logic around `printOperand`; this block applies conditional target rules.
- **CN**: 围绕 `printOperand` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 116-123
```cpp
  // Print register base field
  if (Base.getReg() != MSP430::SR && Base.getReg() != MSP430::PC) {
    O << '(';
    printOperand(MI, OpNum, O);
    O << ')';
  }
}

```
- **EN**: Implements logic around `printOperand`; this block applies conditional target rules.
- **CN**: 围绕 `printOperand` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 124-131
```cpp
/// PrintAsmOperand - Print out an operand for an inline asm expression.
///
bool MSP430AsmPrinter::PrintAsmOperand(const MachineInstr *MI, unsigned OpNo,
                                       const char *ExtraCode, raw_ostream &O) {
  // Does this asm operand have a single letter operand modifier?
  if (ExtraCode && ExtraCode[0])
    return AsmPrinter::PrintAsmOperand(MI, OpNo, ExtraCode, O);

```
- **EN**: Implements logic around `PrintAsmOperand`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `PrintAsmOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 132-145
```cpp
  printOperand(MI, OpNo, O);
  return false;
}

bool MSP430AsmPrinter::PrintAsmMemoryOperand(const MachineInstr *MI,
                                             unsigned OpNo,
                                             const char *ExtraCode,
                                             raw_ostream &O) {
  if (ExtraCode && ExtraCode[0]) {
    return true; // Unknown modifier.
  }
  printSrcMemOperand(MI, OpNo, O);
  return false;
}
```
- **EN**: Implements logic around `printOperand`, `PrintAsmMemoryOperand`, `printSrcMemOperand`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `printOperand`, `PrintAsmMemoryOperand`, `printSrcMemOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 146-153
```cpp

//===----------------------------------------------------------------------===//
void MSP430AsmPrinter::emitInstruction(const MachineInstr *MI) {
  MSP430_MC::verifyInstructionPredicates(MI->getOpcode(),
                                         getSubtargetInfo().getFeatureBits());

  MSP430MCInstLower MCInstLowering(OutContext, *this);

```
- **EN**: Implements logic around `emitInstruction`, `verifyInstructionPredicates`, `getSubtargetInfo`, `MCInstLowering`; this block works at the MachineInstr/MachineFunction layer; works at the MC layer.
- **CN**: 围绕 `emitInstruction`, `verifyInstructionPredicates`, `getSubtargetInfo`, `MCInstLowering` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层，工作在 MC 层。

### Lines 154-167
```cpp
  MCInst TmpInst;
  MCInstLowering.Lower(MI, TmpInst);
  EmitToStreamer(*OutStreamer, TmpInst);
}

void MSP430AsmPrinter::EmitInterruptVectorSection(MachineFunction &ISR) {
  MCSection *Cur = OutStreamer->getCurrentSectionOnly();
  const auto *F = &ISR.getFunction();
  if (F->getCallingConv() != CallingConv::MSP430_INTR) {
    report_fatal_error("Functions with 'interrupt' attribute must have msp430_intrcc CC");
  }
  StringRef IVIdx = F->getFnAttribute("interrupt").getValueAsString();
  MCSection *IV = OutStreamer->getContext().getELFSection(
    "__interrupt_vector_" + IVIdx,
```
- **EN**: Implements logic around `Lower`, `EmitToStreamer`, `EmitInterruptVectorSection`, `getCurrentSectionOnly`, ...; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer; works at the MC layer.
- **CN**: 围绕 `Lower`, `EmitToStreamer`, `EmitInterruptVectorSection`, `getCurrentSectionOnly`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层，工作在 MC 层。

### Lines 168-175
```cpp
    ELF::SHT_PROGBITS, ELF::SHF_ALLOC | ELF::SHF_EXECINSTR);
  OutStreamer->switchSection(IV);

  const MCSymbol *FunctionSymbol = getSymbol(F);
  OutStreamer->emitSymbolValue(FunctionSymbol, TM.getProgramPointerSize());
  OutStreamer->switchSection(Cur);
}

```
- **EN**: Implements logic around `switchSection`, `getSymbol`, `emitSymbolValue`; this block uses `switch`-based dispatch.
- **CN**: 围绕 `switchSection`, `getSymbol`, `emitSymbolValue` 实现具体逻辑；这一段使用 `switch` 分派。

### Lines 176-186
```cpp
bool MSP430AsmPrinter::runOnMachineFunction(MachineFunction &MF) {
  // Emit separate section for an interrupt vector if ISR
  if (MF.getFunction().hasFnAttribute("interrupt")) {
    EmitInterruptVectorSection(MF);
  }

  SetupMachineFunction(MF);
  emitFunctionBody();
  return false;
}

```
- **EN**: Implements logic around `runOnMachineFunction`, `EmitInterruptVectorSection`, `SetupMachineFunction`, `emitFunctionBody`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `runOnMachineFunction`, `EmitInterruptVectorSection`, `SetupMachineFunction`, `emitFunctionBody` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 187-196
```cpp
char MSP430AsmPrinter::ID = 0;

INITIALIZE_PASS(MSP430AsmPrinter, "msp430-asm-printer",
                "MSP430 Assembly Printer", false, false)

// Force static initialization.
extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void
LLVMInitializeMSP430AsmPrinter() {
  RegisterAsmPrinter<MSP430AsmPrinter> X(getTheMSP430Target());
}
```
- **EN**: Implements logic around `INITIALIZE_PASS`, `X`.
- **CN**: 围绕 `INITIALIZE_PASS`, `X` 实现具体逻辑。

## Key Concepts / 关键概念

- **Assembly emission / 汇编发射**:
  - **EN**: Lowers machine instructions to printable/emittable MC form
  - **CN**: 把机器指令降低为可打印/可发射的 MC 形式
- **Machine-level codegen / 机器级代码生成**:
  - **EN**: Operates after instruction selection on machine instructions and blocks
  - **CN**: 在指令选择后处理机器指令与基本块

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `MCTargetDesc/MSP430InstPrinter.h`, `MSP430MCInstLower.h`, `MSP430TargetMachine.h`, `TargetInfo/MSP430TargetInfo.h`, `llvm/BinaryFormat/ELF.h`, `llvm/CodeGen/AsmPrinter.h`, `llvm/CodeGen/MachineConstantPool.h`, `llvm/CodeGen/MachineInstr.h`, `llvm/CodeGen/MachineModuleInfo.h`, `llvm/IR/Mangler.h`, `llvm/MC/MCAsmInfo.h`, `llvm/MC/MCInst.h` ... (+6 more)
- **LLVM subsystems / LLVM 子系统**: MC, CodeGen, IR, Support
