# ShrinkWrapping.h — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/include/bolt/Passes/ShrinkWrapping.h`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: *- C++ -*. It also sits in code that declares BOLT optimization and transformation pass interfaces. / 该文件声明 BOLT 优化与变换 Pass 接口。 源码头部说明其职责是：*- C++ -*。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8

```cpp
//===- bolt/Passes/ShrinkWrapping.h -----------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 9-19

```cpp
#ifndef BOLT_PASSES_SHRINKWRAPPING_H
#define BOLT_PASSES_SHRINKWRAPPING_H

#include "bolt/Passes/FrameAnalysis.h"
#include "llvm/MC/MCRegisterInfo.h"
#include <atomic>

namespace llvm {
namespace bolt {
class DataflowInfoManager;
```

- EN: Pulls in 3 header(s) from local project, LLVM, system dependencies needed by this range. Establishes header-guard or prologue macros so the file can be compiled safely. Works inside namespace scope `llvm`, `bolt` to organize symbols. Introduces type definitions such as `DataflowInfoManager`.
- CN: 这里引入 3 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里位于命名空间 `llvm`, `bolt` 中，用于组织符号作用域。这里引入类型定义，例如 `DataflowInfoManager`。

### Lines 20-29

```cpp
/// Encapsulates logic required to analyze a binary function and detect which
/// registers are being saved as callee-saved, where are these saves and where
/// are the points where their original value are being restored.
class CalleeSavedAnalysis {
  const FrameAnalysis &FA;
  const BinaryContext &BC;
  BinaryFunction &BF;
  DataflowInfoManager &Info;
  MCPlusBuilder::AllocatorIdTy AllocatorId;
```

- EN: Introduces type definitions such as `CalleeSavedAnalysis`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `CalleeSavedAnalysis`.
- CN: 这里引入类型定义，例如 `CalleeSavedAnalysis`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `CalleeSavedAnalysis`。

### Lines 30-41

```cpp
  std::optional<unsigned> SaveTagIndex;
  std::optional<unsigned> RestoreTagIndex;

  /// Compute all stores of callee-saved regs. Those are the ones that stores a
  /// register whose definition is not local.
  void analyzeSaves();

  /// Similar to analyzeSaves, tries to determine all instructions that recover
  /// the original value of the callee-saved register before exiting the
  /// function.
  void analyzeRestores();
```

- EN: Declares or implements routines including `analyzeSaves`, `analyzeRestores`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `analyzeSaves`, `analyzeRestores`.
- CN: 这里声明或实现函数，例如 `analyzeSaves`, `analyzeRestores`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `analyzeSaves`, `analyzeRestores`。

### Lines 42-55

```cpp
  unsigned getSaveTag() {
    if (SaveTagIndex)
      return *SaveTagIndex;
    SaveTagIndex = BC.MIB->getOrCreateAnnotationIndex(getSaveTagName());
    return *SaveTagIndex;
  }

  unsigned getRestoreTag() {
    if (RestoreTagIndex)
      return *RestoreTagIndex;
    RestoreTagIndex = BC.MIB->getOrCreateAnnotationIndex(getRestoreTagName());
    return *RestoreTagIndex;
  }
```

- EN: Declares or implements routines including `getSaveTag`, `getOrCreateAnnotationIndex`, `getRestoreTag`. Notable symbols here include `getSaveTag`, `getOrCreateAnnotationIndex`, `getRestoreTag`.
- CN: 这里声明或实现函数，例如 `getSaveTag`, `getOrCreateAnnotationIndex`, `getRestoreTag`。这里较值得关注的符号包括 `getSaveTag`, `getOrCreateAnnotationIndex`, `getRestoreTag`。

### Lines 56-63

```cpp
public:
  BitVector CalleeSaved;
  std::vector<int64_t> OffsetsByReg;
  BitVector HasRestores;
  std::vector<uint64_t> SavingCost;
  std::vector<const FrameIndexEntry *> SaveFIEByReg;
  std::vector<const FrameIndexEntry *> LoadFIEByReg;
```

- EN: Continues the surrounding implementation, wiring declarations, state, and helper logic together.
- CN: 这里继续展开周边实现，衔接声明、状态与辅助逻辑。

### Lines 64-74

```cpp
  CalleeSavedAnalysis(const FrameAnalysis &FA, BinaryFunction &BF,
                      DataflowInfoManager &Info,
                      MCPlusBuilder::AllocatorIdTy AllocId)
      : FA(FA), BC(BF.getBinaryContext()), BF(BF), Info(Info),
        AllocatorId(AllocId), CalleeSaved(BC.MRI->getNumRegs(), false),
        OffsetsByReg(BC.MRI->getNumRegs(), 0LL),
        HasRestores(BC.MRI->getNumRegs(), false),
        SavingCost(BC.MRI->getNumRegs(), 0ULL),
        SaveFIEByReg(BC.MRI->getNumRegs(), nullptr),
        LoadFIEByReg(BC.MRI->getNumRegs(), nullptr) {}
```

- EN: Declares or implements routines including `FA`, `AllocatorId`, `OffsetsByReg`, `HasRestores`, `SavingCost`, and 2 more. Notable symbols here include `FA`, `AllocatorId`, `OffsetsByReg`, `HasRestores`, `SavingCost`, `SaveFIEByReg`.
- CN: 这里声明或实现函数，例如 `FA`, `AllocatorId`, `OffsetsByReg`, `HasRestores`, `SavingCost`, and 2 more。这里较值得关注的符号包括 `FA`, `AllocatorId`, `OffsetsByReg`, `HasRestores`, `SavingCost`, `SaveFIEByReg`。

### Lines 75-91

```cpp
  ~CalleeSavedAnalysis();

