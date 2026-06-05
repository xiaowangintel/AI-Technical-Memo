# Thumb2SizeReduction.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/Thumb2SizeReduction.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `Thumb2SizeReduction` for the ARM backend, focusing on target-specific code-generation behavior.
- 用途 (CN): 实现 ARM 后端中的 `Thumb2SizeReduction`，重点处理目标相关代码生成行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- Thumb2SizeReduction.cpp - Thumb2 code size reduction pass -*- C++ -*-=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 9-23
```cpp
#include "ARM.h"
#include "ARMBaseInstrInfo.h"
#include "ARMSubtarget.h"
#include "MCTargetDesc/ARMBaseInfo.h"
#include "Thumb2InstrInfo.h"
#include "llvm/ADT/DenseMap.h"
#include "llvm/ADT/PostOrderIterator.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/SmallSet.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/CodeGen/MachineBasicBlock.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 24-38
```cpp
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineOperand.h"
#include "llvm/CodeGen/TargetInstrInfo.h"
#include "llvm/IR/DebugLoc.h"
#include "llvm/IR/Function.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/MC/MCInstrDesc.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/Compiler.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/raw_ostream.h"
#include <cassert>
#include <cstdint>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 39-41
```cpp
#include <functional>
#include <iterator>
#include <utility>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 43-43
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 45-46
```cpp
#define DEBUG_TYPE "thumb2-reduce-size"
#define THUMB2_SIZE_REDUCE_NAME "Thumb2 instruction size reduce pass"
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 48-50
```cpp
STATISTIC(NumNarrows,  "Number of 32-bit instrs reduced to 16-bit ones");
STATISTIC(Num2Addrs,   "Number of 32-bit instrs reduced to 2addr 16-bit ones");
STATISTIC(NumLdSts,    "Number of 32-bit load / store reduced to 16-bit ones");
```
- EN: Declares `STATISTIC`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `STATISTIC`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 52-57
```cpp
static cl::opt<int> ReduceLimit("t2-reduce-limit",
                                cl::init(-1), cl::Hidden);
static cl::opt<int> ReduceLimit2Addr("t2-reduce-limit2",
                                     cl::init(-1), cl::Hidden);
static cl::opt<int> ReduceLimitLdSt("t2-reduce-limit3",
                                     cl::init(-1), cl::Hidden);
```
- EN: Declares `cl::init`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `cl::init`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 59-59
```cpp
namespace {
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 61-75
```cpp
  /// ReduceTable - A static table with information on mapping from wide
  /// opcodes to narrow
  struct ReduceEntry {
    uint16_t WideOpc;      // Wide opcode
    uint16_t NarrowOpc1;   // Narrow opcode to transform to
    uint16_t NarrowOpc2;   // Narrow opcode when it's two-address
    uint8_t  Imm1Limit;    // Limit of immediate field (bits)
    uint8_t  Imm2Limit;    // Limit of immediate field when it's two-address
    unsigned LowRegs1 : 1; // Only possible if low-registers are used
    unsigned LowRegs2 : 1; // Only possible if low-registers are used (2addr)
    unsigned PredCC1  : 2; // 0 - If predicated, cc is on and vice versa.
                           // 1 - No cc field.
                           // 2 - Always set CPSR.
    unsigned PredCC2  : 2;
    unsigned PartFlag : 1; // 16-bit instruction does partial flag update
```
- EN: Declares `ReduceEntry`, packaging target-specific state and APIs around `Thumb2SizeReduction`.
- CN: 这里声明 `ReduceEntry`，把与 `Thumb2SizeReduction` 相关的目标特定状态和 API 组织在一起。

### Lines 76-78
```cpp
    unsigned Special  : 1; // Needs to be dealt with specially
    unsigned AvoidMovs: 1; // Avoid movs with shifter operand (for Swift)
  };
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 80-94
```cpp
  static const ReduceEntry ReduceTable[] = {
  // Wide,        Narrow1,      Narrow2,     imm1,imm2, lo1, lo2, P/C,PF,S,AM
  { ARM::t2ADCrr, 0,            ARM::tADC,     0,   0,   0,   1,  0,0, 0,0,0 },
  { ARM::t2ADDri, ARM::tADDi3,  ARM::tADDi8,   3,   8,   1,   1,  0,0, 0,1,0 },
  { ARM::t2ADDrr, ARM::tADDrr,  ARM::tADDhirr, 0,   0,   1,   0,  0,1, 0,0,0 },
  { ARM::t2ADDSri,ARM::tADDi3,  ARM::tADDi8,   3,   8,   1,   1,  2,2, 0,1,0 },
  { ARM::t2ADDSrr,ARM::tADDrr,  0,             0,   0,   1,   0,  2,0, 0,1,0 },
  { ARM::t2ANDrr, 0,            ARM::tAND,     0,   0,   0,   1,  0,0, 1,0,0 },
  { ARM::t2ASRri, ARM::tASRri,  0,             5,   0,   1,   0,  0,0, 1,0,1 },
  { ARM::t2ASRrr, 0,            ARM::tASRrr,   0,   0,   0,   1,  0,0, 1,0,1 },
  { ARM::t2BICrr, 0,            ARM::tBIC,     0,   0,   0,   1,  0,0, 1,0,0 },
  { ARM::t2CMNrr, ARM::tCMN,    0,             0,   0,   1,   0,  2,0, 0,0,0 },
  { ARM::t2CMPri, ARM::tCMPi8,  0,             8,   0,   1,   0,  2,0, 0,0,0 },
  { ARM::t2CMPrr, ARM::tCMPhir, 0,             0,   0,   0,   0,  2,0, 0,1,0 },
  { ARM::t2EORrr, 0,            ARM::tEOR,     0,   0,   0,   1,  0,0, 1,0,0 },
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 95-109
```cpp
  // FIXME: adr.n immediate offset must be multiple of 4.
  //{ ARM::t2LEApcrelJT,ARM::tLEApcrelJT, 0,   0,   0,   1,   0,  1,0, 0,0,0 },
  { ARM::t2LSLri, ARM::tLSLri,  0,             5,   0,   1,   0,  0,0, 1,0,1 },
  { ARM::t2LSLrr, 0,            ARM::tLSLrr,   0,   0,   0,   1,  0,0, 1,0,1 },
  { ARM::t2LSRri, ARM::tLSRri,  0,             5,   0,   1,   0,  0,0, 1,0,1 },
  { ARM::t2LSRrr, 0,            ARM::tLSRrr,   0,   0,   0,   1,  0,0, 1,0,1 },
  { ARM::t2MOVi,  ARM::tMOVi8,  0,             8,   0,   1,   0,  0,0, 1,0,0 },
  { ARM::t2MOVi16,ARM::tMOVi8,  0,             8,   0,   1,   0,  0,0, 1,1,0 },
  // FIXME: Do we need the 16-bit 'S' variant?
  { ARM::t2MOVr,ARM::tMOVr,     0,             0,   0,   0,   0,  1,0, 0,0,0 },
  { ARM::t2MUL,   0,            ARM::tMUL,     0,   0,   0,   1,  0,0, 1,0,0 },
  { ARM::t2MVNr,  ARM::tMVN,    0,             0,   0,   1,   0,  0,0, 0,0,0 },
  { ARM::t2ORRrr, 0,            ARM::tORR,     0,   0,   0,   1,  0,0, 1,0,0 },
  { ARM::t2REV,   ARM::tREV,    0,             0,   0,   1,   0,  1,0, 0,0,0 },
  { ARM::t2REV16, ARM::tREV16,  0,             0,   0,   1,   0,  1,0, 0,0,0 },
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 110-124
```cpp
  { ARM::t2REVSH, ARM::tREVSH,  0,             0,   0,   1,   0,  1,0, 0,0,0 },
  { ARM::t2RORrr, 0,            ARM::tROR,     0,   0,   0,   1,  0,0, 1,0,0 },
  { ARM::t2RSBri, ARM::tRSB,    0,             0,   0,   1,   0,  0,0, 0,1,0 },
  { ARM::t2RSBSri,ARM::tRSB,    0,             0,   0,   1,   0,  2,0, 0,1,0 },
  { ARM::t2SBCrr, 0,            ARM::tSBC,     0,   0,   0,   1,  0,0, 0,0,0 },
  { ARM::t2SUBri, ARM::tSUBi3,  ARM::tSUBi8,   3,   8,   1,   1,  0,0, 0,0,0 },
  { ARM::t2SUBrr, ARM::tSUBrr,  0,             0,   0,   1,   0,  0,0, 0,0,0 },
  { ARM::t2SUBSri,ARM::tSUBi3,  ARM::tSUBi8,   3,   8,   1,   1,  2,2, 0,0,0 },
  { ARM::t2SUBSrr,ARM::tSUBrr,  0,             0,   0,   1,   0,  2,0, 0,0,0 },
  { ARM::t2SXTB,  ARM::tSXTB,   0,             0,   0,   1,   0,  1,0, 0,1,0 },
  { ARM::t2SXTH,  ARM::tSXTH,   0,             0,   0,   1,   0,  1,0, 0,1,0 },
  { ARM::t2TEQrr, ARM::tEOR,    0,             0,   0,   1,   0,  2,0, 0,1,0 },
  { ARM::t2TSTrr, ARM::tTST,    0,             0,   0,   1,   0,  2,0, 0,0,0 },
  { ARM::t2UXTB,  ARM::tUXTB,   0,             0,   0,   1,   0,  1,0, 0,1,0 },
  { ARM::t2UXTH,  ARM::tUXTH,   0,             0,   0,   1,   0,  1,0, 0,1,0 },
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 126-140
```cpp
  // FIXME: Clean this up after splitting each Thumb load / store opcode
  // into multiple ones.
  { ARM::t2LDRi12,ARM::tLDRi,   ARM::tLDRspi,  5,   8,   1,   0,  0,0, 0,1,0 },
  { ARM::t2LDRs,  ARM::tLDRr,   0,             0,   0,   1,   0,  0,0, 0,1,0 },
  { ARM::t2LDRBi12,ARM::tLDRBi, 0,             5,   0,   1,   0,  0,0, 0,1,0 },
  { ARM::t2LDRBs, ARM::tLDRBr,  0,             0,   0,   1,   0,  0,0, 0,1,0 },
  { ARM::t2LDRHi12,ARM::tLDRHi, 0,             5,   0,   1,   0,  0,0, 0,1,0 },
  { ARM::t2LDRHs, ARM::tLDRHr,  0,             0,   0,   1,   0,  0,0, 0,1,0 },
  { ARM::t2LDRSBs,ARM::tLDRSB,  0,             0,   0,   1,   0,  0,0, 0,1,0 },
  { ARM::t2LDRSHs,ARM::tLDRSH,  0,             0,   0,   1,   0,  0,0, 0,1,0 },
  { ARM::t2LDR_POST,ARM::tLDMIA_UPD,0,         0,   0,   1,   0,  0,0, 0,1,0 },
  { ARM::t2STRi12,ARM::tSTRi,   ARM::tSTRspi,  5,   8,   1,   0,  0,0, 0,1,0 },
  { ARM::t2STRs,  ARM::tSTRr,   0,             0,   0,   1,   0,  0,0, 0,1,0 },
  { ARM::t2STRBi12,ARM::tSTRBi, 0,             5,   0,   1,   0,  0,0, 0,1,0 },
  { ARM::t2STRBs, ARM::tSTRBr,  0,             0,   0,   1,   0,  0,0, 0,1,0 },
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 141-143
```cpp
  { ARM::t2STRHi12,ARM::tSTRHi, 0,             5,   0,   1,   0,  0,0, 0,1,0 },
  { ARM::t2STRHs, ARM::tSTRHr,  0,             0,   0,   1,   0,  0,0, 0,1,0 },
  { ARM::t2STR_POST,ARM::tSTMIA_UPD,0,         0,   0,   1,   0,  0,0, 0,1,0 },
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 145-154
```cpp
  { ARM::t2LDMIA, ARM::tLDMIA,  0,             0,   0,   1,   1,  1,1, 0,1,0 },
  { ARM::t2LDMIA_RET,0,         ARM::tPOP_RET, 0,   0,   1,   1,  1,1, 0,1,0 },
  { ARM::t2LDMIA_UPD,ARM::tLDMIA_UPD,ARM::tPOP,0,   0,   1,   1,  1,1, 0,1,0 },
  // ARM::t2STMIA (with no basereg writeback) has no Thumb1 equivalent.
  // tSTMIA_UPD is a change in semantics which can only be used if the base
  // register is killed. This difference is correctly handled elsewhere.
  { ARM::t2STMIA, ARM::tSTMIA_UPD, 0,          0,   0,   1,   1,  1,1, 0,1,0 },
  { ARM::t2STMIA_UPD,ARM::tSTMIA_UPD, 0,       0,   0,   1,   1,  1,1, 0,1,0 },
  { ARM::t2STMDB_UPD, 0,        ARM::tPUSH,    0,   0,   1,   1,  1,1, 0,1,0 }
  };
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 156-158
```cpp
  class Thumb2SizeReduce : public MachineFunctionPass {
  public:
    static char ID;
```
- EN: Declares `Thumb2SizeReduce`, packaging target-specific state and APIs around `Thumb2SizeReduction`.
- CN: 这里声明 `Thumb2SizeReduce`，把与 `Thumb2SizeReduction` 相关的目标特定状态和 API 组织在一起。

