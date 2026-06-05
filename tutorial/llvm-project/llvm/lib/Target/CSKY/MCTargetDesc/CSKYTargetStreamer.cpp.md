# CSKYTargetStreamer.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/CSKY/MCTargetDesc/CSKYTargetStreamer.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Defines target-specific assembly streamer hooks for directives and custom MC output.
- 目的（中文）: 定义目标专用的汇编 streamer 钩子，用于处理指令伪操作和自定义 MC 输出。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- CSKYTargetStreamer.h - CSKY Target Streamer ----------*- C++ -*----===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: #include "CSKYTargetStreamer.h"
  10: #include "MCTargetDesc/CSKYMCAsmInfo.h"
  11: #include "llvm/CodeGen/MachineFrameInfo.h"
  12: #include "llvm/CodeGen/TargetSubtargetInfo.h"
  13: #include "llvm/MC/MCContext.h"
  14: #include "llvm/MC/MCSectionELF.h"
  15: #include "llvm/Support/FormattedStream.h"
  16: 
  17: using namespace llvm;
  18: 
  19: //
  20: // ConstantPool implementation
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。

### Lines 21-40

```cpp
  21: //
  22: // Emit the contents of the constant pool using the provided streamer.
  23: void CSKYConstantPool::emitAll(MCStreamer &Streamer) {
  24:   if (Entries.empty())
  25:     return;
  26: 
  27:   if (CurrentSection != nullptr)
  28:     Streamer.switchSection(CurrentSection);
  29: 
  30:   Streamer.emitDataRegion(MCDR_DataRegion);
  31:   for (const ConstantPoolEntry &Entry : Entries) {
  32:     Streamer.emitCodeAlignment(
  33:         Align(Entry.Size),
  34:         Streamer.getContext().getSubtargetInfo()); // align naturally
  35:     Streamer.emitLabel(Entry.Label);
  36:     Streamer.emitValue(Entry.Value, Entry.Size, Entry.Loc);
  37:   }
  38:   Streamer.emitDataRegion(MCDR_DataRegionEnd);
  39:   Entries.clear();
  40: }
```

- EN: Function bodies or method definitions such as emitAll contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: emitAll 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 41-60

```cpp
  41: 
  42: const MCExpr *CSKYConstantPool::addEntry(MCStreamer &Streamer,
  43:                                          const MCExpr *Value, unsigned Size,
  44:                                          SMLoc Loc, const MCExpr *AdjustExpr) {
  45:   if (CurrentSection == nullptr)
  46:     CurrentSection = Streamer.getCurrentSectionOnly();
  47: 
  48:   auto &Context = Streamer.getContext();
  49: 
  50:   const MCConstantExpr *C = dyn_cast<MCConstantExpr>(Value);
  51: 
  52:   // Check if there is existing entry for the same constant. If so, reuse it.
  53:   auto Itr = C ? CachedEntries.find(C->getValue()) : CachedEntries.end();
  54:   if (Itr != CachedEntries.end())
  55:     return Itr->second;
  56: 
  57:   MCSymbol *CPEntryLabel = Context.createTempSymbol();
  58:   const auto SymRef = MCSymbolRefExpr::create(CPEntryLabel, Context);
  59: 
  60:   if (AdjustExpr) {
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 61-80

```cpp
  61:     auto *CSKYExpr = cast<MCSpecifierExpr>(Value);
  62: 
  63:     Value = MCBinaryExpr::createSub(AdjustExpr, SymRef, Context);
  64:     Value = MCBinaryExpr::createSub(CSKYExpr->getSubExpr(), Value, Context);
  65:     Value = MCSpecifierExpr::create(Value, CSKYExpr->getSpecifier(), Context);
  66:   }
  67: 
  68:   Entries.push_back(ConstantPoolEntry(CPEntryLabel, Value, Size, Loc));
  69: 
  70:   if (C)
  71:     CachedEntries[C->getValue()] = SymRef;
  72:   return SymRef;
  73: }
  74: 
  75: bool CSKYConstantPool::empty() { return Entries.empty(); }
  76: 
  77: void CSKYConstantPool::clearCache() {
  78:   CurrentSection = nullptr;
  79:   CachedEntries.clear();
  80: }
