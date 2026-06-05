# AMDGPURegBankLegalizeHelper.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPURegBankLegalizeHelper.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This header declares the interfaces, data structures, and pass entry points for AMDGPURegBankLegalizeHelper in the LLVM AMDGPU backend. It defines the contracts consumed by other AMDGPU backend components. / 该头文件声明 LLVM AMDGPU 后端中 AMDGPURegBankLegalizeHelper 的接口、数据结构与 Pass 入口。它定义了其他 AMDGPU 后端组件依赖的契约。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: File banner, includes, and setup
```cpp
//===- AMDGPURegBankLegalizeHelper ------------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_AMDGPU_AMDGPUREGBANKLEGALIZEHELPER_H
#define LLVM_LIB_TARGET_AMDGPU_AMDGPUREGBANKLEGALIZEHELPER_H

#include "AMDGPURegBankLegalizeRules.h"
#include "llvm/ADT/SmallSet.h"
#include "llvm/CodeGen/GlobalISel/GenericMachineInstrs.h"
#include "llvm/CodeGen/MachineOptimizationRemarkEmitter.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"

namespace llvm {

class MachineIRBuilder;

namespace AMDGPU {

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation. Main symbols: `MachineIRBuilder`.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。 主要符号：`MachineIRBuilder`。

### Lines 24-47: Declares struct WaterfallInfo
```cpp
/// Holds waterfall loop information: the set of SGPR operand registers that
/// need waterfalling, and an instruction range [Start, End) to wrap in the
/// loop.
struct WaterfallInfo {
  SmallSet<Register, 4> SgprWaterfallOperandRegs;
  MachineBasicBlock::iterator Start;
  MachineBasicBlock::iterator End;
};

// Receives list of RegBankLLTMappingApplyID and applies register banks on all
// operands. It is user's responsibility to provide RegBankLLTMappingApplyIDs
// for all register operands, there is no need to specify NonReg for trailing
// imm operands. This finishes selection of register banks if there is no need
// to replace instruction. In other case InstApplyMethod will create new
// instruction(s).
class RegBankLegalizeHelper {
  MachineFunction &MF;
  const GCNSubtarget &ST;
  MachineIRBuilder &B;
  MachineRegisterInfo &MRI;
  const MachineUniformityInfo &MUI;
  const RegisterBankInfo &RBI;
  MachineOptimizationRemarkEmitter MORE;
  const RegBankLegalizeRules &RBLRules;
```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `WaterfallInfo`, `RegBankLegalizeHelper`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`WaterfallInfo`, `RegBankLegalizeHelper`。

### Lines 48-68: Implements LLT::scalar
```cpp
  const bool IsWave32;
  const RegisterBank *SgprRB;
  const RegisterBank *VgprRB;
  const RegisterBank *AgprRB;
  const RegisterBank *VccRB;

  static constexpr LLT S1 = LLT::scalar(1);
  static constexpr LLT S16 = LLT::scalar(16);
  static constexpr LLT S32 = LLT::scalar(32);
  static constexpr LLT S64 = LLT::scalar(64);
  static constexpr LLT S96 = LLT::scalar(96);
  static constexpr LLT S128 = LLT::scalar(128);
  static constexpr LLT S256 = LLT::scalar(256);

  static constexpr LLT V2S16 = LLT::fixed_vector(2, 16);
  static constexpr LLT V4S16 = LLT::fixed_vector(4, 16);
  static constexpr LLT V6S16 = LLT::fixed_vector(6, 16);
  static constexpr LLT V8S16 = LLT::fixed_vector(8, 16);
  static constexpr LLT V16S16 = LLT::fixed_vector(16, 16);
  static constexpr LLT V32S16 = LLT::fixed_vector(32, 16);

```
**EN:** This section contains concrete logic for LLT::scalar. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `LLT::scalar`, `LLT::fixed_vector`.
**CN:** 本节包含与 LLT::scalar 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`LLT::scalar`, `LLT::fixed_vector`。

### Lines 69-91: Implements LLT::fixed_vector
```cpp
  static constexpr LLT V2S32 = LLT::fixed_vector(2, 32);
  static constexpr LLT V3S32 = LLT::fixed_vector(3, 32);
  static constexpr LLT V4S32 = LLT::fixed_vector(4, 32);
  static constexpr LLT V6S32 = LLT::fixed_vector(6, 32);
  static constexpr LLT V7S32 = LLT::fixed_vector(7, 32);
  static constexpr LLT V8S32 = LLT::fixed_vector(8, 32);
  static constexpr LLT V16S32 = LLT::fixed_vector(16, 32);

  static constexpr LLT V2S64 = LLT::fixed_vector(2, 64);
  static constexpr LLT V3S64 = LLT::fixed_vector(3, 64);
  static constexpr LLT V4S64 = LLT::fixed_vector(4, 64);
  static constexpr LLT V8S64 = LLT::fixed_vector(8, 64);
  static constexpr LLT V16S64 = LLT::fixed_vector(16, 64);