  void compute() {
    analyzeSaves();
    analyzeRestores();
  }

  /// Retrieves the value of the callee-saved register that is saved by this
  /// instruction or 0 if this is not a CSR save instruction.
  uint16_t getSavedReg(const MCInst &Inst) {
    auto Val = BC.MIB->tryGetAnnotationAs<decltype(FrameIndexEntry::RegOrImm)>(
        Inst, getSaveTag());
    if (Val)
      return *Val;
    return 0;
  }
```

- EN: Declares or implements routines including `CalleeSavedAnalysis`, `compute`, `analyzeSaves`, `analyzeRestores`, `getSavedReg`, and 2 more. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `CalleeSavedAnalysis`, `compute`, `analyzeSaves`, `analyzeRestores`, `getSavedReg`, `decltype`.
- CN: 这里声明或实现函数，例如 `CalleeSavedAnalysis`, `compute`, `analyzeSaves`, `analyzeRestores`, `getSavedReg`, and 2 more。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `CalleeSavedAnalysis`, `compute`, `analyzeSaves`, `analyzeRestores`, `getSavedReg`, `decltype`。

### Lines 92-101

```cpp
  /// Retrieves the value of the callee-saved register that is restored by this
  /// instruction or 0 if this is not a CSR restore instruction.
  uint16_t getRestoredReg(const MCInst &Inst) {
    auto Val = BC.MIB->tryGetAnnotationAs<decltype(FrameIndexEntry::RegOrImm)>(
        Inst, getRestoreTag());
    if (Val)
      return *Val;
    return 0;
  }
```

- EN: Declares or implements routines including `getRestoredReg`, `decltype`, `getRestoreTag`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getRestoredReg`, `decltype`, `getRestoreTag`.
- CN: 这里声明或实现函数，例如 `getRestoredReg`, `decltype`, `getRestoreTag`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getRestoredReg`, `decltype`, `getRestoreTag`。

### Lines 102-110

```cpp
  /// Routines to compute all saves/restores for a Reg (needs to traverse all
  /// instructions).
  std::vector<MCInst *> getSavesByReg(uint16_t Reg);
  std::vector<MCInst *> getRestoresByReg(uint16_t Reg);

  /// Returns the identifying string used to annotate instructions with metadata
  /// for this analysis. These are deleted in the destructor.
  static StringRef getSaveTagName() { return StringRef("CSA-SavedReg"); }
```

- EN: Declares or implements routines including `getSavesByReg`, `getRestoresByReg`, `getSaveTagName`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getSavesByReg`, `getRestoresByReg`, `getSaveTagName`.
- CN: 这里声明或实现函数，例如 `getSavesByReg`, `getRestoresByReg`, `getSaveTagName`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getSavesByReg`, `getRestoresByReg`, `getSaveTagName`。

### Lines 111-123

```cpp
  static StringRef getRestoreTagName() { return StringRef("CSA-RestoredReg"); }
};

/// Identifies in a given binary function all stack regions being used and allow
/// us to edit the layout, removing or inserting new regions. When the layout is
/// modified, all affected stack-accessing instructions are updated.
class StackLayoutModifier {
  const FrameAnalysis &FA;
  const BinaryContext &BC;
  BinaryFunction &BF;
  DataflowInfoManager &Info;
  MCPlusBuilder::AllocatorIdTy AllocatorId;
```

- EN: Introduces type definitions such as `StackLayoutModifier`. Declares or implements routines including `getRestoreTagName`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `StackLayoutModifier`, `getRestoreTagName`.
- CN: 这里引入类型定义，例如 `StackLayoutModifier`。这里声明或实现函数，例如 `getRestoreTagName`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `StackLayoutModifier`, `getRestoreTagName`。

### Lines 124-133

```cpp
  // Keep track of stack slots we know how to safely move
  std::map<int64_t, int64_t> AvailableRegions;

  DenseSet<int64_t> CollapsedRegions;
  DenseSet<int64_t> InsertedRegions;

