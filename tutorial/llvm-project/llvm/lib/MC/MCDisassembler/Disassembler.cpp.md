# Disassembler.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/MC/MCDisassembler/Disassembler.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements Disassembler Public C Interface.
  - **CN**: 实现 LLVM MC 反汇编支持，把机器码字节解码为 MC 指令与符号化操作数。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- lib/MC/Disassembler.cpp - Disassembler Public C Interface ---------===//
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

#include "Disassembler.h"
#include "llvm-c/Disassembler.h"
#include "llvm/ADT/ArrayRef.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCDisassembler/MCDisassembler.h"
#include "llvm/MC/MCDisassembler/MCRelocationInfo.h"
#include "llvm/MC/MCDisassembler/MCSymbolizer.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCInstPrinter.h"
#include "llvm/MC/MCInstrInfo.h"
#include "llvm/MC/MCRegisterInfo.h"
```
- **EN**: Pulls in the headers needed for this implementation, including `Disassembler.h`, `llvm-c/Disassembler.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/SmallVector.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `Disassembler.h`, `llvm-c/Disassembler.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/SmallVector.h`。

### Lines 22-32
```cpp
#include "llvm/MC/MCSchedule.h"
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/MC/MCTargetOptions.h"
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/FormattedStream.h"
#include "llvm/Support/raw_ostream.h"
#include "llvm/TargetParser/Triple.h"
#include <cassert>
#include <cstring>

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/MC/MCSchedule.h`, `llvm/MC/MCSubtargetInfo.h`, `llvm/MC/MCTargetOptions.h`, `llvm/MC/TargetRegistry.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/MC/MCSchedule.h`, `llvm/MC/MCSubtargetInfo.h`, `llvm/MC/MCTargetOptions.h`, `llvm/MC/TargetRegistry.h`。

### Lines 33-46
```cpp
using namespace llvm;

// LLVMCreateDisasm() creates a disassembler for the TripleName.  Symbolic
// disassembly is supported by passing a block of information in the DisInfo
// parameter and specifying the TagType and callback functions as described in
// the header llvm-c/Disassembler.h .  The pointer to the block and the
// functions can all be passed as NULL.  If successful, this returns a
// disassembler context.  If not, it returns NULL.
//
LLVMDisasmContextRef
LLVMCreateDisasmCPUFeatures(const char *TT, const char *CPU,
                            const char *Features, void *DisInfo, int TagType,
                            LLVMOpInfoCallback GetOpInfo,
                            LLVMSymbolLookupCallback SymbolLookUp) {
```
- **EN**: Introduces declarations for `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 47-54
```cpp
  Triple TheTriple(TT);

  // Get the target.
  std::string Error;
  const Target *TheTarget = TargetRegistry::lookupTarget(TheTriple, Error);
  if (!TheTarget)
    return nullptr;

```
- **EN**: Implements logic around `TheTriple`, `lookupTarget`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `TheTriple`, `lookupTarget` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 55-66
```cpp
  std::unique_ptr<const MCRegisterInfo> MRI(
      TheTarget->createMCRegInfo(TheTriple));
  if (!MRI)
    return nullptr;

  MCTargetOptions MCOptions;
  // Get the assembler info needed to setup the MCContext.
  std::unique_ptr<const MCAsmInfo> MAI(
      TheTarget->createMCAsmInfo(*MRI, TheTriple, MCOptions));
  if (!MAI)
    return nullptr;

```
- **EN**: Implements logic around `MRI`, `createMCRegInfo`, `MAI`, `createMCAsmInfo`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `MRI`, `createMCRegInfo`, `MAI`, `createMCAsmInfo` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 67-75
```cpp
  std::unique_ptr<const MCInstrInfo> MII(TheTarget->createMCInstrInfo());
  if (!MII)
    return nullptr;

  std::unique_ptr<const MCSubtargetInfo> STI(
      TheTarget->createMCSubtargetInfo(TheTriple, CPU, Features));
  if (!STI)
    return nullptr;

```
- **EN**: Implements logic around `MII`, `STI`, `createMCSubtargetInfo`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `MII`, `STI`, `createMCSubtargetInfo` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 76-86
```cpp
  // Set up the MCContext for creating symbols and MCExpr's.
  std::unique_ptr<MCContext> Ctx(new MCContext(TheTriple, *MAI, *MRI, *STI));
  if (!Ctx)
    return nullptr;

  // Set up disassembler.
  std::unique_ptr<MCDisassembler> DisAsm(
      TheTarget->createMCDisassembler(*STI, *Ctx));
  if (!DisAsm)
    return nullptr;

