# TargetRegistry.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/MC/TargetRegistry.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements Target registration.
  - **CN**: 实现 LLVM MC 与目标选择基础设施中的目标注册与查找逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===--- TargetRegistry.cpp - Target registration -------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件级说明。

### Lines 8-21
```cpp

#include "llvm/MC/TargetRegistry.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/MC/MCAsmBackend.h"
#include "llvm/MC/MCCodeEmitter.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCInstPrinter.h"
#include "llvm/MC/MCLFI.h"
#include "llvm/MC/MCObjectStreamer.h"
#include "llvm/MC/MCObjectWriter.h"
#include "llvm/Support/Regex.h"
#include "llvm/Support/raw_ostream.h"
#include <cassert>
```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/MC/TargetRegistry.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/StringRef.h`, `llvm/MC/MCAsmBackend.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/MC/TargetRegistry.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/StringRef.h`, `llvm/MC/MCAsmBackend.h`。

### Lines 22-31
```cpp
#include <vector>
using namespace llvm;

// Clients are responsible for avoid race conditions in registration.
static Target *FirstTarget = nullptr;

bool Target::isValidFeatureListFormat(StringRef Features) {
  if (Features.empty())
    return true;

```
- **EN**: Pulls in the headers needed for this implementation, including `vector`.
- **CN**: 引入该实现所需的头文件，其中包括 `vector`。

### Lines 32-45
```cpp
  static const llvm::Regex pattern("^([+-][^,]+)(,[+-][^,]+)*,?$");
  return pattern.match(Features);
}

MCStreamer *Target::createMCObjectStreamer(
    const Triple &T, MCContext &Ctx, std::unique_ptr<MCAsmBackend> TAB,
    std::unique_ptr<MCObjectWriter> OW, std::unique_ptr<MCCodeEmitter> Emitter,
    const MCSubtargetInfo &STI) const {
  MCStreamer *S = nullptr;
  switch (T.getObjectFormat()) {
  case Triple::UnknownObjectFormat:
    llvm_unreachable("Unknown object format");
  case Triple::COFF:
    assert((T.isOSWindows() || T.isUEFI()) &&
```
- **EN**: Implements logic around `pattern`, `match`, `createMCObjectStreamer`, `llvm_unreachable`, and 1 more symbols; this block drives emission, layout, or binary encoding behavior; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `pattern`, `match`, `createMCObjectStreamer`, `llvm_unreachable`, and 1 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 46-59
```cpp
           "only Windows and UEFI COFF are supported");
    S = COFFStreamerCtorFn(Ctx, std::move(TAB), std::move(OW),
                           std::move(Emitter));
    break;
  case Triple::MachO:
    if (MachOStreamerCtorFn)
      S = MachOStreamerCtorFn(Ctx, std::move(TAB), std::move(OW),
                              std::move(Emitter));
    else
      S = createMachOStreamer(Ctx, std::move(TAB), std::move(OW),
                              std::move(Emitter), false);
    break;
  case Triple::ELF:
    if (ELFStreamerCtorFn)
```
- **EN**: Implements logic around `COFFStreamerCtorFn`, `move`, `MachOStreamerCtorFn`, `createMachOStreamer`; this block drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `COFFStreamerCtorFn`, `move`, `MachOStreamerCtorFn`, `createMachOStreamer` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为。

### Lines 60-73
```cpp
      S = ELFStreamerCtorFn(T, Ctx, std::move(TAB), std::move(OW),
                            std::move(Emitter));
    else
      S = createELFStreamer(Ctx, std::move(TAB), std::move(OW),
                            std::move(Emitter));
    break;
  case Triple::Wasm:
    S = createWasmStreamer(Ctx, std::move(TAB), std::move(OW),
                           std::move(Emitter));
    break;
  case Triple::GOFF:
    S = createGOFFStreamer(Ctx, std::move(TAB), std::move(OW),
                           std::move(Emitter));
    break;
```
- **EN**: Implements logic around `ELFStreamerCtorFn`, `move`, `createELFStreamer`, `createWasmStreamer`, and 1 more symbols; this block drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `ELFStreamerCtorFn`, `move`, `createELFStreamer`, `createWasmStreamer`, and 1 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为。

### Lines 74-87
```cpp
  case Triple::XCOFF:
    S = XCOFFStreamerCtorFn(T, Ctx, std::move(TAB), std::move(OW),
                            std::move(Emitter));
    break;
  case Triple::SPIRV:
    S = createSPIRVStreamer(Ctx, std::move(TAB), std::move(OW),
                            std::move(Emitter));
    break;
  case Triple::DXContainer:
    S = createDXContainerStreamer(Ctx, std::move(TAB), std::move(OW),
                                  std::move(Emitter));
    break;
  }
  if (ObjectTargetStreamerCtorFn)
```
- **EN**: Implements logic around `XCOFFStreamerCtorFn`, `move`, `createSPIRVStreamer`, `createDXContainerStreamer`; this block drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `XCOFFStreamerCtorFn`, `move`, `createSPIRVStreamer`, `createDXContainerStreamer` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为。

### Lines 88-101
```cpp
    ObjectTargetStreamerCtorFn(*S, STI);
  if (T.isLFI())
    initializeLFIMCStreamer(*S, Ctx, T);
  return S;
}