  // A map of chunks of stack memory we don't really know what's happening there
  // and we need to leave it untouched.
  std::map<int64_t, int64_t> BlacklistedRegions;
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 134-142

```cpp
  // Maps stack slots to the regs that are saved to them
  DenseMap<int64_t, std::set<MCPhysReg>> RegionToRegMap;
  DenseMap<int, std::set<int64_t>> RegToRegionMap;

  // If we can't understand how to move stack slots, IsSimple will be false
  bool IsSimple{true};

  bool IsInitialized{false};
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 143-156

```cpp
  std::optional<unsigned> TodoTagIndex;
  std::optional<unsigned> SlotTagIndex;
  std::optional<unsigned> OffsetCFIRegTagIndex;

public:
  // Keep a worklist of operations to perform on the function to perform
  // the requested layout modifications via collapseRegion()/insertRegion().
  struct WorklistItem {
    enum ActionType : uint8_t {
      None = 0,
      AdjustLoadStoreOffset,
      AdjustCFI,
    } Action;
```

- EN: Introduces type definitions such as `WorklistItem`. Defines enumerations such as `ActionType` to encode states or modes. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `WorklistItem`, `ActionType`.
- CN: 这里引入类型定义，例如 `WorklistItem`。这里定义枚举 `ActionType`，用于表达状态或模式。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `WorklistItem`, `ActionType`。

### Lines 157-168

```cpp
    int64_t OffsetUpdate{0};
    WorklistItem() : Action(None) {}
    WorklistItem(ActionType Action) : Action(Action) {}
    WorklistItem(ActionType Action, int OffsetUpdate)
        : Action(Action), OffsetUpdate(OffsetUpdate) {}
  };

private:
  /// Mark the stack region identified by \p Offset and \p Size to be a
  /// no-touch zone, whose accesses cannot be relocated to another region.
  void blacklistRegion(int64_t Offset, int64_t Size);
```

- EN: Declares or implements routines including `WorklistItem`, `Action`, `blacklistRegion`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `WorklistItem`, `Action`, `blacklistRegion`.
- CN: 这里声明或实现函数，例如 `WorklistItem`, `Action`, `blacklistRegion`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `WorklistItem`, `Action`, `blacklistRegion`。

### Lines 169-176

```cpp
  /// Check if this region overlaps with blacklisted addresses
  bool isRegionBlacklisted(int64_t Offset, int64_t Size);

  /// Check if the region identified by \p Offset and \p Size has any conflicts
  /// with available regions so far. If it has, blacklist all involved regions
  /// and return true.
  bool blacklistAllInConflictWith(int64_t Offset, int64_t Size);
```

- EN: Declares or implements routines including `isRegionBlacklisted`, `blacklistAllInConflictWith`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isRegionBlacklisted`, `blacklistAllInConflictWith`.
- CN: 这里声明或实现函数，例如 `isRegionBlacklisted`, `blacklistAllInConflictWith`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isRegionBlacklisted`, `blacklistAllInConflictWith`。

### Lines 177-187

```cpp
  /// If \p Point is identified as frame pointer initialization (defining the
  /// value of FP with SP), check for non-standard initialization that precludes
  /// us from changing the stack layout. If positive, update blacklisted
  /// regions.
  void checkFramePointerInitialization(MCInst &Point);

  /// If \p Point is restoring the value with SP with FP plus offset,
  /// add a slottag to this instruction as it needs to be updated when we
  /// change the stack layout.
  void checkStackPointerRestore(MCInst &Point);
```

- EN: Declares or implements routines including `checkFramePointerInitialization`, `checkStackPointerRestore`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `checkFramePointerInitialization`, `checkStackPointerRestore`.
- CN: 这里声明或实现函数，例如 `checkFramePointerInitialization`, `checkStackPointerRestore`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `checkFramePointerInitialization`, `checkStackPointerRestore`。

### Lines 188-195

```cpp
  /// Make sense of each stack offsets we can freely change
  void classifyStackAccesses();
  void classifyCFIs();

  /// Used to keep track of modifications to the function that will later be
  /// performed by performChanges();
  void scheduleChange(MCInst &Inst, WorklistItem Item);
```

- EN: Declares or implements routines including `classifyStackAccesses`, `classifyCFIs`, `scheduleChange`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `classifyStackAccesses`, `classifyCFIs`, `scheduleChange`.
- CN: 这里声明或实现函数，例如 `classifyStackAccesses`, `classifyCFIs`, `scheduleChange`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `classifyStackAccesses`, `classifyCFIs`, `scheduleChange`。

### Lines 196-209

```cpp
  unsigned getTodoTag() {
    if (TodoTagIndex)
      return *TodoTagIndex;
    TodoTagIndex = BC.MIB->getOrCreateAnnotationIndex(getTodoTagName());
    return *TodoTagIndex;
  }

