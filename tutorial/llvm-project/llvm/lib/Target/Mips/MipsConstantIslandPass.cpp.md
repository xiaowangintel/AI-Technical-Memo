# MipsConstantIslandPass.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MipsConstantIslandPass.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `MipsConstantIslandPass` for the Mips backend, focusing on constant-island placement and layout constraints.
- 用途 (CN): 实现 Mips 后端中的 `MipsConstantIslandPass`，重点处理常量岛放置与布局约束。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15
```cpp
//===- MipsConstantIslandPass.cpp - Emit Pc Relative loads ----------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This pass is used to make Pc relative loads of constants.
// For now, only Mips16 will use this.
//
// Loading constants inline is expensive on Mips16 and it's in general better
// to place the constant nearby in code space and then it can be loaded with a
// simple 16 bit load instruction.
//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 16-20
```cpp
// The constants can be not just numbers but addresses of functions and labels.
// This can be particularly helpful in static relocation mode for embedded
// non-linux targets.
//
//===----------------------------------------------------------------------===//
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 22-36
```cpp
#include "Mips.h"
#include "Mips16InstrInfo.h"
#include "MipsMachineFunction.h"
#include "MipsSubtarget.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/CodeGen/MachineBasicBlock.h"
#include "llvm/CodeGen/MachineConstantPool.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineOperand.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 37-51
```cpp
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/Config/llvm-config.h"
#include "llvm/IR/Constants.h"
#include "llvm/IR/DataLayout.h"
#include "llvm/IR/DebugLoc.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/Type.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/Compiler.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/Format.h"
#include "llvm/Support/raw_ostream.h"
#include <cassert>
#include <cstdint>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 52-53
```cpp
#include <iterator>
#include <vector>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 55-55
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 57-57
```cpp
#define DEBUG_TYPE "mips-constant-islands"
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 59-62
```cpp
STATISTIC(NumCPEs,       "Number of constpool entries");
STATISTIC(NumSplit,      "Number of uncond branches inserted");
STATISTIC(NumCBrFixed,   "Number of cond branches fixed");
STATISTIC(NumUBrFixed,   "Number of uncond branches fixed");
```
- EN: Declares `STATISTIC`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `STATISTIC`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 64-67
```cpp
// FIXME: This option should be removed once it has received sufficient testing.
static cl::opt<bool>
AlignConstantIslands("mips-align-constant-islands", cl::Hidden, cl::init(true),
          cl::desc("Align constant islands in code"));
```
- EN: Declares `cl::init`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `cl::init`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 69-75
```cpp
// Rather than do make check tests with huge amounts of code, we force
// the test to use this amount.
static cl::opt<int> ConstantIslandsSmallOffset(
  "mips-constant-islands-small-offset",
  cl::init(0),
  cl::desc("Make small offsets be this amount for testing purposes"),
  cl::Hidden);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 77-83
```cpp
// For testing purposes we tell it to not use relaxed load forms so that it
// will split blocks.
static cl::opt<bool> NoLoadRelaxation(
  "mips-constant-islands-no-load-relaxation",
  cl::init(false),
  cl::desc("Don't relax loads to long loads - for testing purposes"),
  cl::Hidden);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 85-99
