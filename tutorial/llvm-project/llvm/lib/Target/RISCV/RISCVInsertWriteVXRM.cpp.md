# RISCVInsertWriteVXRM.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVInsertWriteVXRM.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements insertion of VXRM configuration writes for vector fixed-point behavior. / 实现为向量定点行为插入 VXRM 配置写入。

## Line-by-Line Analysis / 逐行分析
### Lines 1-26: Commentary and design intent / 注释与设计意图
```cpp
//===-- RISCVInsertWriteVXRM.cpp - Insert Write of RISC-V VXRM CSR --------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This pass inserts writes to the VXRM CSR as needed by vector instructions.
// Each instruction that uses VXRM carries an operand that contains its required
// VXRM value. This pass tries to optimize placement to avoid redundant writes
// to VXRM.
//
// This is done using 2 dataflow algorithms. The first is a forward data flow
// to calculate where a VXRM value is available. The second is a backwards
// dataflow to determine where a VXRM value is anticipated.
//
// Finally, we use the results of these two dataflows to insert VXRM writes
// where a value is anticipated, but not available.
//
// FIXME: This pass does not split critical edges, so there can still be some
// redundancy.
//
// FIXME: If we are willing to have writes that aren't always needed, we could
// reduce the number of VXRM writes in some cases.
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 27-40: Header imports and compile-time dependencies / 头文件导入与编译期依赖
```cpp

#include "MCTargetDesc/RISCVBaseInfo.h"
#include "RISCV.h"
#include "RISCVSubtarget.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include <queue>

using namespace llvm;

#define DEBUG_TYPE "riscv-insert-write-vxrm"
#define RISCV_INSERT_WRITE_VXRM_NAME "RISC-V Insert Write VXRM Pass"

namespace {
```
**EN:** This block gathers the headers required by the file, revealing which LLVM layers and helper utilities the implementation relies on.

**CN:** 该区段汇集文件所需的头文件，体现实现依赖的 LLVM 层次与辅助工具。

### Lines 41-57: Type declaration for VXRMInfo / VXRMInfo 的类型声明
```cpp
class VXRMInfo {
  uint8_t VXRMImm = 0;

  enum : uint8_t {
    Uninitialized,
    Static,
    Unknown,
  } State = Uninitialized;

public:
  VXRMInfo() = default;

  static VXRMInfo getUnknown() {
    VXRMInfo Info;
    Info.setUnknown();
    return Info;
  }
```
**EN:** This block declares a type, its members, and its responsibilities so later code can implement or consume the interface.

**CN:** 该区段声明类型、成员及其职责，供后续代码实现或使用该接口。

### Lines 58-73: Function implementation: isValid / 函数实现：isValid
```cpp

  bool isValid() const { return State != Uninitialized; }
  void setUnknown() { State = Unknown; }
  bool isUnknown() const { return State == Unknown; }

  bool isStatic() const { return State == Static; }

  void setVXRMImm(unsigned Imm) {
    assert(Imm <= 3 && "Unexpected VXRM value");
    VXRMImm = Imm;
    State = Static;
  }
  unsigned getVXRMImm() const {
    assert(isStatic() && VXRMImm <= 3 && "Unexpected state");
    return VXRMImm;
  }
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 74-88: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  bool operator==(const VXRMInfo &Other) const {
    // Uninitialized is only equal to another Uninitialized.
    if (State != Other.State)
      return false;

    if (isStatic())
      return VXRMImm == Other.VXRMImm;

    assert((isValid() || isUnknown()) && "Unexpected state");
    return true;
  }

  bool operator!=(const VXRMInfo &Other) const { return !(*this == Other); }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 89-103: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  // Calculate the VXRMInfo visible to a block assuming this and Other are
  // both predecessors.
  VXRMInfo intersect(const VXRMInfo &Other) const {
    // If the new value isn't valid, ignore it.
    if (!Other.isValid())
      return *this;

    // If this value isn't valid, this must be the first predecessor, use it.
    if (!isValid())
      return Other;

    // If either is unknown, the result is unknown.
    if (isUnknown() || Other.isUnknown())
      return VXRMInfo::getUnknown();
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 104-122: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    // If we have an exact match, return this.
    if (*this == Other)
      return *this;

    // Otherwise the result is unknown.
    return VXRMInfo::getUnknown();
  }

