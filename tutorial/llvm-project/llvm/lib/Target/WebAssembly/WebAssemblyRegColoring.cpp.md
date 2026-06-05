# WebAssemblyRegColoring.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblyRegColoring.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file implements a virtual register coloring pass.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblyRegColoring.cpp`，主要负责 WebAssembly 后端的后端实现逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- WebAssemblyRegColoring.cpp - Register coloring --------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-16

```cpp
//===----------------------------------------------------------------------===//
///
/// \file
/// This file implements a virtual register coloring pass.
///
/// WebAssembly doesn't have a fixed number of registers, but it is still
/// desirable to minimize the total number of registers used in each function.
///
/// This code is modeled after lib/CodeGen/StackSlotColoring.cpp.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file". Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。 该区间与栈帧布局或栈访问相关。

### Lines 17-23

```cpp
//===----------------------------------------------------------------------===//

#include "WebAssembly.h"
#include "WebAssemblyMachineFunctionInfo.h"
#include "llvm/CodeGen/LiveIntervals.h"
#include "llvm/CodeGen/MachineBlockFrequencyInfo.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. Register assignment and register-class constraints matter here.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 24-30

```cpp
#include "llvm/CodeGen/Passes.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/raw_ostream.h"
using namespace llvm;

#define DEBUG_TYPE "wasm-reg-coloring"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 31-68

```cpp
namespace {
class WebAssemblyRegColoring final : public MachineFunctionPass {
public:
  static char ID; // Pass identification, replacement for typeid
  WebAssemblyRegColoring() : MachineFunctionPass(ID) {}

  StringRef getPassName() const override {
    return "WebAssembly Register Coloring";
  }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();
    AU.addRequired<LiveIntervalsWrapperPass>();
    AU.addRequired<MachineBlockFrequencyInfoWrapperPass>();
    AU.addPreserved<MachineBlockFrequencyInfoWrapperPass>();
    AU.addPreservedID(MachineDominatorsID);
    MachineFunctionPass::getAnalysisUsage(AU);
  }

  bool runOnMachineFunction(MachineFunction &MF) override;

private:
};
} // end anonymous namespace

char WebAssemblyRegColoring::ID = 0;
INITIALIZE_PASS(WebAssemblyRegColoring, DEBUG_TYPE,
                "Minimize number of registers used", false, false)

FunctionPass *llvm::createWebAssemblyRegColoring() {
  return new WebAssemblyRegColoring();
}

// Compute the total spill weight for VReg.
static float computeWeight(const MachineRegisterInfo *MRI,
                           const MachineBlockFrequencyInfo *MBFI,
                           unsigned VReg) {
  float Weight = 0.0f;
```
- **EN**: Declares a backend-facing type `WebAssemblyRegColoring`, `MachineFunctionPass`, `getPassName` and outlines the API or state that nearby code will rely on. Register assignment and register-class constraints matter here.
- **CN**: 这里声明面向后端的类型 `WebAssemblyRegColoring`, `MachineFunctionPass`, `getPassName`，并勾勒出周边代码会依赖的接口或状态。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 69-106

```cpp
  for (MachineOperand &MO : MRI->reg_nodbg_operands(VReg))
    Weight += LiveIntervals::getSpillWeight(MO.isDef(), MO.isUse(), MBFI,
                                            *MO.getParent());
  return Weight;
}

