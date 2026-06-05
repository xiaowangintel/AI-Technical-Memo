# XCoreAsmPrinter.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/XCore/XCoreAsmPrinter.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Lowers MachineInstr to the MC layer and emits target-specific assembly/object output.
  - **CN**: 把 MachineInstr 降到 MC 层，并输出目标相关的汇编/目标文件内容。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
//===-- XCoreAsmPrinter.cpp - XCore LLVM assembly writer ------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains a printer that converts from our internal representation
// of machine-dependent LLVM code to the XAS-format XCore assembly language.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 13-30
```cpp

#include "MCTargetDesc/XCoreInstPrinter.h"
#include "MCTargetDesc/XCoreTargetStreamer.h"
#include "TargetInfo/XCoreTargetInfo.h"
#include "XCore.h"
#include "XCoreMCInstLower.h"
#include "XCoreSubtarget.h"
#include "XCoreTargetMachine.h"
#include "llvm/ADT/SmallString.h"
#include "llvm/ADT/StringExtras.h"
#include "llvm/CodeGen/AsmPrinter.h"
#include "llvm/CodeGen/MachineConstantPool.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/MachineJumpTableInfo.h"
#include "llvm/CodeGen/MachineModuleInfo.h"
#include "llvm/IR/DataLayout.h"
#include "llvm/IR/DerivedTypes.h"
#include "llvm/IR/Mangler.h"
```
- **EN**: Pulls in the headers needed for this implementation, including `MCTargetDesc/XCoreInstPrinter.h`, `MCTargetDesc/XCoreTargetStreamer.h`, `TargetInfo/XCoreTargetInfo.h`, `XCore.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `MCTargetDesc/XCoreInstPrinter.h`, `MCTargetDesc/XCoreTargetStreamer.h`, `TargetInfo/XCoreTargetInfo.h`, `XCore.h`。

### Lines 31-43
```cpp
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCStreamer.h"
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Support/Compiler.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/raw_ostream.h"
#include "llvm/Target/TargetLoweringObjectFile.h"
#include <algorithm>
#include <cctype>
using namespace llvm;

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/MC/MCAsmInfo.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCInst.h`, `llvm/MC/MCStreamer.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/MC/MCAsmInfo.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCInst.h`, `llvm/MC/MCStreamer.h`。

### Lines 44-53
```cpp
#define DEBUG_TYPE "asm-printer"

namespace {
  class XCoreAsmPrinter : public AsmPrinter {
    XCoreMCInstLower MCInstLowering;
    XCoreTargetStreamer &getTargetStreamer();

