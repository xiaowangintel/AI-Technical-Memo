# MipsAsmPrinter.h — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MipsAsmPrinter.h`
- Repository: `llvm-project`
- Purpose (EN): Declares `MipsAsmPrinter` for the Mips backend and exposes interfaces for assembly printing and MC emission.
- 用途 (CN): 声明 Mips 后端中的 `MipsAsmPrinter`，并提供与汇编打印与 MC 发射相关的接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===- MipsAsmPrinter.h - Mips LLVM Assembly Printer -----------*- C++ -*--===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Mips Assembly printer class.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 13-14
```cpp
#ifndef LLVM_LIB_TARGET_MIPS_MIPSASMPRINTER_H
#define LLVM_LIB_TARGET_MIPS_MIPSASMPRINTER_H
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 16-24
```cpp
#include "Mips16HardFloatInfo.h"
#include "MipsMCInstLower.h"
#include "MipsSubtarget.h"
#include "llvm/CodeGen/AsmPrinter.h"
#include "llvm/MC/MCStreamer.h"
#include "llvm/Support/Compiler.h"
#include <algorithm>
#include <map>
#include <memory>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 26-26
```cpp
namespace llvm {
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 28-40
```cpp
class MCOperand;
class MCSubtargetInfo;
class MCSymbol;
class MachineBasicBlock;
class MachineConstantPool;
class MachineFunction;
class MachineInstr;
class MachineOperand;
class MipsFunctionInfo;
class MipsTargetStreamer;
class Module;
class raw_ostream;
class TargetMachine;
```
- EN: Declares `MCOperand`, packaging target-specific state and APIs around `MipsAsmPrinter`.
- CN: 这里声明 `MCOperand`，把与 `MipsAsmPrinter` 相关的目标特定状态和 API 组织在一起。

### Lines 42-43
```cpp
class LLVM_LIBRARY_VISIBILITY MipsAsmPrinter : public AsmPrinter {
  MipsTargetStreamer &getTargetStreamer() const;
```
- EN: Declares `LLVM_LIBRARY_VISIBILITY`, packaging target-specific state and APIs around `MipsAsmPrinter`.
- CN: 这里声明 `LLVM_LIBRARY_VISIBILITY`，把与 `MipsAsmPrinter` 相关的目标特定状态和 API 组织在一起。

### Lines 45-45
```cpp
  void EmitInstrWithMacroNoAT(const MachineInstr *MI);
```
- EN: Declares `EmitInstrWithMacroNoAT`, a emission/printing routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `EmitInstrWithMacroNoAT`，它是一个围绕机器指令展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 47-49
```cpp
  //===------------------------------------------------------------------===//
  // XRay implementation
  //===------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 51-55
```cpp
public:
  // XRay-specific lowering for Mips.
  void LowerPATCHABLE_FUNCTION_ENTER(const MachineInstr &MI);
  void LowerPATCHABLE_FUNCTION_EXIT(const MachineInstr &MI);
  void LowerPATCHABLE_TAIL_CALL(const MachineInstr &MI);
```
- EN: Declares `LowerPATCHABLE_FUNCTION_ENTER`, a lowering routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LowerPATCHABLE_FUNCTION_ENTER`，它是一个围绕机器指令展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 57-60
```cpp
private:
  /// MCP - Keep a pointer to constantpool entries of the current
  /// MachineFunction.
  const MachineConstantPool *MCP = nullptr;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 62-64
```cpp
  /// InConstantPool - Maintain state when emitting a sequence of constant
  /// pool entries so we can properly mark them as data regions.
  bool InConstantPool = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 66-67
```cpp
  std::map<const char *, const Mips16HardFloatInfo::FuncSignature *>
      StubsNeeded;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 69-69
```cpp
  void EmitSled(const MachineInstr &MI, SledKind Kind);
```
- EN: Declares `EmitSled`, a emission/printing routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `EmitSled`，它是一个围绕机器指令展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 71-72
```cpp
  // tblgen'erated function.
  bool lowerPseudoInstExpansion(const MachineInstr *MI, MCInst &Inst);
```
- EN: Declares `lowerPseudoInstExpansion`, a lowering routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `lowerPseudoInstExpansion`，它是一个围绕MC 指令构造展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 74-78
```cpp
  // Emit PseudoReturn, PseudoReturn64, PseudoIndirectBranch,
  // and PseudoIndirectBranch64 as a JR, JR_MM, JALR, or JALR64 as appropriate
  // for the target.
  void emitPseudoIndirectBranch(MCStreamer &OutStreamer,
                                const MachineInstr *MI);
```
- EN: Declares `emitPseudoIndirectBranch`, a emission/printing routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitPseudoIndirectBranch`，它是一个围绕MC streamer 交互展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 80-81
```cpp
  // lowerOperand - Convert a MachineOperand into the equivalent MCOperand.
  bool lowerOperand(const MachineOperand &MO, MCOperand &MCOp);
```
- EN: Declares `lowerOperand`, a lowering routine centered on machine operands. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `lowerOperand`，它是一个围绕机器操作数展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 83-83
```cpp
  void emitInlineAsmStart() const override;
```
- EN: Declares `emitInlineAsmStart`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitInlineAsmStart`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 85-87
```cpp
  void emitInlineAsmEnd(const MCSubtargetInfo &StartInfo,
                        const MCSubtargetInfo *EndInfo,
                        const MachineInstr *MI) override;
```
- EN: Declares `emitInlineAsmEnd`, a emission/printing routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitInlineAsmEnd`，它是一个围绕机器指令展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 89-91
```cpp
  void emitJumpTableEntry(const MachineJumpTableInfo &MJTI,
                          const MachineBasicBlock *MBB,
                          unsigned uid) const override;
```
- EN: Declares `emitJumpTableEntry`, a emission/printing routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitJumpTableEntry`，它是一个围绕机器基本块展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 93-93
```cpp
  void EmitJal(const MCSubtargetInfo &STI, MCSymbol *Symbol);
```
- EN: Declares `EmitJal`, a emission/printing routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `EmitJal`，它是一个围绕子目标特性处理展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 95-95
```cpp
  void EmitInstrReg(const MCSubtargetInfo &STI, unsigned Opcode, unsigned Reg);
```
- EN: Declares `EmitInstrReg`, a emission/printing routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `EmitInstrReg`，它是一个围绕子目标特性处理展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 97-98
```cpp
  void EmitInstrRegReg(const MCSubtargetInfo &STI, unsigned Opcode,
                       unsigned Reg1, unsigned Reg2);
```
- EN: Declares `EmitInstrRegReg`, a emission/printing routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `EmitInstrRegReg`，它是一个围绕子目标特性处理展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 100-101
```cpp
  void EmitInstrRegRegReg(const MCSubtargetInfo &STI, unsigned Opcode,
                          unsigned Reg1, unsigned Reg2, unsigned Reg3);
```
- EN: Declares `EmitInstrRegRegReg`, a emission/printing routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `EmitInstrRegRegReg`，它是一个围绕子目标特性处理展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 103-105
```cpp
  void EmitMovFPIntPair(const MCSubtargetInfo &STI, unsigned MovOpc,
                        unsigned Reg1, unsigned Reg2, unsigned FPReg1,
                        unsigned FPReg2, bool LE);
```
- EN: Declares `EmitMovFPIntPair`, a emission/printing routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `EmitMovFPIntPair`，它是一个围绕子目标特性处理展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 107-109
```cpp
  void EmitSwapFPIntParams(const MCSubtargetInfo &STI,
                           Mips16HardFloatInfo::FPParamVariant, bool LE,
                           bool ToFP);
```
- EN: Declares `EmitSwapFPIntParams`, a emission/printing routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `EmitSwapFPIntParams`，它是一个围绕子目标特性处理展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 111-112
```cpp
  void EmitSwapFPIntRetval(const MCSubtargetInfo &STI,
                           Mips16HardFloatInfo::FPReturnVariant, bool LE);
```
- EN: Declares `EmitSwapFPIntRetval`, a emission/printing routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `EmitSwapFPIntRetval`，它是一个围绕子目标特性处理展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 114-114
```cpp
  void EmitFPCallStub(const char *, const Mips16HardFloatInfo::FuncSignature *);
```
- EN: Declares `EmitFPCallStub`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `EmitFPCallStub`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 116-116
```cpp
  bool isLongBranchPseudo(int Opcode) const;
```
- EN: Declares `isLongBranchPseudo`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isLongBranchPseudo`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 118-119
```cpp
public:
  static char ID;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 121-123
```cpp
  const MipsSubtarget *Subtarget;
  const MipsFunctionInfo *MipsFI;
  MipsMCInstLower MCInstLowering;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 125-127
```cpp
  explicit MipsAsmPrinter(TargetMachine &TM,
                          std::unique_ptr<MCStreamer> Streamer)
      : AsmPrinter(TM, std::move(Streamer), ID), MCInstLowering(*this) {}
```
- EN: Implements `std::move`, a target-specific routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `std::move`，它是一个围绕MC streamer 交互展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 129-129
```cpp
  StringRef getPassName() const override { return "Mips Assembly Printer"; }
```
- EN: Implements `getPassName`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getPassName`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 131-131
```cpp
  bool runOnMachineFunction(MachineFunction &MF) override;
```
- EN: Declares `runOnMachineFunction`, a pass-entry routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `runOnMachineFunction`，它是一个围绕机器函数状态展开的Pass 入口例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 133-139
```cpp
  void emitConstantPool() override {
    bool UsingConstantPools =
      (Subtarget->inMips16Mode() && Subtarget->useConstantIslands());
    if (!UsingConstantPools)
      AsmPrinter::emitConstantPool();
    // we emit constant pools customly!
  }
```
- EN: Implements `emitConstantPool`, a emission/printing routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `emitConstantPool`，它是一个围绕子目标特性处理展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 141-155
```cpp
  void emitInstruction(const MachineInstr *MI) override;
  void printSavedRegsBitmask();
  void emitFrameDirective();
  const char *getCurrentABIString() const;
  void emitFunctionEntryLabel() override;
  void emitFunctionBodyStart() override;
  void emitFunctionBodyEnd() override;
  void emitBasicBlockEnd(const MachineBasicBlock &MBB) override;
  bool PrintAsmOperand(const MachineInstr *MI, unsigned OpNo,
                       const char *ExtraCode, raw_ostream &O) override;
  bool PrintAsmMemoryOperand(const MachineInstr *MI, unsigned OpNum,
                             const char *ExtraCode, raw_ostream &O) override;
  void printOperand(const MachineInstr *MI, int opNum, raw_ostream &O);
  void printMemOperand(const MachineInstr *MI, int opNum, raw_ostream &O);
  void printMemOperandEA(const MachineInstr *MI, int opNum, raw_ostream &O);
```
- EN: Declares `emitInstruction`, a emission/printing routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitInstruction`，它是一个围绕机器基本块展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 156-162
```cpp
  void printFCCOperand(const MachineInstr *MI, int opNum, raw_ostream &O);
  void printRegisterList(const MachineInstr *MI, int opNum, raw_ostream &O);
  void emitStartOfAsmFile(Module &M) override;
  void emitEndOfAsmFile(Module &M) override;
  void PrintDebugValueComment(const MachineInstr *MI, raw_ostream &OS);
  void emitDebugValue(const MCExpr *Value, unsigned Size) const override;
};
```
- EN: Declares `printFCCOperand`, a emission/printing routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `printFCCOperand`，它是一个围绕机器指令展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 164-164
```cpp
} // end namespace llvm
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 166-166
```cpp
#endif // LLVM_LIB_TARGET_MIPS_MIPSASMPRINTER_H
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

## Key Concepts / 关键概念

- EN: Primary role: assembly printing and MC emission.
  - CN: 核心职责：汇编打印与 MC 发射。
- EN: MachineInstr-level manipulation is central to this file.
  - CN: 该文件以 MachineInstr 层面的操作为核心。
- EN: MachineBasicBlock flow and branch structure matter to the implementation.
  - CN: 实现高度依赖 MachineBasicBlock 流程与分支结构。
- EN: The MC layer is used to print, encode, or stream target instructions and symbols.
  - CN: MC 层被用于打印、编码或流式输出目标指令与符号。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。

## Dependencies / 依赖关系

- EN: Backend-local headers: `Mips16HardFloatInfo.h`, `MipsMCInstLower.h`, `MipsSubtarget.h`.
  - CN: 后端本地头文件：`Mips16HardFloatInfo.h`, `MipsMCInstLower.h`, `MipsSubtarget.h`。
- EN: LLVM infrastructure headers: `llvm/CodeGen/AsmPrinter.h`, `llvm/MC/MCStreamer.h`, `llvm/Support/Compiler.h`.
  - CN: LLVM 基础设施头文件：`llvm/CodeGen/AsmPrinter.h`, `llvm/MC/MCStreamer.h`, `llvm/Support/Compiler.h`。
- EN: Standard/system headers: `algorithm`, `map`, `memory`.
  - CN: 标准库/系统头文件：`algorithm`, `map`, `memory`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
- EN: The implementation also depends on LLVM MC layer classes for final assembly/object emission.
  - CN: 实现还依赖 LLVM 的 MC 层类来完成最终汇编/目标文件输出。