MCStreamer *Target::createAsmStreamer(MCContext &Ctx,
                                      std::unique_ptr<formatted_raw_ostream> OS,
                                      std::unique_ptr<MCInstPrinter> IP,
                                      std::unique_ptr<MCCodeEmitter> CE,
                                      std::unique_ptr<MCAsmBackend> TAB) const {
  MCInstPrinter *Printer = IP.get();
  formatted_raw_ostream &OSRef = *OS;
  MCStreamer *S;
```
- **EN**: Implements logic around `ObjectTargetStreamerCtorFn`, `initializeLFIMCStreamer`, `createAsmStreamer`, `get`; this block drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `ObjectTargetStreamerCtorFn`, `initializeLFIMCStreamer`, `createAsmStreamer`, `get` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 102-108
```cpp
  if (AsmStreamerCtorFn)
    S = AsmStreamerCtorFn(Ctx, std::move(OS), std::move(IP), std::move(CE),
                          std::move(TAB));
  else
    S = llvm::createAsmStreamer(Ctx, std::move(OS), std::move(IP),
                                std::move(CE), std::move(TAB));

```
- **EN**: Implements logic around `AsmStreamerCtorFn`, `move`, `createAsmStreamer`; this block drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `AsmStreamerCtorFn`, `move`, `createAsmStreamer` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为。

### Lines 109-116
```cpp
  createAsmTargetStreamer(*S, OSRef, Printer);
  return S;
}

iterator_range<TargetRegistry::iterator> TargetRegistry::targets() {
  return make_range(iterator(FirstTarget), iterator());
}

```
- **EN**: Implements logic around `createAsmTargetStreamer`, `targets`, `make_range`; this block drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `createAsmTargetStreamer`, `targets`, `make_range` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 117-127
```cpp
const Target *TargetRegistry::lookupTarget(StringRef ArchName,
                                           Triple &TheTriple,
                                           std::string &Error) {
  // Allocate target machine.  First, check whether the user has explicitly
  // specified an architecture to compile for. If so we have to look it up by
  // name, because it might be a backend that has no mapping to a target triple.
  const Target *TheTarget = nullptr;
  if (!ArchName.empty()) {
    auto I = find_if(targets(),
                     [&](const Target &T) { return ArchName == T.getName(); });

```
- **EN**: Implements logic around `lookupTarget`, `find_if`, `getName`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `lookupTarget`, `find_if`, `getName` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 128-134
```cpp
    if (I == targets().end()) {
      Error = ("invalid target '" + ArchName + "'.").str();
      return nullptr;
    }

    TheTarget = &*I;

```
- **EN**: Implements logic around `str`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `str` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 135-148
```cpp
    // Adjust the triple to match (if known), otherwise stick with the
    // given triple.
    Triple::ArchType Type = Triple::getArchTypeForLLVMName(ArchName);
    if (Type != Triple::UnknownArch)
      TheTriple.setArch(Type);
  } else {
    // Get the target specific parser.
    std::string TempError;
    TheTarget = TargetRegistry::lookupTarget(TheTriple, TempError);
    if (!TheTarget) {
      Error = "unable to get target for '" + TheTriple.getTriple() +
              "', see --version and --triple.";
      return nullptr;
    }
```
- **EN**: Implements logic around `getArchTypeForLLVMName`, `setArch`, `lookupTarget`, `getTriple`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `getArchTypeForLLVMName`, `setArch`, `lookupTarget`, `getTriple` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 149-162
```cpp
  }

  return TheTarget;
}