  public:
    static char ID;

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 54-70
```cpp
    explicit XCoreAsmPrinter(TargetMachine &TM,
                             std::unique_ptr<MCStreamer> Streamer)
        : AsmPrinter(TM, std::move(Streamer), ID), MCInstLowering(*this) {}

    StringRef getPassName() const override { return "XCore Assembly Printer"; }

    void printInlineJT(const MachineInstr *MI, int opNum, raw_ostream &O,
                       const std::string &directive = ".jmptable");
    void printInlineJT32(const MachineInstr *MI, int opNum, raw_ostream &O) {
      printInlineJT(MI, opNum, O, ".jmptable32");
    }
    void printOperand(const MachineInstr *MI, int opNum, raw_ostream &O);
    bool PrintAsmOperand(const MachineInstr *MI, unsigned OpNo,
                         const char *ExtraCode, raw_ostream &O) override;
    bool PrintAsmMemoryOperand(const MachineInstr *MI, unsigned OpNum,
                               const char *ExtraCode, raw_ostream &O) override;

```
- **EN**: Implements logic around `XCoreAsmPrinter`, `AsmPrinter`, `getPassName`, `printInlineJT`, ...; this block returns target-specific results; works at the MachineInstr/MachineFunction layer; works at the MC layer.
- **CN**: 围绕 `XCoreAsmPrinter`, `AsmPrinter`, `getPassName`, `printInlineJT`, ... 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层，工作在 MC 层。

### Lines 71-80
```cpp
    void emitArrayBound(MCSymbol *Sym, const GlobalVariable *GV);
    void emitGlobalVariable(const GlobalVariable *GV) override;

    void emitFunctionEntryLabel() override;
    void emitInstruction(const MachineInstr *MI) override;
    void emitFunctionBodyStart() override;
    void emitFunctionBodyEnd() override;
  };
} // end of anonymous namespace

```
- **EN**: Implements logic around `emitArrayBound`, `emitGlobalVariable`, `emitFunctionEntryLabel`, `emitInstruction`, ...; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `emitArrayBound`, `emitGlobalVariable`, `emitFunctionEntryLabel`, `emitInstruction`, ... 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 81-90
```cpp
XCoreTargetStreamer &XCoreAsmPrinter::getTargetStreamer() {
  return static_cast<XCoreTargetStreamer&>(*OutStreamer->getTargetStreamer());
}

void XCoreAsmPrinter::emitArrayBound(MCSymbol *Sym, const GlobalVariable *GV) {
  assert( ( GV->hasExternalLinkage() || GV->hasWeakLinkage() ||
            GV->hasLinkOnceLinkage() || GV->hasCommonLinkage() ) &&
          "Unexpected linkage");
  if (ArrayType *ATy = dyn_cast<ArrayType>(GV->getValueType())) {

```
- **EN**: Implements logic around `getTargetStreamer`, `emitArrayBound`, `assert`, `hasLinkOnceLinkage`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getTargetStreamer`, `emitArrayBound`, `assert`, `hasLinkOnceLinkage` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 91-103
```cpp
    MCSymbol *SymGlob = OutContext.getOrCreateSymbol(
                          Twine(Sym->getName() + StringRef(".globound")));
    OutStreamer->emitSymbolAttribute(SymGlob, MCSA_Global);
    OutStreamer->emitAssignment(SymGlob,
                                MCConstantExpr::create(ATy->getNumElements(),
                                                       OutContext));
    if (GV->hasWeakLinkage() || GV->hasLinkOnceLinkage() ||
        GV->hasCommonLinkage()) {
      OutStreamer->emitSymbolAttribute(SymGlob, MCSA_Weak);
    }
  }
}

