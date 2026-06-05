# RISCVVectorMaskDAGMutation.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVVectorMaskDAGMutation.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements SelectionDAG mutations for RISC-V vector mask handling. / 实现RISC-V 向量掩码处理的 SelectionDAG 变换。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: Commentary and design intent / 注释与设计意图
```cpp
//===- RISCVVectorMaskDAGMutation.cpp - RISC-V Vector Mask DAGMutation ----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// A schedule mutation that adds an artificial dependency between masks producer
// instructions and masked instructions, so that we can reduce the live range
// overlaps of mask registers.
//
// If there are multiple masks producers followed by multiple masked
// instructions, then at each masked instructions add dependency edges between
// every producer and masked instruction.
//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 17-31: Commentary and design intent / 注释与设计意图
```cpp
// The reason why we need to do this:
// 1. When tracking register pressure, we don't track physical registers.
// 2. We have a RegisterClass for mask register (which is `VMV0`), but we don't
//    use it by the time we reach scheduling. Instead, we use physical
//    register V0 directly and insert a `$v0 = COPY ...` before the use.
// 3. For mask producers, we are using VR RegisterClass (we can allocate V0-V31
//    to it). So if V0 is not available, there are still 31 available registers
//    out there.
//
// This means that the RegPressureTracker can't track the pressure of mask
// registers correctly.
//
// This schedule mutation is a workaround to fix this issue.
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 32-41: Header imports and compile-time dependencies / 头文件导入与编译期依赖
```cpp

#include "MCTargetDesc/RISCVBaseInfo.h"
#include "MCTargetDesc/RISCVMCTargetDesc.h"
#include "RISCVTargetMachine.h"
#include "llvm/CodeGen/LiveIntervals.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/ScheduleDAGInstrs.h"
#include "llvm/CodeGen/ScheduleDAGMutation.h"
#include "llvm/TargetParser/RISCVTargetParser.h"
```
**EN:** This block gathers the headers required by the file, revealing which LLVM layers and helper utilities the implementation relies on.

**CN:** 该区段汇集文件所需的头文件，体现实现依赖的 LLVM 层次与辅助工具。

### Lines 42-49: Header guard and interface framing / 头文件保护与接口框架
```cpp
#define DEBUG_TYPE "machine-scheduler"

namespace llvm {

static bool isCopyToV0(const MachineInstr &MI) {
  return MI.isFullCopy() && MI.getOperand(0).getReg() == RISCV::V0 &&
         MI.getOperand(1).getReg().isVirtual();
}
```
**EN:** This block establishes include guards and the outer structure of the public interface.

**CN:** 该区段建立头文件保护并给出公共接口的外层结构。

### Lines 50-58: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

static bool isSoleUseCopyToV0(SUnit &SU) {
  if (SU.Succs.size() != 1)
    return false;
  SDep &Dep = SU.Succs[0];
  // Ignore dependencies other than data or strong ordering.
  if (Dep.isWeak())
    return false;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 59-68: Type declaration for RISCVVectorMaskDAGMutation / RISCVVectorMaskDAGMutation 的类型声明
```cpp
  SUnit &DepSU = *Dep.getSUnit();
  if (DepSU.isBoundaryNode())
    return false;
  return isCopyToV0(*DepSU.getInstr());
}

class RISCVVectorMaskDAGMutation : public ScheduleDAGMutation {
private:
  const TargetRegisterInfo *TRI;
```
**EN:** This block declares a type, its members, and its responsibilities so later code can implement or consume the interface.

**CN:** 该区段声明类型、成员及其职责，供后续代码实现或使用该接口。

### Lines 69-80: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
public:
  RISCVVectorMaskDAGMutation(const TargetRegisterInfo *TRI) : TRI(TRI) {}

  void apply(ScheduleDAGInstrs *DAG) override {
    SUnit *NearestUseV0SU = nullptr;
    SmallVector<SUnit *, 2> DefMask;
    for (SUnit &SU : DAG->SUnits) {
      const MachineInstr *MI = SU.getInstr();
      bool UseV0 = MI->findRegisterUseOperand(RISCV::V0, TRI);
      if (isSoleUseCopyToV0(SU) && !UseV0)
        DefMask.push_back(&SU);
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 81-89: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
      if (UseV0) {
        NearestUseV0SU = &SU;

        // Copy may not be a real use, so skip it here.
        if (DefMask.size() > 1 && !MI->isCopy()) {
          for (SUnit *Def : DefMask)
            if (DAG->canAddEdge(Def, &SU))
              DAG->addEdge(Def, SDep(&SU, SDep::Artificial));
        }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 90-101: Definitions and supporting logic / 定义与支撑逻辑
```cpp

        if (!DefMask.empty())
          DefMask.erase(DefMask.begin());
      }

      if (NearestUseV0SU && NearestUseV0SU != &SU && isSoleUseCopyToV0(SU) &&
          // For LMUL=8 cases, there will be more possibilities to spill.
          // FIXME: We should use RegPressureTracker to do fine-grained
          // controls.
          RISCVII::getLMul(MI->getDesc().TSFlags) != RISCVVType::LMUL_8)
        DAG->addEdge(&SU, SDep(NearestUseV0SU, SDep::Artificial));
    }
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 102-109: Function implementation: createRISCVVectorMaskDAGMutation / 函数实现：createRISCVVectorMaskDAGMutation
```cpp
  }
};

std::unique_ptr<ScheduleDAGMutation>
createRISCVVectorMaskDAGMutation(const TargetRegisterInfo *TRI) {
  return std::make_unique<RISCVVectorMaskDAGMutation>(TRI);
}
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 110-110: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp
} // namespace llvm
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

## Key Concepts / 关键概念
- **Vector extension support** / **向量扩展支持**

## Dependencies / 依赖关系
- `MCTargetDesc/RISCVBaseInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `MCTargetDesc/RISCVMCTargetDesc.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVTargetMachine.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/LiveIntervals.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/MachineInstr.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/ScheduleDAGInstrs.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/ScheduleDAGMutation.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/TargetParser/RISCVTargetParser.h` — Directly referenced by this file. / 该文件直接引用的依赖。
