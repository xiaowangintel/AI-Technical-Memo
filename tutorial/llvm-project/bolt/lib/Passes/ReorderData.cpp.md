# ReorderData.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Passes/ReorderData.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: Reordering of section data. It also sits in code that implements BOLT optimization and transformation passes. / 该文件实现 BOLT 优化与变换 Pass。 源码头部说明其职责是：Reordering of section data。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12

```cpp
//===- bolt/Passes/ReorderSection.cpp - Reordering of section data --------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements ReorderData class.
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 13-21

```cpp
// TODO:
// - make sure writable data isn't put on same cache line unless temporally
// local
// - estimate temporal locality by looking at CFG?

#include "bolt/Passes/ReorderData.h"
#include "llvm/ADT/MapVector.h"
#include <algorithm>
```

- EN: Pulls in 3 header(s) from local project, LLVM, system dependencies needed by this range. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 3 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里的注释记录了设计假设、不变量或使用说明。

### Lines 22-32

```cpp
#undef  DEBUG_TYPE
#define DEBUG_TYPE "reorder-data"

using namespace llvm;
using namespace bolt;

namespace opts {
extern cl::OptionCategory BoltCategory;
extern cl::OptionCategory BoltOptCategory;
extern cl::opt<JumpTableSupportLevel> JumpTables;
```

- EN: Establishes header-guard or prologue macros so the file can be compiled safely. Works inside namespace scope `llvm`, `bolt`, `opts` to organize symbols. Defines macros such as `DEBUG_TYPE` for constants or compile-time switches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里位于命名空间 `llvm`, `bolt`, `opts` 中，用于组织符号作用域。这里定义宏 `DEBUG_TYPE`，用于常量或编译期开关。这里的注释记录了设计假设、不变量或使用说明。

### Lines 33-44

```cpp
static cl::opt<bool>
    PrintReorderedData("print-reordered-data",
                       cl::desc("print section contents after reordering"),
                       cl::Hidden, cl::cat(BoltCategory));

cl::list<std::string>
ReorderData("reorder-data",
  cl::CommaSeparated,
  cl::desc("list of sections to reorder"),
  cl::value_desc("section1,section2,section3,..."),
  cl::cat(BoltOptCategory));
```

- EN: Declares or implements routines including `desc`, `cat`, `value_desc`. Notable symbols here include `desc`, `cat`, `value_desc`.
- CN: 这里声明或实现函数，例如 `desc`, `cat`, `value_desc`。这里较值得关注的符号包括 `desc`, `cat`, `value_desc`。

### Lines 45-62

```cpp
enum ReorderAlgo : char {
  REORDER_COUNT         = 0,
  REORDER_FUNCS         = 1
};

static cl::opt<ReorderAlgo>
ReorderAlgorithm("reorder-data-algo",
  cl::desc("algorithm used to reorder data sections"),
  cl::init(REORDER_COUNT),
  cl::values(
    clEnumValN(REORDER_COUNT,
      "count",
      "sort hot data by read counts"),
    clEnumValN(REORDER_FUNCS,
      "funcs",
      "sort hot data by hot function usage and count")),
  cl::ZeroOrMore,
  cl::cat(BoltOptCategory));
```

- EN: Defines enumerations such as `ReorderAlgo` to encode states or modes. Declares or implements routines including `desc`, `init`, `cat`. Notable symbols here include `ReorderAlgo`, `desc`, `init`, `cat`.
- CN: 这里定义枚举 `ReorderAlgo`，用于表达状态或模式。这里声明或实现函数，例如 `desc`, `init`, `cat`。这里较值得关注的符号包括 `ReorderAlgo`, `desc`, `init`, `cat`。

### Lines 63-73

```cpp

static cl::opt<unsigned>
    ReorderDataMaxSymbols("reorder-data-max-symbols",
                          cl::desc("maximum number of symbols to reorder"),
                          cl::init(std::numeric_limits<unsigned>::max()),
                          cl::cat(BoltOptCategory));

static cl::opt<unsigned> ReorderDataMaxBytes(
    "reorder-data-max-bytes", cl::desc("maximum number of bytes to reorder"),
    cl::init(std::numeric_limits<unsigned>::max()), cl::cat(BoltOptCategory));
```

- EN: Declares or implements routines including `desc`, `init`, `cat`. Notable symbols here include `desc`, `init`, `cat`.
- CN: 这里声明或实现函数，例如 `desc`, `init`, `cat`。这里较值得关注的符号包括 `desc`, `init`, `cat`。

### Lines 74-81

```cpp
static cl::list<std::string>
ReorderSymbols("reorder-symbols",
  cl::CommaSeparated,
  cl::desc("list of symbol names that can be reordered"),
  cl::value_desc("symbol1,symbol2,symbol3,..."),
  cl::Hidden,
  cl::cat(BoltCategory));
```

- EN: Declares or implements routines including `desc`, `value_desc`, `cat`. Notable symbols here include `desc`, `value_desc`, `cat`.
- CN: 这里声明或实现函数，例如 `desc`, `value_desc`, `cat`。这里较值得关注的符号包括 `desc`, `value_desc`, `cat`。

### Lines 82-89

```cpp
static cl::list<std::string>
SkipSymbols("reorder-skip-symbols",
  cl::CommaSeparated,
  cl::desc("list of symbol names that cannot be reordered"),
  cl::value_desc("symbol1,symbol2,symbol3,..."),
  cl::Hidden,
  cl::cat(BoltCategory));
```

- EN: Declares or implements routines including `desc`, `value_desc`, `cat`. Notable symbols here include `desc`, `value_desc`, `cat`.
- CN: 这里声明或实现函数，例如 `desc`, `value_desc`, `cat`。这里较值得关注的符号包括 `desc`, `value_desc`, `cat`。

### Lines 90-98

```cpp
static cl::opt<bool> ReorderInplace("reorder-data-inplace",
                                    cl::desc("reorder data sections in place"),

                                    cl::cat(BoltOptCategory));
}