```cpp
static unsigned int branchTargetOperand(MachineInstr *MI) {
  switch (MI->getOpcode()) {
  case Mips::Bimm16:
  case Mips::BimmX16:
  case Mips::Bteqz16:
  case Mips::BteqzX16:
  case Mips::Btnez16:
  case Mips::BtnezX16:
  case Mips::JalB16:
    return 0;
  case Mips::BeqzRxImm16:
  case Mips::BeqzRxImmX16:
  case Mips::BnezRxImm16:
  case Mips::BnezRxImmX16:
    return 1;
```
- EN: Implements `branchTargetOperand`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `branchTargetOperand`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 100-102
```cpp
  }
  llvm_unreachable("Unknown branch type");
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 104-118
```cpp
static unsigned int longformBranchOpcode(unsigned int Opcode) {
  switch (Opcode) {
  case Mips::Bimm16:
  case Mips::BimmX16:
    return Mips::BimmX16;
  case Mips::Bteqz16:
  case Mips::BteqzX16:
    return Mips::BteqzX16;
  case Mips::Btnez16:
  case Mips::BtnezX16:
    return Mips::BtnezX16;
  case Mips::JalB16:
    return Mips::JalB16;
  case Mips::BeqzRxImm16:
  case Mips::BeqzRxImmX16:
```
- EN: Implements `longformBranchOpcode`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `longformBranchOpcode`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 119-125
```cpp
    return Mips::BeqzRxImmX16;
  case Mips::BnezRxImm16:
  case Mips::BnezRxImmX16:
    return Mips::BnezRxImmX16;
  }
  llvm_unreachable("Unknown branch type");
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 127-141
```cpp
// FIXME: need to go through this whole constant islands port and check
// the math for branch ranges and clean this up and make some functions
// to calculate things that are done many times identically.
// Need to refactor some of the code to call this routine.
static unsigned int branchMaxOffsets(unsigned int Opcode) {
  unsigned Bits, Scale;
  switch (Opcode) {
    case Mips::Bimm16:
      Bits = 11;
      Scale = 2;
      break;
    case Mips::BimmX16:
      Bits = 16;
      Scale = 2;
      break;
```
- EN: Implements `branchMaxOffsets`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `branchMaxOffsets`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 142-156
```cpp
    case Mips::BeqzRxImm16:
      Bits = 8;
      Scale = 2;
      break;
    case Mips::BeqzRxImmX16:
      Bits = 16;
      Scale = 2;
      break;
    case Mips::BnezRxImm16:
      Bits = 8;
      Scale = 2;
      break;
    case Mips::BnezRxImmX16:
      Bits = 16;
      Scale = 2;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 157-171
```cpp
      break;
    case Mips::Bteqz16:
      Bits = 8;
      Scale = 2;
      break;
    case Mips::BteqzX16:
      Bits = 16;
      Scale = 2;
      break;
    case Mips::Btnez16:
      Bits = 8;
      Scale = 2;
      break;
    case Mips::BtnezX16:
      Bits = 16;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 172-179
```cpp
      Scale = 2;
      break;
    default:
      llvm_unreachable("Unknown branch type");
  }
  unsigned MaxOffs = ((1 << (Bits-1))-1) * Scale;
  return MaxOffs;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 181-181
```cpp
namespace {
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 183-184
```cpp
  using Iter = MachineBasicBlock::iterator;
  using ReverseIter = MachineBasicBlock::reverse_iterator;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 186-196
```cpp
  /// MipsConstantIslands - Due to limited PC-relative displacements, Mips
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
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 198-211
```cpp
  class MipsConstantIslands : public MachineFunctionPass {
    /// BasicBlockInfo - Information about the offset and size of a single
    /// basic block.
    struct BasicBlockInfo {
      /// Offset - Distance from the beginning of the function to the beginning
      /// of this basic block.
      ///
      /// Offsets are computed assuming worst case padding before an aligned
      /// block. This means that subtracting basic block offsets always gives a
      /// conservative estimate of the real distance which may be smaller.
      ///
      /// Because worst case padding is used, the computed offset of an aligned
      /// block may not actually be aligned.
      unsigned Offset = 0;
```
- EN: Declares `MipsConstantIslands`, packaging target-specific state and APIs around `MipsConstantIslandPass`.
- CN: 这里声明 `MipsConstantIslands`，把与 `MipsConstantIslandPass` 相关的目标特定状态和 API 组织在一起。

### Lines 213-218
```cpp
      /// Size - Size of the basic block in bytes.  If the block contains
      /// inline assembly, this is a worst case estimate.
      ///
      /// The size does not include any alignment padding whether from the
      /// beginning of the block, or from an aligned jump table at the end.
      unsigned Size = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 220-220
```cpp
      BasicBlockInfo() = default;
```
- EN: Declares `BasicBlockInfo`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BasicBlockInfo`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 222-223
```cpp
      unsigned postOffset() const { return Offset + Size; }
    };
```
- EN: Implements `postOffset`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `postOffset`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 225-225
```cpp
    std::vector<BasicBlockInfo> BBInfo;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 227-230
```cpp
    /// WaterList - A sorted list of basic blocks where islands could be placed
    /// (i.e. blocks that don't fall through to the following block, due
    /// to a return, unreachable, or unconditional branch).
    std::vector<MachineBasicBlock*> WaterList;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 232-234
```cpp
    /// NewWaterList - The subset of WaterList that was created since the
    /// previous iteration by inserting unconditional branches.
    SmallPtrSet<MachineBasicBlock *, 4> NewWaterList;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 236-236
```cpp
    using water_iterator = std::vector<MachineBasicBlock *>::iterator;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 238-252
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
- EN: Declares `CPUser`, packaging target-specific state and APIs around `MipsConstantIslandPass`.
- CN: 这里声明 `CPUser`，把与 `MipsConstantIslandPass` 相关的目标特定状态和 API 组织在一起。

### Lines 253-253
```cpp
      MachineBasicBlock *HighWaterMark;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 255-259
```cpp
    private:
      unsigned MaxDisp;
      unsigned LongFormMaxDisp; // mips16 has 16/32 bit instructions
                                // with different displacements
      unsigned LongFormOpcode;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 261-262
```cpp
    public:
      bool NegOk;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 264-271
```cpp
      CPUser(MachineInstr *mi, MachineInstr *cpemi, unsigned maxdisp,
             bool neg,
             unsigned longformmaxdisp, unsigned longformopcode)
        : MI(mi), CPEMI(cpemi), MaxDisp(maxdisp),
          LongFormMaxDisp(longformmaxdisp), LongFormOpcode(longformopcode),
          NegOk(neg){
        HighWaterMark = CPEMI->getParent();
      }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 273-278
```cpp
      /// getMaxDisp - Returns the maximum displacement supported by MI.
      unsigned getMaxDisp() const {
        unsigned xMaxDisp = ConstantIslandsSmallOffset?
                            ConstantIslandsSmallOffset: MaxDisp;
        return xMaxDisp;
      }
```
- EN: Implements `getMaxDisp`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getMaxDisp`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 280-282
```cpp
      void setMaxDisp(unsigned val) {
        MaxDisp = val;
      }
```
- EN: Implements `setMaxDisp`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `setMaxDisp`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 284-286
```cpp
      unsigned getLongFormMaxDisp() const {
        return LongFormMaxDisp;
      }
```
- EN: Implements `getLongFormMaxDisp`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getLongFormMaxDisp`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 288-291
```cpp
      unsigned getLongFormOpcode() const {
          return LongFormOpcode;
      }
    };
```
- EN: Implements `getLongFormOpcode`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getLongFormOpcode`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 293-295
```cpp
    /// CPUsers - Keep track of all of the machine instructions that use various
    /// constant pools and their max displacement.
    std::vector<CPUser> CPUsers;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 297-303
```cpp
  /// CPEntry - One per constant pool entry, keeping the machine instruction
  /// pointer, the constpool index, and the number of CPUser's which
  /// reference this entry.
  struct CPEntry {
    MachineInstr *CPEMI;
    unsigned CPI;
    unsigned RefCount;
```
- EN: Declares `CPEntry`, packaging target-specific state and APIs around `MipsConstantIslandPass`.
- CN: 这里声明 `CPEntry`，把与 `MipsConstantIslandPass` 相关的目标特定状态和 API 组织在一起。

### Lines 305-307
```cpp
    CPEntry(MachineInstr *cpemi, unsigned cpi, unsigned rc = 0)
      : CPEMI(cpemi), CPI(cpi), RefCount(rc) {}
  };
```
- EN: Implements `CPEntry`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `CPEntry`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 309-314
```cpp
  /// CPEntries - Keep track of all of the constant pool entry machine
  /// instructions. For each original constpool index (i.e. those that
  /// existed upon entry to this pass), it keeps a vector of entries.
  /// Original elements are cloned as we go along; the clones are
  /// put in the vector of the original element, but have distinct CPIs.
  std::vector<std::vector<CPEntry>> CPEntries;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 316-325
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
    int UncondBr;
```
- EN: Declares `ImmBranch`, packaging target-specific state and APIs around `MipsConstantIslandPass`.
- CN: 这里声明 `ImmBranch`，把与 `MipsConstantIslandPass` 相关的目标特定状态和 API 组织在一起。

### Lines 327-329
```cpp
    ImmBranch(MachineInstr *mi, unsigned maxdisp, bool cond, int ubr)
      : MI(mi), MaxDisp(maxdisp), isCond(cond), UncondBr(ubr) {}
  };
```
- EN: Implements `ImmBranch`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ImmBranch`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 331-333
```cpp
  /// ImmBranches - Keep track of all the immediate branch instructions.
  ///
  std::vector<ImmBranch> ImmBranches;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 335-337
```cpp
  /// HasFarJump - True if any far jump instruction has been emitted during
  /// the branch fix up pass.
  bool HasFarJump;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 339-343
```cpp
  const MipsSubtarget *STI = nullptr;
  const Mips16InstrInfo *TII;
  MipsFunctionInfo *MFI;
  MachineFunction *MF = nullptr;
  MachineConstantPool *MCP = nullptr;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 345-346
```cpp
  unsigned PICLabelUId;
  bool PrescannedForConstants = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 348-350
```cpp
  void initPICLabelUId(unsigned UId) {
    PICLabelUId = UId;
  }
```
- EN: Implements `initPICLabelUId`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `initPICLabelUId`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 352-354
```cpp
  unsigned createPICLabelUId() {
    return PICLabelUId++;
  }
```
- EN: Implements `createPICLabelUId`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `createPICLabelUId`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 356-357
```cpp
  public:
    static char ID;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 359-359
```cpp
    MipsConstantIslands() : MachineFunctionPass(ID) {}
```
- EN: Implements `MipsConstantIslands`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsConstantIslands`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 361-361
```cpp
    StringRef getPassName() const override { return "Mips Constant Islands"; }
```
- EN: Implements `getPassName`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getPassName`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 363-363
```cpp
    bool runOnMachineFunction(MachineFunction &F) override;
```
- EN: Declares `runOnMachineFunction`, a pass-entry routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `runOnMachineFunction`，它是一个围绕机器函数状态展开的Pass 入口例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 365-367
```cpp
    MachineFunctionProperties getRequiredProperties() const override {
      return MachineFunctionProperties().setNoVRegs();
    }
```
- EN: Implements `getRequiredProperties`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getRequiredProperties`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 369-375
```cpp
    void doInitialPlacement(std::vector<MachineInstr*> &CPEMIs);
    CPEntry *findConstPoolEntry(unsigned CPI, const MachineInstr *CPEMI);
    Align getCPEAlign(const MachineInstr &CPEMI);
    void initializeFunctionInfo(const std::vector<MachineInstr*> &CPEMIs);
    unsigned getOffsetOf(MachineInstr *MI) const;
    unsigned getUserOffset(CPUser&) const;
    void dumpBBs();
```
- EN: Declares `doInitialPlacement`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `doInitialPlacement`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 377-380
```cpp
    bool isOffsetInRange(unsigned UserOffset, unsigned TrialOffset,
                         unsigned Disp, bool NegativeOK);
    bool isOffsetInRange(unsigned UserOffset, unsigned TrialOffset,
                         const CPUser &U);
```
- EN: Declares `isOffsetInRange`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isOffsetInRange`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 382-396
```cpp
    void computeBlockSize(MachineBasicBlock *MBB);
    MachineBasicBlock *splitBlockBeforeInstr(MachineInstr &MI);
    void updateForInsertedWaterBlock(MachineBasicBlock *NewBB);
    void adjustBBOffsetsAfter(MachineBasicBlock *BB);
    bool decrementCPEReferenceCount(unsigned CPI, MachineInstr* CPEMI);
    int findInRangeCPEntry(CPUser& U, unsigned UserOffset);
    int findLongFormInRangeCPEntry(CPUser& U, unsigned UserOffset);
    bool findAvailableWater(CPUser&U, unsigned UserOffset,
                            water_iterator &WaterIter);
    void createNewWater(unsigned CPUserIndex, unsigned UserOffset,
                        MachineBasicBlock *&NewMBB);
    bool handleConstantPoolUser(unsigned CPUserIndex);
    void removeDeadCPEMI(MachineInstr *CPEMI);
    bool removeUnusedCPEntries();
    bool isCPEntryInRange(MachineInstr *MI, unsigned UserOffset,
```
- EN: Declares `computeBlockSize`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `computeBlockSize`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 397-404
```cpp
                          MachineInstr *CPEMI, unsigned Disp, bool NegOk,
                          bool DoDump = false);
    bool isWaterInRange(unsigned UserOffset, MachineBasicBlock *Water,
                        CPUser &U, unsigned &Growth);
    bool isBBInRange(MachineInstr *MI, MachineBasicBlock *BB, unsigned Disp);
    bool fixupImmediateBr(ImmBranch &Br);
    bool fixupConditionalBr(ImmBranch &Br);
    bool fixupUnconditionalBr(ImmBranch &Br);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 406-407
```cpp
    void prescanForConstants();
  };
```
- EN: Declares `prescanForConstants`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `prescanForConstants`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 409-409
```cpp
} // end anonymous namespace
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 411-411
```cpp
char MipsConstantIslands::ID = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 413-418
```cpp
bool MipsConstantIslands::isOffsetInRange
  (unsigned UserOffset, unsigned TrialOffset,
   const CPUser &U) {
  return isOffsetInRange(UserOffset, TrialOffset,
                         U.getMaxDisp(), U.NegOk);
}
```
- EN: Implements `MipsConstantIslands::isOffsetInRange`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsConstantIslands::isOffsetInRange`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 420-429
```cpp
#if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
/// print block size and offset information - debugging
LLVM_DUMP_METHOD void MipsConstantIslands::dumpBBs() {
  for (unsigned J = 0, E = BBInfo.size(); J !=E; ++J) {
    const BasicBlockInfo &BBI = BBInfo[J];
    dbgs() << format("%08x %bb.%u\t", BBI.Offset, J)
           << format(" size=%#x\n", BBInfo[J].Size);
  }
}
#endif
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 431-445
```cpp
bool MipsConstantIslands::runOnMachineFunction(MachineFunction &mf) {
  // The intention is for this to be a mips16 only pass for now
  // FIXME:
  MF = &mf;
  MCP = mf.getConstantPool();
  STI = &mf.getSubtarget<MipsSubtarget>();
  LLVM_DEBUG(dbgs() << "constant island machine function "
                    << "\n");
  if (!STI->inMips16Mode() || !MipsSubtarget::useConstantIslands()) {
    return false;
  }
  TII = (const Mips16InstrInfo *)STI->getInstrInfo();
  MFI = MF->getInfo<MipsFunctionInfo>();
  LLVM_DEBUG(dbgs() << "constant island processing "
                    << "\n");
```
- EN: Implements `MipsConstantIslands::runOnMachineFunction`, a pass-entry routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsConstantIslands::runOnMachineFunction`，它是一个围绕机器函数状态展开的Pass 入口例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 446-450
```cpp
  //
  // will need to make predermination if there is any constants we need to
  // put in constant islands. TBD.
  //
  if (!PrescannedForConstants) prescanForConstants();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 452-454
```cpp
  HasFarJump = false;
  // This pass invalidates liveness information when it splits basic blocks.
  MF->getRegInfo().invalidateLiveness();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 456-458
```cpp
  // Renumber all of the machine basic blocks in the function, guaranteeing that
  // the numbers agree with the position of the block in the function.
  MF->RenumberBlocks();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 460-460
```cpp
  bool MadeChange = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 462-466
```cpp
  // Perform the initial placement of the constant pool entries.  To start with,
  // we put them all at the end of the function.
  std::vector<MachineInstr*> CPEMIs;
  if (!MCP->isEmpty())
    doInitialPlacement(CPEMIs);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 468-469
```cpp
  /// The next UID to take is the first unused one.
  initPICLabelUId(CPEMIs.size());
```
- EN: Declares `initPICLabelUId`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `initPICLabelUId`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 471-476
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

### Lines 478-479
```cpp
  /// Remove dead constant pool entries.
  MadeChange |= removeUnusedCPEntries();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 481-492
```cpp
  // Iteratively place constant pool entries and fix up branches until there
  // is no change.
  unsigned NoCPIters = 0, NoBRIters = 0;
  (void)NoBRIters;
  while (true) {
    LLVM_DEBUG(dbgs() << "Beginning CP iteration #" << NoCPIters << '\n');
    bool CPChange = false;
    for (unsigned i = 0, e = CPUsers.size(); i != e; ++i)
      CPChange |= handleConstantPoolUser(i);
    if (CPChange && ++NoCPIters > 30)
      report_fatal_error("Constant Island pass failed to converge!");
    LLVM_DEBUG(dumpBBs());
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 494-496
```cpp
    // Clear NewWaterList now.  If we split a block for branches, it should
    // appear as "new water" for the next iteration of constant pool placement.
    NewWaterList.clear();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 498-508
```cpp
    LLVM_DEBUG(dbgs() << "Beginning BR iteration #" << NoBRIters << '\n');
    bool BRChange = false;
    for (unsigned i = 0, e = ImmBranches.size(); i != e; ++i)
      BRChange |= fixupImmediateBr(ImmBranches[i]);
    if (BRChange && ++NoBRIters > 30)
      report_fatal_error("Branch Fix Up pass failed to converge!");
    LLVM_DEBUG(dumpBBs());
    if (!CPChange && !BRChange)
      break;
    MadeChange = true;
  }
```
- EN: Declares `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 510-510
```cpp
  LLVM_DEBUG(dbgs() << '\n'; dumpBBs());
```
- EN: Declares `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 512-518
```cpp
  BBInfo.clear();
  WaterList.clear();
  CPUsers.clear();
  CPEntries.clear();
  ImmBranches.clear();
  return MadeChange;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 520-526
```cpp
/// doInitialPlacement - Perform the initial placement of the constant pool
/// entries.  To start with, we put them all at the end of the function.
void
MipsConstantIslands::doInitialPlacement(std::vector<MachineInstr*> &CPEMIs) {
  // Create the basic block to hold the CPE's.
  MachineBasicBlock *BB = MF->CreateMachineBasicBlock();
  MF->push_back(BB);
```
- EN: Implements `MipsConstantIslands::doInitialPlacement`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsConstantIslands::doInitialPlacement`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 528-529
```cpp
  // MachineConstantPool measures alignment in bytes. We measure in log2(bytes).
  const Align MaxAlign = MCP->getConstantPoolAlign();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 531-533
```cpp
  // Mark the basic block as required by the const-pool.
  // If AlignConstantIslands isn't set, use 4-byte alignment for everything.
  BB->setAlignment(AlignConstantIslands ? MaxAlign : Align(4));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 535-537
```cpp
  // The function needs to be as aligned as the basic blocks. The linker may
  // move functions around based on their alignment.
  MF->ensureAlignment(BB->getAlignment());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 539-544
```cpp
  // Order the entries in BB by descending alignment.  That ensures correct
  // alignment of all entries as long as BB is sufficiently aligned.  Keep
  // track of the insertion point for each alignment.  We are going to bucket
  // sort the entries as they are created.
  SmallVector<MachineBasicBlock::iterator, 8> InsPoint(Log2(MaxAlign) + 1,
                                                       BB->end());
```
- EN: Declares `InsPoint`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `InsPoint`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 546-548
```cpp
  // Add all of the constants from the constant pool to the end block, use an
  // identity mapping of CPI's to CPE's.
  const std::vector<MachineConstantPoolEntry> &CPs = MCP->getConstants();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 550-557
```cpp
  const DataLayout &TD = MF->getDataLayout();
  for (unsigned i = 0, e = CPs.size(); i != e; ++i) {
    unsigned Size = CPs[i].getSizeInBytes(TD);
    assert(Size >= 4 && "Too small constant pool entry");
    Align Alignment = CPs[i].getAlign();
    // Verify that all constant pool entries are a multiple of their alignment.
    // If not, we would have to pad them out so that instructions stay aligned.
    assert(isAligned(Alignment, Size) && "CP Entry not multiple of 4 bytes!");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 559-561
```cpp
    // Insert CONSTPOOL_ENTRY before entries with a smaller alignment.
    unsigned LogAlign = Log2(Alignment);
    MachineBasicBlock::iterator InsAt = InsPoint[LogAlign];
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 563-565
```cpp
    MachineInstr *CPEMI =
      BuildMI(*BB, InsAt, DebugLoc(), TII->get(Mips::CONSTPOOL_ENTRY))
        .addImm(i).addConstantPoolIndex(i).addImm(Size);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 567-567
```cpp
    CPEMIs.push_back(CPEMI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 569-581
```cpp
    // Ensure that future entries with higher alignment get inserted before
    // CPEMI. This is bucket sort with iterators.
    for (unsigned a = LogAlign + 1; a <= Log2(MaxAlign); ++a)
      if (InsPoint[a] == InsAt)
        InsPoint[a] = CPEMI;
    // Add a new CPEntry, but no corresponding CPUser yet.
    CPEntries.emplace_back(1, CPEntry(CPEMI, i));
    ++NumCPEs;
    LLVM_DEBUG(dbgs() << "Moved CPI#" << i << " to end of function, size = "
                      << Size << ", align = " << Alignment.value() << '\n');
  }
  LLVM_DEBUG(BB->dump());
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 583-590
```cpp
/// BBHasFallthrough - Return true if the specified basic block can fallthrough
/// into the block immediately after it.
static bool BBHasFallthrough(MachineBasicBlock *MBB) {
  // Get the next machine basic block in the function.
  MachineFunction::iterator MBBI = MBB->getIterator();
  // Can't fall off end of function.
  if (std::next(MBBI) == MBB->getParent()->end())
    return false;
```
- EN: Implements `BBHasFallthrough`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `BBHasFallthrough`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 592-594
```cpp
  MachineBasicBlock *NextBB = &*std::next(MBBI);
  return llvm::is_contained(MBB->successors(), NextBB);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 596-609
```cpp
/// findConstPoolEntry - Given the constpool index and CONSTPOOL_ENTRY MI,
/// look up the corresponding CPEntry.
MipsConstantIslands::CPEntry
*MipsConstantIslands::findConstPoolEntry(unsigned CPI,
                                        const MachineInstr *CPEMI) {
  std::vector<CPEntry> &CPEs = CPEntries[CPI];
  // Number of entries per constpool index should be small, just do a
  // linear search.
  for (CPEntry &CPE : CPEs) {
    if (CPE.CPEMI == CPEMI)
      return &CPE;
  }
  return nullptr;
}
```
- EN: Implements `MipsConstantIslands::findConstPoolEntry`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsConstantIslands::findConstPoolEntry`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 611-614
```cpp
/// getCPEAlign - Returns the required alignment of the constant pool entry
/// represented by CPEMI.  Alignment is measured in log2(bytes) units.
Align MipsConstantIslands::getCPEAlign(const MachineInstr &CPEMI) {
  assert(CPEMI.getOpcode() == Mips::CONSTPOOL_ENTRY);
```
- EN: Implements `MipsConstantIslands::getCPEAlign`, a query/helper routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsConstantIslands::getCPEAlign`，它是一个围绕机器指令展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 616-618
```cpp
  // Everything is 4-byte aligned unless AlignConstantIslands is set.
  if (!AlignConstantIslands)
    return Align(4);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 620-623
```cpp
  unsigned CPI = CPEMI.getOperand(1).getIndex();
  assert(CPI < MCP->getConstants().size() && "Invalid constant pool index.");
  return MCP->getConstants()[CPI].getAlign();
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 625-631
```cpp
/// initializeFunctionInfo - Do the initial scan of the function, building up
/// information about the sizes of each block, the location of all the water,
/// and finding all of the constant pool users.
void MipsConstantIslands::
initializeFunctionInfo(const std::vector<MachineInstr*> &CPEMIs) {
  BBInfo.clear();
  BBInfo.resize(MF->getNumBlockIDs());
```
- EN: Implements `initializeFunctionInfo`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `initializeFunctionInfo`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 633-638
```cpp
  // First thing, compute the size of all basic blocks, and see if the function
  // has any inline assembly in it. If so, we have to be conservative about
  // alignment assumptions, as we don't know for sure the size of any
  // instructions in the inline assembly.
  for (MachineBasicBlock &MBB : *MF)
    computeBlockSize(&MBB);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 640-641
```cpp
  // Compute block offsets.
  adjustBBOffsetsAfter(&MF->front());
```
- EN: Declares `adjustBBOffsetsAfter`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `adjustBBOffsetsAfter`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 643-651
```cpp
  // Now go back through the instructions and build up our data structures.
  for (MachineBasicBlock &MBB : *MF) {
    // If this block doesn't fall through into the next MBB, then this is
    // 'water' that a constant pool island could be placed.
    if (!BBHasFallthrough(&MBB))
      WaterList.push_back(&MBB);
    for (MachineInstr &MI : MBB) {
      if (MI.isDebugInstr())
        continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 653-667
```cpp
      int Opc = MI.getOpcode();
      if (MI.isBranch()) {
        bool isCond = false;
        unsigned Bits = 0;
        unsigned Scale = 1;
        int UOpc = Opc;
        switch (Opc) {
        default:
          continue;  // Ignore other branches for now
        case Mips::Bimm16:
          Bits = 11;
          Scale = 2;
          isCond = false;
          break;
        case Mips::BimmX16:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 668-682
```cpp
          Bits = 16;
          Scale = 2;
          isCond = false;
          break;
        case Mips::BeqzRxImm16:
          UOpc=Mips::Bimm16;
          Bits = 8;
          Scale = 2;
          isCond = true;
          break;
        case Mips::BeqzRxImmX16:
          UOpc=Mips::Bimm16;
          Bits = 16;
          Scale = 2;
          isCond = true;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 683-697
```cpp
          break;
        case Mips::BnezRxImm16:
          UOpc=Mips::Bimm16;
          Bits = 8;
          Scale = 2;
          isCond = true;
          break;
        case Mips::BnezRxImmX16:
          UOpc=Mips::Bimm16;
          Bits = 16;
          Scale = 2;
          isCond = true;
          break;
        case Mips::Bteqz16:
          UOpc=Mips::Bimm16;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 698-712
```cpp
          Bits = 8;
          Scale = 2;
          isCond = true;
          break;
        case Mips::BteqzX16:
          UOpc=Mips::Bimm16;
          Bits = 16;
          Scale = 2;
          isCond = true;
          break;
        case Mips::Btnez16:
          UOpc=Mips::Bimm16;
          Bits = 8;
          Scale = 2;
          isCond = true;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 713-724
```cpp
          break;
        case Mips::BtnezX16:
          UOpc=Mips::Bimm16;
          Bits = 16;
          Scale = 2;
          isCond = true;
          break;
        }
        // Record this immediate branch.
        unsigned MaxOffs = ((1 << (Bits-1))-1) * Scale;
        ImmBranches.push_back(ImmBranch(&MI, MaxOffs, isCond, UOpc));
      }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 726-727
```cpp
      if (Opc == Mips::CONSTPOOL_ENTRY)
        continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 729-733
```cpp
      // Scan the instructions for constant pool operands.
      for (const MachineOperand &MO : MI.operands())
        if (MO.isCPI()) {
          // We found one.  The addressing mode tells us the max displacement
          // from the PC that this instruction permits.
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 735-749
```cpp
          // Basic size info comes from the TSFlags field.
          unsigned Bits = 0;
          unsigned Scale = 1;
          bool NegOk = false;
          unsigned LongFormBits = 0;
          unsigned LongFormScale = 0;
          unsigned LongFormOpcode = 0;
          switch (Opc) {
          default:
            llvm_unreachable("Unknown addressing mode for CP reference!");
          case Mips::LwRxPcTcp16:
            Bits = 8;
            Scale = 4;
            LongFormOpcode = Mips::LwRxPcTcpX16;
            LongFormBits = 14;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 750-764
```cpp
            LongFormScale = 1;
            break;
          case Mips::LwRxPcTcpX16:
            Bits = 14;
            Scale = 1;
            NegOk = true;
            break;
          }
          // Remember that this is a user of a CP entry.
          unsigned CPI = MO.getIndex();
          MachineInstr *CPEMI = CPEMIs[CPI];
          unsigned MaxOffs = ((1 << Bits)-1) * Scale;
          unsigned LongFormMaxOffs = ((1 << LongFormBits)-1) * LongFormScale;
          CPUsers.push_back(CPUser(&MI, CPEMI, MaxOffs, NegOk, LongFormMaxOffs,
                                   LongFormOpcode));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 766-769
```cpp
          // Increment corresponding CPEntry reference count.
          CPEntry *CPE = findConstPoolEntry(CPI, CPEMI);
          assert(CPE && "Cannot find a corresponding CPEntry!");
          CPE->RefCount++;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 771-777
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

### Lines 779-783
```cpp
/// computeBlockSize - Compute the size and some alignment information for MBB.
/// This function updates BBInfo directly.
void MipsConstantIslands::computeBlockSize(MachineBasicBlock *MBB) {
  BasicBlockInfo &BBI = BBInfo[MBB->getNumber()];
  BBI.Size = 0;
```
- EN: Implements `MipsConstantIslands::computeBlockSize`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsConstantIslands::computeBlockSize`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 785-787
```cpp
  for (const MachineInstr &MI : *MBB)
    BBI.Size += TII->getInstSizeInBytes(MI);
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 789-793
```cpp
/// getOffsetOf - Return the current offset of the specified machine instruction
/// from the start of the function.  This offset changes as stuff is moved
/// around inside the function.
unsigned MipsConstantIslands::getOffsetOf(MachineInstr *MI) const {
  MachineBasicBlock *MBB = MI->getParent();
```
- EN: Implements `MipsConstantIslands::getOffsetOf`, a query/helper routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsConstantIslands::getOffsetOf`，它是一个围绕机器基本块展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 795-798
```cpp
  // The offset is composed of two things: the sum of the sizes of all MBB's
  // before this instruction's block, and the offset from the start of the block
  // it is in.
  unsigned Offset = BBInfo[MBB->getNumber()].Offset;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 800-806
```cpp
  // Sum instructions before MI in MBB.
  for (MachineBasicBlock::iterator I = MBB->begin(); &*I != MI; ++I) {
    assert(I != MBB->end() && "Didn't find MI in its own basic block?");
    Offset += TII->getInstSizeInBytes(*I);
  }
  return Offset;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 808-813
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

### Lines 815-821
```cpp
/// updateForInsertedWaterBlock - When a block is newly inserted into the
/// machine function, it upsets all of the block numbers.  Renumber the blocks
/// and update the arrays that parallel this numbering.
void MipsConstantIslands::updateForInsertedWaterBlock
  (MachineBasicBlock *NewBB) {
  // Renumber the MBB's to keep them consecutive.
  NewBB->getParent()->RenumberBlocks(NewBB);
```
- EN: Implements `MipsConstantIslands::updateForInsertedWaterBlock`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsConstantIslands::updateForInsertedWaterBlock`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 823-825
```cpp
  // Insert an entry into BBInfo to align it properly with the (newly
  // renumbered) block numbers.
  BBInfo.insert(BBInfo.begin() + NewBB->getNumber(), BasicBlockInfo());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 827-831
```cpp
  // Next, update WaterList.  Specifically, we need to add NewMBB as having
  // available water after it.
  water_iterator IP = llvm::lower_bound(WaterList, NewBB, CompareMBBNumbers);
  WaterList.insert(IP, NewBB);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 833-835
```cpp
unsigned MipsConstantIslands::getUserOffset(CPUser &U) const {
  return getOffsetOf(U.MI);
}
```
- EN: Implements `MipsConstantIslands::getUserOffset`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsConstantIslands::getUserOffset`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 837-842
```cpp
/// Split the basic block containing MI into two blocks, which are joined by
/// an unconditional branch.  Update data structures and renumber blocks to
/// account for this change and returns the newly created block.
MachineBasicBlock *
MipsConstantIslands::splitBlockBeforeInstr(MachineInstr &MI) {
  MachineBasicBlock *OrigBB = MI.getParent();
```
- EN: Implements `MipsConstantIslands::splitBlockBeforeInstr`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsConstantIslands::splitBlockBeforeInstr`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 844-848
```cpp
  // Create a new MBB for the code after the OrigBB.
  MachineBasicBlock *NewBB =
    MF->CreateMachineBasicBlock(OrigBB->getBasicBlock());
  MachineFunction::iterator MBBI = ++OrigBB->getIterator();
  MF->insert(MBBI, NewBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 850-851
```cpp
  // Splice the instructions starting with MI over to NewBB.
  NewBB->splice(NewBB->end(), OrigBB, MI, OrigBB->end());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 853-858
```cpp
  // Add an unconditional branch from OrigBB to NewBB.
  // Note the new unconditional branch is not being recorded.
  // There doesn't seem to be meaningful DebugInfo available; this doesn't
  // correspond to anything in the source.
  BuildMI(OrigBB, DebugLoc(), TII->get(Mips::Bimm16)).addMBB(NewBB);
  ++NumSplit;
```
- EN: Declares `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 860-861
```cpp
  // Update the CFG.  All succs of OrigBB are now succs of NewBB.
  NewBB->transferSuccessors(OrigBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 863-864
```cpp
  // OrigBB branches to NewBB.
  OrigBB->addSuccessor(NewBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 866-869
```cpp
  // Update internal data structures to account for the newly inserted MBB.
  // This is almost the same as updateForInsertedWaterBlock, except that
  // the Water goes after OrigBB, not NewBB.
  MF->RenumberBlocks(NewBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 871-873
```cpp
  // Insert an entry into BBInfo to align it properly with the (newly
  // renumbered) block numbers.
  BBInfo.insert(BBInfo.begin() + NewBB->getNumber(), BasicBlockInfo());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 875-885
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

### Lines 887-892
```cpp
  // Figure out how large the OrigBB is.  As the first half of the original
  // block, it cannot contain a tablejump.  The size includes
  // the new jump we added.  (It should be possible to do this without
  // recounting everything, but it's very confusing, and this is rarely
  // executed.)
  computeBlockSize(OrigBB);
```
- EN: Declares `computeBlockSize`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `computeBlockSize`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 894-896
```cpp
  // Figure out how large the NewMBB is.  As the second half of the original
  // block, it may contain a tablejump.
  computeBlockSize(NewBB);
```
- EN: Declares `computeBlockSize`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `computeBlockSize`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 898-899
```cpp
  // All BBOffsets following these blocks must be modified.
  adjustBBOffsetsAfter(OrigBB);
```
- EN: Declares `adjustBBOffsetsAfter`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `adjustBBOffsetsAfter`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 901-902
```cpp
  return NewBB;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 904-918
```cpp
/// isOffsetInRange - Checks whether UserOffset (the location of a constant pool
/// reference) is within MaxDisp of TrialOffset (a proposed location of a
/// constant pool entry).
bool MipsConstantIslands::isOffsetInRange(unsigned UserOffset,
                                         unsigned TrialOffset, unsigned MaxDisp,
                                         bool NegativeOK) {
  if (UserOffset <= TrialOffset) {
    // User before the Trial.
    if (TrialOffset - UserOffset <= MaxDisp)
      return true;
  } else if (NegativeOK) {
    if (UserOffset - TrialOffset <= MaxDisp)
      return true;
  }
  return false;
```
- EN: Implements `MipsConstantIslands::isOffsetInRange`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsConstantIslands::isOffsetInRange`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 919-919
```cpp
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 921-935
```cpp
/// isWaterInRange - Returns true if a CPE placed after the specified
/// Water (a basic block) will be in range for the specific MI.
///
/// Compute how much the function will grow by inserting a CPE after Water.
bool MipsConstantIslands::isWaterInRange(unsigned UserOffset,
                                        MachineBasicBlock* Water, CPUser &U,
                                        unsigned &Growth) {
  unsigned CPEOffset = BBInfo[Water->getNumber()].postOffset();
  unsigned NextBlockOffset;
  Align NextBlockAlignment;
  MachineFunction::const_iterator NextBlock = ++Water->getIterator();
  if (NextBlock == MF->end()) {
    NextBlockOffset = BBInfo[Water->getNumber()].postOffset();
    NextBlockAlignment = Align(1);
  } else {
```
- EN: Implements `MipsConstantIslands::isWaterInRange`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsConstantIslands::isWaterInRange`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 936-940
```cpp
    NextBlockOffset = BBInfo[NextBlock->getNumber()].Offset;
    NextBlockAlignment = NextBlock->getAlignment();
  }
  unsigned Size = U.CPEMI->getOperand(2).getImm();
  unsigned CPEEnd = CPEOffset + Size;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 942-949
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

### Lines 951-958
```cpp
    // If the CPE is to be inserted before the instruction, that will raise
    // the offset of the instruction. Also account for unknown alignment padding
    // in blocks between CPE and the user.
    if (CPEOffset < UserOffset)
      UserOffset += Growth;
  } else
    // CPE fits in existing padding.
    Growth = 0;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 960-961
```cpp
  return isOffsetInRange(UserOffset, CPEOffset, U);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 963-969
```cpp
/// isCPEntryInRange - Returns true if the distance between specific MI and
/// specific ConstPool entry instruction can fit in MI's displacement field.
bool MipsConstantIslands::isCPEntryInRange
  (MachineInstr *MI, unsigned UserOffset,
   MachineInstr *CPEMI, unsigned MaxDisp,
   bool NegOk, bool DoDump) {
  unsigned CPEOffset  = getOffsetOf(CPEMI);
```
- EN: Implements `MipsConstantIslands::isCPEntryInRange`, a query/helper routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsConstantIslands::isCPEntryInRange`，它是一个围绕机器指令展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 971-983
```cpp
  if (DoDump) {
    LLVM_DEBUG({
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

### Lines 985-986
```cpp
  return isOffsetInRange(UserOffset, CPEOffset, MaxDisp, NegOk);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 988-1001
```cpp
#ifndef NDEBUG
/// BBIsJumpedOver - Return true of the specified basic block's only predecessor
/// unconditionally branches to its only successor.
static bool BBIsJumpedOver(MachineBasicBlock *MBB) {
  if (MBB->pred_size() != 1 || MBB->succ_size() != 1)
    return false;
  MachineBasicBlock *Succ = *MBB->succ_begin();
  MachineBasicBlock *Pred = *MBB->pred_begin();
  MachineInstr *PredMI = &Pred->back();
  if (PredMI->getOpcode() == Mips::Bimm16)
    return PredMI->getOperand(0).getMBB() == Succ;
  return false;
}
#endif
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1003-1011
```cpp
void MipsConstantIslands::adjustBBOffsetsAfter(MachineBasicBlock *BB) {
  unsigned BBNum = BB->getNumber();
  for(unsigned i = BBNum + 1, e = MF->getNumBlockIDs(); i < e; ++i) {
    // Get the offset and known bits at the end of the layout predecessor.
    // Include the alignment of the current block.
    unsigned Offset = BBInfo[i - 1].Offset + BBInfo[i - 1].Size;
    BBInfo[i].Offset = Offset;
  }
}
```
- EN: Implements `MipsConstantIslands::adjustBBOffsetsAfter`, a mutation/build routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsConstantIslands::adjustBBOffsetsAfter`，它是一个围绕机器基本块展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1013-1027
```cpp
/// decrementCPEReferenceCount - find the constant pool entry with index CPI
/// and instruction CPEMI, and decrement its refcount.  If the refcount
/// becomes 0 remove the entry and instruction.  Returns true if we removed
/// the entry, false if we didn't.
bool MipsConstantIslands::decrementCPEReferenceCount(unsigned CPI,
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
- EN: Implements `MipsConstantIslands::decrementCPEReferenceCount`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsConstantIslands::decrementCPEReferenceCount`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1028-1029
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1031-1040
```cpp
/// LookForCPEntryInRange - see if the currently referenced CPE is in range;
/// if not, see if an in-range clone of the CPE is in range, and if so,
/// change the data structures so the user references the clone.  Returns:
/// 0 = no existing entry found
/// 1 = entry found, and there were no code insertions or deletions
/// 2 = entry found, and there were code insertions or deletions
int MipsConstantIslands::findInRangeCPEntry(CPUser& U, unsigned UserOffset)
{
  MachineInstr *UserMI = U.MI;
  MachineInstr *CPEMI  = U.CPEMI;
```
- EN: Implements `MipsConstantIslands::findInRangeCPEntry`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsConstantIslands::findInRangeCPEntry`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1042-1047
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

### Lines 1049-1063
```cpp
  // No.  Look for previously created clones of the CPE that are in range.
  unsigned CPI = CPEMI->getOperand(1).getIndex();
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

### Lines 1064-1078
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

### Lines 1079-1079
```cpp
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1081-1093
```cpp
/// LookForCPEntryInRange - see if the currently referenced CPE is in range;
/// This version checks if the longer form of the instruction can be used to
/// to satisfy things.
/// if not, see if an in-range clone of the CPE is in range, and if so,
/// change the data structures so the user references the clone.  Returns:
/// 0 = no existing entry found
/// 1 = entry found, and there were no code insertions or deletions
/// 2 = entry found, and there were code insertions or deletions
int MipsConstantIslands::findLongFormInRangeCPEntry
  (CPUser& U, unsigned UserOffset)
{
  MachineInstr *UserMI = U.MI;
  MachineInstr *CPEMI  = U.CPEMI;
```
- EN: Implements `MipsConstantIslands::findLongFormInRangeCPEntry`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsConstantIslands::findLongFormInRangeCPEntry`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1095-1103
```cpp
  // Check to see if the CPE is already in-range.
  if (isCPEntryInRange(UserMI, UserOffset, CPEMI,
                       U.getLongFormMaxDisp(), U.NegOk,
                       true)) {
    LLVM_DEBUG(dbgs() << "In range\n");
    UserMI->setDesc(TII->get(U.getLongFormOpcode()));
    U.setMaxDisp(U.getLongFormMaxDisp());
    return 2;  // instruction is longer length now
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1105-1119
```cpp
  // No.  Look for previously created clones of the CPE that are in range.
  unsigned CPI = CPEMI->getOperand(1).getIndex();
  std::vector<CPEntry> &CPEs = CPEntries[CPI];
  for (CPEntry &CPE : CPEs) {
    // We already tried this one
    if (CPE.CPEMI == CPEMI)
      continue;
    // Removing CPEs can leave empty entries, skip
    if (CPE.CPEMI == nullptr)
      continue;
    if (isCPEntryInRange(UserMI, UserOffset, CPE.CPEMI, U.getLongFormMaxDisp(),
                         U.NegOk)) {
      LLVM_DEBUG(dbgs() << "Replacing CPE#" << CPI << " with CPE#" << CPE.CPI
                        << "\n");
      // Point the CPUser node to the replacement
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1120-1134
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

### Lines 1135-1135
```cpp
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1137-1149
```cpp
/// getUnconditionalBrDisp - Returns the maximum displacement that can fit in
/// the specific unconditional branch instruction.
static inline unsigned getUnconditionalBrDisp(int Opc) {
  switch (Opc) {
  case Mips::Bimm16:
    return ((1<<10)-1)*2;
  case Mips::BimmX16:
    return ((1<<16)-1)*2;
  default:
    break;
  }
  return ((1<<16)-1)*2;
}
```
- EN: Implements `getUnconditionalBrDisp`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getUnconditionalBrDisp`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1151-1162
```cpp
/// findAvailableWater - Look for an existing entry in the WaterList in which
/// we can place the CPE referenced from U so it's within range of U's MI.
/// Returns true if found, false if not.  If it returns true, WaterIter
/// is set to the WaterList entry.
/// To ensure that this pass
/// terminates, the CPE location for a particular CPUser is only allowed to
/// move to a lower address, so search backward from the end of the list and
/// prefer the first water that is in range.
bool MipsConstantIslands::findAvailableWater(CPUser &U, unsigned UserOffset,
                                      water_iterator &WaterIter) {
  if (WaterList.empty())
    return false;
```
- EN: Implements `MipsConstantIslands::findAvailableWater`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsConstantIslands::findAvailableWater`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1164-1178
```cpp
  unsigned BestGrowth = ~0u;
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
    unsigned Growth;
    if (isWaterInRange(UserOffset, WaterBB, U, Growth) &&
        (WaterBB->getNumber() < U.HighWaterMark->getNumber() ||
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1179-1184
```cpp
         NewWaterList.count(WaterBB)) && Growth < BestGrowth) {
      // This is the least amount of required padding seen so far.
      BestGrowth = Growth;
      WaterIter = IP;
      LLVM_DEBUG(dbgs() << "Found water after " << printMBBReference(*WaterBB)
                        << " Growth=" << Growth << '\n');
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1186-1194
```cpp
      // Keep looking unless it is perfect.
      if (BestGrowth == 0)
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

### Lines 1196-1210
```cpp
/// createNewWater - No existing WaterList entry will work for
/// CPUsers[CPUserIndex], so create a place to put the CPE.  The end of the
/// block is used if in range, and the conditional branch munged so control
/// flow is correct.  Otherwise the block is split to create a hole with an
/// unconditional branch around it.  In either case NewMBB is set to a
/// block following which the new island can be inserted (the WaterList
/// is not adjusted).
void MipsConstantIslands::createNewWater(unsigned CPUserIndex,
                                        unsigned UserOffset,
                                        MachineBasicBlock *&NewMBB) {
  CPUser &U = CPUsers[CPUserIndex];
  MachineInstr *UserMI = U.MI;
  MachineInstr *CPEMI  = U.CPEMI;
  MachineBasicBlock *UserMBB = UserMI->getParent();
  const BasicBlockInfo &UserBBI = BBInfo[UserMBB->getNumber()];
```
- EN: Implements `MipsConstantIslands::createNewWater`, a query/helper routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsConstantIslands::createNewWater`，它是一个围绕机器基本块展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1212-1218
```cpp
  // If the block does not end in an unconditional branch already, and if the
  // end of the block is within range, make new water there.
  if (BBHasFallthrough(UserMBB)) {
    // Size of branch to insert.
    unsigned Delta = 2;
    // Compute the offset where the CPE will begin.
    unsigned CPEOffset = UserBBI.postOffset() + Delta;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1220-1234
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
      int UncondBr = Mips::Bimm16;
      BuildMI(UserMBB, DebugLoc(), TII->get(UncondBr)).addMBB(NewMBB);
      unsigned MaxDisp = getUnconditionalBrDisp(UncondBr);
      ImmBranches.push_back(ImmBranch(&UserMBB->back(),
                                      MaxDisp, false, UncondBr));
      BBInfo[UserMBB->getNumber()].Size += Delta;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1235-1238
```cpp
      adjustBBOffsetsAfter(UserMBB);
      return;
    }
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1240-1240
```cpp
  // What a big block.  Find a place within the block to split it.
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 1242-1248
```cpp
  // Try to split the block so it's fully aligned.  Compute the latest split
  // point where we can add a 4-byte branch instruction, and then align to
  // Align which is the largest possible alignment in the function.
  const Align Align = MF->getAlignment();
  unsigned BaseInsertOffset = UserOffset + U.getMaxDisp();
  LLVM_DEBUG(dbgs() << format("Split in middle of big block before %#x",
                              BaseInsertOffset));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1250-1253
```cpp
  // The 4 in the following is for the unconditional branch we'll be inserting
  // Alignment of the island is handled
  // inside isOffsetInRange.
  BaseInsertOffset -= 4;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1255-1256
```cpp
  LLVM_DEBUG(dbgs() << format(", adjusted to %#x", BaseInsertOffset)
                    << " la=" << Log2(Align) << '\n');
```
- EN: Declares `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1258-1272
```cpp
  // This could point off the end of the block if we've already got constant
  // pool entries following this block; only the last one is in the water list.
  // Back past any possible branches (allow for a conditional and a maximally
  // long unconditional).
  if (BaseInsertOffset + 8 >= UserBBI.postOffset()) {
    BaseInsertOffset = UserBBI.postOffset() - 8;
    LLVM_DEBUG(dbgs() << format("Move inside block: %#x\n", BaseInsertOffset));
  }
  unsigned EndInsertOffset = BaseInsertOffset + 4 +
    CPEMI->getOperand(2).getImm();
  MachineBasicBlock::iterator MI = UserMI;
  ++MI;
  unsigned CPUIndex = CPUserIndex+1;
  unsigned NumCPUsers = CPUsers.size();
  //MachineInstr *LastIT = 0;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1273-1287
```cpp
  for (unsigned Offset = UserOffset + TII->getInstSizeInBytes(*UserMI);
       Offset < BaseInsertOffset;
       Offset += TII->getInstSizeInBytes(*MI), MI = std::next(MI)) {
    assert(MI != UserMBB->end() && "Fell off end of block");
    if (CPUIndex < NumCPUsers && CPUsers[CPUIndex].MI == MI) {
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
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1288-1291
```cpp
      EndInsertOffset += U.CPEMI->getOperand(2).getImm();
      CPUIndex++;
    }
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1293-1294
```cpp
  NewMBB = splitBlockBeforeInstr(*--MI);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1296-1307
```cpp
/// handleConstantPoolUser - Analyze the specified user, checking to see if it
/// is out-of-range.  If so, pick up the constant pool value and move it some
/// place in-range.  Return true if we changed any addresses (thus must run
/// another pass of branch lengthening), false otherwise.
bool MipsConstantIslands::handleConstantPoolUser(unsigned CPUserIndex) {
  CPUser &U = CPUsers[CPUserIndex];
  MachineInstr *UserMI = U.MI;
  MachineInstr *CPEMI  = U.CPEMI;
  unsigned CPI = CPEMI->getOperand(1).getIndex();
  unsigned Size = CPEMI->getOperand(2).getImm();
  // Compute this only once, it's expensive.
  unsigned UserOffset = getUserOffset(U);
```
- EN: Implements `MipsConstantIslands::handleConstantPoolUser`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsConstantIslands::handleConstantPoolUser`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1309-1313
```cpp
  // See if the current entry is within range, or there is a clone of it
  // in range.
  int result = findInRangeCPEntry(U, UserOffset);
  if (result==1) return false;
  else if (result==2) return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1315-1321
```cpp
  // Look for water where we can place this CPE.
  MachineBasicBlock *NewIsland = MF->CreateMachineBasicBlock();
  MachineBasicBlock *NewMBB;
  water_iterator IP;
  if (findAvailableWater(U, UserOffset, IP)) {
    LLVM_DEBUG(dbgs() << "Found water in range\n");
    MachineBasicBlock *WaterBB = *IP;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1323-1327
```cpp
    // If the original WaterList entry was "new water" on this iteration,
    // propagate that to the new island.  This is just keeping NewWaterList
    // updated to match the WaterList, which will be updated below.
    if (NewWaterList.erase(WaterBB))
      NewWaterList.insert(NewIsland);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1329-1340
```cpp
    // The new CPE goes before the following block (NewMBB).
    NewMBB = &*++WaterBB->getIterator();
  } else {
    // No water found.
    // we first see if a longer form of the instrucion could have reached
    // the constant. in that case we won't bother to split
    if (!NoLoadRelaxation) {
      result = findLongFormInRangeCPEntry(U, UserOffset);
      if (result != 0) return true;
    }
    LLVM_DEBUG(dbgs() << "No water found\n");
    createNewWater(CPUserIndex, UserOffset, NewMBB);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1342-1350
```cpp
    // splitBlockBeforeInstr adds to WaterList, which is important when it is
    // called while handling branches so that the water will be seen on the
    // next iteration for constant pools, but in this context, we don't want
    // it.  Check for this so it will be removed from the WaterList.
    // Also remove any entry from NewWaterList.
    MachineBasicBlock *WaterBB = &*--NewMBB->getIterator();
    IP = llvm::find(WaterList, WaterBB);
    if (IP != WaterList.end())
      NewWaterList.erase(WaterBB);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1352-1354
```cpp
    // We are adding new water.  Update NewWaterList.
    NewWaterList.insert(NewIsland);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1356-1361
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

### Lines 1363-1364
```cpp
  // Okay, we know we can put an island before NewMBB now, do it!
  MF->insert(NewMBB->getIterator(), NewIsland);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1366-1367
```cpp
  // Update internal data structures to account for the newly inserted MBB.
  updateForInsertedWaterBlock(NewIsland);
```
- EN: Declares `updateForInsertedWaterBlock`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `updateForInsertedWaterBlock`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1369-1370
```cpp
  // Decrement the old entry, and remove it if refcount becomes 0.
  decrementCPEReferenceCount(CPI, CPEMI);
```
- EN: Declares `decrementCPEReferenceCount`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `decrementCPEReferenceCount`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1372-1374
```cpp
  // No existing clone of this CPE is within range.
  // We will be generating a new clone.  Get a UID for it.
  unsigned ID = createPICLabelUId();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1376-1382
```cpp
  // Now that we have an island to add the CPE to, clone the original CPE and
  // add it to the island.
  U.HighWaterMark = NewIsland;
  U.CPEMI = BuildMI(NewIsland, DebugLoc(), TII->get(Mips::CONSTPOOL_ENTRY))
                .addImm(ID).addConstantPoolIndex(CPI).addImm(Size);
  CPEntries[CPI].push_back(CPEntry(U.CPEMI, ID, 1));
  ++NumCPEs;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1384-1385
```cpp
  // Mark the basic block as aligned as required by the const-pool entry.
  NewIsland->setAlignment(getCPEAlign(*U.CPEMI));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1387-1389
```cpp
  // Increase the size of the island block to account for the new entry.
  BBInfo[NewIsland->getNumber()].Size += Size;
  adjustBBOffsetsAfter(&*--NewIsland->getIterator());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1391-1396
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

### Lines 1398-1400
```cpp
  LLVM_DEBUG(
      dbgs() << "  Moved CPE to #" << ID << " CPI=" << CPI
             << format(" offset=%#x\n", BBInfo[NewIsland->getNumber()].Offset));
```
- EN: Declares `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1402-1403
```cpp
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1405-1414
```cpp
/// removeDeadCPEMI - Remove a dead constant pool entry instruction. Update
/// sizes and offsets of impacted basic blocks.
void MipsConstantIslands::removeDeadCPEMI(MachineInstr *CPEMI) {
  MachineBasicBlock *CPEBB = CPEMI->getParent();
  unsigned Size = CPEMI->getOperand(2).getImm();
  CPEMI->eraseFromParent();
  BBInfo[CPEBB->getNumber()].Size -= Size;
  // All succeeding offsets have the current size value added in, fix this.
  if (CPEBB->empty()) {
    BBInfo[CPEBB->getNumber()].Size = 0;
```
- EN: Implements `MipsConstantIslands::removeDeadCPEMI`, a mutation/build routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsConstantIslands::removeDeadCPEMI`，它是一个围绕机器基本块展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1416-1421
```cpp
    // This block no longer needs to be aligned.
    CPEBB->setAlignment(Align(1));
  } else {
    // Entries are sorted by descending alignment, so realign from the front.
    CPEBB->setAlignment(getCPEAlign(*CPEBB->begin()));
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1423-1429
```cpp
  adjustBBOffsetsAfter(CPEBB);
  // An island has only one predecessor BB and one successor BB. Check if
  // this BB's predecessor jumps directly to this BB's successor. This
  // shouldn't happen currently.
  assert(!BBIsJumpedOver(CPEBB) && "How did this happen?");
  // FIXME: remove the empty blocks after all the work is done?
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1431-1445
```cpp
/// removeUnusedCPEntries - Remove constant pool entries whose refcounts
/// are zero.
bool MipsConstantIslands::removeUnusedCPEntries() {
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
- EN: Implements `MipsConstantIslands::removeUnusedCPEntries`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsConstantIslands::removeUnusedCPEntries`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1447-1453
```cpp
/// isBBInRange - Returns true if the distance between specific MI and
/// specific BB can fit in MI's displacement field.
bool MipsConstantIslands::isBBInRange
  (MachineInstr *MI,MachineBasicBlock *DestBB, unsigned MaxDisp) {
  unsigned PCAdj = 4;
  unsigned BrOffset   = getOffsetOf(MI) + PCAdj;
  unsigned DestOffset = BBInfo[DestBB->getNumber()].Offset;
```
- EN: Implements `MipsConstantIslands::isBBInRange`, a query/helper routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsConstantIslands::isBBInRange`，它是一个围绕机器基本块展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1455-1459
```cpp
  LLVM_DEBUG(dbgs() << "Branch of destination " << printMBBReference(*DestBB)
                    << " from " << printMBBReference(*MI->getParent())
                    << " max delta=" << MaxDisp << " from " << getOffsetOf(MI)
                    << " to " << DestOffset << " offset "
                    << int(DestOffset - BrOffset) << "\t" << *MI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1461-1470
```cpp
  if (BrOffset <= DestOffset) {
    // Branch before the Dest.
    if (DestOffset-BrOffset <= MaxDisp)
      return true;
  } else {
    if (BrOffset-DestOffset <= MaxDisp)
      return true;
  }
  return false;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1472-1477
```cpp
/// fixupImmediateBr - Fix up an immediate branch whose destination is too far
/// away to fit in its displacement field.
bool MipsConstantIslands::fixupImmediateBr(ImmBranch &Br) {
  MachineInstr *MI = Br.MI;
  unsigned TargetOperand = branchTargetOperand(MI);
  MachineBasicBlock *DestBB = MI->getOperand(TargetOperand).getMBB();
```
- EN: Implements `MipsConstantIslands::fixupImmediateBr`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsConstantIslands::fixupImmediateBr`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1479-1481
```cpp
  // Check to see if the DestBB is already in-range.
  if (isBBInRange(MI, DestBB, Br.MaxDisp))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1483-1486
```cpp
  if (!Br.isCond)
    return fixupUnconditionalBr(Br);
  return fixupConditionalBr(Br);
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1488-1502
```cpp
/// fixupUnconditionalBr - Fix up an unconditional branch whose destination is
/// too far away to fit in its displacement field. If the LR register has been
/// spilled in the epilogue, then we can use BL to implement a far jump.
/// Otherwise, add an intermediate branch instruction to a branch.
bool
MipsConstantIslands::fixupUnconditionalBr(ImmBranch &Br) {
  MachineInstr *MI = Br.MI;
  MachineBasicBlock *MBB = MI->getParent();
  MachineBasicBlock *DestBB = MI->getOperand(0).getMBB();
  // Use BL to implement far jump.
  unsigned BimmX16MaxDisp = ((1 << 16)-1) * 2;
  if (isBBInRange(MI, DestBB, BimmX16MaxDisp)) {
    Br.MaxDisp = BimmX16MaxDisp;
    MI->setDesc(TII->get(Mips::BimmX16));
  }
```
- EN: Implements `MipsConstantIslands::fixupUnconditionalBr`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsConstantIslands::fixupUnconditionalBr`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1503-1517
```cpp
  else {
    // need to give the math a more careful look here
    // this is really a segment address and not
    // a PC relative address. FIXME. But I think that
    // just reducing the bits by 1 as I've done is correct.
    // The basic block we are branching too much be longword aligned.
    // we know that RA is saved because we always save it right now.
    // this requirement will be relaxed later but we also have an alternate
    // way to implement this that I will implement that does not need jal.
    // We should have a way to back out this alignment restriction
    // if we "can" later. but it is not harmful.
    //
    DestBB->setAlignment(Align(4));
    Br.MaxDisp = ((1<<24)-1) * 2;
    MI->setDesc(TII->get(Mips::JalB16));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1518-1522
```cpp
  }
  BBInfo[MBB->getNumber()].Size += 2;
  adjustBBOffsetsAfter(MBB);
  HasFarJump = true;
  ++NumUBrFixed;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1524-1524
```cpp
  LLVM_DEBUG(dbgs() << "  Changed B to long jump " << *MI);
```
- EN: Declares `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1526-1527
```cpp
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1529-1539
```cpp
/// fixupConditionalBr - Fix up a conditional branch whose destination is too
/// far away to fit in its displacement field. It is converted to an inverse
/// conditional branch + an unconditional branch to the destination.
bool
MipsConstantIslands::fixupConditionalBr(ImmBranch &Br) {
  MachineInstr *MI = Br.MI;
  unsigned TargetOperand = branchTargetOperand(MI);
  MachineBasicBlock *DestBB = MI->getOperand(TargetOperand).getMBB();
  unsigned Opcode = MI->getOpcode();
  unsigned LongFormOpcode = longformBranchOpcode(Opcode);
  unsigned LongFormMaxOff = branchMaxOffsets(LongFormOpcode);
```
- EN: Implements `MipsConstantIslands::fixupConditionalBr`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsConstantIslands::fixupConditionalBr`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1541-1546
```cpp
  // Check to see if the DestBB is already in-range.
  if (isBBInRange(MI, DestBB, LongFormMaxOff)) {
    Br.MaxDisp = LongFormMaxOff;
    MI->setDesc(TII->get(LongFormOpcode));
    return true;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1548-1554
```cpp
  // Add an unconditional branch to the destination and invert the branch
  // condition to jump over it:
  // bteqz L1
  // =>
  // bnez L2
  // b   L1
  // L2:
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 1556-1562
```cpp
  // If the branch is at the end of its MBB and that has a fall-through block,
  // direct the updated conditional branch to the fall-through block. Otherwise,
  // split the MBB before the next instruction.
  MachineBasicBlock *MBB = MI->getParent();
  MachineInstr *BMI = &MBB->back();
  bool NeedSplit = (BMI != MI) || !BBHasFallthrough(MBB);
  unsigned OppositeBranchOpcode = TII->getOppositeBranchOpc(Opcode);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1564-1578
```cpp
  ++NumCBrFixed;
  if (BMI != MI) {
    if (std::next(MachineBasicBlock::iterator(MI)) == std::prev(MBB->end()) &&
        BMI->isUnconditionalBranch()) {
      // Last MI in the BB is an unconditional branch. Can we simply invert the
      // condition and swap destinations:
      // beqz L1
      // b   L2
      // =>
      // bnez L2
      // b   L1
      unsigned BMITargetOperand = branchTargetOperand(BMI);
      MachineBasicBlock *NewDest =
        BMI->getOperand(BMITargetOperand).getMBB();
      if (isBBInRange(MI, NewDest, Br.MaxDisp)) {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1579-1588
```cpp
        LLVM_DEBUG(
            dbgs() << "  Invert Bcc condition and swap its destination with "
                   << *BMI);
        MI->setDesc(TII->get(OppositeBranchOpcode));
        BMI->getOperand(BMITargetOperand).setMBB(DestBB);
        MI->getOperand(TargetOperand).setMBB(NewDest);
        return true;
      }
    }
  }
```
- EN: Declares `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1590-1599
```cpp
  if (NeedSplit) {
    splitBlockBeforeInstr(*MI);
    // No need for the branch to the next block. We're adding an unconditional
    // branch to the destination.
    int delta = TII->getInstSizeInBytes(MBB->back());
    BBInfo[MBB->getNumber()].Size -= delta;
    MBB->back().eraseFromParent();
    // BBInfo[SplitBB].Offset is wrong temporarily, fixed below
  }
  MachineBasicBlock *NextBB = &*++MBB->getIterator();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1601-1603
```cpp
  LLVM_DEBUG(dbgs() << "  Insert B to " << printMBBReference(*DestBB)
                    << " also invert condition and change dest. to "
                    << printMBBReference(*NextBB) << "\n");
```
- EN: Declares `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1605-1619
```cpp
  // Insert a new conditional branch and a new unconditional branch.
  // Also update the ImmBranch as well as adding a new entry for the new branch.
  if (MI->getNumExplicitOperands() == 2) {
    BuildMI(MBB, DebugLoc(), TII->get(OppositeBranchOpcode))
           .addReg(MI->getOperand(0).getReg())
           .addMBB(NextBB);
  } else {
    BuildMI(MBB, DebugLoc(), TII->get(OppositeBranchOpcode))
           .addMBB(NextBB);
  }
  Br.MI = &MBB->back();
  BBInfo[MBB->getNumber()].Size += TII->getInstSizeInBytes(MBB->back());
  BuildMI(MBB, DebugLoc(), TII->get(Br.UncondBr)).addMBB(DestBB);
  BBInfo[MBB->getNumber()].Size += TII->getInstSizeInBytes(MBB->back());
  unsigned MaxDisp = getUnconditionalBrDisp(Br.UncondBr);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1620-1620
```cpp
  ImmBranches.push_back(ImmBranch(&MBB->back(), MaxDisp, false, Br.UncondBr));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1622-1627
```cpp
  // Remove the old conditional branch.  It may or may not still be in MBB.
  BBInfo[MI->getParent()->getNumber()].Size -= TII->getInstSizeInBytes(*MI);
  MI->eraseFromParent();
  adjustBBOffsetsAfter(MBB);
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1629-1643
```cpp
void MipsConstantIslands::prescanForConstants() {
  for (MachineBasicBlock &B : *MF) {
    for (MachineInstr &MI : B) {
      switch (MI.getDesc().getOpcode()) {
      case Mips::LwConstant32: {
        PrescannedForConstants = true;
        LLVM_DEBUG(dbgs() << "constant island constant " << MI << "\n");
        LLVM_DEBUG(dbgs() << "num operands " << MI.getNumOperands() << "\n");
        MachineOperand &Literal = MI.getOperand(1);
        if (Literal.isImm()) {
          int64_t V = Literal.getImm();
          LLVM_DEBUG(dbgs() << "literal " << V << "\n");
          Type *Int32Ty = Type::getInt32Ty(MF->getFunction().getContext());
          const Constant *C = ConstantInt::getSigned(Int32Ty, V);
          unsigned index = MCP->getConstantPoolIndex(C, Align(4));
```
- EN: Implements `MipsConstantIslands::prescanForConstants`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsConstantIslands::prescanForConstants`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1644-1658
```cpp
          MI.getOperand(2).ChangeToImmediate(index);
          LLVM_DEBUG(dbgs() << "constant island constant " << MI << "\n");
          MI.setDesc(TII->get(Mips::LwRxPcTcp16));
          MI.removeOperand(1);
          MI.removeOperand(1);
          MI.addOperand(MachineOperand::CreateCPI(index, 0));
        }
        break;
      }
      default:
        break;
      }
    }
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1660-1663
```cpp
/// Returns a pass that converts branches to long branches.
FunctionPass *llvm::createMipsConstantIslandPass() {
  return new MipsConstantIslands();
}
```
- EN: Implements `llvm::createMipsConstantIslandPass`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `llvm::createMipsConstantIslandPass`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

## Key Concepts / 关键概念

- EN: Primary role: constant-island placement and layout constraints.
  - CN: 核心职责：常量岛放置与布局约束。
- EN: MachineInstr-level manipulation is central to this file.
  - CN: 该文件以 MachineInstr 层面的操作为核心。
- EN: MachineBasicBlock flow and branch structure matter to the implementation.
  - CN: 实现高度依赖 MachineBasicBlock 流程与分支结构。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。
- EN: The component plugs into LLVM machine-function passes or codegen pipelines.
  - CN: 该组件会接入 LLVM 的机器函数 Pass 或代码生成流水线。

## Dependencies / 依赖关系

- EN: Backend-local headers: `Mips.h`, `Mips16InstrInfo.h`, `MipsMachineFunction.h`, `MipsSubtarget.h`.
  - CN: 后端本地头文件：`Mips.h`, `Mips16InstrInfo.h`, `MipsMachineFunction.h`, `MipsSubtarget.h`。
- EN: LLVM infrastructure headers: `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/Statistic.h`, `llvm/ADT/StringRef.h`, `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineConstantPool.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineFunctionPass.h` ... (+16 more).
  - CN: LLVM 基础设施头文件：`llvm/ADT/STLExtras.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/Statistic.h`, `llvm/ADT/StringRef.h`, `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineConstantPool.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineFunctionPass.h` ... (+16 more)。
- EN: Standard/system headers: `cassert`, `cstdint`, `iterator`, `vector`.
  - CN: 标准库/系统头文件：`cassert`, `cstdint`, `iterator`, `vector`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
