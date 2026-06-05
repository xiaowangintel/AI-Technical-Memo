# WebAssemblySortRegion.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblySortRegion.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file provides backend implementation logic for the WebAssembly backend.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblySortRegion.cpp`，主要负责 WebAssembly 后端的后端实现逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7

```cpp
#include "WebAssemblySortRegion.h"
#include "WebAssemblyExceptionInfo.h"
#include "llvm/CodeGen/MachineLoopInfo.h"

using namespace llvm;
using namespace WebAssembly;
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 8-45

```cpp
namespace llvm {
namespace WebAssembly {
template <>
bool ConcreteSortRegion<MachineLoop>::isLoop() const {
  return true;
}
} // end namespace WebAssembly
} // end namespace llvm

const SortRegion *SortRegionInfo::getRegionFor(const MachineBasicBlock *MBB) {
  const auto *ML = MLI.getLoopFor(MBB);
  const auto *WE = WEI.getExceptionFor(MBB);
  if (!ML && !WE)
    return nullptr;
  // We determine subregion relationship by domination of their headers, i.e.,
  // if region A's header dominates region B's header, B is a subregion of A.
  // WebAssemblyException contains BBs in all its subregions (loops or
  // exceptions), but MachineLoop may not, because MachineLoop does not
  // contain BBs that don't have a path to its header even if they are
  // dominated by its header. So here we should use
  // WE->contains(ML->getHeader()), but not ML->contains(WE->getHeader()).
  if ((ML && !WE) || (ML && WE && WE->contains(ML->getHeader()))) {
    // If the smallest region containing MBB is a loop
    auto [It, Inserted] = LoopMap.try_emplace(ML);
    if (Inserted)
      It->second = std::make_unique<ConcreteSortRegion<MachineLoop>>(ML);
    return It->second.get();
  } else {
    // If the smallest region containing MBB is an exception
    auto [It, Inserted] = ExceptionMap.try_emplace(WE);
    if (Inserted)
      It->second =
          std::make_unique<ConcreteSortRegion<WebAssemblyException>>(WE);
    return It->second.get();
  }
}

MachineBasicBlock *SortRegionInfo::getBottom(const SortRegion *R) {
```
- **EN**: Opens or re-enters namespace scope so backend symbols stay grouped under LLVM naming conventions. Notable symbols in this range include `isLoop`, `getRegionFor`, `getLoopFor`.
- **CN**: 这里进入或重新进入命名空间作用域，使后端符号按照 LLVM 的命名约定组织在一起。 该区间中较显眼的符号包括 `isLoop`, `getRegionFor`, `getLoopFor`。

### Lines 46-78

```cpp
  if (R->isLoop())
    return getBottom(MLI.getLoopFor(R->getHeader()));
  else
    return getBottom(WEI.getExceptionFor(R->getHeader()));
}

MachineBasicBlock *SortRegionInfo::getBottom(const MachineLoop *ML) {
  MachineBasicBlock *Bottom = ML->getHeader();
  for (MachineBasicBlock *MBB : ML->blocks()) {
    if (MBB->getNumber() > Bottom->getNumber())
      Bottom = MBB;
    // MachineLoop does not contain all BBs dominated by its header. BBs that
    // don't have a path back to the loop header aren't included. But for the
    // purpose of CFG sorting and stackification, we need a bottom BB among all
    // BBs that are dominated by the loop header. So we check if there is any
    // WebAssemblyException contained in this loop, and computes the most bottom
    // BB of them all.
    if (MBB->isEHPad()) {
      MachineBasicBlock *ExBottom = getBottom(WEI.getExceptionFor(MBB));
      if (ExBottom->getNumber() > Bottom->getNumber())
        Bottom = ExBottom;
    }
  }
  return Bottom;
}

MachineBasicBlock *SortRegionInfo::getBottom(const WebAssemblyException *WE) {
  MachineBasicBlock *Bottom = WE->getHeader();
  for (MachineBasicBlock *MBB : WE->blocks())
    if (MBB->getNumber() > Bottom->getNumber())
      Bottom = MBB;
  return Bottom;
}
```
- **EN**: Implements helper routine(s) `isLoop`, `getBottom`, `getLoopFor` for this portion of the WebAssembly backend backend implementation logic.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `isLoop`, `getBottom`, `getLoopFor`。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- Backend implementation logic / 后端实现逻辑
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `WebAssemblySortRegion.h`
- `WebAssemblyExceptionInfo.h`
- `llvm/CodeGen/MachineLoopInfo.h`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