  unsigned getSlotTag() {
    if (SlotTagIndex)
      return *SlotTagIndex;
    SlotTagIndex = BC.MIB->getOrCreateAnnotationIndex(getSlotTagName());
    return *SlotTagIndex;
  }
```

- EN: Declares or implements routines including `getTodoTag`, `getOrCreateAnnotationIndex`, `getSlotTag`. Notable symbols here include `getTodoTag`, `getOrCreateAnnotationIndex`, `getSlotTag`.
- CN: 这里声明或实现函数，例如 `getTodoTag`, `getOrCreateAnnotationIndex`, `getSlotTag`。这里较值得关注的符号包括 `getTodoTag`, `getOrCreateAnnotationIndex`, `getSlotTag`。

### Lines 210-217

```cpp
  unsigned getOffsetCFIRegTag() {
    if (OffsetCFIRegTagIndex)
      return *OffsetCFIRegTagIndex;
    OffsetCFIRegTagIndex =
        BC.MIB->getOrCreateAnnotationIndex(getOffsetCFIRegTagName());
    return *OffsetCFIRegTagIndex;
  }
```

- EN: Declares or implements routines including `getOffsetCFIRegTag`, `getOrCreateAnnotationIndex`. Notable symbols here include `getOffsetCFIRegTag`, `getOrCreateAnnotationIndex`.
- CN: 这里声明或实现函数，例如 `getOffsetCFIRegTag`, `getOrCreateAnnotationIndex`。这里较值得关注的符号包括 `getOffsetCFIRegTag`, `getOrCreateAnnotationIndex`。

### Lines 218-234

```cpp
public:
  StackLayoutModifier(const FrameAnalysis &FA, BinaryFunction &BF,
                      DataflowInfoManager &Info,
                      MCPlusBuilder::AllocatorIdTy AllocId)
      : FA(FA), BC(BF.getBinaryContext()), BF(BF), Info(Info),
        AllocatorId(AllocId) {}

  ~StackLayoutModifier() {
    for (BinaryBasicBlock &BB : BF) {
      for (MCInst &Inst : BB) {
        BC.MIB->removeAnnotation(Inst, getTodoTag());
        BC.MIB->removeAnnotation(Inst, getSlotTag());
        BC.MIB->removeAnnotation(Inst, getOffsetCFIRegTag());
      }
    }
  }
```

- EN: Declares or implements routines including `FA`, `AllocatorId`, `StackLayoutModifier`, `removeAnnotation`. Notable symbols here include `FA`, `AllocatorId`, `StackLayoutModifier`, `removeAnnotation`.
- CN: 这里声明或实现函数，例如 `FA`, `AllocatorId`, `StackLayoutModifier`, `removeAnnotation`。这里较值得关注的符号包括 `FA`, `AllocatorId`, `StackLayoutModifier`, `removeAnnotation`。

### Lines 235-243

```cpp
  /// Retrieves the value of the callee-saved register that is restored by this
  /// instruction or 0 if this is not a CSR restore instruction.
  uint16_t getOffsetCFIReg(const MCInst &Inst) {
    auto Val = BC.MIB->tryGetAnnotationAs<uint16_t>(Inst, getOffsetCFIRegTag());
    if (Val)
      return *Val;
    return 0;
  }
```

- EN: Declares or implements routines including `getOffsetCFIReg`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getOffsetCFIReg`.
- CN: 这里声明或实现函数，例如 `getOffsetCFIReg`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getOffsetCFIReg`。

### Lines 244-255

```cpp
  /// Check if it is possible to delete the push instruction \p DeletedPush.
  /// This involves collapsing the region accessed by this push and updating all
  /// other instructions that access affected memory regions. Return true if we
  /// can update this.
  bool canCollapseRegion(int64_t RegionAddr);
  bool canCollapseRegion(MCInst *DeletedPush);

  /// Notify the layout manager that \p DeletedPush was deleted and that it
  /// needs to update other affected stack-accessing instructions.
  bool collapseRegion(MCInst *Alloc, int64_t RegionAddr, int64_t RegionSize);
  bool collapseRegion(MCInst *DeletedPush);
```

- EN: Declares or implements routines including `canCollapseRegion`, `collapseRegion`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `canCollapseRegion`, `collapseRegion`.
- CN: 这里声明或实现函数，例如 `canCollapseRegion`, `collapseRegion`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `canCollapseRegion`, `collapseRegion`。

### Lines 256-264

```cpp
  /// Set the new stack address difference for load/store instructions that
  /// referenced a stack location that was deleted via collapseRegion.
  void setOffsetForCollapsedAccesses(int64_t NewOffset);

  /// Check if it is possible to insert a push instruction at point \p P.
  /// This involves inserting a new region in the stack, possibly affecting
  /// instructions that access the frame. Return true if we can update them all.
  bool canInsertRegion(ProgramPoint P);
```

- EN: Declares or implements routines including `setOffsetForCollapsedAccesses`, `canInsertRegion`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `setOffsetForCollapsedAccesses`, `canInsertRegion`.
- CN: 这里声明或实现函数，例如 `setOffsetForCollapsedAccesses`, `canInsertRegion`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `setOffsetForCollapsedAccesses`, `canInsertRegion`。

### Lines 265-275

```cpp
  /// Notify the layout manager that a new push instruction has been inserted
  /// at point \p P and that it will need to update relevant instructions.
  bool insertRegion(ProgramPoint P, int64_t RegionSz);

