# ARMAsmPrinter.h — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMAsmPrinter.h`
- Repository: `llvm-project`
- Purpose (EN): Declares `ARMAsmPrinter` for the ARM backend and exposes interfaces for assembly printing and MC emission.
- 用途 (CN): 声明 ARM 后端中的 `ARMAsmPrinter`，并提供与汇编打印与 MC 发射相关的接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- ARMAsmPrinter.h - ARM implementation of AsmPrinter ------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 9-10
```cpp
#ifndef LLVM_LIB_TARGET_ARM_ARMASMPRINTER_H
#define LLVM_LIB_TARGET_ARM_ARMASMPRINTER_H
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 12-13
```cpp
#include "llvm/CodeGen/AsmPrinter.h"
#include "llvm/Target/TargetMachine.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 15-15
```cpp
namespace llvm {
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 17-22
```cpp
class ARMFunctionInfo;
class ARMBaseTargetMachine;
class MCOperand;
class MachineConstantPool;
class MachineOperand;
class MCSymbol;
```
- EN: Declares `ARMFunctionInfo`, packaging target-specific state and APIs around `ARMAsmPrinter`.
- CN: 这里声明 `ARMFunctionInfo`，把与 `ARMAsmPrinter` 相关的目标特定状态和 API 组织在一起。

### Lines 24-29
```cpp
namespace ARM {
  enum DW_ISA {
    DW_ISA_ARM_thumb = 1,
    DW_ISA_ARM_arm = 2
  };
}
```
- EN: Defines enumeration `DW_ISA` to name backend-specific modes, cases, or flags used elsewhere in the file.
- CN: 这里定义枚举 `DW_ISA`，用于命名本文件后续会使用的后端模式、分支情况或标志。

### Lines 31-33
```cpp
class LLVM_LIBRARY_VISIBILITY ARMAsmPrinter : public AsmPrinter {
public:
  static char ID;
```
- EN: Declares `LLVM_LIBRARY_VISIBILITY`, packaging target-specific state and APIs around `ARMAsmPrinter`.
- CN: 这里声明 `LLVM_LIBRARY_VISIBILITY`，把与 `ARMAsmPrinter` 相关的目标特定状态和 API 组织在一起。

### Lines 35-38
```cpp
private:
  /// AFI - Keep a pointer to ARMFunctionInfo for the current
  /// MachineFunction.
  ARMFunctionInfo *AFI;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 40-42
```cpp
  /// MCP - Keep a pointer to constantpool entries of the current
  /// MachineFunction.
  const MachineConstantPool *MCP;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 44-46
```cpp
  /// InConstantPool - Maintain state when emitting a sequence of constant
  /// pool entries so we can properly mark them as data regions.
  bool InConstantPool;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 48-50
```cpp
  /// ThumbIndirectPads - These maintain a per-function list of jump pad
  /// labels used for ARMv4t thumb code to make register indirect calls.
  SmallVector<std::pair<unsigned, MCSymbol*>, 4> ThumbIndirectPads;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 52-55
```cpp
  /// OptimizationGoals - Maintain a combined optimization goal for all
  /// functions in a module: one of Tag_ABI_optimization_goals values,
  /// -1 if uninitialized, 0 if conflicting goals
  int OptimizationGoals;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 57-65
```cpp
  /// List of globals that have had their storage promoted to a constant
  /// pool. This lives between calls to runOnMachineFunction and collects
  /// data from every MachineFunction. It is used during doFinalization
  /// when all non-function globals are emitted.
  SmallPtrSet<const GlobalVariable*,2> PromotedGlobals;
  /// Set of globals in PromotedGlobals that we've emitted labels for.
  /// We need to emit labels even for promoted globals so that DWARF
  /// debug info can link properly.
  SmallPtrSet<const GlobalVariable*,2> EmittedPromotedGlobalLabels;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 67-69
```cpp
public:
  explicit ARMAsmPrinter(TargetMachine &TM,
                         std::unique_ptr<MCStreamer> Streamer);
```
- EN: Declares `ARMAsmPrinter`, a target-specific routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ARMAsmPrinter`，它是一个围绕MC streamer 交互展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 71-73
```cpp
  StringRef getPassName() const override {
    return "ARM Assembly Printer";
  }
```
- EN: Implements `getPassName`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getPassName`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 75-75
```cpp
  const ARMBaseTargetMachine &getTM() const;
```
- EN: Declares `getTM`, a query/helper routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getTM`，它是一个围绕目标机器策略展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 77-77
```cpp
  void printOperand(const MachineInstr *MI, int OpNum, raw_ostream &O);
```
- EN: Declares `printOperand`, a emission/printing routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `printOperand`，它是一个围绕机器指令展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 79-83
```cpp
  void PrintSymbolOperand(const MachineOperand &MO, raw_ostream &O) override;
  bool PrintAsmOperand(const MachineInstr *MI, unsigned OpNum,
                       const char *ExtraCode, raw_ostream &O) override;
  bool PrintAsmMemoryOperand(const MachineInstr *MI, unsigned OpNum,
                             const char *ExtraCode, raw_ostream &O) override;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 85-87
```cpp
  void emitInlineAsmEnd(const MCSubtargetInfo &StartInfo,
                        const MCSubtargetInfo *EndInfo,
                        const MachineInstr *MI) override;
```
- EN: Declares `emitInlineAsmEnd`, a emission/printing routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitInlineAsmEnd`，它是一个围绕机器指令展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 89-97
```cpp
  void emitJumpTableAddrs(const MachineInstr *MI);
  void emitJumpTableInsts(const MachineInstr *MI);
  void emitJumpTableTBInst(const MachineInstr *MI, unsigned OffsetWidth);
  void emitInstruction(const MachineInstr *MI) override;
  bool runOnMachineFunction(MachineFunction &F) override;
  std::tuple<const MCSymbol *, uint64_t, const MCSymbol *,
             codeview::JumpTableEntrySize>
  getCodeViewJumpTableInfo(int JTI, const MachineInstr *BranchInstr,
                           const MCSymbol *BranchLabel) const override;
```
- EN: Declares `emitJumpTableAddrs`, a emission/printing routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitJumpTableAddrs`，它是一个围绕机器函数状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 99-108
```cpp
  void emitConstantPool() override {
    // we emit constant pools customly!
  }
  void emitFunctionBodyEnd() override;
  void emitFunctionEntryLabel() override;
  void emitStartOfAsmFile(Module &M) override;
  void emitEndOfAsmFile(Module &M) override;
  void emitXXStructor(const DataLayout &DL, const Constant *CV) override;
  void emitGlobalVariable(const GlobalVariable *GV) override;
  void emitGlobalAlias(const Module &M, const GlobalAlias &GA) override;
```
- EN: Implements `emitConstantPool`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `emitConstantPool`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 110-110
```cpp
  MCSymbol *GetCPISymbol(unsigned CPID) const override;
```
- EN: Declares `GetCPISymbol`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `GetCPISymbol`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 112-113
```cpp
  // lowerOperand - Convert a MachineOperand into the equivalent MCOperand.
  bool lowerOperand(const MachineOperand &MO, MCOperand &MCOp);
```
- EN: Declares `lowerOperand`, a lowering routine centered on machine operands. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `lowerOperand`，它是一个围绕机器操作数展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 115-122
```cpp
  //===------------------------------------------------------------------===//
  // XRay implementation
  //===------------------------------------------------------------------===//
public:
  // XRay-specific lowering for ARM.
  void LowerPATCHABLE_FUNCTION_ENTER(const MachineInstr &MI);
  void LowerPATCHABLE_FUNCTION_EXIT(const MachineInstr &MI);
  void LowerPATCHABLE_TAIL_CALL(const MachineInstr &MI);
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 124-125
```cpp
  // KCFI check lowering
  void LowerKCFI_CHECK(const MachineInstr &MI);
```
- EN: Declares `LowerKCFI_CHECK`, a lowering routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LowerKCFI_CHECK`，它是一个围绕机器指令展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 127-128
```cpp
private:
  void EmitSled(const MachineInstr &MI, SledKind Kind);
```
- EN: Declares `EmitSled`, a emission/printing routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `EmitSled`，它是一个围绕机器指令展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 130-136
```cpp
  // KCFI check emission helpers
  void EmitKCFI_CHECK_ARM32(Register AddrReg, int64_t Type,
                            const MachineInstr &Call, int64_t PrefixNops);
  void EmitKCFI_CHECK_Thumb2(Register AddrReg, int64_t Type,
                             const MachineInstr &Call, int64_t PrefixNops);
  void EmitKCFI_CHECK_Thumb1(Register AddrReg, int64_t Type,
                             const MachineInstr &Call, int64_t PrefixNops);
```
- EN: Declares `EmitKCFI_CHECK_ARM32`, a emission/printing routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `EmitKCFI_CHECK_ARM32`，它是一个围绕机器指令展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 138-139
```cpp
  // Helpers for emitStartOfAsmFile() and emitEndOfAsmFile()
  void emitAttributes();
```
- EN: Declares `emitAttributes`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitAttributes`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 141-141
```cpp
  void EmitUnwindingInstruction(const MachineInstr *MI);
```
- EN: Declares `EmitUnwindingInstruction`, a emission/printing routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `EmitUnwindingInstruction`，它是一个围绕机器指令展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 143-144
```cpp
  // tblgen'erated.
  bool lowerPseudoInstExpansion(const MachineInstr *MI, MCInst &Inst);
```
- EN: Declares `lowerPseudoInstExpansion`, a lowering routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `lowerPseudoInstExpansion`，它是一个围绕MC 指令构造展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 146-156
```cpp
public:
  unsigned getISAEncoding() override {
    // ARM/Darwin adds ISA to the DWARF info for each function.
    const Triple &TT = TM.getTargetTriple();
    if (!TT.isOSBinFormatMachO())
      return 0;
    bool isThumb = TT.isThumb() ||
                   TT.getSubArch() == Triple::ARMSubArch_v7m ||
                   TT.getSubArch() == Triple::ARMSubArch_v6m;
    return isThumb ? ARM::DW_ISA_ARM_thumb : ARM::DW_ISA_ARM_arm;
  }
```
- EN: Implements `getISAEncoding`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getISAEncoding`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 158-160
```cpp
private:
  MCOperand GetSymbolRef(const MachineOperand &MO, const MCSymbol *Symbol);
  MCSymbol *GetARMJTIPICJumpTableLabel(unsigned uid) const;
```
- EN: Declares `GetSymbolRef`, a query/helper routine centered on machine operands. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `GetSymbolRef`，它是一个围绕机器操作数展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 162-162
```cpp
  MCSymbol *GetARMGVSymbol(const GlobalValue *GV, unsigned char TargetFlags);
```
- EN: Declares `GetARMGVSymbol`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `GetARMGVSymbol`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 164-164
```cpp
  void emitCMSEVeneerAlias(const GlobalAlias &GA);
```
- EN: Declares `emitCMSEVeneerAlias`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitCMSEVeneerAlias`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 166-170
```cpp
public:
  /// EmitMachineConstantPoolValue - Print a machine constantpool value to
  /// the .s file.
  void emitMachineConstantPoolValue(MachineConstantPoolValue *MCPV) override;
};
```
- EN: Declares `emitMachineConstantPoolValue`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitMachineConstantPoolValue`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 172-172
```cpp
} // end namespace llvm
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 174-174
```cpp
#endif
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

## Key Concepts / 关键概念

- EN: Primary role: assembly printing and MC emission.
  - CN: 核心职责：汇编打印与 MC 发射。
- EN: MachineInstr-level manipulation is central to this file.
  - CN: 该文件以 MachineInstr 层面的操作为核心。
- EN: The MC layer is used to print, encode, or stream target instructions and symbols.
  - CN: MC 层被用于打印、编码或流式输出目标指令与符号。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。
- EN: The component plugs into LLVM machine-function passes or codegen pipelines.
  - CN: 该组件会接入 LLVM 的机器函数 Pass 或代码生成流水线。

## Dependencies / 依赖关系

- EN: Backend-local headers: `(none)`.
  - CN: 后端本地头文件：`(none)`。
- EN: LLVM infrastructure headers: `llvm/CodeGen/AsmPrinter.h`, `llvm/Target/TargetMachine.h`.
  - CN: LLVM 基础设施头文件：`llvm/CodeGen/AsmPrinter.h`, `llvm/Target/TargetMachine.h`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
- EN: The implementation also depends on LLVM MC layer classes for final assembly/object emission.
  - CN: 实现还依赖 LLVM 的 MC 层类来完成最终汇编/目标文件输出。