  // Calculate the VXRMInfo visible to a block assuming this and Other
  // are both predecessors. To allow speculatively running WriteVXRM
  // we will ignore Unknowns if one of this and Other have valid
  // WriteVXRM. Rationale: WriteVXRM causes a pipeline flush in some
  // uarchs and moving it outside loops is very important for some
  // workloads.
  VXRMInfo intersectAnticipated(const VXRMInfo &Other) const {
    // If the new value isn't valid, ignore it.
    if (!Other.isValid())
      return *this;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 123-136: Definitions and supporting logic / 定义与支撑逻辑
```cpp
    // If this value isn't valid, this must be the first predecessor, use it.
    if (!isValid())
      return Other;

    // If either is unknown, the result is the other one.
    if (isUnknown())
      return Other;
    if (Other.isUnknown())
      return *this;

    // If we have an exact match, return this.
    if (*this == Other)
      return *this;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 137-157: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    // Otherwise the result is unknown.
    return VXRMInfo::getUnknown();
  }

#if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
  /// Support for debugging, callable in GDB: V->dump()
  LLVM_DUMP_METHOD void dump() const {
    print(dbgs());
    dbgs() << "\n";
  }

  void print(raw_ostream &OS) const {
    OS << '{';
    if (!isValid())
      OS << "Uninitialized";
    else if (isUnknown())
      OS << "Unknown";
    else
      OS << getVXRMImm();
    OS << '}';
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 158-172: Header guard and interface framing / 头文件保护与接口框架
```cpp
#endif
};

#if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
LLVM_ATTRIBUTE_USED
inline raw_ostream &operator<<(raw_ostream &OS, const VXRMInfo &V) {
  V.print(OS);
  return OS;
}
#endif

struct BlockData {
  // Indicates if the block uses VXRM. Uninitialized means no use.
  VXRMInfo VXRMUse;
```
**EN:** This block establishes include guards and the outer structure of the public interface.

**CN:** 该区段建立头文件保护并给出公共接口的外层结构。

### Lines 173-187: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  // Indicates the VXRM output from the block. Uninitialized means transparent.
  VXRMInfo VXRMOut;

  // Keeps track of the available VXRM value at the start of the basic block.
  VXRMInfo AvailableIn;

  // Keeps track of the available VXRM value at the end of the basic block.
  VXRMInfo AvailableOut;

  // Keeps track of what VXRM is anticipated at the start of the basic block.
  VXRMInfo AnticipatedIn;

  // Keeps track of what VXRM is anticipated at the end of the basic block.
  VXRMInfo AnticipatedOut;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 188-202: Type declaration for RISCVInsertWriteVXRM / RISCVInsertWriteVXRM 的类型声明
```cpp
  // Keeps track of whether the block is already in the queue.
  bool InQueue;

  BlockData() = default;
};

class RISCVInsertWriteVXRM : public MachineFunctionPass {
  const TargetInstrInfo *TII;

  std::vector<BlockData> BlockInfo;
  std::queue<const MachineBasicBlock *> WorkList;

public:
  static char ID;
```
**EN:** This block declares a type, its members, and its responsibilities so later code can implement or consume the interface.

**CN:** 该区段声明类型、成员及其职责，供后续代码实现或使用该接口。

### Lines 203-221: Function implementation: MachineFunctionPass::getAnalysisUsage / 函数实现：MachineFunctionPass::getAnalysisUsage
```cpp
  RISCVInsertWriteVXRM() : MachineFunctionPass(ID) {}

  bool runOnMachineFunction(MachineFunction &MF) override;

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();
    MachineFunctionPass::getAnalysisUsage(AU);
  }