### Lines 160-161
```cpp
    const Thumb2InstrInfo *TII;
    const ARMSubtarget *STI;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 163-163
```cpp
    Thumb2SizeReduce(std::function<bool(const Function &)> Ftor = nullptr);
```
- EN: Declares `Thumb2SizeReduce`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `Thumb2SizeReduce`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 165-165
```cpp
    bool runOnMachineFunction(MachineFunction &MF) override;
```
- EN: Declares `runOnMachineFunction`, a pass-entry routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `runOnMachineFunction`，它是一个围绕机器函数状态展开的Pass 入口例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 167-169
```cpp
    MachineFunctionProperties getRequiredProperties() const override {
      return MachineFunctionProperties().setNoVRegs();
    }
```
- EN: Implements `getRequiredProperties`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getRequiredProperties`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 171-173
```cpp
    StringRef getPassName() const override {
      return THUMB2_SIZE_REDUCE_NAME;
    }
```
- EN: Implements `getPassName`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getPassName`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 175-177
```cpp
  private:
    /// ReduceOpcodeMap - Maps wide opcode to index of entry in ReduceTable.
    DenseMap<unsigned, unsigned> ReduceOpcodeMap;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 179-179
```cpp
    bool canAddPseudoFlagDep(MachineInstr *Use, bool IsSelfLoop);
```
- EN: Declares `canAddPseudoFlagDep`, a query/helper routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `canAddPseudoFlagDep`，它是一个围绕机器指令展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 181-183
```cpp
    bool VerifyPredAndCC(MachineInstr *MI, const ReduceEntry &Entry,
                         bool is2Addr, ARMCC::CondCodes Pred,
                         bool LiveCPSR, bool &HasCC, bool &CCDead);
```
- EN: Declares `VerifyPredAndCC`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `VerifyPredAndCC`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 185-186
```cpp
    bool ReduceLoadStore(MachineBasicBlock &MBB, MachineInstr *MI,
                         const ReduceEntry &Entry);
```
- EN: Declares `ReduceLoadStore`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ReduceLoadStore`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 188-189
```cpp
    bool ReduceSpecial(MachineBasicBlock &MBB, MachineInstr *MI,
                       const ReduceEntry &Entry, bool LiveCPSR, bool IsSelfLoop);
