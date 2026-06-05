# SDTRewriter.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Rewrite/SDTRewriter.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: bolt/Rewrite/SDTRewriter.cpp Implement support for System Tap Statically-Defined Trace points stored in .note.stapsdt section.. It also sits in code that implements BOLT rewriting, patching, and binary emission logic. / 该文件实现 BOLT 重写、补丁与二进制输出逻辑。 源码头部说明其职责是：bolt/Rewrite/SDTRewriter.cpp Implement support for System Tap Statically-Defined Trace points stored in .note.stapsdt section.。

## Line-by-Line Analysis / 逐行分析

### Lines 1-13

```cpp
//===- bolt/Rewrite/SDTRewriter.cpp ---------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Implement support for System Tap Statically-Defined Trace points stored in
// .note.stapsdt section.
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 14-22

```cpp
#include "bolt/Core/BinaryFunction.h"
#include "bolt/Core/DebugData.h"
#include "bolt/Rewrite/MetadataRewriter.h"
#include "bolt/Rewrite/MetadataRewriters.h"
#include "bolt/Utils/CommandLineOpts.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/Errc.h"
#include "llvm/Support/Timer.h"
```

- EN: Pulls in 8 header(s) from local project, LLVM dependencies needed by this range. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 8 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里的注释记录了设计假设、不变量或使用说明。

### Lines 23-31

```cpp
using namespace llvm;
using namespace bolt;

namespace opts {
static cl::opt<bool> PrintSDTMarkers("print-sdt",
                                     cl::desc("print all SDT markers"),
                                     cl::Hidden, cl::cat(BoltCategory));
}
```

- EN: Works inside namespace scope `llvm`, `bolt`, `opts` to organize symbols. Declares or implements routines including `desc`, `cat`. Notable symbols here include `desc`, `cat`, `llvm`, `bolt`, `opts`.
- CN: 这里位于命名空间 `llvm`, `bolt`, `opts` 中，用于组织符号作用域。这里声明或实现函数，例如 `desc`, `cat`。这里较值得关注的符号包括 `desc`, `cat`, `llvm`, `bolt`, `opts`。

### Lines 32-43

```cpp
namespace {
class SDTRewriter final : public MetadataRewriter {
  ErrorOr<BinarySection &> SDTSection{std::errc::bad_address};

  struct SDTMarkerInfo {
    uint64_t PC;
    uint64_t Base;
    uint64_t Semaphore;
    StringRef Provider;
    StringRef Name;
    StringRef Args;
```

- EN: Introduces type definitions such as `SDTRewriter`, `SDTMarkerInfo`. Notable symbols here include `SDTRewriter`, `SDTMarkerInfo`.
- CN: 这里引入类型定义，例如 `SDTRewriter`, `SDTMarkerInfo`。这里较值得关注的符号包括 `SDTRewriter`, `SDTMarkerInfo`。

### Lines 44-51

```cpp
    /// The offset of PC within the note section
    unsigned PCOffset;
  };

