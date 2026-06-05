# IdenticalCodeFolding.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Passes/IdenticalCodeFolding.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: bolt/Passes/IdenticalCodeFolding.cpp This file implements the IdenticalCodeFolding class.. It also sits in code that implements BOLT optimization and transformation passes. / 该文件实现 BOLT 优化与变换 Pass。 源码头部说明其职责是：bolt/Passes/IdenticalCodeFolding.cpp This file implements the IdenticalCodeFolding class.。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12

```cpp
//===- bolt/Passes/IdenticalCodeFolding.cpp -------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the IdenticalCodeFolding class.
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 13-26

```cpp
#include "bolt/Passes/IdenticalCodeFolding.h"
#include "bolt/Core/HashUtilities.h"
#include "bolt/Core/ParallelUtilities.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/FormatVariadic.h"
#include "llvm/Support/ThreadPool.h"
#include "llvm/Support/Timer.h"
#include <atomic>
#include <iterator>
#include <map>
#include <set>
#include <unordered_map>
```

- EN: Pulls in 13 header(s) from local project, LLVM, system dependencies needed by this range. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 13 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里的注释记录了设计假设、不变量或使用说明。

### Lines 27-35

```cpp
#define DEBUG_TYPE "bolt-icf"

using namespace llvm;
using namespace bolt;

namespace opts {

extern cl::OptionCategory BoltOptCategory;
```

- EN: Establishes header-guard or prologue macros so the file can be compiled safely. Works inside namespace scope `llvm`, `bolt`, `opts` to organize symbols. Defines macros such as `DEBUG_TYPE` for constants or compile-time switches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里位于命名空间 `llvm`, `bolt`, `opts` 中，用于组织符号作用域。这里定义宏 `DEBUG_TYPE`，用于常量或编译期开关。这里的注释记录了设计假设、不变量或使用说明。

### Lines 36-48

```cpp
extern bool isHotTextMover(const BinaryFunction &Function);

static cl::opt<bool>
    ICFUseDFS("icf-dfs", cl::desc("use DFS ordering when using -icf option"),
              cl::ReallyHidden, cl::cat(BoltOptCategory));

static cl::opt<bool>
TimeICF("time-icf",
  cl::desc("time icf steps"),
  cl::ReallyHidden,
  cl::ZeroOrMore,
  cl::cat(BoltOptCategory));
```

- EN: Declares or implements routines including `isHotTextMover`, `ICFUseDFS`, `cat`, `desc`. Notable symbols here include `isHotTextMover`, `ICFUseDFS`, `cat`, `desc`.
- CN: 这里声明或实现函数，例如 `isHotTextMover`, `ICFUseDFS`, `cat`, `desc`。这里较值得关注的符号包括 `isHotTextMover`, `ICFUseDFS`, `cat`, `desc`。

### Lines 49-66

```cpp
cl::opt<bolt::IdenticalCodeFolding::ICFLevel, false,
        DeprecatedICFNumericOptionParser>
    ICF("icf", cl::desc("fold functions with identical code"),
        cl::init(bolt::IdenticalCodeFolding::ICFLevel::None),
        cl::values(clEnumValN(bolt::IdenticalCodeFolding::ICFLevel::All, "all",
                              "Enable identical code folding"),
                   clEnumValN(bolt::IdenticalCodeFolding::ICFLevel::All, "1",
                              "Enable identical code folding"),
                   clEnumValN(bolt::IdenticalCodeFolding::ICFLevel::All, "",
                              "Enable identical code folding"),
                   clEnumValN(bolt::IdenticalCodeFolding::ICFLevel::None,
                              "none",
                              "Disable identical code folding (default)"),
                   clEnumValN(bolt::IdenticalCodeFolding::ICFLevel::None, "0",
                              "Disable identical code folding (default)"),
                   clEnumValN(bolt::IdenticalCodeFolding::ICFLevel::Safe,
                              "safe", "Enable safe identical code folding")),
        cl::ZeroOrMore, cl::ValueOptional, cl::cat(BoltOptCategory));
```

- EN: Declares or implements routines including `ICF`, `init`, `folding`, `cat`. Notable symbols here include `ICF`, `init`, `folding`, `cat`.
- CN: 这里声明或实现函数，例如 `ICF`, `init`, `folding`, `cat`。这里较值得关注的符号包括 `ICF`, `init`, `folding`, `cat`。

### Lines 67-82

```cpp
} // namespace opts

