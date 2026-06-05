# DynoStats.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Core/DynoStats.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: Dynamic execution stats. It also sits in code that implements BOLT core binary-analysis logic. / 该文件实现 BOLT 核心二进制分析逻辑。 源码头部说明其职责是：Dynamic execution stats。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12

```cpp
//===- bolt/Core/DynoStats.cpp - Dynamic execution stats ------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the DynoStats class.
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 13-22

```cpp
#include "bolt/Core/DynoStats.h"
#include "bolt/Core/BinaryBasicBlock.h"
#include "bolt/Core/BinaryFunction.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/raw_ostream.h"
#include <algorithm>
#include <string>
```

- EN: Pulls in 9 header(s) from local project, LLVM, system dependencies needed by this range. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 9 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里的注释记录了设计假设、不变量或使用说明。

### Lines 23-30

```cpp
#undef  DEBUG_TYPE
#define DEBUG_TYPE "bolt"

using namespace llvm;
using namespace bolt;

namespace opts {
```

- EN: Establishes header-guard or prologue macros so the file can be compiled safely. Works inside namespace scope `llvm`, `bolt`, `opts` to organize symbols. Defines macros such as `DEBUG_TYPE` for constants or compile-time switches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里位于命名空间 `llvm`, `bolt`, `opts` 中，用于组织符号作用域。这里定义宏 `DEBUG_TYPE`，用于常量或编译期开关。这里的注释记录了设计假设、不变量或使用说明。

### Lines 31-40

```cpp
extern cl::OptionCategory BoltCategory;

static cl::opt<uint32_t>
DynoStatsScale("dyno-stats-scale",
  cl::desc("scale to be applied while reporting dyno stats"),
  cl::Optional,
  cl::init(1),
  cl::Hidden,
  cl::cat(BoltCategory));
```

- EN: Declares or implements routines including `desc`, `init`, `cat`. Notable symbols here include `desc`, `init`, `cat`.
- CN: 这里声明或实现函数，例如 `desc`, `init`, `cat`。这里较值得关注的符号包括 `desc`, `init`, `cat`。

### Lines 41-48

```cpp
static cl::opt<uint32_t>
PrintDynoOpcodeStat("print-dyno-opcode-stats",
  cl::desc("print per instruction opcode dyno stats and the function"
              "names:BB offsets of the nth highest execution counts"),
  cl::init(0),
  cl::Hidden,
  cl::cat(BoltCategory));
```

- EN: Declares or implements routines including `init`, `cat`. Notable symbols here include `init`, `cat`.
- CN: 这里声明或实现函数，例如 `init`, `cat`。这里较值得关注的符号包括 `init`, `cat`。

### Lines 49-59

```cpp
} // namespace opts

