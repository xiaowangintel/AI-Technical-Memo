# SystemZTargetStreamer.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/MCTargetDesc/SystemZTargetStreamer.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file connects the backend to LLVM MC layer descriptors for the SystemZ backend.
- **用途 (CN)**: 该文件用于 SystemZ 后端，负责将后端连接到 LLVM MC 层描述符。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //=- SystemZTargetStreamer.h - SystemZ Target Streamer ----------*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: #ifndef LLVM_LIB_TARGET_SYSTEMZ_SYSTEMZTARGETSTREAMER_H
  10: #define LLVM_LIB_TARGET_SYSTEMZ_SYSTEMZTARGETSTREAMER_H
  11: 
  12: #include "llvm/ADT/StringRef.h"
  13: #include "llvm/MC/MCContext.h"
  14: #include "llvm/MC/MCExpr.h"
  15: #include "llvm/MC/MCInst.h"
  16: #include "llvm/MC/MCSectionGOFF.h"
  17: #include "llvm/MC/MCStreamer.h"
  18: #include "llvm/MC/MCSymbol.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `StringRef.h`, `MCContext.h`, `MCExpr.h`, `MCInst.h`, `MCSectionGOFF.h`, `MCStreamer.h` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `StringRef.h`, `MCContext.h`, `MCExpr.h`, `MCInst.h`, `MCSectionGOFF.h`, `MCStreamer.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 19-36 / 第 19-36 行
```cpp
  19: #include "llvm/MC/MCSymbolGOFF.h"
  20: #include "llvm/Support/FormattedStream.h"
  21: #include <map>
  22: #include <utility>
  23: 
  24: namespace llvm {
  25: class MCGOFFStreamer;
  26: class SystemZHLASMAsmStreamer;
  27: 
  28: class SystemZTargetStreamer : public MCTargetStreamer {
  29: public:
  30:   SystemZTargetStreamer(MCStreamer &S) : MCTargetStreamer(S) {}
  31: 
  32:   typedef std::pair<MCInst, const MCSubtargetInfo *> MCInstSTIPair;
  33:   struct CmpMCInst {
  34:     bool operator()(const MCInstSTIPair &MCI_STI_A,
  35:                     const MCInstSTIPair &MCI_STI_B) const {
  36:       if (MCI_STI_A.second != MCI_STI_B.second)
```
- **EN**: It imports dependencies such as `MCSymbolGOFF.h`, `FormattedStream.h`, `map`, `utility` that expose the LLVM and target interfaces used in later logic. This block declares or refines TableGen records such as `MCGOFFStreamer`, `SystemZHLASMAsmStreamer`, `SystemZTargetStreamer`. The range implements or declares functions including `SystemZTargetStreamer`, `operator`.
- **CN**: 它引入了 `MCSymbolGOFF.h`, `FormattedStream.h`, `map`, `utility` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 该代码块声明或细化了 `MCGOFFStreamer`, `SystemZHLASMAsmStreamer`, `SystemZTargetStreamer` 等 TableGen 记录。 这一段实现或声明了 `SystemZTargetStreamer`, `operator` 等函数。

### Lines 37-54 / 第 37-54 行
```cpp
  37:         return uintptr_t(MCI_STI_A.second) < uintptr_t(MCI_STI_B.second);
  38:       const MCInst &A = MCI_STI_A.first;
  39:       const MCInst &B = MCI_STI_B.first;
  40:       assert(A.getNumOperands() == B.getNumOperands() &&
  41:              A.getNumOperands() == 5 && A.getOperand(2).getImm() == 1 &&
  42:              B.getOperand(2).getImm() == 1 && "Unexpected EXRL target MCInst");
  43:       if (A.getOpcode() != B.getOpcode())
  44:         return A.getOpcode() < B.getOpcode();
  45:       if (A.getOperand(0).getReg() != B.getOperand(0).getReg())
  46:         return A.getOperand(0).getReg() < B.getOperand(0).getReg();
  47:       if (A.getOperand(1).getImm() != B.getOperand(1).getImm())
  48:         return A.getOperand(1).getImm() < B.getOperand(1).getImm();
  49:       if (A.getOperand(3).getReg() != B.getOperand(3).getReg())
  50:         return A.getOperand(3).getReg() < B.getOperand(3).getReg();
  51:       if (A.getOperand(4).getImm() != B.getOperand(4).getImm())
  52:         return A.getOperand(4).getImm() < B.getOperand(4).getImm();
  53:       return false;
  54:     }
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 55-72 / 第 55-72 行
```cpp
  55:   };
  56:   typedef std::map<MCInstSTIPair, MCSymbol *, CmpMCInst> EXRLT2SymMap;
  57:   EXRLT2SymMap EXRLTargets2Sym;
  58: 
  59:   void emitConstantPools() override;
  60: 
  61:   virtual void emitMachine(StringRef CPUOrCommand) {};
  62: 
  63:   virtual void emitExternalName(MCSymbol *Sym, StringRef Name) {}
  64:   virtual void emitExternalName(MCSection *Sec, StringRef Name) {}
  65: 
  66:   virtual const MCExpr *createWordDiffExpr(MCContext &Ctx, const MCSymbol *Hi,
  67:                                            const MCSymbol *Lo) {
  68:     return nullptr;
  69:   }
  70: 
  71:   virtual void emitADA(MCSymbol *Sym, MCSection *Section) {}
  72: };