```

- EN: Function bodies or method definitions such as empty, clearCache contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: empty, clearCache 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 81-100

```cpp
  81: 
  82: CSKYTargetStreamer::CSKYTargetStreamer(MCStreamer &S)
  83:     : MCTargetStreamer(S), ConstantPool(new CSKYConstantPool()) {}
  84: 
  85: const MCExpr *
  86: CSKYTargetStreamer::addConstantPoolEntry(const MCExpr *Expr, SMLoc Loc,
  87:                                          const MCExpr *AdjustExpr) {
  88:   uint8_t ELFRefKind = CSKY::S_Invalid;
  89:   ConstantCounter++;
  90: 
  91:   const MCExpr *OrigExpr = Expr;
  92: 
  93:   if (auto *CE = dyn_cast<MCSpecifierExpr>(Expr)) {
  94:     Expr = CE->getSubExpr();
  95:     ELFRefKind = CE->getSpecifier();
  96:   }
  97: 
  98:   if (const MCSymbolRefExpr *SymExpr = dyn_cast<MCSymbolRefExpr>(Expr)) {
  99:     const MCSymbol *Sym = &SymExpr->getSymbol();
 100: 
```

- EN: Function bodies or method definitions such as MCTargetStreamer, addConstantPoolEntry contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: MCTargetStreamer, addConstantPoolEntry 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 101-120

```cpp
 101:     SymbolIndex Index = {Sym, ELFRefKind};
 102: 
 103:     if (ConstantMap.find(Index) == ConstantMap.end()) {
 104:       ConstantMap[Index] =
 105:           ConstantPool->addEntry(getStreamer(), OrigExpr, 4, Loc, AdjustExpr);
 106:     }
 107:     return ConstantMap[Index];
 108:   }
 109: 
 110:   return ConstantPool->addEntry(getStreamer(), Expr, 4, Loc, AdjustExpr);
 111: }
 112: 
 113: void CSKYTargetStreamer::emitCurrentConstantPool() {
 114:   ConstantPool->emitAll(Streamer);
 115:   ConstantPool->clearCache();
 116: }
 117: 
 118: // finish() - write out any non-empty assembler constant pools.
 119: void CSKYTargetStreamer::finish() {
 120:   if (ConstantCounter != 0) {
```

- EN: Function bodies or method definitions such as emitCurrentConstantPool, finish contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: emitCurrentConstantPool, finish 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 121-140

```cpp
 121:     ConstantPool->emitAll(Streamer);
 122:   }
 123: 
 124:   finishAttributeSection();
 125: }
 126: 
 127: void CSKYTargetStreamer::emitTargetAttributes(const MCSubtargetInfo &STI) {}
 128: 
 129: void CSKYTargetStreamer::emitAttribute(unsigned Attribute, unsigned Value) {}
 130: void CSKYTargetStreamer::emitTextAttribute(unsigned Attribute,
 131:                                            StringRef String) {}
 132: void CSKYTargetStreamer::finishAttributeSection() {}
 133: 
 134: void CSKYTargetAsmStreamer::emitAttribute(unsigned Attribute, unsigned Value) {
 135:   OS << "\t.csky_attribute\t" << Attribute << ", " << Twine(Value) << "\n";
 136: }
 137: 
 138: void CSKYTargetAsmStreamer::emitTextAttribute(unsigned Attribute,
 139:                                               StringRef String) {
 140:   OS << "\t.csky_attribute\t" << Attribute << ", \"" << String << "\"\n";
```

- EN: Function bodies or method definitions such as emitTargetAttributes, emitAttribute, emitTextAttribute contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: emitTargetAttributes, emitAttribute, emitTextAttribute 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 141-143

```cpp
 141: }
 142: 
 143: void CSKYTargetAsmStreamer::finishAttributeSection() {}
```

- EN: Function bodies or method definitions such as finishAttributeSection contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: finishAttributeSection 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

## Key Concepts / 关键概念

- MC streaming and emission / MC 流式输出
- CPU feature modelling / CPU 特性建模
- ELF object support / ELF 目标文件支持

## Dependencies / 依赖关系

- Direct includes / 直接包含: `CSKYTargetStreamer.h`, `MCTargetDesc/CSKYMCAsmInfo.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/TargetSubtargetInfo.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCSectionELF.h`, `llvm/Support/FormattedStream.h`
- LLVM subsystems / LLVM 子系统: LLVM MC
- Local companions / 本地配套文件: `CSKYTargetStreamer.h`