namespace llvm {
namespace bolt {
```

- EN: Works inside namespace scope `llvm`, `bolt` to organize symbols. Declares or implements routines including `desc`, `cat`. Notable symbols here include `desc`, `cat`, `llvm`, `bolt`.
- CN: 这里位于命名空间 `llvm`, `bolt` 中，用于组织符号作用域。这里声明或实现函数，例如 `desc`, `cat`。这里较值得关注的符号包括 `desc`, `cat`, `llvm`, `bolt`。

### Lines 99-108

```cpp
namespace {

static constexpr uint16_t MinAlignment = 16;

bool isSupported(const BinarySection &BS) { return BS.isData() && !BS.isTLS(); }

bool filterSymbol(const BinaryData *BD) {
  if (!BD->isAtomic() || BD->isJumpTable() || !BD->isMoveable())
    return false;
```

- EN: Declares or implements routines including `isSupported`, `filterSymbol`. Notable symbols here include `isSupported`, `filterSymbol`.
- CN: 这里声明或实现函数，例如 `isSupported`, `filterSymbol`。这里较值得关注的符号包括 `isSupported`, `filterSymbol`。

### Lines 109-116

```cpp
  bool IsValid = true;

  if (!opts::ReorderSymbols.empty()) {
    IsValid = llvm::any_of(opts::ReorderSymbols, [&](const std::string &Name) {
      return BD->hasName(Name);
    });
  }
```

- EN: Declares or implements routines including `any_of`. Notable symbols here include `any_of`.
- CN: 这里声明或实现函数，例如 `any_of`。这里较值得关注的符号包括 `any_of`。

### Lines 117-128

```cpp
  if (!IsValid)
    return false;

  if (!opts::SkipSymbols.empty()) {
    for (const std::string &Name : opts::SkipSymbols) {
      if (BD->hasName(Name)) {
        IsValid = false;
        break;
      }
    }
  }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 129-143

```cpp
  return IsValid;
}

} // namespace

using DataOrder = ReorderData::DataOrder;

void ReorderData::printOrder(BinaryContext &BC, const BinarySection &Section,
                             DataOrder::const_iterator Begin,
                             DataOrder::const_iterator End) const {
  uint64_t TotalSize = 0;
  bool PrintHeader = false;
  while (Begin != End) {
    const BinaryData *BD = Begin->first;
```

- EN: Works inside namespace scope `using` to organize symbols. Contains local control flow that updates state or selects among execution branches. Notable symbols here include `using`.
- CN: 这里位于命名空间 `using` 中，用于组织符号作用域。这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里较值得关注的符号包括 `using`。

### Lines 144-153

```cpp
    if (!PrintHeader) {
      BC.outs() << "BOLT-INFO: Hot global symbols for " << Section.getName()
                << ":\n";
      PrintHeader = true;
    }

    BC.outs() << "BOLT-INFO: " << *BD << ", moveable=" << BD->isMoveable()
              << format(", weight=%.5f\n",
                        double(Begin->second) / BD->getSize());
```

- EN: Declares or implements routines including `double`. Notable symbols here include `double`.
- CN: 这里声明或实现函数，例如 `double`。这里较值得关注的符号包括 `double`。

### Lines 154-171

```cpp
    TotalSize += BD->getSize();
    ++Begin;
  }
  if (TotalSize)
    BC.outs() << "BOLT-INFO: Total hot symbol size = " << TotalSize << "\n";
}

DataOrder ReorderData::baseOrder(BinaryContext &BC,
                                 const BinarySection &Section) const {
  DataOrder Order;
  for (auto &Entry : BC.getBinaryDataForSection(Section)) {
    BinaryData *BD = Entry.second;
    if (!BD->isAtomic()) // skip sub-symbols
      continue;
    auto BDCI = BinaryDataCounts.find(BD);
    uint64_t BDCount = BDCI == BinaryDataCounts.end() ? 0 : BDCI->second;
    Order.emplace_back(BD, BDCount);
  }
```

- EN: Declares or implements routines including `getSize`. Notable symbols here include `getSize`.
- CN: 这里声明或实现函数，例如 `getSize`。这里较值得关注的符号包括 `getSize`。

### Lines 172-184

```cpp
  return Order;
}

void ReorderData::assignMemData(BinaryContext &BC) {
  // Map of sections (or heap/stack) to count/size.
  MapVector<StringRef, uint64_t> Counts;
  MapVector<StringRef, uint64_t> JumpTableCounts;
  uint64_t TotalCount = 0;
  for (auto &BFI : BC.getBinaryFunctions()) {
    const BinaryFunction &BF = BFI.second;
    if (!BF.hasMemoryProfile())
      continue;
```

- EN: Declares or implements routines including `assignMemData`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `assignMemData`.
- CN: 这里声明或实现函数，例如 `assignMemData`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `assignMemData`。

### Lines 185-192

```cpp
    for (const BinaryBasicBlock &BB : BF) {
      for (const MCInst &Inst : BB) {
        auto ErrorOrMemAccessProfile =
            BC.MIB->tryGetAnnotationAs<MemoryAccessProfile>(
                Inst, "MemoryAccessProfile");
        if (!ErrorOrMemAccessProfile)
          continue;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 193-210

```cpp
        const MemoryAccessProfile &MemAccessProfile =
            ErrorOrMemAccessProfile.get();
        for (const AddressAccess &AccessInfo :
             MemAccessProfile.AddressAccessInfo) {
          if (BinaryData *BD = AccessInfo.MemoryObject) {
            BinaryDataCounts[BD->getAtomicRoot()] += AccessInfo.Count;
            Counts[BD->getSectionName()] += AccessInfo.Count;
            if (BD->getAtomicRoot()->isJumpTable())
              JumpTableCounts[BD->getSectionName()] += AccessInfo.Count;
          } else {
            Counts["Heap/stack"] += AccessInfo.Count;
          }
          TotalCount += AccessInfo.Count;
        }
      }
    }
  }
```

- EN: Declares or implements routines including `getAtomicRoot`, `getSectionName`. Notable symbols here include `getAtomicRoot`, `getSectionName`.
- CN: 这里声明或实现函数，例如 `getAtomicRoot`, `getSectionName`。这里较值得关注的符号包括 `getAtomicRoot`, `getSectionName`。

### Lines 211-227

```cpp
  if (!Counts.empty()) {
    BC.outs() << "BOLT-INFO: Memory stats breakdown:\n";
    for (const auto &KV : Counts) {
      StringRef Section = KV.first;
      const uint64_t Count = KV.second;
      BC.outs() << "BOLT-INFO:   " << Section << " = " << Count
                << format(" (%.1f%%)\n", 100.0 * Count / TotalCount);
      if (JumpTableCounts.count(Section) != 0) {
        const uint64_t JTCount = JumpTableCounts[Section];
        BC.outs() << "BOLT-INFO:     jump tables = " << JTCount
                  << format(" (%.1f%%)\n", 100.0 * JTCount / Count);
      }
    }
    BC.outs() << "BOLT-INFO: Total memory events: " << TotalCount << "\n";
  }
}
```

- EN: Declares or implements routines including `format`. Notable symbols here include `format`.
- CN: 这里声明或实现函数，例如 `format`。这里较值得关注的符号包括 `format`。

### Lines 228-235

```cpp
/// Only consider moving data that is used by the hottest functions with
/// valid profiles.
std::pair<DataOrder, unsigned>
ReorderData::sortedByFunc(BinaryContext &BC, const BinarySection &Section,
                          std::map<uint64_t, BinaryFunction> &BFs) const {
  std::map<BinaryData *, std::set<BinaryFunction *>> BDtoFunc;
  std::map<BinaryData *, uint64_t> BDtoFuncCount;
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 236-248

```cpp
  auto dataUses = [&BC](const BinaryFunction &BF, bool OnlyHot) {
    std::set<BinaryData *> Uses;
    for (const BinaryBasicBlock &BB : BF) {
      if (OnlyHot && BB.isCold())
        continue;

      for (const MCInst &Inst : BB) {
        auto ErrorOrMemAccessProfile =
            BC.MIB->tryGetAnnotationAs<MemoryAccessProfile>(
                Inst, "MemoryAccessProfile");
        if (!ErrorOrMemAccessProfile)
          continue;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 249-260

```cpp
        const MemoryAccessProfile &MemAccessProfile =
            ErrorOrMemAccessProfile.get();
        for (const AddressAccess &AccessInfo :
             MemAccessProfile.AddressAccessInfo) {
          if (AccessInfo.MemoryObject)
            Uses.insert(AccessInfo.MemoryObject);
        }
      }
    }
    return Uses;
  };
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 261-272

