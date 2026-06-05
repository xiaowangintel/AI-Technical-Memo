# ARMMCCodeEmitter.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/MCTargetDesc/ARMMCCodeEmitter.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file implements the ARMMCCodeEmitter class.
- 用途 (CN): 实现 ARM 后端中的 `ARMMCCodeEmitter`，重点处理机器码编码输出。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===-- ARM/ARMMCCodeEmitter.cpp - Convert ARM code to machine code -------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the ARMMCCodeEmitter class.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 13-27
```cpp
#include "MCTargetDesc/ARMAddressingModes.h"
#include "MCTargetDesc/ARMBaseInfo.h"
#include "MCTargetDesc/ARMFixupKinds.h"
#include "MCTargetDesc/ARMMCAsmInfo.h"
#include "llvm/ADT/APFloat.h"
#include "llvm/ADT/APInt.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/MC/MCCodeEmitter.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCFixup.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCInstrDesc.h"
#include "llvm/MC/MCInstrInfo.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 28-37
```cpp
#include "llvm/MC/MCRegisterInfo.h"
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/Support/Casting.h"
#include "llvm/Support/Compiler.h"
#include "llvm/Support/EndianStream.h"
#include "llvm/Support/MathExtras.h"
#include "llvm/TargetParser/Triple.h"
#include <cassert>
#include <cstdint>
#include <cstdlib>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 39-39
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 41-41
```cpp
#define DEBUG_TYPE "mccodeemitter"
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 43-44
```cpp
STATISTIC(MCNumEmitted, "Number of MC instructions emitted.");
STATISTIC(MCNumCPRelocations, "Number of constant pool relocations created.");
```
- EN: Declares `STATISTIC`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `STATISTIC`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 46-46
```cpp
namespace {
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 48-51
```cpp
class ARMMCCodeEmitter : public MCCodeEmitter {
  const MCInstrInfo &MCII;
  MCContext &CTX;
  bool IsLittleEndian;
```
- EN: Declares `ARMMCCodeEmitter`, packaging target-specific state and APIs around `ARMMCCodeEmitter`.
- CN: 这里声明 `ARMMCCodeEmitter`，把与 `ARMMCCodeEmitter` 相关的目标特定状态和 API 组织在一起。

### Lines 53-59
```cpp
public:
  ARMMCCodeEmitter(const MCInstrInfo &mcii, MCContext &ctx, bool IsLittle)
    : MCII(mcii), CTX(ctx), IsLittleEndian(IsLittle) {
  }
  ARMMCCodeEmitter(const ARMMCCodeEmitter &) = delete;
  ARMMCCodeEmitter &operator=(const ARMMCCodeEmitter &) = delete;
  ~ARMMCCodeEmitter() override = default;
```
- EN: Implements `ARMMCCodeEmitter`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMMCCodeEmitter`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 61-63
```cpp
  bool isThumb(const MCSubtargetInfo &STI) const {
    return STI.hasFeature(ARM::ModeThumb);
  }
```
- EN: Implements `isThumb`, a query/helper routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isThumb`，它是一个围绕子目标特性处理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 65-67
```cpp
  bool isThumb2(const MCSubtargetInfo &STI) const {
    return isThumb(STI) && STI.hasFeature(ARM::FeatureThumb2);
  }
```
- EN: Implements `isThumb2`, a query/helper routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isThumb2`，它是一个围绕子目标特性处理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 69-72
```cpp
  bool isTargetMachO(const MCSubtargetInfo &STI) const {
    const Triple &TT = STI.getTargetTriple();
    return TT.isOSBinFormatMachO();
  }
```
- EN: Implements `isTargetMachO`, a query/helper routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isTargetMachO`，它是一个围绕子目标特性处理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 74-74
```cpp
  unsigned getMachineSoImmOpValue(unsigned SoImm) const;
```
- EN: Declares `getMachineSoImmOpValue`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getMachineSoImmOpValue`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 76-80
```cpp
  // getBinaryCodeForInstr - TableGen'erated function for getting the
  // binary encoding for an instruction.
  uint64_t getBinaryCodeForInstr(const MCInst &MI,
                                 SmallVectorImpl<MCFixup> &Fixups,
                                 const MCSubtargetInfo &STI) const;
```
- EN: Declares `getBinaryCodeForInstr`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getBinaryCodeForInstr`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 82-86
```cpp
  /// getMachineOpValue - Return binary encoding of operand. If the machine
  /// operand requires relocation, record the relocation and return zero.
  unsigned getMachineOpValue(const MCInst &MI,const MCOperand &MO,
                             SmallVectorImpl<MCFixup> &Fixups,
                             const MCSubtargetInfo &STI) const;
```
- EN: Declares `getMachineOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getMachineOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 88-94
```cpp
  /// getHiLoImmOpValue - Return the encoding for either the hi / low 16-bit, or
  /// high/middle-high/middle-low/low 8 bits of the specified operand. This is
  /// used for operands with :lower16:, :upper16: :lower0_7:, :lower8_15:,
  /// :higher0_7:, and :higher8_15: prefixes.
  uint32_t getHiLoImmOpValue(const MCInst &MI, unsigned OpIdx,
                             SmallVectorImpl<MCFixup> &Fixups,
                             const MCSubtargetInfo &STI) const;
```
- EN: Declares `getHiLoImmOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getHiLoImmOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 96-99
```cpp
  bool EncodeAddrModeOpValues(const MCInst &MI, unsigned OpIdx,
                              unsigned &Reg, unsigned &Imm,
                              SmallVectorImpl<MCFixup> &Fixups,
                              const MCSubtargetInfo &STI) const;
```
- EN: Declares `EncodeAddrModeOpValues`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `EncodeAddrModeOpValues`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 101-105
```cpp
  /// getThumbBLTargetOpValue - Return encoding info for Thumb immediate
  /// BL branch target.
  uint32_t getThumbBLTargetOpValue(const MCInst &MI, unsigned OpIdx,
                                   SmallVectorImpl<MCFixup> &Fixups,
                                   const MCSubtargetInfo &STI) const;
```
- EN: Declares `getThumbBLTargetOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getThumbBLTargetOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 107-111
```cpp
  /// getThumbBLXTargetOpValue - Return encoding info for Thumb immediate
  /// BLX branch target.
  uint32_t getThumbBLXTargetOpValue(const MCInst &MI, unsigned OpIdx,
                                    SmallVectorImpl<MCFixup> &Fixups,
                                    const MCSubtargetInfo &STI) const;
```
- EN: Declares `getThumbBLXTargetOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getThumbBLXTargetOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 113-116
```cpp
  /// getThumbBRTargetOpValue - Return encoding info for Thumb branch target.
  uint32_t getThumbBRTargetOpValue(const MCInst &MI, unsigned OpIdx,
                                   SmallVectorImpl<MCFixup> &Fixups,
                                   const MCSubtargetInfo &STI) const;
```
- EN: Declares `getThumbBRTargetOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getThumbBRTargetOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 118-121
```cpp
  /// getThumbBCCTargetOpValue - Return encoding info for Thumb branch target.
  uint32_t getThumbBCCTargetOpValue(const MCInst &MI, unsigned OpIdx,
                                    SmallVectorImpl<MCFixup> &Fixups,
                                    const MCSubtargetInfo &STI) const;
```
- EN: Declares `getThumbBCCTargetOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getThumbBCCTargetOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 123-126
```cpp
  /// getThumbCBTargetOpValue - Return encoding info for Thumb branch target.
  uint32_t getThumbCBTargetOpValue(const MCInst &MI, unsigned OpIdx,
                                   SmallVectorImpl<MCFixup> &Fixups,
                                   const MCSubtargetInfo &STI) const;
```
- EN: Declares `getThumbCBTargetOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getThumbCBTargetOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 128-132
```cpp
  /// getBranchTargetOpValue - Return encoding info for 24-bit immediate
  /// branch target.
  uint32_t getBranchTargetOpValue(const MCInst &MI, unsigned OpIdx,
                                  SmallVectorImpl<MCFixup> &Fixups,
                                  const MCSubtargetInfo &STI) const;
```
- EN: Declares `getBranchTargetOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getBranchTargetOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 134-138
```cpp
  /// getThumbBranchTargetOpValue - Return encoding info for 24-bit
  /// immediate Thumb2 direct branch target.
  uint32_t getThumbBranchTargetOpValue(const MCInst &MI, unsigned OpIdx,
                                       SmallVectorImpl<MCFixup> &Fixups,
                                       const MCSubtargetInfo &STI) const;
```
- EN: Declares `getThumbBranchTargetOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getThumbBranchTargetOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 140-150
```cpp
  /// getARMBranchTargetOpValue - Return encoding info for 24-bit immediate
  /// branch target.
  uint32_t getARMBranchTargetOpValue(const MCInst &MI, unsigned OpIdx,
                                     SmallVectorImpl<MCFixup> &Fixups,
                                     const MCSubtargetInfo &STI) const;
  uint32_t getARMBLTargetOpValue(const MCInst &MI, unsigned OpIdx,
                                 SmallVectorImpl<MCFixup> &Fixups,
                                 const MCSubtargetInfo &STI) const;
  uint32_t getARMBLXTargetOpValue(const MCInst &MI, unsigned OpIdx,
                                  SmallVectorImpl<MCFixup> &Fixups,
                                  const MCSubtargetInfo &STI) const;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 152-162
```cpp
  /// getAdrLabelOpValue - Return encoding info for 12-bit immediate
  /// ADR label target.
  uint32_t getAdrLabelOpValue(const MCInst &MI, unsigned OpIdx,
                              SmallVectorImpl<MCFixup> &Fixups,
                              const MCSubtargetInfo &STI) const;
  uint32_t getThumbAdrLabelOpValue(const MCInst &MI, unsigned OpIdx,
                              SmallVectorImpl<MCFixup> &Fixups,
                              const MCSubtargetInfo &STI) const;
  uint32_t getT2AdrLabelOpValue(const MCInst &MI, unsigned OpIdx,
                              SmallVectorImpl<MCFixup> &Fixups,
                              const MCSubtargetInfo &STI) const;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 164-166
```cpp
  uint32_t getITMaskOpValue(const MCInst &MI, unsigned OpIdx,
                            SmallVectorImpl<MCFixup> &Fixups,
                            const MCSubtargetInfo &STI) const;
```
- EN: Declares `getITMaskOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getITMaskOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 168-172
```cpp
  /// getMVEShiftImmOpValue - Return encoding info for the 'sz:imm5'
  /// operand.
  uint32_t getMVEShiftImmOpValue(const MCInst &MI, unsigned OpIdx,
                                 SmallVectorImpl<MCFixup> &Fixups,
                                 const MCSubtargetInfo &STI) const;
```
- EN: Declares `getMVEShiftImmOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getMVEShiftImmOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 174-178
```cpp
  /// getAddrModeImm12OpValue - Return encoding info for 'reg +/- imm12'
  /// operand.
  uint32_t getAddrModeImm12OpValue(const MCInst &MI, unsigned OpIdx,
                                   SmallVectorImpl<MCFixup> &Fixups,
                                   const MCSubtargetInfo &STI) const;
```
- EN: Declares `getAddrModeImm12OpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getAddrModeImm12OpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 180-183
```cpp
  /// getThumbAddrModeRegRegOpValue - Return encoding for 'reg + reg' operand.
  uint32_t getThumbAddrModeRegRegOpValue(const MCInst &MI, unsigned OpIdx,
                                         SmallVectorImpl<MCFixup> &Fixups,
                                         const MCSubtargetInfo &STI) const;
```
- EN: Declares `getThumbAddrModeRegRegOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getThumbAddrModeRegRegOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 185-189
```cpp
  /// getT2AddrModeImm8s4OpValue - Return encoding info for 'reg +/- imm8<<2'
  /// operand.
  uint32_t getT2AddrModeImm8s4OpValue(const MCInst &MI, unsigned OpIdx,
                                   SmallVectorImpl<MCFixup> &Fixups,
                                   const MCSubtargetInfo &STI) const;
```
- EN: Declares `getT2AddrModeImm8s4OpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getT2AddrModeImm8s4OpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 191-195
```cpp
  /// getT2AddrModeImm7s4OpValue - Return encoding info for 'reg +/- imm7<<2'
  /// operand.
  uint32_t getT2AddrModeImm7s4OpValue(const MCInst &MI, unsigned OpIdx,
                                      SmallVectorImpl<MCFixup> &Fixups,
                                      const MCSubtargetInfo &STI) const;
```
- EN: Declares `getT2AddrModeImm7s4OpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getT2AddrModeImm7s4OpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 197-201
```cpp
  /// getT2AddrModeImm0_1020s4OpValue - Return encoding info for 'reg + imm8<<2'
  /// operand.
  uint32_t getT2AddrModeImm0_1020s4OpValue(const MCInst &MI, unsigned OpIdx,
                                   SmallVectorImpl<MCFixup> &Fixups,
                                   const MCSubtargetInfo &STI) const;
```
- EN: Declares `getT2AddrModeImm0_1020s4OpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getT2AddrModeImm0_1020s4OpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 203-208
```cpp
  /// getT2ScaledImmOpValue - Return encoding info for '+/- immX<<Y'
  /// operand.
  template<unsigned Bits, unsigned Shift>
  uint32_t getT2ScaledImmOpValue(const MCInst &MI, unsigned OpIdx,
                                 SmallVectorImpl<MCFixup> &Fixups,
                                 const MCSubtargetInfo &STI) const;
```
- EN: Declares `getT2ScaledImmOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getT2ScaledImmOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 210-214
```cpp
  /// getMveAddrModeRQOpValue - Return encoding info for 'reg, vreg'
  /// operand.
  uint32_t getMveAddrModeRQOpValue(const MCInst &MI, unsigned OpIdx,
                                   SmallVectorImpl<MCFixup> &Fixups,
                                   const MCSubtargetInfo &STI) const;
```
- EN: Declares `getMveAddrModeRQOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getMveAddrModeRQOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 216-221
```cpp
  /// getMveAddrModeQOpValue - Return encoding info for 'reg +/- imm7<<{shift}'
  /// operand.
  template<int shift>
  uint32_t getMveAddrModeQOpValue(const MCInst &MI, unsigned OpIdx,
                                  SmallVectorImpl<MCFixup> &Fixups,
                                  const MCSubtargetInfo &STI) const;
```
- EN: Declares `getMveAddrModeQOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getMveAddrModeQOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 223-227
```cpp
  /// getLdStSORegOpValue - Return encoding info for 'reg +/- reg shop imm'
  /// operand as needed by load/store instructions.
  uint32_t getLdStSORegOpValue(const MCInst &MI, unsigned OpIdx,
                               SmallVectorImpl<MCFixup> &Fixups,
                               const MCSubtargetInfo &STI) const;
```
- EN: Declares `getLdStSORegOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getLdStSORegOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 229-241
```cpp
  /// getLdStmModeOpValue - Return encoding for load/store multiple mode.
  uint32_t getLdStmModeOpValue(const MCInst &MI, unsigned OpIdx,
                               SmallVectorImpl<MCFixup> &Fixups,
                               const MCSubtargetInfo &STI) const {
    ARM_AM::AMSubMode Mode = (ARM_AM::AMSubMode)MI.getOperand(OpIdx).getImm();
    switch (Mode) {
    default: llvm_unreachable("Unknown addressing sub-mode!");
    case ARM_AM::da: return 0;
    case ARM_AM::ia: return 1;
    case ARM_AM::db: return 2;
    case ARM_AM::ib: return 3;
    }
  }
```
- EN: Implements `getLdStmModeOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getLdStmModeOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 243-256
```cpp
  /// getShiftOp - Return the shift opcode (bit[6:5]) of the immediate value.
  ///
  unsigned getShiftOp(ARM_AM::ShiftOpc ShOpc) const {
    switch (ShOpc) {
    case ARM_AM::no_shift:
    case ARM_AM::lsl: return 0;
    case ARM_AM::lsr: return 1;
    case ARM_AM::asr: return 2;
    case ARM_AM::ror:
    case ARM_AM::rrx: return 3;
    default:
      llvm_unreachable("Invalid ShiftOpc!");
    }
  }
```
- EN: Implements `getShiftOp`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getShiftOp`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 258-261
```cpp
  /// getAddrMode2OffsetOpValue - Return encoding for am2offset operands.
  uint32_t getAddrMode2OffsetOpValue(const MCInst &MI, unsigned OpIdx,
                                     SmallVectorImpl<MCFixup> &Fixups,
                                     const MCSubtargetInfo &STI) const;
```
- EN: Declares `getAddrMode2OffsetOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getAddrMode2OffsetOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 263-266
```cpp
  /// getPostIdxRegOpValue - Return encoding for postidx_reg operands.
  uint32_t getPostIdxRegOpValue(const MCInst &MI, unsigned OpIdx,
                                SmallVectorImpl<MCFixup> &Fixups,
                                const MCSubtargetInfo &STI) const;
```
- EN: Declares `getPostIdxRegOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getPostIdxRegOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 268-271
```cpp
  /// getAddrMode3OffsetOpValue - Return encoding for am3offset operands.
  uint32_t getAddrMode3OffsetOpValue(const MCInst &MI, unsigned OpIdx,
                                     SmallVectorImpl<MCFixup> &Fixups,
                                     const MCSubtargetInfo &STI) const;
```
- EN: Declares `getAddrMode3OffsetOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getAddrMode3OffsetOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 273-276
```cpp
  /// getAddrMode3OpValue - Return encoding for addrmode3 operands.
  uint32_t getAddrMode3OpValue(const MCInst &MI, unsigned OpIdx,
                               SmallVectorImpl<MCFixup> &Fixups,
                               const MCSubtargetInfo &STI) const;
```
- EN: Declares `getAddrMode3OpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getAddrMode3OpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 278-282
```cpp
  /// getAddrModeThumbSPOpValue - Return encoding info for 'reg +/- imm12'
  /// operand.
  uint32_t getAddrModeThumbSPOpValue(const MCInst &MI, unsigned OpIdx,
                                     SmallVectorImpl<MCFixup> &Fixups,
                                     const MCSubtargetInfo &STI) const;
```
- EN: Declares `getAddrModeThumbSPOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getAddrModeThumbSPOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 284-287
```cpp
  /// getAddrModeISOpValue - Encode the t_addrmode_is# operands.
  uint32_t getAddrModeISOpValue(const MCInst &MI, unsigned OpIdx,
                                SmallVectorImpl<MCFixup> &Fixups,
                                const MCSubtargetInfo &STI) const;
```
- EN: Declares `getAddrModeISOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getAddrModeISOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 289-292
```cpp
  /// getAddrModePCOpValue - Return encoding for t_addrmode_pc operands.
  uint32_t getAddrModePCOpValue(const MCInst &MI, unsigned OpIdx,
                                SmallVectorImpl<MCFixup> &Fixups,
                                const MCSubtargetInfo &STI) const;
```
- EN: Declares `getAddrModePCOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getAddrModePCOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 294-297
```cpp
  /// getAddrMode5OpValue - Return encoding info for 'reg +/- (imm8 << 2)' operand.
  uint32_t getAddrMode5OpValue(const MCInst &MI, unsigned OpIdx,
                               SmallVectorImpl<MCFixup> &Fixups,
                               const MCSubtargetInfo &STI) const;
```
- EN: Declares `getAddrMode5OpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getAddrMode5OpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 299-302
```cpp
  /// getAddrMode5FP16OpValue - Return encoding info for 'reg +/- (imm8 << 1)' operand.
  uint32_t getAddrMode5FP16OpValue(const MCInst &MI, unsigned OpIdx,
                               SmallVectorImpl<MCFixup> &Fixups,
                               const MCSubtargetInfo &STI) const;
```
- EN: Declares `getAddrMode5FP16OpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getAddrMode5FP16OpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 304-311
```cpp
  /// getCCOutOpValue - Return encoding of the 's' bit.
  unsigned getCCOutOpValue(const MCInst &MI, unsigned Op,
                           SmallVectorImpl<MCFixup> &Fixups,
                           const MCSubtargetInfo &STI) const {
    // The operand is either reg0 or CPSR. The 's' bit is encoded as '0' or
    // '1' respectively.
    return MI.getOperand(Op).getReg() == ARM::CPSR;
  }
```
- EN: Implements `getCCOutOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getCCOutOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 313-315
```cpp
  unsigned getModImmOpValue(const MCInst &MI, unsigned Op,
                            SmallVectorImpl<MCFixup> &Fixups,
                            const MCSubtargetInfo &ST) const;
```
- EN: Declares `getModImmOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getModImmOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 317-320
```cpp
  /// getT2SOImmOpValue - Return an encoded 12-bit shifted-immediate value.
  unsigned getT2SOImmOpValue(const MCInst &MI, unsigned Op,
                             SmallVectorImpl<MCFixup> &Fixups,
                             const MCSubtargetInfo &STI) const;
```
- EN: Declares `getT2SOImmOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getT2SOImmOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 322-331
```cpp
  unsigned getT2AddrModeSORegOpValue(const MCInst &MI, unsigned OpNum,
    SmallVectorImpl<MCFixup> &Fixups,
    const MCSubtargetInfo &STI) const;
  template<unsigned Bits, unsigned Shift>
  unsigned getT2AddrModeImmOpValue(const MCInst &MI, unsigned OpNum,
    SmallVectorImpl<MCFixup> &Fixups,
    const MCSubtargetInfo &STI) const;
  unsigned getT2AddrModeImm8OffsetOpValue(const MCInst &MI, unsigned OpNum,
    SmallVectorImpl<MCFixup> &Fixups,
    const MCSubtargetInfo &STI) const;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 333-342
```cpp
  /// getSORegOpValue - Return an encoded so_reg shifted register value.
  unsigned getSORegRegOpValue(const MCInst &MI, unsigned Op,
                           SmallVectorImpl<MCFixup> &Fixups,
                           const MCSubtargetInfo &STI) const;
  unsigned getSORegImmOpValue(const MCInst &MI, unsigned Op,
                           SmallVectorImpl<MCFixup> &Fixups,
                           const MCSubtargetInfo &STI) const;
  unsigned getT2SORegOpValue(const MCInst &MI, unsigned Op,
                             SmallVectorImpl<MCFixup> &Fixups,
                             const MCSubtargetInfo &STI) const;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 344-348
```cpp
  unsigned getNEONVcvtImm32OpValue(const MCInst &MI, unsigned Op,
                                   SmallVectorImpl<MCFixup> &Fixups,
                                   const MCSubtargetInfo &STI) const {
    return 64 - MI.getOperand(Op).getImm();
  }
```
- EN: Implements `getNEONVcvtImm32OpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getNEONVcvtImm32OpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 350-352
```cpp
  unsigned getBitfieldInvertedMaskOpValue(const MCInst &MI, unsigned Op,
                                      SmallVectorImpl<MCFixup> &Fixups,
                                      const MCSubtargetInfo &STI) const;
```
- EN: Declares `getBitfieldInvertedMaskOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getBitfieldInvertedMaskOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 354-368
```cpp
  unsigned getRegisterListOpValue(const MCInst &MI, unsigned Op,
                                  SmallVectorImpl<MCFixup> &Fixups,
                                  const MCSubtargetInfo &STI) const;
  unsigned getAddrMode6AddressOpValue(const MCInst &MI, unsigned Op,
                                      SmallVectorImpl<MCFixup> &Fixups,
                                      const MCSubtargetInfo &STI) const;
  unsigned getAddrMode6OneLane32AddressOpValue(const MCInst &MI, unsigned Op,
                                        SmallVectorImpl<MCFixup> &Fixups,
                                        const MCSubtargetInfo &STI) const;
  unsigned getAddrMode6DupAddressOpValue(const MCInst &MI, unsigned Op,
                                        SmallVectorImpl<MCFixup> &Fixups,
                                        const MCSubtargetInfo &STI) const;
  unsigned getAddrMode6OffsetOpValue(const MCInst &MI, unsigned Op,
                                     SmallVectorImpl<MCFixup> &Fixups,
                                     const MCSubtargetInfo &STI) const;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 370-381
```cpp
  unsigned getShiftRight8Imm(const MCInst &MI, unsigned Op,
                             SmallVectorImpl<MCFixup> &Fixups,
                             const MCSubtargetInfo &STI) const;
  unsigned getShiftRight16Imm(const MCInst &MI, unsigned Op,
                              SmallVectorImpl<MCFixup> &Fixups,
                              const MCSubtargetInfo &STI) const;
  unsigned getShiftRight32Imm(const MCInst &MI, unsigned Op,
                              SmallVectorImpl<MCFixup> &Fixups,
                              const MCSubtargetInfo &STI) const;
  unsigned getShiftRight64Imm(const MCInst &MI, unsigned Op,
                              SmallVectorImpl<MCFixup> &Fixups,
                              const MCSubtargetInfo &STI) const;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 383-385
```cpp
  unsigned getThumbSRImmOpValue(const MCInst &MI, unsigned Op,
                                 SmallVectorImpl<MCFixup> &Fixups,
                                 const MCSubtargetInfo &STI) const;
```
- EN: Declares `getThumbSRImmOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getThumbSRImmOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 387-398
```cpp
  unsigned NEONThumb2DataIPostEncoder(const MCInst &MI,
                                      unsigned EncodedValue,
                                      const MCSubtargetInfo &STI) const;
  unsigned NEONThumb2LoadStorePostEncoder(const MCInst &MI,
                                          unsigned EncodedValue,
                                          const MCSubtargetInfo &STI) const;
  unsigned NEONThumb2DupPostEncoder(const MCInst &MI,
                                    unsigned EncodedValue,
                                    const MCSubtargetInfo &STI) const;
  unsigned NEONThumb2V8PostEncoder(const MCInst &MI,
                                   unsigned EncodedValue,
                                   const MCSubtargetInfo &STI) const;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 400-402
```cpp
  unsigned VFPThumb2PostEncoder(const MCInst &MI,
                                unsigned EncodedValue,
                                const MCSubtargetInfo &STI) const;
```
- EN: Declares `VFPThumb2PostEncoder`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `VFPThumb2PostEncoder`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 404-406
```cpp
  uint32_t getPowerTwoOpValue(const MCInst &MI, unsigned OpIdx,
                              SmallVectorImpl<MCFixup> &Fixups,
                              const MCSubtargetInfo &STI) const;
```
- EN: Declares `getPowerTwoOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getPowerTwoOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 408-410
```cpp
  void encodeInstruction(const MCInst &MI, SmallVectorImpl<char> &CB,
                         SmallVectorImpl<MCFixup> &Fixups,
                         const MCSubtargetInfo &STI) const override;
```
- EN: Declares `encodeInstruction`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `encodeInstruction`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 412-415
```cpp
  template <bool isNeg, ARM::Fixups fixup>
  uint32_t getBFTargetOpValue(const MCInst &MI, unsigned OpIdx,
                              SmallVectorImpl<MCFixup> &Fixups,
                              const MCSubtargetInfo &STI) const;
```
- EN: Declares `getBFTargetOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getBFTargetOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 417-419
```cpp
  uint32_t getBFAfterTargetOpValue(const MCInst &MI, unsigned OpIdx,
                                   SmallVectorImpl<MCFixup> &Fixups,
                                   const MCSubtargetInfo &STI) const;
```
- EN: Declares `getBFAfterTargetOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getBFAfterTargetOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 421-431
```cpp
  uint32_t getVPTMaskOpValue(const MCInst &MI, unsigned OpIdx,
                             SmallVectorImpl<MCFixup> &Fixups,
                             const MCSubtargetInfo &STI) const;
  uint32_t getRestrictedCondCodeOpValue(const MCInst &MI, unsigned OpIdx,
                                        SmallVectorImpl<MCFixup> &Fixups,
                                        const MCSubtargetInfo &STI) const;
  template <unsigned size>
  uint32_t getMVEPairVectorIndexOpValue(const MCInst &MI, unsigned OpIdx,
                                        SmallVectorImpl<MCFixup> &Fixups,
                                        const MCSubtargetInfo &STI) const;
};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 433-433
```cpp
} // end anonymous namespace
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 435-449
```cpp
static void addFixup(SmallVectorImpl<MCFixup> &Fixups, uint32_t Offset,
                     const MCExpr *Value, uint16_t Kind) {
  bool PCRel = false;
  switch (Kind) {
  case ARM::fixup_arm_ldst_pcrel_12:
  case ARM::fixup_t2_ldst_pcrel_12:
  case ARM::fixup_arm_pcrel_10_unscaled:
  case ARM::fixup_arm_pcrel_10:
  case ARM::fixup_t2_pcrel_10:
  case ARM::fixup_arm_pcrel_9:
  case ARM::fixup_t2_pcrel_9:
  case ARM::fixup_thumb_adr_pcrel_10:
  case ARM::fixup_arm_adr_pcrel_12:
  case ARM::fixup_t2_adr_pcrel_12:
  case ARM::fixup_arm_condbranch:
```
- EN: Implements `addFixup`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `addFixup`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 450-464
```cpp
  case ARM::fixup_arm_uncondbranch:
  case ARM::fixup_t2_condbranch:
  case ARM::fixup_t2_uncondbranch:
  case ARM::fixup_arm_thumb_br:
  case ARM::fixup_arm_uncondbl:
  case ARM::fixup_arm_condbl:
  case ARM::fixup_arm_blx:
  case ARM::fixup_arm_thumb_bl:
  case ARM::fixup_arm_thumb_blx:
  case ARM::fixup_arm_thumb_cb:
  case ARM::fixup_arm_thumb_cp:
  case ARM::fixup_arm_thumb_bcc:
  case ARM::fixup_bf_branch:
  case ARM::fixup_bf_target:
  case ARM::fixup_bfl_target:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 465-471
```cpp
  case ARM::fixup_bfc_target:
  case ARM::fixup_wls:
  case ARM::fixup_le:
    PCRel = true;
  }
  Fixups.push_back(MCFixup::create(Offset, Value, Kind, PCRel));
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 473-487
```cpp
/// NEONThumb2DataIPostEncoder - Post-process encoded NEON data-processing
/// instructions, and rewrite them to their Thumb2 form if we are currently in
/// Thumb2 mode.
unsigned ARMMCCodeEmitter::NEONThumb2DataIPostEncoder(const MCInst &MI,
                                                 unsigned EncodedValue,
                                                 const MCSubtargetInfo &STI) const {
  if (isThumb2(STI)) {
    // NEON Thumb2 data-processing encodings are very simple: bit 24 is moved
    // to bit 12 of the high half-word (i.e. bit 28), and bits 27-24 are
    // set to 1111.
    unsigned Bit24 = EncodedValue & 0x01000000;
    unsigned Bit28 = Bit24 << 4;
    EncodedValue &= 0xEFFFFFFF;
    EncodedValue |= Bit28;
    EncodedValue |= 0x0F000000;
```
- EN: Implements `ARMMCCodeEmitter::NEONThumb2DataIPostEncoder`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMMCCodeEmitter::NEONThumb2DataIPostEncoder`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 488-488
```cpp
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 490-491
```cpp
  return EncodedValue;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 493-502
```cpp
/// NEONThumb2LoadStorePostEncoder - Post-process encoded NEON load/store
/// instructions, and rewrite them to their Thumb2 form if we are currently in
/// Thumb2 mode.
unsigned ARMMCCodeEmitter::NEONThumb2LoadStorePostEncoder(const MCInst &MI,
                                                 unsigned EncodedValue,
                                                 const MCSubtargetInfo &STI) const {
  if (isThumb2(STI)) {
    EncodedValue &= 0xF0FFFFFF;
    EncodedValue |= 0x09000000;
  }
```
- EN: Implements `ARMMCCodeEmitter::NEONThumb2LoadStorePostEncoder`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMMCCodeEmitter::NEONThumb2LoadStorePostEncoder`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 504-505
```cpp
  return EncodedValue;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 507-516
```cpp
/// NEONThumb2DupPostEncoder - Post-process encoded NEON vdup
/// instructions, and rewrite them to their Thumb2 form if we are currently in
/// Thumb2 mode.
unsigned ARMMCCodeEmitter::NEONThumb2DupPostEncoder(const MCInst &MI,
                                                 unsigned EncodedValue,
                                                 const MCSubtargetInfo &STI) const {
  if (isThumb2(STI)) {
    EncodedValue &= 0x00FFFFFF;
    EncodedValue |= 0xEE000000;
  }
```
- EN: Implements `ARMMCCodeEmitter::NEONThumb2DupPostEncoder`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMMCCodeEmitter::NEONThumb2DupPostEncoder`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 518-519
```cpp
  return EncodedValue;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 521-528
```cpp
/// Post-process encoded NEON v8 instructions, and rewrite them to Thumb2 form
/// if we are in Thumb2.
unsigned ARMMCCodeEmitter::NEONThumb2V8PostEncoder(const MCInst &MI,
                                                 unsigned EncodedValue,
                                                 const MCSubtargetInfo &STI) const {
  if (isThumb2(STI)) {
    EncodedValue |= 0xC000000; // Set bits 27-26
  }
```
- EN: Implements `ARMMCCodeEmitter::NEONThumb2V8PostEncoder`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMMCCodeEmitter::NEONThumb2V8PostEncoder`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 530-531
```cpp
  return EncodedValue;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 533-543
```cpp
/// VFPThumb2PostEncoder - Post-process encoded VFP instructions and rewrite
/// them to their Thumb2 form if we are currently in Thumb2 mode.
unsigned ARMMCCodeEmitter::
VFPThumb2PostEncoder(const MCInst &MI, unsigned EncodedValue,
                     const MCSubtargetInfo &STI) const {
  if (isThumb2(STI)) {
    EncodedValue &= 0x0FFFFFFF;
    EncodedValue |= 0xE0000000;
  }
  return EncodedValue;
}
```
- EN: Implements `VFPThumb2PostEncoder`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `VFPThumb2PostEncoder`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 545-553
```cpp
/// getMachineOpValue - Return binary encoding of operand. If the machine
/// operand requires relocation, record the relocation and return zero.
unsigned ARMMCCodeEmitter::
getMachineOpValue(const MCInst &MI, const MCOperand &MO,
                  SmallVectorImpl<MCFixup> &Fixups,
                  const MCSubtargetInfo &STI) const {
  if (MO.isReg()) {
    MCRegister Reg = MO.getReg();
    unsigned RegNo = CTX.getRegisterInfo()->getEncodingValue(Reg);
```
- EN: Implements `getMachineOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getMachineOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 555-559
```cpp
    // In NEON, Q registers are encoded as 2x their register number,
    // because they're using the same indices as the D registers they
    // overlap. In MVE, there are no 64-bit vector instructions, so
    // the encodings all refer to Q-registers by their literal
    // register number.
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 561-562
```cpp
    if (STI.hasFeature(ARM::HasMVEIntegerOps))
      return RegNo;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 564-578
```cpp
    switch (Reg.id()) {
    default:
      return RegNo;
    case ARM::Q0:  case ARM::Q1:  case ARM::Q2:  case ARM::Q3:
    case ARM::Q4:  case ARM::Q5:  case ARM::Q6:  case ARM::Q7:
    case ARM::Q8:  case ARM::Q9:  case ARM::Q10: case ARM::Q11:
    case ARM::Q12: case ARM::Q13: case ARM::Q14: case ARM::Q15:
      return 2 * RegNo;
    }
  } else if (MO.isImm()) {
    return static_cast<unsigned>(MO.getImm());
  } else if (MO.isDFPImm()) {
    return static_cast<unsigned>(APFloat(bit_cast<double>(MO.getDFPImm()))
                                     .bitcastToAPInt()
                                     .getHiBits(32)
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 579-580
```cpp
                                     .getLimitedValue());
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 582-583
```cpp
  llvm_unreachable("Unable to encode MCOperand!");
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 585-591
```cpp
/// getAddrModeImmOpValue - Return encoding info for 'reg +/- imm' operand.
bool ARMMCCodeEmitter::
EncodeAddrModeOpValues(const MCInst &MI, unsigned OpIdx, unsigned &Reg,
                       unsigned &Imm, SmallVectorImpl<MCFixup> &Fixups,
                       const MCSubtargetInfo &STI) const {
  const MCOperand &MO  = MI.getOperand(OpIdx);
  const MCOperand &MO1 = MI.getOperand(OpIdx + 1);
```
- EN: Implements `EncodeAddrModeOpValues`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `EncodeAddrModeOpValues`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 593-593
```cpp
  Reg = CTX.getRegisterInfo()->getEncodingValue(MO.getReg());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 595-596
```cpp
  int32_t SImm = MO1.getImm();
  bool isAdd = true;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 598-602
```cpp
  // Special value for #-0
  if (SImm == INT32_MIN) {
    SImm = 0;
    isAdd = false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 604-608
```cpp
  // Immediate is always encoded as positive. The 'U' bit controls add vs sub.
  if (SImm < 0) {
    SImm = -SImm;
    isAdd = false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 610-612
```cpp
  Imm = SImm;
  return isAdd;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 614-620
```cpp
/// getBranchTargetOpValue - Helper function to get the branch target operand,
/// which is either an immediate or requires a fixup.
static uint32_t getBranchTargetOpValue(const MCInst &MI, unsigned OpIdx,
                                       unsigned FixupKind,
                                       SmallVectorImpl<MCFixup> &Fixups,
                                       const MCSubtargetInfo &STI) {
  const MCOperand &MO = MI.getOperand(OpIdx);
```
- EN: Implements `getBranchTargetOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getBranchTargetOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 622-627
```cpp
  // If the destination is an immediate, we have nothing to do.
  if (MO.isImm()) return MO.getImm();
  assert(MO.isExpr() && "Unexpected branch target type!");
  const MCExpr *Expr = MO.getExpr();
  MCFixupKind Kind = MCFixupKind(FixupKind);
  addFixup(Fixups, 0, Expr, Kind);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 629-631
```cpp
  // All of the information is in the fixup.
  return 0;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 633-643
```cpp
// Thumb BL and BLX use a strange offset encoding where bits 22 and 21 are
// determined by negating them and XOR'ing them with bit 23.
static int32_t encodeThumbBLOffset(int32_t offset) {
  offset >>= 1;
  uint32_t S  = (offset & 0x800000) >> 23;
  uint32_t J1 = (offset & 0x400000) >> 22;
  uint32_t J2 = (offset & 0x200000) >> 21;
  J1 = (~J1 & 0x1);
  J2 = (~J2 & 0x1);
  J1 ^= S;
  J2 ^= S;
```
- EN: Implements `encodeThumbBLOffset`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `encodeThumbBLOffset`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 645-647
```cpp
  offset &= ~0x600000;
  offset |= J1 << 22;
  offset |= J2 << 21;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 649-650
```cpp
  return offset;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 652-662
```cpp
/// getThumbBLTargetOpValue - Return encoding info for immediate branch target.
uint32_t ARMMCCodeEmitter::
getThumbBLTargetOpValue(const MCInst &MI, unsigned OpIdx,
                        SmallVectorImpl<MCFixup> &Fixups,
                        const MCSubtargetInfo &STI) const {
  const MCOperand MO = MI.getOperand(OpIdx);
  if (MO.isExpr())
    return ::getBranchTargetOpValue(MI, OpIdx, ARM::fixup_arm_thumb_bl,
                                    Fixups, STI);
  return encodeThumbBLOffset(MO.getImm());
}
```
- EN: Implements `getThumbBLTargetOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getThumbBLTargetOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 664-675
```cpp
/// getThumbBLXTargetOpValue - Return encoding info for Thumb immediate
/// BLX branch target.
uint32_t ARMMCCodeEmitter::
getThumbBLXTargetOpValue(const MCInst &MI, unsigned OpIdx,
                         SmallVectorImpl<MCFixup> &Fixups,
                         const MCSubtargetInfo &STI) const {
  const MCOperand MO = MI.getOperand(OpIdx);
  if (MO.isExpr())
    return ::getBranchTargetOpValue(MI, OpIdx, ARM::fixup_arm_thumb_blx,
                                    Fixups, STI);
  return encodeThumbBLOffset(MO.getImm());
}
```
- EN: Implements `getThumbBLXTargetOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getThumbBLXTargetOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 677-687
```cpp
/// getThumbBRTargetOpValue - Return encoding info for Thumb branch target.
uint32_t ARMMCCodeEmitter::
getThumbBRTargetOpValue(const MCInst &MI, unsigned OpIdx,
                        SmallVectorImpl<MCFixup> &Fixups,
                        const MCSubtargetInfo &STI) const {
  const MCOperand MO = MI.getOperand(OpIdx);
  if (MO.isExpr())
    return ::getBranchTargetOpValue(MI, OpIdx, ARM::fixup_arm_thumb_br,
                                    Fixups, STI);
  return (MO.getImm() >> 1);
}
```
- EN: Implements `getThumbBRTargetOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getThumbBRTargetOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 689-699
```cpp
/// getThumbBCCTargetOpValue - Return encoding info for Thumb branch target.
uint32_t ARMMCCodeEmitter::
getThumbBCCTargetOpValue(const MCInst &MI, unsigned OpIdx,
                         SmallVectorImpl<MCFixup> &Fixups,
                         const MCSubtargetInfo &STI) const {
  const MCOperand MO = MI.getOperand(OpIdx);
  if (MO.isExpr())
    return ::getBranchTargetOpValue(MI, OpIdx, ARM::fixup_arm_thumb_bcc,
                                    Fixups, STI);
  return (MO.getImm() >> 1);
}
```
- EN: Implements `getThumbBCCTargetOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getThumbBCCTargetOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 701-710
```cpp
/// getThumbCBTargetOpValue - Return encoding info for Thumb branch target.
uint32_t ARMMCCodeEmitter::
getThumbCBTargetOpValue(const MCInst &MI, unsigned OpIdx,
                        SmallVectorImpl<MCFixup> &Fixups,
                        const MCSubtargetInfo &STI) const {
  const MCOperand MO = MI.getOperand(OpIdx);
  if (MO.isExpr())
    return ::getBranchTargetOpValue(MI, OpIdx, ARM::fixup_arm_thumb_cb, Fixups, STI);
  return (MO.getImm() >> 1);
}
```
- EN: Implements `getThumbCBTargetOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getThumbCBTargetOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 712-726
```cpp
/// Return true if this branch has a non-always predication
static bool HasConditionalBranch(const MCInst &MI) {
  int NumOp = MI.getNumOperands();
  if (NumOp >= 2) {
    for (int i = 0; i < NumOp-1; ++i) {
      const MCOperand &MCOp1 = MI.getOperand(i);
      const MCOperand &MCOp2 = MI.getOperand(i + 1);
      if (MCOp1.isImm() && MCOp2.isReg() &&
          (!MCOp2.getReg() || MCOp2.getReg() == ARM::CPSR)) {
        if (ARMCC::CondCodes(MCOp1.getImm()) != ARMCC::AL)
          return true;
      }
    }
  }
  return false;
```
- EN: Implements `HasConditionalBranch`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `HasConditionalBranch`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 727-727
```cpp
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 729-741
```cpp
/// getBranchTargetOpValue - Return encoding info for 24-bit immediate branch
/// target.
uint32_t ARMMCCodeEmitter::
getBranchTargetOpValue(const MCInst &MI, unsigned OpIdx,
                       SmallVectorImpl<MCFixup> &Fixups,
                       const MCSubtargetInfo &STI) const {
  // FIXME: This really, really shouldn't use TargetMachine. We don't want
  // coupling between MC and TM anywhere we can help it.
  if (isThumb2(STI))
    return
      ::getBranchTargetOpValue(MI, OpIdx, ARM::fixup_t2_condbranch, Fixups, STI);
  return getARMBranchTargetOpValue(MI, OpIdx, Fixups, STI);
}
```
- EN: Implements `getBranchTargetOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getBranchTargetOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 743-756
```cpp
/// getBranchTargetOpValue - Return encoding info for 24-bit immediate branch
/// target.
uint32_t ARMMCCodeEmitter::
getARMBranchTargetOpValue(const MCInst &MI, unsigned OpIdx,
                          SmallVectorImpl<MCFixup> &Fixups,
                          const MCSubtargetInfo &STI) const {
  const MCOperand MO = MI.getOperand(OpIdx);
  if (MO.isExpr()) {
    if (HasConditionalBranch(MI))
      return ::getBranchTargetOpValue(MI, OpIdx,
                                      ARM::fixup_arm_condbranch, Fixups, STI);
    return ::getBranchTargetOpValue(MI, OpIdx,
                                    ARM::fixup_arm_uncondbranch, Fixups, STI);
  }
```
- EN: Implements `getARMBranchTargetOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getARMBranchTargetOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 758-759
```cpp
  return MO.getImm() >> 2;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 761-771
```cpp
uint32_t ARMMCCodeEmitter::
getARMBLTargetOpValue(const MCInst &MI, unsigned OpIdx,
                          SmallVectorImpl<MCFixup> &Fixups,
                          const MCSubtargetInfo &STI) const {
  const MCOperand MO = MI.getOperand(OpIdx);
  if (MO.isExpr()) {
    if (HasConditionalBranch(MI))
      return ::getBranchTargetOpValue(MI, OpIdx,
                                      ARM::fixup_arm_condbl, Fixups, STI);
    return ::getBranchTargetOpValue(MI, OpIdx, ARM::fixup_arm_uncondbl, Fixups, STI);
  }
```
- EN: Implements `getARMBLTargetOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getARMBLTargetOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 773-774
```cpp
  return MO.getImm() >> 2;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 776-782
```cpp
uint32_t ARMMCCodeEmitter::
getARMBLXTargetOpValue(const MCInst &MI, unsigned OpIdx,
                          SmallVectorImpl<MCFixup> &Fixups,
                          const MCSubtargetInfo &STI) const {
  const MCOperand MO = MI.getOperand(OpIdx);
  if (MO.isExpr())
    return ::getBranchTargetOpValue(MI, OpIdx, ARM::fixup_arm_blx, Fixups, STI);
```
- EN: Implements `getARMBLXTargetOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getARMBLXTargetOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 784-785
```cpp
  return MO.getImm() >> 1;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 787-793
```cpp
/// getUnconditionalBranchTargetOpValue - Return encoding info for 24-bit
/// immediate branch target.
uint32_t ARMMCCodeEmitter::getThumbBranchTargetOpValue(
    const MCInst &MI, unsigned OpIdx, SmallVectorImpl<MCFixup> &Fixups,
    const MCSubtargetInfo &STI) const {
  unsigned Val = 0;
  const MCOperand MO = MI.getOperand(OpIdx);
```
- EN: Implements `ARMMCCodeEmitter::getThumbBranchTargetOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMMCCodeEmitter::getThumbBranchTargetOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 795-798
```cpp
  if(MO.isExpr())
    return ::getBranchTargetOpValue(MI, OpIdx, ARM::fixup_t2_uncondbranch, Fixups, STI);
  else
    Val = MO.getImm() >> 1;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 800-806
```cpp
  bool I  = (Val & 0x800000);
  bool J1 = (Val & 0x400000);
  bool J2 = (Val & 0x200000);
  if (I ^ J1)
    Val &= ~0x400000;
  else
    Val |= 0x400000;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 808-811
```cpp
  if (I ^ J2)
    Val &= ~0x200000;
  else
    Val |= 0x200000;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 813-814
```cpp
  return Val;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 816-827
```cpp
/// getAdrLabelOpValue - Return encoding info for 12-bit shifted-immediate
/// ADR label target.
uint32_t ARMMCCodeEmitter::
getAdrLabelOpValue(const MCInst &MI, unsigned OpIdx,
                   SmallVectorImpl<MCFixup> &Fixups,
                   const MCSubtargetInfo &STI) const {
  const MCOperand MO = MI.getOperand(OpIdx);
  if (MO.isExpr())
    return ::getBranchTargetOpValue(MI, OpIdx, ARM::fixup_arm_adr_pcrel_12,
                                    Fixups, STI);
  int64_t offset = MO.getImm();
  uint32_t Val = 0x2000;
```
- EN: Implements `getAdrLabelOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getAdrLabelOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 829-843
```cpp
  int SoImmVal;
  if (offset == INT32_MIN) {
    Val = 0x1000;
    SoImmVal = 0;
  } else if (offset < 0) {
    Val = 0x1000;
    offset *= -1;
    SoImmVal = ARM_AM::getSOImmVal(offset);
    if(SoImmVal == -1) {
      Val = 0x2000;
      offset *= -1;
      SoImmVal = ARM_AM::getSOImmVal(offset);
    }
  } else {
    SoImmVal = ARM_AM::getSOImmVal(offset);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 844-849
```cpp
    if(SoImmVal == -1) {
      Val = 0x1000;
      offset *= -1;
      SoImmVal = ARM_AM::getSOImmVal(offset);
    }
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 851-851
```cpp
  assert(SoImmVal != -1 && "Not a valid so_imm value!");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 853-855
```cpp
  Val |= SoImmVal;
  return Val;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 857-871
```cpp
/// getT2AdrLabelOpValue - Return encoding info for 12-bit immediate ADR label
/// target.
uint32_t ARMMCCodeEmitter::
getT2AdrLabelOpValue(const MCInst &MI, unsigned OpIdx,
                   SmallVectorImpl<MCFixup> &Fixups,
                   const MCSubtargetInfo &STI) const {
  const MCOperand MO = MI.getOperand(OpIdx);
  if (MO.isExpr())
    return ::getBranchTargetOpValue(MI, OpIdx, ARM::fixup_t2_adr_pcrel_12,
                                    Fixups, STI);
  int32_t Val = MO.getImm();
  if (Val == INT32_MIN)
    Val = 0x1000;
  else if (Val < 0) {
    Val *= -1;
```
- EN: Implements `getT2AdrLabelOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getT2AdrLabelOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 872-875
```cpp
    Val |= 0x1000;
  }
  return Val;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 877-884
```cpp
/// getITMaskOpValue - Return the architectural encoding of an IT
/// predication mask, given the MCOperand format.
uint32_t ARMMCCodeEmitter::
getITMaskOpValue(const MCInst &MI, unsigned OpIdx,
                 SmallVectorImpl<MCFixup> &Fixups,
                 const MCSubtargetInfo &STI) const {
  const MCOperand MaskMO = MI.getOperand(OpIdx);
  assert(MaskMO.isImm() && "Unexpected operand type!");
```
- EN: Implements `getITMaskOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getITMaskOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 886-886
```cpp
  unsigned Mask = MaskMO.getImm();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 888-899
```cpp
  // IT masks are encoded as a sequence of replacement low-order bits
  // for the condition code. So if the low bit of the starting
  // condition code is 1, then we have to flip all the bits above the
  // terminating bit (which is the lowest 1 bit).
  assert(OpIdx > 0 && "IT mask appears first!");
  const MCOperand CondMO = MI.getOperand(OpIdx-1);
  assert(CondMO.isImm() && "Unexpected operand type!");
  if (CondMO.getImm() & 1) {
    unsigned LowBit = Mask & -Mask;
    unsigned BitsAboveLowBit = 0xF & (-LowBit << 1);
    Mask ^= BitsAboveLowBit;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 901-902
```cpp
  return Mask;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 904-915
```cpp
/// getThumbAdrLabelOpValue - Return encoding info for 8-bit immediate ADR label
/// target.
uint32_t ARMMCCodeEmitter::
getThumbAdrLabelOpValue(const MCInst &MI, unsigned OpIdx,
                   SmallVectorImpl<MCFixup> &Fixups,
                   const MCSubtargetInfo &STI) const {
  const MCOperand MO = MI.getOperand(OpIdx);
  if (MO.isExpr())
    return ::getBranchTargetOpValue(MI, OpIdx, ARM::fixup_thumb_adr_pcrel_10,
                                    Fixups, STI);
  return MO.getImm();
}
```
- EN: Implements `getThumbAdrLabelOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getThumbAdrLabelOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 917-931
```cpp
/// getThumbAddrModeRegRegOpValue - Return encoding info for 'reg + reg'
/// operand.
uint32_t ARMMCCodeEmitter::
getThumbAddrModeRegRegOpValue(const MCInst &MI, unsigned OpIdx,
                              SmallVectorImpl<MCFixup> &,
                              const MCSubtargetInfo &STI) const {
  // [Rn, Rm]
  //   {5-3} = Rm
  //   {2-0} = Rn
  const MCOperand &MO1 = MI.getOperand(OpIdx);
  const MCOperand &MO2 = MI.getOperand(OpIdx + 1);
  unsigned Rn = CTX.getRegisterInfo()->getEncodingValue(MO1.getReg());
  unsigned Rm = CTX.getRegisterInfo()->getEncodingValue(MO2.getReg());
  return (Rm << 3) | Rn;
}
```
- EN: Implements `getThumbAddrModeRegRegOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getThumbAddrModeRegRegOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 933-945
```cpp
/// getMVEShiftImmOpValue - Return encoding info for the 'sz:imm5'
/// operand.
uint32_t
ARMMCCodeEmitter::getMVEShiftImmOpValue(const MCInst &MI, unsigned OpIdx,
                                        SmallVectorImpl<MCFixup> &Fixups,
                                        const MCSubtargetInfo &STI) const {
  // {4-0} = szimm5
  // The value we are trying to encode is an immediate between either the
  // range of [1-7] or [1-15] depending on whether we are dealing with the
  // u8/s8 or the u16/s16 variants respectively.
  // This value is encoded as follows, if ShiftImm is the value within those
  // ranges then the encoding szimm5 = ShiftImm + size, where size is either 8
  // or 16.
```
- EN: Implements `ARMMCCodeEmitter::getMVEShiftImmOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMMCCodeEmitter::getMVEShiftImmOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 947-961
```cpp
  unsigned Size, ShiftImm;
  switch(MI.getOpcode()) {
    case ARM::MVE_VSHLL_imms16bh:
    case ARM::MVE_VSHLL_imms16th:
    case ARM::MVE_VSHLL_immu16bh:
    case ARM::MVE_VSHLL_immu16th:
      Size = 16;
      break;
    case ARM::MVE_VSHLL_imms8bh:
    case ARM::MVE_VSHLL_imms8th:
    case ARM::MVE_VSHLL_immu8bh:
    case ARM::MVE_VSHLL_immu8th:
      Size = 8;
      break;
    default:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 962-966
```cpp
      llvm_unreachable("Use of operand not supported by this instruction");
  }
  ShiftImm = MI.getOperand(OpIdx).getImm();
  return Size + ShiftImm;
}
```
- EN: Declares `llvm_unreachable`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `llvm_unreachable`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 968-982
```cpp
/// getAddrModeImm12OpValue - Return encoding info for 'reg +/- imm12' operand.
uint32_t ARMMCCodeEmitter::
getAddrModeImm12OpValue(const MCInst &MI, unsigned OpIdx,
                        SmallVectorImpl<MCFixup> &Fixups,
                        const MCSubtargetInfo &STI) const {
  // {17-13} = reg
  // {12}    = (U)nsigned (add == '1', sub == '0')
  // {11-0}  = imm12
  unsigned Reg = 0, Imm12 = 0;
  bool isAdd = true;
  // If The first operand isn't a register, we have a label reference.
  const MCOperand &MO = MI.getOperand(OpIdx);
  if (MO.isReg()) {
    const MCOperand &MO1 = MI.getOperand(OpIdx + 1);
    if (MO1.isImm()) {
```
- EN: Implements `getAddrModeImm12OpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getAddrModeImm12OpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 983-997
```cpp
      isAdd = EncodeAddrModeOpValues(MI, OpIdx, Reg, Imm12, Fixups, STI);
    } else if (MO1.isExpr()) {
      assert(!isThumb(STI) && !isThumb2(STI) &&
             "Thumb mode requires different encoding");
      Reg = CTX.getRegisterInfo()->getEncodingValue(MO.getReg());
      isAdd = false; // 'U' bit is set as part of the fixup.
      MCFixupKind Kind = ARM::fixup_arm_ldst_abs_12;
      addFixup(Fixups, 0, MO1.getExpr(), Kind);
    }
  } else if (MO.isExpr()) {
    Reg = CTX.getRegisterInfo()->getEncodingValue(ARM::PC); // Rn is PC.
    isAdd = false; // 'U' bit is set as part of the fixup.
    MCFixupKind Kind;
    if (isThumb2(STI))
      Kind = ARM::fixup_t2_ldst_pcrel_12;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 998-1000
```cpp
    else
      Kind = ARM::fixup_arm_ldst_pcrel_12;
    addFixup(Fixups, 0, MO.getExpr(), Kind);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1002-1016
```cpp
    ++MCNumCPRelocations;
  } else {
    Reg = ARM::PC;
    int32_t Offset = MO.getImm();
    if (Offset == INT32_MIN) {
      Offset = 0;
      isAdd = false;
    } else if (Offset < 0) {
      Offset *= -1;
      isAdd = false;
    }
    Imm12 = Offset;
  }
  uint32_t Binary = Imm12 & 0xfff;
  // Immediate is always encoded as positive. The 'U' bit controls add vs sub.
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1017-1021
```cpp
  if (isAdd)
    Binary |= (1 << 12);
  Binary |= (Reg << 13);
  return Binary;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1023-1033
```cpp
template<unsigned Bits, unsigned Shift>
uint32_t ARMMCCodeEmitter::
getT2ScaledImmOpValue(const MCInst &MI, unsigned OpIdx,
                      SmallVectorImpl<MCFixup> &Fixups,
                      const MCSubtargetInfo &STI) const {
  // FIXME: The immediate operand should have already been encoded like this
  // before ever getting here. The encoder method should just need to combine
  // the MI operands for the register and the offset into a single
  // representation for the complex operand in the .td file. This isn't just
  // style, unfortunately. As-is, we can't represent the distinct encoding
  // for #-0.
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1035-1038
```cpp
  // {Bits}    = (U)nsigned (add == '1', sub == '0')
  // {(Bits-1)-0}  = immediate
  int32_t Imm = MI.getOperand(OpIdx).getImm();
  bool isAdd = Imm >= 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1040-1042
```cpp
  // Immediate is always encoded as positive. The 'U' bit controls add vs sub.
  if (Imm < 0)
    Imm = -(uint32_t)Imm;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1044-1044
```cpp
  Imm >>= Shift;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1046-1051
```cpp
  uint32_t Binary = Imm & ((1U << Bits) - 1);
  // Immediate is always encoded as positive. The 'U' bit controls add vs sub.
  if (isAdd)
    Binary |= (1U << Bits);
  return Binary;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1053-1062
```cpp
/// getMveAddrModeRQOpValue - Return encoding info for 'reg, vreg'
/// operand.
uint32_t ARMMCCodeEmitter::
getMveAddrModeRQOpValue(const MCInst &MI, unsigned OpIdx,
                        SmallVectorImpl<MCFixup> &Fixups,
                        const MCSubtargetInfo &STI) const {
    // {6-3} Rn
    // {2-0} Qm
    const MCOperand &M0 = MI.getOperand(OpIdx);
    const MCOperand &M1 = MI.getOperand(OpIdx + 1);
```
- EN: Implements `getMveAddrModeRQOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getMveAddrModeRQOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1064-1065
```cpp
    unsigned Rn = CTX.getRegisterInfo()->getEncodingValue(M0.getReg());
    unsigned Qm = CTX.getRegisterInfo()->getEncodingValue(M1.getReg());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1067-1067
```cpp
    assert(Qm < 8 && "Qm is supposed to be encodable in 3 bits");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1069-1070
```cpp
    return (Rn << 3) | Qm;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1072-1082
```cpp
/// getMveAddrModeRQOpValue - Return encoding info for 'reg, vreg'
/// operand.
template<int shift>
uint32_t ARMMCCodeEmitter::
getMveAddrModeQOpValue(const MCInst &MI, unsigned OpIdx,
                        SmallVectorImpl<MCFixup> &Fixups,
                        const MCSubtargetInfo &STI) const {
    // {10-8} Qm
    // {7-0} Imm
    const MCOperand &M0 = MI.getOperand(OpIdx);
    const MCOperand &M1 = MI.getOperand(OpIdx + 1);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1084-1085
```cpp
    unsigned Qm = CTX.getRegisterInfo()->getEncodingValue(M0.getReg());
    int32_t Imm = M1.getImm();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1087-1087
```cpp
    bool isAdd = Imm >= 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1089-1089
```cpp
    Imm >>= shift;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1091-1092
```cpp
    if (!isAdd)
      Imm = -(uint32_t)Imm;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1094-1094
```cpp
    Imm &= 0x7f;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1096-1097
```cpp
    if (isAdd)
      Imm |= 0x80;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1099-1099
```cpp
    assert(Qm < 8 && "Qm is supposed to be encodable in 3 bits");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1101-1102
```cpp
    return (Qm << 8) | Imm;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1104-1118
```cpp
/// getT2AddrModeImm8s4OpValue - Return encoding info for
/// 'reg +/- imm8<<2' operand.
uint32_t ARMMCCodeEmitter::
getT2AddrModeImm8s4OpValue(const MCInst &MI, unsigned OpIdx,
                        SmallVectorImpl<MCFixup> &Fixups,
                        const MCSubtargetInfo &STI) const {
  // {12-9} = reg
  // {8}    = (U)nsigned (add == '1', sub == '0')
  // {7-0}  = imm8
  unsigned Reg, Imm8;
  bool isAdd = true;
  // If The first operand isn't a register, we have a label reference.
  const MCOperand &MO = MI.getOperand(OpIdx);
  if (!MO.isReg()) {
    Reg = CTX.getRegisterInfo()->getEncodingValue(ARM::PC);   // Rn is PC.
```
- EN: Implements `getT2AddrModeImm8s4OpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getT2AddrModeImm8s4OpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1119-1120
```cpp
    Imm8 = 0;
    isAdd = false ; // 'U' bit is set as part of the fixup.
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1122-1125
```cpp
    assert(MO.isExpr() && "Unexpected machine operand type!");
    const MCExpr *Expr = MO.getExpr();
    MCFixupKind Kind = ARM::fixup_t2_pcrel_10;
    addFixup(Fixups, 0, Expr, Kind);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1127-1129
```cpp
    ++MCNumCPRelocations;
  } else
    isAdd = EncodeAddrModeOpValues(MI, OpIdx, Reg, Imm8, Fixups, STI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1131-1144
```cpp
  // FIXME: The immediate operand should have already been encoded like this
  // before ever getting here. The encoder method should just need to combine
  // the MI operands for the register and the offset into a single
  // representation for the complex operand in the .td file. This isn't just
  // style, unfortunately. As-is, we can't represent the distinct encoding
  // for #-0.
  assert(((Imm8 & 0x3) == 0) && "Not a valid immediate!");
  uint32_t Binary = (Imm8 >> 2) & 0xff;
  // Immediate is always encoded as positive. The 'U' bit controls add vs sub.
  if (isAdd)
    Binary |= (1 << 8);
  Binary |= (Reg << 9);
  return Binary;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1146-1157
```cpp
/// getT2AddrModeImm7s4OpValue - Return encoding info for
/// 'reg +/- imm7<<2' operand.
uint32_t
ARMMCCodeEmitter::getT2AddrModeImm7s4OpValue(const MCInst &MI, unsigned OpIdx,
                                             SmallVectorImpl<MCFixup> &Fixups,
                                             const MCSubtargetInfo &STI) const {
  // {11-8} = reg
  // {7}    = (A)dd (add == '1', sub == '0')
  // {6-0}  = imm7
  unsigned Reg, Imm7;
  // If The first operand isn't a register, we have a label reference.
  bool isAdd = EncodeAddrModeOpValues(MI, OpIdx, Reg, Imm7, Fixups, STI);
```
- EN: Implements `ARMMCCodeEmitter::getT2AddrModeImm7s4OpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMMCCodeEmitter::getT2AddrModeImm7s4OpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1159-1171
```cpp
  // FIXME: The immediate operand should have already been encoded like this
  // before ever getting here. The encoder method should just need to combine
  // the MI operands for the register and the offset into a single
  // representation for the complex operand in the .td file. This isn't just
  // style, unfortunately. As-is, we can't represent the distinct encoding
  // for #-0.
  uint32_t Binary = (Imm7 >> 2) & 0xff;
  // Immediate is always encoded as positive. The 'A' bit controls add vs sub.
  if (isAdd)
    Binary |= (1 << 7);
  Binary |= (Reg << 8);
  return Binary;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1173-1186
```cpp
/// getT2AddrModeImm0_1020s4OpValue - Return encoding info for
/// 'reg + imm8<<2' operand.
uint32_t ARMMCCodeEmitter::
getT2AddrModeImm0_1020s4OpValue(const MCInst &MI, unsigned OpIdx,
                        SmallVectorImpl<MCFixup> &Fixups,
                        const MCSubtargetInfo &STI) const {
  // {11-8} = reg
  // {7-0}  = imm8
  const MCOperand &MO = MI.getOperand(OpIdx);
  const MCOperand &MO1 = MI.getOperand(OpIdx + 1);
  unsigned Reg = CTX.getRegisterInfo()->getEncodingValue(MO.getReg());
  unsigned Imm8 = MO1.getImm();
  return (Reg << 8) | Imm8;
}
```
- EN: Implements `getT2AddrModeImm0_1020s4OpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getT2AddrModeImm0_1020s4OpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1188-1196
```cpp
uint32_t ARMMCCodeEmitter::getHiLoImmOpValue(const MCInst &MI, unsigned OpIdx,
                                             SmallVectorImpl<MCFixup> &Fixups,
                                             const MCSubtargetInfo &STI) const {
  // {20-16} = imm{15-12}
  // {11-0}  = imm{11-0}
  const MCOperand &MO = MI.getOperand(OpIdx);
  if (MO.isImm())
    // Hi / lo bits already extracted during earlier passes.
    return static_cast<unsigned>(MO.getImm());
```
- EN: Implements `ARMMCCodeEmitter::getHiLoImmOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMMCCodeEmitter::getHiLoImmOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1198-1204
```cpp
  // Handle :upper16:, :lower16:, :upper8_15:, :upper0_7:, :lower8_15:
  // :lower0_7: assembly prefixes.
  const MCExpr *E = MO.getExpr();
  MCFixupKind Kind;
  if (E->getKind() == MCExpr::Specifier) {
    auto *ARM16Expr = cast<MCSpecifierExpr>(E);
    E = ARM16Expr->getSubExpr();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1206-1209
```cpp
    if (const MCConstantExpr *MCE = dyn_cast<MCConstantExpr>(E)) {
      const int64_t Value = MCE->getValue();
      if (Value > UINT32_MAX)
        report_fatal_error("constant value truncated (limited to 32-bit)");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1211-1215
```cpp
      switch (ARM16Expr->getSpecifier()) {
      case ARM::S_HI16:
        return (int32_t(Value) & 0xffff0000) >> 16;
      case ARM::S_LO16:
        return (int32_t(Value) & 0x0000ffff);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1217-1224
```cpp
      case ARM::S_HI_8_15:
        return (int32_t(Value) & 0xff000000) >> 24;
      case ARM::S_HI_0_7:
        return (int32_t(Value) & 0x00ff0000) >> 16;
      case ARM::S_LO_8_15:
        return (int32_t(Value) & 0x0000ff00) >> 8;
      case ARM::S_LO_0_7:
        return (int32_t(Value) & 0x000000ff);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1226-1228
```cpp
      default: llvm_unreachable("Unsupported ARMFixup");
      }
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1230-1244
```cpp
    switch (ARM16Expr->getSpecifier()) {
    default: llvm_unreachable("Unsupported ARMFixup");
    case ARM::S_HI16:
      Kind = MCFixupKind(isThumb(STI) ? ARM::fixup_t2_movt_hi16
                                      : ARM::fixup_arm_movt_hi16);
      break;
    case ARM::S_LO16:
      Kind = MCFixupKind(isThumb(STI) ? ARM::fixup_t2_movw_lo16
                                      : ARM::fixup_arm_movw_lo16);
      break;
    case ARM::S_HI_8_15:
      if (!isThumb(STI))
        llvm_unreachable(":upper_8_15: not supported in Arm state");
      Kind = ARM::fixup_arm_thumb_upper_8_15;
      break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1245-1259
```cpp
    case ARM::S_HI_0_7:
      if (!isThumb(STI))
        llvm_unreachable(":upper_0_7: not supported in Arm state");
      Kind = ARM::fixup_arm_thumb_upper_0_7;
      break;
    case ARM::S_LO_8_15:
      if (!isThumb(STI))
        llvm_unreachable(":lower_8_15: not supported in Arm state");
      Kind = ARM::fixup_arm_thumb_lower_8_15;
      break;
    case ARM::S_LO_0_7:
      if (!isThumb(STI))
        llvm_unreachable(":lower_0_7: not supported in Arm state");
      Kind = ARM::fixup_arm_thumb_lower_0_7;
      break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1260-1260
```cpp
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1262-1273
```cpp
    addFixup(Fixups, 0, E, Kind);
    return 0;
  }
  // If the expression doesn't have :upper16:, :lower16: on it, it's just a
  // plain immediate expression, previously those evaluated to the lower 16 bits
  // of the expression regardless of whether we have a movt or a movw, but that
  // led to misleadingly results.  This is disallowed in the AsmParser in
  // validateInstruction() so this should never happen.  The same holds for
  // thumb1 :upper8_15:, :upper0_7:, lower8_15: or :lower0_7: with movs or adds.
  llvm_unreachable("expression without :upper16:, :lower16:, :upper8_15:,"
                   ":upper0_7:, lower8_15: or :lower0_7:");
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1275-1287
```cpp
uint32_t ARMMCCodeEmitter::
getLdStSORegOpValue(const MCInst &MI, unsigned OpIdx,
                    SmallVectorImpl<MCFixup> &Fixups,
                    const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(OpIdx);
  const MCOperand &MO1 = MI.getOperand(OpIdx+1);
  const MCOperand &MO2 = MI.getOperand(OpIdx+2);
  unsigned Rn = CTX.getRegisterInfo()->getEncodingValue(MO.getReg());
  unsigned Rm = CTX.getRegisterInfo()->getEncodingValue(MO1.getReg());
  unsigned ShImm = ARM_AM::getAM2Offset(MO2.getImm());
  bool isAdd = ARM_AM::getAM2Op(MO2.getImm()) == ARM_AM::add;
  ARM_AM::ShiftOpc ShOp = ARM_AM::getAM2ShiftOpc(MO2.getImm());
  unsigned SBits = getShiftOp(ShOp);
```
- EN: Implements `getLdStSORegOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getLdStSORegOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1289-1291
```cpp
  // While "lsr #32" and "asr #32" exist, they are encoded with a 0 in the shift
  // amount. However, it would be an easy mistake to make so check here.
  assert((ShImm & ~0x1f) == 0 && "Out of range shift amount");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1293-1307
```cpp
  // {16-13} = Rn
  // {12}    = isAdd
  // {11-0}  = shifter
  //  {3-0}  = Rm
  //  {4}    = 0
  //  {6-5}  = type
  //  {11-7} = imm
  uint32_t Binary = Rm;
  Binary |= Rn << 13;
  Binary |= SBits << 5;
  Binary |= ShImm << 7;
  if (isAdd)
    Binary |= 1 << 12;
  return Binary;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1309-1323
```cpp
uint32_t ARMMCCodeEmitter::
getAddrMode2OffsetOpValue(const MCInst &MI, unsigned OpIdx,
                          SmallVectorImpl<MCFixup> &Fixups,
                          const MCSubtargetInfo &STI) const {
  // {13}     1 == imm12, 0 == Rm
  // {12}     isAdd
  // {11-0}   imm12/Rm
  const MCOperand &MO = MI.getOperand(OpIdx);
  const MCOperand &MO1 = MI.getOperand(OpIdx+1);
  unsigned Imm = MO1.getImm();
  bool isAdd = ARM_AM::getAM2Op(Imm) == ARM_AM::add;
  bool isReg = MO.getReg().isValid();
  uint32_t Binary = ARM_AM::getAM2Offset(Imm);
  // if reg +/- reg, Rm will be non-zero. Otherwise, we have reg +/- imm12
  if (isReg) {
```
- EN: Implements `getAddrMode2OffsetOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getAddrMode2OffsetOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1324-1330
```cpp
    ARM_AM::ShiftOpc ShOp = ARM_AM::getAM2ShiftOpc(Imm);
    Binary <<= 7;                    // Shift amount is bits [11:7]
    Binary |= getShiftOp(ShOp) << 5; // Shift type is bits [6:5]
    Binary |= CTX.getRegisterInfo()->getEncodingValue(MO.getReg()); // Rm is bits [3:0]
  }
  return Binary | (isAdd << 12) | (isReg << 13);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1332-1342
```cpp
uint32_t ARMMCCodeEmitter::
getPostIdxRegOpValue(const MCInst &MI, unsigned OpIdx,
                     SmallVectorImpl<MCFixup> &Fixups,
                     const MCSubtargetInfo &STI) const {
  // {4}      isAdd
  // {3-0}    Rm
  const MCOperand &MO = MI.getOperand(OpIdx);
  const MCOperand &MO1 = MI.getOperand(OpIdx+1);
  bool isAdd = MO1.getImm() != 0;
  return CTX.getRegisterInfo()->getEncodingValue(MO.getReg()) | (isAdd << 4);
}
```
- EN: Implements `getPostIdxRegOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getPostIdxRegOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1344-1358
```cpp
uint32_t ARMMCCodeEmitter::
getAddrMode3OffsetOpValue(const MCInst &MI, unsigned OpIdx,
                          SmallVectorImpl<MCFixup> &Fixups,
                          const MCSubtargetInfo &STI) const {
  // {9}      1 == imm8, 0 == Rm
  // {8}      isAdd
  // {7-4}    imm7_4/zero
  // {3-0}    imm3_0/Rm
  const MCOperand &MO = MI.getOperand(OpIdx);
  const MCOperand &MO1 = MI.getOperand(OpIdx+1);
  unsigned Imm = MO1.getImm();
  bool isAdd = ARM_AM::getAM3Op(Imm) == ARM_AM::add;
  bool isImm = !MO.getReg().isValid();
  uint32_t Imm8 = ARM_AM::getAM3Offset(Imm);
  // if reg +/- reg, Rm will be non-zero. Otherwise, we have reg +/- imm8
```
- EN: Implements `getAddrMode3OffsetOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getAddrMode3OffsetOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1359-1362
```cpp
  if (!isImm)
    Imm8 = CTX.getRegisterInfo()->getEncodingValue(MO.getReg());
  return Imm8 | (isAdd << 8) | (isImm << 9);
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1364-1375
```cpp
uint32_t ARMMCCodeEmitter::
getAddrMode3OpValue(const MCInst &MI, unsigned OpIdx,
                    SmallVectorImpl<MCFixup> &Fixups,
                    const MCSubtargetInfo &STI) const {
  // {13}     1 == imm8, 0 == Rm
  // {12-9}   Rn
  // {8}      isAdd
  // {7-4}    imm7_4/zero
  // {3-0}    imm3_0/Rm
  const MCOperand &MO = MI.getOperand(OpIdx);
  const MCOperand &MO1 = MI.getOperand(OpIdx+1);
  const MCOperand &MO2 = MI.getOperand(OpIdx+2);
```
- EN: Implements `getAddrMode3OpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getAddrMode3OpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1377-1379
```cpp
  // If The first operand isn't a register, we have a label reference.
  if (!MO.isReg()) {
    unsigned Rn = CTX.getRegisterInfo()->getEncodingValue(ARM::PC);   // Rn is PC.
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1381-1384
```cpp
    assert(MO.isExpr() && "Unexpected machine operand type!");
    const MCExpr *Expr = MO.getExpr();
    MCFixupKind Kind = ARM::fixup_arm_pcrel_10_unscaled;
    addFixup(Fixups, 0, Expr, Kind);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1386-1398
```cpp
    ++MCNumCPRelocations;
    return (Rn << 9) | (1 << 13);
  }
  unsigned Rn = CTX.getRegisterInfo()->getEncodingValue(MO.getReg());
  unsigned Imm = MO2.getImm();
  bool isAdd = ARM_AM::getAM3Op(Imm) == ARM_AM::add;
  bool isImm = !MO1.getReg().isValid();
  uint32_t Imm8 = ARM_AM::getAM3Offset(Imm);
  // if reg +/- reg, Rm will be non-zero. Otherwise, we have reg +/- imm8
  if (!isImm)
    Imm8 = CTX.getRegisterInfo()->getEncodingValue(MO1.getReg());
  return (Rn << 9) | Imm8 | (isAdd << 8) | (isImm << 13);
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1400-1409
```cpp
/// getAddrModeThumbSPOpValue - Encode the t_addrmode_sp operands.
uint32_t ARMMCCodeEmitter::
getAddrModeThumbSPOpValue(const MCInst &MI, unsigned OpIdx,
                          SmallVectorImpl<MCFixup> &Fixups,
                          const MCSubtargetInfo &STI) const {
  // [SP, #imm]
  //   {7-0} = imm8
  const MCOperand &MO1 = MI.getOperand(OpIdx + 1);
  assert(MI.getOperand(OpIdx).getReg() == ARM::SP &&
         "Unexpected base register!");
```
- EN: Implements `getAddrModeThumbSPOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getAddrModeThumbSPOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1411-1414
```cpp
  // The immediate is already shifted for the implicit zeroes, so no change
  // here.
  return MO1.getImm() & 0xff;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1416-1429
```cpp
/// getAddrModeISOpValue - Encode the t_addrmode_is# operands.
uint32_t ARMMCCodeEmitter::
getAddrModeISOpValue(const MCInst &MI, unsigned OpIdx,
                     SmallVectorImpl<MCFixup> &Fixups,
                     const MCSubtargetInfo &STI) const {
  // [Rn, #imm]
  //   {7-3} = imm5
  //   {2-0} = Rn
  const MCOperand &MO = MI.getOperand(OpIdx);
  const MCOperand &MO1 = MI.getOperand(OpIdx + 1);
  unsigned Rn = CTX.getRegisterInfo()->getEncodingValue(MO.getReg());
  unsigned Imm5 = MO1.getImm();
  return ((Imm5 & 0x1f) << 3) | Rn;
}
```
- EN: Implements `getAddrModeISOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getAddrModeISOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1431-1440
```cpp
/// getAddrModePCOpValue - Return encoding for t_addrmode_pc operands.
uint32_t ARMMCCodeEmitter::
getAddrModePCOpValue(const MCInst &MI, unsigned OpIdx,
                     SmallVectorImpl<MCFixup> &Fixups,
                     const MCSubtargetInfo &STI) const {
  const MCOperand MO = MI.getOperand(OpIdx);
  if (MO.isExpr())
    return ::getBranchTargetOpValue(MI, OpIdx, ARM::fixup_arm_thumb_cp, Fixups, STI);
  return (MO.getImm() >> 2);
}
```
- EN: Implements `getAddrModePCOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getAddrModePCOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1442-1456
```cpp
/// getAddrMode5OpValue - Return encoding info for 'reg +/- (imm8 << 2)' operand.
uint32_t ARMMCCodeEmitter::
getAddrMode5OpValue(const MCInst &MI, unsigned OpIdx,
                    SmallVectorImpl<MCFixup> &Fixups,
                    const MCSubtargetInfo &STI) const {
  // {12-9} = reg
  // {8}    = (U)nsigned (add == '1', sub == '0')
  // {7-0}  = imm8
  unsigned Reg, Imm8;
  bool isAdd;
  // If The first operand isn't a register, we have a label reference.
  const MCOperand &MO = MI.getOperand(OpIdx);
  if (!MO.isReg()) {
    Reg = CTX.getRegisterInfo()->getEncodingValue(ARM::PC);   // Rn is PC.
    Imm8 = 0;
```
- EN: Implements `getAddrMode5OpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getAddrMode5OpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1457-1457
```cpp
    isAdd = false; // 'U' bit is handled as part of the fixup.
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1459-1466
```cpp
    assert(MO.isExpr() && "Unexpected machine operand type!");
    const MCExpr *Expr = MO.getExpr();
    MCFixupKind Kind;
    if (isThumb2(STI))
      Kind = ARM::fixup_t2_pcrel_10;
    else
      Kind = ARM::fixup_arm_pcrel_10;
    addFixup(Fixups, 0, Expr, Kind);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1468-1472
```cpp
    ++MCNumCPRelocations;
  } else {
    EncodeAddrModeOpValues(MI, OpIdx, Reg, Imm8, Fixups, STI);
    isAdd = ARM_AM::getAM5Op(Imm8) == ARM_AM::add;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1474-1480
```cpp
  uint32_t Binary = ARM_AM::getAM5Offset(Imm8);
  // Immediate is always encoded as positive. The 'U' bit controls add vs sub.
  if (isAdd)
    Binary |= (1 << 8);
  Binary |= (Reg << 9);
  return Binary;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1482-1496
```cpp
/// getAddrMode5FP16OpValue - Return encoding info for 'reg +/- (imm8 << 1)' operand.
uint32_t ARMMCCodeEmitter::
getAddrMode5FP16OpValue(const MCInst &MI, unsigned OpIdx,
                    SmallVectorImpl<MCFixup> &Fixups,
                    const MCSubtargetInfo &STI) const {
  // {12-9} = reg
  // {8}    = (U)nsigned (add == '1', sub == '0')
  // {7-0}  = imm8
  unsigned Reg, Imm8;
  bool isAdd;
  // If The first operand isn't a register, we have a label reference.
  const MCOperand &MO = MI.getOperand(OpIdx);
  if (!MO.isReg()) {
    Reg = CTX.getRegisterInfo()->getEncodingValue(ARM::PC);   // Rn is PC.
    Imm8 = 0;
```
- EN: Implements `getAddrMode5FP16OpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getAddrMode5FP16OpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1497-1497
```cpp
    isAdd = false; // 'U' bit is handled as part of the fixup.
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1499-1506
```cpp
    assert(MO.isExpr() && "Unexpected machine operand type!");
    const MCExpr *Expr = MO.getExpr();
    MCFixupKind Kind;
    if (isThumb2(STI))
      Kind = ARM::fixup_t2_pcrel_9;
    else
      Kind = ARM::fixup_arm_pcrel_9;
    addFixup(Fixups, 0, Expr, Kind);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1508-1512
```cpp
    ++MCNumCPRelocations;
  } else {
    EncodeAddrModeOpValues(MI, OpIdx, Reg, Imm8, Fixups, STI);
    isAdd = ARM_AM::getAM5Op(Imm8) == ARM_AM::add;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1514-1520
```cpp
  uint32_t Binary = ARM_AM::getAM5Offset(Imm8);
  // Immediate is always encoded as positive. The 'U' bit controls add vs sub.
  if (isAdd)
    Binary |= (1 << 8);
  Binary |= (Reg << 9);
  return Binary;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1522-1525
```cpp
unsigned ARMMCCodeEmitter::getModImmOpValue(const MCInst &MI, unsigned Op,
                                            SmallVectorImpl<MCFixup> &Fixups,
                                            const MCSubtargetInfo &ST) const {
  const MCOperand &MO = MI.getOperand(Op);
```
- EN: Implements `ARMMCCodeEmitter::getModImmOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMMCCodeEmitter::getModImmOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1527-1534
```cpp
  // Support for fixups (MCFixup)
  if (MO.isExpr()) {
    const MCExpr *Expr = MO.getExpr();
    // Fixups resolve to plain values that need to be encoded.
    MCFixupKind Kind = ARM::fixup_arm_mod_imm;
    addFixup(Fixups, 0, Expr, Kind);
    return 0;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1536-1538
```cpp
  // Immediate is already in its encoded format
  return MO.getImm();
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1540-1543
```cpp
unsigned ARMMCCodeEmitter::getT2SOImmOpValue(const MCInst &MI, unsigned Op,
                                             SmallVectorImpl<MCFixup> &Fixups,
                                             const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(Op);
```
- EN: Implements `ARMMCCodeEmitter::getT2SOImmOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMMCCodeEmitter::getT2SOImmOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1545-1557
```cpp
  // Support for fixups (MCFixup)
  if (MO.isExpr()) {
    const MCExpr *Expr = MO.getExpr();
    // Fixups resolve to plain values that need to be encoded.
    MCFixupKind Kind = ARM::fixup_t2_so_imm;
    addFixup(Fixups, 0, Expr, Kind);
    return 0;
  }
  unsigned SoImm = MO.getImm();
  unsigned Encoded = ARM_AM::getT2SOImmVal(SoImm);
  assert(Encoded != ~0U && "Not a Thumb2 so_imm value?");
  return Encoded;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1559-1571
```cpp
unsigned ARMMCCodeEmitter::
getSORegRegOpValue(const MCInst &MI, unsigned OpIdx,
                SmallVectorImpl<MCFixup> &Fixups,
                const MCSubtargetInfo &STI) const {
  // Sub-operands are [reg, reg, imm]. The first register is Rm, the reg to be
  // shifted. The second is Rs, the amount to shift by, and the third specifies
  // the type of the shift.
  //
  // {3-0} = Rm.
  // {4}   = 1
  // {6-5} = type
  // {11-8} = Rs
  // {7}    = 0
```
- EN: Implements `getSORegRegOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getSORegRegOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1573-1576
```cpp
  const MCOperand &MO  = MI.getOperand(OpIdx);
  const MCOperand &MO1 = MI.getOperand(OpIdx + 1);
  const MCOperand &MO2 = MI.getOperand(OpIdx + 2);
  ARM_AM::ShiftOpc SOpc = ARM_AM::getSORegShOp(MO2.getImm());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1578-1579
```cpp
  // Encode Rm.
  unsigned Binary = CTX.getRegisterInfo()->getEncodingValue(MO.getReg());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1581-1595
```cpp
  // Encode the shift opcode.
  unsigned SBits = 0;
  MCRegister Rs = MO1.getReg();
  if (Rs) {
    // Set shift operand (bit[7:4]).
    // LSL - 0001
    // LSR - 0011
    // ASR - 0101
    // ROR - 0111
    switch (SOpc) {
    default: llvm_unreachable("Unknown shift opc!");
    case ARM_AM::lsl: SBits = 0x1; break;
    case ARM_AM::lsr: SBits = 0x3; break;
    case ARM_AM::asr: SBits = 0x5; break;
    case ARM_AM::ror: SBits = 0x7; break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1596-1597
```cpp
    }
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1599-1599
```cpp
  Binary |= SBits << 4;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1601-1605
```cpp
  // Encode the shift operation Rs.
  // Encode Rs bit[11:8].
  assert(ARM_AM::getSORegOffset(MO2.getImm()) == 0);
  return Binary | (CTX.getRegisterInfo()->getEncodingValue(Rs) << ARMII::RegRsShift);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1607-1617
```cpp
unsigned ARMMCCodeEmitter::
getSORegImmOpValue(const MCInst &MI, unsigned OpIdx,
                SmallVectorImpl<MCFixup> &Fixups,
                const MCSubtargetInfo &STI) const {
  // Sub-operands are [reg, imm]. The first register is Rm, the reg to be
  // shifted. The second is the amount to shift by.
  //
  // {3-0} = Rm.
  // {4}   = 0
  // {6-5} = type
  // {11-7} = imm
```
- EN: Implements `getSORegImmOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getSORegImmOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1619-1621
```cpp
  const MCOperand &MO  = MI.getOperand(OpIdx);
  const MCOperand &MO1 = MI.getOperand(OpIdx + 1);
  ARM_AM::ShiftOpc SOpc = ARM_AM::getSORegShOp(MO1.getImm());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1623-1624
```cpp
  // Encode Rm.
  unsigned Binary = CTX.getRegisterInfo()->getEncodingValue(MO.getReg());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1626-1627
```cpp
  // Encode the shift opcode.
  unsigned SBits = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1629-1643
```cpp
  // Set shift operand (bit[6:4]).
  // LSL - 000
  // LSR - 010
  // ASR - 100
  // ROR - 110
  // RRX - 110 and bit[11:8] clear.
  switch (SOpc) {
  default: llvm_unreachable("Unknown shift opc!");
  case ARM_AM::lsl: SBits = 0x0; break;
  case ARM_AM::lsr: SBits = 0x2; break;
  case ARM_AM::asr: SBits = 0x4; break;
  case ARM_AM::ror: SBits = 0x6; break;
  case ARM_AM::rrx:
    Binary |= 0x60;
    return Binary;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1644-1644
```cpp
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1646-1651
```cpp
  // Encode shift_imm bit[11:7].
  Binary |= SBits << 4;
  unsigned Offset = ARM_AM::getSORegOffset(MO1.getImm());
  assert(Offset < 32 && "Offset must be in range 0-31!");
  return Binary | (Offset << 7);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1654-1660
```cpp
unsigned ARMMCCodeEmitter::
getT2AddrModeSORegOpValue(const MCInst &MI, unsigned OpNum,
                SmallVectorImpl<MCFixup> &Fixups,
                const MCSubtargetInfo &STI) const {
  const MCOperand &MO1 = MI.getOperand(OpNum);
  const MCOperand &MO2 = MI.getOperand(OpNum+1);
  const MCOperand &MO3 = MI.getOperand(OpNum+2);
```
- EN: Implements `getT2AddrModeSORegOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getT2AddrModeSORegOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1662-1668
```cpp
  // Encoded as [Rn, Rm, imm].
  // FIXME: Needs fixup support.
  unsigned Value = CTX.getRegisterInfo()->getEncodingValue(MO1.getReg());
  Value <<= 4;
  Value |= CTX.getRegisterInfo()->getEncodingValue(MO2.getReg());
  Value <<= 2;
  Value |= MO3.getImm();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1670-1671
```cpp
  return Value;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1673-1679
```cpp
template<unsigned Bits, unsigned Shift>
unsigned ARMMCCodeEmitter::
getT2AddrModeImmOpValue(const MCInst &MI, unsigned OpNum,
                        SmallVectorImpl<MCFixup> &Fixups,
                        const MCSubtargetInfo &STI) const {
  const MCOperand &MO1 = MI.getOperand(OpNum);
  const MCOperand &MO2 = MI.getOperand(OpNum+1);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1681-1682
```cpp
  // FIXME: Needs fixup support.
  unsigned Value = CTX.getRegisterInfo()->getEncodingValue(MO1.getReg());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1684-1697
```cpp
  // If the immediate is B bits long, we need B+1 bits in order
  // to represent the (inverse of the) sign bit.
  Value <<= (Bits + 1);
  int32_t tmp = (int32_t)MO2.getImm();
  if (tmp == INT32_MIN) { // represents subtracting zero rather than adding it
    tmp = 0;
  } else if (tmp < 0) {
    tmp = abs(tmp);
  } else {
    Value |= (1U << Bits); // Set the ADD bit
  }
  Value |= (tmp >> Shift) & ((1U << Bits) - 1);
  return Value;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1699-1703
```cpp
unsigned ARMMCCodeEmitter::
getT2AddrModeImm8OffsetOpValue(const MCInst &MI, unsigned OpNum,
                         SmallVectorImpl<MCFixup> &Fixups,
                         const MCSubtargetInfo &STI) const {
  const MCOperand &MO1 = MI.getOperand(OpNum);
```
- EN: Implements `getT2AddrModeImm8OffsetOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getT2AddrModeImm8OffsetOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1705-1714
```cpp
  // FIXME: Needs fixup support.
  unsigned Value = 0;
  auto tmp = static_cast<uint32_t>(MO1.getImm());
  if (static_cast<int32_t>(tmp) < 0)
    tmp = -tmp;
  else
    Value |= 256; // Set the ADD bit
  Value |= tmp & 255;
  return Value;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1716-1726
```cpp
unsigned ARMMCCodeEmitter::
getT2SORegOpValue(const MCInst &MI, unsigned OpIdx,
                SmallVectorImpl<MCFixup> &Fixups,
                const MCSubtargetInfo &STI) const {
  // Sub-operands are [reg, imm]. The first register is Rm, the reg to be
  // shifted. The second is the amount to shift by.
  //
  // {3-0} = Rm.
  // {4}   = 0
  // {6-5} = type
  // {11-7} = imm
```
- EN: Implements `getT2SORegOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getT2SORegOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1728-1730
```cpp
  const MCOperand &MO  = MI.getOperand(OpIdx);
  const MCOperand &MO1 = MI.getOperand(OpIdx + 1);
  ARM_AM::ShiftOpc SOpc = ARM_AM::getSORegShOp(MO1.getImm());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1732-1733
```cpp
  // Encode Rm.
  unsigned Binary = CTX.getRegisterInfo()->getEncodingValue(MO.getReg());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1735-1749
```cpp
  // Encode the shift opcode.
  unsigned SBits = 0;
  // Set shift operand (bit[6:4]).
  // LSL - 000
  // LSR - 010
  // ASR - 100
  // ROR - 110
  switch (SOpc) {
  default: llvm_unreachable("Unknown shift opc!");
  case ARM_AM::lsl: SBits = 0x0; break;
  case ARM_AM::lsr: SBits = 0x2; break;
  case ARM_AM::asr: SBits = 0x4; break;
  case ARM_AM::rrx: [[fallthrough]];
  case ARM_AM::ror: SBits = 0x6; break;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1751-1753
```cpp
  Binary |= SBits << 4;
  if (SOpc == ARM_AM::rrx)
    return Binary;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1755-1757
```cpp
  // Encode shift_imm bit[11:7].
  return Binary | ARM_AM::getSORegOffset(MO1.getImm()) << 7;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1759-1771
```cpp
unsigned ARMMCCodeEmitter::
getBitfieldInvertedMaskOpValue(const MCInst &MI, unsigned Op,
                               SmallVectorImpl<MCFixup> &Fixups,
                               const MCSubtargetInfo &STI) const {
  // 10 bits. lower 5 bits are the lsb of the mask, high five bits are the
  // msb of the mask.
  const MCOperand &MO = MI.getOperand(Op);
  uint32_t v = ~MO.getImm();
  uint32_t lsb = llvm::countr_zero(v);
  uint32_t msb = llvm::Log2_32(v);
  assert(v != 0 && lsb < 32 && msb < 32 && "Illegal bitfield mask!");
  return lsb | (msb << 5);
}
```
- EN: Implements `getBitfieldInvertedMaskOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getBitfieldInvertedMaskOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1773-1785
```cpp
unsigned ARMMCCodeEmitter::
getRegisterListOpValue(const MCInst &MI, unsigned Op,
                       SmallVectorImpl<MCFixup> &Fixups,
                       const MCSubtargetInfo &STI) const {
  // VLDM/VSTM/VSCCLRM:
  //   {12-8} = Vd
  //   {7-0}  = Number of registers
  //
  // LDM/STM:
  //   {15-0}  = Bitfield of GPRs.
  MCRegister Reg = MI.getOperand(Op).getReg();
  bool SPRRegs = ARMMCRegisterClasses[ARM::SPRRegClassID].contains(Reg);
  bool DPRRegs = ARMMCRegisterClasses[ARM::DPRRegClassID].contains(Reg);
```
- EN: Implements `getRegisterListOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getRegisterListOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1787-1787
```cpp
  unsigned Binary = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1789-1793
```cpp
  if (SPRRegs || DPRRegs || Reg == ARM::VPR) {
    // VLDM/VSTM/VSCCLRM
    unsigned RegNo = CTX.getRegisterInfo()->getEncodingValue(Reg);
    unsigned NumRegs = (MI.getNumOperands() - Op) & 0xff;
    Binary |= (RegNo & 0x1f) << 8;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1795-1809
```cpp
    if (MI.getOpcode() == ARM::VSCCLRMD)
      // Ignore VPR
      --NumRegs;
    else if (MI.getOpcode() == ARM::VSCCLRMS) {
      // The register list can contain both S registers and D registers, with D
      // registers counting as two registers. VPR doesn't count towards the
      // number of registers.
      NumRegs = 0;
      for (unsigned I = Op, E = MI.getNumOperands(); I < E; ++I) {
        Reg = MI.getOperand(I).getReg();
        if (ARMMCRegisterClasses[ARM::SPRRegClassID].contains(Reg))
          NumRegs += 1;
        else if (ARMMCRegisterClasses[ARM::DPRRegClassID].contains(Reg))
          NumRegs += 2;
      }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1810-1824
```cpp
    }
    if (SPRRegs)
      Binary |= NumRegs;
    else
      Binary |= NumRegs * 2;
  } else {
    const MCRegisterInfo &MRI = *CTX.getRegisterInfo();
    assert(is_sorted(drop_begin(MI, Op),
                     [&](const MCOperand &LHS, const MCOperand &RHS) {
                       return MRI.getEncodingValue(LHS.getReg()) <
                              MRI.getEncodingValue(RHS.getReg());
                     }));
    for (unsigned I = Op, E = MI.getNumOperands(); I < E; ++I) {
      unsigned RegNo = MRI.getEncodingValue(MI.getOperand(I).getReg());
      Binary |= 1 << RegNo;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1825-1826
```cpp
    }
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1828-1829
```cpp
  return Binary;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1831-1838
```cpp
/// getAddrMode6AddressOpValue - Encode an addrmode6 register number along
/// with the alignment operand.
unsigned ARMMCCodeEmitter::
getAddrMode6AddressOpValue(const MCInst &MI, unsigned Op,
                           SmallVectorImpl<MCFixup> &Fixups,
                           const MCSubtargetInfo &STI) const {
  const MCOperand &Reg = MI.getOperand(Op);
  const MCOperand &Imm = MI.getOperand(Op + 1);
```
- EN: Implements `getAddrMode6AddressOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getAddrMode6AddressOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1840-1841
```cpp
  unsigned RegNo = CTX.getRegisterInfo()->getEncodingValue(Reg.getReg());
  unsigned Align = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1843-1850
```cpp
  switch (Imm.getImm()) {
  default: break;
  case 2:
  case 4:
  case 8:  Align = 0x01; break;
  case 16: Align = 0x02; break;
  case 32: Align = 0x03; break;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1852-1853
```cpp
  return RegNo | (Align << 4);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1855-1862
```cpp
/// getAddrMode6OneLane32AddressOpValue - Encode an addrmode6 register number
/// along  with the alignment operand for use in VST1 and VLD1 with size 32.
unsigned ARMMCCodeEmitter::
getAddrMode6OneLane32AddressOpValue(const MCInst &MI, unsigned Op,
                                    SmallVectorImpl<MCFixup> &Fixups,
                                    const MCSubtargetInfo &STI) const {
  const MCOperand &Reg = MI.getOperand(Op);
  const MCOperand &Imm = MI.getOperand(Op + 1);
```
- EN: Implements `getAddrMode6OneLane32AddressOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getAddrMode6OneLane32AddressOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1864-1865
```cpp
  unsigned RegNo = CTX.getRegisterInfo()->getEncodingValue(Reg.getReg());
  unsigned Align = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1867-1874
```cpp
  switch (Imm.getImm()) {
  default: break;
  case 8:
  case 16:
  case 32: // Default '0' value for invalid alignments of 8, 16, 32 bytes.
  case 2: Align = 0x00; break;
  case 4: Align = 0x03; break;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1876-1877
```cpp
  return RegNo | (Align << 4);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1880-1889
```cpp
/// getAddrMode6DupAddressOpValue - Encode an addrmode6 register number and
/// alignment operand for use in VLD-dup instructions.  This is the same as
/// getAddrMode6AddressOpValue except for the alignment encoding, which is
/// different for VLD4-dup.
unsigned ARMMCCodeEmitter::
getAddrMode6DupAddressOpValue(const MCInst &MI, unsigned Op,
                              SmallVectorImpl<MCFixup> &Fixups,
                              const MCSubtargetInfo &STI) const {
  const MCOperand &Reg = MI.getOperand(Op);
  const MCOperand &Imm = MI.getOperand(Op + 1);
```
- EN: Implements `getAddrMode6DupAddressOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getAddrMode6DupAddressOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1891-1892
```cpp
  unsigned RegNo = CTX.getRegisterInfo()->getEncodingValue(Reg.getReg());
  unsigned Align = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1894-1900
```cpp
  switch (Imm.getImm()) {
  default: break;
  case 2:
  case 4:
  case 8:  Align = 0x01; break;
  case 16: Align = 0x03; break;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1902-1903
```cpp
  return RegNo | (Align << 4);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1905-1913
```cpp
unsigned ARMMCCodeEmitter::
getAddrMode6OffsetOpValue(const MCInst &MI, unsigned Op,
                          SmallVectorImpl<MCFixup> &Fixups,
                          const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(Op);
  if (!MO.getReg())
    return 0x0D;
  return CTX.getRegisterInfo()->getEncodingValue(MO.getReg());
}
```
- EN: Implements `getAddrMode6OffsetOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getAddrMode6OffsetOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1915-1920
```cpp
unsigned ARMMCCodeEmitter::
getShiftRight8Imm(const MCInst &MI, unsigned Op,
                  SmallVectorImpl<MCFixup> &Fixups,
                  const MCSubtargetInfo &STI) const {
  return 8 - MI.getOperand(Op).getImm();
}
```
- EN: Implements `getShiftRight8Imm`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getShiftRight8Imm`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1922-1927
```cpp
unsigned ARMMCCodeEmitter::
getShiftRight16Imm(const MCInst &MI, unsigned Op,
                   SmallVectorImpl<MCFixup> &Fixups,
                   const MCSubtargetInfo &STI) const {
  return 16 - MI.getOperand(Op).getImm();
}
```
- EN: Implements `getShiftRight16Imm`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getShiftRight16Imm`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1929-1934
```cpp
unsigned ARMMCCodeEmitter::
getShiftRight32Imm(const MCInst &MI, unsigned Op,
                   SmallVectorImpl<MCFixup> &Fixups,
                   const MCSubtargetInfo &STI) const {
  return 32 - MI.getOperand(Op).getImm();
}
```
- EN: Implements `getShiftRight32Imm`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getShiftRight32Imm`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1936-1941
```cpp
unsigned ARMMCCodeEmitter::
getShiftRight64Imm(const MCInst &MI, unsigned Op,
                   SmallVectorImpl<MCFixup> &Fixups,
                   const MCSubtargetInfo &STI) const {
  return 64 - MI.getOperand(Op).getImm();
}
```
- EN: Implements `getShiftRight64Imm`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getShiftRight64Imm`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1943-1951
```cpp
void ARMMCCodeEmitter::encodeInstruction(const MCInst &MI,
                                         SmallVectorImpl<char> &CB,
                                         SmallVectorImpl<MCFixup> &Fixups,
                                         const MCSubtargetInfo &STI) const {
  // Pseudo instructions don't get encoded.
  const MCInstrDesc &Desc = MCII.get(MI.getOpcode());
  uint64_t TSFlags = Desc.TSFlags;
  if ((TSFlags & ARMII::FormMask) == ARMII::Pseudo)
    return;
```
- EN: Implements `ARMMCCodeEmitter::encodeInstruction`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMMCCodeEmitter::encodeInstruction`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1953-1957
```cpp
  int Size;
  if (Desc.getSize() == 2 || Desc.getSize() == 4)
    Size = Desc.getSize();
  else
    llvm_unreachable("Unexpected instruction size!");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1959-1973
```cpp
  auto Endian =
      IsLittleEndian ? llvm::endianness::little : llvm::endianness::big;
  uint32_t Binary = getBinaryCodeForInstr(MI, Fixups, STI);
  if (Size == 2) {
    support::endian::write<uint16_t>(CB, Binary, Endian);
  } else if (isThumb(STI)) {
    // Thumb 32-bit wide instructions need to emit the high order halfword
    // first.
    support::endian::write<uint16_t>(CB, Binary >> 16, Endian);
    support::endian::write<uint16_t>(CB, Binary & 0xffff, Endian);
  } else {
    support::endian::write<uint32_t>(CB, Binary, Endian);
  }
  ++MCNumEmitted;  // Keep track of the # of mi's emitted.
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1975-1984
```cpp
template <bool isNeg, ARM::Fixups fixup>
uint32_t
ARMMCCodeEmitter::getBFTargetOpValue(const MCInst &MI, unsigned OpIdx,
                                     SmallVectorImpl<MCFixup> &Fixups,
                                     const MCSubtargetInfo &STI) const {
  const MCOperand MO = MI.getOperand(OpIdx);
  if (MO.isExpr())
    return ::getBranchTargetOpValue(MI, OpIdx, fixup, Fixups, STI);
  return isNeg ? -(MO.getImm() >> 1) : (MO.getImm() >> 1);
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1986-1991
```cpp
uint32_t
ARMMCCodeEmitter::getBFAfterTargetOpValue(const MCInst &MI, unsigned OpIdx,
                                          SmallVectorImpl<MCFixup> &Fixups,
                                          const MCSubtargetInfo &STI) const {
  const MCOperand MO = MI.getOperand(OpIdx);
  const MCOperand BranchMO = MI.getOperand(0);
```
- EN: Implements `ARMMCCodeEmitter::getBFAfterTargetOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMMCCodeEmitter::getBFAfterTargetOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1993-2000
```cpp
  if (MO.isExpr()) {
    assert(BranchMO.isExpr());
    const MCExpr *DiffExpr = MCBinaryExpr::createSub(
        MO.getExpr(), BranchMO.getExpr(), CTX);
    MCFixupKind Kind = ARM::fixup_bfcsel_else_target;
    addFixup(Fixups, 0, DiffExpr, Kind);
    return 0;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2002-2004
```cpp
  assert(MO.isImm() && BranchMO.isImm());
  int Diff = MO.getImm() - BranchMO.getImm();
  assert(Diff == 4 || Diff == 2);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2006-2007
```cpp
  return Diff == 4;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2009-2013
```cpp
uint32_t ARMMCCodeEmitter::getVPTMaskOpValue(const MCInst &MI, unsigned OpIdx,
                                             SmallVectorImpl<MCFixup> &Fixups,
                                             const MCSubtargetInfo &STI)const {
  const MCOperand MO = MI.getOperand(OpIdx);
  assert(MO.isImm() && "Unexpected operand type!");
```
- EN: Implements `ARMMCCodeEmitter::getVPTMaskOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMMCCodeEmitter::getVPTMaskOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2015-2016
```cpp
  int Value = MO.getImm();
  int Imm = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2018-2022
```cpp
  // VPT Masks are actually encoded as a series of invert/don't invert bits,
  // rather than true/false bits.
  unsigned PrevBit = 0;
  for (int i = 3; i >= 0; --i) {
    unsigned Bit = (Value >> i) & 1;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2024-2028
```cpp
    // Check if we are at the end of the mask.
    if ((Value & ~(~0U << i)) == 0) {
      Imm |= (1 << i);
      break;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2030-2032
```cpp
    // Convert the bit in the mask based on the previous bit.
    if (Bit != PrevBit)
      Imm |= (1 << i);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2034-2035
```cpp
    PrevBit = Bit;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2037-2038
```cpp
  return Imm;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2040-2042
```cpp
uint32_t ARMMCCodeEmitter::getRestrictedCondCodeOpValue(
    const MCInst &MI, unsigned OpIdx, SmallVectorImpl<MCFixup> &Fixups,
    const MCSubtargetInfo &STI) const {
```
- EN: Implements `ARMMCCodeEmitter::getRestrictedCondCodeOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMMCCodeEmitter::getRestrictedCondCodeOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2044-2045
```cpp
  const MCOperand MO = MI.getOperand(OpIdx);
  assert(MO.isImm() && "Unexpected operand type!");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2047-2061
```cpp
  switch (MO.getImm()) {
  default:
    assert(0 && "Unexpected Condition!");
    return 0;
  case ARMCC::HS:
  case ARMCC::EQ:
    return 0;
  case ARMCC::HI:
  case ARMCC::NE:
    return 1;
  case ARMCC::GE:
    return 4;
  case ARMCC::LT:
    return 5;
  case ARMCC::GT:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2062-2066
```cpp
    return 6;
  case ARMCC::LE:
    return 7;
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2068-2075
```cpp
uint32_t ARMMCCodeEmitter::
getPowerTwoOpValue(const MCInst &MI, unsigned OpIdx,
                   SmallVectorImpl<MCFixup> &Fixups,
                   const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(OpIdx);
  assert(MO.isImm() && "Unexpected operand type!");
  return llvm::countr_zero((uint64_t)MO.getImm());
}
```
- EN: Implements `getPowerTwoOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getPowerTwoOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2077-2083
```cpp
template <unsigned start>
uint32_t ARMMCCodeEmitter::
getMVEPairVectorIndexOpValue(const MCInst &MI, unsigned OpIdx,
                             SmallVectorImpl<MCFixup> &Fixups,
                             const MCSubtargetInfo &STI) const {
  const MCOperand MO = MI.getOperand(OpIdx);
  assert(MO.isImm() && "Unexpected operand type!");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2085-2087
```cpp
  int Value = MO.getImm();
  return Value - start;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2089-2089
```cpp
#include "ARMGenMCCodeEmitter.inc"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 2091-2094
```cpp
MCCodeEmitter *llvm::createARMLEMCCodeEmitter(const MCInstrInfo &MCII,
                                              MCContext &Ctx) {
  return new ARMMCCodeEmitter(MCII, Ctx, true);
}
```
- EN: Implements `llvm::createARMLEMCCodeEmitter`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `llvm::createARMLEMCCodeEmitter`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2096-2099
```cpp
MCCodeEmitter *llvm::createARMBEMCCodeEmitter(const MCInstrInfo &MCII,
                                              MCContext &Ctx) {
  return new ARMMCCodeEmitter(MCII, Ctx, false);
}
```
- EN: Implements `llvm::createARMBEMCCodeEmitter`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `llvm::createARMBEMCCodeEmitter`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

## Key Concepts / 关键概念

- EN: Primary role: machine-code encoding emission.
  - CN: 核心职责：机器码编码输出。
- EN: The MC layer is used to print, encode, or stream target instructions and symbols.
  - CN: MC 层被用于打印、编码或流式输出目标指令与符号。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。

## Dependencies / 依赖关系

- EN: Backend-local headers: `MCTargetDesc/ARMAddressingModes.h`, `MCTargetDesc/ARMBaseInfo.h`, `MCTargetDesc/ARMFixupKinds.h`, `MCTargetDesc/ARMMCAsmInfo.h`, `ARMGenMCCodeEmitter.inc`.
  - CN: 后端本地头文件：`MCTargetDesc/ARMAddressingModes.h`, `MCTargetDesc/ARMBaseInfo.h`, `MCTargetDesc/ARMFixupKinds.h`, `MCTargetDesc/ARMMCAsmInfo.h`, `ARMGenMCCodeEmitter.inc`。
- EN: LLVM infrastructure headers: `llvm/ADT/APFloat.h`, `llvm/ADT/APInt.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/Statistic.h`, `llvm/MC/MCCodeEmitter.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCFixup.h` ... (+10 more).
  - CN: LLVM 基础设施头文件：`llvm/ADT/APFloat.h`, `llvm/ADT/APInt.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/Statistic.h`, `llvm/MC/MCCodeEmitter.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCFixup.h` ... (+10 more)。
- EN: Standard/system headers: `cassert`, `cstdint`, `cstdlib`.
  - CN: 标准库/系统头文件：`cassert`, `cstdint`, `cstdlib`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
- EN: The implementation also depends on LLVM MC layer classes for final assembly/object emission.
  - CN: 实现还依赖 LLVM 的 MC 层类来完成最终汇编/目标文件输出。
