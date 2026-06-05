# ARMConstantIslandPass.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMConstantIslandPass.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file contains a pass that splits the constant pool up into 'islands' which are scattered through-out the function.  This is required due to the limited pc-relative displacements that ARM has.
- 用途 (CN): 实现 ARM 后端中的 `ARMConstantIslandPass`，重点处理常量岛放置与布局约束。

## Line-by-Line Analysis / 逐行分析

### Lines 1-13
```cpp
//===- ARMConstantIslandPass.cpp - ARM constant islands -------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains a pass that splits the constant pool up into 'islands'
// which are scattered through-out the function.  This is required due to the
// limited pc-relative displacements that ARM has.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 15-29
```cpp
#include "ARM.h"
#include "ARMBaseInstrInfo.h"
#include "ARMBasicBlockInfo.h"
#include "ARMMachineFunctionInfo.h"
#include "ARMSubtarget.h"
#include "MCTargetDesc/ARMBaseInfo.h"
#include "MVETailPredUtils.h"
#include "Thumb2InstrInfo.h"
#include "Utils/ARMBaseInfo.h"
#include "llvm/ADT/DenseMap.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/CodeGen/LivePhysRegs.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 30-44
```cpp
#include "llvm/CodeGen/MachineBasicBlock.h"
#include "llvm/CodeGen/MachineConstantPool.h"
#include "llvm/CodeGen/MachineDominators.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/MachineJumpTableInfo.h"
#include "llvm/CodeGen/MachineOperand.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/Config/llvm-config.h"
#include "llvm/IR/DataLayout.h"
#include "llvm/IR/DebugLoc.h"
#include "llvm/MC/MCInstrDesc.h"
#include "llvm/Pass.h"
#include "llvm/Support/CommandLine.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 45-54
```cpp
#include "llvm/Support/Compiler.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/Format.h"
#include "llvm/Support/raw_ostream.h"
#include <algorithm>
#include <cassert>
#include <cstdint>
#include <iterator>
#include <vector>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 56-56
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 58-58
```cpp
#define DEBUG_TYPE "arm-cp-islands"
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 60-72
```cpp
#define ARM_CP_ISLANDS_OPT_NAME \
  "ARM constant island placement and branch shortening pass"
STATISTIC(NumCPEs,       "Number of constpool entries");
STATISTIC(NumSplit,      "Number of uncond branches inserted");
STATISTIC(NumCBrFixed,   "Number of cond branches fixed");
STATISTIC(NumUBrFixed,   "Number of uncond branches fixed");
STATISTIC(NumTBs,        "Number of table branches generated");
STATISTIC(NumT2CPShrunk, "Number of Thumb2 constantpool instructions shrunk");
STATISTIC(NumT2BrShrunk, "Number of Thumb2 immediate branches shrunk");
STATISTIC(NumCBZ,        "Number of CBZ / CBNZ formed");
STATISTIC(NumJTMoved,    "Number of jump table destination blocks moved");
STATISTIC(NumJTInserted, "Number of jump table intermediate blocks inserted");
STATISTIC(NumLEInserted, "Number of LE backwards branches inserted");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 74-76
```cpp
static cl::opt<bool>
AdjustJumpTableBlocks("arm-adjust-jump-tables", cl::Hidden, cl::init(true),
          cl::desc("Adjust basic block layout to better use TB[BH]"));
```
- EN: Declares `cl::init`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `cl::init`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 78-80
```cpp
static cl::opt<unsigned>
CPMaxIteration("arm-constant-island-max-iteration", cl::Hidden, cl::init(30),
          cl::desc("The max number of iteration for converge"));
```
- EN: Declares `cl::init`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `cl::init`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 82-85
```cpp
static cl::opt<bool> SynthesizeThumb1TBB(
    "arm-synthesize-thumb-1-tbb", cl::Hidden, cl::init(true),
    cl::desc("Use compressed jump tables in Thumb-1 by synthesizing an "
             "equivalent to the TBB/TBH instructions"));
```
- EN: Declares `cl::init`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `cl::init`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 87-87
```cpp
namespace {
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 89-101
```cpp
  /// ARMConstantIslands - Due to limited PC-relative displacements, ARM
  /// requires constant pool entries to be scattered among the instructions
  /// inside a function.  To do this, it completely ignores the normal LLVM
  /// constant pool; instead, it places constants wherever it feels like with
  /// special instructions.
  ///
  /// The terminology used in this pass includes:
  ///   Islands - Clumps of constants placed in the function.
  ///   Water   - Potential places where an island could be formed.
  ///   CPE     - A constant pool entry that has been placed somewhere, which
  ///             tracks a list of users.
  class ARMConstantIslands : public MachineFunctionPass {
    std::unique_ptr<ARMBasicBlockUtils> BBUtils = nullptr;
```
- EN: Declares `ARMConstantIslands`, packaging target-specific state and APIs around `ARMConstantIslandPass`.
- CN: 这里声明 `ARMConstantIslands`，把与 `ARMConstantIslandPass` 相关的目标特定状态和 API 组织在一起。

### Lines 103-106
```cpp
    /// WaterList - A sorted list of basic blocks where islands could be placed
    /// (i.e. blocks that don't fall through to the following block, due
    /// to a return, unreachable, or unconditional branch).
    std::vector<MachineBasicBlock*> WaterList;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 108-110
```cpp
    /// NewWaterList - The subset of WaterList that was created since the
    /// previous iteration by inserting unconditional branches.
    SmallPtrSet<MachineBasicBlock *, 4> NewWaterList;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 112-112
```cpp
    using water_iterator = std::vector<MachineBasicBlock *>::iterator;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 114-128
```cpp
    /// CPUser - One user of a constant pool, keeping the machine instruction
    /// pointer, the constant pool being referenced, and the max displacement
    /// allowed from the instruction to the CP.  The HighWaterMark records the
    /// highest basic block where a new CPEntry can be placed.  To ensure this
    /// pass terminates, the CP entries are initially placed at the end of the
    /// function and then move monotonically to lower addresses.  The
    /// exception to this rule is when the current CP entry for a particular
    /// CPUser is out of range, but there is another CP entry for the same
    /// constant value in range.  We want to use the existing in-range CP
    /// entry, but if it later moves out of range, the search for new water
    /// should resume where it left off.  The HighWaterMark is used to record
    /// that point.
    struct CPUser {
      MachineInstr *MI;
      MachineInstr *CPEMI;
```
- EN: Declares `CPUser`, packaging target-specific state and APIs around `ARMConstantIslandPass`.
- CN: 这里声明 `CPUser`，把与 `ARMConstantIslandPass` 相关的目标特定状态和 API 组织在一起。

### Lines 129-133
```cpp
      MachineBasicBlock *HighWaterMark;
      unsigned MaxDisp;
      bool NegOk;
      bool IsSoImm;
      bool KnownAlignment = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 135-139
```cpp
      CPUser(MachineInstr *mi, MachineInstr *cpemi, unsigned maxdisp,
             bool neg, bool soimm)
        : MI(mi), CPEMI(cpemi), MaxDisp(maxdisp), NegOk(neg), IsSoImm(soimm) {
        HighWaterMark = CPEMI->getParent();
      }
```
- EN: Implements `CPUser`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `CPUser`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 141-147
```cpp
      /// getMaxDisp - Returns the maximum displacement supported by MI.
      /// Correct for unknown alignment.
      /// Conservatively subtract 2 bytes to handle weird alignment effects.
      unsigned getMaxDisp() const {
        return (KnownAlignment ? MaxDisp : MaxDisp - 2) - 2;
      }
    };
```
- EN: Implements `getMaxDisp`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getMaxDisp`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 149-151
```cpp
    /// CPUsers - Keep track of all of the machine instructions that use various
    /// constant pools and their max displacement.
    std::vector<CPUser> CPUsers;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 153-159
```cpp
    /// CPEntry - One per constant pool entry, keeping the machine instruction
    /// pointer, the constpool index, and the number of CPUser's which
    /// reference this entry.
    struct CPEntry {
      MachineInstr *CPEMI;
      unsigned CPI;
      unsigned RefCount;
```
- EN: Declares `CPEntry`, packaging target-specific state and APIs around `ARMConstantIslandPass`.
- CN: 这里声明 `CPEntry`，把与 `ARMConstantIslandPass` 相关的目标特定状态和 API 组织在一起。

### Lines 161-163
```cpp
      CPEntry(MachineInstr *cpemi, unsigned cpi, unsigned rc = 0)
        : CPEMI(cpemi), CPI(cpi), RefCount(rc) {}
    };
```
- EN: Implements `CPEntry`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `CPEntry`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 165-174
```cpp
    /// CPEntries - Keep track of all of the constant pool entry machine
    /// instructions. For each original constpool index (i.e. those that existed
    /// upon entry to this pass), it keeps a vector of entries.  Original
    /// elements are cloned as we go along; the clones are put in the vector of
    /// the original element, but have distinct CPIs.
    ///
    /// The first half of CPEntries contains generic constants, the second half
    /// contains jump tables. Use getCombinedIndex on a generic CPEMI to look up
    /// which vector it will be in here.
    std::vector<std::vector<CPEntry>> CPEntries;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 176-178
```cpp
    /// Maps a JT index to the offset in CPEntries containing copies of that
    /// table. The equivalent map for a CONSTPOOL_ENTRY is the identity.
    DenseMap<int, int> JumpTableEntryIndices;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 180-182
```cpp
    /// Maps a JT index to the LEA that actually uses the index to calculate its
    /// base address.
    DenseMap<int, int> JumpTableUserIndices;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 184-193
```cpp
    /// ImmBranch - One per immediate branch, keeping the machine instruction
    /// pointer, conditional or unconditional, the max displacement,
    /// and (if isCond is true) the corresponding unconditional branch
    /// opcode.
    struct ImmBranch {
      MachineInstr *MI;
      unsigned MaxDisp : 31;
      LLVM_PREFERRED_TYPE(bool)
      unsigned isCond : 1;
      unsigned UncondBr;
```
- EN: Declares `ImmBranch`, packaging target-specific state and APIs around `ARMConstantIslandPass`.
- CN: 这里声明 `ImmBranch`，把与 `ARMConstantIslandPass` 相关的目标特定状态和 API 组织在一起。

### Lines 195-197
```cpp
      ImmBranch(MachineInstr *mi, unsigned maxdisp, bool cond, unsigned ubr)
        : MI(mi), MaxDisp(maxdisp), isCond(cond), UncondBr(ubr) {}
    };
```
- EN: Implements `ImmBranch`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ImmBranch`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 199-200
```cpp
    /// ImmBranches - Keep track of all the immediate branch instructions.
    std::vector<ImmBranch> ImmBranches;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 202-203
```cpp
    /// PushPopMIs - Keep track of all the Thumb push / pop instructions.
    SmallVector<MachineInstr*, 4> PushPopMIs;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 205-206
```cpp
    /// T2JumpTables - Keep track of all the Thumb2 jumptable instructions.
    SmallVector<MachineInstr*, 4> T2JumpTables;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 208-217
```cpp
    MachineFunction *MF;
    MachineConstantPool *MCP;
    const ARMBaseInstrInfo *TII;
    const ARMSubtarget *STI;
    ARMFunctionInfo *AFI;
    MachineDominatorTree *DT = nullptr;
    bool isThumb;
    bool isThumb1;
    bool isThumb2;
    bool isPositionIndependentOrROPI;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 219-220
```cpp
  public:
    static char ID;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 222-222
```cpp
    ARMConstantIslands() : MachineFunctionPass(ID) {}
```
- EN: Implements `ARMConstantIslands`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMConstantIslands`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 224-224
```cpp
    bool runOnMachineFunction(MachineFunction &MF) override;
```
- EN: Declares `runOnMachineFunction`, a pass-entry routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `runOnMachineFunction`，它是一个围绕机器函数状态展开的Pass 入口例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 226-229
```cpp
    void getAnalysisUsage(AnalysisUsage &AU) const override {
      AU.addRequired<MachineDominatorTreeWrapperPass>();
      MachineFunctionPass::getAnalysisUsage(AU);
    }
```
- EN: Implements `getAnalysisUsage`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getAnalysisUsage`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 231-233
```cpp
    MachineFunctionProperties getRequiredProperties() const override {
      return MachineFunctionProperties().setNoVRegs();
    }
```
- EN: Implements `getRequiredProperties`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getRequiredProperties`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 235-237
```cpp
    StringRef getPassName() const override {
      return ARM_CP_ISLANDS_OPT_NAME;
    }
```
- EN: Implements `getPassName`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getPassName`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 239-253
```cpp
  private:
    void doInitialConstPlacement(std::vector<MachineInstr *> &CPEMIs);
    void doInitialJumpTablePlacement(std::vector<MachineInstr *> &CPEMIs);
    bool BBHasFallthrough(MachineBasicBlock *MBB);
    CPEntry *findConstPoolEntry(unsigned CPI, const MachineInstr *CPEMI);
    Align getCPEAlign(const MachineInstr *CPEMI);
    void scanFunctionJumpTables();
    void initializeFunctionInfo(const std::vector<MachineInstr*> &CPEMIs);
    MachineBasicBlock *splitBlockBeforeInstr(MachineInstr *MI);
    void updateForInsertedWaterBlock(MachineBasicBlock *NewBB);
    bool decrementCPEReferenceCount(unsigned CPI, MachineInstr* CPEMI);
    unsigned getCombinedIndex(const MachineInstr *CPEMI);
    int findInRangeCPEntry(CPUser& U, unsigned UserOffset);
    bool findAvailableWater(CPUser&U, unsigned UserOffset,
                            water_iterator &WaterIter, bool CloserWater);
```
- EN: Declares `doInitialConstPlacement`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `doInitialConstPlacement`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 254-268
```cpp
    void createNewWater(unsigned CPUserIndex, unsigned UserOffset,
                        MachineBasicBlock *&NewMBB);
    bool handleConstantPoolUser(unsigned CPUserIndex, bool CloserWater);
    void removeDeadCPEMI(MachineInstr *CPEMI);
    bool removeUnusedCPEntries();
    bool isCPEntryInRange(MachineInstr *MI, unsigned UserOffset,
                          MachineInstr *CPEMI, unsigned Disp, bool NegOk,
                          bool DoDump = false);
    bool isWaterInRange(unsigned UserOffset, MachineBasicBlock *Water,
                        CPUser &U, unsigned &Growth);
    bool fixupImmediateBr(ImmBranch &Br);
    bool fixupConditionalBr(ImmBranch &Br);
    bool fixupUnconditionalBr(ImmBranch &Br);
    bool optimizeThumb2Instructions();
    bool optimizeThumb2Branches();
```
- EN: Declares `createNewWater`, a query/helper routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `createNewWater`，它是一个围绕机器基本块展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 269-276
```cpp
    bool reorderThumb2JumpTables();
    bool preserveBaseRegister(MachineInstr *JumpMI, MachineInstr *LEAMI,
                              unsigned &DeadSize, bool &CanDeleteLEA,
                              bool &BaseRegKill);
    bool optimizeThumb2JumpTables();
    MachineBasicBlock *adjustJTTargetBlockForward(unsigned JTI,
                                                  MachineBasicBlock *BB,
                                                  MachineBasicBlock *JTBB);
```
- EN: Declares `reorderThumb2JumpTables`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `reorderThumb2JumpTables`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 278-280
```cpp
    unsigned getUserOffset(CPUser&) const;
    void dumpBBs();
    void verify();
```
- EN: Declares `getUserOffset`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getUserOffset`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 282-289
```cpp
    bool isOffsetInRange(unsigned UserOffset, unsigned TrialOffset,
                         unsigned Disp, bool NegativeOK, bool IsSoImm = false);
    bool isOffsetInRange(unsigned UserOffset, unsigned TrialOffset,
                         const CPUser &U) {
      return isOffsetInRange(UserOffset, TrialOffset,
                             U.getMaxDisp(), U.NegOk, U.IsSoImm);
    }
  };