```cpp
  for (auto &Entry : BFs) {
    BinaryFunction &BF = Entry.second;
    if (BF.hasValidProfile()) {
      for (BinaryData *BD : dataUses(BF, true)) {
        if (!BC.getFunctionForSymbol(BD->getSymbol())) {
          BDtoFunc[BD->getAtomicRoot()].insert(&BF);
          BDtoFuncCount[BD->getAtomicRoot()] += BF.getKnownExecutionCount();
        }
      }
    }
  }
```

- EN: Declares or implements routines including `getAtomicRoot`. Notable symbols here include `getAtomicRoot`.
- CN: 这里声明或实现函数，例如 `getAtomicRoot`。这里较值得关注的符号包括 `getAtomicRoot`。

### Lines 273-290

```cpp
  DataOrder Order = baseOrder(BC, Section);
  unsigned SplitPoint = Order.size();

  llvm::sort(
      Order,
      [&](const DataOrder::value_type &A, const DataOrder::value_type &B) {
        // Total execution counts of functions referencing BD.
        const uint64_t ACount = BDtoFuncCount[A.first];
        const uint64_t BCount = BDtoFuncCount[B.first];
        // Weight by number of loads/data size.
        const double AWeight = double(A.second) / A.first->getSize();
        const double BWeight = double(B.second) / B.first->getSize();
        return (ACount > BCount ||
                (ACount == BCount &&
                 (AWeight > BWeight ||
                  (AWeight == BWeight &&
                   A.first->getAddress() < B.first->getAddress()))));
      });
```