bool IdenticalCodeFolding::shouldOptimize(const BinaryFunction &BF) const {
  if (BF.hasUnknownControlFlow())
    return false;
  if (BF.isFolded())
    return false;
  if (BF.hasSDTMarker())
    return false;
  if (BF.isPseudo())
    return false;
  if (opts::ICF == ICFLevel::Safe && BF.hasAddressTaken())
    return false;
  return BinaryFunctionPass::shouldOptimize(BF);
}
```

- EN: Works inside namespace scope `opts` to organize symbols. Declares or implements routines including `shouldOptimize`. Notable symbols here include `shouldOptimize`, `opts`.
- CN: 这里位于命名空间 `opts` 中，用于组织符号作用域。这里声明或实现函数，例如 `shouldOptimize`。这里较值得关注的符号包括 `shouldOptimize`, `opts`。

### Lines 83-91

```cpp
/// Compare two jump tables in 2 functions. The function relies on consistent
/// ordering of basic blocks in both binary functions (e.g. DFS).
static bool equalJumpTables(const JumpTable &JumpTableA,
                            const JumpTable &JumpTableB,
                            const BinaryFunction &FunctionA,
                            const BinaryFunction &FunctionB) {
  if (JumpTableA.EntrySize != JumpTableB.EntrySize)
    return false;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 92-101

```cpp
  if (JumpTableA.Type != JumpTableB.Type)
    return false;

  if (JumpTableA.getSize() != JumpTableB.getSize())
    return false;

  for (uint64_t Index = 0; Index < JumpTableA.Entries.size(); ++Index) {
    const MCSymbol *LabelA = JumpTableA.Entries[Index];
    const MCSymbol *LabelB = JumpTableB.Entries[Index];
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 102-110

```cpp
    const BinaryBasicBlock *TargetA = FunctionA.getBasicBlockForLabel(LabelA);
    const BinaryBasicBlock *TargetB = FunctionB.getBasicBlockForLabel(LabelB);

    if (!TargetA || !TargetB) {
      assert((TargetA || LabelA == FunctionA.getFunctionEndLabel()) &&
             "no target basic block found");
      assert((TargetB || LabelB == FunctionB.getFunctionEndLabel()) &&
             "no target basic block found");
```

- EN: Declares or implements routines including `assert`. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里较值得关注的符号包括 `assert`。

### Lines 111-118

```cpp
      if (TargetA != TargetB)
        return false;

      continue;
    }

    assert(TargetA && TargetB && "cannot locate target block(s)");
```

- EN: Declares or implements routines including `assert`. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里较值得关注的符号包括 `assert`。

### Lines 119-136

```cpp
    if (TargetA->getLayoutIndex() != TargetB->getLayoutIndex())
      return false;
  }

  return true;
}