  /// Perform all changes scheduled by collapseRegion()/insertRegion()
  void performChanges();

  /// Perform initial assessment of the function trying to understand its stack
  /// accesses.
  void initialize();
```

- EN: Declares or implements routines including `insertRegion`, `performChanges`, `initialize`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `insertRegion`, `performChanges`, `initialize`.
- CN: 这里声明或实现函数，例如 `insertRegion`, `performChanges`, `initialize`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `insertRegion`, `performChanges`, `initialize`。

### Lines 276-284

```cpp
  static StringRef getTodoTagName() { return StringRef("SLM-TodoTag"); }

  static StringRef getSlotTagName() { return StringRef("SLM-SlotTag"); }

  static StringRef getOffsetCFIRegTagName() {
    return StringRef("SLM-OffsetCFIReg");
  }
};
```

- EN: Declares or implements routines including `getTodoTagName`, `getSlotTagName`, `getOffsetCFIRegTagName`. Notable symbols here include `getTodoTagName`, `getSlotTagName`, `getOffsetCFIRegTagName`.
- CN: 这里声明或实现函数，例如 `getTodoTagName`, `getSlotTagName`, `getOffsetCFIRegTagName`。这里较值得关注的符号包括 `getTodoTagName`, `getSlotTagName`, `getOffsetCFIRegTagName`。

### Lines 285-302

```cpp
/// Implements a pass to optimize callee-saved register spills. These spills
/// typically happen at function prologue/epilogue. When these are hot basic
/// blocks, this pass will try to move these spills to cold blocks whenever
/// possible.
class ShrinkWrapping {
  const FrameAnalysis &FA;
  const BinaryContext &BC;
  BinaryFunction &BF;
  DataflowInfoManager &Info;
  MCPlusBuilder::AllocatorIdTy AllocatorId;
  StackLayoutModifier SLM;
  /// For each CSR, store a vector of all CFI indexes deleted as a consequence
  /// of moving this Callee-Saved Reg
  DenseMap<unsigned, std::vector<uint32_t>> DeletedPushCFIs;
  DenseMap<unsigned, std::vector<uint32_t>> DeletedPopCFIs;
  BitVector HasDeletedOffsetCFIs;
  SmallPtrSet<const MCCFIInstruction *, 16> UpdatedCFIs;
  std::vector<BitVector> UsesByReg;
```

- EN: Introduces type definitions such as `ShrinkWrapping`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `ShrinkWrapping`.
- CN: 这里引入类型定义，例如 `ShrinkWrapping`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `ShrinkWrapping`。

### Lines 303-317

```cpp
  std::vector<int64_t> PushOffsetByReg;
  std::vector<int64_t> PopOffsetByReg;
  std::vector<MCPhysReg> DomOrder;
  CalleeSavedAnalysis CSA;
  std::vector<std::vector<uint64_t>> BestSaveCount;
  std::vector<std::vector<MCInst *>> BestSavePos;