- EN: Declares or implements routines including `baseOrder`, `double`, `getAddress`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `baseOrder`, `double`, `getAddress`.
- CN: 这里声明或实现函数，例如 `baseOrder`, `double`, `getAddress`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `baseOrder`, `double`, `getAddress`。

### Lines 291-298

```cpp

  for (unsigned Idx = 0; Idx < Order.size(); ++Idx) {
    if (!BDtoFuncCount[Order[Idx].first]) {
      SplitPoint = Idx;
      break;
    }
  }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 299-307

```cpp
  return std::make_pair(Order, SplitPoint);
}

std::pair<DataOrder, unsigned>
ReorderData::sortedByCount(BinaryContext &BC,
                           const BinarySection &Section) const {
  DataOrder Order = baseOrder(BC, Section);
  unsigned SplitPoint = Order.size();
```

- EN: Declares or implements routines including `baseOrder`. Notable symbols here include `baseOrder`.
- CN: 这里声明或实现函数，例如 `baseOrder`。这里较值得关注的符号包括 `baseOrder`。

### Lines 308-319

```cpp
  llvm::sort(Order, [](const DataOrder::value_type &A,
                       const DataOrder::value_type &B) {
    // Weight by number of loads/data size.
    const double AWeight = double(A.second) / A.first->getSize();
    const double BWeight = double(B.second) / B.first->getSize();
    return (AWeight > BWeight ||
            (AWeight == BWeight &&
             (A.first->getSize() < B.first->getSize() ||
              (A.first->getSize() == B.first->getSize() &&
               A.first->getAddress() < B.first->getAddress()))));
  });
```

- EN: Declares or implements routines including `double`, `getAddress`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `double`, `getAddress`.
- CN: 这里声明或实现函数，例如 `double`, `getAddress`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `double`, `getAddress`。

### Lines 320-329

```cpp
  for (unsigned Idx = 0; Idx < Order.size(); ++Idx) {
    if (!Order[Idx].second) {
      SplitPoint = Idx;
      break;
    }
  }

  return std::make_pair(Order, SplitPoint);
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 330-341

```cpp
// TODO
// add option for cache-line alignment (or just use cache-line when section
// is writable)?
void ReorderData::setSectionOrder(BinaryContext &BC,
                                  BinarySection &OutputSection,
                                  DataOrder::iterator Begin,
                                  DataOrder::iterator End) {
  std::vector<BinaryData *> NewOrder;
  unsigned NumReordered = 0;
  uint64_t Offset = 0;
  uint64_t Count = 0;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 342-349

```cpp
  // Get the total count just for stats
  uint64_t TotalCount = 0;
  for (auto Itr = Begin; Itr != End; ++Itr)
    TotalCount += Itr->second;

  LLVM_DEBUG(dbgs() << "BOLT-DEBUG: setSectionOrder for "
                    << OutputSection.getName() << "\n");
```

- EN: Declares or implements routines including `LLVM_DEBUG`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `LLVM_DEBUG`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `LLVM_DEBUG`。

### Lines 350-364

```cpp
  for (; Begin != End; ++Begin) {
    BinaryData *BD = Begin->first;

    // We can't move certain symbols.
    if (!filterSymbol(BD))
      continue;

    ++NumReordered;
    if (NumReordered > opts::ReorderDataMaxSymbols) {
      if (!NewOrder.empty())
        LLVM_DEBUG(dbgs() << "BOLT-DEBUG: processing ending on symbol "
                          << *NewOrder.back() << "\n");
      break;
    }
```

- EN: Declares or implements routines including `LLVM_DEBUG`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `LLVM_DEBUG`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `LLVM_DEBUG`。

### Lines 365-374

```cpp
    uint16_t Alignment = std::max(BD->getAlignment(), MinAlignment);
    Offset = alignTo(Offset, Alignment);

    if ((Offset + BD->getSize()) > opts::ReorderDataMaxBytes) {
      if (!NewOrder.empty())
        LLVM_DEBUG(dbgs() << "BOLT-DEBUG: processing ending on symbol "
                          << *NewOrder.back() << "\n");
      break;
    }
```

- EN: Declares or implements routines including `max`, `alignTo`, `LLVM_DEBUG`. Notable symbols here include `max`, `alignTo`, `LLVM_DEBUG`.
- CN: 这里声明或实现函数，例如 `max`, `alignTo`, `LLVM_DEBUG`。这里较值得关注的符号包括 `max`, `alignTo`, `LLVM_DEBUG`。

### Lines 375-391

```cpp
    LLVM_DEBUG(dbgs() << "BOLT-DEBUG: " << BD->getName() << " @ 0x"
                      << Twine::utohexstr(Offset) << "\n");

    BD->setOutputLocation(OutputSection, Offset);

    // reorder sub-symbols
    for (std::pair<const uint64_t, BinaryData *> &SubBD :
         BC.getSubBinaryData(BD)) {
      if (!SubBD.second->isJumpTable()) {
        uint64_t SubOffset =
            Offset + SubBD.second->getAddress() - BD->getAddress();
        LLVM_DEBUG(dbgs() << "BOLT-DEBUG: SubBD " << SubBD.second->getName()
                          << " @ " << SubOffset << "\n");
        SubBD.second->setOutputLocation(OutputSection, SubOffset);
      }
    }
```

- EN: Declares or implements routines including `LLVM_DEBUG`, `utohexstr`, `setOutputLocation`, `getAddress`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `LLVM_DEBUG`, `utohexstr`, `setOutputLocation`, `getAddress`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`, `utohexstr`, `setOutputLocation`, `getAddress`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `LLVM_DEBUG`, `utohexstr`, `setOutputLocation`, `getAddress`。

### Lines 392-403

```cpp
    Offset += BD->getSize();
    Count += Begin->second;
    NewOrder.push_back(BD);
  }

  OutputSection.reorderContents(NewOrder, opts::ReorderInplace);

  BC.outs() << "BOLT-INFO: reorder-data: " << Count << "/" << TotalCount
            << format(" (%.1f%%)", 100.0 * Count / TotalCount) << " events, "
            << Offset << " hot bytes\n";
}
```

- EN: Declares or implements routines including `getSize`, `format`. Notable symbols here include `getSize`, `format`.
- CN: 这里声明或实现函数，例如 `getSize`, `format`。这里较值得关注的符号包括 `getSize`, `format`。

### Lines 404-421

```cpp
bool ReorderData::markUnmoveableSymbols(BinaryContext &BC,
                                        BinarySection &Section) const {
  // Private symbols currently can't be moved because data can "leak" across
  // the boundary of one symbol to the next, e.g. a string that has a common
  // suffix might start in one private symbol and end with the common
  // suffix in another.
  auto isPrivate = [&](const BinaryData *BD) {
    auto Prefix = std::string("PG") + BC.AsmInfo->getInternalSymbolPrefix();
    return BD->getName().starts_with(Prefix.str());
  };
  auto Range = BC.getBinaryDataForSection(Section);
  bool FoundUnmoveable = false;
  for (auto Itr = Range.begin(); Itr != Range.end(); ++Itr) {
    BinaryData *Next =
        std::next(Itr) != Range.end() ? std::next(Itr)->second : nullptr;
    if (Itr->second->getName().starts_with("PG.")) {
      BinaryData *Prev =
          Itr != Range.begin() ? std::prev(Itr)->second : nullptr;
```

- EN: Declares or implements routines including `string`, `next`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `string`, `next`.
- CN: 这里声明或实现函数，例如 `string`, `next`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `string`, `next`。

### Lines 422-438

```cpp
      bool PrevIsPrivate = Prev && isPrivate(Prev);
      bool NextIsPrivate = Next && isPrivate(Next);
      if (isPrivate(Itr->second) && (PrevIsPrivate || NextIsPrivate))
        Itr->second->setIsMoveable(false);
    } else {
      // check for overlapping symbols.
      if (Next && Itr->second->getEndAddress() != Next->getAddress() &&
          Next->containsAddress(Itr->second->getEndAddress())) {
        Itr->second->setIsMoveable(false);
        Next->setIsMoveable(false);
      }
    }
    FoundUnmoveable |= !Itr->second->isMoveable();
  }
  return FoundUnmoveable;
}
```

- EN: Declares or implements routines including `isPrivate`, `setIsMoveable`, `containsAddress`, `isMoveable`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isPrivate`, `setIsMoveable`, `containsAddress`, `isMoveable`.
- CN: 这里声明或实现函数，例如 `isPrivate`, `setIsMoveable`, `containsAddress`, `isMoveable`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isPrivate`, `setIsMoveable`, `containsAddress`, `isMoveable`。

### Lines 439-451

```cpp
Error ReorderData::runOnFunctions(BinaryContext &BC) {
  static const char *DefaultSections[] = {".rodata", ".data", ".bss", nullptr};

  if (!BC.HasRelocations || opts::ReorderData.empty())
    return Error::success();

  // For now
  if (opts::JumpTables > JTS_BASIC) {
    BC.outs() << "BOLT-WARNING: jump table support must be basic for "
              << "data reordering to work.\n";
    return Error::success();
  }
```

- EN: Declares or implements routines including `runOnFunctions`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `runOnFunctions`.
- CN: 这里声明或实现函数，例如 `runOnFunctions`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `runOnFunctions`。

### Lines 452-464

```cpp
  assignMemData(BC);

  std::vector<BinarySection *> Sections;

  for (const std::string &SectionName : opts::ReorderData) {
    if (SectionName == "default") {
      for (unsigned I = 0; DefaultSections[I]; ++I)
        if (ErrorOr<BinarySection &> Section =
                BC.getUniqueSectionByName(DefaultSections[I]))
          Sections.push_back(&*Section);
      continue;
    }
```

- EN: Declares or implements routines including `assignMemData`. Notable symbols here include `assignMemData`.
- CN: 这里声明或实现函数，例如 `assignMemData`。这里较值得关注的符号包括 `assignMemData`。

### Lines 465-476

```cpp
    ErrorOr<BinarySection &> Section = BC.getUniqueSectionByName(SectionName);
    if (!Section) {
      BC.outs() << "BOLT-WARNING: Section " << SectionName
                << " not found, skipping.\n";
      continue;
    }

    if (!isSupported(*Section)) {
      BC.errs() << "BOLT-ERROR: Section " << SectionName << " not supported.\n";
      return createFatalBOLTError("");
    }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 477-485

```cpp
    Sections.push_back(&*Section);
  }

  for (BinarySection *Section : Sections) {
    const bool FoundUnmoveable = markUnmoveableSymbols(BC, *Section);

    DataOrder Order;
    unsigned SplitPointIdx;
```

- EN: Declares or implements routines including `markUnmoveableSymbols`. Notable symbols here include `markUnmoveableSymbols`.
- CN: 这里声明或实现函数，例如 `markUnmoveableSymbols`。这里较值得关注的符号包括 `markUnmoveableSymbols`。

### Lines 486-495

```cpp
    if (opts::ReorderAlgorithm == opts::ReorderAlgo::REORDER_COUNT) {
      BC.outs() << "BOLT-INFO: reorder-sections: ordering data by count\n";
      std::tie(Order, SplitPointIdx) = sortedByCount(BC, *Section);
    } else {
      BC.outs() << "BOLT-INFO: reorder-sections: ordering data by funcs\n";
      std::tie(Order, SplitPointIdx) =
          sortedByFunc(BC, *Section, BC.getBinaryFunctions());
    }
    auto SplitPoint = Order.begin() + SplitPointIdx;
```

- EN: Declares or implements routines including `tie`, `sortedByFunc`. Notable symbols here include `tie`, `sortedByFunc`.
- CN: 这里声明或实现函数，例如 `tie`, `sortedByFunc`。这里较值得关注的符号包括 `tie`, `sortedByFunc`。

### Lines 496-504

```cpp
    if (opts::PrintReorderedData)
      printOrder(BC, *Section, Order.begin(), SplitPoint);

    if (!opts::ReorderInplace || FoundUnmoveable) {
      if (opts::ReorderInplace && FoundUnmoveable)
        BC.outs() << "BOLT-INFO: Found unmoveable symbols in "
                  << Section->getName() << " falling back to splitting "
                  << "instead of in-place reordering.\n";
```

- EN: Declares or implements routines including `printOrder`, `getName`. Notable symbols here include `printOrder`, `getName`.
- CN: 这里声明或实现函数，例如 `printOrder`, `getName`。这里较值得关注的符号包括 `printOrder`, `getName`。

### Lines 505-513

```cpp
      // Rename sections.
      BinarySection &Hot =
          BC.registerSection(Section->getName() + ".hot", *Section);
      Hot.setOutputName(Section->getName());
      Section->setOutputName(".bolt.org" + Section->getName());

      // Reorder contents of original section.
      setSectionOrder(BC, Hot, Order.begin(), SplitPoint);
```

- EN: Declares or implements routines including `setOutputName`, `setSectionOrder`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `setOutputName`, `setSectionOrder`.
- CN: 这里声明或实现函数，例如 `setOutputName`, `setSectionOrder`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `setOutputName`, `setSectionOrder`。

### Lines 514-530

```cpp
      // This keeps the original data from thinking it has been moved.
      for (std::pair<const uint64_t, BinaryData *> &Entry :
           BC.getBinaryDataForSection(*Section)) {
        if (!Entry.second->isMoved()) {
          Entry.second->setSection(*Section);
          Entry.second->setOutputSection(*Section);
        }
      }
    } else {
      BC.outs()
          << "BOLT-WARNING: Inplace section reordering not supported yet.\n";
      setSectionOrder(BC, *Section, Order.begin(), Order.end());
    }
  }
  return Error::success();
}
```

- EN: Declares or implements routines including `setSection`, `setOutputSection`, `setSectionOrder`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `setSection`, `setOutputSection`, `setSectionOrder`.
- CN: 这里声明或实现函数，例如 `setSection`, `setOutputSection`, `setSectionOrder`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `setSection`, `setOutputSection`, `setSectionOrder`。

### Lines 531-532

```cpp
} // namespace bolt
} // namespace llvm
```

- EN: Works inside namespace scope `bolt`, `llvm` to organize symbols. Notable symbols here include `bolt`, `llvm`.
- CN: 这里位于命名空间 `bolt`, `llvm` 中，用于组织符号作用域。这里较值得关注的符号包括 `bolt`, `llvm`。

## Key Concepts / 关键概念

- `ReorderAlgo`: enumeration of modes or states / 模式或状态枚举
- `desc`: function or method entry point / 函数或方法入口
- `cat`: function or method entry point / 函数或方法入口
- `value_desc`: function or method entry point / 函数或方法入口
- `init`: function or method entry point / 函数或方法入口
- `isSupported`: function or method entry point / 函数或方法入口
- `DEBUG_TYPE`: macro or compile-time switch / 宏或编译期开关
- `llvm`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Passes/ReorderData.h`
- LLVM headers / LLVM 头文件: `llvm/ADT/MapVector.h`
- System headers / 系统头文件: `algorithm`
- Directory context / 目录上下文: `bolt/lib/Passes` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Passes` 下的相邻文件通常与本文件协作组成对应子系统