```
- **EN**: Implements logic around `getOrCreateSymbol`, `Twine`, `emitSymbolAttribute`, `emitAssignment`, ...; this block applies conditional target rules.
- **CN**: 围绕 `getOrCreateSymbol`, `Twine`, `emitSymbolAttribute`, `emitAssignment`, ... 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 104-115
```cpp
void XCoreAsmPrinter::emitGlobalVariable(const GlobalVariable *GV) {
  // Check to see if this is a special global used by LLVM, if so, emit it.
  if (!GV->hasInitializer() || emitSpecialLLVMGlobal(GV))
    return;

  const DataLayout &DL = getDataLayout();
  OutStreamer->switchSection(getObjFileLowering().SectionForGlobal(GV, TM));

  MCSymbol *GVSym = getSymbol(GV);
  const Constant *C = GV->getInitializer();
  const Align Alignment = DL.getPrefTypeAlign(C->getType());

```
- **EN**: Implements logic around `emitGlobalVariable`, `getDataLayout`, `switchSection`, `getSymbol`, ...; this block uses `switch`-based dispatch; applies conditional target rules.
- **CN**: 围绕 `emitGlobalVariable`, `getDataLayout`, `switchSection`, `getSymbol`, ... 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则。

### Lines 116-130
```cpp
  // Mark the start of the global
  getTargetStreamer().emitCCTopData(GVSym->getName());

  switch (GV->getLinkage()) {
  case GlobalValue::AppendingLinkage:
    report_fatal_error("AppendingLinkage is not supported by this target!");
  case GlobalValue::LinkOnceAnyLinkage:
  case GlobalValue::LinkOnceODRLinkage:
  case GlobalValue::WeakAnyLinkage:
  case GlobalValue::WeakODRLinkage:
  case GlobalValue::ExternalLinkage:
  case GlobalValue::CommonLinkage:
    emitArrayBound(GVSym, GV);
    OutStreamer->emitSymbolAttribute(GVSym, MCSA_Global);

```
- **EN**: Implements logic around `getTargetStreamer`, `report_fatal_error`, `emitArrayBound`, `emitSymbolAttribute`; this block uses `switch`-based dispatch.
- **CN**: 围绕 `getTargetStreamer`, `report_fatal_error`, `emitArrayBound`, `emitSymbolAttribute` 实现具体逻辑；这一段使用 `switch` 分派。

### Lines 131-141
```cpp
    if (GV->hasWeakLinkage() || GV->hasLinkOnceLinkage() ||
        GV->hasCommonLinkage())
      OutStreamer->emitSymbolAttribute(GVSym, MCSA_Weak);
    [[fallthrough]];
  case GlobalValue::InternalLinkage:
  case GlobalValue::PrivateLinkage:
    break;
  default:
    llvm_unreachable("Unknown linkage type!");
  }

```
- **EN**: Implements logic around `hasCommonLinkage`, `emitSymbolAttribute`, `llvm_unreachable`; this block applies conditional target rules.
- **CN**: 围绕 `hasCommonLinkage`, `emitSymbolAttribute`, `llvm_unreachable` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 142-153
```cpp
  emitAlignment(std::max(Alignment, Align(4)), GV);

  if (GV->isThreadLocal()) {
    report_fatal_error("TLS is not supported by this target!");
  }
  unsigned Size = DL.getTypeAllocSize(C->getType());
  if (MAI.hasDotTypeDotSizeDirective()) {
    OutStreamer->emitSymbolAttribute(GVSym, MCSA_ELF_TypeObject);
    OutStreamer->emitELFSize(GVSym, MCConstantExpr::create(Size, OutContext));
  }
  OutStreamer->emitLabel(GVSym);

```
- **EN**: Implements logic around `emitAlignment`, `report_fatal_error`, `getTypeAllocSize`, `emitSymbolAttribute`, ...; this block applies conditional target rules.
- **CN**: 围绕 `emitAlignment`, `report_fatal_error`, `getTypeAllocSize`, `emitSymbolAttribute`, ... 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 154-163
```cpp
  emitGlobalConstant(DL, C);
  // The ABI requires that unsigned scalar types smaller than 32 bits
  // are padded to 32 bits.
  if (Size < 4)
    OutStreamer->emitZeros(4 - Size);

  // Mark the end of the global
  getTargetStreamer().emitCCBottomData(GVSym->getName());
}

```
- **EN**: Implements logic around `emitGlobalConstant`, `emitZeros`, `getTargetStreamer`; this block applies conditional target rules.
- **CN**: 围绕 `emitGlobalConstant`, `emitZeros`, `getTargetStreamer` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 164-174
```cpp
void XCoreAsmPrinter::emitFunctionBodyStart() {
  MCInstLowering.Initialize(&MF->getContext());
}

/// EmitFunctionBodyEnd - Targets can override this to emit stuff after
/// the last basic block in the function.
void XCoreAsmPrinter::emitFunctionBodyEnd() {
  // Emit function end directives
  getTargetStreamer().emitCCBottomFunction(CurrentFnSym->getName());
}

```
- **EN**: Implements logic around `emitFunctionBodyStart`, `Initialize`, `emitFunctionBodyEnd`, `getTargetStreamer`; this block works at the MC layer.
- **CN**: 围绕 `emitFunctionBodyStart`, `Initialize`, `emitFunctionBodyEnd`, `getTargetStreamer` 实现具体逻辑；这一段工作在 MC 层。

### Lines 175-192
```cpp
void XCoreAsmPrinter::emitFunctionEntryLabel() {
  // Mark the start of the function
  getTargetStreamer().emitCCTopFunction(CurrentFnSym->getName());
  OutStreamer->emitLabel(CurrentFnSym);
}