  /// Pass stats
  static std::atomic<std::uint64_t> SpillsMovedRegularMode;
  static std::atomic<std::uint64_t> SpillsMovedPushPopMode;
  static std::atomic<std::uint64_t> SpillsMovedDynamicCount;
  static std::atomic<std::uint64_t> SpillsFailedDynamicCount;
  static std::atomic<std::uint64_t> InstrDynamicCount;
  static std::atomic<std::uint64_t> StoreDynamicCount;
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 318-335

```cpp
  std::optional<unsigned> AnnotationIndex;

  /// Allow our custom worklist-sensitive analysis
  /// PredictiveStackPointerTracking to access WorklistItem
public:
  struct WorklistItem {
    enum ActionType : uint8_t {
      Erase = 0,
      ChangeToAdjustment,
      InsertLoadOrStore,
      InsertPushOrPop
    } Action;
    FrameIndexEntry FIEToInsert;
    unsigned AffectedReg;
    int Adjustment{0};
    WorklistItem(ActionType Action, unsigned AffectedReg)
        : Action(Action), FIEToInsert(), AffectedReg(AffectedReg) {}
    WorklistItem(ActionType Action, unsigned AffectedReg, int Adjustment)
```

- EN: Introduces type definitions such as `WorklistItem`. Defines enumerations such as `ActionType` to encode states or modes. Declares or implements routines including `WorklistItem`, `Action`. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入类型定义，例如 `WorklistItem`。这里定义枚举 `ActionType`，用于表达状态或模式。这里声明或实现函数，例如 `WorklistItem`, `Action`。这里的注释记录了设计假设、不变量或使用说明。

### Lines 336-346

```cpp
        : Action(Action), FIEToInsert(), AffectedReg(AffectedReg),
          Adjustment(Adjustment) {}
    WorklistItem(ActionType Action, const FrameIndexEntry &FIE,
                 unsigned AffectedReg)
        : Action(Action), FIEToInsert(FIE), AffectedReg(AffectedReg) {}
  };

  /// Insertion todo items scheduled to happen at the end of BBs. Since we
  /// can't annotate BBs we maintain this bookkeeping here.
  DenseMap<BinaryBasicBlock *, std::vector<WorklistItem>> Todo;
```

- EN: Declares or implements routines including `Action`, `Adjustment`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `Action`, `Adjustment`.
- CN: 这里声明或实现函数，例如 `Action`, `Adjustment`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `Action`, `Adjustment`。

### Lines 347-356

```cpp
  /// Annotation name used to tag instructions with removal or insertion actions
  static StringRef getAnnotationName() { return StringRef("ShrinkWrap-Todo"); }

  unsigned getAnnotationIndex() {
    if (AnnotationIndex)
      return *AnnotationIndex;
    AnnotationIndex = BC.MIB->getOrCreateAnnotationIndex(getAnnotationName());
    return *AnnotationIndex;
  }
```

- EN: Declares or implements routines including `getAnnotationName`, `getAnnotationIndex`, `getOrCreateAnnotationIndex`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getAnnotationName`, `getAnnotationIndex`, `getOrCreateAnnotationIndex`.
- CN: 这里声明或实现函数，例如 `getAnnotationName`, `getAnnotationIndex`, `getOrCreateAnnotationIndex`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getAnnotationName`, `getAnnotationIndex`, `getOrCreateAnnotationIndex`。

### Lines 357-367

```cpp
private:
  using BBIterTy = BinaryBasicBlock::iterator;

  /// Calculate all possible uses/defs of these callee-saved regs
  void classifyCSRUses();

  // Ensure we don't work on cases where there are no uses of the callee-saved
  // register. These unnecessary spills should have been removed by previous
  // passes.
  void pruneUnwantedCSRs();
```

- EN: Declares or implements routines including `classifyCSRUses`, `pruneUnwantedCSRs`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `classifyCSRUses`, `pruneUnwantedCSRs`.
- CN: 这里声明或实现函数，例如 `classifyCSRUses`, `pruneUnwantedCSRs`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `classifyCSRUses`, `pruneUnwantedCSRs`。

### Lines 368-377

```cpp
  // Map regs to their possible save possibilities (at start of these BBs)
  void computeSaveLocations();

  /// Look into the best save location found for saving callee-saved reg
  /// \p CSR and evaluates whether we would benefit by moving the spill to this
  /// new save location. Returns true in case it is profitable to perform the
  /// move.
  bool validateBestSavePos(unsigned CSR, MCInst *&BestPosSave,
                           uint64_t &TotalEstimatedWin);
```

- EN: Declares or implements routines including `computeSaveLocations`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `computeSaveLocations`.
- CN: 这里声明或实现函数，例如 `computeSaveLocations`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `computeSaveLocations`。

### Lines 378-394

```cpp
  /// Populate the Todo map with worklistitems to change the function
  template <typename... T> void scheduleChange(ProgramPoint PP, T &&...Item) {
    if (PP.isInst()) {
      auto &WList = BC.MIB->getOrCreateAnnotationAs<std::vector<WorklistItem>>(
          *PP.getInst(), getAnnotationIndex(), AllocatorId);
      WList.emplace_back(std::forward<T>(Item)...);
      return;
    }
    BinaryBasicBlock *BB = PP.getBB();
    // Avoid inserting on BBs with no instructions because we have a dataflow
    // analysis that depends on insertions happening before real instructions
    // (PredictiveStackPointerTracking)
    assert(BB->size() != 0 &&
           "doRestorePlacement() should have handled empty BBs");
    Todo[BB].emplace_back(std::forward<T>(Item)...);
  }
```

- EN: Declares or implements routines including `scheduleChange`, `assert`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `scheduleChange`, `assert`.
- CN: 这里声明或实现函数，例如 `scheduleChange`, `assert`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `scheduleChange`, `assert`。

### Lines 395-405

```cpp
  /// Determine the POP ordering according to which CSR save is the dominator.
  void computeDomOrder();

  /// Check that the best possible location for a spill save (as determined by
  /// computeSaveLocations) is cold enough to be worth moving the save to it.
  /// \p CSR is the callee-saved register number, \p BestPosSave returns the
  /// pointer to the cold location in case the function returns true, while
  /// \p TotalEstimatedWin contains the ins dyn count reduction after moving.
  bool isBestSavePosCold(unsigned CSR, MCInst *&BestPosSave,
                         uint64_t &TotalEstimatedWin);
```

- EN: Declares or implements routines including `computeDomOrder`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `computeDomOrder`.
- CN: 这里声明或实现函数，例如 `computeDomOrder`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `computeDomOrder`。

### Lines 406-413

```cpp
  /// Auxiliary function used to create basic blocks for critical edges and
  /// update the dominance frontier with these new locations
  void splitFrontierCritEdges(
      BinaryFunction *Func, SmallVector<ProgramPoint, 4> &Frontier,
      const SmallVector<bool, 4> &IsCritEdge,
      const SmallVector<BinaryBasicBlock *, 4> &From,
      const SmallVector<SmallVector<BinaryBasicBlock *, 4>, 4> &To);
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 414-422

```cpp
  /// After the best save location for a spill has been established in
  /// \p BestPosSave for reg \p CSR, compute adequate locations to restore
  /// the spilled value. This will be at the dominance frontier.
  /// Returns an empty vector if we failed. In case of success, set
  /// \p UsePushPops to true if we can operate in the push/pops mode.
  SmallVector<ProgramPoint, 4> doRestorePlacement(MCInst *BestPosSave,
                                                  unsigned CSR,
                                                  uint64_t TotalEstimatedWin);
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 423-433

```cpp
  /// Checks whether using push and pops (instead of the longer load-store
  /// counterparts) is correct for reg \p CSR
  bool validatePushPopsMode(unsigned CSR, MCInst *BestPosSave,
                            int64_t SaveOffset);

  /// Adjust restore locations to the correct SP offset if we are using POPs
  /// instead of random-access load instructions.
  SmallVector<ProgramPoint, 4>
  fixPopsPlacements(const SmallVector<ProgramPoint, 4> &RestorePoints,
                    int64_t SaveOffset, unsigned CSR);
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 434-443

```cpp
  /// When moving spills, mark all old spill locations to be deleted
  void scheduleOldSaveRestoresRemoval(unsigned CSR, bool UsePushPops);
  /// Return true if \p Inst uses reg \p CSR
  bool doesInstUsesCSR(const MCInst &Inst, uint16_t CSR);
  /// When moving spills, mark all new spill locations for insertion
  void
  scheduleSaveRestoreInsertions(unsigned CSR, MCInst *BestPosSave,
                                SmallVector<ProgramPoint, 4> &RestorePoints,
                                bool UsePushPops);
```

- EN: Declares or implements routines including `scheduleOldSaveRestoresRemoval`, `doesInstUsesCSR`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `scheduleOldSaveRestoresRemoval`, `doesInstUsesCSR`.
- CN: 这里声明或实现函数，例如 `scheduleOldSaveRestoresRemoval`, `doesInstUsesCSR`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `scheduleOldSaveRestoresRemoval`, `doesInstUsesCSR`。

### Lines 444-452

```cpp
  /// Coordinate the replacement of callee-saved spills from their original
  /// place (at prologue and epilogues) to colder basic blocks as determined
  /// by computeSaveLocations().
  void moveSaveRestores();

