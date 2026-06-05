# MipsSubtarget.h — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MipsSubtarget.h`
- Repository: `llvm-project`
- Purpose (EN): This file declares the Mips specific subclass of TargetSubtargetInfo.
- 用途 (CN): 声明 Mips 后端中的 `MipsSubtarget`，并提供与子目标特性选择与 CPU 特定行为相关的接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===-- MipsSubtarget.h - Define Subtarget for the Mips ---------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file declares the Mips specific subclass of TargetSubtargetInfo.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 13-14
```cpp
#ifndef LLVM_LIB_TARGET_MIPS_MIPSSUBTARGET_H
#define LLVM_LIB_TARGET_MIPS_MIPSSUBTARGET_H
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 16-26
```cpp
#include "MCTargetDesc/MipsABIInfo.h"
#include "MipsFrameLowering.h"
#include "MipsISelLowering.h"
#include "MipsInstrInfo.h"
#include "llvm/CodeGen/GlobalISel/CallLowering.h"
#include "llvm/CodeGen/GlobalISel/InstructionSelector.h"
#include "llvm/CodeGen/GlobalISel/LegalizerInfo.h"
#include "llvm/CodeGen/RegisterBankInfo.h"
#include "llvm/CodeGen/TargetSubtargetInfo.h"
#include "llvm/IR/DataLayout.h"
#include "llvm/Support/ErrorHandling.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 28-29
```cpp
#define GET_SUBTARGETINFO_HEADER
#include "MipsGenSubtargetInfo.inc"
```
- EN: Connects this file to TableGen-generated declarations so target-specific enums and helper tables become available to C++ code.
- CN: 这里把文件连接到 TableGen 生成的声明，使目标相关枚举和辅助表能够在 C++ 代码中使用。