void XCoreAsmPrinter::
printInlineJT(const MachineInstr *MI, int opNum, raw_ostream &O,
              const std::string &directive) {
  unsigned JTI = MI->getOperand(opNum).getIndex();
  const MachineFunction *MF = MI->getParent()->getParent();
  const MachineJumpTableInfo *MJTI = MF->getJumpTableInfo();
  const std::vector<MachineJumpTableEntry> &JT = MJTI->getJumpTables();
  const std::vector<MachineBasicBlock*> &JTBBs = JT[JTI].MBBs;
  O << "\t" << directive << " ";
  for (unsigned i = 0, e = JTBBs.size(); i != e; ++i) {
    MachineBasicBlock *MBB = JTBBs[i];
    if (i > 0)
```
- **EN**: Implements logic around `emitFunctionEntryLabel`, `getTargetStreamer`, `emitLabel`, `printInlineJT`, ...; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `emitFunctionEntryLabel`, `getTargetStreamer`, `emitLabel`, `printInlineJT`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 193-210
```cpp
      O << ",";
    MBB->getSymbol()->print(O, MAI);
  }
}

void XCoreAsmPrinter::printOperand(const MachineInstr *MI, int opNum,
                                   raw_ostream &O) {
  const DataLayout &DL = getDataLayout();
  const MachineOperand &MO = MI->getOperand(opNum);
  switch (MO.getType()) {
  case MachineOperand::MO_Register:
    O << XCoreInstPrinter::getRegisterName(MO.getReg());
    break;
  case MachineOperand::MO_Immediate:
    O << MO.getImm();
    break;
  case MachineOperand::MO_MachineBasicBlock:
    MO.getMBB()->getSymbol()->print(O, MAI);
```
- **EN**: Implements logic around `getSymbol`, `printOperand`, `getDataLayout`, `getOperand`, ...; this block uses `switch`-based dispatch; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getSymbol`, `printOperand`, `getDataLayout`, `getOperand`, ... 实现具体逻辑；这一段使用 `switch` 分派，工作在 MachineInstr/MachineFunction 层。

### Lines 211-226
```cpp
    break;
  case MachineOperand::MO_GlobalAddress:
    PrintSymbolOperand(MO, O);
    break;
  case MachineOperand::MO_ConstantPoolIndex:
    O << DL.getInternalSymbolPrefix() << "CPI" << getFunctionNumber() << '_'
      << MO.getIndex();
    break;
  case MachineOperand::MO_BlockAddress:
    GetBlockAddressSymbol(MO.getBlockAddress())->print(O, MAI);
    break;
  default:
    llvm_unreachable("not implemented");
  }
}

```
- **EN**: Implements logic around `PrintSymbolOperand`, `getInternalSymbolPrefix`, `getIndex`, `GetBlockAddressSymbol`, ....
- **CN**: 围绕 `PrintSymbolOperand`, `getInternalSymbolPrefix`, `getIndex`, `GetBlockAddressSymbol`, ... 实现具体逻辑。

### Lines 227-236
```cpp
/// PrintAsmOperand - Print out an operand for an inline asm expression.
///
bool XCoreAsmPrinter::PrintAsmOperand(const MachineInstr *MI, unsigned OpNo,
                                      const char *ExtraCode, raw_ostream &O) {
  // Print the operand if there is no operand modifier.
  if (!ExtraCode || !ExtraCode[0]) {
    printOperand(MI, OpNo, O);
    return false;
  }

```
- **EN**: Implements logic around `PrintAsmOperand`, `printOperand`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `PrintAsmOperand`, `printOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 237-254
```cpp
  // Otherwise fallback on the default implementation.
  return AsmPrinter::PrintAsmOperand(MI, OpNo, ExtraCode, O);
}

bool XCoreAsmPrinter::PrintAsmMemoryOperand(const MachineInstr *MI,
                                            unsigned OpNum,
                                            const char *ExtraCode,
                                            raw_ostream &O) {
  if (ExtraCode && ExtraCode[0]) {
    return true; // Unknown modifier.
  }
  printOperand(MI, OpNum, O);
  O << '[';
  printOperand(MI, OpNum + 1, O);
  O << ']';
  return false;
}

```
- **EN**: Implements logic around `PrintAsmOperand`, `PrintAsmMemoryOperand`, `printOperand`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `PrintAsmOperand`, `PrintAsmMemoryOperand`, `printOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 255-272
```cpp
void XCoreAsmPrinter::emitInstruction(const MachineInstr *MI) {
  XCore_MC::verifyInstructionPredicates(MI->getOpcode(),
                                        getSubtargetInfo().getFeatureBits());

  SmallString<128> Str;
  raw_svector_ostream O(Str);

  switch (MI->getOpcode()) {
  case XCore::DBG_VALUE:
    llvm_unreachable("Should be handled target independently");
  case XCore::ADD_2rus:
    if (MI->getOperand(2).getImm() == 0) {
      O << "\tmov "
        << XCoreInstPrinter::getRegisterName(MI->getOperand(0).getReg()) << ", "
        << XCoreInstPrinter::getRegisterName(MI->getOperand(1).getReg());
      OutStreamer->emitRawText(O.str());
      return;
    }
```
- **EN**: Implements logic around `emitInstruction`, `verifyInstructionPredicates`, `getSubtargetInfo`, `O`, ...; this block uses `switch`-based dispatch; applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `emitInstruction`, `verifyInstructionPredicates`, `getSubtargetInfo`, `O`, ... 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 273-286
```cpp
    break;
  case XCore::BR_JT:
  case XCore::BR_JT32:
    O << "\tbru "
      << XCoreInstPrinter::getRegisterName(MI->getOperand(1).getReg()) << '\n';
    if (MI->getOpcode() == XCore::BR_JT)
      printInlineJT(MI, 0, O);
    else
      printInlineJT32(MI, 0, O);
    O << '\n';
    OutStreamer->emitRawText(O.str());
    return;
  }

```
- **EN**: Implements logic around `getRegisterName`, `printInlineJT`, `printInlineJT32`, `emitRawText`; this block applies conditional target rules.
- **CN**: 围绕 `getRegisterName`, `printInlineJT`, `printInlineJT32`, `emitRawText` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 287-297
```cpp
  MCInst TmpInst;
  MCInstLowering.Lower(MI, TmpInst);

  EmitToStreamer(*OutStreamer, TmpInst);
}

char XCoreAsmPrinter::ID = 0;

INITIALIZE_PASS(XCoreAsmPrinter, "xcore-asm-printer", "XCore Assembly Printer",
                false, false)

```
- **EN**: Implements logic around `Lower`, `EmitToStreamer`, `INITIALIZE_PASS`; this block works at the MC layer.
- **CN**: 围绕 `Lower`, `EmitToStreamer`, `INITIALIZE_PASS` 实现具体逻辑；这一段工作在 MC 层。

### Lines 298-302
```cpp
// Force static initialization.
extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void
LLVMInitializeXCoreAsmPrinter() {
  RegisterAsmPrinter<XCoreAsmPrinter> X(getTheXCoreTarget());
}
```
- **EN**: Implements logic around `X`.
- **CN**: 围绕 `X` 实现具体逻辑。

## Key Concepts / 关键概念

- **Assembly emission / 汇编发射**:
  - **EN**: Lowers machine instructions to printable/emittable MC form
  - **CN**: 把机器指令降低为可打印/可发射的 MC 形式
- **Machine-level codegen / 机器级代码生成**:
  - **EN**: Operates after instruction selection on machine instructions and blocks
  - **CN**: 在指令选择后处理机器指令与基本块

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `MCTargetDesc/XCoreInstPrinter.h`, `MCTargetDesc/XCoreTargetStreamer.h`, `TargetInfo/XCoreTargetInfo.h`, `XCore.h`, `XCoreMCInstLower.h`, `XCoreSubtarget.h`, `XCoreTargetMachine.h`, `llvm/ADT/SmallString.h`, `llvm/ADT/StringExtras.h`, `llvm/CodeGen/AsmPrinter.h`, `llvm/CodeGen/MachineConstantPool.h`, `llvm/CodeGen/MachineInstr.h` ... (+16 more)
- **LLVM subsystems / LLVM 子系统**: MC, CodeGen, IR, Support
