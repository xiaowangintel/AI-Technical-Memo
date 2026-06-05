# ARMFixCortexA57AES1742098Pass.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMFixCortexA57AES1742098Pass.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `ARMFixCortexA57AES1742098Pass` for the ARM backend, focusing on target-specific code-generation behavior.
- 用途 (CN): 实现 ARM 后端中的 `ARMFixCortexA57AES1742098Pass`，重点处理目标相关代码生成行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15
```cpp
//===-- ARMFixCortexA57AES1742098Pass.cpp ---------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
// This pass works around a Cortex Core Fused AES erratum:
// - Cortex-A57 Erratum 1742098
// - Cortex-A72 Erratum 1655431
//
// The erratum may be triggered if an input vector register to AESE or AESD was
// last written by an instruction that only updated 32 bits of it. This can
// occur for either of the input registers.
//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 16-30
```cpp
// The workaround chosen is to update the input register using `r = VORRq r, r`,
// as this updates all 128 bits of the register unconditionally, but does not
// change the values observed in `r`, making the input safe.
//
// This pass has to be conservative in a few cases:
// - an input vector register to the AES instruction is defined outside the
//   current function, where we have to assume the register was updated in an
//   unsafe way; and
// - an input vector register to the AES instruction is updated along multiple
//   different control-flow paths, where we have to ensure all the register
//   updating instructions are safe.
//
// Both of these cases may apply to a input vector register. In either case, we
// need to ensure that, when the pass is finished, there exists a safe
// instruction between every unsafe register updating instruction and the AES
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 31-33
```cpp
// instruction.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 35-49
```cpp
#include "ARM.h"
#include "ARMBaseInstrInfo.h"
#include "ARMBaseRegisterInfo.h"
#include "ARMSubtarget.h"
#include "Utils/ARMBaseInfo.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/SmallPtrSet.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/CodeGen/MachineBasicBlock.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineInstrBundleIterator.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 50-59
```cpp
#include "llvm/CodeGen/MachineOperand.h"
#include "llvm/CodeGen/ReachingDefAnalysis.h"
#include "llvm/CodeGen/Register.h"
#include "llvm/CodeGen/TargetRegisterInfo.h"
#include "llvm/IR/DebugLoc.h"
#include "llvm/Pass.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/raw_ostream.h"
#include <assert.h>
#include <stdint.h>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 61-61
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 63-63
```cpp
#define DEBUG_TYPE "arm-fix-cortex-a57-aes-1742098"
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 65-65
```cpp
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 67-71
```cpp
namespace {
class ARMFixCortexA57AES1742098 : public MachineFunctionPass {
public:
  static char ID;
  explicit ARMFixCortexA57AES1742098() : MachineFunctionPass(ID) {}
```
- EN: Declares `ARMFixCortexA57AES1742098`, packaging target-specific state and APIs around `ARMFixCortexA57AES1742098Pass`.
- CN: 这里声明 `ARMFixCortexA57AES1742098`，把与 `ARMFixCortexA57AES1742098Pass` 相关的目标特定状态和 API 组织在一起。

### Lines 73-73
```cpp
  bool runOnMachineFunction(MachineFunction &F) override;
```
- EN: Declares `runOnMachineFunction`, a pass-entry routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `runOnMachineFunction`，它是一个围绕机器函数状态展开的Pass 入口例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 75-77
```cpp
  MachineFunctionProperties getRequiredProperties() const override {
    return MachineFunctionProperties().setNoVRegs();
  }
```
- EN: Implements `getRequiredProperties`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getRequiredProperties`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 79-81
```cpp
  StringRef getPassName() const override {
    return "ARM fix for Cortex-A57 AES Erratum 1742098";
  }
```
- EN: Implements `getPassName`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getPassName`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 83-87
```cpp
  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.addRequired<ReachingDefInfoWrapperPass>();
    AU.setPreservesCFG();
    MachineFunctionPass::getAnalysisUsage(AU);
  }
```
- EN: Implements `getAnalysisUsage`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getAnalysisUsage`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 89-96
```cpp
private:
  // This is the information needed to insert the fixup in the right place.
  struct AESFixupLocation {
    MachineBasicBlock *Block;
    // The fixup instruction will be inserted *before* InsertionPt.
    MachineInstr *InsertionPt;
    MachineOperand *MOp;
  };
```
- EN: Declares `AESFixupLocation`, packaging target-specific state and APIs around `ARMFixCortexA57AES1742098Pass`.
- CN: 这里声明 `AESFixupLocation`，把与 `ARMFixCortexA57AES1742098Pass` 相关的目标特定状态和 API 组织在一起。

### Lines 98-100
```cpp
  void analyzeMF(MachineFunction &MF, ReachingDefInfo &RDI,
                 const ARMBaseRegisterInfo *TRI,
                 SmallVectorImpl<AESFixupLocation> &FixupLocsForFn) const;
```
- EN: Declares `analyzeMF`, a analysis routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `analyzeMF`，它是一个围绕机器函数状态展开的分析例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 102-103
```cpp
  void insertAESFixup(AESFixupLocation &FixupLoc, const ARMBaseInstrInfo *TII,
                      const ARMBaseRegisterInfo *TRI) const;
```
- EN: Declares `insertAESFixup`, a mutation/build routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `insertAESFixup`，它是一个围绕寄存器管理展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 105-108
```cpp
  static bool isFirstAESPairInstr(MachineInstr &MI);
  static bool isSafeAESInput(MachineInstr &MI);
};
char ARMFixCortexA57AES1742098::ID = 0;
```
- EN: Declares `isFirstAESPairInstr`, a query/helper routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isFirstAESPairInstr`，它是一个围绕机器指令展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 110-110
```cpp
} // end anonymous namespace
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 112-117
```cpp
INITIALIZE_PASS_BEGIN(ARMFixCortexA57AES1742098, DEBUG_TYPE,
                      "ARM fix for Cortex-A57 AES Erratum 1742098", false,
                      false)
INITIALIZE_PASS_DEPENDENCY(ReachingDefInfoWrapperPass);
INITIALIZE_PASS_END(ARMFixCortexA57AES1742098, DEBUG_TYPE,
                    "ARM fix for Cortex-A57 AES Erratum 1742098", false, false)
```
- EN: Declares `INITIALIZE_PASS_BEGIN`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `INITIALIZE_PASS_BEGIN`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 119-119
```cpp
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 121-124
```cpp
bool ARMFixCortexA57AES1742098::isFirstAESPairInstr(MachineInstr &MI) {
  unsigned Opc = MI.getOpcode();
  return Opc == ARM::AESD || Opc == ARM::AESE;
}
```
- EN: Implements `ARMFixCortexA57AES1742098::isFirstAESPairInstr`, a query/helper routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMFixCortexA57AES1742098::isFirstAESPairInstr`，它是一个围绕机器指令展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 126-132
```cpp
bool ARMFixCortexA57AES1742098::isSafeAESInput(MachineInstr &MI) {
  auto CondCodeIsAL = [](MachineInstr &MI) -> bool {
    int CCIdx = MI.findFirstPredOperandIdx();
    if (CCIdx == -1)
      return false;
    return MI.getOperand(CCIdx).getImm() == (int64_t)ARMCC::AL;
  };
```
- EN: Implements `ARMFixCortexA57AES1742098::isSafeAESInput`, a query/helper routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMFixCortexA57AES1742098::isSafeAESInput`，它是一个围绕机器指令展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 134-148
```cpp
  switch (MI.getOpcode()) {
  // Unknown: Assume not safe.
  default:
    return false;
  // 128-bit wide AES instructions
  case ARM::AESD:
  case ARM::AESE:
  case ARM::AESMC:
  case ARM::AESIMC:
    // No CondCode.
    return true;
  // 128-bit and 64-bit wide bitwise ops (when condition = al)
  case ARM::VANDd:
  case ARM::VANDq:
  case ARM::VORRd:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 149-163
```cpp
  case ARM::VORRq:
  case ARM::VEORd:
  case ARM::VEORq:
  case ARM::VMVNd:
  case ARM::VMVNq:
  // VMOV of 64-bit value between D registers (when condition = al)
  case ARM::VMOVD:
  // VMOV of 64 bit value from GPRs (when condition = al)
  case ARM::VMOVDRR:
  // VMOV of immediate into D or Q registers (when condition = al)
  case ARM::VMOVv2i64:
  case ARM::VMOVv1i64:
  case ARM::VMOVv2f32:
  case ARM::VMOVv4f32:
  case ARM::VMOVv2i32:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 164-178
```cpp
  case ARM::VMOVv4i32:
  case ARM::VMOVv4i16:
  case ARM::VMOVv8i16:
  case ARM::VMOVv8i8:
  case ARM::VMOVv16i8:
  // Loads (when condition = al)
  // VLD Dn, [Rn, #imm]
  case ARM::VLDRD:
  // VLDM
  case ARM::VLDMDDB_UPD:
  case ARM::VLDMDIA_UPD:
  case ARM::VLDMDIA:
  // VLDn to all lanes.
  case ARM::VLD1d64:
  case ARM::VLD1q64:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 179-193
```cpp
  case ARM::VLD1d32:
  case ARM::VLD1q32:
  case ARM::VLD2b32:
  case ARM::VLD2d32:
  case ARM::VLD2q32:
  case ARM::VLD1d16:
  case ARM::VLD1q16:
  case ARM::VLD2d16:
  case ARM::VLD2q16:
  case ARM::VLD1d8:
  case ARM::VLD1q8:
  case ARM::VLD2b8:
  case ARM::VLD2d8:
  case ARM::VLD2q8:
  case ARM::VLD3d32:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 194-208
```cpp
  case ARM::VLD3q32:
  case ARM::VLD3d16:
  case ARM::VLD3q16:
  case ARM::VLD3d8:
  case ARM::VLD3q8:
  case ARM::VLD4d32:
  case ARM::VLD4q32:
  case ARM::VLD4d16:
  case ARM::VLD4q16:
  case ARM::VLD4d8:
  case ARM::VLD4q8:
  // VLD1 (single element to one lane)
  case ARM::VLD1LNd32:
  case ARM::VLD1LNd32_UPD:
  case ARM::VLD1LNd8:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 209-223
```cpp
  case ARM::VLD1LNd8_UPD:
  case ARM::VLD1LNd16:
  case ARM::VLD1LNd16_UPD:
  // VLD1 (single element to all lanes)
  case ARM::VLD1DUPd32:
  case ARM::VLD1DUPd32wb_fixed:
  case ARM::VLD1DUPd32wb_register:
  case ARM::VLD1DUPd16:
  case ARM::VLD1DUPd16wb_fixed:
  case ARM::VLD1DUPd16wb_register:
  case ARM::VLD1DUPd8:
  case ARM::VLD1DUPd8wb_fixed:
  case ARM::VLD1DUPd8wb_register:
  case ARM::VLD1DUPq32:
  case ARM::VLD1DUPq32wb_fixed:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 224-236
```cpp
  case ARM::VLD1DUPq32wb_register:
  case ARM::VLD1DUPq16:
  case ARM::VLD1DUPq16wb_fixed:
  case ARM::VLD1DUPq16wb_register:
  case ARM::VLD1DUPq8:
  case ARM::VLD1DUPq8wb_fixed:
  case ARM::VLD1DUPq8wb_register:
  // VMOV
  case ARM::VSETLNi32:
  case ARM::VSETLNi16:
  case ARM::VSETLNi8:
    return CondCodeIsAL(MI);
  };
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 238-239
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 241-243
```cpp
bool ARMFixCortexA57AES1742098::runOnMachineFunction(MachineFunction &F) {
  LLVM_DEBUG(dbgs() << "***** ARMFixCortexA57AES1742098 *****\n");
  auto &STI = F.getSubtarget<ARMSubtarget>();
```
- EN: Implements `ARMFixCortexA57AES1742098::runOnMachineFunction`, a pass-entry routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMFixCortexA57AES1742098::runOnMachineFunction`，它是一个围绕机器函数状态展开的Pass 入口例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 245-247
```cpp
  // Fix not requested or AES instructions not present: skip pass.
  if (!STI.hasAES() || !STI.fixCortexA57AES1742098())
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 249-250
```cpp
  const ARMBaseRegisterInfo *TRI = STI.getRegisterInfo();
  const ARMBaseInstrInfo *TII = STI.getInstrInfo();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 252-252
```cpp
  auto &RDI = getAnalysis<ReachingDefInfoWrapperPass>().getRDI();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 254-256