### Lines 31-32
```cpp
namespace llvm {
class StringRef;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 34-42
```cpp
enum CompactBranchPolicy {
  CB_Never,   ///< The policy 'never' may in some circumstances or for some
              ///< ISAs not be absolutely adhered to.
  CB_Optimal, ///< Optimal is the default and will produce compact branches
              ///< when appropriate.
  CB_Always   ///< 'always' may in some circumstances may not be
              ///< absolutely adhered to, there may not be a corresponding
              ///< compact form of a branch.
};
```
- EN: Defines enumeration `CompactBranchPolicy` to name backend-specific modes, cases, or flags used elsewhere in the file.
- CN: 这里定义枚举 `CompactBranchPolicy`，用于命名本文件后续会使用的后端模式、分支情况或标志。

### Lines 44-44
```cpp
class MipsTargetMachine;
```
- EN: Declares `MipsTargetMachine`, packaging target-specific state and APIs around `MipsSubtarget`.
- CN: 这里声明 `MipsTargetMachine`，把与 `MipsSubtarget` 相关的目标特定状态和 API 组织在一起。

### Lines 46-47
```cpp
class MipsSubtarget : public MipsGenSubtargetInfo {
  virtual void anchor();
```
- EN: Declares `MipsSubtarget`, packaging target-specific state and APIs around `MipsSubtarget`.
- CN: 这里声明 `MipsSubtarget`，把与 `MipsSubtarget` 相关的目标特定状态和 API 组织在一起。

### Lines 49-53
```cpp
  enum MipsArchEnum {
    MipsDefault,
    Mips1, Mips2, Mips32, Mips32r2, Mips32r3, Mips32r5, Mips32r6, Mips32Max,
    Mips3, Mips4, Mips5, Mips64, Mips64r2, Mips64r3, Mips64r5, Mips64r6
  };
```
- EN: Defines enumeration `MipsArchEnum` to name backend-specific modes, cases, or flags used elsewhere in the file.
- CN: 这里定义枚举 `MipsArchEnum`，用于命名本文件后续会使用的后端模式、分支情况或标志。

### Lines 55-55
```cpp
  enum class CPU { Others, P5600, I6400, I6500 };
```
- EN: Declares `CPU`, packaging target-specific state and APIs around `MipsSubtarget`.
- CN: 这里声明 `CPU`，把与 `MipsSubtarget` 相关的目标特定状态和 API 组织在一起。

### Lines 57-58
```cpp
  // Used to avoid printing dsp warnings multiple times.
  static bool DspWarningPrinted;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 60-61
```cpp
  // Used to avoid printing msa warnings multiple times.
  static bool MSAWarningPrinted;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 63-64
```cpp
  // Used to avoid printing crc warnings multiple times.
  static bool CRCWarningPrinted;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 66-67
```cpp
  // Used to avoid printing ginv warnings multiple times.
  static bool GINVWarningPrinted;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 69-70
```cpp
  // Used to avoid printing Mips1 warnings multiple times.
  static bool MIPS1WarningPrinted;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 72-73
```cpp
  // Used to avoid printing virt warnings multiple times.
  static bool VirtWarningPrinted;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 75-76
```cpp
  // Mips architecture version
  MipsArchEnum MipsArchVersion;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 78-79
```cpp
  // Processor implementation
  CPU ProcImpl = CPU::Others;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 81-82
```cpp
  // IsLittle - The target is Little Endian
  bool IsLittle;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 84-85
```cpp
  // IsSoftFloat - The target does not support any floating point instructions.
  bool IsSoftFloat;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 87-90
```cpp
  // IsSingleFloat - The target only supports single precision float
  // point operations. This enable the target to use all 32 32-bit
  // floating point registers instead of only using even ones.
  bool IsSingleFloat;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 92-93
```cpp
  // IsFPXX - MIPS O32 modeless ABI.
  bool IsFPXX;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 95-96
```cpp
  // NoABICalls - Disable SVR4-style position-independent code.
  bool NoABICalls;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 98-99
```cpp
  // Abs2008 - Use IEEE 754-2008 abs.fmt instruction.
  bool Abs2008;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 101-102
```cpp
  // IsFP64bit - The target processor has 64-bit floating point registers.
  bool IsFP64bit;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 104-106
```cpp
  /// Are odd single-precision registers permitted?
  /// This corresponds to -modd-spreg and -mno-odd-spreg
  bool UseOddSPReg;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 108-109
```cpp
  // IsNan2008 - IEEE 754-2008 NaN encoding.
  bool IsNaN2008bit;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 111-112
```cpp
  // IsGP64bit - General-purpose registers are 64 bits wide
  bool IsGP64bit;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 114-115
```cpp
  // IsPTR64bit - Pointers are 64 bit wide
  bool IsPTR64bit;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 117-118
```cpp
  // HasVFPU - Processor has a vector floating point unit.
  bool HasVFPU;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 120-121
```cpp
  // CPU supports cnMIPS (Cavium Networks Octeon CPU).
  bool HasCnMips;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 123-124
```cpp
  // CPU supports cnMIPSP (Cavium Networks Octeon+ CPU).
  bool HasCnMipsP;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 126-127
```cpp
  // IsR5900 - CPU is R5900 (PlayStation 2 Emotion Engine).
  bool IsR5900;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 129-130
```cpp
  // FixR5900 - Enable R5900 short loop erratum fix.
  bool FixR5900;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 132-133
```cpp
  // isLinux - Target system is Linux. Is false we consider ELFOS for now.
  bool IsLinux;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 135-136
```cpp
  // UseSmallSection - Small section is used.
  bool UseSmallSection;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 138-138
```cpp
  /// Features related to the presence of specific instructions.
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 140-141
```cpp
  // HasMips3_32 - The subset of MIPS-III instructions added to MIPS32
  bool HasMips3_32;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 143-144
```cpp
  // HasMips3_32r2 - The subset of MIPS-III instructions added to MIPS32r2
  bool HasMips3_32r2;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 146-147
```cpp
  // HasMips4_32 - Has the subset of MIPS-IV present in MIPS32
  bool HasMips4_32;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 149-150
```cpp
  // HasMips4_32r2 - Has the subset of MIPS-IV present in MIPS32r2
  bool HasMips4_32r2;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 152-153
```cpp
  // HasMips5_32r2 - Has the subset of MIPS-V present in MIPS32r2
  bool HasMips5_32r2;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 155-156
```cpp
  // InMips16 -- can process Mips16 instructions
  bool InMips16Mode;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 158-159
```cpp
  // Mips16 hard float
  bool InMips16HardFloat;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 161-162
```cpp
  // InMicroMips -- can process MicroMips instructions
  bool InMicroMipsMode;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 164-165
```cpp
  // HasDSP, HasDSPR2, HasDSPR3 -- supports DSP ASE.
  bool HasDSP, HasDSPR2, HasDSPR3;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 167-168
```cpp
  // Has3D -- Supports Mips3D ASE.
  bool Has3D;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 170-171
```cpp
  // Allow mixed Mips16 and Mips32 in one source file
  bool AllowMixed16_32;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 173-176
```cpp
  // Optimize for space by compiling all functions as Mips 16 unless
  // it needs floating point. Functions needing floating point are
  // compiled as Mips32
  bool Os16;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 178-179
```cpp
  // HasMSA -- supports MSA ASE.
  bool HasMSA;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 181-182
```cpp
  // UseTCCInDIV -- Enables the use of trapping in the assembler.
  bool UseTCCInDIV;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 184-185
```cpp
  // Sym32 -- On Mips64 symbols are 32 bits.
  bool HasSym32;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 187-188
```cpp
  // HasEVA -- supports EVA ASE.
  bool HasEVA;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 190-192
```cpp
  // nomadd4 - disables generation of 4-operand madd.s, madd.d and
  // related instructions.
  bool DisableMadd4;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 194-195
```cpp
  // HasMT -- support MT ASE.
  bool HasMT;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 197-198
```cpp
  // HasCRC -- supports R6 CRC ASE
  bool HasCRC;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 200-201
```cpp
  // HasVirt -- supports Virtualization ASE
  bool HasVirt;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 203-204
```cpp
  // HasGINV -- supports R6 Global INValidate ASE
  bool HasGINV;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 206-208
```cpp
  // Use hazard variants of the jump register instructions for indirect
  // function calls and jump tables.
  bool UseIndirectJumpsHazard;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 210-211
```cpp
  // Disable use of the `jal` instruction.
  bool UseLongCalls = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 213-214
```cpp
  // Assume 32-bit GOT.
  bool UseXGOT = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 216-217
```cpp
  // Disable unaligned load store for r6.
  bool StrictAlign;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 219-220
```cpp
  // Use compact branch instructions for R6.
  bool UseCompactBranches = true;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 222-224
```cpp
  /// The minimum alignment known to hold of the stack frame on
  /// entry to the function and which must be maintained by every function.
  Align stackAlignment;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 226-227
```cpp
  /// The overridden stack alignment.
  MaybeAlign StackAlignOverride;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 229-231
```cpp
  // We can override the determination of whether we are in mips16 mode
  // as from the command line
  enum {NoOverride, Mips16Override, NoMips16Override} OverrideMode;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 233-233
```cpp
  const MipsTargetMachine &TM;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 235-235
```cpp
  Triple TargetTriple;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 237-240
```cpp
  std::unique_ptr<const SelectionDAGTargetInfo> TSInfo;
  std::unique_ptr<const MipsInstrInfo> InstrInfo;
  std::unique_ptr<const MipsFrameLowering> FrameLowering;
  std::unique_ptr<const MipsTargetLowering> TLInfo;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 242-247
```cpp
public:
  bool isPositionIndependent() const;
  /// This overrides the PostRAScheduler bit in the SchedModel for each CPU.
  bool enablePostRAScheduler() const override;
  void getCriticalPathRCs(RegClassVector &CriticalPathRCs) const override;
  CodeGenOptLevel getOptLevelToEnablePostRAScheduler() const override;
```
- EN: Declares `isPositionIndependent`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isPositionIndependent`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 249-253
```cpp
  bool isABI_N64() const;
  bool isABI_N32() const;
  bool isABI_O32() const;
  const MipsABIInfo &getABI() const;
  bool isABI_FPXX() const { return isABI_O32() && IsFPXX; }
```
- EN: Declares `isABI_N64`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isABI_N64`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 255-258
```cpp
  /// This constructor initializes the data members to match that
  /// of the specified triple.
  MipsSubtarget(const Triple &TT, StringRef CPU, StringRef FS, bool little,
                const MipsTargetMachine &TM, MaybeAlign StackAlignOverride);
```
- EN: Declares `MipsSubtarget`, a target-specific routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `MipsSubtarget`，它是一个围绕目标机器策略展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 260-260
```cpp
  ~MipsSubtarget() override;
```
- EN: Declares `~MipsSubtarget`, a target-specific routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `~MipsSubtarget`，它是一个围绕子目标特性处理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 262-264
```cpp
  /// ParseSubtargetFeatures - Parses features string setting specified
  /// subtarget options.  Definition of function is auto generated by tblgen.
  void ParseSubtargetFeatures(StringRef CPU, StringRef TuneCPU, StringRef FS);
```
- EN: Declares `ParseSubtargetFeatures`, a target-specific routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ParseSubtargetFeatures`，它是一个围绕子目标特性处理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 266-266
```cpp
  static const RTLIB::LibcallImpl HardFloatLibCalls[34];
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 268-282
```cpp
  bool hasMips1() const { return MipsArchVersion >= Mips1; }
  bool hasMips2() const { return MipsArchVersion >= Mips2; }
  bool hasMips3() const { return MipsArchVersion >= Mips3; }
  bool hasMips4() const { return MipsArchVersion >= Mips4; }
  bool hasMips5() const { return MipsArchVersion >= Mips5; }
  bool hasMips4_32() const { return HasMips4_32; }
  bool hasMips4_32r2() const { return HasMips4_32r2; }
  bool hasMips32() const {
    return (MipsArchVersion >= Mips32 && MipsArchVersion < Mips32Max) ||
           hasMips64();
  }
  bool hasMips32r2() const {
    return (MipsArchVersion >= Mips32r2 && MipsArchVersion < Mips32Max) ||
           hasMips64r2();
  }
```
- EN: Implements `hasMips1`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasMips1`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 283-297
```cpp
  bool hasMips32r3() const {
    return (MipsArchVersion >= Mips32r3 && MipsArchVersion < Mips32Max) ||
           hasMips64r2();
  }
  bool hasMips32r5() const {
    return (MipsArchVersion >= Mips32r5 && MipsArchVersion < Mips32Max) ||
           hasMips64r5();
  }
  bool hasMips32r6() const {
    return (MipsArchVersion >= Mips32r6 && MipsArchVersion < Mips32Max) ||
           hasMips64r6();
  }
  bool hasMips64() const { return MipsArchVersion >= Mips64; }
  bool hasMips64r2() const { return MipsArchVersion >= Mips64r2; }
  bool hasMips64r3() const { return MipsArchVersion >= Mips64r3; }
```
- EN: Implements `hasMips32r3`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasMips32r3`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 298-299
```cpp
  bool hasMips64r5() const { return MipsArchVersion >= Mips64r5; }
  bool hasMips64r6() const { return MipsArchVersion >= Mips64r6; }
```
- EN: Implements `hasMips64r5`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasMips64r5`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 301-304
```cpp
  bool hasCnMips() const { return HasCnMips; }
  bool hasCnMipsP() const { return HasCnMipsP; }
  bool isR5900() const { return IsR5900; }
  bool fixR5900() const { return FixR5900; }
```
- EN: Implements `hasCnMips`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasCnMips`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 306-320
```cpp
  bool isLittle() const { return IsLittle; }
  bool isABICalls() const { return !NoABICalls; }
  bool isFPXX() const { return IsFPXX; }
  bool isFP64bit() const { return IsFP64bit; }
  bool useOddSPReg() const { return UseOddSPReg; }
  bool noOddSPReg() const { return !UseOddSPReg; }
  bool isNaN2008() const { return IsNaN2008bit; }
  bool inAbs2008Mode() const { return Abs2008; }
  bool isGP64bit() const { return IsGP64bit; }
  bool isGP32bit() const { return !IsGP64bit; }
  unsigned getGPRSizeInBytes() const { return isGP64bit() ? 8 : 4; }
  bool isPTR64bit() const { return IsPTR64bit; }
  bool isPTR32bit() const { return !IsPTR64bit; }
  bool hasSym32() const {
    return (HasSym32 && isABI_N64()) || isABI_N32() || isABI_O32();
```
- EN: Implements `isLittle`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isLittle`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 321-335
```cpp
  }
  bool isSingleFloat() const { return IsSingleFloat; }
  bool isTargetCOFF() const { return TargetTriple.isOSBinFormatCOFF(); }
  bool isTargetELF() const { return TargetTriple.isOSBinFormatELF(); }
  bool hasVFPU() const { return HasVFPU; }
  bool inMips16Mode() const { return InMips16Mode; }
  bool inMips16ModeDefault() const {
    return InMips16Mode;
  }
  // Hard float for mips16 means essentially to compile as soft float
  // but to use a runtime library for soft float that is written with
  // native mips32 floating point instructions (those runtime routines
  // run in mips32 hard float mode).
  bool inMips16HardFloat() const {
    return inMips16Mode() && InMips16HardFloat;
```
- EN: Implements `isSingleFloat`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isSingleFloat`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 336-350
```cpp
  }
  bool inMicroMipsMode() const { return InMicroMipsMode && !InMips16Mode; }
  bool inMicroMips32r6Mode() const {
    return inMicroMipsMode() && hasMips32r6();
  }
  bool hasDSP() const { return HasDSP; }
  bool hasDSPR2() const { return HasDSPR2; }
  bool hasDSPR3() const { return HasDSPR3; }
  bool has3D() const { return Has3D; }
  bool hasMSA() const { return HasMSA; }
  bool disableMadd4() const { return DisableMadd4; }
  bool hasEVA() const { return HasEVA; }
  bool hasMT() const { return HasMT; }
  bool hasCRC() const { return HasCRC; }
  bool hasVirt() const { return HasVirt; }
```
- EN: Implements `inMicroMipsMode`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `inMicroMipsMode`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 351-355
```cpp
  bool hasGINV() const { return HasGINV; }
  bool useIndirectJumpsHazard() const {
    return UseIndirectJumpsHazard && hasMips32r2();
  }
  bool useSmallSection() const { return UseSmallSection; }
```
- EN: Implements `hasGINV`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasGINV`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 357-359
```cpp
  bool useCompactBranches() const {
    return UseCompactBranches && hasMips32r6();
  }
```
- EN: Implements `useCompactBranches`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `useCompactBranches`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 361-361
```cpp
  bool hasStandardEncoding() const { return !InMips16Mode && !InMicroMipsMode; }
```
- EN: Implements `hasStandardEncoding`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasStandardEncoding`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 363-363
```cpp
  bool useSoftFloat() const { return IsSoftFloat; }
```
- EN: Implements `useSoftFloat`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `useSoftFloat`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 365-365
```cpp
  bool useLongCalls() const { return UseLongCalls; }
```
- EN: Implements `useLongCalls`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `useLongCalls`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 367-367
```cpp
  bool useXGOT() const { return UseXGOT; }
```
- EN: Implements `useXGOT`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `useXGOT`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 369-371
```cpp
  bool enableLongBranchPass() const {
    return hasStandardEncoding() || inMicroMipsMode() || allowMixed16_32();
  }
```
- EN: Implements `enableLongBranchPass`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `enableLongBranchPass`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 373-375
```cpp
  /// Features related to the presence of specific instructions.
  bool hasExtractInsert() const { return !inMips16Mode() && hasMips32r2(); }
  bool hasMTHC1() const { return hasMips32r2(); }
```
- EN: Implements `hasExtractInsert`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasExtractInsert`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 377-378
```cpp
  bool allowMixed16_32() const { return inMips16ModeDefault() |
                                        AllowMixed16_32; }
```
- EN: Implements `allowMixed16_32`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `allowMixed16_32`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 380-380
```cpp
  bool os16() const { return Os16; }
```
- EN: Implements `os16`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `os16`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 382-382
```cpp
  bool isTargetWindows() const { return TargetTriple.isOSWindows(); }
```
- EN: Implements `isTargetWindows`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isTargetWindows`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 384-384
```cpp
  bool isXRaySupported() const override { return true; }
```
- EN: Implements `isXRaySupported`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isXRaySupported`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 386-388
```cpp
  // for now constant islands are on for the whole compilation unit but we only
  // really use them if in addition we are in mips16 mode
  static bool useConstantIslands();
```
- EN: Declares `useConstantIslands`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `useConstantIslands`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 390-390
```cpp
  Align getStackAlignment() const { return stackAlignment; }
```
- EN: Implements `getStackAlignment`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getStackAlignment`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 392-393
```cpp
  // Grab relocation model
  Reloc::Model getRelocationModel() const;
```
- EN: Declares `getRelocationModel`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getRelocationModel`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 395-396
```cpp
  MipsSubtarget &initializeSubtargetDependencies(StringRef CPU, StringRef FS,
                                                 const TargetMachine &TM);
```
- EN: Declares `initializeSubtargetDependencies`, a target-specific routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `initializeSubtargetDependencies`，它是一个围绕目标机器策略展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 398-405
```cpp
  /// Does the system support unaligned memory access.
  ///
  /// MIPS32r6/MIPS64r6 require full unaligned access support but does not
  /// specify which component of the system provides it. Hardware, software, and
  /// hybrid implementations are all valid.
  bool systemSupportsUnalignedAccess() const {
    return hasMips32r6() && !StrictAlign;
  }
```
- EN: Implements `systemSupportsUnalignedAccess`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `systemSupportsUnalignedAccess`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 407-409
```cpp
  // Set helper classes
  void setHelperClassesMips16();
  void setHelperClassesMipsSE();
```
- EN: Declares `setHelperClassesMips16`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setHelperClassesMips16`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 411-411
```cpp
  const SelectionDAGTargetInfo *getSelectionDAGInfo() const override;
```
- EN: Declares `getSelectionDAGInfo`, a query/helper routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getSelectionDAGInfo`，它是一个围绕SelectionDAG 降级展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 413-423
```cpp
  const MipsInstrInfo *getInstrInfo() const override { return InstrInfo.get(); }
  const TargetFrameLowering *getFrameLowering() const override {
    return FrameLowering.get();
  }
  const MipsRegisterInfo *getRegisterInfo() const override {
    return &InstrInfo->getRegisterInfo();
  }
  const MipsTargetLowering *getTargetLowering() const override {
    return TLInfo.get();
  }
  void initLibcallLoweringInfo(LibcallLoweringInfo &Info) const override;
```
- EN: Implements `getInstrInfo`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getInstrInfo`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 425-430
```cpp
protected:
  // GlobalISel related APIs.
  std::unique_ptr<CallLowering> CallLoweringInfo;
  std::unique_ptr<LegalizerInfo> Legalizer;
  std::unique_ptr<RegisterBankInfo> RegBankInfo;
  std::unique_ptr<InstructionSelector> InstSelector;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 432-438
```cpp
public:
  const CallLowering *getCallLowering() const override;
  const LegalizerInfo *getLegalizerInfo() const override;
  const RegisterBankInfo *getRegBankInfo() const override;
  InstructionSelector *getInstructionSelector() const override;
};
} // End llvm namespace
```
- EN: Declares `getCallLowering`, a lowering routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getCallLowering`，它是一个围绕寄存器管理展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 440-440
```cpp
#endif
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

## Key Concepts / 关键概念

- EN: Primary role: subtarget feature selection and CPU-specific behavior.
  - CN: 核心职责：子目标特性选择与 CPU 特定行为。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。
- EN: Stack-frame and spill/reload concerns are reflected in the API or implementation.
  - CN: API 或实现中体现了栈帧以及溢出/重载相关问题。

## Dependencies / 依赖关系

- EN: Backend-local headers: `MCTargetDesc/MipsABIInfo.h`, `MipsFrameLowering.h`, `MipsISelLowering.h`, `MipsInstrInfo.h`, `MipsGenSubtargetInfo.inc`.
  - CN: 后端本地头文件：`MCTargetDesc/MipsABIInfo.h`, `MipsFrameLowering.h`, `MipsISelLowering.h`, `MipsInstrInfo.h`, `MipsGenSubtargetInfo.inc`。
- EN: LLVM infrastructure headers: `llvm/CodeGen/GlobalISel/CallLowering.h`, `llvm/CodeGen/GlobalISel/InstructionSelector.h`, `llvm/CodeGen/GlobalISel/LegalizerInfo.h`, `llvm/CodeGen/RegisterBankInfo.h`, `llvm/CodeGen/TargetSubtargetInfo.h`, `llvm/IR/DataLayout.h`, `llvm/Support/ErrorHandling.h`.
  - CN: LLVM 基础设施头文件：`llvm/CodeGen/GlobalISel/CallLowering.h`, `llvm/CodeGen/GlobalISel/InstructionSelector.h`, `llvm/CodeGen/GlobalISel/LegalizerInfo.h`, `llvm/CodeGen/RegisterBankInfo.h`, `llvm/CodeGen/TargetSubtargetInfo.h`, `llvm/IR/DataLayout.h`, `llvm/Support/ErrorHandling.h`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
