# WebAssemblyTargetStreamer.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/MCTargetDesc/WebAssemblyTargetStreamer.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file defines WebAssembly-specific target streamer classes. These are for implementing support for target-specific assembly directives.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/MCTargetDesc/WebAssemblyTargetStreamer.cpp`，主要负责 WebAssembly 后端的该后端的 MC 层支持。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//==-- WebAssemblyTargetStreamer.cpp - WebAssembly Target Streamer Methods --=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-12

```cpp
//===----------------------------------------------------------------------===//
///
/// \file
/// This file defines WebAssembly-specific target streamer classes.
/// These are for implementing support for target-specific assembly directives.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。

### Lines 13-19

```cpp
//===----------------------------------------------------------------------===//

#include "MCTargetDesc/WebAssemblyTargetStreamer.h"
#include "MCTargetDesc/WebAssemblyMCAsmInfo.h"
#include "MCTargetDesc/WebAssemblyMCTypeUtilities.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCSectionWasm.h"
```
- **EN**: Pulls in direct dependencies required by this MC layer support for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. The logic interacts with LLVM's MC layer.
- **CN**: 这一段引入该该后端的 MC 层支持所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 20-57

```cpp
#include "llvm/MC/MCSymbolWasm.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/FormattedStream.h"
using namespace llvm;

WebAssemblyTargetStreamer::WebAssemblyTargetStreamer(MCStreamer &S)
    : MCTargetStreamer(S) {}

void WebAssemblyTargetStreamer::emitValueType(wasm::ValType Type) {
  Streamer.emitIntValue(uint8_t(Type), 1);
}

WebAssemblyTargetAsmStreamer::WebAssemblyTargetAsmStreamer(
    MCStreamer &S, formatted_raw_ostream &OS)
    : WebAssemblyTargetStreamer(S), OS(OS) {}

WebAssemblyTargetWasmStreamer::WebAssemblyTargetWasmStreamer(MCStreamer &S)
    : WebAssemblyTargetStreamer(S) {}

static void printTypes(formatted_raw_ostream &OS,
                       ArrayRef<wasm::ValType> Types) {
  bool First = true;
  for (auto Type : Types) {
    if (First)
      First = false;
    else
      OS << ", ";
    OS << WebAssembly::typeToString(Type);
  }
  OS << '\n';
}

void WebAssemblyTargetAsmStreamer::emitLocal(ArrayRef<wasm::ValType> Types) {
  if (!Types.empty()) {
    OS << "\t.local  \t";
    printTypes(OS, Types);
  }
}
```
- **EN**: Pulls in direct dependencies required by this MC layer support for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. The logic interacts with LLVM's MC layer.
- **CN**: 这一段引入该该后端的 MC 层支持所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 58-95

```cpp

void WebAssemblyTargetAsmStreamer::emitFunctionType(const MCSymbolWasm *Sym) {
  assert(Sym->isFunction());
  OS << "\t.functype\t" << Sym->getName() << " ";
  OS << WebAssembly::signatureToString(Sym->getSignature());
  OS << "\n";
}

void WebAssemblyTargetAsmStreamer::emitGlobalType(const MCSymbolWasm *Sym) {
  assert(Sym->isGlobal());
  OS << "\t.globaltype\t" << Sym->getName() << ", "
     << WebAssembly::typeToString(
            static_cast<wasm::ValType>(Sym->getGlobalType().Type));
  if (!Sym->getGlobalType().Mutable)
    OS << ", immutable";
  OS << '\n';
}

void WebAssemblyTargetAsmStreamer::emitTableType(const MCSymbolWasm *Sym) {
  assert(Sym->isTable());
  const wasm::WasmTableType &Type = Sym->getTableType();
  OS << "\t.tabletype\t" << Sym->getName() << ", "
     << WebAssembly::typeToString(static_cast<wasm::ValType>(Type.ElemType));
  bool HasMaximum = Type.Limits.Flags & wasm::WASM_LIMITS_FLAG_HAS_MAX;
  if (Type.Limits.Minimum != 0 || HasMaximum) {
    OS << ", " << Type.Limits.Minimum;
    if (HasMaximum)
      OS << ", " << Type.Limits.Maximum;
  }
  OS << '\n';
}