// Create a map of "Register -> vector of <SlotIndex, DBG_VALUE>".
// The SlotIndex is the slot index of the next non-debug instruction or the end
// of a BB, because DBG_VALUE's don't have slot index themselves.
// Adapted from RegisterCoalescer::buildVRegToDbgValueMap.
static DenseMap<Register, std::vector<std::pair<SlotIndex, MachineInstr *>>>
buildVRegToDbgValueMap(MachineFunction &MF, const LiveIntervals *Liveness) {
  DenseMap<Register, std::vector<std::pair<SlotIndex, MachineInstr *>>>
      DbgVRegToValues;
  const SlotIndexes *Slots = Liveness->getSlotIndexes();
  SmallVector<MachineInstr *, 8> ToInsert;

  // After collecting a block of DBG_VALUEs into ToInsert, enter them into the
  // map.
  auto CloseNewDVRange = [&DbgVRegToValues, &ToInsert](SlotIndex Slot) {
    for (auto *X : ToInsert) {
      for (const auto &Op : X->debug_operands()) {
        if (Op.isReg() && Op.getReg().isVirtual())
          DbgVRegToValues[Op.getReg()].push_back({Slot, X});
      }
    }

    ToInsert.clear();
  };

  // Iterate over all instructions, collecting them into the ToInsert vector.
  // Once a non-debug instruction is found, record the slot index of the
  // collected DBG_VALUEs.
  for (auto &MBB : MF) {
    SlotIndex CurrentSlot = Slots->getMBBStartIdx(&MBB);

    for (auto &MI : MBB) {
      if (MI.isDebugValue()) {
```
- **EN**: Implements helper routine(s) `reg_nodbg_operands`, `getSpillWeight`, `isDef` for this portion of the WebAssembly backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `reg_nodbg_operands`, `getSpillWeight`, `isDef`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 107-134

```cpp
        if (any_of(MI.debug_operands(), [](const MachineOperand &MO) {
              return MO.isReg() && MO.getReg().isVirtual();
            }))
          ToInsert.push_back(&MI);
      } else if (!MI.isDebugOrPseudoInstr()) {
        CurrentSlot = Slots->getInstructionIndex(MI);
        CloseNewDVRange(CurrentSlot);
      }
    }

    // Close range of DBG_VALUEs at the end of blocks.
    CloseNewDVRange(Slots->getMBBEndIdx(&MBB));
  }

  // Sort all DBG_VALUEs we've seen by slot number.
  for (auto &Pair : DbgVRegToValues)
    llvm::sort(Pair.second);
  return DbgVRegToValues;
}

// After register coalescing, some DBG_VALUEs will be invalid. Set them undef.
// This function has to run before the actual coalescing, i.e., the register
// changes.
static void undefInvalidDbgValues(
    const LiveIntervals *Liveness,
    ArrayRef<SmallVector<LiveInterval *, 4>> Assignments,
    DenseMap<Register, std::vector<std::pair<SlotIndex, MachineInstr *>>>
        &DbgVRegToValues) {
```
- **EN**: Implements helper routine(s) `any_of`, `debug_operands`, `isReg` for this portion of the WebAssembly backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `any_of`, `debug_operands`, `isReg`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 135-142

```cpp
#ifndef NDEBUG
  DenseSet<Register> SeenRegs;
#endif
  for (const auto &CoalescedIntervals : Assignments) {
    if (CoalescedIntervals.empty())
      continue;
    for (LiveInterval *LI : CoalescedIntervals) {
      Register Reg = LI->reg();
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. Notable symbols in this range include `empty`, `reg`.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 该区间中较显眼的符号包括 `empty`, `reg`。

### Lines 143-180

```cpp
#ifndef NDEBUG
      // Ensure we don't process the same register twice
      assert(SeenRegs.insert(Reg).second);
#endif
      auto RegMapIt = DbgVRegToValues.find(Reg);
      if (RegMapIt == DbgVRegToValues.end())
        continue;
      SlotIndex LastSlot;
      bool LastUndefResult = false;
      for (auto [Slot, DbgValue] : RegMapIt->second) {
        // All consecutive DBG_VALUEs have the same slot because the slot
        // indices they have is the one for the first non-debug instruction
        // after it, because DBG_VALUEs don't have slot index themselves. Before
        // doing live range queries, quickly check if the current DBG_VALUE has
        // the same slot index as the previous one, in which case we should do
        // the same. Note that RegMapIt->second, the vector of {SlotIndex,
        // DBG_VALUE}, is sorted by SlotIndex, which is necessary for this
        // check.
        if (Slot == LastSlot) {
          if (LastUndefResult) {
            LLVM_DEBUG(dbgs() << "Undefed: " << *DbgValue << "\n");
            DbgValue->setDebugValueUndef();
          }
          continue;
        }
        LastSlot = Slot;
        LastUndefResult = false;
        for (LiveInterval *OtherLI : CoalescedIntervals) {
          if (LI == OtherLI)
            continue;

          // This DBG_VALUE has 'Reg' (the current LiveInterval's register) as
          // its operand. If this DBG_VALUE's slot index is within other
          // registers' live ranges, this DBG_VALUE should be undefed. For
          // example, suppose %0 and %1 are to be coalesced into %0.
          //   ; %0's live range starts
          //   %0 = value_0
          //   DBG_VALUE %0, !"a", ...      (a)
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. Notable symbols in this range include `insert`, `find`, `end`.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 该区间中较显眼的符号包括 `insert`, `find`, `end`。

### Lines 181-218

```cpp
          //   DBG_VALUE %1, !"b", ...      (b)
          //   use %0
          //   ; %0's live range ends
          //   ...
          //   ; %1's live range starts
          //   %1 = value_1
          //   DBG_VALUE %0, !"c", ...      (c)
          //   DBG_VALUE %1, !"d", ...      (d)
          //   use %1
          //   ; %1's live range ends
          //
          // In this code, (b) and (c) should be set to undef. After the two
          // registers are coalesced, (b) will incorrectly say the variable
          // "b"'s value is 'value_0', and (c) will also incorrectly say the
          // variable "c"'s value is value_1. Note it doesn't actually matter
          // which register they are coalesced into (%0 or %1); (b) and (c)
          // should be set to undef as well if they are coalesced into %1.
          //
          // This happens DBG_VALUEs are not included when computing live
          // ranges.
          //
          // Note that it is not possible for this DBG_VALUE to be
          // simultaneously within 'Reg''s live range and one of other coalesced
          // registers' live ranges because if their live ranges overlapped they
          // would have not been selected as a coalescing candidate in the first
          // place.
          auto *SegmentIt = OtherLI->find(Slot);
          if (SegmentIt != OtherLI->end() && SegmentIt->contains(Slot)) {
            LLVM_DEBUG(dbgs() << "Undefed: " << *DbgValue << "\n");
            DbgValue->setDebugValueUndef();
            LastUndefResult = true;
            break;
          }
        }
      }
    }
  }
}
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "DBG_VALUE %1, !"b", ...      (b)". Notable symbols in this range include `and`, `into`, `find`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“DBG_VALUE %1, !"b", ...      (b)”。 该区间中较显眼的符号包括 `and`, `into`, `find`。

