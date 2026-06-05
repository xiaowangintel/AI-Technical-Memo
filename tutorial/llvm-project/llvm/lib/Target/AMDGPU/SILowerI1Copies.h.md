# SILowerI1Copies.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/SILowerI1Copies.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This header declares the interfaces, data structures, and pass entry points for SILowerI1Copies in the LLVM AMDGPU backend. It defines the contracts consumed by other AMDGPU backend components. / 该头文件声明 LLVM AMDGPU 后端中 SILowerI1Copies 的接口、数据结构与 Pass 入口。它定义了其他 AMDGPU 后端组件依赖的契约。

## Line-by-Line Analysis / 逐行分析
### Lines 1-21: File banner, includes, and setup
```cpp
//===-- SILowerI1Copies.h --------------------------------------*- C++ -*--===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// Interface definition of the PhiLoweringHelper class that implements lane
/// mask merging algorithm for divergent i1 phis.
//
//===----------------------------------------------------------------------===//

#include "AMDGPULaneMaskUtils.h"
#include "GCNSubtarget.h"
#include "llvm/CodeGen/MachineBasicBlock.h"
#include "llvm/CodeGen/MachinePostDominators.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/CodeGen/MachineSSAUpdater.h"

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 22-44: Declares struct Incoming
```cpp
namespace llvm {
namespace AMDGPU {
/// Incoming for lane mask phi as machine instruction, incoming register \p Reg
/// and incoming block \p Block are taken from machine instruction.
/// \p UpdatedReg (if valid) is \p Reg lane mask merged with another lane mask.
struct Incoming {
  Register Reg;
  MachineBasicBlock *Block;
  Register UpdatedReg;

  Incoming(Register Reg, MachineBasicBlock *Block, Register UpdatedReg)
      : Reg(Reg), Block(Block), UpdatedReg(UpdatedReg) {}
};

Register createLaneMaskReg(MachineRegisterInfo *MRI,
                           MachineRegisterInfo::VRegAttrs LaneMaskRegAttrs);

class PhiLoweringHelper {
public:
  PhiLoweringHelper(MachineFunction *MF, MachineDominatorTree *DT,
                    MachinePostDominatorTree *PDT);
  virtual ~PhiLoweringHelper() = default;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `Incoming`, `PhiLoweringHelper`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`Incoming`, `PhiLoweringHelper`。

### Lines 45-65: Preprocessor guards and macros
```cpp
protected:
  bool IsWave32 = false;
  MachineFunction *MF = nullptr;
  MachineDominatorTree *DT = nullptr;
  MachinePostDominatorTree *PDT = nullptr;
  MachineRegisterInfo *MRI = nullptr;
  const GCNSubtarget *ST = nullptr;
  const SIInstrInfo *TII = nullptr;
  MachineRegisterInfo::VRegAttrs LaneMaskRegAttrs;
  const AMDGPU::LaneMaskConstants *LMC = nullptr;

#ifndef NDEBUG
  DenseSet<Register> PhiRegisters;
#endif

public:
  bool lowerPhis();
  bool isConstantLaneMask(Register Reg, bool &Val) const;
  MachineBasicBlock::iterator
  getSaluInsertionAtEnd(MachineBasicBlock &MBB) const;

```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。

### Lines 66-82: Defines initializeLaneMaskRegisterAttributes
```cpp
  void initializeLaneMaskRegisterAttributes(Register LaneMask) {
    LaneMaskRegAttrs = MRI->getVRegAttrs(LaneMask);
  }

  void
  initializeLaneMaskRegisterAttributes(MachineRegisterInfo::VRegAttrs Attrs) {
    LaneMaskRegAttrs = Attrs;
  }

  bool isLaneMaskReg(Register Reg) const {
    return TII->getRegisterInfo().isSGPRReg(*MRI, Reg) &&
           TII->getRegisterInfo().getRegSizeInBits(Reg, *MRI) ==
               ST->getWavefrontSize();
  }

  // Helpers from lowerPhis that are different between sdag and global-isel.

```
**EN:** This section contains concrete logic for initializeLaneMaskRegisterAttributes. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 initializeLaneMaskRegisterAttributes 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 83-98: Declares markAsLaneMask
```cpp
  virtual void markAsLaneMask(Register DstReg) const = 0;
  virtual void getCandidatesForLowering(
      SmallVectorImpl<MachineInstr *> &Vreg1Phis) const = 0;
  virtual void
  collectIncomingValuesFromPhi(const MachineInstr *MI,
                               SmallVectorImpl<Incoming> &Incomings) const = 0;
  virtual void replaceDstReg(Register NewReg, Register OldReg,
                             MachineBasicBlock *MBB) = 0;
  virtual void buildMergeLaneMasks(MachineBasicBlock &MBB,
                                   MachineBasicBlock::iterator I,
                                   const DebugLoc &DL, Register DstReg,
                                   Register PrevReg, Register CurReg) = 0;
  virtual void constrainAsLaneMask(Incoming &In) = 0;
};
} // namespace AMDGPU
} // end namespace llvm
```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

## Key Concepts / 关键概念
- **Language / 语言**: C++ header
- **Primary symbols / 主要符号**: `Incoming`, `PhiLoweringHelper`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; lowering / 降低
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPULaneMaskUtils.h"`
- `"GCNSubtarget.h"`
- `"llvm/CodeGen/MachineBasicBlock.h"`
- `"llvm/CodeGen/MachinePostDominators.h"`
- `"llvm/CodeGen/MachineRegisterInfo.h"`
- `"llvm/CodeGen/MachineSSAUpdater.h"`