```
- **EN**: The range implements or declares functions including `emitMachine`, `emitExternalName`, `emitADA`.
- **CN**: 这一段实现或声明了 `emitMachine`, `emitExternalName`, `emitADA` 等函数。

### Lines 73-90 / 第 73-90 行
```cpp
  73: 
  74: class SystemZTargetGOFFStreamer : public SystemZTargetStreamer {
  75: public:
  76:   SystemZTargetGOFFStreamer(MCStreamer &S) : SystemZTargetStreamer(S) {}
  77:   const MCExpr *createWordDiffExpr(MCContext &Ctx, const MCSymbol *Hi,
  78:                                    const MCSymbol *Lo) override;
  79:   virtual void emitExternalName(MCSymbol *Sym, StringRef Name) override {
  80:     static_cast<MCSymbolGOFF *>(Sym)->setExternalName(Name);
  81:   }
  82:   virtual void emitExternalName(MCSection *Sec, StringRef Name) override {
  83:     static_cast<MCSectionGOFF *>(Sec)->setExternalName(Name);
  84:   }
  85:   void emitADA(MCSymbol *Sym, MCSection *Section) override {
  86:     static_cast<MCSymbolGOFF *>(Sym)->setADA(
  87:         static_cast<MCSectionGOFF *>(Section));
  88:   }
  89: };
  90: 
```
- **EN**: This block declares or refines TableGen records such as `SystemZTargetGOFFStreamer`. The range implements or declares functions including `SystemZTargetGOFFStreamer`.
- **CN**: 该代码块声明或细化了 `SystemZTargetGOFFStreamer` 等 TableGen 记录。 这一段实现或声明了 `SystemZTargetGOFFStreamer` 等函数。

### Lines 91-108 / 第 91-108 行
```cpp
  91: class SystemZTargetHLASMStreamer : public SystemZTargetStreamer {
  92:   formatted_raw_ostream &OS;
  93: 
  94: public:
  95:   SystemZTargetHLASMStreamer(MCStreamer &S, formatted_raw_ostream &OS)
  96:       : SystemZTargetStreamer(S), OS(OS) {}
  97:   SystemZHLASMAsmStreamer &getHLASMStreamer();
  98:   const MCExpr *createWordDiffExpr(MCContext &Ctx, const MCSymbol *Hi,
  99:                                    const MCSymbol *Lo) override;
 100:   virtual void emitExternalName(MCSymbol *Sym, StringRef Name) override {
 101:     static_cast<MCSymbolGOFF *>(Sym)->setExternalName(Name);
 102:   }
 103:   virtual void emitExternalName(MCSection *Sec, StringRef Name) override {
 104:     static_cast<MCSectionGOFF *>(Sec)->setExternalName(Name);
 105:   }
 106:   void emitADA(MCSymbol *Sym, MCSection *Section) override {
 107:     static_cast<MCSymbolGOFF *>(Sym)->setADA(
 108:         static_cast<MCSectionGOFF *>(Section));
```
- **EN**: This block declares or refines TableGen records such as `SystemZTargetHLASMStreamer`. The range implements or declares functions including `SystemZTargetHLASMStreamer`.
- **CN**: 该代码块声明或细化了 `SystemZTargetHLASMStreamer` 等 TableGen 记录。 这一段实现或声明了 `SystemZTargetHLASMStreamer` 等函数。

### Lines 109-126 / 第 109-126 行
```cpp
 109:   }
 110: };
 111: 
 112: class SystemZTargetELFStreamer : public SystemZTargetStreamer {
 113: public:
 114:   SystemZTargetELFStreamer(MCStreamer &S) : SystemZTargetStreamer(S) {}
 115:   void emitMachine(StringRef CPUOrCommand) override {}
 116: };
 117: 
 118: class SystemZTargetGNUStreamer : public SystemZTargetStreamer {
 119:   formatted_raw_ostream &OS;
 120: 
 121: public:
 122:   SystemZTargetGNUStreamer(MCStreamer &S, formatted_raw_ostream &OS)
 123:       : SystemZTargetStreamer(S), OS(OS) {}
 124:   void emitMachine(StringRef CPUOrCommand) override {
 125:     OS << "\t.machine " << CPUOrCommand << "\n";
 126:   }
```
- **EN**: This block declares or refines TableGen records such as `SystemZTargetELFStreamer`, `SystemZTargetGNUStreamer`. The range implements or declares functions including `SystemZTargetELFStreamer`, `SystemZTargetGNUStreamer`.
- **CN**: 该代码块声明或细化了 `SystemZTargetELFStreamer`, `SystemZTargetGNUStreamer` 等 TableGen 记录。 这一段实现或声明了 `SystemZTargetELFStreamer`, `SystemZTargetGNUStreamer` 等函数。

### Lines 127-131 / 第 127-131 行
```cpp
 127: };
 128: 
 129: } // end namespace llvm
 130: 
 131: #endif // LLVM_LIB_TARGET_SYSTEMZ_SYSTEMZTARGETSTREAMER_H
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **MC layer**: Uses LLVM MC structures for encoding, printing, parsing, or disassembly. / 使用 LLVM MC 结构进行编码、打印、解析或反汇编。
- **MC streaming**: Emits sections, symbols, expressions, and encoded bytes to object or assembly output. / 向目标文件或汇编输出节、符号、表达式和编码字节。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。

## Dependencies / 依赖关系
- `llvm/ADT/StringRef.h`
- `llvm/MC/MCContext.h`
- `llvm/MC/MCExpr.h`
- `llvm/MC/MCInst.h`
- `llvm/MC/MCSectionGOFF.h`
- `llvm/MC/MCStreamer.h`
- `llvm/MC/MCSymbol.h`
- `llvm/MC/MCSymbolGOFF.h`
- `llvm/Support/FormattedStream.h`
- `map`
- `utility`
