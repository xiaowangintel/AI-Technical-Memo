# ARMExpandPseudoInsts.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMExpandPseudoInsts.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file contains a pass that expands pseudo instructions into target instructions to allow proper scheduling, if-conversion, and other late optimizations. This pass should be run after register allocation but before the post-regalloc scheduling pass.
- 用途 (CN): 实现 ARM 后端中的 `ARMExpandPseudoInsts`，重点处理目标相关代码生成行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14
```cpp
//===-- ARMExpandPseudoInsts.cpp - Expand pseudo instructions -------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains a pass that expands pseudo instructions into target
// instructions to allow proper scheduling, if-conversion, and other late
// optimizations. This pass should be run after register allocation but before
// the post-regalloc scheduling pass.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 16-28
```cpp
#include "ARM.h"
#include "ARMBaseInstrInfo.h"
#include "ARMBaseRegisterInfo.h"
#include "ARMConstantPoolValue.h"
#include "ARMMachineFunctionInfo.h"
#include "ARMSubtarget.h"
#include "MCTargetDesc/ARMAddressingModes.h"
#include "llvm/CodeGen/LivePhysRegs.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineJumpTableInfo.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/Support/Debug.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 30-30
```cpp
#include <atomic>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 32-32
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 34-34
```cpp
#define DEBUG_TYPE "arm-pseudo"
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 36-38
```cpp
static cl::opt<bool>
VerifyARMPseudo("verify-arm-pseudo-expand", cl::Hidden,
                cl::desc("Verify machine code after expanding ARM pseudos"));
```
- EN: Declares `cl::desc`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `cl::desc`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 40-40
```cpp
#define ARM_EXPAND_PSEUDO_NAME "ARM pseudo instruction expansion pass"
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 42-46
```cpp
namespace {
  class ARMExpandPseudo : public MachineFunctionPass {
  public:
    static char ID;
    ARMExpandPseudo() : MachineFunctionPass(ID) {}
```
- EN: Declares `ARMExpandPseudo`, packaging target-specific state and APIs around `ARMExpandPseudoInsts`.
- CN: 这里声明 `ARMExpandPseudo`，把与 `ARMExpandPseudoInsts` 相关的目标特定状态和 API 组织在一起。

### Lines 48-51
```cpp
    const ARMBaseInstrInfo *TII;
    const TargetRegisterInfo *TRI;
    const ARMSubtarget *STI;
    ARMFunctionInfo *AFI;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 53-53
```cpp
    bool runOnMachineFunction(MachineFunction &Fn) override;
```
- EN: Declares `runOnMachineFunction`, a pass-entry routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `runOnMachineFunction`，它是一个围绕机器函数状态展开的Pass 入口例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 55-57
```cpp
    MachineFunctionProperties getRequiredProperties() const override {
      return MachineFunctionProperties().setNoVRegs();
    }
```
- EN: Implements `getRequiredProperties`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getRequiredProperties`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 59-61
```cpp
    StringRef getPassName() const override {
      return ARM_EXPAND_PSEUDO_NAME;
    }
```
- EN: Implements `getPassName`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getPassName`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 63-77
```cpp
  private:
    bool ExpandMI(MachineBasicBlock &MBB,
                  MachineBasicBlock::iterator MBBI,
                  MachineBasicBlock::iterator &NextMBBI);
    bool ExpandMBB(MachineBasicBlock &MBB);
    void ExpandVLD(MachineBasicBlock::iterator &MBBI);
    void ExpandVST(MachineBasicBlock::iterator &MBBI);
    void ExpandLaneOp(MachineBasicBlock::iterator &MBBI);
    void ExpandVTBL(MachineBasicBlock::iterator &MBBI,
                    unsigned Opc, bool IsExt);
    void ExpandMQQPRLoadStore(MachineBasicBlock::iterator &MBBI);
    void ExpandTMOV32BitImm(MachineBasicBlock &MBB,
                            MachineBasicBlock::iterator &MBBI);
    void ExpandMOV32BitImm(MachineBasicBlock &MBB,
                           MachineBasicBlock::iterator &MBBI);
```
- EN: Declares `ExpandMI`, a mutation/build routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ExpandMI`，它是一个围绕机器基本块展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 78-92
```cpp
    void CMSEClearGPRegs(MachineBasicBlock &MBB,
                         MachineBasicBlock::iterator MBBI, const DebugLoc &DL,
                         const SmallVectorImpl<unsigned> &ClearRegs,
                         unsigned ClobberReg);
    MachineBasicBlock &CMSEClearFPRegs(MachineBasicBlock &MBB,
                                       MachineBasicBlock::iterator MBBI);
    MachineBasicBlock &CMSEClearFPRegsV8(MachineBasicBlock &MBB,
                                         MachineBasicBlock::iterator MBBI,
                                         const BitVector &ClearRegs);
    MachineBasicBlock &CMSEClearFPRegsV81(MachineBasicBlock &MBB,
                                          MachineBasicBlock::iterator MBBI,
                                          const BitVector &ClearRegs);
    void CMSESaveClearFPRegs(MachineBasicBlock &MBB,
                             MachineBasicBlock::iterator MBBI, DebugLoc &DL,
                             const LivePhysRegs &LiveRegs,
```
- EN: Declares `CMSEClearGPRegs`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `CMSEClearGPRegs`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 93-107
```cpp
                             SmallVectorImpl<unsigned> &AvailableRegs);
    void CMSESaveClearFPRegsV8(MachineBasicBlock &MBB,
                               MachineBasicBlock::iterator MBBI, DebugLoc &DL,
                               const LivePhysRegs &LiveRegs,
                               SmallVectorImpl<unsigned> &ScratchRegs);
    void CMSESaveClearFPRegsV81(MachineBasicBlock &MBB,
                                MachineBasicBlock::iterator MBBI, DebugLoc &DL,
                                const LivePhysRegs &LiveRegs);
    void CMSERestoreFPRegs(MachineBasicBlock &MBB,
                           MachineBasicBlock::iterator MBBI, DebugLoc &DL,
                           SmallVectorImpl<unsigned> &AvailableRegs);
    void CMSERestoreFPRegsV8(MachineBasicBlock &MBB,
                             MachineBasicBlock::iterator MBBI, DebugLoc &DL,
                             SmallVectorImpl<unsigned> &AvailableRegs);
    void CMSERestoreFPRegsV81(MachineBasicBlock &MBB,
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 108-113
```cpp
                              MachineBasicBlock::iterator MBBI, DebugLoc &DL,
                              SmallVectorImpl<unsigned> &AvailableRegs);
    bool ExpandCMP_SWAP(MachineBasicBlock &MBB,
                        MachineBasicBlock::iterator MBBI, unsigned LdrexOp,
                        unsigned StrexOp, unsigned UxtOp,
                        MachineBasicBlock::iterator &NextMBBI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 115-120
```cpp
    bool ExpandCMP_SWAP_64(MachineBasicBlock &MBB,
                           MachineBasicBlock::iterator MBBI,
                           MachineBasicBlock::iterator &NextMBBI);
  };
  char ARMExpandPseudo::ID = 0;
}
```
- EN: Declares `ExpandCMP_SWAP_64`, a mutation/build routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ExpandCMP_SWAP_64`，它是一个围绕机器基本块展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 122-123
```cpp
INITIALIZE_PASS(ARMExpandPseudo, DEBUG_TYPE, ARM_EXPAND_PSEUDO_NAME, false,
                false)
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 125-137
```cpp
namespace {
  // Constants for register spacing in NEON load/store instructions.
  // For quad-register load-lane and store-lane pseudo instructors, the
  // spacing is initially assumed to be EvenDblSpc, and that is changed to
  // OddDblSpc depending on the lane number operand.
  enum NEONRegSpacing {
    SingleSpc,
    SingleLowSpc ,  // Single spacing, low registers, three and four vectors.
    SingleHighQSpc, // Single spacing, high registers, four vectors.
    SingleHighTSpc, // Single spacing, high registers, three vectors.
    EvenDblSpc,
    OddDblSpc
  };
```
- EN: Defines enumeration `NEONRegSpacing` to name backend-specific modes, cases, or flags used elsewhere in the file.
- CN: 这里定义枚举 `NEONRegSpacing`，用于命名本文件后续会使用的后端模式、分支情况或标志。

### Lines 139-153
```cpp
  // Entries for NEON load/store information table.  The table is sorted by
  // PseudoOpc for fast binary-search lookups.
  struct NEONLdStTableEntry {
    uint16_t PseudoOpc;
    uint16_t RealOpc;
    bool IsLoad;
    bool isUpdating;
    bool hasWritebackOperand;
    uint8_t RegSpacing; // One of type NEONRegSpacing
    uint8_t NumRegs; // D registers loaded or stored
    uint8_t RegElts; // elements per D register; used for lane ops
    // FIXME: Temporary flag to denote whether the real instruction takes
    // a single register (like the encoding) or all of the registers in
    // the list (like the asm syntax and the isel DAG). When all definitions
    // are converted to take only the single encoded register, this will
```
- EN: Declares `NEONLdStTableEntry`, packaging target-specific state and APIs around `ARMExpandPseudoInsts`.
- CN: 这里声明 `NEONLdStTableEntry`，把与 `ARMExpandPseudoInsts` 相关的目标特定状态和 API 组织在一起。

### Lines 154-155
```cpp
    // go away.
    bool copyAllListRegs;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 157-169
```cpp
    // Comparison methods for binary search of the table.
    bool operator<(const NEONLdStTableEntry &TE) const {
      return PseudoOpc < TE.PseudoOpc;
    }
    friend bool operator<(const NEONLdStTableEntry &TE, unsigned PseudoOpc) {
      return TE.PseudoOpc < PseudoOpc;
    }
    [[maybe_unused]] friend bool operator<(unsigned PseudoOpc,
                                           const NEONLdStTableEntry &TE) {
      return PseudoOpc < TE.PseudoOpc;
    }
  };
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 171-177
```cpp
static const NEONLdStTableEntry NEONLdStTable[] = {
{ ARM::VLD1LNq16Pseudo,     ARM::VLD1LNd16,     true, false, false, EvenDblSpc, 1, 4 ,true},
{ ARM::VLD1LNq16Pseudo_UPD, ARM::VLD1LNd16_UPD, true, true, true,  EvenDblSpc, 1, 4 ,true},
{ ARM::VLD1LNq32Pseudo,     ARM::VLD1LNd32,     true, false, false, EvenDblSpc, 1, 2 ,true},
{ ARM::VLD1LNq32Pseudo_UPD, ARM::VLD1LNd32_UPD, true, true, true,  EvenDblSpc, 1, 2 ,true},
{ ARM::VLD1LNq8Pseudo,      ARM::VLD1LNd8,      true, false, false, EvenDblSpc, 1, 8 ,true},
{ ARM::VLD1LNq8Pseudo_UPD,  ARM::VLD1LNd8_UPD, true, true, true,  EvenDblSpc, 1, 8 ,true},
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 179-184
```cpp
{ ARM::VLD1d16QPseudo,      ARM::VLD1d16Q,     true,  false, false, SingleSpc,  4, 4 ,false},
{ ARM::VLD1d16QPseudoWB_fixed,  ARM::VLD1d16Qwb_fixed,   true, true, false, SingleSpc,  4, 4 ,false},
{ ARM::VLD1d16QPseudoWB_register,  ARM::VLD1d16Qwb_register, true, true, true, SingleSpc,  4, 4 ,false},
{ ARM::VLD1d16TPseudo,      ARM::VLD1d16T,     true,  false, false, SingleSpc,  3, 4 ,false},
{ ARM::VLD1d16TPseudoWB_fixed,  ARM::VLD1d16Twb_fixed,   true, true, false, SingleSpc,  3, 4 ,false},
{ ARM::VLD1d16TPseudoWB_register,  ARM::VLD1d16Twb_register, true, true, true, SingleSpc,  3, 4 ,false},
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 186-191
```cpp
{ ARM::VLD1d32QPseudo,      ARM::VLD1d32Q,     true,  false, false, SingleSpc,  4, 2 ,false},
{ ARM::VLD1d32QPseudoWB_fixed,  ARM::VLD1d32Qwb_fixed,   true, true, false, SingleSpc,  4, 2 ,false},
{ ARM::VLD1d32QPseudoWB_register,  ARM::VLD1d32Qwb_register, true, true, true, SingleSpc,  4, 2 ,false},
{ ARM::VLD1d32TPseudo,      ARM::VLD1d32T,     true,  false, false, SingleSpc,  3, 2 ,false},
{ ARM::VLD1d32TPseudoWB_fixed,  ARM::VLD1d32Twb_fixed,   true, true, false, SingleSpc,  3, 2 ,false},
{ ARM::VLD1d32TPseudoWB_register,  ARM::VLD1d32Twb_register, true, true, true, SingleSpc,  3, 2 ,false},
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 193-198
```cpp
{ ARM::VLD1d64QPseudo,      ARM::VLD1d64Q,     true,  false, false, SingleSpc,  4, 1 ,false},
{ ARM::VLD1d64QPseudoWB_fixed,  ARM::VLD1d64Qwb_fixed,   true,  true, false, SingleSpc,  4, 1 ,false},
{ ARM::VLD1d64QPseudoWB_register,  ARM::VLD1d64Qwb_register,   true,  true, true, SingleSpc,  4, 1 ,false},
{ ARM::VLD1d64TPseudo,      ARM::VLD1d64T,     true,  false, false, SingleSpc,  3, 1 ,false},
{ ARM::VLD1d64TPseudoWB_fixed,  ARM::VLD1d64Twb_fixed,   true,  true, false, SingleSpc,  3, 1 ,false},
{ ARM::VLD1d64TPseudoWB_register,  ARM::VLD1d64Twb_register, true, true, true,  SingleSpc,  3, 1 ,false},
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 200-205
```cpp
{ ARM::VLD1d8QPseudo,       ARM::VLD1d8Q,      true,  false, false, SingleSpc,  4, 8 ,false},
{ ARM::VLD1d8QPseudoWB_fixed,   ARM::VLD1d8Qwb_fixed,    true,  true, false, SingleSpc,  4, 8 ,false},
{ ARM::VLD1d8QPseudoWB_register,   ARM::VLD1d8Qwb_register,  true, true, true, SingleSpc,  4, 8 ,false},
{ ARM::VLD1d8TPseudo,       ARM::VLD1d8T,      true,  false, false, SingleSpc,  3, 8 ,false},
{ ARM::VLD1d8TPseudoWB_fixed,   ARM::VLD1d8Twb_fixed,    true,  true, false, SingleSpc,  3, 8 ,false},
{ ARM::VLD1d8TPseudoWB_register,   ARM::VLD1d8Twb_register,  true,  true, true, SingleSpc,  3, 8 ,false},
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 207-212
```cpp
{ ARM::VLD1q16HighQPseudo,  ARM::VLD1d16Q,     true,  false, false, SingleHighQSpc,  4, 4 ,false},
{ ARM::VLD1q16HighQPseudo_UPD, ARM::VLD1d16Qwb_fixed,   true,  true, true, SingleHighQSpc,  4, 4 ,false},
{ ARM::VLD1q16HighTPseudo,  ARM::VLD1d16T,     true,  false, false, SingleHighTSpc,  3, 4 ,false},
{ ARM::VLD1q16HighTPseudo_UPD, ARM::VLD1d16Twb_fixed,   true,  true, true, SingleHighTSpc,  3, 4 ,false},
{ ARM::VLD1q16LowQPseudo_UPD,  ARM::VLD1d16Qwb_fixed,   true,  true, true, SingleLowSpc,  4, 4 ,false},
{ ARM::VLD1q16LowTPseudo_UPD,  ARM::VLD1d16Twb_fixed,   true,  true, true, SingleLowSpc,  3, 4 ,false},
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 214-219
```cpp
{ ARM::VLD1q32HighQPseudo,  ARM::VLD1d32Q,     true,  false, false, SingleHighQSpc,  4, 2 ,false},
{ ARM::VLD1q32HighQPseudo_UPD, ARM::VLD1d32Qwb_fixed,   true,  true, true, SingleHighQSpc,  4, 2 ,false},
{ ARM::VLD1q32HighTPseudo,  ARM::VLD1d32T,     true,  false, false, SingleHighTSpc,  3, 2 ,false},
{ ARM::VLD1q32HighTPseudo_UPD, ARM::VLD1d32Twb_fixed,   true,  true, true, SingleHighTSpc,  3, 2 ,false},
{ ARM::VLD1q32LowQPseudo_UPD,  ARM::VLD1d32Qwb_fixed,   true,  true, true, SingleLowSpc,  4, 2 ,false},
{ ARM::VLD1q32LowTPseudo_UPD,  ARM::VLD1d32Twb_fixed,   true,  true, true, SingleLowSpc,  3, 2 ,false},
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 221-226
```cpp
{ ARM::VLD1q64HighQPseudo,  ARM::VLD1d64Q,     true,  false, false, SingleHighQSpc,  4, 1 ,false},
{ ARM::VLD1q64HighQPseudo_UPD, ARM::VLD1d64Qwb_fixed,   true,  true, true, SingleHighQSpc,  4, 1 ,false},
{ ARM::VLD1q64HighTPseudo,  ARM::VLD1d64T,     true,  false, false, SingleHighTSpc,  3, 1 ,false},
{ ARM::VLD1q64HighTPseudo_UPD, ARM::VLD1d64Twb_fixed,   true,  true, true, SingleHighTSpc,  3, 1 ,false},
{ ARM::VLD1q64LowQPseudo_UPD,  ARM::VLD1d64Qwb_fixed,   true,  true, true, SingleLowSpc,  4, 1 ,false},
{ ARM::VLD1q64LowTPseudo_UPD,  ARM::VLD1d64Twb_fixed,   true,  true, true, SingleLowSpc,  3, 1 ,false},
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 228-233
```cpp
{ ARM::VLD1q8HighQPseudo,   ARM::VLD1d8Q,     true,  false, false, SingleHighQSpc,  4, 8 ,false},
{ ARM::VLD1q8HighQPseudo_UPD, ARM::VLD1d8Qwb_fixed,   true,  true, true, SingleHighQSpc,  4, 8 ,false},
{ ARM::VLD1q8HighTPseudo,   ARM::VLD1d8T,     true,  false, false, SingleHighTSpc,  3, 8 ,false},
{ ARM::VLD1q8HighTPseudo_UPD, ARM::VLD1d8Twb_fixed,   true,  true, true, SingleHighTSpc,  3, 8 ,false},
{ ARM::VLD1q8LowQPseudo_UPD,  ARM::VLD1d8Qwb_fixed,   true,  true, true, SingleLowSpc,  4, 8 ,false},
{ ARM::VLD1q8LowTPseudo_UPD,  ARM::VLD1d8Twb_fixed,   true,  true, true, SingleLowSpc,  3, 8 ,false},
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 235-246
```cpp
{ ARM::VLD2DUPq16EvenPseudo,  ARM::VLD2DUPd16x2,  true, false, false, EvenDblSpc, 2, 4 ,false},
{ ARM::VLD2DUPq16OddPseudo,   ARM::VLD2DUPd16x2,  true, false, false, OddDblSpc,  2, 4 ,false},
{ ARM::VLD2DUPq16OddPseudoWB_fixed,   ARM::VLD2DUPd16x2wb_fixed, true, true, false, OddDblSpc,  2, 4 ,false},
{ ARM::VLD2DUPq16OddPseudoWB_register,   ARM::VLD2DUPd16x2wb_register, true, true, true, OddDblSpc,  2, 4 ,false},
{ ARM::VLD2DUPq32EvenPseudo,  ARM::VLD2DUPd32x2,  true, false, false, EvenDblSpc, 2, 2 ,false},
{ ARM::VLD2DUPq32OddPseudo,   ARM::VLD2DUPd32x2,  true, false, false, OddDblSpc,  2, 2 ,false},
{ ARM::VLD2DUPq32OddPseudoWB_fixed,   ARM::VLD2DUPd32x2wb_fixed, true, true, false, OddDblSpc,  2, 2 ,false},
{ ARM::VLD2DUPq32OddPseudoWB_register,   ARM::VLD2DUPd32x2wb_register, true, true, true, OddDblSpc,  2, 2 ,false},
{ ARM::VLD2DUPq8EvenPseudo,   ARM::VLD2DUPd8x2,   true, false, false, EvenDblSpc, 2, 8 ,false},
{ ARM::VLD2DUPq8OddPseudo,    ARM::VLD2DUPd8x2,   true, false, false, OddDblSpc,  2, 8 ,false},
{ ARM::VLD2DUPq8OddPseudoWB_fixed,    ARM::VLD2DUPd8x2wb_fixed, true, true, false, OddDblSpc,  2, 8 ,false},
{ ARM::VLD2DUPq8OddPseudoWB_register,    ARM::VLD2DUPd8x2wb_register, true, true, true, OddDblSpc,  2, 8 ,false},
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 248-257
```cpp
{ ARM::VLD2LNd16Pseudo,     ARM::VLD2LNd16,     true, false, false, SingleSpc,  2, 4 ,true},
{ ARM::VLD2LNd16Pseudo_UPD, ARM::VLD2LNd16_UPD, true, true, true,  SingleSpc,  2, 4 ,true},
{ ARM::VLD2LNd32Pseudo,     ARM::VLD2LNd32,     true, false, false, SingleSpc,  2, 2 ,true},
{ ARM::VLD2LNd32Pseudo_UPD, ARM::VLD2LNd32_UPD, true, true, true,  SingleSpc,  2, 2 ,true},
{ ARM::VLD2LNd8Pseudo,      ARM::VLD2LNd8,      true, false, false, SingleSpc,  2, 8 ,true},
{ ARM::VLD2LNd8Pseudo_UPD,  ARM::VLD2LNd8_UPD, true, true, true,  SingleSpc,  2, 8 ,true},
{ ARM::VLD2LNq16Pseudo,     ARM::VLD2LNq16,     true, false, false, EvenDblSpc, 2, 4 ,true},
{ ARM::VLD2LNq16Pseudo_UPD, ARM::VLD2LNq16_UPD, true, true, true,  EvenDblSpc, 2, 4 ,true},
{ ARM::VLD2LNq32Pseudo,     ARM::VLD2LNq32,     true, false, false, EvenDblSpc, 2, 2 ,true},
{ ARM::VLD2LNq32Pseudo_UPD, ARM::VLD2LNq32_UPD, true, true, true,  EvenDblSpc, 2, 2 ,true},
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 259-267
```cpp
{ ARM::VLD2q16Pseudo,       ARM::VLD2q16,      true,  false, false, SingleSpc,  4, 4 ,false},
{ ARM::VLD2q16PseudoWB_fixed,   ARM::VLD2q16wb_fixed, true, true, false,  SingleSpc,  4, 4 ,false},
{ ARM::VLD2q16PseudoWB_register,   ARM::VLD2q16wb_register, true, true, true,  SingleSpc,  4, 4 ,false},
{ ARM::VLD2q32Pseudo,       ARM::VLD2q32,      true,  false, false, SingleSpc,  4, 2 ,false},
{ ARM::VLD2q32PseudoWB_fixed,   ARM::VLD2q32wb_fixed, true, true, false,  SingleSpc,  4, 2 ,false},
{ ARM::VLD2q32PseudoWB_register,   ARM::VLD2q32wb_register, true, true, true,  SingleSpc,  4, 2 ,false},
{ ARM::VLD2q8Pseudo,        ARM::VLD2q8,       true,  false, false, SingleSpc,  4, 8 ,false},
{ ARM::VLD2q8PseudoWB_fixed,    ARM::VLD2q8wb_fixed, true, true, false,  SingleSpc,  4, 8 ,false},
{ ARM::VLD2q8PseudoWB_register,    ARM::VLD2q8wb_register, true, true, true,  SingleSpc,  4, 8 ,false},
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 269-283
```cpp
{ ARM::VLD3DUPd16Pseudo,     ARM::VLD3DUPd16,     true, false, false, SingleSpc, 3, 4,true},
{ ARM::VLD3DUPd16Pseudo_UPD, ARM::VLD3DUPd16_UPD, true, true, true,  SingleSpc, 3, 4,true},
{ ARM::VLD3DUPd32Pseudo,     ARM::VLD3DUPd32,     true, false, false, SingleSpc, 3, 2,true},
{ ARM::VLD3DUPd32Pseudo_UPD, ARM::VLD3DUPd32_UPD, true, true, true,  SingleSpc, 3, 2,true},
{ ARM::VLD3DUPd8Pseudo,      ARM::VLD3DUPd8,      true, false, false, SingleSpc, 3, 8,true},
{ ARM::VLD3DUPd8Pseudo_UPD,  ARM::VLD3DUPd8_UPD, true, true, true,  SingleSpc, 3, 8,true},
{ ARM::VLD3DUPq16EvenPseudo, ARM::VLD3DUPq16,     true, false, false, EvenDblSpc, 3, 4 ,true},
{ ARM::VLD3DUPq16OddPseudo,  ARM::VLD3DUPq16,     true, false, false, OddDblSpc,  3, 4 ,true},
{ ARM::VLD3DUPq16OddPseudo_UPD,  ARM::VLD3DUPq16_UPD, true, true, true, OddDblSpc,  3, 4 ,true},
{ ARM::VLD3DUPq32EvenPseudo, ARM::VLD3DUPq32,     true, false, false, EvenDblSpc, 3, 2 ,true},
{ ARM::VLD3DUPq32OddPseudo,  ARM::VLD3DUPq32,     true, false, false, OddDblSpc,  3, 2 ,true},
{ ARM::VLD3DUPq32OddPseudo_UPD,  ARM::VLD3DUPq32_UPD, true, true, true, OddDblSpc,  3, 2 ,true},
{ ARM::VLD3DUPq8EvenPseudo,  ARM::VLD3DUPq8,      true, false, false, EvenDblSpc, 3, 8 ,true},
{ ARM::VLD3DUPq8OddPseudo,   ARM::VLD3DUPq8,      true, false, false, OddDblSpc,  3, 8 ,true},
{ ARM::VLD3DUPq8OddPseudo_UPD,   ARM::VLD3DUPq8_UPD, true, true, true, OddDblSpc,  3, 8 ,true},
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 285-294
```cpp
{ ARM::VLD3LNd16Pseudo,     ARM::VLD3LNd16,     true, false, false, SingleSpc,  3, 4 ,true},
{ ARM::VLD3LNd16Pseudo_UPD, ARM::VLD3LNd16_UPD, true, true, true,  SingleSpc,  3, 4 ,true},
{ ARM::VLD3LNd32Pseudo,     ARM::VLD3LNd32,     true, false, false, SingleSpc,  3, 2 ,true},
{ ARM::VLD3LNd32Pseudo_UPD, ARM::VLD3LNd32_UPD, true, true, true,  SingleSpc,  3, 2 ,true},
{ ARM::VLD3LNd8Pseudo,      ARM::VLD3LNd8,      true, false, false, SingleSpc,  3, 8 ,true},
{ ARM::VLD3LNd8Pseudo_UPD,  ARM::VLD3LNd8_UPD, true, true, true,  SingleSpc,  3, 8 ,true},
{ ARM::VLD3LNq16Pseudo,     ARM::VLD3LNq16,     true, false, false, EvenDblSpc, 3, 4 ,true},
{ ARM::VLD3LNq16Pseudo_UPD, ARM::VLD3LNq16_UPD, true, true, true,  EvenDblSpc, 3, 4 ,true},
{ ARM::VLD3LNq32Pseudo,     ARM::VLD3LNq32,     true, false, false, EvenDblSpc, 3, 2 ,true},
{ ARM::VLD3LNq32Pseudo_UPD, ARM::VLD3LNq32_UPD, true, true, true,  EvenDblSpc, 3, 2 ,true},
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 296-301
```cpp
{ ARM::VLD3d16Pseudo,       ARM::VLD3d16,      true,  false, false, SingleSpc,  3, 4 ,true},
{ ARM::VLD3d16Pseudo_UPD,   ARM::VLD3d16_UPD, true, true, true,  SingleSpc,  3, 4 ,true},
{ ARM::VLD3d32Pseudo,       ARM::VLD3d32,      true,  false, false, SingleSpc,  3, 2 ,true},
{ ARM::VLD3d32Pseudo_UPD,   ARM::VLD3d32_UPD, true, true, true,  SingleSpc,  3, 2 ,true},
{ ARM::VLD3d8Pseudo,        ARM::VLD3d8,       true,  false, false, SingleSpc,  3, 8 ,true},
{ ARM::VLD3d8Pseudo_UPD,    ARM::VLD3d8_UPD, true, true, true,  SingleSpc,  3, 8 ,true},
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 303-311
```cpp
{ ARM::VLD3q16Pseudo_UPD,    ARM::VLD3q16_UPD, true, true, true,  EvenDblSpc, 3, 4 ,true},
{ ARM::VLD3q16oddPseudo,     ARM::VLD3q16,     true,  false, false, OddDblSpc,  3, 4 ,true},
{ ARM::VLD3q16oddPseudo_UPD, ARM::VLD3q16_UPD, true, true, true,  OddDblSpc,  3, 4 ,true},
{ ARM::VLD3q32Pseudo_UPD,    ARM::VLD3q32_UPD, true, true, true,  EvenDblSpc, 3, 2 ,true},
{ ARM::VLD3q32oddPseudo,     ARM::VLD3q32,     true,  false, false, OddDblSpc,  3, 2 ,true},
{ ARM::VLD3q32oddPseudo_UPD, ARM::VLD3q32_UPD, true, true, true,  OddDblSpc,  3, 2 ,true},
{ ARM::VLD3q8Pseudo_UPD,     ARM::VLD3q8_UPD, true, true, true,  EvenDblSpc, 3, 8 ,true},
{ ARM::VLD3q8oddPseudo,      ARM::VLD3q8,      true,  false, false, OddDblSpc,  3, 8 ,true},
{ ARM::VLD3q8oddPseudo_UPD,  ARM::VLD3q8_UPD, true, true, true,  OddDblSpc,  3, 8 ,true},
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 313-327
```cpp
{ ARM::VLD4DUPd16Pseudo,     ARM::VLD4DUPd16,     true, false, false, SingleSpc, 4, 4,true},
{ ARM::VLD4DUPd16Pseudo_UPD, ARM::VLD4DUPd16_UPD, true, true, true,  SingleSpc, 4, 4,true},
{ ARM::VLD4DUPd32Pseudo,     ARM::VLD4DUPd32,     true, false, false, SingleSpc, 4, 2,true},
{ ARM::VLD4DUPd32Pseudo_UPD, ARM::VLD4DUPd32_UPD, true, true, true,  SingleSpc, 4, 2,true},
{ ARM::VLD4DUPd8Pseudo,      ARM::VLD4DUPd8,      true, false, false, SingleSpc, 4, 8,true},
{ ARM::VLD4DUPd8Pseudo_UPD,  ARM::VLD4DUPd8_UPD, true, true, true,  SingleSpc, 4, 8,true},
{ ARM::VLD4DUPq16EvenPseudo, ARM::VLD4DUPq16,     true, false, false, EvenDblSpc, 4, 4 ,true},
{ ARM::VLD4DUPq16OddPseudo,  ARM::VLD4DUPq16,     true, false, false, OddDblSpc,  4, 4 ,true},
{ ARM::VLD4DUPq16OddPseudo_UPD,  ARM::VLD4DUPq16_UPD, true, true, true, OddDblSpc,  4, 4 ,true},
{ ARM::VLD4DUPq32EvenPseudo, ARM::VLD4DUPq32,     true, false, false, EvenDblSpc, 4, 2 ,true},
{ ARM::VLD4DUPq32OddPseudo,  ARM::VLD4DUPq32,     true, false, false, OddDblSpc,  4, 2 ,true},
{ ARM::VLD4DUPq32OddPseudo_UPD,  ARM::VLD4DUPq32_UPD, true, true, true, OddDblSpc,  4, 2 ,true},
{ ARM::VLD4DUPq8EvenPseudo,  ARM::VLD4DUPq8,      true, false, false, EvenDblSpc, 4, 8 ,true},
{ ARM::VLD4DUPq8OddPseudo,   ARM::VLD4DUPq8,      true, false, false, OddDblSpc,  4, 8 ,true},
{ ARM::VLD4DUPq8OddPseudo_UPD,   ARM::VLD4DUPq8_UPD, true, true, true, OddDblSpc,  4, 8 ,true},
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 329-338
```cpp
{ ARM::VLD4LNd16Pseudo,     ARM::VLD4LNd16,     true, false, false, SingleSpc,  4, 4 ,true},
{ ARM::VLD4LNd16Pseudo_UPD, ARM::VLD4LNd16_UPD, true, true, true,  SingleSpc,  4, 4 ,true},
{ ARM::VLD4LNd32Pseudo,     ARM::VLD4LNd32,     true, false, false, SingleSpc,  4, 2 ,true},
{ ARM::VLD4LNd32Pseudo_UPD, ARM::VLD4LNd32_UPD, true, true, true,  SingleSpc,  4, 2 ,true},
{ ARM::VLD4LNd8Pseudo,      ARM::VLD4LNd8,      true, false, false, SingleSpc,  4, 8 ,true},
{ ARM::VLD4LNd8Pseudo_UPD,  ARM::VLD4LNd8_UPD, true, true, true,  SingleSpc,  4, 8 ,true},
{ ARM::VLD4LNq16Pseudo,     ARM::VLD4LNq16,     true, false, false, EvenDblSpc, 4, 4 ,true},
{ ARM::VLD4LNq16Pseudo_UPD, ARM::VLD4LNq16_UPD, true, true, true,  EvenDblSpc, 4, 4 ,true},
{ ARM::VLD4LNq32Pseudo,     ARM::VLD4LNq32,     true, false, false, EvenDblSpc, 4, 2 ,true},
{ ARM::VLD4LNq32Pseudo_UPD, ARM::VLD4LNq32_UPD, true, true, true,  EvenDblSpc, 4, 2 ,true},
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 340-345
```cpp
{ ARM::VLD4d16Pseudo,       ARM::VLD4d16,      true,  false, false, SingleSpc,  4, 4 ,true},
{ ARM::VLD4d16Pseudo_UPD,   ARM::VLD4d16_UPD, true, true, true,  SingleSpc,  4, 4 ,true},
{ ARM::VLD4d32Pseudo,       ARM::VLD4d32,      true,  false, false, SingleSpc,  4, 2 ,true},
{ ARM::VLD4d32Pseudo_UPD,   ARM::VLD4d32_UPD, true, true, true,  SingleSpc,  4, 2 ,true},
{ ARM::VLD4d8Pseudo,        ARM::VLD4d8,       true,  false, false, SingleSpc,  4, 8 ,true},
{ ARM::VLD4d8Pseudo_UPD,    ARM::VLD4d8_UPD, true, true, true,  SingleSpc,  4, 8 ,true},
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 347-355
```cpp
{ ARM::VLD4q16Pseudo_UPD,    ARM::VLD4q16_UPD, true, true, true,  EvenDblSpc, 4, 4 ,true},
{ ARM::VLD4q16oddPseudo,     ARM::VLD4q16,     true,  false, false, OddDblSpc,  4, 4 ,true},
{ ARM::VLD4q16oddPseudo_UPD, ARM::VLD4q16_UPD, true, true, true,  OddDblSpc,  4, 4 ,true},
{ ARM::VLD4q32Pseudo_UPD,    ARM::VLD4q32_UPD, true, true, true,  EvenDblSpc, 4, 2 ,true},
{ ARM::VLD4q32oddPseudo,     ARM::VLD4q32,     true,  false, false, OddDblSpc,  4, 2 ,true},
{ ARM::VLD4q32oddPseudo_UPD, ARM::VLD4q32_UPD, true, true, true,  OddDblSpc,  4, 2 ,true},
{ ARM::VLD4q8Pseudo_UPD,     ARM::VLD4q8_UPD, true, true, true,  EvenDblSpc, 4, 8 ,true},
{ ARM::VLD4q8oddPseudo,      ARM::VLD4q8,      true,  false, false, OddDblSpc,  4, 8 ,true},
{ ARM::VLD4q8oddPseudo_UPD,  ARM::VLD4q8_UPD, true, true, true,  OddDblSpc,  4, 8 ,true},
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 357-362
```cpp
{ ARM::VST1LNq16Pseudo,     ARM::VST1LNd16,    false, false, false, EvenDblSpc, 1, 4 ,true},
{ ARM::VST1LNq16Pseudo_UPD, ARM::VST1LNd16_UPD, false, true, true,  EvenDblSpc, 1, 4 ,true},
{ ARM::VST1LNq32Pseudo,     ARM::VST1LNd32,    false, false, false, EvenDblSpc, 1, 2 ,true},
{ ARM::VST1LNq32Pseudo_UPD, ARM::VST1LNd32_UPD, false, true, true,  EvenDblSpc, 1, 2 ,true},
{ ARM::VST1LNq8Pseudo,      ARM::VST1LNd8,     false, false, false, EvenDblSpc, 1, 8 ,true},
{ ARM::VST1LNq8Pseudo_UPD,  ARM::VST1LNd8_UPD, false, true, true,  EvenDblSpc, 1, 8 ,true},
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 364-369
```cpp
{ ARM::VST1d16QPseudo,      ARM::VST1d16Q,     false, false, false, SingleSpc,  4, 4 ,false},
{ ARM::VST1d16QPseudoWB_fixed,  ARM::VST1d16Qwb_fixed, false, true, false, SingleSpc,  4, 4 ,false},
{ ARM::VST1d16QPseudoWB_register, ARM::VST1d16Qwb_register, false, true, true, SingleSpc,  4, 4 ,false},
{ ARM::VST1d16TPseudo,      ARM::VST1d16T,     false, false, false, SingleSpc,  3, 4 ,false},
{ ARM::VST1d16TPseudoWB_fixed,  ARM::VST1d16Twb_fixed, false, true, false, SingleSpc,  3, 4 ,false},
{ ARM::VST1d16TPseudoWB_register, ARM::VST1d16Twb_register, false, true, true, SingleSpc,  3, 4 ,false},
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 371-376
```cpp
{ ARM::VST1d32QPseudo,      ARM::VST1d32Q,     false, false, false, SingleSpc,  4, 2 ,false},
{ ARM::VST1d32QPseudoWB_fixed,  ARM::VST1d32Qwb_fixed, false, true, false, SingleSpc,  4, 2 ,false},
{ ARM::VST1d32QPseudoWB_register, ARM::VST1d32Qwb_register, false, true, true, SingleSpc,  4, 2 ,false},
{ ARM::VST1d32TPseudo,      ARM::VST1d32T,     false, false, false, SingleSpc,  3, 2 ,false},
{ ARM::VST1d32TPseudoWB_fixed,  ARM::VST1d32Twb_fixed, false, true, false, SingleSpc,  3, 2 ,false},
{ ARM::VST1d32TPseudoWB_register, ARM::VST1d32Twb_register, false, true, true, SingleSpc,  3, 2 ,false},
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 378-383
```cpp
{ ARM::VST1d64QPseudo,      ARM::VST1d64Q,     false, false, false, SingleSpc,  4, 1 ,false},
{ ARM::VST1d64QPseudoWB_fixed,  ARM::VST1d64Qwb_fixed, false, true, false,  SingleSpc,  4, 1 ,false},
{ ARM::VST1d64QPseudoWB_register, ARM::VST1d64Qwb_register, false, true, true,  SingleSpc,  4, 1 ,false},
{ ARM::VST1d64TPseudo,      ARM::VST1d64T,     false, false, false, SingleSpc,  3, 1 ,false},
{ ARM::VST1d64TPseudoWB_fixed,  ARM::VST1d64Twb_fixed, false, true, false,  SingleSpc,  3, 1 ,false},
{ ARM::VST1d64TPseudoWB_register, ARM::VST1d64Twb_register, false, true, true,  SingleSpc,  3, 1 ,false},
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 385-390
```cpp
{ ARM::VST1d8QPseudo,       ARM::VST1d8Q,      false, false, false, SingleSpc,  4, 8 ,false},
{ ARM::VST1d8QPseudoWB_fixed,   ARM::VST1d8Qwb_fixed, false, true, false, SingleSpc,  4, 8 ,false},
{ ARM::VST1d8QPseudoWB_register,  ARM::VST1d8Qwb_register, false, true, true, SingleSpc,  4, 8 ,false},
{ ARM::VST1d8TPseudo,       ARM::VST1d8T,      false, false, false, SingleSpc,  3, 8 ,false},
{ ARM::VST1d8TPseudoWB_fixed,   ARM::VST1d8Twb_fixed, false, true, false, SingleSpc,  3, 8 ,false},
{ ARM::VST1d8TPseudoWB_register,  ARM::VST1d8Twb_register, false, true, true, SingleSpc,  3, 8 ,false},
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 392-397
```cpp
{ ARM::VST1q16HighQPseudo,  ARM::VST1d16Q,     false, false, false, SingleHighQSpc,   4, 4 ,false},
{ ARM::VST1q16HighQPseudo_UPD,  ARM::VST1d16Qwb_fixed,  false, true, true, SingleHighQSpc,   4, 8 ,false},
{ ARM::VST1q16HighTPseudo,  ARM::VST1d16T,     false, false, false, SingleHighTSpc,   3, 4 ,false},
{ ARM::VST1q16HighTPseudo_UPD,  ARM::VST1d16Twb_fixed,  false, true, true, SingleHighTSpc,   3, 4 ,false},
{ ARM::VST1q16LowQPseudo_UPD,   ARM::VST1d16Qwb_fixed,  false, true, true, SingleLowSpc,   4, 4 ,false},
{ ARM::VST1q16LowTPseudo_UPD,   ARM::VST1d16Twb_fixed,  false, true, true, SingleLowSpc,   3, 4 ,false},
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 399-404
```cpp
{ ARM::VST1q32HighQPseudo,  ARM::VST1d32Q,     false, false, false, SingleHighQSpc,   4, 2 ,false},
{ ARM::VST1q32HighQPseudo_UPD,  ARM::VST1d32Qwb_fixed,  false, true, true, SingleHighQSpc,   4, 8 ,false},
{ ARM::VST1q32HighTPseudo,  ARM::VST1d32T,     false, false, false, SingleHighTSpc,   3, 2 ,false},
{ ARM::VST1q32HighTPseudo_UPD,  ARM::VST1d32Twb_fixed,  false, true, true, SingleHighTSpc,   3, 2 ,false},
{ ARM::VST1q32LowQPseudo_UPD,   ARM::VST1d32Qwb_fixed,  false, true, true, SingleLowSpc,   4, 2 ,false},
{ ARM::VST1q32LowTPseudo_UPD,   ARM::VST1d32Twb_fixed,  false, true, true, SingleLowSpc,   3, 2 ,false},
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 406-411
```cpp
{ ARM::VST1q64HighQPseudo,  ARM::VST1d64Q,     false, false, false, SingleHighQSpc,   4, 1 ,false},
{ ARM::VST1q64HighQPseudo_UPD,  ARM::VST1d64Qwb_fixed,  false, true, true, SingleHighQSpc,   4, 8 ,false},
{ ARM::VST1q64HighTPseudo,  ARM::VST1d64T,     false, false, false, SingleHighTSpc,   3, 1 ,false},
{ ARM::VST1q64HighTPseudo_UPD,  ARM::VST1d64Twb_fixed,  false, true, true, SingleHighTSpc,   3, 1 ,false},
{ ARM::VST1q64LowQPseudo_UPD,   ARM::VST1d64Qwb_fixed,  false, true, true, SingleLowSpc,   4, 1 ,false},
{ ARM::VST1q64LowTPseudo_UPD,   ARM::VST1d64Twb_fixed,  false, true, true, SingleLowSpc,   3, 1 ,false},
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 413-418
```cpp
{ ARM::VST1q8HighQPseudo,   ARM::VST1d8Q,      false, false, false, SingleHighQSpc,   4, 8 ,false},
{ ARM::VST1q8HighQPseudo_UPD,  ARM::VST1d8Qwb_fixed,  false, true, true, SingleHighQSpc,   4, 8 ,false},
{ ARM::VST1q8HighTPseudo,   ARM::VST1d8T,      false, false, false, SingleHighTSpc,   3, 8 ,false},
{ ARM::VST1q8HighTPseudo_UPD,  ARM::VST1d8Twb_fixed,  false, true, true, SingleHighTSpc,   3, 8 ,false},
{ ARM::VST1q8LowQPseudo_UPD,   ARM::VST1d8Qwb_fixed,  false, true, true, SingleLowSpc,   4, 8 ,false},
{ ARM::VST1q8LowTPseudo_UPD,   ARM::VST1d8Twb_fixed,  false, true, true, SingleLowSpc,   3, 8 ,false},
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 420-429
```cpp
{ ARM::VST2LNd16Pseudo,     ARM::VST2LNd16,     false, false, false, SingleSpc, 2, 4 ,true},
{ ARM::VST2LNd16Pseudo_UPD, ARM::VST2LNd16_UPD, false, true, true,  SingleSpc, 2, 4 ,true},
{ ARM::VST2LNd32Pseudo,     ARM::VST2LNd32,     false, false, false, SingleSpc, 2, 2 ,true},
{ ARM::VST2LNd32Pseudo_UPD, ARM::VST2LNd32_UPD, false, true, true,  SingleSpc, 2, 2 ,true},
{ ARM::VST2LNd8Pseudo,      ARM::VST2LNd8,      false, false, false, SingleSpc, 2, 8 ,true},
{ ARM::VST2LNd8Pseudo_UPD,  ARM::VST2LNd8_UPD, false, true, true,  SingleSpc, 2, 8 ,true},
{ ARM::VST2LNq16Pseudo,     ARM::VST2LNq16,     false, false, false, EvenDblSpc, 2, 4,true},
{ ARM::VST2LNq16Pseudo_UPD, ARM::VST2LNq16_UPD, false, true, true,  EvenDblSpc, 2, 4,true},
{ ARM::VST2LNq32Pseudo,     ARM::VST2LNq32,     false, false, false, EvenDblSpc, 2, 2,true},
{ ARM::VST2LNq32Pseudo_UPD, ARM::VST2LNq32_UPD, false, true, true,  EvenDblSpc, 2, 2,true},
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 431-439
```cpp
{ ARM::VST2q16Pseudo,       ARM::VST2q16,      false, false, false, SingleSpc,  4, 4 ,false},
{ ARM::VST2q16PseudoWB_fixed,   ARM::VST2q16wb_fixed, false, true, false,  SingleSpc,  4, 4 ,false},
{ ARM::VST2q16PseudoWB_register,   ARM::VST2q16wb_register, false, true, true,  SingleSpc,  4, 4 ,false},
{ ARM::VST2q32Pseudo,       ARM::VST2q32,      false, false, false, SingleSpc,  4, 2 ,false},
{ ARM::VST2q32PseudoWB_fixed,   ARM::VST2q32wb_fixed, false, true, false,  SingleSpc,  4, 2 ,false},
{ ARM::VST2q32PseudoWB_register,   ARM::VST2q32wb_register, false, true, true,  SingleSpc,  4, 2 ,false},
{ ARM::VST2q8Pseudo,        ARM::VST2q8,       false, false, false, SingleSpc,  4, 8 ,false},
{ ARM::VST2q8PseudoWB_fixed,    ARM::VST2q8wb_fixed, false, true, false,  SingleSpc,  4, 8 ,false},
{ ARM::VST2q8PseudoWB_register,    ARM::VST2q8wb_register, false, true, true,  SingleSpc,  4, 8 ,false},
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 441-450
```cpp
{ ARM::VST3LNd16Pseudo,     ARM::VST3LNd16,     false, false, false, SingleSpc, 3, 4 ,true},
{ ARM::VST3LNd16Pseudo_UPD, ARM::VST3LNd16_UPD, false, true, true,  SingleSpc, 3, 4 ,true},
{ ARM::VST3LNd32Pseudo,     ARM::VST3LNd32,     false, false, false, SingleSpc, 3, 2 ,true},
{ ARM::VST3LNd32Pseudo_UPD, ARM::VST3LNd32_UPD, false, true, true,  SingleSpc, 3, 2 ,true},
{ ARM::VST3LNd8Pseudo,      ARM::VST3LNd8,      false, false, false, SingleSpc, 3, 8 ,true},
{ ARM::VST3LNd8Pseudo_UPD,  ARM::VST3LNd8_UPD, false, true, true,  SingleSpc, 3, 8 ,true},
{ ARM::VST3LNq16Pseudo,     ARM::VST3LNq16,     false, false, false, EvenDblSpc, 3, 4,true},
{ ARM::VST3LNq16Pseudo_UPD, ARM::VST3LNq16_UPD, false, true, true,  EvenDblSpc, 3, 4,true},
{ ARM::VST3LNq32Pseudo,     ARM::VST3LNq32,     false, false, false, EvenDblSpc, 3, 2,true},
{ ARM::VST3LNq32Pseudo_UPD, ARM::VST3LNq32_UPD, false, true, true,  EvenDblSpc, 3, 2,true},
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 452-457
```cpp
{ ARM::VST3d16Pseudo,       ARM::VST3d16,      false, false, false, SingleSpc,  3, 4 ,true},
{ ARM::VST3d16Pseudo_UPD,   ARM::VST3d16_UPD, false, true, true,  SingleSpc,  3, 4 ,true},
{ ARM::VST3d32Pseudo,       ARM::VST3d32,      false, false, false, SingleSpc,  3, 2 ,true},
{ ARM::VST3d32Pseudo_UPD,   ARM::VST3d32_UPD, false, true, true,  SingleSpc,  3, 2 ,true},
{ ARM::VST3d8Pseudo,        ARM::VST3d8,       false, false, false, SingleSpc,  3, 8 ,true},
{ ARM::VST3d8Pseudo_UPD,    ARM::VST3d8_UPD, false, true, true,  SingleSpc,  3, 8 ,true},
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 459-467
```cpp
{ ARM::VST3q16Pseudo_UPD,    ARM::VST3q16_UPD, false, true, true,  EvenDblSpc, 3, 4 ,true},
{ ARM::VST3q16oddPseudo,     ARM::VST3q16,     false, false, false, OddDblSpc,  3, 4 ,true},
{ ARM::VST3q16oddPseudo_UPD, ARM::VST3q16_UPD, false, true, true,  OddDblSpc,  3, 4 ,true},
{ ARM::VST3q32Pseudo_UPD,    ARM::VST3q32_UPD, false, true, true,  EvenDblSpc, 3, 2 ,true},
{ ARM::VST3q32oddPseudo,     ARM::VST3q32,     false, false, false, OddDblSpc,  3, 2 ,true},
{ ARM::VST3q32oddPseudo_UPD, ARM::VST3q32_UPD, false, true, true,  OddDblSpc,  3, 2 ,true},
{ ARM::VST3q8Pseudo_UPD,     ARM::VST3q8_UPD, false, true, true,  EvenDblSpc, 3, 8 ,true},
{ ARM::VST3q8oddPseudo,      ARM::VST3q8,      false, false, false, OddDblSpc,  3, 8 ,true},
{ ARM::VST3q8oddPseudo_UPD,  ARM::VST3q8_UPD, false, true, true,  OddDblSpc,  3, 8 ,true},
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 469-478
```cpp
{ ARM::VST4LNd16Pseudo,     ARM::VST4LNd16,     false, false, false, SingleSpc, 4, 4 ,true},
{ ARM::VST4LNd16Pseudo_UPD, ARM::VST4LNd16_UPD, false, true, true,  SingleSpc, 4, 4 ,true},
{ ARM::VST4LNd32Pseudo,     ARM::VST4LNd32,     false, false, false, SingleSpc, 4, 2 ,true},
{ ARM::VST4LNd32Pseudo_UPD, ARM::VST4LNd32_UPD, false, true, true,  SingleSpc, 4, 2 ,true},
{ ARM::VST4LNd8Pseudo,      ARM::VST4LNd8,      false, false, false, SingleSpc, 4, 8 ,true},
{ ARM::VST4LNd8Pseudo_UPD,  ARM::VST4LNd8_UPD, false, true, true,  SingleSpc, 4, 8 ,true},
{ ARM::VST4LNq16Pseudo,     ARM::VST4LNq16,     false, false, false, EvenDblSpc, 4, 4,true},
{ ARM::VST4LNq16Pseudo_UPD, ARM::VST4LNq16_UPD, false, true, true,  EvenDblSpc, 4, 4,true},
{ ARM::VST4LNq32Pseudo,     ARM::VST4LNq32,     false, false, false, EvenDblSpc, 4, 2,true},
{ ARM::VST4LNq32Pseudo_UPD, ARM::VST4LNq32_UPD, false, true, true,  EvenDblSpc, 4, 2,true},
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 480-485
```cpp
{ ARM::VST4d16Pseudo,       ARM::VST4d16,      false, false, false, SingleSpc,  4, 4 ,true},
{ ARM::VST4d16Pseudo_UPD,   ARM::VST4d16_UPD, false, true, true,  SingleSpc,  4, 4 ,true},
{ ARM::VST4d32Pseudo,       ARM::VST4d32,      false, false, false, SingleSpc,  4, 2 ,true},
{ ARM::VST4d32Pseudo_UPD,   ARM::VST4d32_UPD, false, true, true,  SingleSpc,  4, 2 ,true},
{ ARM::VST4d8Pseudo,        ARM::VST4d8,       false, false, false, SingleSpc,  4, 8 ,true},
{ ARM::VST4d8Pseudo_UPD,    ARM::VST4d8_UPD, false, true, true,  SingleSpc,  4, 8 ,true},
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 487-496
```cpp
{ ARM::VST4q16Pseudo_UPD,    ARM::VST4q16_UPD, false, true, true,  EvenDblSpc, 4, 4 ,true},
{ ARM::VST4q16oddPseudo,     ARM::VST4q16,     false, false, false, OddDblSpc,  4, 4 ,true},
{ ARM::VST4q16oddPseudo_UPD, ARM::VST4q16_UPD, false, true, true,  OddDblSpc,  4, 4 ,true},
{ ARM::VST4q32Pseudo_UPD,    ARM::VST4q32_UPD, false, true, true,  EvenDblSpc, 4, 2 ,true},
{ ARM::VST4q32oddPseudo,     ARM::VST4q32,     false, false, false, OddDblSpc,  4, 2 ,true},
{ ARM::VST4q32oddPseudo_UPD, ARM::VST4q32_UPD, false, true, true,  OddDblSpc,  4, 2 ,true},
{ ARM::VST4q8Pseudo_UPD,     ARM::VST4q8_UPD, false, true, true,  EvenDblSpc, 4, 8 ,true},
{ ARM::VST4q8oddPseudo,      ARM::VST4q8,      false, false, false, OddDblSpc,  4, 8 ,true},
{ ARM::VST4q8oddPseudo_UPD,  ARM::VST4q8_UPD, false, true, true,  OddDblSpc,  4, 8 ,true}
};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 498-508
```cpp
/// LookupNEONLdSt - Search the NEONLdStTable for information about a NEON
/// load or store pseudo instruction.
static const NEONLdStTableEntry *LookupNEONLdSt(unsigned Opcode) {
#ifndef NDEBUG
  // Make sure the table is sorted.
  static std::atomic<bool> TableChecked(false);
  if (!TableChecked.load(std::memory_order_relaxed)) {
    assert(llvm::is_sorted(NEONLdStTable) && "NEONLdStTable is not sorted!");
    TableChecked.store(true, std::memory_order_relaxed);
  }
#endif
```
- EN: Implements `LookupNEONLdSt`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `LookupNEONLdSt`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 510-514
```cpp
  auto I = llvm::lower_bound(NEONLdStTable, Opcode);
  if (I != std::end(NEONLdStTable) && I->PseudoOpc == Opcode)
    return I;
  return nullptr;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 516-530
```cpp
/// GetDSubRegs - Get 4 D subregisters of a Q, QQ, or QQQQ register,
/// corresponding to the specified register spacing.  Not all of the results
/// are necessarily valid, e.g., a Q register only has 2 D subregisters.
static void GetDSubRegs(unsigned Reg, NEONRegSpacing RegSpc,
                        const TargetRegisterInfo *TRI, MCRegister &D0,
                        MCRegister &D1, MCRegister &D2, MCRegister &D3) {
  if (RegSpc == SingleSpc || RegSpc == SingleLowSpc) {
    D0 = TRI->getSubReg(Reg, ARM::dsub_0);
    D1 = TRI->getSubReg(Reg, ARM::dsub_1);
    D2 = TRI->getSubReg(Reg, ARM::dsub_2);
    D3 = TRI->getSubReg(Reg, ARM::dsub_3);
  } else if (RegSpc == SingleHighQSpc) {
    D0 = TRI->getSubReg(Reg, ARM::dsub_4);
    D1 = TRI->getSubReg(Reg, ARM::dsub_5);
    D2 = TRI->getSubReg(Reg, ARM::dsub_6);
```
- EN: Implements `GetDSubRegs`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `GetDSubRegs`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 531-545
```cpp
    D3 = TRI->getSubReg(Reg, ARM::dsub_7);
  } else if (RegSpc == SingleHighTSpc) {
    D0 = TRI->getSubReg(Reg, ARM::dsub_3);
    D1 = TRI->getSubReg(Reg, ARM::dsub_4);
    D2 = TRI->getSubReg(Reg, ARM::dsub_5);
    D3 = TRI->getSubReg(Reg, ARM::dsub_6);
  } else if (RegSpc == EvenDblSpc) {
    D0 = TRI->getSubReg(Reg, ARM::dsub_0);
    D1 = TRI->getSubReg(Reg, ARM::dsub_2);
    D2 = TRI->getSubReg(Reg, ARM::dsub_4);
    D3 = TRI->getSubReg(Reg, ARM::dsub_6);
  } else {
    assert(RegSpc == OddDblSpc && "unknown register spacing");
    D0 = TRI->getSubReg(Reg, ARM::dsub_1);
    D1 = TRI->getSubReg(Reg, ARM::dsub_3);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 546-549
```cpp
    D2 = TRI->getSubReg(Reg, ARM::dsub_5);
    D3 = TRI->getSubReg(Reg, ARM::dsub_7);
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 551-556
```cpp
/// ExpandVLD - Translate VLD pseudo instructions with Q, QQ or QQQQ register
/// operands to real VLD instructions with D register operands.
void ARMExpandPseudo::ExpandVLD(MachineBasicBlock::iterator &MBBI) {
  MachineInstr &MI = *MBBI;
  MachineBasicBlock &MBB = *MI.getParent();
  LLVM_DEBUG(dbgs() << "Expanding: "; MI.dump());
```
- EN: Implements `ARMExpandPseudo::ExpandVLD`, a mutation/build routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMExpandPseudo::ExpandVLD`，它是一个围绕机器基本块展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 558-561
```cpp
  const NEONLdStTableEntry *TableEntry = LookupNEONLdSt(MI.getOpcode());
  assert(TableEntry && TableEntry->IsLoad && "NEONLdStTable lookup failed");
  NEONRegSpacing RegSpc = (NEONRegSpacing)TableEntry->RegSpacing;
  unsigned NumRegs = TableEntry->NumRegs;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 563-565
```cpp
  MachineInstrBuilder MIB = BuildMI(MBB, MBBI, MI.getDebugLoc(),
                                    TII->get(TableEntry->RealOpc));
  unsigned OpIdx = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 567-568
```cpp
  bool DstIsDead = MI.getOperand(OpIdx).isDead();
  Register DstReg = MI.getOperand(OpIdx++).getReg();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 570-578
```cpp
  bool IsVLD2DUP = TableEntry->RealOpc == ARM::VLD2DUPd8x2 ||
                   TableEntry->RealOpc == ARM::VLD2DUPd16x2 ||
                   TableEntry->RealOpc == ARM::VLD2DUPd32x2 ||
                   TableEntry->RealOpc == ARM::VLD2DUPd8x2wb_fixed ||
                   TableEntry->RealOpc == ARM::VLD2DUPd16x2wb_fixed ||
                   TableEntry->RealOpc == ARM::VLD2DUPd32x2wb_fixed ||
                   TableEntry->RealOpc == ARM::VLD2DUPd8x2wb_register ||
                   TableEntry->RealOpc == ARM::VLD2DUPd16x2wb_register ||
                   TableEntry->RealOpc == ARM::VLD2DUPd32x2wb_register;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 580-594
```cpp
  if (IsVLD2DUP) {
    unsigned SubRegIndex;
    if (RegSpc == EvenDblSpc) {
      SubRegIndex = ARM::dsub_0;
    } else {
      assert(RegSpc == OddDblSpc && "Unexpected spacing!");
      SubRegIndex = ARM::dsub_1;
    }
    Register SubReg = TRI->getSubReg(DstReg, SubRegIndex);
    MCRegister DstRegPair =
        TRI->getMatchingSuperReg(SubReg, ARM::dsub_0, &ARM::DPairSpcRegClass);
    MIB.addReg(DstRegPair, RegState::Define | getDeadRegState(DstIsDead));
  } else {
    MCRegister D0, D1, D2, D3;
    GetDSubRegs(DstReg, RegSpc, TRI, D0, D1, D2, D3);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 595-602
```cpp
    MIB.addReg(D0, RegState::Define | getDeadRegState(DstIsDead));
    if (NumRegs > 1 && TableEntry->copyAllListRegs)
      MIB.addReg(D1, RegState::Define | getDeadRegState(DstIsDead));
    if (NumRegs > 2 && TableEntry->copyAllListRegs)
      MIB.addReg(D2, RegState::Define | getDeadRegState(DstIsDead));
    if (NumRegs > 3 && TableEntry->copyAllListRegs)
      MIB.addReg(D3, RegState::Define | getDeadRegState(DstIsDead));
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 604-605
```cpp
  if (TableEntry->isUpdating)
    MIB.add(MI.getOperand(OpIdx++));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 607-609
```cpp
  // Copy the addrmode6 operands.
  MIB.add(MI.getOperand(OpIdx++));
  MIB.add(MI.getOperand(OpIdx++));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 611-625
```cpp
  // Copy the am6offset operand.
  if (TableEntry->hasWritebackOperand) {
    // TODO: The writing-back pseudo instructions we translate here are all
    // defined to take am6offset nodes that are capable to represent both fixed
    // and register forms. Some real instructions, however, do not rely on
    // am6offset and have separate definitions for such forms. When this is the
    // case, fixed forms do not take any offset nodes, so here we skip them for
    // such instructions. Once all real and pseudo writing-back instructions are
    // rewritten without use of am6offset nodes, this code will go away.
    const MachineOperand &AM6Offset = MI.getOperand(OpIdx++);
    if (TableEntry->RealOpc == ARM::VLD1d8Qwb_fixed ||
        TableEntry->RealOpc == ARM::VLD1d16Qwb_fixed ||
        TableEntry->RealOpc == ARM::VLD1d32Qwb_fixed ||
        TableEntry->RealOpc == ARM::VLD1d64Qwb_fixed ||
        TableEntry->RealOpc == ARM::VLD1d8Twb_fixed ||
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 626-638
```cpp
        TableEntry->RealOpc == ARM::VLD1d16Twb_fixed ||
        TableEntry->RealOpc == ARM::VLD1d32Twb_fixed ||
        TableEntry->RealOpc == ARM::VLD1d64Twb_fixed ||
        TableEntry->RealOpc == ARM::VLD2DUPd8x2wb_fixed ||
        TableEntry->RealOpc == ARM::VLD2DUPd16x2wb_fixed ||
        TableEntry->RealOpc == ARM::VLD2DUPd32x2wb_fixed) {
      assert(AM6Offset.getReg() == 0 &&
             "A fixed writing-back pseudo instruction provides an offset "
             "register!");
    } else {
      MIB.add(AM6Offset);
    }
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 640-646
```cpp
  // For an instruction writing double-spaced subregs, the pseudo instruction
  // has an extra operand that is a use of the super-register.  Record the
  // operand index and skip over it.
  unsigned SrcOpIdx = 0;
  if (RegSpc == EvenDblSpc || RegSpc == OddDblSpc || RegSpc == SingleLowSpc ||
      RegSpc == SingleHighQSpc || RegSpc == SingleHighTSpc)
    SrcOpIdx = OpIdx++;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 648-650
```cpp
  // Copy the predicate operands.
  MIB.add(MI.getOperand(OpIdx++));
  MIB.add(MI.getOperand(OpIdx++));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 652-661
```cpp
  // Copy the super-register source operand used for double-spaced subregs over
  // to the new instruction as an implicit operand.
  if (SrcOpIdx != 0) {
    MachineOperand MO = MI.getOperand(SrcOpIdx);
    MO.setImplicit(true);
    MIB.add(MO);
  }
  // Add an implicit def for the super-register.
  MIB.addReg(DstReg, RegState::ImplicitDefine | getDeadRegState(DstIsDead));
  MIB.copyImplicitOps(MI);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 663-667
```cpp
  // Transfer memoperands.
  MIB.cloneMemRefs(MI);
  MI.eraseFromParent();
  LLVM_DEBUG(dbgs() << "To:        "; MIB.getInstr()->dump(););
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 669-674
```cpp
/// ExpandVST - Translate VST pseudo instructions with Q, QQ or QQQQ register
/// operands to real VST instructions with D register operands.
void ARMExpandPseudo::ExpandVST(MachineBasicBlock::iterator &MBBI) {
  MachineInstr &MI = *MBBI;
  MachineBasicBlock &MBB = *MI.getParent();
  LLVM_DEBUG(dbgs() << "Expanding: "; MI.dump());
```
- EN: Implements `ARMExpandPseudo::ExpandVST`, a mutation/build routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMExpandPseudo::ExpandVST`，它是一个围绕机器基本块展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 676-679
```cpp
  const NEONLdStTableEntry *TableEntry = LookupNEONLdSt(MI.getOpcode());
  assert(TableEntry && !TableEntry->IsLoad && "NEONLdStTable lookup failed");
  NEONRegSpacing RegSpc = (NEONRegSpacing)TableEntry->RegSpacing;
  unsigned NumRegs = TableEntry->NumRegs;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 681-685
```cpp
  MachineInstrBuilder MIB = BuildMI(MBB, MBBI, MI.getDebugLoc(),
                                    TII->get(TableEntry->RealOpc));
  unsigned OpIdx = 0;
  if (TableEntry->isUpdating)
    MIB.add(MI.getOperand(OpIdx++));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 687-689
```cpp
  // Copy the addrmode6 operands.
  MIB.add(MI.getOperand(OpIdx++));
  MIB.add(MI.getOperand(OpIdx++));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 691-705
```cpp
  if (TableEntry->hasWritebackOperand) {
    // TODO: The writing-back pseudo instructions we translate here are all
    // defined to take am6offset nodes that are capable to represent both fixed
    // and register forms. Some real instructions, however, do not rely on
    // am6offset and have separate definitions for such forms. When this is the
    // case, fixed forms do not take any offset nodes, so here we skip them for
    // such instructions. Once all real and pseudo writing-back instructions are
    // rewritten without use of am6offset nodes, this code will go away.
    const MachineOperand &AM6Offset = MI.getOperand(OpIdx++);
    if (TableEntry->RealOpc == ARM::VST1d8Qwb_fixed ||
        TableEntry->RealOpc == ARM::VST1d16Qwb_fixed ||
        TableEntry->RealOpc == ARM::VST1d32Qwb_fixed ||
        TableEntry->RealOpc == ARM::VST1d64Qwb_fixed ||
        TableEntry->RealOpc == ARM::VST1d8Twb_fixed ||
        TableEntry->RealOpc == ARM::VST1d16Twb_fixed ||
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 706-714
```cpp
        TableEntry->RealOpc == ARM::VST1d32Twb_fixed ||
        TableEntry->RealOpc == ARM::VST1d64Twb_fixed) {
      assert(AM6Offset.getReg() == 0 &&
             "A fixed writing-back pseudo instruction provides an offset "
             "register!");
    } else {
      MIB.add(AM6Offset);
    }
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 716-727
```cpp
  bool SrcIsKill = MI.getOperand(OpIdx).isKill();
  bool SrcIsUndef = MI.getOperand(OpIdx).isUndef();
  Register SrcReg = MI.getOperand(OpIdx++).getReg();
  MCRegister D0, D1, D2, D3;
  GetDSubRegs(SrcReg, RegSpc, TRI, D0, D1, D2, D3);
  MIB.addReg(D0, getUndefRegState(SrcIsUndef));
  if (NumRegs > 1 && TableEntry->copyAllListRegs)
    MIB.addReg(D1, getUndefRegState(SrcIsUndef));
  if (NumRegs > 2 && TableEntry->copyAllListRegs)
    MIB.addReg(D2, getUndefRegState(SrcIsUndef));
  if (NumRegs > 3 && TableEntry->copyAllListRegs)
    MIB.addReg(D3, getUndefRegState(SrcIsUndef));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 729-731
```cpp
  // Copy the predicate operands.
  MIB.add(MI.getOperand(OpIdx++));
  MIB.add(MI.getOperand(OpIdx++));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 733-737
```cpp
  if (SrcIsKill && !SrcIsUndef) // Add an implicit kill for the super-reg.
    MIB->addRegisterKilled(SrcReg, TRI, true);
  else if (!SrcIsUndef)
    MIB.addReg(SrcReg, RegState::Implicit); // Add implicit uses for src reg.
  MIB.copyImplicitOps(MI);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 739-743
```cpp
  // Transfer memoperands.
  MIB.cloneMemRefs(MI);
  MI.eraseFromParent();
  LLVM_DEBUG(dbgs() << "To:        "; MIB.getInstr()->dump(););
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 745-750
```cpp
/// ExpandLaneOp - Translate VLD*LN and VST*LN instructions with Q, QQ or QQQQ
/// register operands to real instructions with D register operands.
void ARMExpandPseudo::ExpandLaneOp(MachineBasicBlock::iterator &MBBI) {
  MachineInstr &MI = *MBBI;
  MachineBasicBlock &MBB = *MI.getParent();
  LLVM_DEBUG(dbgs() << "Expanding: "; MI.dump());
```
- EN: Implements `ARMExpandPseudo::ExpandLaneOp`, a mutation/build routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMExpandPseudo::ExpandLaneOp`，它是一个围绕机器基本块展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 752-756
```cpp
  const NEONLdStTableEntry *TableEntry = LookupNEONLdSt(MI.getOpcode());
  assert(TableEntry && "NEONLdStTable lookup failed");
  NEONRegSpacing RegSpc = (NEONRegSpacing)TableEntry->RegSpacing;
  unsigned NumRegs = TableEntry->NumRegs;
  unsigned RegElts = TableEntry->RegElts;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 758-763
```cpp
  MachineInstrBuilder MIB = BuildMI(MBB, MBBI, MI.getDebugLoc(),
                                    TII->get(TableEntry->RealOpc));
  unsigned OpIdx = 0;
  // The lane operand is always the 3rd from last operand, before the 2
  // predicate operands.
  unsigned Lane = MI.getOperand(MI.getDesc().getNumOperands() - 3).getImm();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 765-771
```cpp
  // Adjust the lane and spacing as needed for Q registers.
  assert(RegSpc != OddDblSpc && "unexpected register spacing for VLD/VST-lane");
  if (RegSpc == EvenDblSpc && Lane >= RegElts) {
    RegSpc = OddDblSpc;
    Lane -= RegElts;
  }
  assert(Lane < RegElts && "out of range lane for VLD/VST-lane");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 773-787
```cpp
  MCRegister D0, D1, D2, D3;
  unsigned DstReg = 0;
  bool DstIsDead = false;
  if (TableEntry->IsLoad) {
    DstIsDead = MI.getOperand(OpIdx).isDead();
    DstReg = MI.getOperand(OpIdx++).getReg();
    GetDSubRegs(DstReg, RegSpc, TRI, D0, D1, D2, D3);
    MIB.addReg(D0, RegState::Define | getDeadRegState(DstIsDead));
    if (NumRegs > 1)
      MIB.addReg(D1, RegState::Define | getDeadRegState(DstIsDead));
    if (NumRegs > 2)
      MIB.addReg(D2, RegState::Define | getDeadRegState(DstIsDead));
    if (NumRegs > 3)
      MIB.addReg(D3, RegState::Define | getDeadRegState(DstIsDead));
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 789-790
```cpp
  if (TableEntry->isUpdating)
    MIB.add(MI.getOperand(OpIdx++));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 792-797
```cpp
  // Copy the addrmode6 operands.
  MIB.add(MI.getOperand(OpIdx++));
  MIB.add(MI.getOperand(OpIdx++));
  // Copy the am6offset operand.
  if (TableEntry->hasWritebackOperand)
    MIB.add(MI.getOperand(OpIdx++));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 799-802
```cpp
  // Grab the super-register source.
  MachineOperand MO = MI.getOperand(OpIdx++);
  if (!TableEntry->IsLoad)
    GetDSubRegs(MO.getReg(), RegSpc, TRI, D0, D1, D2, D3);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 804-813
```cpp
  // Add the subregs as sources of the new instruction.
  RegState SrcFlags =
      (getUndefRegState(MO.isUndef()) | getKillRegState(MO.isKill()));
  MIB.addReg(D0, SrcFlags);
  if (NumRegs > 1)
    MIB.addReg(D1, SrcFlags);
  if (NumRegs > 2)
    MIB.addReg(D2, SrcFlags);
  if (NumRegs > 3)
    MIB.addReg(D3, SrcFlags);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 815-817
```cpp
  // Add the lane number operand.
  MIB.addImm(Lane);
  OpIdx += 1;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 819-821
```cpp
  // Copy the predicate operands.
  MIB.add(MI.getOperand(OpIdx++));
  MIB.add(MI.getOperand(OpIdx++));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 823-833
```cpp
  // Copy the super-register source to be an implicit source.
  MO.setImplicit(true);
  MIB.add(MO);
  if (TableEntry->IsLoad)
    // Add an implicit def for the super-register.
    MIB.addReg(DstReg, RegState::ImplicitDefine | getDeadRegState(DstIsDead));
  MIB.copyImplicitOps(MI);
  // Transfer memoperands.
  MIB.cloneMemRefs(MI);
  MI.eraseFromParent();
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 835-841
```cpp
/// ExpandVTBL - Translate VTBL and VTBX pseudo instructions with Q or QQ
/// register operands to real instructions with D register operands.
void ARMExpandPseudo::ExpandVTBL(MachineBasicBlock::iterator &MBBI,
                                 unsigned Opc, bool IsExt) {
  MachineInstr &MI = *MBBI;
  MachineBasicBlock &MBB = *MI.getParent();
  LLVM_DEBUG(dbgs() << "Expanding: "; MI.dump());
```
- EN: Implements `ARMExpandPseudo::ExpandVTBL`, a mutation/build routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMExpandPseudo::ExpandVTBL`，它是一个围绕机器基本块展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 843-844
```cpp
  MachineInstrBuilder MIB = BuildMI(MBB, MBBI, MI.getDebugLoc(), TII->get(Opc));
  unsigned OpIdx = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 846-851
```cpp
  // Transfer the destination register operand.
  MIB.add(MI.getOperand(OpIdx++));
  if (IsExt) {
    MachineOperand VdSrc(MI.getOperand(OpIdx++));
    MIB.add(VdSrc);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 853-857
```cpp
  bool SrcIsKill = MI.getOperand(OpIdx).isKill();
  Register SrcReg = MI.getOperand(OpIdx++).getReg();
  MCRegister D0, D1, D2, D3;
  GetDSubRegs(SrcReg, SingleSpc, TRI, D0, D1, D2, D3);
  MIB.addReg(D0);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 859-861
```cpp
  // Copy the other source register operand.
  MachineOperand VmSrc(MI.getOperand(OpIdx++));
  MIB.add(VmSrc);
```
- EN: Declares `VmSrc`, a target-specific routine centered on machine operands. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `VmSrc`，它是一个围绕机器操作数展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 863-865
```cpp
  // Copy the predicate operands.
  MIB.add(MI.getOperand(OpIdx++));
  MIB.add(MI.getOperand(OpIdx++));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 867-872
```cpp
  // Add an implicit kill and use for the super-reg.
  MIB.addReg(SrcReg, RegState::Implicit | getKillRegState(SrcIsKill));
  MIB.copyImplicitOps(MI);
  MI.eraseFromParent();
  LLVM_DEBUG(dbgs() << "To:        "; MIB.getInstr()->dump(););
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 874-882
```cpp
void ARMExpandPseudo::ExpandMQQPRLoadStore(MachineBasicBlock::iterator &MBBI) {
  MachineInstr &MI = *MBBI;
  MachineBasicBlock &MBB = *MI.getParent();
  unsigned NewOpc =
      MI.getOpcode() == ARM::MQQPRStore || MI.getOpcode() == ARM::MQQQQPRStore
          ? ARM::VSTMDIA
          : ARM::VLDMDIA;
  MachineInstrBuilder MIB =
      BuildMI(MBB, MBBI, MI.getDebugLoc(), TII->get(NewOpc));
```
- EN: Implements `ARMExpandPseudo::ExpandMQQPRLoadStore`, a mutation/build routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMExpandPseudo::ExpandMQQPRLoadStore`，它是一个围绕机器基本块展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 884-886
```cpp
  RegState Flags = getKillRegState(MI.getOperand(0).isKill()) |
                   getDefRegState(MI.getOperand(0).isDef());
  Register SrcReg = MI.getOperand(0).getReg();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 888-901
```cpp
  // Copy the destination register.
  MIB.add(MI.getOperand(1));
  MIB.add(predOps(ARMCC::AL));
  MIB.addReg(TRI->getSubReg(SrcReg, ARM::dsub_0), Flags);
  MIB.addReg(TRI->getSubReg(SrcReg, ARM::dsub_1), Flags);
  MIB.addReg(TRI->getSubReg(SrcReg, ARM::dsub_2), Flags);
  MIB.addReg(TRI->getSubReg(SrcReg, ARM::dsub_3), Flags);
  if (MI.getOpcode() == ARM::MQQQQPRStore ||
      MI.getOpcode() == ARM::MQQQQPRLoad) {
    MIB.addReg(TRI->getSubReg(SrcReg, ARM::dsub_4), Flags);
    MIB.addReg(TRI->getSubReg(SrcReg, ARM::dsub_5), Flags);
    MIB.addReg(TRI->getSubReg(SrcReg, ARM::dsub_6), Flags);
    MIB.addReg(TRI->getSubReg(SrcReg, ARM::dsub_7), Flags);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 903-904
```cpp
  if (NewOpc == ARM::VSTMDIA)
    MIB.addReg(SrcReg, RegState::Implicit);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 906-909
```cpp
  MIB.copyImplicitOps(MI);
  MIB.cloneMemRefs(MI);
  MI.eraseFromParent();
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 911-925
```cpp
static bool IsAnAddressOperand(const MachineOperand &MO) {
  // This check is overly conservative.  Unless we are certain that the machine
  // operand is not a symbol reference, we return that it is a symbol reference.
  // This is important as the load pair may not be split up Windows.
  switch (MO.getType()) {
  case MachineOperand::MO_Register:
  case MachineOperand::MO_Immediate:
  case MachineOperand::MO_CImmediate:
  case MachineOperand::MO_FPImmediate:
  case MachineOperand::MO_ShuffleMask:
    return false;
  case MachineOperand::MO_MachineBasicBlock:
    return true;
  case MachineOperand::MO_FrameIndex:
    return false;
```
- EN: Implements `IsAnAddressOperand`, a query/helper routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `IsAnAddressOperand`，它是一个围绕机器基本块展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 926-940
```cpp
  case MachineOperand::MO_ConstantPoolIndex:
  case MachineOperand::MO_TargetIndex:
  case MachineOperand::MO_JumpTableIndex:
  case MachineOperand::MO_ExternalSymbol:
  case MachineOperand::MO_GlobalAddress:
  case MachineOperand::MO_BlockAddress:
    return true;
  case MachineOperand::MO_RegisterMask:
  case MachineOperand::MO_RegisterLiveOut:
  case MachineOperand::MO_LaneMask:
    return false;
  case MachineOperand::MO_Metadata:
  case MachineOperand::MO_MCSymbol:
    return true;
  case MachineOperand::MO_DbgInstrRef:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 941-948
```cpp
  case MachineOperand::MO_CFIIndex:
    return false;
  case MachineOperand::MO_IntrinsicID:
  case MachineOperand::MO_Predicate:
    llvm_unreachable("should not exist post-isel");
  }
  llvm_unreachable("unhandled machine operand type");
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 950-954
```cpp
static MachineOperand makeImplicit(const MachineOperand &MO) {
  MachineOperand NewMO = MO;
  NewMO.setImplicit();
  return NewMO;
}
```
- EN: Implements `makeImplicit`, a target-specific routine centered on machine operands. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `makeImplicit`，它是一个围绕机器操作数展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 956-970
```cpp
static MachineOperand getMovOperand(const MachineOperand &MO,
                                    unsigned TargetFlag) {
  unsigned TF = MO.getTargetFlags() | TargetFlag;
  switch (MO.getType()) {
  case MachineOperand::MO_Immediate: {
    unsigned Imm = MO.getImm();
    switch (TargetFlag) {
    case ARMII::MO_HI_8_15:
      Imm = (Imm >> 24) & 0xff;
      break;
    case ARMII::MO_HI_0_7:
      Imm = (Imm >> 16) & 0xff;
      break;
    case ARMII::MO_LO_8_15:
      Imm = (Imm >> 8) & 0xff;
```
- EN: Implements `getMovOperand`, a query/helper routine centered on machine operands. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getMovOperand`，它是一个围绕机器操作数展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 971-985
```cpp
      break;
    case ARMII::MO_LO_0_7:
      Imm = Imm & 0xff;
      break;
    case ARMII::MO_HI16:
      Imm = (Imm >> 16) & 0xffff;
      break;
    case ARMII::MO_LO16:
      Imm = Imm & 0xffff;
      break;
    default:
      llvm_unreachable("Only HI/LO target flags are expected");
    }
    return MachineOperand::CreateImm(Imm);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 986-993
```cpp
  case MachineOperand::MO_ExternalSymbol:
    return MachineOperand::CreateES(MO.getSymbolName(), TF);
  case MachineOperand::MO_JumpTableIndex:
    return MachineOperand::CreateJTI(MO.getIndex(), TF);
  default:
    return MachineOperand::CreateGA(MO.getGlobal(), MO.getOffset(), TF);
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 995-1001
```cpp
void ARMExpandPseudo::ExpandTMOV32BitImm(MachineBasicBlock &MBB,
                                         MachineBasicBlock::iterator &MBBI) {
  MachineInstr &MI = *MBBI;
  Register DstReg = MI.getOperand(0).getReg();
  bool DstIsDead = MI.getOperand(0).isDead();
  const MachineOperand &MO = MI.getOperand(1);
  unsigned MIFlags = MI.getFlags();
```
- EN: Implements `ARMExpandPseudo::ExpandTMOV32BitImm`, a mutation/build routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMExpandPseudo::ExpandTMOV32BitImm`，它是一个围绕机器基本块展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1003-1003
```cpp
  LLVM_DEBUG(dbgs() << "Expanding: "; MI.dump());
```
- EN: Declares `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1005-1018
```cpp
  // Expand the mov into a sequence of mov/add+lsl of the individual bytes. We
  // want to avoid emitting any zero bytes, as they won't change the result, and
  // also don't want any pointless shifts, so instead of immediately emitting
  // the shift for a byte we keep track of how much we will need to shift and do
  // it before the next nonzero byte.
  unsigned PendingShift = 0;
  for (unsigned Byte = 0; Byte < 4; ++Byte) {
    unsigned Flag = Byte == 0   ? ARMII::MO_HI_8_15
                    : Byte == 1 ? ARMII::MO_HI_0_7
                    : Byte == 2 ? ARMII::MO_LO_8_15
                                : ARMII::MO_LO_0_7;
    MachineOperand Operand = getMovOperand(MO, Flag);
    bool ZeroImm = Operand.isImm() && Operand.getImm() == 0;
    unsigned Op = PendingShift ? ARM::tADDi8 : ARM::tMOVi8;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1020-1033
```cpp
    // Emit the pending shift if we're going to emit this byte or if we've
    // reached the end.
    if (PendingShift && (!ZeroImm || Byte == 3)) {
      MachineInstr *Lsl =
          BuildMI(MBB, MBBI, MI.getDebugLoc(), TII->get(ARM::tLSLri), DstReg)
              .add(t1CondCodeOp(true))
              .addReg(DstReg)
              .addImm(PendingShift)
              .add(predOps(ARMCC::AL))
              .setMIFlags(MIFlags);
      (void)Lsl;
      LLVM_DEBUG(dbgs() << "And:       "; Lsl->dump(););
      PendingShift = 0;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1035-1047
```cpp
    // Emit this byte if it's nonzero.
    if (!ZeroImm) {
      MachineInstrBuilder MIB =
          BuildMI(MBB, MBBI, MI.getDebugLoc(), TII->get(Op), DstReg)
              .add(t1CondCodeOp(true));
      if (Op == ARM::tADDi8)
        MIB.addReg(DstReg);
      MIB.add(Operand);
      MIB.add(predOps(ARMCC::AL));
      MIB.setMIFlags(MIFlags);
      LLVM_DEBUG(dbgs() << (Op == ARM::tMOVi8 ? "To: " : "And:") << "       ";
                 MIB.getInstr()->dump(););
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1049-1052
```cpp
    // Don't accumulate the shift value if we've not yet seen a nonzero byte.
    if (PendingShift || !ZeroImm)
      PendingShift += 8;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1054-1056
```cpp
  // The dest is dead on the last instruction we emitted if it was dead on the
  // original instruction.
  (--MBBI)->getOperand(0).setIsDead(DstIsDead);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1058-1059
```cpp
  MI.eraseFromParent();
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1061-1073
```cpp
void ARMExpandPseudo::ExpandMOV32BitImm(MachineBasicBlock &MBB,
                                        MachineBasicBlock::iterator &MBBI) {
  MachineInstr &MI = *MBBI;
  unsigned Opcode = MI.getOpcode();
  Register PredReg;
  ARMCC::CondCodes Pred = getInstrPredicate(MI, PredReg);
  Register DstReg = MI.getOperand(0).getReg();
  bool DstIsDead = MI.getOperand(0).isDead();
  bool isCC = Opcode == ARM::MOVCCi32imm || Opcode == ARM::t2MOVCCi32imm;
  const MachineOperand &MO = MI.getOperand(isCC ? 2 : 1);
  bool RequiresBundling = STI->isTargetWindows() && IsAnAddressOperand(MO);
  MachineInstrBuilder LO16, HI16;
  LLVM_DEBUG(dbgs() << "Expanding: "; MI.dump());
```
- EN: Implements `ARMExpandPseudo::ExpandMOV32BitImm`, a mutation/build routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMExpandPseudo::ExpandMOV32BitImm`，它是一个围绕机器基本块展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1075-1078
```cpp
  if (!STI->hasV6T2Ops() &&
      (Opcode == ARM::MOVi32imm || Opcode == ARM::MOVCCi32imm)) {
    // FIXME Windows CE supports older ARM CPUs
    assert(!STI->isTargetWindows() && "Windows on ARM requires ARMv7+");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1080-1082
```cpp
    assert (MO.isImm() && "MOVi32imm w/ non-immediate source operand!");
    unsigned ImmVal = (unsigned)MO.getImm();
    unsigned SOImmValV1 = 0, SOImmValV2 = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1084-1098
```cpp
    if (ARM_AM::isSOImmTwoPartVal(ImmVal)) { // Expand into a movi + orr.
      LO16 = BuildMI(MBB, MBBI, MI.getDebugLoc(), TII->get(ARM::MOVi), DstReg);
      HI16 = BuildMI(MBB, MBBI, MI.getDebugLoc(), TII->get(ARM::ORRri))
          .addReg(DstReg, RegState::Define | getDeadRegState(DstIsDead))
          .addReg(DstReg);
      SOImmValV1 = ARM_AM::getSOImmTwoPartFirst(ImmVal);
      SOImmValV2 = ARM_AM::getSOImmTwoPartSecond(ImmVal);
    } else { // Expand into a mvn + sub.
      LO16 = BuildMI(MBB, MBBI, MI.getDebugLoc(), TII->get(ARM::MVNi), DstReg);
      HI16 = BuildMI(MBB, MBBI, MI.getDebugLoc(), TII->get(ARM::SUBri))
          .addReg(DstReg, RegState::Define | getDeadRegState(DstIsDead))
          .addReg(DstReg);
      SOImmValV1 = ARM_AM::getSOImmTwoPartFirst(-ImmVal);
      SOImmValV2 = ARM_AM::getSOImmTwoPartSecond(-ImmVal);
      SOImmValV1 = ~(-SOImmValV1);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1099-1099
```cpp
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1101-1115
```cpp
    unsigned MIFlags = MI.getFlags();
    LO16 = LO16.addImm(SOImmValV1);
    HI16 = HI16.addImm(SOImmValV2);
    LO16.cloneMemRefs(MI);
    HI16.cloneMemRefs(MI);
    LO16.setMIFlags(MIFlags);
    HI16.setMIFlags(MIFlags);
    LO16.addImm(Pred).addReg(PredReg).add(condCodeOp());
    HI16.addImm(Pred).addReg(PredReg).add(condCodeOp());
    if (isCC)
      LO16.add(makeImplicit(MI.getOperand(1)));
    LO16.copyImplicitOps(MI);
    HI16.copyImplicitOps(MI);
    MI.eraseFromParent();
    return;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1116-1116
```cpp
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1118-1127
```cpp
  unsigned LO16Opc = 0;
  unsigned HI16Opc = 0;
  unsigned MIFlags = MI.getFlags();
  if (Opcode == ARM::t2MOVi32imm || Opcode == ARM::t2MOVCCi32imm) {
    LO16Opc = ARM::t2MOVi16;
    HI16Opc = ARM::t2MOVTi16;
  } else {
    LO16Opc = ARM::MOVi16;
    HI16Opc = ARM::MOVTi16;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1129-1137
```cpp
  LO16 = BuildMI(MBB, MBBI, MI.getDebugLoc(), TII->get(LO16Opc), DstReg);
  LO16.setMIFlags(MIFlags);
  LO16.add(getMovOperand(MO, ARMII::MO_LO16));
  LO16.cloneMemRefs(MI);
  LO16.addImm(Pred).addReg(PredReg);
  if (isCC)
    LO16.add(makeImplicit(MI.getOperand(1)));
  LO16.copyImplicitOps(MI);
  LLVM_DEBUG(dbgs() << "To:        "; LO16.getInstr()->dump(););
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1139-1152
```cpp
  MachineOperand HIOperand = getMovOperand(MO, ARMII::MO_HI16);
  if (!(HIOperand.isImm() && HIOperand.getImm() == 0)) {
    HI16 = BuildMI(MBB, MBBI, MI.getDebugLoc(), TII->get(HI16Opc))
               .addReg(DstReg, RegState::Define | getDeadRegState(DstIsDead))
               .addReg(DstReg);
    HI16.setMIFlags(MIFlags);
    HI16.add(HIOperand);
    HI16.cloneMemRefs(MI);
    HI16.addImm(Pred).addReg(PredReg);
    HI16.copyImplicitOps(MI);
    LLVM_DEBUG(dbgs() << "And:       "; HI16.getInstr()->dump(););
  } else {
    LO16->getOperand(0).setIsDead(DstIsDead);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1154-1155
```cpp
  if (RequiresBundling)
    finalizeBundle(MBB, LO16->getIterator(), MBBI->getIterator());
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1157-1158
```cpp
  MI.eraseFromParent();
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1160-1162
```cpp
// The size of the area, accessed by that VLSTM/VLLDM
// S0-S31 + FPSCR + 8 more bytes (VPR + pad, or just pad)
static const int CMSE_FP_SAVE_SIZE = 136;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1164-1173
```cpp
static void determineGPRegsToClear(const MachineInstr &MI,
                                   const std::initializer_list<unsigned> &Regs,
                                   SmallVectorImpl<unsigned> &ClearRegs) {
  SmallVector<unsigned, 4> OpRegs;
  for (const MachineOperand &Op : MI.operands()) {
    if (!Op.isReg() || !Op.isUse())
      continue;
    OpRegs.push_back(Op.getReg());
  }
  llvm::sort(OpRegs);
```
- EN: Implements `determineGPRegsToClear`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `determineGPRegsToClear`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1175-1177
```cpp
  std::set_difference(Regs.begin(), Regs.end(), OpRegs.begin(), OpRegs.end(),
                      std::back_inserter(ClearRegs));
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1179-1182
```cpp
void ARMExpandPseudo::CMSEClearGPRegs(
    MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
    const DebugLoc &DL, const SmallVectorImpl<unsigned> &ClearRegs,
    unsigned ClobberReg) {
```
- EN: Implements `ARMExpandPseudo::CMSEClearGPRegs`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMExpandPseudo::CMSEClearGPRegs`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1184-1198
```cpp
  if (STI->hasV8_1MMainlineOps()) {
    // Clear the registers using the CLRM instruction.
    MachineInstrBuilder CLRM =
        BuildMI(MBB, MBBI, DL, TII->get(ARM::t2CLRM)).add(predOps(ARMCC::AL));
    for (unsigned R : ClearRegs)
      CLRM.addReg(R, RegState::Define);
    CLRM.addReg(ARM::APSR, RegState::Define);
    CLRM.addReg(ARM::CPSR, RegState::Define | RegState::Implicit);
  } else {
    // Clear the registers and flags by copying ClobberReg into them.
    // (Baseline can't do a high register clear in one instruction).
    for (unsigned Reg : ClearRegs) {
      if (Reg == ClobberReg)
        continue;
      BuildMI(MBB, MBBI, DL, TII->get(ARM::tMOVr), Reg)
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1199-1201
```cpp
          .addReg(ClobberReg)
          .add(predOps(ARMCC::AL));
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1203-1208
```cpp
    BuildMI(MBB, MBBI, DL, TII->get(ARM::t2MSR_M))
        .addImm(STI->hasDSP() ? 0xc00 : 0x800)
        .addReg(ClobberReg)
        .add(predOps(ARMCC::AL));
  }
}
```
- EN: Declares `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1210-1219
```cpp
// Find which FP registers need to be cleared.  The parameter `ClearRegs` is
// initialised with all elements set to true, and this function resets all the
// bits, which correspond to register uses. Returns true if any floating point
// register is defined, false otherwise.
static bool determineFPRegsToClear(const MachineInstr &MI,
                                   BitVector &ClearRegs) {
  bool DefFP = false;
  for (const MachineOperand &Op : MI.operands()) {
    if (!Op.isReg())
      continue;
```
- EN: Implements `determineFPRegsToClear`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `determineFPRegsToClear`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1221-1228
```cpp
    Register Reg = Op.getReg();
    if (Op.isDef()) {
      if ((Reg >= ARM::Q0 && Reg <= ARM::Q7) ||
          (Reg >= ARM::D0 && Reg <= ARM::D15) ||
          (Reg >= ARM::S0 && Reg <= ARM::S31))
        DefFP = true;
      continue;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1230-1241
```cpp
    if (Reg >= ARM::Q0 && Reg <= ARM::Q7) {
      int R = Reg - ARM::Q0;
      ClearRegs.reset(R * 4, (R + 1) * 4);
    } else if (Reg >= ARM::D0 && Reg <= ARM::D15) {
      int R = Reg - ARM::D0;
      ClearRegs.reset(R * 2, (R + 1) * 2);
    } else if (Reg >= ARM::S0 && Reg <= ARM::S31) {
      ClearRegs[Reg - ARM::S0] = false;
    }
  }
  return DefFP;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1243-1247
```cpp
MachineBasicBlock &
ARMExpandPseudo::CMSEClearFPRegs(MachineBasicBlock &MBB,
                                 MachineBasicBlock::iterator MBBI) {
  BitVector ClearRegs(16, true);
  (void)determineFPRegsToClear(*MBBI, ClearRegs);
```
- EN: Implements `ARMExpandPseudo::CMSEClearFPRegs`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMExpandPseudo::CMSEClearFPRegs`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1249-1253
```cpp
  if (STI->hasV8_1MMainlineOps())
    return CMSEClearFPRegsV81(MBB, MBBI, ClearRegs);
  else
    return CMSEClearFPRegsV8(MBB, MBBI, ClearRegs);
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1255-1262
```cpp
// Clear the FP registers for v8.0-M, by copying over the content
// of LR. Uses R12 as a scratch register.
MachineBasicBlock &
ARMExpandPseudo::CMSEClearFPRegsV8(MachineBasicBlock &MBB,
                                   MachineBasicBlock::iterator MBBI,
                                   const BitVector &ClearRegs) {
  if (!STI->hasFPRegs())
    return MBB;
```
- EN: Implements `ARMExpandPseudo::CMSEClearFPRegsV8`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMExpandPseudo::CMSEClearFPRegsV8`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1264-1265
```cpp
  auto &RetI = *MBBI;
  const DebugLoc &DL = RetI.getDebugLoc();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1267-1276
```cpp
  // If optimising for minimum size, clear FP registers unconditionally.
  // Otherwise, check the CONTROL.SFPA (Secure Floating-Point Active) bit and
  // don't clear them if they belong to the non-secure state.
  MachineBasicBlock *ClearBB, *DoneBB;
  if (STI->hasMinSize()) {
    ClearBB = DoneBB = &MBB;
  } else {
    MachineFunction *MF = MBB.getParent();
    ClearBB = MF->CreateMachineBasicBlock(MBB.getBasicBlock());
    DoneBB = MF->CreateMachineBasicBlock(MBB.getBasicBlock());
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1278-1279
```cpp
    MF->insert(++MBB.getIterator(), ClearBB);
    MF->insert(++ClearBB->getIterator(), DoneBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1281-1285
```cpp
    DoneBB->splice(DoneBB->end(), &MBB, MBBI, MBB.end());
    DoneBB->transferSuccessors(&MBB);
    MBB.addSuccessor(ClearBB);
    MBB.addSuccessor(DoneBB);
    ClearBB->addSuccessor(DoneBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1287-1300
```cpp
    // At the new basic blocks we need to have live-in the registers, used
    // for the return value as well as LR, used to clear registers.
    for (const MachineOperand &Op : RetI.operands()) {
      if (!Op.isReg())
        continue;
      Register Reg = Op.getReg();
      if (Reg == ARM::NoRegister || Reg == ARM::LR)
        continue;
      assert(Reg.isPhysical() && "Unallocated register");
      ClearBB->addLiveIn(Reg);
      DoneBB->addLiveIn(Reg);
    }
    ClearBB->addLiveIn(ARM::LR);
    DoneBB->addLiveIn(ARM::LR);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1302-1316
```cpp
    // Read the CONTROL register.
    BuildMI(MBB, MBB.end(), DL, TII->get(ARM::t2MRS_M), ARM::R12)
        .addImm(20)
        .add(predOps(ARMCC::AL));
    // Check bit 3 (SFPA).
    BuildMI(MBB, MBB.end(), DL, TII->get(ARM::t2TSTri))
        .addReg(ARM::R12)
        .addImm(8)
        .add(predOps(ARMCC::AL));
    // If SFPA is clear, jump over ClearBB to DoneBB.
    BuildMI(MBB, MBB.end(), DL, TII->get(ARM::tBcc))
        .addMBB(DoneBB)
        .addImm(ARMCC::EQ)
        .addReg(ARM::CPSR, RegState::Kill);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1318-1332
```cpp
  // Emit the clearing sequence
  for (unsigned D = 0; D < 8; D++) {
    // Attempt to clear as double
    if (ClearRegs[D * 2 + 0] && ClearRegs[D * 2 + 1]) {
      unsigned Reg = ARM::D0 + D;
      BuildMI(ClearBB, DL, TII->get(ARM::VMOVDRR), Reg)
          .addReg(ARM::LR)
          .addReg(ARM::LR)
          .add(predOps(ARMCC::AL));
    } else {
      // Clear first part as single
      if (ClearRegs[D * 2 + 0]) {
        unsigned Reg = ARM::S0 + D * 2;
        BuildMI(ClearBB, DL, TII->get(ARM::VMOVSR), Reg)
            .addReg(ARM::LR)
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1333-1343
```cpp
            .add(predOps(ARMCC::AL));
      }
      // Clear second part as single
      if (ClearRegs[D * 2 + 1]) {
        unsigned Reg = ARM::S0 + D * 2 + 1;
        BuildMI(ClearBB, DL, TII->get(ARM::VMOVSR), Reg)
            .addReg(ARM::LR)
            .add(predOps(ARMCC::AL));
      }
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1345-1359
```cpp
  // Clear FPSCR bits 0-4, 7, 28-31
  // The other bits are program global according to the AAPCS
  BuildMI(ClearBB, DL, TII->get(ARM::VMRS), ARM::R12)
      .add(predOps(ARMCC::AL));
  BuildMI(ClearBB, DL, TII->get(ARM::t2BICri), ARM::R12)
      .addReg(ARM::R12)
      .addImm(0x0000009F)
      .add(predOps(ARMCC::AL))
      .add(condCodeOp());
  BuildMI(ClearBB, DL, TII->get(ARM::t2BICri), ARM::R12)
      .addReg(ARM::R12)
      .addImm(0xF0000000)
      .add(predOps(ARMCC::AL))
      .add(condCodeOp());
  BuildMI(ClearBB, DL, TII->get(ARM::VMSR))
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1360-1361
```cpp
      .addReg(ARM::R12)
      .add(predOps(ARMCC::AL));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1363-1364
```cpp
  return *DoneBB;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1366-1370
```cpp
MachineBasicBlock &
ARMExpandPseudo::CMSEClearFPRegsV81(MachineBasicBlock &MBB,
                                    MachineBasicBlock::iterator MBBI,
                                    const BitVector &ClearRegs) {
  auto &RetI = *MBBI;
```
- EN: Implements `ARMExpandPseudo::CMSEClearFPRegsV81`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMExpandPseudo::CMSEClearFPRegsV81`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1372-1386
```cpp
  // Emit a sequence of VSCCLRM <sreglist> instructions, one instruction for
  // each contiguous sequence of S-registers.
  int Start = -1, End = -1;
  for (int S = 0, E = ClearRegs.size(); S != E; ++S) {
    if (ClearRegs[S] && S == End + 1) {
      End = S; // extend range
      continue;
    }
    // Emit current range.
    if (Start < End) {
      MachineInstrBuilder VSCCLRM =
          BuildMI(MBB, MBBI, RetI.getDebugLoc(), TII->get(ARM::VSCCLRMS))
              .add(predOps(ARMCC::AL));
      while (++Start <= End)
        VSCCLRM.addReg(ARM::S0 + Start, RegState::Define);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1387-1399
```cpp
      VSCCLRM.addReg(ARM::VPR, RegState::Define);
    }
    Start = End = S;
  }
  // Emit last range.
  if (Start < End) {
    MachineInstrBuilder VSCCLRM =
        BuildMI(MBB, MBBI, RetI.getDebugLoc(), TII->get(ARM::VSCCLRMS))
            .add(predOps(ARMCC::AL));
    while (++Start <= End)
      VSCCLRM.addReg(ARM::S0 + Start, RegState::Define);
    VSCCLRM.addReg(ARM::VPR, RegState::Define);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1401-1402
```cpp
  return MBB;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1404-1411
```cpp
void ARMExpandPseudo::CMSESaveClearFPRegs(
    MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI, DebugLoc &DL,
    const LivePhysRegs &LiveRegs, SmallVectorImpl<unsigned> &ScratchRegs) {
  if (STI->hasV8_1MMainlineOps())
    CMSESaveClearFPRegsV81(MBB, MBBI, DL, LiveRegs);
  else if (STI->hasV8MMainlineOps())
    CMSESaveClearFPRegsV8(MBB, MBBI, DL, LiveRegs, ScratchRegs);
}
```
- EN: Implements `ARMExpandPseudo::CMSESaveClearFPRegs`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMExpandPseudo::CMSESaveClearFPRegs`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1413-1416
```cpp
// Save and clear FP registers if present
void ARMExpandPseudo::CMSESaveClearFPRegsV8(
    MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI, DebugLoc &DL,
    const LivePhysRegs &LiveRegs, SmallVectorImpl<unsigned> &ScratchRegs) {
```
- EN: Implements `ARMExpandPseudo::CMSESaveClearFPRegsV8`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMExpandPseudo::CMSESaveClearFPRegsV8`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1418-1420
```cpp
  // Store an available register for FPSCR clearing
  assert(!ScratchRegs.empty());
  unsigned SpareReg = ScratchRegs.front();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1422-1426
```cpp
  // save space on stack for VLSTM
  BuildMI(MBB, MBBI, DL, TII->get(ARM::tSUBspi), ARM::SP)
      .addReg(ARM::SP)
      .addImm(CMSE_FP_SAVE_SIZE >> 2)
      .add(predOps(ARMCC::AL));
```
- EN: Declares `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1428-1442
```cpp
  // Use ScratchRegs to store the fp regs
  std::vector<std::tuple<unsigned, unsigned, unsigned>> ClearedFPRegs;
  std::vector<unsigned> NonclearedFPRegs;
  bool ReturnsFPReg = false;
  for (const MachineOperand &Op : MBBI->operands()) {
    if (Op.isReg() && Op.isUse()) {
      Register Reg = Op.getReg();
      assert(!ARM::DPRRegClass.contains(Reg) ||
             ARM::DPR_VFP2RegClass.contains(Reg));
      assert(!ARM::QPRRegClass.contains(Reg));
      if (ARM::DPR_VFP2RegClass.contains(Reg)) {
        if (ScratchRegs.size() >= 2) {
          unsigned SaveReg2 = ScratchRegs.pop_back_val();
          unsigned SaveReg1 = ScratchRegs.pop_back_val();
          ClearedFPRegs.emplace_back(Reg, SaveReg1, SaveReg2);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1444-1456
```cpp
          // Save the fp register to the normal registers
          BuildMI(MBB, MBBI, DL, TII->get(ARM::VMOVRRD))
              .addReg(SaveReg1, RegState::Define)
              .addReg(SaveReg2, RegState::Define)
              .addReg(Reg)
              .add(predOps(ARMCC::AL));
        } else {
          NonclearedFPRegs.push_back(Reg);
        }
      } else if (ARM::SPRRegClass.contains(Reg)) {
        if (ScratchRegs.size() >= 1) {
          unsigned SaveReg = ScratchRegs.pop_back_val();
          ClearedFPRegs.emplace_back(Reg, SaveReg, 0);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1458-1472
```cpp
          // Save the fp register to the normal registers
          BuildMI(MBB, MBBI, DL, TII->get(ARM::VMOVRS), SaveReg)
              .addReg(Reg)
              .add(predOps(ARMCC::AL));
        } else {
          NonclearedFPRegs.push_back(Reg);
        }
      }
    } else if (Op.isReg() && Op.isDef()) {
      Register Reg = Op.getReg();
      if (ARM::SPRRegClass.contains(Reg) || ARM::DPRRegClass.contains(Reg) ||
          ARM::QPRRegClass.contains(Reg))
        ReturnsFPReg = true;
    }
  }
```
- EN: Implements `BuildMI`, a mutation/build routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `BuildMI`，它是一个围绕寄存器管理展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1474-1474
```cpp
  bool PassesFPReg = (!NonclearedFPRegs.empty() || !ClearedFPRegs.empty());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1476-1477
```cpp
  if (PassesFPReg || ReturnsFPReg)
    assert(STI->hasFPRegs() && "Subtarget needs fpregs");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1479-1493
```cpp
  // CVE-2024-7883
  //
  // The VLLDM/VLSTM instructions set up lazy state preservation, but they
  // execute as NOPs if the FP register file is not considered to contain
  // secure data, represented by the CONTROL_S.SFPA bit. This means that the
  // state of CONTROL_S.SFPA must be the same when these two instructions are
  // executed. That might not be the case if we haven't used any FP
  // instructions before the VLSTM, so CONTROL_S.SFPA is clear, but do have one
  // before the VLLDM, which sets it..
  //
  // If we can't prove that SFPA will be the same for the VLSTM and VLLDM, we
  // execute a "vmov s0, s0" instruction before the VLSTM to ensure that
  // CONTROL_S.SFPA is set for both.
  //
  // That can only happen for callees which take no FP arguments (or we'd have
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 1494-1508
```cpp
  // inserted a VMOV above) and which return values in FP regs (so that we need
  // to use a VMOV to back-up the return value before the VLLDM). It also can't
  // happen if the call is dominated by other existing floating-point
  // instructions, but we don't currently check for that case.
  //
  // These conditions mean that we only emit this instruction when using the
  // hard-float ABI, which means we can assume that FP instructions are
  // available, and don't need to make it conditional like we do for the
  // CVE-2021-35465 workaround.
  if (ReturnsFPReg && !PassesFPReg) {
    bool S0Dead = !LiveRegs.contains(ARM::S0);
    BuildMI(MBB, MBBI, DL, TII->get(ARM::VMOVS))
        .addReg(ARM::S0, RegState::Define | getDeadRegState(S0Dead))
        .addReg(ARM::S0, getUndefRegState(S0Dead))
        .add(predOps(ARMCC::AL));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1509-1509
```cpp
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1511-1525
```cpp
  // Lazy store all fp registers to the stack.
  // This executes as NOP in the absence of floating-point support.
  MachineInstrBuilder VLSTM =
      BuildMI(MBB, MBBI, DL, TII->get(ARM::VLSTM))
          .addReg(ARM::SP)
          .add(predOps(ARMCC::AL))
          .addImm(0); // Represents a pseoudo register list, has no effect on
                      // the encoding.
  // Mark non-live registers as undef
  for (MachineOperand &MO : VLSTM->implicit_operands()) {
    if (MO.isReg() && !MO.isDef()) {
      Register Reg = MO.getReg();
      MO.setIsUndef(!LiveRegs.contains(Reg));
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1527-1540
```cpp
  // Restore all arguments
  for (const auto &Regs : ClearedFPRegs) {
    unsigned Reg, SaveReg1, SaveReg2;
    std::tie(Reg, SaveReg1, SaveReg2) = Regs;
    if (ARM::DPR_VFP2RegClass.contains(Reg))
      BuildMI(MBB, MBBI, DL, TII->get(ARM::VMOVDRR), Reg)
          .addReg(SaveReg1)
          .addReg(SaveReg2)
          .add(predOps(ARMCC::AL));
    else if (ARM::SPRRegClass.contains(Reg))
      BuildMI(MBB, MBBI, DL, TII->get(ARM::VMOVSR), Reg)
          .addReg(SaveReg1)
          .add(predOps(ARMCC::AL));
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1542-1556
```cpp
  for (unsigned Reg : NonclearedFPRegs) {
    if (ARM::DPR_VFP2RegClass.contains(Reg)) {
      if (STI->isLittle()) {
        BuildMI(MBB, MBBI, DL, TII->get(ARM::VLDRD), Reg)
            .addReg(ARM::SP)
            .addImm((Reg - ARM::D0) * 2)
            .add(predOps(ARMCC::AL));
      } else {
        // For big-endian targets we need to load the two subregisters of Reg
        // manually because VLDRD would load them in wrong order
        MCRegister SReg0 = TRI->getSubReg(Reg, ARM::ssub_0);
        BuildMI(MBB, MBBI, DL, TII->get(ARM::VLDRS), SReg0)
            .addReg(ARM::SP)
            .addImm((Reg - ARM::D0) * 2)
            .add(predOps(ARMCC::AL));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1557-1571
```cpp
        BuildMI(MBB, MBBI, DL, TII->get(ARM::VLDRS), SReg0 + 1)
            .addReg(ARM::SP)
            .addImm((Reg - ARM::D0) * 2 + 1)
            .add(predOps(ARMCC::AL));
      }
    } else if (ARM::SPRRegClass.contains(Reg)) {
      BuildMI(MBB, MBBI, DL, TII->get(ARM::VLDRS), Reg)
          .addReg(ARM::SP)
          .addImm(Reg - ARM::S0)
          .add(predOps(ARMCC::AL));
    }
  }
  // restore FPSCR from stack and clear bits 0-4, 7, 28-31
  // The other bits are program global according to the AAPCS
  if (PassesFPReg) {
```
- EN: Declares `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1572-1586
```cpp
    BuildMI(MBB, MBBI, DL, TII->get(ARM::tLDRspi), SpareReg)
        .addReg(ARM::SP)
        .addImm(0x10)
        .add(predOps(ARMCC::AL));
    BuildMI(MBB, MBBI, DL, TII->get(ARM::t2BICri), SpareReg)
        .addReg(SpareReg)
        .addImm(0x0000009F)
        .add(predOps(ARMCC::AL))
        .add(condCodeOp());
    BuildMI(MBB, MBBI, DL, TII->get(ARM::t2BICri), SpareReg)
        .addReg(SpareReg)
        .addImm(0xF0000000)
        .add(predOps(ARMCC::AL))
        .add(condCodeOp());
    BuildMI(MBB, MBBI, DL, TII->get(ARM::VMSR))
```
- EN: Declares `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1587-1593
```cpp
        .addReg(SpareReg)
        .add(predOps(ARMCC::AL));
    // The ldr must happen after a floating point instruction. To prevent the
    // post-ra scheduler to mess with the order, we create a bundle.
    finalizeBundle(MBB, VLSTM->getIterator(), MBBI->getIterator());
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1595-1600
```cpp
void ARMExpandPseudo::CMSESaveClearFPRegsV81(MachineBasicBlock &MBB,
                                             MachineBasicBlock::iterator MBBI,
                                             DebugLoc &DL,
                                             const LivePhysRegs &LiveRegs) {
  BitVector ClearRegs(32, true);
  bool DefFP = determineFPRegsToClear(*MBBI, ClearRegs);
```
- EN: Implements `ARMExpandPseudo::CMSESaveClearFPRegsV81`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMExpandPseudo::CMSESaveClearFPRegsV81`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1602-1610
```cpp
  // If the instruction does not write to a FP register and no elements were
  // removed from the set, then no FP registers were used to pass
  // arguments/returns.
  if (!DefFP && ClearRegs.count() == ClearRegs.size()) {
    // save space on stack for VLSTM
    BuildMI(MBB, MBBI, DL, TII->get(ARM::tSUBspi), ARM::SP)
        .addReg(ARM::SP)
        .addImm(CMSE_FP_SAVE_SIZE >> 2)
        .add(predOps(ARMCC::AL));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1612-1626
```cpp
    // Lazy store all FP registers to the stack
    MachineInstrBuilder VLSTM =
        BuildMI(MBB, MBBI, DL, TII->get(ARM::VLSTM))
            .addReg(ARM::SP)
            .add(predOps(ARMCC::AL))
            .addImm(0); // Represents a pseoudo register list, has no effect on
                        // the encoding.
    // Mark non-live registers as undef
    for (MachineOperand &MO : VLSTM->implicit_operands()) {
      if (MO.isReg() && !MO.isDef()) {
        Register Reg = MO.getReg();
        MO.setIsUndef(!LiveRegs.contains(Reg));
      }
    }
  } else {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1627-1633
```cpp
    // Push all the callee-saved registers (s16-s31).
    MachineInstrBuilder VPUSH =
        BuildMI(MBB, MBBI, DL, TII->get(ARM::VSTMSDB_UPD), ARM::SP)
            .addReg(ARM::SP)
            .add(predOps(ARMCC::AL));
    for (unsigned Reg = ARM::S16; Reg <= ARM::S31; ++Reg)
      VPUSH.addReg(Reg);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1635-1636
```cpp
    // Clear FP registers with a VSCCLRM.
    (void)CMSEClearFPRegsV81(MBB, MBBI, ClearRegs);
```
- EN: Declares `CMSEClearFPRegsV81`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `CMSEClearFPRegsV81`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1638-1644
```cpp
    // Save floating-point context.
    BuildMI(MBB, MBBI, DL, TII->get(ARM::VSTR_FPCXTS_pre), ARM::SP)
        .addReg(ARM::SP)
        .addImm(-8)
        .add(predOps(ARMCC::AL));
  }
}
```
- EN: Declares `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1646-1654
```cpp
// Restore FP registers if present
void ARMExpandPseudo::CMSERestoreFPRegs(
    MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI, DebugLoc &DL,
    SmallVectorImpl<unsigned> &AvailableRegs) {
  if (STI->hasV8_1MMainlineOps())
    CMSERestoreFPRegsV81(MBB, MBBI, DL, AvailableRegs);
  else if (STI->hasV8MMainlineOps())
    CMSERestoreFPRegsV8(MBB, MBBI, DL, AvailableRegs);
}
```
- EN: Implements `ARMExpandPseudo::CMSERestoreFPRegs`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMExpandPseudo::CMSERestoreFPRegs`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1656-1658
```cpp
void ARMExpandPseudo::CMSERestoreFPRegsV8(
    MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI, DebugLoc &DL,
    SmallVectorImpl<unsigned> &AvailableRegs) {
```
- EN: Implements `ARMExpandPseudo::CMSERestoreFPRegsV8`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMExpandPseudo::CMSERestoreFPRegsV8`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1660-1663
```cpp
  // Keep a scratch register for the mitigation sequence.
  unsigned ScratchReg = ARM::NoRegister;
  if (STI->fixCMSE_CVE_2021_35465())
    ScratchReg = AvailableRegs.pop_back_val();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1665-1678
```cpp
  // Use AvailableRegs to store the fp regs
  std::vector<std::tuple<unsigned, unsigned, unsigned>> ClearedFPRegs;
  std::vector<unsigned> NonclearedFPRegs;
  for (const MachineOperand &Op : MBBI->operands()) {
    if (Op.isReg() && Op.isDef()) {
      Register Reg = Op.getReg();
      assert(!ARM::DPRRegClass.contains(Reg) ||
             ARM::DPR_VFP2RegClass.contains(Reg));
      assert(!ARM::QPRRegClass.contains(Reg));
      if (ARM::DPR_VFP2RegClass.contains(Reg)) {
        if (AvailableRegs.size() >= 2) {
          unsigned SaveReg2 = AvailableRegs.pop_back_val();
          unsigned SaveReg1 = AvailableRegs.pop_back_val();
          ClearedFPRegs.emplace_back(Reg, SaveReg1, SaveReg2);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1680-1692
```cpp
          // Save the fp register to the normal registers
          BuildMI(MBB, MBBI, DL, TII->get(ARM::VMOVRRD))
              .addReg(SaveReg1, RegState::Define)
              .addReg(SaveReg2, RegState::Define)
              .addReg(Reg)
              .add(predOps(ARMCC::AL));
        } else {
          NonclearedFPRegs.push_back(Reg);
        }
      } else if (ARM::SPRRegClass.contains(Reg)) {
        if (AvailableRegs.size() >= 1) {
          unsigned SaveReg = AvailableRegs.pop_back_val();
          ClearedFPRegs.emplace_back(Reg, SaveReg, 0);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1694-1703
```cpp
          // Save the fp register to the normal registers
          BuildMI(MBB, MBBI, DL, TII->get(ARM::VMOVRS), SaveReg)
              .addReg(Reg)
              .add(predOps(ARMCC::AL));
        } else {
          NonclearedFPRegs.push_back(Reg);
        }
      }
    }
  }
```
- EN: Implements `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1705-1705
```cpp
  bool returnsFPReg = (!NonclearedFPRegs.empty() || !ClearedFPRegs.empty());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1707-1708
```cpp
  if (returnsFPReg)
    assert(STI->hasFPRegs() && "Subtarget needs fpregs");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1710-1724
```cpp
  // Push FP regs that cannot be restored via normal registers on the stack
  for (unsigned Reg : NonclearedFPRegs) {
    if (ARM::DPR_VFP2RegClass.contains(Reg))
      BuildMI(MBB, MBBI, DL, TII->get(ARM::VSTRD))
          .addReg(Reg)
          .addReg(ARM::SP)
          .addImm((Reg - ARM::D0) * 2)
          .add(predOps(ARMCC::AL));
    else if (ARM::SPRRegClass.contains(Reg))
      BuildMI(MBB, MBBI, DL, TII->get(ARM::VSTRS))
          .addReg(Reg)
          .addReg(ARM::SP)
          .addImm(Reg - ARM::S0)
          .add(predOps(ARMCC::AL));
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1726-1733
```cpp
  // Lazy load fp regs from stack.
  // This executes as NOP in the absence of floating-point support.
  MachineInstrBuilder VLLDM =
      BuildMI(MBB, MBBI, DL, TII->get(ARM::VLLDM))
          .addReg(ARM::SP)
          .add(predOps(ARMCC::AL))
          .addImm(0); // Represents a pseoudo register list, has no effect on
                      // the encoding.
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1735-1749
```cpp
  if (STI->fixCMSE_CVE_2021_35465()) {
    auto Bundler = MIBundleBuilder(MBB, VLLDM);
    // Read the CONTROL register.
    Bundler.append(BuildMI(*MBB.getParent(), DL, TII->get(ARM::t2MRS_M))
                       .addReg(ScratchReg, RegState::Define)
                       .addImm(20)
                       .add(predOps(ARMCC::AL)));
    // Check bit 3 (SFPA).
    Bundler.append(BuildMI(*MBB.getParent(), DL, TII->get(ARM::t2TSTri))
                       .addReg(ScratchReg)
                       .addImm(8)
                       .add(predOps(ARMCC::AL)));
    // Emit the IT block.
    Bundler.append(BuildMI(*MBB.getParent(), DL, TII->get(ARM::t2IT))
                       .addImm(ARMCC::NE)
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1750-1764
```cpp
                       .addImm(8));
    // If SFPA is clear jump over to VLLDM, otherwise execute an instruction
    // which has no functional effect apart from causing context creation:
    // vmovne s0, s0. In the absence of FPU we emit .inst.w 0xeeb00a40,
    // which is defined as NOP if not executed.
    if (STI->hasFPRegs())
      Bundler.append(BuildMI(*MBB.getParent(), DL, TII->get(ARM::VMOVS))
                         .addReg(ARM::S0, RegState::Define)
                         .addReg(ARM::S0, RegState::Undef)
                         .add(predOps(ARMCC::NE)));
    else
      Bundler.append(BuildMI(*MBB.getParent(), DL, TII->get(ARM::INLINEASM))
                         .addExternalSymbol(".inst.w 0xeeb00a40")
                         .addImm(InlineAsm::Extra_HasSideEffects));
    finalizeBundle(MBB, Bundler.begin(), Bundler.end());
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1765-1765
```cpp
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1767-1780
```cpp
  // Restore all FP registers via normal registers
  for (const auto &Regs : ClearedFPRegs) {
    unsigned Reg, SaveReg1, SaveReg2;
    std::tie(Reg, SaveReg1, SaveReg2) = Regs;
    if (ARM::DPR_VFP2RegClass.contains(Reg))
      BuildMI(MBB, MBBI, DL, TII->get(ARM::VMOVDRR), Reg)
          .addReg(SaveReg1)
          .addReg(SaveReg2)
          .add(predOps(ARMCC::AL));
    else if (ARM::SPRRegClass.contains(Reg))
      BuildMI(MBB, MBBI, DL, TII->get(ARM::VMOVSR), Reg)
          .addReg(SaveReg1)
          .add(predOps(ARMCC::AL));
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1782-1787
```cpp
  // Pop the stack space
  BuildMI(MBB, MBBI, DL, TII->get(ARM::tADDspi), ARM::SP)
      .addReg(ARM::SP)
      .addImm(CMSE_FP_SAVE_SIZE >> 2)
      .add(predOps(ARMCC::AL));
}
```
- EN: Declares `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1789-1800
```cpp
static bool definesOrUsesFPReg(const MachineInstr &MI) {
  for (const MachineOperand &Op : MI.operands()) {
    if (!Op.isReg())
      continue;
    Register Reg = Op.getReg();
    if ((Reg >= ARM::Q0 && Reg <= ARM::Q7) ||
        (Reg >= ARM::D0 && Reg <= ARM::D15) ||
        (Reg >= ARM::S0 && Reg <= ARM::S31))
      return true;
  }
  return false;
}
```
- EN: Implements `definesOrUsesFPReg`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `definesOrUsesFPReg`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1802-1810
```cpp
void ARMExpandPseudo::CMSERestoreFPRegsV81(
    MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI, DebugLoc &DL,
    SmallVectorImpl<unsigned> &AvailableRegs) {
  if (!definesOrUsesFPReg(*MBBI)) {
    if (STI->fixCMSE_CVE_2021_35465()) {
      BuildMI(MBB, MBBI, DL, TII->get(ARM::VSCCLRMS))
          .add(predOps(ARMCC::AL))
          .addReg(ARM::VPR, RegState::Define);
    }
```
- EN: Implements `ARMExpandPseudo::CMSERestoreFPRegsV81`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMExpandPseudo::CMSERestoreFPRegsV81`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1812-1817
```cpp
    // Load FP registers from stack.
    BuildMI(MBB, MBBI, DL, TII->get(ARM::VLLDM))
        .addReg(ARM::SP)
        .add(predOps(ARMCC::AL))
        .addImm(0); // Represents a pseoudo register list, has no effect on the
                    // encoding.
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1819-1830
```cpp
    // Pop the stack space
    BuildMI(MBB, MBBI, DL, TII->get(ARM::tADDspi), ARM::SP)
        .addReg(ARM::SP)
        .addImm(CMSE_FP_SAVE_SIZE >> 2)
        .add(predOps(ARMCC::AL));
  } else {
    // Restore the floating point context.
    BuildMI(MBB, MBBI, MBBI->getDebugLoc(), TII->get(ARM::VLDR_FPCXTS_post),
            ARM::SP)
        .addReg(ARM::SP)
        .addImm(8)
        .add(predOps(ARMCC::AL));
```
- EN: Declares `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1832-1840
```cpp
    // Pop all the callee-saved registers (s16-s31).
    MachineInstrBuilder VPOP =
        BuildMI(MBB, MBBI, DL, TII->get(ARM::VLDMSIA_UPD), ARM::SP)
            .addReg(ARM::SP)
            .add(predOps(ARMCC::AL));
    for (unsigned Reg = ARM::S16; Reg <= ARM::S31; ++Reg)
      VPOP.addReg(Reg, RegState::Define);
  }
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1842-1856
```cpp
/// Expand a CMP_SWAP pseudo-inst to an ldrex/strex loop as simply as
/// possible. This only gets used at -O0 so we don't care about efficiency of
/// the generated code.
bool ARMExpandPseudo::ExpandCMP_SWAP(MachineBasicBlock &MBB,
                                     MachineBasicBlock::iterator MBBI,
                                     unsigned LdrexOp, unsigned StrexOp,
                                     unsigned UxtOp,
                                     MachineBasicBlock::iterator &NextMBBI) {
  bool IsThumb = STI->isThumb();
  bool IsThumb1Only = STI->isThumb1Only();
  MachineInstr &MI = *MBBI;
  DebugLoc DL = MI.getDebugLoc();
  const MachineOperand &Dest = MI.getOperand(0);
  Register TempReg = MI.getOperand(1).getReg();
  // Duplicating undef operands into 2 instructions does not guarantee the same
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1857-1861
```cpp
  // value on both; However undef should be replaced by xzr anyway.
  assert(!MI.getOperand(2).isUndef() && "cannot handle undef");
  Register AddrReg = MI.getOperand(2).getReg();
  Register DesiredReg = MI.getOperand(3).getReg();
  Register NewReg = MI.getOperand(4).getReg();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1863-1870
```cpp
  if (IsThumb) {
    assert(STI->hasV8MBaselineOps() &&
           "CMP_SWAP not expected to be custom expanded for Thumb1");
    assert((UxtOp == 0 || UxtOp == ARM::tUXTB || UxtOp == ARM::tUXTH) &&
           "ARMv8-M.baseline does not have t2UXTB/t2UXTH");
    assert((UxtOp == 0 || ARM::tGPRRegClass.contains(DesiredReg)) &&
           "DesiredReg used for UXT op must be tGPR");
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1872-1875
```cpp
  MachineFunction *MF = MBB.getParent();
  auto LoadCmpBB = MF->CreateMachineBasicBlock(MBB.getBasicBlock());
  auto StoreBB = MF->CreateMachineBasicBlock(MBB.getBasicBlock());
  auto DoneBB = MF->CreateMachineBasicBlock(MBB.getBasicBlock());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1877-1879
```cpp
  MF->insert(++MBB.getIterator(), LoadCmpBB);
  MF->insert(++LoadCmpBB->getIterator(), StoreBB);
  MF->insert(++StoreBB->getIterator(), DoneBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1881-1888
```cpp
  if (UxtOp) {
    MachineInstrBuilder MIB =
        BuildMI(MBB, MBBI, DL, TII->get(UxtOp), DesiredReg)
            .addReg(DesiredReg, RegState::Kill);
    if (!IsThumb)
      MIB.addImm(0);
    MIB.add(predOps(ARMCC::AL));
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1890-1893
```cpp
  // .Lloadcmp:
  //     ldrex rDest, [rAddr]
  //     cmp rDest, rDesired
  //     bne .Ldone
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 1895-1900
```cpp
  MachineInstrBuilder MIB;
  MIB = BuildMI(LoadCmpBB, DL, TII->get(LdrexOp), Dest.getReg());
  MIB.addReg(AddrReg);
  if (LdrexOp == ARM::t2LDREX)
    MIB.addImm(0); // a 32-bit Thumb ldrex (only) allows an offset.
  MIB.add(predOps(ARMCC::AL));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1902-1913
```cpp
  unsigned CMPrr = IsThumb ? ARM::tCMPhir : ARM::CMPrr;
  BuildMI(LoadCmpBB, DL, TII->get(CMPrr))
      .addReg(Dest.getReg(), getKillRegState(Dest.isDead()))
      .addReg(DesiredReg)
      .add(predOps(ARMCC::AL));
  unsigned Bcc = IsThumb ? ARM::tBcc : ARM::Bcc;
  BuildMI(LoadCmpBB, DL, TII->get(Bcc))
      .addMBB(DoneBB)
      .addImm(ARMCC::NE)
      .addReg(ARM::CPSR, RegState::Kill);
  LoadCmpBB->addSuccessor(DoneBB);
  LoadCmpBB->addSuccessor(StoreBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1915-1924
```cpp
  // .Lstore:
  //     strex rTempReg, rNew, [rAddr]
  //     cmp rTempReg, #0
  //     bne .Lloadcmp
  MIB = BuildMI(StoreBB, DL, TII->get(StrexOp), TempReg)
    .addReg(NewReg)
    .addReg(AddrReg);
  if (StrexOp == ARM::t2STREX)
    MIB.addImm(0); // a 32-bit Thumb strex (only) allows an offset.
  MIB.add(predOps(ARMCC::AL));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1926-1937
```cpp
  unsigned CMPri =
      IsThumb ? (IsThumb1Only ? ARM::tCMPi8 : ARM::t2CMPri) : ARM::CMPri;
  BuildMI(StoreBB, DL, TII->get(CMPri))
      .addReg(TempReg, RegState::Kill)
      .addImm(0)
      .add(predOps(ARMCC::AL));
  BuildMI(StoreBB, DL, TII->get(Bcc))
      .addMBB(LoadCmpBB)
      .addImm(ARMCC::NE)
      .addReg(ARM::CPSR, RegState::Kill);
  StoreBB->addSuccessor(LoadCmpBB);
  StoreBB->addSuccessor(DoneBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1939-1940
```cpp
  DoneBB->splice(DoneBB->end(), &MBB, MI, MBB.end());
  DoneBB->transferSuccessors(&MBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1942-1942
```cpp
  MBB.addSuccessor(LoadCmpBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1944-1945
```cpp
  NextMBBI = MBB.end();
  MI.eraseFromParent();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1947-1956
```cpp
  // Recompute livein lists.
  LivePhysRegs LiveRegs;
  computeAndAddLiveIns(LiveRegs, *DoneBB);
  computeAndAddLiveIns(LiveRegs, *StoreBB);
  computeAndAddLiveIns(LiveRegs, *LoadCmpBB);
  // Do an extra pass around the loop to get loop carried registers right.
  StoreBB->clearLiveIns();
  computeAndAddLiveIns(LiveRegs, *StoreBB);
  LoadCmpBB->clearLiveIns();
  computeAndAddLiveIns(LiveRegs, *LoadCmpBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1958-1959
```cpp
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1961-1974
```cpp
/// ARM's ldrexd/strexd take a consecutive register pair (represented as a
/// single GPRPair register), Thumb's take two separate registers so we need to
/// extract the subregs from the pair.
static void addExclusiveRegPair(MachineInstrBuilder &MIB, MachineOperand &Reg,
                                RegState Flags, bool IsThumb,
                                const TargetRegisterInfo *TRI) {
  if (IsThumb) {
    Register RegLo = TRI->getSubReg(Reg.getReg(), ARM::gsub_0);
    Register RegHi = TRI->getSubReg(Reg.getReg(), ARM::gsub_1);
    MIB.addReg(RegLo, Flags);
    MIB.addReg(RegHi, Flags);
  } else
    MIB.addReg(Reg.getReg(), Flags);
}
```
- EN: Implements `addExclusiveRegPair`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `addExclusiveRegPair`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1976-1990
```cpp
/// Expand a 64-bit CMP_SWAP to an ldrexd/strexd loop.
bool ARMExpandPseudo::ExpandCMP_SWAP_64(MachineBasicBlock &MBB,
                                        MachineBasicBlock::iterator MBBI,
                                        MachineBasicBlock::iterator &NextMBBI) {
  bool IsThumb = STI->isThumb();
  assert(!STI->isThumb1Only() && "CMP_SWAP_64 unsupported under Thumb1!");
  MachineInstr &MI = *MBBI;
  DebugLoc DL = MI.getDebugLoc();
  MachineOperand &Dest = MI.getOperand(0);
  // Duplicating undef operands into 2 instructions does not guarantee the same
  // value on both; However undef should be replaced by xzr anyway.
  assert(!MI.getOperand(1).isUndef() && "cannot handle undef");
  Register AddrAndTempReg = MI.getOperand(1).getReg();
  Register AddrReg = TRI->getSubReg(AddrAndTempReg, ARM::gsub_0);
  Register TempReg = TRI->getSubReg(AddrAndTempReg, ARM::gsub_1);
```
- EN: Implements `ARMExpandPseudo::ExpandCMP_SWAP_64`, a mutation/build routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMExpandPseudo::ExpandCMP_SWAP_64`，它是一个围绕机器基本块展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1991-1995
```cpp
  assert(MI.getOperand(1).getReg() == MI.getOperand(2).getReg() &&
         "tied operands have different registers");
  Register DesiredReg = MI.getOperand(3).getReg();
  MachineOperand New = MI.getOperand(4);
  New.setIsKill(false);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1997-2000
```cpp
  Register DestLo = TRI->getSubReg(Dest.getReg(), ARM::gsub_0);
  Register DestHi = TRI->getSubReg(Dest.getReg(), ARM::gsub_1);
  Register DesiredLo = TRI->getSubReg(DesiredReg, ARM::gsub_0);
  Register DesiredHi = TRI->getSubReg(DesiredReg, ARM::gsub_1);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2002-2005
```cpp
  MachineFunction *MF = MBB.getParent();
  auto LoadCmpBB = MF->CreateMachineBasicBlock(MBB.getBasicBlock());
  auto StoreBB = MF->CreateMachineBasicBlock(MBB.getBasicBlock());
  auto DoneBB = MF->CreateMachineBasicBlock(MBB.getBasicBlock());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2007-2009
```cpp
  MF->insert(++MBB.getIterator(), LoadCmpBB);
  MF->insert(++LoadCmpBB->getIterator(), StoreBB);
  MF->insert(++StoreBB->getIterator(), DoneBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2011-2020
```cpp
  // .Lloadcmp:
  //     ldrexd rDestLo, rDestHi, [rAddr]
  //     cmp rDestLo, rDesiredLo
  //     sbcs dead rTempReg, rDestHi, rDesiredHi
  //     bne .Ldone
  unsigned LDREXD = IsThumb ? ARM::t2LDREXD : ARM::LDREXD;
  MachineInstrBuilder MIB;
  MIB = BuildMI(LoadCmpBB, DL, TII->get(LDREXD));
  addExclusiveRegPair(MIB, Dest, RegState::Define, IsThumb, TRI);
  MIB.addReg(AddrReg).add(predOps(ARMCC::AL));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2022-2026
```cpp
  unsigned CMPrr = IsThumb ? ARM::tCMPhir : ARM::CMPrr;
  BuildMI(LoadCmpBB, DL, TII->get(CMPrr))
      .addReg(DestLo, getKillRegState(Dest.isDead()))
      .addReg(DesiredLo)
      .add(predOps(ARMCC::AL));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2028-2031
```cpp
  BuildMI(LoadCmpBB, DL, TII->get(CMPrr))
      .addReg(DestHi, getKillRegState(Dest.isDead()))
      .addReg(DesiredHi)
      .addImm(ARMCC::EQ).addReg(ARM::CPSR, RegState::Kill);
```
- EN: Declares `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2033-2039
```cpp
  unsigned Bcc = IsThumb ? ARM::tBcc : ARM::Bcc;
  BuildMI(LoadCmpBB, DL, TII->get(Bcc))
      .addMBB(DoneBB)
      .addImm(ARMCC::NE)
      .addReg(ARM::CPSR, RegState::Kill);
  LoadCmpBB->addSuccessor(DoneBB);
  LoadCmpBB->addSuccessor(StoreBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2041-2049
```cpp
  // .Lstore:
  //     strexd rTempReg, rNewLo, rNewHi, [rAddr]
  //     cmp rTempReg, #0
  //     bne .Lloadcmp
  unsigned STREXD = IsThumb ? ARM::t2STREXD : ARM::STREXD;
  MIB = BuildMI(StoreBB, DL, TII->get(STREXD), TempReg);
  RegState Flags = getKillRegState(New.isDead());
  addExclusiveRegPair(MIB, New, Flags, IsThumb, TRI);
  MIB.addReg(AddrReg).add(predOps(ARMCC::AL));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2051-2061
```cpp
  unsigned CMPri = IsThumb ? ARM::t2CMPri : ARM::CMPri;
  BuildMI(StoreBB, DL, TII->get(CMPri))
      .addReg(TempReg, RegState::Kill)
      .addImm(0)
      .add(predOps(ARMCC::AL));
  BuildMI(StoreBB, DL, TII->get(Bcc))
      .addMBB(LoadCmpBB)
      .addImm(ARMCC::NE)
      .addReg(ARM::CPSR, RegState::Kill);
  StoreBB->addSuccessor(LoadCmpBB);
  StoreBB->addSuccessor(DoneBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2063-2064
```cpp
  DoneBB->splice(DoneBB->end(), &MBB, MI, MBB.end());
  DoneBB->transferSuccessors(&MBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2066-2066
```cpp
  MBB.addSuccessor(LoadCmpBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2068-2069
```cpp
  NextMBBI = MBB.end();
  MI.eraseFromParent();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2071-2080
```cpp
  // Recompute livein lists.
  LivePhysRegs LiveRegs;
  computeAndAddLiveIns(LiveRegs, *DoneBB);
  computeAndAddLiveIns(LiveRegs, *StoreBB);
  computeAndAddLiveIns(LiveRegs, *LoadCmpBB);
  // Do an extra pass around the loop to get loop carried registers right.
  StoreBB->clearLiveIns();
  computeAndAddLiveIns(LiveRegs, *StoreBB);
  LoadCmpBB->clearLiveIns();
  computeAndAddLiveIns(LiveRegs, *LoadCmpBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2082-2083
```cpp
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2085-2097
```cpp
static void CMSEPushCalleeSaves(const TargetInstrInfo &TII,
                                MachineBasicBlock &MBB,
                                MachineBasicBlock::iterator MBBI,
                                Register JumpReg, const LivePhysRegs &LiveRegs,
                                bool Thumb1Only) {
  const DebugLoc &DL = MBBI->getDebugLoc();
  if (Thumb1Only) { // push Lo and Hi regs separately
    MachineInstrBuilder PushMIB =
        BuildMI(MBB, MBBI, DL, TII.get(ARM::tPUSH)).add(predOps(ARMCC::AL));
    for (unsigned Reg = ARM::R4; Reg < ARM::R8; ++Reg) {
      PushMIB.addReg(
          Reg, getUndefRegState(Reg != JumpReg && !LiveRegs.contains(Reg)));
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2099-2113
```cpp
    // Thumb1 can only tPUSH low regs, so we copy the high regs to the low
    // regs that we just saved and push the low regs again, taking care to
    // not clobber JumpReg. If JumpReg is one of the low registers, push first
    // the values of r9-r11, and then r8. That would leave them ordered in
    // memory, and allow us to later pop them with a single instructions.
    // FIXME: Could also use any of r0-r3 that are free (including in the
    // first PUSH above).
    for (unsigned LoReg = ARM::R7, HiReg = ARM::R11; LoReg >= ARM::R4;
         --LoReg) {
      if (JumpReg == LoReg)
        continue;
      BuildMI(MBB, MBBI, DL, TII.get(ARM::tMOVr), LoReg)
          .addReg(HiReg, getUndefRegState(!LiveRegs.contains(HiReg)))
          .add(predOps(ARMCC::AL));
      --HiReg;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2114-2121
```cpp
    }
    MachineInstrBuilder PushMIB2 =
        BuildMI(MBB, MBBI, DL, TII.get(ARM::tPUSH)).add(predOps(ARMCC::AL));
    for (unsigned Reg = ARM::R4; Reg < ARM::R8; ++Reg) {
      if (Reg == JumpReg)
        continue;
      PushMIB2.addReg(Reg, RegState::Kill);
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2123-2137
```cpp
    // If we couldn't use a low register for temporary storage (because it was
    // the JumpReg), use r4 or r5, whichever is not JumpReg. It has already been
    // saved.
    if (JumpReg >= ARM::R4 && JumpReg <= ARM::R7) {
      Register LoReg = JumpReg == ARM::R4 ? ARM::R5 : ARM::R4;
      BuildMI(MBB, MBBI, DL, TII.get(ARM::tMOVr), LoReg)
          .addReg(ARM::R8, getUndefRegState(!LiveRegs.contains(ARM::R8)))
          .add(predOps(ARMCC::AL));
      BuildMI(MBB, MBBI, DL, TII.get(ARM::tPUSH))
          .add(predOps(ARMCC::AL))
          .addReg(LoReg, RegState::Kill);
    }
  } else { // push Lo and Hi registers with a single instruction
    MachineInstrBuilder PushMIB =
        BuildMI(MBB, MBBI, DL, TII.get(ARM::t2STMDB_UPD), ARM::SP)
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2138-2145
```cpp
            .addReg(ARM::SP)
            .add(predOps(ARMCC::AL));
    for (unsigned Reg = ARM::R4; Reg < ARM::R12; ++Reg) {
      PushMIB.addReg(
          Reg, getUndefRegState(Reg != JumpReg && !LiveRegs.contains(Reg)));
    }
  }
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2147-2161
```cpp
static void CMSEPopCalleeSaves(const TargetInstrInfo &TII,
                               MachineBasicBlock &MBB,
                               MachineBasicBlock::iterator MBBI,
                               bool Thumb1Only) {
  const DebugLoc &DL = MBBI->getDebugLoc();
  if (Thumb1Only) {
    MachineInstrBuilder PopMIB =
        BuildMI(MBB, MBBI, DL, TII.get(ARM::tPOP)).add(predOps(ARMCC::AL));
    for (int R = 0; R < 4; ++R) {
      PopMIB.addReg(ARM::R4 + R, RegState::Define);
      BuildMI(MBB, MBBI, DL, TII.get(ARM::tMOVr), ARM::R8 + R)
          .addReg(ARM::R4 + R, RegState::Kill)
          .add(predOps(ARMCC::AL));
    }
    MachineInstrBuilder PopMIB2 =
```
- EN: Implements `CMSEPopCalleeSaves`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `CMSEPopCalleeSaves`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2162-2173
```cpp
        BuildMI(MBB, MBBI, DL, TII.get(ARM::tPOP)).add(predOps(ARMCC::AL));
    for (int R = 0; R < 4; ++R)
      PopMIB2.addReg(ARM::R4 + R, RegState::Define);
  } else { // pop Lo and Hi registers with a single instruction
    MachineInstrBuilder PopMIB =
        BuildMI(MBB, MBBI, DL, TII.get(ARM::t2LDMIA_UPD), ARM::SP)
            .addReg(ARM::SP)
            .add(predOps(ARMCC::AL));
    for (unsigned Reg = ARM::R4; Reg < ARM::R12; ++Reg)
      PopMIB.addReg(Reg, RegState::Define);
  }
}
```
- EN: Implements `BuildMI`, a mutation/build routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `BuildMI`，它是一个围绕机器指令展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2175-2182
```cpp
bool ARMExpandPseudo::ExpandMI(MachineBasicBlock &MBB,
                               MachineBasicBlock::iterator MBBI,
                               MachineBasicBlock::iterator &NextMBBI) {
  MachineInstr &MI = *MBBI;
  unsigned Opcode = MI.getOpcode();
  switch (Opcode) {
    default:
      return false;
```
- EN: Implements `ARMExpandPseudo::ExpandMI`, a mutation/build routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMExpandPseudo::ExpandMI`，它是一个围绕机器基本块展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2184-2198
```cpp
    case ARM::VBSPd:
    case ARM::VBSPq: {
      Register DstReg = MI.getOperand(0).getReg();
      if (DstReg == MI.getOperand(3).getReg()) {
        // Expand to VBIT
        unsigned NewOpc = Opcode == ARM::VBSPd ? ARM::VBITd : ARM::VBITq;
        BuildMI(MBB, MBBI, MI.getDebugLoc(), TII->get(NewOpc))
            .add(MI.getOperand(0))
            .add(MI.getOperand(3))
            .add(MI.getOperand(2))
            .add(MI.getOperand(1))
            .addImm(MI.getOperand(4).getImm())
            .add(MI.getOperand(5));
      } else if (DstReg == MI.getOperand(2).getReg()) {
        // Expand to VBIF
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2199-2213
```cpp
        unsigned NewOpc = Opcode == ARM::VBSPd ? ARM::VBIFd : ARM::VBIFq;
        BuildMI(MBB, MBBI, MI.getDebugLoc(), TII->get(NewOpc))
            .add(MI.getOperand(0))
            .add(MI.getOperand(2))
            .add(MI.getOperand(3))
            .add(MI.getOperand(1))
            .addImm(MI.getOperand(4).getImm())
            .add(MI.getOperand(5));
      } else {
        // Expand to VBSL
        unsigned NewOpc = Opcode == ARM::VBSPd ? ARM::VBSLd : ARM::VBSLq;
        if (DstReg == MI.getOperand(1).getReg()) {
          BuildMI(MBB, MBBI, MI.getDebugLoc(), TII->get(NewOpc))
              .add(MI.getOperand(0))
              .add(MI.getOperand(1))
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2214-2228
```cpp
              .add(MI.getOperand(2))
              .add(MI.getOperand(3))
              .addImm(MI.getOperand(4).getImm())
              .add(MI.getOperand(5));
        } else {
          // Use move to satisfy constraints
          unsigned MoveOpc = Opcode == ARM::VBSPd ? ARM::VORRd : ARM::VORRq;
          RegState MO1Flags = getRegState(MI.getOperand(1)) & ~RegState::Kill;
          BuildMI(MBB, MBBI, MI.getDebugLoc(), TII->get(MoveOpc))
              .addReg(DstReg,
                      RegState::Define |
                          getRenamableRegState(MI.getOperand(0).isRenamable()))
              .addReg(MI.getOperand(1).getReg(), MO1Flags)
              .addReg(MI.getOperand(1).getReg(), MO1Flags)
              .addImm(MI.getOperand(4).getImm())
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2229-2243
```cpp
              .add(MI.getOperand(5));
          BuildMI(MBB, MBBI, MI.getDebugLoc(), TII->get(NewOpc))
              .add(MI.getOperand(0))
              .addReg(DstReg,
                      RegState::Kill |
                          getRenamableRegState(MI.getOperand(0).isRenamable()))
              .add(MI.getOperand(2))
              .add(MI.getOperand(3))
              .addImm(MI.getOperand(4).getImm())
              .add(MI.getOperand(5));
        }
      }
      MI.eraseFromParent();
      return true;
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2245-2258
```cpp
    case ARM::TCRETURNdi:
    case ARM::TCRETURNri:
    case ARM::TCRETURNrinotr12: {
      MachineBasicBlock::iterator MBBI = MBB.getLastNonDebugInstr();
      if (MBBI->getOpcode() == ARM::SEH_EpilogEnd)
        MBBI--;
      if (MBBI->getOpcode() == ARM::SEH_Nop_Ret)
        MBBI--;
      assert(MBBI->isReturn() &&
             "Can only insert epilog into returning blocks");
      unsigned RetOpcode = MBBI->getOpcode();
      DebugLoc dl = MBBI->getDebugLoc();
      const ARMBaseInstrInfo &TII = *static_cast<const ARMBaseInstrInfo *>(
          MBB.getParent()->getSubtarget().getInstrInfo());
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2260-2266
```cpp
      // Tail call return: adjust the stack pointer and jump to callee.
      MBBI = MBB.getLastNonDebugInstr();
      if (MBBI->getOpcode() == ARM::SEH_EpilogEnd)
        MBBI--;
      if (MBBI->getOpcode() == ARM::SEH_Nop_Ret)
        MBBI--;
      MachineOperand &JumpTarget = MBBI->getOperand(0);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2268-2282
```cpp
      // Jump to label or value in register.
      if (RetOpcode == ARM::TCRETURNdi) {
        MachineFunction *MF = MBB.getParent();
        bool NeedsWinCFI = MF->getTarget().getMCAsmInfo().usesWindowsCFI() &&
                           MF->getFunction().needsUnwindTableEntry();
        unsigned TCOpcode =
            STI->isThumb()
                ? ((STI->isTargetMachO() || NeedsWinCFI) ? ARM::tTAILJMPd
                                                         : ARM::tTAILJMPdND)
                : ARM::TAILJMPd;
        MachineInstrBuilder MIB = BuildMI(MBB, MBBI, dl, TII.get(TCOpcode));
        if (JumpTarget.isGlobal())
          MIB.addGlobalAddress(JumpTarget.getGlobal(), JumpTarget.getOffset(),
                               JumpTarget.getTargetFlags());
        else {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2283-2286
```cpp
          assert(JumpTarget.isSymbol());
          MIB.addExternalSymbol(JumpTarget.getSymbolName(),
                                JumpTarget.getTargetFlags());
        }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2288-2299
```cpp
        // Add the default predicate in Thumb mode.
        if (STI->isThumb())
          MIB.add(predOps(ARMCC::AL));
      } else if (RetOpcode == ARM::TCRETURNri ||
                 RetOpcode == ARM::TCRETURNrinotr12) {
        unsigned Opcode =
          STI->isThumb() ? ARM::tTAILJMPr
                         : (STI->hasV4TOps() ? ARM::TAILJMPr : ARM::TAILJMPr4);
        BuildMI(MBB, MBBI, dl,
                TII.get(Opcode))
            .addReg(JumpTarget.getReg(), RegState::Kill);
      }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2301-2303
```cpp
      auto NewMI = std::prev(MBBI);
      for (unsigned i = 2, e = MBBI->getNumOperands(); i != e; ++i)
        NewMI->addOperand(MBBI->getOperand(i));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2305-2305
```cpp
      NewMI->setCFIType(*MBB.getParent(), MI.getCFIType());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2307-2313
```cpp
      // Update call info and delete the pseudo instruction TCRETURN.
      if (MI.isCandidateForAdditionalCallInfo())
        MI.getMF()->moveAdditionalCallInfo(&MI, &*NewMI);
      // Copy nomerge flag over to new instruction.
      if (MI.getFlag(MachineInstr::NoMerge))
        NewMI->setFlag(MachineInstr::NoMerge);
      MBB.erase(MBBI);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2315-2322
```cpp
      MBBI = NewMI;
      return true;
    }
    case ARM::tBXNS_RET: {
      // For v8.0-M.Main we need to authenticate LR before clearing FPRs, which
      // uses R12 as a scratch register.
      if (!STI->hasV8_1MMainlineOps() && AFI->shouldSignReturnAddress())
        BuildMI(MBB, MBBI, DebugLoc(), TII->get(ARM::t2AUT));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2324-2324
```cpp
      MachineBasicBlock &AfterBB = CMSEClearFPRegs(MBB, MBBI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2326-2332
```cpp
      if (STI->hasV8_1MMainlineOps()) {
        // Restore the non-secure floating point context.
        BuildMI(MBB, MBBI, MBBI->getDebugLoc(),
                TII->get(ARM::VLDR_FPCXTNS_post), ARM::SP)
            .addReg(ARM::SP)
            .addImm(4)
            .add(predOps(ARMCC::AL));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2334-2336
```cpp
        if (AFI->shouldSignReturnAddress())
          BuildMI(AfterBB, AfterBB.end(), DebugLoc(), TII->get(ARM::t2AUT));
      }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2338-2346
```cpp
      // Clear all GPR that are not a use of the return instruction.
      assert(llvm::all_of(MBBI->operands(), [](const MachineOperand &Op) {
        return !Op.isReg() || Op.getReg() != ARM::R12;
      }));
      SmallVector<unsigned, 5> ClearRegs;
      determineGPRegsToClear(
          *MBBI, {ARM::R0, ARM::R1, ARM::R2, ARM::R3, ARM::R12}, ClearRegs);
      CMSEClearGPRegs(AfterBB, AfterBB.end(), MBBI->getDebugLoc(), ClearRegs,
                      ARM::LR);
```
- EN: Implements `llvm::all_of`, a target-specific routine centered on machine operands. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `llvm::all_of`，它是一个围绕机器操作数展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2348-2360
```cpp
      MachineInstrBuilder NewMI =
          BuildMI(AfterBB, AfterBB.end(), MBBI->getDebugLoc(),
                  TII->get(ARM::tBXNS))
              .addReg(ARM::LR)
              .add(predOps(ARMCC::AL));
      for (const MachineOperand &Op : MI.operands())
        NewMI->addOperand(Op);
      MI.eraseFromParent();
      return true;
    }
    case ARM::tBLXNS_CALL: {
      DebugLoc DL = MBBI->getDebugLoc();
      Register JumpReg = MBBI->getOperand(0).getReg();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2362-2370
```cpp
      // Figure out which registers are live at the point immediately before the
      // call. When we indiscriminately push a set of registers, the live
      // registers are added as ordinary use operands, whereas dead registers
      // are "undef".
      LivePhysRegs LiveRegs(*TRI);
      LiveRegs.addLiveOuts(MBB);
      for (const MachineInstr &MI : make_range(MBB.rbegin(), MBBI.getReverse()))
        LiveRegs.stepBackward(MI);
      LiveRegs.stepBackward(*MBBI);
```
- EN: Declares `LiveRegs`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LiveRegs`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2372-2373
```cpp
      CMSEPushCalleeSaves(*TII, MBB, MBBI, JumpReg, LiveRegs,
                          AFI->isThumb1OnlyFunction());
```
- EN: Declares `CMSEPushCalleeSaves`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `CMSEPushCalleeSaves`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2375-2381
```cpp
      SmallVector<unsigned, 16> ClearRegs;
      determineGPRegsToClear(*MBBI,
                             {ARM::R0, ARM::R1, ARM::R2, ARM::R3, ARM::R4,
                              ARM::R5, ARM::R6, ARM::R7, ARM::R8, ARM::R9,
                              ARM::R10, ARM::R11, ARM::R12},
                             ClearRegs);
      auto OriginalClearRegs = ClearRegs;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2383-2385
```cpp
      // Get the first cleared register as a scratch (to use later with tBIC).
      // We need to use the first so we can ensure it is a low register.
      unsigned ScratchReg = ClearRegs.front();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2387-2401
```cpp
      // Clear LSB of JumpReg
      if (AFI->isThumb2Function()) {
        BuildMI(MBB, MBBI, DL, TII->get(ARM::t2BICri), JumpReg)
            .addReg(JumpReg)
            .addImm(1)
            .add(predOps(ARMCC::AL))
            .add(condCodeOp());
      } else {
        // We need to use an extra register to cope with 8M Baseline,
        // since we have saved all of the registers we are ok to trash a non
        // argument register here.
        BuildMI(MBB, MBBI, DL, TII->get(ARM::tMOVi8), ScratchReg)
            .add(condCodeOp())
            .addImm(1)
            .add(predOps(ARMCC::AL));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2402-2407
```cpp
        BuildMI(MBB, MBBI, DL, TII->get(ARM::tBIC), JumpReg)
            .addReg(ARM::CPSR, RegState::Define)
            .addReg(JumpReg)
            .addReg(ScratchReg)
            .add(predOps(ARMCC::AL));
      }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2409-2411
```cpp
      CMSESaveClearFPRegs(MBB, MBBI, DL, LiveRegs,
                          ClearRegs); // save+clear FP regs with ClearRegs
      CMSEClearGPRegs(MBB, MBBI, DL, ClearRegs, JumpReg);
```
- EN: Declares `CMSESaveClearFPRegs`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `CMSESaveClearFPRegs`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2413-2416
```cpp
      const MachineInstrBuilder NewCall =
          BuildMI(MBB, MBBI, DL, TII->get(ARM::tBLXNSr))
              .add(predOps(ARMCC::AL))
              .addReg(JumpReg, RegState::Kill);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2418-2421
```cpp
      for (const MachineOperand &MO : llvm::drop_begin(MI.operands()))
        NewCall->addOperand(MO);
      if (MI.isCandidateForAdditionalCallInfo())
        MI.getMF()->moveAdditionalCallInfo(&MI, NewCall.getInstr());
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2423-2423
```cpp
      CMSERestoreFPRegs(MBB, MBBI, DL, OriginalClearRegs); // restore FP registers
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2425-2425
```cpp
      CMSEPopCalleeSaves(*TII, MBB, MBBI, AFI->isThumb1OnlyFunction());
```
- EN: Declares `CMSEPopCalleeSaves`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `CMSEPopCalleeSaves`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2427-2439
```cpp
      MI.eraseFromParent();
      return true;
    }
    case ARM::VMOVHcc:
    case ARM::VMOVScc:
    case ARM::VMOVDcc: {
      unsigned newOpc = Opcode != ARM::VMOVDcc ? ARM::VMOVS : ARM::VMOVD;
      BuildMI(MBB, MBBI, MI.getDebugLoc(), TII->get(newOpc),
              MI.getOperand(1).getReg())
          .add(MI.getOperand(2))
          .addImm(MI.getOperand(3).getImm()) // 'pred'
          .add(MI.getOperand(4))
          .add(makeImplicit(MI.getOperand(1)));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2441-2453
```cpp
      MI.eraseFromParent();
      return true;
    }
    case ARM::t2MOVCCr:
    case ARM::MOVCCr: {
      unsigned Opc = AFI->isThumbFunction() ? ARM::t2MOVr : ARM::MOVr;
      BuildMI(MBB, MBBI, MI.getDebugLoc(), TII->get(Opc),
              MI.getOperand(1).getReg())
          .add(MI.getOperand(2))
          .addImm(MI.getOperand(3).getImm()) // 'pred'
          .add(MI.getOperand(4))
          .add(condCodeOp()) // 's' bit
          .add(makeImplicit(MI.getOperand(1)));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2455-2466
```cpp
      MI.eraseFromParent();
      return true;
    }
    case ARM::MOVCCsi: {
      BuildMI(MBB, MBBI, MI.getDebugLoc(), TII->get(ARM::MOVsi),
              (MI.getOperand(1).getReg()))
          .add(MI.getOperand(2))
          .addImm(MI.getOperand(3).getImm())
          .addImm(MI.getOperand(4).getImm()) // 'pred'
          .add(MI.getOperand(5))
          .add(condCodeOp()) // 's' bit
          .add(makeImplicit(MI.getOperand(1)));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2468-2480
```cpp
      MI.eraseFromParent();
      return true;
    }
    case ARM::MOVCCsr: {
      BuildMI(MBB, MBBI, MI.getDebugLoc(), TII->get(ARM::MOVsr),
              (MI.getOperand(1).getReg()))
          .add(MI.getOperand(2))
          .add(MI.getOperand(3))
          .addImm(MI.getOperand(4).getImm())
          .addImm(MI.getOperand(5).getImm()) // 'pred'
          .add(MI.getOperand(6))
          .add(condCodeOp()) // 's' bit
          .add(makeImplicit(MI.getOperand(1)));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2482-2496
```cpp
      MI.eraseFromParent();
      return true;
    }
    case ARM::t2MOVCCi16:
    case ARM::MOVCCi16: {
      unsigned NewOpc = AFI->isThumbFunction() ? ARM::t2MOVi16 : ARM::MOVi16;
      BuildMI(MBB, MBBI, MI.getDebugLoc(), TII->get(NewOpc),
              MI.getOperand(1).getReg())
          .addImm(MI.getOperand(2).getImm())
          .addImm(MI.getOperand(3).getImm()) // 'pred'
          .add(MI.getOperand(4))
          .add(makeImplicit(MI.getOperand(1)));
      MI.eraseFromParent();
      return true;
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2497-2506
```cpp
    case ARM::t2MOVCCi:
    case ARM::MOVCCi: {
      unsigned Opc = AFI->isThumbFunction() ? ARM::t2MOVi : ARM::MOVi;
      BuildMI(MBB, MBBI, MI.getDebugLoc(), TII->get(Opc),
              MI.getOperand(1).getReg())
          .addImm(MI.getOperand(2).getImm())
          .addImm(MI.getOperand(3).getImm()) // 'pred'
          .add(MI.getOperand(4))
          .add(condCodeOp()) // 's' bit
          .add(makeImplicit(MI.getOperand(1)));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2508-2520
```cpp
      MI.eraseFromParent();
      return true;
    }
    case ARM::t2MVNCCi:
    case ARM::MVNCCi: {
      unsigned Opc = AFI->isThumbFunction() ? ARM::t2MVNi : ARM::MVNi;
      BuildMI(MBB, MBBI, MI.getDebugLoc(), TII->get(Opc),
              MI.getOperand(1).getReg())
          .addImm(MI.getOperand(2).getImm())
          .addImm(MI.getOperand(3).getImm()) // 'pred'
          .add(MI.getOperand(4))
          .add(condCodeOp()) // 's' bit
          .add(makeImplicit(MI.getOperand(1)));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2522-2536
```cpp
      MI.eraseFromParent();
      return true;
    }
    case ARM::t2MOVCClsl:
    case ARM::t2MOVCClsr:
    case ARM::t2MOVCCasr:
    case ARM::t2MOVCCror: {
      unsigned NewOpc;
      switch (Opcode) {
      case ARM::t2MOVCClsl: NewOpc = ARM::t2LSLri; break;
      case ARM::t2MOVCClsr: NewOpc = ARM::t2LSRri; break;
      case ARM::t2MOVCCasr: NewOpc = ARM::t2ASRri; break;
      case ARM::t2MOVCCror: NewOpc = ARM::t2RORri; break;
      default: llvm_unreachable("unexpected conditional move");
      }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2537-2551
```cpp
      BuildMI(MBB, MBBI, MI.getDebugLoc(), TII->get(NewOpc),
              MI.getOperand(1).getReg())
          .add(MI.getOperand(2))
          .addImm(MI.getOperand(3).getImm())
          .addImm(MI.getOperand(4).getImm()) // 'pred'
          .add(MI.getOperand(5))
          .add(condCodeOp()) // 's' bit
          .add(makeImplicit(MI.getOperand(1)));
      MI.eraseFromParent();
      return true;
    }
    case ARM::Int_eh_sjlj_dispatchsetup: {
      MachineFunction &MF = *MI.getParent()->getParent();
      const ARMBaseRegisterInfo &RI = TII->getRegisterInfo();
      // For functions using a base pointer, we rematerialize it (via the frame
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2552-2558
```cpp
      // pointer) here since eh.sjlj.setjmp and eh.sjlj.longjmp don't do it
      // for us. Otherwise, expand to nothing.
      if (RI.hasBasePointer(MF)) {
        int32_t NumBytes = AFI->getFramePtrSpillOffset();
        Register FramePtr = RI.getFrameRegister(MF);
        assert(MF.getSubtarget().getFrameLowering()->hasFP(MF) &&
               "base pointer without frame pointer?");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2560-2574
```cpp
        if (AFI->isThumb2Function()) {
          emitT2RegPlusImmediate(MBB, MBBI, MI.getDebugLoc(), ARM::R6,
                                 FramePtr, -NumBytes, ARMCC::AL, 0, *TII);
        } else if (AFI->isThumbFunction()) {
          emitThumbRegPlusImmediate(MBB, MBBI, MI.getDebugLoc(), ARM::R6,
                                    FramePtr, -NumBytes, *TII, RI);
        } else {
          emitARMRegPlusImmediate(MBB, MBBI, MI.getDebugLoc(), ARM::R6,
                                  FramePtr, -NumBytes, ARMCC::AL, 0,
                                  *TII);
        }
        // If there's dynamic realignment, adjust for it.
        if (RI.hasStackRealignment(MF)) {
          MachineFrameInfo &MFI = MF.getFrameInfo();
          Align MaxAlign = MFI.getMaxAlign();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2575-2589
```cpp
          assert (!AFI->isThumb1OnlyFunction());
          // Emit bic r6, r6, MaxAlign
          assert(MaxAlign <= Align(256) &&
                 "The BIC instruction cannot encode "
                 "immediates larger than 256 with all lower "
                 "bits set.");
          unsigned bicOpc = AFI->isThumbFunction() ?
            ARM::t2BICri : ARM::BICri;
          BuildMI(MBB, MBBI, MI.getDebugLoc(), TII->get(bicOpc), ARM::R6)
              .addReg(ARM::R6, RegState::Kill)
              .addImm(MaxAlign.value() - 1)
              .add(predOps(ARMCC::AL))
              .add(condCodeOp());
        }
      }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2590-2592
```cpp
      MI.eraseFromParent();
      return true;
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2594-2608
```cpp
    case ARM::LSRs1:
    case ARM::ASRs1: {
      // These are just fancy MOVs instructions.
      BuildMI(MBB, MBBI, MI.getDebugLoc(), TII->get(ARM::MOVsi),
              MI.getOperand(0).getReg())
          .add(MI.getOperand(1))
          .addImm(ARM_AM::getSORegOpc(
              (Opcode == ARM::LSRs1 ? ARM_AM::lsr : ARM_AM::asr), 1))
          .add(predOps(ARMCC::AL))
          .addReg(ARM::CPSR, RegState::Define);
      MI.eraseFromParent();
      return true;
    }
    case ARM::RRX: {
      // This encodes as "MOVs Rd, Rm, rrx
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2609-2621
```cpp
      BuildMI(MBB, MBBI, MI.getDebugLoc(), TII->get(ARM::MOVsi),
              MI.getOperand(0).getReg())
          .add(MI.getOperand(1))
          .addImm(ARM_AM::getSORegOpc(ARM_AM::rrx, 0))
          .add(predOps(ARMCC::AL))
          .add(condCodeOp())
          .copyImplicitOps(MI);
      MI.eraseFromParent();
      return true;
    }
    case ARM::tTPsoft:
    case ARM::TPsoft: {
      const bool Thumb = Opcode == ARM::tTPsoft;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2623-2637
```cpp
      MachineInstrBuilder MIB;
      MachineFunction *MF = MBB.getParent();
      if (STI->genLongCalls()) {
        MachineConstantPool *MCP = MF->getConstantPool();
        unsigned PCLabelID = AFI->createPICLabelUId();
        MachineConstantPoolValue *CPV =
            ARMConstantPoolSymbol::Create(MF->getFunction().getContext(),
                                          "__aeabi_read_tp", PCLabelID, 0);
        Register Reg = MI.getOperand(0).getReg();
        MIB =
            BuildMI(MBB, MBBI, MI.getDebugLoc(),
                    TII->get(Thumb ? ARM::tLDRpci : ARM::LDRi12), Reg)
                .addConstantPoolIndex(MCP->getConstantPoolIndex(CPV, Align(4)));
        if (!Thumb)
          MIB.addImm(0);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2638-2638
```cpp
        MIB.add(predOps(ARMCC::AL));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2640-2652
```cpp
        MIB =
            BuildMI(MBB, MBBI, MI.getDebugLoc(),
                    TII->get(Thumb ? gettBLXrOpcode(*MF) : getBLXOpcode(*MF)));
        if (Thumb)
          MIB.add(predOps(ARMCC::AL));
        MIB.addReg(Reg, RegState::Kill);
      } else {
        MIB = BuildMI(MBB, MBBI, MI.getDebugLoc(),
                      TII->get(Thumb ? ARM::tBL : ARM::BL));
        if (Thumb)
          MIB.add(predOps(ARMCC::AL));
        MIB.addExternalSymbol("__aeabi_read_tp", 0);
      }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2654-2668
```cpp
      MIB.cloneMemRefs(MI);
      MIB.copyImplicitOps(MI);
      // Update the call info.
      if (MI.isCandidateForAdditionalCallInfo())
        MF->moveAdditionalCallInfo(&MI, &*MIB);
      MI.eraseFromParent();
      return true;
    }
    case ARM::tLDRpci_pic:
    case ARM::t2LDRpci_pic: {
      unsigned NewLdOpc = (Opcode == ARM::tLDRpci_pic)
        ? ARM::tLDRpci : ARM::t2LDRpci;
      Register DstReg = MI.getOperand(0).getReg();
      bool DstIsDead = MI.getOperand(0).isDead();
      BuildMI(MBB, MBBI, MI.getDebugLoc(), TII->get(NewLdOpc), DstReg)
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2669-2680
```cpp
          .add(MI.getOperand(1))
          .add(predOps(ARMCC::AL))
          .cloneMemRefs(MI)
          .copyImplicitOps(MI);
      BuildMI(MBB, MBBI, MI.getDebugLoc(), TII->get(ARM::tPICADD))
          .addReg(DstReg, RegState::Define | getDeadRegState(DstIsDead))
          .addReg(DstReg)
          .add(MI.getOperand(2))
          .copyImplicitOps(MI);
      MI.eraseFromParent();
      return true;
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2682-2696
```cpp
    case ARM::LDRLIT_ga_abs:
    case ARM::LDRLIT_ga_pcrel:
    case ARM::LDRLIT_ga_pcrel_ldr:
    case ARM::tLDRLIT_ga_abs:
    case ARM::t2LDRLIT_ga_pcrel:
    case ARM::tLDRLIT_ga_pcrel: {
      Register DstReg = MI.getOperand(0).getReg();
      bool DstIsDead = MI.getOperand(0).isDead();
      const MachineOperand &MO1 = MI.getOperand(1);
      auto Flags = MO1.getTargetFlags();
      const GlobalValue *GV = MO1.getGlobal();
      bool IsARM = Opcode != ARM::tLDRLIT_ga_pcrel &&
                   Opcode != ARM::tLDRLIT_ga_abs &&
                   Opcode != ARM::t2LDRLIT_ga_pcrel;
      bool IsPIC =
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2697-2704
```cpp
          Opcode != ARM::LDRLIT_ga_abs && Opcode != ARM::tLDRLIT_ga_abs;
      unsigned LDRLITOpc = IsARM ? ARM::LDRi12 : ARM::tLDRpci;
      if (Opcode == ARM::t2LDRLIT_ga_pcrel)
        LDRLITOpc = ARM::t2LDRpci;
      unsigned PICAddOpc =
          IsARM
              ? (Opcode == ARM::LDRLIT_ga_pcrel_ldr ? ARM::PICLDR : ARM::PICADD)
              : ARM::tPICADD;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2706-2709
```cpp
      // We need a new const-pool entry to load from.
      MachineConstantPool *MCP = MBB.getParent()->getConstantPool();
      unsigned ARMPCLabelIndex = 0;
      MachineConstantPoolValue *CPV;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2711-2721
```cpp
      if (IsPIC) {
        unsigned PCAdj = IsARM ? 8 : 4;
        auto Modifier = (Flags & ARMII::MO_GOT)
                            ? ARMCP::GOT_PREL
                            : ARMCP::no_modifier;
        ARMPCLabelIndex = AFI->createPICLabelUId();
        CPV = ARMConstantPoolConstant::Create(
            GV, ARMPCLabelIndex, ARMCP::CPValue, PCAdj, Modifier,
            /*AddCurrentAddr*/ Modifier == ARMCP::GOT_PREL);
      } else
        CPV = ARMConstantPoolConstant::Create(GV, ARMCP::no_modifier);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2723-2728
```cpp
      MachineInstrBuilder MIB =
          BuildMI(MBB, MBBI, MI.getDebugLoc(), TII->get(LDRLITOpc), DstReg)
              .addConstantPoolIndex(MCP->getConstantPoolIndex(CPV, Align(4)));
      if (IsARM)
        MIB.addImm(0);
      MIB.add(predOps(ARMCC::AL));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2730-2735
```cpp
      if (IsPIC) {
        MachineInstrBuilder MIB =
          BuildMI(MBB, MBBI, MI.getDebugLoc(), TII->get(PICAddOpc))
            .addReg(DstReg, RegState::Define | getDeadRegState(DstIsDead))
            .addReg(DstReg)
            .addImm(ARMPCLabelIndex);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2737-2739
```cpp
        if (IsARM)
          MIB.add(predOps(ARMCC::AL));
      }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2741-2755
```cpp
      MI.eraseFromParent();
      return true;
    }
    case ARM::MOV_ga_pcrel:
    case ARM::MOV_ga_pcrel_ldr:
    case ARM::t2MOV_ga_pcrel: {
      // Expand into movw + movw. Also "add pc" / ldr [pc] in PIC mode.
      unsigned LabelId = AFI->createPICLabelUId();
      Register DstReg = MI.getOperand(0).getReg();
      bool DstIsDead = MI.getOperand(0).isDead();
      const MachineOperand &MO1 = MI.getOperand(1);
      const GlobalValue *GV = MO1.getGlobal();
      unsigned TF = MO1.getTargetFlags();
      bool isARM = Opcode != ARM::t2MOV_ga_pcrel;
      unsigned LO16Opc = isARM ? ARM::MOVi16_ga_pcrel : ARM::t2MOVi16_ga_pcrel;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2756-2765
```cpp
      unsigned HI16Opc = isARM ? ARM::MOVTi16_ga_pcrel :ARM::t2MOVTi16_ga_pcrel;
      unsigned LO16TF = TF | ARMII::MO_LO16;
      unsigned HI16TF = TF | ARMII::MO_HI16;
      unsigned PICAddOpc = isARM
        ? (Opcode == ARM::MOV_ga_pcrel_ldr ? ARM::PICLDR : ARM::PICADD)
        : ARM::tPICADD;
      BuildMI(MBB, MBBI, MI.getDebugLoc(), TII->get(LO16Opc), DstReg)
          .addGlobalAddress(GV, MO1.getOffset(), TF | LO16TF)
          .addImm(LabelId)
          .copyImplicitOps(MI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2767-2771
```cpp
      BuildMI(MBB, MBBI, MI.getDebugLoc(), TII->get(HI16Opc), DstReg)
          .addReg(DstReg)
          .addGlobalAddress(GV, MO1.getOffset(), TF | HI16TF)
          .addImm(LabelId)
          .copyImplicitOps(MI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2773-2785
```cpp
      MachineInstrBuilder MIB3 = BuildMI(MBB, MBBI, MI.getDebugLoc(),
                                         TII->get(PICAddOpc))
        .addReg(DstReg, RegState::Define | getDeadRegState(DstIsDead))
        .addReg(DstReg).addImm(LabelId);
      if (isARM) {
        MIB3.add(predOps(ARMCC::AL));
        if (Opcode == ARM::MOV_ga_pcrel_ldr)
          MIB3.cloneMemRefs(MI);
      }
      MIB3.copyImplicitOps(MI);
      MI.eraseFromParent();
      return true;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2787-2792
```cpp
    case ARM::MOVi32imm:
    case ARM::MOVCCi32imm:
    case ARM::t2MOVi32imm:
    case ARM::t2MOVCCi32imm:
      ExpandMOV32BitImm(MBB, MBBI);
      return true;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2794-2796
```cpp
    case ARM::tMOVi32imm:
      ExpandTMOV32BitImm(MBB, MBBI);
      return true;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2798-2802
```cpp
    case ARM::tLEApcrelJT:
      // Inline jump tables are handled in ARMAsmPrinter.
      if (MI.getMF()->getJumpTableInfo()->getEntryKind() ==
          MachineJumpTableInfo::EK_Inline)
        return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2804-2807
```cpp
      // Use a 32-bit immediate move to generate the address of the jump table.
      assert(STI->isThumb() && "Non-inline jump tables expected only in thumb");
      ExpandTMOV32BitImm(MBB, MBBI);
      return true;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2809-2823
```cpp
    case ARM::SUBS_PC_LR: {
      BuildMI(MBB, MBBI, MI.getDebugLoc(), TII->get(ARM::SUBri), ARM::PC)
          .addReg(ARM::LR)
          .add(MI.getOperand(0))
          .add(MI.getOperand(1))
          .add(MI.getOperand(2))
          .addReg(ARM::CPSR, RegState::Undef)
          .copyImplicitOps(MI);
      MI.eraseFromParent();
      return true;
    }
    case ARM::VLDMQIA: {
      unsigned NewOpc = ARM::VLDMDIA;
      MachineInstrBuilder MIB =
        BuildMI(MBB, MBBI, MI.getDebugLoc(), TII->get(NewOpc));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2824-2824
```cpp
      unsigned OpIdx = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2826-2828
```cpp
      // Grab the Q register destination.
      bool DstIsDead = MI.getOperand(OpIdx).isDead();
      Register DstReg = MI.getOperand(OpIdx++).getReg();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2830-2831
```cpp
      // Copy the source register.
      MIB.add(MI.getOperand(OpIdx++));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2833-2835
```cpp
      // Copy the predicate operands.
      MIB.add(MI.getOperand(OpIdx++));
      MIB.add(MI.getOperand(OpIdx++));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2837-2841
```cpp
      // Add the destination operands (D subregs).
      Register D0 = TRI->getSubReg(DstReg, ARM::dsub_0);
      Register D1 = TRI->getSubReg(DstReg, ARM::dsub_1);
      MIB.addReg(D0, RegState::Define | getDeadRegState(DstIsDead))
        .addReg(D1, RegState::Define | getDeadRegState(DstIsDead));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2843-2849
```cpp
      // Add an implicit def for the super-register.
      MIB.addReg(DstReg, RegState::ImplicitDefine | getDeadRegState(DstIsDead));
      MIB.copyImplicitOps(MI);
      MIB.cloneMemRefs(MI);
      MI.eraseFromParent();
      return true;
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2851-2855
```cpp
    case ARM::VSTMQIA: {
      unsigned NewOpc = ARM::VSTMDIA;
      MachineInstrBuilder MIB =
        BuildMI(MBB, MBBI, MI.getDebugLoc(), TII->get(NewOpc));
      unsigned OpIdx = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2857-2859
```cpp
      // Grab the Q register source.
      bool SrcIsKill = MI.getOperand(OpIdx).isKill();
      Register SrcReg = MI.getOperand(OpIdx++).getReg();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2861-2863
```cpp
      // Copy the destination register.
      MachineOperand Dst(MI.getOperand(OpIdx++));
      MIB.add(Dst);
```
- EN: Declares `Dst`, a target-specific routine centered on machine operands. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `Dst`，它是一个围绕机器操作数展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2865-2867
```cpp
      // Copy the predicate operands.
      MIB.add(MI.getOperand(OpIdx++));
      MIB.add(MI.getOperand(OpIdx++));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2869-2873
```cpp
      // Add the source operands (D subregs).
      Register D0 = TRI->getSubReg(SrcReg, ARM::dsub_0);
      Register D1 = TRI->getSubReg(SrcReg, ARM::dsub_1);
      MIB.addReg(D0, getKillRegState(SrcIsKill))
          .addReg(D1, getKillRegState(SrcIsKill));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2875-2876
```cpp
      if (SrcIsKill)      // Add an implicit kill for the Q register.
        MIB->addRegisterKilled(SrcReg, TRI, true);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2878-2882
```cpp
      MIB.copyImplicitOps(MI);
      MIB.cloneMemRefs(MI);
      MI.eraseFromParent();
      return true;
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2884-2898
```cpp
    case ARM::VLD2q8Pseudo:
    case ARM::VLD2q16Pseudo:
    case ARM::VLD2q32Pseudo:
    case ARM::VLD2q8PseudoWB_fixed:
    case ARM::VLD2q16PseudoWB_fixed:
    case ARM::VLD2q32PseudoWB_fixed:
    case ARM::VLD2q8PseudoWB_register:
    case ARM::VLD2q16PseudoWB_register:
    case ARM::VLD2q32PseudoWB_register:
    case ARM::VLD3d8Pseudo:
    case ARM::VLD3d16Pseudo:
    case ARM::VLD3d32Pseudo:
    case ARM::VLD1d8TPseudo:
    case ARM::VLD1d8TPseudoWB_fixed:
    case ARM::VLD1d8TPseudoWB_register:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2899-2913
```cpp
    case ARM::VLD1d16TPseudo:
    case ARM::VLD1d16TPseudoWB_fixed:
    case ARM::VLD1d16TPseudoWB_register:
    case ARM::VLD1d32TPseudo:
    case ARM::VLD1d32TPseudoWB_fixed:
    case ARM::VLD1d32TPseudoWB_register:
    case ARM::VLD1d64TPseudo:
    case ARM::VLD1d64TPseudoWB_fixed:
    case ARM::VLD1d64TPseudoWB_register:
    case ARM::VLD3d8Pseudo_UPD:
    case ARM::VLD3d16Pseudo_UPD:
    case ARM::VLD3d32Pseudo_UPD:
    case ARM::VLD3q8Pseudo_UPD:
    case ARM::VLD3q16Pseudo_UPD:
    case ARM::VLD3q32Pseudo_UPD:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2914-2928
```cpp
    case ARM::VLD3q8oddPseudo:
    case ARM::VLD3q16oddPseudo:
    case ARM::VLD3q32oddPseudo:
    case ARM::VLD3q8oddPseudo_UPD:
    case ARM::VLD3q16oddPseudo_UPD:
    case ARM::VLD3q32oddPseudo_UPD:
    case ARM::VLD4d8Pseudo:
    case ARM::VLD4d16Pseudo:
    case ARM::VLD4d32Pseudo:
    case ARM::VLD1d8QPseudo:
    case ARM::VLD1d8QPseudoWB_fixed:
    case ARM::VLD1d8QPseudoWB_register:
    case ARM::VLD1d16QPseudo:
    case ARM::VLD1d16QPseudoWB_fixed:
    case ARM::VLD1d16QPseudoWB_register:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2929-2943
```cpp
    case ARM::VLD1d32QPseudo:
    case ARM::VLD1d32QPseudoWB_fixed:
    case ARM::VLD1d32QPseudoWB_register:
    case ARM::VLD1d64QPseudo:
    case ARM::VLD1d64QPseudoWB_fixed:
    case ARM::VLD1d64QPseudoWB_register:
    case ARM::VLD1q8HighQPseudo:
    case ARM::VLD1q8HighQPseudo_UPD:
    case ARM::VLD1q8LowQPseudo_UPD:
    case ARM::VLD1q8HighTPseudo:
    case ARM::VLD1q8HighTPseudo_UPD:
    case ARM::VLD1q8LowTPseudo_UPD:
    case ARM::VLD1q16HighQPseudo:
    case ARM::VLD1q16HighQPseudo_UPD:
    case ARM::VLD1q16LowQPseudo_UPD:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2944-2958
```cpp
    case ARM::VLD1q16HighTPseudo:
    case ARM::VLD1q16HighTPseudo_UPD:
    case ARM::VLD1q16LowTPseudo_UPD:
    case ARM::VLD1q32HighQPseudo:
    case ARM::VLD1q32HighQPseudo_UPD:
    case ARM::VLD1q32LowQPseudo_UPD:
    case ARM::VLD1q32HighTPseudo:
    case ARM::VLD1q32HighTPseudo_UPD:
    case ARM::VLD1q32LowTPseudo_UPD:
    case ARM::VLD1q64HighQPseudo:
    case ARM::VLD1q64HighQPseudo_UPD:
    case ARM::VLD1q64LowQPseudo_UPD:
    case ARM::VLD1q64HighTPseudo:
    case ARM::VLD1q64HighTPseudo_UPD:
    case ARM::VLD1q64LowTPseudo_UPD:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2959-2973
```cpp
    case ARM::VLD4d8Pseudo_UPD:
    case ARM::VLD4d16Pseudo_UPD:
    case ARM::VLD4d32Pseudo_UPD:
    case ARM::VLD4q8Pseudo_UPD:
    case ARM::VLD4q16Pseudo_UPD:
    case ARM::VLD4q32Pseudo_UPD:
    case ARM::VLD4q8oddPseudo:
    case ARM::VLD4q16oddPseudo:
    case ARM::VLD4q32oddPseudo:
    case ARM::VLD4q8oddPseudo_UPD:
    case ARM::VLD4q16oddPseudo_UPD:
    case ARM::VLD4q32oddPseudo_UPD:
    case ARM::VLD3DUPd8Pseudo:
    case ARM::VLD3DUPd16Pseudo:
    case ARM::VLD3DUPd32Pseudo:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2974-2988
```cpp
    case ARM::VLD3DUPd8Pseudo_UPD:
    case ARM::VLD3DUPd16Pseudo_UPD:
    case ARM::VLD3DUPd32Pseudo_UPD:
    case ARM::VLD4DUPd8Pseudo:
    case ARM::VLD4DUPd16Pseudo:
    case ARM::VLD4DUPd32Pseudo:
    case ARM::VLD4DUPd8Pseudo_UPD:
    case ARM::VLD4DUPd16Pseudo_UPD:
    case ARM::VLD4DUPd32Pseudo_UPD:
    case ARM::VLD2DUPq8EvenPseudo:
    case ARM::VLD2DUPq8OddPseudo:
    case ARM::VLD2DUPq16EvenPseudo:
    case ARM::VLD2DUPq16OddPseudo:
    case ARM::VLD2DUPq32EvenPseudo:
    case ARM::VLD2DUPq32OddPseudo:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2989-3003
```cpp
    case ARM::VLD2DUPq8OddPseudoWB_fixed:
    case ARM::VLD2DUPq8OddPseudoWB_register:
    case ARM::VLD2DUPq16OddPseudoWB_fixed:
    case ARM::VLD2DUPq16OddPseudoWB_register:
    case ARM::VLD2DUPq32OddPseudoWB_fixed:
    case ARM::VLD2DUPq32OddPseudoWB_register:
    case ARM::VLD3DUPq8EvenPseudo:
    case ARM::VLD3DUPq8OddPseudo:
    case ARM::VLD3DUPq16EvenPseudo:
    case ARM::VLD3DUPq16OddPseudo:
    case ARM::VLD3DUPq32EvenPseudo:
    case ARM::VLD3DUPq32OddPseudo:
    case ARM::VLD3DUPq8OddPseudo_UPD:
    case ARM::VLD3DUPq16OddPseudo_UPD:
    case ARM::VLD3DUPq32OddPseudo_UPD:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3004-3014
```cpp
    case ARM::VLD4DUPq8EvenPseudo:
    case ARM::VLD4DUPq8OddPseudo:
    case ARM::VLD4DUPq16EvenPseudo:
    case ARM::VLD4DUPq16OddPseudo:
    case ARM::VLD4DUPq32EvenPseudo:
    case ARM::VLD4DUPq32OddPseudo:
    case ARM::VLD4DUPq8OddPseudo_UPD:
    case ARM::VLD4DUPq16OddPseudo_UPD:
    case ARM::VLD4DUPq32OddPseudo_UPD:
      ExpandVLD(MBBI);
      return true;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3016-3030
```cpp
    case ARM::VST2q8Pseudo:
    case ARM::VST2q16Pseudo:
    case ARM::VST2q32Pseudo:
    case ARM::VST2q8PseudoWB_fixed:
    case ARM::VST2q16PseudoWB_fixed:
    case ARM::VST2q32PseudoWB_fixed:
    case ARM::VST2q8PseudoWB_register:
    case ARM::VST2q16PseudoWB_register:
    case ARM::VST2q32PseudoWB_register:
    case ARM::VST3d8Pseudo:
    case ARM::VST3d16Pseudo:
    case ARM::VST3d32Pseudo:
    case ARM::VST1d8TPseudo:
    case ARM::VST1d8TPseudoWB_fixed:
    case ARM::VST1d8TPseudoWB_register:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3031-3045
```cpp
    case ARM::VST1d16TPseudo:
    case ARM::VST1d16TPseudoWB_fixed:
    case ARM::VST1d16TPseudoWB_register:
    case ARM::VST1d32TPseudo:
    case ARM::VST1d32TPseudoWB_fixed:
    case ARM::VST1d32TPseudoWB_register:
    case ARM::VST1d64TPseudo:
    case ARM::VST1d64TPseudoWB_fixed:
    case ARM::VST1d64TPseudoWB_register:
    case ARM::VST3d8Pseudo_UPD:
    case ARM::VST3d16Pseudo_UPD:
    case ARM::VST3d32Pseudo_UPD:
    case ARM::VST3q8Pseudo_UPD:
    case ARM::VST3q16Pseudo_UPD:
    case ARM::VST3q32Pseudo_UPD:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3046-3060
```cpp
    case ARM::VST3q8oddPseudo:
    case ARM::VST3q16oddPseudo:
    case ARM::VST3q32oddPseudo:
    case ARM::VST3q8oddPseudo_UPD:
    case ARM::VST3q16oddPseudo_UPD:
    case ARM::VST3q32oddPseudo_UPD:
    case ARM::VST4d8Pseudo:
    case ARM::VST4d16Pseudo:
    case ARM::VST4d32Pseudo:
    case ARM::VST1d8QPseudo:
    case ARM::VST1d8QPseudoWB_fixed:
    case ARM::VST1d8QPseudoWB_register:
    case ARM::VST1d16QPseudo:
    case ARM::VST1d16QPseudoWB_fixed:
    case ARM::VST1d16QPseudoWB_register:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3061-3075
```cpp
    case ARM::VST1d32QPseudo:
    case ARM::VST1d32QPseudoWB_fixed:
    case ARM::VST1d32QPseudoWB_register:
    case ARM::VST1d64QPseudo:
    case ARM::VST1d64QPseudoWB_fixed:
    case ARM::VST1d64QPseudoWB_register:
    case ARM::VST4d8Pseudo_UPD:
    case ARM::VST4d16Pseudo_UPD:
    case ARM::VST4d32Pseudo_UPD:
    case ARM::VST1q8HighQPseudo:
    case ARM::VST1q8LowQPseudo_UPD:
    case ARM::VST1q8HighTPseudo:
    case ARM::VST1q8LowTPseudo_UPD:
    case ARM::VST1q16HighQPseudo:
    case ARM::VST1q16LowQPseudo_UPD:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3076-3090
```cpp
    case ARM::VST1q16HighTPseudo:
    case ARM::VST1q16LowTPseudo_UPD:
    case ARM::VST1q32HighQPseudo:
    case ARM::VST1q32LowQPseudo_UPD:
    case ARM::VST1q32HighTPseudo:
    case ARM::VST1q32LowTPseudo_UPD:
    case ARM::VST1q64HighQPseudo:
    case ARM::VST1q64LowQPseudo_UPD:
    case ARM::VST1q64HighTPseudo:
    case ARM::VST1q64LowTPseudo_UPD:
    case ARM::VST1q8HighTPseudo_UPD:
    case ARM::VST1q16HighTPseudo_UPD:
    case ARM::VST1q32HighTPseudo_UPD:
    case ARM::VST1q64HighTPseudo_UPD:
    case ARM::VST1q8HighQPseudo_UPD:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3091-3104
```cpp
    case ARM::VST1q16HighQPseudo_UPD:
    case ARM::VST1q32HighQPseudo_UPD:
    case ARM::VST1q64HighQPseudo_UPD:
    case ARM::VST4q8Pseudo_UPD:
    case ARM::VST4q16Pseudo_UPD:
    case ARM::VST4q32Pseudo_UPD:
    case ARM::VST4q8oddPseudo:
    case ARM::VST4q16oddPseudo:
    case ARM::VST4q32oddPseudo:
    case ARM::VST4q8oddPseudo_UPD:
    case ARM::VST4q16oddPseudo_UPD:
    case ARM::VST4q32oddPseudo_UPD:
      ExpandVST(MBBI);
      return true;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3106-3120
```cpp
    case ARM::VLD1LNq8Pseudo:
    case ARM::VLD1LNq16Pseudo:
    case ARM::VLD1LNq32Pseudo:
    case ARM::VLD1LNq8Pseudo_UPD:
    case ARM::VLD1LNq16Pseudo_UPD:
    case ARM::VLD1LNq32Pseudo_UPD:
    case ARM::VLD2LNd8Pseudo:
    case ARM::VLD2LNd16Pseudo:
    case ARM::VLD2LNd32Pseudo:
    case ARM::VLD2LNq16Pseudo:
    case ARM::VLD2LNq32Pseudo:
    case ARM::VLD2LNd8Pseudo_UPD:
    case ARM::VLD2LNd16Pseudo_UPD:
    case ARM::VLD2LNd32Pseudo_UPD:
    case ARM::VLD2LNq16Pseudo_UPD:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3121-3135
```cpp
    case ARM::VLD2LNq32Pseudo_UPD:
    case ARM::VLD3LNd8Pseudo:
    case ARM::VLD3LNd16Pseudo:
    case ARM::VLD3LNd32Pseudo:
    case ARM::VLD3LNq16Pseudo:
    case ARM::VLD3LNq32Pseudo:
    case ARM::VLD3LNd8Pseudo_UPD:
    case ARM::VLD3LNd16Pseudo_UPD:
    case ARM::VLD3LNd32Pseudo_UPD:
    case ARM::VLD3LNq16Pseudo_UPD:
    case ARM::VLD3LNq32Pseudo_UPD:
    case ARM::VLD4LNd8Pseudo:
    case ARM::VLD4LNd16Pseudo:
    case ARM::VLD4LNd32Pseudo:
    case ARM::VLD4LNq16Pseudo:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3136-3150
```cpp
    case ARM::VLD4LNq32Pseudo:
    case ARM::VLD4LNd8Pseudo_UPD:
    case ARM::VLD4LNd16Pseudo_UPD:
    case ARM::VLD4LNd32Pseudo_UPD:
    case ARM::VLD4LNq16Pseudo_UPD:
    case ARM::VLD4LNq32Pseudo_UPD:
    case ARM::VST1LNq8Pseudo:
    case ARM::VST1LNq16Pseudo:
    case ARM::VST1LNq32Pseudo:
    case ARM::VST1LNq8Pseudo_UPD:
    case ARM::VST1LNq16Pseudo_UPD:
    case ARM::VST1LNq32Pseudo_UPD:
    case ARM::VST2LNd8Pseudo:
    case ARM::VST2LNd16Pseudo:
    case ARM::VST2LNd32Pseudo:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3151-3165
```cpp
    case ARM::VST2LNq16Pseudo:
    case ARM::VST2LNq32Pseudo:
    case ARM::VST2LNd8Pseudo_UPD:
    case ARM::VST2LNd16Pseudo_UPD:
    case ARM::VST2LNd32Pseudo_UPD:
    case ARM::VST2LNq16Pseudo_UPD:
    case ARM::VST2LNq32Pseudo_UPD:
    case ARM::VST3LNd8Pseudo:
    case ARM::VST3LNd16Pseudo:
    case ARM::VST3LNd32Pseudo:
    case ARM::VST3LNq16Pseudo:
    case ARM::VST3LNq32Pseudo:
    case ARM::VST3LNd8Pseudo_UPD:
    case ARM::VST3LNd16Pseudo_UPD:
    case ARM::VST3LNd32Pseudo_UPD:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3166-3179
```cpp
    case ARM::VST3LNq16Pseudo_UPD:
    case ARM::VST3LNq32Pseudo_UPD:
    case ARM::VST4LNd8Pseudo:
    case ARM::VST4LNd16Pseudo:
    case ARM::VST4LNd32Pseudo:
    case ARM::VST4LNq16Pseudo:
    case ARM::VST4LNq32Pseudo:
    case ARM::VST4LNd8Pseudo_UPD:
    case ARM::VST4LNd16Pseudo_UPD:
    case ARM::VST4LNd32Pseudo_UPD:
    case ARM::VST4LNq16Pseudo_UPD:
    case ARM::VST4LNq32Pseudo_UPD:
      ExpandLaneOp(MBBI);
      return true;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3181-3184
```cpp
    case ARM::VTBL3Pseudo: ExpandVTBL(MBBI, ARM::VTBL3, false); return true;
    case ARM::VTBL4Pseudo: ExpandVTBL(MBBI, ARM::VTBL4, false); return true;
    case ARM::VTBX3Pseudo: ExpandVTBL(MBBI, ARM::VTBX3, true); return true;
    case ARM::VTBX4Pseudo: ExpandVTBL(MBBI, ARM::VTBX4, true); return true;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3186-3191
```cpp
    case ARM::MQQPRLoad:
    case ARM::MQQPRStore:
    case ARM::MQQQQPRLoad:
    case ARM::MQQQQPRStore:
      ExpandMQQPRLoadStore(MBBI);
      return true;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3193-3203
```cpp
    case ARM::tCMP_SWAP_8:
      assert(STI->isThumb());
      return ExpandCMP_SWAP(MBB, MBBI, ARM::t2LDREXB, ARM::t2STREXB, ARM::tUXTB,
                            NextMBBI);
    case ARM::tCMP_SWAP_16:
      assert(STI->isThumb());
      return ExpandCMP_SWAP(MBB, MBBI, ARM::t2LDREXH, ARM::t2STREXH, ARM::tUXTH,
                            NextMBBI);
    case ARM::tCMP_SWAP_32:
      assert(STI->isThumb());
      return ExpandCMP_SWAP(MBB, MBBI, ARM::t2LDREX, ARM::t2STREX, 0, NextMBBI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3205-3215
```cpp
    case ARM::CMP_SWAP_8:
      assert(!STI->isThumb());
      return ExpandCMP_SWAP(MBB, MBBI, ARM::LDREXB, ARM::STREXB, ARM::UXTB,
                            NextMBBI);
    case ARM::CMP_SWAP_16:
      assert(!STI->isThumb());
      return ExpandCMP_SWAP(MBB, MBBI, ARM::LDREXH, ARM::STREXH, ARM::UXTH,
                            NextMBBI);
    case ARM::CMP_SWAP_32:
      assert(!STI->isThumb());
      return ExpandCMP_SWAP(MBB, MBBI, ARM::LDREX, ARM::STREX, 0, NextMBBI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3217-3218
```cpp
    case ARM::CMP_SWAP_64:
      return ExpandCMP_SWAP_64(MBB, MBBI, NextMBBI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3220-3230
```cpp
    case ARM::tBL_PUSHLR:
    case ARM::BL_PUSHLR: {
      const bool Thumb = Opcode == ARM::tBL_PUSHLR;
      Register Reg = MI.getOperand(0).getReg();
      assert(Reg == ARM::LR && "expect LR register!");
      MachineInstrBuilder MIB;
      if (Thumb) {
        // push {lr}
        BuildMI(MBB, MBBI, MI.getDebugLoc(), TII->get(ARM::tPUSH))
            .add(predOps(ARMCC::AL))
            .addReg(Reg);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3232-3240
```cpp
        // bl __gnu_mcount_nc
        MIB = BuildMI(MBB, MBBI, MI.getDebugLoc(), TII->get(ARM::tBL));
      } else {
        // stmdb   sp!, {lr}
        BuildMI(MBB, MBBI, MI.getDebugLoc(), TII->get(ARM::STMDB_UPD))
            .addReg(ARM::SP, RegState::Define)
            .addReg(ARM::SP)
            .add(predOps(ARMCC::AL))
            .addReg(Reg);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3242-3256
```cpp
        // bl __gnu_mcount_nc
        MIB = BuildMI(MBB, MBBI, MI.getDebugLoc(), TII->get(ARM::BL));
      }
      MIB.cloneMemRefs(MI);
      for (const MachineOperand &MO : llvm::drop_begin(MI.operands()))
        MIB.add(MO);
      MI.eraseFromParent();
      return true;
    }
    case ARM::t2CALL_BTI: {
      MachineFunction &MF = *MI.getMF();
      MachineInstrBuilder MIB =
          BuildMI(MF, MI.getDebugLoc(), TII->get(ARM::tBL));
      MIB.cloneMemRefs(MI);
      for (unsigned i = 0; i < MI.getNumOperands(); ++i)
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3257-3269
```cpp
        MIB.add(MI.getOperand(i));
      if (MI.isCandidateForAdditionalCallInfo())
        MF.moveAdditionalCallInfo(&MI, MIB.getInstr());
      MIBundleBuilder Bundler(MBB, MI);
      Bundler.append(MIB);
      Bundler.append(BuildMI(MF, MI.getDebugLoc(), TII->get(ARM::t2BTI)));
      finalizeBundle(MBB, Bundler.begin(), Bundler.end());
      MI.eraseFromParent();
      return true;
    }
    case ARM::LOADDUAL:
    case ARM::STOREDUAL: {
      Register PairReg = MI.getOperand(0).getReg();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3271-3285
```cpp
      MachineInstrBuilder MIB =
          BuildMI(MBB, MBBI, MI.getDebugLoc(),
                  TII->get(Opcode == ARM::LOADDUAL ? ARM::LDRD : ARM::STRD))
              .addReg(TRI->getSubReg(PairReg, ARM::gsub_0),
                      getDefRegState(Opcode == ARM::LOADDUAL))
              .addReg(TRI->getSubReg(PairReg, ARM::gsub_1),
                      getDefRegState(Opcode == ARM::LOADDUAL));
      for (const MachineOperand &MO : llvm::drop_begin(MI.operands()))
        MIB.add(MO);
      MIB.add(predOps(ARMCC::AL));
      MIB.cloneMemRefs(MI);
      MI.eraseFromParent();
      return true;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3286-3286
```cpp
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3288-3289
```cpp
bool ARMExpandPseudo::ExpandMBB(MachineBasicBlock &MBB) {
  bool Modified = false;
```
- EN: Implements `ARMExpandPseudo::ExpandMBB`, a mutation/build routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMExpandPseudo::ExpandMBB`，它是一个围绕机器基本块展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3291-3296
```cpp
  MachineBasicBlock::iterator MBBI = MBB.begin(), E = MBB.end();
  while (MBBI != E) {
    MachineBasicBlock::iterator NMBBI = std::next(MBBI);
    Modified |= ExpandMI(MBB, MBBI, NMBBI);
    MBBI = NMBBI;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3298-3299
```cpp
  return Modified;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3301-3305
```cpp
bool ARMExpandPseudo::runOnMachineFunction(MachineFunction &MF) {
  STI = &MF.getSubtarget<ARMSubtarget>();
  TII = STI->getInstrInfo();
  TRI = STI->getRegisterInfo();
  AFI = MF.getInfo<ARMFunctionInfo>();
```
- EN: Implements `ARMExpandPseudo::runOnMachineFunction`, a pass-entry routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMExpandPseudo::runOnMachineFunction`，它是一个围绕机器函数状态展开的Pass 入口例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3307-3308
```cpp
  LLVM_DEBUG(dbgs() << "********** ARM EXPAND PSEUDO INSTRUCTIONS **********\n"
                    << "********** Function: " << MF.getName() << '\n');
```
- EN: Declares `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3310-3314
```cpp
  bool Modified = false;
  for (MachineBasicBlock &MBB : MF)
    Modified |= ExpandMBB(MBB);
  if (VerifyARMPseudo)
    MF.verify(this, "After expanding ARM pseudo instructions.");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3316-3318
```cpp
  LLVM_DEBUG(dbgs() << "***************************************************\n");
  return Modified;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3320-3324
```cpp
/// createARMExpandPseudoPass - returns an instance of the pseudo instruction
/// expansion pass.
FunctionPass *llvm::createARMExpandPseudoPass() {
  return new ARMExpandPseudo();
}
```
- EN: Implements `llvm::createARMExpandPseudoPass`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `llvm::createARMExpandPseudoPass`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

## Key Concepts / 关键概念

- EN: Primary role: target-specific code-generation behavior.
  - CN: 核心职责：目标相关代码生成行为。
- EN: MachineInstr-level manipulation is central to this file.
  - CN: 该文件以 MachineInstr 层面的操作为核心。
- EN: MachineBasicBlock flow and branch structure matter to the implementation.
  - CN: 实现高度依赖 MachineBasicBlock 流程与分支结构。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。
- EN: Stack-frame and spill/reload concerns are reflected in the API or implementation.
  - CN: API 或实现中体现了栈帧以及溢出/重载相关问题。

## Dependencies / 依赖关系

- EN: Backend-local headers: `ARM.h`, `ARMBaseInstrInfo.h`, `ARMBaseRegisterInfo.h`, `ARMConstantPoolValue.h`, `ARMMachineFunctionInfo.h`, `ARMSubtarget.h`, `MCTargetDesc/ARMAddressingModes.h`.
  - CN: 后端本地头文件：`ARM.h`, `ARMBaseInstrInfo.h`, `ARMBaseRegisterInfo.h`, `ARMConstantPoolValue.h`, `ARMMachineFunctionInfo.h`, `ARMSubtarget.h`, `MCTargetDesc/ARMAddressingModes.h`。
- EN: LLVM infrastructure headers: `llvm/CodeGen/LivePhysRegs.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunctionPass.h`, `llvm/CodeGen/MachineJumpTableInfo.h`, `llvm/MC/MCAsmInfo.h`, `llvm/Support/Debug.h`.
  - CN: LLVM 基础设施头文件：`llvm/CodeGen/LivePhysRegs.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunctionPass.h`, `llvm/CodeGen/MachineJumpTableInfo.h`, `llvm/MC/MCAsmInfo.h`, `llvm/Support/Debug.h`。
- EN: Standard/system headers: `atomic`.
  - CN: 标准库/系统头文件：`atomic`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
