# MicroMipsSizeReduction.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MicroMipsSizeReduction.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `MicroMipsSizeReduction` for the Mips backend, focusing on target-specific code-generation behavior.
- 用途 (CN): 实现 Mips 后端中的 `MicroMipsSizeReduction`，重点处理目标相关代码生成行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15
```cpp
//=== MicroMipsSizeReduction.cpp - MicroMips size reduction pass --------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
///\file
/// This pass is used to reduce the size of instructions where applicable.
///
/// TODO: Implement microMIPS64 support.
//===----------------------------------------------------------------------===//
#include "Mips.h"
#include "MipsInstrInfo.h"
#include "MipsSubtarget.h"
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 16-18
```cpp
#include "llvm/ADT/Statistic.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/Support/Debug.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 20-20
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 22-23
```cpp
#define DEBUG_TYPE "micromips-reduce-size"
#define MICROMIPS_SIZE_REDUCE_NAME "MicroMips instruction size reduce pass"
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 25-26
```cpp
STATISTIC(NumReduced, "Number of instructions reduced (32-bit to 16-bit ones, "
                      "or two instructions into one");
```
- EN: Declares `STATISTIC`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `STATISTIC`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 28-28
```cpp
namespace {
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 30-41
```cpp
/// Order of operands to transfer
// TODO: Will be extended when additional optimizations are added
enum OperandTransfer {
  OT_NA,            ///< Not applicable
  OT_OperandsAll,   ///< Transfer all operands
  OT_Operands02,    ///< Transfer operands 0 and 2
  OT_Operand2,      ///< Transfer just operand 2
  OT_OperandsXOR,   ///< Transfer operands for XOR16
  OT_OperandsLwp,   ///< Transfer operands for LWP
  OT_OperandsSwp,   ///< Transfer operands for SWP
  OT_OperandsMovep, ///< Transfer operands for MOVEP
};
```
- EN: Defines enumeration `OperandTransfer` to name backend-specific modes, cases, or flags used elsewhere in the file.
- CN: 这里定义枚举 `OperandTransfer`，用于命名本文件后续会使用的后端模式、分支情况或标志。

### Lines 43-48
```cpp
/// Reduction type
// TODO: Will be extended when additional optimizations are added
enum ReduceType {
  RT_TwoInstr, ///< Reduce two instructions into one instruction
  RT_OneInstr  ///< Reduce one instruction into a smaller instruction
};
```
- EN: Defines enumeration `ReduceType` to name backend-specific modes, cases, or flags used elsewhere in the file.
- CN: 这里定义枚举 `ReduceType`，用于命名本文件后续会使用的后端模式、分支情况或标志。

### Lines 50-61
```cpp
// Information about immediate field restrictions
struct ImmField {
  ImmField() : ImmFieldOperand(-1), Shift(0), LBound(0), HBound(0) {}
  ImmField(uint8_t Shift, int16_t LBound, int16_t HBound,
           int8_t ImmFieldOperand)
      : ImmFieldOperand(ImmFieldOperand), Shift(Shift), LBound(LBound),
        HBound(HBound) {}
  int8_t ImmFieldOperand; // Immediate operand, -1 if it does not exist
  uint8_t Shift;          // Shift value
  int16_t LBound;         // Low bound of the immediate operand
  int16_t HBound;         // High bound of the immediate operand
};
```
- EN: Declares `ImmField`, packaging target-specific state and APIs around `MicroMipsSizeReduction`.
- CN: 这里声明 `ImmField`，把与 `MicroMipsSizeReduction` 相关的目标特定状态和 API 组织在一起。

### Lines 63-68
```cpp
/// Information about operands
// TODO: Will be extended when additional optimizations are added
struct OpInfo {
  OpInfo(enum OperandTransfer TransferOperands)
      : TransferOperands(TransferOperands) {}
  OpInfo() : TransferOperands(OT_NA) {}
```
- EN: Declares `OpInfo`, packaging target-specific state and APIs around `MicroMipsSizeReduction`.
- CN: 这里声明 `OpInfo`，把与 `MicroMipsSizeReduction` 相关的目标特定状态和 API 组织在一起。

### Lines 70-72
```cpp
  enum OperandTransfer
      TransferOperands; ///< Operands to transfer to the new instruction
};
```
- EN: Defines enumeration `OperandTransfer` to name backend-specific modes, cases, or flags used elsewhere in the file.
- CN: 这里定义枚举 `OperandTransfer`，用于命名本文件后续会使用的后端模式、分支情况或标志。

### Lines 74-77
```cpp
// Information about opcodes
struct OpCodes {
  OpCodes(unsigned WideOpc, unsigned NarrowOpc)
      : WideOpc(WideOpc), NarrowOpc(NarrowOpc) {}
```
- EN: Declares `OpCodes`, packaging target-specific state and APIs around `MicroMipsSizeReduction`.
- CN: 这里声明 `OpCodes`，把与 `MicroMipsSizeReduction` 相关的目标特定状态和 API 组织在一起。

### Lines 79-81
```cpp
  unsigned WideOpc;   ///< Wide opcode
  unsigned NarrowOpc; ///< Narrow opcode
};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 83-83
```cpp
typedef struct ReduceEntryFunArgs ReduceEntryFunArgs;
```
- EN: Declares `ReduceEntryFunArgs`, packaging target-specific state and APIs around `MicroMipsSizeReduction`.
- CN: 这里声明 `ReduceEntryFunArgs`，把与 `MicroMipsSizeReduction` 相关的目标特定状态和 API 组织在一起。

### Lines 85-87
```cpp
/// ReduceTable - A static table with information on mapping from wide
/// opcodes to narrow
struct ReduceEntry {
```
- EN: Declares `ReduceEntry`, packaging target-specific state and APIs around `MicroMipsSizeReduction`.
- CN: 这里声明 `ReduceEntry`，把与 `MicroMipsSizeReduction` 相关的目标特定状态和 API 组织在一起。

### Lines 89-94
```cpp
  enum ReduceType eRType; ///< Reduction type
  bool (*ReduceFunction)(
      ReduceEntryFunArgs *Arguments); ///< Pointer to reduce function
  struct OpCodes Ops;                 ///< All relevant OpCodes
  struct OpInfo OpInf;                ///< Characteristics of operands
  struct ImmField Imm;                ///< Characteristics of immediate field
```
- EN: Declares `OpCodes`, packaging target-specific state and APIs around `MicroMipsSizeReduction`.
- CN: 这里声明 `OpCodes`，把与 `MicroMipsSizeReduction` 相关的目标特定状态和 API 组织在一起。

### Lines 96-99
```cpp
  ReduceEntry(enum ReduceType RType, struct OpCodes Op,
              bool (*F)(ReduceEntryFunArgs *Arguments), struct OpInfo OpInf,
              struct ImmField Imm)
      : eRType(RType), ReduceFunction(F), Ops(Op), OpInf(OpInf), Imm(Imm) {}
```
- EN: Declares `OpCodes`, packaging target-specific state and APIs around `MicroMipsSizeReduction`.
- CN: 这里声明 `OpCodes`，把与 `MicroMipsSizeReduction` 相关的目标特定状态和 API 组织在一起。

### Lines 101-110
```cpp
  unsigned NarrowOpc() const { return Ops.NarrowOpc; }
  unsigned WideOpc() const { return Ops.WideOpc; }
  int16_t LBound() const { return Imm.LBound; }
  int16_t HBound() const { return Imm.HBound; }
  uint8_t Shift() const { return Imm.Shift; }
  int8_t ImmField() const { return Imm.ImmFieldOperand; }
  enum OperandTransfer TransferOperands() const {
    return OpInf.TransferOperands;
  }
  enum ReduceType RType() const { return eRType; }
```
- EN: Defines enumeration `OperandTransfer` to name backend-specific modes, cases, or flags used elsewhere in the file.
- CN: 这里定义枚举 `OperandTransfer`，用于命名本文件后续会使用的后端模式、分支情况或标志。

### Lines 112-113
```cpp
  // operator used by std::equal_range
  bool operator<(const unsigned int r) const { return (WideOpc() < r); }
```
- EN: Implements `WideOpc`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `WideOpc`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 115-119
```cpp
  // operator used by std::equal_range
  friend bool operator<(const unsigned int r, const struct ReduceEntry &re) {
    return (r < re.WideOpc());
  }
};
```
- EN: Declares `ReduceEntry`, packaging target-specific state and APIs around `MicroMipsSizeReduction`.
- CN: 这里声明 `ReduceEntry`，把与 `MicroMipsSizeReduction` 相关的目标特定状态和 API 组织在一起。

### Lines 121-126
```cpp
// Function arguments for ReduceFunction
struct ReduceEntryFunArgs {
  MachineInstr *MI;         // Instruction
  const ReduceEntry &Entry; // Entry field
  MachineBasicBlock::instr_iterator
      &NextMII; // Iterator to next instruction in block
```
- EN: Declares `ReduceEntryFunArgs`, packaging target-specific state and APIs around `MicroMipsSizeReduction`.
- CN: 这里声明 `ReduceEntryFunArgs`，把与 `MicroMipsSizeReduction` 相关的目标特定状态和 API 组织在一起。

### Lines 128-131
```cpp
  ReduceEntryFunArgs(MachineInstr *argMI, const ReduceEntry &argEntry,
                     MachineBasicBlock::instr_iterator &argNextMII)
      : MI(argMI), Entry(argEntry), NextMII(argNextMII) {}
};
```
- EN: Implements `ReduceEntryFunArgs`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ReduceEntryFunArgs`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 133-133
```cpp
typedef llvm::SmallVector<ReduceEntry, 32> ReduceEntryVector;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 135-138
```cpp
class MicroMipsSizeReduce : public MachineFunctionPass {
public:
  static char ID;
  MicroMipsSizeReduce();
```
- EN: Declares `MicroMipsSizeReduce`, packaging target-specific state and APIs around `MicroMipsSizeReduction`.
- CN: 这里声明 `MicroMipsSizeReduce`，把与 `MicroMipsSizeReduction` 相关的目标特定状态和 API 组织在一起。

### Lines 140-141
```cpp
  static const MipsInstrInfo *MipsII;
  const MipsSubtarget *Subtarget;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 143-143
```cpp
  bool runOnMachineFunction(MachineFunction &MF) override;
```
- EN: Declares `runOnMachineFunction`, a pass-entry routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `runOnMachineFunction`，它是一个围绕机器函数状态展开的Pass 入口例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 145-147
```cpp
  llvm::StringRef getPassName() const override {
    return "microMIPS instruction size reduction pass";
  }
```
- EN: Implements `getPassName`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getPassName`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 149-151
```cpp
private:
  /// Reduces width of instructions in the specified basic block.
  bool ReduceMBB(MachineBasicBlock &MBB);
```
- EN: Declares `ReduceMBB`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ReduceMBB`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 153-155
```cpp
  /// Attempts to reduce MI, returns true on success.
  bool ReduceMI(const MachineBasicBlock::instr_iterator &MII,
                MachineBasicBlock::instr_iterator &NextMII);
```
- EN: Declares `ReduceMI`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ReduceMI`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 157-159
```cpp
  // Attempts to reduce LW/SW instruction into LWSP/SWSP,
  // returns true on success.
  static bool ReduceXWtoXWSP(ReduceEntryFunArgs *Arguments);
```
- EN: Declares `ReduceXWtoXWSP`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ReduceXWtoXWSP`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 161-163
```cpp
  // Attempts to reduce two LW/SW instructions into LWP/SWP instruction,
  // returns true on success.
  static bool ReduceXWtoXWP(ReduceEntryFunArgs *Arguments);
```
- EN: Declares `ReduceXWtoXWP`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ReduceXWtoXWP`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 165-167
```cpp
  // Attempts to reduce LBU/LHU instruction into LBU16/LHU16,
  // returns true on success.
  static bool ReduceLXUtoLXU16(ReduceEntryFunArgs *Arguments);
```
- EN: Declares `ReduceLXUtoLXU16`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ReduceLXUtoLXU16`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 169-171
```cpp
  // Attempts to reduce SB/SH instruction into SB16/SH16,
  // returns true on success.
  static bool ReduceSXtoSX16(ReduceEntryFunArgs *Arguments);
```
- EN: Declares `ReduceSXtoSX16`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ReduceSXtoSX16`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 173-175
```cpp
  // Attempts to reduce two MOVE instructions into MOVEP instruction,
  // returns true on success.
  static bool ReduceMoveToMovep(ReduceEntryFunArgs *Arguments);
```
- EN: Declares `ReduceMoveToMovep`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ReduceMoveToMovep`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 177-178
```cpp
  // Attempts to reduce arithmetic instructions, returns true on success.
  static bool ReduceArithmeticInstructions(ReduceEntryFunArgs *Arguments);
```
- EN: Declares `ReduceArithmeticInstructions`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ReduceArithmeticInstructions`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 180-182
```cpp
  // Attempts to reduce ADDIU into ADDIUSP instruction,
  // returns true on success.
  static bool ReduceADDIUToADDIUSP(ReduceEntryFunArgs *Arguments);
```
- EN: Declares `ReduceADDIUToADDIUSP`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ReduceADDIUToADDIUSP`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 184-186
```cpp
  // Attempts to reduce ADDIU into ADDIUR1SP instruction,
  // returns true on success.
  static bool ReduceADDIUToADDIUR1SP(ReduceEntryFunArgs *Arguments);
```
- EN: Declares `ReduceADDIUToADDIUR1SP`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ReduceADDIUToADDIUR1SP`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 188-190
```cpp
  // Attempts to reduce XOR into XOR16 instruction,
  // returns true on success.
  static bool ReduceXORtoXOR16(ReduceEntryFunArgs *Arguments);
```
- EN: Declares `ReduceXORtoXOR16`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ReduceXORtoXOR16`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 192-198
```cpp
  // Changes opcode of an instruction, replaces an instruction with a
  // new one, or replaces two instructions with a new instruction
  // depending on their order i.e. if these are consecutive forward
  // or consecutive backward
  static bool ReplaceInstruction(MachineInstr *MI, const ReduceEntry &Entry,
                                 MachineInstr *MI2 = nullptr,
                                 bool ConsecutiveForward = true);
```
- EN: Declares `ReplaceInstruction`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ReplaceInstruction`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 200-202
```cpp
  // Table with transformation rules for each instruction.
  static ReduceEntryVector ReduceTable;
};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 204-205
```cpp
char MicroMipsSizeReduce::ID = 0;
const MipsInstrInfo *MicroMipsSizeReduce::MipsII;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 207-209
```cpp
// This table must be sorted by WideOpc as a main criterion and
// ReduceType as a sub-criterion (when wide opcodes are the same).
ReduceEntryVector MicroMipsSizeReduce::ReduceTable = {
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 211-225
```cpp
    // ReduceType, OpCodes, ReduceFunction,
    // OpInfo(TransferOperands),
    // ImmField(Shift, LBound, HBound, ImmFieldPosition)
    {RT_OneInstr, OpCodes(Mips::ADDiu, Mips::ADDIUR1SP_MM),
     ReduceADDIUToADDIUR1SP, OpInfo(OT_Operands02), ImmField(2, 0, 64, 2)},
    {RT_OneInstr, OpCodes(Mips::ADDiu, Mips::ADDIUSP_MM), ReduceADDIUToADDIUSP,
     OpInfo(OT_Operand2), ImmField(0, 0, 0, 2)},
    {RT_OneInstr, OpCodes(Mips::ADDiu_MM, Mips::ADDIUR1SP_MM),
     ReduceADDIUToADDIUR1SP, OpInfo(OT_Operands02), ImmField(2, 0, 64, 2)},
    {RT_OneInstr, OpCodes(Mips::ADDiu_MM, Mips::ADDIUSP_MM),
     ReduceADDIUToADDIUSP, OpInfo(OT_Operand2), ImmField(0, 0, 0, 2)},
    {RT_OneInstr, OpCodes(Mips::ADDu, Mips::ADDU16_MM),
     ReduceArithmeticInstructions, OpInfo(OT_OperandsAll),
     ImmField(0, 0, 0, -1)},
    {RT_OneInstr, OpCodes(Mips::ADDu_MM, Mips::ADDU16_MM),
```
- EN: Implements `OpCodes`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `OpCodes`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 226-240
```cpp
     ReduceArithmeticInstructions, OpInfo(OT_OperandsAll),
     ImmField(0, 0, 0, -1)},
    {RT_OneInstr, OpCodes(Mips::LBu, Mips::LBU16_MM), ReduceLXUtoLXU16,
     OpInfo(OT_OperandsAll), ImmField(0, -1, 15, 2)},
    {RT_OneInstr, OpCodes(Mips::LBu_MM, Mips::LBU16_MM), ReduceLXUtoLXU16,
     OpInfo(OT_OperandsAll), ImmField(0, -1, 15, 2)},
    {RT_OneInstr, OpCodes(Mips::LEA_ADDiu, Mips::ADDIUR1SP_MM),
     ReduceADDIUToADDIUR1SP, OpInfo(OT_Operands02), ImmField(2, 0, 64, 2)},
    {RT_OneInstr, OpCodes(Mips::LEA_ADDiu_MM, Mips::ADDIUR1SP_MM),
     ReduceADDIUToADDIUR1SP, OpInfo(OT_Operands02), ImmField(2, 0, 64, 2)},
    {RT_OneInstr, OpCodes(Mips::LHu, Mips::LHU16_MM), ReduceLXUtoLXU16,
     OpInfo(OT_OperandsAll), ImmField(1, 0, 16, 2)},
    {RT_OneInstr, OpCodes(Mips::LHu_MM, Mips::LHU16_MM), ReduceLXUtoLXU16,
     OpInfo(OT_OperandsAll), ImmField(1, 0, 16, 2)},
    {RT_TwoInstr, OpCodes(Mips::LW, Mips::LWP_MM), ReduceXWtoXWP,
```
- EN: Implements `OpInfo`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `OpInfo`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 241-255
```cpp
     OpInfo(OT_OperandsLwp), ImmField(0, -2048, 2048, 2)},
    {RT_OneInstr, OpCodes(Mips::LW, Mips::LWSP_MM), ReduceXWtoXWSP,
     OpInfo(OT_OperandsAll), ImmField(2, 0, 32, 2)},
    {RT_TwoInstr, OpCodes(Mips::LW16_MM, Mips::LWP_MM), ReduceXWtoXWP,
     OpInfo(OT_OperandsLwp), ImmField(0, -2048, 2048, 2)},
    {RT_TwoInstr, OpCodes(Mips::LW_MM, Mips::LWP_MM), ReduceXWtoXWP,
     OpInfo(OT_OperandsLwp), ImmField(0, -2048, 2048, 2)},
    {RT_OneInstr, OpCodes(Mips::LW_MM, Mips::LWSP_MM), ReduceXWtoXWSP,
     OpInfo(OT_OperandsAll), ImmField(2, 0, 32, 2)},
    {RT_TwoInstr, OpCodes(Mips::MOVE16_MM, Mips::MOVEP_MM), ReduceMoveToMovep,
     OpInfo(OT_OperandsMovep), ImmField(0, 0, 0, -1)},
    {RT_OneInstr, OpCodes(Mips::SB, Mips::SB16_MM), ReduceSXtoSX16,
     OpInfo(OT_OperandsAll), ImmField(0, 0, 16, 2)},
    {RT_OneInstr, OpCodes(Mips::SB_MM, Mips::SB16_MM), ReduceSXtoSX16,
     OpInfo(OT_OperandsAll), ImmField(0, 0, 16, 2)},
```
- EN: Implements `OpInfo`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `OpInfo`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 256-270
```cpp
    {RT_OneInstr, OpCodes(Mips::SH, Mips::SH16_MM), ReduceSXtoSX16,
     OpInfo(OT_OperandsAll), ImmField(1, 0, 16, 2)},
    {RT_OneInstr, OpCodes(Mips::SH_MM, Mips::SH16_MM), ReduceSXtoSX16,
     OpInfo(OT_OperandsAll), ImmField(1, 0, 16, 2)},
    {RT_OneInstr, OpCodes(Mips::SUBu, Mips::SUBU16_MM),
     ReduceArithmeticInstructions, OpInfo(OT_OperandsAll),
     ImmField(0, 0, 0, -1)},
    {RT_OneInstr, OpCodes(Mips::SUBu_MM, Mips::SUBU16_MM),
     ReduceArithmeticInstructions, OpInfo(OT_OperandsAll),
     ImmField(0, 0, 0, -1)},
    {RT_TwoInstr, OpCodes(Mips::SW, Mips::SWP_MM), ReduceXWtoXWP,
     OpInfo(OT_OperandsSwp), ImmField(0, -2048, 2048, 2)},
    {RT_OneInstr, OpCodes(Mips::SW, Mips::SWSP_MM), ReduceXWtoXWSP,
     OpInfo(OT_OperandsAll), ImmField(2, 0, 32, 2)},
    {RT_TwoInstr, OpCodes(Mips::SW16_MM, Mips::SWP_MM), ReduceXWtoXWP,
```
- EN: Implements `OpCodes`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `OpCodes`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 271-280
```cpp
     OpInfo(OT_OperandsSwp), ImmField(0, -2048, 2048, 2)},
    {RT_TwoInstr, OpCodes(Mips::SW_MM, Mips::SWP_MM), ReduceXWtoXWP,
     OpInfo(OT_OperandsSwp), ImmField(0, -2048, 2048, 2)},
    {RT_OneInstr, OpCodes(Mips::SW_MM, Mips::SWSP_MM), ReduceXWtoXWSP,
     OpInfo(OT_OperandsAll), ImmField(2, 0, 32, 2)},
    {RT_OneInstr, OpCodes(Mips::XOR, Mips::XOR16_MM), ReduceXORtoXOR16,
     OpInfo(OT_OperandsXOR), ImmField(0, 0, 0, -1)},
    {RT_OneInstr, OpCodes(Mips::XOR_MM, Mips::XOR16_MM), ReduceXORtoXOR16,
     OpInfo(OT_OperandsXOR), ImmField(0, 0, 0, -1)}};
} // end anonymous namespace
```
- EN: Implements `OpInfo`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `OpInfo`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 282-283
```cpp
INITIALIZE_PASS(MicroMipsSizeReduce, DEBUG_TYPE, MICROMIPS_SIZE_REDUCE_NAME,
                false, false)
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 285-290
```cpp
// Returns true if the machine operand MO is register SP.
static bool IsSP(const MachineOperand &MO) {
  if (MO.isReg() && ((MO.getReg() == Mips::SP)))
    return true;
  return false;
}
```
- EN: Implements `IsSP`, a query/helper routine centered on machine operands. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `IsSP`，它是一个围绕机器操作数展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 292-297
```cpp
// Returns true if the machine operand MO is register $16, $17, or $2-$7.
static bool isMMThreeBitGPRegister(const MachineOperand &MO) {
  if (MO.isReg() && Mips::GPRMM16RegClass.contains(MO.getReg()))
    return true;
  return false;
}
```
- EN: Implements `isMMThreeBitGPRegister`, a query/helper routine centered on machine operands. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isMMThreeBitGPRegister`，它是一个围绕机器操作数展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 299-304
```cpp
// Returns true if the machine operand MO is register $0, $17, or $2-$7.
static bool isMMSourceRegister(const MachineOperand &MO) {
  if (MO.isReg() && Mips::GPRMM16ZeroRegClass.contains(MO.getReg()))
    return true;
  return false;
}
```
- EN: Implements `isMMSourceRegister`, a query/helper routine centered on machine operands. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isMMSourceRegister`，它是一个围绕机器操作数展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 306-308
```cpp
// Returns true if the operand Op is an immediate value
// and writes the immediate value into variable Imm.
static bool GetImm(MachineInstr *MI, unsigned Op, int64_t &Imm) {
```
- EN: Implements `GetImm`, a query/helper routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `GetImm`，它是一个围绕机器指令展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 310-314
```cpp
  if (!MI->getOperand(Op).isImm())
    return false;
  Imm = MI->getOperand(Op).getImm();
  return true;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 316-323
```cpp
// Returns true if the value is a valid immediate for ADDIUSP.
static bool AddiuspImmValue(int64_t Value) {
  int64_t Value2 = Value >> 2;
  if (((Value & (int64_t)maskTrailingZeros<uint64_t>(2)) == Value) &&
      ((Value2 >= 2 && Value2 <= 257) || (Value2 >= -258 && Value2 <= -3)))
    return true;
  return false;
}
```
- EN: Implements `AddiuspImmValue`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `AddiuspImmValue`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 325-334
```cpp
// Returns true if the variable Value has the number of least-significant zero
// bits equal to Shift and if the shifted value is between the bounds.
static bool InRange(int64_t Value, unsigned short Shift, int LBound,
                    int HBound) {
  int64_t Value2 = Value >> Shift;
  if (((Value & (int64_t)maskTrailingZeros<uint64_t>(Shift)) == Value) &&
      (Value2 >= LBound) && (Value2 < HBound))
    return true;
  return false;
}
```
- EN: Implements `InRange`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `InRange`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 336-337
```cpp
// Returns true if immediate operand is in range.
static bool ImmInRange(MachineInstr *MI, const ReduceEntry &Entry) {
```
- EN: Implements `ImmInRange`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ImmInRange`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 339-339
```cpp
  int64_t offset;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 341-342
```cpp
  if (!GetImm(MI, Entry.ImmField(), offset))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 344-345
```cpp
  if (!InRange(offset, Entry.Shift(), Entry.LBound(), Entry.HBound()))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 347-348
```cpp
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 350-352
```cpp
// Returns true if MI can be reduced to lwp/swp instruction
static bool CheckXWPInstr(MachineInstr *MI, bool ReduceToLwp,
                          const ReduceEntry &Entry) {
```
- EN: Implements `CheckXWPInstr`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `CheckXWPInstr`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 354-357
```cpp
  if (ReduceToLwp &&
      !(MI->getOpcode() == Mips::LW || MI->getOpcode() == Mips::LW_MM ||
        MI->getOpcode() == Mips::LW16_MM))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 359-362
```cpp
  if (!ReduceToLwp &&
      !(MI->getOpcode() == Mips::SW || MI->getOpcode() == Mips::SW_MM ||
        MI->getOpcode() == Mips::SW16_MM))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 364-366
```cpp
  Register reg = MI->getOperand(0).getReg();
  if (reg == Mips::RA)
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 368-369
```cpp
  if (!ImmInRange(MI, Entry))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 371-372
```cpp
  if (ReduceToLwp && (MI->getOperand(0).getReg() == MI->getOperand(1).getReg()))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 374-375
```cpp
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 377-384
```cpp
// Returns true if the registers Reg1 and Reg2 are consecutive
static bool ConsecutiveRegisters(unsigned Reg1, unsigned Reg2) {
  constexpr std::array<unsigned, 31> Registers = {
      {Mips::AT, Mips::V0, Mips::V1, Mips::A0, Mips::A1, Mips::A2, Mips::A3,
       Mips::T0, Mips::T1, Mips::T2, Mips::T3, Mips::T4, Mips::T5, Mips::T6,
       Mips::T7, Mips::S0, Mips::S1, Mips::S2, Mips::S3, Mips::S4, Mips::S5,
       Mips::S6, Mips::S7, Mips::T8, Mips::T9, Mips::K0, Mips::K1, Mips::GP,
       Mips::SP, Mips::FP, Mips::RA}};
```
- EN: Implements `ConsecutiveRegisters`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ConsecutiveRegisters`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 386-395
```cpp
  for (uint8_t i = 0; i < Registers.size() - 1; i++) {
    if (Registers[i] == Reg1) {
      if (Registers[i + 1] == Reg2)
        return true;
      else
        return false;
    }
  }
  return false;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 397-398
```cpp
// Returns true if registers and offsets are consecutive
static bool ConsecutiveInstr(MachineInstr *MI1, MachineInstr *MI2) {
```
- EN: Implements `ConsecutiveInstr`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ConsecutiveInstr`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 400-404
```cpp
  int64_t Offset1, Offset2;
  if (!GetImm(MI1, 2, Offset1))
    return false;
  if (!GetImm(MI2, 2, Offset2))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 406-407
```cpp
  Register Reg1 = MI1->getOperand(0).getReg();
  Register Reg2 = MI2->getOperand(0).getReg();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 409-410
```cpp
  return ((Offset1 == (Offset2 - 4)) && (ConsecutiveRegisters(Reg1, Reg2)));
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 412-412
```cpp
MicroMipsSizeReduce::MicroMipsSizeReduce() : MachineFunctionPass(ID) {}
```
- EN: Implements `MicroMipsSizeReduce::MicroMipsSizeReduce`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MicroMipsSizeReduce::MicroMipsSizeReduce`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 414-415
```cpp
bool MicroMipsSizeReduce::ReduceMI(const MachineBasicBlock::instr_iterator &MII,
                                   MachineBasicBlock::instr_iterator &NextMII) {
```
- EN: Implements `MicroMipsSizeReduce::ReduceMI`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MicroMipsSizeReduce::ReduceMI`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 417-418
```cpp
  MachineInstr *MI = &*MII;
  unsigned Opcode = MI->getOpcode();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 420-422
```cpp
  // Search the table.
  ReduceEntryVector::const_iterator Start = std::begin(ReduceTable);
  ReduceEntryVector::const_iterator End = std::end(ReduceTable);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 424-426
```cpp
  std::pair<ReduceEntryVector::const_iterator,
            ReduceEntryVector::const_iterator>
      Range = std::equal_range(Start, End, Opcode);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 428-429
```cpp
  if (Range.first == Range.second)
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 431-438
```cpp
  for (ReduceEntryVector::const_iterator Entry = Range.first;
       Entry != Range.second; ++Entry) {
    ReduceEntryFunArgs Arguments(&(*MII), *Entry, NextMII);
    if (((*Entry).ReduceFunction)(&Arguments))
      return true;
  }
  return false;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 440-440
```cpp
bool MicroMipsSizeReduce::ReduceXWtoXWSP(ReduceEntryFunArgs *Arguments) {
```
- EN: Implements `MicroMipsSizeReduce::ReduceXWtoXWSP`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MicroMipsSizeReduce::ReduceXWtoXWSP`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 442-443
```cpp
  MachineInstr *MI = Arguments->MI;
  const ReduceEntry &Entry = Arguments->Entry;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 445-446
```cpp
  if (!ImmInRange(MI, Entry))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 448-449
```cpp
  if (!IsSP(MI->getOperand(1)))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 451-452
```cpp
  return ReplaceInstruction(MI, Entry);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 454-454
```cpp
bool MicroMipsSizeReduce::ReduceXWtoXWP(ReduceEntryFunArgs *Arguments) {
```
- EN: Implements `MicroMipsSizeReduce::ReduceXWtoXWP`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MicroMipsSizeReduce::ReduceXWtoXWP`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 456-459
```cpp
  const ReduceEntry &Entry = Arguments->Entry;
  MachineBasicBlock::instr_iterator &NextMII = Arguments->NextMII;
  const MachineBasicBlock::instr_iterator &E =
      Arguments->MI->getParent()->instr_end();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 461-462
```cpp
  if (NextMII == E)
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 464-465
```cpp
  MachineInstr *MI1 = Arguments->MI;
  MachineInstr *MI2 = &*NextMII;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 467-470
```cpp
  // ReduceToLwp = true/false - reduce to LWP/SWP instruction
  bool ReduceToLwp = (MI1->getOpcode() == Mips::LW) ||
                     (MI1->getOpcode() == Mips::LW_MM) ||
                     (MI1->getOpcode() == Mips::LW16_MM);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 472-473
```cpp
  if (!CheckXWPInstr(MI1, ReduceToLwp, Entry))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 475-476
```cpp
  if (!CheckXWPInstr(MI2, ReduceToLwp, Entry))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 478-479
```cpp
  Register Reg1 = MI1->getOperand(1).getReg();
  Register Reg2 = MI2->getOperand(1).getReg();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 481-482
```cpp
  if (Reg1 != Reg2)
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 484-485
```cpp
  bool ConsecutiveForward = ConsecutiveInstr(MI1, MI2);
  bool ConsecutiveBackward = ConsecutiveInstr(MI2, MI1);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 487-488
```cpp
  if (!(ConsecutiveForward || ConsecutiveBackward))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 490-492
```cpp
  NextMII = std::next(NextMII);
  return ReplaceInstruction(MI1, Entry, MI2, ConsecutiveForward);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 494-495
```cpp
bool MicroMipsSizeReduce::ReduceArithmeticInstructions(
    ReduceEntryFunArgs *Arguments) {
```
- EN: Implements `MicroMipsSizeReduce::ReduceArithmeticInstructions`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MicroMipsSizeReduce::ReduceArithmeticInstructions`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 497-498
```cpp
  MachineInstr *MI = Arguments->MI;
  const ReduceEntry &Entry = Arguments->Entry;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 500-503
```cpp
  if (!isMMThreeBitGPRegister(MI->getOperand(0)) ||
      !isMMThreeBitGPRegister(MI->getOperand(1)) ||
      !isMMThreeBitGPRegister(MI->getOperand(2)))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 505-506
```cpp
  return ReplaceInstruction(MI, Entry);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 508-509
```cpp
bool MicroMipsSizeReduce::ReduceADDIUToADDIUR1SP(
    ReduceEntryFunArgs *Arguments) {
```
- EN: Implements `MicroMipsSizeReduce::ReduceADDIUToADDIUR1SP`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MicroMipsSizeReduce::ReduceADDIUToADDIUR1SP`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 511-512
```cpp
  MachineInstr *MI = Arguments->MI;
  const ReduceEntry &Entry = Arguments->Entry;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 514-515
```cpp
  if (!ImmInRange(MI, Entry))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 517-518
```cpp
  if (!isMMThreeBitGPRegister(MI->getOperand(0)) || !IsSP(MI->getOperand(1)))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 520-521
```cpp
  return ReplaceInstruction(MI, Entry);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 523-523
```cpp
bool MicroMipsSizeReduce::ReduceADDIUToADDIUSP(ReduceEntryFunArgs *Arguments) {
```
- EN: Implements `MicroMipsSizeReduce::ReduceADDIUToADDIUSP`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MicroMipsSizeReduce::ReduceADDIUToADDIUSP`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 525-526
```cpp
  MachineInstr *MI = Arguments->MI;
  const ReduceEntry &Entry = Arguments->Entry;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 528-530
```cpp
  int64_t ImmValue;
  if (!GetImm(MI, Entry.ImmField(), ImmValue))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 532-533
```cpp
  if (!AddiuspImmValue(ImmValue))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 535-536
```cpp
  if (!IsSP(MI->getOperand(0)) || !IsSP(MI->getOperand(1)))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 538-539
```cpp
  return ReplaceInstruction(MI, Entry);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 541-541
```cpp
bool MicroMipsSizeReduce::ReduceLXUtoLXU16(ReduceEntryFunArgs *Arguments) {
```
- EN: Implements `MicroMipsSizeReduce::ReduceLXUtoLXU16`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MicroMipsSizeReduce::ReduceLXUtoLXU16`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 543-544
```cpp
  MachineInstr *MI = Arguments->MI;
  const ReduceEntry &Entry = Arguments->Entry;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 546-547
```cpp
  if (!ImmInRange(MI, Entry))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 549-551
```cpp
  if (!isMMThreeBitGPRegister(MI->getOperand(0)) ||
      !isMMThreeBitGPRegister(MI->getOperand(1)))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 553-554
```cpp
  return ReplaceInstruction(MI, Entry);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 556-556
```cpp
bool MicroMipsSizeReduce::ReduceSXtoSX16(ReduceEntryFunArgs *Arguments) {
```
- EN: Implements `MicroMipsSizeReduce::ReduceSXtoSX16`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MicroMipsSizeReduce::ReduceSXtoSX16`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 558-559
```cpp
  MachineInstr *MI = Arguments->MI;
  const ReduceEntry &Entry = Arguments->Entry;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 561-562
```cpp
  if (!ImmInRange(MI, Entry))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 564-566
```cpp
  if (!isMMSourceRegister(MI->getOperand(0)) ||
      !isMMThreeBitGPRegister(MI->getOperand(1)))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 568-569
```cpp
  return ReplaceInstruction(MI, Entry);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 571-573
```cpp
// Returns true if Reg can be a source register
// of MOVEP instruction
static bool IsMovepSrcRegister(unsigned Reg) {
```
- EN: Implements `IsMovepSrcRegister`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `IsMovepSrcRegister`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 575-578
```cpp
  if (Reg == Mips::ZERO || Reg == Mips::V0 || Reg == Mips::V1 ||
      Reg == Mips::S0 || Reg == Mips::S1 || Reg == Mips::S2 ||
      Reg == Mips::S3 || Reg == Mips::S4)
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 580-581
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 583-585
```cpp
// Returns true if Reg can be a destination register
// of MOVEP instruction
static bool IsMovepDestinationReg(unsigned Reg) {
```
- EN: Implements `IsMovepDestinationReg`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `IsMovepDestinationReg`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 587-589
```cpp
  if (Reg == Mips::A0 || Reg == Mips::A1 || Reg == Mips::A2 ||
      Reg == Mips::A3 || Reg == Mips::S5 || Reg == Mips::S6)
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 591-592
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 594-596
```cpp
// Returns true if the registers can be a pair of destination
// registers in MOVEP instruction
static bool IsMovepDestinationRegPair(unsigned R0, unsigned R1) {
```
- EN: Implements `IsMovepDestinationRegPair`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `IsMovepDestinationRegPair`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 598-606
```cpp
  if ((R0 == Mips::A0 && R1 == Mips::S5) ||
      (R0 == Mips::A0 && R1 == Mips::S6) ||
      (R0 == Mips::A0 && R1 == Mips::A1) ||
      (R0 == Mips::A0 && R1 == Mips::A2) ||
      (R0 == Mips::A0 && R1 == Mips::A3) ||
      (R0 == Mips::A1 && R1 == Mips::A2) ||
      (R0 == Mips::A1 && R1 == Mips::A3) ||
      (R0 == Mips::A2 && R1 == Mips::A3))
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 608-609
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 611-611
```cpp
bool MicroMipsSizeReduce::ReduceMoveToMovep(ReduceEntryFunArgs *Arguments) {
```
- EN: Implements `MicroMipsSizeReduce::ReduceMoveToMovep`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MicroMipsSizeReduce::ReduceMoveToMovep`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 613-616
```cpp
  const ReduceEntry &Entry = Arguments->Entry;
  MachineBasicBlock::instr_iterator &NextMII = Arguments->NextMII;
  const MachineBasicBlock::instr_iterator &E =
      Arguments->MI->getParent()->instr_end();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 618-619
```cpp
  if (NextMII == E)
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 621-622
```cpp
  MachineInstr *MI1 = Arguments->MI;
  MachineInstr *MI2 = &*NextMII;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 624-625
```cpp
  Register RegDstMI1 = MI1->getOperand(0).getReg();
  Register RegSrcMI1 = MI1->getOperand(1).getReg();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 627-628
```cpp
  if (!IsMovepSrcRegister(RegSrcMI1))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 630-631
```cpp
  if (!IsMovepDestinationReg(RegDstMI1))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 633-634
```cpp
  if (MI2->getOpcode() != Entry.WideOpc())
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 636-637
```cpp
  Register RegDstMI2 = MI2->getOperand(0).getReg();
  Register RegSrcMI2 = MI2->getOperand(1).getReg();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 639-640
```cpp
  if (!IsMovepSrcRegister(RegSrcMI2))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 642-648
```cpp
  bool ConsecutiveForward;
  if (IsMovepDestinationRegPair(RegDstMI1, RegDstMI2)) {
    ConsecutiveForward = true;
  } else if (IsMovepDestinationRegPair(RegDstMI2, RegDstMI1)) {
    ConsecutiveForward = false;
  } else
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 650-652
```cpp
  NextMII = std::next(NextMII);
  return ReplaceInstruction(MI1, Entry, MI2, ConsecutiveForward);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 654-654
```cpp
bool MicroMipsSizeReduce::ReduceXORtoXOR16(ReduceEntryFunArgs *Arguments) {
```
- EN: Implements `MicroMipsSizeReduce::ReduceXORtoXOR16`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MicroMipsSizeReduce::ReduceXORtoXOR16`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 656-657
```cpp
  MachineInstr *MI = Arguments->MI;
  const ReduceEntry &Entry = Arguments->Entry;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 659-662
```cpp
  if (!isMMThreeBitGPRegister(MI->getOperand(0)) ||
      !isMMThreeBitGPRegister(MI->getOperand(1)) ||
      !isMMThreeBitGPRegister(MI->getOperand(2)))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 664-666
```cpp
  if (!(MI->getOperand(0).getReg() == MI->getOperand(2).getReg()) &&
      !(MI->getOperand(0).getReg() == MI->getOperand(1).getReg()))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 668-669
```cpp
  return ReplaceInstruction(MI, Entry);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 671-675
```cpp
bool MicroMipsSizeReduce::ReduceMBB(MachineBasicBlock &MBB) {
  bool Modified = false;
  MachineBasicBlock::instr_iterator MII = MBB.instr_begin(),
                                    E = MBB.instr_end();
  MachineBasicBlock::instr_iterator NextMII;
```
- EN: Implements `MicroMipsSizeReduce::ReduceMBB`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MicroMipsSizeReduce::ReduceMBB`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 677-680
```cpp
  // Iterate through the instructions in the basic block
  for (; MII != E; MII = NextMII) {
    NextMII = std::next(MII);
    MachineInstr *MI = &*MII;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 682-684
```cpp
    // Don't reduce bundled instructions or pseudo operations
    if (MI->isBundle() || MI->isTransient())
      continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 686-688
```cpp
    // Try to reduce 32-bit instruction into 16-bit instruction
    Modified |= ReduceMI(MII, NextMII);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 690-691
```cpp
  return Modified;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 693-696
```cpp
bool MicroMipsSizeReduce::ReplaceInstruction(MachineInstr *MI,
                                             const ReduceEntry &Entry,
                                             MachineInstr *MI2,
                                             bool ConsecutiveForward) {
```
- EN: Implements `MicroMipsSizeReduce::ReplaceInstruction`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MicroMipsSizeReduce::ReplaceInstruction`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 698-698
```cpp
  enum OperandTransfer OpTransfer = Entry.TransferOperands();
```
- EN: Defines enumeration `OperandTransfer` to name backend-specific modes, cases, or flags used elsewhere in the file.
- CN: 这里定义枚举 `OperandTransfer`，用于命名本文件后续会使用的后端模式、分支情况或标志。

### Lines 700-701
```cpp
  LLVM_DEBUG(dbgs() << "Converting 32-bit: " << *MI);
  ++NumReduced;
```
- EN: Declares `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 703-717
```cpp
  if (OpTransfer == OT_OperandsAll) {
    MI->setDesc(MipsII->get(Entry.NarrowOpc()));
    LLVM_DEBUG(dbgs() << "       to 16-bit: " << *MI);
    return true;
  } else {
    MachineBasicBlock &MBB = *MI->getParent();
    const MCInstrDesc &NewMCID = MipsII->get(Entry.NarrowOpc());
    DebugLoc dl = MI->getDebugLoc();
    MachineInstrBuilder MIB = BuildMI(MBB, MI, dl, NewMCID);
    switch (OpTransfer) {
    case OT_Operand2:
      MIB.add(MI->getOperand(2));
      break;
    case OT_Operands02: {
      MIB.add(MI->getOperand(0));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 718-732
```cpp
      MIB.add(MI->getOperand(2));
      break;
    }
    case OT_OperandsXOR: {
      if (MI->getOperand(0).getReg() == MI->getOperand(2).getReg()) {
        MIB.add(MI->getOperand(0));
        MIB.add(MI->getOperand(1));
        MIB.add(MI->getOperand(2));
      } else {
        MIB.add(MI->getOperand(0));
        MIB.add(MI->getOperand(2));
        MIB.add(MI->getOperand(1));
      }
      break;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 733-747
```cpp
    case OT_OperandsMovep:
    case OT_OperandsLwp:
    case OT_OperandsSwp: {
      if (ConsecutiveForward) {
        MIB.add(MI->getOperand(0));
        MIB.add(MI2->getOperand(0));
        MIB.add(MI->getOperand(1));
        if (OpTransfer == OT_OperandsMovep)
          MIB.add(MI2->getOperand(1));
        else
          MIB.add(MI->getOperand(2));
      } else { // consecutive backward
        MIB.add(MI2->getOperand(0));
        MIB.add(MI->getOperand(0));
        MIB.add(MI2->getOperand(1));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 748-752
```cpp
        if (OpTransfer == OT_OperandsMovep)
          MIB.add(MI->getOperand(1));
        else
          MIB.add(MI2->getOperand(2));
      }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 754-755
```cpp
      LLVM_DEBUG(dbgs() << "and converting 32-bit: " << *MI2
                        << "       to: " << *MIB);
```
- EN: Declares `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 757-763
```cpp
      MBB.erase_instr(MI);
      MBB.erase_instr(MI2);
      return true;
    }
    default:
      llvm_unreachable("Unknown operand transfer!");
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 765-766
```cpp
    // Transfer MI flags.
    MIB.setMIFlags(MI->getFlags());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 768-773
```cpp
    LLVM_DEBUG(dbgs() << "       to 16-bit: " << *MIB);
    MBB.erase_instr(MI);
    return true;
  }
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 775-775
```cpp
bool MicroMipsSizeReduce::runOnMachineFunction(MachineFunction &MF) {
```
- EN: Implements `MicroMipsSizeReduce::runOnMachineFunction`, a pass-entry routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MicroMipsSizeReduce::runOnMachineFunction`，它是一个围绕机器函数状态展开的Pass 入口例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 777-777
```cpp
  Subtarget = &MF.getSubtarget<MipsSubtarget>();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 779-782
```cpp
  // TODO: Add support for the subtarget microMIPS32R6.
  if (!Subtarget->inMicroMipsMode() || !Subtarget->hasMips32r2() ||
      Subtarget->hasMips32r6())
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 784-784
```cpp
  MipsII = Subtarget->getInstrInfo();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 786-787
```cpp
  bool Modified = false;
  MachineFunction::iterator I = MF.begin(), E = MF.end();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 789-792
```cpp
  for (; I != E; ++I)
    Modified |= ReduceMBB(*I);
  return Modified;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 794-797
```cpp
/// Returns an instance of the MicroMips size reduction pass.
FunctionPass *llvm::createMicroMipsSizeReducePass() {
  return new MicroMipsSizeReduce();
}
```
- EN: Implements `llvm::createMicroMipsSizeReducePass`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `llvm::createMicroMipsSizeReducePass`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

## Key Concepts / 关键概念

- EN: Primary role: target-specific code-generation behavior.
  - CN: 核心职责：目标相关代码生成行为。
- EN: MachineInstr-level manipulation is central to this file.
  - CN: 该文件以 MachineInstr 层面的操作为核心。
- EN: MachineBasicBlock flow and branch structure matter to the implementation.
  - CN: 实现高度依赖 MachineBasicBlock 流程与分支结构。
- EN: The MC layer is used to print, encode, or stream target instructions and symbols.
  - CN: MC 层被用于打印、编码或流式输出目标指令与符号。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。

## Dependencies / 依赖关系

- EN: Backend-local headers: `Mips.h`, `MipsInstrInfo.h`, `MipsSubtarget.h`.
  - CN: 后端本地头文件：`Mips.h`, `MipsInstrInfo.h`, `MipsSubtarget.h`。
- EN: LLVM infrastructure headers: `llvm/ADT/Statistic.h`, `llvm/CodeGen/MachineFunctionPass.h`, `llvm/Support/Debug.h`.
  - CN: LLVM 基础设施头文件：`llvm/ADT/Statistic.h`, `llvm/CodeGen/MachineFunctionPass.h`, `llvm/Support/Debug.h`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
- EN: The implementation also depends on LLVM MC layer classes for final assembly/object emission.
  - CN: 实现还依赖 LLVM 的 MC 层类来完成最终汇编/目标文件输出。
