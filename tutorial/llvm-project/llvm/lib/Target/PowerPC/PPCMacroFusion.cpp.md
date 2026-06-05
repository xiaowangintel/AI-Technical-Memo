# PPCMacroFusion.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCMacroFusion.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: mutation to pair instructions back to back.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCMacroFusion.cpp`，主要负责 PowerPC 后端的后端实现逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===- PPCMacroFusion.cpp - PowerPC Macro Fusion --------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-11

```cpp
//===----------------------------------------------------------------------===//
//
/// \file This file contains the PowerPC implementation of the DAG scheduling
///  mutation to pair instructions back to back.
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file This file contains the PowerPC implementation of the DAG scheduling".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file This file contains the PowerPC implementation of the DAG scheduling”。

### Lines 12-18

```cpp
//===----------------------------------------------------------------------===//

#include "PPC.h"
#include "PPCSubtarget.h"
#include "llvm/ADT/DenseSet.h"
#include "llvm/CodeGen/MacroFusion.h"
#include "llvm/CodeGen/ScheduleDAGMutation.h"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. Subtarget feature gating influences the behavior here.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 子目标特性裁剪会影响这里的行为。

### Lines 19-27

```cpp
#include <optional>

using namespace llvm;
namespace {

class FusionFeature {
public:
  typedef SmallDenseSet<unsigned> FusionOpSet;
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. Subtarget feature gating influences the behavior here.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 子目标特性裁剪会影响这里的行为。

### Lines 28-65

```cpp
  enum FusionKind {
  #define FUSION_KIND(KIND) FK_##KIND
  #define FUSION_FEATURE(KIND, HAS_FEATURE, DEP_OP_IDX, OPSET1, OPSET2) \
    FUSION_KIND(KIND),
  #include "PPCMacroFusion.def"
  FUSION_KIND(END)
  };
private:
  // Each fusion feature is assigned with one fusion kind. All the
  // instructions with the same fusion kind have the same fusion characteristic.
  FusionKind Kd;
  // True if this feature is enabled.
  bool Supported;
  // li rx, si
  // load rt, ra, rx
  // The dependent operand index in the second op(load). And the negative means
  // it could be any one. 
  int DepOpIdx;
  // The first fusion op set.
  FusionOpSet OpSet1;
  // The second fusion op set.
  FusionOpSet OpSet2;
public:
  FusionFeature(FusionKind Kind, bool HasFeature, int Index,
                const FusionOpSet &First, const FusionOpSet &Second) :
    Kd(Kind), Supported(HasFeature), DepOpIdx(Index), OpSet1(First), 
    OpSet2(Second) {}

  bool hasOp1(unsigned Opc) const { return OpSet1.contains(Opc); }
  bool hasOp2(unsigned Opc) const { return OpSet2.contains(Opc); }
  bool isSupported() const { return Supported; }
  std::optional<unsigned> depOpIdx() const {
    if (DepOpIdx < 0)
      return std::nullopt;
    return DepOpIdx;
  }

  FusionKind getKind() const { return Kd; }
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. Subtarget feature gating influences the behavior here.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 子目标特性裁剪会影响这里的行为。

### Lines 66-103

```cpp
};

static bool matchingRegOps(const MachineInstr &FirstMI,
                           int FirstMIOpIndex,
                           const MachineInstr &SecondMI,
                           int SecondMIOpIndex) {
  const MachineOperand &Op1 = FirstMI.getOperand(FirstMIOpIndex);
  const MachineOperand &Op2 = SecondMI.getOperand(SecondMIOpIndex);
  if (!Op1.isReg() || !Op2.isReg())
    return false;

  return Op1.getReg() == Op2.getReg();
}

static bool matchingImmOps(const MachineInstr &MI,
                           int MIOpIndex,
                           int64_t Expect,
                           unsigned ExtendFrom = 64) {
  const MachineOperand &Op = MI.getOperand(MIOpIndex);
  if (!Op.isImm())
    return false;
  int64_t Imm = Op.getImm();
  if (ExtendFrom < 64)
    Imm = SignExtend64(Imm, ExtendFrom);
  return Imm == Expect;
}

// Return true if the FirstMI meets the constraints of SecondMI according to
// fusion specification.
static bool checkOpConstraints(FusionFeature::FusionKind Kd,
                               const MachineInstr &FirstMI,
                               const MachineInstr &SecondMI) {
  switch (Kd) {
  // The hardware didn't require any specific check for the fused instructions'
  // operands. Therefore, return true to indicate that, it is fusable.
  default: return true;
  // [addi rt,ra,si - lxvd2x xt,ra,rb] etc.
  case FusionFeature::FK_AddiLoad: {
```
- **EN**: Implements helper routine(s) `matchingRegOps`, `getOperand`, `isReg` for this portion of the PowerPC backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `matchingRegOps`, `getOperand`, `isReg`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 104-141