  /// Compare multiple basic blocks created by splitting critical edges. If they
  /// have the same contents and successor, fold them into one.
  bool foldIdenticalSplitEdges();
```

- EN: Declares or implements routines including `moveSaveRestores`, `foldIdenticalSplitEdges`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `moveSaveRestores`, `foldIdenticalSplitEdges`.
- CN: 这里声明或实现函数，例如 `moveSaveRestores`, `foldIdenticalSplitEdges`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `moveSaveRestores`, `foldIdenticalSplitEdges`。

### Lines 453-462

```cpp
  /// After the spill locations for reg \p CSR has been moved and all affected
  /// CFI has been removed, insert new updated CFI information for these
  /// locations.
  void insertUpdatedCFI(unsigned CSR, int SPValPush, int SPValPop);

  /// In case the function anchors the CFA reg as SP and we inserted pushes/pops
  /// insert def_cfa_offsets at appropriate places (and delete old
  /// def_cfa_offsets)
  void rebuildCFIForSP();
```

- EN: Declares or implements routines including `insertUpdatedCFI`, `rebuildCFIForSP`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `insertUpdatedCFI`, `rebuildCFIForSP`.
- CN: 这里声明或实现函数，例如 `insertUpdatedCFI`, `rebuildCFIForSP`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `insertUpdatedCFI`, `rebuildCFIForSP`。

### Lines 463-473

```cpp
  /// Rebuild all CFI for affected Callee-Saved Registers.
  void rebuildCFI();

  /// Create a load-store instruction (depending on the contents of \p FIE).
  /// If \p CreatePushOrPop is true, create a push/pop instead. Current SP/FP
  /// values, as determined by StackPointerTracking, should be informed via
  /// \p SPVal and \p FPVal in order to emit the correct offset form SP/FP.
  Expected<MCInst> createStackAccess(int SPVal, int FPVal,
                                     const FrameIndexEntry &FIE,
                                     bool CreatePushOrPop);
```

- EN: Declares or implements routines including `rebuildCFI`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `rebuildCFI`.
- CN: 这里声明或实现函数，例如 `rebuildCFI`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `rebuildCFI`。

### Lines 474-482

```cpp
  /// Update the CFI referenced by \p Inst with \p NewOffset, if the CFI has
  /// an offset.
  void updateCFIInstOffset(MCInst &Inst, int64_t NewOffset);