namespace llvm {
namespace bolt {

bool DynoStats::operator<(const DynoStats &Other) const {
  return std::lexicographical_compare(
      &Stats[FIRST_DYNO_STAT], &Stats[LAST_DYNO_STAT],
      &Other.Stats[FIRST_DYNO_STAT], &Other.Stats[LAST_DYNO_STAT]);
}
```

- EN: Works inside namespace scope `opts`, `llvm`, `bolt` to organize symbols. Contains local control flow that updates state or selects among execution branches. Notable symbols here include `opts`, `llvm`, `bolt`.
- CN: 这里位于命名空间 `opts`, `llvm`, `bolt` 中，用于组织符号作用域。这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里较值得关注的符号包括 `opts`, `llvm`, `bolt`。

### Lines 60-73

```cpp
bool DynoStats::operator==(const DynoStats &Other) const {
  return std::equal(&Stats[FIRST_DYNO_STAT], &Stats[LAST_DYNO_STAT],
                    &Other.Stats[FIRST_DYNO_STAT]);
}

bool DynoStats::lessThan(const DynoStats &Other,
                         ArrayRef<Category> Keys) const {
  return std::lexicographical_compare(
      Keys.begin(), Keys.end(), Keys.begin(), Keys.end(),
      [this, &Other](const Category A, const Category) {
        return Stats[A] < Other.Stats[A];
      });
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 74-90

```cpp
void DynoStats::print(raw_ostream &OS, const DynoStats *Other,
                      MCInstPrinter *Printer) const {
  auto printStatWithDelta = [&](const std::string &Name, uint64_t Stat,
                                uint64_t OtherStat) {
    OS << format("%'20lld : ", Stat * opts::DynoStatsScale) << Name;
    if (Other) {
      if (Stat != OtherStat) {
        OtherStat = std::max(OtherStat, uint64_t(1)); // to prevent divide by 0
        OS << format(" (%+.1f%%)", ((float)Stat - (float)OtherStat) * 100.0 /
                                       (float)(OtherStat));
      } else {
        OS << " (=)";
      }
    }
    OS << '\n';
  };
```

- EN: Declares or implements routines including `format`, `max`. Notable symbols here include `format`, `max`.
- CN: 这里声明或实现函数，例如 `format`, `max`。这里较值得关注的符号包括 `format`, `max`。

### Lines 91-106

```cpp
  for (auto Stat = DynoStats::FIRST_DYNO_STAT + 1;
       Stat < DynoStats::LAST_DYNO_STAT; ++Stat) {

    if (!PrintAArch64Stats && Stat == DynoStats::VENEER_CALLS_AARCH64)
      continue;

    printStatWithDelta(Desc[Stat], Stats[Stat], Other ? (*Other)[Stat] : 0);
  }
  if (opts::PrintDynoOpcodeStat && Printer) {
    OS << "\nProgram-wide opcode histogram:\n";
    OS << "              Opcode,   Execution Count,     Max Exec Count, "
          "Function Name:Offset ...\n";
    std::vector<std::pair<uint64_t, unsigned>> SortedHistogram;
    for (const OpcodeStatTy &Stat : OpcodeHistogram)
      SortedHistogram.emplace_back(Stat.second.first, Stat.first);
```

- EN: Declares or implements routines including `printStatWithDelta`. Notable symbols here include `printStatWithDelta`.
- CN: 这里声明或实现函数，例如 `printStatWithDelta`。这里较值得关注的符号包括 `printStatWithDelta`。

### Lines 107-124

```cpp
    // Sort using lexicographic ordering
    llvm::sort(SortedHistogram);

    // Dump in ascending order: Start with Opcode with Highest execution
    // count.
    for (auto &Stat : llvm::reverse(SortedHistogram)) {
      OS << format("%20s,%'18lld", Printer->getOpcodeName(Stat.second).data(),
                   Stat.first * opts::DynoStatsScale);
      auto It = OpcodeHistogram.find(Stat.second);
      assert(It != OpcodeHistogram.end());
      MaxOpcodeHistogramTy MaxMultiMap = It->second.second;
      // Start with function name:BB offset with highest execution count.
      for (auto &Max : llvm::reverse(MaxMultiMap)) {
        OS << format(", %'18lld, ", Max.first * opts::DynoStatsScale)
           << Max.second.first.str() << ':' << Max.second.second;
      }
      OS << '\n';
    }
```

- EN: Declares or implements routines including `sort`, `format`, `assert`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `sort`, `format`, `assert`.
- CN: 这里声明或实现函数，例如 `sort`, `format`, `assert`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `sort`, `format`, `assert`。

### Lines 125-142

```cpp
  }
}

void DynoStats::operator+=(const DynoStats &Other) {
  for (auto Stat = DynoStats::FIRST_DYNO_STAT + 1;
       Stat < DynoStats::LAST_DYNO_STAT; ++Stat) {
    Stats[Stat] += Other[Stat];
  }
  for (const OpcodeStatTy &Stat : Other.OpcodeHistogram) {
    auto I = OpcodeHistogram.find(Stat.first);
    if (I == OpcodeHistogram.end()) {
      OpcodeHistogram.emplace(Stat);
    } else {
      // Merge other histograms, log only the opts::PrintDynoOpcodeStat'th
      // maximum counts.
      I->second.first += Stat.second.first;
      auto &MMap = I->second.second;
      auto &OtherMMap = Stat.second.second;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 143-157

```cpp
      auto Size = MMap.size();
      assert(Size <= opts::PrintDynoOpcodeStat);
      for (auto OtherMMapPair : llvm::reverse(OtherMMap)) {
        if (Size++ >= opts::PrintDynoOpcodeStat) {
          auto First = MMap.begin();
          if (OtherMMapPair.first <= First->first)
            break;
          MMap.erase(First);
        }
        MMap.emplace(OtherMMapPair);
      }
    }
  }
}
```

- EN: Declares or implements routines including `assert`. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里较值得关注的符号包括 `assert`。

### Lines 158-166

```cpp
DynoStats getDynoStats(BinaryFunction &BF) {
  auto &BC = BF.getBinaryContext();

  DynoStats Stats(/*PrintAArch64Stats*/ BC.isAArch64());

  // Return empty-stats about the function we don't completely understand.
  if (!BF.isSimple() || !BF.hasValidProfile() || !BF.hasCanonicalCFG())
    return Stats;
```

- EN: Declares or implements routines including `getDynoStats`, `Stats`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getDynoStats`, `Stats`.
- CN: 这里声明或实现函数，例如 `getDynoStats`, `Stats`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getDynoStats`, `Stats`。

### Lines 167-177

```cpp
  // Update enumeration of basic blocks for correct detection of branch'
  // direction.
  BF.getLayout().updateLayoutIndices();

  for (BinaryBasicBlock *const BB : BF.getLayout().blocks()) {
    // The basic block execution count equals to the sum of incoming branch
    // frequencies. This may deviate from the sum of outgoing branches of the
    // basic block especially since the block may contain a function that
    // does not return or a function that throws an exception.
    const uint64_t BBExecutionCount = BB->getKnownExecutionCount();
```

- EN: Declares or implements routines including `getKnownExecutionCount`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getKnownExecutionCount`.
- CN: 这里声明或实现函数，例如 `getKnownExecutionCount`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getKnownExecutionCount`。

### Lines 178-185

```cpp
    // Ignore empty blocks and blocks that were not executed.
    if (BB->getNumNonPseudos() == 0 || BBExecutionCount == 0)
      continue;

    // Count AArch64 linker-inserted veneers
    if (BF.isAArch64Veneer())
      Stats[DynoStats::VENEER_CALLS_AARCH64] += BF.getKnownExecutionCount();
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 186-203

```cpp
    // Count various instruction types by iterating through all instructions.
    // When -print-dyno-opcode-stats is on, count per each opcode and record
    // maximum execution counts.
    for (const MCInst &Instr : *BB) {
      if (opts::PrintDynoOpcodeStat) {
        unsigned Opcode = Instr.getOpcode();
        auto I = Stats.OpcodeHistogram.find(Opcode);
        if (I == Stats.OpcodeHistogram.end()) {
          DynoStats::MaxOpcodeHistogramTy MMap;
          MMap.emplace(BBExecutionCount,
                       std::make_pair(BF.getOneName(), BB->getOffset()));
          Stats.OpcodeHistogram.emplace(Opcode,
                                        std::make_pair(BBExecutionCount, MMap));
        } else {
          I->second.first += BBExecutionCount;
          bool Insert = true;
          if (I->second.second.size() == opts::PrintDynoOpcodeStat) {
            auto First = I->second.second.begin();
```

- EN: Declares or implements routines including `make_pair`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `make_pair`.
- CN: 这里声明或实现函数，例如 `make_pair`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `make_pair`。

### Lines 204-216

```cpp
            if (First->first < BBExecutionCount)
              I->second.second.erase(First);
            else
              Insert = false;
          }
          if (Insert) {
            I->second.second.emplace(
                BBExecutionCount,
                std::make_pair(BF.getOneName(), BB->getOffset()));
          }
        }
      }
```

- EN: Declares or implements routines including `make_pair`. Notable symbols here include `make_pair`.
- CN: 这里声明或实现函数，例如 `make_pair`。这里较值得关注的符号包括 `make_pair`。

### Lines 217-225

```cpp
      if (BC.MIB->mayStore(Instr)) {
        Stats[DynoStats::STORES] += BBExecutionCount;
      }
      if (BC.MIB->mayLoad(Instr)) {
        Stats[DynoStats::LOADS] += BBExecutionCount;
      }
      if (!BC.MIB->isCall(Instr))
        continue;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 226-238

```cpp
      uint64_t CallFreq = BBExecutionCount;
      if (BC.MIB->getConditionalTailCall(Instr)) {
        CallFreq =
            BC.MIB->getAnnotationWithDefault<uint64_t>(Instr, "CTCTakenCount");
      }
      Stats[DynoStats::FUNCTION_CALLS] += CallFreq;
      if (BC.MIB->isIndirectCall(Instr)) {
        Stats[DynoStats::INDIRECT_CALLS] += CallFreq;
      } else if (const MCSymbol *CallSymbol = BC.MIB->getTargetSymbol(Instr)) {
        const BinaryFunction *BF = BC.getFunctionForSymbol(CallSymbol);
        if (BF && BF->isPLTFunction()) {
          Stats[DynoStats::PLT_CALLS] += CallFreq;
```

- EN: Declares or implements routines including `if`. Notable symbols here include `if`.
- CN: 这里声明或实现函数，例如 `if`。这里较值得关注的符号包括 `if`。

### Lines 239-252

```cpp
          // We don't process PLT functions and hence have to adjust relevant
          // dynostats here for:
          //
          //   jmp *GOT_ENTRY(%rip)
          //
          // NOTE: this is arch-specific.
          Stats[DynoStats::FUNCTION_CALLS] += CallFreq;
          Stats[DynoStats::INDIRECT_CALLS] += CallFreq;
          Stats[DynoStats::LOADS] += CallFreq;
          Stats[DynoStats::INSTRUCTIONS] += CallFreq;
        }
      }
    }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 253-270

```cpp
    Stats[DynoStats::INSTRUCTIONS] += BB->getNumNonPseudos() * BBExecutionCount;

    // Jump tables.
    const MCInst *LastInstr = BB->getLastNonPseudoInstr();
    if (BC.MIB->getJumpTable(*LastInstr)) {
      Stats[DynoStats::JUMP_TABLE_BRANCHES] += BBExecutionCount;
      LLVM_DEBUG(
        static uint64_t MostFrequentJT;
        if (BBExecutionCount > MostFrequentJT) {
          MostFrequentJT = BBExecutionCount;
          dbgs() << "BOLT-INFO: most frequently executed jump table is in "
                 << "function " << BF << " in basic block " << BB->getName()
                 << " executed totally " << BBExecutionCount << " times.\n";
        }
      );
      continue;
    }
```

- EN: Declares or implements routines including `getNumNonPseudos`, `getLastNonPseudoInstr`, `dbgs`, `getName`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getNumNonPseudos`, `getLastNonPseudoInstr`, `dbgs`, `getName`.
- CN: 这里声明或实现函数，例如 `getNumNonPseudos`, `getLastNonPseudoInstr`, `dbgs`, `getName`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getNumNonPseudos`, `getLastNonPseudoInstr`, `dbgs`, `getName`。

### Lines 271-283

```cpp
    if (BC.MIB->isIndirectBranch(*LastInstr) && !BC.MIB->isCall(*LastInstr)) {
      Stats[DynoStats::UNKNOWN_INDIRECT_BRANCHES] += BBExecutionCount;
      continue;
    }

    // Update stats for branches.
    const MCSymbol *TBB = nullptr;
    const MCSymbol *FBB = nullptr;
    MCInst *CondBranch = nullptr;
    MCInst *UncondBranch = nullptr;
    if (!BB->analyzeBranch(TBB, FBB, CondBranch, UncondBranch))
      continue;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 284-292

```cpp
    if (!CondBranch && !UncondBranch)
      continue;

    // Simple unconditional branch.
    if (!CondBranch) {
      Stats[DynoStats::UNCOND_BRANCHES] += BBExecutionCount;
      continue;
    }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 293-300

```cpp
    // CTCs: instruction annotations could be stripped, hence check the number
    // of successors to identify conditional tail calls.
    if (BB->succ_size() == 1) {
      if (BB->branch_info_begin() != BB->branch_info_end())
        Stats[DynoStats::UNCOND_BRANCHES] += BB->branch_info_begin()->Count;
      continue;
    }
```

- EN: Declares or implements routines including `branch_info_begin`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `branch_info_begin`.
- CN: 这里声明或实现函数，例如 `branch_info_begin`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `branch_info_begin`。

### Lines 301-309

```cpp
    // Conditional branch that could be followed by an unconditional branch.
    uint64_t TakenCount = BB->getTakenBranchInfo().Count;
    if (TakenCount == BinaryBasicBlock::COUNT_NO_PROFILE)
      TakenCount = 0;

    uint64_t NonTakenCount = BB->getFallthroughBranchInfo().Count;
    if (NonTakenCount == BinaryBasicBlock::COUNT_NO_PROFILE)
      NonTakenCount = 0;
```

- EN: Declares or implements routines including `getTakenBranchInfo`, `getFallthroughBranchInfo`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getTakenBranchInfo`, `getFallthroughBranchInfo`.
- CN: 这里声明或实现函数，例如 `getTakenBranchInfo`, `getFallthroughBranchInfo`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getTakenBranchInfo`, `getFallthroughBranchInfo`。

### Lines 310-317

```cpp
    if (BF.isForwardBranch(BB, BB->getConditionalSuccessor(true))) {
      Stats[DynoStats::FORWARD_COND_BRANCHES] += BBExecutionCount;
      Stats[DynoStats::FORWARD_COND_BRANCHES_TAKEN] += TakenCount;
    } else {
      Stats[DynoStats::BACKWARD_COND_BRANCHES] += BBExecutionCount;
      Stats[DynoStats::BACKWARD_COND_BRANCHES_TAKEN] += TakenCount;
    }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 318-325

```cpp
    if (UncondBranch) {
      Stats[DynoStats::UNCOND_BRANCHES] += NonTakenCount;
    }
  }

  return Stats;
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 326-327

```cpp
} // namespace bolt
} // namespace llvm
```

- EN: Works inside namespace scope `bolt`, `llvm` to organize symbols. Notable symbols here include `bolt`, `llvm`.
- CN: 这里位于命名空间 `bolt`, `llvm` 中，用于组织符号作用域。这里较值得关注的符号包括 `bolt`, `llvm`。

## Key Concepts / 关键概念

- `desc`: function or method entry point / 函数或方法入口
- `init`: function or method entry point / 函数或方法入口
- `cat`: function or method entry point / 函数或方法入口
- `format`: function or method entry point / 函数或方法入口
- `max`: function or method entry point / 函数或方法入口
- `DEBUG_TYPE`: macro or compile-time switch / 宏或编译期开关
- `llvm`: namespace scope / 命名空间作用域
- `bolt`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Core/DynoStats.h`, `bolt/Core/BinaryBasicBlock.h`, `bolt/Core/BinaryFunction.h`
- LLVM headers / LLVM 头文件: `llvm/ADT/StringRef.h`, `llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`
- System headers / 系统头文件: `algorithm`, `string`
- Directory context / 目录上下文: `bolt/lib/Core` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Core` 下的相邻文件通常与本文件协作组成对应子系统