```cpp
    // lxvd2x(ra) cannot be zero
    const MachineOperand &RA = SecondMI.getOperand(1);
    if (!RA.isReg())
      return true;

    return RA.getReg().isVirtual() ||
           (RA.getReg() != PPC::ZERO && RA.getReg() != PPC::ZERO8);
  }
  // [addis rt,ra,si - ld rt,ds(ra)] etc.
  case FusionFeature::FK_AddisLoad: {
    const MachineOperand &RT = SecondMI.getOperand(0);
    if (!RT.isReg())
      return true;

    // Only check it for non-virtual register.
    if (!RT.getReg().isVirtual())
      // addis(rt) = ld(ra) = ld(rt)
      // ld(rt) cannot be zero
      if (!matchingRegOps(SecondMI, 0, SecondMI, 2) ||
          (RT.getReg() == PPC::ZERO || RT.getReg() == PPC::ZERO8))
          return false;

    // addis(si) first 12 bits must be all 1s or all 0s
    const MachineOperand &SI = FirstMI.getOperand(2);
    if (!SI.isImm())
      return true;
    int64_t Imm = SI.getImm();
    if (((Imm & 0xFFF0) != 0) && ((Imm & 0xFFF0) != 0xFFF0))
      return false;

    // If si = 1111111111110000 and the msb of the d/ds field of the load equals
    // 1, then fusion does not occur.
    if ((Imm & 0xFFF0) == 0xFFF0) {
      const MachineOperand &D = SecondMI.getOperand(1);
      if (!D.isImm())
        return true;

      // 14 bit for DS field, while 16 bit for D field.
```
- **EN**: Implements helper routine(s) `lxvd2x`, `getOperand`, `isReg` for this portion of the PowerPC backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `lxvd2x`, `getOperand`, `isReg`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 142-179

```cpp
      int MSB = 15;
      if (SecondMI.getOpcode() == PPC::LD)
        MSB = 13;

      return (D.getImm() & (1ULL << MSB)) == 0;
    }
    return true;
  }

  case FusionFeature::FK_SldiAdd:
    return (matchingImmOps(FirstMI, 2, 3) && matchingImmOps(FirstMI, 3, 60)) ||
           (matchingImmOps(FirstMI, 2, 6) && matchingImmOps(FirstMI, 3, 57));

  // rldicl rx, ra, 1, 0  - xor
  case FusionFeature::FK_RotateLeftXor:
    return matchingImmOps(FirstMI, 2, 1) && matchingImmOps(FirstMI, 3, 0);

  // rldicr rx, ra, 1, 63 - xor
  case FusionFeature::FK_RotateRightXor:
    return matchingImmOps(FirstMI, 2, 1) && matchingImmOps(FirstMI, 3, 63);

  // We actually use CMPW* and CMPD*, 'l' doesn't exist as an operand in instr.

  // { lbz,lbzx,lhz,lhzx,lwz,lwzx } - cmpi 0,1,rx,{ 0,1,-1 }
  // { lbz,lbzx,lhz,lhzx,lwz,lwzx } - cmpli 0,L,rx,{ 0,1 }
  case FusionFeature::FK_LoadCmp1:
  // { ld,ldx } - cmpi 0,1,rx,{ 0,1,-1 }
  // { ld,ldx } - cmpli 0,1,rx,{ 0,1 }
  case FusionFeature::FK_LoadCmp2: {
    const MachineOperand &BT = SecondMI.getOperand(0);
    if (!BT.isReg() || (!BT.getReg().isVirtual() && BT.getReg() != PPC::CR0))
      return false;
    if (SecondMI.getOpcode() == PPC::CMPDI &&
        matchingImmOps(SecondMI, 2, -1, 16))
      return true;
    return matchingImmOps(SecondMI, 2, 0) || matchingImmOps(SecondMI, 2, 1);
  }
```
- **EN**: Implements helper routine(s) `getOpcode`, `getImm`, `matchingImmOps` for this portion of the PowerPC backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `getOpcode`, `getImm`, `matchingImmOps`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 180-217

```cpp
  // { lha,lhax,lwa,lwax } - cmpi 0,L,rx,{ 0,1,-1 }
  case FusionFeature::FK_LoadCmp3: {
    const MachineOperand &BT = SecondMI.getOperand(0);
    if (!BT.isReg() || (!BT.getReg().isVirtual() && BT.getReg() != PPC::CR0))
      return false;
    return matchingImmOps(SecondMI, 2, 0) || matchingImmOps(SecondMI, 2, 1) ||
           matchingImmOps(SecondMI, 2, -1, 16);
  }

  // mtctr - { bcctr,bcctrl }
  case FusionFeature::FK_ZeroMoveCTR:
    // ( mtctr rx ) is alias of ( mtspr 9, rx )
    return (FirstMI.getOpcode() != PPC::MTSPR &&
            FirstMI.getOpcode() != PPC::MTSPR8) ||
           matchingImmOps(FirstMI, 0, 9);

  // mtlr - { bclr,bclrl }
  case FusionFeature::FK_ZeroMoveLR:
    // ( mtlr rx ) is alias of ( mtspr 8, rx )
    return (FirstMI.getOpcode() != PPC::MTSPR &&
            FirstMI.getOpcode() != PPC::MTSPR8) ||
           matchingImmOps(FirstMI, 0, 8);

  // addis rx,ra,si - addi rt,rx,SI, SI >= 0
  case FusionFeature::FK_AddisAddi: {
    const MachineOperand &RA = FirstMI.getOperand(1);
    const MachineOperand &SI = SecondMI.getOperand(2);
    if (!SI.isImm() || !RA.isReg())
      return false;
    if (RA.getReg() == PPC::ZERO || RA.getReg() == PPC::ZERO8)
      return false;
    return SignExtend64(SI.getImm(), 16) >= 0;
  }

  // addi rx,ra,si - addis rt,rx,SI, ra > 0, SI >= 2
  case FusionFeature::FK_AddiAddis: {
    const MachineOperand &RA = FirstMI.getOperand(1);
    const MachineOperand &SI = FirstMI.getOperand(2);
```
- **EN**: Implements helper routine(s) `getOperand`, `isReg`, `getReg` for this portion of the PowerPC backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `getOperand`, `isReg`, `getReg`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 218-243