/// Helper function that compares an instruction of this function to the
/// given instruction of the given function. The functions should have
/// identical CFG.
template <class Compare>
static bool isInstrEquivalentWith(const MCInst &InstA,
                                  const BinaryBasicBlock &BBA,
                                  const MCInst &InstB,
                                  const BinaryBasicBlock &BBB, Compare Comp) {
  if (InstA.getOpcode() != InstB.getOpcode())
    return false;
```

- EN: Introduces type definitions such as `Compare`. Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `Compare`.
- CN: 这里引入类型定义，例如 `Compare`。这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `Compare`。

### Lines 137-151

```cpp
  const BinaryContext &BC = BBA.getFunction()->getBinaryContext();

  // In this function we check for special conditions:
  //
  //    * instructions with landing pads
  //
  // Most of the common cases should be handled by MCPlus::equals()
  // that compares regular instruction operands.
  //
  // NB: there's no need to compare jump table indirect jump instructions
  //     separately as jump tables are handled by comparing corresponding
  //     symbols.
  const std::optional<MCPlus::MCLandingPad> EHInfoA = BC.MIB->getEHInfo(InstA);
  const std::optional<MCPlus::MCLandingPad> EHInfoB = BC.MIB->getEHInfo(InstB);
```

- EN: Declares or implements routines including `getEHInfo`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getEHInfo`.
- CN: 这里声明或实现函数，例如 `getEHInfo`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getEHInfo`。

### Lines 152-163

```cpp
  if (EHInfoA || EHInfoB) {
    if (!EHInfoA && (EHInfoB->first || EHInfoB->second))
      return false;

    if (!EHInfoB && (EHInfoA->first || EHInfoA->second))
      return false;

    if (EHInfoA && EHInfoB) {
      // Action indices should match.
      if (EHInfoA->second != EHInfoB->second)
        return false;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 164-171

```cpp
      if (!EHInfoA->first != !EHInfoB->first)
        return false;

      if (EHInfoA->first && EHInfoB->first) {
        const BinaryBasicBlock *LPA = BBA.getLandingPad(EHInfoA->first);
        const BinaryBasicBlock *LPB = BBB.getLandingPad(EHInfoB->first);
        assert(LPA && LPB && "cannot locate landing pad(s)");
```

- EN: Declares or implements routines including `assert`. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里较值得关注的符号包括 `assert`。

### Lines 172-180

```cpp
        if (LPA->getLayoutIndex() != LPB->getLayoutIndex())
          return false;
      }
    }
  }

  return BC.MIB->equals(InstA, InstB, Comp);
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 181-190

```cpp
/// Returns true if this function has identical code and CFG with
/// the given function \p BF.
///
/// If \p CongruentSymbols is set to true, then symbolic operands that reference
/// potentially identical but different functions are ignored during the
/// comparison.
static bool isIdenticalWith(const BinaryFunction &A, const BinaryFunction &B,
                            bool CongruentSymbols) {
  assert(A.hasCFG() && B.hasCFG() && "both functions should have CFG");
```

- EN: Declares or implements routines including `assert`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `assert`。

### Lines 191-200

```cpp
  // Hot text mover functions should not be folded. They need to stay in their
  // original section to avoid being placed on hot/huge pages.
  if (opts::isHotTextMover(A) || opts::isHotTextMover(B))
    return false;

  // Compare the two functions, one basic block at a time.
  // Currently we require two identical basic blocks to have identical
  // instruction sequences and the same index in their corresponding
  // functions. The latter is important for CFG equality.
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 201-210

```cpp
  if (A.getLayout().block_size() != B.getLayout().block_size())
    return false;

  // Comparing multi-entry functions could be non-trivial.
  if (A.isMultiEntry() || B.isMultiEntry())
    return false;

  if (A.hasIslandsInfo() || B.hasIslandsInfo())
    return false;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 211-221

```cpp
  // Process both functions in either DFS or existing order.
  SmallVector<const BinaryBasicBlock *, 0> OrderA;
  SmallVector<const BinaryBasicBlock *, 0> OrderB;
  if (opts::ICFUseDFS) {
    copy(A.dfs(), std::back_inserter(OrderA));
    copy(B.dfs(), std::back_inserter(OrderB));
  } else {
    copy(A.getLayout().blocks(), std::back_inserter(OrderA));
    copy(B.getLayout().blocks(), std::back_inserter(OrderB));
  }
```

- EN: Declares or implements routines including `copy`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `copy`.
- CN: 这里声明或实现函数，例如 `copy`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `copy`。

### Lines 222-230

```cpp
  const BinaryContext &BC = A.getBinaryContext();

  auto BBI = OrderB.begin();
  for (const BinaryBasicBlock *BB : OrderA) {
    const BinaryBasicBlock *OtherBB = *BBI;

    if (BB->getLayoutIndex() != OtherBB->getLayoutIndex())
      return false;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 231-243

```cpp
    // Compare successor basic blocks.
    // NOTE: the comparison for jump tables is only partially verified here.
    if (BB->succ_size() != OtherBB->succ_size())
      return false;

    auto SuccBBI = OtherBB->succ_begin();
    for (const BinaryBasicBlock *SuccBB : BB->successors()) {
      const BinaryBasicBlock *SuccOtherBB = *SuccBBI;
      if (SuccBB->getLayoutIndex() != SuccOtherBB->getLayoutIndex())
        return false;
      ++SuccBBI;
    }
```

- EN: Declares or implements routines including `succ_begin`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `succ_begin`.
- CN: 这里声明或实现函数，例如 `succ_begin`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `succ_begin`。

### Lines 244-253

```cpp
    // Compare all instructions including pseudos.
    auto I = BB->begin(), E = BB->end();
    auto OtherI = OtherBB->begin(), OtherE = OtherBB->end();
    while (I != E && OtherI != OtherE) {
      // Compare symbols.
      auto AreSymbolsIdentical = [&](const MCSymbol *SymbolA,
                                     const MCSymbol *SymbolB) {
        if (SymbolA == SymbolB)
          return true;
```

- EN: Declares or implements routines including `begin`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `begin`.
- CN: 这里声明或实现函数，例如 `begin`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `begin`。

### Lines 254-261

```cpp
        // All local symbols are considered identical since they affect a
        // control flow and we check the control flow separately.
        // If a local symbol is escaped, then the function (potentially) has
        // multiple entry points and we exclude such functions from
        // comparison.
        if (SymbolA->isTemporary() && SymbolB->isTemporary())
          return true;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 262-277

```cpp
        // Compare symbols as functions.
        uint64_t EntryIDA = 0;
        uint64_t EntryIDB = 0;
        const BinaryFunction *FunctionA =
            BC.getFunctionForSymbol(SymbolA, &EntryIDA);
        const BinaryFunction *FunctionB =
            BC.getFunctionForSymbol(SymbolB, &EntryIDB);
        if (FunctionA && EntryIDA)
          FunctionA = nullptr;
        if (FunctionB && EntryIDB)
          FunctionB = nullptr;
        if (FunctionA && FunctionB) {
          // Self-referencing functions and recursive calls.
          if (FunctionA == &A && FunctionB == &B)
            return true;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 278-285

```cpp
          // Functions with different hash values can never become identical,
          // hence A and B are different.
          if (CongruentSymbols)
            return FunctionA->getHash() == FunctionB->getHash();

          return FunctionA == FunctionB;
        }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 286-297

```cpp
        // One of the symbols represents a function, the other one does not.
        if (FunctionA != FunctionB)
          return false;

        // Check if symbols are jump tables.
        const BinaryData *SIA = BC.getBinaryDataByName(SymbolA->getName());
        if (!SIA)
          return false;
        const BinaryData *SIB = BC.getBinaryDataByName(SymbolB->getName());
        if (!SIB)
          return false;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 298-305

```cpp
        assert((SIA->getAddress() != SIB->getAddress()) &&
               "different symbols should not have the same value");

        const JumpTable *JumpTableA =
            A.getJumpTableContainingAddress(SIA->getAddress());
        if (!JumpTableA)
          return false;
```

- EN: Declares or implements routines including `assert`. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里较值得关注的符号包括 `assert`。

### Lines 306-314

```cpp
        const JumpTable *JumpTableB =
            B.getJumpTableContainingAddress(SIB->getAddress());
        if (!JumpTableB)
          return false;

        if ((SIA->getAddress() - JumpTableA->getAddress()) !=
            (SIB->getAddress() - JumpTableB->getAddress()))
          return false;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 315-325

```cpp
        return equalJumpTables(*JumpTableA, *JumpTableB, A, B);
      };

      if (!isInstrEquivalentWith(*I, *BB, *OtherI, *OtherBB,
                                 AreSymbolsIdentical))
        return false;

      ++I;
      ++OtherI;
    }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 326-335

```cpp
    // One of the identical blocks may have a trailing unconditional jump that
    // is ignored for CFG purposes.
    const MCInst *TrailingInstr =
        (I != E ? &(*I) : (OtherI != OtherE ? &(*OtherI) : nullptr));
    if (TrailingInstr && !BC.MIB->isUnconditionalBranch(*TrailingInstr))
      return false;

    ++BBI;
  }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 336-344

```cpp
  // Compare exceptions action tables.
  if (A.getLSDAActionTable() != B.getLSDAActionTable() ||
      A.getLSDATypeTable() != B.getLSDATypeTable() ||
      A.getLSDATypeIndexTable() != B.getLSDATypeIndexTable())
    return false;

  return true;
}
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 345-358

```cpp
// This hash table is used to identify identical functions. It maps
// a function to a bucket of functions identical to it.
struct KeyHash {
  size_t operator()(const BinaryFunction *F) const { return F->getHash(); }
};

/// Identify two congruent functions. Two functions are considered congruent,
/// if they are identical/equal except for some of their instruction operands
/// that reference potentially identical functions, i.e. functions that could
/// be folded later. Congruent functions are candidates for folding in our
/// iterative ICF algorithm.
///
/// Congruent functions are required to have identical hash.
struct KeyCongruent {
```

- EN: Introduces type definitions such as `KeyHash`, `KeyCongruent`. Declares or implements routines including `operator`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `KeyHash`, `KeyCongruent`, `operator`.
- CN: 这里引入类型定义，例如 `KeyHash`, `KeyCongruent`。这里声明或实现函数，例如 `operator`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `KeyHash`, `KeyCongruent`, `operator`。

### Lines 359-373

```cpp
  bool operator()(const BinaryFunction *A, const BinaryFunction *B) const {
    if (A == B)
      return true;
    return isIdenticalWith(*A, *B, /*CongruentSymbols=*/true);
  }
};

struct KeyEqual {
  bool operator()(const BinaryFunction *A, const BinaryFunction *B) const {
    if (A == B)
      return true;
    return isIdenticalWith(*A, *B, /*CongruentSymbols=*/false);
  }
};
```

- EN: Introduces type definitions such as `KeyEqual`. Declares or implements routines including `operator`. Notable symbols here include `KeyEqual`, `operator`.
- CN: 这里引入类型定义，例如 `KeyEqual`。这里声明或实现函数，例如 `operator`。这里较值得关注的符号包括 `KeyEqual`, `operator`。

### Lines 374-381

```cpp
typedef std::unordered_map<BinaryFunction *, std::set<BinaryFunction *>,
                           KeyHash, KeyCongruent>
    CongruentBucketsMap;

typedef std::unordered_map<BinaryFunction *, BinaryFunctionListType, KeyHash,
                           KeyEqual>
    IdenticalBucketsMap;
```

- EN: Continues the surrounding implementation, wiring declarations, state, and helper logic together.
- CN: 这里继续展开周边实现，衔接声明、状态与辅助逻辑。

### Lines 382-395

```cpp
namespace llvm {
namespace bolt {
void IdenticalCodeFolding::initVTableReferences(const BinaryContext &BC) {
  for (const auto &[Address, Data] : BC.getBinaryData()) {
    // Filter out all symbols that are not vtables.
    if (!Data->getName().starts_with("_ZTV") && // vtable
        !Data->getName().starts_with("_ZTCN"))  // construction vtable
      continue;
    for (uint64_t I = Address, End = I + Data->getSize(); I < End;
         I += VTableAddressGranularity)
      setAddressUsedInVTable(I);
  }
}
```

- EN: Works inside namespace scope `llvm`, `bolt` to organize symbols. Declares or implements routines including `initVTableReferences`, `getName`, `setAddressUsedInVTable`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `initVTableReferences`, `getName`, `setAddressUsedInVTable`, `llvm`, `bolt`.
- CN: 这里位于命名空间 `llvm`, `bolt` 中，用于组织符号作用域。这里声明或实现函数，例如 `initVTableReferences`, `getName`, `setAddressUsedInVTable`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `initVTableReferences`, `getName`, `setAddressUsedInVTable`, `llvm`, `bolt`。

### Lines 396-413

```cpp
void IdenticalCodeFolding::analyzeDataRelocations(BinaryContext &BC) {
  initVTableReferences(BC);
  // For static relocations there should be a symbol for function references.
  for (const BinarySection &Sec : BC.sections()) {
    if (!Sec.hasSectionRef() || !Sec.isData())
      continue;
    for (const auto &Rel : Sec.relocations()) {
      const uint64_t RelAddr = Rel.Offset + Sec.getAddress();
      if (isAddressInVTable(RelAddr))
        continue;
      if (BinaryFunction *BF = BC.getFunctionForSymbol(Rel.Symbol))
        BF->setHasAddressTaken(true);
    }
    // For dynamic relocations there are two cases:
    // 1: No symbol and only addend.
    // 2: There is a symbol, but it does not references a function in a binary.
    for (const auto &Rel : Sec.dynamicRelocations()) {
      const uint64_t RelAddr = Rel.Offset + Sec.getAddress();
```

- EN: Declares or implements routines including `analyzeDataRelocations`, `initVTableReferences`, `setHasAddressTaken`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `analyzeDataRelocations`, `initVTableReferences`, `setHasAddressTaken`.
- CN: 这里声明或实现函数，例如 `analyzeDataRelocations`, `initVTableReferences`, `setHasAddressTaken`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `analyzeDataRelocations`, `initVTableReferences`, `setHasAddressTaken`。

### Lines 414-421

```cpp
      if (isAddressInVTable(RelAddr))
        continue;
      if (BinaryFunction *BF = BC.getBinaryFunctionAtAddress(Rel.Addend))
        BF->setHasAddressTaken(true);
    }
  }
}
```

- EN: Declares or implements routines including `setHasAddressTaken`. Notable symbols here include `setHasAddressTaken`.
- CN: 这里声明或实现函数，例如 `setHasAddressTaken`。这里较值得关注的符号包括 `setHasAddressTaken`。

### Lines 422-434

```cpp
void IdenticalCodeFolding::analyzeFunctions(BinaryContext &BC) {
  ParallelUtilities::WorkFuncTy WorkFun = [&](BinaryFunction &BF) {
    for (const BinaryBasicBlock &BB : BF)
      for (const MCInst &Inst : BB)
        if (!(BC.MIB->isCall(Inst) || BC.MIB->isBranch(Inst)))
          BF.analyzeInstructionForFuncReference(Inst);
  };
  ParallelUtilities::PredicateTy SkipFunc =
      [&](const BinaryFunction &BF) -> bool { return !BF.hasCFG(); };
  ParallelUtilities::runOnEachFunction(
      BC, ParallelUtilities::SchedulingPolicy::SP_INST_LINEAR, WorkFun,
      SkipFunc, "markUnsafe");
```

- EN: Declares or implements routines including `analyzeFunctions`. Notable symbols here include `analyzeFunctions`.
- CN: 这里声明或实现函数，例如 `analyzeFunctions`。这里较值得关注的符号包括 `analyzeFunctions`。

### Lines 435-444

```cpp
  LLVM_DEBUG({
    for (const auto &BFIter : BC.getBinaryFunctions()) {
      if (!BFIter.second.hasAddressTaken())
        continue;
      dbgs() << "BOLT-DEBUG: skipping function with reference taken "
             << BFIter.second.getOneName() << '\n';
    }
  });
}
```

- EN: Declares or implements routines including `dbgs`. Notable symbols here include `dbgs`.
- CN: 这里声明或实现函数，例如 `dbgs`。这里较值得关注的符号包括 `dbgs`。

### Lines 445-455

```cpp
void IdenticalCodeFolding::markFunctionsUnsafeToFold(BinaryContext &BC) {
  NamedRegionTimer MarkFunctionsUnsafeToFoldTimer(
      "markFunctionsUnsafeToFold", "markFunctionsUnsafeToFold", "ICF breakdown",
      "ICF breakdown", opts::TimeICF);
  if (!BC.isX86() && !BC.isAArch64())
    BC.outs()
        << "BOLT-WARNING: safe ICF is only supported for x86 and AArch64\n";
  analyzeDataRelocations(BC);
  analyzeFunctions(BC);
}
```

- EN: Declares or implements routines including `markFunctionsUnsafeToFold`, `analyzeDataRelocations`, `analyzeFunctions`. Notable symbols here include `markFunctionsUnsafeToFold`, `analyzeDataRelocations`, `analyzeFunctions`.
- CN: 这里声明或实现函数，例如 `markFunctionsUnsafeToFold`, `analyzeDataRelocations`, `analyzeFunctions`。这里较值得关注的符号包括 `markFunctionsUnsafeToFold`, `analyzeDataRelocations`, `analyzeFunctions`。

### Lines 456-464

```cpp
Error IdenticalCodeFolding::runOnFunctions(BinaryContext &BC) {
  const size_t OriginalFunctionCount = BC.getBinaryFunctions().size();
  uint64_t NumFunctionsFolded = 0;
  std::atomic<uint64_t> NumJTFunctionsFolded{0};
  std::atomic<uint64_t> BytesSavedEstimate{0};
  std::atomic<uint64_t> NumCalled{0};
  std::atomic<uint64_t> NumFoldedLastIteration{0};
  CongruentBucketsMap CongruentBuckets;
```

- EN: Declares or implements routines including `runOnFunctions`. Notable symbols here include `runOnFunctions`.
- CN: 这里声明或实现函数，例如 `runOnFunctions`。这里较值得关注的符号包括 `runOnFunctions`。

### Lines 465-475

```cpp
  // Hash all the functions
  auto hashFunctions = [&]() {
    NamedRegionTimer HashFunctionsTimer("hashing", "hashing", "ICF breakdown",
                                        "ICF breakdown", opts::TimeICF);
    ParallelUtilities::WorkFuncTy WorkFun = [&](BinaryFunction &BF) {
      // Make sure indices are in-order.
      if (opts::ICFUseDFS)
        BF.getLayout().updateLayoutIndices(BF.dfs());
      else
        BF.getLayout().updateLayoutIndices();
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 476-486

```cpp
      // Pre-compute hash before pushing into hashtable.
      // Hash instruction operands to minimize hash collisions.
      BF.computeHash(
          opts::ICFUseDFS, HashFunction::Default,
          [&BC](const MCOperand &Op) { return hashInstOperand(BC, Op); });
    };

    ParallelUtilities::PredicateTy SkipFunc = [&](const BinaryFunction &BF) {
      return !shouldOptimize(BF);
    };
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 487-504

```cpp
    ParallelUtilities::runOnEachFunction(
        BC, ParallelUtilities::SchedulingPolicy::SP_TRIVIAL, WorkFun, SkipFunc,
        "hashFunctions", /*ForceSequential*/ false, 2);
  };

  // Creates buckets with congruent functions - functions that potentially
  // could  be folded.
  auto createCongruentBuckets = [&]() {
    NamedRegionTimer CongruentBucketsTimer("congruent buckets",
                                           "congruent buckets", "ICF breakdown",
                                           "ICF breakdown", opts::TimeICF);
    for (auto &BFI : BC.getBinaryFunctions()) {
      BinaryFunction &BF = BFI.second;
      if (!shouldOptimize(BF))
        continue;
      CongruentBuckets[&BF].emplace(&BF);
    }
  };
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 505-514

```cpp

  // Partition each set of congruent functions into sets of identical functions
  // and fold them
  auto performFoldingPass = [&]() {
    NamedRegionTimer FoldingPassesTimer("folding passes", "folding passes",
                                        "ICF breakdown", "ICF breakdown",
                                        opts::TimeICF);
    Timer SinglePass("single fold pass", "single fold pass");
    LLVM_DEBUG(SinglePass.startTimer());
```

- EN: Declares or implements routines including `SinglePass`, `LLVM_DEBUG`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `SinglePass`, `LLVM_DEBUG`.
- CN: 这里声明或实现函数，例如 `SinglePass`, `LLVM_DEBUG`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `SinglePass`, `LLVM_DEBUG`。

### Lines 515-523

```cpp
    ThreadPoolInterface *ThPool;
    if (!opts::NoThreads)
      ThPool = &ParallelUtilities::getThreadPool();

    // Fold identical functions within a single congruent bucket
    auto processSingleBucket = [&](std::set<BinaryFunction *> &Candidates) {
      Timer T("folding single congruent list", "folding single congruent list");
      LLVM_DEBUG(T.startTimer());
```

- EN: Declares or implements routines including `getThreadPool`, `T`, `LLVM_DEBUG`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getThreadPool`, `T`, `LLVM_DEBUG`.
- CN: 这里声明或实现函数，例如 `getThreadPool`, `T`, `LLVM_DEBUG`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getThreadPool`, `T`, `LLVM_DEBUG`。

### Lines 524-535

```cpp
      // Identical functions go into the same bucket.
      IdenticalBucketsMap IdenticalBuckets;
      for (BinaryFunction *BF : Candidates) {
        IdenticalBuckets[BF].emplace_back(BF);
      }

      for (auto &IBI : IdenticalBuckets) {
        // Functions identified as identical.
        BinaryFunctionListType &Twins = IBI.second;
        if (Twins.size() < 2)
          continue;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 536-550

```cpp
        // Fold functions. Keep the order consistent across invocations with
        // different options.
        llvm::stable_sort(
            Twins, [](const BinaryFunction *A, const BinaryFunction *B) {
              return A->getFunctionNumber() < B->getFunctionNumber();
            });

        BinaryFunction *ParentBF = Twins[0];
        if (!ParentBF->hasFunctionsFoldedInto())
          NumCalled += ParentBF->getKnownExecutionCount();
        for (unsigned I = 1; I < Twins.size(); ++I) {
          BinaryFunction *ChildBF = Twins[I];
          LLVM_DEBUG(dbgs() << "BOLT-DEBUG: folding " << *ChildBF << " into "
                            << *ParentBF << '\n');
```

- EN: Declares or implements routines including `getKnownExecutionCount`, `LLVM_DEBUG`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getKnownExecutionCount`, `LLVM_DEBUG`.
- CN: 这里声明或实现函数，例如 `getKnownExecutionCount`, `LLVM_DEBUG`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getKnownExecutionCount`, `LLVM_DEBUG`。

### Lines 551-562

```cpp
          // Remove child function from the list of candidates.
          auto FI = Candidates.find(ChildBF);
          assert(FI != Candidates.end() &&
                 "function expected to be in the set");
          Candidates.erase(FI);

          // Fold the function and remove from the list of processed functions.
          BytesSavedEstimate += ChildBF->getSize();
          if (!ChildBF->hasFunctionsFoldedInto())
            NumCalled += ChildBF->getKnownExecutionCount();
          BC.foldFunction(*ChildBF, *ParentBF);
```

- EN: Declares or implements routines including `assert`, `getSize`, `getKnownExecutionCount`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `assert`, `getSize`, `getKnownExecutionCount`.
- CN: 这里声明或实现函数，例如 `assert`, `getSize`, `getKnownExecutionCount`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `assert`, `getSize`, `getKnownExecutionCount`。

### Lines 563-572

```cpp
          ++NumFoldedLastIteration;

          if (ParentBF->hasJumpTables())
            ++NumJTFunctionsFolded;
        }
      }

      LLVM_DEBUG(T.stopTimer());
    };
```

- EN: Declares or implements routines including `LLVM_DEBUG`. Notable symbols here include `LLVM_DEBUG`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`。这里较值得关注的符号包括 `LLVM_DEBUG`。

### Lines 573-584

```cpp
    // Create a task for each congruent bucket
    for (auto &Entry : CongruentBuckets) {
      std::set<BinaryFunction *> &Bucket = Entry.second;
      if (Bucket.size() < 2)
        continue;

      if (opts::NoThreads)
        processSingleBucket(Bucket);
      else
        ThPool->async(processSingleBucket, std::ref(Bucket));
    }
```

- EN: Declares or implements routines including `processSingleBucket`, `async`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `processSingleBucket`, `async`.
- CN: 这里声明或实现函数，例如 `processSingleBucket`, `async`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `processSingleBucket`, `async`。

### Lines 585-594

```cpp
    if (!opts::NoThreads)
      ThPool->wait();

    LLVM_DEBUG(SinglePass.stopTimer());
  };
  if (opts::ICF == ICFLevel::Safe)
    markFunctionsUnsafeToFold(BC);
  hashFunctions();
  createCongruentBuckets();
```

- EN: Declares or implements routines including `wait`, `LLVM_DEBUG`, `markFunctionsUnsafeToFold`, `hashFunctions`, `createCongruentBuckets`. Notable symbols here include `wait`, `LLVM_DEBUG`, `markFunctionsUnsafeToFold`, `hashFunctions`, `createCongruentBuckets`.
- CN: 这里声明或实现函数，例如 `wait`, `LLVM_DEBUG`, `markFunctionsUnsafeToFold`, `hashFunctions`, `createCongruentBuckets`。这里较值得关注的符号包括 `wait`, `LLVM_DEBUG`, `markFunctionsUnsafeToFold`, `hashFunctions`, `createCongruentBuckets`。

### Lines 595-602

```cpp
  unsigned Iteration = 1;
  // We repeat the pass until no new modifications happen.
  do {
    NumFoldedLastIteration = 0;
    LLVM_DEBUG(dbgs() << "BOLT-DEBUG: ICF iteration " << Iteration << "...\n");

    performFoldingPass();
```

- EN: Declares or implements routines including `LLVM_DEBUG`, `performFoldingPass`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `LLVM_DEBUG`, `performFoldingPass`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`, `performFoldingPass`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `LLVM_DEBUG`, `performFoldingPass`。

### Lines 603-620

```cpp
    NumFunctionsFolded += NumFoldedLastIteration;
    ++Iteration;

  } while (NumFoldedLastIteration > 0);

  // Flatten folded function chains so FoldedIntoFunction always points
  // to the root parent.
  for (auto &BFI : BC.getBinaryFunctions()) {
    BinaryFunction &BF = BFI.second;
    if (!BF.isFolded())
      continue;
    BinaryFunction *Parent = BF.getFoldedIntoFunction();
    while (Parent->isFolded())
      Parent = Parent->getFoldedIntoFunction();
    if (Parent != BF.getFoldedIntoFunction())
      BF.setFolded(Parent);
  }
```

- EN: Declares or implements routines including `while`, `getFoldedIntoFunction`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `while`, `getFoldedIntoFunction`.
- CN: 这里声明或实现函数，例如 `while`, `getFoldedIntoFunction`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `while`, `getFoldedIntoFunction`。

### Lines 621-638

```cpp
  LLVM_DEBUG({
    // Print functions that are congruent but not identical.
    for (auto &CBI : CongruentBuckets) {
      std::set<BinaryFunction *> &Candidates = CBI.second;
      if (Candidates.size() < 2)
        continue;
      dbgs() << "BOLT-DEBUG: the following " << Candidates.size()
             << " functions (each of size " << (*Candidates.begin())->getSize()
             << " bytes) are congruent but not identical:\n";
      for (BinaryFunction *BF : Candidates) {
        dbgs() << "  " << *BF;
        if (BF->getKnownExecutionCount())
          dbgs() << " (executed " << BF->getKnownExecutionCount() << " times)";
        dbgs() << '\n';
      }
    }
  });
```

- EN: Declares or implements routines including `dbgs`, `functions`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `dbgs`, `functions`.
- CN: 这里声明或实现函数，例如 `dbgs`, `functions`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `dbgs`, `functions`。

### Lines 639-648

```cpp
  if (NumFunctionsFolded)
    BC.outs() << "BOLT-INFO: ICF folded " << NumFunctionsFolded << " out of "
              << OriginalFunctionCount << " functions in " << Iteration
              << " passes. " << NumJTFunctionsFolded
              << " functions had jump tables.\n"
              << "BOLT-INFO: Removing all identical functions will save "
              << format("%.2lf", (double)BytesSavedEstimate / 1024)
              << " KB of code space. Folded functions were called " << NumCalled
              << " times based on profile.\n";
```

- EN: Declares or implements routines including `format`. Notable symbols here include `format`.
- CN: 这里声明或实现函数，例如 `format`。这里较值得关注的符号包括 `format`。

### Lines 649-653

```cpp
  return Error::success();
}

} // namespace bolt
} // namespace llvm
```

- EN: Works inside namespace scope `bolt`, `llvm` to organize symbols. Contains local control flow that updates state or selects among execution branches. Notable symbols here include `bolt`, `llvm`.
- CN: 这里位于命名空间 `bolt`, `llvm` 中，用于组织符号作用域。这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里较值得关注的符号包括 `bolt`, `llvm`。

## Key Concepts / 关键概念

- `Compare`: class or struct interface / 类或结构体接口
- `KeyHash`: class or struct interface / 类或结构体接口
- `KeyCongruent`: class or struct interface / 类或结构体接口
- `KeyEqual`: class or struct interface / 类或结构体接口
- `isHotTextMover`: function or method entry point / 函数或方法入口
- `ICFUseDFS`: function or method entry point / 函数或方法入口
- `cat`: function or method entry point / 函数或方法入口
- `desc`: function or method entry point / 函数或方法入口

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Passes/IdenticalCodeFolding.h`, `bolt/Core/HashUtilities.h`, `bolt/Core/ParallelUtilities.h`
- LLVM headers / LLVM 头文件: `llvm/ADT/SmallVector.h`, `llvm/Support/CommandLine.h`, `llvm/Support/FormatVariadic.h`, `llvm/Support/ThreadPool.h`, `llvm/Support/Timer.h`
- System headers / 系统头文件: `atomic`, `iterator`, `map`, `set`, `unordered_map`
- Directory context / 目录上下文: `bolt/lib/Passes` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Passes` 下的相邻文件通常与本文件协作组成对应子系统