```
- **EN**: Implements logic around `Ctx`, `DisAsm`, `createMCDisassembler`; this block decodes machine-code bytes into symbolic instruction form; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `Ctx`, `DisAsm`, `createMCDisassembler` 实现具体逻辑；这一段把机器码字节解码为符号化指令形式，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 87-96
```cpp
  std::unique_ptr<MCRelocationInfo> RelInfo(
      TheTarget->createMCRelocationInfo(TheTriple, *Ctx));
  if (!RelInfo)
    return nullptr;

  std::unique_ptr<MCSymbolizer> Symbolizer(
      TheTarget->createMCSymbolizer(TheTriple, GetOpInfo, SymbolLookUp, DisInfo,
                                    Ctx.get(), std::move(RelInfo)));
  DisAsm->setSymbolizer(std::move(Symbolizer));

```
- **EN**: Implements logic around `RelInfo`, `createMCRelocationInfo`, `Symbolizer`, `createMCSymbolizer`, and 2 more symbols; this block handles relocation, fixup, or symbol-resolution work; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `RelInfo`, `createMCRelocationInfo`, `Symbolizer`, `createMCSymbolizer`, and 2 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 97-103
```cpp
  // Set up the instruction printer.
  int AsmPrinterVariant = MAI->getAssemblerDialect();
  std::unique_ptr<MCInstPrinter> IP(TheTarget->createMCInstPrinter(
      Triple(TT), AsmPrinterVariant, *MAI, *MII, *MRI));
  if (!IP)
    return nullptr;

```
- **EN**: Implements logic around `getAssemblerDialect`, `IP`, `Triple`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getAssemblerDialect`, `IP`, `Triple` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 104-110
```cpp
  LLVMDisasmContext *DC = new LLVMDisasmContext(
      TT, DisInfo, TagType, GetOpInfo, SymbolLookUp, TheTarget, std::move(MAI),
      std::move(MRI), std::move(STI), std::move(MII), std::move(Ctx),
      std::move(DisAsm), std::move(IP));
  if (!DC)
    return nullptr;

```
- **EN**: Implements logic around `move`; this block updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `move` 实现具体逻辑；这一段更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 111-122
```cpp
  DC->setCPU(CPU);
  return DC;
}

LLVMDisasmContextRef
LLVMCreateDisasmCPU(const char *TT, const char *CPU, void *DisInfo, int TagType,
                    LLVMOpInfoCallback GetOpInfo,
                    LLVMSymbolLookupCallback SymbolLookUp) {
  return LLVMCreateDisasmCPUFeatures(TT, CPU, "", DisInfo, TagType, GetOpInfo,
                                     SymbolLookUp);
}

```
- **EN**: Implements logic around `setCPU`; this block updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `setCPU` 实现具体逻辑；这一段更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 123-129
```cpp
LLVMDisasmContextRef LLVMCreateDisasm(const char *TT, void *DisInfo,
                                      int TagType, LLVMOpInfoCallback GetOpInfo,
                                      LLVMSymbolLookupCallback SymbolLookUp) {
  return LLVMCreateDisasmCPUFeatures(TT, "", "", DisInfo, TagType, GetOpInfo,
                                     SymbolLookUp);
}

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 130-137
```cpp
//
// LLVMDisasmDispose() disposes of the disassembler specified by the context.
//
void LLVMDisasmDispose(LLVMDisasmContextRef DCR){
  LLVMDisasmContext *DC = static_cast<LLVMDisasmContext *>(DCR);
  delete DC;
}

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 138-151
```cpp
/// Emits the comments that are stored in \p DC comment stream.
/// Each comment in the comment stream must end with a newline.
static void emitComments(LLVMDisasmContext *DC,
                         formatted_raw_ostream &FormattedOS) {
  // Flush the stream before taking its content.
  StringRef Comments = DC->CommentsToEmit.str();
  // Get the default information for printing a comment.
  const MCAsmInfo *MAI = DC->getAsmInfo();
  StringRef CommentBegin = MAI->getCommentString();
  unsigned CommentColumn = MAI->getCommentColumn();
  bool IsFirst = true;
  while (!Comments.empty()) {
    if (!IsFirst)
      FormattedOS << '\n';
```
- **EN**: Implements logic around `emitComments`, `str`, `getAsmInfo`, `getCommentString`, and 1 more symbols.
- **CN**: 围绕 `emitComments`, `str`, `getAsmInfo`, `getCommentString`, and 1 more symbols 实现具体逻辑。