  /// Insert any CFI that should be attached to a register spill save/restore.
  BBIterTy insertCFIsForPushOrPop(BinaryBasicBlock &BB, BBIterTy Pos,
                                  unsigned Reg, bool IsPush, int Sz,
                                  int64_t NewOffset);
```

- EN: Declares or implements routines including `updateCFIInstOffset`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `updateCFIInstOffset`.
- CN: 这里声明或实现函数，例如 `updateCFIInstOffset`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `updateCFIInstOffset`。

### Lines 483-492

```cpp
  /// Auxiliary function to processInsertionsList, adding a new instruction
  /// before \p InsertionPoint as requested by \p Item. Return an updated
  /// InsertionPoint for other instructions that need to be inserted at the same
  /// original location, since this insertion may have invalidated the previous
  /// location.
  Expected<BBIterTy> processInsertion(BBIterTy InsertionPoint,
                                      BinaryBasicBlock *CurBB,
                                      const WorklistItem &Item, int64_t SPVal,
                                      int64_t FPVal);
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 493-500

```cpp
  /// Auxiliary function to processInsertions(), helping perform all the
  /// insertion tasks in the todo list associated with a single insertion point.
  /// Return true if at least one insertion was performed.
  Expected<BBIterTy> processInsertionsList(BBIterTy InsertionPoint,
                                           BinaryBasicBlock *CurBB,
                                           std::vector<WorklistItem> &TodoList,
                                           int64_t SPVal, int64_t FPVal);
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 501-509

```cpp
  /// Apply all insertion todo tasks regarding insertion of new stores/loads or
  /// push/pops at annotated points. Return false if the entire function had
  /// no todo tasks annotation and this pass has nothing to do.
  Expected<bool> processInsertions();

  /// Apply all deletion todo tasks (or tasks to change a push/pop to a memory
  /// access no-op)
  void processDeletions();
```

- EN: Declares or implements routines including `processInsertions`, `processDeletions`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `processInsertions`, `processDeletions`.
- CN: 这里声明或实现函数，例如 `processInsertions`, `processDeletions`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `processInsertions`, `processDeletions`。

### Lines 510-517

```cpp
public:
  ShrinkWrapping(const FrameAnalysis &FA, BinaryFunction &BF,
                 DataflowInfoManager &Info,
                 MCPlusBuilder::AllocatorIdTy AllocId)
      : FA(FA), BC(BF.getBinaryContext()), BF(BF), Info(Info),
        AllocatorId(AllocId), SLM(FA, BF, Info, AllocId),
        CSA(FA, BF, Info, AllocId) {}
```

- EN: Declares or implements routines including `FA`, `AllocatorId`, `CSA`. Notable symbols here include `FA`, `AllocatorId`, `CSA`.
- CN: 这里声明或实现函数，例如 `FA`, `AllocatorId`, `CSA`。这里较值得关注的符号包括 `FA`, `AllocatorId`, `CSA`。

### Lines 518-525

```cpp
  ~ShrinkWrapping() {
    for (BinaryBasicBlock &BB : BF)
      for (MCInst &Inst : BB)
        BC.MIB->removeAnnotation(Inst, getAnnotationIndex());
  }

  Expected<bool> perform(bool HotOnly = false);
```

- EN: Declares or implements routines including `ShrinkWrapping`, `removeAnnotation`, `perform`. Notable symbols here include `ShrinkWrapping`, `removeAnnotation`, `perform`.
- CN: 这里声明或实现函数，例如 `ShrinkWrapping`, `removeAnnotation`, `perform`。这里较值得关注的符号包括 `ShrinkWrapping`, `removeAnnotation`, `perform`。

### Lines 526-532

```cpp
  static void printStats(BinaryContext &BC);
};

} // end namespace bolt
} // end namespace llvm

#endif
```

- EN: Uses conditional compilation to choose platform- or feature-specific code paths. Works inside namespace scope `bolt`, `llvm` to organize symbols. Declares or implements routines including `printStats`. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里使用条件编译，根据平台或特性切换不同代码路径。这里位于命名空间 `bolt`, `llvm` 中，用于组织符号作用域。这里声明或实现函数，例如 `printStats`。这里的注释记录了设计假设、不变量或使用说明。

## Key Concepts / 关键概念

- `DataflowInfoManager`: class or struct interface / 类或结构体接口
- `CalleeSavedAnalysis`: class or struct interface / 类或结构体接口
- `StackLayoutModifier`: class or struct interface / 类或结构体接口
- `WorklistItem`: class or struct interface / 类或结构体接口
- `ActionType`: enumeration of modes or states / 模式或状态枚举
- `analyzeSaves`: function or method entry point / 函数或方法入口
- `analyzeRestores`: function or method entry point / 函数或方法入口
- `getSaveTag`: function or method entry point / 函数或方法入口

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Passes/FrameAnalysis.h`
- LLVM headers / LLVM 头文件: `llvm/MC/MCRegisterInfo.h`
- System headers / 系统头文件: `atomic`
- Directory context / 目录上下文: `bolt/include/bolt/Passes` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/include/bolt/Passes` 下的相邻文件通常与本文件协作组成对应子系统