```cpp
    if (!SI.isImm() || !RA.isReg())
      return false;
    if (RA.getReg() == PPC::ZERO || RA.getReg() == PPC::ZERO8)
      return false;
    int64_t ExtendedSI = SignExtend64(SI.getImm(), 16);
    return ExtendedSI >= 2;
  }
  }

  llvm_unreachable("All the cases should have been handled");
  return true;
}

/// Check if the instr pair, FirstMI and SecondMI, should be fused together.
/// Given SecondMI, when FirstMI is unspecified, then check if SecondMI may be
/// part of a fused pair at all.
static bool shouldScheduleAdjacent(const TargetInstrInfo &TII,
                                   const TargetSubtargetInfo &TSI,
                                   const MachineInstr *FirstMI,
                                   const MachineInstr &SecondMI) {
  // We use the PPC namespace to avoid the need to prefix opcodes with PPC:: in
  // the def file.
  using namespace PPC;

  const PPCSubtarget &ST = static_cast<const PPCSubtarget&>(TSI);
  static const FusionFeature FusionFeatures[] = {
```
- **EN**: Implements helper routine(s) `isImm`, `isReg`, `getReg` for this portion of the PowerPC backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `isImm`, `isReg`, `getReg`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 244-281

```cpp
  #define FUSION_FEATURE(KIND, HAS_FEATURE, DEP_OP_IDX, OPSET1, OPSET2) { \
    FusionFeature::FUSION_KIND(KIND), ST.HAS_FEATURE(), DEP_OP_IDX, { OPSET1 },\
    { OPSET2 } },
   #include "PPCMacroFusion.def"
  };
  #undef FUSION_KIND

  for (auto &Feature : FusionFeatures) {
    // Skip if the feature is not supported.
    if (!Feature.isSupported())
      continue;

    // Only when the SecondMI is fusable, we are starting to look for the
    // fusable FirstMI.
    if (Feature.hasOp2(SecondMI.getOpcode())) {
      // If FirstMI == nullptr, that means, we're only checking whether SecondMI
      // can be fused at all.
      if (!FirstMI)
        return true;

      // Checking if the FirstMI is fusable with the SecondMI.
      if (!Feature.hasOp1(FirstMI->getOpcode()))
        continue;

      auto DepOpIdx = Feature.depOpIdx();
      if (DepOpIdx) {
        // Checking if the result of the FirstMI is the desired operand of the
        // SecondMI if the DepOpIdx is set. Otherwise, ignore it.
        if (!matchingRegOps(*FirstMI, 0, SecondMI, *DepOpIdx))
          return false;
      }

      // Checking more on the instruction operands.
      if (checkOpConstraints(Feature.getKind(), *FirstMI, SecondMI))
        return true;
    }
  }
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. Subtarget feature gating influences the behavior here.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 子目标特性裁剪会影响这里的行为。

### Lines 282-293

```cpp
  return false;
}

} // end anonymous namespace

namespace llvm {

std::unique_ptr<ScheduleDAGMutation> createPowerPCMacroFusionDAGMutation() {
  return createMacroFusionDAGMutation(shouldScheduleAdjacent);
}

} // end namespace llvm
```
- **EN**: Opens or re-enters namespace scope so backend symbols stay grouped under LLVM naming conventions. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里进入或重新进入命名空间作用域，使后端符号按照 LLVM 的命名约定组织在一起。 这一段包含调度或处理器模型元数据。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- Backend implementation logic / 后端实现逻辑
- Subtarget features / 子目标特性
- Instruction selection or opcode handling / 指令选择或操作码处理
- Declarative TableGen records / 声明式 TableGen 记录
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `PPC.h`
- `PPCSubtarget.h`
- `llvm/ADT/DenseSet.h`
- `llvm/CodeGen/MacroFusion.h`
- `llvm/CodeGen/ScheduleDAGMutation.h`
- `optional`
- `PPCMacroFusion.def`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