### Lines 152-161
```cpp
    // Emit a line of comments.
    FormattedOS.PadToColumn(CommentColumn);
    size_t Position = Comments.find('\n');
    FormattedOS << CommentBegin << ' ' << Comments.substr(0, Position);
    // Move after the newline character.
    Comments = Comments.substr(Position+1);
    IsFirst = false;
  }
  FormattedOS.flush();

```
- **EN**: Implements logic around `PadToColumn`, `find`, `substr`, `flush`.
- **CN**: 围绕 `PadToColumn`, `find`, `substr`, `flush` 实现具体逻辑。

### Lines 162-173
```cpp
  // Tell the comment stream that the vector changed underneath it.
  DC->CommentsToEmit.clear();
}

/// Emits latency information in DC->CommentStream for \p Inst, based
/// on the information available in \p DC.
static void emitLatency(LLVMDisasmContext *DC, const MCInst &Inst) {
  const MCSubtargetInfo *STI = DC->getSubtargetInfo();
  const MCInstrInfo *MCII = DC->getInstrInfo();
  const MCSchedModel &SCModel = STI->getSchedModel();
  int Latency = SCModel.computeInstrLatency(*STI, *MCII, Inst);

```
- **EN**: Implements logic around `clear`, `emitLatency`, `getSubtargetInfo`, `getInstrInfo`, and 2 more symbols.
- **CN**: 围绕 `clear`, `emitLatency`, `getSubtargetInfo`, `getInstrInfo`, and 2 more symbols 实现具体逻辑。