### Lines 219-256

```cpp

bool WebAssemblyRegColoring::runOnMachineFunction(MachineFunction &MF) {
  LLVM_DEBUG({
    dbgs() << "********** Register Coloring **********\n"
           << "********** Function: " << MF.getName() << '\n';
  });

  // If there are calls to setjmp or sigsetjmp, don't perform coloring. Virtual
  // registers could be modified before the longjmp is executed, resulting in
  // the wrong value being used afterwards.
  // TODO: Does WebAssembly need to care about setjmp for register coloring?
  if (MF.exposesReturnsTwice())
    return false;

  MachineRegisterInfo *MRI = &MF.getRegInfo();
  LiveIntervals *Liveness = &getAnalysis<LiveIntervalsWrapperPass>().getLIS();
  const MachineBlockFrequencyInfo *MBFI =
      &getAnalysis<MachineBlockFrequencyInfoWrapperPass>().getMBFI();
  WebAssemblyFunctionInfo &MFI = *MF.getInfo<WebAssemblyFunctionInfo>();

  // We don't preserve SSA form.
  MRI->leaveSSA();

  // Gather all register intervals into a list and sort them.
  unsigned NumVRegs = MRI->getNumVirtRegs();
  SmallVector<LiveInterval *, 0> SortedIntervals;
  SortedIntervals.reserve(NumVRegs);

  // Record DBG_VALUEs and their SlotIndexes.
  auto DbgVRegToValues = buildVRegToDbgValueMap(MF, Liveness);

  LLVM_DEBUG(dbgs() << "Interesting register intervals:\n");
  for (unsigned I = 0; I < NumVRegs; ++I) {
    Register VReg = Register::index2VirtReg(I);
    if (MFI.isVRegStackified(VReg))
      continue;
    // Skip unused registers, which can use $drop.
    if (MRI->use_empty(VReg))
```
- **EN**: Implements helper routine(s) `runOnMachineFunction`, `dbgs`, `getName` for this portion of the WebAssembly backend backend implementation logic. Register assignment and register-class constraints matter here.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `runOnMachineFunction`, `dbgs`, `getName`。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 257-294