void WebAssemblyTargetAsmStreamer::emitTagType(const MCSymbolWasm *Sym) {
  assert(Sym->isTag());
  OS << "\t.tagtype\t" << Sym->getName() << " ";
  OS << WebAssembly::typeListToString(Sym->getSignature()->Params);
  OS << "\n";
}
```
- **EN**: Implements helper routine(s) `emitFunctionType`, `isFunction`, `getName` for this portion of the WebAssembly backend MC layer support for the backend.
- **CN**: 这里实现了 WebAssembly 后端该部分该后端的 MC 层支持所需的辅助例程 `emitFunctionType`, `isFunction`, `getName`。

### Lines 96-133

```cpp

void WebAssemblyTargetAsmStreamer::emitImportModule(const MCSymbolWasm *Sym,
                                                    StringRef ImportModule) {
  OS << "\t.import_module\t" << Sym->getName() << ", \"" << ImportModule
     << "\"\n";
}

void WebAssemblyTargetAsmStreamer::emitImportName(const MCSymbolWasm *Sym,
                                                  StringRef ImportName) {
  OS << "\t.import_name\t" << Sym->getName() << ", \"" << ImportName << "\"\n";
}

void WebAssemblyTargetAsmStreamer::emitExportName(const MCSymbolWasm *Sym,
                                                  StringRef ExportName) {
  OS << "\t.export_name\t" << Sym->getName() << ", \"" << ExportName << "\"\n";
}

void WebAssemblyTargetAsmStreamer::emitIndIdx(const MCExpr *Value) {
  OS << "\t.indidx\t";
  getContext().getAsmInfo().printExpr(OS, *Value);
  OS << '\n';
}

void WebAssemblyTargetWasmStreamer::emitLocal(ArrayRef<wasm::ValType> Types) {
  SmallVector<std::pair<wasm::ValType, uint32_t>, 4> Grouped;
  for (auto Type : Types) {
    if (Grouped.empty() || Grouped.back().first != Type)
      Grouped.push_back(std::make_pair(Type, 1));
    else
      ++Grouped.back().second;
  }

  Streamer.emitULEB128IntValue(Grouped.size());
  for (auto Pair : Grouped) {
    Streamer.emitULEB128IntValue(Pair.second);
    emitValueType(Pair.first);
  }
}
```
- **EN**: Implements helper routine(s) `emitImportModule`, `getName`, `emitImportName` for this portion of the WebAssembly backend MC layer support for the backend. The logic interacts with LLVM's MC layer.
- **CN**: 这里实现了 WebAssembly 后端该部分该后端的 MC 层支持所需的辅助例程 `emitImportModule`, `getName`, `emitImportName`。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 134-137

```cpp

void WebAssemblyTargetWasmStreamer::emitIndIdx(const MCExpr *Value) {
  llvm_unreachable(".indidx encoding not yet implemented");
}
```
- **EN**: Implements helper routine(s) `emitIndIdx`, `llvm_unreachable` for this portion of the WebAssembly backend MC layer support for the backend. The logic interacts with LLVM's MC layer.
- **CN**: 这里实现了 WebAssembly 后端该部分该后端的 MC 层支持所需的辅助例程 `emitIndIdx`, `llvm_unreachable`。 相关逻辑会与 LLVM 的 MC 层交互。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- MC layer support for the backend / 该后端的 MC 层支持
- Assembly or MC emission / 汇编或 MC 发射
- Object format integration / 目标文件格式集成
- Vector or SIMD behavior / 向量或 SIMD 行为
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `MCTargetDesc/WebAssemblyTargetStreamer.h`
- `MCTargetDesc/WebAssemblyMCAsmInfo.h`
- `MCTargetDesc/WebAssemblyMCTypeUtilities.h`
- `llvm/MC/MCContext.h`
- `llvm/MC/MCSectionWasm.h`
- `llvm/MC/MCSymbolWasm.h`
- `llvm/Support/ErrorHandling.h`
- `llvm/Support/FormattedStream.h`

### Important Collaborators / 重要协作组件

- LLVM MC layer / LLVM MC 层
- LLVM Support utilities / LLVM Support 工具层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