### Lines 174-180
```cpp
  // Report only interesting latencies.
  if (Latency < 2)
    return;

  DC->CommentStream << "Latency: " << Latency << '\n';
}

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 181-194
```cpp
//
// LLVMDisasmInstruction() disassembles a single instruction using the
// disassembler context specified in the parameter DC.  The bytes of the
// instruction are specified in the parameter Bytes, and contains at least
// BytesSize number of bytes.  The instruction is at the address specified by
// the PC parameter.  If a valid instruction can be disassembled its string is
// returned indirectly in OutString which whos size is specified in the
// parameter OutStringSize.  This function returns the number of bytes in the
// instruction or zero if there was no valid instruction.  If this function
// returns zero the caller will have to pick how many bytes they want to step
// over by printing a .byte, .long etc. to continue.
//
size_t LLVMDisasmInstruction(LLVMDisasmContextRef DCR, uint8_t *Bytes,
                             uint64_t BytesSize, uint64_t PC, char *OutString,
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 195-208
```cpp
                             size_t OutStringSize){
  LLVMDisasmContext *DC = static_cast<LLVMDisasmContext *>(DCR);
  // Wrap the pointer to the Bytes, BytesSize and PC in a MemoryObject.
  ArrayRef<uint8_t> Data(Bytes, BytesSize);

  uint64_t Size;
  MCInst Inst;
  const MCDisassembler *DisAsm = DC->getDisAsm();
  MCInstPrinter *IP = DC->getIP();
  MCDisassembler::DecodeStatus S;
  SmallVector<char, 64> InsnStr;
  raw_svector_ostream Annotations(InsnStr);
  S = DisAsm->getInstruction(Inst, Size, Data, PC, Annotations);
  switch (S) {
```
- **EN**: Implements logic around `Data`, `getDisAsm`, `getIP`, `Annotations`, and 1 more symbols; this block decodes machine-code bytes into symbolic instruction form; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `Data`, `getDisAsm`, `getIP`, `Annotations`, and 1 more symbols 实现具体逻辑；这一段把机器码字节解码为符号化指令形式，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 209-216
```cpp
  case MCDisassembler::Fail:
  case MCDisassembler::SoftFail:
    // FIXME: Do something different for soft failure modes?
    return 0;

  case MCDisassembler::Success: {
    StringRef AnnotationsStr = Annotations.str();

```
- **EN**: Implements logic around `str`; this block decodes machine-code bytes into symbolic instruction form; returns subsystem-specific computed results.
- **CN**: 围绕 `str` 实现具体逻辑；这一段把机器码字节解码为符号化指令形式，返回子系统相关的计算结果。

### Lines 217-225
```cpp
    SmallVector<char, 64> InsnStr;
    raw_svector_ostream OS(InsnStr);
    formatted_raw_ostream FormattedOS(OS);

    if (DC->getOptions() & LLVMDisassembler_Option_Color) {
      FormattedOS.enable_colors(true);
      IP->setUseColor(true);
    }

```
- **EN**: Implements logic around `OS`, `FormattedOS`, `enable_colors`, `setUseColor`; this block decodes machine-code bytes into symbolic instruction form.
- **CN**: 围绕 `OS`, `FormattedOS`, `enable_colors`, `setUseColor` 实现具体逻辑；这一段把机器码字节解码为符号化指令形式。

### Lines 226-233
```cpp
    IP->printInst(&Inst, PC, AnnotationsStr, *DC->getSubtargetInfo(),
                  FormattedOS);

    if (DC->getOptions() & LLVMDisassembler_Option_PrintLatency)
      emitLatency(DC, Inst);

    emitComments(DC, FormattedOS);

```
- **EN**: Implements logic around `printInst`, `emitLatency`, `emitComments`; this block decodes machine-code bytes into symbolic instruction form.
- **CN**: 围绕 `printInst`, `emitLatency`, `emitComments` 实现具体逻辑；这一段把机器码字节解码为符号化指令形式。

### Lines 234-244
```cpp
    assert(OutStringSize != 0 && "Output buffer cannot be zero size");
    size_t OutputSize = std::min(OutStringSize-1, InsnStr.size());
    std::memcpy(OutString, InsnStr.data(), OutputSize);
    OutString[OutputSize] = '\0'; // Terminate string.

    return Size;
  }
  }
  llvm_unreachable("Invalid DecodeStatus!");
}

```
- **EN**: Implements logic around `assert`, `min`, `memcpy`, `llvm_unreachable`; this block decodes machine-code bytes into symbolic instruction form; returns subsystem-specific computed results.
- **CN**: 围绕 `assert`, `min`, `memcpy`, `llvm_unreachable` 实现具体逻辑；这一段把机器码字节解码为符号化指令形式，返回子系统相关的计算结果。

### Lines 245-258
```cpp
//
// LLVMSetDisasmOptions() sets the disassembler's options.  It returns 1 if it
// can set all the Options and 0 otherwise.
//
int LLVMSetDisasmOptions(LLVMDisasmContextRef DCR, uint64_t Options){
  if (Options & LLVMDisassembler_Option_UseMarkup){
      LLVMDisasmContext *DC = static_cast<LLVMDisasmContext *>(DCR);
      MCInstPrinter *IP = DC->getIP();
      IP->setUseMarkup(true);
      DC->addOptions(LLVMDisassembler_Option_UseMarkup);
      Options &= ~LLVMDisassembler_Option_UseMarkup;
  }
  if (Options & LLVMDisassembler_Option_PrintImmHex){
      LLVMDisasmContext *DC = static_cast<LLVMDisasmContext *>(DCR);
```
- **EN**: Implements logic around `getIP`, `setUseMarkup`, `addOptions`; this block decodes machine-code bytes into symbolic instruction form.
- **CN**: 围绕 `getIP`, `setUseMarkup`, `addOptions` 实现具体逻辑；这一段把机器码字节解码为符号化指令形式。

### Lines 259-272
```cpp
      MCInstPrinter *IP = DC->getIP();
      IP->setPrintImmHex(true);
      DC->addOptions(LLVMDisassembler_Option_PrintImmHex);
      Options &= ~LLVMDisassembler_Option_PrintImmHex;
  }
  if (Options & LLVMDisassembler_Option_AsmPrinterVariant){
      LLVMDisasmContext *DC = static_cast<LLVMDisasmContext *>(DCR);
      // Try to set up the new instruction printer.
      const MCAsmInfo *MAI = DC->getAsmInfo();
      const MCInstrInfo *MII = DC->getInstrInfo();
      const MCRegisterInfo *MRI = DC->getRegisterInfo();
      int AsmPrinterVariant = MAI->getAssemblerDialect();
      AsmPrinterVariant = AsmPrinterVariant == 0 ? 1 : 0;
      MCInstPrinter *IP = DC->getTarget()->createMCInstPrinter(
```
- **EN**: Implements logic around `getIP`, `setPrintImmHex`, `addOptions`, `getAsmInfo`, and 4 more symbols; this block decodes machine-code bytes into symbolic instruction form.
- **CN**: 围绕 `getIP`, `setPrintImmHex`, `addOptions`, `getAsmInfo`, and 4 more symbols 实现具体逻辑；这一段把机器码字节解码为符号化指令形式。

### Lines 273-286
```cpp
          Triple(DC->getTripleName()), AsmPrinterVariant, *MAI, *MII, *MRI);
      if (IP) {
        DC->setIP(IP);
        DC->addOptions(LLVMDisassembler_Option_AsmPrinterVariant);
        Options &= ~LLVMDisassembler_Option_AsmPrinterVariant;
      }
  }
  if (Options & LLVMDisassembler_Option_SetInstrComments) {
    LLVMDisasmContext *DC = static_cast<LLVMDisasmContext *>(DCR);
    MCInstPrinter *IP = DC->getIP();
    IP->setCommentStream(DC->CommentStream);
    DC->addOptions(LLVMDisassembler_Option_SetInstrComments);
    Options &= ~LLVMDisassembler_Option_SetInstrComments;
  }
```
- **EN**: Implements logic around `Triple`, `setIP`, `addOptions`, `getIP`, and 1 more symbols; this block decodes machine-code bytes into symbolic instruction form.
- **CN**: 围绕 `Triple`, `setIP`, `addOptions`, `getIP`, and 1 more symbols 实现具体逻辑；这一段把机器码字节解码为符号化指令形式。

### Lines 287-298
```cpp
  if (Options & LLVMDisassembler_Option_PrintLatency) {
    LLVMDisasmContext *DC = static_cast<LLVMDisasmContext *>(DCR);
    DC->addOptions(LLVMDisassembler_Option_PrintLatency);
    Options &= ~LLVMDisassembler_Option_PrintLatency;
  }
  if (Options & LLVMDisassembler_Option_Color) {
    LLVMDisasmContext *DC = static_cast<LLVMDisasmContext *>(DCR);
    DC->addOptions(LLVMDisassembler_Option_Color);
    Options &= ~LLVMDisassembler_Option_Color;
  }
  return (Options == 0);
}
```
- **EN**: Implements logic around `addOptions`; this block decodes machine-code bytes into symbolic instruction form; returns subsystem-specific computed results.
- **CN**: 围绕 `addOptions` 实现具体逻辑；这一段把机器码字节解码为符号化指令形式，返回子系统相关的计算结果。

## Key Concepts / 关键概念

- **Disassembly / 反汇编**:
  - **EN**: Decodes raw bytes into MCInst objects and tries to recover symbolic references
  - **CN**: 将原始字节解码为 MCInst，并尝试恢复符号引用
- **Relocations and fixups / 重定位与 fixup**:
  - **EN**: Bridges symbolic references to concrete addresses or relocation records
  - **CN**: 把符号引用连接到具体地址或重定位记录
- **Directive handling / 伪指令处理**:
  - **EN**: Interprets assembler directives and maps them to streamer operations
  - **CN**: 解释汇编伪指令并把它们映射为 streamer 操作
- **MC instruction model / MC 指令模型**:
  - **EN**: Uses MCInst/MCOperand abstractions to represent machine instructions generically
  - **CN**: 使用 MCInst/MCOperand 抽象以通用方式表示机器指令

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `Disassembler.h`, `llvm-c/Disassembler.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/SmallVector.h`, `llvm/MC/MCAsmInfo.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCDisassembler/MCDisassembler.h`, `llvm/MC/MCDisassembler/MCRelocationInfo.h`, `llvm/MC/MCDisassembler/MCSymbolizer.h`, `llvm/MC/MCInst.h`, `llvm/MC/MCInstPrinter.h`, `llvm/MC/MCInstrInfo.h` ... (+11 more)
- **LLVM subsystems / LLVM 子系统**: MC, Support, Target/TargetParser