```cpp
      continue;

    LiveInterval *LI = &Liveness->getInterval(VReg);
    assert(LI->weight() == 0.0f);
    LI->setWeight(computeWeight(MRI, MBFI, VReg));
    LLVM_DEBUG(LI->dump());
    SortedIntervals.push_back(LI);
  }
  LLVM_DEBUG(dbgs() << '\n');

  // Sort them to put arguments first (since we don't want to rename live-in
  // registers), by weight next, and then by position.
  // TODO: Investigate more intelligent sorting heuristics. For starters, we
  // should try to coalesce adjacent live intervals before non-adjacent ones.
  llvm::sort(SortedIntervals, [MRI](LiveInterval *LHS, LiveInterval *RHS) {
    if (MRI->isLiveIn(LHS->reg()) != MRI->isLiveIn(RHS->reg()))
      return MRI->isLiveIn(LHS->reg());
    if (LHS->weight() != RHS->weight())
      return LHS->weight() > RHS->weight();
    if (LHS->empty() || RHS->empty())
      return !LHS->empty() && RHS->empty();
    return *LHS < *RHS;
  });

  LLVM_DEBUG(dbgs() << "Coloring register intervals:\n");
  SmallVector<unsigned, 16> SlotMapping(SortedIntervals.size(), -1u);
  SmallVector<SmallVector<LiveInterval *, 4>, 16> Assignments(
      SortedIntervals.size());
  BitVector UsedColors(SortedIntervals.size());
  bool Changed = false;
  for (size_t I = 0, E = SortedIntervals.size(); I < E; ++I) {
    LiveInterval *LI = SortedIntervals[I];
    Register Old = LI->reg();
    size_t Color = I;
    const TargetRegisterClass *RC = MRI->getRegClass(Old);

    // Check if it's possible to reuse any of the used colors.
    if (!MRI->isLiveIn(Old))
```
- **EN**: Implements helper routine(s) `getInterval`, `weight`, `setWeight` for this portion of the WebAssembly backend backend implementation logic. Register assignment and register-class constraints matter here.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `getInterval`, `weight`, `setWeight`。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 295-331

```cpp
      for (unsigned C : UsedColors.set_bits()) {
        if (MRI->getRegClass(SortedIntervals[C]->reg()) != RC)
          continue;
        for (LiveInterval *OtherLI : Assignments[C])
          if (!OtherLI->empty() && OtherLI->overlaps(*LI))
            goto continue_outer;
        Color = C;
        break;
      continue_outer:;
      }

    Register New = SortedIntervals[Color]->reg();
    SlotMapping[I] = New;
    Changed |= Old != New;
    UsedColors.set(Color);
    Assignments[Color].push_back(LI);
    // If we reassigned the stack pointer, update the debug frame base info.
    if (Old != New && MFI.isFrameBaseVirtual() && MFI.getFrameBaseVreg() == Old)
      MFI.setFrameBaseVreg(New);
    LLVM_DEBUG(dbgs() << "Assigning vreg " << printReg(LI->reg()) << " to vreg "
                      << printReg(New) << "\n");
  }
  if (!Changed)
    return false;

  // Set DBG_VALUEs that will be invalid after coalescing to undef.
  undefInvalidDbgValues(Liveness, Assignments, DbgVRegToValues);

  // Rewrite register operands.
  for (size_t I = 0, E = SortedIntervals.size(); I < E; ++I) {
    Register Old = SortedIntervals[I]->reg();
    unsigned New = SlotMapping[I];
    if (Old != New)
      MRI->replaceRegWith(Old, New);
  }
  return true;
}
```
- **EN**: Implements helper routine(s) `set_bits`, `getRegClass`, `reg` for this portion of the WebAssembly backend backend implementation logic. Register assignment and register-class constraints matter here.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `set_bits`, `getRegClass`, `reg`。 这里重点涉及寄存器分配与寄存器类约束。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- Backend implementation logic / 后端实现逻辑
- Register modeling / 寄存器建模
- Stack frame management / 栈帧管理
- Instruction selection or opcode handling / 指令选择或操作码处理
- Declarative TableGen records / 声明式 TableGen 记录
- Vector or SIMD behavior / 向量或 SIMD 行为
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `WebAssembly.h`
- `WebAssemblyMachineFunctionInfo.h`
- `llvm/CodeGen/LiveIntervals.h`
- `llvm/CodeGen/MachineBlockFrequencyInfo.h`
- `llvm/CodeGen/MachineRegisterInfo.h`
- `llvm/CodeGen/Passes.h`
- `llvm/Support/Debug.h`
- `llvm/Support/raw_ostream.h`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- LLVM Support utilities / LLVM Support 工具层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