  StringRef getPassName() const override {
    return RISCV_INSERT_WRITE_VXRM_NAME;
  }

private:
  bool computeVXRMChanges(const MachineBasicBlock &MBB);
  void computeAvailable(const MachineBasicBlock &MBB);
  void computeAnticipated(const MachineFunction &MF, const MachineBasicBlock &MBB);
  void emitWriteVXRM(MachineBasicBlock &MBB);
};
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 222-238: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp

} // end anonymous namespace

char RISCVInsertWriteVXRM::ID = 0;

INITIALIZE_PASS(RISCVInsertWriteVXRM, DEBUG_TYPE, RISCV_INSERT_WRITE_VXRM_NAME,
                false, false)

bool RISCVInsertWriteVXRM::computeVXRMChanges(const MachineBasicBlock &MBB) {
  BlockData &BBInfo = BlockInfo[MBB.getNumber()];

  bool NeedVXRMWrite = false;
  for (const MachineInstr &MI : MBB) {
    int VXRMIdx = RISCVII::getVXRMOpNum(MI.getDesc());
    if (VXRMIdx >= 0 && !RISCVInstrInfo::ignoresVXRM(MI)) {
      unsigned NewVXRMImm = MI.getOperand(VXRMIdx).getImm();
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 239-253: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
      if (!BBInfo.VXRMUse.isValid())
        BBInfo.VXRMUse.setVXRMImm(NewVXRMImm);

      BBInfo.VXRMOut.setVXRMImm(NewVXRMImm);
      NeedVXRMWrite = true;
      continue;
    }

    if (MI.isCall() || MI.isInlineAsm() ||
        MI.modifiesRegister(RISCV::VXRM, /*TRI=*/nullptr)) {
      if (!BBInfo.VXRMUse.isValid())
        BBInfo.VXRMUse.setUnknown();

      BBInfo.VXRMOut.setUnknown();
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 254-270: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  }

  return NeedVXRMWrite;
}

void RISCVInsertWriteVXRM::computeAvailable(const MachineBasicBlock &MBB) {
  BlockData &BBInfo = BlockInfo[MBB.getNumber()];

  BBInfo.InQueue = false;

  VXRMInfo Available;
  if (MBB.pred_empty()) {
    Available.setUnknown();
  } else {
    for (const MachineBasicBlock *P : MBB.predecessors())
      Available = Available.intersect(BlockInfo[P->getNumber()].AvailableOut);
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 271-284: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  // If we don't have any valid available info, wait until we do.
  if (!Available.isValid())
    return;

  if (Available != BBInfo.AvailableIn) {
    BBInfo.AvailableIn = Available;
    LLVM_DEBUG(dbgs() << "AvailableIn state of " << printMBBReference(MBB)
                      << " changed to " << BBInfo.AvailableIn << "\n");
  }

  if (BBInfo.VXRMOut.isValid())
    Available = BBInfo.VXRMOut;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 285-298: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  if (Available == BBInfo.AvailableOut)
    return;

  BBInfo.AvailableOut = Available;
  LLVM_DEBUG(dbgs() << "AvailableOut state of " << printMBBReference(MBB)
                    << " changed to " << BBInfo.AvailableOut << "\n");

  // Add the successors to the work list so that we can propagate.
  for (MachineBasicBlock *S : MBB.successors()) {
    if (!BlockInfo[S->getNumber()].InQueue) {
      BlockInfo[S->getNumber()].InQueue = true;
      WorkList.push(S);
    }
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 299-318: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
}

void RISCVInsertWriteVXRM::computeAnticipated(const MachineFunction &MF, const MachineBasicBlock &MBB) {
  BlockData &BBInfo = BlockInfo[MBB.getNumber()];
  const RISCVSubtarget &ST = MF.getSubtarget<RISCVSubtarget>();

  BBInfo.InQueue = false;

  VXRMInfo Anticipated;
  if (MBB.succ_empty()) {
    Anticipated.setUnknown();
  } else {
    for (const MachineBasicBlock *S : MBB.successors())
      if (ST.hasVXRMPipelineFlush())
        Anticipated =
          Anticipated.intersectAnticipated(BlockInfo[S->getNumber()].AnticipatedIn);
      else
        Anticipated =
          Anticipated.intersect(BlockInfo[S->getNumber()].AnticipatedIn);
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 319-333: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  // If we don't have any valid anticipated info, wait until we do.
  if (!Anticipated.isValid())
    return;

  if (Anticipated != BBInfo.AnticipatedOut) {
    BBInfo.AnticipatedOut = Anticipated;
    LLVM_DEBUG(dbgs() << "AnticipatedOut state of " << printMBBReference(MBB)
                      << " changed to " << BBInfo.AnticipatedOut << "\n");
  }

  // If this block reads VXRM, copy it.
  if (BBInfo.VXRMUse.isValid())
    Anticipated = BBInfo.VXRMUse;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 334-347: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  if (Anticipated == BBInfo.AnticipatedIn)
    return;

  BBInfo.AnticipatedIn = Anticipated;
  LLVM_DEBUG(dbgs() << "AnticipatedIn state of " << printMBBReference(MBB)
                    << " changed to " << BBInfo.AnticipatedIn << "\n");

  // Add the predecessors to the work list so that we can propagate.
  for (MachineBasicBlock *P : MBB.predecessors()) {
    if (!BlockInfo[P->getNumber()].InQueue) {
      BlockInfo[P->getNumber()].InQueue = true;
      WorkList.push(P);
    }
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 348-375: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
}

void RISCVInsertWriteVXRM::emitWriteVXRM(MachineBasicBlock &MBB) {
  const BlockData &BBInfo = BlockInfo[MBB.getNumber()];

  VXRMInfo Info = BBInfo.AvailableIn;

  // Flag to indicates we need to insert a VXRM write. We want to delay it as
  // late as possible in this block.
  bool PendingInsert = false;

  // Insert VXRM write if anticipated and not available.
  if (BBInfo.AnticipatedIn.isStatic()) {
    // If this is the entry block and the value is anticipated, insert.
    if (MBB.isEntryBlock()) {
      PendingInsert = true;
    } else {
      // Search for any predecessors that wouldn't satisfy our requirement and
      // insert a write VXRM if needed.
      // NOTE: If one predecessor is able to provide the requirement, but
      // another isn't, it means we have a critical edge. The better placement
      // would be to split the critical edge.
      for (MachineBasicBlock *P : MBB.predecessors()) {
        const BlockData &PInfo = BlockInfo[P->getNumber()];
        // If it's available out of the predecessor, then we're ok.
        if (PInfo.AvailableOut.isStatic() &&
            PInfo.AvailableOut.getVXRMImm() ==
                BBInfo.AnticipatedIn.getVXRMImm())
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 376-390: Definitions and supporting logic / 定义与支撑逻辑
```cpp
          continue;
        // If the predecessor anticipates this value for all its successors,
        // then a write to VXRM would have already occurred before this block is
        // executed.
        if (PInfo.AnticipatedOut.isStatic() &&
            PInfo.AnticipatedOut.getVXRMImm() ==
                BBInfo.AnticipatedIn.getVXRMImm())
          continue;
        PendingInsert = true;
        break;
      }
    }

    Info = BBInfo.AnticipatedIn;
  }
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 391-406: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  for (MachineInstr &MI : MBB) {
    int VXRMIdx = RISCVII::getVXRMOpNum(MI.getDesc());
    if (VXRMIdx >= 0 && !RISCVInstrInfo::ignoresVXRM(MI)) {
      unsigned NewVXRMImm = MI.getOperand(VXRMIdx).getImm();

      if (PendingInsert || !Info.isStatic() ||
          Info.getVXRMImm() != NewVXRMImm) {
        assert((!PendingInsert ||
                (Info.isStatic() && Info.getVXRMImm() == NewVXRMImm)) &&
               "Pending VXRM insertion mismatch");
        LLVM_DEBUG(dbgs() << "Inserting before "; MI.print(dbgs()));
        BuildMI(MBB, MI, MI.getDebugLoc(), TII->get(RISCV::WriteVXRMImm))
            .addImm(NewVXRMImm);
        PendingInsert = false;
      }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 407-434: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

      MI.addOperand(MachineOperand::CreateReg(RISCV::VXRM, /*IsDef*/ false,
                                              /*IsImp*/ true));
      Info.setVXRMImm(NewVXRMImm);
      continue;
    }

    if (MI.isCall() || MI.isInlineAsm() ||
        MI.modifiesRegister(RISCV::VXRM, /*TRI=*/nullptr))
      Info.setUnknown();
  }

  // If all our successors anticipate a value, do the insert.
  // NOTE: It's possible that not all predecessors of our successor provide the
  // correct value. This can occur on critical edges. If we don't split the
  // critical edge we'll also have a write vxrm in the successor that is
  // redundant with this one.
  if (PendingInsert ||
      (BBInfo.AnticipatedOut.isStatic() &&
       (!Info.isStatic() ||
        Info.getVXRMImm() != BBInfo.AnticipatedOut.getVXRMImm()))) {
    assert((!PendingInsert ||
            (Info.isStatic() && BBInfo.AnticipatedOut.isStatic() &&
             Info.getVXRMImm() == BBInfo.AnticipatedOut.getVXRMImm())) &&
           "Pending VXRM insertion mismatch");
    LLVM_DEBUG(dbgs() << "Inserting at end of " << printMBBReference(MBB)
                      << " changing to " << BBInfo.AnticipatedOut << "\n");
    BuildMI(MBB, MBB.getFirstTerminator(), DebugLoc(),
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 435-450: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
            TII->get(RISCV::WriteVXRMImm))
        .addImm(BBInfo.AnticipatedOut.getVXRMImm());
  }
}

bool RISCVInsertWriteVXRM::runOnMachineFunction(MachineFunction &MF) {
  // Skip if the vector extension is not enabled.
  const RISCVSubtarget &ST = MF.getSubtarget<RISCVSubtarget>();
  if (!ST.hasVInstructions())
    return false;

  TII = ST.getInstrInfo();

  assert(BlockInfo.empty() && "Expect empty block infos");
  BlockInfo.resize(MF.getNumBlockIDs());
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 451-465: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  // Phase 1 - collect block information.
  bool NeedVXRMChange = false;
  for (const MachineBasicBlock &MBB : MF)
    NeedVXRMChange |= computeVXRMChanges(MBB);

  if (!NeedVXRMChange) {
    BlockInfo.clear();
    return false;
  }

  // Phase 2 - Compute available VXRM using a forward walk.
  for (const MachineBasicBlock &MBB : MF) {
    WorkList.push(&MBB);
    BlockInfo[MBB.getNumber()].InQueue = true;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 466-481: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  while (!WorkList.empty()) {
    const MachineBasicBlock &MBB = *WorkList.front();
    WorkList.pop();
    computeAvailable(MBB);
  }

  // Phase 3 - Compute anticipated VXRM using a backwards walk.
  for (const MachineBasicBlock &MBB : llvm::reverse(MF)) {
    WorkList.push(&MBB);
    BlockInfo[MBB.getNumber()].InQueue = true;
  }
  while (!WorkList.empty()) {
    const MachineBasicBlock &MBB = *WorkList.front();
    WorkList.pop();
    computeAnticipated(MF, MBB);
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 482-494: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  // Phase 4 - Emit VXRM writes at the earliest place possible.
  for (MachineBasicBlock &MBB : MF)
    emitWriteVXRM(MBB);

  BlockInfo.clear();

  return true;
}

FunctionPass *llvm::createRISCVInsertWriteVXRMPass() {
  return new RISCVInsertWriteVXRM();
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

## Key Concepts / 关键概念
- **RISC-V backend structure** / **RISC-V 后端结构**

## Dependencies / 依赖关系
- `MCTargetDesc/RISCVBaseInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCV.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVSubtarget.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/MachineFunctionPass.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `queue` — Directly referenced by this file. / 该文件直接引用的依赖。