```
- EN: Declares `ReduceSpecial`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ReduceSpecial`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 191-195
```cpp
    /// ReduceTo2Addr - Reduce a 32-bit instruction to a 16-bit two-address
    /// instruction.
    bool ReduceTo2Addr(MachineBasicBlock &MBB, MachineInstr *MI,
                       const ReduceEntry &Entry, bool LiveCPSR,
                       bool IsSelfLoop);
```
- EN: Declares `ReduceTo2Addr`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ReduceTo2Addr`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 197-201
```cpp
    /// ReduceToNarrow - Reduce a 32-bit instruction to a 16-bit
    /// non-two-address instruction.
    bool ReduceToNarrow(MachineBasicBlock &MBB, MachineInstr *MI,
                        const ReduceEntry &Entry, bool LiveCPSR,
                        bool IsSelfLoop);
```
- EN: Declares `ReduceToNarrow`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ReduceToNarrow`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 203-205
```cpp
    /// ReduceMI - Attempt to reduce MI, return true on success.
    bool ReduceMI(MachineBasicBlock &MBB, MachineInstr *MI, bool LiveCPSR,
                  bool IsSelfLoop, bool SkipPrologueEpilogue);
```
- EN: Declares `ReduceMI`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ReduceMI`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 207-208
```cpp
    /// ReduceMBB - Reduce width of instructions in the specified basic block.
    bool ReduceMBB(MachineBasicBlock &MBB, bool SkipPrologueEpilogue);
```
- EN: Declares `ReduceMBB`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ReduceMBB`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 210-211
```cpp
    bool OptimizeSize;
    bool MinimizeSize;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 213-217
```cpp
    // Last instruction to define CPSR in the current block.
    MachineInstr *CPSRDef;
    // Was CPSR last defined by a high latency instruction?
    // When CPSRDef is null, this refers to CPSR defs in predecessors.
    bool HighLatencyCPSR;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 219-223
```cpp
    struct MBBInfo {
      // The flags leaving this block have high latency.
      bool HighLatencyCPSR = false;
      // Has this block been visited yet?
      bool Visited = false;
```
- EN: Declares `MBBInfo`, packaging target-specific state and APIs around `Thumb2SizeReduction`.
- CN: 这里声明 `MBBInfo`，把与 `Thumb2SizeReduction` 相关的目标特定状态和 API 组织在一起。

### Lines 225-226
```cpp
      MBBInfo() = default;
    };
```
- EN: Declares `MBBInfo`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `MBBInfo`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 228-228
```cpp
    SmallVector<MBBInfo, 8> BlockInfo;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 230-231
```cpp
    std::function<bool(const Function &)> PredicateFtor;
  };
```
- EN: Declares `bool`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `bool`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 233-233
```cpp
  char Thumb2SizeReduce::ID = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 235-235
