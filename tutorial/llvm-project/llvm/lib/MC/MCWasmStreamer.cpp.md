# MCWasmStreamer.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/MC/MCWasmStreamer.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file assembles .s files and emits Wasm .o object files.
  - **CN**: 实现该文件在 LLVM 中对应的子系统逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- lib/MC/MCWasmStreamer.cpp - Wasm Object Output ---------------------===//
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
//
// This file assembles .s files and emits Wasm .o object files.
//
//===----------------------------------------------------------------------===//

#include "llvm/MC/MCWasmStreamer.h"
#include "llvm/MC/MCAsmBackend.h"
#include "llvm/MC/MCAssembler.h"
#include "llvm/MC/MCCodeEmitter.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCFixup.h"
#include "llvm/MC/MCObjectStreamer.h"
#include "llvm/MC/MCSection.h"
```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/MC/MCWasmStreamer.h`, `llvm/MC/MCAsmBackend.h`, `llvm/MC/MCAssembler.h`, `llvm/MC/MCCodeEmitter.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/MC/MCWasmStreamer.h`, `llvm/MC/MCAsmBackend.h`, `llvm/MC/MCAssembler.h`, `llvm/MC/MCCodeEmitter.h`。

### Lines 22-33
```cpp
#include "llvm/MC/MCSectionWasm.h"
#include "llvm/MC/MCSymbol.h"
#include "llvm/MC/MCSymbolWasm.h"
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Support/ErrorHandling.h"

namespace llvm {
class MCContext;
class MCStreamer;
class MCSubtargetInfo;
} // namespace llvm

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/MC/MCSectionWasm.h`, `llvm/MC/MCSymbol.h`, `llvm/MC/MCSymbolWasm.h`, `llvm/MC/TargetRegistry.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/MC/MCSectionWasm.h`, `llvm/MC/MCSymbol.h`, `llvm/MC/MCSymbolWasm.h`, `llvm/MC/TargetRegistry.h`。

### Lines 34-41
```cpp
using namespace llvm;

MCWasmStreamer::~MCWasmStreamer() = default; // anchor.

void MCWasmStreamer::emitLabel(MCSymbol *S, SMLoc Loc) {
  auto *Symbol = static_cast<MCSymbolWasm *>(S);
  MCObjectStreamer::emitLabel(Symbol, Loc);

```
- **EN**: Introduces declarations for `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 42-52
```cpp
  const MCSectionWasm &Section =
      static_cast<const MCSectionWasm &>(*getCurrentSectionOnly());
  if (Section.getSegmentFlags() & wasm::WASM_SEG_FLAG_TLS)
    Symbol->setTLS();
}

void MCWasmStreamer::emitLabelAtPos(MCSymbol *S, SMLoc Loc, MCFragment &F,
                                    uint64_t Offset) {
  auto *Symbol = static_cast<MCSymbolWasm *>(S);
  MCObjectStreamer::emitLabelAtPos(Symbol, Loc, F, Offset);

```
- **EN**: Implements logic around `getCurrentSectionOnly`, `setTLS`, `emitLabelAtPos`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `getCurrentSectionOnly`, `setTLS`, `emitLabelAtPos` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 53-65
```cpp
  const MCSectionWasm &Section =
      static_cast<const MCSectionWasm &>(*getCurrentSectionOnly());
  if (Section.getSegmentFlags() & wasm::WASM_SEG_FLAG_TLS)
    Symbol->setTLS();
}

void MCWasmStreamer::changeSection(MCSection *Section, uint32_t Subsection) {
  MCAssembler &Asm = getAssembler();
  auto *SectionWasm = static_cast<const MCSectionWasm *>(Section);
  const MCSymbol *Grp = SectionWasm->getGroup();
  if (Grp)
    Asm.registerSymbol(*Grp);

```
- **EN**: Implements logic around `getCurrentSectionOnly`, `setTLS`, `changeSection`, `getAssembler`, and 2 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `getCurrentSectionOnly`, `setTLS`, `changeSection`, `getAssembler`, and 2 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 66-73
```cpp
  this->MCObjectStreamer::changeSection(Section, Subsection);
  Asm.registerSymbol(*Section->getBeginSymbol());
}

