# SystemZHLASMAsmStreamer.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/MCTargetDesc/SystemZHLASMAsmStreamer.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file connects the backend to LLVM MC layer descriptors for the SystemZ backend.
- **用途 (CN)**: 该文件用于 SystemZ 后端，负责将后端连接到 LLVM MC 层描述符。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //===- SystemZHLASMAsmStreamer.h - HLASM Assembly Text Output ---*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file declares the SystemZHLASMAsmStreamer class.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #ifndef LLVM_LIB_TARGET_SYSTEMZ_MCTARGETDESC_SYSTEMZHLASMASMSTREAMER_H
  14: #define LLVM_LIB_TARGET_SYSTEMZ_MCTARGETDESC_SYSTEMZHLASMASMSTREAMER_H
  15: 
  16: #include "llvm/ADT/SmallString.h"
  17: #include "llvm/ADT/StringRef.h"
  18: #include "llvm/MC/MCAsmBackend.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `SmallString.h`, `StringRef.h`, `MCAsmBackend.h` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `SmallString.h`, `StringRef.h`, `MCAsmBackend.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 19-36 / 第 19-36 行
```cpp
  19: #include "llvm/MC/MCAsmInfo.h"
  20: #include "llvm/MC/MCAssembler.h"
  21: #include "llvm/MC/MCCodeEmitter.h"
  22: #include "llvm/MC/MCContext.h"
  23: #include "llvm/MC/MCInst.h"
  24: #include "llvm/MC/MCInstPrinter.h"
  25: #include "llvm/MC/MCObjectWriter.h"
  26: #include "llvm/MC/MCStreamer.h"
  27: #include "llvm/MC/MCTargetOptions.h"
  28: #include "llvm/Support/FormattedStream.h"
  29: 
  30: namespace llvm {
  31: class MCSymbolGOFF;
  32: 
  33: class SystemZHLASMAsmStreamer final : public MCStreamer {
  34:   constexpr static size_t InstLimit = 80;
  35:   constexpr static size_t ContIndicatorColumn = 72;
  36:   constexpr static size_t ContStartColumn = 15;
```
- **EN**: It imports dependencies such as `MCAsmInfo.h`, `MCAssembler.h`, `MCCodeEmitter.h`, `MCContext.h`, `MCInst.h`, `MCInstPrinter.h` that expose the LLVM and target interfaces used in later logic. This block declares or refines TableGen records such as `MCSymbolGOFF`, `SystemZHLASMAsmStreamer`.
- **CN**: 它引入了 `MCAsmInfo.h`, `MCAssembler.h`, `MCCodeEmitter.h`, `MCContext.h`, `MCInst.h`, `MCInstPrinter.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 该代码块声明或细化了 `MCSymbolGOFF`, `SystemZHLASMAsmStreamer` 等 TableGen 记录。

### Lines 37-54 / 第 37-54 行
```cpp
  37:   constexpr static size_t ContLen = ContIndicatorColumn - ContStartColumn;
  38:   std::unique_ptr<formatted_raw_ostream> FOSOwner;
  39:   formatted_raw_ostream &FOS;
  40:   std::string Str;
  41:   raw_string_ostream OS;
  42:   const MCAsmInfo *MAI;
  43:   std::unique_ptr<MCInstPrinter> InstPrinter;
  44:   std::unique_ptr<MCAssembler> Assembler;
  45:   SmallString<128> CommentToEmit;
  46:   raw_svector_ostream CommentStream;
  47:   raw_null_ostream NullStream;
  48:   bool IsVerboseAsm = false;
  49: 
  50: public:
  51:   SystemZHLASMAsmStreamer(MCContext &Context,
  52:                           std::unique_ptr<formatted_raw_ostream> os,
  53:                           std::unique_ptr<MCInstPrinter> printer,
  54:                           std::unique_ptr<MCCodeEmitter> emitter,
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 55-72 / 第 55-72 行
```cpp
  55:                           std::unique_ptr<MCAsmBackend> asmbackend)
  56:       : MCStreamer(Context), FOSOwner(std::move(os)), FOS(*FOSOwner), OS(Str),
  57:         MAI(&Context.getAsmInfo()), InstPrinter(std::move(printer)),
  58:         Assembler(std::make_unique<MCAssembler>(
  59:             Context, std::move(asmbackend), std::move(emitter),
  60:             (asmbackend) ? asmbackend->createObjectWriter(NullStream)
  61:                          : nullptr)),
  62:         CommentStream(CommentToEmit) {
  63:     assert(InstPrinter);
  64:     if (Assembler->getBackendPtr())
  65:       setAllowAutoPadding(Assembler->getBackend().allowAutoPadding());
  66: 
  67:     Context.setUseNamesOnTempLabels(true);
  68:     IsVerboseAsm = Context.getTargetOptions().AsmVerbose;
  69:     if (IsVerboseAsm)
  70:       InstPrinter->setCommentStream(CommentStream);
  71:   }
  72: 
```
- **EN**: The range implements or declares functions including `MCStreamer`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `MCStreamer` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 73-90 / 第 73-90 行
```cpp
  73:   MCAssembler &getAssembler() { return *Assembler; }
  74: 
  75:   void EmitEOL();
  76:   void EmitComment();
  77: 
  78:   /// Add a comment that can be emitted to the generated .s file to make the
  79:   /// output of the compiler more readable. This only affects the MCAsmStreamer
  80:   /// and only when verbose assembly output is enabled.
  81:   void AddComment(const Twine &T, bool EOL = true) override;
  82: 
  83:   void emitBytes(StringRef Data) override;
  84: 
  85:   void emitAlignmentDS(uint64_t ByteAlignment, std::optional<int64_t> Value,
  86:                        unsigned ValueSize, unsigned MaxBytesToEmit);
  87:   void emitValueToAlignment(Align Alignment, int64_t Fill, uint8_t FillLen,
  88:                             unsigned MaxBytesToEmit) override;
  89: 
  90:   void emitCodeAlignment(Align Alignment, const MCSubtargetInfo *STI,
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 91-108 / 第 91-108 行
```cpp
  91:                          unsigned MaxBytesToEmit = 0) override;
  92: 
  93:   /// Return true if this streamer supports verbose assembly at all.
  94:   bool isVerboseAsm() const override { return IsVerboseAsm; }
  95: 
  96:   /// Do we support EmitRawText?
  97:   bool hasRawTextSupport() const override { return true; }
  98: 
  99:   /// @name MCStreamer Interface
 100:   /// @{
 101:   void visitUsedSymbol(const MCSymbol &Sym) override;
 102: 
 103:   void changeSection(MCSection *Section, uint32_t Subsection) override;
 104: 
 105:   void emitInstruction(const MCInst &Inst, const MCSubtargetInfo &STI) override;
 106:   void emitLabel(MCSymbol *Symbol, SMLoc Loc) override;
 107:   bool emitSymbolAttribute(MCSymbol *Symbol, MCSymbolAttr Attribute) override;
 108: 
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 109-126 / 第 109-126 行
```cpp
 109:   void emitCommonSymbol(MCSymbol *Symbol, uint64_t Size,
 110:                         Align ByteAlignment) override {}
 111: 
 112:   void emitZerofill(MCSection *Section, MCSymbol *Symbol = nullptr,
 113:                     uint64_t Size = 0, Align ByteAlignment = Align(1),
 114:                     SMLoc Loc = SMLoc()) override {}
 115:   void emitRawTextImpl(StringRef String) override;
 116:   void emitValueImpl(const MCExpr *Value, unsigned Size, SMLoc Loc) override;
 117: 
 118:   void emitHLASMValueImpl(const MCExpr *Value, unsigned Size,
 119:                           bool Parens = false);
 120:   /// @}
 121: 
 122:   void finishImpl() override;
 123: };
 124: } // namespace llvm
 125: 
 126: #endif // LLVM_LIB_TARGET_SYSTEMZ_MCTARGETDESC_SYSTEMZHLASMASMSTREAMER_H
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **MC layer**: Uses LLVM MC structures for encoding, printing, parsing, or disassembly. / 使用 LLVM MC 结构进行编码、打印、解析或反汇编。
- **MC streaming**: Emits sections, symbols, expressions, and encoded bytes to object or assembly output. / 向目标文件或汇编输出节、符号、表达式和编码字节。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。

## Dependencies / 依赖关系
- `llvm/ADT/SmallString.h`
- `llvm/ADT/StringRef.h`
- `llvm/MC/MCAsmBackend.h`
- `llvm/MC/MCAsmInfo.h`
- `llvm/MC/MCAssembler.h`
- `llvm/MC/MCCodeEmitter.h`
- `llvm/MC/MCContext.h`
- `llvm/MC/MCInst.h`
- `llvm/MC/MCInstPrinter.h`
- `llvm/MC/MCObjectWriter.h`
- `llvm/MC/MCStreamer.h`
- `llvm/MC/MCTargetOptions.h`
- `llvm/Support/FormattedStream.h`