```cpp
} // end anonymous namespace
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 237-238
```cpp
INITIALIZE_PASS(Thumb2SizeReduce, DEBUG_TYPE, THUMB2_SIZE_REDUCE_NAME, false,
                false)
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 240-248
```cpp
Thumb2SizeReduce::Thumb2SizeReduce(std::function<bool(const Function &)> Ftor)
    : MachineFunctionPass(ID), PredicateFtor(std::move(Ftor)) {
  OptimizeSize = MinimizeSize = false;
  for (unsigned i = 0, e = std::size(ReduceTable); i != e; ++i) {
    unsigned FromOpc = ReduceTable[i].WideOpc;
    if (!ReduceOpcodeMap.insert(std::make_pair(FromOpc, i)).second)
      llvm_unreachable("Duplicated entries?");
  }
}
```
- EN: Implements `Thumb2SizeReduce::Thumb2SizeReduce`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Thumb2SizeReduce::Thumb2SizeReduce`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 250-252
```cpp
static bool HasImplicitCPSRDef(const MCInstrDesc &MCID) {
  return is_contained(MCID.implicit_defs(), ARM::CPSR);
}
```
- EN: Implements `HasImplicitCPSRDef`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `HasImplicitCPSRDef`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 254-262
```cpp
// Check for a likely high-latency flag def.
static bool isHighLatencyCPSR(MachineInstr *Def) {
  switch(Def->getOpcode()) {
  case ARM::FMSTAT:
  case ARM::tMUL:
    return true;
  }
  return false;
}
```
- EN: Implements `isHighLatencyCPSR`, a query/helper routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isHighLatencyCPSR`，它是一个围绕机器指令展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 264-278
```cpp
/// canAddPseudoFlagDep - For A9 (and other out-of-order) implementations,
/// the 's' 16-bit instruction partially update CPSR. Abort the
/// transformation to avoid adding false dependency on last CPSR setting
/// instruction which hurts the ability for out-of-order execution engine
/// to do register renaming magic.
/// This function checks if there is a read-of-write dependency between the
/// last instruction that defines the CPSR and the current instruction. If there
/// is, then there is no harm done since the instruction cannot be retired
/// before the CPSR setting instruction anyway.
/// Note, we are not doing full dependency analysis here for the sake of compile
/// time. We're not looking for cases like:
/// r0 = muls ...
/// r1 = add.w r0, ...
/// ...
///    = mul.w r1
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 279-285
```cpp
/// In this case it would have been ok to narrow the mul.w to muls since there
/// are indirect RAW dependency between the muls and the mul.w
bool
Thumb2SizeReduce::canAddPseudoFlagDep(MachineInstr *Use, bool FirstInSelfLoop) {
  // Disable the check for -Oz (aka OptimizeForSizeHarder).
  if (MinimizeSize || !STI->avoidCPSRPartialUpdate())
    return false;
```
- EN: Implements `Thumb2SizeReduce::canAddPseudoFlagDep`, a query/helper routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Thumb2SizeReduce::canAddPseudoFlagDep`，它是一个围绕机器指令展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 287-290
```cpp
  if (!CPSRDef)
    // If this BB loops back to itself, conservatively avoid narrowing the
    // first instruction that does partial flag update.
    return HighLatencyCPSR || FirstInSelfLoop;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 292-300
```cpp
  SmallSet<unsigned, 2> Defs;
  for (const MachineOperand &MO : CPSRDef->operands()) {
    if (!MO.isReg() || MO.isUndef() || MO.isUse())
      continue;
    Register Reg = MO.getReg();
    if (Reg == 0 || Reg == ARM::CPSR)
      continue;
    Defs.insert(Reg);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 302-308
```cpp
  for (const MachineOperand &MO : Use->operands()) {
    if (!MO.isReg() || MO.isUndef() || MO.isDef())
      continue;
    Register Reg = MO.getReg();
    if (Defs.count(Reg))
      return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 310-312
```cpp
  // If the current CPSR has high latency, try to avoid the false dependency.
  if (HighLatencyCPSR)
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 314-318
```cpp
  // tMOVi8 usually doesn't start long dependency chains, and there are a lot
  // of them, so always shrink them when CPSR doesn't have high latency.
  if (Use->getOpcode() == ARM::t2MOVi ||
      Use->getOpcode() == ARM::t2MOVi16)
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 320-322
```cpp
  // No read-after-write dependency. The narrowing will add false dependency.
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 324-338
```cpp
bool
Thumb2SizeReduce::VerifyPredAndCC(MachineInstr *MI, const ReduceEntry &Entry,
                                  bool is2Addr, ARMCC::CondCodes Pred,
                                  bool LiveCPSR, bool &HasCC, bool &CCDead) {
  if ((is2Addr  && Entry.PredCC2 == 0) ||
      (!is2Addr && Entry.PredCC1 == 0)) {
    if (Pred == ARMCC::AL) {
      // Not predicated, must set CPSR.
      if (!HasCC) {
        // Original instruction was not setting CPSR, but CPSR is not
        // currently live anyway. It's ok to set it. The CPSR def is
        // dead though.
        if (!LiveCPSR) {
          HasCC = true;
          CCDead = true;
```
- EN: Implements `Thumb2SizeReduce::VerifyPredAndCC`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Thumb2SizeReduce::VerifyPredAndCC`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 339-353
```cpp
          return true;
        }
        return false;
      }
    } else {
      // Predicated, must not set CPSR.
      if (HasCC)
        return false;
    }
  } else if ((is2Addr  && Entry.PredCC2 == 2) ||
             (!is2Addr && Entry.PredCC1 == 2)) {
    /// Old opcode has an optional def of CPSR.
    if (HasCC)
      return true;
    // If old opcode does not implicitly define CPSR, then it's not ok since
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 354-362
```cpp
    // these new opcodes' CPSR def is not meant to be thrown away. e.g. CMP.
    if (!HasImplicitCPSRDef(MI->getDesc()))
      return false;
    HasCC = true;
  } else {
    // 16-bit instruction does not set CPSR.
    if (HasCC)
      return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 364-365
```cpp
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 367-381
```cpp
static bool VerifyLowRegs(MachineInstr *MI) {
  unsigned Opc = MI->getOpcode();
  bool isPCOk = (Opc == ARM::t2LDMIA_RET || Opc == ARM::t2LDMIA_UPD);
  bool isLROk = (Opc == ARM::t2STMDB_UPD);
  bool isSPOk = isPCOk || isLROk;
  for (unsigned i = 0, e = MI->getNumOperands(); i != e; ++i) {
    const MachineOperand &MO = MI->getOperand(i);
    if (!MO.isReg() || MO.isImplicit())
      continue;
    Register Reg = MO.getReg();
    if (Reg == 0 || Reg == ARM::CPSR)
      continue;
    if (isPCOk && Reg == ARM::PC)
      continue;
    if (isLROk && Reg == ARM::LR)
```
- EN: Implements `VerifyLowRegs`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `VerifyLowRegs`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 382-394
```cpp
      continue;
    if (Reg == ARM::SP) {
      if (isSPOk)
        continue;
      if (i == 1 && (Opc == ARM::t2LDRi12 || Opc == ARM::t2STRi12))
        // Special case for these ldr / str with sp as base register.
        continue;
    }
    if (!isARMLowRegister(Reg))
      return false;
  }
  return true;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 396-400
```cpp
bool
Thumb2SizeReduce::ReduceLoadStore(MachineBasicBlock &MBB, MachineInstr *MI,
                                  const ReduceEntry &Entry) {
  if (ReduceLimitLdSt != -1 && ((int)NumLdSts >= ReduceLimitLdSt))
    return false;
```
- EN: Implements `Thumb2SizeReduce::ReduceLoadStore`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Thumb2SizeReduce::ReduceLoadStore`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 402-409
```cpp
  unsigned Scale = 1;
  bool HasImmOffset = false;
  bool HasShift = false;
  bool HasOffReg = true;
  bool isLdStMul = false;
  unsigned Opc = Entry.NarrowOpc1;
  unsigned OpNum = 3; // First 'rest' of operands.
  uint8_t  ImmLimit = Entry.Imm1Limit;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 411-419
```cpp
  switch (Entry.WideOpc) {
  default:
    llvm_unreachable("Unexpected Thumb2 load / store opcode!");
  case ARM::t2LDRi12:
  case ARM::t2STRi12:
    if (MI->getOperand(1).getReg() == ARM::SP) {
      Opc = Entry.NarrowOpc2;
      ImmLimit = Entry.Imm2Limit;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 421-435
```cpp
    Scale = 4;
    HasImmOffset = true;
    HasOffReg = false;
    break;
  case ARM::t2LDRBi12:
  case ARM::t2STRBi12:
    HasImmOffset = true;
    HasOffReg = false;
    break;
  case ARM::t2LDRHi12:
  case ARM::t2STRHi12:
    Scale = 2;
    HasImmOffset = true;
    HasOffReg = false;
    break;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 436-450
```cpp
  case ARM::t2LDRs:
  case ARM::t2LDRBs:
  case ARM::t2LDRHs:
  case ARM::t2LDRSBs:
  case ARM::t2LDRSHs:
  case ARM::t2STRs:
  case ARM::t2STRBs:
  case ARM::t2STRHs:
    HasShift = true;
    OpNum = 4;
    break;
  case ARM::t2LDR_POST:
  case ARM::t2STR_POST: {
    if (!MinimizeSize)
      return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 452-454
```cpp
    if (!MI->hasOneMemOperand() ||
        (*MI->memoperands_begin())->getAlign() < Align(4))
      return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 456-466
```cpp
    // We're creating a completely different type of load/store - LDM from LDR.
    // For this reason we can't reuse the logic at the end of this function; we
    // have to implement the MI building here.
    bool IsStore = Entry.WideOpc == ARM::t2STR_POST;
    Register Rt = MI->getOperand(IsStore ? 1 : 0).getReg();
    Register Rn = MI->getOperand(IsStore ? 0 : 1).getReg();
    unsigned Offset = MI->getOperand(3).getImm();
    unsigned PredImm = MI->getOperand(4).getImm();
    Register PredReg = MI->getOperand(5).getReg();
    assert(isARMLowRegister(Rt));
    assert(isARMLowRegister(Rn));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 468-469
```cpp
    if (Offset != 4)
      return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 471-478
```cpp
    // Add the 16-bit load / store instruction.
    DebugLoc dl = MI->getDebugLoc();
    auto MIB = BuildMI(MBB, MI, dl, TII->get(Entry.NarrowOpc1))
                   .addReg(Rn, RegState::Define)
                   .addReg(Rn)
                   .addImm(PredImm)
                   .addReg(PredReg)
                   .addReg(Rt, getDefRegState(!IsStore));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 480-481
```cpp
    // Transfer memoperands.
    MIB.setMemRefs(MI->memoperands());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 483-484
```cpp
    // Transfer MI flags.
    MIB.setMIFlags(MI->getFlags());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 486-493
```cpp
    // Kill the old instruction.
    MI->eraseFromBundle();
    ++NumLdSts;
    return true;
  }
  case ARM::t2LDMIA: {
    Register BaseReg = MI->getOperand(0).getReg();
    assert(isARMLowRegister(BaseReg));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 495-503
```cpp
    // For the non-writeback version (this one), the base register must be
    // one of the registers being loaded.
    bool isOK = false;
    for (const MachineOperand &MO : llvm::drop_begin(MI->operands(), 3)) {
      if (MO.getReg() == BaseReg) {
        isOK = true;
        break;
      }
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 505-506
```cpp
    if (!isOK)
      return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 508-517
```cpp
    OpNum = 0;
    isLdStMul = true;
    break;
  }
  case ARM::t2STMIA: {
    // t2STMIA is reduced to tSTMIA_UPD which has writeback. We can only do this
    // if the base register is killed, as then it doesn't matter what its value
    // is after the instruction.
    if (!MI->getOperand(0).isKill())
      return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 519-525
```cpp
    // If the base register is in the register list and isn't the lowest
    // numbered register (i.e. it's in operand 4 onwards) then with writeback
    // the stored value is unknown, so we can't convert to tSTMIA_UPD.
    Register BaseReg = MI->getOperand(0).getReg();
    for (const MachineOperand &MO : llvm::drop_begin(MI->operands(), 4))
      if (MO.getReg() == BaseReg)
        return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 527-541
```cpp
    break;
  }
  case ARM::t2LDMIA_RET: {
    Register BaseReg = MI->getOperand(1).getReg();
    if (BaseReg != ARM::SP)
      return false;
    Opc = Entry.NarrowOpc2; // tPOP_RET
    OpNum = 2;
    isLdStMul = true;
    break;
  }
  case ARM::t2LDMIA_UPD:
  case ARM::t2STMIA_UPD:
  case ARM::t2STMDB_UPD: {
    OpNum = 0;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 543-553
```cpp
    Register BaseReg = MI->getOperand(1).getReg();
    if (BaseReg == ARM::SP &&
        (Entry.WideOpc == ARM::t2LDMIA_UPD ||
         Entry.WideOpc == ARM::t2STMDB_UPD)) {
      Opc = Entry.NarrowOpc2; // tPOP or tPUSH
      OpNum = 2;
    } else if (!isARMLowRegister(BaseReg) ||
               (Entry.WideOpc != ARM::t2LDMIA_UPD &&
                Entry.WideOpc != ARM::t2STMIA_UPD)) {
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 555-558
```cpp
    isLdStMul = true;
    break;
  }
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 560-566
```cpp
  unsigned OffsetReg = 0;
  bool OffsetKill = false;
  bool OffsetInternal = false;
  if (HasShift) {
    OffsetReg  = MI->getOperand(2).getReg();
    OffsetKill = MI->getOperand(2).isKill();
    OffsetInternal = MI->getOperand(2).isInternalRead();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 568-571
```cpp
    if (MI->getOperand(3).getImm())
      // Thumb1 addressing mode doesn't support shift.
      return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 573-576
```cpp
  unsigned OffsetImm = 0;
  if (HasImmOffset) {
    OffsetImm = MI->getOperand(2).getImm();
    unsigned MaxOffset = ((1 << ImmLimit) - 1) * Scale;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 578-581
```cpp
    if ((OffsetImm & (Scale - 1)) || OffsetImm > MaxOffset)
      // Make sure the immediate field fits.
      return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 583-585
```cpp
  // Add the 16-bit load / store instruction.
  DebugLoc dl = MI->getDebugLoc();
  MachineInstrBuilder MIB = BuildMI(MBB, MI, dl, TII->get(Opc));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 587-590
```cpp
  // tSTMIA_UPD takes a defining register operand. We've already checked that
  // the register is killed, so mark it as dead here.
  if (Entry.WideOpc == ARM::t2STMIA)
    MIB.addReg(MI->getOperand(0).getReg(), RegState::Define | RegState::Dead);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 592-594
```cpp
  if (!isLdStMul) {
    MIB.add(MI->getOperand(0));
    MIB.add(MI->getOperand(1));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 596-597
```cpp
    if (HasImmOffset)
      MIB.addImm(OffsetImm / Scale);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 599-599
```cpp
    assert((!HasShift || OffsetReg) && "Invalid so_reg load / store address!");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 601-604
```cpp
    if (HasOffReg)
      MIB.addReg(OffsetReg, getKillRegState(OffsetKill) |
                            getInternalReadRegState(OffsetInternal));
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 606-608
```cpp
  // Transfer the rest of operands.
  for (const MachineOperand &MO : llvm::drop_begin(MI->operands(), OpNum))
    MIB.add(MO);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 610-611
```cpp
  // Transfer memoperands.
  MIB.setMemRefs(MI->memoperands());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 613-614
```cpp
  // Transfer MI flags.
  MIB.setMIFlags(MI->getFlags());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 616-617
```cpp
  LLVM_DEBUG(dbgs() << "Converted 32-bit: " << *MI
                    << "       to 16-bit: " << *MIB);
```
- EN: Declares `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 619-622
```cpp
  MBB.erase_instr(MI);
  ++NumLdSts;
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 624-638
```cpp
bool
Thumb2SizeReduce::ReduceSpecial(MachineBasicBlock &MBB, MachineInstr *MI,
                                const ReduceEntry &Entry,
                                bool LiveCPSR, bool IsSelfLoop) {
  unsigned Opc = MI->getOpcode();
  if (Opc == ARM::t2ADDri) {
    // If the source register is SP, try to reduce to tADDrSPi, otherwise
    // it's a normal reduce.
    if (MI->getOperand(1).getReg() != ARM::SP) {
      if (ReduceTo2Addr(MBB, MI, Entry, LiveCPSR, IsSelfLoop))
        return true;
      return ReduceToNarrow(MBB, MI, Entry, LiveCPSR, IsSelfLoop);
    }
    // Try to reduce to tADDrSPi.
    unsigned Imm = MI->getOperand(2).getImm();
```
- EN: Implements `Thumb2SizeReduce::ReduceSpecial`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Thumb2SizeReduce::ReduceSpecial`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 639-651
```cpp
    // The immediate must be in range, the destination register must be a low
    // reg, the predicate must be "always" and the condition flags must not
    // be being set.
    if (Imm & 3 || Imm > 1020)
      return false;
    if (!isARMLowRegister(MI->getOperand(0).getReg()))
      return false;
    if (MI->getOperand(3).getImm() != ARMCC::AL)
      return false;
    const MCInstrDesc &MCID = MI->getDesc();
    if (MCID.hasOptionalDef() &&
        MI->getOperand(MCID.getNumOperands()-1).getReg() == ARM::CPSR)
      return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 653-659
```cpp
    MachineInstrBuilder MIB =
        BuildMI(MBB, MI, MI->getDebugLoc(),
                TII->get(ARM::tADDrSPi))
            .add(MI->getOperand(0))
            .add(MI->getOperand(1))
            .addImm(Imm / 4) // The tADDrSPi has an implied scale by four.
            .add(predOps(ARMCC::AL));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 661-662
```cpp
    // Transfer MI flags.
    MIB.setMIFlags(MI->getFlags());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 664-665
```cpp
    LLVM_DEBUG(dbgs() << "Converted 32-bit: " << *MI
                      << "       to 16-bit: " << *MIB);
```
- EN: Declares `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 667-670
```cpp
    MBB.erase_instr(MI);
    ++NumNarrows;
    return true;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 672-673
```cpp
  if (Entry.LowRegs1 && !VerifyLowRegs(MI))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 675-676
```cpp
  if (MI->mayLoadOrStore())
    return ReduceLoadStore(MBB, MI, Entry);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 678-692
```cpp
  switch (Opc) {
  default: break;
  case ARM::t2ADDSri:
  case ARM::t2ADDSrr: {
    Register PredReg;
    if (getInstrPredicate(*MI, PredReg) == ARMCC::AL) {
      switch (Opc) {
      default: break;
      case ARM::t2ADDSri:
        if (ReduceTo2Addr(MBB, MI, Entry, LiveCPSR, IsSelfLoop))
          return true;
        [[fallthrough]];
      case ARM::t2ADDSrr:
        return ReduceToNarrow(MBB, MI, Entry, LiveCPSR, IsSelfLoop);
      }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 693-707
```cpp
    }
    break;
  }
  case ARM::t2RSBri:
  case ARM::t2RSBSri:
  case ARM::t2SXTB:
  case ARM::t2SXTH:
  case ARM::t2UXTB:
  case ARM::t2UXTH:
    if (MI->getOperand(2).getImm() == 0)
      return ReduceToNarrow(MBB, MI, Entry, LiveCPSR, IsSelfLoop);
    break;
  case ARM::t2MOVi16:
    // Can convert only 'pure' immediate operands, not immediates obtained as
    // globals' addresses.
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 708-722
```cpp
    if (MI->getOperand(1).isImm())
      return ReduceToNarrow(MBB, MI, Entry, LiveCPSR, IsSelfLoop);
    break;
  case ARM::t2CMPrr: {
    // Try to reduce to the lo-reg only version first. Why there are two
    // versions of the instruction is a mystery.
    // It would be nice to just have two entries in the main table that
    // are prioritized, but the table assumes a unique entry for each
    // source insn opcode. So for now, we hack a local entry record to use.
    static const ReduceEntry NarrowEntry =
      { ARM::t2CMPrr,ARM::tCMPr, 0, 0, 0, 1, 1,2, 0, 0,1,0 };
    if (ReduceToNarrow(MBB, MI, NarrowEntry, LiveCPSR, IsSelfLoop))
      return true;
    return ReduceToNarrow(MBB, MI, Entry, LiveCPSR, IsSelfLoop);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 723-735
```cpp
  case ARM::t2TEQrr: {
    Register PredReg;
    // Can only convert to eors if we're not in an IT block.
    if (getInstrPredicate(*MI, PredReg) != ARMCC::AL)
      break;
    // TODO if Operand 0 is not killed but Operand 1 is, then we could write
    // to Op1 instead.
    if (MI->getOperand(0).isKill())
      return ReduceToNarrow(MBB, MI, Entry, LiveCPSR, IsSelfLoop);
  }
  }
  return false;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 737-742
```cpp
bool
Thumb2SizeReduce::ReduceTo2Addr(MachineBasicBlock &MBB, MachineInstr *MI,
                                const ReduceEntry &Entry,
                                bool LiveCPSR, bool IsSelfLoop) {
  if (ReduceLimit2Addr != -1 && ((int)Num2Addrs >= ReduceLimit2Addr))
    return false;
```
- EN: Implements `Thumb2SizeReduce::ReduceTo2Addr`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Thumb2SizeReduce::ReduceTo2Addr`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 744-747
```cpp
  if (!OptimizeSize && Entry.AvoidMovs && STI->avoidMOVsShifterOperand())
    // Don't issue movs with shifter operand for some CPUs unless we
    // are optimizing for size.
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 749-763
```cpp
  Register Reg0 = MI->getOperand(0).getReg();
  Register Reg1 = MI->getOperand(1).getReg();
  // t2MUL is "special". The tied source operand is second, not first.
  if (MI->getOpcode() == ARM::t2MUL) {
    // MULS can be slower than MUL
    if (!MinimizeSize && STI->avoidMULS())
      return false;
    Register Reg2 = MI->getOperand(2).getReg();
    // Early exit if the regs aren't all low regs.
    if (!isARMLowRegister(Reg0) || !isARMLowRegister(Reg1)
        || !isARMLowRegister(Reg2))
      return false;
    if (Reg0 != Reg2) {
      // If the other operand also isn't the same as the destination, we
      // can't reduce.
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 764-778
```cpp
      if (Reg1 != Reg0)
        return false;
      // Try to commute the operands to make it a 2-address instruction.
      MachineInstr *CommutedMI = TII->commuteInstruction(*MI);
      if (!CommutedMI)
        return false;
    }
  } else if (Reg0 != Reg1) {
    // Try to commute the operands to make it a 2-address instruction.
    unsigned CommOpIdx1 = 1;
    unsigned CommOpIdx2 = TargetInstrInfo::CommuteAnyOperandIndex;
    if (!TII->findCommutedOpIndices(*MI, CommOpIdx1, CommOpIdx2) ||
        MI->getOperand(CommOpIdx2).getReg() != Reg0)
      return false;
    MachineInstr *CommutedMI =
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 779-793
```cpp
        TII->commuteInstruction(*MI, false, CommOpIdx1, CommOpIdx2);
    if (!CommutedMI)
      return false;
  }
  if (Entry.LowRegs2 && !isARMLowRegister(Reg0))
    return false;
  if (Entry.Imm2Limit) {
    unsigned Imm = MI->getOperand(2).getImm();
    unsigned Limit = (1 << Entry.Imm2Limit) - 1;
    if (Imm > Limit)
      return false;
  } else {
    Register Reg2 = MI->getOperand(2).getReg();
    if (Entry.LowRegs2 && !isARMLowRegister(Reg2))
      return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 794-794
```cpp
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 796-807
```cpp
  // Check if it's possible / necessary to transfer the predicate.
  const MCInstrDesc &NewMCID = TII->get(Entry.NarrowOpc2);
  Register PredReg;
  ARMCC::CondCodes Pred = getInstrPredicate(*MI, PredReg);
  bool SkipPred = false;
  if (Pred != ARMCC::AL) {
    if (!NewMCID.isPredicable())
      // Can't transfer predicate, fail.
      return false;
  } else {
    SkipPred = !NewMCID.isPredicable();
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 809-819
```cpp
  bool HasCC = false;
  bool CCDead = false;
  const MCInstrDesc &MCID = MI->getDesc();
  if (MCID.hasOptionalDef()) {
    unsigned NumOps = MCID.getNumOperands();
    HasCC = (MI->getOperand(NumOps-1).getReg() == ARM::CPSR);
    if (HasCC && MI->getOperand(NumOps-1).isDead())
      CCDead = true;
  }
  if (!VerifyPredAndCC(MI, Entry, true, Pred, LiveCPSR, HasCC, CCDead))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 821-825
```cpp
  // Avoid adding a false dependency on partial flag update by some 16-bit
  // instructions which has the 's' bit set.
  if (Entry.PartFlag && NewMCID.hasOptionalDef() && HasCC &&
      canAddPseudoFlagDep(MI, IsSelfLoop))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 827-832
```cpp
  // Add the 16-bit instruction.
  DebugLoc dl = MI->getDebugLoc();
  MachineInstrBuilder MIB = BuildMI(MBB, MI, dl, NewMCID);
  MIB.add(MI->getOperand(0));
  if (NewMCID.hasOptionalDef())
    MIB.add(HasCC ? t1CondCodeOp(CCDead) : condCodeOp());
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 834-842
```cpp
  // Transfer the rest of operands.
  unsigned NumOps = MCID.getNumOperands();
  for (unsigned i = 1, e = MI->getNumOperands(); i != e; ++i) {
    if (i < NumOps && MCID.operands()[i].isOptionalDef())
      continue;
    if (SkipPred && MCID.operands()[i].isPredicate())
      continue;
    MIB.add(MI->getOperand(i));
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 844-845
```cpp
  // Transfer MI flags.
  MIB.setMIFlags(MI->getFlags());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 847-848
```cpp
  LLVM_DEBUG(dbgs() << "Converted 32-bit: " << *MI
                    << "       to 16-bit: " << *MIB);
```
- EN: Declares `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 850-853
```cpp
  MBB.erase_instr(MI);
  ++Num2Addrs;
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 855-860
```cpp
bool
Thumb2SizeReduce::ReduceToNarrow(MachineBasicBlock &MBB, MachineInstr *MI,
                                 const ReduceEntry &Entry,
                                 bool LiveCPSR, bool IsSelfLoop) {
  if (ReduceLimit != -1 && ((int)NumNarrows >= ReduceLimit))
    return false;
```
- EN: Implements `Thumb2SizeReduce::ReduceToNarrow`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Thumb2SizeReduce::ReduceToNarrow`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 862-865
```cpp
  if (!OptimizeSize && Entry.AvoidMovs && STI->avoidMOVsShifterOperand())
    // Don't issue movs with shifter operand for some CPUs unless we
    // are optimizing for size.
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 867-869
```cpp
  unsigned Limit = ~0U;
  if (Entry.Imm1Limit)
    Limit = (1 << Entry.Imm1Limit) - 1;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 871-885
```cpp
  const MCInstrDesc &MCID = MI->getDesc();
  for (unsigned i = 0, e = MCID.getNumOperands(); i != e; ++i) {
    if (MCID.operands()[i].isPredicate())
      continue;
    const MachineOperand &MO = MI->getOperand(i);
    if (MO.isReg()) {
      Register Reg = MO.getReg();
      if (!Reg || Reg == ARM::CPSR)
        continue;
      if (Entry.LowRegs1 && !isARMLowRegister(Reg))
        return false;
    } else if (MO.isImm() && !MCID.operands()[i].isPredicate()) {
      if (((unsigned)MO.getImm()) > Limit)
        return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 886-886
```cpp
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 888-899
```cpp
  // Check if it's possible / necessary to transfer the predicate.
  const MCInstrDesc &NewMCID = TII->get(Entry.NarrowOpc1);
  Register PredReg;
  ARMCC::CondCodes Pred = getInstrPredicate(*MI, PredReg);
  bool SkipPred = false;
  if (Pred != ARMCC::AL) {
    if (!NewMCID.isPredicable())
      // Can't transfer predicate, fail.
      return false;
  } else {
    SkipPred = !NewMCID.isPredicable();
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 901-910
```cpp
  bool HasCC = false;
  bool CCDead = false;
  if (MCID.hasOptionalDef()) {
    unsigned NumOps = MCID.getNumOperands();
    HasCC = (MI->getOperand(NumOps-1).getReg() == ARM::CPSR);
    if (HasCC && MI->getOperand(NumOps-1).isDead())
      CCDead = true;
  }
  if (!VerifyPredAndCC(MI, Entry, false, Pred, LiveCPSR, HasCC, CCDead))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 912-916
```cpp
  // Avoid adding a false dependency on partial flag update by some 16-bit
  // instructions which has the 's' bit set.
  if (Entry.PartFlag && NewMCID.hasOptionalDef() && HasCC &&
      canAddPseudoFlagDep(MI, IsSelfLoop))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 918-920
```cpp
  // Add the 16-bit instruction.
  DebugLoc dl = MI->getDebugLoc();
  MachineInstrBuilder MIB = BuildMI(MBB, MI, dl, NewMCID);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 922-929
```cpp
  // TEQ is special in that it doesn't define a register but we're converting
  // it into an EOR which does. So add the first operand as a def and then
  // again as a use.
  if (MCID.getOpcode() == ARM::t2TEQrr) {
    MIB.add(MI->getOperand(0));
    MIB->getOperand(0).setIsKill(false);
    MIB->getOperand(0).setIsDef(true);
    MIB->getOperand(0).setIsDead(true);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 931-938
```cpp
    if (NewMCID.hasOptionalDef())
      MIB.add(HasCC ? t1CondCodeOp(CCDead) : condCodeOp());
    MIB.add(MI->getOperand(0));
  } else {
    MIB.add(MI->getOperand(0));
    if (NewMCID.hasOptionalDef())
      MIB.add(HasCC ? t1CondCodeOp(CCDead) : condCodeOp());
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 940-954
```cpp
  // Transfer the rest of operands.
  unsigned NumOps = MCID.getNumOperands();
  for (unsigned i = 1, e = MI->getNumOperands(); i != e; ++i) {
    if (i < NumOps && MCID.operands()[i].isOptionalDef())
      continue;
    if ((MCID.getOpcode() == ARM::t2RSBSri ||
         MCID.getOpcode() == ARM::t2RSBri ||
         MCID.getOpcode() == ARM::t2SXTB ||
         MCID.getOpcode() == ARM::t2SXTH ||
         MCID.getOpcode() == ARM::t2UXTB ||
         MCID.getOpcode() == ARM::t2UXTH) && i == 2)
      // Skip the zero immediate operand, it's now implicit.
      continue;
    bool isPred = (i < NumOps && MCID.operands()[i].isPredicate());
    if (SkipPred && isPred)
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 955-964
```cpp
        continue;
    const MachineOperand &MO = MI->getOperand(i);
    if (MO.isReg() && MO.isImplicit() && MO.getReg() == ARM::CPSR)
      // Skip implicit def of CPSR. Either it's modeled as an optional
      // def now or it's already an implicit def on the new instruction.
      continue;
    MIB.add(MO);
  }
  if (!MCID.isPredicable() && NewMCID.isPredicable())
    MIB.add(predOps(ARMCC::AL));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 966-967
```cpp
  // Transfer MI flags.
  MIB.setMIFlags(MI->getFlags());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 969-970
```cpp
  LLVM_DEBUG(dbgs() << "Converted 32-bit: " << *MI
                    << "       to 16-bit: " << *MIB);
```
- EN: Declares `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 972-975
```cpp
  MBB.erase_instr(MI);
  ++NumNarrows;
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 977-983
```cpp
static bool UpdateCPSRDef(MachineInstr &MI, bool LiveCPSR, bool &DefCPSR) {
  bool HasDef = false;
  for (const MachineOperand &MO : MI.operands()) {
    if (!MO.isReg() || MO.isUndef() || MO.isUse())
      continue;
    if (MO.getReg() != ARM::CPSR)
      continue;
```
- EN: Implements `UpdateCPSRDef`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `UpdateCPSRDef`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 985-988
```cpp
    DefCPSR = true;
    if (!MO.isDead())
      HasDef = true;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 990-991
```cpp
  return HasDef || LiveCPSR;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 993-1004
```cpp
static bool UpdateCPSRUse(MachineInstr &MI, bool LiveCPSR) {
  for (const MachineOperand &MO : MI.operands()) {
    if (!MO.isReg() || MO.isUndef() || MO.isDef())
      continue;
    if (MO.getReg() != ARM::CPSR)
      continue;
    assert(LiveCPSR && "CPSR liveness tracking is wrong!");
    if (MO.isKill()) {
      LiveCPSR = false;
      break;
    }
  }
```
- EN: Implements `UpdateCPSRUse`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `UpdateCPSRUse`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1006-1007
```cpp
  return LiveCPSR;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1009-1019
```cpp
bool Thumb2SizeReduce::ReduceMI(MachineBasicBlock &MBB, MachineInstr *MI,
                                bool LiveCPSR, bool IsSelfLoop,
                                bool SkipPrologueEpilogue) {
  unsigned Opcode = MI->getOpcode();
  auto OPI = ReduceOpcodeMap.find(Opcode);
  if (OPI == ReduceOpcodeMap.end())
    return false;
  if (SkipPrologueEpilogue && (MI->getFlag(MachineInstr::FrameSetup) ||
                               MI->getFlag(MachineInstr::FrameDestroy)))
    return false;
  const ReduceEntry &Entry = ReduceTable[OPI->second];
```
- EN: Implements `Thumb2SizeReduce::ReduceMI`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Thumb2SizeReduce::ReduceMI`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1021-1023
```cpp
  // Don't attempt normal reductions on "special" cases for now.
  if (Entry.Special)
    return ReduceSpecial(MBB, MI, Entry, LiveCPSR, IsSelfLoop);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1025-1028
```cpp
  // Try to transform to a 16-bit two-address instruction.
  if (Entry.NarrowOpc2 &&
      ReduceTo2Addr(MBB, MI, Entry, LiveCPSR, IsSelfLoop))
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1030-1033
```cpp
  // Try to transform to a 16-bit non-two-address instruction.
  if (Entry.NarrowOpc1 &&
      ReduceToNarrow(MBB, MI, Entry, LiveCPSR, IsSelfLoop))
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1035-1036
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1038-1040
```cpp
bool Thumb2SizeReduce::ReduceMBB(MachineBasicBlock &MBB,
                                 bool SkipPrologueEpilogue) {
  bool Modified = false;
```
- EN: Implements `Thumb2SizeReduce::ReduceMBB`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Thumb2SizeReduce::ReduceMBB`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1042-1044
```cpp
  // Yes, CPSR could be livein.
  bool LiveCPSR = MBB.isLiveIn(ARM::CPSR);
  MachineInstr *BundleMI = nullptr;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1046-1047
```cpp
  CPSRDef = nullptr;
  HighLatencyCPSR = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1049-1060
```cpp
  // Check predecessors for the latest CPSRDef.
  for (auto *Pred : MBB.predecessors()) {
    const MBBInfo &PInfo = BlockInfo[Pred->getNumber()];
    if (!PInfo.Visited) {
      // Since blocks are visited in RPO, this must be a back-edge.
      continue;
    }
    if (PInfo.HighLatencyCPSR) {
      HighLatencyCPSR = true;
      break;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1062-1068
```cpp
  // If this BB loops back to itself, conservatively avoid narrowing the
  // first instruction that does partial flag update.
  bool IsSelfLoop = MBB.isSuccessor(&MBB);
  MachineBasicBlock::instr_iterator MII = MBB.instr_begin(),E = MBB.instr_end();
  MachineBasicBlock::instr_iterator NextMII;
  for (; MII != E; MII = NextMII) {
    NextMII = std::next(MII);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1070-1076
```cpp
    MachineInstr *MI = &*MII;
    if (MI->isBundle()) {
      BundleMI = MI;
      continue;
    }
    if (MI->isDebugInstr())
      continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1078-1078
```cpp
    LiveCPSR = UpdateCPSRUse(*MI, LiveCPSR);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1080-1081
```cpp
    // Does NextMII belong to the same bundle as MI?
    bool NextInSameBundle = NextMII != E && NextMII->isBundledWithPred();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1083-1091
```cpp
    if (ReduceMI(MBB, MI, LiveCPSR, IsSelfLoop, SkipPrologueEpilogue)) {
      Modified = true;
      MachineBasicBlock::instr_iterator I = std::prev(NextMII);
      MI = &*I;
      // Removing and reinserting the first instruction in a bundle will break
      // up the bundle. Fix the bundling if it was broken.
      if (NextInSameBundle && !NextMII->isBundledWithPred())
        NextMII->bundleWithPred();
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1093-1107
```cpp
    if (BundleMI && !NextInSameBundle && MI->isInsideBundle()) {
      // FIXME: Since post-ra scheduler operates on bundles, the CPSR kill
      // marker is only on the BUNDLE instruction. Process the BUNDLE
      // instruction as we finish with the bundled instruction to work around
      // the inconsistency.
      if (BundleMI->killsRegister(ARM::CPSR, /*TRI=*/nullptr))
        LiveCPSR = false;
      MachineOperand *MO =
          BundleMI->findRegisterDefOperand(ARM::CPSR, /*TRI=*/nullptr);
      if (MO && !MO->isDead())
        LiveCPSR = true;
      MO = BundleMI->findRegisterUseOperand(ARM::CPSR, /*TRI=*/nullptr);
      if (MO && !MO->isKill())
        LiveCPSR = true;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1109-1122
```cpp
    bool DefCPSR = false;
    LiveCPSR = UpdateCPSRDef(*MI, LiveCPSR, DefCPSR);
    if (MI->isCall()) {
      // Calls don't really set CPSR.
      CPSRDef = nullptr;
      HighLatencyCPSR = false;
      IsSelfLoop = false;
    } else if (DefCPSR) {
      // This is the last CPSR defining instruction.
      CPSRDef = MI;
      HighLatencyCPSR = isHighLatencyCPSR(CPSRDef);
      IsSelfLoop = false;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1124-1128
```cpp
  MBBInfo &Info = BlockInfo[MBB.getNumber()];
  Info.HighLatencyCPSR = HighLatencyCPSR;
  Info.Visited = true;
  return Modified;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1130-1132
```cpp
bool Thumb2SizeReduce::runOnMachineFunction(MachineFunction &MF) {
  if (PredicateFtor && !PredicateFtor(MF.getFunction()))
    return false;
```
- EN: Implements `Thumb2SizeReduce::runOnMachineFunction`, a pass-entry routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Thumb2SizeReduce::runOnMachineFunction`，它是一个围绕机器函数状态展开的Pass 入口例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1134-1136
```cpp
  STI = &MF.getSubtarget<ARMSubtarget>();
  if (STI->isThumb1Only() || STI->prefers32BitThumb())
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1138-1138
```cpp
  TII = static_cast<const Thumb2InstrInfo *>(STI->getInstrInfo());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1140-1142
```cpp
  // Optimizing / minimizing size? Minimizing size implies optimizing for size.
  OptimizeSize = MF.getFunction().hasOptSize();
  MinimizeSize = STI->hasMinSize();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1144-1145
```cpp
  BlockInfo.clear();
  BlockInfo.resize(MF.getNumBlockIDs());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1147-1156
```cpp
  // Visit blocks in reverse post-order so LastCPSRDef is known for all
  // predecessors.
  ReversePostOrderTraversal<MachineFunction*> RPOT(&MF);
  bool Modified = false;
  bool NeedsWinCFI = MF.getTarget().getMCAsmInfo().usesWindowsCFI() &&
                     MF.getFunction().needsUnwindTableEntry();
  for (MachineBasicBlock *MBB : RPOT)
    Modified |= ReduceMBB(*MBB, /*SkipPrologueEpilogue=*/NeedsWinCFI);
  return Modified;
}
```
- EN: Declares `RPOT`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `RPOT`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1158-1163
```cpp
/// createThumb2SizeReductionPass - Returns an instance of the Thumb2 size
/// reduction pass.
FunctionPass *llvm::createThumb2SizeReductionPass(
    std::function<bool(const Function &)> Ftor) {
  return new Thumb2SizeReduce(std::move(Ftor));
}
```
- EN: Implements `llvm::createThumb2SizeReductionPass`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `llvm::createThumb2SizeReductionPass`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

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

- EN: Backend-local headers: `ARM.h`, `ARMBaseInstrInfo.h`, `ARMSubtarget.h`, `MCTargetDesc/ARMBaseInfo.h`, `Thumb2InstrInfo.h`.
  - CN: 后端本地头文件：`ARM.h`, `ARMBaseInstrInfo.h`, `ARMSubtarget.h`, `MCTargetDesc/ARMBaseInfo.h`, `Thumb2InstrInfo.h`。
- EN: LLVM infrastructure headers: `llvm/ADT/DenseMap.h`, `llvm/ADT/PostOrderIterator.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallSet.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/Statistic.h`, `llvm/ADT/StringRef.h`, `llvm/CodeGen/MachineBasicBlock.h` ... (+15 more).
  - CN: LLVM 基础设施头文件：`llvm/ADT/DenseMap.h`, `llvm/ADT/PostOrderIterator.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallSet.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/Statistic.h`, `llvm/ADT/StringRef.h`, `llvm/CodeGen/MachineBasicBlock.h` ... (+15 more)。
- EN: Standard/system headers: `cassert`, `cstdint`, `functional`, `iterator`, `utility`.
  - CN: 标准库/系统头文件：`cassert`, `cstdint`, `functional`, `iterator`, `utility`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
- EN: The implementation also depends on LLVM MC layer classes for final assembly/object emission.
  - CN: 实现还依赖 LLVM 的 MC 层类来完成最终汇编/目标文件输出。