bool MCWasmStreamer::emitSymbolAttribute(MCSymbol *S, MCSymbolAttr Attribute) {
  assert(Attribute != MCSA_IndirectSymbol && "indirect symbols not supported");
  auto *Symbol = static_cast<MCSymbolWasm *>(S);

```
- **EN**: Implements logic around `changeSection`, `registerSymbol`, `emitSymbolAttribute`, `assert`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `changeSection`, `registerSymbol`, `emitSymbolAttribute`, `assert` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 74-87
```cpp
  // Adding a symbol attribute always introduces the symbol; note that an
  // important side effect of calling registerSymbol here is to register the
  // symbol with the assembler.
  getAssembler().registerSymbol(*Symbol);

  switch (Attribute) {
  case MCSA_LazyReference:
  case MCSA_Reference:
  case MCSA_SymbolResolver:
  case MCSA_PrivateExtern:
  case MCSA_WeakDefinition:
  case MCSA_WeakDefAutoPrivate:
  case MCSA_Invalid:
  case MCSA_IndirectSymbol:
```
- **EN**: Implements logic around `getAssembler`; this block handles relocation, fixup, or symbol-resolution work; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `getAssembler` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 88-95
```cpp
  case MCSA_Protected:
  case MCSA_Exported:
    return false;

  case MCSA_Hidden:
    Symbol->setHidden(true);
    break;

```
- **EN**: Implements logic around `setHidden`; this block updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `setHidden` 实现具体逻辑；这一段更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 96-105
```cpp
  case MCSA_Weak:
  case MCSA_WeakReference:
    Symbol->setWeak(true);
    Symbol->setExternal(true);
    break;

  case MCSA_Global:
    Symbol->setExternal(true);
    break;

```
- **EN**: Implements logic around `setWeak`, `setExternal`; this block updates MC section or symbol state.
- **CN**: 围绕 `setWeak`, `setExternal` 实现具体逻辑；这一段更新 MC 节区或符号状态。

### Lines 106-113
```cpp
  case MCSA_ELF_TypeFunction:
    Symbol->setType(wasm::WASM_SYMBOL_TYPE_FUNCTION);
    break;

  case MCSA_ELF_TypeTLS:
    Symbol->setTLS();
    break;

```
- **EN**: Implements logic around `setType`, `setTLS`; this block updates MC section or symbol state.
- **CN**: 围绕 `setType`, `setTLS` 实现具体逻辑；这一段更新 MC 节区或符号状态。

### Lines 114-121
```cpp
  case MCSA_ELF_TypeObject:
  case MCSA_Cold:
    break;

  case MCSA_NoDeadStrip:
    Symbol->setNoStrip();
    break;

```
- **EN**: Implements logic around `setNoStrip`; this block updates MC section or symbol state.
- **CN**: 围绕 `setNoStrip` 实现具体逻辑；这一段更新 MC 节区或符号状态。

### Lines 122-130
```cpp
  default:
    // unrecognized directive
    llvm_unreachable("unexpected MCSymbolAttr");
    return false;
  }

  return true;
}

```
- **EN**: Implements logic around `llvm_unreachable`; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `llvm_unreachable` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 131-137
```cpp
void MCWasmStreamer::emitCommonSymbol(MCSymbol *S, uint64_t Size,
                                      Align ByteAlignment) {
  getContext().reportError(getStartTokLoc(),
                           "common symbols are not yet implemented for Wasm: " +
                               S->getName());
}

```
- **EN**: Implements logic around `emitCommonSymbol`, `getContext`, `getName`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `emitCommonSymbol`, `getContext`, `getName` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 138-149
```cpp
void MCWasmStreamer::emitELFSize(MCSymbol *Symbol, const MCExpr *Value) {
  static_cast<MCSymbolWasm *>(Symbol)->setSize(Value);
}

void MCWasmStreamer::emitLocalCommonSymbol(MCSymbol *S, uint64_t Size,
                                           Align ByteAlignment) {
  getContext().reportError(getStartTokLoc(),
                           "local common symbols are not yet implemented "
                           "for Wasm: " +
                               S->getName());
}

```
- **EN**: Implements logic around `emitELFSize`, `setSize`, `emitLocalCommonSymbol`, `getContext`, and 1 more symbols; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `emitELFSize`, `setSize`, `emitLocalCommonSymbol`, `getContext`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 150-157
```cpp
void MCWasmStreamer::emitIdent(StringRef IdentString) {
  // TODO(sbc): Add the ident section once we support mergable strings
  // sections in the object format
}

void MCWasmStreamer::finishImpl() {
  emitFrames();

```
- **EN**: Implements logic around `emitIdent`, `finishImpl`, `emitFrames`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `emitIdent`, `finishImpl`, `emitFrames` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 158-168
```cpp
  this->MCObjectStreamer::finishImpl();
}

MCStreamer *llvm::createWasmStreamer(MCContext &Context,
                                     std::unique_ptr<MCAsmBackend> &&MAB,
                                     std::unique_ptr<MCObjectWriter> &&OW,
                                     std::unique_ptr<MCCodeEmitter> &&CE) {
  MCWasmStreamer *S =
      new MCWasmStreamer(Context, std::move(MAB), std::move(OW), std::move(CE));
  return S;
}
```
- **EN**: Implements logic around `finishImpl`, `createWasmStreamer`, `MCWasmStreamer`; this block drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `finishImpl`, `createWasmStreamer`, `MCWasmStreamer` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

## Key Concepts / 关键概念

- **Relocations and fixups / 重定位与 fixup**:
  - **EN**: Bridges symbolic references to concrete addresses or relocation records
  - **CN**: 把符号引用连接到具体地址或重定位记录
- **Directive handling / 伪指令处理**:
  - **EN**: Interprets assembler directives and maps them to streamer operations
  - **CN**: 解释汇编伪指令并把它们映射为 streamer 操作
- **Object layout / 目标文件布局**:
  - **EN**: Organizes fragments into sections and computes final offsets
  - **CN**: 把片段组织到节区中并计算最终偏移

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/MC/MCWasmStreamer.h`, `llvm/MC/MCAsmBackend.h`, `llvm/MC/MCAssembler.h`, `llvm/MC/MCCodeEmitter.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCFixup.h`, `llvm/MC/MCObjectStreamer.h`, `llvm/MC/MCSection.h`, `llvm/MC/MCSectionWasm.h`, `llvm/MC/MCSymbol.h`, `llvm/MC/MCSymbolWasm.h` ... (+2 more)
- **LLVM subsystems / LLVM 子系统**: MC, Support
