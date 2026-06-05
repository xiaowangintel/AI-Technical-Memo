# ARMMachineFunctionInfo.h — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMMachineFunctionInfo.h`
- Repository: `llvm-project`
- Purpose (EN): This file declares ARM-specific per-machine-function information.
- 用途 (CN): 声明 ARM 后端中的 `ARMMachineFunctionInfo`，并提供与机器函数特定状态相关的接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===-- ARMMachineFunctionInfo.h - ARM machine function info ----*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file declares ARM-specific per-machine-function information.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 13-14
```cpp
#ifndef LLVM_LIB_TARGET_ARM_ARMMACHINEFUNCTIONINFO_H
#define LLVM_LIB_TARGET_ARM_ARMMACHINEFUNCTIONINFO_H
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 16-21
```cpp
#include "llvm/ADT/DenseMap.h"
#include "llvm/ADT/SmallPtrSet.h"
#include "llvm/CodeGen/MIRYamlMapping.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/IR/GlobalVariable.h"
#include "llvm/Support/ErrorHandling.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 23-23
```cpp
namespace llvm {
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 25-27
```cpp
namespace yaml {
struct ARMFunctionInfo;
} // end namespace yaml
```
- EN: Declares `ARMFunctionInfo`, packaging target-specific state and APIs around `ARMMachineFunctionInfo`.
- CN: 这里声明 `ARMFunctionInfo`，把与 `ARMMachineFunctionInfo` 相关的目标特定状态和 API 组织在一起。

### Lines 29-29
```cpp
class ARMSubtarget;
```
- EN: Declares `ARMSubtarget`, packaging target-specific state and APIs around `ARMMachineFunctionInfo`.
- CN: 这里声明 `ARMSubtarget`，把与 `ARMMachineFunctionInfo` 相关的目标特定状态和 API 组织在一起。

### Lines 31-34
```cpp
/// ARMFunctionInfo - This class is derived from MachineFunctionInfo and
/// contains private ARM-specific information for each MachineFunction.
class ARMFunctionInfo : public MachineFunctionInfo {
  virtual void anchor();
```
- EN: Declares `is`, packaging target-specific state and APIs around `ARMMachineFunctionInfo`.
- CN: 这里声明 `is`，把与 `ARMMachineFunctionInfo` 相关的目标特定状态和 API 组织在一起。

### Lines 36-38
```cpp
  /// isThumb - True if this function is compiled under Thumb mode.
  /// Used to initialized Align, so must precede it.
  bool isThumb = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 40-43
```cpp
  /// hasThumb2 - True if the target architecture supports Thumb2. Do not use
  /// to determine if function is compiled under Thumb mode, for that use
  /// 'isThumb'.
  bool hasThumb2 = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 45-49
```cpp
  /// ArgsRegSaveSize - Size of the register save area for vararg functions or
  /// those making guaranteed tail calls that need more stack argument space
  /// than is provided by this functions incoming parameters.
  ///
  unsigned ArgRegsSaveSize = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 51-52
```cpp
  /// ReturnRegsCount - Number of registers used up in the return.
  unsigned ReturnRegsCount = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 54-56
```cpp
  /// HasStackFrame - True if this function has a stack frame. Set by
  /// determineCalleeSaves().
  bool HasStackFrame = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 58-60
```cpp
  /// RestoreSPFromFP - True if epilogue should restore SP from FP. Set by
  /// emitPrologue.
  bool RestoreSPFromFP = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 62-64
```cpp
  /// LRSpilled - True if the LR register has been for spilled for
  /// any reason, so it's legal to emit an ARM::tBfar (i.e. "bl").
  bool LRSpilled = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 66-68
```cpp
  /// FramePtrSpillOffset - If HasStackFrame, this records the frame pointer
  /// spill stack offset.
  unsigned FramePtrSpillOffset = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 70-83
```cpp
  /// GPRCS1Offset, GPRCS2Offset, DPRCSOffset - Starting offset of callee saved
  /// register spills areas. For Mac OS X:
  ///
  /// GPR callee-saved (1) : r4, r5, r6, r7, lr
  /// --------------------------------------------
  /// GPR callee-saved (2) : r8, r10, r11
  /// --------------------------------------------
  /// DPR callee-saved : d8 - d15
  ///
  /// Also see AlignedDPRCSRegs below. Not all D-regs need to go in area 3.
  /// Some may be spilled after the stack has been realigned.
  unsigned GPRCS1Offset = 0;
  unsigned GPRCS2Offset = 0;
  unsigned DPRCS1Offset = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 85-94
```cpp
  /// GPRCS1Size, GPRCS2Size, DPRCSSize - Sizes of callee saved register spills
  /// areas.
  unsigned FPCXTSaveSize = 0;
  unsigned FRSaveSize = 0;
  unsigned GPRCS1Size = 0;
  unsigned GPRCS2Size = 0;
  unsigned FPStatusSize = 0;
  unsigned DPRCSAlignGapSize = 0;
  unsigned DPRCS1Size = 0;
  unsigned GPRCS3Size = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 96-103
```cpp
  /// NumAlignedDPRCS2Regs - The number of callee-saved DPRs that are saved in
  /// the aligned portion of the stack frame.  This is always a contiguous
  /// sequence of D-registers starting from d8.
  ///
  /// We do not keep track of the frame indices used for these registers - they
  /// behave like any other frame index in the aligned stack frame.  These
  /// registers also aren't included in DPRCSSize above.
  unsigned NumAlignedDPRCS2Regs = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 105-105
```cpp
  unsigned PICLabelUId = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 107-108
```cpp
  /// VarArgsFrameIndex - FrameIndex for start of varargs area.
  int VarArgsFrameIndex = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 110-111
```cpp
  /// HasITBlocks - True if IT blocks have been inserted.
  bool HasITBlocks = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 113-115
```cpp
  // Security Extensions
  bool IsCmseNSEntry;
  bool IsCmseNSCall;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 117-119
```cpp
  /// CPEClones - Track constant pool entries clones created by Constant Island
  /// pass.
  DenseMap<unsigned, unsigned> CPEClones;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 121-123
```cpp
  /// ArgumentStackSize - amount of bytes on stack consumed by the arguments
  /// being passed on the stack
  unsigned ArgumentStackSize = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 125-127
```cpp
  /// ArgumentStackToRestore - amount of bytes on stack consumed that we must
  /// restore on return.
  unsigned ArgumentStackToRestore = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 129-131
```cpp
  /// CoalescedWeights - mapping of basic blocks to the rolling counter of
  /// coalesced weights.
  DenseMap<const MachineBasicBlock*, unsigned> CoalescedWeights;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 133-135
```cpp
  /// True if this function has a subset of CSRs that is handled explicitly via
  /// copies.
  bool IsSplitCSR = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 137-138
```cpp
  /// Globals that have had their storage promoted into the constant pool.
  SmallPtrSet<const GlobalVariable*,2> PromotedGlobals;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 140-141
```cpp
  /// The amount the literal pool has been increasedby due to promoted globals.
  int PromotedGlobalsIncrease = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 143-145
```cpp
  /// True if r0 will be preserved by a call to this function (e.g. C++
  /// con/destructors).
  bool PreservesR0 = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 147-148
```cpp
  /// True if the function should sign its return address.
  bool SignReturnAddress = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 150-152
```cpp
  /// True if the fucntion should sign its return address, even if LR is not
  /// saved.
  bool SignReturnAddressAll = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 154-156
```cpp
  /// True if BTI instructions should be placed at potential indirect jump
  /// destinations.
  bool BranchTargetEnforcement = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 158-159
```cpp
public:
  ARMFunctionInfo() = default;
```
- EN: Declares `ARMFunctionInfo`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ARMFunctionInfo`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 161-161
```cpp
  explicit ARMFunctionInfo(const Function &F, const ARMSubtarget *STI);
```
- EN: Declares `ARMFunctionInfo`, a target-specific routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ARMFunctionInfo`，它是一个围绕子目标特性处理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 163-166
```cpp
  MachineFunctionInfo *
  clone(BumpPtrAllocator &Allocator, MachineFunction &DestMF,
        const DenseMap<MachineBasicBlock *, MachineBasicBlock *> &Src2DstMBB)
      const override;
```
- EN: Declares `clone`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `clone`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 168-170
```cpp
  bool isThumbFunction() const { return isThumb; }
  bool isThumb1OnlyFunction() const { return isThumb && !hasThumb2; }
  bool isThumb2Function() const { return isThumb && hasThumb2; }
```
- EN: Implements `isThumbFunction`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isThumbFunction`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 172-173
```cpp
  bool isCmseNSEntryFunction() const { return IsCmseNSEntry; }
  bool isCmseNSCallFunction() const { return IsCmseNSCall; }
```
- EN: Implements `isCmseNSEntryFunction`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isCmseNSEntryFunction`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 175-176
```cpp
  unsigned getArgRegsSaveSize() const { return ArgRegsSaveSize; }
  void setArgRegsSaveSize(unsigned s) { ArgRegsSaveSize = s; }
```
- EN: Implements `getArgRegsSaveSize`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getArgRegsSaveSize`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 178-179
```cpp
  unsigned getReturnRegsCount() const { return ReturnRegsCount; }
  void setReturnRegsCount(unsigned s) { ReturnRegsCount = s; }
```
- EN: Implements `getReturnRegsCount`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getReturnRegsCount`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 181-182
```cpp
  bool hasStackFrame() const { return HasStackFrame; }
  void setHasStackFrame(bool s) { HasStackFrame = s; }
```
- EN: Implements `hasStackFrame`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasStackFrame`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 184-185
```cpp
  bool shouldRestoreSPFromFP() const { return RestoreSPFromFP; }
  void setShouldRestoreSPFromFP(bool s) { RestoreSPFromFP = s; }
```
- EN: Implements `shouldRestoreSPFromFP`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `shouldRestoreSPFromFP`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 187-188
```cpp
  bool isLRSpilled() const { return LRSpilled; }
  void setLRIsSpilled(bool s) { LRSpilled = s; }
```
- EN: Implements `isLRSpilled`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isLRSpilled`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 190-191
```cpp
  unsigned getFramePtrSpillOffset() const { return FramePtrSpillOffset; }
  void setFramePtrSpillOffset(unsigned o) { FramePtrSpillOffset = o; }
```
- EN: Implements `getFramePtrSpillOffset`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getFramePtrSpillOffset`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 193-194
```cpp
  unsigned getNumAlignedDPRCS2Regs() const { return NumAlignedDPRCS2Regs; }
  void setNumAlignedDPRCS2Regs(unsigned n) { NumAlignedDPRCS2Regs = n; }
```
- EN: Implements `getNumAlignedDPRCS2Regs`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getNumAlignedDPRCS2Regs`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 196-198
```cpp
  unsigned getGPRCalleeSavedArea1Offset() const { return GPRCS1Offset; }
  unsigned getGPRCalleeSavedArea2Offset() const { return GPRCS2Offset; }
  unsigned getDPRCalleeSavedArea1Offset() const { return DPRCS1Offset; }
```
- EN: Implements `getGPRCalleeSavedArea1Offset`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getGPRCalleeSavedArea1Offset`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 200-202
```cpp
  void setGPRCalleeSavedArea1Offset(unsigned o) { GPRCS1Offset = o; }
  void setGPRCalleeSavedArea2Offset(unsigned o) { GPRCS2Offset = o; }
  void setDPRCalleeSavedArea1Offset(unsigned o) { DPRCS1Offset = o; }
```
- EN: Implements `setGPRCalleeSavedArea1Offset`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `setGPRCalleeSavedArea1Offset`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 204-211
```cpp
  unsigned getFPCXTSaveAreaSize() const       { return FPCXTSaveSize; }
  unsigned getFrameRecordSavedAreaSize() const { return FRSaveSize; }
  unsigned getGPRCalleeSavedArea1Size() const { return GPRCS1Size; }
  unsigned getGPRCalleeSavedArea2Size() const { return GPRCS2Size; }
  unsigned getFPStatusSavesSize() const       { return FPStatusSize; }
  unsigned getDPRCalleeSavedGapSize() const   { return DPRCSAlignGapSize; }
  unsigned getDPRCalleeSavedArea1Size() const { return DPRCS1Size; }
  unsigned getGPRCalleeSavedArea3Size() const { return GPRCS3Size; }
```
- EN: Implements `getFPCXTSaveAreaSize`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getFPCXTSaveAreaSize`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 213-220
```cpp
  void setFPCXTSaveAreaSize(unsigned s)       { FPCXTSaveSize = s; }
  void setFrameRecordSavedAreaSize(unsigned s) { FRSaveSize = s; }
  void setGPRCalleeSavedArea1Size(unsigned s) { GPRCS1Size = s; }
  void setGPRCalleeSavedArea2Size(unsigned s) { GPRCS2Size = s; }
  void setFPStatusSavesSize(unsigned s)       { FPStatusSize = s; }
  void setDPRCalleeSavedGapSize(unsigned s)   { DPRCSAlignGapSize = s; }
  void setDPRCalleeSavedArea1Size(unsigned s) { DPRCS1Size = s; }
  void setGPRCalleeSavedArea3Size(unsigned s) { GPRCS3Size = s; }
```
- EN: Implements `setFPCXTSaveAreaSize`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `setFPCXTSaveAreaSize`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 222-223
```cpp
  unsigned getArgumentStackSize() const { return ArgumentStackSize; }
  void setArgumentStackSize(unsigned size) { ArgumentStackSize = size; }
```
- EN: Implements `getArgumentStackSize`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getArgumentStackSize`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 225-226
```cpp
  unsigned getArgumentStackToRestore() const { return ArgumentStackToRestore; }
  void setArgumentStackToRestore(unsigned v) { ArgumentStackToRestore = v; }
```
- EN: Implements `getArgumentStackToRestore`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getArgumentStackToRestore`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 228-230
```cpp
  void initPICLabelUId(unsigned UId) {
    PICLabelUId = UId;
  }
```
- EN: Implements `initPICLabelUId`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `initPICLabelUId`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 232-234
```cpp
  unsigned getNumPICLabels() const {
    return PICLabelUId;
  }
```
- EN: Implements `getNumPICLabels`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getNumPICLabels`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 236-238
```cpp
  unsigned createPICLabelUId() {
    return PICLabelUId++;
  }
```
- EN: Implements `createPICLabelUId`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `createPICLabelUId`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 240-241
```cpp
  int getVarArgsFrameIndex() const { return VarArgsFrameIndex; }
  void setVarArgsFrameIndex(int Index) { VarArgsFrameIndex = Index; }
```
- EN: Implements `getVarArgsFrameIndex`, a query/helper routine centered on stack-frame access. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getVarArgsFrameIndex`，它是一个围绕栈帧访问展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 243-244
```cpp
  bool hasITBlocks() const { return HasITBlocks; }
  void setHasITBlocks(bool h) { HasITBlocks = h; }
```
- EN: Implements `hasITBlocks`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasITBlocks`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 246-247
```cpp
  bool isSplitCSR() const { return IsSplitCSR; }
  void setIsSplitCSR(bool s) { IsSplitCSR = s; }
```
- EN: Implements `isSplitCSR`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isSplitCSR`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 249-252
```cpp
  void recordCPEClone(unsigned CPIdx, unsigned CPCloneIdx) {
    if (!CPEClones.insert(std::make_pair(CPCloneIdx, CPIdx)).second)
      llvm_unreachable("Duplicate entries!");
  }
```
- EN: Implements `recordCPEClone`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `recordCPEClone`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 254-260
```cpp
  unsigned getOriginalCPIdx(unsigned CloneIdx) const {
    auto I = CPEClones.find(CloneIdx);
    if (I != CPEClones.end())
      return I->second;
    else
      return -1U;
  }
```
- EN: Implements `getOriginalCPIdx`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getOriginalCPIdx`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 262-265
```cpp
  DenseMap<const MachineBasicBlock *, unsigned>::iterator
  getCoalescedWeight(MachineBasicBlock *MBB) {
    return CoalescedWeights.try_emplace(MBB, 0).first;
  }
```
- EN: Implements `getCoalescedWeight`, a query/helper routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getCoalescedWeight`，它是一个围绕机器基本块展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 267-281
```cpp
  /// Indicate to the backend that \c GV has had its storage changed to inside
  /// a constant pool. This means it no longer needs to be emitted as a
  /// global variable.
  void markGlobalAsPromotedToConstantPool(const GlobalVariable *GV) {
    PromotedGlobals.insert(GV);
  }
  SmallPtrSet<const GlobalVariable*, 2>& getGlobalsPromotedToConstantPool() {
    return PromotedGlobals;
  }
  int getPromotedConstpoolIncrease() const {
    return PromotedGlobalsIncrease;
  }
  void setPromotedConstpoolIncrease(int Sz) {
    PromotedGlobalsIncrease = Sz;
  }
```
- EN: Implements `markGlobalAsPromotedToConstantPool`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `markGlobalAsPromotedToConstantPool`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 283-284
```cpp
  DenseMap<unsigned, unsigned> EHPrologueRemappedRegs;
  DenseMap<unsigned, unsigned> EHPrologueOffsetInRegs;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 286-287
```cpp
  void setPreservesR0() { PreservesR0 = true; }
  bool getPreservesR0() const { return PreservesR0; }
```
- EN: Implements `setPreservesR0`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `setPreservesR0`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 289-291
```cpp
  bool shouldSignReturnAddress() const {
    return shouldSignReturnAddress(LRSpilled);
  }
```
- EN: Implements `shouldSignReturnAddress`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `shouldSignReturnAddress`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 293-299
```cpp
  bool shouldSignReturnAddress(bool SpillsLR) const {
    if (!SignReturnAddress)
      return false;
    if (SignReturnAddressAll)
      return true;
    return SpillsLR;
  }
```
- EN: Implements `shouldSignReturnAddress`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `shouldSignReturnAddress`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 301-301
```cpp
  bool branchTargetEnforcement() const { return BranchTargetEnforcement; }
```
- EN: Implements `branchTargetEnforcement`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `branchTargetEnforcement`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 303-304
```cpp
  void initializeBaseYamlFields(const yaml::ARMFunctionInfo &YamlMFI);
};
```
- EN: Declares `initializeBaseYamlFields`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `initializeBaseYamlFields`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 306-308
```cpp
namespace yaml {
struct ARMFunctionInfo final : public yaml::MachineFunctionInfo {
  bool LRSpilled;
```
- EN: Declares `ARMFunctionInfo`, packaging target-specific state and APIs around `ARMMachineFunctionInfo`.
- CN: 这里声明 `ARMFunctionInfo`，把与 `ARMMachineFunctionInfo` 相关的目标特定状态和 API 组织在一起。

### Lines 310-311
```cpp
  ARMFunctionInfo() = default;
  ARMFunctionInfo(const llvm::ARMFunctionInfo &MFI);
```
- EN: Declares `ARMFunctionInfo`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ARMFunctionInfo`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 313-315
```cpp
  void mappingImpl(yaml::IO &YamlIO) override;
  ~ARMFunctionInfo() override = default;
};
```
- EN: Declares `mappingImpl`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `mappingImpl`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 317-321
```cpp
template <> struct MappingTraits<ARMFunctionInfo> {
  static void mapping(IO &YamlIO, ARMFunctionInfo &MFI) {
    YamlIO.mapOptional("isLRSpilled", MFI.LRSpilled);
  }
};
```
- EN: Declares `MappingTraits`, packaging target-specific state and APIs around `ARMMachineFunctionInfo`.
- CN: 这里声明 `MappingTraits`，把与 `ARMMachineFunctionInfo` 相关的目标特定状态和 API 组织在一起。

### Lines 323-323
```cpp
} // end namespace yaml
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 325-325
```cpp
} // end namespace llvm
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 327-327
```cpp
#endif // LLVM_LIB_TARGET_ARM_ARMMACHINEFUNCTIONINFO_H
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

## Key Concepts / 关键概念

- EN: Primary role: machine-function specific state.
  - CN: 核心职责：机器函数特定状态。
- EN: MachineBasicBlock flow and branch structure matter to the implementation.
  - CN: 实现高度依赖 MachineBasicBlock 流程与分支结构。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。
- EN: Stack-frame and spill/reload concerns are reflected in the API or implementation.
  - CN: API 或实现中体现了栈帧以及溢出/重载相关问题。

## Dependencies / 依赖关系

- EN: Backend-local headers: `(none)`.
  - CN: 后端本地头文件：`(none)`。
- EN: LLVM infrastructure headers: `llvm/ADT/DenseMap.h`, `llvm/ADT/SmallPtrSet.h`, `llvm/CodeGen/MIRYamlMapping.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/IR/GlobalVariable.h`, `llvm/Support/ErrorHandling.h`.
  - CN: LLVM 基础设施头文件：`llvm/ADT/DenseMap.h`, `llvm/ADT/SmallPtrSet.h`, `llvm/CodeGen/MIRYamlMapping.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/IR/GlobalVariable.h`, `llvm/Support/ErrorHandling.h`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