  static constexpr LLT P1 = LLT::pointer(1, 64);
  static constexpr LLT P4 = LLT::pointer(4, 64);
  static constexpr LLT P6 = LLT::pointer(6, 32);

  MachineRegisterInfo::VRegAttrs SgprRB_S32 = {SgprRB, S32};
  MachineRegisterInfo::VRegAttrs SgprRB_S16 = {SgprRB, S16};
  MachineRegisterInfo::VRegAttrs VgprRB_S32 = {VgprRB, S32};
  MachineRegisterInfo::VRegAttrs VccRB_S1 = {VccRB, S1};

```
**EN:** This section contains concrete logic for LLT::fixed_vector. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `LLT::fixed_vector`, `LLT::pointer`.
**CN:** 本节包含与 LLT::fixed_vector 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`LLT::fixed_vector`, `LLT::pointer`。

### Lines 92-115: Defines RegBankLegalizeHelper
```cpp
public:
  RegBankLegalizeHelper(MachineIRBuilder &B, const MachineUniformityInfo &MUI,
                        const RegisterBankInfo &RBI,
                        const RegBankLegalizeRules &RBLRules);

  bool findRuleAndApplyMapping(MachineInstr &MI);

private:
  bool executeInWaterfallLoop(MachineIRBuilder &B, const WaterfallInfo &WFI);

  LLT getTyFromID(RegBankLLTMappingApplyID ID);
  LLT getBTyFromID(RegBankLLTMappingApplyID ID, LLT Ty);

  const RegisterBank *getRegBankFromID(RegBankLLTMappingApplyID ID);

  bool
  applyMappingDst(MachineInstr &MI, unsigned &OpIdx,
                  const SmallVectorImpl<RegBankLLTMappingApplyID> &MethodIDs);

  bool
  applyMappingSrc(MachineInstr &MI, unsigned &OpIdx,
                  const SmallVectorImpl<RegBankLLTMappingApplyID> &MethodIDs,
                  WaterfallInfo &WFI);

```
**EN:** This section contains concrete logic for RegBankLegalizeHelper. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 RegBankLegalizeHelper 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 116-139: Defines splitLoad
```cpp
  bool splitLoad(MachineInstr &MI, ArrayRef<LLT> LLTBreakdown,
                 LLT MergeTy = LLT());
  bool widenLoad(MachineInstr &MI, LLT WideTy, LLT MergeTy = LLT());
  bool widenMMOToS32(GAnyLoad &MI) const;

  bool lower(MachineInstr &MI, const RegBankLLTMapping &Mapping,
             WaterfallInfo &WFI);

  bool lowerVccExtToSel(MachineInstr &MI);
  std::pair<Register, Register> unpackZExt(Register Reg);
  std::pair<Register, Register> unpackSExt(Register Reg);
  std::pair<Register, Register> unpackAExt(Register Reg);
  std::pair<Register, Register> unpackAExtTruncS16(Register Reg);
  bool lowerUnpackBitShift(MachineInstr &MI);
  bool lowerV_BFE(MachineInstr &MI);
  bool lowerS_BFE(MachineInstr &MI);
  bool lowerUniMAD64(MachineInstr &MI);
  bool lowerSplitTo32(MachineInstr &MI);
  bool lowerSplitTo32Mul(MachineInstr &MI);
  bool lowerSplitTo16(MachineInstr &MI);
  bool lowerSplitTo32Select(MachineInstr &MI);
  bool lowerSplitTo32SExtInReg(MachineInstr &MI);
  bool lowerSplitBitCount64To32(MachineInstr &MI);
  bool lowerUnpackMinMax(MachineInstr &MI);
```
**EN:** This section contains concrete logic for splitLoad. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 splitLoad 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 140-153: Preprocessor guards and macros
```cpp
  bool lowerUnpackAExt(MachineInstr &MI);
  bool lowerExtrVecEltToSel(MachineInstr &MI);
  bool lowerExtrVecEltTo32(MachineInstr &MI);
  bool lowerInsVecEltToSel(MachineInstr &MI);
  bool lowerInsVecEltTo32(MachineInstr &MI);
  bool lowerAbsToNegMax(MachineInstr &MI);
  bool lowerAbsToS32(MachineInstr &MI);
  bool applyRegisterBanksVgprWithSgprRsrc(MachineInstr &MI, unsigned RsrcIdx);
};

} // end namespace AMDGPU
} // end namespace llvm

#endif
```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。

## Key Concepts / 关键概念
- **Language / 语言**: C++ header
- **Primary symbols / 主要符号**: `MachineIRBuilder`, `WaterfallInfo`, `RegBankLegalizeHelper`, `LLT::scalar`, `LLT::fixed_vector`, `LLT::pointer`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; lowering / 降低
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPURegBankLegalizeRules.h"`
- `"llvm/ADT/SmallSet.h"`
- `"llvm/CodeGen/GlobalISel/GenericMachineInstrs.h"`
- `"llvm/CodeGen/MachineOptimizationRemarkEmitter.h"`
- `"llvm/CodeGen/MachineRegisterInfo.h"`