```
- EN: Implements `isOffsetInRange`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isOffsetInRange`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 291-291
```cpp
} // end anonymous namespace
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 293-293
```cpp
char ARMConstantIslands::ID = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 295-309
```cpp
/// verify - check BBOffsets, BBSizes, alignment of islands
void ARMConstantIslands::verify() {
#ifndef NDEBUG
  BBInfoVector &BBInfo = BBUtils->getBBInfo();
  assert(is_sorted(*MF, [&BBInfo](const MachineBasicBlock &LHS,
                                  const MachineBasicBlock &RHS) {
    return BBInfo[LHS.getNumber()].postOffset() <
           BBInfo[RHS.getNumber()].postOffset();
  }));
  LLVM_DEBUG(dbgs() << "Verifying " << CPUsers.size() << " CP users.\n");
  for (CPUser &U : CPUsers) {
    unsigned UserOffset = getUserOffset(U);
    // Verify offset using the real max displacement without the safety
    // adjustment.
    if (isCPEntryInRange(U.MI, UserOffset, U.CPEMI, U.getMaxDisp()+2, U.NegOk,
```
- EN: Implements `ARMConstantIslands::verify`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMConstantIslands::verify`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 310-320
```cpp
                         /* DoDump = */ true)) {
      LLVM_DEBUG(dbgs() << "OK\n");
      continue;
    }
    LLVM_DEBUG(dbgs() << "Out of range.\n");
    dumpBBs();
    LLVM_DEBUG(MF->dump());
    llvm_unreachable("Constant pool entry out of range!");
  }
