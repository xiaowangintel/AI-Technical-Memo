# ResourceManager.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/lib/MCA/HardwareUnits/ResourceManager.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements machine scheduling resources and hardware unit models for llvm-mca.
  - **CN**: 实现 llvm-mca 使用的调度资源与硬件单元模型。

## Line-by-Line Analysis / 逐行分析

### Lines 1-10
```cpp
//===--------------------- ResourceManager.cpp ------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
/// \file
///
/// The classes here represent processor resource units and their management
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件的高层描述。

### Lines 11-20
```cpp
/// strategy.  These classes are managed by the Scheduler.
///
//===----------------------------------------------------------------------===//

#include "llvm/MCA/HardwareUnits/ResourceManager.h"
#include "llvm/MCA/Support.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/raw_ostream.h"

namespace llvm {
```
- **EN**: Pulls in the headers needed by this translation unit, including `llvm/MCA/HardwareUnits/ResourceManager.h`, `llvm/MCA/Support.h`, `llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `llvm/MCA/HardwareUnits/ResourceManager.h`, `llvm/MCA/Support.h`, `llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`。

### Lines 21-33
```cpp
namespace mca {

#define DEBUG_TYPE "llvm-mca"
ResourceStrategy::~ResourceStrategy() = default;

static uint64_t selectImpl(uint64_t CandidateMask,
                           uint64_t &NextInSequenceMask) {
  // The upper bit set in CandidateMask identifies our next candidate resource.
  CandidateMask = 1ULL << getResourceStateIndex(CandidateMask);
  NextInSequenceMask &= (CandidateMask | (CandidateMask - 1));
  return CandidateMask;
}

```
- **EN**: Defines preprocessor-controlled structure, feature gates, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、特性开关或编译期常量。

### Lines 34-45
```cpp
uint64_t DefaultResourceStrategy::select(uint64_t ReadyMask) {
  // This method assumes that ReadyMask cannot be zero.
  uint64_t CandidateMask = ReadyMask & NextInSequenceMask;
  if (CandidateMask)
    return selectImpl(CandidateMask, NextInSequenceMask);

  NextInSequenceMask = ResourceUnitMask ^ RemovedFromNextInSequence;
  RemovedFromNextInSequence = 0;
  CandidateMask = ReadyMask & NextInSequenceMask;
  if (CandidateMask)
    return selectImpl(CandidateMask, NextInSequenceMask);

```
- **EN**: Implements logic around `select`, `selectImpl`; this block models machine-level execution behavior.
- **CN**: 围绕 `select`, `selectImpl` 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 46-56
```cpp
  NextInSequenceMask = ResourceUnitMask;
  CandidateMask = ReadyMask & NextInSequenceMask;
  return selectImpl(CandidateMask, NextInSequenceMask);
}

void DefaultResourceStrategy::used(uint64_t Mask) {
  if (Mask > NextInSequenceMask) {
    RemovedFromNextInSequence |= Mask;
    return;
  }

```
- **EN**: Implements logic around `selectImpl`, `used`; this block models machine-level execution behavior.
- **CN**: 围绕 `selectImpl`, `used` 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 57-71
```cpp
  NextInSequenceMask &= (~Mask);
  if (NextInSequenceMask)
    return;

  NextInSequenceMask = ResourceUnitMask ^ RemovedFromNextInSequence;
  RemovedFromNextInSequence = 0;
}

static uint64_t computeResourceSizeMask(uint64_t Mask, bool IsAGroup,
                                        unsigned NumUnits) {
  if (IsAGroup)
    return Mask ^ (1ULL << getResourceStateIndex(Mask));
  return (1ULL << NumUnits) - 1;
}

```
- **EN**: Implements logic around `computeResourceSizeMask`, `getResourceStateIndex`; this block models machine-level execution behavior.
- **CN**: 围绕 `computeResourceSizeMask`, `getResourceStateIndex` 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 72-82
```cpp
ResourceState::ResourceState(const MCProcResourceDesc &Desc, unsigned Index,
                             uint64_t Mask)
    : ProcResourceDescIndex(Index), ResourceMask(Mask),
      IsAGroup(llvm::popcount(ResourceMask) > 1),
      ResourceSizeMask(computeResourceSizeMask(Mask, IsAGroup, Desc.NumUnits)),
      BufferSize(Desc.BufferSize) {
  ReadyMask = ResourceSizeMask;
  AvailableSlots = BufferSize == -1 ? 0U : static_cast<unsigned>(BufferSize);
  Unavailable = false;
}

```
- **EN**: Implements logic around `ResourceState`, `ProcResourceDescIndex`, `IsAGroup`, `ResourceSizeMask`, and 2 more symbols; this block models machine-level execution behavior.
- **CN**: 围绕 `ResourceState`, `ProcResourceDescIndex`, `IsAGroup`, `ResourceSizeMask`, and 2 more symbols 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 83-95
```cpp
bool ResourceState::isReady(unsigned NumUnits) const {
  return (!isReserved() || isADispatchHazard()) &&
         (unsigned)llvm::popcount(ReadyMask) >= NumUnits;
}

ResourceStateEvent ResourceState::isBufferAvailable() const {
  if (isADispatchHazard() && isReserved())
    return RS_RESERVED;
  if (!isBuffered() || AvailableSlots)
    return RS_BUFFER_AVAILABLE;
  return RS_BUFFER_UNAVAILABLE;
}

```
- **EN**: Implements logic around `isReady`, `isReserved`, `popcount`, `isBufferAvailable`, and 2 more symbols; this block models machine-level execution behavior.
- **CN**: 围绕 `isReady`, `isReserved`, `popcount`, `isBufferAvailable`, and 2 more symbols 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 96-106
```cpp
#ifndef NDEBUG
void ResourceState::dump() const {
  dbgs() << "MASK=" << format_hex(ResourceMask, 16)
         << ", SZMASK=" << format_hex(ResourceSizeMask, 16)
         << ", RDYMASK=" << format_hex(ReadyMask, 16)
         << ", BufferSize=" << BufferSize
         << ", AvailableSlots=" << AvailableSlots
         << ", Reserved=" << Unavailable << '\n';
}
#endif

```
- **EN**: Defines preprocessor-controlled structure, feature gates, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、特性开关或编译期常量。

### Lines 107-123
```cpp
static std::unique_ptr<ResourceStrategy>
getStrategyFor(const ResourceState &RS) {
  if (RS.isAResourceGroup() || RS.getNumUnits() > 1)
    return std::make_unique<DefaultResourceStrategy>(RS.getReadyMask());
  return std::unique_ptr<ResourceStrategy>(nullptr);
}

ResourceManager::ResourceManager(const MCSchedModel &SM)
    : Resources(SM.getNumProcResourceKinds() - 1),
      Strategies(SM.getNumProcResourceKinds() - 1),
      Resource2Groups(SM.getNumProcResourceKinds() - 1, 0),
      ProcResID2Mask(SM.getNumProcResourceKinds(), 0),
      ResIndex2ProcResID(SM.getNumProcResourceKinds() - 1, 0),
      ProcResUnitMask(0), ReservedResourceGroups(0), AvailableBuffers(~0ULL),
      ReservedBuffers(0) {
  computeProcResourceMasks(SM, ProcResID2Mask);

```
- **EN**: Implements logic around `getStrategyFor`, `isAResourceGroup`, `make_unique`, `unique_ptr`, and 9 more symbols; this block models machine-level execution behavior.
- **CN**: 围绕 `getStrategyFor`, `isAResourceGroup`, `make_unique`, `unique_ptr`, and 9 more symbols 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 124-137
```cpp
  // initialize vector ResIndex2ProcResID.
  for (unsigned I = 1, E = SM.getNumProcResourceKinds(); I < E; ++I) {
    unsigned Index = getResourceStateIndex(ProcResID2Mask[I]);
    ResIndex2ProcResID[Index] = I;
  }

  for (unsigned I = 1, E = SM.getNumProcResourceKinds(); I < E; ++I) {
    uint64_t Mask = ProcResID2Mask[I];
    unsigned Index = getResourceStateIndex(Mask);
    Resources[Index] =
        std::make_unique<ResourceState>(*SM.getProcResource(I), I, Mask);
    Strategies[Index] = getStrategyFor(*Resources[Index]);
  }

```
- **EN**: Implements logic around `getNumProcResourceKinds`, `getResourceStateIndex`, `make_unique`, `getStrategyFor`; this block models machine-level execution behavior.
- **CN**: 围绕 `getNumProcResourceKinds`, `getResourceStateIndex`, `make_unique`, `getStrategyFor` 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 138-155
```cpp
  // Print static resource information on debug mode
  LLVM_DEBUG({
    dbgs() << "\nProcessor resources:\n";
    // Print InvalidUnit first to be consistent with scheduling model indexing
    // schema
    const MCProcResourceDesc &InvalidUnit = *SM.getProcResource(0);
    dbgs() << "[ 0]  - " << format_hex(ProcResID2Mask[0], 16) << " - "
           << InvalidUnit.Name << "\n";
    for (unsigned I = 0, E = Resources.size(); I < E; ++I) {
      const ResourceState &RS = *Resources[I];
      const unsigned ProcResID = RS.getProcResourceID();
      const MCProcResourceDesc &Desc = *SM.getProcResource(ProcResID);
      dbgs() << '[' << format_decimal(ProcResID, 2) << "] "
             << " - " << format_hex(RS.getResourceMask(), 16) << " - "
             << Desc.Name << " (BufferSize=" << RS.getBufferSize() << ")\n";
    }
  });

```
- **EN**: Implements logic around `dbgs`, `getProcResource`, `size`, `getProcResourceID`, and 2 more symbols; this block emits or serializes data to an external representation; models machine-level execution behavior.
- **CN**: 围绕 `dbgs`, `getProcResource`, `size`, `getProcResourceID`, and 2 more symbols 实现具体逻辑；该代码块把数据输出或序列化为外部表示，并建模机器级执行行为。

### Lines 156-175
```cpp
  for (unsigned I = 1, E = SM.getNumProcResourceKinds(); I < E; ++I) {
    uint64_t Mask = ProcResID2Mask[I];
    unsigned Index = getResourceStateIndex(Mask);
    const ResourceState &RS = *Resources[Index];
    if (!RS.isAResourceGroup()) {
      ProcResUnitMask |= Mask;
      continue;
    }

    uint64_t GroupMaskIdx = 1ULL << Index;
    Mask -= GroupMaskIdx;
    while (Mask) {
      // Extract lowest set isolated bit.
      uint64_t Unit = Mask & (-Mask);
      unsigned IndexUnit = getResourceStateIndex(Unit);
      Resource2Groups[IndexUnit] |= GroupMaskIdx;
      Mask ^= Unit;
    }
  }

```
- **EN**: Implements logic around `getNumProcResourceKinds`, `getResourceStateIndex`, `isAResourceGroup`; this block manipulates DWARF/debug-info concepts; models machine-level execution behavior.
- **CN**: 围绕 `getNumProcResourceKinds`, `getResourceStateIndex`, `isAResourceGroup` 实现具体逻辑；该代码块处理 DWARF/调试信息概念，并建模机器级执行行为。

### Lines 176-186
```cpp
  AvailableProcResUnits = ProcResUnitMask;
}

void ResourceManager::setCustomStrategyImpl(std::unique_ptr<ResourceStrategy> S,
                                            uint64_t ResourceMask) {
  unsigned Index = getResourceStateIndex(ResourceMask);
  assert(Index < Resources.size() && "Invalid processor resource index!");
  assert(S && "Unexpected null strategy in input!");
  Strategies[Index] = std::move(S);
}

```
- **EN**: Implements logic around `setCustomStrategyImpl`, `getResourceStateIndex`, `assert`, `move`; this block models machine-level execution behavior.
- **CN**: 围绕 `setCustomStrategyImpl`, `getResourceStateIndex`, `assert`, `move` 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 187-203
```cpp
unsigned ResourceManager::resolveResourceMask(uint64_t Mask) const {
  return ResIndex2ProcResID[getResourceStateIndex(Mask)];
}

unsigned ResourceManager::getNumUnits(uint64_t ResourceID) const {
  return Resources[getResourceStateIndex(ResourceID)]->getNumUnits();
}

// Returns the actual resource consumed by this Use.
// First, is the primary resource ID.
// Second, is the specific sub-resource ID.
ResourceRef ResourceManager::selectPipe(uint64_t ResourceID) {
  unsigned Index = getResourceStateIndex(ResourceID);
  assert(Index < Resources.size() && "Invalid resource use!");
  ResourceState &RS = *Resources[Index];
  assert(RS.isReady() && "No available units to select!");

```
- **EN**: Implements logic around `resolveResourceMask`, `getResourceStateIndex`, `getNumUnits`, `selectPipe`, and 1 more symbols; this block parses or classifies structured input; models machine-level execution behavior.
- **CN**: 围绕 `resolveResourceMask`, `getResourceStateIndex`, `getNumUnits`, `selectPipe`, and 1 more symbols 实现具体逻辑；该代码块解析或分类结构化输入，并建模机器级执行行为。

### Lines 204-214
```cpp
  // Special case where RS is not a group, and it only declares a single
  // resource unit.
  if (!RS.isAResourceGroup() && RS.getNumUnits() == 1)
    return std::make_pair(ResourceID, RS.getReadyMask());

  uint64_t SubResourceID = Strategies[Index]->select(RS.getReadyMask());
  if (RS.isAResourceGroup())
    return selectPipe(SubResourceID);
  return std::make_pair(ResourceID, SubResourceID);
}

```
- **EN**: Implements logic around `isAResourceGroup`, `make_pair`, `select`, `selectPipe`; this block manipulates DWARF/debug-info concepts; models machine-level execution behavior.
- **CN**: 围绕 `isAResourceGroup`, `make_pair`, `select`, `selectPipe` 实现具体逻辑；该代码块处理 DWARF/调试信息概念，并建模机器级执行行为。

### Lines 215-224
```cpp
void ResourceManager::use(const ResourceRef &RR) {
  // Mark the sub-resource referenced by RR as used.
  unsigned RSID = getResourceStateIndex(RR.first);
  ResourceState &RS = *Resources[RSID];
  RS.markSubResourceAsUsed(RR.second);
  // Remember to update the resource strategy for non-group resources with
  // multiple units.
  if (RS.getNumUnits() > 1)
    Strategies[RSID]->used(RR.second);

```
- **EN**: Implements logic around `use`, `getResourceStateIndex`, `markSubResourceAsUsed`, `getNumUnits`, and 1 more symbols; this block models machine-level execution behavior.
- **CN**: 围绕 `use`, `getResourceStateIndex`, `markSubResourceAsUsed`, `getNumUnits`, and 1 more symbols 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 225-244
```cpp
  // If there are still available units in RR.first,
  // then we are done.
  if (RS.isReady())
    return;

  AvailableProcResUnits ^= RR.first;

  // Notify groups that RR.first is no longer available.
  uint64_t Users = Resource2Groups[RSID];
  while (Users) {
    // Extract lowest set isolated bit.
    unsigned GroupIndex = getResourceStateIndex(Users & (-Users));
    ResourceState &CurrentUser = *Resources[GroupIndex];
    CurrentUser.markSubResourceAsUsed(RR.first);
    Strategies[GroupIndex]->used(RR.first);
    // Reset lowest set bit.
    Users &= Users - 1;
  }
}

```
- **EN**: Implements logic around `isReady`, `getResourceStateIndex`, `markSubResourceAsUsed`, `used`; this block models machine-level execution behavior.
- **CN**: 围绕 `isReady`, `getResourceStateIndex`, `markSubResourceAsUsed`, `used` 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 245-254
```cpp
void ResourceManager::release(const ResourceRef &RR) {
  unsigned RSID = getResourceStateIndex(RR.first);
  ResourceState &RS = *Resources[RSID];
  bool WasFullyUsed = !RS.isReady();
  RS.releaseSubResource(RR.second);
  if (!WasFullyUsed)
    return;

  AvailableProcResUnits ^= RR.first;

```
- **EN**: Implements logic around `release`, `getResourceStateIndex`, `isReady`, `releaseSubResource`; this block models machine-level execution behavior.
- **CN**: 围绕 `release`, `getResourceStateIndex`, `isReady`, `releaseSubResource` 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 255-264
```cpp
  // Notify groups that RR.first is now available again.
  uint64_t Users = Resource2Groups[RSID];
  while (Users) {
    unsigned GroupIndex = getResourceStateIndex(Users & (-Users));
    ResourceState &CurrentUser = *Resources[GroupIndex];
    CurrentUser.releaseSubResource(RR.first);
    Users &= Users - 1;
  }
}

```
- **EN**: Implements logic around `getResourceStateIndex`, `releaseSubResource`; this block models machine-level execution behavior.
- **CN**: 围绕 `getResourceStateIndex`, `releaseSubResource` 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 265-284
```cpp
ResourceStateEvent
ResourceManager::canBeDispatched(uint64_t ConsumedBuffers) const {
  if (ConsumedBuffers & ReservedBuffers)
    return ResourceStateEvent::RS_RESERVED;
  if (ConsumedBuffers & (~AvailableBuffers))
    return ResourceStateEvent::RS_BUFFER_UNAVAILABLE;
  return ResourceStateEvent::RS_BUFFER_AVAILABLE;
}

void ResourceManager::reserveBuffers(uint64_t ConsumedBuffers) {
  while (ConsumedBuffers) {
    uint64_t CurrentBuffer = ConsumedBuffers & (-ConsumedBuffers);
    ResourceState &RS = *Resources[getResourceStateIndex(CurrentBuffer)];
    ConsumedBuffers ^= CurrentBuffer;
    assert(RS.isBufferAvailable() == ResourceStateEvent::RS_BUFFER_AVAILABLE);
    if (!RS.reserveBuffer())
      AvailableBuffers ^= CurrentBuffer;
    if (RS.isADispatchHazard()) {
      // Reserve this buffer now, and release it once pipeline resources
      // consumed by the instruction become available again.
```
- **EN**: Implements logic around `canBeDispatched`, `reserveBuffers`, `getResourceStateIndex`, `assert`, and 2 more symbols; this block parses or classifies structured input; models machine-level execution behavior.
- **CN**: 围绕 `canBeDispatched`, `reserveBuffers`, `getResourceStateIndex`, `assert`, and 2 more symbols 实现具体逻辑；该代码块解析或分类结构化输入，并建模机器级执行行为。

### Lines 285-302
```cpp
      // We do this to simulate an in-order dispatch/issue of instructions.
      ReservedBuffers ^= CurrentBuffer;
    }
  }
}

void ResourceManager::releaseBuffers(uint64_t ConsumedBuffers) {
  AvailableBuffers |= ConsumedBuffers;
  while (ConsumedBuffers) {
    uint64_t CurrentBuffer = ConsumedBuffers & (-ConsumedBuffers);
    ResourceState &RS = *Resources[getResourceStateIndex(CurrentBuffer)];
    ConsumedBuffers ^= CurrentBuffer;
    RS.releaseBuffer();
    // Do not unreserve dispatch hazard resource buffers. Wait until all
    // pipeline resources have been freed too.
  }
}

```
- **EN**: Implements logic around `releaseBuffers`, `getResourceStateIndex`, `releaseBuffer`; this block parses or classifies structured input; models machine-level execution behavior.
- **CN**: 围绕 `releaseBuffers`, `getResourceStateIndex`, `releaseBuffer` 实现具体逻辑；该代码块解析或分类结构化输入，并建模机器级执行行为。

### Lines 303-315
```cpp
uint64_t ResourceManager::checkAvailability(const InstrDesc &Desc) const {
  uint64_t BusyResourceMask = 0;
  uint64_t ConsumedResourceMask = 0;
  DenseMap<uint64_t, unsigned> AvailableUnits;

  for (const std::pair<uint64_t, ResourceUsage> &E : Desc.Resources) {
    unsigned NumUnits = E.second.isReserved() ? 0U : E.second.NumUnits;
    const ResourceState &RS = *Resources[getResourceStateIndex(E.first)];
    if (!RS.isReady(NumUnits)) {
      BusyResourceMask |= E.first;
      continue;
    }

```
- **EN**: Implements logic around `checkAvailability`, `isReserved`, `getResourceStateIndex`, `isReady`; this block parses or classifies structured input; models machine-level execution behavior.
- **CN**: 围绕 `checkAvailability`, `isReserved`, `getResourceStateIndex`, `isReady` 实现具体逻辑；该代码块解析或分类结构化输入，并建模机器级执行行为。

### Lines 316-328
```cpp
    if (Desc.HasPartiallyOverlappingGroups && !RS.isAResourceGroup()) {
      unsigned NumAvailableUnits = llvm::popcount(RS.getReadyMask());
      NumAvailableUnits -= NumUnits;
      AvailableUnits[E.first] = NumAvailableUnits;
      if (!NumAvailableUnits)
        ConsumedResourceMask |= E.first;
    }
  }

  BusyResourceMask &= ProcResUnitMask;
  if (BusyResourceMask)
    return BusyResourceMask;

```
- **EN**: Implements logic around `isAResourceGroup`, `popcount`; this block parses or classifies structured input; models machine-level execution behavior.
- **CN**: 围绕 `isAResourceGroup`, `popcount` 实现具体逻辑；该代码块解析或分类结构化输入，并建模机器级执行行为。

### Lines 329-343
```cpp
  BusyResourceMask = Desc.UsedProcResGroups & ReservedResourceGroups;
  if (!Desc.HasPartiallyOverlappingGroups || BusyResourceMask)
    return BusyResourceMask;

  // If this instruction has overlapping groups, make sure that we can
  // select at least one unit per group.
  for (const std::pair<uint64_t, ResourceUsage> &E : Desc.Resources) {
    const ResourceState &RS = *Resources[getResourceStateIndex(E.first)];
    if (!E.second.isReserved() && RS.isAResourceGroup()) {
      uint64_t ReadyMask = RS.getReadyMask() & ~ConsumedResourceMask;
      if (!ReadyMask) {
        BusyResourceMask |= RS.getReadyMask();
        continue;
      }

```
- **EN**: Implements logic around `getResourceStateIndex`, `isReserved`, `getReadyMask`; this block parses or classifies structured input; manipulates DWARF/debug-info concepts; models machine-level execution behavior.
- **CN**: 围绕 `getResourceStateIndex`, `isReserved`, `getReadyMask` 实现具体逻辑；该代码块解析或分类结构化输入，并处理 DWARF/调试信息概念，并建模机器级执行行为。

### Lines 344-357
```cpp
      uint64_t ResourceMask = llvm::bit_floor(ReadyMask);

      auto [it, Inserted] = AvailableUnits.try_emplace(ResourceMask);
      if (Inserted) {
        unsigned Index = getResourceStateIndex(ResourceMask);
        unsigned NumUnits = llvm::popcount(Resources[Index]->getReadyMask());
        it->second = NumUnits;
      }

      if (!it->second) {
        BusyResourceMask |= it->first;
        continue;
      }

```
- **EN**: Implements logic around `bit_floor`, `try_emplace`, `getResourceStateIndex`, `popcount`; this block models machine-level execution behavior.
- **CN**: 围绕 `bit_floor`, `try_emplace`, `getResourceStateIndex`, `popcount` 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 358-369
```cpp
      it->second--;
      if (!it->second)
        ConsumedResourceMask |= it->first;
    }
  }

  return BusyResourceMask;
}

void ResourceManager::issueInstructionImpl(
    const InstrDesc &Desc, SmallVectorImpl<ResourceWithCycles> &Pipes) {

```
- **EN**: Implements logic around `issueInstructionImpl`; this block parses or classifies structured input; models machine-level execution behavior.
- **CN**: 围绕 `issueInstructionImpl` 实现具体逻辑；该代码块解析或分类结构化输入，并建模机器级执行行为。

### Lines 370-385
```cpp
  // Step 1.
  // - Issue writes to non-group resources.
  // - Issue writes to groups with only a single resource unit available.
  // - Update reserved groups (if any)
  // - Add any remaining resource usage requests to a Worklist.
  SmallVector<std::pair<uint64_t, ResourceUsage>, 4> Worklist;

  using ResourceWithUsage = std::pair<uint64_t, ResourceUsage>;

  for (const ResourceWithUsage &R : Desc.Resources) {
    const CycleSegment &CS = R.second.CS;
    if (!CS.size()) {
      releaseResource(R.first);
      continue;
    }

```
- **EN**: Implements logic around `size`, `releaseResource`; this block emits or serializes data to an external representation; manipulates DWARF/debug-info concepts; models machine-level execution behavior.
- **CN**: 围绕 `size`, `releaseResource` 实现具体逻辑；该代码块把数据输出或序列化为外部表示，并处理 DWARF/调试信息概念，并建模机器级执行行为。

### Lines 386-395
```cpp
    assert(CS.begin() == 0 && "Invalid {Start, End} cycles!");
    if (R.second.isReserved()) {
      assert((llvm::popcount(R.first) > 1) && "Expected a group!");
      // Mark this group as reserved.
      assert(R.second.isReserved());
      reserveResource(R.first);
      BusyResources[ResourceRef(R.first, R.first)] += CS.size();
      continue;
    }

```
- **EN**: Implements logic around `assert`, `isReserved`, `reserveResource`, `ResourceRef`; this block models machine-level execution behavior.
- **CN**: 围绕 `assert`, `isReserved`, `reserveResource`, `ResourceRef` 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 396-407
```cpp
    const ResourceState &RS = *Resources[getResourceStateIndex(R.first)];
    if (RS.isAResourceGroup() && RS.getNumReadyUnits() > 1) {
      Worklist.push_back(R);
      continue;
    }

    ResourceRef Pipe = selectPipe(R.first);
    use(Pipe);
    BusyResources[Pipe] += CS.size();
    Pipes.emplace_back(std::make_pair(Pipe, ReleaseAtCycles(CS.size())));
  }

```
- **EN**: Implements logic around `getResourceStateIndex`, `isAResourceGroup`, `push_back`, `selectPipe`, and 3 more symbols; this block models machine-level execution behavior.
- **CN**: 围绕 `getResourceStateIndex`, `isAResourceGroup`, `push_back`, `selectPipe`, and 3 more symbols 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 408-426
```cpp
  // Step 2.
  // Prioritize writes to groups with less available resources.
  // NOTE: this algorithm has quadratic complexity in the worst case scenario.
  // On average, this algorithm is expected to perform quite well and always
  // converge in very few iterations. That is mainly because instructions rarely
  // consume more than two or three resource groups.

  while (!Worklist.empty()) {
    sort(Worklist, [&](const ResourceWithUsage &Lhs,
                       const ResourceWithUsage &Rhs) {
      const ResourceState &LhsRS = *Resources[getResourceStateIndex(Lhs.first)];
      const ResourceState &RhsRS = *Resources[getResourceStateIndex(Rhs.first)];
      uint64_t LhsReadyUnits = LhsRS.getNumReadyUnits();
      uint64_t RhsReadyUnits = RhsRS.getNumReadyUnits();
      if (LhsReadyUnits == RhsReadyUnits)
        return Lhs.first < Rhs.first;
      return LhsReadyUnits < RhsReadyUnits;
    });

```
- **EN**: Implements logic around `empty`, `sort`, `getResourceStateIndex`, `getNumReadyUnits`; this block parses or classifies structured input; emits or serializes data to an external representation; models machine-level execution behavior.
- **CN**: 围绕 `empty`, `sort`, `getResourceStateIndex`, `getNumReadyUnits` 实现具体逻辑；该代码块解析或分类结构化输入，并把数据输出或序列化为外部表示，并建模机器级执行行为。

### Lines 427-441
```cpp
    SmallVector<ResourceWithUsage, 4> NewWorklist;

    for (unsigned I = 0, E = Worklist.size(); I < E; ++I) {
      const auto &Elt = Worklist[I];
      const ResourceState &RS = *Resources[getResourceStateIndex(Elt.first)];

      if (I == 0 || RS.getNumReadyUnits() == 1) {
        ResourceRef Pipe = selectPipe(Elt.first);
        use(Pipe);
        const CycleSegment &CS = Elt.second.CS;
        BusyResources[Pipe] += CS.size();
        Pipes.emplace_back(std::make_pair(Pipe, ReleaseAtCycles(CS.size())));
        continue;
      }

```
- **EN**: Implements logic around `size`, `getResourceStateIndex`, `getNumReadyUnits`, `selectPipe`, and 2 more symbols; this block models machine-level execution behavior.
- **CN**: 围绕 `size`, `getResourceStateIndex`, `getNumReadyUnits`, `selectPipe`, and 2 more symbols 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 442-457
```cpp
      NewWorklist.push_back(Elt);
    }

    swap(NewWorklist, Worklist);
  };
}

void ResourceManager::fastIssueInstruction(
    const InstrDesc &Desc, SmallVectorImpl<ResourceWithCycles> &Pipes) {
  for (const std::pair<uint64_t, ResourceUsage> &R : Desc.Resources) {
    const CycleSegment &CS = R.second.CS;
    if (!CS.size()) {
      releaseResource(R.first);
      continue;
    }

```
- **EN**: Implements logic around `push_back`, `swap`, `fastIssueInstruction`, `size`, and 1 more symbols; this block models machine-level execution behavior.
- **CN**: 围绕 `push_back`, `swap`, `fastIssueInstruction`, `size`, and 1 more symbols 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 458-474
```cpp
    assert(CS.begin() == 0 && "Invalid {Start, End} cycles!");
    if (!R.second.isReserved()) {
      ResourceRef Pipe = selectPipe(R.first);
      use(Pipe);
      BusyResources[Pipe] += CS.size();
      Pipes.emplace_back(std::pair<ResourceRef, ReleaseAtCycles>(
          Pipe, ReleaseAtCycles(CS.size())));
    } else {
      assert((llvm::popcount(R.first) > 1) && "Expected a group!");
      // Mark this group as reserved.
      assert(R.second.isReserved());
      reserveResource(R.first);
      BusyResources[ResourceRef(R.first, R.first)] += CS.size();
    }
  }
}

```
- **EN**: Implements logic around `assert`, `isReserved`, `selectPipe`, `use`, and 5 more symbols; this block models machine-level execution behavior.
- **CN**: 围绕 `assert`, `isReserved`, `selectPipe`, `use`, and 5 more symbols 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 475-489
```cpp
void ResourceManager::cycleEvent(SmallVectorImpl<ResourceRef> &ResourcesFreed) {
  for (std::pair<ResourceRef, unsigned> &BR : BusyResources) {
    if (BR.second)
      BR.second--;
    if (!BR.second) {
      // Release this resource.
      const ResourceRef &RR = BR.first;

      if (llvm::popcount(RR.first) == 1)
        release(RR);
      releaseResource(RR.first);
      ResourcesFreed.push_back(RR);
    }
  }

```
- **EN**: Implements logic around `cycleEvent`, `popcount`, `release`, `releaseResource`, and 1 more symbols; this block models machine-level execution behavior.
- **CN**: 围绕 `cycleEvent`, `popcount`, `release`, `releaseResource`, and 1 more symbols 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 490-502
```cpp
  for (const ResourceRef &RF : ResourcesFreed)
    BusyResources.erase(RF);
}

void ResourceManager::reserveResource(uint64_t ResourceID) {
  const unsigned Index = getResourceStateIndex(ResourceID);
  ResourceState &Resource = *Resources[Index];
  assert(Resource.isAResourceGroup() && !Resource.isReserved() &&
         "Unexpected resource state found!");
  Resource.setReserved();
  ReservedResourceGroups ^= 1ULL << Index;
}

```
- **EN**: Implements logic around `erase`, `reserveResource`, `getResourceStateIndex`, `assert`, and 1 more symbols; this block models machine-level execution behavior.
- **CN**: 围绕 `erase`, `reserveResource`, `getResourceStateIndex`, `assert`, and 1 more symbols 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 503-513
```cpp
void ResourceManager::releaseResource(uint64_t ResourceID) {
  const unsigned Index = getResourceStateIndex(ResourceID);
  ResourceState &Resource = *Resources[Index];
  Resource.clearReserved();
  if (Resource.isAResourceGroup())
    ReservedResourceGroups ^= 1ULL << Index;
  // Now it is safe to release dispatch/issue resources.
  if (Resource.isADispatchHazard())
    ReservedBuffers ^= 1ULL << Index;
}

```
- **EN**: Implements logic around `releaseResource`, `getResourceStateIndex`, `clearReserved`, `isAResourceGroup`, and 1 more symbols; this block models machine-level execution behavior.
- **CN**: 围绕 `releaseResource`, `getResourceStateIndex`, `clearReserved`, `isAResourceGroup`, and 1 more symbols 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 514-515
```cpp
} // namespace mca
} // namespace llvm
```
- **EN**: Introduces declarations for `mca`, `llvm`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `mca`, `llvm` 等声明，建立本文件后续使用的类型或命名空间。

## Key Concepts / 关键概念

- **Machine-code analysis / 机器码分析**:
  - **EN**: Models pipeline behavior and instruction scheduling for llvm-mca.
  - **CN**: 为 llvm-mca 建模流水线行为与指令调度。
- **LLVM container usage / LLVM 容器使用**:
  - **EN**: Relies on LLVM ADT containers for performance-conscious in-memory data management.
  - **CN**: 依赖 LLVM ADT 容器来进行注重性能的内存数据管理。
- **Streaming output / 流式输出**:
  - **EN**: Writes diagnostics, serialized data, or textual representations through LLVM stream APIs.
  - **CN**: 通过 LLVM 流式 API 输出诊断、序列化数据或文本表示。

## Dependencies / 依赖关系

- **Direct LLVM/local includes / 直接的 LLVM/本地包含**: `llvm/MCA/HardwareUnits/ResourceManager.h`, `llvm/MCA/Support.h`, `llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`
- **Subsystem categories / 子系统类别**: llvm-mca simulation interfaces / llvm-mca 模拟接口 (2), support-library helpers / Support 库辅助功能 (2)