  /// Map SDT locations to SDT markers info
  using SDTMarkersListType = std::unordered_map<uint64_t, SDTMarkerInfo>;
  SDTMarkersListType SDTMarkers;
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 52-59

```cpp
  /// Read section to populate SDTMarkers.
  void readSection();

  void printSDTMarkers() const;

public:
  SDTRewriter(StringRef Name, BinaryContext &BC) : MetadataRewriter(Name, BC) {}
```

- EN: Declares or implements routines including `readSection`, `printSDTMarkers`, `SDTRewriter`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `readSection`, `printSDTMarkers`, `SDTRewriter`.
- CN: 这里声明或实现函数，例如 `readSection`, `printSDTMarkers`, `SDTRewriter`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `readSection`, `printSDTMarkers`, `SDTRewriter`。

### Lines 60-69

```cpp
  Error preCFGInitializer() override;

  Error postEmitFinalizer() override;
};

void SDTRewriter::readSection() {
  SDTSection = BC.getUniqueSectionByName(".note.stapsdt");
  if (!SDTSection)
    return;
```

- EN: Declares or implements routines including `preCFGInitializer`, `postEmitFinalizer`, `readSection`. Notable symbols here include `preCFGInitializer`, `postEmitFinalizer`, `readSection`.
- CN: 这里声明或实现函数，例如 `preCFGInitializer`, `postEmitFinalizer`, `readSection`。这里较值得关注的符号包括 `preCFGInitializer`, `postEmitFinalizer`, `readSection`。

### Lines 70-79

```cpp
  StringRef Buf = SDTSection->getContents();
  DataExtractor DE = DataExtractor(Buf, BC.AsmInfo->isLittleEndian());
  uint64_t Offset = 0;

  while (DE.isValidOffset(Offset)) {
    uint32_t NameSz = DE.getU32(&Offset);
    DE.getU32(&Offset); // skip over DescSz
    uint32_t Type = DE.getU32(&Offset);
    Offset = alignTo(Offset, 4);
```

- EN: Declares or implements routines including `getContents`, `DataExtractor`, `alignTo`. Notable symbols here include `getContents`, `DataExtractor`, `alignTo`.
- CN: 这里声明或实现函数，例如 `getContents`, `DataExtractor`, `alignTo`。这里较值得关注的符号包括 `getContents`, `DataExtractor`, `alignTo`。

### Lines 80-88

```cpp
    if (Type != 3)
      errs() << "BOLT-WARNING: SDT note type \"" << Type
             << "\" is not expected\n";

    if (NameSz == 0)
      errs() << "BOLT-WARNING: SDT note has empty name\n";

    StringRef Name = DE.getCStr(&Offset);
```

- EN: Declares or implements routines including `errs`. Notable symbols here include `errs`.
- CN: 这里声明或实现函数，例如 `errs`。这里较值得关注的符号包括 `errs`。

### Lines 89-105

```cpp
    if (Name != "stapsdt")
      errs() << "BOLT-WARNING: SDT note name \"" << Name
             << "\" is not expected\n";

    // Parse description
    SDTMarkerInfo Marker;
    Marker.PCOffset = Offset;
    Marker.PC = DE.getU64(&Offset);
    Marker.Base = DE.getU64(&Offset);
    Marker.Semaphore = DE.getU64(&Offset);
    Marker.Provider = DE.getCStr(&Offset);
    Marker.Name = DE.getCStr(&Offset);
    Marker.Args = DE.getCStr(&Offset);
    Offset = alignTo(Offset, 4);
    SDTMarkers[Marker.PC] = Marker;
  }
```

- EN: Declares or implements routines including `errs`, `alignTo`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `errs`, `alignTo`.
- CN: 这里声明或实现函数，例如 `errs`, `alignTo`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `errs`, `alignTo`。

### Lines 106-113

```cpp
  if (opts::PrintSDTMarkers)
    printSDTMarkers();
}

Error SDTRewriter::preCFGInitializer() {
  // Populate SDTMarkers.
  readSection();
```

- EN: Declares or implements routines including `printSDTMarkers`, `preCFGInitializer`, `readSection`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `printSDTMarkers`, `preCFGInitializer`, `readSection`.
- CN: 这里声明或实现函数，例如 `printSDTMarkers`, `preCFGInitializer`, `readSection`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `printSDTMarkers`, `preCFGInitializer`, `readSection`。

### Lines 114-126

```cpp
  // Mark nop instructions referenced by SDT and the containing function.
  for (const uint64_t PC : llvm::make_first_range(SDTMarkers)) {
    BinaryFunction *BF = BC.getBinaryFunctionContainingAddress(PC);

    if (!BF || !BC.shouldEmit(*BF))
      continue;

    const uint64_t Offset = PC - BF->getAddress();
    MCInst *Inst = BF->getInstructionAtOffset(Offset);
    if (!Inst)
      return createStringError(errc::executable_format_error,
                               "no instruction matches SDT offset");
```

- EN: Declares or implements routines including `getAddress`, `getInstructionAtOffset`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getAddress`, `getInstructionAtOffset`.
- CN: 这里声明或实现函数，例如 `getAddress`, `getInstructionAtOffset`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getAddress`, `getInstructionAtOffset`。

### Lines 127-135

```cpp
    if (!BC.MIB->isNoop(*Inst))
      return createStringError(std::make_error_code(std::errc::not_supported),
                               "nop instruction expected at SDT offset");

    BC.MIB->setOffset(*Inst, static_cast<uint32_t>(Offset));

    BF->setHasSDTMarker(true);
  }
```

- EN: Declares or implements routines including `setOffset`, `setHasSDTMarker`. Notable symbols here include `setOffset`, `setHasSDTMarker`.
- CN: 这里声明或实现函数，例如 `setOffset`, `setHasSDTMarker`。这里较值得关注的符号包括 `setOffset`, `setHasSDTMarker`。

### Lines 136-144

```cpp
  return Error::success();
}

Error SDTRewriter::postEmitFinalizer() {
  if (!SDTSection)
    return Error::success();

  SDTSection->registerPatcher(std::make_unique<SimpleBinaryPatcher>());
```

- EN: Declares or implements routines including `postEmitFinalizer`, `registerPatcher`. Notable symbols here include `postEmitFinalizer`, `registerPatcher`.
- CN: 这里声明或实现函数，例如 `postEmitFinalizer`, `registerPatcher`。这里较值得关注的符号包括 `postEmitFinalizer`, `registerPatcher`。

### Lines 145-158

```cpp
  SimpleBinaryPatcher *SDTNotePatcher =
      static_cast<SimpleBinaryPatcher *>(SDTSection->getPatcher());
  for (auto &SDTInfoKV : SDTMarkers) {
    const uint64_t OriginalAddress = SDTInfoKV.first;
    const SDTMarkerInfo &SDTInfo = SDTInfoKV.second;
    const BinaryFunction *F =
        BC.getBinaryFunctionContainingAddress(OriginalAddress);
    if (!F)
      continue;
    const uint64_t NewAddress =
        F->translateInputToOutputAddress(OriginalAddress);
    SDTNotePatcher->addLE64Patch(SDTInfo.PCOffset, NewAddress);
  }
```

- EN: Declares or implements routines including `translateInputToOutputAddress`, `addLE64Patch`. Notable symbols here include `translateInputToOutputAddress`, `addLE64Patch`.
- CN: 这里声明或实现函数，例如 `translateInputToOutputAddress`, `addLE64Patch`。这里较值得关注的符号包括 `translateInputToOutputAddress`, `addLE64Patch`。

### Lines 159-173

```cpp
  return Error::success();
}

void SDTRewriter::printSDTMarkers() const {
  outs() << "BOLT-INFO: Number of SDT markers is " << SDTMarkers.size() << "\n";
  for (const SDTMarkerInfo &Marker : llvm::make_second_range(SDTMarkers)) {
    outs() << "BOLT-INFO: PC: " << utohexstr(Marker.PC)
           << ", Base: " << utohexstr(Marker.Base)
           << ", Semaphore: " << utohexstr(Marker.Semaphore)
           << ", Provider: " << Marker.Provider << ", Name: " << Marker.Name
           << ", Args: " << Marker.Args << "\n";
  }
}
} // namespace
```

- EN: Declares or implements routines including `printSDTMarkers`, `outs`, `utohexstr`. Notable symbols here include `printSDTMarkers`, `outs`, `utohexstr`.
- CN: 这里声明或实现函数，例如 `printSDTMarkers`, `outs`, `utohexstr`。这里较值得关注的符号包括 `printSDTMarkers`, `outs`, `utohexstr`。

### Lines 174-177

```cpp
std::unique_ptr<MetadataRewriter>
llvm::bolt::createSDTRewriter(BinaryContext &BC) {
  return std::make_unique<SDTRewriter>("sdt-rewriter", BC);
}
```

- EN: Declares or implements routines including `createSDTRewriter`. Notable symbols here include `createSDTRewriter`.
- CN: 这里声明或实现函数，例如 `createSDTRewriter`。这里较值得关注的符号包括 `createSDTRewriter`。

## Key Concepts / 关键概念

- `SDTRewriter`: class or struct interface / 类或结构体接口
- `SDTMarkerInfo`: class or struct interface / 类或结构体接口
- `desc`: function or method entry point / 函数或方法入口
- `cat`: function or method entry point / 函数或方法入口
- `readSection`: function or method entry point / 函数或方法入口
- `printSDTMarkers`: function or method entry point / 函数或方法入口
- `SDTRewriter`: function or method entry point / 函数或方法入口
- `llvm`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Core/BinaryFunction.h`, `bolt/Core/DebugData.h`, `bolt/Rewrite/MetadataRewriter.h`, `bolt/Rewrite/MetadataRewriters.h`, `bolt/Utils/CommandLineOpts.h`
- LLVM headers / LLVM 头文件: `llvm/Support/CommandLine.h`, `llvm/Support/Errc.h`, `llvm/Support/Timer.h`
- Directory context / 目录上下文: `bolt/lib/Rewrite` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Rewrite` 下的相邻文件通常与本文件协作组成对应子系统