```cpp
  // Analyze whole function to find instructions which need fixing up...
  SmallVector<AESFixupLocation> FixupLocsForFn{};
  analyzeMF(F, RDI, TRI, FixupLocsForFn);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 258-264
```cpp
  // ... and fix the instructions up all at the same time.
  bool Changed = false;
  LLVM_DEBUG(dbgs() << "Inserting " << FixupLocsForFn.size() << " fixup(s)\n");
  for (AESFixupLocation &FixupLoc : FixupLocsForFn) {
    insertAESFixup(FixupLoc, TII, TRI);
    Changed |= true;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 266-267
```cpp
  return Changed;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 269-272
```cpp
void ARMFixCortexA57AES1742098::analyzeMF(
    MachineFunction &MF, ReachingDefInfo &RDI, const ARMBaseRegisterInfo *TRI,
    SmallVectorImpl<AESFixupLocation> &FixupLocsForFn) const {
  unsigned MaxAllowedFixups = 0;
```
- EN: Implements `ARMFixCortexA57AES1742098::analyzeMF`, a analysis routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMFixCortexA57AES1742098::analyzeMF`，它是一个围绕机器函数状态展开的分析例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 274-277
```cpp
  for (MachineBasicBlock &MBB : MF) {
    for (MachineInstr &MI : MBB) {
      if (!isFirstAESPairInstr(MI))
        continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 279-282
```cpp
      // Found an instruction to check the operands of.
      LLVM_DEBUG(dbgs() << "Found AES Pair starting: " << MI);
      assert(MI.getNumExplicitOperands() == 3 && MI.getNumExplicitDefs() == 1 &&
             "Unknown AES Instruction Format. Expected 1 def, 2 uses.");
```
- EN: Declares `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 284-286
```cpp
      // A maximum of two fixups should be inserted for each AES pair (one per
      // register use).
      MaxAllowedFixups += 2;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 288-291
```cpp
      // Inspect all operands, choosing whether to insert a fixup.
      for (MachineOperand &MOp : MI.uses()) {
        SmallPtrSet<MachineInstr *, 1> AllDefs{};
        RDI.getGlobalReachingDefs(&MI, MOp.getReg(), AllDefs);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 293-294
```cpp
        // Planned Fixup: This should be added to FixupLocsForFn at most once.
        AESFixupLocation NewLoc{&MBB, &MI, &MOp};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 296-298
```cpp
        // In small functions with loops, this operand may be both a live-in and
        // have definitions within the function itself. These will need a fixup.
        bool IsLiveIn = MF.front().isLiveIn(MOp.getReg());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 300-309
```cpp
        // If the register doesn't have defining instructions, and is not a
        // live-in, then something is wrong and the fixup must always be
        // inserted to be safe.
        if (!IsLiveIn && AllDefs.size() == 0) {
          LLVM_DEBUG(dbgs()
                     << "Fixup Planned: No Defining Instrs found, not live-in: "
                     << printReg(MOp.getReg(), TRI) << "\n");
          FixupLocsForFn.emplace_back(NewLoc);
          continue;
        }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 311-314
```cpp
        auto IsUnsafe = [](MachineInstr *MI) -> bool {
          return !isSafeAESInput(*MI);
        };
        size_t UnsafeCount = llvm::count_if(AllDefs, IsUnsafe);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 316-324
```cpp
        // If there are no unsafe definitions...
        if (UnsafeCount == 0) {
          // ... and the register is not live-in ...
          if (!IsLiveIn) {
            // ... then skip the fixup.
            LLVM_DEBUG(dbgs() << "No Fixup: Defining instrs are all safe: "
                              << printReg(MOp.getReg(), TRI) << "\n");
            continue;
          }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 326-337
```cpp
          // Otherwise, the only unsafe "definition" is a live-in, so insert the
          // fixup at the start of the function.
          LLVM_DEBUG(dbgs()
                     << "Fixup Planned: Live-In (with safe defining instrs): "
                     << printReg(MOp.getReg(), TRI) << "\n");
          NewLoc.Block = &MF.front();
          NewLoc.InsertionPt = &*NewLoc.Block->begin();
          LLVM_DEBUG(dbgs() << "Moving Fixup for Live-In to immediately before "
                            << *NewLoc.InsertionPt);
          FixupLocsForFn.emplace_back(NewLoc);
          continue;
        }
```
- EN: Declares `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 339-351
```cpp
        // If a fixup is needed in more than one place, then the best place to
        // insert it is adjacent to the use rather than introducing a fixup
        // adjacent to each def.
        //
        // FIXME: It might be better to hoist this to the start of the BB, if
        // possible.
        if (IsLiveIn || UnsafeCount > 1) {
          LLVM_DEBUG(dbgs() << "Fixup Planned: Multiple unsafe defining instrs "
                               "(including live-ins): "
                            << printReg(MOp.getReg(), TRI) << "\n");
          FixupLocsForFn.emplace_back(NewLoc);
          continue;
        }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 353-360
```cpp
        assert(UnsafeCount == 1 && !IsLiveIn &&
               "At this point, there should be one unsafe defining instrs "
               "and the defined register should not be a live-in.");
        SmallPtrSetIterator<MachineInstr *> It =
            llvm::find_if(AllDefs, IsUnsafe);
        assert(It != AllDefs.end() &&
               "UnsafeCount == 1 but No Unsafe MachineInstr found.");
        MachineInstr *DefMI = *It;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 362-364
```cpp
        LLVM_DEBUG(
            dbgs() << "Fixup Planned: Found single unsafe defining instrs for "
                   << printReg(MOp.getReg(), TRI) << ": " << *DefMI);
```
- EN: Declares `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 366-378
```cpp
        // There is one unsafe defining instruction, which needs a fixup. It is
        // generally good to hoist the fixup to be adjacent to the defining
        // instruction rather than the using instruction, as the using
        // instruction may be inside a loop when the defining instruction is
        // not.
        MachineBasicBlock::iterator DefIt = DefMI;
        ++DefIt;
        if (DefIt != DefMI->getParent()->end()) {
          LLVM_DEBUG(dbgs() << "Moving Fixup to immediately after " << *DefMI
                            << "And immediately before " << *DefIt);
          NewLoc.Block = DefIt->getParent();
          NewLoc.InsertionPt = &*DefIt;
        }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 380-383
```cpp
        FixupLocsForFn.emplace_back(NewLoc);
      }
    }
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 385-388
```cpp
  assert(FixupLocsForFn.size() <= MaxAllowedFixups &&
         "Inserted too many fixups for this function.");
  (void)MaxAllowedFixups;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 390-393
```cpp
void ARMFixCortexA57AES1742098::insertAESFixup(
    AESFixupLocation &FixupLoc, const ARMBaseInstrInfo *TII,
    const ARMBaseRegisterInfo *TRI) const {
  MachineOperand *OperandToFixup = FixupLoc.MOp;
```
- EN: Implements `ARMFixCortexA57AES1742098::insertAESFixup`, a mutation/build routine centered on machine operands. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMFixCortexA57AES1742098::insertAESFixup`，它是一个围绕机器操作数展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 395-396
```cpp
  assert(OperandToFixup->isReg() && "OperandToFixup must be a register");
  Register RegToFixup = OperandToFixup->getReg();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 398-399
```cpp
  LLVM_DEBUG(dbgs() << "Inserting VORRq of " << printReg(RegToFixup, TRI)
                    << " before: " << *FixupLoc.InsertionPt);
```
- EN: Declares `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 401-415
```cpp
  // Insert the new `VORRq qN, qN, qN`. There are a few details here:
  //
  // The uses are marked as killed, even if the original use of OperandToFixup
  // is not killed, as the new instruction is clobbering the register. This is
  // safe even if there are other uses of `qN`, as the VORRq value-wise a no-op
  // (it is inserted for microarchitectural reasons).
  //
  // The def and the uses are still marked as Renamable if the original register
  // was, to avoid having to rummage through all the other uses and defs and
  // unset their renamable bits.
  RegState Renamable = getRenamableRegState(OperandToFixup->isRenamable());
  BuildMI(*FixupLoc.Block, FixupLoc.InsertionPt, DebugLoc(),
          TII->get(ARM::VORRq))
      .addReg(RegToFixup, RegState::Define | Renamable)
      .addReg(RegToFixup, RegState::Kill | Renamable)
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 416-419
```cpp
      .addReg(RegToFixup, RegState::Kill | Renamable)
      .addImm((uint64_t)ARMCC::AL)
      .addReg(ARM::NoRegister);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 421-425
```cpp
// Factory function used by AArch64TargetMachine to add the pass to
// the passmanager.
FunctionPass *llvm::createARMFixCortexA57AES1742098Pass() {
  return new ARMFixCortexA57AES1742098();
}
```
- EN: Implements `llvm::createARMFixCortexA57AES1742098Pass`, a query/helper routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `llvm::createARMFixCortexA57AES1742098Pass`，它是一个围绕目标机器策略展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

## Key Concepts / 关键概念

- EN: Primary role: target-specific code-generation behavior.
  - CN: 核心职责：目标相关代码生成行为。
- EN: MachineInstr-level manipulation is central to this file.
  - CN: 该文件以 MachineInstr 层面的操作为核心。
- EN: MachineBasicBlock flow and branch structure matter to the implementation.
  - CN: 实现高度依赖 MachineBasicBlock 流程与分支结构。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。
- EN: The component plugs into LLVM machine-function passes or codegen pipelines.
  - CN: 该组件会接入 LLVM 的机器函数 Pass 或代码生成流水线。

## Dependencies / 依赖关系

- EN: Backend-local headers: `ARM.h`, `ARMBaseInstrInfo.h`, `ARMBaseRegisterInfo.h`, `ARMSubtarget.h`, `Utils/ARMBaseInfo.h`, `assert.h`, `stdint.h`.
  - CN: 后端本地头文件：`ARM.h`, `ARMBaseInstrInfo.h`, `ARMBaseRegisterInfo.h`, `ARMSubtarget.h`, `Utils/ARMBaseInfo.h`, `assert.h`, `stdint.h`。
- EN: LLVM infrastructure headers: `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallPtrSet.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/StringRef.h`, `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineFunctionPass.h`, `llvm/CodeGen/MachineInstr.h` ... (+10 more).
  - CN: LLVM 基础设施头文件：`llvm/ADT/STLExtras.h`, `llvm/ADT/SmallPtrSet.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/StringRef.h`, `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineFunctionPass.h`, `llvm/CodeGen/MachineInstr.h` ... (+10 more)。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
