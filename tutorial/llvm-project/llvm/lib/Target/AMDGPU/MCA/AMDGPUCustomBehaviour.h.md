# AMDGPUCustomBehaviour.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/MCA/AMDGPUCustomBehaviour.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This header declares the interfaces, data structures, and pass entry points for AMDGPUCustomBehaviour in the LLVM AMDGPU backend. It defines the contracts consumed by other AMDGPU backend components. / 该头文件声明 LLVM AMDGPU 后端中 AMDGPUCustomBehaviour 的接口、数据结构与 Pass 入口。它定义了其他 AMDGPU 后端组件依赖的契约。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: File banner, includes, and setup
```cpp
//===------------------- AMDGPUCustomBehaviour.h ----------------*-C++ -* -===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
/// \file
///
/// This file defines the AMDGPUCustomBehaviour class which inherits from
/// CustomBehaviour. This class is used by the tool llvm-mca to enforce
/// target specific behaviour that is not expressed well enough in the
/// scheduling model for mca to enforce it automatically.
///
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_AMDGPU_MCA_AMDGPUCUSTOMBEHAVIOUR_H
#define LLVM_LIB_TARGET_AMDGPU_MCA_AMDGPUCUSTOMBEHAVIOUR_H

#include "Utils/AMDGPUBaseInfo.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/MCA/CustomBehaviour.h"
#include "llvm/TargetParser/TargetParser.h"

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 25-46: Declares class AMDGPUInstrPostProcess
```cpp
namespace llvm {
namespace mca {

class AMDGPUInstrPostProcess : public InstrPostProcess {
  void processWaitCnt(Instruction &Inst, const MCInst &MCI);

public:
  AMDGPUInstrPostProcess(const MCSubtargetInfo &STI, const MCInstrInfo &MCII)
      : InstrPostProcess(STI, MCII) {}

  ~AMDGPUInstrPostProcess() override = default;

  void postProcessInstruction(Instruction &Inst, const MCInst &MCI) override;
};

struct WaitCntInfo {
  bool VmCnt = false;
  bool ExpCnt = false;
  bool LgkmCnt = false;
  bool VsCnt = false;
};

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPUInstrPostProcess`, `WaitCntInfo`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPUInstrPostProcess`, `WaitCntInfo`。

### Lines 47-70: Declares class AMDGPUCustomBehaviour
```cpp
class AMDGPUCustomBehaviour : public CustomBehaviour {
  /// Whenever MCA would like to dispatch an s_waitcnt instructions,
  /// we must check all the instruction that are still executing to see if
  /// they modify the same CNT as we need to wait for. This vector
  /// gets built in the constructor and contains 1 WaitCntInfo struct
  /// for each instruction within the SrcManager. Each element
  /// tells us which CNTs that instruction may interact with.
  /// We conservatively assume some instructions interact with more
  /// CNTs than they do in reality, so we will occasionally wait
  /// longer than necessary, but we shouldn't ever wait for shorter.
  std::vector<WaitCntInfo> InstrWaitCntInfo;

  /// This method gets called from the constructor and is
  /// where we setup the InstrWaitCntInfo vector.
  /// The core logic for determining which CNTs an instruction
  /// interacts with is taken from SIInsertWaitcnts::updateEventWaitcntAfter().
  /// Unfortunately, some of the logic from that function is not available to us
  /// in this scope so we conservatively end up assuming that some
  /// instructions interact with more CNTs than they do in reality.
  void generateWaitCntInfo();
  /// Helper function used in generateWaitCntInfo()
  bool hasModifiersSet(const std::unique_ptr<Instruction> &Inst,
                       AMDGPU::OpName OpName) const;
  /// Helper function used in generateWaitCntInfo()
```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPUCustomBehaviour`, `SIInsertWaitcnts::updateEventWaitcntAfter`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPUCustomBehaviour`, `SIInsertWaitcnts::updateEventWaitcntAfter`。

### Lines 71-90: Declares isGWS
```cpp
  bool isGWS(uint32_t Opcode) const;
  /// Helper function used in generateWaitCntInfo()
  bool isAlwaysGDS(uint32_t Opcode) const;
  /// Helper function used in generateWaitCntInfo()
  bool isVMEM(const MCInstrDesc &MCID);
  /// This method gets called from checkCustomHazard when mca is attempting to
  /// dispatch an s_waitcnt instruction (or one of its variants). The method
  /// looks at each of the instructions that are still executing in the pipeline
  /// to determine if the waitcnt should force a wait.
  unsigned handleWaitCnt(ArrayRef<InstRef> IssuedInst, const InstRef &IR);
  /// Based on the type of s_waitcnt instruction we are looking at, and what its
  /// operands are, this method will set the values for each of the cnt
  /// references provided as arguments.
  void computeWaitCnt(const InstRef &IR, unsigned &Vmcnt, unsigned &Expcnt,
                      unsigned &Lgkmcnt, unsigned &Vscnt);

public:
  AMDGPUCustomBehaviour(const MCSubtargetInfo &STI,
                        const mca::SourceMgr &SrcMgr, const MCInstrInfo &MCII);

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 91-105: Preprocessor guards and macros
```cpp
  ~AMDGPUCustomBehaviour() override = default;
  /// This method is used to determine if an instruction
  /// should be allowed to be dispatched. The return value is
  /// how many cycles until the instruction can be dispatched.
  /// This method is called after MCA has already checked for
  /// register and hardware dependencies so this method should only
  /// implement custom behaviour and dependencies that are not picked up
  /// by MCA naturally.
  unsigned checkCustomHazard(ArrayRef<InstRef> IssuedInst,
                             const InstRef &IR) override;
};
} // namespace mca
} // namespace llvm

#endif
```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。

## Key Concepts / 关键概念
- **Language / 语言**: C++ header
- **Primary symbols / 主要符号**: `AMDGPUInstrPostProcess`, `WaitCntInfo`, `AMDGPUCustomBehaviour`, `SIInsertWaitcnts::updateEventWaitcntAfter`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; scheduling / 调度
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"Utils/AMDGPUBaseInfo.h"`
- `"llvm/ADT/SmallVector.h"`
- `"llvm/MCA/CustomBehaviour.h"`
- `"llvm/TargetParser/TargetParser.h"`