#endif
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 322-336
```cpp
#if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
/// print block size and offset information - debugging
LLVM_DUMP_METHOD void ARMConstantIslands::dumpBBs() {
  LLVM_DEBUG({
    BBInfoVector &BBInfo = BBUtils->getBBInfo();
    for (unsigned J = 0, E = BBInfo.size(); J !=E; ++J) {
      const BasicBlockInfo &BBI = BBInfo[J];
      dbgs() << format("%08x %bb.%u\t", BBI.Offset, J)
             << " kb=" << unsigned(BBI.KnownBits)
             << " ua=" << unsigned(BBI.Unalign) << " pa=" << Log2(BBI.PostAlign)
             << format(" size=%#x\n", BBInfo[J].Size);
    }
  });
}
#endif
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 338-344
```cpp
// Align blocks where the previous block does not fall through. This may add
// extra NOP's but they will not be executed. It uses the PrefLoopAlignment as a
// measure of how much to align, and only runs at CodeGenOptLevel::Aggressive.
static bool AlignBlocks(MachineFunction *MF, const ARMSubtarget *STI) {
  if (MF->getTarget().getOptLevel() != CodeGenOptLevel::Aggressive ||
      MF->getFunction().hasOptSize())
    return false;
```
- EN: Implements `AlignBlocks`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `AlignBlocks`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 346-349
```cpp
  auto *TLI = STI->getTargetLowering();
  const Align Alignment = TLI->getPrefLoopAlignment();
  if (Alignment < 4)
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 351-357
```cpp
  bool Changed = false;
  bool PrevCanFallthrough = true;
  for (auto &MBB : *MF) {
    if (!PrevCanFallthrough) {
      Changed = true;
      MBB.setAlignment(Alignment);
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 359-359
```cpp
    PrevCanFallthrough = MBB.canFallThrough();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 361-375
```cpp
    // For LOB's, the ARMLowOverheadLoops pass may remove the unconditional
    // branch later in the pipeline.
    if (STI->hasLOB()) {
      for (const auto &MI : reverse(MBB.terminators())) {
        if (MI.getOpcode() == ARM::t2B &&
            MI.getOperand(0).getMBB() == MBB.getNextNode())
          continue;
        if (isLoopStart(MI) || MI.getOpcode() == ARM::t2LoopEnd ||
            MI.getOpcode() == ARM::t2LoopEndDec) {
          PrevCanFallthrough = true;
          break;
        }
        // Any other terminator - nothing to do
        break;
      }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 376-377
```cpp
    }
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 379-380
```cpp
  return Changed;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 382-385
```cpp
bool ARMConstantIslands::runOnMachineFunction(MachineFunction &mf) {
  MF = &mf;
  MCP = mf.getConstantPool();
  BBUtils = std::make_unique<ARMBasicBlockUtils>(mf);
```
- EN: Implements `ARMConstantIslands::runOnMachineFunction`, a pass-entry routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMConstantIslands::runOnMachineFunction`，它是一个围绕机器函数状态展开的Pass 入口例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 387-389
```cpp
  LLVM_DEBUG(dbgs() << "***** ARMConstantIslands: "
                    << MCP->getConstants().size() << " CP entries, aligned to "
                    << MCP->getConstantPoolAlign().value() << " bytes *****\n");
```
- EN: Declares `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 391-396
```cpp
  STI = &MF->getSubtarget<ARMSubtarget>();
  TII = STI->getInstrInfo();
  isPositionIndependentOrROPI =
      STI->getTargetLowering()->isPositionIndependent() || STI->isROPI();
  AFI = MF->getInfo<ARMFunctionInfo>();
  DT = &getAnalysis<MachineDominatorTreeWrapperPass>().getDomTree();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 398-400
```cpp
  isThumb = AFI->isThumbFunction();
  isThumb1 = AFI->isThumb1OnlyFunction();
  isThumb2 = AFI->isThumb2Function();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 402-406
```cpp
  bool GenerateTBB = isThumb2 || (isThumb1 && SynthesizeThumb1TBB);
  // TBB generation code in this constant island pass has not been adapted to
  // deal with speculation barriers.
  if (STI->hardenSlsRetBr())
    GenerateTBB = false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 408-410
```cpp
  // Renumber all of the machine basic blocks in the function, guaranteeing that
  // the numbers agree with the position of the block in the function.
  MF->RenumberBlocks();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 412-422
```cpp
  // Try to reorder and otherwise adjust the block layout to make good use
  // of the TB[BH] instructions.
  bool MadeChange = false;
  if (GenerateTBB && AdjustJumpTableBlocks) {
    scanFunctionJumpTables();
    MadeChange |= reorderThumb2JumpTables();
    // Data is out of date, so clear it. It'll be re-computed later.
    T2JumpTables.clear();
    // Blocks may have shifted around. Keep the numbering up to date.
    MF->RenumberBlocks();
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 424-425
```cpp
  // Align any non-fallthrough blocks
  MadeChange |= AlignBlocks(MF, STI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 427-431
```cpp
  // Perform the initial placement of the constant pool entries.  To start with,
  // we put them all at the end of the function.
  std::vector<MachineInstr*> CPEMIs;
  if (!MCP->isEmpty())
    doInitialConstPlacement(CPEMIs);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 433-434
```cpp
  if (MF->getJumpTableInfo())
    doInitialJumpTablePlacement(CPEMIs);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 436-437
```cpp
  /// The next UID to take is the first unused one.
  AFI->initPICLabelUId(CPEMIs.size());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 439-444
```cpp
  // Do the initial scan of the function, building up information about the
  // sizes of each block, the location of all the water, and finding all of the
  // constant pool users.
  initializeFunctionInfo(CPEMIs);
  CPEMIs.clear();
  LLVM_DEBUG(dumpBBs());
```
- EN: Declares `initializeFunctionInfo`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `initializeFunctionInfo`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 446-449
```cpp
  // Functions with jump tables need an alignment of 4 because they use the ADR
  // instruction, which aligns the PC to 4 bytes before adding an offset.
  if (!T2JumpTables.empty())
    MF->ensureAlignment(Align(4));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 451-452
```cpp
  /// Remove dead constant pool entries.
  MadeChange |= removeUnusedCPEntries();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 454-467
```cpp
  // Iteratively place constant pool entries and fix up branches until there
  // is no change.
  unsigned NoCPIters = 0, NoBRIters = 0;
  while (true) {
    LLVM_DEBUG(dbgs() << "Beginning CP iteration #" << NoCPIters << '\n');
    bool CPChange = false;
    for (unsigned i = 0, e = CPUsers.size(); i != e; ++i)
      // For most inputs, it converges in no more than 5 iterations.
      // If it doesn't end in 10, the input may have huge BB or many CPEs.
      // In this case, we will try different heuristics.
      CPChange |= handleConstantPoolUser(i, NoCPIters >= CPMaxIteration / 2);
    if (CPChange && ++NoCPIters > CPMaxIteration)
      report_fatal_error("Constant Island pass failed to converge!");
    LLVM_DEBUG(dumpBBs());
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 469-471
```cpp
    // Clear NewWaterList now.  If we split a block for branches, it should
    // appear as "new water" for the next iteration of constant pool placement.
    NewWaterList.clear();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 473-481
```cpp
    LLVM_DEBUG(dbgs() << "Beginning BR iteration #" << NoBRIters << '\n');
    bool BRChange = false;
    for (unsigned i = 0, e = ImmBranches.size(); i != e; ++i) {
      // Note: fixupImmediateBr can append to ImmBranches.
      BRChange |= fixupImmediateBr(ImmBranches[i]);
    }
    if (BRChange && ++NoBRIters > 30)
      report_fatal_error("Branch Fix Up pass failed to converge!");
    LLVM_DEBUG(dumpBBs());
```
- EN: Implements `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 483-486
```cpp
    if (!CPChange && !BRChange)
      break;
    MadeChange = true;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 488-490
```cpp
  // Shrink 32-bit Thumb2 load and store instructions.
  if (isThumb2 && !STI->prefers32BitThumb())
    MadeChange |= optimizeThumb2Instructions();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 492-494
```cpp
  // Shrink 32-bit branch instructions.
  if (isThumb && STI->hasV8MBaselineOps())
    MadeChange |= optimizeThumb2Branches();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 496-498
```cpp
  // Optimize jump tables using TBB / TBH.
  if (GenerateTBB && !STI->genExecuteOnly())
    MadeChange |= optimizeThumb2JumpTables();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 500-501
```cpp
  // After a while, this might be made debug-only, but it is not expensive.
  verify();
```
- EN: Declares `verify`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `verify`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 503-510
```cpp
  // Save the mapping between original and cloned constpool entries.
  for (unsigned i = 0, e = CPEntries.size(); i != e; ++i) {
    for (unsigned j = 0, je = CPEntries[i].size(); j != je; ++j) {
      const CPEntry & CPE = CPEntries[i][j];
      if (CPE.CPEMI && CPE.CPEMI->getOperand(1).isCPI())
        AFI->recordCPEClone(i, CPE.CPI);
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 512-512
```cpp
  LLVM_DEBUG(dbgs() << '\n'; dumpBBs());
```
- EN: Declares `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 514-522
```cpp
  BBUtils->clear();
  WaterList.clear();
  CPUsers.clear();
  CPEntries.clear();
  JumpTableEntryIndices.clear();
  JumpTableUserIndices.clear();
  ImmBranches.clear();
  PushPopMIs.clear();
  T2JumpTables.clear();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 524-525
```cpp
  return MadeChange;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 527-533
```cpp
/// Perform the initial placement of the regular constant pool entries.
/// To start with, we put them all at the end of the function.
void
ARMConstantIslands::doInitialConstPlacement(std::vector<MachineInstr*> &CPEMIs) {
  // Create the basic block to hold the CPE's.
  MachineBasicBlock *BB = MF->CreateMachineBasicBlock();
  MF->push_back(BB);
```
- EN: Implements `ARMConstantIslands::doInitialConstPlacement`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMConstantIslands::doInitialConstPlacement`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 535-537
```cpp
  // MachineConstantPool measures alignment in bytes.
  const Align MaxAlign = MCP->getConstantPoolAlign();
  const unsigned MaxLogAlign = Log2(MaxAlign);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 539-540
```cpp
  // Mark the basic block as required by the const-pool.
  BB->setAlignment(MaxAlign);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 542-548
```cpp
  // The function needs to be as aligned as the basic blocks. The linker may
  // move functions around based on their alignment.
  // Special case: halfword literals still need word alignment on the function.
  Align FuncAlign = MaxAlign;
  if (MaxAlign == 2)
    FuncAlign = Align(4);
  MF->ensureAlignment(FuncAlign);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 550-555
```cpp
  // Order the entries in BB by descending alignment.  That ensures correct
  // alignment of all entries as long as BB is sufficiently aligned.  Keep
  // track of the insertion point for each alignment.  We are going to bucket
  // sort the entries as they are created.
  SmallVector<MachineBasicBlock::iterator, 8> InsPoint(MaxLogAlign + 1,
                                                       BB->end());
```
- EN: Declares `InsPoint`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `InsPoint`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 557-559
```cpp
  // Add all of the constants from the constant pool to the end block, use an
  // identity mapping of CPI's to CPE's.
  const std::vector<MachineConstantPoolEntry> &CPs = MCP->getConstants();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 561-567
```cpp
  const DataLayout &TD = MF->getDataLayout();
  for (unsigned i = 0, e = CPs.size(); i != e; ++i) {
    unsigned Size = CPs[i].getSizeInBytes(TD);
    Align Alignment = CPs[i].getAlign();
    // Verify that all constant pool entries are a multiple of their alignment.
    // If not, we would have to pad them out so that instructions stay aligned.
    assert(isAligned(Alignment, Size) && "CP Entry not multiple of 4 bytes!");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 569-575
```cpp
    // Insert CONSTPOOL_ENTRY before entries with a smaller alignment.
    unsigned LogAlign = Log2(Alignment);
    MachineBasicBlock::iterator InsAt = InsPoint[LogAlign];
    MachineInstr *CPEMI =
      BuildMI(*BB, InsAt, DebugLoc(), TII->get(ARM::CONSTPOOL_ENTRY))
        .addImm(i).addConstantPoolIndex(i).addImm(Size);
    CPEMIs.push_back(CPEMI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 577-581
```cpp
    // Ensure that future entries with higher alignment get inserted before
    // CPEMI. This is bucket sort with iterators.
    for (unsigned a = LogAlign + 1; a <= MaxLogAlign; ++a)
      if (InsPoint[a] == InsAt)
        InsPoint[a] = CPEMI;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 583-590
```cpp
    // Add a new CPEntry, but no corresponding CPUser yet.
    CPEntries.emplace_back(1, CPEntry(CPEMI, i));
    ++NumCPEs;
    LLVM_DEBUG(dbgs() << "Moved CPI#" << i << " to end of function, size = "
                      << Size << ", align = " << Alignment.value() << '\n');
  }
  LLVM_DEBUG(BB->dump());
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 592-601
```cpp
/// Do initial placement of the jump tables. Because Thumb2's TBB and TBH
/// instructions can be made more efficient if the jump table immediately
/// follows the instruction, it's best to place them immediately next to their
/// jumps to begin with. In almost all cases they'll never be moved from that
/// position.
void ARMConstantIslands::doInitialJumpTablePlacement(
    std::vector<MachineInstr *> &CPEMIs) {
  unsigned i = CPEntries.size();
  auto MJTI = MF->getJumpTableInfo();
  const std::vector<MachineJumpTableEntry> &JT = MJTI->getJumpTables();
```
- EN: Implements `ARMConstantIslands::doInitialJumpTablePlacement`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMConstantIslands::doInitialJumpTablePlacement`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 603-605
```cpp
  // Only inline jump tables are placed in the function.
  if (MJTI->getEntryKind() != MachineJumpTableInfo::EK_Inline)
    return;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 607-614
```cpp
  MachineBasicBlock *LastCorrectlyNumberedBB = nullptr;
  for (MachineBasicBlock &MBB : *MF) {
    auto MI = MBB.getLastNonDebugInstr();
    // Look past potential SpeculationBarriers at end of BB.
    while (MI != MBB.end() &&
           (isSpeculationBarrierEndBBOpcode(MI->getOpcode()) ||
            MI->isDebugInstr()))
      --MI;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 616-617
```cpp
    if (MI == MBB.end())
      continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 619-633
```cpp
    unsigned JTOpcode;
    switch (MI->getOpcode()) {
    default:
      continue;
    case ARM::BR_JTadd:
    case ARM::BR_JTr:
    case ARM::tBR_JTr:
    case ARM::BR_JTm_i12:
    case ARM::BR_JTm_rs:
      // These instructions are emitted only in ARM or Thumb1 modes which do not
      // support PACBTI. Hence we don't add BTI instructions in the destination
      // blocks.
      assert(!MF->getInfo<ARMFunctionInfo>()->branchTargetEnforcement() &&
             "Branch protection must not be enabled for Arm or Thumb1 modes");
      JTOpcode = ARM::JUMPTABLE_ADDRS;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 634-646
```cpp
      break;
    case ARM::t2BR_JT:
      JTOpcode = ARM::JUMPTABLE_INSTS;
      break;
    case ARM::tTBB_JT:
    case ARM::t2TBB_JT:
      JTOpcode = ARM::JUMPTABLE_TBB;
      break;
    case ARM::tTBH_JT:
    case ARM::t2TBH_JT:
      JTOpcode = ARM::JUMPTABLE_TBH;
      break;
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 648-662
```cpp
    unsigned NumOps = MI->getDesc().getNumOperands();
    MachineOperand JTOp =
      MI->getOperand(NumOps - (MI->isPredicable() ? 2 : 1));
    unsigned JTI = JTOp.getIndex();
    unsigned Size = JT[JTI].MBBs.size() * sizeof(uint32_t);
    MachineBasicBlock *JumpTableBB = MF->CreateMachineBasicBlock();
    MF->insert(std::next(MachineFunction::iterator(MBB)), JumpTableBB);
    MachineInstr *CPEMI = BuildMI(*JumpTableBB, JumpTableBB->begin(),
                                  DebugLoc(), TII->get(JTOpcode))
                              .addImm(i++)
                              .addJumpTableIndex(JTI)
                              .addImm(Size);
    CPEMIs.push_back(CPEMI);
    CPEntries.emplace_back(1, CPEntry(CPEMI, JTI));
    JumpTableEntryIndices.insert(std::make_pair(JTI, CPEntries.size() - 1));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 663-665
```cpp
    if (!LastCorrectlyNumberedBB)
      LastCorrectlyNumberedBB = &MBB;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 667-670
```cpp
  // If we did anything then we need to renumber the subsequent blocks.
  if (LastCorrectlyNumberedBB)
    MF->RenumberBlocks(LastCorrectlyNumberedBB);
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 672-679
```cpp
/// BBHasFallthrough - Return true if the specified basic block can fallthrough
/// into the block immediately after it.
bool ARMConstantIslands::BBHasFallthrough(MachineBasicBlock *MBB) {
  // Get the next machine basic block in the function.
  MachineFunction::iterator MBBI = MBB->getIterator();
  // Can't fall off end of function.
  if (std::next(MBBI) == MBB->getParent()->end())
    return false;
```
- EN: Implements `ARMConstantIslands::BBHasFallthrough`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMConstantIslands::BBHasFallthrough`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 681-683
```cpp
  MachineBasicBlock *NextBB = &*std::next(MBBI);
  if (!MBB->isSuccessor(NextBB))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 685-691
```cpp
  // Try to analyze the end of the block. A potential fallthrough may already
  // have an unconditional branch for whatever reason.
  MachineBasicBlock *TBB, *FBB;
  SmallVector<MachineOperand, 4> Cond;
  bool TooDifficult = TII->analyzeBranch(*MBB, TBB, FBB, Cond);
  return TooDifficult || FBB == nullptr;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 693-705
```cpp
/// findConstPoolEntry - Given the constpool index and CONSTPOOL_ENTRY MI,
/// look up the corresponding CPEntry.
ARMConstantIslands::CPEntry *
ARMConstantIslands::findConstPoolEntry(unsigned CPI,
                                       const MachineInstr *CPEMI) {
  std::vector<CPEntry> &CPEs = CPEntries[CPI];
  // Number of entries per constpool index should be small, just do a
  // linear search.
  for (CPEntry &CPE : CPEs)
    if (CPE.CPEMI == CPEMI)
      return &CPE;
  return nullptr;
}
```
- EN: Implements `ARMConstantIslands::findConstPoolEntry`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMConstantIslands::findConstPoolEntry`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 707-721
```cpp
/// getCPEAlign - Returns the required alignment of the constant pool entry
/// represented by CPEMI.
Align ARMConstantIslands::getCPEAlign(const MachineInstr *CPEMI) {
  switch (CPEMI->getOpcode()) {
  case ARM::CONSTPOOL_ENTRY:
    break;
  case ARM::JUMPTABLE_TBB:
    return isThumb1 ? Align(4) : Align(1);
  case ARM::JUMPTABLE_TBH:
    return isThumb1 ? Align(4) : Align(2);
  case ARM::JUMPTABLE_INSTS:
    return Align(2);
  case ARM::JUMPTABLE_ADDRS:
    return Align(4);
  default:
```
- EN: Implements `ARMConstantIslands::getCPEAlign`, a query/helper routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMConstantIslands::getCPEAlign`，它是一个围绕机器指令展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 722-723
```cpp
    llvm_unreachable("unknown constpool entry kind");
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 725-728
```cpp
  unsigned CPI = getCombinedIndex(CPEMI);
  assert(CPI < MCP->getConstants().size() && "Invalid constant pool index.");
  return MCP->getConstants()[CPI].getAlign();
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 730-740
```cpp
/// scanFunctionJumpTables - Do a scan of the function, building up
/// information about the sizes of each block and the locations of all
/// the jump tables.
void ARMConstantIslands::scanFunctionJumpTables() {
  for (MachineBasicBlock &MBB : *MF) {
    for (MachineInstr &I : MBB)
      if (I.isBranch() &&
          (I.getOpcode() == ARM::t2BR_JT || I.getOpcode() == ARM::tBR_JTr))
        T2JumpTables.push_back(&I);
  }
}
```
- EN: Implements `ARMConstantIslands::scanFunctionJumpTables`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMConstantIslands::scanFunctionJumpTables`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 742-746
```cpp
/// initializeFunctionInfo - Do the initial scan of the function, building up
/// information about the sizes of each block, the location of all the water,
/// and finding all of the constant pool users.
void ARMConstantIslands::
initializeFunctionInfo(const std::vector<MachineInstr*> &CPEMIs) {
```
- EN: Implements `initializeFunctionInfo`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `initializeFunctionInfo`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 748-752
```cpp
  BBUtils->computeAllBlockSizes();
  BBInfoVector &BBInfo = BBUtils->getBBInfo();
  // The known bits of the entry block offset are determined by the function
  // alignment.
  BBInfo.front().KnownBits = Log2(MF->getAlignment());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 754-755
```cpp
  // Compute block offsets and known bits.
  BBUtils->adjustBBOffsetsAfter(&MF->front());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 757-760
```cpp
  // We only care about jump table instructions when jump tables are inline.
  MachineJumpTableInfo *MJTI = MF->getJumpTableInfo();
  bool InlineJumpTables =
      MJTI && MJTI->getEntryKind() == MachineJumpTableInfo::EK_Inline;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 762-767
```cpp
  // Now go back through the instructions and build up our data structures.
  for (MachineBasicBlock &MBB : *MF) {
    // If this block doesn't fall through into the next MBB, then this is
    // 'water' that a constant pool island could be placed.
    if (!BBHasFallthrough(&MBB))
      WaterList.push_back(&MBB);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 769-771
```cpp
    for (MachineInstr &I : MBB) {
      if (I.isDebugInstr())
        continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 773-787
```cpp
      unsigned Opc = I.getOpcode();
      if (I.isBranch()) {
        bool isCond = false;
        unsigned Bits = 0;
        unsigned Scale = 1;
        int UOpc = Opc;
        switch (Opc) {
        default:
          continue;  // Ignore other JT branches
        case ARM::t2BR_JT:
        case ARM::tBR_JTr:
          if (InlineJumpTables)
            T2JumpTables.push_back(&I);
          continue;   // Does not get an entry in ImmBranches
        case ARM::Bcc:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 788-802
```cpp
          isCond = true;
          UOpc = ARM::B;
          [[fallthrough]];
        case ARM::B:
          Bits = 24;
          Scale = 4;
          break;
        case ARM::tBcc:
          isCond = true;
          UOpc = ARM::tB;
          Bits = 8;
          Scale = 2;
          break;
        case ARM::tB:
          Bits = 11;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 803-815
```cpp
          Scale = 2;
          break;
        case ARM::t2Bcc:
          isCond = true;
          UOpc = ARM::t2B;
          Bits = 20;
          Scale = 2;
          break;
        case ARM::t2B:
          Bits = 24;
          Scale = 2;
          break;
        }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 817-820
```cpp
        // Record this immediate branch.
        unsigned MaxOffs = ((1 << (Bits-1))-1) * Scale;
        ImmBranches.push_back(ImmBranch(&I, MaxOffs, isCond, UOpc));
      }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 822-823
```cpp
      if (Opc == ARM::tPUSH || Opc == ARM::tPOP_RET)
        PushPopMIs.push_back(&I);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 825-828
```cpp
      if (Opc == ARM::CONSTPOOL_ENTRY || Opc == ARM::JUMPTABLE_ADDRS ||
          Opc == ARM::JUMPTABLE_INSTS || Opc == ARM::JUMPTABLE_TBB ||
          Opc == ARM::JUMPTABLE_TBH)
        continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 830-835
```cpp
      // Scan the instructions for constant pool operands.
      for (unsigned op = 0, e = I.getNumOperands(); op != e; ++op)
        if (I.getOperand(op).isCPI() ||
            (I.getOperand(op).isJTI() && InlineJumpTables)) {
          // We found one.  The addressing mode tells us the max displacement
          // from the PC that this instruction permits.
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 837-841
```cpp
          // Basic size info comes from the TSFlags field.
          unsigned Bits = 0;
          unsigned Scale = 1;
          bool NegOk = false;
          bool IsSoImm = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 843-845
```cpp
          switch (Opc) {
          default:
            llvm_unreachable("Unknown addressing mode for CP reference!");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 847-861
```cpp
          // Taking the address of a CP entry.
          case ARM::LEApcrel:
          case ARM::LEApcrelJT: {
              // This takes a SoImm, which is 8 bit immediate rotated. We'll
              // pretend the maximum offset is 255 * 4. Since each instruction
              // 4 byte wide, this is always correct. We'll check for other
              // displacements that fits in a SoImm as well.
              Bits = 8;
              NegOk = true;
              IsSoImm = true;
              unsigned CPI = I.getOperand(op).getIndex();
              assert(CPI < CPEMIs.size());
              MachineInstr *CPEMI = CPEMIs[CPI];
              const Align CPEAlign = getCPEAlign(CPEMI);
              const unsigned LogCPEAlign = Log2(CPEAlign);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 862-876
```cpp
              if (LogCPEAlign >= 2)
                Scale = 4;
              else
                // For constants with less than 4-byte alignment,
                // we'll pretend the maximum offset is 255 * 1.
                Scale = 1;
            }
            break;
          case ARM::t2LEApcrel:
          case ARM::t2LEApcrelJT:
            Bits = 12;
            NegOk = true;
            break;
          case ARM::tLEApcrel:
          case ARM::tLEApcrelJT:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 877-879
```cpp
            Bits = 8;
            Scale = 4;
            break;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 881-891
```cpp
          case ARM::LDRBi12:
          case ARM::LDRi12:
          case ARM::LDRcp:
          case ARM::t2LDRpci:
          case ARM::t2LDRHpci:
          case ARM::t2LDRSHpci:
          case ARM::t2LDRBpci:
          case ARM::t2LDRSBpci:
            Bits = 12;  // +-offset_12
            NegOk = true;
            break;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 893-896
```cpp
          case ARM::tLDRpci:
            Bits = 8;
            Scale = 4;  // +(offset_8*4)
            break;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 898-909
```cpp
          case ARM::VLDRD:
          case ARM::VLDRS:
            Bits = 8;
            Scale = 4;  // +-(offset_8*4)
            NegOk = true;
            break;
          case ARM::VLDRH:
            Bits = 8;
            Scale = 2;  // +-(offset_8*2)
            NegOk = true;
            break;
          }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 911-916
```cpp
          // Remember that this is a user of a CP entry.
          unsigned CPI = I.getOperand(op).getIndex();
          if (I.getOperand(op).isJTI()) {
            JumpTableUserIndices.insert(std::make_pair(CPI, CPUsers.size()));
            CPI = JumpTableEntryIndices[CPI];
          }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 918-920
```cpp
          MachineInstr *CPEMI = CPEMIs[CPI];
          unsigned MaxOffs = ((1 << Bits)-1) * Scale;
          CPUsers.push_back(CPUser(&I, CPEMI, MaxOffs, NegOk, IsSoImm));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 922-925
```cpp
          // Increment corresponding CPEntry reference count.
          CPEntry *CPE = findConstPoolEntry(CPI, CPEMI);
          assert(CPE && "Cannot find a corresponding CPEntry!");
          CPE->RefCount++;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 927-933
```cpp
          // Instructions can only use one CP entry, don't bother scanning the
          // rest of the operands.
          break;
        }
    }
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 935-940
```cpp
/// CompareMBBNumbers - Little predicate function to sort the WaterList by MBB
/// ID.
static bool CompareMBBNumbers(const MachineBasicBlock *LHS,
                              const MachineBasicBlock *RHS) {
  return LHS->getNumber() < RHS->getNumber();
}
```
- EN: Implements `CompareMBBNumbers`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `CompareMBBNumbers`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 942-947
```cpp
/// updateForInsertedWaterBlock - When a block is newly inserted into the
/// machine function, it upsets all of the block numbers.  Renumber the blocks
/// and update the arrays that parallel this numbering.
void ARMConstantIslands::updateForInsertedWaterBlock(MachineBasicBlock *NewBB) {
  // Renumber the MBB's to keep them consecutive.
  NewBB->getParent()->RenumberBlocks(NewBB);
```
- EN: Implements `ARMConstantIslands::updateForInsertedWaterBlock`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMConstantIslands::updateForInsertedWaterBlock`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 949-951
```cpp
  // Insert an entry into BBInfo to align it properly with the (newly
  // renumbered) block numbers.
  BBUtils->insert(NewBB->getNumber(), BasicBlockInfo());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 953-957
```cpp
  // Next, update WaterList.  Specifically, we need to add NewMBB as having
  // available water after it.
  water_iterator IP = llvm::lower_bound(WaterList, NewBB, CompareMBBNumbers);
  WaterList.insert(IP, NewBB);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 959-963
```cpp
/// Split the basic block containing MI into two blocks, which are joined by
/// an unconditional branch.  Update data structures and renumber blocks to
/// account for this change and returns the newly created block.
MachineBasicBlock *ARMConstantIslands::splitBlockBeforeInstr(MachineInstr *MI) {
  MachineBasicBlock *OrigBB = MI->getParent();
```
- EN: Implements `ARMConstantIslands::splitBlockBeforeInstr`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMConstantIslands::splitBlockBeforeInstr`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 965-970
```cpp
  // Collect liveness information at MI.
  LivePhysRegs LRs(*MF->getSubtarget().getRegisterInfo());
  LRs.addLiveOuts(*OrigBB);
  auto LivenessEnd = ++MachineBasicBlock::iterator(MI).getReverse();
  for (MachineInstr &LiveMI : make_range(OrigBB->rbegin(), LivenessEnd))
    LRs.stepBackward(LiveMI);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 972-976
```cpp
  // Create a new MBB for the code after the OrigBB.
  MachineBasicBlock *NewBB =
    MF->CreateMachineBasicBlock(OrigBB->getBasicBlock());
  MachineFunction::iterator MBBI = ++OrigBB->getIterator();
  MF->insert(MBBI, NewBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 978-979
```cpp
  // Splice the instructions starting with MI over to NewBB.
  NewBB->splice(NewBB->end(), OrigBB, MI, OrigBB->end());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 981-992
```cpp
  // Add an unconditional branch from OrigBB to NewBB.
  // Note the new unconditional branch is not being recorded.
  // There doesn't seem to be meaningful DebugInfo available; this doesn't
  // correspond to anything in the source.
  unsigned Opc = isThumb ? (isThumb2 ? ARM::t2B : ARM::tB) : ARM::B;
  if (!isThumb)
    BuildMI(OrigBB, DebugLoc(), TII->get(Opc)).addMBB(NewBB);
  else
    BuildMI(OrigBB, DebugLoc(), TII->get(Opc))
        .addMBB(NewBB)
        .add(predOps(ARMCC::AL));
  ++NumSplit;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 994-995
```cpp
  // Update the CFG.  All succs of OrigBB are now succs of NewBB.
  NewBB->transferSuccessors(OrigBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 997-998
```cpp
  // OrigBB branches to NewBB.
  OrigBB->addSuccessor(NewBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1000-1004
```cpp
  // Update live-in information in the new block.
  MachineRegisterInfo &MRI = MF->getRegInfo();
  for (MCPhysReg L : LRs)
    if (!MRI.isReserved(L))
      NewBB->addLiveIn(L);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1006-1009
```cpp
  // Update internal data structures to account for the newly inserted MBB.
  // This is almost the same as updateForInsertedWaterBlock, except that
  // the Water goes after OrigBB, not NewBB.
  MF->RenumberBlocks(NewBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1011-1013
```cpp
  // Insert an entry into BBInfo to align it properly with the (newly
  // renumbered) block numbers.
  BBUtils->insert(NewBB->getNumber(), BasicBlockInfo());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1015-1025
```cpp
  // Next, update WaterList.  Specifically, we need to add OrigMBB as having
  // available water after it (but not if it's already there, which happens
  // when splitting before a conditional branch that is followed by an
  // unconditional branch - in that case we want to insert NewBB).
  water_iterator IP = llvm::lower_bound(WaterList, OrigBB, CompareMBBNumbers);
  MachineBasicBlock* WaterBB = *IP;
  if (WaterBB == OrigBB)
    WaterList.insert(std::next(IP), NewBB);
  else
    WaterList.insert(IP, OrigBB);
  NewWaterList.insert(OrigBB);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1027-1032
```cpp
  // Figure out how large the OrigBB is.  As the first half of the original
  // block, it cannot contain a tablejump.  The size includes
  // the new jump we added.  (It should be possible to do this without
  // recounting everything, but it's very confusing, and this is rarely
  // executed.)
  BBUtils->computeBlockSize(OrigBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1034-1036
```cpp
  // Figure out how large the NewMBB is.  As the second half of the original
  // block, it may contain a tablejump.
  BBUtils->computeBlockSize(NewBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1038-1039
```cpp
  // All BBOffsets following these blocks must be modified.
  BBUtils->adjustBBOffsetsAfter(OrigBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1041-1042
```cpp
  return NewBB;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1044-1048
```cpp
/// getUserOffset - Compute the offset of U.MI as seen by the hardware
/// displacement computation.  Update U.KnownAlignment to match its current
/// basic block location.
unsigned ARMConstantIslands::getUserOffset(CPUser &U) const {
  unsigned UserOffset = BBUtils->getOffsetOf(U.MI);
```
- EN: Implements `ARMConstantIslands::getUserOffset`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMConstantIslands::getUserOffset`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1050-1052
```cpp
  SmallVectorImpl<BasicBlockInfo> &BBInfo = BBUtils->getBBInfo();
  const BasicBlockInfo &BBI = BBInfo[U.MI->getParent()->getNumber()];
  unsigned KnownBits = BBI.internalKnownBits();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1054-1055
```cpp
  // The value read from PC is offset from the actual instruction address.
  UserOffset += (isThumb ? 4 : 8);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1057-1059
```cpp
  // Because of inline assembly, we may not know the alignment (mod 4) of U.MI.
  // Make sure U.getMaxDisp() returns a constrained range.
  U.KnownAlignment = (KnownBits >= 2);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1061-1065
```cpp
  // On Thumb, offsets==2 mod 4 are rounded down by the hardware for
  // purposes of the displacement computation; compensate for that here.
  // For unknown alignments, getMaxDisp() constrains the range instead.
  if (isThumb && U.KnownAlignment)
    UserOffset &= ~3u;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1067-1068
```cpp
  return UserOffset;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1070-1084
```cpp
/// isOffsetInRange - Checks whether UserOffset (the location of a constant pool
/// reference) is within MaxDisp of TrialOffset (a proposed location of a
/// constant pool entry).
/// UserOffset is computed by getUserOffset above to include PC adjustments. If
/// the mod 4 alignment of UserOffset is not known, the uncertainty must be
/// subtracted from MaxDisp instead. CPUser::getMaxDisp() does that.
bool ARMConstantIslands::isOffsetInRange(unsigned UserOffset,
                                         unsigned TrialOffset, unsigned MaxDisp,
                                         bool NegativeOK, bool IsSoImm) {
  if (UserOffset <= TrialOffset) {
    // User before the Trial.
    if (TrialOffset - UserOffset <= MaxDisp)
      return true;
    // FIXME: Make use full range of soimm values.
  } else if (NegativeOK) {
```
- EN: Implements `ARMConstantIslands::isOffsetInRange`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMConstantIslands::isOffsetInRange`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1085-1090
```cpp
    if (UserOffset - TrialOffset <= MaxDisp)
      return true;
    // FIXME: Make use full range of soimm values.
  }
  return false;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1092-1106
```cpp
/// isWaterInRange - Returns true if a CPE placed after the specified
/// Water (a basic block) will be in range for the specific MI.
///
/// Compute how much the function will grow by inserting a CPE after Water.
bool ARMConstantIslands::isWaterInRange(unsigned UserOffset,
                                        MachineBasicBlock* Water, CPUser &U,
                                        unsigned &Growth) {
  BBInfoVector &BBInfo = BBUtils->getBBInfo();
  const Align CPEAlign = getCPEAlign(U.CPEMI);
  const unsigned CPEOffset = BBInfo[Water->getNumber()].postOffset(CPEAlign);
  unsigned NextBlockOffset;
  Align NextBlockAlignment;
  MachineFunction::const_iterator NextBlock = Water->getIterator();
  if (++NextBlock == MF->end()) {
    NextBlockOffset = BBInfo[Water->getNumber()].postOffset();
```
- EN: Implements `ARMConstantIslands::isWaterInRange`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMConstantIslands::isWaterInRange`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1107-1112
```cpp
  } else {
    NextBlockOffset = BBInfo[NextBlock->getNumber()].Offset;
    NextBlockAlignment = NextBlock->getAlignment();
  }
  unsigned Size = U.CPEMI->getOperand(2).getImm();
  unsigned CPEEnd = CPEOffset + Size;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1114-1121
```cpp
  // The CPE may be able to hide in the alignment padding before the next
  // block. It may also cause more padding to be required if it is more aligned
  // that the next block.
  if (CPEEnd > NextBlockOffset) {
    Growth = CPEEnd - NextBlockOffset;
    // Compute the padding that would go at the end of the CPE to align the next
    // block.
    Growth += offsetToAlignment(CPEEnd, NextBlockAlignment);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1123-1130
```cpp
    // If the CPE is to be inserted before the instruction, that will raise
    // the offset of the instruction. Also account for unknown alignment padding
    // in blocks between CPE and the user.
    if (CPEOffset < UserOffset)
      UserOffset += Growth + UnknownPadding(MF->getAlignment(), Log2(CPEAlign));
  } else
    // CPE fits in existing padding.
    Growth = 0;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1132-1133
```cpp
  return isOffsetInRange(UserOffset, CPEOffset, U);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1135-1140
```cpp
/// isCPEntryInRange - Returns true if the distance between specific MI and
/// specific ConstPool entry instruction can fit in MI's displacement field.
bool ARMConstantIslands::isCPEntryInRange(MachineInstr *MI, unsigned UserOffset,
                                      MachineInstr *CPEMI, unsigned MaxDisp,
                                      bool NegOk, bool DoDump) {
  unsigned CPEOffset = BBUtils->getOffsetOf(CPEMI);
```
- EN: Implements `ARMConstantIslands::isCPEntryInRange`, a query/helper routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMConstantIslands::isCPEntryInRange`，它是一个围绕机器指令展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1142-1155
```cpp
  if (DoDump) {
    LLVM_DEBUG({
        BBInfoVector &BBInfo = BBUtils->getBBInfo();
      unsigned Block = MI->getParent()->getNumber();
      const BasicBlockInfo &BBI = BBInfo[Block];
      dbgs() << "User of CPE#" << CPEMI->getOperand(0).getImm()
             << " max delta=" << MaxDisp
             << format(" insn address=%#x", UserOffset) << " in "
             << printMBBReference(*MI->getParent()) << ": "
             << format("%#x-%x\t", BBI.Offset, BBI.postOffset()) << *MI
             << format("CPE address=%#x offset=%+d: ", CPEOffset,
                       int(CPEOffset - UserOffset));
    });
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1157-1158
```cpp
  return isOffsetInRange(UserOffset, CPEOffset, MaxDisp, NegOk);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1160-1165
```cpp
#ifndef NDEBUG
/// BBIsJumpedOver - Return true of the specified basic block's only predecessor
/// unconditionally branches to its only successor.
static bool BBIsJumpedOver(MachineBasicBlock *MBB) {
  if (MBB->pred_size() != 1 || MBB->succ_size() != 1)
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1167-1175
```cpp
  MachineBasicBlock *Succ = *MBB->succ_begin();
  MachineBasicBlock *Pred = *MBB->pred_begin();
  MachineInstr *PredMI = &Pred->back();
  if (PredMI->getOpcode() == ARM::B || PredMI->getOpcode() == ARM::tB
      || PredMI->getOpcode() == ARM::t2B)
    return PredMI->getOperand(0).getMBB() == Succ;
  return false;
}
#endif // NDEBUG
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1177-1191
```cpp
/// decrementCPEReferenceCount - find the constant pool entry with index CPI
/// and instruction CPEMI, and decrement its refcount.  If the refcount
/// becomes 0 remove the entry and instruction.  Returns true if we removed
/// the entry, false if we didn't.
bool ARMConstantIslands::decrementCPEReferenceCount(unsigned CPI,
                                                    MachineInstr *CPEMI) {
  // Find the old entry. Eliminate it if it is no longer used.
  CPEntry *CPE = findConstPoolEntry(CPI, CPEMI);
  assert(CPE && "Unexpected!");
  if (--CPE->RefCount == 0) {
    removeDeadCPEMI(CPEMI);
    CPE->CPEMI = nullptr;
    --NumCPEs;
    return true;
  }
```
- EN: Implements `ARMConstantIslands::decrementCPEReferenceCount`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMConstantIslands::decrementCPEReferenceCount`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1192-1193
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1195-1197
```cpp
unsigned ARMConstantIslands::getCombinedIndex(const MachineInstr *CPEMI) {
  if (CPEMI->getOperand(1).isCPI())
    return CPEMI->getOperand(1).getIndex();
```
- EN: Implements `ARMConstantIslands::getCombinedIndex`, a query/helper routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMConstantIslands::getCombinedIndex`，它是一个围绕机器指令展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1199-1200
```cpp
  return JumpTableEntryIndices[CPEMI->getOperand(1).getIndex()];
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1202-1210
```cpp
/// LookForCPEntryInRange - see if the currently referenced CPE is in range;
/// if not, see if an in-range clone of the CPE is in range, and if so,
/// change the data structures so the user references the clone.  Returns:
/// 0 = no existing entry found
/// 1 = entry found, and there were no code insertions or deletions
/// 2 = entry found, and there were code insertions or deletions
int ARMConstantIslands::findInRangeCPEntry(CPUser& U, unsigned UserOffset) {
  MachineInstr *UserMI = U.MI;
  MachineInstr *CPEMI  = U.CPEMI;
```
- EN: Implements `ARMConstantIslands::findInRangeCPEntry`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMConstantIslands::findInRangeCPEntry`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1212-1217
```cpp
  // Check to see if the CPE is already in-range.
  if (isCPEntryInRange(UserMI, UserOffset, CPEMI, U.getMaxDisp(), U.NegOk,
                       true)) {
    LLVM_DEBUG(dbgs() << "In range\n");
    return 1;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1219-1233
```cpp
  // No.  Look for previously created clones of the CPE that are in range.
  unsigned CPI = getCombinedIndex(CPEMI);
  std::vector<CPEntry> &CPEs = CPEntries[CPI];
  for (CPEntry &CPE : CPEs) {
    // We already tried this one
    if (CPE.CPEMI == CPEMI)
      continue;
    // Removing CPEs can leave empty entries, skip
    if (CPE.CPEMI == nullptr)
      continue;
    if (isCPEntryInRange(UserMI, UserOffset, CPE.CPEMI, U.getMaxDisp(),
                         U.NegOk)) {
      LLVM_DEBUG(dbgs() << "Replacing CPE#" << CPI << " with CPE#" << CPE.CPI
                        << "\n");
      // Point the CPUser node to the replacement
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1234-1248
```cpp
      U.CPEMI = CPE.CPEMI;
      // Change the CPI in the instruction operand to refer to the clone.
      for (MachineOperand &MO : UserMI->operands())
        if (MO.isCPI()) {
          MO.setIndex(CPE.CPI);
          break;
        }
      // Adjust the refcount of the clone...
      CPE.RefCount++;
      // ...and the original.  If we didn't remove the old entry, none of the
      // addresses changed, so we don't need another pass.
      return decrementCPEReferenceCount(CPI, CPEMI) ? 2 : 1;
    }
  }
  return 0;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1249-1249
```cpp
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1251-1261
```cpp
/// getUnconditionalBrDisp - Returns the maximum displacement that can fit in
/// the specific unconditional branch instruction.
static inline unsigned getUnconditionalBrDisp(int Opc) {
  switch (Opc) {
  case ARM::tB:
    return ((1<<10)-1)*2;
  case ARM::t2B:
    return ((1<<23)-1)*2;
  default:
    break;
  }
```
- EN: Implements `getUnconditionalBrDisp`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getUnconditionalBrDisp`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1263-1264
```cpp
  return ((1<<23)-1)*4;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1266-1278
```cpp
/// findAvailableWater - Look for an existing entry in the WaterList in which
/// we can place the CPE referenced from U so it's within range of U's MI.
/// Returns true if found, false if not.  If it returns true, WaterIter
/// is set to the WaterList entry.  For Thumb, prefer water that will not
/// introduce padding to water that will.  To ensure that this pass
/// terminates, the CPE location for a particular CPUser is only allowed to
/// move to a lower address, so search backward from the end of the list and
/// prefer the first water that is in range.
bool ARMConstantIslands::findAvailableWater(CPUser &U, unsigned UserOffset,
                                            water_iterator &WaterIter,
                                            bool CloserWater) {
  if (WaterList.empty())
    return false;
```
- EN: Implements `ARMConstantIslands::findAvailableWater`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMConstantIslands::findAvailableWater`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1280-1294
```cpp
  unsigned BestGrowth = ~0u;
  // The nearest water without splitting the UserBB is right after it.
  // If the distance is still large (we have a big BB), then we need to split it
  // if we don't converge after certain iterations. This helps the following
  // situation to converge:
  //   BB0:
  //      Big BB
  //   BB1:
  //      Constant Pool
  // When a CP access is out of range, BB0 may be used as water. However,
  // inserting islands between BB0 and BB1 makes other accesses out of range.
  MachineBasicBlock *UserBB = U.MI->getParent();
  BBInfoVector &BBInfo = BBUtils->getBBInfo();
  const Align CPEAlign = getCPEAlign(U.CPEMI);
  unsigned MinNoSplitDisp = BBInfo[UserBB->getNumber()].postOffset(CPEAlign);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1295-1309
```cpp
  if (CloserWater && MinNoSplitDisp > U.getMaxDisp() / 2)
    return false;
  for (water_iterator IP = std::prev(WaterList.end()), B = WaterList.begin();;
       --IP) {
    MachineBasicBlock* WaterBB = *IP;
    // Check if water is in range and is either at a lower address than the
    // current "high water mark" or a new water block that was created since
    // the previous iteration by inserting an unconditional branch.  In the
    // latter case, we want to allow resetting the high water mark back to
    // this new water since we haven't seen it before.  Inserting branches
    // should be relatively uncommon and when it does happen, we want to be
    // sure to take advantage of it for all the CPEs near that block, so that
    // we don't insert more branches than necessary.
    // When CloserWater is true, we try to find the lowest address after (or
    // equal to) user MI's BB no matter of padding growth.
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1310-1319
```cpp
    unsigned Growth;
    if (isWaterInRange(UserOffset, WaterBB, U, Growth) &&
        (WaterBB->getNumber() < U.HighWaterMark->getNumber() ||
         NewWaterList.count(WaterBB) || WaterBB == U.MI->getParent()) &&
        Growth < BestGrowth) {
      // This is the least amount of required padding seen so far.
      BestGrowth = Growth;
      WaterIter = IP;
      LLVM_DEBUG(dbgs() << "Found water after " << printMBBReference(*WaterBB)
                        << " Growth=" << Growth << '\n');
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1321-1332
```cpp
      if (CloserWater && WaterBB == U.MI->getParent())
        return true;
      // Keep looking unless it is perfect and we're not looking for the lowest
      // possible address.
      if (!CloserWater && BestGrowth == 0)
        return true;
    }
    if (IP == B)
      break;
  }
  return BestGrowth != ~0u;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1334-1348
```cpp
/// createNewWater - No existing WaterList entry will work for
/// CPUsers[CPUserIndex], so create a place to put the CPE.  The end of the
/// block is used if in range, and the conditional branch munged so control
/// flow is correct.  Otherwise the block is split to create a hole with an
/// unconditional branch around it.  In either case NewMBB is set to a
/// block following which the new island can be inserted (the WaterList
/// is not adjusted).
void ARMConstantIslands::createNewWater(unsigned CPUserIndex,
                                        unsigned UserOffset,
                                        MachineBasicBlock *&NewMBB) {
  CPUser &U = CPUsers[CPUserIndex];
  MachineInstr *UserMI = U.MI;
  MachineInstr *CPEMI  = U.CPEMI;
  const Align CPEAlign = getCPEAlign(CPEMI);
  MachineBasicBlock *UserMBB = UserMI->getParent();
```
- EN: Implements `ARMConstantIslands::createNewWater`, a query/helper routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMConstantIslands::createNewWater`，它是一个围绕机器基本块展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1349-1350
```cpp
  BBInfoVector &BBInfo = BBUtils->getBBInfo();
  const BasicBlockInfo &UserBBI = BBInfo[UserMBB->getNumber()];
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1352-1360
```cpp
  // If the block does not end in an unconditional branch already, and if the
  // end of the block is within range, make new water there.  (The addition
  // below is for the unconditional branch we will be adding: 4 bytes on ARM +
  // Thumb2, 2 on Thumb1.
  if (BBHasFallthrough(UserMBB)) {
    // Size of branch to insert.
    unsigned Delta = isThumb1 ? 2 : 4;
    // Compute the offset where the CPE will begin.
    unsigned CPEOffset = UserBBI.postOffset(CPEAlign) + Delta;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1362-1376
```cpp
    if (isOffsetInRange(UserOffset, CPEOffset, U)) {
      LLVM_DEBUG(dbgs() << "Split at end of " << printMBBReference(*UserMBB)
                        << format(", expected CPE offset %#x\n", CPEOffset));
      NewMBB = &*++UserMBB->getIterator();
      // Add an unconditional branch from UserMBB to fallthrough block.  Record
      // it for branch lengthening; this new branch will not get out of range,
      // but if the preceding conditional branch is out of range, the targets
      // will be exchanged, and the altered branch may be out of range, so the
      // machinery has to know about it.
      int UncondBr = isThumb ? ((isThumb2) ? ARM::t2B : ARM::tB) : ARM::B;
      if (!isThumb)
        BuildMI(UserMBB, DebugLoc(), TII->get(UncondBr)).addMBB(NewMBB);
      else
        BuildMI(UserMBB, DebugLoc(), TII->get(UncondBr))
            .addMBB(NewMBB)
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1377-1385
```cpp
            .add(predOps(ARMCC::AL));
      unsigned MaxDisp = getUnconditionalBrDisp(UncondBr);
      ImmBranches.push_back(ImmBranch(&UserMBB->back(),
                                      MaxDisp, false, UncondBr));
      BBUtils->computeBlockSize(UserMBB);
      BBUtils->adjustBBOffsetsAfter(UserMBB);
      return;
    }
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1387-1397
```cpp
  // What a big block.  Find a place within the block to split it.  This is a
  // little tricky on Thumb1 since instructions are 2 bytes and constant pool
  // entries are 4 bytes: if instruction I references island CPE, and
  // instruction I+1 references CPE', it will not work well to put CPE as far
  // forward as possible, since then CPE' cannot immediately follow it (that
  // location is 2 bytes farther away from I+1 than CPE was from I) and we'd
  // need to create a new island.  So, we make a first guess, then walk through
  // the instructions between the one currently being looked at and the
  // possible insertion point, and make sure any other instructions that
  // reference CPEs will be able to use the same island area; if not, we back
  // up the insertion point.
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 1399-1408
```cpp
  // Try to split the block so it's fully aligned.  Compute the latest split
  // point where we can add a 4-byte branch instruction, and then align to
  // Align which is the largest possible alignment in the function.
  const Align Align = MF->getAlignment();
  assert(Align >= CPEAlign && "Over-aligned constant pool entry");
  unsigned KnownBits = UserBBI.internalKnownBits();
  unsigned UPad = UnknownPadding(Align, KnownBits);
  unsigned BaseInsertOffset = UserOffset + U.getMaxDisp() - UPad;
  LLVM_DEBUG(dbgs() << format("Split in middle of big block before %#x",
                              BaseInsertOffset));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1410-1413
```cpp
  // The 4 in the following is for the unconditional branch we'll be inserting
  // (allows for long branch on Thumb1).  Alignment of the island is handled
  // inside isOffsetInRange.
  BaseInsertOffset -= 4;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1415-1417
```cpp
  LLVM_DEBUG(dbgs() << format(", adjusted to %#x", BaseInsertOffset)
                    << " la=" << Log2(Align) << " kb=" << KnownBits
                    << " up=" << UPad << '\n');
```
- EN: Declares `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1419-1433
```cpp
  // This could point off the end of the block if we've already got constant
  // pool entries following this block; only the last one is in the water list.
  // Back past any possible branches (allow for a conditional and a maximally
  // long unconditional).
  if (BaseInsertOffset + 8 >= UserBBI.postOffset()) {
    // Ensure BaseInsertOffset is larger than the offset of the instruction
    // following UserMI so that the loop which searches for the split point
    // iterates at least once.
    BaseInsertOffset =
        std::max(UserBBI.postOffset() - UPad - 8,
                 UserOffset + TII->getInstSizeInBytes(*UserMI) + 1);
    // If the CP is referenced(ie, UserOffset) is in first four instructions
    // after IT, this recalculated BaseInsertOffset could be in the middle of
    // an IT block. If it is, change the BaseInsertOffset to just after the
    // IT block. This still make the CP Entry is in range because of the
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1434-1448
```cpp
    // following reasons.
    //   1. The initial BaseseInsertOffset calculated is (UserOffset +
    //   U.getMaxDisp() - UPad).
    //   2. An IT block is only at most 4 instructions plus the "it" itself (18
    //   bytes).
    //   3. All the relevant instructions support much larger Maximum
    //   displacement.
    MachineBasicBlock::iterator I = UserMI;
    ++I;
    Register PredReg;
    for (unsigned Offset = UserOffset + TII->getInstSizeInBytes(*UserMI);
         I->getOpcode() != ARM::t2IT &&
         getITInstrPredicate(*I, PredReg) != ARMCC::AL;
         Offset += TII->getInstSizeInBytes(*I), I = std::next(I)) {
      BaseInsertOffset =
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1449-1463
```cpp
          std::max(BaseInsertOffset, Offset + TII->getInstSizeInBytes(*I) + 1);
      assert(I != UserMBB->end() && "Fell off end of block");
    }
    LLVM_DEBUG(dbgs() << format("Move inside block: %#x\n", BaseInsertOffset));
  }
  unsigned EndInsertOffset = BaseInsertOffset + 4 + UPad +
    CPEMI->getOperand(2).getImm();
  MachineBasicBlock::iterator MI = UserMI;
  ++MI;
  unsigned CPUIndex = CPUserIndex+1;
  unsigned NumCPUsers = CPUsers.size();
  MachineInstr *LastIT = nullptr;
  for (unsigned Offset = UserOffset + TII->getInstSizeInBytes(*UserMI);
       Offset < BaseInsertOffset;
       Offset += TII->getInstSizeInBytes(*MI), MI = std::next(MI)) {
```
- EN: Declares `std::max`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `std::max`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1464-1478
```cpp
    assert(MI != UserMBB->end() && "Fell off end of block");
    if (CPUIndex < NumCPUsers && CPUsers[CPUIndex].MI == &*MI) {
      CPUser &U = CPUsers[CPUIndex];
      if (!isOffsetInRange(Offset, EndInsertOffset, U)) {
        // Shift intertion point by one unit of alignment so it is within reach.
        BaseInsertOffset -= Align.value();
        EndInsertOffset -= Align.value();
      }
      // This is overly conservative, as we don't account for CPEMIs being
      // reused within the block, but it doesn't matter much.  Also assume CPEs
      // are added in order with alignment padding.  We may eventually be able
      // to pack the aligned CPEs better.
      EndInsertOffset += U.CPEMI->getOperand(2).getImm();
      CPUIndex++;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1480-1483
```cpp
    // Remember the last IT instruction.
    if (MI->getOpcode() == ARM::t2IT)
      LastIT = &*MI;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1485-1485
```cpp
  --MI;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1487-1493
```cpp
  // Avoid splitting an IT block.
  if (LastIT) {
    Register PredReg;
    ARMCC::CondCodes CC = getITInstrPredicate(*MI, PredReg);
    if (CC != ARMCC::AL)
      MI = LastIT;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1495-1509
```cpp
  // Avoid splitting a MOVW+MOVT pair with a relocation on Windows.
  // On Windows, this instruction pair is covered by one single
  // IMAGE_REL_ARM_MOV32T relocation which covers both instructions. If a
  // constant island is injected inbetween them, the relocation will clobber
  // the instruction and fail to update the MOVT instruction.
  // (These instructions are bundled up until right before the ConstantIslands
  // pass.)
  if (STI->isTargetWindows() && isThumb && MI->getOpcode() == ARM::t2MOVTi16 &&
      (MI->getOperand(2).getTargetFlags() & ARMII::MO_OPTION_MASK) ==
          ARMII::MO_HI16) {
    --MI;
    assert(MI->getOpcode() == ARM::t2MOVi16 &&
           (MI->getOperand(1).getTargetFlags() & ARMII::MO_OPTION_MASK) ==
               ARMII::MO_LO16);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1511-1517
```cpp
  // We really must not split an IT block.
#ifndef NDEBUG
  Register PredReg;
  assert(!isThumb || getITInstrPredicate(*MI, PredReg) == ARMCC::AL);
#endif
  NewMBB = splitBlockBeforeInstr(&*MI);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1519-1531
```cpp
/// handleConstantPoolUser - Analyze the specified user, checking to see if it
/// is out-of-range.  If so, pick up the constant pool value and move it some
/// place in-range.  Return true if we changed any addresses (thus must run
/// another pass of branch lengthening), false otherwise.
bool ARMConstantIslands::handleConstantPoolUser(unsigned CPUserIndex,
                                                bool CloserWater) {
  CPUser &U = CPUsers[CPUserIndex];
  MachineInstr *UserMI = U.MI;
  MachineInstr *CPEMI  = U.CPEMI;
  unsigned CPI = getCombinedIndex(CPEMI);
  unsigned Size = CPEMI->getOperand(2).getImm();
  // Compute this only once, it's expensive.
  unsigned UserOffset = getUserOffset(U);
```
- EN: Implements `ARMConstantIslands::handleConstantPoolUser`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMConstantIslands::handleConstantPoolUser`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1533-1537
```cpp
  // See if the current entry is within range, or there is a clone of it
  // in range.
  int result = findInRangeCPEntry(U, UserOffset);
  if (result==1) return false;
  else if (result==2) return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1539-1541
```cpp
  // No existing clone of this CPE is within range.
  // We will be generating a new clone.  Get a UID for it.
  unsigned ID = AFI->createPICLabelUId();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1543-1549
```cpp
  // Look for water where we can place this CPE.
  MachineBasicBlock *NewIsland = MF->CreateMachineBasicBlock();
  MachineBasicBlock *NewMBB;
  water_iterator IP;
  if (findAvailableWater(U, UserOffset, IP, CloserWater)) {
    LLVM_DEBUG(dbgs() << "Found water in range\n");
    MachineBasicBlock *WaterBB = *IP;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1551-1555
```cpp
    // If the original WaterList entry was "new water" on this iteration,
    // propagate that to the new island.  This is just keeping NewWaterList
    // updated to match the WaterList, which will be updated below.
    if (NewWaterList.erase(WaterBB))
      NewWaterList.insert(NewIsland);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1557-1562
```cpp
    // The new CPE goes before the following block (NewMBB).
    NewMBB = &*++WaterBB->getIterator();
  } else {
    // No water found.
    LLVM_DEBUG(dbgs() << "No water found\n");
    createNewWater(CPUserIndex, UserOffset, NewMBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1564-1572
```cpp
    // splitBlockBeforeInstr adds to WaterList, which is important when it is
    // called while handling branches so that the water will be seen on the
    // next iteration for constant pools, but in this context, we don't want
    // it.  Check for this so it will be removed from the WaterList.
    // Also remove any entry from NewWaterList.
    MachineBasicBlock *WaterBB = &*--NewMBB->getIterator();
    IP = find(WaterList, WaterBB);
    if (IP != WaterList.end())
      NewWaterList.erase(WaterBB);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1574-1582
```cpp
    // We are adding new water.  Update NewWaterList.
    NewWaterList.insert(NewIsland);
  }
  // Always align the new block because CP entries can be smaller than 4
  // bytes. Be careful not to decrease the existing alignment, e.g. NewMBB may
  // be an already aligned constant pool block.
  const Align Alignment = isThumb ? Align(2) : Align(4);
  if (NewMBB->getAlignment() < Alignment)
    NewMBB->setAlignment(Alignment);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1584-1589
```cpp
  // Remove the original WaterList entry; we want subsequent insertions in
  // this vicinity to go after the one we're about to insert.  This
  // considerably reduces the number of times we have to move the same CPE
  // more than once and is also important to ensure the algorithm terminates.
  if (IP != WaterList.end())
    WaterList.erase(IP);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1591-1592
```cpp
  // Okay, we know we can put an island before NewMBB now, do it!
  MF->insert(NewMBB->getIterator(), NewIsland);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1594-1595
```cpp
  // Update internal data structures to account for the newly inserted MBB.
  updateForInsertedWaterBlock(NewIsland);
```
- EN: Declares `updateForInsertedWaterBlock`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `updateForInsertedWaterBlock`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1597-1605
```cpp
  // Now that we have an island to add the CPE to, clone the original CPE and
  // add it to the island.
  U.HighWaterMark = NewIsland;
  U.CPEMI = BuildMI(NewIsland, DebugLoc(), CPEMI->getDesc())
                .addImm(ID)
                .add(CPEMI->getOperand(1))
                .addImm(Size);
  CPEntries[CPI].push_back(CPEntry(U.CPEMI, ID, 1));
  ++NumCPEs;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1607-1608
```cpp
  // Decrement the old entry, and remove it if refcount becomes 0.
  decrementCPEReferenceCount(CPI, CPEMI);
```
- EN: Declares `decrementCPEReferenceCount`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `decrementCPEReferenceCount`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1610-1611
```cpp
  // Mark the basic block as aligned as required by the const-pool entry.
  NewIsland->setAlignment(getCPEAlign(U.CPEMI));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1613-1615
```cpp
  // Increase the size of the island block to account for the new entry.
  BBUtils->adjustBBSize(NewIsland, Size);
  BBUtils->adjustBBOffsetsAfter(&*--NewIsland->getIterator());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1617-1622
```cpp
  // Finally, change the CPI in the instruction operand to be ID.
  for (MachineOperand &MO : UserMI->operands())
    if (MO.isCPI()) {
      MO.setIndex(ID);
      break;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1624-1627
```cpp
  LLVM_DEBUG(
      dbgs() << "  Moved CPE to #" << ID << " CPI=" << CPI
             << format(" offset=%#x\n",
                       BBUtils->getBBInfo()[NewIsland->getNumber()].Offset));
```
- EN: Declares `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1629-1630
```cpp
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1632-1642
```cpp
/// removeDeadCPEMI - Remove a dead constant pool entry instruction. Update
/// sizes and offsets of impacted basic blocks.
void ARMConstantIslands::removeDeadCPEMI(MachineInstr *CPEMI) {
  MachineBasicBlock *CPEBB = CPEMI->getParent();
  unsigned Size = CPEMI->getOperand(2).getImm();
  CPEMI->eraseFromParent();
  BBInfoVector &BBInfo = BBUtils->getBBInfo();
  BBUtils->adjustBBSize(CPEBB, -Size);
  // All succeeding offsets have the current size value added in, fix this.
  if (CPEBB->empty()) {
    BBInfo[CPEBB->getNumber()].Size = 0;
```
- EN: Implements `ARMConstantIslands::removeDeadCPEMI`, a mutation/build routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMConstantIslands::removeDeadCPEMI`，它是一个围绕机器基本块展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1644-1649
```cpp
    // This block no longer needs to be aligned.
    CPEBB->setAlignment(Align(1));
  } else {
    // Entries are sorted by descending alignment, so realign from the front.
    CPEBB->setAlignment(getCPEAlign(&*CPEBB->begin()));
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1651-1657
```cpp
  BBUtils->adjustBBOffsetsAfter(CPEBB);
  // An island has only one predecessor BB and one successor BB. Check if
  // this BB's predecessor jumps directly to this BB's successor. This
  // shouldn't happen currently.
  assert(!BBIsJumpedOver(CPEBB) && "How did this happen?");
  // FIXME: remove the empty blocks after all the work is done?
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1659-1673
```cpp
/// removeUnusedCPEntries - Remove constant pool entries whose refcounts
/// are zero.
bool ARMConstantIslands::removeUnusedCPEntries() {
  unsigned MadeChange = false;
  for (std::vector<CPEntry> &CPEs : CPEntries) {
    for (CPEntry &CPE : CPEs) {
      if (CPE.RefCount == 0 && CPE.CPEMI) {
        removeDeadCPEMI(CPE.CPEMI);
        CPE.CPEMI = nullptr;
        MadeChange = true;
      }
    }
  }
  return MadeChange;
}
```
- EN: Implements `ARMConstantIslands::removeUnusedCPEntries`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMConstantIslands::removeUnusedCPEntries`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1676-1680
```cpp
/// fixupImmediateBr - Fix up an immediate branch whose destination is too far
/// away to fit in its displacement field.
bool ARMConstantIslands::fixupImmediateBr(ImmBranch &Br) {
  MachineInstr *MI = Br.MI;
  MachineBasicBlock *DestBB = MI->getOperand(0).getMBB();
```
- EN: Implements `ARMConstantIslands::fixupImmediateBr`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMConstantIslands::fixupImmediateBr`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1682-1684
```cpp
  // Check to see if the DestBB is already in-range.
  if (BBUtils->isBBInRange(MI, DestBB, Br.MaxDisp))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1686-1689
```cpp
  if (!Br.isCond)
    return fixupUnconditionalBr(Br);
  return fixupConditionalBr(Br);
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1691-1700
```cpp
/// fixupUnconditionalBr - Fix up an unconditional branch whose destination is
/// too far away to fit in its displacement field. If the LR register has been
/// spilled in the epilogue, then we can use BL to implement a far jump.
/// Otherwise, add an intermediate branch instruction to a branch.
bool
ARMConstantIslands::fixupUnconditionalBr(ImmBranch &Br) {
  MachineInstr *MI = Br.MI;
  MachineBasicBlock *MBB = MI->getParent();
  if (!isThumb1)
    llvm_unreachable("fixupUnconditionalBr is Thumb1 only!");
```
- EN: Implements `ARMConstantIslands::fixupUnconditionalBr`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMConstantIslands::fixupUnconditionalBr`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1702-1703
```cpp
  if (!AFI->isLRSpilled())
    report_fatal_error("underestimated function size");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1705-1711
```cpp
  // Use BL to implement far jump.
  Br.MaxDisp = (1 << 21) * 2;
  MI->setDesc(TII->get(ARM::tBfar));
  BBInfoVector &BBInfo = BBUtils->getBBInfo();
  BBInfo[MBB->getNumber()].Size += 2;
  BBUtils->adjustBBOffsetsAfter(MBB);
  ++NumUBrFixed;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1713-1713
```cpp
  LLVM_DEBUG(dbgs() << "  Changed B to long jump " << *MI);
```
- EN: Declares `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1715-1716
```cpp
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1718-1724
```cpp
/// fixupConditionalBr - Fix up a conditional branch whose destination is too
/// far away to fit in its displacement field. It is converted to an inverse
/// conditional branch + an unconditional branch to the destination.
bool
ARMConstantIslands::fixupConditionalBr(ImmBranch &Br) {
  MachineInstr *MI = Br.MI;
  MachineBasicBlock *DestBB = MI->getOperand(0).getMBB();
```
- EN: Implements `ARMConstantIslands::fixupConditionalBr`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMConstantIslands::fixupConditionalBr`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1726-1735
```cpp
  // Add an unconditional branch to the destination and invert the branch
  // condition to jump over it:
  // blt L1
  // =>
  // bge L2
  // b   L1
  // L2:
  ARMCC::CondCodes CC = (ARMCC::CondCodes)MI->getOperand(1).getImm();
  CC = ARMCC::getOppositeCondition(CC);
  Register CCReg = MI->getOperand(2).getReg();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1737-1742
```cpp
  // If the branch is at the end of its MBB and that has a fall-through block,
  // direct the updated conditional branch to the fall-through block. Otherwise,
  // split the MBB before the next instruction.
  MachineBasicBlock *MBB = MI->getParent();
  MachineInstr *BMI = &MBB->back();
  bool NeedSplit = (BMI != MI) || !BBHasFallthrough(MBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1744-1758
```cpp
  ++NumCBrFixed;
  if (BMI != MI) {
    if (std::next(MachineBasicBlock::iterator(MI)) == std::prev(MBB->end()) &&
        BMI->getOpcode() == Br.UncondBr) {
      // Last MI in the BB is an unconditional branch. Can we simply invert the
      // condition and swap destinations:
      // beq L1
      // b   L2
      // =>
      // bne L2
      // b   L1
      MachineBasicBlock *NewDest = BMI->getOperand(0).getMBB();
      if (BBUtils->isBBInRange(MI, NewDest, Br.MaxDisp)) {
        LLVM_DEBUG(
            dbgs() << "  Invert Bcc condition and swap its destination with "
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1759-1766
```cpp
                   << *BMI);
        BMI->getOperand(0).setMBB(DestBB);
        MI->getOperand(0).setMBB(NewDest);
        MI->getOperand(1).setImm(CC);
        return true;
      }
    }
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1768-1774
```cpp
  if (NeedSplit) {
    splitBlockBeforeInstr(MI);
    // No need for the branch to the next block. We're adding an unconditional
    // branch to the destination.
    int delta = TII->getInstSizeInBytes(MBB->back());
    BBUtils->adjustBBSize(MBB, -delta);
    MBB->back().eraseFromParent();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1776-1779
```cpp
    // The conditional successor will be swapped between the BBs after this, so
    // update CFG.
    MBB->addSuccessor(DestBB);
    std::next(MBB->getIterator())->removeSuccessor(DestBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1781-1783
```cpp
    // BBInfo[SplitBB].Offset is wrong temporarily, fixed below
  }
  MachineBasicBlock *NextBB = &*++MBB->getIterator();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1785-1787
```cpp
  LLVM_DEBUG(dbgs() << "  Insert B to " << printMBBReference(*DestBB)
                    << " also invert condition and change dest. to "
                    << printMBBReference(*NextBB) << "\n");
```
- EN: Declares `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1789-1803
```cpp
  // Insert a new conditional branch and a new unconditional branch.
  // Also update the ImmBranch as well as adding a new entry for the new branch.
  BuildMI(MBB, DebugLoc(), TII->get(MI->getOpcode()))
    .addMBB(NextBB).addImm(CC).addReg(CCReg);
  Br.MI = &MBB->back();
  BBUtils->adjustBBSize(MBB, TII->getInstSizeInBytes(MBB->back()));
  if (isThumb)
    BuildMI(MBB, DebugLoc(), TII->get(Br.UncondBr))
        .addMBB(DestBB)
        .add(predOps(ARMCC::AL));
  else
    BuildMI(MBB, DebugLoc(), TII->get(Br.UncondBr)).addMBB(DestBB);
  BBUtils->adjustBBSize(MBB, TII->getInstSizeInBytes(MBB->back()));
  unsigned MaxDisp = getUnconditionalBrDisp(Br.UncondBr);
  ImmBranches.push_back(ImmBranch(&MBB->back(), MaxDisp, false, Br.UncondBr));
```
- EN: Declares `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1805-1810
```cpp
  // Remove the old conditional branch.  It may or may not still be in MBB.
  BBUtils->adjustBBSize(MI->getParent(), -TII->getInstSizeInBytes(*MI));
  MI->eraseFromParent();
  BBUtils->adjustBBOffsetsAfter(MBB);
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1812-1813
```cpp
bool ARMConstantIslands::optimizeThumb2Instructions() {
  bool MadeChange = false;
```
- EN: Implements `ARMConstantIslands::optimizeThumb2Instructions`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMConstantIslands::optimizeThumb2Instructions`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1815-1829
```cpp
  // Shrink ADR and LDR from constantpool.
  for (CPUser &U : CPUsers) {
    unsigned Opcode = U.MI->getOpcode();
    unsigned NewOpc = 0;
    unsigned Scale = 1;
    unsigned Bits = 0;
    switch (Opcode) {
    default: break;
    case ARM::t2LEApcrel:
      if (isARMLowRegister(U.MI->getOperand(0).getReg())) {
        NewOpc = ARM::tLEApcrel;
        Bits = 8;
        Scale = 4;
      }
      break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1830-1837
```cpp
    case ARM::t2LDRpci:
      if (isARMLowRegister(U.MI->getOperand(0).getReg())) {
        NewOpc = ARM::tLDRpci;
        Bits = 8;
        Scale = 4;
      }
      break;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1839-1840
```cpp
    if (!NewOpc)
      continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1842-1843
```cpp
    unsigned UserOffset = getUserOffset(U);
    unsigned MaxOffs = ((1 << Bits) - 1) * Scale;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1845-1847
```cpp
    // Be conservative with inline asm.
    if (!U.KnownAlignment)
      MaxOffs -= 2;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1849-1859
```cpp
    // FIXME: Check if offset is multiple of scale if scale is not 4.
    if (isCPEntryInRange(U.MI, UserOffset, U.CPEMI, MaxOffs, false, true)) {
      LLVM_DEBUG(dbgs() << "Shrink: " << *U.MI);
      U.MI->setDesc(TII->get(NewOpc));
      MachineBasicBlock *MBB = U.MI->getParent();
      BBUtils->adjustBBSize(MBB, -2);
      BBUtils->adjustBBOffsetsAfter(MBB);
      ++NumT2CPShrunk;
      MadeChange = true;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1861-1862
```cpp
  return MadeChange;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1865-1865
```cpp
bool ARMConstantIslands::optimizeThumb2Branches() {
```
- EN: Implements `ARMConstantIslands::optimizeThumb2Branches`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMConstantIslands::optimizeThumb2Branches`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1867-1881
```cpp
  auto TryShrinkBranch = [this](ImmBranch &Br) {
    unsigned Opcode = Br.MI->getOpcode();
    unsigned NewOpc = 0;
    unsigned Scale = 1;
    unsigned Bits = 0;
    switch (Opcode) {
    default: break;
    case ARM::t2B:
      NewOpc = ARM::tB;
      Bits = 11;
      Scale = 2;
      break;
    case ARM::t2Bcc:
      NewOpc = ARM::tBcc;
      Bits = 8;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1882-1896
```cpp
      Scale = 2;
      break;
    }
    if (NewOpc) {
      unsigned MaxOffs = ((1 << (Bits-1))-1) * Scale;
      MachineBasicBlock *DestBB = Br.MI->getOperand(0).getMBB();
      if (BBUtils->isBBInRange(Br.MI, DestBB, MaxOffs)) {
        LLVM_DEBUG(dbgs() << "Shrink branch: " << *Br.MI);
        Br.MI->setDesc(TII->get(NewOpc));
        MachineBasicBlock *MBB = Br.MI->getParent();
        BBUtils->adjustBBSize(MBB, -2);
        BBUtils->adjustBBOffsetsAfter(MBB);
        ++NumT2BrShrunk;
        return true;
      }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1897-1899
```cpp
    }
    return false;
  };
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1901-1904
```cpp
  struct ImmCompare {
    MachineInstr* MI = nullptr;
    unsigned NewOpc = 0;
  };
```
- EN: Declares `ImmCompare`, packaging target-specific state and APIs around `ARMConstantIslandPass`.
- CN: 这里声明 `ImmCompare`，把与 `ARMConstantIslandPass` 相关的目标特定状态和 API 组织在一起。

### Lines 1906-1909
```cpp
  auto FindCmpForCBZ = [this](ImmBranch &Br, ImmCompare &ImmCmp,
                              MachineBasicBlock *DestBB) {
    ImmCmp.MI = nullptr;
    ImmCmp.NewOpc = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1911-1914
```cpp
    // If the conditional branch doesn't kill CPSR, then CPSR can be liveout
    // so this transformation is not safe.
    if (!Br.MI->killsRegister(ARM::CPSR, /*TRI=*/nullptr))
      return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1916-1924
```cpp
    Register PredReg;
    unsigned NewOpc = 0;
    ARMCC::CondCodes Pred = getInstrPredicate(*Br.MI, PredReg);
    if (Pred == ARMCC::EQ)
      NewOpc = ARM::tCBZ;
    else if (Pred == ARMCC::NE)
      NewOpc = ARM::tCBNZ;
    else
      return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1926-1932
```cpp
    // Check if the distance is within 126. Subtract starting offset by 2
    // because the cmp will be eliminated.
    unsigned BrOffset = BBUtils->getOffsetOf(Br.MI) + 4 - 2;
    BBInfoVector &BBInfo = BBUtils->getBBInfo();
    unsigned DestOffset = BBInfo[DestBB->getNumber()].Offset;
    if (BrOffset >= DestOffset || (DestOffset - BrOffset) > 126)
      return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1934-1938
```cpp
    // Search backwards to find a tCMPi8
    auto *TRI = STI->getRegisterInfo();
    MachineInstr *CmpMI = findCMPToFoldIntoCBZ(Br.MI, TRI);
    if (!CmpMI || CmpMI->getOpcode() != ARM::tCMPi8)
      return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1940-1943
```cpp
    ImmCmp.MI = CmpMI;
    ImmCmp.NewOpc = NewOpc;
    return true;
  };
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1945-1948
```cpp
  auto TryConvertToLE = [this](ImmBranch &Br, ImmCompare &Cmp) {
    if (Br.MI->getOpcode() != ARM::t2Bcc || !STI->hasLOB() ||
        STI->hasMinSize())
      return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1950-1954
```cpp
    MachineBasicBlock *MBB = Br.MI->getParent();
    MachineBasicBlock *DestBB = Br.MI->getOperand(0).getMBB();
    if (BBUtils->getOffsetOf(MBB) < BBUtils->getOffsetOf(DestBB) ||
        !BBUtils->isBBInRange(Br.MI, DestBB, 4094))
      return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1956-1957
```cpp
    if (!DT->dominates(DestBB, MBB))
      return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1959-1961
```cpp
    // We queried for the CBN?Z opcode based upon the 'ExitBB', the opposite
    // target of Br. So now we need to reverse the condition.
    Cmp.NewOpc = Cmp.NewOpc == ARM::tCBZ ? ARM::tCBNZ : ARM::tCBZ;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1963-1971
```cpp
    MachineInstrBuilder MIB = BuildMI(*MBB, Br.MI, Br.MI->getDebugLoc(),
                                      TII->get(ARM::t2LE));
    // Swapped a t2Bcc for a t2LE, so no need to update the size of the block.
    MIB.add(Br.MI->getOperand(0));
    Br.MI->eraseFromParent();
    Br.MI = MIB;
    ++NumLEInserted;
    return true;
  };
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1973-1973
```cpp
  bool MadeChange = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1975-1985
```cpp
  // The order in which branches appear in ImmBranches is approximately their
  // order within the function body. By visiting later branches first, we reduce
  // the distance between earlier forward branches and their targets, making it
  // more likely that the cbn?z optimization, which can only apply to forward
  // branches, will succeed.
  for (ImmBranch &Br : reverse(ImmBranches)) {
    MachineBasicBlock *DestBB = Br.MI->getOperand(0).getMBB();
    MachineBasicBlock *MBB = Br.MI->getParent();
    MachineBasicBlock *ExitBB = &MBB->back() == Br.MI ?
      MBB->getFallThrough() :
      MBB->back().getOperand(0).getMBB();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1987-1994
```cpp
    ImmCompare Cmp;
    if (FindCmpForCBZ(Br, Cmp, ExitBB) && TryConvertToLE(Br, Cmp)) {
      DestBB = ExitBB;
      MadeChange = true;
    } else {
      FindCmpForCBZ(Br, Cmp, DestBB);
      MadeChange |= TryShrinkBranch(Br);
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1996-1998
```cpp
    unsigned Opcode = Br.MI->getOpcode();
    if ((Opcode != ARM::tBcc && Opcode != ARM::t2LE) || !Cmp.NewOpc)
      continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2000-2000
```cpp
    Register Reg = Cmp.MI->getOperand(0).getReg();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2002-2014
```cpp
    // Check for Kill flags on Reg. If they are present remove them and set kill
    // on the new CBZ.
    auto *TRI = STI->getRegisterInfo();
    MachineBasicBlock::iterator KillMI = Br.MI;
    bool RegKilled = false;
    do {
      --KillMI;
      if (KillMI->killsRegister(Reg, TRI)) {
        KillMI->clearRegisterKills(Reg, TRI);
        RegKilled = true;
        break;
      }
    } while (KillMI != Cmp.MI);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2016-2022
```cpp
    // Create the new CBZ/CBNZ
    LLVM_DEBUG(dbgs() << "Fold: " << *Cmp.MI << " and: " << *Br.MI);
    MachineInstr *NewBR =
        BuildMI(*MBB, Br.MI, Br.MI->getDebugLoc(), TII->get(Cmp.NewOpc))
            .addReg(Reg, getKillRegState(RegKilled) |
                             getRegState(Cmp.MI->getOperand(0)))
            .addMBB(DestBB, Br.MI->getOperand(0).getTargetFlags());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2024-2024
```cpp
    Cmp.MI->eraseFromParent();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2026-2040
```cpp
    if (Br.MI->getOpcode() == ARM::tBcc) {
      Br.MI->eraseFromParent();
      Br.MI = NewBR;
      BBUtils->adjustBBSize(MBB, -2);
    } else if (MBB->back().getOpcode() != ARM::t2LE) {
      // An LE has been generated, but it's not the terminator - that is an
      // unconditional branch. However, the logic has now been reversed with the
      // CBN?Z being the conditional branch and the LE being the unconditional
      // branch. So this means we can remove the redundant unconditional branch
      // at the end of the block.
      MachineInstr *LastMI = &MBB->back();
      BBUtils->adjustBBSize(MBB, -LastMI->getDesc().getSize());
      LastMI->eraseFromParent();
    }
    BBUtils->adjustBBOffsetsAfter(MBB);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2041-2043
```cpp
    ++NumCBZ;
    MadeChange = true;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2045-2046
```cpp
  return MadeChange;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2048-2051
```cpp
static bool isSimpleIndexCalc(MachineInstr &I, unsigned EntryReg,
                              unsigned BaseReg) {
  if (I.getOpcode() != ARM::t2ADDrs)
    return false;
```
- EN: Implements `isSimpleIndexCalc`, a query/helper routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isSimpleIndexCalc`，它是一个围绕机器指令展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2053-2054
```cpp
  if (I.getOperand(0).getReg() != EntryReg)
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2056-2057
```cpp
  if (I.getOperand(1).getReg() != BaseReg)
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2059-2061
```cpp
  // FIXME: what about CC and IdxReg?
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2063-2075
```cpp
/// While trying to form a TBB/TBH instruction, we may (if the table
/// doesn't immediately follow the BR_JT) need access to the start of the
/// jump-table. We know one instruction that produces such a register; this
/// function works out whether that definition can be preserved to the BR_JT,
/// possibly by removing an intervening addition (which is usually needed to
/// calculate the actual entry to jump to).
bool ARMConstantIslands::preserveBaseRegister(MachineInstr *JumpMI,
                                              MachineInstr *LEAMI,
                                              unsigned &DeadSize,
                                              bool &CanDeleteLEA,
                                              bool &BaseRegKill) {
  if (JumpMI->getParent() != LEAMI->getParent())
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2077-2091
```cpp
  // Now we hope that we have at least these instructions in the basic block:
  //     BaseReg = t2LEA ...
  //     [...]
  //     EntryReg = t2ADDrs BaseReg, ...
  //     [...]
  //     t2BR_JT EntryReg
  //
  // We have to be very conservative about what we recognise here though. The
  // main perturbing factors to watch out for are:
  //    + Spills at any point in the chain: not direct problems but we would
  //      expect a blocking Def of the spilled register so in practice what we
  //      can do is limited.
  //    + EntryReg == BaseReg: this is the one situation we should allow a Def
  //      of BaseReg, but only if the t2ADDrs can be removed.
  //    + Some instruction other than t2ADDrs computing the entry. Not seen in
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 2092-2094
```cpp
  //      the wild, but we should be careful.
  Register EntryReg = JumpMI->getOperand(0).getReg();
  Register BaseReg = LEAMI->getOperand(0).getReg();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2096-2104
```cpp
  CanDeleteLEA = true;
  BaseRegKill = false;
  MachineInstr *RemovableAdd = nullptr;
  MachineBasicBlock::iterator I(LEAMI);
  for (++I; &*I != JumpMI; ++I) {
    if (isSimpleIndexCalc(*I, EntryReg, BaseReg)) {
      RemovableAdd = &*I;
      break;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2106-2116
```cpp
    for (const MachineOperand &MO : I->operands()) {
      if (!MO.isReg() || !MO.getReg())
        continue;
      if (MO.isDef() && MO.getReg() == BaseReg)
        return false;
      if (MO.isUse() && MO.getReg() == BaseReg) {
        BaseRegKill = BaseRegKill || MO.isKill();
        CanDeleteLEA = false;
      }
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2118-2119
```cpp
  if (!RemovableAdd)
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2121-2132
```cpp
  // Check the add really is removable, and that nothing else in the block
  // clobbers BaseReg.
  for (++I; &*I != JumpMI; ++I) {
    for (const MachineOperand &MO : I->operands()) {
      if (!MO.isReg() || !MO.getReg())
        continue;
      if (MO.isDef() && MO.getReg() == BaseReg)
        return false;
      if (MO.isUse() && MO.getReg() == EntryReg)
        RemovableAdd = nullptr;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2134-2141
```cpp
  if (RemovableAdd) {
    RemovableAdd->eraseFromParent();
    DeadSize += isThumb2 ? 4 : 2;
  } else if (BaseReg == EntryReg) {
    // The add wasn't removable, but clobbered the base for the TBB. So we can't
    // preserve it.
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2143-2147
```cpp
  // We reached the end of the block without seeing another definition of
  // BaseReg (except, possibly the t2ADDrs, which was removed). BaseReg can be
  // used in the TBB/TBH if necessary.
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2149-2156
```cpp
/// Returns whether CPEMI is the first instruction in the block
/// immediately following JTMI (assumed to be a TBB or TBH terminator). If so,
/// we can switch the first register to PC and usually remove the address
/// calculation that preceded it.
static bool jumpTableFollowsTB(MachineInstr *JTMI, MachineInstr *CPEMI) {
  MachineFunction::iterator MBB = JTMI->getParent()->getIterator();
  MachineFunction *MF = MBB->getParent();
  ++MBB;
```
- EN: Implements `jumpTableFollowsTB`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `jumpTableFollowsTB`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2158-2159
```cpp
  return MBB != MF->end() && !MBB->empty() && &*MBB->begin() == CPEMI;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2161-2168
```cpp
static void RemoveDeadAddBetweenLEAAndJT(MachineInstr *LEAMI,
                                         MachineInstr *JumpMI,
                                         unsigned &DeadSize) {
  // Remove a dead add between the LEA and JT, which used to compute EntryReg,
  // but the JT now uses PC. Finds the last ADD (if any) that def's EntryReg
  // and is not clobbered / used.
  MachineInstr *RemovableAdd = nullptr;
  Register EntryReg = JumpMI->getOperand(0).getReg();
```
- EN: Implements `RemoveDeadAddBetweenLEAAndJT`, a mutation/build routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `RemoveDeadAddBetweenLEAAndJT`，它是一个围绕机器指令展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2170-2175
```cpp
  // Find the last ADD to set EntryReg
  MachineBasicBlock::iterator I(LEAMI);
  for (++I; &*I != JumpMI; ++I) {
    if (I->getOpcode() == ARM::t2ADDrs && I->getOperand(0).getReg() == EntryReg)
      RemovableAdd = &*I;
  }
```
- EN: Implements `I`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `I`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2177-2178
```cpp
  if (!RemovableAdd)
    return;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2180-2191
```cpp
  // Ensure EntryReg is not clobbered or used.
  MachineBasicBlock::iterator J(RemovableAdd);
  for (++J; &*J != JumpMI; ++J) {
    for (const MachineOperand &MO : J->operands()) {
      if (!MO.isReg() || !MO.getReg())
        continue;
      if (MO.isDef() && MO.getReg() == EntryReg)
        return;
      if (MO.isUse() && MO.getReg() == EntryReg)
        return;
    }
  }
```
- EN: Implements `J`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `J`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2193-2196
```cpp
  LLVM_DEBUG(dbgs() << "Removing Dead Add: " << *RemovableAdd);
  RemovableAdd->eraseFromParent();
  DeadSize += 4;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2198-2201
```cpp
/// optimizeThumb2JumpTables - Use tbb / tbh instructions to generate smaller
/// jumptables when it's possible.
bool ARMConstantIslands::optimizeThumb2JumpTables() {
  bool MadeChange = false;
```
- EN: Implements `ARMConstantIslands::optimizeThumb2JumpTables`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMConstantIslands::optimizeThumb2JumpTables`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2203-2206
```cpp
  // FIXME: After the tables are shrunk, can we get rid some of the
  // constantpool tables?
  MachineJumpTableInfo *MJTI = MF->getJumpTableInfo();
  if (!MJTI) return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2208-2215
```cpp
  const std::vector<MachineJumpTableEntry> &JT = MJTI->getJumpTables();
  for (MachineInstr *MI : T2JumpTables) {
    const MCInstrDesc &MCID = MI->getDesc();
    unsigned NumOps = MCID.getNumOperands();
    unsigned JTOpIdx = NumOps - (MI->isPredicable() ? 2 : 1);
    MachineOperand JTOP = MI->getOperand(JTOpIdx);
    unsigned JTI = JTOP.getIndex();
    assert(JTI < JT.size());
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2217-2231
```cpp
    bool ByteOk = true;
    bool HalfWordOk = true;
    unsigned JTOffset = BBUtils->getOffsetOf(MI) + 4;
    const std::vector<MachineBasicBlock*> &JTBBs = JT[JTI].MBBs;
    BBInfoVector &BBInfo = BBUtils->getBBInfo();
    for (MachineBasicBlock *MBB : JTBBs) {
      unsigned DstOffset = BBInfo[MBB->getNumber()].Offset;
      // Negative offset is not ok. FIXME: We should change BB layout to make
      // sure all the branches are forward.
      if (ByteOk && (DstOffset - JTOffset) > ((1<<8)-1)*2)
        ByteOk = false;
      unsigned TBHLimit = ((1<<16)-1)*2;
      if (HalfWordOk && (DstOffset - JTOffset) > TBHLimit)
        HalfWordOk = false;
      if (!ByteOk && !HalfWordOk)
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2232-2233
```cpp
        break;
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2235-2236
```cpp
    if (!ByteOk && !HalfWordOk)
      continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2238-2241
```cpp
    CPUser &User = CPUsers[JumpTableUserIndices[JTI]];
    MachineBasicBlock *MBB = MI->getParent();
    if (!MI->getOperand(0).isKill()) // FIXME: needed now?
      continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2243-2245
```cpp
    unsigned DeadSize = 0;
    bool CanDeleteLEA = false;
    bool BaseRegKill = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2247-2251
```cpp
    unsigned IdxReg = ~0U;
    bool IdxRegKill = true;
    if (isThumb2) {
      IdxReg = MI->getOperand(1).getReg();
      IdxRegKill = MI->getOperand(1).isKill();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2253-2262
```cpp
      bool PreservedBaseReg =
        preserveBaseRegister(MI, User.MI, DeadSize, CanDeleteLEA, BaseRegKill);
      if (!jumpTableFollowsTB(MI, User.CPEMI) && !PreservedBaseReg)
        continue;
    } else {
      // We're in thumb-1 mode, so we must have something like:
      //   %idx = tLSLri %idx, 2
      //   %base = tLEApcrelJT
      //   %t = tLDRr %base, %idx
      Register BaseReg = User.MI->getOperand(0).getReg();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2264-2267
```cpp
      MachineBasicBlock *UserMBB = User.MI->getParent();
      MachineBasicBlock::iterator Shift = User.MI->getIterator();
      if (Shift == UserMBB->begin())
        continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2269-2275
```cpp
      Shift = prev_nodbg(Shift, UserMBB->begin());
      if (Shift->getOpcode() != ARM::tLSLri ||
          Shift->getOperand(3).getImm() != 2 ||
          !Shift->getOperand(2).isKill())
        continue;
      IdxReg = Shift->getOperand(2).getReg();
      Register ShiftedIdxReg = Shift->getOperand(0).getReg();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2277-2281
```cpp
      // It's important that IdxReg is live until the actual TBB/TBH. Most of
      // the range is checked later, but the LEA might still clobber it and not
      // actually get removed.
      if (BaseReg == IdxReg && !jumpTableFollowsTB(MI, User.CPEMI))
        continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2283-2289
```cpp
      MachineInstr *Load = User.MI->getNextNode();
      if (Load->getOpcode() != ARM::tLDRr)
        continue;
      if (Load->getOperand(1).getReg() != BaseReg ||
          Load->getOperand(2).getReg() != ShiftedIdxReg ||
          !Load->getOperand(2).isKill())
        continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2291-2292
```cpp
      // If we're in PIC mode, there should be another ADD following.
      auto *TRI = STI->getRegisterInfo();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2294-2300
```cpp
      // %base cannot be redefined after the load as it will appear before
      // TBB/TBH like:
      //      %base =
      //      %base =
      //      tBB %base, %idx
      if (registerDefinedBetween(BaseReg, Load->getNextNode(), MBB->end(), TRI))
        continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2302-2316
```cpp
      if (isPositionIndependentOrROPI) {
        MachineInstr *Add = Load->getNextNode();
        if (Add->getOpcode() != ARM::tADDrr ||
            Add->getOperand(2).getReg() != BaseReg ||
            Add->getOperand(3).getReg() != Load->getOperand(0).getReg() ||
            !Add->getOperand(3).isKill())
          continue;
        if (Add->getOperand(0).getReg() != MI->getOperand(0).getReg())
          continue;
        if (registerDefinedBetween(IdxReg, Add->getNextNode(), MI, TRI))
          // IdxReg gets redefined in the middle of the sequence.
          continue;
        Add->eraseFromParent();
        DeadSize += 2;
      } else {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2317-2322
```cpp
        if (Load->getOperand(0).getReg() != MI->getOperand(0).getReg())
          continue;
        if (registerDefinedBetween(IdxReg, Load->getNextNode(), MI, TRI))
          // IdxReg gets redefined in the middle of the sequence.
          continue;
      }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2324-2329
```cpp
      // Now safe to delete the load and lsl. The LEA will be removed later.
      CanDeleteLEA = true;
      Shift->eraseFromParent();
      Load->eraseFromParent();
      DeadSize += 4;
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2331-2335
```cpp
    LLVM_DEBUG(dbgs() << "Shrink JT: " << *MI);
    MachineInstr *CPEMI = User.CPEMI;
    unsigned Opc = ByteOk ? ARM::t2TBB_JT : ARM::t2TBH_JT;
    if (!isThumb2)
      Opc = ByteOk ? ARM::tTBB_JT : ARM::tTBH_JT;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2337-2345
```cpp
    MachineBasicBlock::iterator MI_JT = MI;
    MachineInstr *NewJTMI =
        BuildMI(*MBB, MI_JT, MI->getDebugLoc(), TII->get(Opc))
            .addReg(User.MI->getOperand(0).getReg(),
                    getKillRegState(BaseRegKill))
            .addReg(IdxReg, getKillRegState(IdxRegKill))
            .addJumpTableIndex(JTI, JTOP.getTargetFlags())
            .addImm(CPEMI->getOperand(0).getImm());
    LLVM_DEBUG(dbgs() << printMBBReference(*MBB) << ": " << *NewJTMI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2347-2348
```cpp
    unsigned JTOpc = ByteOk ? ARM::JUMPTABLE_TBB : ARM::JUMPTABLE_TBH;
    CPEMI->setDesc(TII->get(JTOpc));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2350-2352
```cpp
    if (jumpTableFollowsTB(MI, User.CPEMI)) {
      NewJTMI->getOperand(0).setReg(ARM::PC);
      NewJTMI->getOperand(0).setIsKill(false);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2354-2356
```cpp
      if (CanDeleteLEA) {
        if (isThumb2)
          RemoveDeadAddBetweenLEAAndJT(User.MI, MI, DeadSize);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2358-2359
```cpp
        User.MI->eraseFromParent();
        DeadSize += isThumb2 ? 4 : 2;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2361-2375
```cpp
        // The LEA was eliminated, the TBB instruction becomes the only new user
        // of the jump table.
        User.MI = NewJTMI;
        User.MaxDisp = 4;
        User.NegOk = false;
        User.IsSoImm = false;
        User.KnownAlignment = false;
      } else {
        // The LEA couldn't be eliminated, so we must add another CPUser to
        // record the TBB or TBH use.
        int CPEntryIdx = JumpTableEntryIndices[JTI];
        auto &CPEs = CPEntries[CPEntryIdx];
        auto Entry =
            find_if(CPEs, [&](CPEntry &E) { return E.CPEMI == User.CPEMI; });
        ++Entry->RefCount;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2376-2378
```cpp
        CPUsers.emplace_back(CPUser(NewJTMI, User.CPEMI, 4, false, false));
      }
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2380-2382
```cpp
    unsigned NewSize = TII->getInstSizeInBytes(*NewJTMI);
    unsigned OrigSize = TII->getInstSizeInBytes(*MI);
    MI->eraseFromParent();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2384-2386
```cpp
    int Delta = OrigSize - NewSize + DeadSize;
    BBInfo[MBB->getNumber()].Size -= Delta;
    BBUtils->adjustBBOffsetsAfter(MBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2388-2390
```cpp
    ++NumTBs;
    MadeChange = true;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2392-2393
```cpp
  return MadeChange;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2395-2398
```cpp
/// reorderThumb2JumpTables - Adjust the function's block layout to ensure that
/// jump tables always branch forwards, since that's what tbb and tbh need.
bool ARMConstantIslands::reorderThumb2JumpTables() {
  bool MadeChange = false;
```
- EN: Implements `ARMConstantIslands::reorderThumb2JumpTables`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMConstantIslands::reorderThumb2JumpTables`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2400-2401
```cpp
  MachineJumpTableInfo *MJTI = MF->getJumpTableInfo();
  if (!MJTI) return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2403-2410
```cpp
  const std::vector<MachineJumpTableEntry> &JT = MJTI->getJumpTables();
  for (MachineInstr *MI : T2JumpTables) {
    const MCInstrDesc &MCID = MI->getDesc();
    unsigned NumOps = MCID.getNumOperands();
    unsigned JTOpIdx = NumOps - (MI->isPredicable() ? 2 : 1);
    MachineOperand JTOP = MI->getOperand(JTOpIdx);
    unsigned JTI = JTOP.getIndex();
    assert(JTI < JT.size());
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2412-2418
```cpp
    // We prefer if target blocks for the jump table come after the jump
    // instruction so we can use TB[BH]. Loop through the target blocks
    // and try to adjust them such that that's true.
    int JTNumber = MI->getParent()->getNumber();
    const std::vector<MachineBasicBlock*> &JTBBs = JT[JTI].MBBs;
    for (MachineBasicBlock *MBB : JTBBs) {
      int DTNumber = MBB->getNumber();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2420-2430
```cpp
      if (DTNumber < JTNumber) {
        // The destination precedes the switch. Try to move the block forward
        // so we have a positive offset.
        MachineBasicBlock *NewBB =
            adjustJTTargetBlockForward(JTI, MBB, MI->getParent());
        if (NewBB)
          MJTI->ReplaceMBBInJumpTable(JTI, MBB, NewBB);
        MadeChange = true;
      }
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2432-2433
```cpp
  return MadeChange;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2435-2446
```cpp
MachineBasicBlock *ARMConstantIslands::adjustJTTargetBlockForward(
    unsigned JTI, MachineBasicBlock *BB, MachineBasicBlock *JTBB) {
  // If the destination block is terminated by an unconditional branch,
  // try to move it; otherwise, create a new block following the jump
  // table that branches back to the actual target. This is a very simple
  // heuristic. FIXME: We can definitely improve it.
  MachineBasicBlock *TBB = nullptr, *FBB = nullptr;
  SmallVector<MachineOperand, 4> Cond;
  SmallVector<MachineOperand, 4> CondPrior;
  MachineFunction::iterator BBi = BB->getIterator();
  MachineFunction::iterator OldPrior = std::prev(BBi);
  MachineFunction::iterator OldNext = std::next(BBi);
```
- EN: Implements `ARMConstantIslands::adjustJTTargetBlockForward`, a mutation/build routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMConstantIslands::adjustJTTargetBlockForward`，它是一个围绕机器函数状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2448-2449
```cpp
  // If the block terminator isn't analyzable, don't try to move the block
  bool B = TII->analyzeBranch(*BB, TBB, FBB, Cond);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2451-2463
```cpp
  // If the block ends in an unconditional branch, move it. The prior block
  // has to have an analyzable terminator for us to move this one. Be paranoid
  // and make sure we're not trying to move the entry block of the function.
  if (!B && Cond.empty() && BB != &MF->front() &&
      !TII->analyzeBranch(*OldPrior, TBB, FBB, CondPrior)) {
    BB->moveAfter(JTBB);
    OldPrior->updateTerminator(BB);
    BB->updateTerminator(OldNext != MF->end() ? &*OldNext : nullptr);
    // Update numbering to account for the block being moved.
    MF->RenumberBlocks();
    ++NumJTMoved;
    return nullptr;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2465-2469
```cpp
  // Create a new MBB for the code after the jump BB.
  MachineBasicBlock *NewBB =
    MF->CreateMachineBasicBlock(JTBB->getBasicBlock());
  MachineFunction::iterator MBBI = ++JTBB->getIterator();
  MF->insert(MBBI, NewBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2471-2473
```cpp
  // Copy live-in information to new block.
  for (const MachineBasicBlock::RegisterMaskPair &RegMaskPair : BB->liveins())
    NewBB->addLiveIn(RegMaskPair);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2475-2485
```cpp
  // Add an unconditional branch from NewBB to BB.
  // There doesn't seem to be meaningful DebugInfo available; this doesn't
  // correspond directly to anything in the source.
  if (isThumb2)
    BuildMI(NewBB, DebugLoc(), TII->get(ARM::t2B))
        .addMBB(BB)
        .add(predOps(ARMCC::AL));
  else
    BuildMI(NewBB, DebugLoc(), TII->get(ARM::tB))
        .addMBB(BB)
        .add(predOps(ARMCC::AL));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2487-2488
```cpp
  // Update internal data structures to account for the newly inserted MBB.
  MF->RenumberBlocks(NewBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2490-2492
```cpp
  // Update the CFG.
  NewBB->addSuccessor(BB);
  JTBB->replaceSuccessor(BB, NewBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2494-2496
```cpp
  ++NumJTInserted;
  return NewBB;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2498-2502
```cpp
/// createARMConstantIslandPass - returns an instance of the constpool
/// island pass.
FunctionPass *llvm::createARMConstantIslandPass() {
  return new ARMConstantIslands();
}
```
- EN: Implements `llvm::createARMConstantIslandPass`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `llvm::createARMConstantIslandPass`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2504-2505
```cpp
INITIALIZE_PASS(ARMConstantIslands, "arm-cp-islands", ARM_CP_ISLANDS_OPT_NAME,
                false, false)
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

## Key Concepts / 关键概念

- EN: Primary role: constant-island placement and layout constraints.
  - CN: 核心职责：常量岛放置与布局约束。
- EN: MachineInstr-level manipulation is central to this file.
  - CN: 该文件以 MachineInstr 层面的操作为核心。
- EN: MachineBasicBlock flow and branch structure matter to the implementation.
  - CN: 实现高度依赖 MachineBasicBlock 流程与分支结构。
- EN: The MC layer is used to print, encode, or stream target instructions and symbols.
  - CN: MC 层被用于打印、编码或流式输出目标指令与符号。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。

## Dependencies / 依赖关系

- EN: Backend-local headers: `ARM.h`, `ARMBaseInstrInfo.h`, `ARMBasicBlockInfo.h`, `ARMMachineFunctionInfo.h`, `ARMSubtarget.h`, `MCTargetDesc/ARMBaseInfo.h`, `MVETailPredUtils.h`, `Thumb2InstrInfo.h` ... (+1 more).
  - CN: 后端本地头文件：`ARM.h`, `ARMBaseInstrInfo.h`, `ARMBasicBlockInfo.h`, `ARMMachineFunctionInfo.h`, `ARMSubtarget.h`, `MCTargetDesc/ARMBaseInfo.h`, `MVETailPredUtils.h`, `Thumb2InstrInfo.h` ... (+1 more)。
- EN: LLVM infrastructure headers: `llvm/ADT/DenseMap.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/Statistic.h`, `llvm/ADT/StringRef.h`, `llvm/CodeGen/LivePhysRegs.h`, `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineConstantPool.h` ... (+18 more).
  - CN: LLVM 基础设施头文件：`llvm/ADT/DenseMap.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/Statistic.h`, `llvm/ADT/StringRef.h`, `llvm/CodeGen/LivePhysRegs.h`, `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineConstantPool.h` ... (+18 more)。
- EN: Standard/system headers: `algorithm`, `cassert`, `cstdint`, `iterator`, `vector`.
  - CN: 标准库/系统头文件：`algorithm`, `cassert`, `cstdint`, `iterator`, `vector`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
- EN: The implementation also depends on LLVM MC layer classes for final assembly/object emission.
  - CN: 实现还依赖 LLVM 的 MC 层类来完成最终汇编/目标文件输出。