const Target *TargetRegistry::lookupTarget(const Triple &TT,
                                           std::string &Error) {
  // Provide special warning when no targets are initialized.
  if (targets().begin() == targets().end()) {
    Error = "Unable to find target for this triple (no targets are registered)";
    return nullptr;
  }
  Triple::ArchType Arch = TT.getArch();
  auto ArchMatch = [&](const Target &T) { return T.ArchMatchFn(Arch); };
```
- **EN**: Implements logic around `lookupTarget`, `triple`, `getArch`, `ArchMatchFn`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `lookupTarget`, `triple`, `getArch`, `ArchMatchFn` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 163-170
```cpp
  auto I = find_if(targets(), ArchMatch);

  if (I == targets().end()) {
    Error =
        "No available targets are compatible with triple \"" + TT.str() + "\"";
    return nullptr;
  }

```
- **EN**: Implements logic around `find_if`, `str`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `find_if`, `str` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 171-177
```cpp
  auto J = std::find_if(std::next(I), targets().end(), ArchMatch);
  if (J != targets().end()) {
    Error = std::string("Cannot choose between targets \"") + I->Name +
            "\" and \"" + J->Name + "\"";
    return nullptr;
  }

```
- **EN**: Implements logic around `find_if`, `string`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `find_if`, `string` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 178-188
```cpp
  return &*I;
}

void TargetRegistry::RegisterTarget(Target &T, const char *Name,
                                    const char *ShortDesc,
                                    const char *BackendName,
                                    Target::ArchMatchFnTy ArchMatchFn,
                                    bool HasJIT) {
  assert(Name && ShortDesc && ArchMatchFn &&
         "Missing required target information!");

```
- **EN**: Implements logic around `RegisterTarget`, `assert`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `RegisterTarget`, `assert` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 189-197
```cpp
  // Check if this target has already been initialized, we allow this as a
  // convenience to some clients.
  if (T.Name)
    return;

  // Add to the list of targets.
  T.Next = FirstTarget;
  FirstTarget = &T;

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 198-204
```cpp
  T.Name = Name;
  T.ShortDesc = ShortDesc;
  T.BackendName = BackendName;
  T.ArchMatchFn = ArchMatchFn;
  T.HasJIT = HasJIT;
}

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 205-218
```cpp
static int TargetArraySortFn(const std::pair<StringRef, const Target *> *LHS,
                             const std::pair<StringRef, const Target *> *RHS) {
  return LHS->first.compare(RHS->first);
}

void TargetRegistry::printRegisteredTargetsForVersion(raw_ostream &OS) {
  std::vector<std::pair<StringRef, const Target*> > Targets;
  size_t Width = 0;
  for (const auto &T : TargetRegistry::targets()) {
    Targets.push_back(std::make_pair(T.getName(), &T));
    Width = std::max(Width, Targets.back().first.size());
  }
  array_pod_sort(Targets.begin(), Targets.end(), TargetArraySortFn);

```
- **EN**: Implements logic around `TargetArraySortFn`, `compare`, `printRegisteredTargetsForVersion`, `push_back`, and 2 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `TargetArraySortFn`, `compare`, `printRegisteredTargetsForVersion`, `push_back`, and 2 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 219-228
```cpp
  OS << "\n";
  OS << "  Registered Targets:\n";
  for (const auto &Target : Targets) {
    OS << "    " << Target.first;
    OS.indent(Width - Target.first.size())
        << " - " << Target.second->getShortDescription() << '\n';
  }
  if (Targets.empty())
    OS << "    (none)\n";
}
```
- **EN**: Implements logic around `indent`, `getShortDescription`.
- **CN**: 围绕 `indent`, `getShortDescription` 实现具体逻辑。

## Key Concepts / 关键概念

- **Target registration / 目标注册**:
  - **EN**: Maps LLVM targets to MC factories such as asm info, code emitters, and object writers
  - **CN**: 将 LLVM 目标映射到 asm info、编码器、写出器等 MC 工厂函数
- **Directive handling / 伪指令处理**:
  - **EN**: Interprets assembler directives and maps them to streamer operations
  - **CN**: 解释汇编伪指令并把它们映射为 streamer 操作
- **MC instruction model / MC 指令模型**:
  - **EN**: Uses MCInst/MCOperand abstractions to represent machine instructions generically
  - **CN**: 使用 MCInst/MCOperand 抽象以通用方式表示机器指令

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/MC/TargetRegistry.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/StringRef.h`, `llvm/MC/MCAsmBackend.h`, `llvm/MC/MCCodeEmitter.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCInstPrinter.h`, `llvm/MC/MCLFI.h`, `llvm/MC/MCObjectStreamer.h`, `llvm/MC/MCObjectWriter.h`, `llvm/Support/Regex.h`, `llvm/Support/raw_ostream.h` ... (+2 more)
- **LLVM subsystems / LLVM 子系统**: MC, Support
